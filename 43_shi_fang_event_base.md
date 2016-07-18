# 4.3 释放event_base

使用完 event_base 之后,使用 event_base_free()进行释放。

```cpp
void event_base_free(struct event_base *base);
```

>注意:这个函数不会释放当前与 event_base 关联的任何事件,或者关闭他们的套接字 ,或 者释放任何指针。


event_base_free()定义在<event2/event.h>中,首次由 libevent 1.2实现。