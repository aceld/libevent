# 7.5.3 操作bufferevent中的数据

##通过bufferevent得到evbuffer
如果只是通过网络读取或者写入数据 ,而不能观察操作过程,是没什么好处的。bufferevent 提供了下列函数用于观察要写入或者读取的数据。 

```cpp
struct evbuffer *bufferevent_get_input(struct bufferevent *bufev);
struct evbuffer *bufferevent_get_output(struct bufferevent *bufev);
```

这两个函数提供了非常强大的基础 :它们分别返回输入和输出缓冲区 。关于可以对 evbuffer 类型进行的所有操作的完整信息,请看下一章。

如果写入操作因为数据量太少而停止(或者读取操作因为太多数据而停止 ),则向输出缓冲 区添加数据(或者从输入缓冲区移除数据)将自动重启操作。

##向bufferevent的输出缓冲区添加数据

```cpp
int bufferevent_write(struct bufferevent *bufev,
    const void *data, size_t size);
int bufferevent_write_buffer(struct bufferevent *bufev,
    struct evbuffer *buf);
```

这些函数向 bufferevent 的输出缓冲区添加数据。 bufferevent_write()将内存中从 data 处开 始的 size 字节数据添加到输出缓冲区的末尾 。bufferevent_write_buffer()移除 buf 的所有内 容,将其放置到输出缓冲区的末尾。成功时这些函数都返回 0,发生错误时则返回-1。


##从bufferevent的输入缓冲区移除数据
```cpp
size_t bufferevent_read(struct bufferevent *bufev, void *data, size_t size);
int bufferevent_read_buffer(struct bufferevent *bufev,
    struct evbuffer *buf);
```

这些函数从 bufferevent 的输入缓冲区移除数据。bufferevent_read()至多从输入缓冲区移除 size 字节的数据,将其存储到内存中 data 处。函数返回实际移除的字节数。 bufferevent_read_buffer()函数抽空输入缓冲区的所有内容,将其放置到 buf 中,成功时返 回0,失败时返回 -1。

注意,对于 bufferevent_read(),data 处的内存块必须有足够的空间容纳 size 字节数据。

###示例

```cpp
#include <event2/bufferevent.h>
#include <event2/buffer.h>

#include <ctype.h>

void
read_callback_uppercase(struct bufferevent *bev, void *ctx)
{
        /* This callback removes the data from bev's input buffer 128
           bytes at a time, uppercases it, and starts sending it
           back.

           (Watch out!  In practice, you shouldn't use toupper to implement
           a network protocol, unless you know for a fact that the current
           locale is the one you want to be using.)
         */

        char tmp[128];
        size_t n;
        int i;
        while (1) {
                n = bufferevent_read(bev, tmp, sizeof(tmp));
                if (n <= 0)
                        break; /* No more data. */
                for (i=0; i<n; ++i)
                        tmp[i] = toupper(tmp[i]);
                bufferevent_write(bev, tmp, n);
        }
}

struct proxy_info {
        struct bufferevent *other_bev;
};
void
read_callback_proxy(struct bufferevent *bev, void *ctx)
{
        /* You might use a function like this if you're implementing
           a simple proxy: it will take data from one connection (on
           bev), and write it to another, copying as little as
           possible. */
        struct proxy_info *inf = ctx;

        bufferevent_read_buffer(bev,
            bufferevent_get_output(inf->other_bev));
}

struct count {
        unsigned long last_fib[2];
};

void
write_callback_fibonacci(struct bufferevent *bev, void *ctx)
{
        /* Here's a callback that adds some Fibonacci numbers to the
           output buffer of bev.  It stops once we have added 1k of
           data; once this data is drained, we'll add more. */
        struct count *c = ctx;

        struct evbuffer *tmp = evbuffer_new();
        while (evbuffer_get_length(tmp) < 1024) {
                 unsigned long next = c->last_fib[0] + c->last_fib[1];
                 c->last_fib[0] = c->last_fib[1];
                 c->last_fib[1] = next;

                 evbuffer_add_printf(tmp, "%lu", next);
        }

        /* Now we add the whole contents of tmp to bev. */
        bufferevent_write_buffer(bev, tmp);

        /* We don't need tmp any longer. */
        evbuffer_free(tmp);
}
```