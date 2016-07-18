# 8.1 创建和释放evconnlistener


```cpp
struct evconnlistener *
evconnlistener_new(struct event_base *base,
    evconnlistener_cb cb, void *ptr, unsigned flags, int backlog,
    evutil_socket_t fd);
    
struct evconnlistener *
evconnlistener_new_bind(struct event_base *base,
    evconnlistener_cb cb, void *ptr, unsigned flags, int backlog,
    const struct sockaddr *sa, int socklen);
    
void evconnlistener_free(struct evconnlistener *lev);
```

两个 evconnlistener_new*()函数都分配和返回一个新的连接监听器对象。连接监听器使 用 event_base 来得知什么时候在给定的监听套接字上有新的 TCP 连接。新连接到达时,监听 器调用你给出的回调函数。

两个函数中,base参数都是监听器用于监听连接的 event_base。cb是收到新连接时要调 用的回调函数;

如果 cb 为 NULL,则监听器是禁用的,直到设置了回调函数为止。

ptr 指针将传递给回调函数。 

flags 参数控制回调函数的行为,下面会更详细论述。 

backlog 是任何 时刻网络栈允许处于还未接受状态的最大未决连接数。

更多细节请查看系统的 listen()函数文档。

如果 backlog 是负的,libevent 会试图挑选一个较好的值 ;
如果为0,libevent 认为已 经对提供的套接字调用了listen()。

---

两个函数的不同在于如何建立监听套接字。
evconnlistener_new()函数假定已经将套接字绑定到要监听的端口,然后通过 fd 传入这个套接字。

如果要 libevent 分配和绑定套接字,可以调用 evconnlistener_new_bind() ,传输要绑定到的地址和地址长度。


要释放连接监听器,调用 evconnlistener_free()。

##可识别的标志

可以给 evconnlistener_new() 函数的 flags 参数传入一些标志。可以用或 (OR)运算任意连接 下述标志:

* LEV_OPT_LEAVE_SOCKETS_BLOCKING 

默认情况下,连接监听器接收新套接字后,会将其设置为非阻塞的,以便将其用于 libevent。如果不想要这种行为,可以设置这个标志。


* LEV_OPT_CLOSE_ON_FREE 
 
如果设置了这个选项,释放连接监听器会关闭底层套接字。

* LEV_OPT_CLOSE_ON_EXEC

如果设置了这个选项,连接监听器会为底层套接字设置 close-on-exec 标志。更多信息请查 看 fcntl 和 FD_CLOEXEC 的平台文档。

* LEV_OPT_REUSEABLE

某些平台在默认情况下 ,关闭某监听套接字后 ,要过一会儿其他套接字才可以绑定到同一个 端口。设置这个标志会让 libevent 标记套接字是可重用的,这样一旦关闭,可以立即打开其 他套接字,在相同端口进行监听。

* LEV_OPT_THREADSAFE 
 
为监听器分配锁,这样就可以在多个线程中安全地使用了。这是 2.0.8-rc 的新功能。


##链接监听器回调

```cpp
typedef void (*evconnlistener_cb)(struct evconnlistener *listener,
    evutil_socket_t sock, struct sockaddr *addr, int len, void *ptr);
```
接收到新连接会调用提供的回调函数 。

`listener` 参数是接收连接的连接监听器 。


`sock` 参数是 新接收的套接字。 

`addr` 和 `len` 参数是接收连接的地址和地址长度。 

`ptr` 是调 用 evconnlistener_new() 时用户提供的指针。


