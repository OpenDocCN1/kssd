
```asm
0072007C  ^ 70 A7             jo short un\_Photo.00720025
0072007E    E3 00             jecxz short un\_Photo.00720080
```

修改为：

```asm
00720078    0000              add byte ptr ds:[eax],al
0072007A    0000              add byte ptr ds:[eax],al
0072007C    0000              add byte ptr ds:[eax],al
0072007E    0000              add byte ptr ds:[eax],al
```

---

## 2.解决问题: 写代码进行破解!

### 下面是破解过程(花了小Q两天的时间,与你分享一下):

直接运行程序进行假码分析破解! `bp MessageBoxA` 断下返回如下：

```asm
004C1D12   > \FFB6 C8010000 push    dword ptr [esi+1C8]
004C1D18   .  8B86 A0010000 mov     eax, [esi+1A0]
004C1D1E   .  8B0F          mov     ecx, [edi]
004C1D20   .  8B13          mov     edx, [ebx]
004C1D22   .  50            push    eax
004C1D23   .  51            push    ecx
004C1D24   .  52            push    edx
004C1D25   .  FF96 B4010000 call    [esi+1B4]---------------->这里重点(等一下要重载进入) F2下断
004C1D2B   .  83C4 10       add     esp, 10
004C1D2E   .  84C0          test    al, al
004C1D30   .^ 0F84 52FFFFFF je      004C1C88----------------->不跳就出现注册成功(不过没用的)
004C1D36   .  68 884B6300   push    00634B88                         ;  ASCII "Message"
004C1D3B   .  8D4D E4       lea     ecx, [ebp-1C]
004C1D3E   .  E8 ADF5F3FF   call    004012F0
004C1D43   .  68 489A6200   push    00629A48                         ;  ASCII "Unlocking successful!"
004C1D48   .  C745 FC 07000>mov     dword ptr [ebp-4], 7
004C1D4F   .  E8 93780100   call    004D95E7
004C1D54   .  59            pop     ecx
004C1D55   .  50            push    eax
004C1D56   .  8D4D E0       lea     ecx, [ebp-20]
```

```asm
004C1E60   .  E8 ABF4F4FF   call    00411310
004C1E65   .  834D FC FF    or      dword ptr [ebp-4], FFFFFFFF >>>>返回这里,看上面
004C1E69   .  8D4D EC       lea     ecx, [ebp-14]
004C1E6C   .  E8 DFF4F3FF   call    00401350
004C1E71   .  32C0          xor     al, al
```

---

进入`call [esi+1B4]`向下看：

```asm
004023A0   .  8A4C24 10     mov     cl, [esp+10]
004023A4   .  33C0          xor     eax, eax
004023A6   .  F6C1 07       test    cl, 7
004023A9   .  75 07         jnz     short 004023B2
004023AB   .  B8 04000000   mov     eax, 4
004023B0   .  EB 0A         jmp     short 004023BC
004023B2   >  F6C1 01       test    cl, 1
004023B5   .  75 05         jnz     short 004023BC
004023B7   .  B8 02000000   mov     eax, 2
004023BC   >  8B4C24 0C     mov     ecx, [esp+C]
004023C0   .  8B5424 08     mov     edx, [esp+8]
004023C4   .  51            push    ecx                              ; /Arg5
004023C5   .  8B4C24 08     mov     ecx, [esp+8]                     ; |
004023C9   .  52            push    edx                              ; |Arg4-->邮件地址
004023CA   .  51            push    ecx                              ; |Arg3-->用户名
004023CB   .  BA 06000000   mov     edx, 6                           ; |
004023D0   .  2BD0          sub     edx, eax                         ; |
004023D2   .  52            push    edx                              ; |Arg2 假码
004023D3   .  68 04507000   push    00705004                         ; |Arg1 = 00705004
004023D8   .  E8 1CCD0B00   call    004BF0F9  ------------>重点进行比较解码! F7 进
004023DD   .  83C4 14       add     esp, 14
004023E0   .  85C0          test    eax, eax
004023E2   .  A3 90FF7100   mov     [71FF90], eax
004023E7   .  0F9DC0        setge   al
004023EA   .  C3            retn
```

---

```asm
004BF0F9  /$  55            push    ebp
004BF0FA  |.  8BEC          mov     ebp, esp
004BF0FC  |.  56            push    esi
004BF0FD  |.  33F6          xor     esi, esi
004BF0FF  |.  3975 0C       cmp     [ebp+C], esi
004BF102  |.  7E 21         jle     short 004BF125
004BF104  |>  FF75 18       /push    dword ptr [ebp+18]
004BF107  |.  8B45 08       |mov     eax, [ebp+8]
004BF10A  |.  FF75 14       |push    dword ptr [ebp+14]
004BF10D  |.  FF75 10       |push    dword ptr [ebp+10]
004BF110  |.  FF34B0        |push    dword ptr [eax+esi\*4]
004BF113  |.  E8 34FDFFFF   |call    004BEE4C-------------->比较,算法全在这里进行!(小Q跟得大多了,不进)
004BF118  |.  83C4 10       |add     esp, 10
004BF11B  |.  84C0          |test    al, al
004BF11D  |.  75 0C         |jnz     short 004BF12B--->呵呵//很重要,注册比较成功,第一个将会跳!
004BF11F  |.  46            |inc     esi              ;要不然就再比较其它操作系统的注册码! 这里 JMP 保存
004BF120  |.  3B75 0C       |cmp     esi, [ebp+C]
004BF123  |.^ 7C DF         \jl      short 004BF104
004BF125  |>  83C8 FF       or      eax, FFFFFFFF
004BF128  |>  5E            pop     esi
004BF129  |.  5D            pop     ebp
004BF12A  |.  C3            retn
004BF12B  |>  8BC6          mov     eax, esi
004BF12D  \.^ EB F9         jmp     short 004BF128
```

---

保存后进行注册!说是成功了后软件被关闭了//哈哈//再运行//软件被无情关闭,软件作者毒啊! 以下代码小Q跟了一天!

```asm
004059AB   .  E8 B6990B00   call    004BF366---------------> 这里检验是否进行注册过
004059B0   .  83C4 10       add     esp, 10
004059B3   .  84C0          test    al, al
004059B5   .  0F84 1C010000 je      00405AD7--------->没注册就跳(为什么我没注册修改上面004BF11D,不会关闭,就是这里跳了,呵呵)
004059BB   .  8B4C24 44     mov     ecx, [esp+44]
004059BF   .  8B5424 4C     mov     edx, [esp+4C]
004059C3   .  8B4424 54     mov     eax, [esp+54]
004059C7   .  51            push    ecx                              ; /Arg5
004059C8   .  52            push    edx                              ; |Arg4
004059C9   .  50            push    eax                              ; |Arg3
004059CA   .  6A 06         push    6                                ; |Arg2 = 00000006
004059CC   .  68 04507000   push    00705004                         ; |Arg1 = 00705004
004059D1   .  E8 23970B00   call    004BF0F9 --------->这里检验注册码是正确(也是检验老注册或被放水的注册码)
004059D6   .  83C4 14       add     esp, 14
004059D9   .  85C0          test    eax, eax
004059DB   .  A3 00507000   mov     [705000], eax
004059E0   .  7C 3C         jl      short 00405A1E---->放水的,假的注册码,都会跳去告诉你此注册非法
004059E2   .  8B4C24 54     mov     ecx, [esp+54]
004059E6   .  51            push    ecx
004059E7   .  8D8D AC000000 lea     ecx, [ebp+AC]
004059ED   .  E8 3EE20200   call    00433C30
004059F2   .  8B5424 4C     mov     edx, [esp+4C]
004059F6   .  52            push    edx
004059F7   .  8D8D B0000000 lea     ecx, [ebp+B0]
004059FD   .  E8 2EE20200   call    00433C30
00405A02   .  8B4424 44     mov     eax, [esp+44]
00405A06   .  50            push    eax
00405A07   .  8D8D B4000000 lea     ecx, [ebp+B4]
00405A0D   .  E8 1EE20200   call    00433C30
00405A12   .  C685 A9000000>mov     byte ptr [ebp+A9], 1
00405A19   .  E9 B9000000   jmp     00405AD7
00405A1E   .  A1 4CE27000   mov     eax, [70E24C]
00405A23   .  85C0          test    eax, eax
00405A25   .  75 05         jnz     short 00405A2C
00405A27   .  B8 55F55E00   mov     eax, 005EF555
```
