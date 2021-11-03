# 1 Libevent官方

 * 官方网站：http://libevent.org/ 

libevent版本一共有1.4系列和2.0系列两个稳定版本。



>1.4系列比较古老，但是源码简单，适合源码的学习

>2.0系列比较新，建议直接使用2.0

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

>注意
>
如果在libevent安装目录make之后会生成一个.libs/， 里面如果没有libevent_openssl.so说明系统没有安装openssl库。
但是如果安装了，依然没有这个文件生成，可能需要制定openssl路径

```bash
ln -s  /usr/local/ssl/include/openssl    /usr/include/openssl  
```


#1.3 libevent开源包

在`.libs`隐藏文件中包含全部libevent已经编译好的so文件。

其中core为libevent的核心文件，libevent.so为主链接文件，会关联到其他全部so文件。


在sample目录下会有已经编译好的服务器应用程序。

可以拿`hello-world`程序用来测试。

服务端:

```bash
./hello-world
```

客户端:

```bash
netcat 192.168.2.105 9995
```

如果客户端收到“hello world”字符串，表示libevent在本机可以正常使用。
