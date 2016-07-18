# 7.5 通用bufferevent操作

## 7.5.1 释放bufferevent

```cpp
void bufferevent_free(struct bufferevent *bev);
```

这个函数释放 bufferevent。bufferevent 内部具有引用计数,所以,如果释放 时还有未决的延迟回调,则在回调完成之前 bufferevent 不会被删除。

如果设置了 BEV_OPT_CLOSE_ON_FREE 标志,并且 bufferevent 有一个套接字或者底层 bufferevent 作为其传输端口,则释放 bufferevent 将关闭这个传输端口。

## 7.5.2 操作回调、水位和启动/禁用


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