# 7.5.4 读写超时

```cpp
void bufferevent_set_timeouts(struct bufferevent *bufev,
    const struct timeval *timeout_read, const struct timeval *timeout_write);
```

设置超时为 NULL 会移除超时回调。

试图读取数据的时候,如果至少等待了 timeout_read 秒,则读取超时事件将被触发。试图写入数据的时候,如果至少等待了 timeout_write 秒,则写入超时事件将被触发。

注意,只有在读取或者写入的时候才会计算超时。也就是说,如果 bufferevent的读取被禁止,或者输入缓冲区满(达到其高水位),则读取超时被禁止。类似的,如果写入被禁止, 或者没有数据待写入,则写入超时被禁止。

读取或者写入超时发生时,相应的读取或者写入操作被禁止,然后超时事件回调被调用 ,带有标志 BEV_EVENT_TIMEOUT | BEV_EVENT_READING 或者 BEV_EVENT_TIMEOUT | BEV_EVENT_WRITING 。