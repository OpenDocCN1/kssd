
# 破解某国产编程语言

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-28.htm`](https://www.pediy.com/kssd/pediy09/pediy09-28.htm)

> 【文章标题】: 破解某国产编程语言  
> 【文章作者】: `yaoyuan[CCG]`  
> 【软件名称】: 某国产编程语言  
> 【下载地址】: 自己搜索下载  
> 【加壳方式】: 无  
> 【保护方式】: 硬盘序列号+`RSA2048`  
> 【使用工具】: `OD`  
> 【操作平台】: `WIN32`  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

简单分析一下其在 XP 下的加密过程，软件使用 `deviceiocontrol` 取出硬盘信息，计算出用户号，根据此号生成注册文件分发给最终用户。由于其算法使用了高位的 `RSA`，做注册机基本不可能，除非去偷个私钥^\_^，所以以下分析基于已有一个自已硬盘的正确的注册文件。

这样的加密方法只能通过仿真硬盘信息达到破解，以下为部分代码及注解：

`DeviceIoControl` 原型：

```
BOOL DeviceIoControl(HANDLE hDevice, DWORD dwIoControlCode, LPVOID lpInBuffer,
                    DWORD nInBufferSize, LPVOID lpOutBuffer, DWORD nOutBufferSize,
                    LPDWORD lpBytesReturned, LPOVERLAPPED lpOverlapped);
```

以 IDE 硬盘代码为例，取硬盘信息代码如下：

```
00430230     8B4424 08       mov eax,dword ptr ss:[esp+8]
00430234     B1 01           mov cl,1
00430236     53              push ebx
00430237     33D2            xor edx,edx
00430239     8848 05         mov byte ptr ds:[eax+5],cl
0043023C     8848 06         mov byte ptr ds:[eax+6],cl
0043023F     8A4C24 18       mov cl,byte ptr ss:[esp+18]
00430243     8850 04         mov byte ptr ds:[eax+4],dl
00430246     8AD9            mov bl,cl
00430248     8848 0C         mov byte ptr ds:[eax+C],cl
0043024B     8B4C24 1C       mov ecx,dword ptr ss:[esp+1C]
0043024F     80E3 01         and bl,1
00430252     80CB FA         or bl,0FA
00430255     8850 07         mov byte ptr ds:[eax+7],dl
00430258     8850 08         mov byte ptr ds:[eax+8],dl
0043025B     52              push edx
0043025C     8B5424 14       mov edx,dword ptr ss:[esp+14]
00430260     51              push ecx
00430261     C0E3 04         shl bl,4
00430264     8858 09         mov byte ptr ds:[eax+9],bl
00430267     8A5C24 1C       mov bl,byte ptr ss:[esp+1C]
0043026B     68 10020000     push 210                        ;OutBufferSize
00430270     52              push edx        ;OutBuffer
00430271     6A 20           push 20                     ;InBufferSize
00430273     8858 0A         mov byte ptr ds:[eax+A],bl                ;InBuffer
00430276     C700 00020000   mov dword ptr ds:[eax],200
0043027C     50              push eax
0043027D     8B4424 20       mov eax,dword ptr ss:[esp+20]
00430281     68 88C00700     push 7C088              ;IoControlCode=SMART_RCV_DRIVE_DATA
00430286     50              push eax                ;hDevice
00430287     FF15 34625400   call dword ptr ds:[<&KERNEL32.DeviceIoControl>]  ; kernel32.DeviceIoControl
0043028D     5B              pop ebx
0043028E     C3              retn
```

调用取硬盘信息的代码

```
004304D8     50              push eax
004304D9     8D5424 38       lea edx,dword ptr ss:[esp+38]
004304DD     51              push ecx                ;ECX 就是 OutBuffer
004304DE     52              push edx
004304DF     56              push esi
004304E0     E8 4BFDFFFF     call 00430230
004304E5     83C4 18         add esp,18
004304E8     85C0            test eax,eax
004304EA     74 2D           je short 00430519
```

执行完 `00430230` 这个 `CALL` 后，硬盘信息被取出，我们要替换的正是这一部分数据。将此 `210h` 数据存盘为 `hd.dat`。

替换方法：

用 `topo` 在程序中增加 `300h` 字节，将取出的 `210` 字节数据粘到新加的字节中，将 `4304E0` 处代码改为跳转到自己的处理程序。

```
004304E0     E9 06301B00     jmp 005E34EB
```

```
005E34EB     E9 C6000000     jmp 005E35B6
```

以下代码传送了 `e2` 个字节(`210` 字节并不是全都有用),`5e34f0-5e35b2` 为记录数据。用到的小技巧注解如下：

```
005E35B6     50              push eax
005E35B7     E8 00000000     call 005E35BC
005E35BC     58              pop eax
005E35BD     51              push ecx
005E35BE     59              pop ecx                 ;在栈中保存 OutBuffer 值
005E35BF     90              nop
005E35C0     58              pop eax
005E35C1     E8 6ACCE4FF     call 00430230           ;调用 DeviceIoControl 平衡堆栈
005E35C6     51              push ecx
005E35C7     56              push esi
005E35C8     57              push edi
005E35C9     E8 00000000     call 005E35CE
005E35CE     5E              pop esi                 ;取 EIP 值到 ESI
005E35CF     81EE DE000000   sub esi,0DE             ;减去增加代码长度得到源数据地址 5e34f0
005E35D5     8B7C24 F4       mov edi,dword ptr ss:[esp-C]      ;取出栈中保存 OutBuffer 值
005E35D9     B9 E2000000     mov ecx,0E2
005E35DE     F3:A4           rep movs byte ptr es:[edi],byte ptr ds:[esi]
005E35E0     5F              pop edi
005E35E1     5E              pop esi
005E35E2     59              pop ecx
005E35E3     - E9 FDCEE4FF   jmp 004304E5
```

经过如此操作后，程序读到的已是正确的注册硬盘的信息。下面分析程序的 `CRC` 部分。

程序修改后，出现错误提示：

```
:0045C3C1 E8BAAAFFFF          call 00456E80
:0045C3C6 85C0               test eax, eax
:0045C3C8 750A               jne 0045C3D4

* Possible StringData Ref from Data Obj ->"系统执行文件被非法修改,请检查病毒并重新安装!"
                           |
:0045C3CA 68E8B45700          push 0057B4E8
:0045C3CF E9BF090000          jmp 0045CD93
```

追入 `456E80`：

```
:00456E80 55                  push ebp
:00456E81 8BEC                mov ebp, esp

* Possible Reference to Dialog: DialogID_0067, CONTROL_ID:00FF, ""
                           |
:00456E83 6AFF                push FFFFFFFF
:00456E85 6848A65300          push 0053A648
:00456E8A 64A100000000        mov eax, dword ptr fs:[00000000]
:00456E90 50                  push eax
:00456E91 64892500000000      mov dword ptr fs:[00000000], esp
:00456E98 81ECC40F0000        sub esp, 00000FC4
:00456E9E 53                  push ebx
:00456E9F 56                  push esi
:00456EA0 57                  push edi
:00456EA1 8BF1                mov esi, ecx
:00456EA3 8965F0              mov dword ptr [ebp-10], esp
:00456EA6 8975EC              mov dword ptr [ebp-14], esi
:00456EA9 EB01                jmp 00456EAC
:00456EAB 89                  BYTE 89

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00456EA9(U)
|
:00456EAC 8D4DDC              lea ecx, dword ptr [ebp-24]
:00456EAF E84D220C00          call 00519101
:00456EB4 8B0D00AB5800        mov ecx, dword ptr [0058AB00]
:00456EBA A104AB5800          mov eax, dword ptr [0058AB04]
:00456EBF 894DE4              mov dword ptr [ebp-1C], ecx
:00456EC2 33DB                xor ebx, ebx
:00456EC4 03C8                add ecx, eax
:00456EC6 895DFC              mov dword ptr [ebp-04], ebx
:00456EC9 85C9                test ecx, ecx
:00456ECB C645FC01            mov [ebp-04], 01
:00456ECF 8945E8              mov dword ptr [ebp-18], eax
:00456ED2 0F84BE000000        je 00456F96
:00456ED8 8B0D08AB5800        mov ecx, dword ptr [0058AB08]       ;取回预置的 CRC 校验码到 ECX
:00456EDE A10CAB5800          mov eax, dword ptr [0058AB0C]       ;取回预置的 CRC 校验码到 EAX
:00456EE3 8B9604090000        mov edx, dword ptr [esi+00000904]
:00456EE9 894DD4              mov dword ptr [ebp-2C], ecx
:00456EEC 8D8E04090000        lea ecx, dword ptr [esi+00000904]
```



```
:00456EF2 53                    push ebx
:00456EF3 53                    push ebx
:00456EF4 8945D8                mov dword ptr [ebp-28], eax
:00456EF7 FF5228                call [edx+28]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F86(U)
```

```
:00456EFA 83FB01                cmp ebx, 00000001
:00456EFD 0F8FB2000000          jg 00456FB5
:00456F03 33FF                  xor edi, edi
:00456F05 EB03                  jmp 00456F0A
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F58(C)
```

```
:00456F07 8B75EC                mov esi, dword ptr [ebp-14]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F05(U)
```

```
:00456F0A 8B449DE4              mov eax, dword ptr [ebp+4*ebx-1C]
:00456F0E 3DA00F0000            cmp eax, 00000FA0
:00456F13 7205                  jb 00456F1A
:00456F15 B8A00F0000            mov eax, 00000FA0
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F13(C)
```

```
:00456F1A 8B9604090000          mov edx, dword ptr [esi+00000904]
:00456F20 8D8E04090000          lea ecx, dword ptr [esi+00000904]
:00456F26 50                    push eax
:00456F27 8D8530F0FFFF          lea eax, dword ptr [ebp+FFFFF030]
:00456F2D 50                    push eax
:00456F2E FF5234                call [edx+34]
:00456F31 8BF0                  mov esi, eax
:00456F33 85F6                  test esi, esi
:00456F35 7E23                  jle 00456F5A
:00456F37 57                    push edi
:00456F38 8D8D30F0FFFF          lea ecx, dword ptr [ebp+FFFFF030]
:00456F3E 56                    push esi
:00456F3F 51                    push ecx
:00456F40 E8EB270800            call 004D9730
:00456F45 8B4C9DE4              mov ecx, dword ptr [ebp+4*ebx-1C]
:00456F49 8BF8                  mov edi, eax
:00456F4B 2BCE                  sub ecx, esi
:00456F4D 83C40C                add esp, 0000000C
:00456F50 8BC1                  mov eax, ecx
:00456F52 894C9DE4              mov dword ptr [ebp+4*ebx-1C], ecx
:00456F56 85C0                  test eax, eax
:00456F58 7FAD                  jg 00456F07
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F35(C)
```

```
:00456F5A 85DB                  test ebx, ebx
:00456F5C 7516                  jne 00456F74
:00456F5E 8B55EC                mov edx, dword ptr [ebp-14]
:00456F61 6A01                  push 00000001
```

\* Possible Reference to Dialog: `DialogID_0067`, `CONTROL_ID:0008`, "??e 湍(&E)"

```
:00456F63 6A08                  push 00000008
:00456F65 8B8204090000          mov eax, dword ptr [edx+00000904]
:00456F6B 8D8A04090000          lea ecx, dword ptr [edx+00000904]
:00456F71 FF5028                call [eax+28]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00456F5C(C)
```

```
:00456F74 3B7C9DD4              cmp edi, dword ptr [ebp+4*ebx-2C]    ;比较校验码
:00456F78 7511                  jne 00456F8B
:00456F7A 8B449DE4              mov eax, dword ptr [ebp+4*ebx-1C]
:00456F7E 85C0                  test eax, eax
:00456F80 7509                  jne 00456F8B
:00456F82 8B75EC                mov esi, dword ptr [ebp-14]
:00456F85 43                    inc ebx
:00456F86 E96FFFFFFF            jmp 00456EFA
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:00456F78(C), :00456F80(C)
```

```
:00456F8B E800000000            call 00456F90
```

\* Referenced by a CALL at Address:

```
:00456F8B
```

```
:00456F90 83042406              add dword ptr [esp], 00000006
:00456F94 C3                    ret
```

在 `456F74` 中记录下两次程序计算的校验码和程序源校验码，修改源程序中预置校验码。至此破解完成。

`4dd466d0e1e0cc123afbe504da2462ca`
