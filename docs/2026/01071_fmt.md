
# [原创]踩坑小条。。不断踩坑中

在学习的过程中，遇到了很多无语的问题，有些地方可能是没注意，经过一点拨就能发现，还有是一些小技巧。这里记录下来提醒自己并且分享出来。

在这里我把自己遇到的，以及以后将碰到的以条目的形式给出，也欢迎大家在帖子下面评论，分享自己遇到的问题。

***

* 如果调试器`f7`或者`f8`不管用了，可以看看和哪些应用冲突了，我的是`f8`和网易有道词典的热键冲突了。。
* `ida`远程调试`wsl1`不能成功，可以尝试`wsl1`升级到`wsl2`
* `ida7.5`可以`ctrl+z`撤销，反编译窗口可以回车换行
* 在`pwntools`中`gdb`调试的时候，如果显示`*gdb server not found*`，很有可能不是路径的问题，而是真的需要`sudo apt install`它（我是憨憨）。
* `pwntools`调用`gdb`后如果出来的窗口太丑，可以在之前尝试设置`context.terminal`，常用的有

```
context.terminal = ['gnome-terminal', '-x', 'sh', '-c']
context.terminal = ['tmux', 'splitw', '-h']
context.terminal = ['tmux', 'splitw', '-v']
```

* `wsl`不能运行32位程序，需要用`qemu`，并且加上下面第一段代码。为了方便，我直接在`~/.bashrc`里写了第二段代码。但是`wsl`不能`gdb`调试32位...至今没找到解决办法。

```
sudo update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
```

```
echo toor |sudo -S update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
```

* 如果`ida`在反编译结果上下断点，程序运行起来停不住，或者出现下面的提示，那么就要到菜单中进行修改：`Debugger`->`Use source-level debugging`

![ida源代码层级调试](upload/attach/202105/857437_UM6F8BP8QTT4E2P.png)

* 今天打开夜神模拟器的时候打不开，提示开启了`hyper-v`，需要关闭。想来自己开启了`wsl2`,在此记录开启关闭`hyper-v`的方法。按照上述方法执行之后仍然无法开启`wsl2`，尝试这个。
* 如果程序中使用了复杂的宏函数，最后表达式的结果不对，尝试将宏函数套上括号，否则有可能会因为运算优先级而结果错误。更新：建议宏定义的时候就写好括号。
* `C`多线程编程，`printf`打印不出来东西，换成`write`就好了，或者输出缓冲区设置成`0`也可以解决问题。
