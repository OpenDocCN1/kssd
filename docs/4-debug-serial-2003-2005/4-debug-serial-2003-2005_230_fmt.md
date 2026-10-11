
# QuickCD 1.0.320

> 原文：[`www.pediy.com/kssd/pediy05/pediy50522.htm`](https://www.pediy.com/kssd/pediy05/pediy50522.htm)

QuickCD 1.0.320 破解手记--算法分析

作者：`newlaos[DFCG]`

软件名称：`QuickCD 1.0.320`(系统设置)

整理日期：2003.3.28(华军网)

最新版本：1.0.320

文件大小：59KB

软件授权：共享软件

使用平台：`Win9x/Me/NT/2000/XP`

发布公司：[`www.websamba.com/morequick`](http://www.websamba.com/morequick)

软件简介：QuickCD 可以帮助你方便快捷的打开关闭光驱门,你可以选择热键或者鼠标点击两种方式:1.热键(`F9`:打开光驱门,`F10`:关闭光驱门,`F11`:弹出设置对话框),当然你可以设置新的热键.2.鼠标点击(点击托盘图标:打开光驱门,右击托盘图标:关闭光驱门,右双击托盘图标:弹出菜单)

加密方式：注册码

功能限制：未注册信息提示

PJ 工具：`TRW20001.23` 注册版，`W32Dasm8.93` 黄金版，`FI2.5`，`eXeScope6.30`

PJ 日期：2003-03-31

作者 `newlaos` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 1、先用 FI2.5 看一下主文件“QuickCD.exe”，没加壳。程序是用 VC++6.0 编的

## 2、用 W32Dasm8.93 黄金版对 QuickCD.exe 进行静态反汇编，再用串式数据参考，找不到什么经典的句子，怎么办？先用 eXeScope6.30 对文件的资源进行分析，在“资源\字串表\1”，可以看见：

`7,注册成功`

`8,注册码错误!`

`9,感谢你的注册,我们以后将为你提供更好的服务!`

再回到 `W32Dasm8.93`，找到 `String Resource ID=00007: "鑼?"`(这就是注册成功)

双击有很多，很难找出那一个才是关键的部分。

## 3、再用 TRW20001.23 注册版进行动态跟踪，发现位 0040227E 处的才是关键部分，重新下断 BPX 004021ED(通常在注册成功与否的前面一些下断，这样，才能找到关键部分)

先输入假码: `78787878`

```
:004021ED 55                 push ebp
:004021EE 56                 push esi
:004021EF 57                 push edi
:004021F0 8BF1               mov esi, ecx
:004021F2 89442414           mov dword ptr [esp+14], eax
:004021F6 C744242000000000   mov [esp+20], 00000000
:004021FE 8944240C           mov dword ptr [esp+0C], eax
:00402202 8D442414           lea eax, dword ptr [esp+14]
:00402206 8D8E98000000       lea ecx, dword ptr [esi+00000098]
:0040220C 50                 push eax
:0040220D C644242401         mov [esp+24], 01
:00402212 E843860000         call 0040A85A
:00402217 8D4C240C           lea ecx, dword ptr [esp+0C]
:0040221B 51                 push ecx
:0040221C 8D4E5C             lea ecx, dword ptr [esi+5C]
:0040221F E836860000         call 0040A85A
:00402224 8B54240C           mov edx, dword ptr [esp+0C]
:00402228 837AF801           cmp dword ptr [edx-08], 00000001
:0040222C 0F8EB2000000       jle 004022E4
:00402232 8B442414           mov eax, dword ptr [esp+14] <===EAX=20608611(机器码)，此时，ECX=78787878
:00402236 50                 push eax
:00402237 E81B180000         call 00403A57    <===机器码进行第一次变形，EAX=13A7663
:0040223C 8B4C2410           mov ecx, dword ptr [esp+10] <===ECX=78787878
:00402240 8BF8               mov edi, eax
:00402242 51                 push ecx
:00402243 E80F180000         call 00403A57    <===将输入的注册码也做同样的变形处理
:00402248 83C408             add esp, 00000008
:0040224B 8BCE               mov ecx, esi
:0040224D 8BE8               mov ebp, eax      <===EBP=变形后的注册码
:0040224F 57                 push edi
:00402250 E8EB020000         call 00402540    <===将变形后的机器码再做一次变形
:00402255 3BC5               cmp eax, ebp
  <===进行对比，EAX 为两次变形后的机器码(5B3D7CC)，EBP 为一次变形的注册码(4B2356)，两者必须相等，注册才能成功，开始分析算法
:00402257 0F8587000000       jne 004022E4        <===关键跳转，跳过去，就 OVER 了

* Possible Reference to Dialog: DialogID_0084, CONTROL_ID:03F2, ""
                  |
:0040225D 68F2030000         push 000003F2
:00402262 8BCE               mov ecx, esi
:00402264 E8F1A40000         call 0040C75A
:00402269 8BF8               mov edi, eax
:0040226B 6A00               push 00000000
:0040226D 8BCF               mov ecx, edi
:0040226F E88EA60000         call 0040C902
:00402274 8B1524B54100       mov edx, dword ptr [0041B524]
:0040227A 89542410           mov dword ptr [esp+10], edx

* Possible Reference to String Resource ID=00007: "鑼?"<==="注册成功"
                  |
:0040227E 6A07               push 00000007
:00402280 8D4C2414           lea ecx, dword ptr [esp+14]
:00402284 C644242402         mov [esp+24], 02
:00402289 E87ABF0000         call 0040E208
:0040228E 8B442410           mov eax, dword ptr [esp+10]
:00402292 8BCF               mov ecx, edi
:00402294 50                 push eax
:00402295 E8B0A50000         call 0040C84A

* Possible Reference to Dialog: DialogID_0084, CONTROL_ID:03F1, ""
                  |
:0040229A 68F1030000         push 000003F1
:0040229F 8BCE               mov ecx, esi
:004022A1 E8B4A40000         call 0040C75A
:004022A6 6A00               push 00000000
:004022A8 8BC8               mov ecx, eax
:004022AA E853A60000         call 0040C902
:004022AF E839020100         call 004124ED
:004022B4 8B4004             mov eax, dword ptr [eax+04]
:004022B7 55                 push ebp

* Possible StringData Ref from Data Obj ->"RKey"
                  |
:004022B8 68F4B14100         push 0041B1F4

* Possible StringData Ref from Data Obj ->"Settings"
                  |
:004022BD 68E8B14100         push 0041B1E8
:004022C2 8BC8               mov ecx, eax
:004022C4 E8F0D50000         call 0040F8B9
:004022C9 6AFF               push FFFFFFFF
:004022CB 6A00               push 00000000

* Possible Reference to String Resource ID=00009: ""`勮?戾:`袥魙??"
                  |        <==="感谢你的注册,我们以后将为你提供更好的服务!"
:004022CD 6A09               push 00000009
:004022CF E8E4D40000         call 0040F7B8
:004022D4 8D4C2410           lea ecx, dword ptr [esp+10]
:004022D8 C644242001         mov [esp+20], 01
:004022DD E8DCBD0000         call 0040E0BE
:004022E2 EB0B               jmp 004022EF

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040222C(C), :00402257(C)
|
:004022E4 6AFF               push FFFFFFFF
:004022E6 6A00               push 00000000

* Possible Reference to String Resource ID=00008: "鑼?"<==="注册码错误!"
                  |
:004022E8 6A08               push 00000008
:004022EA E8C9D40000         call 0040F7B8

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004022E2(U)
|
:004022EF 8D4C240C           lea ecx, dword ptr [esp+0C]
:004022F3 C644242000         mov [esp+20], 00
:004022F8 E8C1BD0000         call 0040E0BE
:004022FD 8D4C2414           lea ecx, dword ptr [esp+14]
:00402301 C7442420FFFFFFFF   mov [esp+20], FFFFFFFF
:00402309 E8B0BD0000         call 0040E0BE
:0040230E 8B4C2418           mov ecx, dword ptr [esp+18]
:00402312 5F                 pop edi
:00402313 5E                 pop esi
:00402314 5D                 pop ebp
:00402315 64890D00000000     mov dword ptr fs:[00000000], ecx
:0040231C 83C418             add esp, 00000018
:0040231F C3                 ret

```

`---------00402237 call 00403A57` 一样数据变形处理的 `CALL`，`F8` 跟进

注：机器码和输入的注册码都要做的同样的变形处理，变形处理后的值返回在 `EAX` 上

```
:00403A57 53                 push ebx
:00403A58 55                 push ebp
:00403A59 56                 push esi
:00403A5A 57                 push edi
:00403A5B 8B7C2414           mov edi, dword ptr [esp+14]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403A8B(U)
|
:00403A5F 833D24BE410001     cmp dword ptr [0041BE24], 00000001
:00403A66 7E0F               jle 00403A77  <===这里跳走
:00403A68 0FB607             movzx eax, byte ptr [edi]

* Possible Reference to String Resource ID=00008: "鑼?"  <==="注册码错误!"
                  |
:00403A6B 6A08               push 00000008
```



:00403A6D 50 push eax

:00403A6E E872340000 call 00406EE5

:00403A73 59 pop ecx

:00403A74 59 pop ecx

:00403A75 EB0F jmp 00403A86

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403A66(C)`

:00403A77 0FB607 movzx eax, byte ptr `[edi]` <===跳来这里

* Possible StringData Ref from Data Obj ->" ((((( "
->" H"
|

:00403A7A 8B0D18BC4100 mov ecx, dword ptr `[0041BC18]`

:00403A80 8A0441 mov al, byte ptr `[ecx+2*eax]`

:00403A83 83E008 and eax, 00000008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403A75(U)`

:00403A86 85C0 test eax, eax

:00403A88 7403 je 00403A8D <===这里再次跳走(好象是检验机器码是否被修改)

:00403A8A 47 inc edi

:00403A8B EBD2 jmp 00403A5F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403A88(C)`

:00403A8D 0FB637 movzx esi, byte ptr `[edi]` <===跳到这里

:00403A90 47 inc edi

:00403A91 83FE2D cmp esi, 0000002D <===“=”字符检测

:00403A94 8BEE mov ebp, esi

:00403A96 7405 je 00403A9D

:00403A98 83FE2B cmp esi, 0000002B <===“+”字符检测

:00403A9B 7504 jne 00403AA1 <===从这里跳走

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403A96(C)`

:00403A9D 0FB637 movzx esi, byte ptr `[edi]`

:00403AA0 47 inc edi

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403A9B(C)`

:00403AA1 33DB xor ebx, ebx <===跳到这里，EBX 清 0，准备存贮计算的最终值

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403AD2(U)` <===这一行开始循环

:00403AA3 833D24BE410001 cmp dword ptr `[0041BE24]`, 00000001 <===判断循环结束的标志

:00403AAA 7E0C jle 00403AB8 <===从这里跳走，就说明循环未结束

* Possible Reference to String Resource ID=00004: "
(X?:?,1 伨 n9軫勲 w. !"
|

:00403AAC 6A04 push 00000004

:00403AAE 56 push esi

:00403AAF E831340000 call 00406EE5

:00403AB4 59 pop ecx

:00403AB5 59 pop ecx

:00403AB6 EB0B jmp 00403AC3 <===循环结束，则从这里跳走

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403AAA(C)`

* Possible StringData Ref from Data Obj ->" ((((( "
->" H"
|

:00403AB8 A118BC4100 mov eax, dword ptr `[0041BC18]` <===循环未结束时跳到这里

:00403ABD 8A0470 mov al, byte ptr `[eax+2*esi]`

:00403AC0 83E004 and eax, 00000004

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403AB6(U)`

:00403AC3 85C0 test eax, eax

:00403AC5 740D je 00403AD4 <===将本 CALL 收进来的值(第一次是机器码 20608611，第二次输入的注册码 78787878)处理完后，从这里跳走

:00403AC7 8D049B lea eax, dword ptr `[ebx+4*ebx]`

:00403ACA 8D5C46D0 lea ebx, dword ptr `[esi+2*eax-30]`

:00403ACE 0FB637 movzx esi, byte ptr `[edi]`

:00403AD1 47 inc edi

:00403AD2 EBCF jmp 00403AA3 <===这里构成一个大循环

```
***机器码 20608611 的循环结果如下：     ***注册码 78787878 的循环结果如下：

     EBX=32+2*(0+4*0)-30=2                   EBX=37+2*(0+4*0)-30=7
     EBX=30+2*(2+4*2)-30=14                  EBX=38+2*(7+4*7)-30=4E
     ...                                      ...
     ...                                      ...
     ...                                      ...
     EBX=31+2*(1F723D+4*1F723D)-30=13A7663   EBX=38+2*(783883+4*783883)-30=4B2356
     ***其实就是机器码 16 进制表示形式        ***其实就是注册码的 16 进制表示形式
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403AC5(C)`

:00403AD4 83FD2D cmp ebp, 0000002D

:00403AD7 8BC3 mov eax, ebx

:00403AD9 7502 jne 00403ADD

:00403ADB F7D8 neg eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00403AD9(C)`

:00403ADD 5F pop edi

:00403ADE 5E pop esi

:00403ADF 5D pop ebp

:00403AE0 5B pop ebx

:00403AE1 C3 ret

此段小结：机器处理后的结果是 `13A7663`，注册码处理后的结果是 `4B2356`

----------------------------------------------------------------------------------------

##### 00402250 call 00402540 <===将变形后的机器码 13A7663 再做一次变形

:00402540 8B442404 mov eax, dword ptr `[esp+04]` <===EAX=13A7663

:00402544 35AC0BBB02 xor eax, 02BB0BAC <===EAX=13A7663 xor 02BB0BAC=3817DCF

:00402549 05FD593202 add eax, 023259FD <===EAX=3817DCF + 023259FD=5B3D7CC

:0040254E 7905 jns 00402555

:00402550 99 cdq

:00402551 33C2 xor eax, edx

:00402553 2BC2 sub eax, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040254E(C)`

:00402555 C20400 ret 0004

---------------------------------------------------------------------------------------

### 4. 算法分析

---类型：`f1(机器码)=注册码`---

a. 将机器码和注册码都转为 16 进制表示形式。

b. 将 16 进制表示形式的机器码进行如下处理：

```
机器码 1 = (机器码 xor 02BB0BAC) + 023259FD
```

c. 将机器码 1 与注册码 1(16 进制表示形式)做比较，如果相等，就注册成功。

d. 机器码转为 16 进制后与 `02BB0BAC` 异或运算，再加上 `023259FD`，得到出来的值再转为 10 进制，就是注册码了。我的机器码是 `17421339`，那么注册码就是 `98908596`。

### 5. 注册信息保存在文件 QuickCD.ini 中

```ini
[Settings]
RKey=98908596  4dd466d0e1e0cc123afbe504da2462ca
```
