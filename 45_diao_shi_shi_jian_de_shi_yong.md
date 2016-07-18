# 9.5 调试事件的使用

libevent 可以检测使用事件时的一些常见错误并且进行报告。这些错误包括:

* 将未初始化的 event 结构体当作已经初始化的

* 试图重新初始化未决的 event 结构体



跟踪哪些事件已经初始化需要使用额外的内存和处理器时间 ,所以只应该在真正调试程序的 时候才启用调试模式。

```cpp
void event_enable_debug_mode(void);
```

必须在创建任何 event_base 之前调用这个函数。

如果在调试模式下使用大量由 event_assign(而不是 event_new)创建的事件,程序可能 会耗尽内存,这是因为没有方式可以告知 libevent 由 event_assign 创建的事件不会再被使 用了(可以调用 event_free 告知由 event_new 创建的事件已经无效了 )。如果想在调试时 避免耗尽内存,可以显式告知 libevent 这些事件不再被当作已分配的了:

```cpp
void event_debug_unassign(struct event *ev);
```

没有启用调试的时候调用 event_debug_unassign 没有效果。


### 实例

```cpp
#include <event2/event.h>
#include <event2/event_struct.h>

#include <stdlib.h>

void cb(evutil_socket_t fd, short what, void *ptr)
{
    /* We pass 'NULL' as the callback pointer for the heap allocated
     * event, and we pass the event itself as the callback pointer
     * for the stack-allocated event. */
    struct event *ev = ptr;

    if (ev)
        event_debug_unassign(ev);
}

/* Here's a simple mainloop that waits until fd1 and fd2 are both
 * ready to read. */
void mainloop(evutil_socket_t fd1, evutil_socket_t fd2, int debug_mode)
{
    struct event_base *base;
    struct event event_on_stack, *event_on_heap;

    if (debug_mode)
       event_enable_debug_mode();

    base = event_base_new();

    event_on_heap = event_new(base, fd1, EV_READ, cb, NULL);
    event_assign(&event_on_stack, base, fd2, EV_READ, cb, &event_on_stack);

    event_add(event_on_heap, NULL);
    event_add(&event_on_stack, NULL);

    event_base_dispatch(base);

    event_free(event_on_heap);
    event_base_free(base);
}

```