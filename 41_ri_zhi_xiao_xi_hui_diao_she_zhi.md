# 9.1 日志消息回调设置

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


### 实例

```cpp
#include <event2/event.h>
#include <stdio.h>

static void discard_cb(int severity, const char *msg)
{
    /* This callback does nothing. */
}

static FILE *logfile = NULL;
static void write_to_file_cb(int severity, const char *msg)
{
    const char *s;
    if (!logfile)
        return;
    switch (severity) {
        case _EVENT_LOG_DEBUG: s = "debug"; break;
        case _EVENT_LOG_MSG:   s = "msg";   break;
        case _EVENT_LOG_WARN:  s = "warn";  break;
        case _EVENT_LOG_ERR:   s = "error"; break;
        default:               s = "?";     break; /* never reached */
    }
    fprintf(logfile, "[%s] %s\n", s, msg);
}

/* Turn off all logging from Libevent. */
void suppress_logging(void)
{
    event_set_log_callback(discard_cb);
}

/* Redirect all Libevent log messages to the C stdio file 'f'. */
void set_logfile(FILE *f)
{
    logfile = f;
    event_set_log_callback(write_to_file_cb);
}

```

在用户提供的event_log_cb 回调函数中调用libevent 函数是不安全的。

比如说，如果试图编写一个使用bufferevent 将警告信息发送给某个套接字的日志回调函数，可能会遇到奇怪
而难以诊断的bug。未来版本libevent 的某些函数可能会移除这个限制。


这个函数在<event2/event.h>中声明，在libevent 1.0c 版本中首次出现。