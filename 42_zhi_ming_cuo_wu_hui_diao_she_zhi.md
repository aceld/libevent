# 9.2 致命错误回调设置


libevent 在检测到不可恢复的内部错误时的默认行为是调用exit（）或者abort（），退出正在运行的进程。这类错误通常意味着某处有bug：要么在你的代码中，要么在libevent 中。


如果希望更优雅地处理致命错误，可以为libevent 提供在退出时应该调用的函数，覆盖默认
行为。

```cpp
typedef void (*event_fatal_cb)(int err);
void event_set_fatal_callback(event_fatal_cb cb);
```


要使用这些函数，首先定义libevent 在遇到致命错误时应该调用的函数，将其传递给
event_set_fatal_callback（）。



随后libevent 在遇到致命错误时将调用你提供的函数。
你的函数不应该将控制返回到libevent：这样做可能导致不确定的行为。


为了避免崩溃，libevent 还是会退出。你的函数被不应该调用其它libevent 函数。
这些函数声明在<event2/event.h>中，在libevent 2.0.3-alpha 版本中首次出现。