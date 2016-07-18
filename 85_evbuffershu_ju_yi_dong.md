# 8.5 evbuffer数据移动

为提高效率,libevent 具有将数据从一个 evbuffer 移动到另一个的优化函数。

```cpp
int evbuffer_add_buffer(struct evbuffer *dst, struct evbuffer *src);

int evbuffer_remove_buffer(struct evbuffer *src, 
                    struct evbuffer *dst,
                    size_t datlen);
```

evbuffer_add_buffer()将 src 中的所有数据移动到 dst 末尾,成功时返回0,失败时返回-1。


evbuffer_remove_buffer()函数从 src 中移动 datlen 字节到 dst 末尾,尽量少进行复制。如果字节数小于 datlen,所有字节被移动。函数返回移动的字节数。

evbuffer_add_buffer()在0.8版本引入; evbuffer_remove_buffer()是2.0.1-alpha 版本新增加的。