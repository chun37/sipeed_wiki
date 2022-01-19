---
title: Python3 网络应用
keywords: MaixPy3,net, Python3
desc: maixpy doc: net
---


| 更新时间 | 负责人 | 内容 | 备注 |
| --- | --- | --- | --- |
| 2021年1月17日 | dalaoshu | 编写文档 | 本文讲述用户常问的实际需求所写的经验文，有经验的可以忽略本节内容，至于如何联网、配网请看类似于 [MaixII-Dock 连接网络](http://wiki.sipeed.com/soft/maixpy3/zh/tools/0.MaixII-Dock.html#%E5%A6%82%E4%BD%95%E8%BF%9E%E6%8E%A5%E7%BD%91%E7%BB%9C) 的产品说明|

## 关于 Python 网络应用的一些常用例子。

因为是通用的 python3 环境，所以关于网络方面的库很多，最终目的都是为了上云。

- [requests](https://github.com/psf/requests)

- [paho.mqtt.python](https://github.com/eclipse/paho.mqtt.python)

在 AIOT 领域中，常用的网络请求接口为 mqtt 和 http ，它们均基于 socket 这个模块提供的 tcp 、udp 实现帮助用户方便进行网络传输数据。

与经典的 TCP 、UDP 相比，HTTP 和 MQTT 则更是符合互联网、物联网特征的传输协议。

例如 HTTP 将网络行为描述成从某个服务器上获取数据的 GET ，将某个数据提交给服务器的 POST 协议描述，意味着用户不需要对此重新设计协议，根据业务逻辑使用就行。

而 MQTT 让用户忽略对服务器的部署和开发，将其作为数据中转服务，将两个设备的链接行为描述成互相通信行为，而非请求服务行为，让用户只需要关注客户端的发送行为即可，而不需要考虑服务端的接收。

简化的背后是需要很多逻辑维持的，同时还存在一些安全隐患问题，如早期的 HTTP 是明文传输，MQTT 是公开通信，如果要商用就需要配置相关的安全功能。

## HTTP 有什么用？怎么用？

> 如果你想要获取网页的数据，或是向服务器提交数据，那 HTTP 就是你最好的朋友。

Requests 唯一的一个非转基因的 Python HTTP 库，人类可以安全享用。

**警告**：非专业使用其他 HTTP 库会导致危险的副作用，包括：安全缺陷症、冗余代码症、重新发明轮子症、啃文档症、抑郁、头疼、甚至死亡。

看吧，这就是 Requests 的威力：

```
>>> r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
u'{"type":"User"...'
>>> r.json()
{u'private_gists': 419, u'total_private_repos': 77, ...}
```

[参见 未使用 Requests 的相似代码.](https://gist.github.com/kennethreitz/973705)

Requests 允许你发送纯天然，植物饲养的 HTTP/1.1 请求，无需手工劳动。你不需要手动为 URL 添加查询字串，也不需要对 POST 数据进行表单编码。Keep-alive 和 HTTP 连接池的功能是 100% 自动化的，一切动力都来自于根植在 Requests 内部的 urllib3。

更多请看该文档[Requests: 让 HTTP 服务人类](https://docs.python-requests.org/zh_CN/latest/)。

## MQTT 有什么用？怎么用？

> 如果没有服务器也想要一对多采集数据，或是许多设备彼此建立通信，那 MQTT 就是你最好的朋友。

使用 MQTT 服务需要一台运行 MQTT 的服务，网上有许多公共服务器供你使用，而对于任何一台机器，只要是运行 python 代码，只会发生 订阅主题 和 推送主题 两个逻辑，这个主题就是一间房，在这间房的所有连接都会收到关于这个主题的信息，它就像一个聊天室，设备加入了，可以收到所有通知。

### 如何推送？

向公共服务 mqtt.eclipseprojects.io 发送主题 `paho/test/single` 内容为 `boo123` 的消息。

```python
>>> import paho.mqtt.publish as publish
>>>
>>> publish.single("paho/test/single", "boo123", hostname="mqtt.eclipseprojects.io")
>>>
```

### 如何订阅？

那发出去的数据如何收到呢？看下图代码示例

```python
>>> import paho.mqtt.subscribe as subscribe
>>>
>>> topics = ['paho/test/single']
>>>
>>> m = subscribe.simple(topics, hostname="mqtt.eclipseprojects.io", retained=False)
>>> m.topic
'paho/test/single'
>>> m.payload
b'boo123'
>>>
```

这两个逻辑，在任何设备上都是一样的代码，用户不需要写服务端的代码，而服务端的任务就是负责转发数据和保护整个传输内容的安全、可靠性。

> 关于 MQTT 的官网 [MQTT: The Standard for IoT Messaging](https://mqtt.org/)。
