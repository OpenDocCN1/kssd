
# HomeWatcher v1.2

> 原文：[`www.pediy.com/kssd/pediy01/forum018.htm`](https://www.pediy.com/kssd/pediy01/forum018.htm)

最先最先得感谢 `TRW2000` 的作者朱楠灏和刘涛涛。

## 破解 HomeWatcher v1.2 第一回合（转载希望保持完整）

级别：入门级-----前辈们请指点。

作者：丁丁虾 又名：`DDXia` `[CCG]`

软件名称：`HomeWatcher`

整理日期：2000.2.27

最新版本：`1.2 Beta 9`

文件大小：`968KB`

软件授权：共享软件

使用平台：`Win95/98/NT`

发布公司：`Home Page`

软件简介：能够定时拍照，能在景物发生变化时将照片保存下来，然后制作成网页，连接到 `internet`，再通过 `FTP` 将照片上传。

`http://www.newhua.com/down/hw12.exe`

### 第一回合较量

用 `TRW2000`，这个软件比较 `COOL`。用了它就让我有难舍难分的感觉。尤其是它的 `PMODULE`，令我感慨无限，当然还有许多优点。

1、load `HomeWatcher.exe`

2、在 `Full Version` 中填入注册码 `12345678`(任意)。

3、`bpx hmemcpy`－－－－中断读写内存

4、按确定，就会被中断

5、`pmodule`

6、按 `F10`,一直到出现“`invalid key`"

7、瞧一瞧

```
015F:004132B8  CALL    `USER32!MessageBoxA`
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^------------->出现错误窗口 

015F:004132BD  MOV     Byte Ptr [ESP+70],00
015F:004132C2  MOV     Byte Ptr [ESP+00000084],00
```

8、再往上瞧一瞧，都说了看人先看个大慨，然后再仔细看个究竟。

```
015F:00413284  CALL    0047B320
015F:00413289  POP     ECX
015F:0041328A  CMP     EAX,00000011
015F:0041328D  JNZ     004132A2
                  ^^^^^^^^^^^^^^^^-------------------->这是出现错误窗口的跳转

                    试一试，改为 `JZ`     结果可以自己试 

015F:0041328F  LEA     EDX,[ESP+00000084] 
015F:00413296  PUSH    EDX
015F:00413297  CALL    0047B320
015F:0041329C  POP     ECX
015F:0041329D  CMP     EAX,00000010
015F:004132A0  JZ      004132CC
                  ^^^^^^^^^^^^^^^^-------------------->看！还有一个跳转，对，对也要改为 `JNZ`！ 

015F:004132A2  PUSH    00000030
015F:004132A4  PUSH    004A1E35
015F:004132A9  PUSH    004A1E28
015F:004132AE  MOV     EAX,[ESP+0C]
015F:004132B2  CALL    0045F960
015F:004132B7  PUSH    EAX
015F:004132B8  CALL    `USER32!MessageBoxA`
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^------------->出现错误窗口 

015F:004132BD  MOV     Byte Ptr [ESP+70],00
015F:004132C2  MOV     Byte Ptr [ESP+00000084],00
015F:004132CA  JMP     00413304
015F:004132CC  PUSH    004A1E61
015F:004132D1  PUSH    004A1E41
015F:004132D6  LEA     EDX,[ESP+000000A0]
015F:004132DD  PUSH    EDX
015F:004132DE  CALL    0047D430
015F:004132E3  ADD     ESP,0000000C
015F:004132E6  PUSH    00000030
015F:004132E8  PUSH    004A1E6D
015F:004132ED  LEA     ECX,[ESP+000000A0]
015F:004132F4  PUSH    ECX
015F:004132F5  MOV     EAX,[ESP+0C]
015F:004132F9  CALL    0045F960
015F:004132FE  PUSH    EAX
015F:004132FF  CALL    `USER32!MessageBoxA`
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^-------------->这里也出现一个窗口，去瞧瞧。 

015F:00413304  LEA     EDX,[ESP+70]
015F:00413308  PUSH    EDX
```

（快、快，肯定是 `THANK YOU`！）噢！`MY GOD` 当场晕倒。“`restar HomeWatcher to apply the changes`”

(中文：重新启动 `HomeWatcher` 再改变版权）天啊！看来程序虽小，还真的放手严密！

想想还有什么招数！------反正我是饿了！是绝对的想不出来哦！

（待续）

完成时间：`2000.2.27` 中午

## 破解 HomeWatcher v1.2 第二回合（转载希望保持完整）

级别：入门级-----前辈们请指点。

作者：丁丁虾 又名：`DDXia` `[CCG]`

软件名称：`HomeWatcher`

整理日期：2000.2.27

最新版本：`1.2 Beta 9`

文件大小：`968KB`

软件授权：共享软件

使用平台：`Win95/98/NT`

发布公司：`Home Page`

软件简介：能够定时拍照，能在景物发生变化时将照片保存下来，然后制作成网页，连接到 `internet`，再通过 `FTP` 将照片上传。

`http://www.newhua.com/down/hw12.exe`

### 第二回合较量

用 `TRW2000`，看一看能否可以 `TRACE` 出它的注册码，当然我个人感觉有一点麻麻地（广东话）。`Do my best`！

再通看一遍程序（在出现错误窗口的时候）

```
015F:00413250  MOV     CL,[EBX]
015F:00413252  MOV     [EDI],CL
015F:00413254  INC     DWord Ptr [ESP+08]
015F:00413258  INC     EDI
015F:00413259  INC     ESI
015F:0041325A  INC     EBX
015F:0041325B  LEA     EAX,[ESP+0C]
015F:0041325F  PUSH    EAX
015F:00413260  CALL    0047B320
                  ^^^^^^^^^^^^^^^^-------------------------->注意这个 `CALL` 号 

015F:00413265  POP     ECX
015F:00413266  CMP     EAX,ESI
015F:00413268  JA      00413232
015F:0041326A  MOV     EDX,[ESP+04]
015F:0041326E  LEA     EAX,[ESP+70]
015F:00413272  MOV     Byte Ptr [ESP+1*EDX+70],00
015F:00413277  MOV     ECX,[ESP+08]
015F:0041327B  MOV     Byte Ptr [ESP+1*ECX+00000084],00
015F:00413283  PUSH    EAX
015F:00413284  CALL    0047B320
                  ^^^^^^^^^^^^^^^^-------------------------->又注意这个 `CALL` 号 

015F:00413289  POP     ECX
015F:0041328A  CMP     EAX,00000011
015F:0041328D  JNZ     004132A2----------->不等，`KEY` 就非法 

015F:0041328F  LEA     EDX,[ESP+00000084]
015F:00413296  PUSH    EDX
015F:00413297  CALL    0047B320
                  ^^^^^^^^^^^^^^^^-------------------------->又再注意这个 `CALL` 号 

                                   看来这软件的作者的 `CALL` 号一定是 `47B320` 
                                   记得哦，去了美国，别忘了 `CALL` 他^_^ 
                                   一定有鬼，去捉鬼好不好！可要带好武器喔！ 

015F:0041329C  POP     ECX
015F:0041329D  CMP     EAX,00000010
015F:004132A0  JZ      004132CC
015F:004132A2  PUSH    00000030
015F:004132A4  PUSH    004A1E35
015F:004132A9  PUSH    004A1E28
015F:004132AE  MOV     EAX,[ESP+0C]
015F:004132B2  CALL    0045F960
015F:004132B7  PUSH    EAX
015F:004132B8  CALL    `USER32!MessageBoxA`------出错窗口 
```

想想清楚罗、（远处有一个鬼屋）开了门，就不要往回走罗！好！`GO GO` 按 `F8`，进入 `call 47b320`

好象都是黑乎乎的代码！

```
015F:0047B320  MOV     EAX,[ESP+04]------->`EAX` 指向存 `KEY` 的地方 用 `D EAX` 可以看到注册码 
015F:0047B324  TEST    AL,03
015F:0047B326  JNZ     0047B355
015F:0047B328  MOV     EDX,[EAX]---------->把前四位，付给 `EDX`，我的是 `1234`。不过你看到是 `HEX` 
015F:0047B32A  ADD     EAX,00000004------->移动 `KEY` 的指针，到下四位。 
015F:0047B32D  MOV     ECX,EDX
015F:0047B32F  SUB     EDX,01010101------->为什么都减 1 呢??我也不明白. 
015F:0047B335  AND     EDX,80808080------->为什么都 `AND 80` 呢??? 
015F:0047B33B  JZ      0047B328
015F:0047B33D  NOT     ECX         -------> 为什么?? 
015F:0047B33F  AND     EDX,ECX     --------> 为什么?? 
```

经过反复多次的输入 `KEY` 值，跟踪、跟踪、再跟踪。我先说这段程序作用-----计算 `EAX` 指向的字符串的长度。

这段的程序挺有趣的。可以分析一下。

几点 `TIPS`：

1、`HEX-01` 是判断当没有输入值的时候 `HEX` 为 `00`

2、`AND 80` 因为能输入的字符的 `HEX` 都是小于 `80` 的

3、`HEX` 是输入的键盘的 16 进制值，比如：`'1'`--->`HEX` 为 `31`。

```
015F:0047B341  JZ      0047B328
015F:0047B343  TEST    DL,DL
015F:0047B345  JNZ     0047B36F
015F:0047B347  TEST    DH,DH
015F:0047B349  JNZ     0047B370
015F:0047B34B  TEST    EDX,00FF0000
015F:0047B351  JNZ     0047B371
015F:0047B353  JMP     0047B372
015F:0047B355  ADD     EAX,00000004
015F:0047B358  TEST    Byte Ptr [EAX-04],FF
015F:0047B35C  JZ      0047B36F
015F:0047B35E  TEST    Byte Ptr [EAX-03],FF
015F:0047B362  JZ      0047B370
015F:0047B364  TEST    Byte Ptr [EAX-02],FF
015F:0047B368  JZ      0047B371
015F:0047B36A  DEC     EAX
015F:0047B36B  AND     AL,FC
015F:0047B36D  JMP     0047B328
015F:0047B36F  DEC     EAX
015F:0047B370  DEC     EAX
015F:0047B371  DEC     EAX
015F:0047B372  MOV     ECX,[ESP+04]
015F:0047B376  DEC     EAX
015F:0047B377  SUB     EAX,ECX  ------->得出 `Key` 的长度,确切的说是 `EAX` 指向的字符串的长度 
015F:0047B379  RET
```

既然读懂了子程序，再跳回来看它的程序。

```
015F:00413284  CALL    0047B320
                  ^^^^^^^^^^^^^^^^ 

015F:00413289  POP     ECX
015F:0041328A  CMP     EAX,00000011
                           ^^^^^^^^------->看来是 11 位的，错应是 `0X11` 

015F:0041328D  JNZ     004132A2----------->不等，`KEY` 就说是非法 

015F:0041328F  LEA     EDX,[ESP+00000084]
```

看来 `KEY` 是任意 17 位，`YEAH`!!!

试一试，噢，又是 `Invalid`。

再看(眼睛好痛啊!!!)

```
015F:00413250  MOV     CL,[EBX]
015F:00413252  MOV     [EDI],CL
015F:00413254  INC     DWord Ptr [ESP+08]
015F:00413258  INC     EDI
015F:00413259  INC     ESI
015F:0041325A  INC     EBX
015F:0041325B  LEA     EAX,[ESP+0C]
015F:0041325F  PUSH    EAX
015F:00413260  CALL    0047B320
                  ^^^^^^^^^^^^^^^^ 

015F:00413265  POP     ECX
015F:00413266  CMP     EAX,ESI
```



# 破解 HomeWatcher v1.2 终结版

**级别**：入门级——前辈们请指点。  
**作者**：丁丁虾 又名：`DDXia [CCG]`  
**软件名称**：`HomeWatcher`  
**整理日期**：2000.2.27  
**最新版本**：1.2 Beta 9  
**文件大小**：968KB  
**软件授权**：共享软件  
**使用平台**：Win95/98/NT  
**发布公司**：Home Page  

## 软件简介
能够定时拍照，能在景物发生变化时将照片保存下来，然后制作成网页，连接到 Internet，再通过 `FTP` 将照片上传。

下载链接：`http://www.newhua.com/down/hw12.exe`

## 大结局
风萧萧兮易水寒，壮士一去。。。。。（夕阳下，荒芜的沙漠，留下一步一行代码的脚印，延伸到远方）

苦苦追寻三天三夜，不辞劳苦，苦中作乐。`YEAH`。

此程序很小，但它的注册方式，正趋于流行，就是在填注册码时，有一个判断，但不会判断它的对错，然后重新启动过程中，任意的一个地方，再进行判断，所以难就难在找到那个位置。

### [联想]
不过古人很早就说过：不入虎穴焉得虎子。想一想，其实古人很早就有 `CRACK` 的思想指导了，只不过由于当时的条件限制，说不定有甲骨文写的 `CRACK` 教学。

## 破解工具
*   `REGMONITOR` —— 注册表监视器
*   `TRW2000` —— 中国人的极品
*   `W32DASM` —— 和 `TRW2000` 一样等级

### [联想]
准备进入战斗，变形。。。（卡）`REGMONITOR` 变成我护目镜。唰、唰 `TRW2000` 和 `W32DASM` 变成我的左臂右膀。`YEAH`。变形完毕。

填写完注册码后，运行 `REGMONITOR`，单击“确定”，哈哈、千里眼报告前方有两碉堡：

*   `HKCU\Software\HomeWatcher.Com\SecuCam\AltA`
*   `HKCU\Software\HomeWatcher.Com\SecuCam\DetA`

这两个键值分别储存注册码的奇数位和偶数位，同时也加密了。

想一想，如果它要判断注册码是否正确，一定会再次读出来进行判断，这是必然的，如何拦截呢？？？

`TRW2000` 动态拦截读注册表 `BPX RegqueryValueEXA`，这个主意不错。唰、弹出 `TRW2000`，`pmodule` 就到读 `DetA` 的地方。

## 代码分析

为了便于理解，以下代码从 `W32DASM` 反编译中截取：

```assembly
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405542(C)
|
:0040554E 8D95B4FEFFFF         lea edx, dword ptr [ebp+FFFFFEB4]
:00405554 52                   push edx
:00405555 8D8B0E120000         lea ecx, dword ptr [ebx+0000120E]
:0040555B 51                   push ecx
:0040555C 6A00                 push 00000000
:0040555E 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"DetA"
                                      |
:00405560 682BDE4900               push 0049DE2B---------》存放字符串的指向
:00405565 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:0040556B 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:0040556C E8436A0900               Call 0049BFB4---------》读注册表
:00405571 8D5588                   lea edx, dword ptr [ebp-78]
:00405574 52                       push edx
:00405575 8D8BB8030000             lea ecx, dword ptr [ebx+000003B8]
:0040557B 51                       push ecx
:0040557C 6A00                     push 00000000
:0040557E 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"Host"
                                      |
:00405580 6830DE4900               push 0049DE30
:00405585 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:0040558B 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:0040558C E8236A0900               Call 0049BFB4
:00405591 8D5584                   lea edx, dword ptr [ebp-7C]
:00405594 52                       push edx
:00405595 8D8B80040000             lea ecx, dword ptr [ebx+00000480]
:0040559B 51                       push ecx
:0040559C 6A00                     push 00000000
:0040559E 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"Port"
                                      |
:004055A0 6835DE4900               push 0049DE35
:004055A5 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:004055AB 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:004055AC E8036A0900               Call 0049BFB4
:004055B1 8D5580                   lea edx, dword ptr [ebp-80]
:004055B4 52                       push edx
:004055B5 8D8B8A040000             lea ecx, dword ptr [ebx+0000048A]
:004055BB 51                       push ecx
:004055BC 6A00                     push 00000000
:004055BE 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"User"
                                      |
:004055C0 683ADE4900               push 0049DE3A
:004055C5 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:004055CB 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:004055CC E8E3690900               Call 0049BFB4
:004055D1 8D957CFFFFFF             lea edx, dword ptr [ebp+FFFFFF7C]
:004055D7 52                       push edx
:004055D8 8D8BDA040000             lea ecx, dword ptr [ebx+000004DA]
:004055DE 51                       push ecx
:004055DF 6A00                     push 00000000
:004055E1 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"Password"
                                      |
:004055E3 683FDE4900               push 0049DE3F
:004055E8 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:004055EE 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:004055EF E8C0690900               Call 0049BFB4
:004055F4 8D9578FFFFFF             lea edx, dword ptr [ebp+FFFFFF78]
:004055FA 52                       push edx
:004055FB 8D8B2A050000             lea ecx, dword ptr [ebx+0000052A]
:00405601 51                       push ecx
:00405602 6A00                     push 00000000
:00405604 6A00                     push 00000000

* Possible StringData Ref from Data Obj ->"Pasv"
                                      |
:00405606 6848DE4900               push 0049DE48
:0040560B 8B83941D0000             mov eax, dword ptr [ebx+00001D94]
:00405611 50                       push eax

* Reference To: ADVAPI32.RegQueryValueExA, Ord:0000h
                                      |
:00405612 E89D690900               Call 0049BFB4
:00405617 8D9574FFFFFF             lea edx, dword ptr [ebp+FFFFFF74]
:0040561D 52                       push edx
:0040561E 8D8B34050000             lea ecx, dword ptr [ebx+00000534]
:00405624 51                       push ecx
:00405625 6A00                     push 00000000
:00405627 6A00                     push 00000000
```

以上程序是取键入 `KEY` 值的奇数位，很容易的，自己去折磨一下，我现在手好累。

```assembly
015F:00413268  JA       00413232

015F:0041326A  MOV      EDX,[ESP+04]

015F:0041326E  LEA      EAX,[ESP+70]

015F:00413272  MOV      Byte Ptr [ESP+1*EDX+70],00

015F:00413277  MOV      ECX,[ESP+08]

015F:0041327B  MOV      Byte Ptr [ESP+1*ECX+00000084],00

015F:00413283  PUSH     EAX

015F:00413284  CALL     0047B320
                   ^^^^^^^^^^^^^^^^

015F:00413289  POP      ECX

015F:0041328A  CMP      EAX,00000011
                        ^^^^^^^^^^^^---------->因此 KEY 值是 OX11*2=0X22=34(位)

                        计算真确，加 10000 分：）

                        为了好计算我输入 1234567891234567891234567891234567（根据个人爱好）

015F:0041328D  JNZ      004132A2----------->不等，KEY 就非法

015F:0041328F  LEA      EDX,[ESP+00000084]

015F:00413296  PUSH     EDX

015F:00413297  CALL     0047B320
                   ^^^^^^^^^^^^^^^^---------------

015F:0041329C  POP      ECX

015F:0041329D  CMP      EAX,00000010

015F:004132A0  JZ       004132CC
```

填好后，按确定出现“restar HomeWatcher to apply the changes”，重新启动 `HomeWatcher`，瞧一瞧。

好像还是不行。没有关系，`CRACK` 的精神就是一追到底，誓不休。说的远一点，可以培养每个人的坚定不移的斗志。即使不作 `CRACKER`，追女孩法则之一也是一追到底喔！：））

希望有些收获，我可不管这些，反正，我功力可是增加了 3 点!!! 现在经验值是 23 点^_^

（待续）

完成时间：2000.2.28 23:08  
费时 3 个小时

## 致谢
最先最先得感谢 `TRW2000` 的作者朱楠灏和刘涛涛。

特别感谢：看雪、`COOLFLY`、八爪鱼、`SUN BIRD`、`JOJO`、洋白菜、`JACK`------给我极大的精神支持和非常建议。（各家的经典经验，在文中都提到的哦！^_^)



# 堆栈操作与注册表监控

读出后，并不立刻进行比较及判断，只是存在堆栈中。而且还不停的读注册表（`REGMONITOR` 在它启动的时候，可以看到）。这样做的好处就是你对它还不死心。

一直按 `F10`（本人有一点酱，所以不服它）。

看到两段比较有趣的代码：

## 第一段：重新组装代码

重新组装带代码，我倒，它也会变形。

```asm
:00407ED6 8B93B0030000               mov edx, dword ptr [ebx+000003B0]
                                     ; 存放 "AltA" key 的指向

:00407EDC 8B85F4FCFFFF               mov eax, dword ptr [ebp+FFFFFCF4]
                                     ; 存放重新组装的地址

:00407EE2 47                          inc edi
:00407EE3 47                          inc edi

:00407EE4 8A4C32FF                   mov cl, byte ptr [edx+esi-01]
                                     ; 第一次 ESI=01 取 "AltA" 的第一位

:00407EE8 8808                         mov byte ptr [eax], cl
:00407EEA FF85F4FCFFFF               inc dword ptr [ebp+FFFFFCF4]
                                     ; 存放重新组装的地址下一位

:00407EF0 8B93B4030000               mov edx, dword ptr [ebx+000003B4]
:00407EF6 8B85F4FCFFFF               mov eax, dword ptr [ebp+FFFFFCF4]

:00407EFC 8A4C32FF                   mov cl, byte ptr [edx+esi-01]
                                     ; 第一次 ESI=01 取 "DetA" 的第一位

:00407F00 8808                         mov byte ptr [eax], cl
:00407F02 FF85F4FCFFFF               inc dword ptr [ebp+FFFFFCF4]
:00407F08 46                           inc esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407ED4(U)
|
:00407F09 8B93B0030000               mov edx, dword ptr [ebx+000003B0]
                                     ; 存放 "AltA" key 的指向

:00407F0F 4A                           dec edx
:00407F10 52                           push edx
:00407F11 E80A340700                   call 0047B320 ; 取字符串长度
:00407F16 59                           pop ecx
:00407F17 50                           push eax

:00407F18 8B8BB4030000               mov ecx, dword ptr [ebx+000003B4]
                                     ; 存放 "DetA" key 的指向

:00407F1E 49                           dec ecx
:00407F1F 51                           push ecx
:00407F20 E8FB330700                   call 0047B320
:00407F25 59                           pop ecx
:00407F26 5A                           pop edx
:00407F27 03D0                         add edx, eax
:00407F29 3BD6                         cmp edx, esi
:00407F2B 77A9                         ja 00407ED6
:00407F2D C6843DDCF9FFFF00             mov byte ptr [ebp+edi-00000624], 00
```

## 第二段：对照密码表转换

对照密码表转换。（具体的 `ESI`、`EDI` 的值实在是记不起来了，`EVERYBODY` 都可以 `DOWNLOAD` 它进行实习，我写的和破的软件都很新，终究一句话：修炼在个人）

```asm
015F:0047F0C8  PUSH    EBP
015F:0047F0C9  MOV     EBP,ESP
015F:0047F0CB  PUSH    EBX
015F:0047F0CC  PUSH    ESI
015F:0047F0CD  PUSH    EDI
015F:0047F0CE  XOR     EDI,EDI
015F:0047F0D0  MOV     ESI,[EBP+08]
015F:0047F0D3  MOV     BL,[ESI]
015F:0047F0D5  INC     ESI
015F:0047F0D6  MOVSX   EAX,BL
015F:0047F0D9  PUSH    EAX
015F:0047F0DA  CALL    00480DAC ; 进行转换（CALL 的代码接着就是）
015F:0047F0DF  POP     ECX
015F:0047F0E0  TEST    EAX,EAX
015F:0047F0E2  JNZ     0047F0D3
015F:0047F0E4  CMP     BL,2B ; 为什么是比较 2B、2D？？
015F:0047F0E7  JZ      0047F0EE
015F:0047F0E9  CMP     BL,2D ; 是因为转换后的值都是小于等于 30
015F:0047F0EC  JNZ     0047F0FC
015F:0047F0EE  CMP     BL,2D ; 至少我的是
015F:0047F0F1  SETZ    AL
015F:0047F0F4  AND     EAX,00000001
015F:0047F0F7  MOV     BL,[ESI]
015F:0047F0F9  INC     ESI
015F:0047F0FA  JMP     0047F114
015F:0047F0FC  XOR     EAX,EAX
015F:0047F0FE  JMP     0047F114
015F:0047F100  MOVSX   ECX,BL
015F:0047F103  MOV     EDX,EDI
015F:0047F105  MOV     BL,[ESI]
015F:0047F107  ADD     EDX,EDX
015F:0047F109  LEA     EDX,[EDX+4*EDX]
015F:0047F10C  ADD     EDX,ECX
015F:0047F10E  ADD     EDX,FFFFFFD0
015F:0047F111  INC     ESI
015F:0047F112  MOV     EDI,EDX
015F:0047F114  CMP     BL,30
015F:0047F117  JL      0047F11E
015F:0047F119  CMP     BL,39
015F:0047F11C  JLE     0047F100
015F:0047F11E  TEST    EAX,EAX
015F:0047F120  JZ      0047F128
015F:0047F122  MOV     EAX,EDI
015F:0047F124  NEG     EAX
015F:0047F126  JMP     0047F12A
015F:0047F128  MOV     EAX,EDI
015F:0047F12A  POP     EDI
015F:0047F12B  POP     ESI
015F:0047F12C  POP     EBX
015F:0047F12D  POP     EBP
015F:0047F12E  RET
```

### `CALL 00480DAC` 的代码

```asm
015F:00480DAC  PUSH    EBP
015F:00480DAD  MOV     EBP,ESP
015F:00480DAF  MOV     EAX,[EBP+08]
015F:00480DB2  CMP     EAX,000000FF
015F:00480DB7  JNA     00480DBD
015F:00480DB9  XOR     EAX,EAX
015F:00480DBB  POP     EBP
015F:00480DBC  RET
; (015F:00480DAB RET 为前序或异常分支，此处略去或保持原样)
015F:00480DBD  MOVZX   EDX,Word Ptr [004A883A+2*EAX]
                ; 这里就代码转换入口
015F:00480DC5  AND     EDX,00000008
015F:00480DC8  MOV     EAX,EDX
015F:00480DCA  POP     EBP
015F:00480DCB  RET
```

## 调试过程与社区互动

接着就被程序的 `CALL`----`RET`----`CALL`----`RET`----`JMP`----`CALL`----`RET` 弄得迷迷糊糊了两天两夜，工作有点烦。

与此同时，开始和文章开头的各位大虾（`看雪`、`COOLFLY`、`八爪鱼`、`SUN BIRD`、`JOJO`、`洋白菜`、`JACK`）“大呼小叫”，无论谁都回应，提出好的建议和鼓励。一人有难，各路武林门派纷纷----几道寒光 刀出鞘

* `看雪`兄：试用 `W32DASM` 静态分析
* `COOLFLY` 兄：拦截后，用 `F12` 进行消短（`SOFT ICE`）
* `八爪鱼`兄：工作忙得浇头滥额，还答应帮俺看。
* `SUN BIRD`、`JOJO` 兄：同一武林门派，那还分谁和谁啊！
* `洋白菜`兄：多看一些外国的破文，更易提高功力级别。比如`菠萝密经`
* `JACK` 兄：精神鼓励更重要----斗志比剑气更重要

> 于是采用[动、静结合，大处入手]
>
> 用 `W32DASM` 反编译后，再全部过一遍程序，从读取 `AltA` 和 `DetA` 值后，往后看是否有可疑的地方。
>
> 找阿找 找阿找 找阿找。。。。。。
>
> "Start of program"----程序开始，一定会有比较的：
>
> * Possible StringData Ref from Data Obj ->"Start of program"
>
> ```asm
> :00407FF4 6864E54900               push 0049E564
> :00407FF9 53                        push ebx
> :00407FFA E8D1A7FFFF               call 004027D0
> :00407FFF 83C408                   add esp, 00000008
> :00408002 6A05                      push 00000005
> :00408004 8D8DEAF9FFFF             lea ecx, dword ptr [ebp+FFFFF9EA]
> :0040800A 51                       push ecx
> :0040800B 8D83B5160000             lea eax, dword ptr [ebx+000016B5]
> :00408011 50                       push eax
> :00408012 E85D350700               call 0047B574
> :00408017 83C40C                   add esp, 0000000C
> :0040801A C683BA16000000           mov byte ptr [ebx+000016BA], 00
> :00408021 6A06                      push 00000006
> :00408023 8D95EFF9FFFF             lea edx, dword ptr [ebp+FFFFF9EF]
> :00408029 52                       push edx
> :0040802A 8D8BBB160000             lea ecx, dword ptr [ebx+000016BB]
> :00408030 51                       push ecx
> :00408031 E83E350700               call 0047B574
> :00408036 83C40C                   add esp, 0000000C
> :00408039 C683C116000000           mov byte ptr [ebx+000016C1], 00
> :00408040 B101                     mov cl, 01
> :00408042 B201                     mov dl, 01
> :00408044 A12C5C4A00               mov eax, dword ptr [004A5C2C]
> :00408049 E8BE760100               call 0041F70C
> :0040804E 8983001F0000             mov dword ptr [ebx+00001F00], eax
> :00408054 6A05                      push 00000005
> :00408056 8D85F5F9FFFF             lea eax, dword ptr [ebp+FFFFF9F5]
> :0040805C 50                       push eax
> ```



```
> :0040805D 8D93C2160000             lea edx, dword ptr [ebx+000016C2]
> :00408063 52                       push edx
> :00408064 E80B350700               call 0047B574
> :00408069 83C40C                   add esp, 0000000C
> :0040806C C683C716000000           mov byte ptr [ebx+000016C7], 00
> :00408073 6A03                     push 00000003
> :00408075 8D8DFAF9FFFF             lea ecx, dword ptr [ebp+FFFFF9FA]
> :0040807B 51                       push ecx
> :0040807C 8D83C8160000             lea eax, dword ptr [ebx+000016C8]
> :00408082 50                       push eax
> :00408083 E8EC340700               call 0047B574
> :00408088 C683CB16000000           mov byte ptr [ebx+000016CB], 00
> :0040808F 83C40C                   add esp, 0000000C
> :00408092 8D93A6160000             lea edx, dword ptr [ebx+000016A6]
> :00408098 8995FCFCFFFF             mov dword ptr [ebp+FFFFFCFC], edx
> :0040809E 8B8DFCFCFFFF             mov ecx, dword ptr [ebp+FFFFFCFC]
> :004080A4 51                       push ecx
> :004080A5 E81E700700               call 0047F0C8
> :004080AA 59                       pop ecx
> :004080AB 8BF0                     mov esi, eax
> :004080AD 8D83B5160000             lea eax, dword ptr [ebx+000016B5]
> :004080B3 8985F8FCFFFF             mov dword ptr [ebp+FFFFFCF8], eax
> :004080B9 8B95F8FCFFFF             mov edx, dword ptr [ebp+FFFFFCF8]
> :004080BF 52                       push edx
> :004080C0 E803700700               call 0047F0C8
> :004080C5 0FAFF0                   imul esi, eax
> :004080C8 8BC6                     mov eax, esi
> :004080CA 59                       pop ecx
> :004080CB 99                       cdq
> :004080CC B968030000               mov ecx, 00000368
> :004080D1 F7F9                     idiv ecx
> :004080D3 899500FDFFFF             mov dword ptr [ebp+FFFFFD00], edx
> :004080D9 8B8500FDFFFF             mov eax, dword ptr [ebp+FFFFFD00]
> :004080DF 99                       cdq
> :004080E0 33C2                     xor eax, edx
> :004080E2 2BC2                     sub eax, edx
> :004080E4 8BF0                     mov esi, eax
> :004080E6 8D83C8160000             lea eax, dword ptr [ebx+000016C8]
> :004080EC 898504FDFFFF             mov dword ptr [ebp+FFFFFD04], eax
> :004080F2 83C613                   add esi, 00000013
> :004080F5 8B9504FDFFFF             mov edx, dword ptr [ebp+FFFFFD04]
> :004080FB 52                       push edx
> :004080FC E8C76F0700               call 0047F0C8
> :00408101 59                       pop ecx
> :00408102 3BF0                     cmp esi, eax
>                     ^^^^^^^^^^^^^^
>                     非常非常非常的可疑
> :00408104 750C                     jne 00408112
>                     ^^^^^^^^^^^^^
>                     用 TRW2000 改为 jn 如何，希望没有见意。
>                     YEAH。。。YEAH。。。。。激动跳起来（老兄别人还在睡觉）
>                     不管了，先来一段 DISK 何如？？武曲呢？谁的眼泪在飞。
>                     导演兼主角：丁丁虾 （现实中没有办法实现，但我们可以有梦嘛^\_^)
> :00408106 C783E41D000001000000     mov dword ptr [ebx+00001DE4], 00000001
> :00408110 EB08                     jmp 0040811A
> 好想睡觉！啊！！！床 床在哪里？？？？？
> 决定要好好休息一个星期再破再写了。3 月 17、18 日再见吧！！！！
> 感谢那些鼓励我的人！！！
> 衷心希望每个人都有一个好梦 ！！：）））
> 完成时间
> 2000.3.7 22:37
> 费时 3 天 3 夜
```

使用的版本为`HomeWatcher 1.2 beta 9`。

若`[ebx+00001DE4]=1`且`[ebx+00001DE8]=1`，程序就是`Full Version`。

只须改动一个字节

```
:00408102 3BF0                     cmp esi, eax
:00408104 750C                     jne 00408112 <- 改 jne 为 je
:00408106 C783E41D000001000000     mov dword ptr [ebx+00001DE4], 00000001
:00408110 EB08                     jmp 0040811A
```

此后，程序会找`HomeWatcher.lic`文件。而无论是否找到，当`[ebx+00001DE4]=1`且`[ebx+00001DE8]=1`时则为`Full`；仅`[ebx+00001DE8]=1`为`Lite+`；否则为`Lite`。而事实上我们根本不用`.lic`文件，因为程序总会走到

```
:004081E5 83BBE41D000000           cmp dword ptr [ebx+00001DE4], 00000000
:004081EC 7424                      je 00408212 <- 因这里不会跳走,所以...
:004081EE C783E81D000001000000     mov dword ptr [ebx+00001DE8], 00000001
:004081F8 33D2                      xor edx, edx
```

不过我没法试程序是否真的`100%`破解了，只有试了才可肯定可以不用`.lic`文件。
