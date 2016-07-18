# 9.3 内存管理回调设置


默认情况下，libevent 使用C 库的内存管理函数在堆上分配内存。

通过提供malloc、realloc和free 的替代函数，可以让libevent 使用其他的内存管理器。


希望libevent 使用一个更高效的分配器时；或者希望libevent 使用一个工具分配器，以便检查内存泄漏时，可能需要这样做。



```java
void event_set_mem_functions(void *(*malloc_fn)(size_t sz),
                             void *(*realloc_fn)(void *ptr, size_t sz),
                             void (*free_fn)(void *ptr));
```

这里有个替换libevent 分配器函数的示例，它可以计算已经分配的字节数。

实际应用中可能
需要添加锁，以避免运行在多个线程中时发生错误。

### 实例

```cpp

#include <event2/event.h>
#include <sys/types.h>
#include <stdlib.h>

/* This union's purpose is to be as big as the largest of all the
 * types it contains. */
union alignment {
    size_t sz;
    void *ptr;
    double dbl;
};
/* We need to make sure that everything we return is on the right
   alignment to hold anything, including a double. */
#define ALIGNMENT sizeof(union alignment)

/* We need to do this cast-to-char* trick on our pointers to adjust
   them; doing arithmetic on a void* is not standard. */
#define OUTPTR(ptr) (((char*)ptr)+ALIGNMENT)
#define INPTR(ptr) (((char*)ptr)-ALIGNMENT)

static size_t total_allocated = 0;
static void *replacement_malloc(size_t sz)
{
    void *chunk = malloc(sz + ALIGNMENT);
    if (!chunk) return chunk;
    total_allocated += sz;
    *(size_t*)chunk = sz;
    return OUTPTR(chunk);
}
static void *replacement_realloc(void *ptr, size_t sz)
{
    size_t old_size = 0;
    if (ptr) {
        ptr = INPTR(ptr);
        old_size = *(size_t*)ptr;
    }
    ptr = realloc(ptr, sz + ALIGNMENT);
    if (!ptr)
        return NULL;
    *(size_t*)ptr = sz;
    total_allocated = total_allocated - old_size + sz;
    return OUTPTR(ptr);
}
static void replacement_free(void *ptr)
{
    ptr = INPTR(ptr);
    total_allocated -= *(size_t*)ptr;
    free(ptr);
}
void start_counting_bytes(void)
{
    event_set_mem_functions(replacement_malloc,
                            replacement_realloc,
                            replacement_free);
}

```


### 注意

* 替换内存管理函数影响libevent 随后的所有分配、调整大小和释放内存操作。所以，必
须保证在调用任何其他libevent 函数之前进行替换。否则，libevent 可能用你的free 函
数释放用C 库的malloc 分配的内存。


* 你的malloc 和realloc 函数返回的内存块应该具有和C 库返回的内存块一样的地址对
齐。


* 你的realloc 函数应该正确处理realloc(NULL,sz)（也就是当作malloc(sz)处理）

* 你的realloc 函数应该正确处理realloc(ptr,0)（也就是当作free(ptr)处理）

* 你的free 函数不必处理free(NULL)

* 你的malloc 函数不必处理malloc(0)

* 如果在多个线程中使用libevent，替代的内存管理函数需要是线程安全的。

* libevent 将使用这些函数分配返回给你的内存。所以，如果要释放由libevent 函数分配
和返回的内存，而你已经替换malloc 和realloc 函数，那么应该使用替代的free 函数。


event_set_mem_functions 函数声明在<event2/event.h>中，在libevent 2.0.1-alpha 版本中
首次出现。

可以在禁止event_set_mem_functions 函数的配置下编译libevent 。这时候使用
event_set_mem_functions 将不会编译或者链接。

在2.0.2-alpha 及以后版本中，可以通过检查是否定义了EVENT_SET_MEM_FUNCTIONS_IMPLEMENTED 宏来确定event_set_mem_functions 函数是否存在。









