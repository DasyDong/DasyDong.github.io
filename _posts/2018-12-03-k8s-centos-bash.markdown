---
title: "在centos 7上部署kubernetes-下篇自动化"
layout: post
date: 2018-12-03 15:38
image: /img/images/markdown.jpg
headerImage: false
tag:
- k8s
- linux
category: blog
author: dong

---

本文是在去年写了哪篇上文centos 7上部署kubernetes基础上, 介绍一个自动化部署方式， 直接贴一个git地址。 很详细
https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster

毕竟我也懒得写了 哈哈
此文只贴出一下我走一遍流程后遇到的几个FQA


# Setting up a distributed Kubernetes cluster along with Istio service mesh locally with Vagrant and VirtualBox

## FQA

### 1 因为mac机器上内存不足， 所以只选择搭建1台master，也当node节点.
（因为之前我用的vagrant centos7起的名字是centos， 所以这里也改掉了，可以跳过）
```bash
vim Vagrantfile
$num_instances = 1
node.vm.box = "centos"
vb.memory = "5120"
```

### 2 发现flannel 启动配置网络和虚机不一致，默认的是enp0s3, 虚机上用的是enp0s8 ip
```bash
systemctl status flanneld.service
```
发现flannel用的网络是enp0s3, ip是10.0.2.15,  和我们用的172.17.8.101 不符

ifconfig 查看， 应该要采用enp0s8
```
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fede:e0e  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:de:0e:0e  txqueuelen 1000  (Ethernet)
        RX packets 593284  bytes 783594015 (747.2 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 134800  bytes 8768373 (8.3 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.8.101  netmask 255.255.255.0  broadcast 172.17.8.255
        ether 08:00:27:a8:ff:6d  txqueuelen 1000  (Ethernet)
        RX packets 1741  bytes 262206 (256.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2079  bytes 3051240 (2.9 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
如果Node有多个网卡的话，参考flannel issues 39701，需要使用–iface参数指定集群主机内网网卡的名称，否则可能会出现dns无法解析。flanneld启动参数加上–iface=<iface-name>

```bash
vim /etc/sysconfig/flanneld
FLANNEL_OPTIONS="-iface=enp0s8"
systemctl start flanneld
```


### 3 解决 error creating overlay mount to /var/lib/docker/overlay2
在kubectl apply -f addon/dashboard/kubernetes-dashboard.yaml 时pod起不来
kubectl describe pod xxx -n kube-system 查看log发现报错
```
/usr/bin/docker-current: Error response from daemon: error creating overlay mount to /var/lib/docker/overlay2/7c5620a26d727cf7580849fe731f6b2349c1183f40e8279f864187e783f9ea90/merged: invalid argument
```
docker run -d -it docker.io/busybox sh 临时拉起一个最小的容器， 发现遇到同样的说， 说明是共性问题。

解决 error creating overlay mount to /var/lib/docker/overlay2  参考https://www.centos.bz/2018/06/%E8%A7%A3%E5%86%B3-error-creating-overlay-mount-to-var-lib-docker-overlay2/
```
1    systemctl stop docker
2    rm -rf /var/lib/docker  #会删除docker images
3    vi /etc/sysconfig/docker-storage
 指定  DOCKER_STORAGE_OPTIONS="--storage-driver overlay"
4   vi  /etc/sysconfig/docker
#OPTIONS='--selinux-enabled --log-driver=journald --signature-verification=false'
改为
OPTIONS='--log-driver=journald --signature-verification=false'
5  systemctl start docker
6 重新docker run -d -it docker.io/busybox sh

```
### 4 解决完上述问题3后， 发现node节点变成了 NotReady
systemctl service kubelet
发现出现如下报错：
```
kubelet: Failed to find subsystem mount for required subsystem: pids
```
发现git issue https://github.com/kubernetes/kubernetes/issues/79046 要在1.17的版本中才会修复

最后重新启动服务解决问题
```
systemctl daemon-reload
systemctl start docker
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-scheduler
systemctl start kubelet
systemctl start kube-proxy
 ```
 最后查看,节点Ready
 ```
 kubectl get nodes
 ```

 ### 5  本机访问dashboard https://172.17.8.101:8443/ 无服务

 关闭防火墙后可以访问：
 ```
 systemctl stop firewalld
 systemctl disable firewalld

 ```
