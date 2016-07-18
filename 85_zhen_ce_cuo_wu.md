# 8.5 侦测错误

可以设置一个一旦监听器上的 accept()调用失败就被调用的错误回调函数 。对于一个不解决就会锁定进程的错误条件,这很重要。

```cpp
typedef void (*evconnlistener_errorcb)(struct evconnlistener *lis, void *ptr);
void evconnlistener_set_error_cb(struct evconnlistener *lev,
    evconnlistener_errorcb errorcb);
```

如果使用 evconnlistener_set_error_cb() 为监听器设置了错误回调函数,则监听器发生错误 时回调函数就会被调用。

第一个参数是监听器,

第二个参数是调用 evconnlistener_new() 时传入的 ptr。
