---
title: 网络相关知识总结
description: 国际标准化组织 ISO 提出的开放系统互联基本参考模型（Open Systems Interconnection Reference Model），简称为 OSI ，是一个概念模型。
summary: 国际标准化组织 ISO 提出的开放系统互联基本参考模型（Open Systems Interconnection Reference Model），简称为 OSI ，是一个概念模型。
date: 2020-12-20 11:01:32
categories: [interview]
tags: [网络]
cover:
  image: "https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172139.png"
  caption: ""
  alt: ""
  relative: false
---

![计算机网络](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172139.png)

## 网络基础知识

国际标准化组织 ISO 提出的开放系统互联基本参考模型（Open Systems Interconnection Reference Model），简称为 OSI ，是一个概念模型。TCP/IP 是一个四层的体系结构，它包含应用层、 运输层、网际层和网络接口层。在阐述计算机网络原理时，通常综合 OSI 和 TCP/IP 的优点，采用一种只有五层协议的体系结构。

![image-20201223085010412](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172207.png)

### 应用层（application layer）

应用层的作用是通过应用进程间的交互来完成特定网络应用。应用层协议定义的是应用进程（主机中正在运行的程序）间通信和交互的规则。如域名系统 DNS、支持万维网的 HTTP 协议和支持电子邮件的 SMTP 协议。应用层交互的数据单元称为报文（message）。

### 运输层（transport layer）

运输层的任务就是负责向两台主机中进程间的通信提供通用的数据传输服务。应用进程利用该服务传送应用层报文。运输层具有复用和分用两个功能。复用是指多个应用层进程可以同时使用下面运输层的服务，分用是指运输层把收到的信息分别交付给上面应用层中的相应进程。运输层主要使用 TCP 和 UDP 两种协议。

* 传输控制协议 TCP（Transmission Control Protocol）：提供面向连接的、可靠的数据传输服务，其数据传输的单位是报文段。
* 用户数据报协议 UDP（User Datagram Protocol）：提供无连接的、尽最大努力交付（不保证数据传输可靠性）的数据传输服务，其数据传输的单位是用户数据报。

### 网络层（network layer）

网络层主要有两个任务：

1. 为分组交换组网上的不同主机提供通信服务，网络层把运输层产生的报文段或用户数据报封装成分组或包进行传送。
2. 选择合适的路由，使源主机运输层所传下来的分组，能够通过网络中的路由器找到目的主机。

### 数据链路层（data link layer）

两台主机之间的数据传输，总是在一段一段的数据链路上传送的，这就需要专门的链路层协议。在两个相邻节点传送数据时，数据链路层将网络层交下来的 IP 数据报组装成帧，在两个相邻节点间的链路上传递帧。每一帧包括数据和必要的控制信息（如同步信息、地址信息、差错控制等）。

### 物理层（physical layer）

在物理层上传输的单位是比特。物理层需要考虑多大的电压表示 “1” 或 “0” ，以及接收方如何识别发送方所发送的比特。物理层还要确定连接电缆的插头应该有多少根引脚以及各引脚应如何连接。

## TCP/IP 协议族

TCP/IP 的体系结构只有四层。

![image-20201224220142706](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172109.png)

可以分层画出具体的协议来表示 TCP/IP 协议族，它的特点是上下两头大而中间小：应用层和网络接口层都有多种协议，而中间的 IP 层很小，上层的各种协议都向下汇聚到一个 IP 协议中。这表明：TCP/IP 协议可以为各式各样的应用提供服务，同时 TCP/IP 协议也允许 IP 协议在各式各样的网络构成的互联网上运行。

![image-20201224220539509](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172347.png)

## TCP 基础知识

### TCP 的主要特点

* TCP 是面向连接的运输层协议。应用程序在使用 TCP 协议之前，必须建立 TCP 连接。在传送完数据之后，必须释放已经建立的 TCP 连接。
* 每一条 TCP 连接只能有两个端点，每一条 TCP 连接只能是点对点的。
* TCP 连接提供可靠交付的服务。通过 TCP 连接传送的数据，无差错、不丢失、不重复，并且按序到达。
* TCP 提供全双工通信。TCP 允许通信双方的应用进程在任何时候都能发送数据。
* 面向字节流。TCP 中的 流（stream）指的是流入到进程或从进程流出的字节序列。

### TCP 报文段的首部格式

TCP 报文段的前 20 个字节是固定的，后面有 4n 字节是根据需要而增加的选项。因此 TCP 首部的最小长度是 20 字节。

![image-20201223220408024](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172357.png)

* 源端口和目的端口：各占 2 个字节，分别写入源端口号和目的端口号。
* 序号：占 4 个字节。序号范围是 [0, 2^32 -1] ，且使用 mod 2^32 运算。TCP 是面向字节流的，在一个 TCP 连接中传送的字节流中的每一个字节都按顺序编号。整个要传送的字节流的起始序号必须在建立连接时设置。首部中的序号字段值则指的是本报文段所发送的数据的第一个字节序号。
* 确认号：占 4 个字节，是期望收到对方下一个报文段的第一个数据字节的编号。例如，B 正确收到了 A 发送的一个报文段，其序号字段值是 501，而数据长度是 200 字节，这表明 B 正确收到了 A 发送的到序号 700 为止的数据。因此，B 期望收到 A 的下一个数据序号是 701，于是 B 在发给 A 的确认报文段中把确认号设置为 701 。
* 数据偏移：占 4 位，指出 TCP 报文段的首部长度。“数据偏移”的单位是 32 位字（4 字节），4 位二进制最大表示十进制 15，因此“数据偏移”字段的最大值是 60 字节，这同时是 TCP 首部的最大长度。
* 保留：占 6 位，目前应置为 0 。
* 六个标志位：
  * 紧急 URG ：当 URG = 1 时，表明紧急字段有效。发送应用进程告诉系统 TCP 有紧急数据要传送，于是 TCP 就把紧急数据插入本报文段数据的最前面，而后面的数据仍是普通数据。这时需要与首部中紧急指针字段配合使用。
  * 确认 ACK ： 仅当 ACK = 1 时确认号字段才有效。当 ACK = 0 时，确认号无效。TCP 规定，在建立连接后所有传送的报文段都必须把 ACK 置 1 。
  * 推送 PSH ：发送方 TCP 把 PSH 置 1，并理立即创建一个报文段发送出去。接收方 TCP 收到 PSH = 1 的报文段，就尽快地交付接收应用程序。
  * 复位 RST ：当 RST = 1 时，表明 TCP 连接中出现严重差错，必须释放连接，然后再重新建立运输连接。
  * 同步 SYN ：在连接建立时用来同步序号。当 SYN = 1 而 ACK = 0 时，表明这是一个连接请求报文段。对方若同意建立连接，则在相应的报文段中使用 SYN = 1 和 ACK = 1 。因此，SYN 置为 1 就表示这是一个请求或接收建立连接报文。
  * 终止 FIN ：用来释放一个连接。当 FIN = 1 时，表明此报文段的发送方的数据已发送完毕，并要求释放运输连接。
* 窗口：占 2 个字节。窗口值是 [0, 2^16 -1] 之间的整数。窗口指发送本报文段的一端的接收窗口。窗口值作为接收方让发送方设置其发送窗口的依据。
* 校验和：占 2 个字节。校验和字段校验的范围包括首部和数据这两部分。同样需要伪首部。
* 紧急指针：占 2 个字节。仅在 URG = 1 时才有意义。即使窗口为零时也可以发送紧急数据。
* 选项：长度可变，最长可达 40 字节。

### TCP 可靠传输的工作原理

TCP 所发送的报文段是交给 IP 层传送的，但是 IP 层只能提供最大努力服务，也就是说，TCP 下面的网络所提供的是不可靠的传输。TCP 必须采用适当的措施才能使得两个运输层之间的通信变得可靠。

#### 停止等待协议

“停止等待协议”就是每发送完一个分组就停止发送，等待对方的确认。在收到确认后再发送下一个分组。

1. 无差错情况

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172406.png" alt="image-20201224221251156" style="zoom:50%;" />

   A 发送分组 M1，发送完后暂停发送，等待 B 的确认。B 收到了 M1 就向 A 发送确认。A 在收到对 M1 的确认后，在发送下一个分组 M2 。同样，在收到 B 对 M2 的确认后，再发送 M3 。

2. 出现差错

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172416.png" alt="image-20201224221531171" style="zoom:50%;" />

   分组在传输过程中出现差错，B 在接收 M1 时检测到差错，就丢弃 M1，其它什么也不做（也不通知 A 收到有差错的分组）。也可能是 M1 在传输过程中丢失了，这时 B 什么都不知道。可靠传输协议中超时重传的设计是：A 只要超过一段时间仍然没有收到确认，就会认为刚才发送的分组丢失了，因而重传前面发送的分组。

3. 确认丢失和确认迟到

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172430.png" alt="image-20201224222109531" style="zoom:67%;" />

   A 发送分组后等待超时就会重传分组，B 在接受到重复分组时：丢弃重复的分组，对分组进行确认。A 在收到重复的确认时：收下后就丢弃。

#### 连续 ARQ 协议

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172443.png" alt="image-20201224222656888" style="zoom:67%;" />

发送方维持一个发送窗口，它的意义是：位于发送窗口的 5 个分组都可以连续发送出去，而不需要等待对方的确认。可以提高信道利用率。发送方每收到一个确认，就把发送窗口向前滑动一个分组的位置。

接收方一般都是采用累积确认的方式。不必对接收到的分组逐个发送确认，而是在收到几个分组后，对按顺序到达的最后一个分组发送确认，表示到这个分组为止的所有分组都已经正确接收了。

### TCP 可靠传输的实现

* 以字节为单位的滑动窗口

* 超时重传时间的选择

* 选择确认 SACK

## TCP 滑动窗口协议

TCP 的滑动窗口是以字节为单位的。

### 发送方的滑动窗口

1. 构建发送窗口

   现假定 A 收到了 B 发来的确认报文段，其窗口值是 20 字节，确认号是 31 。根据这两个数据，A 可以构造出自己的发送窗口。

   ![image-20201224223718570](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172450.png)

   发送窗口表示：在没有收到 B 的确认的情况下，A 可以连续把窗口内的数据都发送出去。凡是已经发送过的数据，在未收到确认之前都必须暂时保留，以便在超时重传时使用。

   发送窗口后沿的后面部分表示已经发送且已收到确认。这些数据不需要再保留。而发送窗口前沿的前面部分表示不允许发送，因为接收方都没有为这部分数据保留临时存放的缓冲区。

2. 部分发送但未收到确认

   假定 A 发送了序号为 31 ~ 41 的数据。这时，发送窗口位置未改变，但发送窗口内靠后面有 11 个字节表示已发送但未收到确认。发送窗口内靠前面的 9 个字节是允许发送但尚未发送的。

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172459.png" alt="image-20201224224655408" style="zoom:67%;" />

3. 收到确认后发送窗口向前滑动

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172502.png" alt="image-20201224231519787" style="zoom:67%;" />

   发送方 A 在收到 B 的确认号 33 后，就可以把发送窗口向前滑动 3 个序号，此时可发送的序号范围是 42 ~ 53 。

### 接收方的滑动窗口

接收窗口的状态有三种：已经接收且已回复、允许接收但未收到、不允许接收。

1.  接收方接收窗口的初始状态

   ![image-20201224231946417](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172519.png)

   B 的接收窗口大小是 20 ，在接收窗口外面，到 30 号为止的数据是已经发送过确认且上交给主机的。接收窗口内的序号 31 ~ 50 是允许接收的。

   B 只能对按顺序收到的数据中的最高序号给予确认，因此 B 即使收到了序号为 32、33 的数据，因为序号为 31 的数据未收到，此时 B 对 A 的确认报文中的确认号仍然是 31 。

2. B 接收到序号连续的数据

   <img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172614.png" alt="image-20201224233004635" style="zoom:67%;" />

   当 B 接收到序号为 31 的数据，并把序号为 31 ~ 33 的数据交付主机，然后 B 删除这些数据。接着把接收窗口向前移动 3 个序号，同时给 A 发送确认，其中窗口值仍是 20 ，但确认号是 34 。

### 窗口与缓存的关系

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172633.png" alt="image-20201224233353821" style="zoom:67%;" />

## TCP 的运输连接管理

TCP 是面向链接的协议。运输连接是用来传送 TCP 报文的。TCP 运输连接的建立和释放是每一次面向连接的通信中必不可少的过程。因此，运输连接就有三个阶段，即：建立连接、数据传输和释放连接。在 TCP 连接建立过程中要解决以下三个问题：

1. 要使每一方能够明知对方的存在。
2. 要允许双方协商一些参数。
3. 能够对运输实体资源进行分配。 

TCP 连接的建立采用客户端服务器方式。主动发起连接建立的应用进程叫做客户端，而被动等待连接建立的应用进程叫做服务器。

### TCP 的三次握手

TCP 建立连接的过程叫做握手，握手需要在客户端和服务器之间交换三个 TCP 报文段。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172716.png" alt="image-20201226143243361" style="zoom:67%;" />

最初两端的 TCP 进程都处于 CLOSED 状态。一开始，B 的 TCP 服务器进程先创建传输控制块 TCB，准备接受客户进程的连接请求。然后服务器就处于 LISTEN 状态，等待客户的连接请求。如有，即作出相应。A 的 TCP 客户进程也是首先创建传输控制模块 TCB。

1. 在打算建立 TCP 连接时，A 向 B 发出连接请求报文段，这时首部中的同步位 SYN = 1，同时选择一个初始序号 seq = x 。TCP规定，SYN 报文段不能携带数据，但要消耗一个序号。这时，TCP 客户端进程进入 SYN-SENT 状态。

2. B 在接受到连接请求报文段后，如果同意建立连接，则向 A 发送确认。在确认报文段中应把 SYN 位和 ACK 位都置 1，确认号是 ack = x + 1，同时也为自己选择一个初始序号 seq = y。这个报文段也不能携带数据，但同样需要消耗掉一个序号。这时 TCP 服务器进程进入 SYN-RCVD 状态。

3. TCP 客户端进程收到 B 的确认后，还要向 B 给出确认。确认报文段的 ACK 置 1 。确认号 ack = y + 1，而自己的序号 seq = x + 1 。TCP 的标准规定， ACK 报文段可以携带数据。但如果不携带数据则不消耗序号，下一个数据报文段的序号仍是 seq = x + 1 。这时，TCP 连接已经建立，A 进入 ESTABLISHED 状态。

   当 B 收到 A 的确认后，也进入 ESTABLISHED 状态。

#### 为什么客户端 TCP 最后还要发送一次确认

主要是为了防止已失效的连接请求报文段突然又传到了服务器 TCP，因而产生错误。即 A 发出的第一个连接请求报文段并没有丢失，而是在某些网络节点长时间滞留了，以至延误到连接释放以后的某个时间才来到 B。本来这是一个早就失效的报文段。但 B 收到此失效的连接请求报文段后，就误认为是 A 又发出一次新的连接请求。于是就向 A 发出确认报文段，同意建立连接。假定不采用报文握手，那么只要 B 发出确认，新的连接就建立了。但是实际上 A 并没有发出建立连接的请求，因此不会理睬 B 的确认报文，也不会向 B 发送数据。导致 B 一直等待，浪费资源。

### TCP 的四次挥手

数据传输结束后，通信双方都可以释放连接。现在 A 和 B 都处于 ESTABLISHED 状态。

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172724.png" alt="image-20201226150209725" style="zoom:67%;" />

1. A 的应用进程先向其 TCP 进程发出连接释放报文段，再停止发送数据，主动关闭 TCP 连接。

   A 把连接释放报文段首部的终止控制位 FIN 置 1 ， 其序号 seq = u 。这时 A 进入 FIN-WAIT1 状态，等待 B 的确认。TCP 规定，FIN 报文段即使不携带数据，也要消耗一个序号。

2. B 收到连接释放报文段后即发出确认，确认号 ack = u + 1，而确认报文段自己的序号时 seq = v 。然后 B 就进入 CLOSE-WAIT 状态。TCP 服务器进程这时应通知高层应用进程，因而从 A 到 B 这个方向的连接就释放了，这时 TCP 连接处于半关闭状态，即使 A 已经没有数据要发送来了，但 B 若发送数据，A 仍要接收，即从 B 到 A 这个方向的连接并未关闭。

   A 收到来自 B 的确认后，就进入 FIN-WAIT2 状态.等待 B 发出连接释放报文段。

3. 若 B 已经没有要向 A 发送的数据，其应用进程就通知 TCP 释放连接。这时 B 发出的连接释放报文段把 FIN 置为 1 。假定 B 的序号为 seq = w，B 还必须重复上次已发送过的确认号 ack = u + 1 。这时 B 就进入 LAST-ACK 状态，等待 A 的确认。

4. A 在收到 B 的连接释放报文段后，必须对此发出确认。在确认报文段中把 ACK 置 1，确认号 ack = w + 1，而自己的序号是 seq = u + 1 。然后进入到 TIME-WAIT 状态，现在 TCP 连接还没有释放掉，必须经过时间等待计时器设置的时间 2MSL后，A 才进入到CLOSED 状态。因此，从 A 进入到 TIME-WAIT 状态后，要经过 2MSL 时间才能进入到 CLOSED 状态，才能开始建立下一个新的连接。当 A 撤销相应的 TCB 后，就结束了这次的 TCP 连接。

   B 在收到 A 的确认后进入 CLOSED 状态。

   时间 MSL 叫做最长报文段寿命，TCP 建议设为 2 分钟。Linux 服务器默认设置为 60 秒。

   ```shell
   # xianglin @ manjaro in ~ [15:24:57] 
   $ cat /proc/sys/net/ipv4/tcp_fin_timeout 
   60
   ```

#### 为什么在 TIME-WAIT 状态要等待 2MSL 的时间

1. 为了保证 A 发送的最后一个确认释放连接的 ACK 报文能到达 B。这个报文可能丢失，因而处于 LAST-ACK 状态的 B 收不到对已发送的 FIN + ACK 报文段的确认会重传这个 FIN + ACK 报文段，而 A 就能在 2MSL 时间内收到这个重传的 FIN + ACK 报文段。A 重新确认，重新启动 2MSL 计时器。
2. A 在发送完最后一个 ACK 报文段后，再经过时间 2MSL，就可以使本连接持续时间内所产生的报文段都从网络中消失。这样就避免了下一次新的连接中不会出现这种旧的连接请求报文段。

#### 查看 Linux 服务器下 TCP 连接状态

```shell
➜  ~ netstat -n | awk '/^tcp/{++S[$NF]}END{for(a in S) print a,S[a]}'
ESTABLISHED 17
FIN_WAIT1 1
TIME_WAIT 2
```



## UDP 基础知识

### UDP 的主要特点

* UDP 是无连接的，即发送数据之前不需要建立连接，因此减少了开销和发送数据之前的时延。
* UDP 使用尽最大努力交付，即不保证可靠交付，因此主机不需要维持复杂的连接状态。
* UDP 是面向报文的。UDP 对应用层交下来的报文，既不合并，也不拆分。在添加首部后就向下交付给 IP 层。
* UDP 没有拥塞控制，网络出现拥塞不会使源主机的发送速率降低。适用于 IP 电话、实时视频会议、视频直播等场景。
* UDP 支持一对一、一对多、多对多的交互通信。
* UDP 的首部开销小，只有 8 个字节。

### UDP 的首部格式

用户数据报 UDP 有两个字段：数据字段和首部字段。首部字段占用 8 个字节，由四个字段组成。

![image-20201223214151844](https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172735.png)

* 源端口：源端口号，在需要对方回信时选用，不需要时可用全 0 。
* 目的端口：目的端口号，在终点交付报文时必须使用。
* 长度：UDP 用户数据报的长度，其值最小是 8 （仅有首部）。
* 校验和：检测 UDP 用户数据报在传输中是否有错，有错就丢弃。

*伪首部用于 UDP 计算校验和，既不向下传送也不向上递交*

## HTTP 协议

HTTP 协议定义了浏览器怎样向万维网服务器请求万维网文档，以及服务器怎样把文档传送给浏览器。HTTP 是面向事务的应用层协议，它是万维网上能够可靠地交换文件的重要基础。HTTP 使用了面向连接的 TCP 作为运输层协议，保证了数据的可靠传输。HTTP 不必考虑数据在传输过程中被丢弃后又怎样重传的问题。但是，HTTP 协议本身是无连接的，HTTP 协议是无状态的。

HTTP/1.1 协议的持续连接有两种工作方式。即非流水线方式和流水线方式。非流水线方式是指客户端在收到前一个相应后才能发出下一请求。流水线方式是指客户端在收到 HTTP 的相应报文之前就能够接着发送新的请求报文。

### HTTP 报文格式

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172751.png" alt="img" style="zoom:67%;" />

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172802.png" alt="Response HTTP响应报文格式" style="zoom:67%;" />

### 常见状态码

|      | 类别                             | 原因短语                   |
| :--: | -------------------------------- | -------------------------- |
| 1XX  | Information（信息性状态码）      | 接收的请求正在处理         |
| 2XX  | Success（成功状态码）            | 请求正常处理完毕           |
| 3XX  | Redirection（重定向状态码）      | 需要进行附加操作以完成请求 |
| 4XX  | Client Error（客户端错误状态码） | 服务器无法处理请求         |
| 5XX  | Server Error（服务器错误状态码） | 服务器无法处理请求         |

* 2XX 成功
  * **200 OK**：表示从客户端发来的请求在服务器被正常处理了。
  * 204 No Content：表示服务器接收的请求已被成功处理，但在返回的响应报文中不含实体的主体部分。
  * 206 Partial Content：表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求。响应报文中包含由 `Content-Range` 指定范围的实体内容。
* 3XX
  * 301 moved Permanently：永久性重定向，表示资源已被分配新的 URI。
  * 302 Found：临时性重定向。
  * 304 Not Modified：服务器资源未改变，可以直接使用客户端未过期的缓存。
* 4XX
  * **400 Bad Request**：表示请求报文中存在语法错误。
  * **401 Unauthorized**：表示发送的请求需要通过有 HTTP 认证的认证信息。
  * **403 Forbidden**：表示对请求资源的访问被服务器拒绝了。
  * **404 Not Found**：表示服务器上无法找到请求的资源。
* 5XX
  * **500 Internal Server Error**：表示服务端在执行请求时发生了错误。
  * **503 Service Unavailable**：表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

### GET 请求与 POST 请求的区别

* 从 HTTP 报文层面：GET 请求将信息附加在 URL 上，POST 请求一般是将请求参数放在报文体中。
* 数据库层面：GET 请求符合幂等性和安全性。
* GET 请求可以被缓存、被存储，POST 请求不行。

GET 和 POST 到底有什么区别？ - 大宽宽的回答 - 知乎 https://www.zhihu.com/question/28586791/answer/767316172

### Cookis 和 Session 的区别

HTTP 是无状态的协议。

Cookies：

* 是由服务器发给客户端的特殊信息，以文本的形式存放在客户端。
* 客户端再次请求的时候，会把 Cookies 回发。
* 服务器接收到请求后会解析 Cookies 相关的内容。

Session：

* 服务端的机制，在服务器上保存会话信息。
* 解析客户端请求并操作 sessionId，按需保存状态信息。

Session 的实现方式

* 使用 Cookies 来实现：JSESSIONID。
* 使用 URL 回写来实现。

主要区别

* Cookies 数据存放在客户端的浏览器上，Session 数据放在服务器上
* Session 相对于 Cookies 更安全
* 应当使用 Cookies 来减轻服务器的开销

### HTTP 和 HTTPS 协议的区别

SSL（Security Sockets Layer）

* 为网络通信提供安全及数据完整性的一种安全协议。
* 是操作系统对外的 API，SSL3.0 之后更名为 TLS。
* 采用身份验证和数据加密保证网络通信的安全和数据完整性。

HTTPS 数据传输流程

* 浏览器将支持的加密算法信息发送给服务器；
* 服务器选择一套浏览器支持的加密算法，以证书的形式回发给浏览器；
* 浏览器验证证书合法性，并结合证书公钥加密信息并发送给服务器；
* 服务器使用私钥解密信息，验证哈希，加密响应消息回发给浏览器；
* 浏览器解密响应消息，并对消息进行验真，之后进行加密交互数据。

HTTPS 和 HTTP 协议的区别

* HTTPS 需要到 CA 申请证书。
* HTTPS 密文传输，HTTP 明文传输。
* 连接方式不同，HTTPS 默认使用 443 端口，HTTP 使用 80 端口。
* HTTPS = HTTP + 加密 + 认证 + 完整性保护。

HTTPS 存在的问题：浏览器默认填充 http://，请求需要进行跳转，有被劫持的风险

### 浏览器地址栏键入 URL，按下回车之后经历的流程

1. DNS 解析
2. TCP 连接
3. 发送 HTTP 请求
4. 服务器处理请求并返回 HTTP 报文
5. 浏览器解析并渲染页面
6. 关闭连接

## Socket

Socket 是对 TCP/IP 协议的抽象，是操作系统对外开放的接口。

### Socket 通信过程 

<img src="https://cdn.jsdelivr.net/gh/xianglin2020/gallery@master/202102/172823.jpg" alt="Socket通信过程" style="zoom:50%;" />

### Java 使用 Socket 通信

使用 Socket 编写网络应用程序，分别用 TCP 和 UDP 的方式实现。

* 使用 TCP 实现

  ```java
  /**
   * 使用 Sockets 实现 Tcp服务端
   */
  public class TcpServer {
      public static void main(String[] args) {
          try {
              // 创建一个Sockets服务，并绑定在本机的9509端口上
              ServerSocket serverSocket = new ServerSocket(9509);
              // 循环监听客户端链接
              while (true) {
                  // 有客户端链接才返回
                  Socket socket = serverSocket.accept();
                  // 新建一个服务线程处理任务
                  new TcpService(socket).start();
              }
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  
  
  class TcpService extends Thread {
      private final Socket socket;
  
      TcpService(Socket socket) {
          this.socket = socket;
      }
  
      @Override
      public void run() {
          try {
              // 获取socket对应的输入流和输出流
              InputStream inputStream = this.socket.getInputStream();
              OutputStream outputStream = this.socket.getOutputStream();
              // 读取客户端发送的数据
              byte[] bytes = new byte[1024];
              int read = inputStream.read(bytes);
              String content = new String(bytes, 0, read);
              System.out.println("收到客户端信息：" + content);
              // 返回字符串长度
              outputStream.write(String.valueOf(content.length()).getBytes(StandardCharsets.UTF_8));
              // 关闭连接资源
              outputStream.close();
              inputStream.close();
              this.socket.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  
  /**
   * 使用Sockets 完成Tcp客户端
   */
  public class TcpClient {
      public static void main(String[] args) {
          try {
              // 创建Socket对象，指定ip地址和端口
              Socket socket = new Socket("127.0.0.1", 9509);
              // 使用输出流向服务端发送数据
              OutputStream outputStream = socket.getOutputStream();
              outputStream.write("Hello World!".getBytes(StandardCharsets.UTF_8));
              // 使用输入流接收服务端返回的数据
              InputStream inputStream = socket.getInputStream();
              byte[] bytes = new byte[1024];
              int read = inputStream.read(bytes);
              System.out.println("收到服务端回应：" + new String(bytes, 0, read));
              // 关闭连接资源
              outputStream.close();
              inputStream.close();
              socket.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

* 使用 UDP 实现

  ```java
  /**
   * 使用Socket实现UDP服务端
   */
  public class UdpServer {
      public static void main(String[] args) {
          try {
              // 为本机创建一个DatagramSocket对象，并绑定到9509端口
              DatagramSocket datagramSocket = new DatagramSocket(9509);
              // 创建DatagramPacket对应接收客户端发送的数据
              byte[] bytes = new byte[1024];
              DatagramPacket datagramPacket = new DatagramPacket(bytes, bytes.length);
              // 将客户端发送的数据保存到DatagramPacket对象中
              datagramSocket.receive(datagramPacket);
              // 打印内容
              byte[] data = datagramPacket.getData();
              String content = new String(data, 0, datagramPacket.getLength());
              System.out.println("接收客户端请求：" + content);
  
              // 创建DatagramPacket对象，封装需要发送给客户端的内容
              byte[] sendBytes = String.valueOf(content.length()).getBytes(StandardCharsets.UTF_8);
              // 获取客户端的地址和端口
              DatagramPacket datagramPacket1 = new DatagramPacket(sendBytes, sendBytes.length, datagramPacket.getAddress(), datagramPacket.getPort());
              // 将数据发送给客户端
              datagramSocket.send(datagramPacket1);
              datagramSocket.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  
  /**
   * 使用Socket实现UDP客户端
   */
  public class UdpClient {
      public static void main(String[] args) {
          try {
              // 客户端发送给服务端的内容
              byte[] bytes = "Hello World!".getBytes(StandardCharsets.UTF_8);
              // 获取本机对应的InetAddress对象
              InetAddress localHost = InetAddress.getLocalHost();
              // 创建DatagramSocket对象
              DatagramSocket datagramSocket = new DatagramSocket();
              // 将需要发送的内容封装成DatagramPacket对象，并指定服务端的ip和端口
              DatagramPacket datagramPacket = new DatagramPacket(bytes, bytes.length, localHost, 9509);
              // 发送数据给服务端
              datagramSocket.send(datagramPacket);
  
              // 接收服务端的返回
              byte[] receive = new byte[1024];
              // 创建DatagramPacket对象存储服务端返回的数据
              DatagramPacket datagramPacket1 = new DatagramPacket(receive, receive.length);
              // 接收数据，并存储到DatagramPacket中
              datagramSocket.receive(datagramPacket1);
              // 打印服务端返回的内容
              System.out.println("服务端返回：" + new String(datagramPacket1.getData(), 0, datagramPacket1.getLength()));
              datagramSocket.close();
          } catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```

  
