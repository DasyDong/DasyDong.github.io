---
title: "部署prometheus"
layout: post
date: 2020-03-03 15:38
image: /img/images/markdown.jpg
headerImage: false
tag:
- Tech
category: blog
author: dong
---

# 部署prometheus

本文是一步步从部署operator， prometheus， serviceaccount，alertmanager 来熟悉部署流程。

如果已经很熟悉了，可以通过[kube-prometheus](https://github.com/coreos/kube-prometheus) 完成快速一键部署

## 通过部署prometheus operator

### minikube准备

本机操作直接用的minikube

**重新安装最新版**

[install-minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

```
minikube stop

minikube delete ( 确保先先删掉)

brew install minikube

sudo mv minikube /usr/local/bin

通过minikube version查看是否成功

(可以通过which minikube  查找安装路径)

```

```
brew upgrade kubectl

安装完毕，通过 kubectl version 查看下是否安装成功, 看gitversion

Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.1", GitCommit:"7879fc12a63337efff607952a323df90cdc7a335", GitTreeState:"clean", BuildDate:"2020-04-10T21:53:40Z", GoVersion:"go1.14.2", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.3", GitCommit:"06ad960bfd03b39c8310aaf92d1e7c12ce618213", GitTreeState:"clean", BuildDate:"2020-02-11T18:07:13Z", GoVersion:"go1.13.6", Compiler:"gc", Platform:"linux/amd64"}

```


### 启动minikube
通常国内墙的问题还是配一下代理

```
export  HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
export no_proxy=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24,192.168.64.4
export no_proxy=$no_proxy,$(minikube ip)

curl https://k8s.gcr.io/robots.txt  验证
```

```
➜  ~ minikube start
😄  Darwin 10.13.6 上的 minikube v1.9.2
✨  根据现有的配置文件使用 hyperkit 驱动程序
👍  Kubernetes 1.18.0 现在可用了。如果您想升级，请指定 --kubernetes-version=1.18.0
👍  Starting control plane node m01 in cluster minikube
💾  Downloading Kubernetes v1.17.3 preload ...
🏃  Updating the running hyperkit "minikube" VM ...
🌐  找到的网络选项：
    ▪ HTTP_PROXY=http://proxy:8080
❗  您似乎正在使用代理，但您的 NO_PROXY 环境不包含 minikube IP (192.168.64.4)。如需了解详情，请参阅 https://minikube.sigs.k8s.io/docs/reference/networking/proxy/
    ▪ HTTPS_PROXY=http://proxy:8080
    ▪ NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24
    ▪ no_proxy=,192.168.64.4
🐳  正在 Docker 19.03.8 中准备 Kubernetes v1.17.3…
    ▪ env HTTP_PROXY=http://proxy:8080
    ▪ env HTTPS_PROXY=http://proxy:8080
    ▪ env NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,192.168.99.0/24,192.168.39.0/24
    ▪ env NO_PROXY=,192.168.64.4
🌟  Enabling addons: default-storageclass, storage-provisioner
❗  Enabling 'default-storageclass' returned an error: running callbacks: [Error making standard the default storage class: Error listing StorageClasses: Get "https://192.168.64.4:8443/apis/storage.k8s.io/v1/storageclasses": Service Unavailable]
🏄  完成！kubectl 已经配置至 "minikube"

```

### 部署Prometheus operator


在Kubernetes中安装Prometheus Operator非常简单，用户可以从以下地址中过去Prometheus Operator的源码：
```
git clone https://github.com/coreos/prometheus-operator.git
```
这里，我们为Promethues Operator创建一个单独的命名空间monitoring：
```
kubectl create namespace monitoring
```
由于需要对Prometheus Operator进行RBAC授权，而默认的bundle.yaml中使用了default命名空间，因此，在安装Prometheus Operator之前需要先替换一下bundle.yaml文件中所有namespace定义，由default修改为monitoring。 通过运行一下命令安装Prometheus Operator的Deployment实例：
```
$ kubectl -n monitoring apply -f bundle.yaml
clusterrolebinding.rbac.authorization.k8s.io/prometheus-operator created
clusterrole.rbac.authorization.k8s.io/prometheus-operator created
deployment.apps/prometheus-operator created
serviceaccount/prometheus-operator created
service/prometheus-operator created

```

### 创建Prometheus实例, alertmanager, example-app

具体步骤可以参考[prometheus-book](https://github.com/yunlzheng/prometheus-book/blob/master/operator/use-operator-manage-monitor.md)
[Demo代码](../codes/prometheus-operator-demo)

最后全部部署完成后，这些地址访问prometheus

http://127.0.0.1:8080/

http://127.0.0.1:9090/

http://127.0.0.1:9093/

pod应该有这些 kubectl get all

```
NAME                                      READY   STATUS    RESTARTS   AGE
pod/alertmanager-inst-0                   2/2     Running   0          16h
pod/alertmanager-inst-1                   2/2     Running   0          16h
pod/alertmanager-inst-2                   2/2     Running   0          15h
pod/example-app-5cc7db675f-5vmxc          1/1     Running   0          105m
pod/example-app-5cc7db675f-bv7s5          1/1     Running   0          21h
pod/example-app-5cc7db675f-tgnx5          1/1     Running   0          105m
pod/prometheus-inst-0                     3/3     Running   1          17h
pod/prometheus-operator-7676db7c9-ksrvw   1/1     Running   0          22h

NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated   ClusterIP   None            <none>        9093/TCP,9094/TCP,9094/UDP   16h
service/example-app             ClusterIP   10.98.190.166   <none>        8080/TCP                     22h
service/prometheus-operated     ClusterIP   None            <none>        9090/TCP                     22h
service/prometheus-operator     ClusterIP   None            <none>        8080/TCP                     22h

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/example-app           3/3     3            3           21h
deployment.apps/prometheus-operator   1/1     1            1           22h

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/example-app-5cc7db675f          3         3         3       21h
replicaset.apps/prometheus-operator-7676db7c9   1         1         1       22h

NAME                                 READY   AGE
statefulset.apps/alertmanager-inst   3/3     16h
statefulset.apps/prometheus-inst     1/1     22h
```

### FQA

通过--docker-env HTTP_PROXY=http://proxy:8080 安装， 还是遇到了报错
```
minikube start --docker-env HTTP_PROXY=http://proxy:8080  --docker-env HTTPS_PROXY=http://proxy:8080
```

在minikube start 后面加个参数 -v 8 --alsologtostderr  看详细日志
```
E0415 19:19:20.533152   33433 cache.go:63] save image to file "gcr.io/k8s-minikube/storage-provisioner:v1.8.1" -> "/Users/dongshu/.minikube/cache/images/gcr.io/k8s-minikube/storage-provisioner_v1.8.1" failed: nil image for gcr.io/k8s-minikube/storage-provisioner:v1.8.1: Get "https://gcr.io/v2/": dial tcp 64.233.189.82:443: i/o timeout
W0415 19:19:20.592870   33433 cache.go:118] unable to retrieve image: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout
I0415 19:19:20.592942   33433 cache.go:81] cache image "k8s.gcr.io/kube-proxy:v1.17.3" -> "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/kube-proxy_v1.17.3" took 5m8.804223272s
E0415 19:19:20.592963   33433 cache.go:63] save image to file "k8s.gcr.io/kube-proxy:v1.17.3" -> "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/kube-proxy_v1.17.3" failed: nil image for k8s.gcr.io/kube-proxy:v1.17.3: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout
W0415 19:19:20.596016   33433 cache.go:118] unable to retrieve image: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout
I0415 19:19:20.596068   33433 cache.go:81] cache image "k8s.gcr.io/pause:3.1" -> "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/pause_3.1" took 5m8.803245971s
E0415 19:19:20.596083   33433 cache.go:63] save image to file "k8s.gcr.io/pause:3.1" -> "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/pause_3.1" failed: nil image for k8s.gcr.io/pause:3.1: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout
W0415 19:19:20.596612   33433 exit.go:101] Failed to cache images: Caching images for kubeadm: caching images: caching image "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/kube-controller-manager_v1.17.3": nil image for k8s.gcr.io/kube-controller-manager:v1.17.3: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout

💣  缓存镜像时失败: Caching images for kubeadm: caching images: caching image "/Users/dongshu/.minikube/cache/images/k8s.gcr.io/kube-controller-manager_v1.17.3": nil image for k8s.gcr.io/kube-controller-manager:v1.17.3: Get "https://k8s.gcr.io/v2/": dial tcp 74.125.203.82:443: i/o timeout
```
