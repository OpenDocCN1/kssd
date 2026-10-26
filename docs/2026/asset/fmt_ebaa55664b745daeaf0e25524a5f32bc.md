
```text
00FCB23B    8B15 B4B6FF00   mov     edx, dword ptr [FFB6B4]          ; kernel32.7C8246E6
00FCB241    83C2 64         add     edx, 64                          ; 注意这个加0x64
00FCB244    B9 05000000     mov     ecx, 5
00FCB249    803A CC         cmp     byte ptr [edx], 0CC
00FCB24C    74 07           je      short 00FCB255
00FCB24E  ^ E2 F9           loopd   short 00FCB249
00FCB250    FF75 08         push    dword ptr [ebp+8]
00FCB253    FFD2            call    near edx                         ; kernel32.GetModuleHandleA，这是完成实际功能的Win32 API
00FCB255    8945 FC         mov     dword ptr [ebp-4], eax
00FCB258    61              popad
00FCB259    8B45 FC         mov     eax, dword ptr [ebp-4]
00FCB25C    5F              pop     edi
00FCB25D    5E              pop     esi
00FCB25E    5B              pop     ebx
00FCB25F    C9              leave
00FCB260    C2 0400         retn    4
```

--------------------------------------------------------------------------

关于针对IAT中部分Entry进行特殊处理，参看:

Armadillo标准壳完全扫盲 - [2007-05-17]

929K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3U0g2o6b7#2)9J5y4f1g2o6i4K6t1#2b7@1c8Q4x3U0g2q4x3W2)9J5y4f1x3K6i4K6t1#2b7f1u0Q4x3U0g2n7x3#2)9J5y4f1f1$3i4K6u0r3j5X3I4G2k6#2)9J5c8X3W2@1k6h3#2Q4x3V1j5&6x3e0x3I4x3$3c8X3y4h3p5H3y4o6R3K6k6e0t1@1j5X3x3K6x3e0l9&6j5U0c8Q4x3X3g2Z5N6r3#2D9

(解释了一些原理，适合新手，但有些术语并不正确，自己修正着理解吧)

重新用`OD`调试`origspider_trial.exe`，欺骗`OpenMutexA()`之后清除所有断点。

```text
hw 008C912C
F9
dd 008C9078
```

--------------------------------------------------------------------------

```text
008C9078  004C9192  origspid.004C9192
008C907C  004C917A  origspid.004C917A
008C9080  004C9168  origspid.004C9168
008C9084  004C9156  origspid.004C9156
008C9088  004C9148  origspid.004C9148
008C908C  004C9130  origspid.004C9130
008C9090  00000000
008C9094  004C9192  origspid.004C9192
008C9098  004C917A  origspid.004C917A
008C909C  004C9168  origspid.004C9168
008C90A0  004C9156  origspid.004C9156
008C90A4  004C9148  origspid.004C9148
008C90A8  004C9130  origspid.004C9130
008C90AC  00000000
008C90B0  004C926E  origspid.004C926E
008C90B4  004C925E  origspid.004C925E
008C90B8  004C9252  origspid.004C9252
008C90BC  004C9246  origspid.004C9246
008C90C0  004C9236  origspid.004C9236
008C90C4  004C9224  origspid.004C9224
008C90C8  004C9212  origspid.004C9212
008C90CC  004C91FE  origspid.004C91FE
008C90D0  004C91E8  origspid.004C91E8
008C90D4  004C91CE  origspid.004C91CE
008C90D8  004C91BC  origspid.004C91BC
008C90DC  004C91B0  origspid.004C91B0
008C90E0  004C91A2  origspid.004C91A2
008C90E4  00000000
008C90E8  004C926E  origspid.004C926E
008C90EC  004C925E  origspid.004C925E
008C90F0  004C9252  origspid.004C9252
008C90F4  004C9246  origspid.004C9246
008C90F8  004C9236  origspid.004C9236
008C90FC  004C9224  origspid.004C9224
008C9100  004C9212  origspid.004C9212
008C9104  004C91FE  origspid.004C91FE
008C9108  004C91E8  origspid.004C91E8
008C910C  004C91CE  origspid.004C91CE
008C9110  004C91BC  origspid.004C91BC
008C9114  004C91B0  origspid.004C91B0
008C9118  004C91A2  origspid.004C91A2
008C911C  00000000
008C9120  004C927A  origspid.004C927A
008C9124  00000000
008C9128  004C927A  origspid.004C927A
008C912C  00000000
```

--------------------------------------------------------------------------

清除所有断点。与处理后的IAT比较一下，设置如下数据断点:

`hw 008C90A0`

--------------------------------------------------------------------------

```text
00FDDB8E    8908            mov     dword ptr [eax], ecx             ; 数据断点命中
00FDDB90    8B85 10D9FFFF   mov     eax, dword ptr [ebp-26F0]        ; origspid.008C90A0
00FDDB96    83C0 04         add     eax, 4
00FDDB99    8985 10D9FFFF   mov     dword ptr [ebp-26F0], eax
00FDDB9F  ^ E9 4DFCFFFF     jmp     00FDD7F1
```

--------------------------------------------------------------------------

单步跟踪这附近的代码执行情况，可以定位如下代码:

--------------------------------------------------------------------------

```text
00FDD9D2    50              push    eax                              ; 比较Win32 API的名字
00FDD9D3    FF15 7873FE00   call    near dword ptr [FE7378]          ; msvcrt.\_stricmp
00FDD9D9    59              pop     ecx
00FDD9DA    59              pop     ecx
00FDD9DB    85C0            test    eax, eax
00FDD9DD   /75 11           jnz     short 00FDD9F0                   ; 改成jmp，否则开始针对IAT中部分Entry进行特殊处理
00FDD9DF    8B85 58C2FFFF   mov     eax, dword ptr [ebp-3DA8]
00FDD9E5    8B40 08         mov     eax, dword ptr [eax+8]
00FDD9E8    8985 68CAFFFF   mov     dword ptr [ebp-3598], eax
00FDD9EE    EB 02           jmp     short 00FDD9F2
00FDD9F0  ^ EB 9C           jmp     short 00FDD98E
```

--------------------------------------------------------------------------

`0x00FDD9D3`处进行名字比较，检查是否是需要特殊处理的Win32 API。`0x00FDD9DD`处不跳转表明找到一个需要特殊处理的Win32 API，最终向IAT中写入`hook code`入口地址。若不需要特殊处理则直接在IAT中写入Win32 API入口地址。

重新用`OD`调试`origspider_trial.exe`，欺骗`OpenMutexA()`之后清除所有断点。

```text
hw 008C912C
F9
dd 008C9078
```

清除所有断点。

```text
hw 008C9094
F9
at 00FDD9DD
```

将`0x00FDD9DD`处的`jnz`改成`jmp`。

```text
hw 008C912C
F9
dd 008C9078
```

--------------------------------------------------------------------------

```text
008C9078  004C9192  origspid.004C9192
008C907C  004C917A  origspid.004C917A
008C9080  004C9168  origspid.004C9168
008C9084  004C9156  origspid.004C9156
008C9088  004C9148  origspid.004C9148
008C908C  004C9130  origspid.004C9130
008C9090  00000000
008C9094  77F3F9D8  ADVAPI32.ReportEventA
008C9098  77F3B857  ADVAPI32.RegisterEventSourceA
008C909C  77F3EBD7  ADVAPI32.RegSetValueExA
008C90A0  77F46A17  ADVAPI32.RegCreateKeyExA
008C90A4  77F56CCE  ADVAPI32.RegCloseKey
008C90A8  77F3F160  ADVAPI32.DeregisterEventSource
008C90AC  00FC7E3F
008C90B0  004C926E  origspid.004C926E
008C90B4  004C925E  origspid.004C925E
008C90B8  004C9252  origspid.004C9252
008C90BC  004C9246  origspid.004C9246
008C90C0  004C9236  origspid.004C9236
008C90C4  004C9224  origspid.004C9224
008C90C8  004C9212  origspid.004C9212
008C90CC  004C91FE  origspid.004C91FE
008C90D0  004C91E8  origspid.004C91E8
008C90D4  004C91CE  origspid.004C91CE
008C90D8  004C91BC  origspid.004C91BC
008C90DC  004C91B0  origspid.004C91B0
008C90E0  004C91A2  origspid.004C91A2
008C90E4  00000000
008C90E8  7C825529  kernel32.WriteFile
008C90EC  7C801DC6  kernel32.LoadLibraryA
008C90F0  7C959E17  ntdll.RtlFreeHeap
008C90F4  7C959FD6  ntdll.RtlAllocateHeap
008C90F8  7C82B437  kernel32.GetStdHandle
008C90FC  7C823EC7  kernel32.GetProcessHeap
008C9100  7C823D7A  kernel32.GetProcAddress
008C9104  7C82474A  kernel32.GetModuleHandleA   // 对比前述0x00FCB253处的代码
008C9108  7C8245FF  kernel32.GetModuleFileNameA
008C910C  7C830BE4  kernel32.GetEnvironmentVariableA
008C9110  7C831FE1  kernel32.FindFirstFileA
008C9114  7C82BFB3  kernel32.FindClose
008C9118  7C8268F1  kernel32.ExitProcess
008C911C  00FC7E49
008C9120  004C927A  origspid.004C927A
008C9124  00000000
008C9128  77E519CC  USER32.OemToCharA
008C912C  00FC7D67
```

--------------------------------------------------------------------------

务必撤消`0x00FDD9DD`处的修改，将`jmp`改回成`jnz`，清除所有断点。

```text
he 00401000
F9
```

断在`0x00401000`之后清除所有断点。现在我们有未被处理过的IAT。

运行`ArmInline v0.96 Final`，选中`origspider_trial.exe`，`Remove Splices`。

运行`LordPE`选中`origspider_trial.exe`，`dump full`，保存成`dumped.exe`。

运行`ImportREC`，选中`origspider_trial.exe`，将`OEP`改成`00001000`，自动查找`IAT`，现在`RVA`变成`004C9078`。获取输入表，显示无效函数，剪切指针，修复转存文件，这将生成一个`dumped_.exe`。

退出`OD`，双击执行`dumped_.exe`，搞定。脱壳后30天试用期自动去除。破解IP限制与脱壳无关，与本文无关。注意`origauditor_trial.exe`也加了壳，手脱原理同上。

本来还有个脱壳后的优化，照着下文简单折腾了一下:

脱壳后软件减肥大法 - yesky1 [2003-11-20]

eb1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3j5X3u0K6K9s2c8E0L8q4)9J5c8V1u0n7f1K6k6Q4x3V1k6H3k6h3c8A6P5e0j5K6x3e0y4Q4x3X3g2Z5N6r3@1\`.

但未成功，不打算深究这一步了。前面有几次提到撤消某某处的修改，是为了躲避内存中的自校验过程。

曾参看过一些手脱`Armadillo`壳的文章，不过不太适合我入门，主要原因是之前我完全不了解该壳的一些保护机制，如果照着TA们的步骤做，会有很多细节对不上号，比如那个所谓的`magic jmp`，照葫芦画瓢只会让我更困惑，还好我一开始也没打算照葫芦画瓢。由于这次的保护机制只用到两个:

`Debug-Blocker`

`Enable Strategic Code Splicing`

所以像我这样的脱壳新手还可以承受，再复杂的以后有需求时再学习吧。

没有技术含量的操作备忘录。

我把这次对我产生原理性的直接帮助的几篇文章列在正文中了。后面的参考资源则是
