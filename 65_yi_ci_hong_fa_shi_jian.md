# 6.5 一次触发事件


如果不需要多次添加一个事件,或者要在添加后立即删除事件,而事件又不需要是持久的 , 则可以使用 event_base_once()。

```cpp
int event_base_once(struct event_base *, evutil_socket_t, short,
  void (*)(evutil_socket_t, short, void *), void *, const struct timeval *);
```

除了不支持 EV_SIGNAL 或者 EV_PERSIST 之外,这个函数的接口与 event_new()相同。 安排的事件将以默认的优先级加入到 event_base并执行。回调被执行后,libevent内部将 会释放 event 结构。成功时函数返回0,失败时返回-1。

不能删除或者手动激活使用 event_base_once ()插入的事件:如果希望能够取消事件, 应该使用 event_new()或者 event_assign()。
