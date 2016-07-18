# 4.1 创建默认event_base

***event_base_new()***函数分配并且返回一个新的具有默认设置的 event_base。函数会检测环境变量,返回一个到 event_base 的指针。如果发生错误,则返回 NULL。选择各种方法时,函数会选择 OS 支持的最快方法。

```cpp
struct event_base *event_base_new(void);
```

>大多数程序使用这个函数就够了。

event_base_new()函数声明在<event2/event.h>中,首次出现在 libevent 1.4.3版。
