
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
