
# 创想游戏控制软件 V1.03

- 下载页面：`http://www.skycn.com/soft/11652.html`
- 软件大小：468 KB
- 软件语言：简体中文
- 软件类别：国产软件 / 共享版 / 开关定时
- 应用平台：Win9x/NT/2000/XP
- 加入时间：2003-04-15 14:33:11
- 下载次数：368
- 推荐等级：\*\*\*
- 开 发 商：`http://www.truethink.net/`

## 软件简介

“创想游戏控制器软件”是为了防止未成年人过度沉溺于游戏之中以至耽误学习和影响身体而开发的一款软件。它主要具备两大功能。一、监控游戏的功能；二、终止游戏的功能。监控功能主要是让家长了解小孩一天倒底在电脑中玩了哪些游戏，分别玩了多久，什么时候开始玩的，什么时候退出的，等等。终止功能主要是电脑自动根据家长的设置，对游戏在玩到一定时间后进行终止，如设置今天只允许玩半个小时，则开始玩游戏半个小时后，电脑将自动关闭游戏。而且当天就不允许再玩。“创想游戏控制器软件”的推出，彻底解决了学生沉溺于电脑游戏而耽误学习和影响身体的不良弊端。

## 软件限制

功能限制。

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

TRW2000娃娃修改版、Ollydbg1.09、PEiD、UnAspacka、GUW32、W32Dasm 9.0白金版

## 破解过程

安装后主文件放在`C:\WINDOWS\SYSTEM\`目录下，`sgame.exe`。无壳，VC++ 6.0 编写。呵呵，程序比较简单，只是做完美爆破时想了会儿才搞定的。

机器码：`33050A7B`  
试炼码：`13572468`

### 主程序分析

```asm
* Reference To: MFC42.Ordinal:18BE, Ord:18BEh
 |
:00404986 E863610000              Call 0040AAEE
:0040498B 51     push ecx
:0040498C 8D8664010000   lea eax, dword ptr [esi+00000164]
:00404992 8BCC               mov ecx, esp
:00404994 89642408                mov dword ptr [esp+08], esp
:00404998 50       push eax
* Reference To: MFC42.Ordinal:0217, Ord:0217h
                       |
:00404999 E8E6610000              Call 0040AB84
:0040499E B9141D4100              mov ecx, 00411D14
:004049A3 E868FCFFFF              call 00404610
       ====>关键CALL！进入！
:004049A8 85C0                    test eax, eax
:004049AA 741D   je 004049C9
                   ====>跳则OVER！
:004049AC 6A40                    push 00000040
* Possible StringData Ref from Data Obj ->"提示信息"
 |
:004049AE 6898104100              push 00411098
* Possible StringData Ref from Data Obj ->"注册成功!"
 ====>呵呵，胜利女神！
:004049B3 689C144100             push 0041149C
:004049B8 8BCE                   mov ecx, esi
* Reference To: MFC42.Ordinal:1080, Ord:1080h
                   |
:004049BA E853610000              Call 0040AB12
:004049BF 8BCE                    mov ecx, esi
* Reference To: MFC42.Ordinal:12F5, Ord:12F5h
                     |
:004049C1 E852610000              Call 0040AB18
:004049C6 5E                      pop esi
:004049C7 59   pop ecx
:004049C8 C3         ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004049AA(C)
|
:004049C9 6A10                    push 00000010
* Possible StringData Ref from Data Obj ->"提示信息"
 |
:004049CB 6898104100              push 00411098
* Possible StringData Ref from Data Obj ->"注册失败: 注册码不正确，请检查是否输入有误。"
       ====>BAD BOY！
:004049D0 686C144100     push 0041146C
:004049D5 8BCE           mov ecx, esi
* Reference To: MFC42.Ordinal:1080, Ord:1080h
                       |
:004049D7 E836610000              Call 0040AB12
:004049DC 5E                      pop esi
:004049DD 59   pop ecx
:004049DE C3         ret
```

---

### 关键CALL分析

进入关键CALL：`004049A3 call 00404610`

```asm
* Referenced by a CALL at Address:
|:004049A3
|
:00404610 6AFF                   push FFFFFFFF
:00404612 68D8B84000              push 0040B8D8
:00404617 64A100000000            mov eax, dword ptr fs:[00000000]
:0040461D 50                      push eax
:0040461E 64892500000000          mov dword ptr fs:[00000000], esp
:00404625 51       push ecx
:00404626 53             push ebx
:00404627 8D442404               lea eax, dword ptr [esp+04]
:0040462B C744241000000000        mov [esp+10], 00000000
:00404633 50                      push eax
:00404634 E827FDFFFF              call 00404360
             ====>算法CALL！进入！
:00404639 8B00                    mov eax, dword ptr [eax]
                 ====>EAX=231C6021       注册码
:0040463B 8B4C2418                mov ecx, dword ptr [esp+18]
               ====>ECX=13572468     试炼码
:0040463F 50                      push eax
:00404640 51   push ecx
* Reference To: MSVCRT._mbscmp, Ord:0159h
                       |
:00404641 FF15A8D34000            Call dword ptr [0040D3A8]
         ====>比较CALL！
:00404647 83C408                  add esp, 00000008
:0040464A 8D4C2404 lea ecx, dword ptr [esp+04]
:0040464E 85C0           test eax, eax
   ====>爆破点 ①
:00404650 0F94C3                  sete bl
         ====>设置BL值
* Reference To: MFC42.Ordinal:0320, Ord:0320h
                       |
:00404653 E860640000              Call 0040AAB8
:00404658 84DB                    test bl, bl
:0040465A 5B     pop ebx
:0040465B 743C         je 00404699
 ====>跳则OVER！
:0040465D 8B542414                mov edx, dword ptr [esp+14]
               ====>爆破点 ②
:00404661 52                      push edx
* Possible StringData Ref from Data Obj ->"RegisterCode"
 |
:00404662 685C144100              push 0041145C
* Possible StringData Ref from Data Obj ->"TrueThink"
   |
:00404667 6858114100
 push 00411158
* Reference To: KERNEL32.WriteProfileStringA, Ord:02EDh
   |
:0040466C FF1570D04000            Call dword ptr [0040D070]
                 ====>保存注册信息！
:00404672 8D4C2414                lea ecx, dword ptr [esp+14]
:00404676 C744240CFFFFFFFF        mov [esp+0C], FFFFFFFF
* Reference To: MFC42.Ordinal:0320, Ord:0320h
                     |
:0040467E E835640000              Call 0040AAB8
:00404683 B801000000              mov eax, 00000001
         ====>置1则OK！
:00404688 8B4C2404                mov ecx, dword ptr [esp+04]
:0040468C 64890D00000000          mov dword ptr fs:[00000000], ecx
:00404693 83C410       add esp, 00000010
:00404696 C20400             ret 0004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040465B(C)
|
:00404699 8D4C2414                lea ecx, dword ptr [esp+14]
:0040469D C744240CFFFFFFFF        mov [esp+0C], FFFFFFFF
* Reference To: MFC42.Ordinal:0320, Ord:0320h
                     |
:004046A5 E80E640000              Call 0040AAB8
:004046AA 8B4C2404                mov ecx, dword ptr [esp+04]
:004046AE 33C0     xor eax, eax
                     ====>清0则OVER！
:004046B0 64890D00000000          mov dword ptr fs:[00000000], ecx
:004046B7 83C410 add esp, 00000010
:004046BA C20400       ret 0004
```

---

### 算法CALL分析

进入算法CALL：`00404634 call 00404360` 转到下面的代码处

```asm
* Possible StringData Ref from Data Obj ->"c:\"
   |
:00404389 6858144100
 push 00411458
:0040438E 895C2434
   mov dword ptr [esp+34], ebx
* Reference To: KERNEL32.GetVolumeInformationA, Ord:0177h
     |
:00404392 FF1534D04000
 Call dword ptr [0040D034]
                     ====>取硬盘序列号
:00404398 8B44240C                mov eax, dword ptr [esp+0C]
               ====>EAX=[esp+0C]=211C1E09
:0040439C 8D4C2408                lea ecx, dword ptr [esp+08]
:004043A0 3515360002 xor eax, 02003615
                 ====>EAX=211C1E09 XOR 02003615=231C281C
:004043A5 8944240C                mov dword ptr [esp+0C], eax
* Reference To: MFC42.Ordinal:021C, Ord:021Ch
                     |
:004043A9 E81C670000              Call 0040AACA
:004043AE 8B4C240C                mov ecx, dword ptr [esp+0C]
:004043B2 8D542408 lea edx, dword ptr [esp+08]
:004043B6 51             push ecx
* Possible StringData Ref from Data Obj ->"%X"
   |
:004043B7 6854144100
 push 00411454
:004043BC 52         push edx
:004043BD C744242C01000000
     mov [esp+2C], 00000001
* Reference To: MFC42.Ordinal:0B02, Ord:0B02h
                       |
:004043C5 E856680000              Call 0040AC20
:004043CA 8B4C2414                mov ecx, dword ptr [esp+14]
:004043CE 8B442418 mov eax, dword ptr [esp+18]
:004043D2 83C40C         add esp, 0000000C
:004043D5 89442410             mov dword ptr [esp+10], eax
         ====>[esp+10]=EAX=231C281C
:004043D9 8B71F8                  mov esi, dword ptr [ecx-08]
                 ====>ESI=8   取231C281C长度
:004043DC 3BF3                    cmp esi, ebx
:004043DE 7E4C   jle 0040442C
:004043E0 55
```
