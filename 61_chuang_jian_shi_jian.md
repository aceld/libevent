# 6.1 创建事件

##6.1.1 生成新事件

使用 event_new()接口创建事件。

```cpp
#define EV_TIMEOUT      0x01
#define EV_READ         0x02
#define EV_WRITE        0x04
#define EV_SIGNAL       0x08
#define EV_PERSIST      0x10
#define EV_ET           0x20

typedef void (*event_callback_fn)(evutil_socket_t, short, void *);

struct event *event_new(struct event_base *base, evutil_socket_t fd,
    short what, event_callback_fn cb,
    void *arg);

void event_free(struct event *event);
```

event_new()试图分配和构造一个用于 base 的新的事件。
what 参数是上述标志的集合。

如果 fd 非负,则它是将被观察其读写事件的文件。

事件被激活时, libevent 将调用 cb 函数, 

传递这些参数:文件描述符 fd,表示所有被触发事件的位字段 ,以及构造事件时的 arg 参数。


发生内部错误,或者传入无效参数时, event_new()将返回 NULL。


>所有新创建的事件都处于已初始化和非未决状态 ,调用 event_add()可以使其成为未决的。


要释放事件,调用 event_free()。对未决或者激活状态的事件调用 event_free()是安全 的:在释放事件之前,函数将会使事件成为非激活和非未决的。

###实例：

```cpp
#include <event2/event.h>

void cb_func(evutil_socket_t fd, short what, void *arg)
{
        const char *data = arg;
        printf("Got an event on socket %d:%s%s%s%s [%s]",
            (int) fd,
            (what&EV_TIMEOUT) ? " timeout" : "",
            (what&EV_READ)    ? " read" : "",
            (what&EV_WRITE)   ? " write" : "",
            (what&EV_SIGNAL)  ? " signal" : "",
            data);
}

void main_loop(evutil_socket_t fd1, evutil_socket_t fd2)
{
        struct event *ev1, *ev2;
        struct timeval five_seconds = {5,0};
        struct event_base *base = event_base_new();

        /* The caller has already set up fd1, fd2 somehow, and make them
           nonblocking. */

        ev1 = event_new(base, fd1, EV_TIMEOUT|EV_READ|EV_PERSIST, cb_func,
           (char*)"Reading event");
        ev2 = event_new(base, fd2, EV_WRITE|EV_PERSIST, cb_func,
           (char*)"Writing event");

        event_add(ev1, &five_seconds);
        event_add(ev2, NULL);
        event_base_dispatch(base);
}
```

上述函数定义在 <event2/event.h> 中,首次出现在 libevent 2.0.1-alpha 版本中。 event_callback_fn 类型首次在2.0.4-alpha 版本中作为 typedef 出现。


## 6.1.2 事件标志

* EV_TIMEOUT

这个标志表示某超时时间流逝后事件成为激活的。构造事件的时候,EV_TIMEOUT 标志是 被忽略的:可以在添加事件的时候设置超时 ,也可以不设置。超时发生时,回调函数的 what 参数将带有这个标志。

* EV_READ

表示指定的文件描述符已经就绪,可以读取的时候,事件将成为激活的。

* EV_WRITE

表示指定的文件描述符已经就绪,可以写入的时候,事件将成为激活的。

* EV_SIGNAL
用于实现信号检测,请看下面的 “构造信号事件”节。

* EV_PERSIST
表示事件是“持久的”,请看下面的“关于事件持久性”节。

* EV_ET

表示如果底层的 event_base 后端支持边沿触发事件,则事件应该是边沿触发的。这个标志 影响 EV_READ 和 EV_WRITE 的语义。


从2.0.1-alpha 版本开始,可以有任意多个事件因为同样的条件而未决。比如说,可以有两 个事件因为某个给定的 fd 已经就绪,可以读取而成为激活的。这种情况下,多个事件回调 被执行的次序是不确定的。

>这些标志定义在<event2/event.h>中。除了 EV_ET 在2.0.1-alpha 版本中引入外,所有标志 从1.0版本开始就存在了。



