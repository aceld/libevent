# 1 Libevent官方

 * 官方网站：http://libevent.org/ 

libevent版本一共有1.4系列和2.0系列两个稳定版本。



>1.4系列比较古老，但是源码简单，适合源码的学习

>2.0系列比较新，见识直接使用2.0

需要注意的是，1.4系列和2.0系列两个版本的接口并不兼容，就是2.0将一些接口的原型发>生了改变，所以将1.4升级到2.0需要重新编码。

#1.1 libevent 特点

* 事件驱动，高性能；
* 轻量级，专注于网络； 
* 跨平台，支持 Windows、Linux、Mac Os等； 
* 支持多种 I/O多路复用技术， epoll、poll、dev/poll、select 和kqueue 等； 
* 支持 I/O，定时器和信号等事件；


#1.2 libevent下载与安装


在官网上找到`libevent-2.0.22-stable.tar.gz`下载地址。

```bash

tar -zxvf libevent-2.0.22-stable.tar.gz

cd libevent-2.0.22-stable/

./configure

make

sudo make install
```

