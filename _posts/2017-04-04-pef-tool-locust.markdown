---
title: "用locust做压测"
layout: post
date: 2017-04-04 22:48
image: /img/images/markdown.jpg
headerImage: false
tag:
- performance
- git
- python
category: blog
author: dong
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

### 目录
- [先说下Jmeter](#jmeter)
- [Locust介绍和常见问题](#locust)
- [locust-demo](#locust-demo)

<h2 id="yiny">引言</h2>
当我们做Web系统性能测试方案时，压力模拟工具的选择通常是一个绕不开的环节。对于大部分互联网公司的业务规模和测试资源投入，JMeter这个老牌开源性能测试工具能够满足大部分测试需求，它也可能是世面上书籍、博客教程丰富程度仅次于LoadRunner的性能测试工具。然而当我们的场景需要模拟的并发用户数以千为单位时，使用JMeter的成本越来越大，甚至超出我们掌握的资源。此时，我们开始寻找更低成本的方案，而Locust，为这样的方案带来了一种可能。

<h2 id="jmeter">使用Jmeter</h2>

<p>Jmeter是个纯java的开源的轻量级性能测试工具，功能强大。因为是轻量级的，与loadrunner相比，报告统计的相对较少。不过有jmeter的插件-<a href="http://jmeter-plugins.org/">JMeterPlugins</a>，可以提供不少其他的报告,包括各种响应时间、吞吐率、线程等的变化曲线等<a href="http://www.yeetrack.com/?p=858">http://www.yeetrack.com/?p=858</a>。</p>

<p>并且这个插件提供了命令行工具，可以将我们看到的各种曲线，各种报告统计成png图片，或者csv文件。这样我们就完全可以通过命令行来运行jmeter，生成jtl文件，然后在解析jtl文件，产生各种报告，或者展示到网页，或者插入到数据库，等等。<span id="more-1028"></span></p>

<p>英文地址：<a href="http://jmeter-plugins.org/wiki/JMeterPluginsCMD/">http://jmeter-plugins.org/wiki/JMeterPluginsCMD/</a></p>

<p>我们可以像使用命令 java -jar CMDRunner.jar  --tool Reporter --generate-png PerfMon.png --input-jtl  C:\loveme.jtl  --plugin-type PerfMon</p>
生成这样的报告
<img src="/img/images/locust/jmeter.png" alt="">

<h2 id="locust">使用locust</h2>
<p>
Locust是开源、使用Python开发、基于事件、支持分布式并且提供Web UI进行测试执行和结果展示的性能测试工具。而它之所以能够在资源占用方面明显优于JMeter，一个关键点在于两者模拟虚拟用户的方式不同，JMeter通过线程来作为虚拟用户，而Locust借助gevent库对协程的支持，以greenlet来实现对用户的模拟，相同配置下Locust能支持的并发用户数相比JMeter可以达到一个数量级的提升。
Locust使用Python代码定义测试场景，目前支持Python 2.7, 3.3, 3.4, 3.5, 和3.6。它自带一个Web UI,用于定义用户模型，发起测试，实时测试数据，错误统计等，在最新未正式发布的v0.8a2(当前最新发布版本v0.8a1)，还提供QPS、评价响应时间等几个简单的图表。
</p>


<p>本文不会介绍Locust最基础的部署、运行等Quick start式内容，这部分内容请直接参照<a href="https://link.jianshu.com?t=http://docs.locust.io/en/latest/quickstart.html" target="_blank" rel="nofollow">官网Quick start</a>或者搜索Locust入门的博客，本节主要介绍一些目前网络上还比较缺少的，真正要用Locust来做Web系统性能测试时通常需要用到的内容或者可能遇到的问题</p>

<h3 id="locust-h">指定Web host</h3>
<p>在Linux系统多网卡情况下，Locust自动选择网卡时可能会遇到<code>error: [Errno 97] Address family not supported by protocol</code>错误,此时可以通过直接指定web host来解决问题，使用选项<code>--web-host</code>来指定可用的地址，例：</p>
```python
locust -f xxx.py --web-host=127.0.0.1
locust -f xxx.py --web-host=192.168.1.2
locust -f xxx.py --web-host=localhost
```


<h3 id="locust-d">断言</h3>
<p>当我们没有自定义断言时，测试请求结果的状态(success/fail)取决于Http请求是否有异常出现，而在对我们的Web系统实施性能测试时，当我们需要更准确的业务成功率数据时，就需要通过对响应状态码、Response body等数据进行校验来给出结果，此时，可以通过ResponseContextManager来实现。首先在场景代码的发起请求参数中通过<code>catch_response=True</code>来捕获响应数据，然后对响应数据进行校验，最后使用success()/failure()两个方法来标识请求结果的状态。例：</p>
```python
@task(2)
def foo(self):
    with self.client.get("/", catch_response=True) as response:
        if response.status_code == 200:
                response.success()
@task(1)
 def bar(self):
    reqBody = '{"username":"ellen_key", "password":"education"}'
    with self.client.post("/login", reqBody, catch_response=True) as response:
            if response.content == "":
                    response.failure("No data")
```

<h3 id="locust-j">Json</h3>
<p>Json作为一种轻量级的数据交换格式，以及被如今的互联网系统广泛采用。上一节的示例中，我们使用content获取完整的响应内容，实际测试实施中，对于动态的响应结果，可能更多的采用校验关键字段的方式对于Json格式的响应数据，要获取特定字段的值，可以直接使用内置的Json解析实现，例：<br>
对于如下的响应结果：</p>
```python
@task(1)
def list_header(self):
    r = self.client.get("/homepage/list_header.html")
    if json.loads((r.content))["result"] != 100:
        r.failure("Got wrong response:"+r.content)
```

<h3 id="locust-l">自定义标签</h3>
<p>从简介的Summary report图中可以看到，Locust的结果展示中，请求的默认名称是url的path部分，而为了报告更直观，或者当同一个业务有动态的path(如/user/[userid])，需要聚合时，可以通过<code>name</code>参数来自定义标签实现,例：</p>
```python
with  self.client.get("/account/{accountID}",
        catch_response=True, name = "getAccount") as resp
```

<h3 id="locust-f">分布式运行</h3>
<p>Locust的分布式运行，master和slave节点都需要有场景脚本，分别以如下命令启动：
```python
master:locust -f locustfile.py --master --web-host=x.x.x.x
slave:locust -f locustfile.py --slave --master-host=x.x.x.x
```
master节点将运行Locust的Web UI服务，不会承担任何施压任务(不会模拟虚拟用户)。
如前面的简介，Locust模拟并发用户是使用协程，也因此对于多核CPU的服务器，为良好的利用多核能力，建议一台slave服务器运行与CPU核数相当的slave。
</p>

<h3 id ='locust-demo'>Locust 代码练习</h3>

```python
from locust import HttpLocust,TaskSet,task
import subprocess
import json

#This is the TaskSet class.
class UserBehavior(TaskSet):
    #Execute before any task.
    def on_start(self):
        pass

    #the @task takes an optional weight argument.
    @task(1)
    def list_header(self):
        r = self.client.get("/homepage/list_header.html")
        if json.loads((r.content))["result"] != 100:
            r.failure("Got wrong response:"+r.content)

    @task(2)
    def list_goods(self):
        r = self.client.get("/homepage/list_goods.html")
        if json.loads((r.content))["result"] != 100:
            r.failure("Got wrong response:"+r.content)

#This is one HttpLocust class.
class WebUserLocust(HttpLocust):
    #Speicify the weight of the locust.
    weight = 1
    #The taskset class name is the value of the task_set.
    task_set = UserBehavior
    #Wait time between the execution of tasks.
    min_wait = 5000
    max_wait = 15000

#This is another HttpLocust class.
class MobileUserLocust(HttpLocust):
    weight = 3
    task_set = UserBehavior
    min_wait = 3000
    max_wait = 6000
```
Locust的启动
```python
locust -f .\locust_test_1.py --host=http://api.g.caipiao.163.com
```
http://localhost:8089/
<img src="/img/images/locust/locust-demo-start.png" alt="">

<img src="/img/images/locust/locust-demo-result.png" alt="">
<p>
从上图可以看出在Statistics标签下列出了一些性能相关的测试结果，比如总的请求数量、请求失败的个数、每秒钟的请求数、最小\最大响应时间、平均响应时间等。右上角显示了请求失败率和总的RPS（每秒钟请求数）。对应在Statistic右侧的Failures、Exceptions、Download Data标签下我们分别可以查看失败的请求、捕获的异常以及下载测试结果。这里不做过多介绍了，可以实际应用看一下。如果想深入的了解Locust性能测试框架，去官网上看看吧。
</p>

如果不喜欢界面， 我们也可以直接通过命令行方式运行
```python
locust -f .\locust_test_1.py --host='http://api.winyyg.com' --no-web -c 1000 -r 10 -n 1000
```
<img src="/img/images/locust/locust-demo-cmd.png" alt="">

随着技术的迭代变更，这里的内容可能不是最新的， 请参考官网获取最新内容

<https://www.locust.io/>

<https://docs.locust.io/en/latest/quickstart.html>
