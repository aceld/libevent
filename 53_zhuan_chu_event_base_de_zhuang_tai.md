# 5.3 转储event_base的状态

为帮助调试程序(或者调试 libevent),有时候可能需要加入到 event_base 的事件及其状态 的完整列表。调用 event_base_dump_events()可以将这个列表输出到指定的文件中。


```cpp
void event_base_dump_events(struct event_base *base, FILE *f);
```

这个列表是人可读的,未来版本的 libevent 将会改变其格式。

