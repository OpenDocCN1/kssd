
# Dasxx

> 原文：[`www.pediy.com/kssd/pediy05/pediy50174.htm`](https://www.pediy.com/kssd/pediy05/pediy50174.htm)

大老的打狗教程第一篇如解掉 `hasp` 的狗! 希望对大家有所帮助! 大老=`[DCG]`

程序名: 国外的工程类软件 `dasxx`

保护 : `hasp4` `m1` 这是以色列的狗 `m1` 是代表他有储存器

所用工具: `trw2000` `wasm32`

我写的打狗教程这是第一篇! 我共会写 3 篇的! 写第一篇写个网上中文的教程比较少的 `hasp4` 的狗保护的软件!

我只是大体说一下破解的思路!

希望对大家有所帮助!

(1) 第一部分

```
=============================================================================================

:0042659A 50                         push eax
:0042659B 51                         push ecx
:0042659C 52                         push edx
:0042659D 53                         push ebx
:0042659E 68FE3F0000                 push 00003FFE  ===>这就是 hasp 狗读狗时要用到的密码! (1)
:004265A3 687B1D0000                 push 00001D7B  ===>hasp 狗的密码!  (2)
:004265A8 6800000000                 push 00000000
:004265AD 6800000000                 push 00000000
:004265B2 6801000000                 push 00000001
:004265B7 E8A7FBFFFF                 call 00426163  ====>读狗  (1)
:004265BC 83C424                     add esp, 00000024
:004265BF 8B45FC                     mov eax, dword ptr [ebp-04]==> 读狗后返回值=1 就是有狗!
:004265C2 B901000000                 mov ecx, 00000001
:004265C7 39C8                       cmp eax, ecx
:004265C9 0F85EF020000               jne 004268BE     ===> 跳就完蛋
:004265CF 8D45F0                     lea eax, dword ptr [ebp-10]
:004265D2 8D4DF4                     lea ecx, dword ptr [ebp-0C]
:004265D5 8D55F8                     lea edx, dword ptr [ebp-08]
:004265D8 8D5DFC                     lea ebx, dword ptr [ebp-04]
:004265DB 50                         push eax
:004265DC 51                         push ecx
:004265DD 52                         push edx
:004265DE 53                         push ebx
:004265DF 68FE3F0000                 push 00003FFE
:004265E4 687B1D0000                 push 00001D7B
:004265E9 6800000000                 push 00000000
:004265EE 6800000000                 push 00000000
:004265F3 6805000000                 push 00000005
:004265F8 E866FBFFFF                 call 00426163  ========>读狗(2)
:004265FD 83C424                     add esp, 00000024
:00426600 8B45FC                     mov eax, dword ptr [ebp-04] ==> 读狗后返回值=1 就是有狗!
:00426603 B901000000                 mov ecx, 00000001
:00426608 39C8                       cmp eax, ecx
:0042660A 0F85C2010000               jne 004267D2      ===> 跳就完蛋
:00426610 8B45F8                     mov eax, dword ptr [ebp-08]  ===>另外一个返回值
:00426613 39C8                       cmp eax, ecx
:00426615 0F85B7010000               jne 004267D2  ====>跳就完蛋!
:0042661B 8D0518E74500               lea eax, dword ptr [0045E718]
:00426621 8B4DF4                     mov ecx, dword ptr [ebp-0C]
:00426624 668908                     mov word ptr [eax], cx
:00426627 6885510000                 push 00005185
:0042662C 8D05BC614200               lea eax, dword ptr [004261BC]
:00426632 8D4DE0                     lea ecx, dword ptr [ebp-20]
:00426635 51                         push ecx
:00426636 FFD0                       call eax    计算返回的数据
:00426638 83C408                     add esp, 00000008
:0042663B 8B45E0                     mov eax, dword ptr [ebp-20]====>返回数据(1) 正确值是 bb2
:0042663E B9B20B0000                 mov ecx, 00000BB2  这里是要比较的值!
:00426643 39C8                       cmp eax, ecx ===>比较
:00426645 0F8530000000               jne 0042667B ===>跳到报错
:0042664B 8B45E4                     mov eax, dword ptr [ebp-1C] ====>返回数据(2) 正确值是 A6FE
:0042664E B9FEA60000                 mov ecx, 0000A6FE
:00426653 39C8                       cmp eax, ecx ===>比较
:00426655 0F8520000000               jne 0042667B ===>跳到报错
:0042665B 8B45E8                     mov eax, dword ptr [ebp-18] ====>返回数据(3) 正确值是 6A14
:0042665E B9146A0000                 mov ecx, 00006A14
:00426663 39C8                       cmp eax, ecx  ===>比较
:0426665 0F8510000000               jne 0042667B ===>跳到报错 !
:0042666B 8B45EC                     mov eax, dword ptr [ebp-14]====>返回数据(4) 正确值是 714D
:0042666E B94D710000                 mov ecx, 0000714D
:00426673 39C8                       cmp eax, ecx ===>比较 相等的话跳到正确处理流程
:00426675 0F84FC000000               je 00426777 ===>跳到正确处理流程  ===关键(1)====

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00426645(C), :00426655(C), :00426665(C)

:0042667B 8D0552924700               lea eax, dword ptr [00479252]
:00426681 6801000000                 push 00000001
:00426686 50                         push eax
:00426687 6800000000                 push 00000000
:00426687 6800000000                 push 00000000

* Reference To: cvirt.LoadPanel, Ord:0133h

:0042668C E891B3FDFF                 Call 00401A22
:00426691 8D4DDC                     lea ecx, dword ptr [ebp-24]
:00426694 8901                       mov dword ptr [ecx], eax
:00426696 8B45DC                     mov eax, dword ptr [ebp-24]
:00426699 B900000000                 mov ecx, 00000000
:0042669E 39C8                       cmp eax, ecx
:004266A0 0F8D20000000               jnl 004266C6

* Reference To: cvirt.CVI_Beep, Ord:0259h

:004266A6 E845B8FDFF                 Call 00401EF0
:004266AB 8D05EA924700               lea eax, dword ptr [004792EA]
:004266B1 8D0DAA924700               lea ecx, dword ptr [004792AA]
:004266B7 50                         push eax
:004266B8 51                         push ecx

* Reference To: cvirt.MessagePopup, Ord:014Dh  ===>报错信息!

:004266B9 E8CCB7FDFF                 Call 00401E8A
:004266BE 8D056A674200               lea eax, dword ptr [0042676A]
:004266C4 FFE0                       jmp eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004266A0(C)

:004266C6 6800000000                 push 00000000
:004266CB 6812020000                 push 00000212
:004266D0 6803000000                 push 00000003

==================================================================================================
```

你这样处理后运行程序还会有问题的! 看样子是没有解决完! 咱们在来看看!

第二部分

```
===================================================================================================
```

第一部分的程序(===关键(1)====)跳转后就到了这里 `le's go`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00426675(C)

:00426777 E8CBFBFFFF                 call 00426347
:0042677C 8D45FC                     lea eax, dword ptr [ebp-04]
:0042677F B903000000                 mov ecx, 00000003
:00426784 8908                       mov dword ptr [eax], ecx
:00426786 8D4DF0                     lea ecx, dword ptr [ebp-10]
:00426789 8D55F4                     lea edx, dword ptr [ebp-0C]
:0042678C 8D5DF8                     lea ebx, dword ptr [ebp-08]
:0042678F 51                         push ecx
:00426790 52                         push edx
:00426791 53                         push ebx
:00426792 50                         push eax
:00426793 68FE3F0000                 push 00003FFE
:00426798 687B1D0000                 push 00001D7B
:0042679D 6800000000                 push 00000000
:004267A2 6800000000                 push 00000000
:004267A7 6803000000                 push 00000003
:004267AC E8B2F9FFFF                 call 00426163 ====>这里又有一处读狗!
:004267B1 83C424                     add esp, 00000024
:004267B4 8B45F4                     mov eax, dword ptr [ebp-0C] ====>返回值(1)应该是 0
:004267B7 B900000000                 mov ecx, 00000000
:004267BC 39C8                       cmp eax, ecx ===>比较
:004267BE 0F85DE010000               jne 004269A2  不跳
:004267C4 8B45F8                     mov eax, dword ptr [ebp-08]
:004267C7 0FB7C0                     movzx eax, ax
:004267CA 8D0DA7694200               lea ecx, dword ptr [004269A7] 注意这里 ecx 的值是从这里的地址里来的
```



```asm
:004267D0 FFE1                   jmp ecx    =======>跳到下一个部分!go   ====关键 2===

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0042660A(C), :00426615(C)
|
:004267D2 8D0545924700           lea eax, dword ptr [00479245]
:004267D8 6801000000             push 00000001
:004267DD 50                     push eax
:004267DE 6800000000             push 00000000

* Reference To: cvirt.LoadPanel, Ord:0133h
|
:004267E3 E83AB2FDFF             Call 00401A22
:004267E8 8D4DDC                 lea ecx, dword ptr [ebp-24]
:004267EB 8901                   mov dword ptr [ecx], eax
:004267ED 8B45DC                 mov eax, dword ptr [ebp-24]
:004267F0 B900000000             mov ecx, 00000000
:004267F5 39C8                   cmp eax, ecx
:004267F7 0F8D20000000           jnl 0042681D

* Reference To: cvirt.CVI_Beep, Ord:0259h
|
:004267FD E8EEB6FDFF             Call 00401EF0
:00426802 8D05BE924700           lea eax, dword ptr [004792BE]
:00426808 8D0D96924700           lea ecx, dword ptr [00479296]
:0042680E 50                     push eax
:0042680F 51                     push ecx

* Reference To: cvirt.MessagePopup, Ord:014Dh ====>出错信息!
|
:00426810 E875B6FDFF             Call 00401E8A
:00426815 8D05A9684200           lea eax, dword ptr [004268A9]
:0042681B FFE0                   jmp eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004267F7(C)
|
:0042681D 6800000000             push 00000000
:00426822 6812020000             push 00000212
:00426827 6803000000             push 00000003
:0042682C 8B45DC                 mov eax, dword ptr [ebp-24]
:0042682F 50                     push eax

* Reference To: cvirt.SetCtrlAttribute, Ord:00AEh
|
:00426830 E8BFACFDFF             Call 004014F4
:00426835 83C410                 add esp, 00000010
:00426838 6800000000             push 00000000
:0042683D 6812020000             push 00000212
:00426842 6804000000             push 00000004
```

=======================================================================================================

经过上部分!咱们看看下面部分如何! 经过对====关键 2===的跟踪发现!到了下面的程序!

```asm
:0042AFCE 8908                   mov dword ptr [eax], ecx
:0042AFD0 E8B1B5FFFF             call 00426586
:0042AFD5 8D8DE8FEFFFF           lea ecx, dword ptr [ebp+FFFFFEE8]
:0042AFDB 668901                 mov word ptr [ecx], ax
:0042AFDE 668B85E8FEFFFF         mov ax, word ptr [ebp+FFFFFEE8]
:0042AFE5 0FB7C0                 movzx eax, ax
:0042AFE8 B901000000             mov ecx, 00000001
:0042AFED 39C8                   cmp eax, ecx ======注意这个比较
:0042AFEF 0F8432000000           je 0042B027    =====>不跳就 over

* Possible Reference to String Resource ID=65535: "Das32"
|
:0042AFF5 B9FFFF0000             mov ecx, 0000FFFF
:0042AFFA 39C8                   cmp eax, ecx
:0042AFFC 0F8425000000           je 0042B027

* Reference To: cvirt.CVI_Beep, Ord:0259h
|
:0042B002 E8E96EFDFF             Call 00401EF0
:0042B007 8D0504B04700           lea eax, dword ptr [0047B004]
:0042B00D 8D0DAFB34700           lea ecx, dword ptr [0047B3AF]
:0042B013 50                     push eax
:0042B014 51                     push ecx

* Reference To: cvirt.MessagePopup, Ord:014Dh =====出错信息!
|
:0042B015 E8706EFDFF             Call 00401E8A
:0042B01A 6800000000             push 00000000
:0042B01F E82F75FDFF             call 00402553
:0042B024 83C404                 add esp, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0042AFEF(C), :0042AFFC(C)
|
:0042B027 8D45FC                 lea eax, dword ptr [ebp-04] ====正确的流程!
:0042B02A 50                     push eax
:0042B02B 6801000000             push 00000001
```

======================================================================================================

经过了这部分后狗部分就解掉了!

## 总结

上面的部分只是解狗里面的一种而已!想这个软件还有好几种解法!这种解法比较容易理解!呵呵~我就献丑了!希望大家不要笑我!

希望大家经常来我的论坛来看看交流一下!现在有些人对我有意见!哪是不可避免的!也是很正常的!!谢谢大家看完此文! 如果你觉得写的还行请回个贴子!支持一下!谢谢!

如果要转载请保留完整

大老= [DCG]

`dalao@top86.com`

`http://dalao2002.yeah.net`

2002.6.25 `4dd466d0e1e0cc123afbe504da2462ca`
