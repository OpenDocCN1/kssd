
`mov eax, dword ptr [ebp-20]`

`:004617B7 3B45EC`

`cmp eax, dword ptr [ebp-14]` <------异或后的值与注册码的1、2位比较

`:004617BA 7F07`

`jg 004617C3`

`:004617BC 8145E0FF000000` `add dword ptr [ebp-20],`

`000000FF` <---如果小于，则加0xff

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004617BA(C)

|

`:004617C3 8B45EC`

`mov eax, dword ptr [ebp-14]`

`:004617C6 2945E0`

`sub dword ptr [ebp-20], eax` <-----然后减去注册码的1、2位 (0x12)

`:004617C9 8D45D8`

`lea eax, dword ptr [ebp-28]` <-----第3、4位 (0x34)

`:004617CC 8B55E0`

`mov edx, dword ptr [ebp-20]` <-----减完后的值 (0x30)

`:004617CF E85424FAFF` `call 00403C28` <-------------------???????????

`:004617D4 8B55D8`

`mov edx, dword ptr [ebp-28]` <-----减完后的值0x30

`:004617D7 8B4508`

`mov eax, dword ptr [ebp+08]` <-----0

`:004617DA E82925FAFF` `call 00403D08` <-------------------放入堆栈

`:004617DF 8B4508`

`mov eax, dword ptr [ebp+08]` <-----0x30

`:004617E2 8B45E4`

`mov eax, dword ptr [ebp-1C]` <-----第3、4位 (0x34)

`:004617E5 8945EC`

`mov dword ptr [ebp-14], eax` <-----把第1、2位换成3、4位

`:004617E8 8345E802` `add`

`dword ptr [ebp-18], 00000002` <-----3+2=5

`:004617EC 8B45FC`

`mov eax, dword ptr [ebp-04]` <-----假注册码

`:004617EF E80C25FAFF` `call 00403D00` <-------------------假注册码的长度0x14=>eax

`:004617F4 3B45E8`

`cmp eax, dword ptr [ebp-18]` <-----注册码长度与[ebp-18]比较

`:004617F7 0F8F6CFFFFFF` `jg 00461769` <---------------------小则跳，循环计算

=================================================================================================

### 总结一下：

每1位用户名需要2位注册码，9位的用户名就需要18位的注册码，再加最后两位(见下面），共需20位注册码

每2位注册妈组成一个单元，如下：

注册码：** ** ** ** ** ** ** ** ** **

单元： i0 i1 i2 i3 i4 i5 i6 i7 i8 i9

用户名：`lancelot[CCG]` 的前9位 `lancelot[` 转换成16进制 0x6c 0x61 0x6e 0x63 0x65 0x6c 0x6f

0x74 0x5b

固定字符串：`vE9nT$Ma.` 转换成16进制-----------------> 0x76 0x45 0x39 0x6e 0x54 0x24 0x4d

0x61 0x2e

我们要使得：1) (0x76 XOR i1)-i0==0x6c--->l

2) (0x45 XOR i2)-i1==0x61--->a

3) (0x39 XOR i3)-i2==0x6e--->n

4) (0x6e XOR i4)-i3==0x63--->c

5) (0x54 XOR i5)-i4==0x65--->e

6) (0x24 XOR i6)-i5==0x6c--->l

7) (0x4d XOR i7)-i6==0x6f--->o

8) (0x61 XOR i8)-i7==0x74--->t

9) (0x2e XOR i9)-i8==0x5b--->[<---------现在知道为什么要再加2位了吧

因此可进行如下逆运算：先假设 `i0==0x6e` <---------可任意

1) (0x6c+i0) XOR 0x76==0xac=======>i1

2) 因为：0x61+i1==0x61+0xac==0x10d>0xff 所以: 0x10d-0xff==0xe,

0xe XOR 0x45==0x4b, 以下同

(0x61
