# 7.3 bufferevent 选项标志


创建 bufferevent 时可以使用一个或者多个标志修改其行为。可识别的标志有:

* BEV_OPT_CLOSE_ON_FREE :释放 bufferevent 时关闭底层传输端口。这将关闭底层套接字,释放底层 bufferevent 等。


* BEV_OPT_THREADSAFE :自动为 bufferevent 分配锁,这样就可以安全地在多个线程中使用 bufferevent。


* BEV_OPT_DEFER_CALLBACKS :设置这个标志时, bufferevent 延迟所有回调,如上所述。


* BEV_OPT_UNLOCK_CALLBACKS :默认情况下,如果设置 bufferevent 为线程安全 的,则 bufferevent 会在调用用户提供的回调时进行锁定。设置这个选项会让 libevent 在执行回调的时候不进行锁定。


(BEV_OPT_UNLOCK_CALLBACKS 由2.0.5-beta 版引入,其他选项由 2.0.1-alpha 版引 入)
