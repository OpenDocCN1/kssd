
# BLOWFISH算法

**BLOWFISH算法**

作者：夜月

联系：`luoyi_ly1@sina.com`

时间：2001年10月6日

范例：[BlowFish's CrackMe1](https://www.kanxue.com/exercise/cryptogram/Blowfish/Blowfish.zip)

注册机：[Bfkeygen](https://www.kanxue.com/exercise/cryptogram/Blowfish/bfkeygen.zip)

## 一、BlowFish算法说明 （文中数据类型以Tc2.0为准）

BlowFish算法用来加密64Bit长度的字符串。

BlowFish算法使用两个“盒”――`unsigned long pbox[18]`和`unsigned long sbox[4,256]`。

BlowFish算法中，有一个核心加密函数:`BF_En`(后文详细介绍）。该函数输入64位信息，运算后，以64位密文的形式输出。 用BlowFish算法加密信息，需要两个过程：

1. 密钥预处理
2. 信息加密

分别说明如下：

密钥预处理：

BlowFish算法的源密钥――`pbox`和`sbox`是固定的。我们要加密一个信息，需要自己选择一个`key`，用这个`key`对`pbox`和`sbox`进行变换，得到下一步信息加密所要用的`key_pbox`和`key_sbox`。具体的变化算法如下：

1) 用`sbox`填充`key_sbox`

2) 用自己选择的`key`8个一组地去异或`pbox`，用异或的结果填充`key_pbox`。`key`可以循环使用。

比如说：选的`key`是`"abcdefghijklmn"`。则异或过程为：

`key_pbox[0]=pbox[0]^abcdefgh`

`key_pbox[1]=pbox[1]^ijklmnab`

…………

…………

如此循环，直到`key_box`填充完毕。

3) 用`BF_En`加密一个全0的64位信息，用输出的结果替换`key_pbox[0]`和`key_pbox[1]`。`i=0`

4) 用`BF_En`加密替换后的`key_pbox[i],key_pbox[i+1]`，用输出替代`key_pbox[i+2]`和`key_pbox[i+3]`

5) `i+2`，继续第4步，直到`key_pbox`全部被替换

6) 用`key_pbox[16]`和`key_pbox[17]`做首次输入（相当于上面的全0的输入），用类似的方法，替换`key_sbox`

信息加密。信息加密就是用函数把待加密信息`x`分成32位的两部分:`xL,xR` `BF_En`对输入信息进行变换，`BF_En`函数详细过程如下：

对于`i=1`至`16`

`xL=xL^Pi`

`xR=F(xL)^xR`

交换`xL`和`xR`(最后一轮取消该运算)

`xR=xR^P17`

`xL=xL^P18`

重新合并`xL`和`xR`

函数`F`见下图:

输出为：`F(xL)=((((S[1,a]+S[2,b])MOD 4294967296)^s[3,c])+S[4,d])MOD 4294967296`

把`xL`分成4个8位分组:`a,b,c`和`d`

重新合并后输出的结果就是我们需要的密文。

用BlowFish算法解密，同样也需要两个过程。

1. 密钥预处理
2. 信息解密

密钥预处理的过程与加密时完全相同

信息解密的过程就是把信息加密过程的`key_pbox`逆序使用即可。

可以看出，选择不同的`key`，用BlowFish算法加密同样的信息，可以得出不同的结果。

要破解BlowFish算法，就是要得到BlowFish算法的`key`。所以，使用BlowFish算法进行加密，最重要的也就是`key`的选择以及`key`的保密。其中`key`的选择可以使用`bf_sdk`中的`_WeakKey`函数进行检验。以下是该函数的说明：

源文：

---------------------------------------------------------------------------------------
`_WeakKey`
Function  : Test if the generated Boxes are weak
Argument  : none
Return    : AX = Status (1=weak, 0=good)
Affects    : AX, BX, CX, DX, SI, DI, direction Flag
Description:
After "_InitCrypt" you should test the Boxes with this function.
          If they provide a weakness which a cryptoanalyst
could use to
               break the cipher
a "1" is returned. In this case you should
    reload the original boxes and let the user choose a different
              password.
---------------------------------------------------------------------------------------

译文：

---------------------------------------------------------------------------------------
`_WeakKey`
功能：测试产生的box是否安全
参数：无
返回：AX=1 不安全；AX=0  安全
影响：AX, BX, CX, DX, SI, DI, 方向标志
描述：使用"`_InitCrypt`"函数产生用于加密的Boxes后，你应该用这个函数测试产生的Boxes是否安全。如果该key产生的Boxes不安全――可以被密码分析者通过分析Boxes得到key，那么，你应该采用另外一个key产生一个安全的Boxes用来加密。
---------------------------------------------------------------------------------------

## 二、BlowFish's CrackMe1分析

由于该CrackMe主要是测试你的密码学知识，所以没有在其他方面设关卡。为了减小文件体积，缩短大家下载的时间，用`upx`加了壳，直接用Trw2000的`"PNewSec+Makepe"`很方便地就能脱掉壳。

用常规的方法，很快找到下面关键比较处：

```
:004015D9 51
          push ecx
:004015DA 52
                push edx
:004015DB
6880894000              push 00408980
:004015E0 E8EBFAFFFF              call 004010D0
    //BF_De(sn)
:004015E5 8B442464
      mov eax, dword ptr [esp+64]
:004015E9 8B0DF0994000
          mov ecx, dword ptr [004099F0]
:004015EF
83C41C                  add esp,
0000001C
:004015F2 3BC1
    cmp eax, ecx      //比较
:004015F4 7529
                  jne 0040161F
:004015F6 8B4C244C                mov
ecx, dword ptr [esp+4C]
:004015FA A1EC994000
    mov eax, dword ptr [004099EC]
:004015FF 3BC8
              cmp ecx, eax
//比较
:00401601 751C
    jne 0040161F
:00401603 6A30
          push 00000030
```

由于BlowFish算法加密，解密输出的信息都是64Bit的，所以要进行两次比较。

我们既然知道了他对我们的`sn`进行的变换是`BF_De`，那么，很显然，我们要找到程序初始化`key_pbox`和`key_sbox`的地方。跟进`4015E0`的Call，找到`key_pbox`在`408980`处，下`bpm`，然后跟踪，分析，找到程序初始化`key_pbox`和`key_sbox`的地方，如下：

```
:004016C0 50
      push eax
* Possible StringData Ref from Data Obj
->"CrackingForFun"
                  |
:004016C1
6844804000              push 00408044
:004016C6 6880894000              push 00408980
:004016CB E860FAFFFF              call
00401130  //初始化Boxes
```

由此我们知道了`BF_De(sn)`的`key`是`"CrackingForFun"`。

问题的一半已经解决了。下面我们来看用来比较的另外的64Bit的数是从何而来。

`bpm 4099EC w`

跟踪分析后，发现这个用来比较的数是由`BF_En(ComputerID,key="ChinaCrackingGroup")`生成。

至此，我们可以写出注册机的算法：

`sn=BF_En((BF_En(ComputerID,key="ChinaCrackingGroup"),key="CrackingForFun")`

只要你编程够强，密码学也还过得去，写出这个东西的注册机就不是困难的事情了。

附：

ComputerID的产生

如果你对这个CrackMe很有兴趣，还想研究一下他的ComputerID是如何产生的，也可以继续跟踪，分析，在这里，我给处我分析的结果：

`ComputerID=BF_En(0776f6c62h, 068736966h,key=PW_1)`

其中，`PW_1`就是你的Windows版本号，可以在“系统属性”里头看到，也就是注册表中的

`H_L_M\Software\Microsoft\Windows\CurrentVersion`

中的`ProductId`项。在我的机器上是：

`"25001-OEM-0080247-46673"`

注册机源码里头有一些语句没有派上用场，用“;”屏蔽了，如果你有兴趣，可以把前面的`;`号去掉然后把`.data`段里头的`PW_1`换成你机器的`ComputerID`,再按照程序中的说明自己修改一下源程序，用`Masm32V6`重新编译，直接按Generate，也能得到正确的序列号。

## 三、注册机源码

```
;BlowFish's Crackme's KeyGen Writen By 夜月[CCG]

;Any Questions,Please E-Mail To luoyi.ly@yeah.net
;Thancks To Garfield,BlowFish,Toye

;软件流程：
;1.GetVersion得到机器Windows版本号。PW_1
;2.固定字符串"ChinaCrackingGroup"。PW_2

;3.固定字符串"CrackingForFun"。PW_3
;4.你输入的字符串。sn
;BF_En(0776f6c62h,
068736966h,key=PW_1)得到Computer ID
;BF_En(ComputerID,key=PW_2)得到MagicNum
;IF(BF_De(sn,key=PW_3)==MagicNum) Then Registed OK!


      .386
       .model flat,stdcall

    option casemap:none
 include        windows.inc

 include        user32.inc
 include
  kernel32.inc
 include        comctl32.inc
 include        comdlg32.inc
 include
  masm32.inc

 includelib      masm32.lib
 includelib
    user32.lib
 includelib      kernel32.lib
 includelib      comctl32.lib
 includelib
 comdlg32.lib

 DLG_MAIN        equ    100

 IDGEN          equ    10
 Edit1
         equ    11
 Edit2
     equ    12

 len_PW_1
   equ offset data1_p - offset PW_1

 _ProcDlgMain    PROTO
   :DWORD,:DWORD,:DWORD,:DWORD
 _Math
 PROTO    :DWORD,:DWORD,:DWORD
 BlowFish_En
   PROTO    :DWORD,:DWORD
 BlowFish_Fun    PROTO
   :DWORD
 BlowFish_Init  PROTO    :DWORD,:DWORD


         .data?
 hInstance      dd
     ?

         .data
 ;如果你直接用ComputerID产生序列号，你应该把PW_1换成你自己机器的Windows版本号

 ;PW_1    db "25001-OEM-0080247-46673"
 PW_2    db
 "ChinaCrackingGroup"
 PW_3    db "CrackingForFun"
 szID
       db 20 dup(0)
 szText      db 9 dup(0)

 data1_p     dd 0776f6c62h, 068736966h
 key    dd
 1058 dup (0)
 BFLOW      dd 0

 BFHIGH    dd
 0
 MYBFLOW    DD 0
 MYBFHIGH    DD 0

 pbox
   dd 0243f6a88h, 085a308d3h, 013198a2eh, 003707344h, 0a4093822h, 0299f31d0h

      dd 0082efa98h, 0ec4e6c89h, 0452821e6h, 038d01377h, 0be5466cfh,
 034e90c6ch
      dd 0c0ac29b7h, 0c97c50ddh, 03f84d5b5h, 0b5470917h,
 09216d5d9h, 08979fb1bh


 sbox1    dd 0d1310ba6h, 098dfb5ach,
 02ffd72dbh, 0d01adfb7h, 0b8e1afedh, 06a267e96h
      dd 0ba7c9045h,
 0f12c7f99h, 024a19947h, 0b3916cf7h, 00801f2e2h, 0858efc16h
      dd
 0636920d8h, 071574e69h, 0a458fea3h, 0f4933d7eh, 00d95748fh, 0728eb658h

   dd 0718bcd58h, 082154aeeh, 07b54a41dh, 0c25a59b5h, 09c30d539h, 02af26013h

      dd 0c5d1b023h, 0286085f0h, 0ca417918h, 0b8db38efh, 08e79dcb0h,
 0603a180eh
```
