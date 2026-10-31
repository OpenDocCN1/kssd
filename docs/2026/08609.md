
# [讨论]关于XP的三处debugport清零

处理掉一下五个函数：

`NtOpenProcess`

`NtOpenThread`

`KiAttachProcess`

`NtReadVirtualMemory`

`NtWriteVirtualMemory`

接下来说下`debugport`清零

<http://bbs.pediy.com/showthread.php?p=908680>

在 `ggdd` 的帖子里对前两处清零做了很详细的分析。

那我就分析下第三处清零。

```
A51D2470    78 65    js     A51D24D7

A51D2472    01FB            add    ebx, edi

A51D2474    8B3C24    mov    edi, dword ptr [esp]

A51D2477    83C4 04    add    esp, 4

A51D247A    E9 5BF4FFFF    jmp    A51D18DA

A51D247F    8F02            pop    dword ptr [edx]  //在这里做了清零，此时的栈顶值为0，edx就是debugport地址。出栈指令POP:把栈顶的一个字传送至指定的目的操作数。

A51D2481    E9 30E6FFFF  jmp    A51D0AB6

A51D2486    83C4 04    add    esp, 4

A51D2489    E9 C6EDFFFF  jmp    A51D1254

A51D248E    89E3    mov    ebx, esp
```

## 绕过第三处清零

我的做法很简单，就是在函数头直接写入`{0xC3,0x90}`;

当然这种做法可能有错误。导致我调试器附加后，无限网络连接中段。

求解，这是为什么？大牛看见不要沉默呀！

## 如何找到第三处清零

做法和前两两处一模一样。关于传送的中的监测的点，好像不能说出来的样子，很多帖子都没说，

我就提示一些，如果有人在你的代码空间里乱涂乱画，你要怎么监视？

第一次发帖，而且还是刚注册的临时会员，申请个邀请码。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
