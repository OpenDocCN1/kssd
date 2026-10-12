
# 算法分析第一篇：网页梦工厂 XP1.5 算法发析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-68.htm`](https://www.pediy.com/kssd/pediy07/pediy7-68.htm)

网页梦工厂 XP1.5（来自一不知名光盘，光盘标为 2002 年，想必现在的新版都出来了好几版了。

主程序：

`jsbuilder.exe`

目的：找注册算法，并作出注册算号器

13:05 2005-2-6

`peid` v0.92 2004 版

`UPX` 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo

`OEP`: `004F5ACC`

用 `Peid2004` 脱壳

`RecImport.exe` 修复输入表，得到：

`JSbuilderunpack.exe`

1.35MB

原文件 432KB

未注册版只能用 40 次

软件在本机产生注册申请码（机器码）：`66050761`

填入假认证码：`1234567890`，弹出出错信息：错误，网页特效梦工厂 XP 注册认证失败

启动 `W32Dasm` V8.93 Gold 版

查找字串，发现其中有：

还有：请重新启动本软件，以完成软件的注册，不会有重启效验吧，那刚才怎么不出现这一现象呢？

* 软件试用次数已到，是否马上注册本软件？
* 请输入注册认证码
* 网页特效梦工厂 XP 注册成功，谢谢你对本软件的支持 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;双击这个，会弹出 `Data Reference Not Found`
* 网页特效梦工厂 XP1.5(注册版）

有这么多好的信息呀！！

来：借用 `W32Dasm8.93Gold` 版，进行初步分析，然后用 `Ollydbg1.10` 正式版运态分析

追注册码算法，先看个初步：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004D69A3(C)

|

:004D69E1 8D55F0              lea edx, dword ptr [ebp-10]
:004D69E4 8B861C0A0000        mov eax, dword ptr [esi+00000A1C]
:004D69EA E8F961F6FF          call 0043CBE8              这里获取机器码
:004D69EF 8B45F0              mov eax, dword ptr [ebp-10]
:004D69F2 E8F126F3FF          call 004090E8              关键 Call1
:004D69F7 E88829FFFF          call 004C9384              关键 Call2
:004D69FC 8BD8                mov ebx, eax
:004D69FE 8B45FC              mov eax, dword ptr [ebp-04]
:004D6A01 E8E226F3FF          call 004090E8             关键 Call3
:004D6A06 3BD8                cmp ebx, eax               关键比较
:004D6A08 0F8581000000        jne 004D6A8F               这里跳向注册认证错误
:004D6A0E B201                mov dl, 01
:004D6A10 A140B34600          mov eax, dword ptr [0046B340]
:004D6A15 E8264AF9FF          call 0046B440
:004D6A1A 8BD8                mov ebx, eax
:004D6A1C BA02000080          mov edx, 80000002
:004D6A21 8BC3                mov eax, ebx
:004D6A23 E8B84AF9FF          call 0046B4E0
:004D6A28 33C9                xor ecx, ecx

具体分析：

它们来自这里：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004D68A4(C)

|

:004D690B 00                BYTE 0

省略许多代码

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004D6916(C)

|

:004D698C 8B86200A0000      mov eax, dword ptr [esi+00000A20]      就在这里按 F2 下断吧！！！！！，F9 运行。
:004D6992 E85162F6FF        call 0043CBE8                      得到假注册码的位数
:004D6997 8B45F4            mov eax, dword ptr [ebp-0C]       EAX=A 即获得假认证码的位为为 10 位
:004D699A E88DE3F2FF        call 00404D2C
:004D699F 8BD0              mov edx, eax                 EDX 中得到假注册码的位数 A
:004D69A1 85D2              test edx, edx
:004D69A3 7E3C              jle 004D69E1               因为输入了十位的假码，这里没有跳，下面是 OD 中得到的数据：
:004D69A5 B801000000        mov eax, 00000001               EAX 是计数器，赋值 1
004D69AA      8B4D  FC       mov ecx,dword ptr ss:[ebp-4]
004D69AD      8A4C01 FF      mov cl,byte ptr ds:[ecx+eax-1]  假码逐位与 30，即数字 0 比较，没有跳
004D69B1      80F9  30       cmp cl,30
004D69B4      72  08         jb short JSbuilde.004D69BE
004D69B6      8B5D  FC       mov ebx,dword ptr ss:[ebp-4]
004D69B9      80F9  39       cmp cl,39                      与 9 比较（即应该全为 0-9 之间的数字，否则会弹出“注册码输入有误……”）
004D69BC      76  1F         jbe short JSbuilde.004D69DD      跳了
004D69BE      B8  F06A4D00    mov eax,JSbuilde.004D6AF0      ; ASCII "注册码输入有误,请重新输入!"
004D69C3      E8  DCF4F5FF     call JSbuilde.00435EA4
004D69C8      8B96  200A0000   mov edx,dword ptr ds:[esi+A20]
004D69CE      A1  EC405000     mov eax,dword ptr ds:[5040EC]
004D69D3      E8  C412F8FF     call JSbuilde.00457C9C
004D69D8      E9  BC000000     jmp JSbuilde.004D6A99
004D69DD      40               inc eax               因为 1 少于 9，就到了这里，计数器自加 1
004D69DE      4A               dec edx               EDX 是假注册码的位数 A，EDX 自减 1
004D69DF     ^ 75  C9          jnz short JSbuilde.004D69AA
004D69E1      8D55  F0       lea edx,dword ptr ss:[ebp-10]
004D69E4      8B86  1C0A0000  mov eax,dword ptr ds:[esi+A1C]
004D69EA      E8  F961F6FF     call JSbuilde.0043CBE8      这个 Call 获得机器码
004D69EF      8B45  F0       mov eax,dword ptr ss:[ebp-10]   将机器给 EAX
004D69F2      E8  F126F3FF     call JSbuilde.004090E8  第一个关键 Call  这个 Call 跟进看下面：雄飞（将机器码进行运算）
004D69F7      E8  8829FFFF     call JSbuilde.004C9384  第二个关键 Call  这个还不是比较用的 Call 跟进去，看：爱兰（与 wangshuang,yaoyuan,JSBuilder 运算，得到一结果）
004D69FC      8BD8             mov ebx,eax
004D69FE      8B45  FC       mov eax,dword ptr ss:[ebp-4]
004D6A01      E8  E226F3FF     call JSbuilde.004090E8  第三个关键 Call   这个 Call 就重要了，它会将假注册码运算，产生一个中间数，跟进看：珊册
004D6A06      3BD8             cmp ebx,eax                   这里 EBX=666C1398（机器码经过一系列运算得到的结果） EAX=499602D2 （假注册码经过运算后和结果）
004D6A08      0F85  81000000   jnz JSbuilde.004D6A8F      这里注跳向注册认证错误！！
004D6A0E      B2  01          mov dl,1
004D6A10      A1  40B34600      mov eax,dword ptr ds:[46B340]
004D6A15      E8  264AF9FF     call JSbuilde.0046B440
004D6A1A      8BD8             mov ebx,eax
004D6A1C      BA  02000080     mov edx,80000002
004D6A21      8BC3             mov eax,ebx
004D6A23      E8  B84AF9FF     call JSbuilde.0046B4E0
004D6A28      33C9             xor ecx,ecx
004D6A2A      BA  146B4D00     mov edx,JSbuilde.004D6B14      ; ASCII "System\\CurrentControlSet\\Services\\Class\\knightsoft\\JSBuilder"
004D6A2F      8BC3             mov eax,ebx
004D6A31      E8  EA4BF9FF     call JSbuilde.0046B620
004D6A36      8D55  EC       lea edx,dword ptr ss:[ebp-14]
004D6A39      8B86  1C0A0000  mov eax,dword ptr ds:[esi+A1C]

第一个关键 Call，雄飞

004035D0      53               push ebx
004035D1      56               push esi
004035D2      57               push edi
004035D3      89C6             mov esi,eax
004035D5      50               push eax
004035D6      85C0             test eax,eax
004035D8      74  6C           je short JSbuilde.00403646
004035DA      31C0             xor eax,eax
004035DC      31DB             xor ebx,ebx
004035DE      BF  CCCCCC0C      mov edi,0CCCCCCC
004035E3      8A1E             mov bl,byte ptr ds:[esi]      机器码逐位放入 BL，第一位是 6，即 ASCII 码 36
004035E5      46               inc esi               ESi 中是机器码的地址，自加 1，原为 00F10A4C，自加 1 变为 00F10A4D，即到指向第二位机器码
004035E6      80FB  20         cmp bl,20             与空格相比较
004035E9     ^ 74  F8          je short JSbuilde.004035E3
004035EB      B5  00            mov ch,0             ECX 高位置 0
004035ED      80FB  2D         cmp bl,2D             与短横相比较
004035F0      74  62           je short JSbuilde.00403654
004035F2      80FB  2B         cmp bl,2B               与 +   相比较
004035F5      74  5F           je short JSbuilde.00403656
004035F7      80FB  24         cmp bl,24             不能是 $
004035FA      74  5F           je short JSbuilde.0040365B
004035FC      80FB  78         cmp bl,78                     N
004035FF      74  5A           je short JSbuilde.0040365B
00403601      80FB  58         cmp bl,58                     :
00403604      74  55           je short JSbuilde.0040365B
00403606      80FB  30         cmp bl,30                     0



```
00403609       75 13                 jnz short JSbuilde.0040361E

0040360B       8A1E                  mov bl,byte ptr ds:[esi]

0040360D       46                    inc esi

0040360E       80FB 78               cmp bl,78

00403611       74 48                 je short JSbuilde.0040365B

00403613       80FB 58               cmp bl,58

00403616       74 43                 je short JSbuilde.0040365B

00403618       84DB                  test bl,bl

0040361A       74 20                 je short JSbuilde.0040363C

0040361C       EB 04                 jmp short JSbuilde.00403622

0040361E       84DB                  test bl,bl

00403620       74 2D                 je short JSbuilde.0040364F

00403622       80EB 30               sub bl,30                         得到机器码的十进制数 6

00403625       80FB 09               cmp bl,9                          与 9 相比较

00403628       77 25                 ja short JSbuilde.0040364F

0040362A       39F8                  cmp eax,edi               EAX=0000000 EDI=0CCCCCCC

0040362C       77 21                 ja short JSbuilde.0040364F       不会跳

0040362E       8D0480                lea eax,dword ptr ds:[eax+eax*4]       EAX*5 第一次为 0

00403631       01C0                  add eax,eax           EAX=0＋0

00403633       01D8                  add eax,ebx            EAX=0+6

00403635       8A1E                  mov bl,byte ptr ds:[esi]            取得第二位机器码

00403637       46                    inc esi                          ESI 指向第三位机器码

00403638       84DB                  test bl,bl

0040363A     ^ 75 E6                 jnz short JSbuilde.00403622
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

第一次 `EAX＝6`

`EAX＝6*5*2+6＝6*A+6=42`

`EAX＝42*A+0=294`

`EAX＝294*A+5=19cd`

`EAX＝19cd*A+0=10202`

`EAX＝10202*A+7=A141B`

`EAX＝A141B*A+6=64C914`

`EAX＝64C914*A+1=3EFDAC9` (最终结果)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

省略许多代码

```
0040369A       C3                    retn     返回

0040369B       90                    nop
```

返回到这里

```
004090F8       8BF0                  mov esi,eax 将机器码转换的注册码（下面也会将假码运算后的结果放在这里）送到 ESI 中 ESI 中为 3EFDAC9
```

省略许多代码

```
00409120       C3                    retn     返回，然后进入第二个关键 Call
```

### 第二个关键 Call 爱兰

看寄存器的值：

`EAX 03EFDAC9` （这是上一 Call，通过机器码计算的结果）

```
004C9384       55                    push ebp

004C9385       8BEC                  mov ebp,esp

004C9387       6A 00                 push 0

004C9389       53                    push ebx

004C938A       56                    push esi

004C938B       8BF0                  mov esi,eax

004C938D       33C0                  xor eax,eax

004C938F       55                    push ebp

004C9390       68 52944C00           push JSbuilde.004C9452

004C9395       64:FF30               push dword ptr fs:[eax]

004C9398       64:8920               mov dword ptr fs:[eax],esp

004C939B       BB D9D10E00           mov ebx,0ED1D9                 这个数用来干什么？看下面：

004C93A0       8D45 FC               lea eax,dword ptr ss:[ebp-4]

004C93A3       BA 68944C00           mov edx,JSbuilde.004C9468      ; ASCII "wangshuang"

004C93A8       E8 5FB7F3FF           call JSbuilde.00404B0C

004C93AD       8B45 FC               mov eax,dword ptr ss:[ebp-4]

004C93B0       E8 77B9F3FF           call JSbuilde.00404D2C

004C93B5       85C0                  test eax,eax

004C93B7       7E 1B                 jle short JSbuilde.004C93D4

004C93B9       BA 01000000           mov edx,1

004C93BE       8B4D FC               mov ecx,dword ptr ss:[ebp-4]

004C93C1       0FB64C11 FF           movzx ecx,byte ptr ds:[ecx+edx-1]

004C93C6       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C93C9       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C93CC       03D9                  add ebx,ecx

004C93CE       03DE                  add ebx,esi

004C93D0       42                    inc edx

004C93D1       48                    dec eax

004C93D2     ^ 75 EA                 jnz short JSbuilde.004C93BE
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

这个过程总结来说是：

`EBX=ED1D9+(77*5*5)＋3EFDAC9=ED1D9+(77*19)＋3EFDAC9＝3FEB841`

`EBX=3FEB841+(61*19)+3EFDAC9=7EE9C83`

……

`wangshuang` 的 ASCII 码分别为

`77`       `61`      `6E`      `67`      `73`      `68`      `75`      `61`      `6N`      `67`
(与十六进制数 `19` 相乘分别为：)
`B9F`      `979`     `ABE`     `A0F`     `B3B`     `A28`     `B6D`     `979`     `ABE`     `A0F`
(再与 `3EFDAC9` 相加分别为：)
`3EFE688`  `3EFE442` `3EFE587` `3EFE4D8` `3EFE604` `3EFE4F1` `3EFE636` `3EFE442` `3EFE587` `3EFE4D8`
`3FEB841`  `7EE9C83` > `BDE820A` > `FCE66E2` > `13BE4CE6` > `17AE31D7` > `1B9E180D` > `1F8DFC4F` > `237DE1D6` > `276DC51E`

最终：`EBX=276DC6AE`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

这一段代码对 `yaoyuan` 进行运算，位数为 7 位

```
004C93E9       85C0                  test eax,eax

004C93EB       7E 1B                 jle short JSbuilde.004C9408

004C93ED       BA 01000000           mov edx,1

004C93F2       8B4D FC               mov ecx,dword ptr ss:[ebp-4]

004C93F5       0FB64C11 FF           movzx ecx,byte ptr ds:[ecx+edx-1]

004C93FA       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C93FD       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C9400       03D9                  add ebx,ecx          第一次 EBX=276DC6AE＋79*5*5＝276DD27F

004C9402       03DE                  add ebx,esi           第一次：EBX=276DD27F+3EFDAC9＝2B5DAD48

004C9404       42                    inc edx

004C9405       48                    dec eax

004C9406     ^ 75 EA                 jnz short JSbuilde.004C93F2
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`yaoyuan` 的 ASCII 码分别是：

`79`     `61`     `6F`     `79`     `75`     `61`     `6N`
(分别与十六进制 `19` 相乘，结果如下：)
`BD1`    `979`    `AD7`    `BD1`    `B6D`    `979`    `ABE`
(又……，结果如下：)
`2B5DAD48` `2F4D918A` `333D772A` `372D5DC4` `3B1D43FA` `3F0D283C` `42FD0DC3`

`EBX=276DC6AE+79*5*5+3EFDAC9=276DC6AE+79*19+3EFDAC9＝2B5DAD48`

`EBX=2B5DAD48+61*19+3EFDAC9＝……`

结束得到：

`EBX 42FD0DC3` 得到这个结果

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

这一段对：`jsbuilder` 进行运算

`JSBuilder` 为 9 位  `ASCII` 码分别为 `4A 53 42 75 69 6C 64 65 72`

```
004C9421       BA 01000000           mov edx,1

004C9426       8B4D FC               mov ecx,dword ptr ss:[ebp-4]        ; JSbuilde.004C948C

004C9429       0FB64C11 FF           movzx ecx,byte ptr ds:[ecx+edx-1]

004C942E       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C9431       8D0C89                lea ecx,dword ptr ds:[ecx+ecx*4]

004C9434       03D9                  add ebx,ecx

004C9436       03DE                  add ebx,esi
```



### 汇编代码段

```
004C9438       42                  inc edx  
004C9439       48                  dec eax  
004C943A     ^ 75 EA               jnz short JSbuilde.004C9426  
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

### 第一次计算 EBX 值

第一次 `EBX=4A*5*5+42FD0DC3+3EFDAC9=4A*19+42FD0DC3+3EFDAC9`  

……  

结果得到：  

`EBX 666C1398` 得到这个结果   

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

省略一些代码           

```
004C9451       C3                  retn   返回，进入第三个关键 Call  
```

### 第三个关键 Call

```
004090E8       53                  push ebx  
004090E9       56                  push esi  
004090EA       83C4 F4             add esp,-0C  
004090ED       8BD8                mov ebx,eax                 将假码地址放到 EBX，EBX 原来保存机器码经过一系列运算后的数 666C1398  
004090EF       8BD4                mov edx,esp  
004090F1       8BC3                mov eax,ebx  
004090F3       E8 D8AFFFF          call JSbuilde.004035D0               Call1  （对假注册码进行运算，产生一个中间值）         
004090F8       8BF0                mov esi,eax  
004090FA       833C24 00           cmp dword ptr ss:[esp],0  
004090FE       74 19               je short JSbuilde.00409119  
00409100       895C24 04           mov dword ptr ss:[esp+4],ebx  
00409104       C64424 08 0B        mov byte ptr ss:[esp+8],0B  
00409109       8D5424 04           lea edx,dword ptr ss:[esp+4]  
0040910D       A1 C87B4F00         mov eax,dword ptr ds:[4F7BC8]  
00409112       33C9                xor ecx,ecx  
00409114       E8 17FAFFFF         call JSbuilde.00408B30               Call2（这个 Call 不是关键 ，别管它）  
00409119       8BC6                mov eax,esi  
0040911B       83C4 0C             add esp,0C  
0040911E       5E                  pop esi  
0040911F       5B                  pop ebx  
00409120       C3                  retn  
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*CALL1\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

```
004035D0       53                  push ebx  
```

省略许多代码，无非是注册码要求是在 0-9 之间的数字

```
0040361E       84DB                test bl,bl  
00403620       74 2D               je short JSbuilde.0040364F  
00403622       80EB 30             sub bl,30                       *****这里对假码进行运算  
00403625       80FB 09             cmp bl,9  
00403628       77 25               ja short JSbuilde.0040364F  
0040362A       39F8                cmp eax,edi                       (注意，前面省略的代码中，有给 EAX 赋值 0 的语句，现在 EAX 中为 0）  
0040362C       77 21               ja short JSbuilde.0040364F  
0040362E       8D0480              lea eax,dword ptr ds:[eax+eax*4]    第一次，在这里，EAX 中为 0  
00403631       01C0                add eax,eax                     EAX 作为累加器  
00403633       01D8                add eax,ebx                     EBX 中是假注册码，第一位为 1（我输入的假注册码是 1234567890），那么第一次，这里 EAX=1  
00403635       8A1E                mov bl,byte ptr ds:[esi]        下一位假码放在 EBX 的低八位  
00403637       46                  inc esi  
00403638       84DB                test bl,bl  
0040363A     ^ 75 E6               jnz short JSbuilde.00403622         *****  
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

因为我输入的假注册码为 1234567890，所以第一次，`EAX=1`   

以后分别为：        第二次 `EAX＝1*A+2＝C`  

                    第三次：`EAX＝C*A+3＝7B`

                    第四次 `EAX=7B*A+4=4D2`

                    第五次 `EAX=4D2*A+5=3039`

                    第六次 `EAX=3039*A+6=1E240`

                    第七次 `EAX=1E240*A+7=12D687`

                    第八次 `EAX=12D687*A+8=BC614E` 

                    第九次 `EAX=BV614E*A+9=75BCD15`

                    第十次 `EAX=75BCD15*A+0=499602D2` 

 最后得到一个中间数：  

`EAX 499602D2`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  

省略许多代码

```
0040369A       C3                  retn                返回去:关键比较  
```

现在用逆运算算出我的本机注册码：  

十六进制数：`666C1398 mod A` 余数就是注册码  

除数 被除数（商）     余数  

`A`    `666C1398`    

`A`    `A3E01F5`             6  

`A`    `1063365`             3  

`A`    `1A3856`              9  

`A`    `29F3B`               8  

`A`    `431F`                5  

`A`    `6B6`                 3  

`A`    `AB`                  8  

`A`    `11`                  1  

`1`                    7  

我的注册码：1718358936  

用 VB6 编译的注册算号器源码：  

省略

网络情缘

QQ339171218

UC:79656864

           禁止依据此文发放注册机！！ 

`4dd466d0e1e0cc123afbe504da2462ca`
