## TCP的三次握手

两个教程：https://zhuanlan.zhihu.com/p/53374516

https://blog.csdn.net/whuslei/article/details/6667471

#### 为什么要三次握手

为了防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误。

![img](https://img-blog.csdn.net/20170104214009596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2h1c2xlaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

自己对ACK和Seq的理解，对于每一方来说，我的**Seq代表着我发给你的顺序，而ACK代表着我现在接受到了哪一些**（如ack=k+1，说明第k个已经收到）

#### time wait和Close Wait知识点

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200323235326410.gif)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200323235336455.gif)

## 为什么要四次握手

因为TCP是全双工的，每两次握手断开一条连接。

## 如果TCP建立连接的ACK包丢失

https://blog.csdn.net/zerooffdate/article/details/79359726

当Client端收到Server的SYN+ACK应答后，其状态变为ESTABLISHED，并发送ACK包给Server；

如果此时ACK在网络中丢失，那么Server端该TCP连接的状态为SYN_RECV，并且**依次等待3秒、6秒、12秒后重新发送SYN+ACK包，以便Client重新发送ACK包**。

Server重发SYN+ACK包的次数，可以通过设置/proc/sys/net/ipv4/tcp_synack_retries修改，默认值为5。

如果重发指定次数后，仍然未收到ACK应答，那么一段时间后，Server自动关闭这个连接。

但是Client认为这个连接已经建立，如果Client端向Server写数据，Server端将以RST包响应，方能感知到Server的错误。