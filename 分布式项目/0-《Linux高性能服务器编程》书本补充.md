重点看 **第二篇“深入解析高性能服务器编程”**，尤其是网络编程、epoll、线程池几个部分。你的目标不是成为网络库开发者，而是做到：
1. 能自己写 TCP Server/Client；
2. 理解 epoll 为什么高性能；
3. 能把项目升级成事件驱动模型；
4. 面试时能解释网络模块设计。
# 最推荐阅读路线

按照你的项目进度：现在网络模块阶段：
```
第5章
 ↓
第8章
 ↓
第9章
 ↓
第14章
 ↓
第15章
 ↓
第11章
```

如果时间紧：最低版本：
```
5
8
9
15
```
这四章吃透，已经足够支撑你的项目升级。
对于你的秋招项目来说，这本书最大的价值不是让你把所有Linux网络API背下来，而是让你从“会写socket通信”提升到“知道一个工业级服务器为什么这样设计”。你现在的 Master-Worker 项目其实正好处于从简单TCP程序向高性能服务器架构过渡的位置，所以第5、8、9、15章是最值得投入的。

## 第一阶段：必须精读（和当前项目直接相关）

### 第5章 Linux网络编程基础API ⭐⭐⭐⭐⭐

**优先级最高。** 这章基本对应你现在 `network/` 模块。
重点看：
- `socket`
- `bind`
- `listen`
- `accept`
- `connect`
- `close`
- `send/recv`
- socket选项
对应你的代码：
```
network/
├── TCPServer.h
├── TCPClient.h
└── Connection.h
```

你现在写：
```
int fd = socket(...);

bind();

listen();

accept();

recv();

send();
```

这些东西不要只会调用，要知道：
- socket 本质是什么？
- fd 为什么代表一个socket？
- accept为什么返回新的fd？
- TCP连接建立后为什么需要新的socket？
- recv返回0意味着什么？
这些都是面试高频。这一章建议：

**5.1~5.8 全看。**

5.9 跳过。

5.10 地址信息简单看。

5.11 socket选项-重点看：

- SO_REUSEADDR
- SO_RCVBUF
- SO_SNDBUF

因为以后你的Server启动经常会遇到：
```
bind: Address already in use
```
# 第二阶段：项目升级必须看

## 第8章 高性能服务器程序框架 ⭐⭐⭐⭐⭐

这一章非常适合你的项目。
因为你的架构：
```
Client
 |
TCP
 |
Master
 |
Scheduler
 |
Worker
```
本质就是服务器程序。
重点：
### 8.1 服务器模型
理解：
- C/S模型
- 多进程服务器
- 多线程服务器
- Reactor模型
### 8.2 服务器编程框架
重点理解：
```
I/O处理单元
      |
      |
逻辑单元
      |
      |
存储单元
```

对应你的：

```
Network
   |
Message
   |
TaskManager
   |
Scheduler
```

### 8.3 I/O模型

必须看。理解：
- 阻塞IO
- 非阻塞IO
- IO复用
- 异步IO
因为后面 epoll 就建立在这里。

### 8.4 两种事件处理模式

重点：
- Reactor
- Proactor

你的项目未来升级：
现在：
```
一个线程recv
一个线程处理
```
未来：
```
epoll监听socket
事件发生
交给Worker线程处理
```

就是 Reactor。
### 8.5 并发模式

重点：
- 半同步/半异步
- 领导者/追随者

不用深入实现。理解思想即可。

### 8.6 有限状态机

这个对你的任务状态特别有用。
你的：
```
PENDING

RUNNING

DONE

FAILED
```
其实就是状态机。这一章建议：
**8.1~8.6 全看，但不用死磕代码。**

# 第三阶段：epoll核心

## 第9章 I/O复用 ⭐⭐⭐⭐⭐

这个是你项目简历亮点：

> 基于 epoll 实现高并发 TCP 通信模型

所以必须看。重点：
## 9.1 select

了解即可。

## 9.2 poll

了解即可。

## 9.3 epoll

重点精读。必须理解：
### epoll三个函数：

```
epoll_create

epoll_ctl

epoll_wait
```

以及：

```
socket fd

注册到epoll

等待事件

处理事件
```

重点理解：

LT模式：

```
数据没读完
一直通知
```

ET模式：

```
只通知一次
必须一次读完
```

你的项目以后：Master监听：

```
worker连接

client提交任务

heartbeat
```

全部可以进入epoll。

# 第四阶段：线程和线程池

## 第14章 多线程编程 ⭐⭐⭐⭐

你的项目：Master：

```
accept线程

scheduler线程

heartbeat线程

worker通信线程
```

Worker：

```
通信线程

执行线程
```

所以需要。

重点：

14.2 创建线程

14.5 互斥锁

14.6 条件变量

尤其：
条件变量：

```
condition_variable
```
对应：
任务队列：

```
Scheduler

      |
      v

TaskQueue

      |
      v

Worker线程等待
```

你的线程池一定会用。

## 第15章 进程池和线程池 ⭐⭐⭐⭐⭐

非常推荐。

因为你的项目规划里面本来就有：

> ThreadPool + Async Logger

重点：

15.1理解：为什么需要线程池？

15.5 重点看：

半同步/半反应堆线程池。

虽然代码不用完全照搬。

理解：

```
主线程:
接受连接

↓

工作线程:
处理任务
```

这个思想和你的：

```
Master
 |
Worker线程池
```

很接近。

# 第五阶段：辅助阅读

## 第11章 定时器 ⭐⭐⭐⭐

你的项目有：

```
heartbeat检测

Worker超时

任务timeout
```

所以有价值。

重点：11.1，11.3，11.4

尤其：定时器管理：

```
Worker lastHeartbeat

当前时间-lastHeartbeat > timeout

worker dead
```

## 第16章 调试测试 ⭐⭐⭐

后期看。重点：

- gdb
- 压力测试

你的项目后期：

测试：

```
100 worker连接

1000 task提交
```

会用到。


# 其他的直接跳过

