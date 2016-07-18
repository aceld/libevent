# 6.3 事件的优先级

多个事件同时触发时,libevent 没有定义各个回调的执行次序。可以使用优先级来定义某些事件比其他事件更重要。

在前一章讨论过,每个 event_base 有与之相关的一个或者多个优先级。在初始化事件之后, 但是在添加到 event_base 之前,可以为其设置优先级。

```cpp
int event_priority_set(struct event *event, int priority);
```

事件的优先级是一个在 0和 event_base 的优先级减去1之间的数值。成功时函数返回 0,失 败时返回-1。

多个不同优先级的事件同时成为激活的时候 ,低优先级的事件不会运行 。libevent 会执行高优先级的事件,然后重新检查各个事件。只有在没有高优先级的事件是激活的时候 ,低优先级的事件才会运行。

###实例：
```cpp
#include <event2/event.h>

void read_cb(evutil_socket_t, short, void *);
void write_cb(evutil_socket_t, short, void *);

void main_loop(evutil_socket_t fd)
{
  struct event *important, *unimportant;
  struct event_base *base;

  base = event_base_new();
  event_base_priority_init(base, 2);
  /* Now base has priority 0, and priority 1 */
  important = event_new(base, fd, EV_WRITE|EV_PERSIST, write_cb, NULL);
  unimportant = event_new(base, fd, EV_READ|EV_PERSIST, read_cb, NULL);
  event_priority_set(important, 0);
  event_priority_set(unimportant, 1);

  /* Now, whenever the fd is ready for writing, the write callback will
     happen before the read callback.  The read callback won't happen at
     all until the write callback is no longer active. */
}
```

如果不为事件设置优先级,则默认的优先级将会是 event_base 的优先级数目除以2。