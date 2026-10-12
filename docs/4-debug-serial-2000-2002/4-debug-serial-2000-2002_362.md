
# Internet Maniac ver 1.2b

> 原文：[`www.pediy.com/kssd/pediy03/forum301.htm`](https://www.pediy.com/kssd/pediy03/forum301.htm)

`Internet Maniac ver 1.2b` Released: Sept.15, 1999

简介：

追踪：`name:dahuilang`

```
RN:01234567
```

今天应网友的要求，破解这个软件。现在我也是很少再破解软件了。毕竟没有时间了，所以今天拿来这个软件破解时感觉它不是很难，破解以后发现它的确不难。破解过程如下：使用 `bpx hmemcpy` 后拦下来

```
* Reference To: USER32.GetDlgItemTextA, Ord:0104h

:00405183 8B3DE8114100             mov edi, dword ptr [004111E8]
:00405189 8D442448                 lea eax, dword ptr [esp+48]
:0040518D 6800010000               push 00000100
:00405192 50                       push eax

* Possible Reference to Dialog: DialogID_007C, CONTROL_ID:044A, ""

:00405193 684A040000               push 0000044A
:00405198 56                       push esi
:00405199 FFD7                     call edi
:0040519B 8D4C2408                 lea ecx, dword ptr [esp+08]  <-从系统出来
:0040519F 6800010000               push 00000100
:004051A4 51                       push ecx

* Possible Reference to Dialog: DialogID_007C, CONTROL_ID:0449, ""

:004051A5 6849040000               push 00000449
:004051AA 56                       push esi
:004051AB FFD7                     call edi
:004051AD 8D542448                 lea edx, dword ptr [esp+48]
:004051B1 8D442408                 lea eax, dword ptr [esp+08]
:004051B5 52                       push edx
:004051B6 50                       push eax
:004051B7 E8142E0000               call 00407FD0
:004051BC 83C408                   add esp, 00000008
:004051BF 85C0                     test eax, eax
:004051C1 744A                     je 0040520D

* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:02E5h

:004051C3 8B3D28104100             mov edi, dword ptr [00411028]
:004051C9 8D4C2408                 lea ecx, dword ptr [esp+08]
:004051CD 68A0004300               push 004300A0
:004051D2 51                       push ecx

* Possible StringData Ref from Data Obj ->"User Name"

:004051D3 6824344100               push 00413424

* Possible StringData Ref from Data Obj ->"Internet Maniac"

:004051D8 6884314100               push 00413184
:004051DD FFD7                     call edi
:004051DF 8D542448                 lea edx, dword ptr [esp+48]
:004051E3 68A0004300               push 004300A0
:004051E8 52                       push edx

* Possible StringData Ref from Data Obj ->"Registration Code"

:004051E9 6838344100               push 00413438

* Possible StringData Ref from Data Obj ->"Internet Maniac"

:004051EE 6884314100               push 00413184
:004051F3 FFD7                     call edi
:004051F5 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Registration Successful"

:004051F7 6804364100               push 00413604

* Possible StringData Ref from Data Obj ->"Thank you for registering Internet "
                                     ->"Maniac. Enjoy the program."

:004051FC 68C4354100               push 004135C4
:00405201 56                       push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

:00405202 FF1554114100             Call dword ptr [00411154]
:00405208 6A01                     push 00000001
:0040520A 56                       push esi
:0040520B EB39                     jmp 00405246

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004051C1(C)

:0040520D 6A10                     push 00000010

* Possible StringData Ref from Data Obj ->"Failure!"

:0040520F 68B8354100               push 004135B8

* Possible StringData Ref from Data Obj ->"Incorrect registration code!"

:00405214 6898354100               push 00413598
:00405219 56                       push esi

* Reference To: USER32.MessageBoxA, Ord:01BEh

:0040521A FF1554114100             Call dword ptr [00411154]
:00405220 5F                       pop edi
:00405221 B801000000               mov eax, 00000001
:00405226 5E                       pop esi
:00405227 81C480000000             add esp, 00000080
:0040522D C21000                   ret 0010
```

大家可以先看一看上面的代码，你可以先看到从 `call edi` 出来，你可以发现这个 `edi` 值就为 `USER32.GetDlgItemTextA`。我想这个函数大家一定熟悉吧，就是得到你输入的 `name` 和 `RN`，那么现在它应使用两次这样的函数，所以在它的下面就还有一个 `call`，这就说明程序的确读取了两次数据。现在向下看你可以发现关键的比较地方，这个我在初学天地中介绍过的关键比较形式。大家不知道记住没有，如果记住的话，在这里就应用上了。因为下面是正确的信息和错误的信息，所以就进入 `call 00407FD0` 中。

```
:00407FD0 83EC20                   sub esp, 00000020
:00407FD3 56                       push esi
:00407FD4 8B742428                 mov esi, dword ptr [esp+28]
:00407FD8 56                       push esi  name

* Reference To: KERNEL32.lstrlenA, Ord:0308h

:00407FD9 FF1560104100             Call dword ptr [00411060]
:00407FDF 83F804                   cmp eax, 00000004
:00407FE2 7D07                     jge 00407FEB
```

这里当然是比较你的 `name` 个数是否小于 4，如果不是话，进行下面的比较。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00407FE2(C)

:00407FEB 0FBE4601                movsx eax, byte ptr [esi+01]
:00407FEF 0FBE4E02                movsx ecx, byte ptr [esi+02]
:00407FF3 D1E0                     shl eax, 1
:00407FF5 50                       push eax
:00407FF6 0FBE4603                movsx eax, byte ptr [esi+03]
:00407FFA C1E102                  shl ecx, 02
:00407FFD 51                       push ecx
:00407FFE B90A000000               mov ecx, 0000000A
:00408003 99                       cdq
:00408004 F7F9                     idiv ecx
:00408006 B8A0C634FA               mov eax, FA34C6A0
:0040800B 8BCA                     mov ecx, edx
:0040800D D3E0                     shl eax, cl
:0040800F 8D4C240C                 lea ecx, dword ptr [esp+0C]
:00408013 50                       push eax

* Possible StringData Ref from Data Obj ->"%010u-%d%d"

:00408014 68383B4100               push 00413B38
:00408019 51                       push ecx

* Reference To: USER32.wsprintfA, Ord:02ACh

:0040801A FF1544114100             Call dword ptr [00411144]
:00408020 8B542440                 mov edx, dword ptr [esp+40]
:00408024 83C414                   add esp, 00000014
:00408027 8D442404                 lea eax, dword ptr [esp+04]
:0040802B 52                       push edx
:0040802C 50                       push eax

* Reference To: KERNEL32.lstrcmpA, Ord:02FCh

:0040802D FF1544104100             Call dword ptr [00411044]
:00408033 F7D8                     neg eax
:00408035 1BC0                     sbb eax, eax
:00408037 5E                       pop esi
:00408038 40                       inc eax
:00408039 83C420                   add esp, 00000020
:0040803C C3                       ret
```

在这里你可以看到计算注册码的过程，如果这个程序你想制作注册机的话，分析一下上面的地方就可以了。注册码可以在

```
:0040802B 52                       push edx
:0040802C 50                       push eax
```

这里看到。我的注册码如下：

```
**********************
*   name:dahuilang       *
*   RN:0442716160-416194   *
**********************
```



`4dd466d0e1e0cc123afbe504da2462ca`
