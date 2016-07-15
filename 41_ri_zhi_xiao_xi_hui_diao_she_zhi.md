# 4.1 日志消息回调设置

libevent 可以记录内部错误和警告。如果编译进日志支持，还会记录调试信息。默认配置下
这些信息被写到stderr。通过提供定制的日志函数可以覆盖默认行为。



```cpp
#define EVENT_LOG_DEBUG 0
#define EVENT_LOG_MSG   1
#define EVENT_LOG_WARN  2
#define EVENT_LOG_ERR   3

/* Deprecated; see note at the end of this section */
#define _EVENT_LOG_DEBUG EVENT_LOG_DEBUG
#define _EVENT_LOG_MSG   EVENT_LOG_MSG
#define _EVENT_LOG_WARN  EVENT_LOG_WARN
#define _EVENT_LOG_ERR   EVENT_LOG_ERR


typedef void (*event_log_cb)(int severity, const char *msg);
void event_set_log_callback(event_log_cb cb);

```

要覆盖libevent 的日志行为，编写匹配event_log_cb 签名的定制函数，将其作为参数传递
给event_set_log_callback（）。

随后libevent 在日志信息的时候，将会把信息传递给你提供的函数。再次调用event_set_log_callback（），传递参数NULL，就可以恢复默认行为。

实例
