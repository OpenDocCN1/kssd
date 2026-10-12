
## 调试输出与系统差异分析

```
kd> dq nt!KeServiceDescriptorTableShadow

fffff800`03eab880  ffff800`03c75b00 00000000`00000000
fffff800`03eab890  00000000`00000191 ffff800`03c7678c
fffff800`03eab8a0  ffff960`00111c00 00000000`00000000
fffff800`03eab8b0  00000000`0000033b ffff960`0011391c
fffff800`03eab8c0  00000000`7771fdd6 00000000`00000000
fffff800`03eab8d0  ffff800`00a01400 ffff800`00a013b0
fffff800`03eab8e0  00000000`00000002 00000000`00005bdb
fffff800`03eab8f0  00000000`00023f05 00000000`00000000
```

两种方法找到的位置一样。我这里的输出都针对 WIN7 64 位系统的调用，通过对 2003 进行调试发现这样的方法也适用。因为暂时没有其它版本操作系统，在其它操作系统上的适用性就无从知道了。个人推测 `XP`、`VISTA` 也应该差不多，但需要事实证明。

不过 `Win7` 与 `2003` 中 `SSDT` 表项内容略有不同，虽然都是四个节，最后四位是例程的参数个数，但在 `2003` 上偏移的计算是 `SSDT` 基址 + `表项值 & 0xFFFFFFF0`，而 `Win7` 上则变成了 `SSDT` 基址 + `表项值 >> 4`。如果 `HOOK` 表项还是要做一个分别对待。

我是第一次发帖，大家轻拍啊。上面阐述如有错误请个位大侠多多指教，小弟不胜感激。

## 获取 Win x64 系统调用地址的方法

转一个老外给的公式：

`Get Win x64 Nt syscall from its number: ln nt!KiServiceTable+((poi(nt!KiServiceTable+4\*<syscall nbr>)&0x00000000\`ffffffff)>>4)`

`Win x64 Win32k syscall: ln win32k!W32pServiceTable+((poi(win32k!W32pServiceTable+4\*(<syscall nbr>-1000))&0x00000000\`ffffffff)>>4)-10000000`

> *最初由 **lcz** 发布* ![查看帖子](img/showthread.php?p=899503#post899503) ...

再有就是通过在 `nt!KeAddSystemServiceTable` 函数中进行反汇编搜索。可是在 64 位 `WINDOWS` 中这两种方法都行不通...

## `KeAddSystemServiceTable` 实现分析

`KeAddSystemServiceTable` 不访问 `SystemServiceTable` 如何实现 `Add`？下面是 `Vista64` 的 `KeAddSystemServiceTable` 代码：

```asm
PAGE:0000000000753DF0 KeAddSystemServiceTable proc near
PAGE:0000000000753DF0
PAGE:0000000000753DF0 arg_20           = dword ptr 28h
PAGE:0000000000753DF0
PAGE:0000000000753DF0                 mov     eax, [rsp+arg_20]
PAGE:0000000000753DF4                 cmp     eax, 1
PAGE:0000000000753DF7                 ja      short loc_753E65
PAGE:0000000000753DF7
PAGE:0000000000753DF9                 mov     r10, rax
PAGE:0000000000753DFC                 lea     r11, cs:400000h
PAGE:0000000000753E03                 shl     r10, 5
PAGE:0000000000753E07                 cmp     qword ptr [r10+r11+KeServiceDescriptorTable.ServiceTable-400000h], 0
PAGE:0000000000753E10                 jnz     short loc_753E65
PAGE:0000000000753E10
PAGE:0000000000753E12                 cmp     qword ptr [r10+r11+KeServiceDescriptorTableShadow.ServiceTable-400000h], 0
PAGE:0000000000753E1B                 jnz     short loc_753E65
PAGE:0000000000753E1B
PAGE:0000000000753E1D                 cmp     eax, 1
PAGE:0000000000753E20                 mov     [r10+r11+KeServiceDescriptorTableShadow.ServiceTable-400000h], rcx
PAGE:0000000000753E28                 mov     [r10+r11+KeServiceDescriptorTableShadow.ServiceBase-400000h], rdx
PAGE:0000000000753E30                 mov     [r10+r11+KeServiceDescriptorTableShadow.ServiceLimit-400000h], r8d
PAGE:0000000000753E38                 mov     [r10+r11+KeServiceDescriptorTableShadow.ArgumentTable-400000h], r9
PAGE:0000000000753E40                 jz      short loc_753E62
PAGE:0000000000753E40
PAGE:0000000000753E42                 mov     [r10+r11+KeServiceDescriptorTable.ServiceTable-400000h], rcx
PAGE:0000000000753E4A                 mov     [r10+r11+KeServiceDescriptorTable.ServiceBase-400000h], rdx
PAGE:0000000000753E52                 mov     [r10+r11+KeServiceDescriptorTable.ServiceLimit-400000h], r8d
PAGE:0000000000753E5A                 mov     [r10+r11+KeServiceDescriptorTable.ArgumentTable-400000h], r9
PAGE:0000000000753E5A
PAGE:0000000000753E62
PAGE:0000000000753E62 loc_753E62:                    ; CODE XREF: KeAddSystemServiceTable+50
PAGE:0000000000753E62                 mov     al, 1
PAGE:0000000000753E64                 retn
PAGE:0000000000753E64
PAGE:0000000000753E65 ; ---------------------------------------------------------------------------
PAGE:0000000000753E65
PAGE:0000000000753E65 loc_753E65:                    ; CODE XREF: KeAddSystemServiceTable+7
PAGE:0000000000753E65                                        ; KeAddSystemServiceTable+20
PAGE:0000000000753E65                                        ; KeAddSystemServiceTable+2B
PAGE:0000000000753E65                 xor     al, al
PAGE:0000000000753E67                 retn
PAGE:0000000000753E67
PAGE:0000000000753E67 KeAddSystemServiceTable endp
```

`WIN7` 会如何实现呢？希望楼主能上传一份 64 位 `WIN7` 的 `NTOSKRNL.EXE`。

小弟汇编能力有限，对于

`4b898c1a80882a00 mov qword ptr [r10+r11+2A8880h],rcx`

不会解析，所以觉得在这个函数里搜索不太方便。我文中的描述确实不对。谢谢，大侠指点。

## `nt!KeAddSystemServiceTable` 调试输出

`nt!KeAddSystemServiceTable:`

```
fffff800`03fe5cd0 8b442428        mov     eax,dword ptr [rsp+28h]
fffff800`03fe5cd4 83f801          cmp     eax,1
fffff800`03fe5cd7 776c            ja      nt!KeAddSystemServiceTable+0x75 (fffff800`03fe5d45)
fffff800`03fe5cd9 4c8bd0          mov     r10,rax
fffff800`03fe5cdc 4c8d1d1dd3c1ff lea     r11,[nt!MmIsSessionAddress <PERF> (nt+0x0) (fffff800`03c03000)]
fffff800`03fe5ce3 49c1e205        shl     r10,5
fffff800`03fe5ce7 4b83bc1a40882a0000 cmp    qword ptr [r10+r11+2A8840h],0
fffff800`03fe5cf0 7553            jne     nt!KeAddSystemServiceTable+0x75 (fffff800`03fe5d45)
fffff800`03fe5cf2 4b83bc1a80882a0000 cmp    qword ptr [r10+r11+2A8880h],0
fffff800`03fe5cfb 7548            jne     nt!KeAddSystemServiceTable+0x75 (fffff800`03fe5d45)
fffff800`03fe5cfd 4b898c1a80882a00 mov     qword ptr [r10+r11+2A8880h],rcx
fffff800`03fe5d05 4b89941a88882a00 mov     qword ptr [r10+r11+2A8888h],rdx
fffff800`03fe5d0d 4789841a90882a00 mov     dword ptr [r10+r11+2A8890h],r8d
fffff800`03fe5d15 4f898c1a98882a00 mov     qword ptr [r10+r11+2A8898h],r9
fffff800`03fe5d1d 83f801          cmp     eax,1
fffff800`03fe5d20 7420            je      nt!KeAddSystemServiceTable+0x72 (fffff800`03fe5d42)
fffff800`03fe5d22 4b898c1a40882a00 mov     qword ptr [r10+r11+2A8840h],rcx
fffff800`03fe5d2a 4b89941a48882a00 mov     qword ptr [r10+r11+2A8848h],rdx
fffff800`03fe5d32 4789841a50882a00 mov     dword ptr [r10+r11+2A8850h],r8d
fffff800`03fe5d3a 4f898c1a58882a00 mov     qword ptr [r10+r11+2A8858h],r9
fffff800`03fe5d42 b001            mov     al,1
fffff800`03fe5d44 c3              ret
fffff800`03fe5d45 32c0            xor     al,al
fffff800`03fe5d47 c3              ret
fffff800`03fe5d48 90             nop
fffff800`03fe5d49 90             nop
fffff800`03fe5d4a 90             nop
fffff800`03fe5d4b 90             nop
fffff800`03fe5d4c 90             nop
fffff800`03fe5d4d 90             nop
fffff800`03fe5d4e 90             nop
fffff800`03fe5d4f 90             nop
```

## 上传的附件

| 文件 |
| :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) `ntoskrnl.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) `ntoskrnl00.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) `ntoskrnl01.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) `ntoskrnl1_rev.rar` |
