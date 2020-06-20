---
title: TCP
date: 2019-08-26
tags:
 - 计算机网络
categories:
 - 计算机网络
---

## TCP概要

### TCP包头
![gUy4v7.png](https://t1.picb.cc/uploads/2019/10/10/gUy4v7.png)

**源端口号和目标端口号不可少，表示发送给哪个应用**

**包的序号是为了*解决乱序*的问题，确定发送的顺序**

**确认序号可以解决*不丢包*的问题，对发出去的包进行确认**

**状态位：SYN发起连接、ACK回复、RST重新连接、FIN结束连接等。TCP是面向连接的，因而双方要*维护连接*的状态，带状态位的包的发送，会引起双方的状态变更**

**窗口大小：TCP要做*流量控制*，通信双方声明的窗口标识自己当前的处理能力**

**TCP还会做*拥塞控制*，根据情况控制发送的速度**

### TCP三次握手
![gUyCjN.jpg](https://t1.picb.cc/uploads/2019/10/10/gUyCjN.jpg)

***三次握手除了双方建立连接外，主要还为了沟通一件事情，就是TCP包的序号的问题***

1. **客户端和服务端都处于CLOSE状态**
2. **服务端主动监听某个端口，处于LISTEN状态**
3. **客户端主动发起连接SYN，之后处于SYN_SENT状态，同时告诉服务器传输的起始序列号**
4. **服务端收到发起的连接，返回SYN，并且ACK客户端的SYN，之后处于SYN_RCVD状态，同时告诉客户端传输的起始序列号**
5. **客户端收到服务端发送的SYN和ACK之后，发送ACK的ACK，之后处于ESTABLISHED状态，因为一发一收都成功了**
6. **服务端收到ACK的ACK之后，处于ESTABLISHED状态，因为一发一收也成功了**

### TCP四次挥手
![gUylVc.jpg](https://t1.picb.cc/uploads/2019/10/10/gUylVc.jpg)

1. **A发送关闭请求FIN，之后进入FIN_WAIT_1的状态**
2. **B收到消息后，发送ACK应答，然后进入CLOSED_WAIT状态**
3. **A收到B的ACK后则进入FIN_WAIT_2状态**
4. **B发送关闭请求FIN,进入LAST_ACK状态**
5. **A收到关闭请求后，发送ACK，然后从FIN_WAIT_2状态结束，进入TIME_WAIT状态，等待时间为2MSL（MSL：报文最大生存时间）**
6. **B在收到A响应的ACK之后进入CLOSED状态**
7. **A在等待2MSL之后进入CLOSED状态**