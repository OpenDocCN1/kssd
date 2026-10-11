
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
