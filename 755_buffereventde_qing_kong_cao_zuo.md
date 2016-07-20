# 7.5.4 bufferevent的清空操作

```cpp
int bufferevent_flush(struct bufferevent *bufev,
    short iotype, enum bufferevent_flush_mode state);
```

清空 bufferevent 要求 bufferevent 强制从底层传输端口读取或者写入尽可能多的数据 ,而忽略其他可能保持数据不被写入的限制条件 。函数的细节功能依赖于 bufferevent 的具体类型。


otype 参数应该是 EV_READ、EV_WRITE 或者 EV_READ | EV_WRITE,用于指示应该处 理读取、写入,还是二者都处理。 state 参数可以是 BEV_NORMAL、BEV_FLUSH 或者 BEV_FINISHED。BEV_FINISHED 指示应该告知另一端,没有更多数据需要发送了; 而 BEV_NORMAL 和 BEV_FLUSH 的区别依赖于具体的 bufferevent 类型。


失败时 bufferevent_flush()返回-1,如果没有数据被清空则返回 0,有数据被清空则返回 1。