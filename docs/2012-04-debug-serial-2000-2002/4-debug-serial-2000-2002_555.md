
# winimage

> 原文：[`www.pediy.com/kssd/pediy03/forum1124.htm`](https://www.pediy.com/kssd/pediy03/forum1124.htm)

`winimage` 完全破解

===================================================

Henan China peiyou

先运行注册，出现错误提示，怀疑是 `messageboxa` 中断。

随用 `trw2000` 输入 `peiyou 88888888` 下 `bpx messageboxa` 设断，中断于：

```
:00421F35 FF1588F94400                 Call user!messageboxa
```

向上找:

注:以下代码出自 `w32dasm`.

```
:00421F16 55                           push ebp
:00421F17 8BEC                         mov ebp, esp
:00421F19 51                           push ecx
:00421F1A A1D0C34400                   mov eax, dword ptr [0044C3D0]
:00421F1F 85C0                         test eax, eax
:00421F21 7405                         je 00421F28
                                       ^^^^^^^^^^^^跳到错误处
:00421F23 FFD0                         call eax
:00421F25 8945FC                       mov dword ptr [ebp-04], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00421F21(C)

:00421F28 56                           push esi
:00421F29 FF7514                       push [ebp+14]
:00421F2C FF7510                       push [ebp+10]
:00421F2F FF750C                       push [ebp+0C]
:00421F32 FF7508                       push [ebp+08]

* Reference To: USER32.MessageBoxA, Ord:01BEh

:00421F35 FF1588F94400                 Call dword ptr [0044F988]
                                       ^^^^^^^^^^^^^^^^^^^^^^^^^错误提示
:00421F3B 8BF0                         mov esi, eax
:00421F3D A1D4C34400                   mov eax, dword ptr [0044C3D4]
:00421F42 85C0                         test eax, eax
:00421F44 7406                         je 00421F4C
:00421F46 FF75FC                       push [ebp-04]
:00421F49 FFD0                         call eax
:00421F4B 59                           pop ecx
```

来到 `421f21` 处:

```
:00421F1F 85C0                         test eax, eax
:00421F21 7405                         je 00421F28
                                       ^^^^^^^^^^^^跳到错误处
```

改 `74 05` 为 `90 90` 吗？照死！再向上分析？

```
:00402FAA  , :00403BC7  , :00403C49  , :00405946  , :00409017
:004090AD  , :0040B82F  , :0040BB65  , :00421F9E  , :0042201C
```

我靠，你杀了我吧！！！

只好从注册成功信息如手。

用 `exescope` 找成功信息，看到了 `"你的注册码是正确的，$0A 现在你是一个注册用户了，$感谢你的使用。"`

用 `w32dsam` 找注册信息吧，慢！！！`w32dsam` 找中文注册信息?乱码,我头疼

(国之悲衷,我真希望中国的 cracker 们能写出中文 `w32dsam`,而不是汉化)

在信息前加入 `kissyou`,你没记录改动？我倒！！

运行 `w32dsam` 查 `kissyou`,找到 `kissyou+乱码`,乱吧，反正就是你了，向上看：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004340B8(U)
^^^^^^^^^^^^^^来自 4340b8

:004340E3 6800200000                   push 00002000
* Possible Reference to String Resource ID=01069: "WinImage 鑼"
:004340E8 682D040000                   push 0000042D
:004340ED 89150CD34400                 mov dword ptr [0044D30C], edx
:004340F3 891564D44400                 mov dword ptr [0044D464], edx

注册成功提示
* Possible Reference to String Resource ID=01066: "kissyou勮?/cn? ?/"
:004340F9 682A040000                   push 0000042A
```

到 `4340B8` 看一下：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004340AC(C)

:004340B3 6A01                         push 00000001
:004340B5 3BC2                         cmp eax, edx
:004340B7 5E                           pop esi
:004340B8 7529                         jnz 004340E3
                                       ^^^^^^^^^^^跳到注册成功处
:004340BA 6800200000                   push 00002000
```

我等你很久了，`75 29` 改为 `EB 29`.

重运行，随便输入注册，出现注册成功信息，我跳....不起来，怎么？

未注册信息仍未消除，重运行，讨厌的注册提示仍然存在，注册不成功!!!

我起不来了，拉我一把.

看来注册成功信息出现之前，作者已做了手脚。我很懒，总爱暴破，

但这次看来要玩一把了。

重新用 `trw2000` 载入调试,这次输入 `peiyou 88888888` 用 `hmemcpy` 设断

中断于 `434066 mov EDI 004d06c` 跟踪:

```
:00434066 BF6CD04400                   mov edi, 0044D06C
* Possible Ref to Menu: WINIMAGMENU, Item: "U(D)..."
:0043406B 6A7F                         push 0000007F
:0043406D 57                           push edi
* Possible Reference to Dialog: REGISTER, CONTROL_ID:0817, ""
:0043406E 6817080000                   push 00000817
:00434073 FF7508                       push [ebp+08]
:00434076 FFD6                         call esi
:00434078 6840D44400                   push 0044D440
:0043407D 57                           push edi
:0043407E 53                           push ebx
:0043407F E89C5C0000                   call 00439D20
                                       ^^^^^^^^^^^^^^
:00434084 8B0D40D44400                 mov ecx, dword ptr [0044D440]
:0043408A 83C40C                       add esp, 0000000C
:0043408D 33D2                         xor edx, edx
:0043408F A334D24400                   mov dword ptr [0044D234], eax
:00434094 3BC2                         cmp eax, edx
:00434096 5F                           pop edi
:00434097 5B                           pop ebx
:00434098 7406                         je 004340A0
:0043409A 890D04D44400                 mov dword ptr [0044D404], ecx
```

到这里

```
:0043407F E89C5C0000                   call 00439D20
                                       ^^^^^^^^^^^^^^它最有可能是密码处理,F8 追进去.
```

```
:00439D83 8D8500FFFFFF                 lea eax, dword ptr [ebp+FFFFFF00]
:00439D89 50                           push eax
:00439D8A 8D8748190514                 lea eax, dword ptr [edi+14051948]
:00439D90 50                           push eax
:00439D91 8D8500FEFFFF                 lea eax, dword ptr [ebp+FFFFFE00]
:00439D97 50                           push eax
:00439D98 E836FFFFFF                   call 00439CD3  处理 I
:00439D9D 59                           pop ecx
:00439D9E 59                           pop ecx
:00439D9F 50                           push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
:00439DA0 E8D33A0000                   Call 0043D878
:00439DA5 59                           pop ecx
:00439DA6 85C0                         test eax, eax
:00439DA8 59                           pop ecx
:00439DA9 7478                         je 00439E23
:00439DAB 8D8500FFFFFF                 lea eax, dword ptr [ebp+FFFFFF00]
:00439DB1 50                           push eax
:00439DB2 8D8754190617                 lea eax, dword ptr [edi+17061954]
:00439DB8 50                           push eax
:00439DB9 8D8500FEFFFF                 lea eax, dword ptr [ebp+FFFFFE00]
:00439DBF 50                           push eax
:00439DC0 E80EFFFFFF                   call 00439CD3  处理 II
:00439DC5 59                           pop ecx
:00439DC6 59                           pop ecx
:00439DC7 50                           push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
:00439DC8 E8AB3A0000                   Call 0043D878
:00439DCD 59                           pop ecx
:00439DCE 85C0                         test eax, eax
:00439DD0 59                           pop ecx
:00439DD1 7450                         je 00439E23
:00439DD3 8D8500FFFFFF                 lea eax, dword ptr [ebp+FFFFFF00]
:00439DD9 50                           push eax
:00439DDA 8D8781190510                 lea eax, dword ptr [edi+10051981]
:00439DE0 50                           push eax
:00439DE1 8D8500FEFFFF                 lea eax, dword ptr [ebp+FFFFFE00]
:00439DE7 50                           push eax
:00439DE8 E8E6FEFFFF                   call 00439CD3  处理 III
:00439DED 59                           pop ecx
:00439DEE 59                           pop ecx
:00439DEF 50                           push eax
* Reference To: CRTDLL.strcmp, Ord:01CFh
:00439DF0 E8833A0000                   Call 0043D878
:00439DF5 59                           pop ecx
:00439DF6 85C0                         test eax, eax
:00439DF8 59                           pop ecx
:00439DF9 7455                         je 00439E50
```



```
:00439DFB 8D8500FFFFFF             lea eax, dword ptr [ebp+FFFFFF00]

:00439E01 50                      push eax

:00439E02 8D8795190104            lea eax, dword ptr [edi+04011995]

:00439E08 50                      push eax

:00439E09 8D8500FEFFFF            lea eax, dword ptr [ebp+FFFFFE00]

:00439E0F 50                      push eax

:00439E10 E8BEFEFFFF              call 00439CD3

                                  ^^^^^^^^^^^^^

:00439E15 59                      pop ecx

:00439E16 59                      pop ecx

:00439E17 50                      push eax
```

经过三次处理，到`439f10`我`CALL`处出现错框:

再次跟踪到`:00439E0F 50` `push eax`处,下`D` `eax`

出现`105E9A34`.

注册`peiyou` `105E9A34`

注册成功，什么污七八糟的东西，全没了，成功注册。

另外,通过对注册表的分析发现它的注册信息存在于:

`HKEY_CURRENT_USER\Software\WinImage`

`"WinImageUseRegistry"="TRUE"` 是否过 30 天期

`"CDImageSetting"="0"`

`"ConnectedFileOption"="1"` 已使用的天数

`"NameRegistered"=""` 注册姓名

`"CodeRegistered"=""` 注册码

`"ProMode"="TRUE"` 运行于专业版，还是标准版模

如果不想破,把`promode`中的`Ture`改为`FAlSE`即为专业版，但提醒注册照出.

为了那片放飞梦想的天空而努力!!!!!!!!!1 `4dd466d0e1e0cc123afbe504da2462ca`
