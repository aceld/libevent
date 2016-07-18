# 7 数据缓冲Bufferevent
很多时候,除了响应事件之外,应用还希望做一定的数据缓冲。比如说,写入数据的时候 ,通常的运行模式是:

* 决定要向连接写入一些数据,把数据放入到缓冲区中

* 等待连接可以写入

* 写入尽量多的数据

* 记住写入了多少数据,如果还有更多数据要写入,等待连接再次可以写入


这种缓冲 IO 模式很通用,libevent 为此提供了一种通用机制,即bufferevent。

bufferevent 由一个底层的传输端口(如套接字 ),一个读取缓冲区和一个写入缓冲区组成。与通常的事件在底层传输端口已经就绪,可以读取或者写入的时候执行回调不同的是,bufferevent 在读取或者写入了足够量的数据之后调用用户提供的回调。

有多种共享公用接口的 bufferevent 类型,编写本文时已存在以下类型:


* `基于套接字的 bufferevent`:使用 event_*接口作为后端,通过底层流式套接字发送或者接收数据的 bufferevent


* `异步 IO bufferevent`:使用 Windows IOCP 接口,通过底层流式套接字发送或者接收数据的 bufferevent(仅用于 Windows,试验中)


* `过滤型 bufferevent`:将数据传输到底层 bufferevent 对象之前,处理输入或者输出数据的 bufferevent:比如说,为了压缩或者转换数据。


* `成对的 bufferevent`:相互传输数据的两个 bufferevent。




>`注意`:截止2.0.2-alpha 版,这里列出的 bufferevent 接口还没有完全正交于所有 的 bufferevent 类型。也就是说,下面将要介绍的接口不是都能用于所有bufferevent 类型。libevent 开发 者在未来版本中将修正这个问题。

>`也请注意` :当前 bufferevent 只能用于像 TCP 这样的面向流的协议,将来才可能会支持 像 UDP 这样的面向数据报的协议。


## bufferevent和evbuffer
每个 bufferevent 都有一个输入缓冲区和一个输出缓冲区 ,它们的类型都是“struct evbuffer”。 有数据要写入到 bufferevent 时,添加数据到输出缓冲区 ;bufferevent 中有数据供读取的时候,从输入缓冲区抽取(drain)数据。
evbuffer 接口支持很多种操作,后面的章节将讨论这些操作。
