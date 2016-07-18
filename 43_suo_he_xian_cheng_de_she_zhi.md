# 9.4 锁和线程的设置

编写多线程程序的时候,在多个线程中同时访问同样的数据并不总是安全的。


libevent 的结构体在多线程下通常有三种工作方式:

* 某些结构体内在地是单线程的:同时在多个线程中使用它们总是不安全的。

* 某些结构体具有可选的锁:可以告知 libevent 是否需要在多个线程中使用每个对象。

* 某些结构体总是锁定的 :如果 libevent 在支持锁的配置下运行 ,在多个线程中使用它们 总是安全的。


为获取锁,在调用分配需要在多个线程间共享的结构体的 libevent 函数之前,必须告 知 libevent 使用哪个锁函数。

如果使用 pthreads 库,或者使用 Windows 本地线程代码,那么你是幸运的:已经有设 置 libevent 使用正确的 pthreads 或者 Windows 函数的预定义函数。


### 接口

```cpp
#ifdef WIN32
  int evthread_use_windows_threads(void);
#define EVTHREAD_USE_WINDOWS_THREADS_IMPLEMENTED
#endif

#ifdef _EVENT_HAVE_PTHREADS
  int evthread_use_pthreads(void);
#define EVTHREAD_USE_PTHREADS_IMPLEMENTED

#endif
```


这些函数在成功时都返回 0,失败时返回 -1。 


如果使用不同的线程库,则需要一些额外的工作,必须使用你的线程库来定义函数去实现:


* 锁
* 锁定
* 解锁
* 分配锁
* 析构锁
* 条件变量
* 创建条件变量
* 析构条件变量
* 等待条件变量
* 触发/广播某条件变量
* 线程
* 线程ID检测


使用 evthread_set_lock_callbacks 和 evthread_set_id_callback 接口告知 libevent 这些函数。


### 接口


```cpp
#define EVTHREAD_WRITE  0x04
#define EVTHREAD_READ   0x08
#define EVTHREAD_TRY    0x10

#define EVTHREAD_LOCKTYPE_RECURSIVE 1
#define EVTHREAD_LOCKTYPE_READWRITE 2

#define EVTHREAD_LOCK_API_VERSION 1

struct evthread_lock_callbacks {
       int lock_api_version;
       unsigned supported_locktypes;
       void *(*alloc)(unsigned locktype);
       void (*free)(void *lock, unsigned locktype);
       int (*lock)(unsigned mode, void *lock);
       int (*unlock)(unsigned mode, void *lock);
};

int evthread_set_lock_callbacks(const struct evthread_lock_callbacks *);

void evthread_set_id_callback(unsigned long (*id_fn)(void));

struct evthread_condition_callbacks {
        int condition_api_version;
        void *(*alloc_condition)(unsigned condtype);
        void (*free_condition)(void *cond);
        int (*signal_condition)(void *cond, int broadcast);
        int (*wait_condition)(void *cond, void *lock,
            const struct timeval *timeout);
};

int evthread_set_condition_callbacks(
        const struct evthread_condition_callbacks *);
```


evthread_lock_callbacks 结构体描述的锁回调函数及其能力。对于上述版本,

lock_api_version 字段必须设置为 EVTHREAD_LOCK_API_VERSION 。必须设置 supported_locktypes 字段为 EVTHREAD_LOCKTYPE_* 常量的组合以描述支持的锁类型 (在 2.0.4-alpha 版本中),

EVTHREAD_LOCK_RECURSIVE 是必须的, 

EVTHREAD_LOCK_READWRITE 则没有使用)。

alloc 函数必须返回指定类型的新锁 ;

free 函数必须释放指定类型锁持有的所有资源 ;

lock 函数必须试图以指定模式请求锁定 ,如果成 功则返回0,失败则返回非零; 

unlock 函数必须试图解锁,成功则返回 0,否则返回非零。



### 可识别的锁类型有:
* 0:通常的,不必递归的锁。
* EVTHREAD_LOCKTYPE_RECURSIVE :不会阻塞已经持有它的线程的锁 。一旦持有它的线程进行原来锁定次数的解锁,其他线程立刻就可以请求它了。


* EVTHREAD_LOCKTYPE_READWRITE :可以让多个线程同时因为读而持有它 ,但是 任何时刻只有一个线程因为写而持有它。写操作排斥所有读操作。

### 可识别的锁模式有:

* EVTHREAD_READ :仅用于读写锁:为读操作请求或者释放锁
* EVTHREAD_WRITE :仅用于读写锁:为写操作请求或者释放锁
* EVTHREAD_TRY :仅用于锁定:仅在可以立刻锁定的时候才请求锁定

id_fn 参数必须是一个函数,它返回一个无符号长整数,标识调用此函数的线程。对于相同 线程,这个函数应该总是返回同样的值 ;而对于同时调用该函数的不同线程 ,必须返回不同 的值。



vthread_condition_callbacks 结构体描述了与条件变量相关的回调函数。对于上述版本 , condition_api_version 字段必须设置为 EVTHREAD_CONDITION_API_VERSION 。 alloc_condition 函数必须返回到新条件变量的指针 。它接受0作为其参数。free_condition 函 数必须释放条件变量持有的存储器和资源。 wait_condition 函数要求三个参数:一个 由 alloc_condition 分配的条件变量 ,一个由你提供的 evthread_lock_callbacks.alloc 函数分配 的锁,以及一个可选的超时值 。调用本函数时 ,必须已经持有参数指定的锁 ;本函数应该释 放指定的锁,等待条件变量成为授信状态,或者直到指定的超时时间已经流逝(可选 )。 wait_condition 应该在错误时返回-1,条件变量授信时返回0,超时时返回1。返回之前,函 数应该确定其再次持有锁。最后, signal_condition 函数应该唤醒等待该条件变量的某个线 程(broadcast 参数为 false 时),或者唤醒等待条件变量的所有线程(broadcast 参数为 true 时)。只有在持有与条件变量相关的锁的时候,才能够进行这些操作。


>关于条件变量的更多信息,请查看 pthreads 的 pthread_cond_*函数文档,或者 Windows 的 CONDITION_VARIABLE(Windows Vista 新引入的)函数文档。


### 实例：

```cpp
关于使用这些函数的示例,

请查看 Libevent 源代码发布版本中的 
evthread_pthread.c 和 evthread_win32.c 文件。
```


这些函数在 <event2/thread.h> 中声明,其中大多数在 2.0.4-alpha 版本中首次出现。 2.0.1-alpha 到2.0.3-alpha 使用较老版本的锁函数 。event_use_pthreads 函数要求程序链接 到 event_pthreads 库。


条件变量函数是2.0.7-rc 版本新引入的,用于解决某些棘手的死锁问题。 

可以创建禁止锁支持的libevent。这时候已创建的使用上述线程相关函数的程序将不能运行。



## 调试做的使用

为帮助调试锁的使用,libevent 有一个可选的“锁调试”特征。这个特征包装了锁调用,以便捕获典型的锁错误,包括:

* 解锁并没有持有的锁
* 重新锁定一个非递归锁


如果发生这些错误中的某一个, libevent 将给出断言失败并且退出。

```cpp
void event_enable_debug_mode(void);
```

必须在创建或者使用任何锁之前调用这个函数。为安全起见,请在设置完线程函数后立即调用这个函数。
