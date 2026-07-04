---

title: "【HCIA 路由与交换 Wakin 谢Sir】TCP/IP网络模型"
slug: "【HCIA 路由与交换 Wakin 谢Sir】TCP/IP网络模型"
description: 
date: "2024-11-11T22:05:23+08:00"
image: icipie.png
math: 
license: 
hidden: false
draft: false 
categories: ["网络技术"]
tags: ["HCIA"]

---

---

## TCP/IP网络模型

> TCP/IP模型是互联网的基础，它是一系列网络协议的总称。这些协议可以划分为四层，分别为链路层、网络层、传输层和应用层。
>
> - 链路层：负责封装和解封装IP报文，发送和接受ARP/RARP报文等。
> - 网络层：负责路由以及把分组报文发送给目标网络或主机。
> - 传输层：负责对报文进行分组和重组，并以TCP或UDP协议格式封装报文。
> - 应用层：负责向用户提供应用程序，比如HTTP、FTP、Telnet、DNS、SMTP等。
>
> 在网络体系结构中网络通信的建立必须是在通信双方的对等层进行，不能交错。 在整个数据传输过程中，数据在发送端时经过各层时都要附加上相应层的协议头和协议尾（仅数据链路层需要封装协议尾）部分，也就是要对数据进行协议封装，以标识对应层所用的通信协议。

| <font color="#ffff00">TCP/IP模型</font> |                              | <font color="#ffff00">OSI模型</font> |
| :-------------------------------------: | :--------------------------: | :----------------------------------: |
|                   应                    |    为应用程序提供网络服务    |                应用层                |
|                   用                    |    数据格式化，加密、解密    |                表示层                |
|                   层                    |   建立、维护、管理会话连接   |                会话层                |
|                 传输层                  |  建立、维护、管理端到端连接  |                传输层                |
|                 网络层                  |       IP寻址和路由选择       |                网络层                |
|               数据链路层                | 控制网络层与物理层之间的通信 |              数据链路层              |
|                 物理层                  |          比特流传输          |                物理层                |

---

## 应用层 （Data 数据）

>应用层：Application Layer
>为应用软件提供接口，使应用程序能够使用网络服务。
>
>应用层协议指定相应的传输层协议，以及传输层所使用的端口等

![应用层协议指定相应的传输层协议，以及传输层所使用的端口等](assets/file-20240823222745085.png)

![应用层](assets/file-20240825092901628.png)

### DNS
>Domain Name System，域名解析系统
>Port：53（TCP & UDP）

![DNS，域名解析系统](assets/file-20240825093242520.png)

![域名的结构](assets/file-20240825093316148.png)

```powershell
nslookup    #DNS调试工具
ipconfig /displaydns    #查看DNS缓存
ipconfig /flushdns    #清空DNS缓存
```

![DNS查询过程](assets/file-20240825093946418.png)

![DNS查询过程](assets/file-20240825094250753.png)

### HTTP & HTTPS
>Hypertext Transfer Protocol(Secure)，超文本传输（安全）协议
>HTTP Port：80（TCP）
>HTTPS Port：443（TCP,SSL）

![HTTP & HTTPS，超文本传输（安全）协议](assets/file-20240825094358706.png)

### Telnet & SSH

>远程管理服务
>Telnet Port：23（TCP）
>SSH Port：22（TCP）

![远程管理服务](assets/file-20240825094740754.png)

### FTP & TFTP

>File Transfer Protocol，文件传输协议；<br>
>FTP Port：20（FTP-Data，TCP）21（FTP-Control，TCP）
>
>Trivial File Transfer Protoco，简单传输协议<br>
>TFTP Port：69（UDP）

![File Transfer Protocol，文件传输协议](assets/file-20240825095845042.png)



## 传输层 （Segment 段）

> 建立“**端到端**”（**Port to Port**）的连接。

![传输层：建立“端到端”（Port to Port）的连接 ](assets/image-20241111225124233.png)

### Port

> 端口号，区分不同的网络服务（应用层协议）
>
> 【[服务名称和传输协议端口号注册表](https://www.iana.org/assignments/service-names-port-numbers/service-names-port-numbers.xhtml)】

![端口号，区分不同的网络服务（应用层协议）](assets/image-20241111222504052.png)



### TCP & UDP

![传输层协议](assets/file-20240825101704921.png)

![TCP和UDP的对比](assets/image-20241111225207249.png)

![TCP和UDP数据包结构对比](assets/image-20241111225334694.png)

#### TCP

![TCP会话的建立-三次握手：任何基于TCP的应用，发送数据前都需要先建立连接。](assets/image-20241111225751135.png)

![TCP的确认和重传：保证数据传输的可靠性](assets/file-20240825103358926.png)

![TCP的窗口滑动机制：控制数据的传输速率](assets/file-20240825103452945.png)

![TCP会话的关闭-四次挥手：断开连接，释放资源](assets/file-20240825103554359.png)

#### UDP

![不可靠的、无连接的服务](assets/file-20240825103714402.png)

## 网络层 （Packet 包）

> 建立“**点到点**”（**End to End**）的连接
>
> 提供了无连接数据传输服务，即在发送数据报文时不需要先建立连接，每一个IP数据报文独立发送。

![网络层：建立“点到点”（End to End）的连接](assets/image-20241111230622250.png)



### IP

![IP：Internet Protocol，因特网协议，TCP/IP协议簇中最核心的协议](assets/image-20241111230943731.png)

#### IPv4报文结构

![IPv4报文结构](assets/file-20240825110905153.png)

#### TTL

![TTL：Time to Live,生存时间（类似生命值）](assets/image-20241111231239289.png)

![路由跟踪：利用TTL特性，可以显示路径上的每一跳，一种非常重要的排错方法](assets/image-20241111231313349.png)

#### Protocol

![协议号：用于标识上层协议](assets/image-20241111231508778.png)

![一些常用的协议号](assets/file-20240825112526562.png)

### IP地址及子网划分

![IP地址：IP Address，用来标识网络中的一个节点或接口，用于寻址](assets/image-20241111231829126.png)

#### IP地址结构

![IP地址结构：由32位二进制（32bits）组成，采用“点分十进制”表示，分4组](assets/image-20241111231951550.png)

![IP地址关键术语](assets/file-20240825112924638.png)

![IP地址具体表示](assets/file-20240825113048636.png)

#### IP地址分类

![IP地址分类：根据第一组8位二进制的不同规则定义](assets/image-20241111232254889.png)

![默认子网掩码](assets/image-20241111232447829.png)

![特殊IP地址及作用](assets/file-20240825155238877.png)

![A、B、C类私网地址范围](assets/file-20240825155519414.png)

#### 网络地址、主机地址、广播地址

![网络地址和广播地址](assets/file-20240825155715143.png)

![网络地址、主机地址、广播地址](assets/file-20240825160146960.png)

#### 子网划分

![子网划分](assets/file-20240825160258884.png)

![子网个数和子网中的可用主机个数](assets/file-20240825160334613.png)

>快=块（Block）=2^主机位数 =256-掩码

![快=块（block）](assets/file-20240825160923937.png)

#### IP地址规划

![IP地址规划参考原则](assets/file-20240825161705555.png)

### ICMP

![ICMP：Internet Control Message Protocol，因特网控制消息协议](assets/image-20241111233327609.png)

#### ICMP报文结构

![ICMP报文结构](assets/file-20240825161849633.png)

![字段类型含义](assets/file-20240825161909701.png)

|   情况   |                       说明                        |
| :------: | :-----------------------------------------------: |
|   超时   |         对方主机不在线、屏蔽、网络拥塞等          |
| 传输失败 |            地址无效、主机本身没有路由             |
| 无法访问 | 无法访问目标网、中转设备没有路由、没有获取MAC地址 |



## 数据链路层 （Frame 帧）

![数据链路层](assets/image-20241111233648839.png)

### 以太网
>Ethernet：当今主导地位的“局域网组网技术”

#### 以太网帧结构

![以太网帧结构](assets/image-20241111233749182.png)

![五种字段](assets/20240825163647.png)

### MAC地址

![MAC地址结构，全球唯一](assets/file-20240825163947682.png)

![MAC地址与IP地址的关系](assets/image-20241111234136563.png)

### ARP协议

![ARP：Address Resolution Protocol，地址解析协议](assets/image-20241111234318338.png)

![ARP工作流程](assets/file-20240825164432653.png)

![免费ARP：用来检测IP地址是否冲突](assets/file-20240825164628957.png)‘

![代理ARP：可以帮助同一网段，不同物理网络上的计算机之间实现通信](assets/file-20240825164917444.png)

![ARP欺骗：攻击者发送“无故ARP响应”来伪装其它设备，导致通讯失败](assets/file-20241111141851616.png)

![使用科来更改ARP数据包](assets/image-20241111235025076.png)



## 附录

### 参考文献

《[Wakin 谢Sir 最新数通精品课程_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1qP4y1w75v/?spm_id_from=333.999.0.0&vd_source=00d49c6b1d7b58728495868451fb3d19)》

《[TCP/IP网络模型 · GitBook (tonydeng.github.io)](https://tonydeng.github.io/sdn-handbook/basic/tcpip.html)》

### 版权信息

本文原载于 [Ranch's Blog](https://ranch007.github.io)，遵循 CC BY-NC-SA 4.0 协议，复制请保留原文出处。
