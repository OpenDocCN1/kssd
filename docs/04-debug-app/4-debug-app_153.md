
# 一个光刻软件在 winnt 内核下的修复

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-211.htm`](https://www.pediy.com/kssd/pediy09/pediy09-211.htm)

> 朋友请我帮忙修复一个工业用光刻软件，他描述该软件在 `WIN98` 下能正常运行，但是在 `WIN2000` 及 `WINXP` 下无法运行，首先我就想到工业用软件，可能会操作硬件端口，非常可能是权限问题。问他拿到软件后直接运行，出错。

> 开 `OD` 加载 `6020dll.dll` 跑到偏移 `0x578b` 处

```
10005780   $  33C0          XOR EAX,EAX
10005782   .  66:8B5424 04 MOV DX,WORD PTR SS:[ESP+4]
10005787   .  8A4424 08    MOV AL,BYTE PTR SS:[ESP+8]
1000578B   .  EE           OUT DX,AL                      ; I/O 命令
1000578C   .  C3           RETN
```

> 看到和我开始想的相同，操作了硬件端口。由于 `WINNT` 不能直接操作端口所以产生了此错误，知道问题现在就好办了。确立解决方法：

> 使用 `WINIO`，在他首次操作端口前加载 `WINIO` 即可。我选择的是他首次调用 `0x10005780` 前跳转到模块空的代码段处加载 `WINIO`，补上代码如下

```
1000B200   >  \\60         PUSHAD
1000B201   .  68 E0B10010  PUSH 6020dll.1000B1E0         ; /FileName = "WinIo.dll"
1000B206   .  FF15 ACC00010 CALL DWORD PTR DS:[<&KERNEL32.LoadLibrar>; \\LoadLibraryA
1000B20C   .  68 EAB10010  PUSH 6020dll.1000B1EA         ; /ProcNameOrOrdinal = "InitializeWinIo"
1000B211   .  50         PUSH EAX                         ; |hModule
1000B212   .  FF15 48C00010 CALL DWORD PTR DS:[<&KERNEL32.GetProcAdd>; \\GetProcAddress
1000B218   .  FFD0       CALL EAX
1000B21A   .  61         POPAD
1000B21B   .  E8 60A5FFFF CALL 6020dll.10005780 //补上由于跳转到空代码段处覆盖掉的代码
1000B220   .^ E9 E59DFFFF JMP 6020dll.1000500A //跳回原处继续执行
```

> 发还给我朋友已能正常运行了，至此该光刻软件在 `winnt` 下的修复已经完成，个人认为，修复此类东西需要平时经验的积累，恰当的使用工具，即可达到你所想要的目的。

---

【版权声明】本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
