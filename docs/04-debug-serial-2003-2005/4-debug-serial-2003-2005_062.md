
# AQUA 3D Screen Saver v1.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50122.htm`](https://www.pediy.com/kssd/pediy05/pediy50122.htm)

程序名：`AQUA 3D Screen Saver`  
版本：`v1.5`  
大小：`976KB`  
语言：`VC++ 6.0`  
运行平台：`Windows 98/Me/NT/2000`  
保护方式：未注册时，运行屏保 `3` 分钟后在屏幕中间会出现一个黑色的注册提示框。进入设置程序时提示 `"Welcome to Aqua 3D Screensaver UNREGISTERED Version"`。进入后 `Registered to:UNREGISTERED`，下面提示：`"Note:This computer program is shareware.Try it before you buy."` 无功能限制。  
破解方式：追注册码  
破解难度：中等  
破解工具：`TRW2000123`,`W32Dasm8.93`,`UltraEdit32`,`Fi`,`exescope4.0`  
程序下载：`http://www.digimindsoft.com`  
破解：`xbb[NCG] (2002/05/04)`

## 破解步骤

第一步，让它变成任意注册版本的软件。

首先将 `Aqua.src` 更名为 `Aqua.exe`，再用 `fi` (或者用 `language2000`,`GTW` 等侦测软件的软件)可以知道该软件未加壳。用 `W32Dasm8.93` 反汇编。在串式参考中找 `"Thanks for suppor!"` 双击到下面的地址：

```
* Possible Reference to String Resource ID=00001: "Aqua 3D Screen Saver"
                                  |
:0040C083 6A01                   push 00000001
:0040C085 C745FC00000000         mov [ebp-04], 00000000
:0040C08C E844480200             call 004308D5
:0040C091 8B465C                 mov eax, dword ptr [esi+5C]
:0040C094 8D4DEE                 lea ecx, dword ptr [ebp-12]
:0040C097 8D55E0                 lea edx, dword ptr [ebp-20]
:0040C09A 8D7E5C                 lea edi, dword ptr [esi+5C]
:0040C09D 51                     push ecx
:0040C09E 52                     push edx
:0040C09F 8D4DE4                 lea ecx, dword ptr [ebp-1C]
:0040C0A2 8D55E8                 lea edx, dword ptr [ebp-18]
:0040C0A5 51                     push ecx
:0040C0A6 52                     push edx
:0040C0A7 50                     push eax
:0040C0A8 E8A38EFFFF             call 00404F50
:0040C0AD 83C414                 add esp, 00000014
:0040C0B0 84C0                   test al, al
:0040C0B2 0F8493000000           je 0040C14B -->不能跳。
:0040C0B8 8B45E8                 mov eax, dword ptr [ebp-18]
:0040C0BB 8B55E4                 mov edx, dword ptr [ebp-1C]
:0040C0BE 8B4DE0                 mov ecx, dword ptr [ebp-20]
:0040C0C1 057E340000             add eax, 0000347E
:0040C0C6 33C2                   xor eax, edx
:0040C0C8 33C1                   xor eax, ecx
:0040C0CA 35CD540000             xor eax, 000054CD
:0040C0CF 663945EE               cmp word ptr [ebp-12], ax
:0040C0D3 7576                   jne 0040C14B -->不能跳。
:0040C0D5 A1F0454400             mov eax, dword ptr [004445F0]
:0040C0DA 8945E8                 mov dword ptr [ebp-18], eax
:0040C0DD 8945EC                 mov dword ptr [ebp-14], eax

* Possible Reference to String Resource ID=59145: "Registration"
                                  |
:0040C0E0 6809E70000             push 0000E709
:0040C0E5 8D4DE8                 lea ecx, dword ptr [ebp-18]
:0040C0E8 C645FC02               mov [ebp-04], 02
:0040C0EC E8540A0200             call 0042CB45

* Possible Reference to String Resource ID=59147: "Thanks for support!"
                                  |
:0040C0F1 680BE70000             push 0000E70B
:0040C0F6 8D4DEC                 lea ecx, dword ptr [ebp-14]
:0040C0F9 E8470A0200             call 0042CB45
:0040C0FE 85F6                   test esi, esi
:0040C100 7504                   jne 0040C106 -->不能跳。
:0040C102 33C0                   xor eax, eax
:0040C104 EB03                   jmp 0040C109

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C100(C)
|
:0040C106 8B461C                 mov eax, dword ptr [esi+1C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C104(U)
|
:0040C109 8B4DE8                 mov ecx, dword ptr [ebp-18]
:0040C10C 8B55EC                 mov edx, dword ptr [ebp-14]
:0040C10F 6A40                   push 00000040
:0040C111 51                     push ecx
:0040C112 52                     push edx
:0040C113 50                     push eax
```

将上面标记处更改即可注册。

1. `je`->`jne` `0F8493000000`->`0F8593000000`
2. `jne`->`nop` `7576`->`9090`
3. `jne`->`nop` `7504`->`9090`

---

第二步、去除 `"Welcome to Aqua 3D Screensaver UNREGISTERED Version"` 注册提示

用 `W32Dasm8.93` 反汇编。在串式参考中找 `"Welcome to Aqua 3D Screen Saver UNREGISTERED Version"` 双击到下面的地址：

```
* Possible StringData Ref from Data Obj ->"Aqua 3D: Setup"
                                  |
:0040B5F8 68802F4400             push 00442F80
:0040B5FD 8BCE                   mov ecx, esi
:0040B5FF E8D45C0200             call 004312D8
:0040B604 A1BC8F4500             mov eax, dword ptr [00458FBC]
:0040B609 8D4DEE                 lea ecx, dword ptr [ebp-12]
:0040B60C 8D55E0                 lea edx, dword ptr [ebp-20]
:0040B60F 51                     push ecx
:0040B610 52                     push edx
:0040B611 8D4DE4                 lea ecx, dword ptr [ebp-1C]
:0040B614 8D55E8                 lea edx, dword ptr [ebp-18]
:0040B617 51                     push ecx
:0040B618 52                     push edx
:0040B619 50                     push eax
:0040B61A E83199FFFF             call 00404F50
:0040B61F 83C414                 add esp, 00000014
:0040B622 84C0                   test al, al
:0040B624 741D                   je 0040B643 -->不能跳。
:0040B626 8B45E8                 mov eax, dword ptr [ebp-18]
:0040B629 8B5DE4                 mov ebx, dword ptr [ebp-1C]
:0040B62C 8B7DE0                 mov edi, dword ptr [ebp-20]
:0040B62F 057E340000             add eax, 0000347E
:0040B634 33C3                   xor eax, ebx
:0040B636 33C7                   xor eax, edi
:0040B638 35CD540000             xor eax, 000054CD
:0040B63D 663945EE               cmp word ptr [ebp-12], ax
:0040B641 745D                   je 0040B6A0 -->不能不跳。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040B624(C)
|
:0040B643 A1F0454400             mov eax, dword ptr [004445F0]
:0040B648 8945E8                 mov dword ptr [ebp-18], eax
:0040B64B 8945EC                 mov dword ptr [ebp-14], eax

* Possible Reference to String Resource ID=59148: "Aqua 3D"
                                  |
:0040B64E 680CE70000             push 0000E70C
:0040B653 8D4DE8                 lea ecx, dword ptr [ebp-18]
:0040B656 C645FC02               mov [ebp-04], 02
:0040B65A E8E6140200             call 0042CB45

* Possible Reference to String Resource ID=59149: "Welcome to Aqua 3D Screensaver UNREGISTERED Version"
                                  |
:0040B65F 680DE70000             push 0000E70D
:0040B664 8D4DEC                 lea ecx, dword ptr [ebp-14]
:0040B667 E8D9140200             call 0042CB45
:0040B66C 85F6                   test esi, esi
:0040B66E 7504                   jne 0040B674
:0040B670 33C0                   xor eax, eax
:0040B672 EB03                   jmp 0040B677

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040B66E(C)
|
:0040B674 8B461C                 mov eax, dword ptr [esi+1C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040B672(U)
|
:0040B677 8B4DE8                 mov ecx, dword ptr [ebp-18]
:0040B67A 8B55EC                 mov edx, dword ptr [ebp-14]
:0040B67D 6A00                   push 00000000
:0040B67F 51                     push ecx
:0040B680 52                     push edx
:0040B681 50                     push eax
```

将上面标记处修改即可去除注册提示。

1. `je`->`9090` `741D`->`9090`
2. `je`->`jmp` `745D`->`EB5D`



### 第三步、去除运行时出现的黑色的注册提示框

使用 `W32Dasm8.93` 反汇编。在串式参考中找 `"REGISTER TODAY TO REMOVE THIS MESSAGE"`，双击跳转到下面的地址：

……

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00404C85(C)`

:00404CC1 84D2                   test dl, dl
:00404CC3 7408                   je 00404CCD <---这里不跳即可去除黑色提示框。
:00404CC5 84C0                   test al, al
:00404CC7 0F85B8010000           jne 00404E85

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00404CC3(C)`

:00404CCD DD4318                 fld qword ptr [ebx+18]
:00404CD0 DC0DC8874300           fmul qword ptr [004387C8]
:00404CD6 680000803F             push 3F800000
:00404CDB 6A00                   push 00000000
:00404CDD 51                     push ecx
:00404CDE DC25C0854300           fsub qword ptr [004385C0]
:00404CE4 D91C24                 fstp dword ptr [esp]
:00404CE7 E8F4120000             call 00405FE0
:00404CEC D95DC4                 fstp dword ptr [ebp-3C]
:00404CEF 8B45C4                 mov eax, dword ptr [ebp-3C]
:00404CF2 83C40C                 add esp, 0000000C
:00404CF5 8945C4                 mov dword ptr [ebp-3C], eax
:00404CF8 68E20B0000             push 00000BE2

* Reference To: `OPENGL32.glEnable`, Ord:0050h

:00404CFD FF1510834300           Call dword ptr [00438310]
:00404D03 6803030000             push 00000303
:00404D08 6802030000             push 00000302

* Reference To: `OPENGL32.glBlendFunc`, Ord:000Eh

:00404D0D FF15F0824300           Call dword ptr [004382F0]
:00404D13 8B7DC4                 mov edi, dword ptr [ebp-3C]
:00404D16 57                     push edi
:00404D17 6A00                   push 00000000
:00404D19 6A00                   push 00000000
:00404D1B 6A00                   push 00000000

* Reference To: `OPENGL32.glColor4f`, Ord:002Ch

:00404D1D FF15F4824300           Call dword ptr [004382F4]
:00404D23 68E10D0000             push 00000DE1

* Reference To: `OPENGL32.glDisable`, Ord:0047h

:00404D28 FF15DC824300           Call dword ptr [004382DC]
:00404D2E E88D0F0000             call 00405CC0
:00404D33 6A07                   push 00000007

* Reference To: `OPENGL32.glBegin`, Ord:000Bh

:00404D35 FF15F8824300           Call dword ptr [004382F8]
:00404D3B 6A00                   push 00000000
:00404D3D 680000803E             push 3E800000
:00404D42 68EC51B83D             push 3DB851EC
:00404D47 FFD6                   call esi
:00404D49 6A00                   push 00000000
:00404D4B 680000003F             push 3F000000
:00404D50 68EC51B83D             push 3DB851EC
:00404D55 FFD6                   call esi
:00404D57 6A00                   push 00000000
:00404D59 680000003F             push 3F000000
:00404D5E 68C3F5683F             push 3F68F5C3
:00404D63 FFD6                   call esi
:00404D65 6A00                   push 00000000
:00404D67 680000803E             push 3E800000
:00404D6C 68C3F5683F             push 3F68F5C3
:00404D71 FFD6                   call esi

* Reference To: `OPENGL32.glEnd`, Ord:0052h

:00404D73 FF1504834300           Call dword ptr [00438304]
:00404D79 E8A20F0000             call 00405D20
:00404D7E 8B0DF0454400           mov ecx, dword ptr [004445F0]
:00404D84 894DDC                 mov dword ptr [ebp-24], ecx

* Possible Reference to String Resource ID=59152: "Aqua 3D Screensaver"

:00404D87 6810E70000             push 0000E710
:00404D8C 8D4DDC                 lea ecx, dword ptr [ebp-24]
:00404D8F C645FC04               mov [ebp-04], 04
:00404D93 E8AD7D0200             call 0042CB45

* Reference To: `OPENGL32.glColor4f`, Ord:002Ch

:00404D98 8B35F4824300           mov esi, dword ptr [004382F4]
:00404D9E 57                     push edi
:00404D9F 680000803F             push 3F800000
:00404DA4 680000803F             push 3F800000
:00404DA9 6A00                   push 00000000
:00404DAB FFD6                   call esi
:00404DAD 8B55DC                 mov edx, dword ptr [ebp-24]
:00404DB0 83C330                 add ebx, 00000030
:00404DB3 52                     push edx
:00404DB4 688FC2F53C             push 3CF5C28F
:00404DB9 680AD7A33C             push 3CA3D70A
:00404DBE 685C8F023F             push 3F028F5C
:00404DC3 689A99993E             push 3E99999A
:00404DC8 8BCB                   mov ecx, ebx
:00404DCA E8B10C0000             call 00405A80
:00404DCF A1F0454400             mov eax, dword ptr [004445F0]
:00404DD4 8945D8                 mov dword ptr [ebp-28], eax

* Possible Reference to String Resource ID=59150: "REGISTER TODAY TO REMOVE THIS MESSAGE"

:00404DD7 680EE70000             push 0000E70E <---我们来到这里,向上找跳转。
:00404DDC 8D4DD8                 lea ecx, dword ptr [ebp-28]
:00404DDF C645FC05               mov [ebp-04], 05
:00404DE3 E85D7D0200             call 0042CB45
:00404DE8 57                     push edi
:00404DE9 680000803F             push 3F800000
:00404DEE 680000803F             push 3F800000
:00404DF3 680000803F             push 3F800000
:00404DF8 FFD6                   call esi
:00404DFA 8B4DD8                 mov ecx, dword ptr [ebp-28]
:00404DFD 51                     push ecx
:00404DFE 688FC2F53C             push 3CF5C28F
:00404E03 680AD7A33C             push 3CA3D70A
:00404E08 689A99193F             push 3F19999A
:00404E0D 68B81E053E             push 3E051EB8
:00404E12 8BCB                   mov ecx, ebx
:00404E14 E8670C0000             call 00405A80
:00404E19 8B15F0454400           mov edx, dword ptr [004445F0]
:00404E1F 8955D4                 mov dword ptr [ebp-2C], edx

* Possible Reference to String Resource ID=59151: "press the space bar to find out how to register"

:00404E22 680FE70000             push 0000E70F
:00404E27 8D4DD4                 lea ecx, dword ptr [ebp-2C]
:00404E2A C645FC06               mov [ebp-04], 06
:00404E2E E8127D0200             call 0042CB45
:00404E33 57                     push edi
:00404E34 680000803F             push 3F800000
:00404E39 680000803F             push 3F800000
:00404E3E 6A00                   push 00000000
:00404E40 FFD6                   call esi
:00404E42 8B45D4                 mov eax, dword ptr [ebp-2C]
:00404E45 8BCB                   mov ecx, ebx
:00404E47 50                     push eax
:00404E48 680AD7A33C             push 3CA3D70A
:00404E4D 6896438B3C             push 3C8B4396
:00404E52 688FC2353F             push 3F35C28F
:00404E57 68CDCCCC3D             push 3DCCCCCD
:00404E5C E81F0C0000             call 00405A80
:00404E61 8D4DD4                 lea ecx, dword ptr [ebp-2C]
:00404E64 C645FC05               mov [ebp-04], 05
:00404E68 E8217A0200             call 0042C88E
:00404E6D 8D4DD8                 lea ecx, dword ptr [ebp-28]
:00404E70 C645FC04               mov [ebp-04], 04
:00404E74 E8157A0200             call 0042C88E
:00404E79 8D4DDC                 lea ecx, dword ptr [ebp-24]
:00404E7C C645FC00               mov [ebp-04], 00



`:00404E80 E8097A0200  call 0042C88E`

……

将上面`00404CC3 7408`地址处的`7408`改为`9090`即可。

-------------------------------------------------------------------------------------------

### 第四步、将 Registered to:UNREGISTERED 改为 Registered to:你的名字

用`exescope4.0`打开`Aqua.exe`。点击`Resource->Dialog->102`在`exescope4.0`右边的窗口中找`Static:Registered to:%s`,然后在标题处把`%s`改为你的名字保存即可。

-------------------------------------------------------------------------------------------

### 第五步、将"Note:This computer program is shareware.Try it before you buy."改掉

用`exescope4.0`打开`Aqua.exe`。点击`Resource->Dialog->102`在`exescope4.0`右边的窗口中找`Note: This computer program is shareware.$0A Try it before you buy.`在标题处改为注册提示(此处可随你的意思更改)即可。

```
xbb\[NCG\]
2002.5.4 4dd466d0e1e0cc123afbe504da2462ca
```
