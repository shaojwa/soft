https://tldp.org/HOWTO/html_single/TCP-Keepalive-HOWTO/

#### 什么是TCP的长连接
TCP的连接，也就会socket抽象层，在没有数据发送的时候，是需要发包来保活的。

#### 多久就需要心跳保活呢？
我们从TCP的状态图可以看到，TCP的连接并会自动关闭，所以client 和server测的socket可以一致保持Established状态。
但是，可能此时这个连接已经断开，比如网线断掉，client再想发送数据，会一致超时，所以只是建立是没有用的，我们需要确保可用。
而可用的状态是需要检测的，所以我们需要检测alive，而一般来说，这个功能是通过心跳来实现的。

#### 如何实现保活
这个是Linux系统内建的机制，也就是说linux操作系统TCP协议栈已经实现的。针对保活，有专门的保活报文KeepAlive。
但是这个不是默认配置。如果需要使用保活机制，那么需要调用`setsockopt`来设定。

#### KeepAlive保活的缺陷
这个机制只能探测连接的死活，但是无法确定双方的工作状态，比如服务器负载很高，已经无法响应状态，这个还KA无法检测出来的。
所以，很多时候，应用层的心跳是更完善的检测机制，更具有灵活新，比如时间间隔的动态自适应，触发的时间，以及可以定制报文。

#### 我们如何查看？
sysctl中关于keepalive的内核参数有三个：
```
net.ipv4.tcp_keepalive_time
```
表示，最后一份数据发送完成之后，间隔多久，TCP协议开始发送探测（probe）报文，一般是7200秒。
```
net.ipv4.tcp_keepalive_intvl
```
TCP协议连续两次保活探测的时间间隔，默认是75秒。最后一个是：
```
net.ipv4.tcp_keepalive_probes
```
表示客户端在发送测报文且没有收到回应的次数达到这个设定值时，会把与服务器的连接认为断开，并通知引用层。