# 8.4 向evbuffer添加数据

```cpp
int evbuffer_add(struct evbuffer *buf, const void *data, size_t datlen);
//这个函数添加 data 处的 datalen 字节到 buf 的末尾,
//成功时返回0,失败时返回-1。
```


```cpp
int evbuffer_add_printf(struct evbuffer *buf, const char *fmt, ...)
int evbuffer_add_vprintf(struct evbuffer *buf, const char *fmt, va_list ap);
//这些函数添加格式化的数据到 buf 末尾。
//格式参数和其他参数的处理分别与 C 库函数 printf 和 vprintf 相同。函数返回添加的字节数。
```

```cpp
int evbuffer_expand(struct evbuffer *buf, size_t datlen);
//这个函数修改缓冲区的最后一块,或者添加一个新的块,
//使得缓冲区足以容纳 datlen 字节, 而不需要更多的内存分配。
```

###示例
```cpp
/* Here are two ways to add "Hello world 2.0.1" to a buffer. */
/* Directly: */
evbuffer_add(buf, "Hello world 2.0.1", 17);

/* Via printf: */
evbuffer_add_printf(buf, "Hello %s %d.%d.%d", "world", 2, 0, 1);
```