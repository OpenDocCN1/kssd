
# Download Boost 2002 Go 2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6154.htm`](https://www.pediy.com/kssd/pediy06/pediy6154.htm)

Download Boost 2002 Go 2.0 汉化版算法破解手记

验证方式：序列号验证，必须注册，否则无法运行。

注册名：`李逍遥\cschina\`

注册码：`2B5BA-U77A-AX76-5Z3K`

假码：`87654321`

下 `hmemcpy`，`bd *`，`n` 次来到下面，也可以反汇编，很容易找到关键，（因为是汉化的，所以我猜壳给汉化作者脱了，感谢他们！！）

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AAD4B(U)

|

:004AAC93 8B45FC                 mov eax, dword ptr [ebp-04]
:004AAC96 E8B1FEFFFF             call 004AAB4C       <===关键 call，F8 跟进。
:004AAC9B 833D10524B0001         cmp dword ptr [004B5210], 00000001
:004AACA2 1BC0                   sbb eax, eax
:004AACA4 40                     inc eax
:004AACA5 3C01                   cmp al, 01          <===al 和 1 比较，
:004AACA7 757C                   jne 004AAD25        <===不为 0 跳到出错处，跳则 Over。爆破就是这里。哈哈！！

* Possible StringData Ref from Code Obj ->"Download Boost 2002 已经成功注册!!!"

                                |

:004AACA9 B8ACAD4A00             mov eax, 004AADAC
:004AACAE E845EEF8FF             call 00439AF8
:004AACB3 B201                   mov dl, 01          <===dl 置 1 则 OK。
:004AACB5 A1BC534600             mov eax, dword ptr [004653BC]
:004AACBA E8FDA7FBFF             call 004654BC
:004AACBF 8BD8                   mov ebx, eax
:004AACC1 BA02000080             mov edx, 80000002
:004AACC6 8BC3                   mov eax, ebx
:004AACC8 E88FA8FBFF             call 0046555C
:004AACCD B101                   mov cl, 01

* Possible StringData Ref from Code Obj ->"\\Software\\Magellass\\DownloadBoost"       <====注册成功后写入注册表中的地方

                                |

:004AACCF BAECAD4A00             mov edx, 004AADEC
:004AACD4 8BC3                   mov eax, ebx
:004AACD6 E8C5A9FBFF             call 004656A0
:004AACDB 8B0D14524B00           mov ecx, dword ptr [004B5214]

* Possible StringData Ref from Code Obj ->"RegName"

                                |

:004AACE1 BA18AE4A00             mov edx, 004AAE18
:004AACE6 8BC3                   mov eax, ebx
:004AACE8 E817AEFBFF             call 00465B04
:004AACED C745F836E7CD00          mov [ebp-08], 00CDE736
:004AACF4 6A04                   push 00000004
:004AACF6 8D4DF8                 lea ecx, dword ptr [ebp-08]

* Possible StringData Ref from Code Obj ->"Registered"

                                |

:004AACF9 BA28AE4A00             mov edx, 004AAE28
:004AACFE 8BC3                   mov eax, ebx
:004AAD00 E82FAFFBFF             call 00465C34
:004AAD05 8BC3                   mov eax, ebx
:004AAD07 E820A8FBFF             call 0046552C
:004AAD0C 8BC3                   mov eax, ebx
:004AAD0E E8418DF5FF             call 00403A54
:004AAD13 8B45FC                 mov eax, dword ptr [ebp-04]
:004AAD16 E84926FBFF             call 0045D364
:004AAD1B A1E4334B00             mov eax, dword ptr [004B33E4]
:004AAD20 C60001                 mov byte ptr [eax], 01
:004AAD23 EB20                   jmp 004AAD45

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004AACA7(C)

|

:004AAD25 8B45FC                 mov eax, dword ptr [ebp-04]       <===这里出错！！
:004AAD28 8B8010030000            mov eax, dword ptr [eax+00000310]
:004AAD2E 33D2                    xor edx, edx
:004AAD30 E8FB5CF9FF              call 00440A30
```

`\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

跟进 `004AAC96 E8B1FEFFFF call 004AAB4C` 这个 `call`：

```
0187:004AAB6A 8B8310030000      MOV       EAX,[EBX+0310]
0187:004AAB70 E88B5EF9FF        CALL      00440A00
0187:004AAB75 8B55FC            MOV       EDX,[EBP-04]    <===用户名送 edx
0187:004AAB78 B814524B00        MOV       EAX,004B5214
0187:004AAB7D E8169DF5FF        CALL      00404898
0187:004AAB82 8D55F8            LEA       EDX,[EBP-08]
0187:004AAB85 8B8314030000      MOV       EAX,[EBX+0314]
0187:004AAB8B E8705EF9FF        CALL      00440A00
0187:004AAB90 8B55F8            MOV       EDX,[EBP-08]    <===假码送 edx
0187:004AAB93 B818524B00        MOV       EAX,004B5218
0187:004AAB98 E8FB9CF5FF        CALL      00404898
0187:004AAB9D 33DB              XOR       EBX,EBX         <===计数器 ebx 清 0。
0187:004AAB9F 8D4DF4            LEA       ECX,[EBP-0C]
0187:004AABA2 0FBFD3            MOVSX     EDX,BX
0187:004AABA5 A184354B00        MOV       EAX,[004B3584]
0187:004AABAA 8B00              MOV       EAX,[EAX]
0187:004AABAC 8B80D0030000      MOV       EAX,[EAX+03D0]
0187:004AABB2 8B4030            MOV       EAX,[EAX+30]
0187:004AABB5 8B30              MOV       ESI,[EAX]
0187:004AABB7 FF560C            CALL      NEAR [ESI+0C]    <===算法 call，跟进！
0187:004AABBA 8B55F4            MOV       EDX,[EBP-0C]     <===真码送 edx
0187:004AABBD A118524B00        MOV       EAX,[004B5218]   <===假码送 eax
0187:004AABC2 E879A0F5FF        CALL      00404C40         <===关键比较
0187:004AABC7 750A              JNZ       004AABD3         <===不相等跳。
0187:004AABC9 C70510524B00FFFF+MOV        DWORD [004B5210],FFFFFFFF
0187:004AABD3 43                INC       EBX              <===计数器 ebx 加 1
0187:004AABD4 6681FBF401       CMP       BX,01F4          <===共有 500 个序列号
0187:004AABD9 75C4              JNZ       004AAB9F         <===500 没取完，返回循环继续比较。
0187:004AABDB 33C0              XOR       EAX,EAX
0187:004AABDD 5A                POP       EDX
0187:004AABDE 59                POP       ECX
0187:004AABDF 59                POP       ECX
0187:004AABE0 648910            MOV       [FS:EAX],EDX
0187:004AABE3 6805AC4A00        PUSH      DWORD 004AAC05
0187:004AABE8 8D45F4            LEA       EAX,[EBP-0C]
0187:004AABEB E8549CF5FF        CALL      00404844
0187:004AABF0 8D45F8            LEA       EAX,[EBP-08]
0187:004AABF3 BA02000000        MOV       EDX,02
0187:004AABF8 E86B9CF5FF        CALL      00404868
0187:004AABFD C3                RET
```

`\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*`

跟进 `4AABB7 FF560C CALL NEAR [ESI+0C]` 看看里面什么东西？

```
0187:00419535 8BF2              MOV       ESI,EDX       <===esi 赋初值 0
0187:00419537 8BD8              MOV       EBX,EAX
0187:00419539 85F6              TEST      ESI,ESI
0187:0041953B 7C05              JL        00419542
0187:0041953D 3B7314            CMP       ESI,[EBX+14]  <===和序列号表的值 1F4 比较，
0187:00419540 7C0F              JL        00419551       <===小于则跳
0187:00419542 8B15EC364B00      MOV       EDX,[004B36EC]
0187:00419548 8BCE              MOV       ECX,ESI
0187:0041954A 8BC3              MOV       EAX,EBX
0187:0041954C E863F2FFFF        CALL      004187B4
0187:00419551 8BC7              MOV       EAX,EDI
0187:00419553 8B5310            MOV       EDX,[EBX+10]
0187:00419556 8B14F2            MOV       EDX,[EDX+ESI*8]   <===序列号表送 edx
0187:00419559 E83AB3FEFF        CALL      00404898
0187:0041955E 5F                POP       EDI
0187:0041955F 5E                POP       ESI
0187:00419560 5B                POP       EBX
0187:00419561 C3                RET
```

删除注册表键值 `[HKEY_LOCAL_MACHINE\\Software\\Magellass\\DownloadBoost]` 即为未注册。

算法总结：取软件中固定序列号比较。

`李逍遥[cschina]`
`2003.04.19`

`4dd466d0e1e0cc123afbe504da2462ca`
