# 6.6 手动激活事件

极少数情况下,需要在事件的条件没有触发的时候让事件成为激活的。

```cpp
void event_active(struct event *ev, int what, short ncalls);
```

这个函数让事件 ev 带有标志 what(EV_READ、EV_WRITE 和 EV_TIMEOUT 的组合)成 为激活的。事件不需要已经处于未决状态,激活事件也不会让它成为未决的。