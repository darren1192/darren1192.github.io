---
title: 网络编程 - TCP/IP协议
date: 2018-10-27 10:03:15
tags: 网络
---
之前简单的通过[TCP/IP模型](https://darren1192.github.io/2018/10/25/%E7%BD%91%E7%BB%9C%E7%BC%96%E7%A8%8B-%E6%A6%82%E8%BF%B0/)介绍了网络编程，这篇主要介绍TCP/IP协议。
TCP/IP协议其实是一个协议簇，其中比较重要的有SLIP协议、PPP协议、IP协议、ICMP协议、ARP协议、TCP协议、UDP协议、FTP协议、DNS协议、SMTP协议等。这里我们主要介绍几个常见的协议。
## IP协议
IP协议(Internet Protocol)，又叫网际协议，它解决了多个局域网互通的问题，提供不可靠、无连接的数据报传送服务。
- 不可靠：它不能保证IP数据报能成功到达目的地。IP协议会尽最大的努力，把数据包发送到目的地，但如果期间出现问题，则发送失败。比如路由器暂时用完了缓冲区。
- 无连接：IP并不维护任何关于后续数据报的状态信息，可以不按发送顺序接收。比如先后发送A、B两条数据报。A、B是相互独立的，可能会有不同的路由选择，选择不同的路线，所以并不能判定A、B谁先到达。

### IP首部
![IP首部.png](https://upload-images.jianshu.io/upload_images/2855070-a0f12175f3f246dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
现在是看图说话时间（捡重要的说）。
- 普通的IP首部长为20个字节，除非含有选项字段。最高位在左面，记0bit，最低位在右面，记31bit。传输次序是**大端序（Big Endian）**。先传0～7bit，其次8～15 bit，然后16～23 bit，最后是24~31 bit。
关于大端序小端序可以看[这篇](https://blog.csdn.net/lisonglisonglisong/article/details/45421091)
- 目前通用的版本号是4，所以也叫IPv4。
- 首部长度指的是首部占32 bit字的数目，包括任何选项。
-  总长度字段是指整个IP数据报的长度，最长可达65535字节。
- 标识字段唯一地标识主机发送的每一份数据报。
- 生存时间字段设置了数据报可以经过的最多路由器数，初始值由源主机设置（通常为32或64），一旦经过一个处理它的路由器，它的值就减去1。当该字段的值为0时，数据报就被丢弃，并发送ICMP报文通知源主机。
- 通过首部中的协议确定接收数据的上层协议（TCP/UDP/ICMP等）。
- 首部检验和字段是根据IP首部计算的[检验和](https://blog.csdn.net/dongfei2033/article/details/75172292)码。它可以判断首部在传输过程中有没有发生任何差错。如果发生差错就丢弃收到的数据报。
- 源IP地址和目的IP地址在[上篇](https://www.jianshu.com/p/37122c626436)提到。
- 选项，是数据报中的一个可变长的可选信息。

## ICMP协议
ICMP(nternet Control Message Protocol)，Internet控制报文协议，从技术角度来说，ICMP就是一个“错误侦测与回报机制”，其目的就是让我们能够检测网路的连线状况﹐也能确保连线的准确性﹐其主要功能包括，确认 IP 包是否成功送达目标地址，通知在发送过程当中 IP 包被废弃的具体原因，改善网络设置等。
## TCP协议
TCP协议(Transmission Control Protocol)，传输控制协议，位于传输层。它提供一种面向连接的、可靠的字节流服务。
- 面向连接：两个使用TCP的应用（通常是一个客户和一个服务器）在彼此交换数据之前必须先建立一个TCP连接。广播和多播不能用于TCP。
- 可靠的：TCP协议将会通过以下几个方法提供可靠性
1. 应用数据被分割成TCP认为最适合发送的数据块。
2. 当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。
3. 当TCP收到发自TCP连接另一端的数据，它将发送一个确认。这个确认不是立即发送，通常将推迟几分之一秒。
4. TCP将保持它首部和数据的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP将丢弃这个报文段和不确认收到此报文段。
5. TCP数据是在IP数据包中传输，如果IP数据丢失，那么TCP数据也会丢失，进行重发。如有必要，TCP将对收到的数据进行重新排序，将收到的数据以正确的顺序交给应用层。
6. TCP的接收端会丢弃重复的数据。
7. TCP还能提供流量控制。TCP连接的每一方都有固定大小的缓冲空间。TCP的接收端只允许另一端发送接收端缓冲区所能接纳的数据。这将防止较快主机致使较慢主机的缓冲区溢出。

其实我觉得[Halfrost](https://github.com/halfrost)总结的很好：TCP就是通过检验和、序列号、确认应答、重发控制、连接管理以及窗口控制等机制实现可靠性传输。
### TCP首部
![TCP首部.png](https://upload-images.jianshu.io/upload_images/2855070-d2eb8abdcf1cf9bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果不计任选字段，它通常是20个字节。
依然进入看图说话环节：
- 每个TCP段都包含源端和目的端的端口号，用于寻找发端和收端应用进程。
- 序号，即Sequence Number，用来解决网络包乱序问题。
- 确认序号，即Acknowledgement Number，就是ACK——用于确认收到，用来解决不丢包的问题。
- 首部长度，或叫数据偏移，该字段表示 TCP 所传输的数据部分应该从 TCP 包的哪个位开始计算。
- 保留，该字段主要是为了以后扩展使用。
- 窗口控制，TCP的流量控制由连接的每一端通过声明的窗口大小来提供。
- 检验和，它覆盖了整个的TCP报文段：TCP首部和TCP数据。这是一个强制性的字段，一定是由发端计算和存储，并由收端进行验证。
- 紧急指针，只有当URG标志置1时紧急指针才有效

### TCP三次握手、四次挥手
#### 三次握手
![TCP三次握手.png](https://upload-images.jianshu.io/upload_images/2855070-bf96793d9b37dcc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
简单来说过程就是这样的：
1. 客户端首先向服务端发送一个SYN包和一个随机序列号 J
2. 服务端收到后会回复客户端一个 SYN-ACK 包以及一个确认号（用于确认收到 SYN）J+1，同时再发送一个随机序列号 K
3. 客户端收到后会发送一个 ACK 包和确定号 K+1 给服务端

我们看一个例子：
```
% sudo tcpdump -c 3 -i en3 -nS host 23.63.125.15
18:31:29.140787 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [S], seq 1721092979, win 65535, options [mss 1460,nop,wscale 4,nop,nop,TS val 743929763 ecr 0,sackOK,eol], length 0
18:31:29.150866 IP 23.63.125.15.80 > 10.0.1.6.52181: Flags [S.], seq 673593777, ack 1721092980, win 14480, options [mss 1460,sackOK,TS val 1433256622 ecr 743929763,nop,wscale 1], length 0
18:31:29.150908 IP 10.0.1.6.52181 > 23.63.125.15.80: Flags [.], ack 673593778, win 8235, options [nop,nop,TS val 743929773 ecr 1433256622], length 0
```
**第一行**：
client -> server
左面是发送时间`18:31`，`IP`代表的是这些都是 IP 协议数据包。
`10.0.1.6.52181 > 23.63.125.15.80`，代表源和目标端的 IP 地址＋端口。比如`10.0.1.6.52181`，`10.0.1.6`是IP地址，`52181`是端口号。
`Flags`表示 TCP 报文段 header 信息中的一些缩写标识。`S`代表 `SYN`，`.` 代表`ACK`，`P`代表`PUSH`，`F`是`FIN`。
`1721092979`是随机号`seq`
**第二行**
server -> client  
前面几个就不多做说明，`Flags [S.]`，表示报文段 header中带有`SYN`和`ACK`。
`ack `为`1721092980 `，即`第一行的seq+1`，`seq `为`673593777 `
**第三行**
client -> server
客户端接收到信息后，发送`Flags [.]`，报文段 header中`ACK`。
`ack`为`673593778 `，即`第二行的seq+1`。

#### 四次挥手

![TCP四次分手.png](https://upload-images.jianshu.io/upload_images/2855070-a03b215b3ce31fd9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. 客户端要断开连接了，向服务器发送`FIN`信号，告诉它我的数据发送完了，但是如果你有数据，我还可以接受，然后进入`FIN_WAIT_1 `状态
2. 服务器接确认`FIN`包后，发送一个`ack`确认包，告诉客户端我已接收到信息，但还没有做好关闭准备。发送完毕后，服务器端进入 `CLOSE_WAIT`状态，客户端接收到这个确认包之后，进入 `FIN_WAIT_2`状态，等待服务器端关闭连接。
3. 服务器端准备好关闭连接时，向客户端发送`FIN`包，要断开连接。然后进入服务器进入`LAST_ACK `状态。
4. 客户端接收到来自服务器端的关闭请求，发送一个确认包，并进入 `TIME_WAIT`状态，等待可能出现的要求重传的`ACK`包。服务器端接收到这个确认包之后，关闭连接，进入`CLOSED`状态。等过了一定时间后，客户端没有收到服务器的`ACK`包，就知道已经断开连接，进入`CLOSED `状态。

#### 为什么要进行三次握手？
通常我们会觉得，客户端告诉服务器，我要和你握手了，然后服务器告诉客户端，我知道了，这样就可以了。为什么要多客户端再给服务器确认一遍呢？
这是因为我们现实生活中的网络是复杂的，TCP数据在传输的过程中，很有可能会出现延迟到达或者重发的情况。我们在之前谈到TCP的可靠性时，说到当TCP发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。这个时候TCP首部的序列号是相同的。
如果因为网络问题，第二个`SYN`包提前到达，在连接结束后第一个`SYN`包才到，服务器又会发一次`ACK`建立连接。此时服务器建立了连接，客户端没有连接，从而导致服务端建立了一个空的连接，浪费资源。
如果是三次握手，客户端再次收到相同的`ACK`时，会丢弃这个包，不向服务端发送`ACK`和`ack`，从而避免了空连接。
#### 为什么进行四次挥手
因为TCP是双全工模式，即可以同时发送和接收数据，两条通道是完全独立的。客户端向服务器挥手关闭的时候，服务器会继续传送之前没有传完的数据。在二三挥手之间，多了一个数据传送的过程，这也是为什么`ACK`和`FIN`不能同时发送的原因。
### SYN攻击
#### SYN攻击是什么
在三次握手过程中，服务器发送`SYN-ACK`之后，收到客户端的`ACK`之前的TCP连接成为半连接(half-open connect)。此时服务器处于`SYN_RCVD `状态。当收到`ACK `之后，服务器才能转入`ESTABLISHED  `状态。
SYN攻击指的是，攻击客户端在短时间内伪造大量不存在的IP地址，向服务器不断发送`SYN`包，服务器回复确认包，并等待客户的确认。由于源地址是不存在的，服务器需要不断地重发直至超时，这些伪造的`SYN`包将长时间占用未连接队列，正常的`SYN`请求被丢弃，导致目标系统运行缓慢，严重会造成网络堵塞甚至系统瘫痪。
SYN攻击是一种典型的`DoS/DDoS`攻击。
#### 如何检测SYN攻击
检测SYN攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击。在 Linux/Unix 上可以使用系统自带的`netstats`命令来检测 SYN 攻击。
####  如何防御SYN攻击
SYN攻击不能完全被阻止，除非将TCP协议重新设计。我们所做的是尽可能的减轻SYN攻击的危害，常见的防御 SYN 攻击的方法有如下几种：
- 缩短超时（SYN Timeout）时间
- 增加最大半连接数
- 过滤网关防护
- SYN cookies技术

### TCP KeepAlive
TCP 通信双方建立交互的连接，但是并不是一直存在数据交互，有些连接会在数据交互完毕后，主动释放连接，而有些不会。交互双方出现死机、异常重启等情况都不会使TCP连接及时正常释放，造成端系统资源的消耗和浪费。为了解决这个问题，在传输层可以利用 TCP 的 KeepAlive 机制实现来实现。主流的操作系统基本都在内核里支持了这个特性。
TCP KeepAlive 的基本原理是，隔一段时间给连接对端发送一个探测包，如果收到对方回应的 ACK，则认为连接还是存活的，在超过一定重试次数之后还是没有收到对方的回应，则丢弃该 TCP 连接。
但TCP KeepAlive 是有局限。首先 TCP KeepAlive 监测的方式是发送一个 probe 包，会给网络带来额外的流量，另外 TCP KeepAlive 只能在内核层级监测连接的存活与否，而连接的存活不一定代表服务的可用。例如当一个服务器 CPU 进程服务器占用达到 100%，已经卡死不能响应请求了，此时 TCP KeepAlive 依然会认为连接是存活的。因此 TCP KeepAlive 对于应用层程序的价值是相对较小的。需要做连接保活的应用层程序，例如 QQ，往往会在应用层实现自己的心跳功能。

## UDP协议
UDP协议(User Datagram Protocol)，用户数据报协议。UDP 不提供复杂的控制机制，利用 IP 提供面向无连接的通信服务。传输途中即使出现丢包，UDP 也不负责重发，甚至当出现包的到达顺序乱掉时也没有纠正的功能。UDP 面向无连接，它可以随时发送数据。
- 面向无连接：通信双方不需要事先建立一条连接，而是把每个带有目的地址的包送到线路上，由系统自主选定路线进行传输。
- 不可靠性：它把应用程序传给IP层的数据发送出去，但是并不保证它们能到达目的。

### UDP首部

![UDP首部.png](https://upload-images.jianshu.io/upload_images/2855070-7ad5cbcf7a6bccf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 端口号不在说明了，和TCP一样，只不过TCP端口号由TCP来查看，而UDP端口号由UDP来查看，这是根据IP数据报里面的协议来区分的。
- UDP长度字段指的是UDP首部和UDP数据的字节长度
- UDP检验和覆盖UDP首部和UDP数据。和TCP区别是，TCP检验和是必需的，而UDP的检验和是可选的。

### TCP和UDP区别
- 数据发送方式区别：TCP是建立在两端连接之上的协议，UDP本身发送的就是一份份数据报。
- 数据大小的区别：TCP理论上发送的数据流没有上限，但是由于缓冲区有大小限制，但如果TCP数据过大，会被截为好几段，接收方依次接收。UDP报文长度不能超过2^16=65536(如果想具体了解UDP包长限制，可以看[这篇](http://www.52im.net/thread-29-1-1.html)).
- 数据有序性的区别：TCP本身有着超时重传、错误重传、还有等等一系列复杂的算法保证了 TCP 的数据是有序的。UDP并不具备数据纠正功能。
- 数据可靠性的区别：TCP 的超时重传、错误重传、TCP 的流量控制、阻塞控制、慢热启动算法、拥塞避免算法、快速恢复算法等方法都使得TCP在保持连接的过程中是可靠的。UDP是一个面向非连接的协议，UDP 发送的每个数据报带有自己的 IP 地址和接收方的 IP 地址，它本身对这个数据报是否出错，是否到达不关心，只要发出去了就好了。
- 使用场景区别：UDP 主要用于那些对高速传输和实时性有较高要求的通信或广播通信，比如：流媒体、物联网、实时游戏等。其他就可以考虑使用TCP（只是考虑，具体问题具体分析），比如QQ文件传输。

## DNS协议
通常我们去请求一个网址输入的是`https://github.com`这种形式，但这并不是IP地址。那我们怎么去寻找他的地址呢？这时候就需要DNS。
DNS域名系统是一种用于TCP/IP应用程序的分布式数据库。我们向DNS服务器发送一个DNS数据包，然后DNS服务器做出响应告诉我们github的IP地址是`192.30.253.113`。
目前有很多DNS服务器，比如google的`8.8.8.8`，阿里的`223.5.5.0`，百度的`180.76.76.76`。
有时候我们会遇到DNS被污染的情况，这个时候我们就没办法去请求某一个网页。一个小技巧：在使用MAC的情况下，在跳转路径输入：`/private/etc/`，找到hosts文件，添加内容即可。
![hosts文件.png](https://upload-images.jianshu.io/upload_images/2855070-acb13583bdfe3b0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以上便是我对TCP/IP协议的理解，如果不当之处还望指出。
参考：[《TCP/IP详解 卷1：协议》](http://www.52im.net/topic-tcpipvol1.html?mobile=no)
[TCP/IP基础概述](https://github.com/halfrost/Halfrost-Field/blob/master/contents/Protocol/TCP:IP.md)
[IP，TCP 和 HTTP](https://objccn.io/issue-10-6/)
