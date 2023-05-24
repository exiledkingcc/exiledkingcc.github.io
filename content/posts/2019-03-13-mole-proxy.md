---
date:     2019-03-13
title:    mole proxy
summary:  a simple encrypted proxy based on socks5, inspired by shadowsocks
categories: ["coding"]
---

前段时间用`Python`写了一个简单的加密代理代理程序，是仿照`shadowsocks`写的。它是可以work的，
但也就是仅仅work而已。我并不打算将它作为日常的程序使用，以取代`shadowsocks`，只是为了试一下`Python`
的`asyncio`。当然，假如以后`shadowsocks`无法使用了，也是可以将它完善一下，用起来的。

代码托管在我的`github`上：[mole](https://github.com/exiledkingcc/mole)。

顺便说一下，`mole`这个名字来源于一本童话**`The Wind in the Willows`**，推荐大家看看。

---

设计上比较简单，由两部分组成：`MoleClient`和`MoleServer`。`MoleClient`运行在本地，
对于实际的clients，它相当于一个[`socks5`](https://www.ietf.org/rfc/rfc1928.txt)服务（目前只支持`Connect`），
clients与它之间使用`socks5`协议。`MoleServer`运行在防火墙外，它与`MoleClient`之间的通信是加密的。

一个具体的流程大致如下：一个client，比如浏览器，它先使用`socks5`协议与`MoleClient`连接，
然后`MoleClient`与`MoleServer`连接，`MoleClient`给`MoleServer`发送（加密的）`Connect`命令，
然后`MoleServer`连接`Connect`命令中的目的地址，这样通信就建立起来了。然后只需要把双方的通信加密转发就行了。
也就是：浏览器发给`MoleClient`的数据，由`MoleClient`加密后发给`MoleServer`，`MoleServer`解密后发给目的地址；
目的地址发给`MoleServer`的数据，由`MoleServer`加密后发给`MoleClient`，`MoleClient`解密后发回浏览器。
注意这个过程是双向的，而且特别重要的是`TCP`是数据流，而不是数据包。

加密使用的是`libsodium`库，只用了`xchacha20poly1305_ietf`这个加密算法。加密数据包结构为`len(2bytes)+nonce(2bytes)+data`，
最终的`nonce`有`24bytes`，使用`hash`计算出来。`key`是预先定义的，最终的`key`也是由`hash`计算。连接命令使用相同的`key`，
后续每个新的连接都使用随机的`key`。

---

简单说一下代码。

`cclog.py`：给log加上颜色，基本上我所有的python程序都会使用它。

`utils.py`：地址与端口转换

`libsodium.py`：对`libsodium`库的简单`wrapper`。只是封装了和`xchacha20poly1305_ietf`算法相关的几个函数。

`crypto.py`：加密与解密。这是对具体加密算法的封装，为了以后可以方便使用其它加密算法。这里可以看到密文数据的结构。

`socks5.py`：这是一个简单的`socks5`代理代理程序。

`moles.py`：包含主要的`MoleClient`和`MoleServer`，也就是把`socks5`拆成两部分，然后中间数据加密。

`main.py`：程序入口。

---

测试结果还可以。`Python`虽然是动态语言，开销比较大，而且存在`GIL`，被普遍认为是比较低效的。然而这个任务是`I/O bound`的，
而`Python`的`asyncio`正好适合这样的任务，使用其它语言不会有太大的增益。但是还是有可以优化的地方，比如`multithreading+epoll`。
不过即便这样，网速仍然是最大的瓶颈。

