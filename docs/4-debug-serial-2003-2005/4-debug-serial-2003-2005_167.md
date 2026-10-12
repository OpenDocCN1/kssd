
# HTMLock 1.9.3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6132.htm`](https://www.pediy.com/kssd/pediy06/pediy6132.htm)

HTMLock 1.9.3 破解手记---算法分析

作者：`newlaos\[DFCG\]\[CCG\]`

文件大小：668KB

软件授权：共享软件

使用平台：`Win9x/Me/NT/2000`

下载链接：`http://sq.onlinedown.net/down/htl.zip`

软件简介：帮助您创建基于 `javascript` 的密码保护页面。但对中文不支持。

加密方式：注册码

功能限制：日期+功能限制

`PJ` 工具：`TRW20001.23` 注册版，`W32Dasm8.93` 黄金版，`FI2.5`

`PJ` 日期：2003-06-24 ~ 6-26

作者 `newlaos` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 一、奇妙的开始

首先我们用常用方法，对软件进行分析。运行软件，发现有注册对话框，却无法击活“注册”按钮。呵呵，我想起了“按钮突破专家”，果然有效，但只是显示感谢谁谁谁的注册，功能上还是受限制，而且重启后还是显示没有注册。

## 二、兴奋的过程

### 1、找到第一个注册码检证点

因为没有关键字符串可以找，所以只有用 `bpx hmemcpy`(`万能断点大法`)，很快你就能定位到下列这断代码段。

经过调试，你会发现这段代码是一个循环事件，只要用户名或注册码的框内有变化，它就运行一遍，检查输入的注册码是否符合要求，如果符合这个注册码检证点的要求，“注册”按钮就会自动击活，原来不用“按钮突破专家”啊!

```
.......

.......

* Referenced by a CALL at Addresses:
:00471567   , :0047160F   , :004718C0   , :00471CB8   

|

:00470B80 55                       push ebp  
:00470B81 8BEC                     mov ebp, esp  
:00470B83 83C4F8                   add esp, FFFFFFF8  
:00470B86 53                       push ebx  
:00470B87 56                       push esi  
:00470B88 33C9                     xor ecx, ecx  
:00470B8A 894DF8                   mov dword ptr [ebp-08], ecx  
:00470B8D 8BDA                     mov ebx, edx  
:00470B8F 8BF0                     mov esi, eax  
:00470B91 33C0                     xor eax, eax  
:00470B93 55                       push ebp  
:00470B94 681A0C4700               push 00470C1A  
:00470B99 64FF30                   push dword ptr fs:[eax]  
:00470B9C 648920                   mov dword ptr fs:[eax], esp  
:00470B9F C645FF00                 mov [ebp-01], 00  
:00470BA3 8BC3                     mov eax, ebx  
:00470BA5 E80E37F9FF               call 004042B8  
:00470BAA 83F802                   cmp eax, 00000002  <===注册名必须长于 2 位  
:00470BAD 7C55                     jl 00470C04  <===这里跳了就 OVER 了  
:00470BAF 8BC6                     mov eax, esi  
:00470BB1 E80237F9FF               call 004042B8  
:00470BB6 83F824                   cmp eax, 00000024  <===很明显，这里是要求注册码必须长于 36 位  
:00470BB9 7C49                     jl 00470C04  <===这里跳了就 OVER 了  
:00470BBB 33DB                     xor ebx, ebx  
:00470BBD 8BC6                     mov eax, esi  
:00470BBF E8F436F9FF               call 004042B8 <===求出注册码的长度来  
:00470BC4 83E802                   sub eax, 00000002 <===这个长度减去 2，因为前面部分是用来计算后两位的  
:00470BC7 85C0                     test eax, eax  
:00470BC9 7E0F                     jle 00470BDA  
:00470BCB BA01000000               mov edx, 00000001 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00470BD8(C)  
|  
:00470BD0 8A4C16FF                 mov cl, byte ptr [esi+edx-01] <===取字符的 ASC 值  
:00470BD4 02D9                     add bl, cl  
:00470BD6 42                       inc edx  
:00470BD7 48                       dec eax  
:00470BD8 75F6                    jne 00470BD0    <===这里构成一个循环结构，对注册码的前面部分运算(不包括后两位)，但不超过 255。 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00470BC9(C)  
|  
:00470BDA 33C0                     xor eax, eax  
:00470BDC 8AC3                     mov al, bl    <===BL 就是上面循环的结果  
:00470BDE 33D2                     xor edx, edx  
:00470BE0 52                       push edx  
:00470BE1 50                       push eax  
:00470BE2 8D45F8                   lea eax, dword ptr [ebp-08]  
:00470BE5 E85AFEFFFF               call 00470A44    <===一个关键 CALL，F8 跟进  
:00470BEA 8B45F8                   mov eax, dword ptr [ebp-08]  
:00470BED 8A400E                   mov al, byte ptr [eax+0E]  
:00470BF0 3A4622                   cmp al, byte ptr [esi+22]  <===对比输入假码的第 35 位  
:00470BF3 750F                     jne 00470C04  <===跳了就 OVER 了  
:00470BF5 8B45F8                   mov eax, dword ptr [ebp-08]  
:00470BF8 8A400F                   mov al, byte ptr [eax+0F]  
:00470BFB 3A4623                   cmp al, byte ptr [esi+23]  <===对比输入假码的第 36 位，因为这里的特殊性使得注册码只是 36 位的长度(根据 BL 的取值决定最后两位，因为通常情况下我无法将一个值等于这个值做为中间值参与计算后的结果)  
:00470BFE 7504                     jne 00470C04  <===跳了就 OVER 了  
:00470C00 C645FF01                 mov [ebp-01], 01    <===很明显，必须经过这里才能正确注册 

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00470BAD(C), :00470BB9(C), :00470BF3(C), :00470BFE(C)  <===这四处都是错误的跳转了  
|  
:00470C04 33C0                     xor eax, eax  
:00470C06 5A                       pop edx  
:00470C07 59                       pop ecx  
:00470C08 59                       pop ecx  
:00470C09 648910                   mov dword ptr fs:[eax], edx  
:00470C0C 68210C4700               push 00470C21 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00470C1F(U)  
|  
:00470C11 8D45F8                   lea eax, dword ptr [ebp-08]  
:00470C14 E8DF33F9FF               call 00403FF8  
:00470C19 C3                        ret 

:00470C1A E9DD2DF9FF               jmp 004039FC  
:00470C1F EBF0                      jmp 00470C11  
:00470C21 8A45FF                   mov al, byte ptr [ebp-01]  <===最为关键的赋值，要求 AL 不能为 0，(当然这里也是爆破点，即让 AL 始终为 1)  
:00470C24 5E                       pop esi  
:00470C25 5B                       pop ebx  
:00470C26 59                       pop ecx  
:00470C27 59                       pop ecx  
:00470C28 5D                       pop ebp  
:00470C29 C3                        ret  

.......

.......

-----------:00470BE5 call 00470A44    <===一个关键 CALL，F8 跟进-----------------  
:00470A44 55                       push ebp  
:00470A45 8BEC                     mov ebp, esp  
:00470A47 83C4F8                   add esp, FFFFFFF8  
:00470A4A 53                       push ebx  
:00470A4B 56                       push esi  
:00470A4C 8BF0                     mov esi, eax  
:00470A4E 8BC6                     mov eax, esi  
:00470A50 BA10000000               mov edx, 00000010  
:00470A55 E8EA3BF9FF               call 00404644  
:00470A5A 8B4508                   mov eax, dword ptr [ebp+08]  
:00470A5D 8945F8                   mov dword ptr [ebp-08], eax  
:00470A60 8B450C                   mov eax, dword ptr [ebp+0C]  
:00470A63 8945FC                   mov dword ptr [ebp-04], eax  
:00470A66 BB10000000               mov ebx, 00000010 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00470AC4(C)  
|  
:00470A6B 8B45F8                   mov eax, dword ptr [ebp-08]  
:00470A6E 8B55FC                   mov edx, dword ptr [ebp-04]  
:00470A71 52                       push edx  
:00470A72 50                       push eax  
:00470A73 6A00                     push 00000000  
:00470A75 6A10                     push 00000010  
:00470A77 8B45F8                   mov eax, dword ptr [ebp-08]  
:00470A7A 8B55FC                   mov edx, dword ptr [ebp-04]  
:00470A7D 0FACD004                 shrd eax, edx, 04  
:00470A81 C1EA04                   shr edx, 04  
:00470A84 E84343F9FF               call 00404DCC  
:00470A89 290424                   sub dword ptr [esp], eax  
:00470A8C 19542404                 sbb dword ptr [esp+04], edx  
:00470A90 58                        pop eax  
:00470A91 5A                        pop edx  
:00470A92 83C001                   add eax, 00000001  
:00470A95 83D200                   adc edx, 00000000 

* Possible StringData Ref from Code Obj ->"HACKERYouMUSTDie"      


:00470A98 BAD80A4700               mov edx, 00470AD8  
:00470A9D 8A4402FF                 mov al, byte ptr [edx+eax-01]  
:00470AA1 50                        push eax  
:00470AA2 8BC6                      mov eax, esi  
:00470AA4 E8673AF9FF               call 00404510  
:00470AA9 5A                        pop edx  
:00470AAA 885418FF                 mov byte ptr [eax+ebx-01], dl  
:00470AAE 8B45F8                   mov eax, dword ptr [ebp-08]  
:00470AB1 8B55FC                   mov edx, dword ptr [ebp-04]
```



# 注册码校验逻辑分析

## 1. 注册码合法性验证

以下代码段用于验证注册码的初步合法性，此阶段未与注册名发生关系。初步确定的注册码为 `"1234567890123456789012345678901234eA"`。

```
:00470AB4 0FACD004                shrd eax, edx, 04  

:00470AB8 C1EA04                  shr edx, 04  

:00470ABB 8945F8                  mov dword ptr [ebp-08], eax  

:00470ABE 8955FC                  mov dword ptr [ebp-04], edx  

:00470AC1 4B                      dec ebx  

:00470AC2 85DB                    test ebx, ebx  

:00470AC4 75A5                    jne 00470A6B  

:00470AC6 5E                      pop esi  

:00470AC7 5B                      pop ebx  

:00470AC8 59                      pop ecx  

:00470AC9 59                      pop ecx  

:00470ACA 5D                      pop ebp  

:00470ACB C20800                  ret 0008  
```

## 2. 第 2 个注册码检证点

上述注册码存在局限，在运行生成加密网页时，程序会报告“未注册版本”并限制加密网页大小不超过 `4KB`。

可使用 `BPX SENDMESSAGE` 断点方法定位代码段，但推荐使用 `BPX HMEMCPY`（提示：虽然有时离关键地方很远，但至少位于注册校验的前面，只需注意寄存器里的数据变化，一旦发现自己输入的假码，就立即停下来仔细观察）。关键算法段位于 `00472208` 行的 `CALL` 指令中。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004721D5(C)

|

:004721EC 8B45F4                  mov eax, dword ptr [ebp-0C]  

:004721EF E8CCE2FFFF              call 004704C0  

:004721F4 3D00100000              cmp eax, 00001000   <===0x1000 就是 10 进制的 4096 也就是软件未注册时 4K 大小的限制了，改这里可以实现功能上的爆破  

:004721F9 0F9745D7                seta byte ptr [ebp-29]  

:004721FD 807DD700                cmp byte ptr [ebp-29], 00  

:00472201 7464                    je 00472267   <===这里是第一点，从这里跳，就是说所加密的软件小于 4KB  

:00472203 A17C7C4700              mov eax, dword ptr [00477C7C]   <===EAX="1234567890123456789012345678901234eA"就是我们输入的假注册码了  

:00472208 E8E7ECFFFF              call 00470EF4  <===不用问了，还要 F8 跟进  

:0047220D 84C0                    test al, al   <===要求是返回时 AL 应该为 1，才能正确运行  

:0047220F 7556                    jne 00472267   <===这里必须跳走  

:00472211 6A10                    push 00000010  

:00472213 8D559C                  lea edx, dword ptr [ebp-64] 

* Possible StringData Ref from Code Obj ->`vG"  

                                 |  

:00472216 B868174700              mov eax, 00471768  

:0047221B E8003AF9FF              call 00405C20  

:00472220 8B459C                  mov eax, dword ptr [ebp-64]  

:00472223 E89022F9FF              call 004044B8  

:00472228 50                      push eax  

:00472229 8D5598                  lea edx, dword ptr [ebp-68] 

* Possible StringData Ref from Code Obj ->`vG"  

                                 |  

:0047222C B818184700              mov eax, 00471818  

:00472231 E8EA39F9FF              call 00405C20  

:00472236 8B4598                  mov eax, dword ptr [ebp-68]  

:00472239 E87A22F9FF              call 004044B8  

:0047223E 50                      push eax  

:0047223F 6A00                    push 00000000 

* Reference To: user32.MessageBoxA, Ord:0000h  

                                 |  

:00472241 E85246F9FF              Call 00406898   <===这里就是出现未注册对话框的信息  

:00472246 A1F4684700              mov eax, dword ptr [004768F4]  

:0047224B 8B00                    mov eax, dword ptr [eax]  

:0047224D E8B2190000              call 00473C04  

:00472252 8B45F4                  mov eax, dword ptr [ebp-0C]  

:00472255 E84E10F9FF              call 004032A8  

:0047225A 8B45F0                  mov eax, dword ptr [ebp-10]  

:0047225D E84610F9FF              call 004032A8  

:00472262 E998030000              jmp 004725FF 

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:00472201(C), :0047220F(C)           <===有两处可以跳过去了，向上看  

|  

:00472267 B930750000              mov ecx, 00007530  

.......

.......
```

## 3. 关键检测点详解 (`00472208 call 00470EF4`)

无需多问，这是第 2 个关键检测点，需使用 `F8` 指令单步跟进。

```
----------:00472208 call 00470EF4  不用问了，第 2 个关键检测点，还要 F8 跟进---------  

:00470EF4 55                      push ebp  

:00470EF5 8BEC                    mov ebp, esp  

:00470EF7 83C4E0                  add esp, FFFFFFE0  

:00470EFA 53                      push ebx  

:00470EFB 56                      push esi  

:00470EFC 33D2                    xor edx, edx  

:00470EFE 8955FC                  mov dword ptr [ebp-04], edx  

:00470F01 8955F8                  mov dword ptr [ebp-08], edx  

:00470F04 8BD8                    mov ebx, eax  <===EBX="1234567890123456789012345678901234eA"  

:00470F06 33C0                    xor eax, eax  

:00470F08 55                      push ebp  

:00470F09 68EA104700              push 004710EA  

:00470F0E 64FF30                  push dword ptr fs:[eax]  

:00470F11 648920                  mov dword ptr fs:[eax], esp  

:00470F14 E8F7F9FFFF              call 00470910  

:00470F19 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F1C BA10000000              mov edx, 00000010  

:00470F21 E81E37F9FF              call 00404644  

:00470F26 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F29 E8E235F9FF              call 00404510  

:00470F2E 8A5301                  mov dl, byte ptr [ebx+01]  <===取第 2 位 2  

:00470F31 8810                    mov byte ptr [eax], dl  

:00470F33 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F36 E8D535F9FF              call 00404510  

:00470F3B 8A5303                  mov dl, byte ptr [ebx+03]  <===取第 4 位 4  

:00470F3E 885001                  mov byte ptr [eax+01], dl  

:00470F41 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F44 E8C735F9FF              call 00404510  

:00470F49 8A5304                  mov dl, byte ptr [ebx+04]  <===取第 5 位 5  

:00470F4C 885002                  mov byte ptr [eax+02], dl  

:00470F4F 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F52 E8B935F9FF              call 00404510  

:00470F57 8A5309                  mov dl, byte ptr [ebx+09]  <===取第 10 位 0  

:00470F5A 885003                  mov byte ptr [eax+03], dl  

:00470F5D 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F60 E8AB35F9FF              call 00404510  

:00470F65 8A530A                  mov dl, byte ptr [ebx+0A]  <===取第 11 位 1  

:00470F68 885004                  mov byte ptr [eax+04], dl  

:00470F6B 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F6E E89D35F9FF              call 00404510  

:00470F73 8A530B                  mov dl, byte ptr [ebx+0B]  <===取第 12 位 2  

:00470F76 885005                  mov byte ptr [eax+05], dl  

:00470F79 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F7C E88F35F9FF              call 00404510  

:00470F81 8A530D                  mov dl, byte ptr [ebx+0D]  <===取第 14 位 4  

:00470F84 885006                  mov byte ptr [eax+06], dl  

:00470F87 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F8A E88135F9FF              call 00404510  

:00470F8F 8A530E                  mov dl, byte ptr [ebx+0E]  <===取第 15 位 5  

:00470F92 885007                  mov byte ptr [eax+07], dl  

:00470F95 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470F98 E87335F9FF              call 00404510  

:00470F9D 8A530F                  mov dl, byte ptr [ebx+0F]  <===取第 16 位 6  

:00470FA0 885008                  mov byte ptr [eax+08], dl  

:00470FA3 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470FA6 E86535F9FF              call 00404510  

:00470FAB 8A5313                  mov dl, byte ptr [ebx+13]  <===取第 20 位 0  

:00470FAE 885009                  mov byte ptr [eax+09], dl  

:00470FB1 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470FB4 E85735F9FF              call 00404510  

:00470FB9 8A5314                  mov dl, byte ptr [ebx+14]  <===取第 21 位 1  

:00470FBC 88500A                  mov byte ptr [eax+0A], dl  

:00470FBF 8D45FC                  lea eax, dword ptr [ebp-04]  

:00470FC2 E84935F9FF              call 00404510  

:00470FC7 8A5316                  mov dl, byte ptr [ebx+16]  <===取第 23 位 3  
```



:00470FCA `88500B`  `mov byte ptr [eax+0B], dl`

:00470FCD `8D45FC`  `lea eax, dword ptr [ebp-04]`

:00470FD0 `E83B35F9FF`  `call 00404510`

:00470FD5 `8A5319`  `mov dl, byte ptr [ebx+19]` <===取第 26 位 6

:00470FD8 `88500C`  `mov byte ptr [eax+0C], dl`

:00470FDB `8D45FC`  `lea eax, dword ptr [ebp-04]`

:00470FDE `E82D35F9FF`  `call 00404510`

:00470FE3 `8A531A`  `mov dl, byte ptr [ebx+1A]` <===取第 27 位 7

:00470FE6 `88500D`  `mov byte ptr [eax+0D], dl`

:00470FE9 `8D45FC`  `lea eax, dword ptr [ebp-04]`

:00470FEC `E81F35F9FF`  `call 00404510`

:00470FF1 `8A531B`  `mov dl, byte ptr [ebx+1B]` <===取第 28 位 8

:00470FF4 `88500E`  `mov byte ptr [eax+0E], dl`

:00470FF7 `8D45FC`  `lea eax, dword ptr [ebp-04]`

:00470FFA `E81135F9FF`  `call 00404510`

:00470FFF `8A531C`  `mov dl, byte ptr [ebx+1C]` <===取第 29 位 9,根据不的位置取出注册码的值(构成 2450124560136789)

:00471002 `88500F`  `mov byte ptr [eax+0F], dl`

:00471005 `8B45FC`  `mov eax, dword ptr [ebp-04]`

:00471008 `E8DFFAFFFF`  `call 00470AEC` <===特殊的处理的 CALL(下面还会用到的) ，所有的迷都在这里面只是 F8 跟进了:-)

:0047100D `8945F0`  `mov dword ptr [ebp-10], eax` <===得出值 1，应该为 `0x2535022E`，根据 `call 00470AEC` 的功能推出来，`EAX` 的值对应的是取出的注册码的后 8 位在内定字符串 `"HACKERYouMUSTDie"` 的位置减 1，所以： `2535022E`

`2+1=3`  --->对应 `"HACKERYouMUSTDie"` 的第 3 位--->`"C"` --->对应的就是注册码的第 16 位

`5+1=6`  --->对应 `"HACKERYouMUSTDie"` 的第 6 位--->`"R"` --->对应的就是注册码的第 20 位

`3+1=4`  --->对应 `"HACKERYouMUSTDie"` 的第 4 位--->`"K"` --->对应的就是注册码的第 21 位

`5+1=6`  --->对应 `"HACKERYouMUSTDie"` 的第 6 位--->`"R"` --->对应的就是注册码的第 23 位

`0+1=1`  --->对应 `"HACKERYouMUSTDie"` 的第 1 位--->`"H"` --->对应的就是注册码的第 26 位

`2+1=3`  --->对应 `"HACKERYouMUSTDie"` 的第 3 位--->`"C"` --->对应的就是注册码的第 27 位

`2+1=3`  --->对应 `"HACKERYouMUSTDie"` 的第 3 位--->`"C"` --->对应的就是注册码的第 28 位

`E+1=F`  --->对应 `"HACKERYouMUSTDie"` 的第 15 位--->`"i"` --->对应的就是注册码的第 29 位

:00471010 `8955F4`  `mov dword ptr [ebp-0C], edx` <===得出值 2 (下面将用到)，从下面推出结果应该为 `0x354310AC`，再根据 `call 00470AEC` 的功能推出来，`EDX` 的值对应的是取出的注册码的前 8 位在内定字符串 `"HACKERYouMUSTDie"` 的位置减 1，所以： `354310AC`

`3+1=4`  --->对应 `"HACKERYouMUSTDie"` 的第 4 位--->`"K"` --->对应的就是注册码的第 2 位

`5+1=6`  --->对应 `"HACKERYouMUSTDie"` 的第 6 位--->`"R"` --->对应的就是注册码的第 4 位

`4+1=5`  --->对应 `"HACKERYouMUSTDie"` 的第 5 位--->`"E"` --->对应的就是注册码的第 5 位

`3+1=4`  --->对应 `"HACKERYouMUSTDie"` 的第 4 位--->`"K"` --->对应的就是注册码的第 10 位

`1+1=2`  --->对应 `"HACKERYouMUSTDie"` 的第 2 位--->`"A"` --->对应的就是注册码的第 11 位

`0+1=1`  --->对应 `"HACKERYouMUSTDie"` 的第 1 位--->`"H"` --->对应的就是注册码的第 12 位

`A+1=B`  --->对应 `"HACKERYouMUSTDie"` 的第 11 位--->`"U"` --->对应的就是注册码的第 14 位

`C+1=D`  --->对应 `"HACKERYouMUSTDie"` 的第 13 位--->`"T"` --->对应的就是注册码的第 15 位

推出注册码前 34 位就是 `"1K3RE6789KAH3UTC789RK2R45HCCi01234"`

再根据 `00470BF0` 和 `00470BFB` 行，得出最后两位 `oU`

:00471013 `8D45F8`  `lea eax, dword ptr [ebp-08]`

:00471016 `BA10000000`  `mov edx, 00000010`

:0047101B `E82436F9FF`  `call 00404644`

:00471020 `BE01000000`  `mov esi, 00000001`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

| :0047103C(C)
|

:00471025 `8D45F8`  `lea eax, dword ptr [ebp-08]`

:00471028 `E8E334F9FF`  `call 00404510`

`* Possible StringData Ref from Code Obj -> "HACKERYouMUSTDie"`

:0047102D `BA04114700`  `mov edx, 00471104`

:00471032 `8A12`  `mov dl, byte ptr [edx]`

:00471034 `885430FF`  `mov byte ptr [eax+esi-01], dl` <===晕，这个循环无非就是给出 16 个 H

:00471038 `46`  `inc esi`

:00471039 `83FE11`  `cmp esi, 00000011` <===共循环 16 次

:0047103C `75E7`  `jne 00471025` <===从这里向上跳构成循环结构

:0047103E `8D45F8`  `lea eax, dword ptr [ebp-08]` <===>"HHHHHHHHHHHHHHHH"

:00471041 `E8CA34F9FF`  `call 00404510`

:00471046 `8A531F`  `mov dl, byte ptr [ebx+1F]` <===取第 32 位 2

:00471049 `88500D`  `mov byte ptr [eax+0D], dl` <===放入 H 串的第 14 位 `"HHHHHHHHHHHHH2HH"`

:0047104C `8D45F8`  `lea eax, dword ptr [ebp-08]`

:0047104F `E8BC34F9FF`  `call 00404510`

:00471054 `8A5320`  `mov dl, byte ptr [ebx+20]` <===取第 33 位 3

:00471057 `88500E`  `mov byte ptr [eax+0E], dl` <===放入 H 串的第 15 位 `"HHHHHHHHHHHHH23H"`

:0047105A `8D45F8`  `lea eax, dword ptr [ebp-08]`

:0047105D `E8AE34F9FF`  `call 00404510`

:00471062 `8A5321`  `mov dl, byte ptr [ebx+21]` <===取第 34 位 4

:00471065 `88500F`  `mov byte ptr [eax+0F], dl` <===放入 H 串的第 16 位 `"HHHHHHHHHHHHH234"`

:00471068 `8B45F8`  `mov eax, dword ptr [ebp-08]`

:0047106B `E87CFAFFFF`  `call 00470AEC` <===又是这个特殊处理的 CALL，这次处理的是 `"HHHHHHHHHHHHH234"`

:00471070 `8945E8`  `mov dword ptr [ebp-18], eax` <===得出值 3 为 0

` ******经过仔细观察这里的 EAX 的值就是后三位字符在"HACKERYouMUSTDie"中的位置减 1，例如：后三位为 Die，那么这里的 EAX=DEF，如果是 ACK，那么 EAX=123(也就是说 EAX 对应的只是"HHHHHHHHHHHHH234"的后 8 位)`

:00471073 `8955EC`  `mov dword ptr [ebp-14], edx` <===得出值 4 为 0 ，这里 `EDX` 始终为 0(对应 `"HHHHHHHHHHHHH234"` 中的前 8 位，因为都是 H，所以 `EDX` 必定等于 0)

:00471076 `33DB`  `xor ebx, ebx`

:00471078 `8B45F0`  `mov eax, dword ptr [ebp-10]` <===取出值 1 为 0，要想正确注册码则 `[ebp-10]=0x2535022E`，向上看 `0047100D` 行

:0047107B `8B55F4`  `mov edx, dword ptr [ebp-0C]` <===取出值 2 为 0，`[ebp-0C]` 就应该为 `0x354310AC`，再向上看 `00471010` 行

:0047107E `2B45E8`  `sub eax, dword ptr [ebp-18]` <===由于注册码的第 32、33、34 为 234，所以 `[ebp-18]=0`，那么 `EAX=0x2535022E`，再向上看 `00471078`

:00471081 `1B55EC`  `sbb edx, dword ptr [ebp-14]` <===由于 `[ebp-14]` 始终为 0，所以 `EDX` 应该为 `0x354310AC`,向上看 `0047107B` 行

:00471084 `8945F0`  `mov dword ptr [ebp-10], eax` <===算出的结果又放入特殊位 1，则 `EAX` 应该为 `0x2535022E`

:00471087 `8955F4`  `mov dword ptr [ebp-0C], edx` <===算出的结果又放入特殊位 2，则 `EDX` 应该为 `0x354310AC`

:0047108A `FF35D4674700`  `push dword ptr [004767D4]` <===内定值？ 为 0

:00471090 `FF35D0674700`  `push dword ptr [004767D0]` <===内定值？ 为 `0x31EC0A2`

:00471096 `6A00`  `push 00000000`

:00471098 `6A01`  `push 00000001`

:0047109A `FF35DC674700`  `push dword ptr [004767DC]` <===内定值？ 为 0

:004710A0 `FF35D8674700`  `push dword ptr [004767D8]` <===内定值？ 为 `0x12C05687`

:004710A6 `E8B5F8FFFF`  `call 00470960` <===进行处理后，得出不同的 `EAX` 和 `EDX` 值

:004710AB `8945E0`  `mov dword ptr [ebp-20], eax` <===得出值 5 为 `031EC0A2` (因为是内定值，所以这里也是固定的)

:004710AE `8955E4`  `mov dword ptr [ebp-1C], edx` <===得出值 6 为 0 (因为是内定值，所以这里也是固定的)

:004710B1 `8B45F0`  `mov eax, dword ptr [ebp-10]`

:004710B4 `8B55F4`  `mov edx, dword ptr [ebp-0C]`

:004710B7 `3345E0`  `xor eax, dword ptr [ebp-20]` <===`[ebp-20]=0x31EC0A2`(定值)，推出 `EAX` 为 `0x2535022E`,向上看 `00471084` 行

:004710BA `3355E4`  `xor edx, dword ptr [ebp-1C]` <===`[ebp-1C]` 因为是定值为 0，所以 `EDX` 应该为 `0x354310AC`，向上看 `471087` 行

:004710BD `3B15E4674700`  `cmp edx, dword ptr [004767E4]` <===这里也必须相等，而 `[004767E4]` 却为 `0x354310AC`，所以 `EDX` 也应该为 `0x354310AC`，向上看

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

| :004710C3(C)
|

:004710C3 `7506`  `jne 004710CB`

:004710C5 `3B05E0674700`  `cmp eax, dword ptr [004767E0]` <===要想成功，则这里必须两等(两处都必须相等),`[004767E0]` 为 `0x262BC28C`，那么 `EAX` 也应该为 `0x262BC28C`，向上看 `004710B7` 行

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

| :004710CB(C)
|

:004710CB `7502`  `jne 004710CF` <===这里不能跳了

:004710CD `B301`  `mov bl, 01` <===世上最高兴的事莫于经过此处

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

| :004710CB(C)
|

:004710CF `33C0`  `xor eax, eax`

:004710D1 `5A`  `pop edx`



```
:004710D2 59                  pop ecx
:004710D3 59                  pop ecx
:004710D4 648910              mov dword ptr fs:[eax], edx
:004710D7 68F1104700          push 004710F1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004710EF(U)
|
:004710DC 8D45F8              lea eax, dword ptr [ebp-08]
:004710DF BA02000000          mov edx, 00000002
:004710E4 E8332FF9FF          call 0040401C
:004710E9 C3                 ret
:004710EA E90D29F9FF          jmp 004039FC
:004710EF EBEB               jmp 004710DC
:004710F1 8BC3               mov eax, ebx  <===最后一处关键的赋值，向上看 EBX 的值
:004710F3 5E                 pop esi
:004710F4 5B                 pop ebx
:004710F5 8BE5               mov esp, ebp
:004710F7 5D                 pop ebp
:004710F8 C3                 ret

-----------所有的迷都在这里面只是 F8 跟进了:-)------------------------------
```

```
:00470AEC 53                 push ebx
:00470AED 56                 push esi
:00470AEE 57                 push edi
:00470AEF 55                 push ebp
:00470AF0 83C4F8             add esp, FFFFFFF8
:00470AF3 8BF8               mov edi, eax
:00470AF5 BDE07B4700         mov ebp, 00477BE0
:00470AFA C7042400000000     mov dword ptr [esp], 00000000
:00470B01 C744240400000000   mov [esp+04], 00000000
:00470B09 BE10000000         mov esi, 00000010

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470B53(C)       ************外部循环************
|
:00470B0E BB01000000         mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470B4E(C)    ************内部循环************
|
:00470B13 8A4437FF          mov al, byte ptr [edi+esi-01]  <===依次反向取 2450124560136789 的字符

* Possible StringData Ref from Code Obj ->"HACKERYouMUSTDie"
                        |
:00470B17 BA6C0B4700         mov edx, 00470B6C
:00470B1C 3A441AFF          cmp al, byte ptr [edx+ebx-01]  <===依次取 HACKERYouMUSTDie 的字符
:00470B20 7528               jne 00470B4A   <===如果不相等就跳出不做任何处理
:00470B22 8BC3               mov eax, ebx
:00470B24 48                 dec eax
:00470B25 99                 cdq
:00470B26 52                 push edx
:00470B27 50                 push eax
:00470B28 B811000000         mov eax, 00000011
:00470B2D 2BC6               sub eax, esi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470AE0(C)
|
:00470B2F 8B54C5FC          mov edx, dword ptr [ebp+8*eax-04] <===有个列表每隔 8 位一个数字<对应下面码表的高 4 位>
:00470B33 8B44C5F8          mov eax, dword ptr [ebp+8*eax-08] <===每隔 8 位一个数字<对应下面码表的低 4 位>
```

---------------码表(这是一个极有意思的码表)（共 16 行）---------------------------
```
00477BE0  01 00 00 00 00 00 00 00  .....
00477BE8  10 00 00 00 00 00 00 00  .....
00477BF0  00 01 00 00 00 00 00 00  .....
00477BF8  00 10 00 00 00 00 00 00  .....
00477C00  00 00 01 00 00 00 00 00  .....
00477C08  00 00 10 00 00 00 00 00  .....
00477C10  00 00 00 01 00 00 00 00  .....
00477C18  00 00 00 10 00 00 00 00  .....
00477C20  00 00 00 00 01 00 00 00  .....
00477C28  00 00 00 00 10 00 00 00  .....
00477C30  00 00 00 00 00 01 00 00  .....
00477C38  00 00 00 00 00 10 00 00  .....
00477C40  00 00 00 00 00 00 01 00  .....
00477C48  00 00 00 00 00 00 10 00  .....
00477C50  00 00 00 00 00 00 00 01  .....
00477C58  00 00 00 00 00 00 00 10  .....
```
----------------------------------------------

```
:00470B37 E89042F9FF         call 00404DCC　　　<===一个小变换的 CALL
:00470B3C 030424             add eax, dword ptr [esp]
:00470B3F 13542404           adc edx, dword ptr [esp+04]
:00470B43 890424             mov dword ptr [esp], eax
:00470B46 89542404           mov dword ptr [esp+04], edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00470B20(C)
|
:00470B4A 43                 inc ebx
:00470B4B 83FB11             cmp ebx, 00000011
:00470B4E 75C3               jne 00470B13
      ************内部循环结束************
:00470B50 4E                 dec esi
:00470B51 85F6               test esi, esi
:00470B53 75B9               jne 00470B0E
      ************外部循环结束************
:00470B55 8B0424             mov eax, dword ptr [esp]
:00470B58 8B542404           mov edx, dword ptr [esp+04]
:00470B5C 59                 pop ecx
:00470B5D 5A                 pop edx
:00470B5E 5D                 pop ebp
:00470B5F 5F                 pop edi
:00470B60 5E                 pop esi
:00470B61 5B                 pop ebx
:00470B62 C3                 ret
```

.......
.......

3、找到第 3 个注册码校验点。按照上面的步骤得到 `1K3RE6789KAH3UTC789RK2R45HCCi01234oU` 看起来还真有点像，但很快你会发现，每次退出软件后，这个注册信息就会被删除。呵呵，说明还有问题，想想正确的注册信息总不会这样吧。如何来找到它呢？因为是在退出时删除，所以肯定要对注册表进行操作吧，所用字符串参考找到 `"NOBODY"` (每次退出时就被改成这个)，有两个地方，干脆用 `trw2000` 在这两个地方都设上断点。然后退出 `HTMLock`，很快就会断在 `00471D97`，上下看看，有门了：

.......
.......

--------------------------

```
:00471D54 53                 push ebx
:00471D55 DD05907C4700       fld qword ptr [00477C90]
:00471D5B D81DCC1D4700       fcomp dword ptr [00471DCC]
:00471D61 DFE0               fstsw ax
:00471D63 9E                 sahf
:00471D64 7661               jbe 00471DC7
:00471D66 E8D5FFFFFF          call 00471D40  <===这里一个关键 CALL，F8 跟进
:00471D6B 84C0               test al, al  <===要想正确注册，AL 必须为 1
:00471D6D 7558               jne 00471DC7 <===这里必须跳走
:00471D6F B201               mov dl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00471D13(C)
|
:00471D71 A180D64200         mov eax, dword ptr [0042D680]
:00471D76 E805BAFBFF         call 0042D780
:00471D7B 8BD8               mov ebx, eax
:00471D7D BA02000080         mov edx, 80000002
:00471D82 8BC3               mov eax, ebx
:00471D84 E897BAFBFF         call 0042D820
:00471D89 B101               mov cl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00471D1C(C)       <===呵呵，到这里就意味着注册表的信息要被清空了。向上看
|       :00471D1C 这里一个跳转，也应该向上看

* Possible StringData Ref from Code Obj ->"SOFTWARE\\Atrise\\HTMLock 1.9.3"
                        |
:00471D8B BAD81D4700         mov edx, 00471DD8
:00471D90 8BC3               mov eax, ebx
:00471D92 E8EDBAFBFF         call 0042D884

* Possible StringData Ref from Code Obj ->"NOBODY"
                        |
:00471D97 B9001E4700         mov ecx, 00471E00

* Possible StringData Ref from Code Obj ->"UserName"
                        |
:00471D9C BA101E4700         mov edx, 00471E10
:00471DA1 8BC3               mov eax, ebx
:00471DA3 E878BCFBFF         call 0042DA20

* Possible StringData Ref from Code Obj ->"NOKEY"
                        |
:00471DA8 B9241E4700         mov ecx, 00471E24

* Possible StringData Ref from Code Obj ->"RegKey"
                        |
:00471DAD BA341E4700         mov edx, 00471E34
:00471DB2 8BC3               mov eax, ebx
:00471DB4 E867BCFBFF         call 0042DA20
:00471DB9 8BC3               mov eax, ebx
:00471DBB E830BAFBFF         call 0042D7F0
:00471DC0 8BC3               mov eax, ebx
:00471DC2 E8E114F9FF         call 004032A8
```



# 三、无奈的结局

## 反汇编代码段 1

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00471D64(C), :00471D6D(C)
|
:00471DC7 5B                      pop ebx
:00471DC8 C3                      ret
---------:00471D66 call 00471D40 <===这里一个关键 CALL，F8 跟进----------
:00471D40 8B15787C4700           mov edx, dword ptr [00477C78] <===这里就是我们的注册名 newlaos
:00471D46 A17C7C4700             mov eax, dword ptr [00477C7C] <===注册码 1K3RE6789KAH3UTC789RK2R45HCCi01234oU
:00471D4B E8ECF3FFFF             call 0047113C <===很明显，F8 再跟进
:00471D50 C3                     ret
------------------------------------------------------------
:0047113C 8078062D               cmp byte ptr [eax+06], 2D <===呵呵，注册码的第 7 位应为-，否则就认为是这个暗桩好深呀
:00471140 0F94C0                 sete al
:00471143 C3                     ret
```

最后，真正的注册码就是 `1K3RE6-89KAH3UTC789RK2R45HCCi01234oH`(注：最后两位也有所变化)

原以为可以收工了，但发现生成的加密的文件不正确，就是说更本没有加密。还是在生成加密网页的地方，用 `BPX HMEMCPY` 设断点，绕过上面第 2 个检测点，我又发现了第 4 个检测点。这也是最让我无奈的地方，因为如果要通过第 2 个检测点，就无法通过这个检测点。请看：

## 反汇编代码段 2

```asm
.......
.......
:00472267 B930750000             mov ecx, 00007530
* Possible StringData Ref from Code Obj ->"//Page Begin"
                                  |
:0047226C BA6C264700             mov edx, 0047266C
:00472271 8B45F0                mov eax, dword ptr [ebp-10]
:00472274 E8BBE5FFFF            call 00470834
:00472279 84C0                  test al, al
:0047227B 754A                  jne 004722C7 <===这里跳走，不跳走，就报告临时文件生成错误
:0047227D 8B45F4                mov eax, dword ptr [ebp-0C]
.......
.......
:004722E6 E849E5FFFF            call 00470834
:004722EB 84C0                  test al, al
:004722ED 754A                  jne 00472339 <===这里跳走，不跳走，就报告临时文件生成错误
:004722EF 8B45F4                mov eax, dword ptr [ebp-0C]
.......
.......
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004722ED(C)
|
:00472339 8B45F0                mov eax, dword ptr [ebp-10]
:0047233C 8B8014080000          mov eax, dword ptr [eax+00000814]
:00472342 8945D8                mov dword ptr [ebp-28], eax
:00472345 8D45EC                lea eax, dword ptr [ebp-14]
:00472348 E8AB1CF9FF            call 00403FF8
:0047234D 33DB                  xor ebx, ebx
:0047234F 8B450C                mov eax, dword ptr [ebp+0C] <===我们输入的 78787878(网页文件解锁密码)
:00472352 E8611FF9FF            call 004042B8
:00472357 8BF0                  mov esi, eax
:00472359 85F6                  test esi, esi
:0047235B 7E10                  jle 0047236D
.......
.......(网页加密的代码段，有兴趣的朋友可以研究一下)
* Possible StringData Ref from Code Obj ->"4096"
                                  |
:004725B1 BA08274700            mov edx, 00472708
:004725B6 A17C7C4700            mov eax, dword ptr [00477C7C]
:004725BB E86CE6FFFF            call 00470C2C <===呵呵，第 4 个检测点 CALL，F8 跟进
:004725C0 84C0                  test al, al <===AL 必须为 1
:004725C2 7516                  jne 004725DA <===这里必须跳走
:004725C4 8B45F4                mov eax, dword ptr [ebp-0C]
:004725C7 E8DC0CF9FF            call 004032A8
:004725CC 8B45F0                mov eax, dword ptr [ebp-10]
:004725CF E8D40CF9FF            call 004032A8
:004725D4 C645FB01              mov [ebp-05], 01
:004725D8 EB25                  jmp 004725FF
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004725AF(C), :004725C2(C)
|
:004725DA 8B4DD8                mov ecx, dword ptr [ebp-28]
.......
.......
-----------004725BB call 00470C2C <===呵呵，第 4 个注册码检测点，F8 跟进-----------------
```

## 反汇编代码段 3 (00470C2C)

```asm
:00470C2C 55                    push ebp
:00470C2D 8BEC                  mov ebp, esp
:00470C2F 83C4E4                add esp, FFFFFFE4
:00470C32 53                    push ebx
:00470C33 33C9                  xor ecx, ecx
:00470C35 894DFC                mov dword ptr [ebp-04], ecx
:00470C38 894DE4                mov dword ptr [ebp-1C], ecx
:00470C3B 8BD8                  mov ebx, eax <===1K3RE6-89KAH3UTC789RK2R45HCCi01234oH
:00470C3D 33C0                  xor eax, eax
:00470C3F 55                    push ebp
:00470C40 683C0E4700            push 00470E3C
:00470C45 64FF30                push dword ptr fs:[eax]
:00470C48 648920                mov dword ptr fs:[eax], esp
:00470C4B 8D45E4                lea eax, dword ptr [ebp-1C]
:00470C4E 8BD3                  mov edx, ebx
:00470C50 E83B34F9FF            call 00404090
:00470C55 8D45E4                lea eax, dword ptr [ebp-1C]
:00470C58 BA1E000000            mov edx, 0000001E
:00470C5D E8E239F9FF            call 00404644
:00470C62 E8A9FCFFFF            call 00470910
:00470C67 8D45FC                lea eax, dword ptr [ebp-04]
:00470C6A BA10000000            mov edx, 00000010
:00470C6F E8D039F9FF            call 00404644
:00470C74 8D45FC                lea eax, dword ptr [ebp-04]
:00470C77 E89438F9FF            call 00404510
:00470C7C 8B55E4                mov edx, dword ptr [ebp-1C]
:00470C7F 8A5201                mov dl, byte ptr [edx+01] <===取注册码的第 2 位 K
:00470C82 8810                  mov byte ptr [eax], dl
:00470C84 8D45FC                lea eax, dword ptr [ebp-04]
:00470C87 E88438F9FF            call 00404510
:00470C8C 8B55E4                mov edx, dword ptr [ebp-1C]
:00470C8F 8A5203                mov dl, byte ptr [edx+03] <===取注册码的第 4 位 R
:00470C92 885001                mov byte ptr [eax+01], dl
:00470C95 8D45FC                lea eax, dword ptr [ebp-04]
:00470C98 E87338F9FF            call 00404510
:00470C9D 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CA0 8A5204                mov dl, byte ptr [edx+04] <===取注册码的第 5 位 E
:00470CA3 885002                mov byte ptr [eax+02], dl
:00470CA6 8D45FC                lea eax, dword ptr [ebp-04]
:00470CA9 E86238F9FF            call 00404510
:00470CAE 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CB1 8A5209                mov dl, byte ptr [edx+09] <===取注册码的第 10 位 K
:00470CB4 885003                mov byte ptr [eax+03], dl
:00470CB7 8D45FC                lea eax, dword ptr [ebp-04]
:00470CBA E85138F9FF            call 00404510
:00470CBF 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CC2 8A520A                mov dl, byte ptr [edx+0A] <===取注册码的第 11 位 A
:00470CC5 885004                mov byte ptr [eax+04], dl
:00470CC8 8D45FC                lea eax, dword ptr [ebp-04]
:00470CCB E84038F9FF            call 00404510
:00470CD0 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CD3 8A520B                mov dl, byte ptr [edx+0B] <===取注册码的第 12 位 H
:00470CD6 885005                mov byte ptr [eax+05], dl
:00470CD9 8D45FC                lea eax, dword ptr [ebp-04]
:00470CDC E82F38F9FF            call 00404510
:00470CE1 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CE4 8A520D                mov dl, byte ptr [edx+0D] <===取注册码的第 14 位 U
:00470CE7 885006                mov byte ptr [eax+06], dl
:00470CEA 8D45FC                lea eax, dword ptr [ebp-04]
:00470CED E81E38F9FF            call 00404510
:00470CF2 8B55E4                mov edx, dword ptr [ebp-1C]
:00470CF5 8A520E                mov dl, byte ptr [edx+0E] <===取注册码的第 15 位 T
:00470CF8 885007                mov byte ptr [eax+07], dl
:00470CFB 8D45FC                lea eax, dword ptr [ebp-04]
:00470CFE E80D38F9FF            call 00404510
:00470D03 8B55E4                mov edx, dword ptr [ebp-1C]
```



```assembly
:00470D06 8A520F             mov dl, byte ptr [edx+0F]   <===取注册码的第 16 位 C
:00470D09 885008             mov byte ptr [eax+08], dl
:00470D0C 8D45FC             lea eax, dword ptr [ebp-04]
:00470D0F E8FC37F9FF         call 00404510
:00470D14 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D17 8A5213             mov dl, byte ptr [edx+13]   <===取注册码的第 20 位 R
:00470D1A 885009             mov byte ptr [eax+09], dl
:00470D1D 8D45FC             lea eax, dword ptr [ebp-04]
:00470D20 E8EB37F9FF         call 00404510
:00470D25 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D28 8A5214             mov dl, byte ptr [edx+14]   <===取注册码的第 21 位 K
:00470D2B 88500A             mov byte ptr [eax+0A], dl
:00470D2E 8D45FC             lea eax, dword ptr [ebp-04]
:00470D31 E8DA37F9FF         call 00404510
:00470D36 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D39 8A5216             mov dl, byte ptr [edx+16]   <===取注册码的第 23 位 R
:00470D3C 88500B             mov byte ptr [eax+0B], dl
:00470D3F 8D45FC             lea eax, dword ptr [ebp-04]
:00470D42 E8C937F9FF         call 00404510
:00470D47 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D4A 8A5219             mov dl, byte ptr [edx+19]   <===取注册码的第 26 位 H
:00470D4D 88500C             mov byte ptr [eax+0C], dl
:00470D50 8D45FC             lea eax, dword ptr [ebp-04]
:00470D53 E8B837F9FF         call 00404510
:00470D58 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D5B 8A521A             mov dl, byte ptr [edx+1A]   <===取注册码的第 27 位 C
:00470D5E 88500D             mov byte ptr [eax+0D], dl
:00470D61 8D45FC             lea eax, dword ptr [ebp-04]
:00470D64 E8A737F9FF         call 00404510
:00470D69 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D6C 8A521B             mov dl, byte ptr [edx+1B]   <===取注册码的第 28 位 C
:00470D6F 88500E             mov byte ptr [eax+0E], dl
:00470D72 8D45FC             lea eax, dword ptr [ebp-04]
:00470D75 E89637F9FF         call 00404510
:00470D7A 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D7D 8A521C             mov dl, byte ptr [edx+1C]   <===取注册码的第 29 位 i
:00470D80 88500F             mov byte ptr [eax+0F], dl
:00470D83 8B45FC             mov eax, dword ptr [ebp-04]  <===EAX=KREKAHUTCRKRHCCi
:00470D86 E861FDFFFF         call 00470AEC          <===这里的 CALL 和上面是一样的
:00470D8B 8945F0             mov dword ptr [ebp-10], eax  <===eax=0x2535022E
:00470D8E 8955F4             mov dword ptr [ebp-0C], edx  <===edx=0x354310AC
:00470D91 837DF400           cmp dword ptr [ebp-0C], 00000000
:00470D95 7508               jne 00470D9F           <===这里跳走
:00470D97 837DF000           cmp dword ptr [ebp-10], 00000000
:00470D9B 7317               jnb 00470DB4
:00470D9D EB02               jmp 00470DA1
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470D95(C)

|

```assembly
:00470D9F 7D13               jge 00470DB4           <===到这里再跳
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470D9D(U)

|

```assembly
:00470DA1 8B45F0             mov eax, dword ptr [ebp-10]
:00470DA4 8B55F4             mov edx, dword ptr [ebp-0C]
:00470DA7 F7D8               neg eax
:00470DA9 83D200             adc edx, 00000000
:00470DAC F7DA               neg edx
:00470DAE 8945F0             mov dword ptr [ebp-10], eax
:00470DB1 8955F4             mov dword ptr [ebp-0C], edx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470D9B(C), :00470D9F(C)

|

```assembly
:00470DB4 C745E802000000     mov [ebp-18], 00000002   <===跳到这里
:00470DBB C745EC00000000     mov [ebp-14], 00000000
:00470DC2 B301               mov bl, 01             <===一个关键的赋值
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470E15(C), :00470E1A(C)

|

```assembly
:00470DC4 FF75EC             push [ebp-14]    <===0
:00470DC7 FF75E8             push [ebp-18]    <===2
:00470DCA FF75EC             push [ebp-14]    <===0
:00470DCD FF75E8             push [ebp-18]    <===2
:00470DD0 FF75EC             push [ebp-14]    <===0
:00470DD3 FF75E8             push [ebp-18]    <===2
:00470DD6 8B45E8             mov eax, dword ptr [ebp-18]  <===EAX=2
:00470DD9 8B55EC             mov edx, dword ptr [ebp-14]  <===EDX=0
:00470DDC E8EB3FF9FF         call 00404DCC
:00470DE1 E8E63FF9FF         call 00404DCC
:00470DE6 E8E13FF9FF         call 00404DCC  <===进行 3 次同样的计算，等效于 EAX 的值*2*2*2=0x10
:00470DEB 3B55F4             cmp edx, dword ptr [ebp-0C]  <===[ebp-0C]=0x354310AC (要通过第 2 个检测点，则这里只可能是这个值，不可能有其它值)，EDX=0
:00470DEE 7507               jne 00470DF7     <===这里肯定要跳走了
:00470DF0 3B45F0             cmp eax, dword ptr [ebp-10]  <===eax=0x2535022E(这里可以有一些变化，但又影响不到大局)
:00470DF3 7604               jbe 00470DF9
:00470DF5 EB27               jmp 00470E1E     <===如果能这里跳走自然就成功了
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470DEE(C)

|

```assembly
:00470DF7 7F25               jg 00470E1E      <===什么时候从这里跳走呢？(我试了一下如果将 0x354310AC 改为 0xF54310AC 的话，就可以通过了，但要通过第 2 个检测点的话，就不可能有其它值)
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470DF3(C)

|

```assembly
:00470DF9 8345E801           add dword ptr [ebp-18], 00000001
:00470DFD 8355EC00           adc dword ptr [ebp-14], 00000000
:00470E01 FF75EC             push [ebp-14]       <===0
:00470E04 FF75E8             push [ebp-18]       <===3
:00470E07 8B45F0             mov eax, dword ptr [ebp-10]  <===eax=0x2535022E
:00470E0A 8B55F4             mov edx, dword ptr [ebp-0C]  <===edx=0x354310AC
:00470E0D E85A40F9FF         call 00404E6C
:00470E12 83FA00             cmp edx, 00000000
:00470E15 75AD               jne 00470DC4      <===向上跳，构成一个循环结构
:00470E17 83F800             cmp eax, 00000000
:00470E1A 75A8               jne 00470DC4      <===向上跳，构成一个循环结构
:00470E1C 33DB               xor ebx, ebx    <===世上最痛苦的事莫于经过此处(也就是说当 EAX 和 EDX 都为 0 时就 OVER 了)
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470DF5(U), :00470DF7(C)

|

```assembly
:00470E1E 33C0               xor eax, eax
:00470E20 5A                 pop edx
:00470E21 59                 pop ecx
:00470E22 59                 pop ecx
:00470E23 648910             mov dword ptr fs:[eax], edx
:00470E26 68430E4700         push 00470E43
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470E41(U)

|

```assembly
:00470E2B 8D45E4             lea eax, dword ptr [ebp-1C]
:00470E2E E8C531F9FF         call 00403FF8
:00470E33 8D45FC             lea eax, dword ptr [ebp-04]
:00470E36 E8BD31F9FF         call 00403FF8
:00470E3B C3                 ret
:00470E3C E9BB2BF9FF         jmp 004039FC
:00470E41 EBE8               jmp 00470E2B
:00470E43 8BC3               mov eax, ebx     <===EAX 必须为 1
:00470E45 5B                 pop ebx
:00470E46 8BE5               mov esp, ebp
:00470E48 5D                 pop ebp
:00470E49 C3                 ret
```

.......
.......

已经到了最后一关，却出现这个无奈的结局! 不知是我的水平有限，某个地方没有分析正确，还是作者就是故意留下的陷井，自相矛盾无法注册(当你汇去款后，再给你正式版。)

不能得到完全正确的注册算法，写注册机也就没有意思了。干脆来个完美爆破版

```assembly
00470C21 8A45FF    mov al, byte ptr [ebp-01] ------>改为 B00190    mov al,01    nop
```



`0047220F 7556 jne 00472267` ------> 改为 `7456 je 00472267`

`004725C2 7516 jne 004725DA` ------> 改为 `7416 je 004725DA`

`00471D6D 7558 jne 00471DC7` ------> 改为 `7458 je 00471DC7`

```
4dd466d0e1e0cc123afbe504da2462ca
```
