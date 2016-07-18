# 7.5.1 释放bufferevent操作

```cpp
void bufferevent_free(struct bufferevent *bev);
```

这个函数释放 bufferevent。bufferevent 内部具有引用计数,所以,如果释放 时还有未决的延迟回调,则在回调完成之前 bufferevent 不会被删除。

如果设置了 BEV_OPT_CLOSE_ON_FREE 标志,并且 bufferevent 有一个套接字或者底层 bufferevent 作为其传输端口,则释放 bufferevent 将关闭这个传输端口。