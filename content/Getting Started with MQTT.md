Title: Getting Started with MQTT
Date: 2015/11/27
Category: IoT
Tags: MQTT
Author: WangXinyu
# MQTT入门介绍
本文参考[MQTT Version 3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)，根据个人的理解对MQTT进行入门级的介绍。
## 1. 概述
MQTT（MQ Telemetry Transport）是应用于M2M/IoT的传输协议，来自于IBM，已于2014年正式成为了OASIS标准。

MQTT是极其简单、轻量级的协议，这是针对IoT的带宽低、延迟高、不稳定、终端设备功能简单等特点而设计的。MQTT在简单轻量的同时保证了通信的有效性和可靠性。

MQTT采用客户端和服务器之间发布／订阅的方式进行消息传递。客户端可向服务器发布消息，也可向服务器订阅其它客户端发布的消息。服务器可被视为发布消息的客户端和订阅消息的客户端之间的中介。客户端之间不可直接传递消息，必须通过中介（服务器）来发布／订阅消息。

MQTT可基于TCP/IP或其它有序的、丢包率低的、双向的网络协议。

MQTT控制包（Control Packet）是网络中传输的MQTT信息的总称，控制包分为14种类型。
## 2. MQTT内容概况
[MQTT Version 3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)标准内容非常少，整个标准只有81页，包括的主要内容只有：

（1）MQTT控制包（Control Packet）的通用格式；

（2）14种控制包的具体说明；

（3）通信的具体流程；

（4）加密问题。
## 3. 控制包格式
MQTT控制包只有三部分，固定头、可选头、净荷，如下图所示：

![MQTTctrlPktStruc.png](/images/MQTTctrlPktStruc.png)

固定头是每一种控制包都有的，大小为2Byte，包括控制包类型、标志和剩余长度三个字段，如下图所示：

![MQTTfixHead.png](/images/MQTTfixHead.png)

可选头和净荷都存在于其中几种类型的控制包中，内容取决于包类型。
## 4. 控制包类型
下表是MQTT控制包的14种类型，C2S代表客户端到服务器，S2C反之。各类型控制包的具体内容请参阅[MQTT Version 3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html)。

![MQTTctrlPktCtgry.png](/images/MQTTctrlPktCtgry.png)

## 5. QoS等级和消息传递流程
MQTT应用消息的传递有3个保障级别，即3个QoS等级。不同QoS等级对应着不同的消息传递流程。

（1）QoS 0: At most once delivery，至多一次传递。发送端仅发送一次，无重传，接收端无响应。消息可能到达接收端一次或零次。

（2）QoS 1: At least once delivery，至少一次传递。接收端收到消息后返回PUBACK，消息至少到达接收端一次。

（3）QoS 2: Exactly once delivery，消息传且仅传一次。此级别不允许丢包和消息重复。消息要经过两次确认，第一次发送端发送PUBLISH，接收端返回PUBREC；随后第二次发送端发送PUBREL，接收端返回PUBCOMP。


注：本文为作者原创，转载请注明出处。