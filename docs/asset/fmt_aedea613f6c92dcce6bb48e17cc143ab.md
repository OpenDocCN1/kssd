
# 红色警报 2 防拷贝

> 原文：[www.pediy.com/kssd/pediy03/forum1013.htm](https://www.pediy.com/kssd/pediy03/forum1013.htm)

## 概述

(safedisc 2) 红色警报 2 破软件防拷贝部分

CrackBy: `machoman\[CCG\]`

AllRight Reserved : `\[CCG\]`

其他不同 `SD2` 保护采用方式也是相似的，其保护方法其实采用 `Crack` 的方式是不太舒服的。在驱动级截获其命令，对付 `SD2` 是最简单的方法。

红色警报 2 的安装文件 `setup.exe` 和可执行文件 `ra2.exe` 采用了多种防拷贝措施，在这里面加入了 `safedisc2` 的防拷贝部分的代码，和一个不知道的壳，在这个壳的保护下。通过一系列代码保护的外加程序，变形的外接 `dll` 文件。对程序进行一部分部分的还原。并且在其中加入了防止 `debug` 的措施，在以下的部分主要就是要通过对这几部分的保护，找到程序的完全还原的映射。把它重新构件为一个无壳,无防 `debug`,无 `safedisc2` 调用的可执行文件。我通过以下步骤想完成该部分的无壳程序的重建。

## 第一步：脱壳与导入表重构

1. 首先用 `trw 1.03` 装入程序 `ra2.exe`，然后跟踪到程序完全还原处，在这个程序中的解密还原处为 `40787f` 绝对地址。用 `suspend` 挂起程序，再把内存中的影象用 `pdump1.60` 保存在一个文件中。得到了该程序的无壳影象。但这个程序还不可执行，这个程序破坏了导入表。无法装入外界的动态连接库。

再重构 `import_table`，让程序能够找到入口，这个程序的导入表也采用了加密措施，它把导入表的解密部分放在 `~df349b.tmp` 的里面，在这里进行导入表的再装配，但其实只是对 `kernel32.dll` 和 `user32.dll` 的 `import_table` 进行了加密，`ra2.exe` 程序在对这两个的内含函数进行调用时，需要通过对 `~df349b.tmp` 调用，才能间接的访问这些函数，但这个 `~df349b.tmp`(`~df349b.dll`) 本身也是加有外壳，被一个不知道的壳付在上面，而且也防调试，只能通过自己重构一个 `import_table` 跳过对这的调用，给 `ra2.exe` 中的调用重新建立隐射。

### 重建 import_table 步骤

(1) 首先在 dump 出来的文件中查找 `kernel32.dll` 这个字符串，它的导入表需要重建。在文件中可以找到几个位置，其中在 `0131a4`，这个位置的字符就是该在 `import_table` 的 `IMAGE_IMPORT_DESCRIPTOR.name` 指针指向的内容，于是就查找到在 `ra2.exe` 的正确的导入表位置在 `12cd8`，在这里开始的就是程序的导入表的 `IMAGE_IMPORT_DESCRIPTOR` 结构所在。然后把程序的 `IMAGE_DATA_DIRECTORY` 的第二项，就是 `import_table` 的地址改为 `12cd8` 就可以找到部分的导入函数，就是除 `kernel32.dll`,和 `user32.dll` 以外的其他函数都定位正确，但是这两个函数的 `RVA` 地址全没有正确定位，看来程序是对这个部分的导入进行了手工的重新装配，它的手工装配程序在 `~df349b.tmp` 里执行，这部分也是有壳加密的，脱起来很繁。最好的办法就是跳过这个动态库函数的调用。自己去给它重构它的 `RVA`,但我对 `pe` 的装入机制不了解，现在还不能重构好，重构出来的也不能完全对应。但是程序只要完成了导入表的重构，就能够做到破软件防拷贝。不会在对硬件要求是 `safedisc2` 加密后的盘应该也能读，因为在脱壳完成后就已经跳过了检测部分的代码。

有一个很奇怪的情况，程序在我下脱壳断点的位置，时我还 dump 出了一个很奇怪的文件。这个文件好象正跟我上面要找的导入表互补，它的导入表是完整的，但是程序是一个部分的影射，不能执行。但我用静态反汇编就已经发现了我所有的调用的函数的入口，但是我现在还是没太搞懂 `import_table` 的结构，我自己去找这个文件的入口结构时又是怪怪的。找不到。我有种感觉，这个程序是分段解密调入执行的，所有的 `tmp` 都是临时的可执行影象，只要搞清楚这两个的联系，完全脱壳该是只有一步了，但我现在就是不能把这两个联系起来。。。

这样解后程序就能够跳过对 `soft_ice` 和 `safedisc2` 的检测，但还是不能对光盘的物理结构的 `section` 进行拷贝，这样做只是跳过了软件部分的检测，对光盘的拷贝破解还是需要对 `safdisc2` 的结构继续研究。

在 `safedisc2` 加密的文件中有两种切入导入表的方法，第一种是在 `ra2.exe` 中除了导入表的位置改动，其他没有变化的方法，这样的部分可以重新指正确就可以改动，在 `safedisc2` 中对这部分的处理采用了以下方法，先把程序的导入表位置改为加密 `dll` 的入口，在加密 `dll` 中先有以下代码对程序的环境压入堆栈，保护然后进入加密部分切入核心模块，

(2) 在已经 dump 出的部分的程序中，也还有部分的 `import` 参数不是直接调用的情况它采用一个 `jmp` 指令跳到下面的地方进行切换进入外接的 `dll`，这样也造成了程序的代码返回后的不可读性，只有把这样的调用也给它重新构造，才能把代码完全还原，它是采用的分别跳转的方法，有很多个类似的 `jmp` 和这样的切入真的好烦！！！每个都要去给它打掉才可以重构，这个加密给解密的工作量是很大的。。

```
/*******************************************************************************

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040C488(U)
|
:0041B025 53                         push ebx  //这个 ebx 该是外接 dll 的代码段描述符
:0041B026 E800000000                 call 0041B02B//这个调用的目的是把 IP 压入堆栈

* Referenced by a CALL at Address:
|:0041B026
|
:0041B02B 870424                     xchg dword ptr [esp], eax//eax 中为新的 IP 跟堆栈中的进行交换
:0041B02E 9C                         pushfd//32 位标志压入堆栈
:0041B02F 05D5FFFFFF                 add eax, FFFFFFD5//IP-2bh=41b02b-2bh=41b000,每个调用都是定位这里
:0041B034 8B18                       mov ebx, dword ptr [eax]//把这个位置的内容送给 ebx
:0041B036 6BDB01                     imul ebx, 00000001
:0041B039 035804                     add ebx, dword ptr [eax+04]//41b004 中该是存储的入口 dll 的基地址
:0041B03C 9D                         popfd
:0041B03D 58                         pop eax
:0041B03E 871C24                     xchg dword ptr [esp], ebx//改变 ret 的方向，实际上是切换进入 dll
:0041B041 C3                         ret//进入 dll,调用函数，加密挺厉害的！！！！！！

*********************************************************************************/
```

## 临时文件与调试器检测

红色警报 2，该程序在安装及执行前先在系统的 `mywindows\temp` 生成一个子目录 `~ef87a`，这个目录里有几个临时文件，其中的 `~df394b.tmp` 其实是一个动态连接库，该为 `~df394b.dll` 在其中内有判断调试器存在的代码，要跟踪就一定要打掉这几个保护，该类文件在程序退出时会删除，下面就是其判断 `Soft_ice` 的方法

### Kill_ice (1)

```
/*******************************************************************************

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:1000F415(C), :1000F419(C)
|
:1000F41E EB03                         jmp 1000F423

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000F413(C)
|
:1000F420 73F3                         jnb 1000F415
:1000F422 18C7                         sbb bh, al
:1000F424 45                         inc ebp
:1000F425 FC                         cld
:1000F426 00000000                    BYTE  4 DUP(0)

;This method of detection of SoftICE (as well as the following one) is
;used by the majority of packers/encryptors found on Internet.
;It seeks the signature of BoundsChecker in SoftICE
:1000F42A 55                         push ebp
:1000F42B BD4B484342                 mov ebp, 4243484B ; 'BCHK'
:1000F430 B804000000                 mov eax, 00000004
:1000F435 CC                         int 03//这里是判断 soft_ice 的一种方法
:1000F436 5D                         pop ebp
:1000F437 C745FCFFFFFFFF             mov [ebp-04], FFFFFFFF
:1000F43E EB14                         jmp 1000F454
:1000F440 8B55EC                    mov edx, dword ptr [ebp-14]
:1000F443 52                         push edx
:1000F444 E8E7000000                 call 1000F530
:1000F449 C3                         ret

*********************************************************************************/
```

### Kill_ice(2)

```
/*******************************************************************************

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:1000E493(C), :1000E4AF(U)
|
:1000E4A8 8BFF                         mov edi, edi
:1000E4AA 7006                         jo 1000E4B2
:1000E4AC 90                         nop
:1000E4AD 7103                         jno 1000E4B2
:1000E4AF EBF7                         jmp 1000E4A8
:1000E4B1 DBE8                         fucomi st(0), st(0)
:1000E4B3 C9                         leave
:1000E4B4 000000                     BYTE  3 DUP(0)

//Method of detection of the WinICE handler in the int68h (V86)
//    mov    ah,43h
//    int    68h
//    cmp    ax,0F386h
//    jz      SoftICE_Detected
:1000E4B7 25FFFF0000                 and eax, 0000FFFF
:1000E4BC 85C0                         test eax, eax
:1000E4BE 7522                         jne 1000E4E2
:1000E4C0 60                         pushad
:1000E4C1 33C0                         xor eax, eax
:1000E4C3 66B80043                   mov ax, 4300
:1000E4C7 CD68                         int 68
:1000E4C9 89855CFFFFFF                mov dword ptr [ebp+FFFFFF5C], eax
:1000E4CF 3D00430000                 cmp eax, 00004300
:1000E4D4 740B                         je 1000E4E1
:1000E4D6 B801000000                 mov eax, 00000001
:1000E4DB 898560FFFFFF                mov dword ptr [ebp+FFFFFF60], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000E4D4(C)
|
:1000E4E1 61                         popad

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000E4BE(C)
|
:1000E4E2 33D2                         xor edx, edx
:1000E4E4 83BD60FFFFFF00             cmp dword ptr [ebp+FFFFFF60], 00000000
:1000E4EB 0F95C2                     setne dl
:1000E4EE 668955FC                   mov word ptr [ebp-04], dx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000E43E(C)
|
:1000E4F2 EB07                         jmp 1000E4FB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000E4FB(U)
|
*********************************************************************************/
```
