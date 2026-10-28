
# QQ密码框的那点破事

前阵子把QQ密码搞出来了，现在想复习下发现又生疏了，看来要写个文章利用看雪论坛帮自己保存下，特别是`WinDbg`的命令忘的太快了，恩要记下来了。

注：本文没有用到驱动来实现，也没有用到假界面这个方法，只是借助于`WinDbg`分析，代码是`Ring3`的，`QQ`版本是2011,版本号是1.65.2221.488即最新的版本beta 3.3

## 钩子分析

`QQ`密码框是靠不停的安装卸载钩子来实现密码保护的，这地球人都知道，主要的2个钩子是`WH_DEBUG`和`WH_KEYBOARD_LL`,但你断`SetWindowsHookExA`或者`SetWindowsHookExW`都是断不下来的，只能在`QQ`启动时断下来几个无关紧要的键盘钩子，那么`WH_DEBUG`和`WH_KEYBOARD_LL`这2个钩子从哪里来的，下面一步步分析：

1、毫无疑问`QQ`已经将代码进行了转移，但作为一个无壳无驱动的聊天程序，肯定还是要经过内核中的函数的，`SetWindowsHookEx`以及`SetWindowsHook`的内核函数都是`Win32k!NtUserSetWindowsHookEx`,它是在`ShadowSSDT`表中的。

2、开虚拟机，`WinDbg`连上,先将`QQ`打开停在登陆界面，这时候`QQ`内部就会不停的加载和卸载钩子，这是因为有定时器的缘故，后面再说。然后在`WinDbg`中`Ctrl+Break`,输入下面的命令

```
kd> !process 0 0 qq.exe
```

```
PROCESS 81a98260  SessionId: 0  Cid: 0640    Peb: 7ffdf000  ParentCid: 0540
    DirBase: 084001a0  ObjectTable: e12cb3b0  HandleCount: 280.
    Image: QQ.exe
```

找到`qq`的`EPROCESS`为`81a98260`

3、下断点：

```
bu /p 81a98260  Win32k!NtUserSetWindowsHookEx
```

则断在`QQ`的进程空间中，这就是加上`/p`的好处,详解参见帮助文档

4、输入命令 `kb`

```
kd> kb
ChildEBP RetAddr  Args to Child
f8202d44 8053e638 00400000 0012f9a8 00000000 win32k!NtUserSetWindowsHookEx
f8202d44 7c92e4f4 00400000 0012f9a8 00000000 nt!KiFastCallEntry+0xf8
0012f980 1000a20c 100072e1 00400000 0012f9a8 ntdll!KiFastSystemCallRet
```

看见了吧 被我们纠出来了，分析堆栈结构

`win32k!NtUserSetWindowsHookEx`会返回到`nt!KiFastCallEntry`中而`nt!KiFastCallEntry`会调用`sysexit`返回到`Ring3`,返回`Ring3`的地址是`7c92e4f4`

这是`ntdll!KiFastSystemCallRet`函数,只有一个指令是`ret`,这个函数又会返回到`1000a20c` 中，`1000a20c` 地址是`QQ`的`TsSafeEdit.dat`模块中

5、将断点清除，系统继续运行，用`OD`附加，来到`1000a20c` 处，往上看就会看到你想要的,如图:

![](https://bbs.kanxue.com/upload/attach/201111/325827_oouc09xsf5bnts5.png)

`1125`就是`Win32k!NtUserSetWindowsHookEx`在`ShadowSSDT`中的序列号，因为在`ShadowSSDT`中所以算函数表中的偏移要减`0x1000`,

具体的原因请参见http://bbs.pediy.com/showthread.php?t=89407

6、`1000A200`就是`QQ`秘密调用生成`WH_DEBUG`和`WH_KEYBOARD_LL`钩子的地方，而且一直地址不变，始终是模块`TsSafeEdit.dat`的地址+`A200`,但这个地址一开始是没有代码的，`QQ`会用`memcpy`函数将代码复制过来执行，

所以我们要`Hook`住`memcpy`,判断目标地址是不是`TsSafeEdit.dat`的地址+`A200`，然后再到这个`1000A200`进行猥琐,当然我这个是笨方法。。。

7、好了，怎么得到`QQ`密码的思路就出来了，

（1）`Hook`住`memcpy`,判断目标地址是不是`TsSafeEdit.dat`的地址+`A200`，然后再那个地址写入`mov eax,1 ret 18h`,即成功的返回

（2）加载我们自己的钩子来得到密码。。。。。。。。。不罗嗦了，都在代码里，以下`QQ`账号和密码的截图：

![](https://bbs.kanxue.com/upload/attach/201111/325827_vnp0zl12i361h9e.png)

8、实际上`QQ`不停的加载和卸载钩子是`SetTimer`的缘故，只要`Hook`住`SetTimer`就可以了，`Hook`中判断句柄的窗口类型是不是`Edit`就可以了，但这种方法，`WH_DEBUG`和`WH_KEYBOARD_LL`的钩子还是存在的，得到的数据虽然不完全是按键数据，但又规律可循

9、附件中的工程编译出来的是`DLL`,用`OD`在程序载入点的时候加载`DLL`就可以了，写着玩的所以代码不规范。。。。，还有`BUG`,但基本思路就这样，另外如果想做成木马的话，

还需要过`360`对`QQ`目录下的文件保护，保证在`QQ`启动的时候加载`DLL`,至于怎么过我就不透露了![](https://bbs.kanxue.com/view/img/face/1.gif)，另外如果用户是记住密码的，可以在将用户的`db`文件删除，这样用户

必须输入密码了，只想到这种，代码中获取`QQ`号是`Hook`住了明文处，恩 ，就这样了

[HookQQ.rar](https://bbs.kanxue.com/attach-download-62372.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [钩子.png](javascript:void(0);)

  （6.18kb，1621次下载）

- [密码.png](javascript:void(0);)

  （80.04kb，1625次下载）

- [HookQQ.rar](javascript:void(0);)

  （9.39kb，682次下载）
