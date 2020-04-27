---
title: "在k8s上跑jmeter"
layout: post
date: 2018-05-03 15:38
image: /img/images/markdown.jpg
headerImage: false
tag:
- k8s
- Jmeter
category: blog
author: dong

---

试玩了下在docker上跑jmeter, 通过influx和grafana做数据收集和展示


https://blog.kubernauts.io/load-testing-as-a-service-with-jmeter-on-kubernetes-fc5288bb0c8b

更新jmx文件里 influxdb的服务url  http://jmeter-influxdb:8086      应替换成influxdb的ip地址
更新dashboard中 $grafana_pod 配置的influxdb url
更新jmeter_master_configmap.yaml 中 取slave ip的 方法
    -R 172.17.0.14, 172.17.0.12
    -R getent ahostsv4 | grep jmeter-slave （fail）
运行服务 start.sh or 进容器中执行jmeter
* Creating summariser <summary>
* Created the tree successfully using cloudssky.jmx
* Starting the test @ Mon May 28 10:35:21 UTC 2018 (1527503721326)
* Waiting for possible Shutdown/StopTestNow/Heapdump message on port 4445
* summary +      5 in 00:01:38 =    0.1/s Avg: 19885 Min: 19267 Max: 20116 Err:     5 (100.00%) Active: 2 Started: 2 Finished: 0
* summary +     26 in 00:06:33 =    0.1/s Avg: 15398 Min:     0 Max: 20033 Err:    26 (100.00%) Active: 2 Started: 2 Finished: 0
* summary =     31 in 00:08:12 =    0.1/s Avg: 16121 Min:     0 Max: 20116 Err:    31 (100.00%)
验证granfana中数据， import json文件模版
    http://localhost:3001
验证influxdb中jmeter数据
    查容器kubectl get pods -n jmeter -o wide
    进容器kubectl exec -it -n jmeter influxdb-jmeter-5cbd7b8469-h7df5 /bin/bash
    进db  influx
            Connected to http://localhost:8086 version 1.5.2
            InfluxDB shell version: 1.5.2
        show databases;
        use jmeter;
        select * from jmeter
   在其它容器中可以 curl 或者wget  http://172.17.0.9:8086/query?q=show+databases  拿到数据 说明influxdb可以正常使用


http://localhost:3001/d/ltaas/jmeter-metric-template?orgId=1&from=now-6h&to=now    记得执行。kubectl port-forward -n jmeter jmeter-grafana-65dd5f6b79-22hz4 3001:3000

任何文件的更改 可以用kubectl replace -f jmeter_master_configmap.yaml -n jmeter

查详细信息例如容器ip 用 kubectl get pods --all-namespaces -o wide
