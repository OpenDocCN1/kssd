
# [原创]Hide idt Hook By DrxHOOK

前段时间看了zhouws的[hide idt hook src](http://bbs.pediy.com/showthread.php?t=142627),突然以前自己也瞎搞了一个隐藏IDT HOOK的玩意,差不多快烂了,于是整理一下也发布出来.

原理:与zhouws不同, 这个主要Pass检测, DrX Hook + 代码流离

```
lkd> u KiTrap01
nt!KiTrap01:
8054311c 6a00            push    0
8054311e 66c74424020000  mov     word ptr [esp+2],0
80543125 55              push    ebp
80543126 53              push    ebx
80543127 56              push    esi
80543128 57              push    edi
80543129 0fa0            push    fs
8054312b bb30000000      mov     ebx,30h
```

`A`为`8054311e`, `B`为`80543125`

首先在`B`处设Inline Hook,这里我使用`int 0C7`做代理跳转到`NewDBEntryAB`, 然后在`A`下读写硬件断点,注意读写跟执行硬件断点是分离

当`A`处被读取的时候,`NewDBEntryAB`会捉捕到,然后清除断点,还原INLINE HOOK,接着在`A`处设INLINE HOOK,在`B`处设读断点

同理,`B`处被读取的时候,`A`处Inline HOOK, `B`处读断点,这样循环执行,总的来说就是跟检测代码躲猫猫.

效果如下:

![](https://bbs.kanxue.com/upload/attach/201112/108202_fgkrdaalldb32pl.jpg)

不过`WINDBG u KiTrap01`貌似没被PASS,代码正常运行,弄了好久都搞不明白,求指教![](https://bbs.kanxue.com/view/img/face/50.gif)

![](https://bbs.kanxue.com/upload/attach/201112/108202_4qjvjrwpgkey0fy.jpg)

附上代码,环境VS2008,WinDDK 7600.16385.1

系统XP SP2

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`自检.jpg`](javascript:void(0);)
  （221.20kb，431次下载）
- [`UnPass.jpg`](javascript:void(0);)
  （20.16kb，414次下载）
- [`HideIDTHook.rar`](javascript:void(0);)
  （11.45kb，165次下载）
