# 8.3 调整 evconnlistener 的回调函数

```cpp
void evconnlistener_set_cb(struct evconnlistener *lev,
    evconnlistener_cb cb, void *arg);
```

函数调整 evconnlistener 的回调函数和其参数。它是 2.0.9-rc 版本引入的。