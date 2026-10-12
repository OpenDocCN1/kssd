
# 飘雪动画秀 V3.02

> 原文：[`www.pediy.com/kssd/pediy05/pediy50430.htm`](https://www.pediy.com/kssd/pediy05/pediy50430.htm)

算法分析——飘雪动画秀 V3.02

**下载地址**：[`www.skycn.com/soft/7563.html`](http://www.skycn.com/soft/7563.html)

**软件大小**：332 KB

**软件语言**：简体中文

**软件类别**：国产软件 / 共享版 / 动画制作

**应用平台**：`Win9x/NT/2000/XP`

**加入时间**：2002-11-05 08:21:40

**下载次数**：8800

**推荐等级**：***

**开发商**：[`piaoxue666.51.net/`](http://piaoxue666.51.net/)

### 【软件简介】
飘雪动画秀是一套非常好的动画制作软件，她几乎拥有制作 `GIF` 动画所需的所有功能，无须再用其它的图型软件辅助。它可以处理背景透明化而且做法非常容易，做好的图片可以做最佳化处理使图片减肥，另外它除了可以把做好的图片存成 `GIF` 的动画图外，还可支援 `PSD`，`JPEG`，`AVI`，`BMP`，`GIF`，与 `AVI` 格式输出！

### 【软件限制】
30 天试用。

### 【作者声明】
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

### 【破解工具】
`TRW2000` 娃娃修改版、`FI2.5`、`AspackDie`、`W32Dasm8.93` 黄金版

---

### 【过程】

呵呵，脱壳、反汇编、调试、整理解文，老规矩了。

我会把我近日所写的解文贴上来，算是交作业，亦算是我学 CRACK 以来的一个总结吧！

---

### 一、脱壳

`move.exe` 是 `ASPACK 2.12` 壳，用 `AspackDie` 脱之。202K->564K。

---

### 二、反汇编、调试

填入试炼信息：

*   **Name**：`fly`
*   **Code**：`13572468`

`BPX HMEMCPY`，当然也可设置其它的断点，呵呵，万能断点用惯了。

点“确定”，拦下！

`BD`，暂停断点。`PMODULE` 返至程序领空。至 `431972`。

`F10` 走。多看看，多留心跳转。很快我们就会来到了核心。

```assembly
:00431972 8D9424C4000000          lea edx, dword ptr [esp+000000C4]

* Possible Reference to Dialog: DialogID_0064   

                       |
:00431979 6A64                  push 00000064  
:0043197B 52                    push edx

* Possible Reference to Dialog: DialogID_0091, CONTROL_ID:0450, ""  

                       |
:0043197C 6850040000             push 00000450  
:00431981 56                    push esi  
:00431982 FFD7                  call edi  
:00431984 50                    push eax  
:00431985 FFD3                  call ebx  
:00431987 8D8424C4000000         lea eax, dword ptr [esp+000000C4]  
                       ==== >13572468 入 EAX  
:0043198E 8D4C2460               lea ecx, dword ptr [esp+60]  
                       ==== >fly 入 ECX  
:00431992 50                    push eax  
:00431993 51                    push ecx  
:00431994 E8F7FBFFFF             call 00431590  
                       ==== >关键 CALL！F8 进入！

:00431999 83C408                 add esp, 00000008  
:0043199C 85C0                   test eax, eax  
:0043199E 0F84AD000000           je 00431A51  
                       ==== >跳则 OVER！

:004319A4 8D542410               lea edx, dword ptr [esp+10]  
:004319A8 8D44240C               lea eax, dword ptr [esp+0C]  
:004319AC 52                    push edx  
:004319AD 50                    push eax  
:004319AE 6A00                  push 00000000  
:004319B0 683F000F00             push 000F003F  
:004319B5 6A00                  push 00000000  
:004319B7 6814ED4400             push 0044ED14  
:004319BC 6A00                  push 00000000

* Possible StringData Ref from Data Obj ->"Software\gamani\GIFMovieGear\2.0"  

                       ==== >注册信息存放位置！  
:004319BE 68F8B34400             push 0044B3F8  
:004319C3 6801000080             push 80000001

* Reference To: ADVAPI32.RegCreateKeyExA, Ord:015Fh  

                       |
:004319C8 FF150C804400           Call dword ptr [0044800C]  
:004319CE 8D7C2460               lea edi, dword ptr [esp+60]  
:004319D2 83C9FF                 or ecx, FFFFFFFF  
:004319D5 33C0                   xor eax, eax  
:004319D7 8B54240C               mov edx, dword ptr [esp+0C]  
:004319DB F2                    repnz  
:004319DC AE                    scasb  
:004319DD F7D1                  not ecx

* Reference To: ADVAPI32.RegSetvalueExA, Ord:0186h  

                       |
:004319DF 8B1D18804400           mov ebx, dword ptr [00448018]  
:004319E5 51                    push ecx  
:004319E6 8D4C2464               lea ecx, dword ptr [esp+64]  
:004319EA 51                    push ecx  
:004319EB 6A01                  push 00000001  
:004319ED 50                    push eax

* Possible StringData Ref from Data Obj ->"RegName3"  

                       |
:004319EE 6898D44400             push 0044D498  
:004319F3 52                    push edx  
:004319F4 FFD3                  call ebx  
:004319F6 8DBC24C4000000         lea edi, dword ptr [esp+000000C4]  
:004319FD 83C9FF                 or ecx, FFFFFFFF  
:00431A00 33C0                   xor eax, eax  
:00431A02 F2                    repnz  
:00431A03 AE                    scasb  
:00431A04 F7D1                  not ecx  
:00431A06 8D8424C4000000         lea eax, dword ptr [esp+000000C4]  
:00431A0D 51                    push ecx  
:00431A0E 8B4C2410               mov ecx, dword ptr [esp+10]  
:00431A12 50                    push eax  
:00431A13 6A01                  push 00000001  
:00431A15 6A00                  push 00000000

* Possible StringData Ref from Data Obj ->"RegCode3"  

                       |
:00431A17 68A4D44400             push 0044D4A4  
:00431A1C 51                    push ecx  
:00431A1D FFD3                  call ebx  
:00431A1F 8B54240C               mov edx, dword ptr [esp+0C]  
:00431A23 52                    push edx

* Reference To: ADVAPI32.RegCloseKey, Ord:015Bh  

                       |
:00431A24 FF1500804400           Call dword ptr [00448000]

* Possible StringData Ref from Data Obj ->"Software\Loani\MG3t"  

                       |
:00431A2A 68B0D44400             push 0044D4B0
```

---

`F8` 进入关键 CALL：`00431994 call 00431590`

*   Referenced by a CALL at Addresses:
    *   `:004316D9`
    *   `:00431994`

```assembly
:00431590 53                    push ebx  
:00431591 55                    push ebp  
:00431592 8B6C2410               mov ebp, dword ptr [esp+10]  
                       ==== >试炼码入 EBP  
:00431596 56                    push esi  
:00431597 57                    push edi  
:00431598 807D006D               cmp byte ptr [ebp+00], 6D  
                       ==== >试炼码的第 1 位是否是 m？

:0043159C 0F85A0000000           jne 00431642  
                       ==== >跳则 OVER！R FL Z  
:004315A2 807D0167               cmp byte ptr [ebp+01], 67  
                       ==== >试炼码的第 2 位是否是 g？

:004315A6 0F8596000000           jne 00431642  
                       ==== >跳则 OVER！R FL Z  
:004315AC 807D0233               cmp byte ptr [ebp+02], 33  
                       ==== >试炼码的第 3 位是否是 3？

:004315B0 0F858C000000           jne 00431642  
                       ==== >跳则 OVER！R FL Z  
:004315B6 807D0337               cmp byte ptr [ebp+03], 37  
                       ==== >试炼码的第 4 位是否是 7？

:004315BA 0F8582000000           jne 00431642  
                       ==== >跳则 OVER！R FL Z

所以：你可以把你的试炼码的前 4 位改为 `mg37`，也可不改，直接 `R FL Z` 跳过！

*   Possible Indirect StringData Ref from Data Obj ->"mvg21951736"

    ==== >注意此字符串。好象用处不大。

:004315C0 BBC4D44400             mov ebx, 0044D4C4  
                       ==== >mvg21951736 入 EBX  

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004315E6(C)`

:004315C5 8B13                   mov edx, dword ptr [ebx]  
:004315C7 83C9FF                 or ecx, FFFFFFFF  
:004315CA 8BFA                   mov edi, edx  
:004315CC 33C0                   xor eax, eax  
:004315CE F2                    repnz  
:004315CF AE                    scasb  
:004315D0 F7D1                  not ecx  
:004315D2 49                    dec ecx  
:004315D3 8BFA                   mov edi, edx  
:004315D5 8BF5                   mov esi, ebp  
:004315D7 33C0                   xor eax, eax  
:004315D9 F3                    repz  
:004315DA A6                    cmpsb  
:004315DB 7465                  je 00431642  
```



# 汇编代码分析

## 0x004315C5 - 0x004315EF

```asm
:004315DD 83C304 add ebx, 00000004
:004315E0 81FBC8D44400 cmp ebx, 0044D4C8
:004315E6 7CDD jl 004315C5
:004315E8 807D0473 cmp byte ptr [ebp+04], 73
```

====> 试炼码的第 5 位是否是 `s`？
若是 `s`，则成 `Site License` 版本。

```asm
:004315EC 7501 jne 004315EF
```

====> 若不是 `s`，则跳到 `0x4315EF` 处。

```asm
:004315EE 45 inc ebp
```

====> 若是 `s`，则增 1。
即下面的 `0x004315EF` 处从试炼码的第 8 位取后面的字符。

## 0x004315EF - 0x0043160C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004315EC(C)

```asm
:004315EF 83C507 add ebp, 00000007
```

====> 取试炼码第 7 位后的字符，我的是 8。

```asm
:004315F2 55 push ebp
:004315F3 E8D0DD0000 call 0043F3C8
```

====> 此 `CALL` 把 8 存入 `EAX`。跟进去就明白了。
用 `EAX` 的值进行最后的比较！
如果你输入的试炼码很多，那么此 `CALL` 只取你第 7 位后的数字，字母及其后的值全舍弃。

```asm
:004315F8 8B542418 mov edx, dword ptr [esp+18]
```

====> `fly` 入 `EDX`。

```asm
:004315FC 83C404 add esp, 00000004
:004315FF 8BFA mov edi, edx
:00431601 33C9 xor ecx, ecx
:00431603 8A12 mov dl, byte ptr [edx]
```

====> 依次从用户名中取字符的 `HEX` 值。
1、？`DL=66` 即 `f` 的 `HEX` 值
2、？`DL=6C` 即 `l` 的 `HEX` 值
3、？`DL=79` 即 `y` 的 `HEX` 值

## 0x0043160E - 0x00431632

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431632(C)

```asm
:004315EF 83C507 add ebp, 00000007
:004315F2 55 push ebp
:004315F3 E8D0DD0000 call 0043F3C8
:004315F8 8B542418 mov edx, dword ptr [esp+18]
:004315FC 83C404 add esp, 00000004
:004315FF 8BFA mov edi, edx
:00431601 33C9 xor ecx, ecx
:00431603 8A12 mov dl, byte ptr [edx]
```

(注意：原文中 004315EF 重复出现在此段前，根据逻辑流，此处应为 00431605 开始)

```asm
:00431605 BEDF0B0000 mov esi, 00000BDF
```

====> `BDF` 入 `ESI`。程序给的值。

```asm
:0043160A 84D2 test dl, dl
:0043160C 7426 je 00431634
```

## 0x0043160E - 0x00431617

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431632(C)

```asm
:0043160E 0FBED2 movsx edx, dl
```

====> 用户名的 `HEX` 值依次入 `EDX`。

```asm
:00431611 41 inc ecx
```

====> `ECX` 依次增 1。

```asm
:00431612 0FAFD1 imul edx, ecx
```

====> 用户名字符值依次与次数相乘
1、`EDX=66*1=66`
2、`EDX=6C*2=D8`
3、`EDX=79*3=16B`

```asm
:00431615 03F2 add esi, edx
```

====> 先加 `BDF`，再累加！
1、`ESI=66 +BDF=C45`
2、`ESI=D8 +C45=D1D`
3、`ESI=16B+D1D=E88`

## 0x00431617 - 0x0043162C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043161D(C)

```asm
:00431617 81FEBE170000 cmp esi, 000017BE
```

====> 所得结果与 `17BE` 比较。

```asm
:0043161D 7E06 jle 00431625
```

====> 小于则跳。

```asm
:0043161F 81EEBE170000 sub esi, 000017BE
```

====> 大于 `17BE` 则再减去 `17BE`。

## 0x00431625 - 0x00431634

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431628(C)

```asm
:00431625 83F90A cmp ecx, 0000000A
:00431628 7E02 jle 0043162C
:0043162A 33C9 xor ecx, ecx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043160C(C)

```asm
:0043162C 8A5701 mov dl, byte ptr [edi+01]
```

====> 继续取下一位字符。

```asm
:0043162F 47 inc edi
:00431630 84D2 test dl, dl
:00431632 75DA jne 0043160E
```

====> 继续循环，直至取完所有用户名字符。

## 0x00431634 - 0x00431641

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043160C(C)

```asm
:00431634 3BF0 cmp esi, eax
```

====> 最后所得结果与 `EAX` 比较。
`EAX` 就是试炼码第 7 位后的数字，所以？ `ESI=1720` 我们的最后几位真码！

```asm
:00431636 750A jne 00431642
```

====> 跳则 `OVER`！

```asm
:00431638 5F pop edi
:00431639 5E pop esi
:0043163A 5D pop ebp
:0043163B B801000000 mov eax, 00000001
```

====> 喜欢看到这儿的 1，呵呵！

```asm
:00431640 5B pop ebx
:00431641 C3 ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0043159C(C), :004315A6(C), :004315B0(C), :004315BA(C), :004315DB(C)
|:00431636(C)

## 0x00431642 - 0x00431648

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00431636(C)

```asm
:00431642 5F pop edi
:00431643 5E pop esi
:00431644 5D pop ebp
:00431645 33C0 xor eax, eax
```

====> 跳到这儿！`EAX` 清 0，就 `OVER` 了。
若你还想做个完美破解版，就改这儿最好了，只要改此一处！

```asm
:00431647 5B pop ebx
:00431648 C3 ret
```

# 完美爆破

只要改此一处即可！

`431645 33C0 xor eax, eax`

改为：`431645 INC eax` 即：`33C0 -->FEC0`

让其返回的 `EAX` 值永远是 1。
呵呵，与上面的 `43163B mov eax, 00000001` 相映成趣。

# 总结

注册码的前 4 位固定为：`mg37`（第 5 位若是 `s`，则为 `Site License` 版本）

第 5、6、7 位可为任意数，第 7 位后的数字为用户名与程序给的数字进行简单运算得出！

（第 5 位若是 `s`，则第 6、7、8 位可为任意数，第 8 位后的数字为用户名与程序给的数字进行简单运算得出！）

# 注册信息保存

`[HKEY_CURRENT_USER\Software\gamani\GIFMovieGear\2.0]`

`"RegName3"="fly"`

`"RegCode3"="mg372463720"`

# 整理

`Name`: `fly`
`Code`: `mg372463720`

或者：

`Name`=`fly`
`Code`=`mg37s1233720` ====> `Site License` 版本

---

`Cracked By` 巢水工作坊——`fly`【`OCN`】
`2003-2-6 21:40`
`4dd466d0e1e0cc123afbe504da2462ca`
