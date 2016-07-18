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


## 6.1.3 关于事件持久性

默认情况下,每当未决事件成为激活的(因为 fd 已经准备好读取或者写入,或者因为超时), 事件将在其回调被执行前成为非未决的。如果想让事件再次成为未决的 ,可以在回调函数中 再次对其调用 event_add()。


然而,如果设置了 EV_PERSIST 标志,事件就是持久的。这意味着即使其回调被激活 ,事件还是会保持为未决状态 。如果想在回调中让事件成为非未决的 ,可以对其调用 event_del ()。

每次执行事件回调的时候,持久事件的超时值会被复位。因此,如果具有 EV_READ|EV_PERSIST 标志,以及5秒的超时值,则事件将在以下情况下成为激活的:

* 套接字已经准备好被读取的时候
* 从最后一次成为激活的开始,已经逝去 5秒

## 6.1.4 信号事件
libevent 也可以监测 POSIX 风格的信号。要构造信号处理器,使用:

```cpp
#define evsignal_new(base, signum, cb, arg) \
    event_new(base, signum, EV_SIGNAL|EV_PERSIST, cb, arg)
```

除了提供一个信号编号代替文件描述符之外,各个参数与 event_new()相同。


###实例
```cpp
struct event *hup_event;
struct event_base *base = event_base_new();

/* call sighup_function on a HUP signal */
hup_event = evsignal_new(base, SIGHUP, sighup_function, NULL);
```

>注意 :信号回调是信号发生后在事件循环中被执行的,所以可以安全地调用通常不能 在 POSIX 风格信号处理器中使用的函数。


**`警告`:不要在信号事件上设置超时,这可能是不被支持的。 [待修正:真是这样的吗?]**

libevent 也提供了一组方便使用的宏用于处理信号事件:

```cpp
#define evsignal_add(ev, tv) \
    event_add((ev),(tv))
#define evsignal_del(ev) \
    event_del(ev)
#define evsignal_pending(ev, what, tv_out) \
    event_pending((ev), (what), (tv_out))
```
evsignal_*宏从2.0.1-alpha 版本开始存在。先前版本中这些宏叫做 signal_add()、signal_del ()等等。

### 关于信号的警告

在当前版本的 libevent 和大多数后端中,每个进程任何时刻只能有一个 event_base 可以监 听信号。如果同时向两个 event_base 添加信号事件,即使是不同的信号,也只有一 个 event_base 可以取得信号。
kqueue 后端没有这个限制。