# 4.4 event_base优先级

libevent支持为事件设置多个优先级。然而, event_base默认只支持单个优先级。可以调用 event_base_priority_init()设置 event_base 的优先级数目。

```cpp
int event_base_priority_init(struct event_base *base, int n_priorities);
```

成功时这个函数返回 0,失败时返回 -1。base 是要修改的 event_base,n_priorities 是要支 持的优先级数目,这个数目至少是 1 。每个新的事件可用的优先级将从 0 (最高) 到 n_priorities-1(最低)。


常量 EVENT_MAX_PRIORITIES 表示 n_priorities 的上限。调用这个函数时为 n_priorities 给出更大的值是错误的。

>必须在任何事件激活之前调用这个函数,最好在创建 event_base 后立刻调用。