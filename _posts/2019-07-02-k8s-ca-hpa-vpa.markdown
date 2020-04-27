---
title: "kubernetes cluster autoscaler调研与hpa/vpa联动(二)"
layout: post
date: 2019-07-02 15:38
image: /img/images/markdown.jpg
headerImage: false
tag:
- k8s
category: blog
author: dong

---

## kubernetes cluster autoscaler调研与hpa/vpa联动

Kubernetes作为容器编排工具，应用部署在集群中，应用的负载本身是会随着时间动态发生变化的，为了更好的平衡资源使用率以及性能，kubernetes引入了autoscaler。可以根据应用负载的情况动态的扩缩容资源
Kubernetes的autoscaler分成两个层次:

* pod级别的扩容，包含横向扩容(HPA)以及纵向扩容(VPA),扩容容器可用的资源使用量。
* 集群级别的扩容，通过CA(Cluster Autoscaler)来控制扩容或者缩小集群中Node的数量。集群级别的扩容，通过CA(Cluster Autoscaler)来控制扩容或者缩小集群中Node的数量。

## 横向扩容(HPA)
扩容pod的副本数，通过容器的CPU以及Ｍemory来触发扩容或者缩容操作，并且支持自定义指标、多个指标甚至是外部的指标来作为触发扩容或者缩容操作的条件。
HPA的工作流
![hpa](/img/images/k8s/hpa.png 'hpa')

* HPA每隔30sec来检查指标的值
* 如果SPECIFIFD 阈值满足条件将会增加pod副本的数量
* HPA主要更新deployment/replication controller控制器对象的副本数
* Deployment/replication controller将会创建出来额外需要的pods


当使用HPA的时候需要注意的地方

* HPA检查周期为30s可以通过设置controller manager的horizontal-pod-autoscaler-sync-period参数来改变
* 默认的HPA相对指标公差为10%
* HPA在最后一次扩容事件后等待3分钟，以使指标稳定下来。可通过 - horizontal-pod-autoscaler-upscale-delay参数来配置
* HPA从最后一次缩容事件开始等待5分钟，以避免自动调节器抖动。可通过 - horizontal-pod-autoscaler-downscale-delay参数来配置
* 相对于replication controller而言，ｈｐａ更加适合与deployment一起配置工作


## 纵向扩容(VPA)

Vertical Pods Autoscaler（VPA）为现有pod分配更多（或更少）的CPU或内存。它可以适用于有状态和无状态的pod，但它主要是为有状态服务而构建的。但是，如果您希望实现最初为pod分配的资源的自动更正，则可以将其用于无状态容器。VPA还可以对OOM（内存不足）事件做出反应。VPA当前要求重新启动pod以更改已分配的CPU和内存。当VPA重新启动pod时，它会考虑pods分发预算（PDB）以确保始终具有所需的最小pod数。您可以设置VPA可以分配给任何pod的资源的最小值和最大值。例如，您可以将最大内存限制限制为不超过8 GB。当您知道当前节点无法为每个容器分配超过8 GB时，这尤其有用。

VPA还有一个名为VPA Recommender的有趣功能。它监视所有pod的历史资源使用情况和OOM事件，以建议request资源的新值。推荐器使用一些智能算法来根据历史指标计算内存和CPU值。它还提供了一个API，通过它可以获取pod描述符并提供建议的request值。

值得一提的是，VPA推荐者不会设置资源的limit值。这可能导致pod垄断节点内的资源。建议你在namespac级别设置一个“限制”值，以避免疯狂消耗内存或CPU

VPA工作流

![vpa](/img/images/k8s/vpa.png 'vpa')


VPA每隔１０ｓ检查指标的值
* 当阈值达到的时候，VPA尝试修改分配的memory和CPU
* VPA主要是更新deployment或者replication controller specs中的resources定义
* 当Pod重启的时候，所有请求的资源得到调整
使用VPA的时候需要注意点

* 如果不重新启动pod，则无法进行资源更改。到目前为止主要理性，就是这种变化可能会造成很多不稳定。因此，想要重新启动pod并根据新分配的资源进行调度。
* VPA和HPA尚未相互兼容，无法在相同的pod上运行。如果您在同一群集中使用它们，请确保将它们的范围分开。
* VPA仅根据观察到的过去和当前资源使用情况调整容器的资源请求。它没有设置资源限制。对于行为不端的应用程序而言，这可能会出现问题，这些应用程序开始使用越来越多的资源导致pod被Kubernetes杀死。


## 集群扩容(Cluster Autoscaler)

Cluster Autoscaler（CA）根据pending状态的pod来扩展您的群集节点。它会定期检查是否有pending状态的pod，如果需要更多资源并且扩展后的群集仍在用户提供的约束范围内，则会增加群集的大小。CA与云提供商接口以请求更多节点或释放空闲节点。它适用于GCP，AWS和Azure。版本1.0（GA）与kubernetes 1.8一起发布。

CA工作流
![ca](/img/images/k8s/ca.png 'ca')

* CA每隔10s检查以下pending状态的容器
* 如果存在因为资源不足导致pending状态的pod存在的时候，尝试创建一个或多个nodes
* 当node是被cloud provider所管理的，node将会被添加到集群中，成为ready的节点来创建pod
* Kubernetes调度器分配pending状态的pods到新的node节点上。如果一些pod仍然处于pending状态，这个过程将会继续，将会有更多的nodes添加到集群中

CA使用的时候注意事项

* Cluster Autoscaler确保群集中的所有pod都有一个可以运行的位置，无论是否有任何CPU负载。此外，它会尝试确保群集中没有不需要的节点。（资源）
* CA在大约30秒内实现了可扩展性需求。
* 在节点变为不需要之前，CA默认等待10分钟，然后再缩小节点。
* CA具有扩展器的概念。扩展器提供了不同的策略来选择要添加新节点的节点组。
* 负责任地使用"cluster-autoscaler.kubernetes.io/safe-to-evict"："true"。如果您设置了所有节点上的许多pod或足够的pod，则会失去很大的缩小灵活性。
* 使用PodDisruptionBudgets可以防止删除pod并使应用程序的一部分完全无法运行。

Kubernetes autoscalers交互一起怎么工作
如果您希望自动扩展您的Kubernetes集群，则需要在CA中使用pod层自动缩放器。他们彼此合作的方式相对简单，如下图所示。

![ca-hpa-vpa](/img/images/k8s/ca-hpa-vpa.png 'ca-hpa-vpa')

* HPA或者VPA来更新已经存在的pod副本数或者使用的resources
* 如果没有足够的节点在可伸缩性事件后运行pod，则CA会发现部分或全部已缩放的pod处于挂起状态的事实。
* CA扩容新的node到集群中
* Pods将会被调度到被新管理的node上

常见的错误
我在不同的论坛上看过，比如Kubernetes　slack和StackOverflow问题，由于一些事实导致的常见问题，许多DevOps错过了自动缩放器。
HPA和VPA依赖于指标和一些历史数据。如果您没有分配足够的资源，您的pod将被OOM杀死，并且永远不会有机会生成指标。在这种情况下，pods上的扩展器可能永远不会发生。扩容是时间敏感的操作。在用户遇到应用程序中的任何中断或崩溃之前，您希望您的pod和群集能够相当快地扩展。您应该考虑容器和群集扩展的平均时间。

最佳案例场景－４分钟

* 30秒 - 目标指标值更新：30-60秒
* 30秒 - HPA检查指标值：30秒 - >30秒 - HPA检查指标值：30秒 - >
* <2秒 - Pods创建之后进入pending状态<2秒　－Pods创建之后进入pending状态
* <2秒 - CA看到pending状态的pods，之后调用来创建node 1秒<2秒　－CA看到pending状态的pods，之后调用来创建node 1秒
* 3分钟 - cloud provider创建node，之后加入k8s之后等待node变成ready,上线是10分钟
(合理)最糟糕的情况 - 12分钟

* 60 秒 —目标指标值更新
* 30 秒 — HPA检查指标值
* < 2 秒 — Pods创建之后进入pending状态
* < 2 秒 —CA看到pending状态的pods，之后调用来创建node 1秒
* 10 分钟 — cloud provider创建ｎｏｄｅ，之后加入ｋ8s之后等待node变成ready,上线是10分钟

## 不要将云提供程序可伸缩性机制与CA混淆。CA在集群内部工作，而云提供商的可扩展性机制（例如AWS内部的ASG）基于节点分配工作。它不知道您的pod或应用程序正在发生什么。一起使用它们会使您的群集不稳定并且难以预测行为。
