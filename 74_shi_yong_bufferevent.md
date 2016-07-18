# 7.4 使用bufferevent

基于套接字的 bufferevent 是最简单的,它使用 libevent 的底层事件机制来检测底层网络套 接字是否已经就绪,可以进行读写操作,并且使用底层网络调用(如 readv 、 writev 、 WSASend、WSARecv)来发送和接收数据。

## 7.4.1 创建基于套接字的bufferevent

可以使用 bufferevent_socket_new()创建基于套接字的 bufferevent。

```cpp
struct bufferevent *bufferevent_socket_new(
    struct event_base *base,
    evutil_socket_t fd,
    enum bufferevent_options options);
```

base 是 event_base,options 是表示 bufferevent 选项(BEV_OPT_CLOSE_ON_FREE 等) 的位掩码, fd是一个可选的表示套接字的文件描述符。如果想以后设置文件描述符,可以设置fd为-1。

成功时函数返回一个 bufferevent,失败则返回 NULL。

## 7.4.2 在bufferevent上启动链接

```cpp
int bufferevent_socket_connect(struct bufferevent *bev,
    struct sockaddr *address, int addrlen);
```

address 和 addrlen 参数跟标准调用 connect()的参数相同。如果还没有为 bufferevent 设置套接字,调用函数将为其分配一个新的流套接字,并且设置为非阻塞的。


如果已经为 bufferevent 设置套接字,调用bufferevent_socket_connect() 将告知 libevent 套接字还未连接,直到连接成功之前不应该对其进行读取或者写入操作。

连接完成之前可以向输出缓冲区添加数据。

如果连接成功启动,函数返回 0;如果发生错误则返回 -1。

```cpp
#include <event2/event.h>
#include <event2/bufferevent.h>
#include <sys/socket.h>
#include <string.h>

void eventcb(struct bufferevent *bev, short events, void *ptr)
{
    if (events & BEV_EVENT_CONNECTED) {
         /* We're connected to 127.0.0.1:8080.   Ordinarily we'd do
            something here, like start reading or writing. */
    } else if (events & BEV_EVENT_ERROR) {
         /* An error occured while connecting. */
    }
}

int main_loop(void)
{
    struct event_base *base;
    struct bufferevent *bev;
    struct sockaddr_in sin;

    base = event_base_new();

    memset(&sin, 0, sizeof(sin));
    sin.sin_family = AF_INET;
    sin.sin_addr.s_addr = htonl(0x7f000001); /* 127.0.0.1 */
    sin.sin_port = htons(8080); /* Port 8080 */

    bev = bufferevent_socket_new(base, -1, BEV_OPT_CLOSE_ON_FREE);

    bufferevent_setcb(bev, NULL, NULL, eventcb, NULL);

    if (bufferevent_socket_connect(bev,
        (struct sockaddr *)&sin, sizeof(sin)) < 0) {
        /* Error starting connection */
        bufferevent_free(bev);
        return -1;
    }

    event_base_dispatch(base);
    return 0;
}
```


**`注意`:如果使用 bufferevent_socket_connect() 发起连接,将只会收 到 BEV_EVENT_CONNECTED 事件。如果自己调用 connect(),则连接上将被报告为写入事 件。**

## 7.4.3 通过主机名启动链接

常常需要将解析主机名和连接到主机合并成单个操作, libevent 为此提供了:

```cpp
int bufferevent_socket_connect_hostname(struct bufferevent *bev,
    struct evdns_base *dns_base, int family, const char *hostname,
    int port);
int bufferevent_socket_get_dns_error(struct bufferevent *bev);
```

这个函数解析名字 hostname ,查找其 family 类型的地址(允许的地址族类型 有 AF_INET,AF_INET6和 AF_UNSPEC)。如果名字解析失败,函数将调用事件回调,报告错误事件。如果解析成功,函数将启动连接请求,就像bufferevent_socket_connect()一样。

dns_base 参数是可选的:如果为 NULL,等待名字查找完成期间调用线程将被阻塞,而这 通常不是期望的行为;如果提供 dns_base 参数,libevent 将使用它来异步地查询主机名。

跟bufferevent_socket_connect()一样,函数告知 libevent,bufferevent上现存的套接字还没有连接,在名字解析和连接操作成功完成之前,不应该对套接字进行读取或者写入操作。


函数返回的错误可能是 DNS 主机名查询错误 ,可以调用 bufferevent_socket_get_dns_error() 来获取最近的错误。返回值 0表示没有检测到 DNS 错误。

###示例：简单的HTTP客户端

```cpp
/* Don't actually copy this code: it is a poor way to implement an
   HTTP client.  Have a look at evhttp instead.
*/
#include <event2/dns.h>
#include <event2/bufferevent.h>
#include <event2/buffer.h>
#include <event2/util.h>
#include <event2/event.h>

#include <stdio.h>

void readcb(struct bufferevent *bev, void *ptr)
{
    char buf[1024];
    int n;
    struct evbuffer *input = bufferevent_get_input(bev);
    while ((n = evbuffer_remove(input, buf, sizeof(buf))) > 0) {
        fwrite(buf, 1, n, stdout);
    }
}

void eventcb(struct bufferevent *bev, short events, void *ptr)
{
    if (events & BEV_EVENT_CONNECTED) {
         printf("Connect okay.\n");
    } else if (events & (BEV_EVENT_ERROR|BEV_EVENT_EOF)) {
         struct event_base *base = ptr;
         if (events & BEV_EVENT_ERROR) {
                 int err = bufferevent_socket_get_dns_error(bev);
                 if (err)
                         printf("DNS error: %s\n", evutil_gai_strerror(err));
         }
         printf("Closing\n");
         bufferevent_free(bev);
         event_base_loopexit(base, NULL);
    }
}

int main(int argc, char **argv)
{
    struct event_base *base;
    struct evdns_base *dns_base;
    struct bufferevent *bev;

    if (argc != 3) {
        printf("Trivial HTTP 0.x client\n"
               "Syntax: %s [hostname] [resource]\n"
               "Example: %s www.google.com /\n",argv[0],argv[0]);
        return 1;
    }

    base = event_base_new();
    dns_base = evdns_base_new(base, 1);

    bev = bufferevent_socket_new(base, -1, BEV_OPT_CLOSE_ON_FREE);
    bufferevent_setcb(bev, readcb, NULL, eventcb, base);
    bufferevent_enable(bev, EV_READ|EV_WRITE);
    evbuffer_add_printf(bufferevent_get_output(bev), "GET %s\r\n", argv[2]);
    bufferevent_socket_connect_hostname(
        bev, dns_base, AF_UNSPEC, argv[1], 80);
    event_base_dispatch(base);
    return 0;
}
```

