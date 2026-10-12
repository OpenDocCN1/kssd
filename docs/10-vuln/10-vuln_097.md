
# 32 字节的退出进程 Shellcode

> 原文：[`www.pediy.com/kssd/pediy10/68560.html`](https://www.pediy.com/kssd/pediy10/68560.html)

今天在做 PE 优化的时候,挖掘的一种方法,这样任意情况下都可以退出进程了(PE 里可以无 `ExitProcess` 的引入表项目)。

`64 A1 00 00 00 00 83 38 FF 74 0A 8B 00 64 A3 00 00 00 00 EB F1 8B 40 08 8B 40 08 83 C0 13 FF E0`

代码:

```
MOV EAX,DWORD PTR FS:[0]
L001:
CMP DWORD PTR DS:[EAX],-1
JE L006
MOV EAX,DWORD PTR DS:[EAX]
MOV DWORD PTR FS:[0],EAX
JMP L001
L006:
MOV EAX,DWORD PTR DS:[EAX+8]
MOV EAX,DWORD PTR DS:[EAX+8]
ADD EAX,13
JMP EAX
```

方法及特点:枚举 `SEH` 链接至顶层,动态定位到 `ExitProcess` 点。适用于 `Win2000`/`XP`/`2003`,没有 `Vista` 未测试。

呵呵，再小点。26 字节。

`64 A1 00 00 00 00 80 38 FF 74 04 8B 00 EB F7 8B 40 08 8B 40 08 83 C0 13 FF E0`
