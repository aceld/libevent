# 4.5 event_base和fork

不是所有事件后端都在调用 fork()之后可以正确工作。所以,如果在使用 fork()或者其 他相关系统调用启动新进程之后,希望在新进程中继续使用 event_base,就需要进行重新初始化。



```cpp
int event_reinit(struct event_base *base);
```
成功时这个函数返回 0,失败时返回 -1。


###实例
```cpp
struct event_base *base = event_base_new();

/* ... add some events to the event_base ... */

if (fork()) {
    /* In parent */
    continue_running_parent(base); /*...*/
} else {
    /* In child */
    event_reinit(base);
    continue_running_child(base); /*...*/
}
```

event_reinit()定义在<event2/event.h>中,在 libevent 1.4.3-alpha 版中首次可用。