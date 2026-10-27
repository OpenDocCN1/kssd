
实际上对于`str_g`我们可以用一个简单的运算找到与其等价的大数从而将其缩小

在Elgamal Signature中有如下等式:

`Base^Exponent = Power (mod Modulus)`

即`g^x = y mod p`,也可以写成`y = g^x mod p`

那么如果觉得上面的`str_g`太冗长的话，

我们可以做如下的运算

`FGIntMod(FGInt_g, FGInt_p, FGInt_base)`,我们得到`FGInt_base = 456996142523`

也就是说用`FGInt_g`先去模`FGInt_p`得到小于`FGInt_p`的`FGInt_base`

根据模运算的性质，我们可以知道`FGInt_base`与`FGInt_g`是等价的。

---------------------------------------------------------------

```asm
0047D709   |.  8D55 E8     lea edx,dword ptr ss:[ebp-18]
0047D70C   |.  E8 A7D0FFFF call OTP.0047A7B8
0047D711   |.  8D45 DC     lea eax,dword ptr ss:[ebp-24]
0047D714   |.  E8 736DF8FF call OTP.0040448C
0047D719   |.  BB 01000000 mov ebx,1
0047D71E   |>  8D45 C8     /lea eax,dword ptr ss:[ebp-38]
0047D721   |.  8B55 F8     |mov edx,dword ptr ss:[ebp-8]
0047D724   |.  8A541A FF   |mov dl,byte ptr ds:[edx+ebx-1]
0047D728   |.  E8 3F6FF8FF |call OTP.0040466C                    ;LStrFromChar
0047D72D   |.  8B55 C8     |mov edx,dword ptr ss:[ebp-38]
0047D730   |.  8D45 DC     |lea eax,dword ptr ss:[ebp-24]
0047D733   |.  E8 1470F8FF |call OTP.0040474C                    ;LStrCat
0047D738   |.  43          |inc ebx
0047D739   |.  8B45 F8     |mov eax,dword ptr ss:[ebp-8]
0047D73C   |.  807C18 FF 2>|cmp byte ptr ds:[eax+ebx-1],2D
0047D741   |.^ 75 DB       \jnz short OTP.0047D71E               ;取'-'前面的字符串
0047D743   |.  43          inc ebx
0047D744   |>  8D45 C4     /lea eax,dword ptr ss:[ebp-3C]
0047D747   |.  8B55 F8     |mov edx,dword ptr ss:[ebp-8]
0047D74A   |.  8A541A FF   |mov dl,byte ptr ds:[edx+ebx-1]
0047D74E   |.  E8 196FF8FF |call OTP.0040466C
0047D753   |.  8B55 C4     |mov edx,dword ptr ss:[ebp-3C]
0047D756   |.  8D45 D8     |lea eax,dword ptr ss:[ebp-28]
0047D759   |.  E8 EE6FF8FF |call OTP.0040474C
0047D75E   |.  43          |inc ebx
0047D75F   |.  8B45 F8     |mov eax,dword ptr ss:[ebp-8]
0047D762   |.  E8 DD6FF8FF |call OTP.00404744
0047D767   |.  3BD8        |cmp ebx,eax
0047D769   |.^ 7E D9       \jle short OTP.0047D744              ;取剩下的字符串
0047D76B   |.  8D45 F0     lea eax,dword ptr ss:[ebp-10]      ;  FGInt_p
```

------------------------------------------------------------------

`[ebp-10]`所指向的是刚刚初始化后的大数,在这里不妨总结一下`FGInt`的大数格式，并与其它几种大数库的格式进行一些简单的比较。

在内存中`[ebp-10]`的数据是这样的。

```
0012F5E0   01 54 E4 00 B4 73 E5 00                          T?大?容?\_X
```

注意到第一个`byte`是`01`,这是`FGInt`的符号位，后面一个`dword`是`00E573B4`

这是指向真正的大数数据地址的`pointer`

跟到`00E573B4`处看看

```
00E573B4   02 00 00 00 00 00 00 00 85 82 17 76 00 00 00 00  .......?v....
00E573C4   F8 02 00 00 00 00 00 00                          ?......(
```

其中的`02 00 00 00`即`00000002`,这是大数的长度，`FGInt`是用`2^31`进制（即2的31次方)来表示大数的,而`miracl`是用`2^32`进制，`freelip`是用`2^30`进制,在相互转换的时候要注意了.

可以看看`FGInt`的Delphi定义:

```delphi
   TSign = (negative, positive);

   TFGInt = Record
      Sign : TSign;
      Number : Array Of LongWord;
```

呵呵，与我们的判断正好相符。

兄弟们可能会注意到在两个数据中间会有一个值为`0`的dword,可能这是在老版本的`FGInt`中的

在使用主页的最新`FGInt`在`Delphi7`下编译后，这中间是没有值为`0`的dword的，即是如下

格式

```
00D53FE4   02 00 00 00 85 82 17 76 F8 02 00 00 00 00 00 00  ...?v?......
```

在上面，程序取了`'-'`前面的字符串，分别作为Elgamal签名的字符串，即`str_a`, `str_b`

至此，我们得到了用于`ElGamalVerify`的全部参数

`Procedure ElGamalVerify(Var g, y, p : TFGInt; a, b, M : String; Var ok : Boolean);`

--------------------------------------------------------------------------
