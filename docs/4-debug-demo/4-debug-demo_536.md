
# HyperSnap-DX ver.4.20.00

> 原文：[`www.pediy.com/kssd/pediy04/kanxue250.htm`](https://www.pediy.com/kssd/pediy04/kanxue250.htm)

去除 `HyperSnap-DX ver.4.20.00` `CRC` 校验

一个具有很强 `CRC` 校验的软件，将 `hprsnap.exe`、`hprsnap.xxx`、`hprres.dll`、`hprres.xxx` 四个文件更名或修改（汉化）都不能运行程序。如果随便修改程序跳转将很容易导致“程序执行非法操作”。

1\. 复制 `HprSnap.exe` 更名为 `test.exe`，复制 `HprRes.xxx` 更名为 `test.xxx`。用 `smartcheck` 载入后运行，执行完毕后终止进程。

选择 `smartcheck` 中的“显示所有事件”按钮，了解一下作了些什么?

`hprsnap.exe` 中

```
createfilea 路径\hprsnap.exe (hprsnap.exe offset 00076E9D)
createfilea 路径\hprsnap.xxx (hprsnap.exe offset 00076E9D)
createfilea 路径\hprres.dll  (hprsnap.exe offset 00076E9D)
createfilea 路径\hprres.xxx  (hprsnap.exe offset 00076E9D)
```

`test.exe` 中

```
createfilea 路径\test.exe (test.exe offset 00076E9D)
createfilea 路径\test.xxx (test.exe offset 00076E9D)
```

随后不久就是 `nag`。

2\. `wdasm` 反编译 `test.exe`

```
* Reference To: KERNEL32.CreateFileA, Ord:0037h

                |

:00476E9C     Call dword ptr [00524358]

//smartcheck 计算 offset 与 wdasm 有少许偏差

:00476EA2     mov esi, eax
:00476EA4     cmp esi, edi

//文件不存在就 end，所以要同时伪造 test.xxx

:00476EA6     jne 00476EBC

F10 一直走，直到开始标准 CRC32 校验

               |

:0045FD2D     mov al, byte ptr [ebp+edx-00000430]

//自第一个字节开始，顺次取 test.exe 每个字节，每个字段长度为 0x400

:0045FD34     mov ebx, ecx
:0045FD36     and ebx, 000000FF
:0045FD3C     xor eax, ebx
:0045FD3E     shr ecx, 08
:0045FD41     mov eax, dword ptr [4*eax+0056E6F0]

//自 0056E6F0，length=0x400，为 CRC32 密码表

:0045FD48     xor eax, ecx
:0045FD4A     inc edx
:0045FD4B     cmp edx, edi
:0045FD4D     mov dword ptr [esi], eax
:0045FD4F     jl 0045FD29
:0045FD51     mov ebx, dword ptr [ebp+0C]
:0045FD54     mov eax, dword ptr [ebp-18]
:0045FD57     add esi, 00000004
:0045FD5A     dec eax
:0045FD5B     mov dword ptr [ebp-18], eax
:0045FD5E     jne 0045FCF9
:0045FD34     mov ebx, ecx
:0045FD36     and ebx, 000000FF
:0045FD3C     xor eax, ebx
:0045FD3E     shr ecx, 08
:0045FD41     mov eax, dword ptr [4*eax+0056E6F0]
:0045FD48     xor eax, ecx
:0045FD4A     inc edx
:0045FD4B     cmp edx, edi
:0045FD4D     mov dword ptr [esi], eax

//每个字段的 CRC 值依次保存在 dword ptr [esi]，长度为文件大小除以 0x400

:0045FD4F     jl 0045FD29
:0045FD51     mov ebx, dword ptr [ebp+0C]
:0045FD54     mov eax, dword ptr [ebp-18]
:0045FD57     add esi, 00000004
:0045FD5A     dec eax
:0045FD5B     mov dword ptr [ebp-18], eax
:0045FD5E     jne 0045FCF9
```

再来，看看以后对 `CRC` 校验表作什么

`bpm dword ptr [esi]` （参照 `:0045FD4D mov dword ptr [esi], eax` 处）

中断于 `:004537B0 mov ecx, ebx` 处，

由此开始进行第二次计算，也不知道是如何作的，`hash`？

3. 从 `:004537B0 mov ecx, ebx` 那里用 `F12` 出来，随后就是 `F10` 一直走，比较 `hprsnap.exe` 与 `test.exe` 运行的不同，关键在

```
:0045FF04     call 004C71A7
:0045FF09     add esp, 0000000C

//hprsnap.exe: EAX==0
//test.exe:    EAX==FFFFFF68
```

以 `EAX` 值为旗帜，层层深入

```
:004C7301       call 004C6F82  <== F8

  :004C7088       call [eax+54]  <== F8

    :00445B76       call 004463D8
    :00445B7B       pop ecx
    :00445B7C       test eax, eax  <-- EAX=0, CRC 正确
    :00445B7E       pop ecx
    :00445B7F       je 00445B91    <-- je==> jmp, job done.
    :00445B81       xor eax, eax
    :00445B83       cmp dword ptr [ebp-04], eax
    :00445B86       jne 00445B93
    :00445B88       mov [ebp-04], FFFFFF66

    <--否则最后赋值 EAX=FFFFFF66，这就是随便改跳转导致“程序非法操作”的原因

    如果愿意进入:00445B76   call 004463D8 看看，最后就是

    :00446435       mov edx, dword ptr [esi+ecx]
    :00446438       mov ebx, dword ptr [ecx]
    :0044643A       cmp edx, ebx

    <--一切归根到比较两组长度为 0x14 的 hash 值。
```

4. 总结

```
改:00445B7F   7410    je   00445B91

为           EB10    jmp 00445B91
```

`end.`

`FROM` `CCG` `4dd466d0e1e0cc123afbe504da2462ca`
