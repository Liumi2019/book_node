# 学习 Binder

## 1. 什么是Binder
Binder是使用在Android上的一种跨进程通信机制。 众所周知Android中一个进程不能直接访问另一个进程的内存（进程隔离）。为了解决这种问题，进程与进程之间的通信需要一个桥梁。比如你的应用想要知道现在手机还剩下多少电量，此时就需要使用系统服务PowerManager，这就是一个典型的Binder机制使用。