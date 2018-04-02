# 名词解释

- I/O多路复用(IO multiplexing): 通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。**需要注意的是，这和http2的多路复用不是一回事**。

- select,pselect,poll,epoll: 支持I/O多路复用，在读写事件就绪后自己负责进行读写的话其实也还是同步I/O，而异步I/O则无需自己负责进行读写，实现异步I/O只需要把读写的数据从内核拷贝到用户空间。

- 文件描述符(file descriptor/fd): 打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。当读写数据准备好时，变成就绪状态。

- 水平触发(level triggered/lt): 同时支持block和non-block socket。在这种模式下，当内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的fd进行IO操作。如果你这一次不对该fd作任何操作或者有未完成的操作(数据未读取完毕)，下一次内核还是会继续通知你的。

- 边缘触发(edge triggered/et): 只支持non-block socket。在这种模式下，如果你在第一次不对该就绪的fd作任何操作或者有未完成的操作(数据未读取完毕)，下一次内核不会再通知你的，而是认为你已经完成操作。

# select

`select(int max_fd, fd_set *readset, fd_set *writeset, fd_set *exceptset, struct timeval *timeout)`

参数说明:
  1. max_fd: 监视文件描述符最大数量，32位上限1024，64位上限2048
  2. fd_set *readset, *writeset, *exceptset: 分别为监视到的read,write,expect事件，之后可以遍历这三个数组获取就绪的文件描述符
  3. timeout: 超时时间

优点:
  1. select目前几乎在所有的平台上支持

缺点:
  1. 单个进程所打开的FD是有一定限制的，它由FD_SETSIZE设置，默认值是1024。
  2. 对socket进行扫描时是线性扫描，即采用轮询的方法。当套接字比较多的时候，每次select()都要通过遍历FD_SETSIZE个Socket来完成调度，不管哪个Socket是活跃的，都遍历一遍。这会浪费很多CPU时间。
  3. 需要维护一个用来存放大量fd的数据结构，这样会使得用户空间和内核空间在传递该结构时复制开销大。

# pselect

`int pselect(int maxfdp1,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,const struct timespec *tsptr,const sigset_t *sigmask);`

参数说明:
  1. max_fd: 同select
  2. fd_set *readset, *writeset, *exceptset: 同select
  3. tsptr: 与select的timeval相比，timespec结构以秒和纳秒表示超时值，如果平台支持这样精细的粒度，那么timespec就提供了更精准的超时时间
  4. sigmask: 指向一信号屏蔽字，在调用pselect时，以原子操作的方式安装该信号屏蔽字，在返回时恢复以前的信号屏蔽字

# poll

`int poll(struct pollfd fds[], nfds_t nfds, int timeout);`

```
typedef struct pollfd {
        int fd;                         // 需要被检测或选择的文件描述符
        short events;                   // 对文件描述符fd上感兴趣的事件
        short revents;                  // 文件描述符fd上当前实际发生的事件
}
```

参数说明:
  1. fds[]: 数据结构如上，代替select中的三个指针参数，由pollfd这个结构中的events来区分感兴趣的事件
  2. nfds: 记录数组fds中描述符的总数量，类似于select中的max_fd，但是没有上限
  3. timeout: 超时时间

优点:
  1. 它没有最大连接数的限制，原因是它是基于链表来存储的。
  2. 支持水平触发。

缺点:
  1. 与select一样，需要通过遍历文件描述符来获取已经就绪的socket。
  2. 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。

# epoll

`int epoll_create(int size);`

函数说明: 创建一个epoll句柄，调用成功时返回一个epoll句柄描述符，失败时返回-1。

参数说明:
  1. size: 内核要监听的描述符数量

`int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);`

函数说明: 注册要监听的事件类型

参数说明:
  1. epfd: epoll_create返回的epoll句柄
  2. op： 操作类型，包括从epfd添加，修改，删除已注册的fd或者fd上感兴趣的事件
  3. fd: 要监听的描述符
  4. event: 要监听的事件

`int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);`
    
函数说明: 等待事件的就绪，成功时返回就绪的事件数目，调用失败时返回-1，等待超时返回0

参数说明:
  1. epfd: epoll_create返回的epoll句柄
  2. events: 从内核得到的就绪事件集合
  3. maxevents: 内核events的大小
  4. timeout: 等待的超时时间

优点:
  1. 支持水平触发和边缘触发。
  2. epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。    
  3. 没有最大并发连接的限制，能打开的FD的上限远大于1024(1G的内存上能监听约10万个端口)。
  4. 使用了callback方式，只有活跃可用的fd才会调用callback函数，相对于select和poll，不是轮询的方式，不会随着fd数目的增加效率下降。
  5. 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。

# 三种方式的对比

1. 支持一个进程所能打开的最大连接数

![](./imgs/epoll1.png)

2. FD剧增后带来的IO效率问题

![](./imgs/epoll2.png)

3. 消息传递方式

![](./imgs/epoll3.png)

## 总结

1. 表面上看epoll的性能最好，但是在连接数少并且连接都十分活跃的情况下，select和poll的性能可能比epoll好，毕竟epoll的通知机制需要很多函数回调。
2. select低效是因为每次它都需要轮询。但低效也是相对的，视情况而定，也可通过良好的设计改善。