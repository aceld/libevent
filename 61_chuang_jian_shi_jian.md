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