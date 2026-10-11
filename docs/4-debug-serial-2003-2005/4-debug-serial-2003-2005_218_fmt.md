
# pcmedik V5.4.8.2003

> 原文：[`www.pediy.com/kssd/pediy06/pediy6021.htm`](https://www.pediy.com/kssd/pediy06/pediy6021.htm)

pcmedik V5.4.8.2003 破解手记--算法分析(我和浮点运算的第一次)

作者：newlaos\[CCG\]\[DFCG\]

软件名称：pcmedik V5.4.8.2003(系统工具)

最新版本：V5.4.8.2003

文件大小：854KB

软件授权：共享软件

使用平台：Win9x/Me/NT/2000/XP

下载网址：华军网--->搜索一下

软件简介：中文译名为个人电脑医生，一个傻瓜式的系统优化软件，功能不错，界面也作得漂亮。

加密方式：注册码

功能限制：功能限制

PJ 工具：TRW20001.23 注册版，W32Dasm8.93 黄金版，FI2.5，HMILY\[CCG\]\[BCG\]的“多功能运算器 2.25”，unpecompact1.32

PJ 日期：2003-05-08

作者 newlaos 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 1. 脱壳与分析

1. 先用 `FI2.5` 看一下主文件“`PCMedik.exe`”，加了 `pecompact1.68-84` 的壳。用专用脱壳工具 `unpecompact1.32` 脱壳搞定。
2. 用 `W32Dasm8.93` 黄金版对 `PCMedik.exe` 进行静态反汇编，再用串式数据参考，但找不到任何有用的字符，用 `eXeScope 6.30` 却可以看见，不知道其它高手对于这种情况有没有更好的解决方案。
3. 再用 `TRW20001.23` 注册版进行动态跟踪，下万能断点 `BPX hmemcpy`，

先输入姓名：`newlaos`

假码: `78787878`

断下，你很快就能来到这里

## 2. 动态跟踪分析

```
.......
.......
:0047A7DC E813D1FDFF               call 004578F4
:0047A7E1 837DFC00                 cmp dword ptr [ebp-04], 00000000
:0047A7E5 0F843C010000             je 0047A927
:0047A7EB 8D55F8                   lea edx, dword ptr [ebp-08]
:0047A7EE 8B8324030000             mov eax, dword ptr [ebx+00000324]
:0047A7F4 E8FBD0FDFF               call 004578F4    <===查注册码的长度
:0047A7F9 837DF800                 cmp dword ptr [ebp-08], 00000000  <===如果长度为 0，也就是没有输入注册码
:0047A7FD 0F8424010000             je 0047A927    <===没输入就跳向 OVER 了
:0047A803 8D55F0                   lea edx, dword ptr [ebp-10]
:0047A806 8B8324030000             mov eax, dword ptr [ebx+00000324]
:0047A80C E8E3D0FDFF               call 004578F4
:0047A811 8B45F0                   mov eax, dword ptr [ebp-10] <===EAX=78787878
:0047A814 8D55F4                   lea edx, dword ptr [ebp-0C]
:0047A817 E8ECD8F8FF               call 00408108
:0047A81C 8B45F4                   mov eax, dword ptr [ebp-0C]
:0047A81F 50                       push eax
:0047A820 8D55E8                   lea edx, dword ptr [ebp-18]
:0047A823 8B832C030000             mov eax, dword ptr [ebx+0000032C]
:0047A829 E8C6D0FDFF               call 004578F4
:0047A82E 8B45E8                   mov eax, dword ptr [ebp-18] <===EAX=newlaos
:0047A831 8D55EC                   lea edx, dword ptr [ebp-14]
:0047A834 E8CFD8F8FF               call 00408108
:0047A839 8B45EC                   mov eax, dword ptr [ebp-14] <===EAX=newlaos
:0047A83C 5A                       pop edx                       <===EDX=78787878
:0047A83D E8B63D0000               call 0047E5F8 <===这里就是关键的 CALL 了，F8 跟进
:0047A842 3C01                     cmp al, 01      <===要想正确注册，则出来进 AL 必须为 1
:0047A844 0F85A7000000             jne 0047A8F1   <===这里不能跳，一跳就 OVER 了
:0047A84A A1E04C4800               mov eax, dword ptr [00484CE0]
:0047A84F 8B00                     mov eax, dword ptr [eax]
:0047A851 8B805C030000             mov eax, dword ptr [eax+0000035C]
:0047A857 33D2                     xor edx, edx
.......
.......
```

-----------:0047A83D call 0047E5F8 关键的 CALL 了，F8 跟进-----------

```
:0047E5F8 55                       push ebp
:0047E5F9 8BEC                     mov ebp, esp
:0047E5FB B90C000000               mov ecx, 0000000C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047E605(C)
|

:0047E600 6A00                     push 00000000
:0047E602 6A00                     push 00000000
:0047E604 49                       dec ecx
:0047E605 75F9                     jne 0047E600
:0047E607 51                       push ecx
:0047E608 53                       push ebx
:0047E609 56                       push esi
:0047E60A 57                       push edi
:0047E60B 8955F8                   mov dword ptr [ebp-08], edx
:0047E60E 8945FC                   mov dword ptr [ebp-04], eax
:0047E611 8B45FC                   mov eax, dword ptr [ebp-04]
:0047E614 E8775FF8FF               call 00404590
:0047E619 8B45F8                   mov eax, dword ptr [ebp-08]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047E5A3(C)
|

:0047E61C E86F5FF8FF               call 00404590
:0047E621 33C0                     xor eax, eax
:0047E623 55                       push ebp
:0047E624 68A5E94700               push 0047E9A5
:0047E629 64FF30                   push dword ptr fs:[eax]
:0047E62C 648920                   mov dword ptr fs:[eax], esp
:0047E62F 33C0                     xor eax, eax
:0047E631 55                       push ebp
:0047E632 6851E94700               push 0047E951
:0047E637 64FF30                   push dword ptr fs:[eax]
:0047E63A 648920                   mov dword ptr fs:[eax], esp
:0047E63D B201                     mov dl, 01
:0047E63F A1101A4200               mov eax, dword ptr [00421A10]
:0047E644 E8C734FAFF               call 00421B10
:0047E649 8945EC                   mov dword ptr [ebp-14], eax
:0047E64C BA02000080               mov edx, 80000002
:0047E651 8B45EC                   mov eax, dword ptr [ebp-14]
:0047E654 E85735FAFF               call 00421BB0
:0047E659 B101                     mov cl, 01
:0047E65B BAC0E94700               mov edx, 0047E9C0           ;  ASCII "\Software\PGWARE\PCMedik"
:0047E660 8B45EC                   mov eax, dword ptr [ebp-14]

    <===注册信息保存的位置，程序并不先验证，而是先保存注册信息
:0047E663 E8AC35FAFF               call 00421C14
:0047E668 837DFC00                 cmp dword ptr [ebp-04], 00000000  <===[ebp-04]=newlaos
:0047E66C 7428                     je 0047E696   <===这两个都没有跳转
:0047E66E 837DF800                 cmp dword ptr [ebp-08], 00000000  <===[ebp-08]=78787878
:0047E672 7422                     je 0047E696   <===这两个都没有跳转
:0047E674 8B4DFC                   mov ecx, dword ptr [ebp-04]
:0047E677 BAE4E94700               mov edx, 0047E9E4           ;  ASCII "Name"
:0047E67C 8B45EC                   mov eax, dword ptr [ebp-14]
:0047E67F E82C37FAFF               call 00421DB0
:0047E684 8B4DF8                   mov ecx, dword ptr [ebp-08]
:0047E687 BAF4E94700               mov edx, 0047E9F4           ;  ASCII "Serial"
:0047E68C 8B45EC                   mov eax, dword ptr [ebp-14]
:0047E68F E81C37FAFF               call 00421DB0
:0047E694 EB20                     jmp 0047E6B6   <===我跳

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0047E66C(C), :0047E672(C)
|

:0047E696 8D4DFC                   lea ecx, dword ptr [ebp-04]
:0047E699 BAE4E94700               mov edx, 0047E9E4
:0047E69E 8B45EC                   mov eax, dword ptr [ebp-14]
:0047E6A1 E83637FAFF               call 00421DDC
:0047E6A6 8D4DF8                   lea ecx, dword ptr [ebp-08]
:0047E6A9 BAF4E94700               mov edx, 0047E9F4
:0047E6AE 8B45EC                   mov eax, dword ptr [ebp-14]
:0047E6B1 E82637FAFF               call 00421DDC

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047E694(U)
|

:0047E6B6 33C0                     xor eax, eax
:0047E6B8 55                       push ebp
:0047E6B9 6829E94700               push 0047E929
:0047E6BE 64FF30                   push dword ptr fs:[eax]
:0047E6C1 648920                   mov dword ptr fs:[eax], esp
:0047E6C4 837DFC00                 cmp dword ptr [ebp-04], 00000000
:0047E6C8 0F8451020000             je 0047E91F   <===还是和上面一样不会跳的
:0047E6CE 837DF800                 cmp dword ptr [ebp-08], 00000000
:0047E6D2 0F8447020000             je 0047E91F   <===还是和上面一样不会跳的
:0047E6D8 B85C5F4800               mov eax, 00485F5C
:0047E6DD 8B55FC                   mov edx, dword ptr [ebp-04]
:0047E6E0 E84F5AF8FF               call 00404134
:0047E6E5 68605F4800               push 00485F60
:0047E6EA 8D45E4                   lea eax, dword ptr [ebp-1C]
:0047E6ED 50                       push eax
```



:0047E6EE `8B55F8` mov `edx`, `dword ptr [ebp-08]`

:0047E6F1 `B804EA4700` mov `eax`, `0047EA04`

:0047E6F6 `E8E95FF8FF` `call 004046E4`

:0047E6FB `40` `inc eax`

:0047E6FC `50` `push eax`

:0047E6FD `8B45F8` mov `eax`, `dword ptr [ebp-08]` <===`EAX=78787878`

:0047E700 `E89B5CF8FF` `call 004043A0` <===算法注册码的长度

:0047E705 `8BC8` mov `ecx`, `eax` <===`EAX=ECX=8`

:0047E707 `8B45F8` mov `eax`, `dword ptr [ebp-08]`

:0047E70A `5A` `pop edx`

:0047E70B `E8F05EF8FF` `call 00404600`

:0047E710 `8B55E4` mov `edx`, `dword ptr [ebp-1C]`

:0047E713 `B804EA4700` mov `eax`, `0047EA04`

:0047E718 `E8C75FF8FF` `call 004046E4`

:0047E71D `48` `dec eax`

:0047E71E `50` `push eax`

:0047E71F `8D45E0` `lea eax, dword ptr [ebp-20]`

:0047E722 `50` `push eax`

:0047E723 `8B55F8` mov `edx`, `dword ptr [ebp-08]`

:0047E726 `B804EA4700` mov `eax`, `0047EA04`

:0047E72B `E8B45FF8FF` `call 004046E4`

:0047E730 `40` `inc eax`

:0047E731 `50` `push eax`

:0047E732 `8B45F8` mov `eax`, `dword ptr [ebp-08]`

:0047E735 `E8665CF8FF` `call 004043A0`

:0047E73A `8BC8` mov `ecx`, `eax`

:0047E73C `8B45F8` mov `eax`, `dword ptr [ebp-08]`

:0047E73F `5A` `pop edx`

:0047E740 `E8BB5EF8FF` `call 00404600`

:0047E745 `8B45E0` mov `eax`, `dword ptr [ebp-20]`

:0047E748 `BA01000000` mov `edx`, `00000001`

:0047E74D `59` `pop ecx`

:0047E74E `E8AD5EF8FF` `call 00404600`

:0047E753 `BB01000000` mov `ebx`, `00000001` <===`EBX=1`

* Referenced by a `(U)nconditional or (C)onditional Jump at Address:`

| :0047E7F3(C)

| `**********************循环开始**********************`

:0047E758 `8D45D8` `lea eax, dword ptr [ebp-28]`

:0047E75B `50` `push eax`

:0047E75C `B901000000` mov `ecx`, `00000001` <===这里为取值长度为 1

:0047E761 `8BD3` mov `edx`, `ebx` <===取值开始的位置，依次是 1,2,3,4,5,6,7

:0047E763 `8B45FC` mov `eax`, `dword ptr [ebp-04]` <===`EAX="newlaos`

:0047E766 `E8955EF8FF` `call 00404600` <===将上面给出的参数取值

:0047E76B `8B45D8` mov `eax`, `dword ptr [ebp-28]` <===`EAX` 为依次取出的字符，`n,e,w,l,a,o,s`

:0047E76E `0FB600` `movzx eax, byte ptr [eax]` <===取出字符的 `ASC` 值，`6E,65,77,6C,61,6F,73`

:0047E771 `F7EB` `imul ebx` <===这里的作用是将 `EAX` 的值乘以 `EBX`(依次增加)，结果返回 `EAX`，`EAX` 的值依次为 `6E,CA,165,1B0,1E5,29A,325`

:0047E773 `8945D4` mov `dword ptr [ebp-2C]`, `eax`

:0047E776 `DB45D4` `fild dword ptr [ebp-2C]`

:0047E779 `E85A43F8FF` `call 00402AD8`

:0047E77E `8945CC` mov `dword ptr [ebp-34]`, `eax`

:0047E781 `8955D0` mov `dword ptr [ebp-30]`, `edx`

:0047E784 `DF6DCC` `fild qword ptr [ebp-34]`

:0047E787 `83C4F4` `add esp, FFFFFFF4`

:0047E78A `DB3C24` `fstp tbyte ptr [esp]`

:0047E78D `9B` `wait`

:0047E78E `8D55DC` `lea edx, dword ptr [ebp-24]`

:0047E791 `B810EA4700` mov `eax`, `0047EA10`

:0047E796 `E895B5F8FF` `call 00409D30` <===将前面得出的值转为 10 进制的表示

```
--------------------
   6E    ----> 110
   CA    ----> 202
   165   ----> 357
   1B0   ----> 432
   1E5   ----> 485
   29A   ----> 666
   325   ----> 805
--------------------
```

:0047E79B `FF75DC` `push [ebp-24]`

:0047E79E `8D55C8` `lea edx, dword ptr [ebp-38]`

:0047E7A1 `8BC3` mov `eax`, `ebx`

:0047E7A3 `E8C89BF8FF` `call 00408370`

:0047E7A8 `FF75C8` `push [ebp-38]`

:0047E7AB `FF35605F4800` `push dword ptr [00485F60]`

:0047E7B1 `8D45E8` `lea eax, dword ptr [ebp-18]`

:0047E7B4 `BA03000000` mov `edx`, `00000003`

:0047E7B9 `E8A25CF8FF` `call 00404460` <===将上面的值与 `EBX` 的值以字符串的形式连起来

```
--------------------
   "110" + "1" = "1101"
   "202" + "2" = "2022"
   "357" + "3" = "3573"
   "432" + "4" = "4324"
   "485" + "5" = "4855"
   "666" + "6" = "6666"
   "805" + "7" = "8057"
--------------------
```

:0047E7BE `8B45E8` mov `eax`, `dword ptr [ebp-18]` <===`eax` 依次为上面的字符串

:0047E7C1 `E8E69CF8FF` `call 004084AC` <===将字符串转为数值，`EAX` 依次为 `44D,7E6,DF5,10E4,12F7,1A0A,1F79`

:0047E7C6 `8BF0` mov `esi`, `eax`

:0047E7C8 `8B45E8` mov `eax`, `dword ptr [ebp-18]` <===`eax` 依次为上面的字符串

:0047E7CB `E8DC9CF8FF` `call 004084AC` <===将字符串转为数值，`EAX` 依次为 `44D,7E6,DF5,10E4,12F7,1A0A,1F79`

:0047E7D0 `03F0` `add esi, eax` <===晕，这就等效于每个数值乘以 2

:0047E7D2 `8BC6` mov `eax`, `esi` <===`EAX` 依次为 `89A，FCC，1DEA，21C8,25EE，3414，3EF2`

:0047E7D4 `8D55C4` `lea edx, dword ptr [ebp-3C]`

:0047E7D7 `E8949BF8FF` `call 00408370` <===这个 `CALL` 又将数值转为了字符

:0047E7DC `8B55C4` mov `edx`, `dword ptr [ebp-3C]`

:0047E7DF `8D45E8` `lea eax, dword ptr [ebp-18]`

:0047E7E2 `E89159F8FF` `call 00404178`

:0047E7E7 `43` `inc ebx`

:0047E7E8 `8B45FC` mov `eax`, `dword ptr [ebp-04]` <===`eax=newlaos`

:0047E7EB `E8B05BF8FF` `call 004043A0` <===求注册名的长度

:0047E7F0 `40` `inc eax` <===`EAX=7+1=8`

:0047E7F1 `3BD8` `cmp ebx, eax` <===所以这里就保证了，循环 7 次

:0047E7F3 `0F855FFFFFFF` `jne 0047E758` <===从这里向上跳是一个循环结构

`**********************循环结束**********************`

:0047E7F9 `6A17` `push 00000017`

:0047E7FB `683652C70D` `push 0DC75236`

:0047E800 `8B45E8` mov `eax`, `dword ptr [ebp-18]` <===`EAX="16114"`也就是最后一位的计算结果

:0047E803 `E8A49CF8FF` `call 004084AC` <===将字符转为数值，`EAX=3EF2`

:0047E808 `99` `cdq`

:0047E809 `E8CE67F8FF` `call 00404FDC` <===这里是一小段代码，`F8` 跟进，出来后 `EAX=4C54CB0C`，`EDX=5AB21`

:0047E80E `8945CC` mov `dword ptr [ebp-34]`, `eax`

:0047E811 `8955D0` mov `dword ptr [ebp-30]`, `edx`

:0047E814 `DF6DCC` `fild qword ptr [ebp-34]` <===呵呵，`[ebp-34]`的值为 `5AB214C54CB0C`

:0047E817 `83C4F4` `add esp, FFFFFFF4`

:0047E81A `DB3C24` `fstp tbyte ptr [esp]`

:0047E81D `9B` `wait`

:0047E81E `8D55C0` `lea edx, dword ptr [ebp-40]`

:0047E821 `B810EA4700` mov `eax`, `0047EA10`

:0047E826 `E805B5F8FF` `call 00409D30` <===这个 `CALL` 计算出一串数字 `1595534386449164`(好一个大数运算库，我用了 `HMILY[CCG][BCG]`的“多功能运算器 2.25”才知道的这个结果，这真是个好软件!)

:0047E82B `8B55C0` mov `edx`, `dword ptr [ebp-40]` <===`EDX="1595534386449164"`

:0047E82E `8D45E8` `lea eax, dword ptr [ebp-18]` <===`EAX` 的地址指针指向`"16114"`

:0047E831 `E84259F8FF` `call 00404178`

:0047E836 `8D45F0` `lea eax, dword ptr [ebp-10]`

:0047E839 `50` `push eax`

:0047E83A `8B55F8` mov `edx`, `dword ptr [ebp-08]` <===`EDX="78787878"`

:0047E83D `B804EA4700` mov `eax`, `0047EA04` <===这里是“`+`”

:0047E842 `E89D5EF8FF` `call 004046E4` <===定位在输入的注册码中“`+`”的位置，我们将注册码改为 `78787878+12345678`，重新来

:0047E847 `8BC8` mov `ecx`, `eax` <===`EAX=9`(说时“`+`”在输入注册码的第 9 位)

:0047E849 `49` `dec ecx` <===`ECX=9-1=8` 取字符串的长度



```assembly
:0047E84A BA01000000  mov edx, 00000001  <===从第一个的开始取  
:0047E84F 8B45F8      mov eax, dword ptr [ebp-08] <===EAX="78787878+12345678"  
:0047E852 E8A95DF8FF  call 00404600 <===进行取字符串的动作  
:0047E857 8B45F0      mov eax, dword ptr [ebp-10] <===EAX="78787878"  
:0047E85A E829B5F8FF  call 00409D88   
:0047E85F DB7DB4      fstp tbyte ptr [ebp-4C]  
:0047E862 9B          wait  
:0047E863 8B45E8      mov eax, dword ptr [ebp-18] <===EAX="1595534386449164"  
:0047E866 E81DB5F8FF  call 00409D88  
:0047E86B DB6DB4      fld tbyte ptr [ebp-4C]  
:0047E86E DEE1        fsubrp st(1), st(0)  
:0047E870 D81D14EA4700 fcomp dword ptr [0047EA14]  
:0047E876 DFE0        fstsw ax  
:0047E878 9E          sahf  
:0047E879 0F879C000000 ja 0047E91B <===从这里跳走就 OVER 了，当这里改为 78787878+12345678 后这里不跳走  
:0047E87F 8B45E8      mov eax, dword ptr [ebp-18] <===EAX="1595534386449164"  
:0047E882 E801B5F8FF  call 00409D88  
:0047E887 DB7DA8      fstp tbyte ptr [ebp-58]  
:0047E88A 9B          wait  
:0047E88B 8B45F0      mov eax, dword ptr [ebp-10] <===EAX="78787878"  
:0047E88E E8F5B4F8FF  call 00409D88  
:0047E893 DB6DA8      fld tbyte ptr [ebp-58]  
:0047E896 DEE1        fsubrp st(1), st(0)  
:0047E898 D81D14EA4700 fcomp dword ptr [0047EA14] <===明白了，这里真正的注册码在"+"号前面必须就是"1595534386449164",哈哈，注册成功了!  
:0047E89E DFE0        fstsw ax  
:0047E8A0 9E          sahf  
:0047E8A1 7778        ja 0047E91B <===从这里跳走就 OVER 了  
:0047E8A3 C605585F480001 mov byte ptr [00485F58], 01  
:0047E8AA C645F701    mov [ebp-09], 01 <===哈哈，胜利的希望在这里  
:0047E8AE B201        mov dl, 01  
:0047E8B0 A1682B4100  mov eax, dword ptr [00412B68]  
:0047E8B5 E8024AF8FF  call 004032BC  
:0047E8BA 8BF0        mov esi, eax  
:0047E8BC BA20EA4700  mov edx, 0047EA20  
:0047E8C1 8BC6        mov eax, esi  
:0047E8C3 8B08        mov ecx, dword ptr [eax]  
:0047E8C5 FF5138      call [ecx+38]  
:0047E8C8 33DB        xor ebx, ebx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  | `:0047E910(C)`
  |

`****循环开始****`

```assembly
:0047E8CA 8D55A4      lea edx, dword ptr [ebp-5C]  
:0047E8CD 8B45FC      mov eax, dword ptr [ebp-04]  
:0047E8D0 E8E395F8FF  call 00407EB8  
:0047E8D5 8B45A4      mov eax, dword ptr [ebp-5C]  
:0047E8D8 50          push eax  
:0047E8D9 8D4D9C      lea ecx, dword ptr [ebp-64]  
:0047E8DC 8BD3        mov edx, ebx  
:0047E8DE 8BC6        mov eax, esi  
:0047E8E0 8B38        mov edi, dword ptr [eax]  
:0047E8E2 FF570C      call [edi+0C]  
:0047E8E5 8B459C      mov eax, dword ptr [ebp-64]  
:0047E8E8 8D55A0      lea edx, dword ptr [ebp-60]  
:0047E8EB E8C895F8FF  call 00407EB8  
:0047E8F0 8B55A0      mov edx, dword ptr [ebp-60]  
:0047E8F3 58          pop eax  
:0047E8F4 E8F35BF8FF  call 004044EC  
:0047E8F9 750B        jne 0047E906 <===在这里就必须跳走了  
:0047E8FB C605585F480000 mov byte ptr [00485F58], 00  
:0047E902 C645F700    mov [ebp-09], 00 <===到这里也是错了
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  | `:0047E8F9(C)`
  |

```assembly
:0047E906 43          inc ebx  
:0047E907 8BC6        mov eax, esi  
:0047E909 8B10        mov edx, dword ptr [eax] <===  
:0047E90B FF5214      call [edx+14]  
:0047E90E 3BD8        cmp ebx, eax  
:0047E910 75B8        jne 0047E8CA <===从这里向上跳是一个循环结构
```

`****循环结束****`

```assembly
:0047E912 8BC6        mov eax, esi  
:0047E914 E8D349F8FF  call 004032EC  
:0047E919 EB04        jmp 0047E91F
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  | `:0047E879(C)`, `:0047E8A1(C)` <===向上看两个错误跳转
  |

```assembly
:0047E91B C645F700    mov [ebp-09], 00 <===这里不能经过
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
  | `:0047E6C8(C)`, `:0047E6D2(C)`, `:0047E919(U)`
  |

```assembly
:0047E91F 33C0        xor eax, eax  
:0047E921 5A          pop edx  
:0047E922 59          pop ecx  
:0047E923 59          pop ecx  
:0047E924 648910      mov dword ptr fs:[eax], edx  
:0047E927 EB0E        jmp 0047E937  
:0047E929 E95E4EF8FF  jmp 0040378C  
:0047E92E C645F700    mov [ebp-09], 00 <===这里不能经过，当然也不会经过  
:0047E932 E8BD51F8FF  call 00403AF4
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  | `:0047E927(U)`
  |

```assembly
:0047E937 8B45EC      mov eax, dword ptr [ebp-14]  
:0047E93A E84132FAFF  call 00421B80  
:0047E93F 8B45EC      mov eax, dword ptr [ebp-14]  
:0047E942 E8A549F8FF  call 004032EC  
:0047E947 33C0        xor eax, eax  
:0047E949 5A          pop edx  
:0047E94A 59          pop ecx  
:0047E94B 59          pop ecx  
:0047E94C 648910      mov dword ptr fs:[eax], edx  
:0047E94F EB0A        jmp 0047E95B  
:0047E951 E9364EF8FF  jmp 0040378C  
:0047E956 E89951F8FF  call 00403AF4
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  | `:0047E94F(U)`
  |

```assembly
:0047E95B 33C0        xor eax, eax  
:0047E95D 5A          pop edx  
:0047E95E 59          pop ecx  
:0047E95F 59          pop ecx  
:0047E960 648910      mov dword ptr fs:[eax], edx  
:0047E963 68ACE94700  push 0047E9AC
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  | `:0047E9AA(U)`
  |

```assembly
:0047E968 8D459C      lea eax, dword ptr [ebp-64]  
:0047E96B BA03000000  mov edx, 00000003  
:0047E970 E88F57F8FF  call 00404104  
:0047E975 8D45C0      lea eax, dword ptr [ebp-40]  
:0047E978 BA03000000  mov edx, 00000003  
:0047E97D E88257F8FF  call 00404104  
:0047E982 8D45D8      lea eax, dword ptr [ebp-28]  
:0047E985 BA05000000  mov edx, 00000005  
:0047E98A E87557F8FF  call 00404104  
:0047E98F 8D45F0      lea eax, dword ptr [ebp-10]  
:0047E992 E84957F8FF  call 004040E0  
:0047E997 8D45F8      lea eax, dword ptr [ebp-08]  
:0047E99A BA02000000  mov edx, 00000002  
:0047E99F E86057F8FF  call 00404104  
:0047E9A4 C3          ret
```

```assembly
:0047E9A5 E99650F8FF  jmp 00403A40  
:0047E9AA EBBC        jmp 0047E968  
:0047E9AC 8A45F7      mov al, byte ptr [ebp-09] <===[ebp-09]为至关重要的数值，向上看  
:0047E9AF 5F          pop edi  
:0047E9B0 5E          pop esi  
:0047E9B1 5B          pop ebx  
:0047E9B2 8BE5        mov esp, ebp  
:0047E9B4 5D          pop ebp  
:0047E9B5 C3          ret
```

--- F8 跟进来到这里 ---

```assembly
:00404FDC 52          push edx  
:00404FDD 50          push eax
```



### 汇编代码分析

```asm
:00404FDE 8B442410                mov eax, dword ptr [esp+10] <===EAX=17 为作者内定值
:00404FE2 F72424                  mul dword ptr [esp] <===[esp]为 3EF2，EAX=17 * 3EF2=5A7BE
:00404FE5 89C1                    mov ecx, eax <===ECX=5A7BE
:00404FE7 8B442404                mov eax, dword ptr [esp+04] <===EAX=0
:00404FEB F764240C                mul [esp+0C] <===[esp+0C]=0DC75236 为作者内定值，EAX=0 * DC75236=0
:00404FEF 01C1                    add ecx, eax <===ECX=5A7BE+0=5A7BE
:00404FF1 8B0424                  mov eax, dword ptr [esp] <===[esp]为 3EF2，EAX=3EF2
:00404FF4 F764240C                mul [esp+0C] <===EAX=3EF2 * 0DC75236 =4C54CB0C EDX=363(相乘大于 FFFFFFFF，溢出的高位)
:00404FF8 01CA                    add edx, ecx <===EDX=5A7BE+363=5AB21
:00404FFA 59                      pop ecx
:00404FFB 59                      pop ecx
:00404FFC C20800                  ret 0008
```

### 算法分析

类型：`f(注册名)=注册码`

1. 取出注册名的最后一个字符的 `ASC` 值，乘以它在注册名的位置，结果转为 10 进制的字符表示，后面再接上它在注册名的位置的字符形式。
2. 将这个字符串，转回为数值形式，乘以 `2`，得出新的结果 `firstsum`。
3. 将 `firstsum` 乘以 `17`(10 进制是 `25`)，得到 `highsum`，将 `firstsum` 乘以 `DC75236`，结果高位进入 `EDX`，低位进入 `EAX`，将 `EDX` 的值加上 `highsum`，再以字符串的形式与低位 `EAX` 连成一个大于 8 位的 16 进制形式的字符串，将这个字符串转为 10 进制，就是我们要的注册码前面部分了。
4. 输入的注册码中必须用一个 `+`，在这之前才作为实际校验，之后的没有用。

### VB6.0 编译的注册机源码

```vb
Private Sub Command1_Click()
'每一个 0 代表 4 位数，第一个 0 代表 1-4 位，第 2 个 0 代表 5-8 位，第 3 个 0 代表 9-12 们，类推，第 7 个 0 代表 25-28 位(这是从最低位开始数哟)
namestr = Text1.Text    '取注册名
nlen = Len(namestr) '得出长度
e = 1    '输入正确标志
For i = 1 To nlen  '检查注册名的合法性，不支持中文注册名
    sumsum = Asc(Mid(namestr, i, 1))
    If sumsum <> Abs(sumsum) Then
        e = 2
    End If
Next i

If e = 1 And nlen <> 0 Then
    firstsum = CLng((CStr(sumsum * nlen) + CStr(nlen))) * 2 '因为已经是最后一位，所以直接拿来用,firstsum 为由注册码产生的值

'--------------------开始 firstsum 与 1700000000(16 进制)的相乘------------------------
'这里等效于 firstsum*&H17*4294967296
astr1 = CStr(firstsum * &H17)       '相乘数 1
astr2 = CStr(4294967296#)       '相乘数 2(16 进制的 100000000 等于 10 进制的 4294967296)
a = Array(0, 0, 0, 0, 0, 0, 0)
i = 0
k = 1
Do
    alen = Len(astr1)
    If alen <= 4 Then
        k = 2
        a(i) = CLng(astr1)
    Else
        a(i) = CLng(Right(astr1, 4))                  '每次从低位取 4 位
        astr1 = Mid(astr1, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 a

b = Array(0, 0, 0, 0, 0, 0, 0)
i = 0
k = 1
Do
    alen = Len(astr2)
    If alen <= 4 Then
        k = 2
        b(i) = CLng(astr2)
    Else
        b(i) = CLng(Right(astr2, 4))                  '每次取 4 位
        astr2 = Mid(astr2, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 b

c = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)  '定义最终结果数组，每一个 0 代表 8 位数

For i = 0 To 6
    tmp2 = 0
    For j = 0 To 6
        sumtmp = a(j) * b(i) + tmp2 + c(i + j)
        c(i + j) = (sumtmp Mod 10000)
        tmp2 = Int(sumtmp / 10000)
    Next j
    c(i + j) = tmp2
Next i

k = 1
For i = 13 To 0 Step -1
    If c(i) <> 0 Or k <> 1 Then
        ttmp = CStr(c(i))
        If Len(ttmp) < 4 And k <> 1 Then  '当出现某个数组中的数为 0890 的情况，如果不补 0，就会出错了。当然也要考第一个数防止出现数字最前面出现多余的 0 的情况
            For j = 1 To 4 - Len(ttmp)
                ttmp = "0" + ttmp
            Next
        End If
        laststr1 = laststr1 + ttmp
        k = 2
    End If
Next i
sumstr1 = laststr1   '用于下次相加

'--------------------firstsum 与 1700000000(16 进制)的相乘结束------------------------

'--------------------开始 firstsum 与&HDC75236 的相乘------------------------
astr1 = CStr(firstsum)         '相乘数 1
astr2 = CStr(&HDC75236)        '相乘数 2

For i = 0 To 6          '数组初始化
    a(i) = 0
Next i

i = 0
k = 1
Do
    alen = Len(astr1)
    If alen <= 4 Then
        k = 2
        a(i) = CLng(astr1)
    Else
        a(i) = CLng(Right(astr1, 4))                  '每次从低位取 4 位
        astr1 = Mid(astr1, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 a

For i = 0 To 6          '数组初始化
    b(i) = 0
Next i

i = 0
k = 1
Do
    alen = Len(astr2)
    If alen <= 4 Then
        k = 2
        b(i) = CLng(astr2)
    Else
        b(i) = CLng(Right(astr2, 4))                  '每次取 4 位
        astr2 = Mid(astr2, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 b

For i = 0 To 13          '数组初始化
    c(i) = 0
Next i

For i = 0 To 6
    tmp2 = 0
    For j = 0 To 6
        sumtmp = a(j) * b(i) + tmp2 + c(i + j)
        c(i + j) = (sumtmp Mod 10000)
        tmp2 = Int(sumtmp / 10000)
    Next j
    c(i + j) = tmp2
Next i

k = 1
For i = 13 To 0 Step -1
    If c(i) <> 0 Or k <> 1 Then
        ttmp = CStr(c(i))
        If Len(ttmp) < 4 And k <> 1 Then  '当出现某个数组中的数为 0890 的情况，如果不补 0，就会出错了。当然也要考第一个数防止出现数字最前面出现多余的 0 的情况
            For j = 1 To 4 - Len(ttmp)
                ttmp = "0" + ttmp
            Next
        End If
        laststr2 = laststr2 + ttmp
        k = 2
    End If
Next i
sumstr2 = laststr2   '用于下次相加

'--------------------firstsum 与 DC75236 的相乘结束------------------------

'--------------------开始 sumstr1 与 sumstr2 的相加------------------------
astr1 = sumstr1         '相加数 1
astr2 = sumstr2         '相加数 2

For i = 0 To 6          '数组初始化
    a(i) = 0
Next i

i = 0
k = 1
Do
    alen = Len(astr1)
    If alen <= 4 Then
        k = 2
        a(i) = CLng(astr1)
    Else
        a(i) = CLng(Right(astr1, 4))                  '每次从低位取 4 位
        astr1 = Mid(astr1, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 a

For i = 0 To 6          '数组初始化
    b(i) = 0
Next i

i = 0
k = 1
Do
    alen = Len(astr2)
    If alen <= 4 Then
        k = 2
        b(i) = CLng(astr2)
    Else
        b(i) = CLng(Right(astr2, 4))                  '每次取 4 位
        astr2 = Mid(astr2, 1, alen - 4)
    End If
    i = i + 1
Loop While k = 1
'到这里完成将 10 进制数第 4 位放入数组 b

For i = 0 To 13          '数组初始化
    c(i) = 0
Next i

i = 0
tmp2 = 0

Do While a(i) <> 0 Or b(i) <> 0
    tmp = a(i) + b(i) + tmp2
    c(i) = (tmp Mod 10000)          '只留 4 位
    tmp2 = Int(tmp / 10000)        '商自动进入下一位
    i = i + 1
    If i = 7 Then
        Exit Do
    End If
Loop

c(i) = tmp2         '最后一个进位值

k = 1
For i = 13 To 0 Step -1
    If c(i) <> 0 Or k <> 1 Then
        ttmp = CStr(c(i))
        If Len(ttmp) < 4 And k <> 1 Then  '当出现某个数组中的数为 0890 的情况，如果不补 0，就会出错了。当然也要考第一个数防止出现数字最前面出现多余的 0 的情况
            For j = 1 To 4 - Len(ttmp)
                ttmp = "0" + ttmp
            Next
        End If
        laststr3 = laststr3 + ttmp
        k = 2
    End If
Next i

laststr3=laststr3+"+"

'---------------------sumstr1 与 sumstr2 的相加结束--------------------------------
Text2.Text = laststr3
Else
    h = MsgBox("本注册机暂不支持中文注册名，或是你还没有输入注册名!", 0, "你的输入有误!")
End If
End Sub
```

### 注册信息保存在注册表

```registry
[HKEY_LOCAL_MACHINE\Software\PGWARE\PCMedik]
"Name"="newlaos[CCG][DFCG]"
"Serial"="33153925271260536+"  4dd466d0e1e0cc123afbe504da2462ca
```
