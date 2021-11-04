# IPC

<!--TOC-->



## Overview
**进程间通信 (Inter-process Communication, IPC)**: 两个(或多个)不同的进程，通过内核或其他共享资源进行通信，来传递控制信息或数据。

![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124736.png)

### 直接通信和间接通信
* 直接通信：基于共享内存
	
	> 基于共享内存的消息传递核心抽象仍然是消息，而直接使用共享内存则不存在消息的抽象。
	
* 间接通信：基于操作系统内核，每次发送消息经过一个信箱
	![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124737.png)
### 消息传递的同步与异步
消息可以是阻塞的也可以是非阻塞的
* 阻塞通常被认为是同步通信，发送者/接收者一直处于阻塞状态，直到消息发出/到来。
* 非阻塞通常被认为是异步通信，发送者/接收者不等待操作结果，直接返回。

## 通信方式
### 管道
管道(Pipe)：a communication medium between two or more related or interrelated processes.
![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124738.png)

* 间接消息传递方式
* **单向的IPC**
* 数据类型：字节流
* 分为[匿名管道](#匿名管道)和[命名管道](#命名管道)

例如常见的Shell命令：`ls | grep`
缺陷：

* 固定的缓冲区间，分配过大资源容易造成浪费
* 两个端口，最多对应两个进程
#### 匿名管道 
通常用于建立**父子进程**或者**兄弟进程**的连接，在创建的同时进程会拿到**两个文件描述符**用来使用它。
管道是通过调用**pipe** 函数创建的，fd[0] 用于读，fd[1] 用于写:

```c
#include <unistd.h>
int pipe(int fd[2]);
```

过程：
1. 父进程首先通过pipe创建对应的管道的两端，然后通过fork创建子进程，子进程继承包括管道端口的**文件描述符**。
2. 在完成继承后，父子进程关闭多余的端口。
![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124740.png)
#### 命名管道
命名管道可以实现任意两个进程的通信，也称为**FIFO**。
通过**mkfifo**命令创建，指定一个**全局的文件名**为管道名。

```c++
#include <sys/stat.h>
int mkfifo(const char *path, mode_t mode);
int mkfifoat(int fd, const char *path, mode_t mode);
```
FIFO 常用于C/S架构中，FIFO 用作汇聚点，在客户进程和服务器进程之间传递数据。
![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/2ac50b81-d92a-4401-b9ec-f2113ecc3076.png)

### 消息队列
消息队列: 以链表的方式组织**消息**(类型 + 数据)。

![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124742.png)

* **FIFO**：在队列尾部写入消息，在队首读取消息。

* **允许按照类型查询**: Recv(A, **type**, message)，即根据消息类型有**选择性接收消息**，而不像 FIFO 那样只能默认地接收。
	* 类型为0时返回第一个消息 (FIFO)
	*  类型有值时按照类型查询并返回第一个类型为type的消息


* 消息队列可以**独立于进程**存在，并允许任意数量的进程连接到统一队列上。

### 信号量
信号量一般来说只有一个共享的整型**计数器**，该计数器由内核维护和操作，通常用于**进程同步**。

### 信号

信号(signal)是一个**整型值**，作为一个**异步的通知**发送给某个进程或同进程的某个线程。

当信号发送到某个进程中时，OS会中断该进程的正常流程，并进入相应的**信号处理函数(signal handler)**执行操作，完成后再回到中断的地方继续执行。实际上属于**软件层面的中断**。

信号的响应动作：

* 忽略信号(Ign)

* 结束进程(Term)

* 结束进程并保存内存信息(Core)
* 停止进程(Stop)
* 继续运行进程(Cont)

信号的响应动作可以被修改，但一个进程内的所有线程的响应动作都是相同的。

> 以下Shell的操作属于信号的应用：
>
> - Ctrl-C 发送 INT signal (SIGINT)，通常导致进程结束
> - Ctrl-Z 发送 TSTP signal (SIGTSTP); 通常导致进程挂起(suspend)
> - `kill -9 pid` 会发送 `SIGKILL`信号给进程



### 共享内存

![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424124741.png)


系统内核为多个进程所在的**虚拟内存空间**映射**相同的物理内存空间**。

特点：数据不需要在进程之间复制，是**速度最快**的 IPC。但需要使用[信号量](#信号量)等同步机制。

### 套接字
套接字(Socket)用于不同机器间的进程通信。一般处于计算机网络中的[传输层](../Net/传输层.md)。

## 参考
* [上海交通大学SE-315 · 操作系统（2020春)](https://ipads.se.sjtu.edu.cn/courses/os/2020/schedule.shtml)

* [CS-Notes-计算机操作系统 - 进程管理](http://www.cyc2018.xyz/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9F%BA%E7%A1%80/%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E5%9F%BA%E7%A1%80/%E8%AE%A1%E7%AE%97%E6%9C%BA%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%20-%20%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86.html#_2-fifo)

