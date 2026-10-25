
# 简单算法――拼音之星 2002 V1.5

简单算法――拼音之星 2002 V1.5

- 下载地址：`http://218.30.21.125:8084/chinese/inputmethod/pinyin/p2k2p15.zip`
- 适用平台：Win9x、NT、ME、Win2k、WinXP
- 软件类型：共享
- 收录时间：2002-3-14
- 软件大小：1670KB
- 推荐等级：\*\*\*

## 软件简介

基于汉语拼音方法实现（简体、繁体、`GB/GBK/BIG5`）单字、词语、句子的高速输入，包括全拼、双拼、谭码和简拼等多种输入形式。

作为输入法中的佼佼者，拼音之星给广大用户提供了一种快速输入汉字的简便方法，独创的很多功能为其他输入法所仿效。

★新版2002V1.5增加以下功能：

1. 增加“非句输入模式”中修改任意位置字词功能
2. 增加`Ctrl/Shift`切换中西文选择功能
3. 增加输入行颜色、字体颜色、第一个重码颜色、加粗的用户自定义
4. 增加用快捷键设置输入行颜色、大小
5. 增加导航条全角/半角、中英文标点选择按钮
6. 取消输入过程中任务栏中标题显示
7. 增加词库导入导出
8. 增加用户注册码功能
9. 增加双拼输入时显示全拼的选项
10. 增加全角空格输入功能
11. 增加智能识别标点和英文符号的选项

## 软件限制

30天试用

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

`TRW2000`娃娃修改版、`Ollydbg1.09`、`PEiD`、`W32Dasm 9.0`白金版

## 过程

呵呵，sunboy 朋友已经追出了注册码，我就沾点光，找找算法吧。

`TAIZI32.EXE` 无壳。`Visual C++ 5.0` 编写。反汇编，很容易就找到核心了。

用户码：893503481

试炼码：13572468

---

```asm
:0040FA26 C745FC16016619          mov [ebp-04], 19660116

         ====>[ebp-04]＝19660116

:0040FA2D C745F801049019          mov [ebp-08], 19900401

       ====>[ebp-08]＝19900401

:0040FA34 C745F0DBAC5713          mov [ebp-10], 1357ACDB

       ====>[ebp-10]＝1357ACDB

:0040FA3B C745EC00000000          mov [ebp-14], 00000000

:0040FA42 6A0C                    push 0000000C

:0040FA44 6A00                    push 00000000

:0040FA46 6A00                    push 00000000

:0040FA48 6A00                    push 00000000

:0040FA4A 8D45AC                  lea eax, dword ptr [ebp-54]

:0040FA4D 50                      push eax

:0040FA4E 6A0C                    push 0000000C

:0040FA50 6A00                    push 00000000

\*
Possible StringData Ref from Data Obj ->"c:\"
   |
:0040FA52 6814944300              push 00439414

\*
Reference To: KERNEL32.GetVolumeInformationA, Ord:014Fh
     |
:0040FA57 FF1598434900            Call dword ptr [00494398]

                     ====>取我的硬盘序列号

:0040FA5D 8B4DAC                  mov ecx, dword ptr [ebp-54]

                 ====>ECX＝211C1E09
         硬盘序列号

:0040FA60 894DEC                  mov dword ptr [ebp-14], ecx

:0040FA63 8B55EC                  mov edx, dword ptr [ebp-14]

:0040FA66 3355FC                  xor edx, dword ptr [ebp-04]

         ====>EDX＝211C1E09 XOR 19660116=387A1F1F

:0040FA69 8955EC                  mov dword ptr [ebp-14], edx

:0040FA6C 8B45EC                  mov eax, dword ptr [ebp-14]

:0040FA6F 3345F8                  xor eax, dword ptr [ebp-08]

         ====>EAX＝387A1F1F XOR 19900401=21EA1B1E

:0040FA72 8945EC                  mov dword ptr [ebp-14], eax

:0040FA75 8B4DAC                  mov ecx, dword ptr [ebp-54]

 ====>ECX＝211C1E09          硬盘序列号

:0040FA78 894DF4                  mov dword ptr [ebp-0C], ecx

:0040FA7B 8B55F4                  mov edx, dword ptr [ebp-0C]

:0040FA7E 3355FC                  xor edx, dword ptr [ebp-04]

         ====>EDX＝211C1E09 XOR 19660116=387A1F1F

:0040FA81 8955F4                  mov dword ptr [ebp-0C], edx

:0040FA84 8B45F4                  mov eax, dword ptr [ebp-0C]

:0040FA87 0345FC                  add eax, dword ptr [ebp-04]

         ====>EAX＝387A1F1F + 19660116＝51E02035
         ====>51E02035(H)＝1373642805(D)  这就是我的注册码了

:0040FA8A 8945F4                  mov dword ptr [ebp-0C], eax

:0040FA8D 8B4DF4                  mov ecx, dword ptr [ebp-0C]

:0040FA90 51                      push ecx

\*
Possible StringData Ref from Data Obj ->"%d"
   |
:0040FA91 6818944300              push 00439418n

:0040FA96 8D55D8                  lea edx, dword ptr [ebp-28]

:0040FA99 52                      push edx

:0040FA9A E821850100              call 00427FC0

:0040FA9F 83C40C                  add esp, 0000000C

:0040FAA2 8B45AC                  mov eax, dword ptr [ebp-54]

                       ====>EAX＝211C1E09

:0040FAA5 3345FC                  xor eax, dword ptr [ebp-04]

                 ====>EAX＝211C1E09
XOR 19660116=387A1F1F

:0040FAA8 8945AC                  mov dword ptr [ebp-54], eax

:0040FAAB 8B4DAC                  mov ecx, dword ptr [ebp-54]

:0040FAAE 334DF8                  xor ecx, dword ptr [ebp-08]

         ====>ECX＝387A1F1F XOR 19900401=21EA1B1E

:0040FAB1 894DAC                  mov dword ptr [ebp-54], ecx

:0040FAB4 8B55AC                  mov edx, dword ptr [ebp-54]

:0040FAB7 0355F0                  add edx, dword ptr [ebp-10]

         ====>EAX＝21EA1B1E + 1357ACDB=3541C7F9
         ====>3541C7F9(H)=893503481(D)  这就是显示的用户码了

:0040FABA 8955AC                  mov dword ptr [ebp-54], edx

:0040FABD 8B45AC                  mov eax, dword ptr [ebp-54]

:0040FAC0 50                      push eax

\*
Possible StringData Ref from Data Obj ->"%d"
   |
:0040FAC1 681C944300              push 0043941C

:0040FAC6 8D4DC4                  lea ecx, dword ptr [ebp-3C]

:0040FAC9 51                      push ecx

:0040FACA E8F1840100              call 00427FC0

:0040FACF 83C40C                  add esp, 0000000C

:0040FAD2 8B550C                  mov edx, dword ptr [ebp+0C]

:0040FAD5 8955A8                  mov dword ptr [ebp-58], edx

:0040FAD8 817DA810010000          cmp dword ptr [ebp-58], 00000110

:0040FADF 740E                    je 0040FAEF

:0040FAE1 817DA811010000          cmp dword ptr [ebp-58], 00000111

:0040FAE8 742E                    je 0040FB18

:0040FAEA E94B010000              jmp 0040FC3A

\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040FADF(C)
|
:0040FAEF 8B4508                  mov eax, dword ptr [ebp+08]

:0040FAF2 50                      push eax

:0040FAF3 E805A40000              call 00419EFD

:0040FAF8 83C404                  add esp, 00000004

:0040FAFB 8D4DC4                  lea ecx, dword ptr [ebp-3C]

:0040FAFE 51                      push ecx

:0040FAFF 684A040000              push 0000044A

:0040FB04 8B5508                  mov edx, dword ptr [ebp+08]

:0040FB07 52                      push edx

\* Reference To:
USER32.SetDlgItemTextA, Ord:01F2h
                     |
:0040FB08 FF152C454900            Call dword ptr [0049452C]

:0040FB0E B801000000              mov eax, 00000001

:0040FB13 E928010000              jmp 0040FC40

\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040FAE8(C)
|
:0040FB18 837D1001                cmp dword ptr [ebp+10], 00000001

:0040FB1C 0F85FF000000            jne 0040FC21

:0040FB22 6A0F                    push 0000000F

:0040FB24 8D45B0                  lea eax, dword ptr [ebp-50]

:0040FB27 50                      push eax

:0040FB28 6840040000              push 00000440

:0040FB2D 8B4D08                  mov ecx, dword ptr [ebp+08]

:0040FB30 51                      push ecx

\*
Reference To: USER32.GetDlgItemTextA, Ord:00F5h
 |
:0040FB31 FF1530454900            Call dword ptr [00494530]

:0040FB37 85C0                    test eax, eax

:0040FB39 0F86A9000000            jbe 0040FBE8

:0040FB3F 8D55D8                  lea edx, dword ptr [ebp-28]

       ====>EDX=1373642805
注册码

:0040FB42 52                      push edx

:0040FB43 8D45B0                  lea eax, dword ptr [ebp-50]

                 ====>EAX=13572468
          试炼码

:0040FB46 50                      push eax

:0040FB47 E844810100              call 00427C90

             ====>比较CALL！

:0040FB4C 83C408                  add esp, 00000008

:0040FB4F 85C0                    test eax, eax

:0040FB51 755C                    jne 0040FBAF

 ====>跳则OVER！

:0040FB53 8B4DEC                  mov ecx, dword ptr [ebp-14]

:0040FB56 51                      push ecx

:0040FB57 E816040000              call 0040FF72

                       ====>保存注册信息！

:0040FB5C 83C404                  add esp, 00000004

:0040FB5F C7053400440001000000    mov dword ptr [00440034], 00000001

:0040FB69 C7055059480000000000    mov dword ptr [00485950], 00000000

:0040FB73 833DC0E4430000          cmp dword ptr [0043E4C0], 00000000

:0040FB7A 7518                    jne 0040FB94

:0040FB7C 6A00                    push 00000000

\*
Possible StringData Ref from Data Obj ->"注册成功"
     ====>呵呵，胜利女神！
:0040FB7E 6820944300              push 00439420

\*
Possible StringData Ref from Data Obj ->"感谢使用拼音之星!"
     |
:0040FB83 682C944300              push 0043942C

:0040FB88 8B5508                  mov edx, dword ptr [ebp+08]

:0040FB8B 52                      push edx

\*
Reference To: USER32.MessageBoxA, Ord:0195h
                       |
:0040FB8C
```
