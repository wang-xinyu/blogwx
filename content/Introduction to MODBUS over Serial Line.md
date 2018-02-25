Title: Introduction to MODBUS over Serial Line
Date: 2015/12/2
Category: IoT
Tags: MODBUS
Author: WangXinyu
# MODBUS over Serial Line介绍
本文参考'MODBUS over Serial Line V1.02'，根据个人的理解对MODBUS over Serial Line进行介绍，若要阅读协议文档请访问[MODBUS官网](http://www.modbus.org)。
## 1. MODBUS简介
MODBUS标准定义了一个应用层的消息传输协议，即位于OSI模型的第七层，它为连接至不同类型总线或网络的设备提供了Client/Server通信机制。此外，MODBUS也标定了一个串行线上的协议，即MODBUS over Serial Line，用于主机与从机之间交换MODBUS请求。
## 2. MODBUS over Serial Line概述
MODBUS over Serial Line也可被称为MODBUS Serial Line protocol，该协议位于OSI模型的第二层，即数据链路层。它是一种主从式的协议，支持的物理层协议是TIA/EIA-485或TIA/EIA-232，即RS485和RS232。
## 3. MODBUS over Serial Line明细
### 3.1 主从式协议的原则
MODBUS over Serial Line主从式系统的特点如下：

- 主节点有且仅有一个，从节点有1-247个；
- 主节点向从节点下达命令，并处理从节点的响应；
- 从节点只有在收到主节点的请求后才发送数据，从节点之间不可直接通信；
- 主节点在同一时刻只能进行一项MODBUS交易，即不能同时与多个从节点通信。

主节点下达的MODBUS请求有如下两种模式：

- unicast mode（单播模式），主机向一个从机下达一条请求，从机的地址是1-247，该从机在收到并处理请求后，返回一条'reply'；
- broadcast mode（广播模式），主机可将一条请求下达至所有从机，广播请求中地址值为0，从机在收到广播请求后不返回信息。

### 3.2 寻址规则
MODBUS的寻址空间包括256个地址，主节点没有地址，一条MODBUS串行总线上的从节点必须有一个独立的地址，具体的地址分配如下：

- 0，广播地址；
- 1-247，从节点地址；
- 248-255，保留。

### 3.3 帧结构
帧结构图如下，PDU即Protocol Data Unit。主机下达请求时，将从机的地址写入地址字段，从机返回响应时，将自己的地址写入地址字段。功能码用于指示功能，数据字段包含请求或响应的参数。

![ModbusPDU.png](/images/ModbusPDU.png)
### 3.4 状态转换图
主机状态转换图：

![ModbusMasterStat.png](/images/ModbusMasterStat.png)

从机状态转换图：

![ModbusSlaveStat.png](/images/ModbusSlaveStat.png)
### 3.5 两种串行传输模式——RTU与ASCII
#### 3.5.1 RTU模式
RTU即Remote Terminal Unit，该模式下，数据以二进制数字形式传输，例如需传输的一个字节是0x5B，实际传输的数据也是0x5B。RTU模式主要包含如下几点约定：

- 每个字节都被封装成11位，包含1位起始位、8位数据、1位奇/偶校验位、1位停止位；
- 帧结构包含1字节从机地址、1字节功能码、0－252字节数据、2字节CRC校验；
- 帧间间隔至少为3.5倍的单字符传输时间；
- 帧内的字符间隔至多为1.5倍的单字符传输时间。

#### 3.5.2 ASCII模式
ASCII模式下，数据以ASCII字符形式传输，例如需传输的一个字节是0x5B，实际传输的是0x35（“5”）和0x42（“B”），显然该模式的传输效率比RTU低。通常，只有当设备不满足RTU模式所需的定时器要求时，才使用ASCII模式。ASCII主要包含如下几点约定：

- 每个字节都被封装成10位，包含1位起始位、7位数据、1位奇/偶校验位、1位停止位；
- 帧结构包含1字符起始字段、2字符从机地址、2字符功能码、0－2x252字符数据、2字符LRC校验、2字符结束字段；
- 起始字段是冒号“:”，结束字段是CR和LF。


注：本文为作者原创，欢迎批评指正，转载请注明出处。