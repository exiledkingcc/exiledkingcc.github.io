---
layout:   post
title:    how busybox symlink works
date:     2016-10-22
summary:  busybox通过软连接的名字就可以运行不同的程序
category: linux
tags:     linux busybox python
---

busybox通过软连接的名字就可以运行不同的程序。

比如建立如下软连接：

```bash
$ln -s busybox mv
$ln -s busybox rm
```

执行mv的时候就是移动文件，执行rm的时候就是删除文件。

这是怎么做到的呢？下面用python来演示一下。

首先写一个简单的test.py

```python
import sys

if __name__ == "__main__":
    cmd = sys.argv[0]
    print("exec command", cmd)
```

运行这个程序`python3 test.py`，你会看到输出：`exec command test.py`。


下面试一下改一下文件名：`mv test.py main.py`，然后再执行`python3 main.py`，
你会看到输出`exec command main.py`

再试试软连接：`ln -s main.py test`。运行`python3 test`会看到`exec command test`。

这里的`sys.argv`就是关键所在，
它保存了我们的命令与参数。
对于C/C++的话是就是main函数的参数。
通过这个，就可以在运行时根据文件名来执行不同的逻辑。

所以对于busybox，它的基本逻辑是这样的：

```python
import sys

def command_abc(*args):
    print("exec command abc")

def command_def(*args):
    print("exec command def")

CMDS = {"abc": command_abc, "def": command_def}

if __name__ == "__main__":
    cmd = sys.argv[0]
    args = sys.argv[1:]
    if cmd == "main.py":
        cmd = sys.argv[1]
        args = sys.argv[2:]
    CMDS[cmd](*args)
```

我们现在有main.py，分别建立软连接abd与def到main.py。
执行`python3 main.py abc`与`python3 abc`是一样的。

说到`sys.argv`，一般我喜欢这样写（比如有一个参数表示文件名）：

```python

def main(script, filename, *argv):
    # do something

if __name__ == "__main__":
    main(*sys.argv)

```
