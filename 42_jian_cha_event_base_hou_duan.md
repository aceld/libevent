# 4.2 检查event_base后端

有时候需要检查 event_base 支持哪些特征,或者当前使用哪种方法。

###接口1
```cpp
const char **event_get_supported_methods(void);
```

event_get_supported_methods()函数返回一个指针 ,指向 libevent 支持的方法名字数组 。 这个数组的最后一个元素是 NULL。

###实例：

```cpp
int i;
const char **methods = event_get_supported_methods();
printf("Starting Libevent %s.  Available methods are:\n",
    event_get_version());
for (i=0; methods[i] != NULL; ++i) {
    printf("    %s\n", methods[i]);
}
```

>这个函数返回 libevent 被编译以支持的方法列表 。然而 libevent 运行的时候,操作系统可能 不能支持所有方法。比如说,可能 OS X 版本中的 kqueue 的 bug 太多,无法使用。



###接口2

```cpp
const char *
event_base_get_method(const struct event_base *base);

enum event_method_feature 
event_base_get_features(const struct event_base *base);
```

event_base_get_method()返回 event_base 正在使用的方法。

event_base_get_features ()返回 event_base 支持的特征的比特掩码。


###实例

```cpp
struct event_base *base;
enum event_method_feature f;

base = event_base_new();
if (!base) {
    puts("Couldn't get an event_base!");
} else {
    printf("Using Libevent with backend method %s.",
        event_base_get_method(base));
    f = event_base_get_features(base);
    if ((f & EV_FEATURE_ET))
        printf("  Edge-triggered events are supported.");
    if ((f & EV_FEATURE_O1))
        printf("  O(1) event notification is supported.");
    if ((f & EV_FEATURE_FDS))
        printf("  All FD types are supported.");
    puts("");
}
```

