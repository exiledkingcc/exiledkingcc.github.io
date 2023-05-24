---
date:     2018-04-22
title:    password as short as possible
summary:  密码越短越好
tags: ["password", "hash"]
categories: ["coding"]
---

据传12306网站的密码泄漏了，我决定改一下密码。然而尝试了几次密码才搞对了，最后改密码也不知道改成什么好，
真是麻烦啊。

---

我所有的密码都是自己记忆的，然而除了常见的几个网站，其它的实在是记不住。每次的操作都是选择忘记密码，
然后重置密码，结果弹出提示说不能使用之前的密码，实在是淡疼啊。

然后我想着应该使用密码管理软件了。网上看了很多信息后，还是放弃了。密码管理器都是生成随机密码，
然后用一个主密码（‵master key‵）进行加密，这样就只用记住一个密码了。然而加密的数据需要存起来，
在不同平台同步也很麻烦。一些同步还是通过密码管理软件官方进行同步的，存在一定风险。而且要是这些
数据被别人获取了，也存在很大的安全风险。

最后我想的是，最好能有一个不用存储，可以根据关键字来生成无规律的字符串作为密码，这样只用记忆简单的关键字就好了。
于是想到了一种简单的方法，如下：

1. 给一个随机的值，作为`random seed`
2. 生成（伪）随机盐：`salt`
3. 将`salt`与关键字拼在一起，计算`hash`
4. 将`hash`值进行`base64`编码

python示例代码如下：
```python
import base64
import hashlib
import getpass
import random


def main():
    seed = getpass.getpass("seed: ")
    key = getpass.getpass("keyword: ")

    # python's random seed can be `str`
    random.seed(seed)

    salt = "".join([chr(random.randrange(256)) for _ in range(16)])
    data = (key + salt).encode("utf-8")
    hash = hashlib.sha512(data).digest()
    passwd = base64.b64encode(hash, b"_$").decode("ascii")
    print(passwd)
    for i in range(10, 31):
        print("len={}: {}".format(i, passwd[:i]))


if __name__ == '__main__':
    main()
```

有几个需要注意的地方：
1. python里面`random seed`可以是字符串
2. python里面的伪随机数算法在未来可能会发生改变

就以这个实现来说，比如12306的密码可以是`seed`与`keyword`都是字符串的`12306`。结果会是：
```bash
qOjXx1udxm_cbv00v26PMX8ScOYUuz6b1K_P4ZXgBUU6JR8WWLvpcRBPBYx0MCjGtwZ2Up8kRyKfdIkTY590WA==
len=10: qOjXx1udxm
len=11: qOjXx1udxm_
len=12: qOjXx1udxm_c
len=13: qOjXx1udxm_cb
len=14: qOjXx1udxm_cbv
len=15: qOjXx1udxm_cbv0
len=16: qOjXx1udxm_cbv00
len=17: qOjXx1udxm_cbv00v
len=18: qOjXx1udxm_cbv00v2
len=19: qOjXx1udxm_cbv00v26
len=20: qOjXx1udxm_cbv00v26P
len=21: qOjXx1udxm_cbv00v26PM
len=22: qOjXx1udxm_cbv00v26PMX
len=23: qOjXx1udxm_cbv00v26PMX8
len=24: qOjXx1udxm_cbv00v26PMX8S
len=25: qOjXx1udxm_cbv00v26PMX8Sc
len=26: qOjXx1udxm_cbv00v26PMX8ScO
len=27: qOjXx1udxm_cbv00v26PMX8ScOY
len=28: qOjXx1udxm_cbv00v26PMX8ScOYU
len=29: qOjXx1udxm_cbv00v26PMX8ScOYUu
len=30: qOjXx1udxm_cbv00v26PMX8ScOYUuz
```

你可以从其中选择一个长度作为密码。

当然，对于这个方案，你需要记忆至少三个东西（当然从还可以有更多的参数）：`seed`，`keyword`，`len`，
相比原来只记一个密码看起来要更麻烦。实际上来讲我觉得这样很好记忆。因为你其实不需要刻意去记忆。

对于`seed`，可以是生日，特殊的日子，或者特殊的数字，或者名字（对于上面这个python的代码），这都是不需要特地去记的。不同网站可以就用同一个。
然后是`keyword`，这个就可以是网站的名字，或者网站的域名，或者相关的一个词。比如12306，可以直接用“`12306`”或者“`火车票`”；
比如淘宝网，可以用`taobao`或者“`马云`”或者“`剁手`”等等。最后的`len`，可以也只用一个比较长的值，比如16（好像不少网站密码最长是16位），
当然有些脑残网站会觉得你密码太长“不安全”，那就用网站支持的最长长度。看上去要记三个不同的东西，实际上比原来不同网站一个要好记很多。

---

最后。千万不能在不同网站用一个密码！
以及希望所有网站设置密码时不要跳出“不能包含特殊符号”，“长度超过10个字符”这样的SB的提示。
