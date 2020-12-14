MQTT，全称 Message Queuing Telemetry Transport，中文名 "消息队列遥测传输"，是基于 TCP/IP 的发布 (Publish) / 订阅 (Subscribe) 范式的消息协议，**是为硬件性能低下的远程设备以及网络状况糟糕的情况下而设计的**。它还需要一个消息中间件（相当于服务端），用于处理和转发发布端和订阅端的数据。

![](https://cdn.ethsonliu.com/x1/20200424_01.png)

协议的特点总结就是（以下译自 [MQTT Design principles](https://github.com/mqtt/mqtt.github.io/wiki/Design-Principles)），

1. 精简，但亦提供了健壮的功能模块，易集成，易使用；
2. 发布/订阅（Publish/Subscribe）模式，方便消息在传感器之间传递，并建立联机关系；
3. 尽可能地接近零运维，可以应对异常状况，例如允许用户动态创建主题；
4. 把传输量降到最低以提高传输效率；
5. 把低带宽、高延迟、不稳定的网络等因素考虑在内；
6. 支持连续的会话控制；
7. 可适用在计算能力有限的终端设备上；
8. 提供消息服务质量管理；
9. 不强制传输数据的类型与格式，保持灵活性。

## 协议文档

截至目前为止，常用的有三个版本：V3.1、V3.1.1 和 V5。注意，V3 和 V5 不兼容。

MQTT 协议官网：[http://mqtt.org](http://mqtt.org/)，官方 Github 地址：[https://github.com/mqtt/mqtt.github.io](https://github.com/mqtt/mqtt.github.io)（内含 Wiki 对协议的介绍）。

- V3.1
  - 官方 - 英文：[http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html](http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html)

- V3.1.1
  - 官方 - 英文：[http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html)
  - 翻译 - 中文：[https://mcxiaoke.gitbooks.io/mqtt-cn/content/](https://mcxiaoke.gitbooks.io/mqtt-cn/content/)

- V5
  - 官方 - 英文：[http://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html](http://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
  - 翻译 - 中文：[https://github.com/hui6075/mqtt_v5](https://github.com/hui6075/mqtt_v5)

## 简单使用

客户端我们使用 MQTT.fx，下载地址：[MQTT.fx Download](https://mqttfx.jensd.de/index.php/download)。而服务端可以借助 HiveMQ 提供的公开免费服务 [HiveMQ Public MQTT Dashboard](http://www.mqtt-dashboard.com/)。

打开 MQTT.fx，输入服务器地址（broker.hivemq.com）和端口（1883），点击连接，

![](https://cdn.ethsonliu.com/x1/20200424_02.png)

接着就可以在 MQTT.fx 的 Subscribe 栏输入你要订阅的主题，在 Publish 栏输入相同的主题，并随便填写要发布的消息，你就会在 Subscribe 栏看到刚才发布的消息。注意，因为该服务器是公开的，所以有可能收到别人发布的消息，主要看你的主题是否和别人的重复。

## 发布和订阅

发布和订阅会涉及三个很重要的专业名词：

1. 主题（Topic）
2. 负载（Payload）
3. 服务质量（QoS），英文全称 Quality of Service

下面依次介绍（其余的专业名词介绍请参见该系列后面的文章）。

### 主题（Topic）

主题（Topic）由一个或多个主题层（Topic Levels）组成，每个主题层由一个正斜杠分隔符 "/" 隔开（这个正斜杠我们一般称之为主题层分隔符 Topic Level Separator）。

![](https://cdn.ethsonliu.com/x1/20200424_03.png)

请注意，每个主题必须至少包含 1 个字符，并且主题字符串允许使用空格。主题区分大小写，例如 myname 和 MyName 是两个不同的主题。此外，主题如果只由一个主题层分隔符 "/" 组成，那么它也是有效的。

客户端订阅主题时，它可以订阅一个已发布消息的确切主题，也可以使用通配符（Wildcards）同时订阅多个主题。通配符只能用于订阅，不能用于发布。

有两种不同的通配符：单层通配符（Single Level Wildcard）和多层通配符（Multi Level Wildcard）。

#### 单层通配符（Single Level Wildcard）

顾名思义，单层通配符可以替代一个主题层，用加号 "+" 表示主题中的单层通配符。举例来说，对于主题 "a/b/c/d"，下面的订阅都可以匹配到它，

- a/b/c/d
- +/b/c/d
- a/+/c/d
- a/+/+/d
- +/+/+/+

而下面的订阅则无法匹配，

- a/b/c
- b/+/c/d
- +/+/+

#### 多层通配符（Multi Level Wildcard）

多层通配符可以替代一个或多个主题层，用符号 "#" 表示主题中的多层通配符。注意，多层通配符必须是主题的最后一个字符。如果一个主题只有一个 "#"，也是允许合法的，但这取决于服务器是否允许它被订阅。举例来说，对于主题 "a/b/c/d"，下面的订阅都可以匹配到它，

- a/b/c/d
- \#
- a/#
- a/b/#
- a/b/c/#
- +/b/c/#

### 负载（Payload）

在计算机科学领域，负载（Payload）是数据传输中所欲传输的实际信息（Message），通常也被称作实际数据或者数据体。

信息（Message）的传输一般依靠特定的协议，比如 TCP。对于接收端来说，它们想要的并不是 TCP 协议，而是你通过 TCP 协议传送过去的信息。而这个信息，我们就叫做负载。这很简单，就是多了一个专业词语。

注意，MQTT 协议要求负载必须是 UTF-8 字符串编码格式。

### 服务质量（QoS）

服务质量（Quality of Service）是发布端与订阅端的一种关于交付信息保证的协议。一共有 3 个 QoS 等级（以下等级依次增大）：

- **最多一次 0。**协议对此等级应用信息不要求回应确认，也没有重发机制，这类信息可能会发生消息丢失或重复，取决于 TCP/IP 提供的尽最大努力交互的数据包服务。

- **最少一次 1。**确保信息到达不会丢失，但消息重复可能发生。

- **只有一次 2。**最高级别的服务质量，保证不会消息丢失和重复。

等级越高，对消息的交付保证要求也就越高，同时消耗的流量也就越大。应用程序需要根据自己的网络场景和业务需求，选择合适的 QoS 级别。

## 参考

- [维基百科 - MQTT](https://zh.wikipedia.org/wiki/MQTT)
- [一文读懂MQTT协议](https://www.jianshu.com/p/5c42cb0ed1e9)
- [MQTT Topics & Best Practices - MQTT Essentials: Part 5](https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/)
- [Mosquitto man page](https://mosquitto.org/man/mosquitto-8.html)
