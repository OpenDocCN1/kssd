
# 智能切割机 3.03

> 原文：[www.pediy.com/kssd/pediy04/kanxue363.htm](https://www.pediy.com/kssd/pediy04/kanxue363.htm)

## 不同的断点,不同的效果!!!

**作者**：`ssljx`[`BCG`]

**破解软件**：智能切割机 3.03

**软件简介**：智能切割机，顾名思义，它的操作过程是全智能化的。绿色软件，无须安装，解压后可直接运行。界面简洁美观，全部是仿 XP 图形化的按钮，非常直观友好，易于掌握，所以使用也很简单。它不但可以方便快速地切割任何类型任意大小的文件，而且能将切割块分毫不差地合并还原，并提供自合并功能。若其他软件生成的切割块符合标准，它同样能把它们合并还原。智能切割机支持多种★新型可移动磁盘！在操作过程中，它会根据磁盘的类型和剩余空间给出提示，指导您的下一步操作。

智能切割机不仅仅能方便地将大文件备份到小容量软盘。在 Email 中要附带大文件时，可先切割成多块分别发送，这样就不怕中途断线了。现在有很多个人主页限制文件大小，就可将大文件分割上传了。

**下载**：`http://cyberian.myetang.com/dl/isplitcn.exe`

**工具**：`softice`

**日期**：2002-01-22

## 破解过程

这个软件很小,是个绿色软件.加了 `PECompact` 1.40 - 1.45 的壳,这些是废话..哈...好开始我们破解之旅..这次破解我想用两个不同的设断方法来破解,体会不同断点,不同效果!!好运行软件填上注册码和注册名按确定,提示重新启动验证注册码...ok!用 `filemon` 检测过没读取其他文件..嗯..注册码和注册名放在注册表..就用 `BPX` `RegQueryValueExa`

`DO "bd \*(esp+8)"`

设置断点..哈..在 `softice` 中断了..按了 N 次 `F5` 才看到数据区出现两次 `regcode`,马上按 `F12`,按了 3 次才来到这里..

```
:004862E9 8B55E8                 mov edx, dword ptr [ebp\-18]
:004862EC B830DD4800             mov eax, 0048DD30
:004862F1 E806E5F7FF             call 004047FC
:004862F6 A13CDD4800             mov eax, dword ptr [0048DD3C]
:004862FB E87CC3FDFF             call 0046267C
:00486300 8D55E0                 lea edx, dword ptr [ebp\-20]
:00486303 B8B4614800             mov eax, 004861B4
:00486308 E8D706F8FF             call 004069E4
:0048630D 8B4DE0                 mov ecx, dword ptr [ebp\-20]
:00486310 8D45E4                 lea eax, dword ptr [ebp\-1C]
:00486313 BA68644800             mov edx, 00486468
:00486318 E88FE7F7FF             call 00404AAC
:0048631D 8B55E4                 mov edx, dword ptr [ebp\-1C]
:00486320 B101                   mov cl, 01
:00486322 A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486327 E8E4C3FDFF             call 00462710
:0048632C BAA0644800             mov edx, 004864A0
:00486331 A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486336 E861C7FDFF             call 00462A9C
:0048633B 84C0                   test al, al
:0048633D 751B                   jne 0048635A
:0048633F E8143DF8FF             call 0040A058
:00486344 83C4F8                 add esp, FFFFFFF8
:00486347 DD1C24                 fstp qword ptr [esp]
:0048634A 9B                     wait
:0048634B BAA0644800             mov edx, 004864A0
:00486350 A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486355 E83EC6FDFF             call 00462998
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0048633D(C)
|
:0048635A BAA0644800             mov edx, 004864A0
:0048635F A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486364 E843C6FDFF             call 004629AC
:00486369 DD5DD8                 fstp qword ptr [ebp\-28]
:0048636C 9B                     wait
:0048636D E8E63CF8FF             call 0040A058
:00486372 DC5DD8                 fcomp qword ptr [ebp\-28]
:00486375 DFE0                   fstsw ax
:00486377 9E                     sahf
:00486378 7236                   jb 004863B0
:0048637A E8D93CF8FF             call 0040A058
:0048637F 83C4F8                 add esp, FFFFFFF8
:00486382 DD1C24                 fstp qword ptr [esp]
:00486385 9B                     wait
:00486386 BAA0644800             mov edx, 004864A0
:0048638B A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486390 E817C6FDFF             call 004629AC
:00486395 83C4F8                 add esp, FFFFFFF8
:00486398 DD1C24                 fstp qword ptr [esp]
:0048639B 9B                     wait
:0048639C E877F9FFFF             call 00485D18
:004863A1 BA1E000000             mov edx, 0000001E
:004863A6 2BD0                   sub edx, eax
:004863A8 891534DD4800           mov dword ptr [0048DD34], edx
:004863AE EB07                   jmp 004863B7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486378(C)
|
:004863B0 33C0                   xor eax, eax
:004863B2 A334DD4800             mov dword ptr [0048DD34], eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004863AE(U)
|
:004863B7 33C0                   xor eax, eax
:004863B9 5A                     pop edx
:004863BA 59                     pop ecx
:004863BB 59                     pop ecx
:004863BC 648910                 mov dword ptr fs:[eax], edx
:004863BF 68D6634800             push 004863D6
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004863D4(U)
|
:004863C4 A13CDD4800             mov eax, dword ptr [0048DD3C]
:004863C9 E86ED6F7FF             call 00403A3C
:004863CE C3                     ret
:004863CF E9FCDDF7FF             jmp 004041D0
:004863D4 EBEE                   jmp 004863C4
:004863D6 8D55CC                 lea edx, dword ptr [ebp\-34]
:004863D9 B8BC614800             mov eax, 004861BC
:004863DE E80106F8FF             call 004069E4
:004863E3 8B45CC                 mov eax, dword ptr [ebp\-34]
:004863E6 8A10                   mov dl, byte ptr [eax]
:004863E8 8D45D0                 lea eax, dword ptr [ebp\-30]
:004863EB E898E5F7FF             call 00404988
:004863F0 8B45D0                 mov eax, dword ptr [ebp\-30]
:004863F3 50                     push eax
:004863F4 8D45D4                 lea eax, dword ptr [ebp\-2C]
:004863F7 50                     push eax
:004863F8 8D55C8                 lea edx, dword ptr [ebp\-38]
:004863FB B8B4614800             mov eax, 004861B4
:00486400 E8DF05F8FF             call 004069E4
:00486405 8B4DC8                 mov ecx, dword ptr [ebp\-38]
:00486408 BAB4644800             mov edx, 004864B4
:0048640D A12CDD4800             mov eax, dword ptr [0048DD2C]
:00486412 E8B1FCFFFF             call 004860C8
:00486417 8B55D4                 mov edx, dword ptr [ebp\-2C]  //经过上面漫长而曲节的跟踪才找到这里就是我们要找的注册码
:0048641A A130DD4800             mov eax, dword ptr [0048DD30] //这是我们输入的注册码
```

真累,累吗?哈哈..想想办法就不用这么累了,这个软件有个限制就是 30 天的试用期,那就好办,用时间断点呢,好说干就干..在 `sofice` 设 `BPX` `GetLocalTime`,程序中断了,按 1 次 `F5`,2 次 `F12` 就来到这里...

```
:0048637F 83C4F8                 add esp, FFFFFFF8
:00486382 DD1C24                 fstp qword ptr [esp]
:00486385 9B                     wait
:00486386 BAA0644800             mov edx, 004864A0
:0048638B A13CDD4800             mov eax, dword ptr [0048DD3C]
:00486390 E817C6FDFF             call 004629AC
:00486395 83C4F8                 add esp, FFFFFFF8
:00486398 DD1C24                 fstp qword ptr [esp]
:0048639B 9B                     wait
:0048639C E877F9FFFF             call 00485D18
:004863A1 BA1E000000             mov edx, 0000001E
:004863A6 2BD0                   sub edx, eax
:004863A8 891534DD4800           mov dword ptr [0048DD34], edx
:004863AE EB07                   jmp 004863B7
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00486378(C)
|
:004863B0 33C0                   xor eax, eax
:004863B2 A334DD4800             mov dword ptr [0048DD34], eax
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004863AE(U)
|
:004863B7 33C0                   xor eax, eax
:004863B9 5A                     pop edx
:004863BA 59                     pop ecx
:004863BB 59                     pop ecx
:004863BC 648910                 mov dword ptr fs:[eax], edx
:004863BF 68D6634800             push 004863D6
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004863D4(U)
|
:004863C4 A13CDD4800             mov eax, dword ptr [0048DD3C]
```
