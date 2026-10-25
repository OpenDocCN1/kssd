
# IDA Pro Advanced v5.1.0.899---Bug

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-301.htm`](https://www.pediy.com/kssd/pediy09/pediy09-301.htm)

> 问题：

> 1、菜单 `View->hide all/unhide all` 无法使用

> 2、使用 `DarkNess0ut` 补丁后，当打开文件时，使用菜单 `Options->Colors` 功能，关闭对话框 `IDA` 会自动退出

> 原版就有问题

```asm
0049E350   8D45 C4          LEA EAX,DWORD PTR SS:[EBP-3C]
0049E353   8BCE             MOV ECX,ESI
0049E355   8B38             MOV EDI,DWORD PTR DS:[EAX]
0049E357   2BCF             SUB ECX,EDI
0049E359   8908             MOV DWORD PTR DS:[EAX],ECX
0049E35B   42               INC EDX
0049E35C   83C0 08          ADD EAX,8
0049E35F   83FA 03          CMP EDX,4
0049E362  7C EF            JL SHORT idag.0049E353
```

> 循环次数 4 超出了实际可用的缓冲区空间，破坏了 `seh` 链

> 0049E35F   `CMP EDX,4` 可以改成 `CMP EDX,3`

`4dd466d0e1e0cc123afbe504da2462ca`
