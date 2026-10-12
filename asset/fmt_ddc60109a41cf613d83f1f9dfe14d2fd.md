
# 超级点击机器 II V0.63

> 原文：[`www.pediy.com/kssd/pediy02/forum264.htm`](https://www.pediy.com/kssd/pediy02/forum264.htm)

超级点击机器 II V0.63 破解实录

下载地址：`http://zjfeng.yeah.net`

软件简介: 帮大家多赚点广告费的东东。

作 者：`liangs`

解密工具：`Trw2000 1.22` 、`W32Dasm Ver8.93 超级中文版`

最近，从白菜乐园下载了一个超级点击机器 II 的破解文件，发现它是暴力修改程序，其实，完全不用这样，其注册码的破解也很简单。

首先，用 `W32Dasm Ver8.93 超级中文版` 将 `MoreCounter.exe` 主程序反汇编，然后在 `W32Dasm` 的数据串列表中可以发现 "谢谢你的注册,请重新启动超级点击机器 II" 的字样。该程序输入注册码后，若不对则什么也不提示，以上的提示无疑是我们破解的出发点。

```assembly
:0040513C E85F040000                call 004055A0 <---输入注册码的判断
:00405141 84C0                      test al, al <---注册码错误标志为 0
:00405143 7448                      je 0040518D <---标志为 0，跳出注册程序
:00405145 6A00                      push 00000000
:00405147 6A00                      push 00000000

* Possible StringData Ref from Data Obj -> "谢谢你的注册,请重新启动超级点击机器 II"
                                        |
:00405149 68648C4200                push 00428C64 <---我们停在这儿
:0040514E E87F620100                call 0041B3D2
:00405153 E8E8870100                call 0041D940
:00405158 8B8C24D0010000            mov ecx, dword ptr [esp+000001D0]
:0040515F 8B7004                    mov esi, dword ptr [eax+04]
:00405162 51                       push ecx
```

从 `00405149` 往上看，很明显 `0040513C` 处是注册码的比较判断点，F8 跟入。

-------------------------------------------------

由上面 `call 004055A0` 跟入：

* Referenced by a CALL at Addresses:
| `:004044B0` , `:0040513C`

上面两个 Call 的地址，一个是程序启动用来判断是否注册，另一个当然就是目前注册码判断的 Call，大家都公用的一个子程序。因此，要打补丁的话，也要打在这个子程序里面。

```assembly
:004055A0 6AFF                      push FFFFFFFF
:004055A2 6880F54100                push 0041F580
:004055A7 64A100000000              mov eax, dword ptr fs:[00000000]
:004055AD 50                       push eax
:004055AE 64892500000000            mov dword ptr fs:[00000000], esp
:004055B5 83EC08                    sub esp, 00000008
:004055B8 56                       push esi
:004055B9 68D8A44200                push 0042A4D8
:004055BE 8D4C2408                 lea ecx, dword ptr [esp+08]
:004055C2 C744241801000000         mov [esp+18], 00000001
:004055CA E8D3340100                call 00418AA2
:004055CF 8B44241C                 mov eax, dword ptr [esp+1C]
:004055D3 33F6                      xor esi, esi
:004055D5 C644241402                mov [esp+14], 02
:004055DA 8B48F8                    mov ecx, dword ptr [eax-08]
:004055DD 85C9                      test ecx, ecx
:004055DF 7E28                      jle 00405609

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405607(C)
|

:004055E1 83FE0A                    cmp esi, 0000000A
:004055E4 7D23                      jge 00405609
:004055E6 0FBE0406                  movsx eax, byte ptr [esi+eax]
:004055EA 99                        cdq
:004055EB B90A000000                mov ecx, 0000000A
:004055F0 F7F9                      idiv ecx
:004055F2 8D4C2404                 lea ecx, dword ptr [esp+04]
:004055F6 80C230                   add dl, 30
:004055F9 52                       push edx
:004055FA E886370100                call 00418D85
:004055FF 8B44241C                 mov eax, dword ptr [esp+1C]
:00405603 46                       inc esi
:00405604 3B70F8                   cmp esi, dword ptr [eax-08]
:00405607 7CD8                     jl 004055E1

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004055DF(C), :004055E4(C)

* Possible StringData Ref from Data Obj -> "zjfeng.yeah.net"
                                        |
:00405609 68CC8C4200                push 00428CCC
:0040560E 8D4C2408                 lea ecx, dword ptr [esp+08]
:00405612 E847370100                call 00418D5E
:00405617 8D542408                 lea edx, dword ptr [esp+08]
:0040561B 6A0A                     push 0000000A
:0040561D 52                       push edx
:0040561E 8D4C240C                 lea ecx, dword ptr [esp+0C]
:00405622 E8EBE00000                call 00413712
:00405627 50                       push eax
:00405628 8D4C2408                 lea ecx, dword ptr [esp+08]
:0040562C C644241803                mov [esp+18], 03
:00405631 E8EB340100                call 00418B21
:00405636 8D4C2408                 lea ecx, dword ptr [esp+08]
:0040563A C644241402                mov [esp+14], 02
:0040563F E8F0330100                call 00418A34
:00405644 8B742420                 mov esi, dword ptr [esp+20]
                                                ^^^ <--根据输入的 ID 算出的注册号；
:00405648 8B442404                 mov eax, dword ptr [esp+04]
                                                ^^^ <--输入的假注册号；

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040566A(C)
|

:0040564C 8A10                      mov dl, byte ptr [eax]
                                                取[EAX]的第一位数
:0040564E 8ACA                      mov cl, dl
:00405650 3A16                      cmp dl, byte ptr [esi]
                                                取[ESI]的第一位数
:00405652 751C                      jne 00405670
                                                不等就跳走
:00405654 84C9                      test cl, cl
                                                是否已是最后一位数
:00405656 7414                      je 0040566C
                                                是就跳走
:00405658 8A5001                   mov dl, byte ptr [eax+01]
                                                取[EAX]的下一位数
:0040565B 8ACA                      mov cl, dl
:0040565D 3A5601                   cmp dl, byte ptr [esi+01]
                                                取[ESI]的下一位数
:00405660 750E                     jne 00405670
                                                不等就跳走
:00405662 83C002                   add eax, 00000002
                                                调整指针指向 EAX 再下一位
:00405665 83C602                   add esi, 00000002
                                                调整指针指向 ESI 再下一位
:00405668 84C9                     test cl, cl
                                                是否已是最后一位
:0040566A 75E0                     jne 0040564C
                                                不是就跳到 0040564C 循环取数比较

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405656(C)
|

:0040566C 33C0                      xor eax, eax
:0040566E EB05                      jmp 00405675

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00405652(C), :00405660(C)
|

:00405670 1BC0                      sbb eax, eax
:00405672 83D8FF                   sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040566E(U)
|

:00405675 85C0                      test eax, eax
:00405677 5E                       pop esi
:00405678 C644241001                mov [esp+10], 01
:0040567D 8D4C2400                 lea ecx, dword ptr [esp]
:00405681 7537                     jne 004056BA
:00405683 E8AC330100                call 00418A34
:00405688 8D4C2418                 lea ecx, dword ptr [esp+18]
:0040568C C644241000                mov [esp+10], 00
:00405691 E89E330100                call 00418A34
:00405696 8D4C241C                 lea ecx, dword ptr [esp+1C]
:0040569A C7442410FFFFFFFF          mov [esp+10], FFFFFFFF
:004056A2 E88D330100                call 00418A34
:004056A7 B001                     mov al, 01 <---注册成功标志
:004056A9 8B4C2408                 mov ecx, dword ptr [esp+08]
:004056AD 64890D00000000            mov dword ptr fs:[00000000], ecx
:004056B4 83C414                   add esp, 00000014
:004056B7 C20800                   ret 0008
```
