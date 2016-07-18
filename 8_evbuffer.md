# 8 数据封装evBuffer

libevent 的 evbuffer 实现了为向后面添加数据和从前面移除数据而优化的字节队列。

evbuffer 用于处理缓冲网络 IO 的“缓冲”部分。它不提供调度 IO 或者当 IO 就绪时触发 IO 的 功能:这是 bufferevent 的工作。

除非特别说明,本章描述的函数都在 event2/buffer.h中声明。