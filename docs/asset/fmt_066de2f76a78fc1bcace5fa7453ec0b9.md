
# 新概念英语句霸 2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50622.htm`](https://www.pediy.com/kssd/pediy05/pediy50622.htm)

下载页面：`http://www.en2000.com/download.htm`

软件大小：`5M`

**【软件简介】**：新概念英语句霸 2 是用一种用新思维、新概念来学习英语句型的学习软件。它采用著名的“艾滨豪斯遗忘曲线”原理,采用听，说，读，写，记等方法，并结合动画人物指导，复读，录音对比等软件功能，根据人体记忆曲线反复刺激记忆，以达到牢记所学句型的目的。软件中包含"英语 900 句","走遍美国”等众多学习英语的优秀教材做为学习对象，正式版的用户还可以赠送"新概念英语(1-4 册)的全部课文的句库和新概念英语的真人语音予以学习。

**【软件限制】**：`NAG`、功能限制。

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`TRW2000` 娃娃修改版、`Ollydbg1.09`、`PEiD`、`UnAspacka`、`W32Dasm 9.0` 白金版

—————————————————————————————————

**【过 程】**：

呵呵，朋友用 U 盘拿来这个东东要我看看，我看了一下说明，告诉他即使破解了也无法从其主页下载词库的，朋友说起码没有 `NAG` 了吧，我只好勉强试试了。哎，这个程序的算法已经超出了我的能力范围，我根本无法搞定，仅仅是简单分析 。见笑了。

`新概念英语句霸.exe` 是 `ASPack 2.12` 壳，用 `UnAspacka` 脱之。`534K->1.38M`。`Delphi` 编写。

注册名：`fly`

申请码：`58122273`

试炼码：`13572468-90ABCDEF`

程序要求重启验证，注册码保存在注册表中，于是在反汇编代码里查找"`RegNumOne`"，很容易就找到核心。

```
* Possible StringData Ref from Code Obj ->"JuBaPath"

:0050680A BAF06A5000       mov edx, 00506AF0
:0050680F E868B0F3FF       call 0044187C
:00506814 8D85C4FDFFFF     lea eax, dword ptr [ebp+FFFFFDC4]
:0050681A 50               push eax
:0050681B 8D95C0FDFFFF     lea edx, dword ptr [ebp+FFFFFDC0]
:00506821 A120CD5100       mov eax, dword ptr [0051CD20]
:00506826 8B00             mov eax, dword ptr [eax]
                         ====>EAX=58122273        呵呵，申请码
:00506828 E8AF96FFFF       call 004FFEDC
                         ====>核心 CALL！进入！
:0050682D 8B85C0FDFFFF     mov eax, dword ptr [ebp+FFFFFDC0]
                         ====>EAX=5D7B911B1AC3AD53
:00506833 B908000000       mov ecx, 00000008
:00506838 BA01000000       mov edx, 00000001
:0050683D E8DAE7EFFF       call 0040501C
:00506842 8B85C4FDFFFF     mov eax, dword ptr [ebp+FFFFFDC4]
:00506848 50               push eax
:00506849 8D8DBCFDFFFF     lea ecx, dword ptr [ebp+FFFFFDBC]
:0050684F A12CD05100       mov eax, dword ptr [0051D02C]
:00506854 8B00             mov eax, dword ptr [eax]

* Possible StringData Ref from Code Obj ->"RegNumOne"

:00506856 BA046B5000       mov edx, 00506B04
:0050685B E848B0F3FF       call 004418A8
:00506860 8B95BCFDFFFF     mov edx, dword ptr [ebp+FFFFFDBC]
                         ====>EDX=13572468
:00506866 58               pop eax
                         ====>EAX=5D7B911B
:00506867 E89CE6EFFF       call 00404F08
                         ====>比较前 8 位注册码
:0050686C 7564             jne 005068D2
                         ====>跳则 OVER！
:0050686E 8D85B8FDFFFF     lea eax, dword ptr [ebp+FFFFFDB8]
:00506874 50               push eax
:00506875 8D95B4FDFFFF     lea edx, dword ptr [ebp+FFFFFDB4]
:0050687B A120CD5100       mov eax, dword ptr [0051CD20]
:00506880 8B00             mov eax, dword ptr [eax]
:00506882 E85596FFFF       call 004FFEDC
:00506887 8B85B4FDFFFF     mov eax, dword ptr [ebp+FFFFFDB4]
:0050688D B908000000       mov ecx, 00000008
:00506892 BA09000000       mov edx, 00000009
:00506897 E880E7EFFF       call 0040501C
:0050689C 8B85B8FDFFFF     mov eax, dword ptr [ebp+FFFFFDB8]
:005068A2 50               push eax
:005068A3 8D8DB0FDFFFF     lea ecx, dword ptr [ebp+FFFFFDB0]
:005068A9 A12CD05100       mov eax, dword ptr [0051D02C]
:005068AE 8B00             mov eax, dword ptr [eax]

* Possible StringData Ref from Code Obj ->"RegNumTwo"

:005068B0 BA186B5000       mov edx, 00506B18
:005068B5 E8EEAFF3FF       call 004418A8
:005068BA 8B95B0FDFFFF     mov edx, dword ptr [ebp+FFFFFDB0]
                         ====>EDX=90ABCDEF
:005068C0 58               pop eax
                         ====>EAX=1AC3AD53
:005068C1 E842E6EFFF       call 00404F08
                         ====>比较后 8 位注册码
:005068C6 750A             jne 005068D2
                         ====>跳则 OVER！
:005068C8 A168CF5100       mov eax, dword ptr [0051CF68]
:005068CD C60001           mov byte ptr [eax], 01
                         ====>置 1 则 OK！
:005068D0 EB08             jmp 005068DA

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0050686C(C), :005068C6(C)
|
:005068D2 A168CF5100       mov eax, dword ptr [0051CF68]
:005068D7 C60000           mov byte ptr [eax], 00
                         ====>置 0 则 OVER！        爆破点!

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0050668E(C), :005068D0(U)
|
:005068DA A168CF5100       mov eax, dword ptr [0051CF68]
:005068DF 803800           cmp byte ptr [eax], 00
                         ====>比较
:005068E2 750F             jne 005068F3
                         ====>不跳则 OVER！
:005068E4 A168CA5100       mov eax, dword ptr [0051CA68]
:005068E9 8B00             mov eax, dword ptr [eax]
:005068EB 8B10             mov edx, dword ptr [eax]
:005068ED FF92EC000000     call dword ptr [edx+000000EC]
```

—————————————————————————————————

进入核心 `CALL`：`00506828 call 004FFEDC`

```
* Referenced by a CALL at Addresses:
|:00506828   , :00506882   
|
:004FFEDC 55                push ebp
:004FFEDD 8BEC             mov ebp, esp
:004FFEDF 83C4C4           add esp, FFFFFFC4
:004FFEE2 53               push ebx
:004FFEE3 56               push esi
:004FFEE4 57               push edi
:004FFEE5 33C9             xor ecx, ecx
:004FFEE7 894DC8           mov dword ptr [ebp-38], ecx
:004FFEEA 894DC4           mov dword ptr [ebp-3C], ecx
:004FFEED 894DF4           mov dword ptr [ebp-0C], ecx
:004FFEF0 894DDC           mov dword ptr [ebp-24], ecx
:004FFEF3 8955F8           mov dword ptr [ebp-08], edx
:004FFEF6 8945FC           mov dword ptr [ebp-04], eax
:004FFEF9 8B45FC           mov eax, dword ptr [ebp-04]
                         ====>EAX=58122273        呵呵，申请码
:004FFEFC E8AB50F0FF       call 00404FAC
                         ====>取申请码长度
:004FFF01 33C0             xor eax, eax
:004FFF03 55               push ebp
:004FFF04 68B0005000       push 005000B0
:004FFF09 64FF30           push dword ptr fs:[eax]
:004FFF0C 648920           mov dword ptr fs:[eax], esp
:004FFF0F 8D45F4           lea eax, dword ptr [ebp-0C]
:004FFF12 8B55FC           mov edx, dword ptr [ebp-04]
:004FFF15 E87A4CF0FF       call 00404B94
:004FFF1A 8D45F4           lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"1234567"

:004FFF1D BAC8005000       mov edx, 005000C8
                         ====>EDX=1234567
:004FFF22 E89D4EF0FF       call 00404DC4
:004FFF27 8D45F4           lea eax, dword ptr [ebp-0C]
:004FFF2A BA08000000       mov edx, 00000008
:004FFF2F E81452F0FF       call 00405148
:004FFF34 C745CC516F8550   mov [ebp-34], 50856F51
                         ====>[ebp-34]=50856F51   呵呵，程序给的固定值
:004FFF3B C745D061722F84   mov [ebp-30], 842F7261
                         ====>[ebp-30]=842F7261   呵呵，程序给的固定值
:004FFF42 C745D46265E742   mov [ebp-2C], 42E76562
```
