# 8.1 创建和释放evbuffer

```cpp
struct evbuffer *evbuffer_new(void);
void evbuffer_free(struct evbuffer *buf);
```

这两个函数的功能很简明: evbuffer_new() 分配和返回一个新的空 evbuffer ; 而 evbuffer_free()释放 evbuffer 和其内容。


