
# SantMat's ReverseME #2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6824.htm`](https://www.pediy.com/kssd/pediy06/pediy6824.htm)

SantMat's ReverseME #2 数字游戏

下载地址: [`www.reversemes.de/reversemes/reme2.zip`](http://www.reversemes.de/reversemes/reme2.zip)

这个又是让我们到程序里去看任务，主要有三条：一是去掉开始时的窗口，二是把两个按钮的代码填上，每按一次随机出现 0-100 的数，两个按钮分别对应两个玩家，谁的数大就得到 1 分，先得 50 分者为胜，三是用键盘上的“`M`”和“`Z`”控制两个按钮。

先反汇编，从开头看：

```
//****** Start of Code in Object .text ******
Program Entry Point = 00401000 (SantMat-ReverseMe2.exe File Offset:00001600)
//****** Program Entry Point ******

:00401000 6A00                 push 00000000

* Reference To: KERNEL32.GetModuleHandleA, Ord:0111h
                            |
:00401002 E859010000         Call 00401160 
:00401007 A39C304000         mov dword ptr [0040309C], eax
:0040100C 6A00                 push 00000000

* Possible StringData Ref from Code Obj ->"U 嬱亇 "
                            |
:0040100E 6842104000         push 00401042 <-对话框的消息处理
:00401013 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"MEMO"
                            |
:00401015 6800304000         push 00403000 <-看看,应该是开始时出现的对话框
:0040101A FF359C304000       push dword ptr [0040309C]

* Reference To: USER32.DialogBoxParamA, Ord:0092h
                            |
:00401020 E81D010000         Call 00401142 <-这叫"基于对话框的程序"吧,我不太懂        
:00401025 6A00                 push 00000000

* Reference To: KERNEL32.LoadLibraryA, Ord:01A9h
                            |
:00401027 E840010000         Call 0040116C
:0040102C 6A00                 push 00000000

* Reference To: KERNEL32.FreeLibrary, Ord:00A2h
                            |
:0040102E E827010000         Call 0040115A
:00401033 6A00                 push 00000000
:00401035 6A00                 push 00000000

* Reference To: KERNEL32.GetProcAddress, Ord:0129h
                            |
:00401037 E82A010000         Call 00401166
:0040103C 50                   push eax

* Reference To: KERNEL32.ExitProcess, Ord:0075h
                            |
:0040103D E812010000         Call 00401154
:00401042 55                   push ebp
:00401043 8BEC                 mov ebp, esp
:00401045 817D0C10010000       cmp dword ptr [ebp+0C], 00000110
:0040104C 7502                 jne 00401050
:0040104E EB66                 jmp 004010B6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040104C(C)
|
:00401050 837D0C10             cmp dword ptr [ebp+0C], 00000010
:00401054 750C                 jne 00401062
:00401056 6A00                 push 00000000
:00401058 FF7508             push [ebp+08]

* Reference To: USER32.EndDialog, Ord:00B8h
                            |
:0040105B E8E8000000         Call 00401148
:00401060 EB54                 jmp 004010B6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401054(C)
|
:00401062 817D0C11010000       cmp dword ptr [ebp+0C], 00000111
:00401069 7542                 jne 004010AD
:0040106B 8B4510               mov eax, dword ptr [ebp+10]
:0040106E 663DEC03             cmp ax, 03EC
:00401072 7542                 jne 004010B6
:00401074 C1E810               shr eax, 10
:00401077 660BC0               or ax, ax
:0040107A 752F                 jne 004010AB
:0040107C 6A00                 push 00000000
:0040107E FF7508             push [ebp+08]

* Reference To: USER32.EndDialog, Ord:00B8h
                            |
:00401081 E8C2000000         Call 00401148 <-至此是第一个对话框
:00401086 6A00                 push 00000000

* Reference To: KERNEL32.GetModuleHandleA, Ord:0111h
                            |
:00401088 E8D3000000         Call 00401160 <-又出现一次,第二个对话框开始了
:0040108D A3A0304000         mov dword ptr [004030A0], eax
:00401092 6A00                 push 00000000

* Possible StringData Ref from Code Obj ->"U 嬱亇 "
                            |
:00401094 68BF104000         push 004010BF
:00401099 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"GAME"
                            |
:0040109B 6805304000         push 00403005
:004010A0 FF35A0304000       push dword ptr [004030A0]

* Reference To: USER32.DialogBoxParamA, Ord:0092h
                            |
:004010A6 E897000000         Call 00401142

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040107A(C)
|
:004010AB EB09                 jmp 004010B6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401069(C)
|
:004010AD B800000000         mov eax, 00000000
:004010B2 C9                   leave
:004010B3 C21000             ret 0010
```

可见这两个对话框是“独立”的，只要把关于第一个对话框的代码去掉就可以了，怎么去掉？全 `NOP`？当然可以，不过是不是累点？我干脆把程序的入口点改在了 `401086`，前面全没用了，`HEHE~~~~`

接下来看第二个对话框的消息处理：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004010D1(C)
|
:004010DF 817D0C11010000       cmp dword ptr [ebp+0C], 00000111
:004010E6 7547                 jne 0040112F
:004010E8 8B4510               mov eax, dword ptr [ebp+10]
:004010EB 663DED03             cmp ax, 03ED
:004010EF 751B                 jne 0040110C
:004010F1 C1E810               shr eax, 10
:004010F4 660BC0               or ax, ax
:004010F7 7513                 jne 0040110C
:004010F9 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"Number Game"
                            |
:004010FB 680A304000         push 0040300A

* Possible StringData Ref from Data Obj ->"This is the part you need to change "
                                    ->"into a number generator! :)"
                            |
:00401100 6816304000         push 00403016
:00401105 6A00                 push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BBh
                            |
:00401107 E842000000         Call 0040114E <-按第一个按钮后出现的对话框

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004010EF(C), :004010F7(C)
|
:0040110C 663DEE03             cmp ax, 03EE
:00401110 7526                 jne 00401138
:00401112 C1E810               shr eax, 10
:00401115 660BC0               or ax, ax
:00401118 7513                 jne 0040112D
:0040111A 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"Number Game"
                            |
:0040111C 680A304000         push 0040300A

* Possible StringData Ref from Data Obj ->"This is the other part you need "
                                    ->"to change into a number generator! "
                                    ->":)"
                            |
:00401121 6856304000         push 00403056
:00401126 6A00                 push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BBh
                            |
:00401128 E821000000         Call 0040114E <-按第二个按钮后出现的对话框

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401118(C)
|
:0040112D EB09                 jmp 00401138
```

很显然只要在显示对话框的地方加上我们的代码就行了,还是老方法,跳到后面去.

改动的地方：

```
:004010F9 E974000000           jmp 00401172

:0040111A A138214000           mov eax, dword ptr [00402138] <-这是我写在 .data 块里的代码里的第一句，那里看起来是空地，结果一运行就会把这一句吃掉，所以只好写在这里（建议写在自己新建的块里安全一些）
```
