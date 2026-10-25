
# 侠客系统修改器 1.21

> 原文：[`www.pediy.com/kssd/pediy05/pediy50623.htm`](https://www.pediy.com/kssd/pediy05/pediy50623.htm)

侠客系统修改器 1.21 破解手记--算法分析

作者：`newlaos\[CCG\]\[DFCG\]`

软件名称：侠客系统修改器 1.21(系统设置)
最新版本：1.21
文件大小：604KB
软件授权：共享软件
使用平台：`Win9x/Me/NT/`
发布公司：[`xksoft.yeah.net`](http://xksoft.yeah.net)

软件简介：`Windows 9x/NT` 系统中有很多注册表选项没有直接提供设置界面，如开始菜单项的弹出速度，桌面上系统图标的隐藏，系统安全属性设置等等。当我们需要修改这些选项的时候，就要使用 `Windows` 注册表编辑器来手工修改注册表信息，这样做不仅很麻烦，而且不安全，还需要一定的注册表知识。为了方便大家修改注册表选项，我编写了这个系统设置修改器，可轻松修改 `Windows` 系统中大部分注册表选项，还可以对一些非注册表选项进行设置（如 `IE4` 支持的文件夹图标，电脑启动选项等）。

加密方式：注册码
功能限制：未注册信息提示
`PJ` 工具：`TRW20001.23` 注册版，`W32Dasm8.93` 黄金版，`FI2.5`，`eXeScope6.30`
`PJ` 日期：2003-04-20

作者 `newlaos` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 1、分析与定位

先用 `FI2.5` 看一下主文件 `SysSet.exe`，没加壳。程序是用 `VC++6.0` 编的。

用 `W32Dasm8.93` 黄金版对 `SysSet.exe` 进行静态反汇编，再用串式数据参考，找不到什么经典的句子，怎么办？先用 `eXeScope6.30` 对文件的资源进行分析，在“资源\\字串表\\1”，可以看见：

`136,"    非常感谢您对本软件的支持!!!$0A 再也不会有注册提示对话框出现了，预祝您用的开心。"`
`137,您的注册姓名与注册码不匹配，请检查一下重新输入。`

再回到 `W32Dasm8.93`，找到 `"String Resource ID=00137: "▌鑼?鑼 9M 骼"` (注册失败)，双击来到下面代码段。

## 2、动态跟踪

再用 `TRW20001.23` 注册版进行动态跟踪，下断 `BPX 004240E8` (通常在注册成功与否的前面一些下断，这样，才能找到关键部分)。

先输入注册名：`newlaos`
假码: `78787878`

```
:004240E8 51                           push ecx
:004240E9 8D6E5C                       lea ebp, dword ptr [esi+5C]
:004240EC 8BCC                         mov ecx, esp
:004240EE 89642420                     mov dword ptr [esp+20], esp
:004240F2 55                           push ebp
:004240F3 E8CD5A0200                   call 00449BC5
:004240F8 51                           push ecx
:004240F9 8D7E60                       lea edi, dword ptr [esi+60]
:004240FC 8BCC                         mov ecx, esp
:004240FE 89642428                     mov dword ptr [esp+28], esp
:00424102 57                           push edi
:00424103 C784249001000000000000       mov dword ptr [esp+00000190], 00000000
:0042410E E8B25A0200                   call 00449BC5
:00424113 83CBFF                       or ebx, FFFFFFFF
:00424116 899C248C010000               mov dword ptr [esp+0000018C], ebx
:0042411D E88E040000                   call 004245B0
:00424122 8BC8                         mov ecx, eax
:00424124 E8B7890000                   call 0042CAE0    <===关键的 CALL 了，F8 跟进
:00424129 85C0                         test eax, eax    <===EAX 不能为 0
:0042412B 0F8457010000                 je 00424288      <===关键跳转，
:00424131 6A00                         push 00000000
:00424133 8D4C2434                     lea ecx, dword ptr [esp+34]
:00424137 E8C4020000                   call 00424400
:0042413C BB02000000                   mov ebx, 00000002
:00424141 8D4C2430                     lea ecx, dword ptr [esp+30]
:00424145 899C2484010000               mov dword ptr [esp+00000184], ebx
:0042414C E83A220200                   call 0044638B
:00424151 51                           push ecx
:00424152 8BCC                         mov ecx, esp
:00424154 89642420                     mov dword ptr [esp+20], esp
:00424158 57                           push edi
:00424159 E8675A0200                   call 00449BC5
:0042415E 6A01                         push 00000001
:00424160 889C248C010000               mov byte ptr [esp+0000018C], bl
:00424167 E854040000                   call 004245C0
:0042416C 8BC8                         mov ecx, eax
:0042416E E82DCD0000                   call 00430EA0
:00424173 8D4C2424                     lea ecx, dword ptr [esp+24]
:00424177 E8A4F0FFFF                   call 00423220
:0042417C A154234700                   mov eax, dword ptr [00472354]
:00424181 89442410                     mov dword ptr [esp+10], eax
:00424185 8B0DF85D4700                 mov ecx, dword ptr [00475DF8]
:0042418B 8D44241C                     lea eax, dword ptr [esp+1C]
:0042418F C684248401000005             mov byte ptr [esp+00000184], 05
:00424197 8B11                         mov edx, dword ptr [ecx]
:00424199 52                           push edx
:0042419A 50                           push eax
:0042419B E87020FFFF                   call 00416210
:004241A0 83C408                         add esp, 00000008
```

`Possible StringData Ref from Data Obj ->"\\SysSet.cfg"`

```
:004241A3 68640D4700                   push 00470D64
```

此处删除一段注册信息保存代码

```
:0042427E E8F5230200                   call 00446678
:00424283 E980000000                   jmp 00424308
```

`Referenced by a (U)nconditional or (C)onditional Jump at Address:`
`:0042412B(C)                           <===这里向上就是关键跳转了`

```
:00424288 A154234700                   mov eax, dword ptr [00472354]
:0042428D 89442418                     mov dword ptr [esp+18], eax
:00424291 C784248401000009000000       mov dword ptr [esp+00000184], 00000009
:0042429C 89442414                     mov dword ptr [esp+14], eax
```

`Possible Reference to String Resource ID=00103: "牏?h V1.21"`

```
:004242A0 6A67                         push 00000067
:004242A2 8D4C2418                     lea ecx, dword ptr [esp+18]
:004242A6 C68424880100000A             mov byte ptr [esp+00000188], 0A
:004242AE E835670200                   call 0044A9E8
```

`Possible Reference to String Resource ID=00137: "▌鑼?鑼 9M 骼?`
`"您的注册姓名与注册码不匹配，请检查一下重新输入。"`

```
:004242B3 6889000000                   push 00000089
:004242B8 8D4C241C                     lea ecx, dword ptr [esp+1C]
:004242BC E827670200                   call 0044A9E8
```

`-----00424124 call 0042CAE0    关键的 CALL 了，F8 跟进----------------`

```
:0042CAE0 6AFF                         push FFFFFFFF
:0042CAE2 68B07D4500                   push 00457DB0
:0042CAE7 64A100000000                 mov eax, dword ptr fs:[00000000]
:0042CAED 50                           push eax
:0042CAEE 64892500000000               mov dword ptr fs:[00000000], esp
:0042CAF5 83EC30                       sub esp, 00000030
:0042CAF8 53                           push ebx
:0042CAF9 55                           push ebp
:0042CAFA 56                           push esi
:0042CAFB 57                           push edi
:0042CAFC 33FF                         xor edi, edi
:0042CAFE 897C2448                     mov dword ptr [esp+48], edi
:0042CB02 8B4C2450                     mov ecx, dword ptr [esp+50]
:0042CB06 C644241301                   mov [esp+13], 01
:0042CB0B 8B41F8                       mov eax, dword ptr [ecx-08]
:0042CB0E 83F803                       cmp eax, 00000003
:0042CB11 0F8C74010000                 jl 0042CC8B    <===这里跳了就 OVER 了，注册码长度必须大于 3
:0042CB17 8B442454                     mov eax, dword ptr [esp+54]
:0042CB1B 8378F801                     cmp dword ptr [eax-08], 00000001
:0042CB1F 0F8C66010000                 jl 0042CC8B    <===这里跳了就 OVER 了，注册名长度必须大于 1
```

`Possible StringData Ref from Data Obj ->"opq98"`

```
:0042CB25 68C0144700                   push 004714C0  <===感觉是黑名单
:0042CB2A 51                           push ecx
:0042CB2B E830970000                   call 00436260
:0042CB30 83C408                       add esp, 00000008
:0042CB33 3BC7                         cmp eax, edi
:0042CB35 0F8450010000                 je 0042CC8B <===这里一跳也 OVER，就是黑名单了。
:0042CB3B 8B4C2450                      mov ecx, dword ptr [esp+50]
:0042CB3F 8B542454                      mov edx, dword ptr [esp+54]
:0042CB43 897C2414                      mov dword ptr [esp+14], edi
:0042CB47 8B49F8                        mov ecx, dword ptr [ecx-08]
:0042CB4A 8B42F8                        mov eax, dword ptr [edx-08] <===EAX=8(假码 78787878 的长度)
:0042CB4D 894C2418                      mov dword ptr [esp+18], ecx <===ECX=7(注册名的长度)
:0042CB51 8D1409                        lea edx, dword ptr [ecx+ecx]
:0042CB54 3BC2                          cmp eax, edx  <===呵呵，注册码的长度就是注册名长度的两倍
:0042CB56 0F852F010000                 jne 0042CC8B  <===如果不相等，就跳向 OVER 了。
:0042CB5C 3BCF                          cmp ecx, edi
:0042CB5E 0F8E01010000                 jle 0042CC65  <===这里一跳，就是正确了。呵呵，但是不会跳的
:0042CB64 8A5C2450                      mov bl, byte ptr [esp+50]
```



```text
:0042CB68 EB04  jmp 0042CB6E  <===程序从这里跳走
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CC5F(C)  <===这里是个外部循环
|
:0042CB6A 8B4C2418  mov ecx, dword ptr [esp+18]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CB68(U)
|
:0042CB6E 8D4701  lea eax, dword ptr [edi+01]
:0042CB71 33D2  xor edx, edx
:0042CB73 3BC1  cmp eax, ecx
:0042CB75 8B442450  mov eax, dword ptr [esp+50]
:0042CB79 0F9CC2  setl dl
:0042CB7C 8A0C07  mov cl, byte ptr [edi+eax]
:0042CB7F 8BF2  mov esi, edx
:0042CB81 884C241C  mov byte ptr [esp+1C], cl
:0042CB85 8D542428  lea edx, dword ptr [esp+28]
:0042CB89 8B44241C  mov eax, dword ptr [esp+1C]
:0042CB8D 6A24  push 00000024 <===这是一个关键数值 36(正好为数字 10 位和英文字母 26 位)
:0042CB8F 25FF000000  and eax, 000000FF
:0042CB94 52  push edx
:0042CB95 50  push eax
:0042CB96 89742430  mov dword ptr [esp+30], esi
:0042CB9A E8D2990000  call 00436571
```

*********************

对注册名的奇数位进行变形处理，生成两个数值 `SS1`、`SS2`，(变形处理：取的 `ASC` 值除以 24，余数为 `SS1`，商再除以 24，余数为 `SS2`)，注：作者的程序设计时，设定如果两次除以 24，商还大于 24 的话，就出来 `SS3`，但作为被除数 `AL`，最大也只是 `FF`，所以不可能出现 `SS3`，更不可以出现 `SS4`

*********************

```text
:0042CB9F 83C40C  add esp, 0000000C
:0042CBA2 85F6  test esi, esi
:0042CBA4 742D  je 0042CBD3  <===这是对注册名最后一位的判断，如果是奇数，就从这里跳走了。
:0042CBA6 8B4C2450  mov ecx, dword ptr [esp+50]
:0042CBAA 8D442434  lea eax, dword ptr [esp+34]
:0042CBAE 6A24  push 00000024
:0042CBB0 50  push eax
:0042CBB1 8A540F01  mov dl, byte ptr [edi+ecx+01]
:0042CBB5 88542428  mov byte ptr [esp+28], dl
:0042CBB9 8B4C2428  mov ecx, dword ptr [esp+28]
:0042CBBD 81E1FF000000  and ecx, 000000FF
:0042CBC3 51  push ecx
:0042CBC4 E8A8990000  call 00436571 <===对注册名的偶数位进行变形处理，生成两个数值 SD1、SD2，(变形处理：取的 ASC 值除以 24，余数为 SD1，商再除以 24，余数为 SD2)
:0042CBC9 83C40C  add esp, 0000000C
:0042CBCC BD04000000  mov ebp, 00000004
:0042CBD1 EB05  jmp 0042CBD8
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CBA4(C)
|
:0042CBD3 BD02000000  mov ebp, 00000002  <===到这里说明最后一位是奇数
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CBD1(U)
|
:0042CBD8 33F6  xor esi, esi
:0042CBDA 85ED  test ebp, ebp
:0042CBDC 7E78  jle 0042CC56
:0042CBDE 8A4C2428  mov cl, byte ptr [esp+28]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CC54(C)  <===这里是个内部循环，共四次，就注册名的两个字符生成的 4 位数值，进行不同变换，以生成真注册码的 4 位值。
|
:0042CBE2 83FE03  cmp esi, 00000003
:0042CBE5 7739  ja 0042CC20
:0042CBE7 FF24B5C4CC4200  jmp dword ptr [4*esi+0042CCC4]
<===这里由于 ESI 值的不同产生不同的四次跳转
:0042CBEE 8A5C2429  mov bl, byte ptr [esp+29] <===第一次跳到这里，处理 SS1
:0042CBF2 EB2C  jmp 0042CC20
:0042CBF4 8B442424  mov eax, dword ptr [esp+24] <===第二次跳到这里，处理 SD1
:0042CBF8 85C0  test eax, eax  <=== [esp+24] 是一个标志位，如果注册名最后一位不是偶数位就跳走了。因为不可能有 SD1 和 SD2 了。
:0042CBFA 7406  je 0042CC02  <===如果 EAX 为 0，则这里跳走。
:0042CBFC 8A5C2434  mov bl, byte ptr [esp+34]
:0042CC00 EB1E  jmp 0042CC20
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CBFA(C)
|
:0042CC02 8AD9  mov bl, cl <===如果最后一位是奇数位，则来到这里，就直接用 SS2(商)了
:0042CC04 EB1A  jmp 0042CC20
:0042CC06 0FBED1  movsx edx, cl  <===第三次跳到这里，处理 SS2
:0042CC09 6A01  push 00000001
:0042CC0B 52  push edx
:0042CC0C E8CFFBFFFF  call 0042C7E0  <===因为有了它，所以在真正注册码的 3 的倍数位都是大写的英文字母
:0042CC11 8A4C2430  mov cl, byte ptr [esp+30]
:0042CC15 83C408  add esp, 00000008
:0042CC18 8AD8  mov bl, al
:0042CC1A EB04  jmp 0042CC20
:0042CC1C 8A5C2435  mov bl, byte ptr [esp+35] <===第四次跳到这里，处理 SD2
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```text
|:0042CBE5(C), :0042CBF2(U), :0042CC00(U), :0042CC04(U), :0042CC1A(U)
|
:0042CC20 84DB  test bl, bl
:0042CC22 7519  jne 0042CC3D  <===如果 BL 不为 0，则跳走
:0042CC24 8A442413  mov al, byte ptr [esp+13] <===BL 为 0，则来到这里
:0042CC28 8AD8  mov bl, al
:0042CC2A F6DB  neg bl
:0042CC2C 1ADB  sbb bl, bl
:0042CC2E 80E3E0  and bl, E0
:0042CC31 80C368  add bl, 68
:0042CC34 84C0  test al, al
:0042CC36 0F94C0  sete al
:0042CC39 88442413  mov byte ptr [esp+13], al
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CC22(C)
|
:0042CC3D 8B542414  mov edx, dword ptr [esp+14]
:0042CC41 8BC2  mov eax, edx
:0042CC43 42  inc edx
:0042CC44 89542414  mov dword ptr [esp+14], edx
:0042CC48 8B542454  mov edx, dword ptr [esp+54]
:0042CC4C 381C10  cmp byte ptr [eax+edx], bl
<=== [eax+edx] 为依次取出的注册码值，BL 为注册名的字符算出的正确注册码(仅一位)
:0042CC4F 753A  jne 0042CC8B  <===这里一跳就 OVER 了。
:0042CC51 46  inc esi
:0042CC52 3BF5  cmp esi, ebp
:0042CC54 7C8C  jl 0042CBE2
<===这里是一个内部循环，取两位注册名的字符生成 4 位注册码
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CBDC(C)
|
:0042CC56 8B442418  mov eax, dword ptr [esp+18]
:0042CC5A 83C702  add edi, 00000002
:0042CC5D 3BF8  cmp edi, eax
:0042CC5F 0F8C05FFFFFF  jl 0042CB6A  <===这里是一个外部循环
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CB5E(C)
|
:0042CC65 8D4C2450  lea ecx, dword ptr [esp+50]
:0042CC69 C644244800  mov [esp+48], 00
:0042CC6E E8DDD10100  call 00449E50
:0042CC73 8D4C2454  lea ecx, dword ptr [esp+54]
:0042CC77 C7442448FFFFFFFF  mov [esp+48], FFFFFFFF
:0042CC7F E8CCD10100  call 00449E50
:0042CC84 B801000000  mov eax, 00000001  <===这里是关键的标志赋值，一定要经过
:0042CC89 EB21  jmp 0042CCAC
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```text
|:0042CB11(C), :0042CB1F(C), :0042CB35(C), :0042CB56(C), :0042CC4F(C)
<===这里 5 个，都是错误的跳转
:0042CC8B 8D4C2450  lea ecx, dword ptr [esp+50]
:0042CC8F C644244800  mov [esp+48], 00
:0042CC94 E8B7D10100  call 00449E50
:0042CC99 8D4C2454  lea ecx, dword ptr [esp+54]
:0042CC9D C7442448FFFFFFFF  mov [esp+48], FFFFFFFF
:0042CCA5 E8A6D10100  call 00449E50
:0042CCAA 33C0  xor eax, eax  <===EAX 被清 0，也就是失败了。
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```text
|:0042CC89(U)
|
:0042CCAC 8B4C2440  mov ecx, dword ptr [esp+40]
:0042CCB0 5F  pop edi
```



```
:0042CCB1 5E
:0042CCB2 5D
:0042CCB3 5B
:0042CCB4 64890D00000000
:0042CCBB 83C43C
:0042CCBE C20800

---------0042CC0C call 0042C7E0--对 SS2 进行处理---------------

:0042C7E0 81EC20010000
:0042C7E6 BA37000000
:0042C7EB 56
:0042C7EC 89542420
:0042C7F0 899424B0000000
:0042C7F7 8B94242C010000
:0042C7FE B938000000
:0042C803 B839000000
:0042C808 BE36000000
:0042C80D 894C2424
:0042C811 89442428
:0042C815 898C24B4000000
:0042C81C 898424B8000000
:0042C823 C744240430000000

.......

此处略去一段产生规则的码表，作者真有心一个一个的做，我这里却省了;-)

.......

:0042CA92 C78424200100005A000000
:0042CA9D 33C0
:0042CA9F 8D0CD2
:0042CAA2 8BB42428010000
:0042CAA9 C1E104
:0042CAAC 8D4C0C04

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042CABB(C)

|

:0042CAB0 3B31
:0042CAB2 7409
:0042CAB4 40
:0042CAB5 83C104
:0042CAB8 83F824
:0042CABB 7CF3

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042CAB2(C)

|

:0042CABD 8D14D2
:0042CAC0 C1E202
:0042CAC3 2BD0
:0042CAC5 5E
:0042CAC6 8B84948C000000
:0042CACD 81C420010000
:0042CAD3 C3
```

---------------------------------------------------------------------

### 4. 算法分析：---类型注册: `f(注册名)=注册码`---

*   **a、** 依次取出注册名的两个字符(一个中文为两个字符)，奇数位除以 24(10 进制 36)，余数为 `SS1`，商为 `SS2`。偶数位也除以 24(10 进制 36)，余数为 `SD1`，商为 `SD2`。
*   **b、** 将 `SS2` 单独取出，对字符串 `"STUVWXYZ"` 取倒数 `SS2+1` 位数值，如果 `SS2` 为 0，那么新的 `SS2` 就是 `Z`；如果 `SS2` 是 3，则新的 `SS2` 就是 `W` 了
*   **c、** 这样就得出真注册码的 4 位，`SS1 SD2 SS2(新) SD1`，  依此类推。如果注册名的最后一位是奇数位的话，那么 `SS2` 就不做任何处理(b 步骤)，直接用了。

### 5. -------`VB6.0` 制作的注册机原码(只能用英文注册)-----------

```vb
Private Sub Command1_Click()
strtmp = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
strtmp1 = "0123456789abcdefghijklmnopqrstuvwxyz"
startin = Text1.Text
nlen = Len(startin)

For i = 1 To nlen
 z1 = Asc(Mid(startin, i, 1)) '奇数位字符处理
 ss1 = Mid(strtmp1, (z1 Mod 36) + 1, 1)
 ss2x = Int(z1 / 36)
 ss2 = Mid(strtmp, 36 - ss2x, 1)
 i = i + 1
 If i <= nlen Then
  z2 = Asc(Mid(startin, i, 1)) '偶数位字符处理
  sd1 = Mid(strtmp1, (z2 Mod 36) + 1, 1)
  sd2 = CStr(Int(z2 / 36))
 Else
  sd1 = ""
  sd2 = ""
  ss2=cstr(ss2x)
 End If
laststr = laststr + ss1 + sd2 + ss2 + sd1
Next i

Text2.Text = laststr
End Sub
```

### 6. 注册信息保存在文件 `SysSet.cfg`：

(略)

```ini
[Reg]
RegName=newlaos
RegCode=XXXXXXXXXXXXXX  (此处即为真注册码，为维护作者利益=>隐)
```

`4dd466d0e1e0cc123afbe504da2462ca`
