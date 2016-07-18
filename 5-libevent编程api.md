#4 event_base

>本章主要来源《libevent参考手册（中文版）》。



**使用 libevent 函数之前需要分配一个或者多个 event_base 结构体。每个 event_base 结构 体持有一个事件集合,可以检测以确定哪个事件是激活的。**


如果设置 event_base 使用锁,则可以安全地在多个线程中访问它 。然而,其事件循环只能 运行在一个线程中。如果需要用多个线程检测 IO,则需要为每个线程使用一个 event_base。

每个 event_base 都有一种用于检测哪种事件已经就绪的 “方法”,或者说后端。可以识别的方法有:

* **select **
* poll
* **epoll**
* kqueue 
* devpoll 
* evport 
* win32


