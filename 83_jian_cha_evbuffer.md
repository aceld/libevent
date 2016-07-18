# 8.3 检查evbuffer

```cpp
size_t evbuffer_get_length(const struct evbuffer *buf);
//这个函数返回 evbuffer 存储的字节数,它在2.0.1-alpha 版本中引入。
```

```cpp
int evbuffer_add(struct evbuffer *buf, 
              const void *data, size_t datlen);
//这个函数返回连续地存储在 evbuffer前面的字节数。
//evbuffer中的数据可能存储在多个分隔开的内存块中,
//这个函数返回当前第一个块中的字节数。
```