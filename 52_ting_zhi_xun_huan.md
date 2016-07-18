# 5.2 停止循环

如果想在移除所有已注册的事件之前停止活动的事件循环,可以调用两个稍有不同的函数 。


```cpp
int event_base_loopexit(struct event_base *base,
                        const struct timeval *tv);
int event_base_loopbreak(struct event_base *base);
```

**event_base_loopexit()**让 event_base 在给定时间之后停止循环。如果 tv 参数为 NULL, event_base 会立即停止循环,没有延时。

如果 event_base 当前正在执行任何激活事件的回调,则回调会继续运行,直到运行完所有激活事件的回调之才退出。

**event_base_loopbreak ()**让 event_base 立即退出循环。它与 event_base_loopexit (base,NULL)的不同在于,如果 event_base 当前正在执行激活事件的回调 ,它将在执行完当前正在处理的事件后立即退出。


>注意 event_base_loopexit(base,NULL) 和 event_base_loopbreak(base) 在事件循环没有运行时的行为不同:前者安排下一次事件循环在下一轮回调完成后立即停止(就好像带 EVLOOP_ONCE 标志调用一样);后者却仅仅停止当前正在运行的循环,如果事件循环没 有运行,则没有任何效果。

这两个函数都在成功时返回 0,失败时返回 -1。

###实例：
```cpp


#include <event2/event.h>

/* Here's a callback function that calls loopbreak */
void cb(int sock, short what, void *arg)
{
    struct event_base *base = arg;
    event_base_loopbreak(base);
}

void main_loop(struct event_base *base, evutil_socket_t watchdog_fd)
{
    struct event *watchdog_event;

    /* Construct a new event to trigger whenever there are any bytes to
       read from a watchdog socket.  When that happens, we'll call the
       cb function, which will make the loop exit immediately without
       running any other active events at all.
     */
    watchdog_event = event_new(base, watchdog_fd, EV_READ, cb, base);

    event_add(watchdog_event, NULL);

    event_base_dispatch(base);
}

```

###实例2：执行事件循环10秒，然后退出

```cpp
#include <event2/event.h>

void run_base_with_ticks(struct event_base *base)
{
  struct timeval ten_sec;

  ten_sec.tv_sec = 10;
  ten_sec.tv_usec = 0;

  /* Now we run the event_base for a series of 10-second intervals, printing
     "Tick" after each.  For a much better way to implement a 10-second
     timer, see the section below about persistent timer events. */
  while (1) {
     /* This schedules an exit ten seconds from now. */
     event_base_loopexit(base, &ten_sec);

     event_base_dispatch(base);
     puts("Tick");
  }
}
```


有时候需要知道对 event_base_dispatch()或者 event_base_loop()的调用是正常退出 的,还是因为调用 event_base_loopexit()或者 event_base_break()而退出的。可以调 用下述函数来确定是否调用了 loopexit 或者 break函数。

```cpp
int event_base_got_exit(struct event_base *base);
int event_base_got_break(struct event_base *base);
```

这两个函数分别会在循环是因为调用 event_base_loopexit()或者 event_base_break()而退出的时候返回 true,否则返回 false。下次启动事件循环的时候,这些值会被重设。

这些函数声明在<event2/event.h>中。


event_break_loopexit()函数首次在 libevent 1.0c 版本 中实现;
event_break_loopbreak()首次在 libevent 1.4.3版本中实现。
