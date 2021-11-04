

## I/O模型

一个输入操作通常包括两个阶段：

- 等待数据准备好
- 从内核向进程复制数据

对于一个套接字上的输入操作，第一步通常涉及等待数据从网络中到达。当所等待数据到达时，它被复制到内核中的某个缓冲区。第二步就是把数据从内核缓冲区复制到应用进程缓冲区。

### BIO

**同步阻塞IO（Blocking IO）：**应用进程被阻塞，直到数据从内核缓冲区复制到应用进程缓冲区中才返回。

![image-20210921095625763](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210921095628.png)

### NIO

**同步非阻塞I/O（Non-blocking IO）**：应用进程执行系统调用之后，若所等待数据到达数据没到达，则内核返回一个错误码，否则返回成功信息。应用进程可以通过轮询（polling）的方式不断地执行系统调用来获知数据是否到达。

![image-20210921095602006](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210921095604.png)



### I/O 复用

**I/O多路复用（IO Multiplexing）**:**一个进程**可以监视多个**Socket**，==<u>进程会阻塞直到其中一个Socket变为可读</u>==，可以让单个进程具有处理多个 I/O 事件的能力。

* 如果一个 Web 服务器没有 I/O 复用，那么每一个 Socket 连接都需要创建一个线程去处理。如果同时有几万个连接，那么就需要创建相同数量的线程。
* 相比于多进程和多线程，I/O 复用不需要进程线程创建和切换的开销，系统开销更小。

![image-20210921095537686](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210921100033.png)

### 信号驱动 I/O

**信号驱动 I/O（signal driven I/O）**：应用进程执行系统调用之后，内核立即返回，应用进程可以继续执行，<u>内核在数据到达时会向应用进程发送 SIGIO 信号</u>。

![image-20210921095511279](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210921095520.png)



### AIO

**异步I/O（Asychronous IO）：**应用进程执行 aio_read 系统调用会立即返回，应用进程可以继续执行，不会被阻塞，内核会在所有操作完成之后向应用进程发送信号。

异步 I/O 与信号驱动 I/O 的区别在于，异步 I/O 的信号是通知应用进程 I/O 完成，而信号驱动 I/O 的信号是通知应用进程可以开始 I/O。

![image-20210921095442495](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210921095459.png)



## I/O多路复用实现方式

### select

实现方式：

将已连接的 Socket 都放到用BitsMap存储的**文件描述符集合**，然后调用 select 函数将文件描述符集合**拷贝**到内核空间，让内核<u>通过**遍历**文件描述符集合的方式</u>检测是否有**网络事件**发生。

当检测到有事件产生后，<u>将此 Socket 标记为可读或可写</u>，然后再把整个文件描述符集合**拷贝**回用户态里，然后用户态还需要再通过**遍历**的方法找到可读或可写的 Socket，最后再对其处理。

缺点：

* 需要遍历整个文件描述符集合来找到可读或可写的 Socket，时间复杂度为 O(n)
* 需要在用户态与内核态之间拷贝文件描述符集合

### poll

实现方式和select类似。

结构：链表，不受文件描述符个数限制（会受到系统文件描述符限制）

### epoll

1. epoll 使用**红黑树(RB-tree)**存放被监控的**文件描述符(file descriptor)**，有较高的增删改查效率。
2. 在 epoll 实例上通过 `epoll_ctl()` 函数注册事件时，epoll 会将该事件添加到 epoll 实例的红黑树上并在内核注册一个回调函数，当事件发生时通过回调函数能够将事件添加到**就绪链表**中，调用`epoll_wait`时将就绪链表从内核空间**复制**就绪链表到用户空间，**减少了内核和用户空间大量的数据拷贝和内存分配**。

![preview](https://pic3.zhimg.com/v2-c7246e74cc64c786360ae12e266e0362_r.jpg)

#### 程序接口

```
int epoll_create(int size);
```

在内核中创建`epoll`实例并返回一个`epoll`文件描述符。

```
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

向 epfd 对应的内核`epoll` 实例**添加**(`EPOLL_CTL_ADD`)、**修改**(`EPOLL_CTL_MOD`)或**删除**(`EPOLL_CTL_DEL`)文件描述符上事件的监听。

```
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

当 timeout 为 0 时，epoll_wait 永远会立即返回。而 timeout 为 -1 时，epoll_wait 会一直阻塞直到任一已注册的事件变为就绪。当 timeout 为一正整数时，epoll 会阻塞直到计时 timeout 毫秒终了或已注册的事件变为就绪。

#### 事件触发模式

* **水平触发（level-triggered，LT）**：当新的事件发生时，`epoll_wait`在事件状态未变更前将不断被触发。
* **边缘触发（edge-triggered，ET）**：当新的事件发生时，`epoll_wait`仅会在新的事件首次被加入`epoll`队列时返回。因此程序需要一次性将内核缓冲区的数据读取完。

水平触发是epoll的默认模式，select和poll也有，但边缘触发是epoll独有的。





