---
title: 技术学而思之TCP
date: 2016-03-27 21:07:36
categories:
- 技术总结
tags: 
- netty
---
先来两张图镇一下
TCP协议头格式
{% tcp协议头格式 TCP-Header-01.jpg %}
TCP状态图
{% TCP状态图 Tcp_state_diagram_fixed-02.png %}
###### 相关术语
* Sequence number 包序号
* ISN 初始包序列号
RFC793中说，ISN会和一个假的时钟绑在一起，这个时钟会在每4微秒对ISN做加一操作，直到超过2^32，又从0开始
* MSL Maximum Segment Lifetime 是一个TCP分段可以存在于互联网系统中的最大时间
* MSS Maximum segment size TCP分段最大长度
* SACK Selective Acknowledgment 位于options中，表示是否启用SACK
* RST flags中，为1表示出现严重差错。可能需要重现创建TCP连接。还可以用于拒绝非法的报文段和拒绝连接请求。
* SYN flags中，为1表示这是连接请求或是连接接受请求，用于创建连接和使顺序号同步
* FIN flags中，为1表示发送方没有数据要传输了，要求释放连接
* WS 滑动窗口
###### 建立连接
这里可以结合UDP的协议来看，都说tcp是面向连接的，udp是无连接的。tcp的面向连接主要是指tcp维护了一些状态，主要是SN，ACK，FLAGS，WS，这些状态主要目的是保证tcp的可靠性。这些状态在两端都要维护，一旦一方丢失了这些状态那就必须要重新同步状态，所以说是面向连接的。而udp只管在指定端口上接收数据，不管发送者的状态。
这样再来看tcp连接的建立，在建立连接时必须要同步好两端的SN，ACK才能正确的发送数据。这里分三步
* 客户端发送SYN
客户端通知服务端自己的SN，客户端处于SYN-SEND状态
* 服务端回SYN和ACK
服务端通知客户端自己收到请求，告知客户端自己的SN，客户端通过ACK是不是等于自己的SN+1来校验，此时客户端处于ESTABLISHED，服务端处于SYN-RECEIVED，即半连接状态。对于这种状态有三个参数可以调整:
* * tcp_synack_retries
设置重试次数
* * tcp_max_syn_backlog
此参数在java.net.ServerSocket中可设置，可以增大SYN连接数
* * tcp_abort_on_overflow
为0表示如果三次握手第三步的时候全连接队列满了那么server扔掉client 发过来的ack（在server端认为连接还没建立起来），为1表示第三步的时候如果全连接队列满了，server发送一个reset包给client，表示废掉这个握手过程和这个连接（本来在server端这个连接就还没建立起来）。
* 客户端给服务端发送确认ACK
服务端校验ACK是不是等于自己的SN+1确认客户端没忽悠自己，此时服务端也进入ESTABLISHED状态，连接建立
###### 断开连接
主动关闭方发送FIN，自己进入FIN-WAIT-1状态，被动关闭方收到FIN，发送ACK自身进入CLOSE-WAIT状态，等待上层应用处理完数据后，发送FIN，进入LAST-ACK等待最后一个ACK，收到ACK后进入CLOSED
主动关闭方发送完FIN处于FIN-WAIT-1状态时对端收到ACK进入FIN-WAIT-2状态，处于FIN-WAIT-2状态收到FIN进入TIME-WAIT，并发送ACK等待2个MSL后进入CLOSED状态，关于为什么等待有两点：
* * 确保有足够的时间让对端收到了ACK，如果对端收不到ACK会重发FIN，如果此时连接已经关闭会导致链接重置（那过了两个MSL后不是依然会重置链接么？，难道两个MSL后对端不会再尝试重发FIN，或者每次收到FIN会重新计算MSL时间？）
* * 有足够的时间让这个连接的数据不跟之后源端口，目标端口相同的连接的数据发生错乱（SN和ACK的保证不够么？）
