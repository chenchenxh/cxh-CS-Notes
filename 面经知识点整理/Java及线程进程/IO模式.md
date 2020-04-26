## I/O模式

对于一次IO访问（以read举例），`数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间`。所以说，当一个read操作发生时，它会经历两个阶段：

> 1. 第一阶段：等待数据准备 (Waiting for the data to be ready)。
> 2. 第二阶段：将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)。

- **同步模型（synchronous IO）**
  - 阻塞IO（bloking IO）: 一直等待
  - 非阻塞IO（non-blocking IO）: 隔一段时间就询问
  - 多路复用IO（multiplexing IO）: 不用询问,多一个标志来表示是否准备好.`等待多个socket，能实现同时对多个IO端口进行监听`，
  - 信号驱动式IO（signal-driven IO）:  被动收到信号
- **异步IO（asynchronous IO）**

https://www.jianshu.com/p/486b0965c296