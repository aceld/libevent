# 7.5.2 操作回调、水位和启用/禁用

```cpp
typedef void (*bufferevent_data_cb)(struct bufferevent *bev, void *ctx);
typedef void (*bufferevent_event_cb)(struct bufferevent *bev,
    short events, void *ctx);

void bufferevent_setcb(struct bufferevent *bufev,
    bufferevent_data_cb readcb, bufferevent_data_cb writecb,
    bufferevent_event_cb eventcb, void *cbarg);

void bufferevent_getcb(struct bufferevent *bufev,
    bufferevent_data_cb *readcb_ptr,
    bufferevent_data_cb *writecb_ptr,
    bufferevent_event_cb *eventcb_ptr,
    void **cbarg_ptr);
```

bufferevent_setcb()函数修改 bufferevent 的一个或者多个回调 。readcb、writecb和eventcb函数将分别在已经读取足够的数据 、已经写入足够的数据 ,或者发生错误时被调用 。

每个回调函数的第一个参数都是发生了事件的bufferevent ,最后一个参数都是调用bufferevent_setcb()时用户提供的 cbarg 参数:可以通过它向回调传递数据。事件回调 的 events 参数是一个表示事件标志的位掩码:请看前面的 “回调和水位”节。


要禁用回调,传递 NULL 而不是回调函数 。注意:bufferevent 的所有回调函数共享单 个 cbarg, 所以修改它将影响所有回调函数。



###接口
```cpp
void bufferevent_enable(struct bufferevent *bufev, short events);
void bufferevent_disable(struct bufferevent *bufev, short events);

short bufferevent_get_enabled(struct bufferevent *bufev);
```
可以启用或者禁用 bufferevent 上的 EV_READ、EV_WRITE 或者 EV_READ | EV_WRITE 事件。没有启用读取或者写入事件时, bufferevent 将不会试图进行数据读取或者写入。

没有必要在输出缓冲区空时禁用写入事件: bufferevent 将自动停止写入,然后在有数据等 待写入时重新开始。

类似地,没有必要在输入缓冲区高于高水位时禁用读取事件 :bufferevent 将自动停止读取, 然后在有空间用于读取时重新开始读取。

默认情况下,新创建的 bufferevent 的写入是启用的,但是读取没有启用。
可以调用 bufferevent_get_enabled()确定 bufferevent 上当前启用的事件。

###接口
```cpp
void bufferevent_setwatermark(struct bufferevent *bufev, short events,
    size_t lowmark, size_t highmark);
```
bufferevent_setwatermark()函数调整单个 bufferevent 的读取水位、写入水位,或者同时调 整二者。(如果 events 参数设置了 EV_READ,调整读取水位。如果 events 设置了 EV_WRITE 标志,调整写入水位)

对于高水位,0表示“无限”。

###示例

```cpp
#include <event2/event.h>
#include <event2/bufferevent.h>
#include <event2/buffer.h>
#include <event2/util.h>

#include <stdlib.h>
#include <errno.h>
#include <string.h>

struct info {
    const char *name;
    size_t total_drained;
};

void read_callback(struct bufferevent *bev, void *ctx)
{
    struct info *inf = ctx;
    struct evbuffer *input = bufferevent_get_input(bev);
    size_t len = evbuffer_get_length(input);
    if (len) {
        inf->total_drained += len;
        evbuffer_drain(input, len);
        printf("Drained %lu bytes from %s\n",
             (unsigned long) len, inf->name);
    }
}

void event_callback(struct bufferevent *bev, short events, void *ctx)
{
    struct info *inf = ctx;
    struct evbuffer *input = bufferevent_get_input(bev);
    int finished = 0;

    if (events & BEV_EVENT_EOF) {
        size_t len = evbuffer_get_length(input);
        printf("Got a close from %s.  We drained %lu bytes from it, "
            "and have %lu left.\n", inf->name,
            (unsigned long)inf->total_drained, (unsigned long)len);
        finished = 1;
    }
    if (events & BEV_EVENT_ERROR) {
        printf("Got an error from %s: %s\n",
            inf->name, evutil_socket_error_to_string(EVUTIL_SOCKET_ERROR()));
        finished = 1;
    }
    if (finished) {
        free(ctx);
        bufferevent_free(bev);
    }
}

struct bufferevent *setup_bufferevent(void)
{
    struct bufferevent *b1 = NULL;
    struct info *info1;

    info1 = malloc(sizeof(struct info));
    info1->name = "buffer 1";
    info1->total_drained = 0;

    /* ... Here we should set up the bufferevent and make sure it gets
       connected... */

    /* Trigger the read callback only whenever there is at least 128 bytes
       of data in the buffer. */
    bufferevent_setwatermark(b1, EV_READ, 128, 0);

    bufferevent_setcb(b1, read_callback, NULL, event_callback, info1);

    bufferevent_enable(b1, EV_READ); /* Start reading. */
    return b1;
}
```