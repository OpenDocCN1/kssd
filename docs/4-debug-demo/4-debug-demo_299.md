
# 超级解霸 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum748.htm`](https://www.pediy.com/kssd/pediy03/forum748.htm)

超级解霸 2000(全功能限时版)破解文档

-----------------------------------

雪椰

2001.3.26am

Email:`wocy@263.net`

Http://`wocy.yeah.net`

-----------------------------------

该版本是新电脑 2000.7 提供的限次限时版

时限为一个月内 30 次。效验方式有：

1. 系统目录中`sthvcd.ini`中，`berun=x`项，`x`-->使用次数>30 则盗版提示
2. 注册表中`H.K_SOFTWARE_MICROSOFT_WINDOWS_CURRENTVERSION_SETUP`的`runtime`项<=1 则盗版提示
3. 当前日期距安装一个月则盗版提示

破解目标：

1. 无上述 3 项限制
2. 改时间提示时的-豪杰超级解霸-字符

工具：

`w32dasm`,`soft_ice`,`hiew`,`apatch`

------------------------------------

`sthsvcd.exe`

------------------------------------

1. `soft_ice`->`bpx getsystemtime`       破第 3 项限制

   `going....`

   `:00414981 50 push eax`

   `* Reference To: KERNEL32.GetSystemTime, Ord:01C6h ----------> | :00414982 FF15A4064A00 Call dword ptr [004A06A4]`

   `:00414988 8B4C2468 mov ecx, dword ptr [esp+68]`

   `:0041498C 33C0 xor eax, eax`

   `:0041498E 668B44246A mov ax, word ptr [esp+6A]`

   `:00414993 81E1FFFF0000 and ecx, 0000FFFF`

   `:00414999 C1E104 shl ecx, 04`

   `:0041499C 8B1580874200 mov edx, dword ptr [00428780]`

   `:004149A2 0BC8 or ecx, eax`

   `:004149A4 33C0 xor eax, eax`

   `:004149A6 C1E108 shl ecx, 08`

   `:004149A9 668B44246E mov ax, word ptr [esp+6E]`

   `:004149AE 0BC8 or ecx, eax`

   `:004149B0 3BCA cmp ecx, edx ----------->(a)`

   `:004149B2 7622 jbe 004149D6`

   `:004149B4 E807B3FFFF call 0040FCC0`

   if you modify (a) to ...
   `cmp ecx,ecx`
   `jz 4149d6`
   then ok

2. `w32dasm`->`string find (berun)`      破 no.1 限制

   `going...`

   `* Possible StringData Ref from Data Obj ->"BERUN" | :0041493D 68C48C4200 push 00428CC4`

   `* Possible StringData Ref from Data Obj ->"SETTING" | :00414942 6810674200 push 00426710`

   `* Reference To: KERNEL32.WritePrivateProfileStringA, Ord:033Bh | :00414947 FF1568064A00 Call dword ptr [004A0668]`

   `:0041494D 83FB1E cmp ebx, 0000001E ----------->(b)`

   `:00414950 7E22 jle 00414974`

   `:00414952 E869B3FFFF call 0040FCC0`

   if you modify (b) to...
   `cmp ecx,ecx`
   `nop`
   `jz 414974`
   then ok

3. 这两项挺近的，所以查看附近代码，`find...`       破 no.2

   `* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: |:0041497B(C), :004149B2(C) | :004149D6 E875B3FFFF call 0040FD50`

   `:004149DB 85C0 test eax, eax ------------>(c)`

   `:004149DD 7522 jne 00414A01`

   if you modify (c) to...
   `cmp ecx,ecx`
   `jz 414a01`
   then ok

4. 改显示字符，

   该字符在资源中找不到，所以

   `soft-ice->`
   `s 0 l ffffffff ba c0 bd dc b3 ac (豪杰超)`
   `:find at....(467b48)`
   `bpm 467b48 w`
   `g`
   重启程式.`break and trace out at....(d)`

   `:00414A1A 6A50 push 00000050`
   `:00414A1C 68487B4600 push 00467B48`
   `:00414A21 A1BC874200 mov eax, dword ptr [004287BC]`

   `* Possible Reference to String Resource ID=50058: "HeroSDVD 2000" | :00414A26 688AC30000 push 0000C38A`

   `:00414A2B 50 push eax`

   `* Reference To: USER32.LoadStringA, Ord:01A9h | :00414A2C FF15D4084A00 Call dword ptr [004A08D4] --------->(d)`

   `:00414A32 68D0664200 push 004266D0`

   实验发现在(d)执行完后 `467b48` 处为显示的字符----〉豪杰超级解霸，此字符从何来，不知道，懒的查了，决定编段程式修改 `467b48` 处的字符，

   经过上面的破解，有 3 处空间可用

   `414952-414973`     (用作主程式)
   `4149b4-4149d5`     (用作存放我的字符串)
   `4149df-414a00`     (no use)

   modify 1:
   `:00414A2C E921FFFFFF jmp 00414952 //run my main prog`
   `:00414A31 90 nop`

   modify 2:
   `:00414952 FF15D4084A00 Call dword ptr [004A08D4] //first run the old func`
   `:00414958 56 push esi`
   `:00414959 57 push edi`
   `:0041495A BEB4494100 mov esi, 004149B4 //your char address`
   `:0041495F BF487B4600 mov edi, 00467B48 //the dest address`
   `:00414964 FC cld`
   `:00414965 B922000000 mov ecx, 00000022 //char number`
   `:0041496A F3 repz`
   `:0041496B A4 movsb`
   `:0041496C 5F pop edi`
   `:0041496D 5E pop esi`
   `:0041496E 90 nop`
   `:0041496F E9BE000000 jmp 00414A32 //ret`

   modify 3:
   `4149b4:" Crack:wocy wocy@263.net",00,00 //the new char, 00 at end`

---------------------------------------------

`mmxado.exe`

差不多

----------------------------------------------

5. use `apatch` to patch.

my .aps file here.

---------------------------------------------------------

```
^TITLE^
Crack:Wocy
^MESSAGE^
MAKED OF APATCH
^PRINT^"Cracking the sth2000,(2001,3,25)\n\n"
^PRINT^"Check the mmxado.exe....\n"
^FILE^ "mmxado.exe"
^SIZE^ 5f000
^GOTO^ e2eb
^WRITE^ 90 90
^GOTO^ e33a
^WRITE^ 90 90
^GOTO^ e341
^WRITE^ 3b c9 74
^PRINT^ "Done!\n"
^PRINT^"Check the sthsvcd.exe\n"
^FILE^ "sthsvcd.exe"
^SIZE" 9ca00
^GOTO^ 13d4d
^WRITE^ 3b c9 90 74
^GOTO^ 13db1
^WRITE^ c9 74
^GOTO^ 13ddb
^WRITE^ 3b c9 74
^GOTO^ 13d52
^WRITE^ ff 15 d4 08 4a 00 56 57 be b4 49 41 00 bf 48 7b 46 00 fc b9 22 00 00 00 f3 a4 5f 5e 90 e9 be 00 00 00
^GOTO^ 13e2c
^WRITE^ e9 21 ff ff ff 90
^GOTO^ 13db4
^WRITE^ 20 20 20 20 20 20 20 20 43 72 61 63 6b 3a d1 a9 d2 ac 20 77 6f 63 79 40 32 36 33 2e 6e 65 74 00 00
^PRINT^ "Done!\n"
^PRINT^"\nThanks.\n"
^PRINT^"Email:wocy@263.net\n"
^PRINT^"Http://wocy.yeah.net\n"
^END^
```

----------------------------------------------

6. the world clearing  `4dd466d0e1e0cc123afbe504da2462ca`
