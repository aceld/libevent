# 5.1 运行循环

一旦有了一个已经注册了某些事件的 event_base(关于如何创建和注册事件请看下一节 ), 就需要让 libevent 等待事件并且通知事件的发生。

```cpp
#define EVLOOP_ONCE             0x01
#define EVLOOP_NONBLOCK         0x02
#define EVLOOP_NO_EXIT_ON_EMPTY 0x04

int event_base_loop(struct event_base *base, int flags);
```

默认情况下,event_base_loop()函数运行 event_base 直到其中没有已经注册的事件为止。执行循环的时候 ,函数重复地检查是否有任何已经注册的事件被触发 (比如说,读事件 的文件描述符已经就绪,可以读取了;或者超时事件的超时时间即将到达 )。如果有事件被触发,函数标记被触发的事件为 “激活的”,并且执行这些事件。


在 flags 参数中设置一个或者多个标志就可以改变 event_base_loop()的行为。如果设置了 EVLOOP_ONCE ,循环将等待某些事件成为激活的 ,执行激活的事件直到没有更多的事件可以执行,然会返回。如果设置了 EVLOOP_NONBLOCK,循环不会等待事件被触发: 循环将仅仅检测是否有事件已经就绪,可以立即触发,如果有,则执行事件的回调。

完成工作后,如果正常退出, event_base_loop()返回0;如果因为后端中的某些未处理 错误而退出,则返回 -1。


为帮助理解,这里给出 event_base_loop()的算法概要:

```cpp
while (any events are registered with the loop,
        or EVLOOP_NO_EXIT_ON_EMPTY was set) {

    if (EVLOOP_NONBLOCK was set, or any events are already active)
        If any registered events have triggered, mark them active.
    else
        Wait until at least one event has triggered, and mark it active.

    for (p = 0; p < n_priorities; ++p) {
       if (any event with priority of p is active) {
          Run all active events with priority of p.
          break; /* Do not run any events of a less important priority */
       }
    }

    if (EVLOOP_ONCE was set or EVLOOP_NONBLOCK was set)
       break;
}
```

为方便起见,也可以调用

```cpp
int event_base_dispatch(struct event_base *base);
```

event_base_dispatch ()等同于没有设置标志的 event_base_loop ( )。所以, event_base_dispatch ()将一直运行,直到没有已经注册的事件了,或者调用 了 event_base_loopbreak()或者 event_base_loopexit()为止。


这些函数定义在<event2/event.h>中,从 libevent 1.0版就存在了。