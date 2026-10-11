
# Solid Encryption 注册算法分析(Elgamal Signature)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-755.htm`](https://www.pediy.com/kssd/pediy07/pediy7-755.htm)

题记：
少见使用基于 `DLP(Discrete Logarithm Problem,离散对数问题)`的 `Elgamal` 公钥算法来作为注册验证机制的程序，谨以此例作为对其的研究。

关于 `Elgamal` 可以参考：
`http://www.cacr.math.uwaterloo.ca/hac/about/chap8.pdf`
另外本站的教学也有 `http://www.pediy.com/tutorial/chap6/Chap6-3-3.htm`
以及《加密与解密（第二版）》

关于 `FGInt` 请参考：
`http://triade.studentenweb.org/GInt/gint.html`
简单的说是用于 `Delphi` 的一种实现了大数运算及几种公钥算法的库。

软件说明：
Official Site: `http://www.solid-programs.com/`
Introduction: `Solid Encryption`'s `One-Time Pad (OTP)` encryption a.k.a. `infinite key cryptography` is absolutely unbreakable if used correctly.
This encryption algorithm is well known and has been used for a long time by various intelligence agencies.
When no mistakes were made it has survived unbroken even when the resources off great world powers have been employed to break it.
Unlike most encryption schemes today, not only can you be certain that your communications will not be decoded this year, but that your secrets are safe for eternity.

用 `PEiD` 查无壳，`Delphi` 所写，`Krypto ANALyzer` 识别出 `RIPEMD-160` 散列算法和 `FGInt ElgmalVerify` 函数，使我们对程序中所有的算法有个大概的了解。有人说 `kanal` 插件有时候会不准，有的算法查不出来，但是这只是个辅助工具。
我平时习惯拿到一个程序后，如果没有加壳就用此插件看一看，有哪些算法，甚至有些加密算法库它都可以识别出来，如 `miracl`,`FGInt`.那么这样至少自己心里大概有个数，程序中有哪些算法，虽然这些算法并不一定用于注册的验证。
接着，用 `DeDe` 反编译一下主程序看看，在 `DeDe` 的单元信息中可以看到程序的确使用了 `FGInt` 和 `FGIntElGamal`
另外还使用了 `DCP`,这是一个在 `Delphi` 程序中使用相当广泛的加密算法库。
可以找到 `RegForm`，其中有个 `RegButtonClick` 事件，看了一下，注册验证就在其中。将从 `DeDe` 导出的 `map` 文件导入 `OD`.
可以有两种以上的办法找到注册验证的地方，一种是在 `DeDe` 中就可以看到事件处理的地址，另一种就是在 `OD` 中搜索参考字符串，找到与注册有关的字符串。在本例中，这两种方法都可以使用。
另外，还可以在下载 `FGInt` 并且编译 `FGInt` 后，做出其 `sig`,然后在 `IDA` 中 `apply` 一下就可以识别 `FGInt` 中的一些函数了。

```
代码:

* * *

  0047D2BC <>/.  55           push ebp                       ;  <-TRegForm@RegButtonClick
0047D2BD   |.  8BEC        mov ebp,esp
0047D2BF   |.  33C9        xor ecx,ecx
0047D2C1   |.  51          push ecx
0047D2C2   |.  51          push ecx
0047D2C3   |.  51          push ecx
0047D2C4   |.  51          push ecx
0047D2C5   |.  51          push ecx
0047D2C6   |.  51          push ecx
0047D2C7   |.  51          push ecx
0047D2C8   |.  53          push ebx
0047D2C9   |.  56          push esi
0047D2CA   |.  8BD8        mov ebx,eax
0047D2CC   |.  BE 684D4800 mov esi,OTP.00484D68
0047D2D1   |.  33C0        xor eax,eax
0047D2D3   |.  55          push ebp
0047D2D4   |.  68 C5D34700 push <OTP.->System.@HandleFinally;>
0047D2D9   |.  64:FF30     push dword ptr fs:\[eax\]
0047D2DC   |.  64:8920     mov dword ptr fs:\[eax\],esp
0047D2DF   |.  8BC3        mov eax,ebx
0047D2E1 <>|.  E8 16FEFFFF call <OTP.<-TRegForm@WriteCodeIni> ;  ->:TRegForm.WriteCodeIni()
0047D2E6   |.  8BC3        mov eax,ebx                      ;这里是将注册信息写入 code.ini 中,无关紧要
0047D2E8   |.  E8 33060000 call OTP.0047D920
0047D2ED   |.  8D55 FC     lea edx,dword ptr ss:\[ebp-4\]
0047D2F0   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D2F2 <>|.  8B80 1C0300>mov eax,dword ptr ds:\[eax+31C\]     ;  \*EditSerial:TEdit
0047D2F8 <>|.  E8 CF84FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D2FD   |.  8B45 FC     mov eax,dword ptr ss:\[ebp-4\]
0047D300   |.  50          push eax
0047D301   |.  8D55 F4     lea edx,dword ptr ss:\[ebp-C\]
0047D304   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D306 <>|.  8B80 080300>mov eax,dword ptr ds:\[eax+308\]     ;  \*EditName:TEdit
0047D30C <>|.  E8 BB84FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D311   |.  FF75 F4     push dword ptr ss:\[ebp-C\]
0047D314   |.  8D55 F0     lea edx,dword ptr ss:\[ebp-10\]
0047D317   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D319 <>|.  8B80 0C0300>mov eax,dword ptr ds:\[eax+30C\]     ;  \*EditAdr1:TEdit
0047D31F <>|.  E8 A884FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D324   |.  FF75 F0     push dword ptr ss:\[ebp-10\]
0047D327   |.  8D55 EC     lea edx,dword ptr ss:\[ebp-14\]
0047D32A   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D32C <>|.  8B80 100300>mov eax,dword ptr ds:\[eax+310\]     ;  \*EditAdr2:TEdit
0047D332 <>|.  E8 9584FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D337   |.  FF75 EC     push dword ptr ss:\[ebp-14\]
0047D33A   |.  6A 00       push 0
0047D33C   |.  8D55 E8     lea edx,dword ptr ss:\[ebp-18\]
0047D33F   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D341 <>|.  8B80 140300>mov eax,dword ptr ds:\[eax+314\]     ;  \*EditCountry:TEdit
0047D347 <>|.  E8 8084FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D34C   |.  FF75 E8     push dword ptr ss:\[ebp-18\]
0047D34F   |.  8D55 E4     lea edx,dword ptr ss:\[ebp-1C\]
0047D352   |.  8B06        mov eax,dword ptr ds:\[esi\]
0047D354 <>|.  8B80 180300>mov eax,dword ptr ds:\[eax+318\]     ;  \*EditEmail:TEdit
0047D35A <>|.  E8 6D84FCFF call OTP.004457CC                  ;  ->Controls.TControl.GetText(TControl):TCaption;
0047D35F   |.  FF75 E4     push dword ptr ss:\[ebp-1C\]
0047D362   |.  8D45 F8     lea eax,dword ptr ss:\[ebp-8\]
0047D365   |.  BA 06000000 mov edx,6
0047D36A <>|.  E8 9574F8FF call OTP.00404804                  ;  ->System.@LStrCatN;

* * *

```

-----------------------------------------------------------------------
将输入的信息连结起来

在后面可以看到，这是将要被 `signature` 的信息

将其记为 `str_M`

-----------------------------------------------------------------------

```
代码:

* * *

0047D36F   |.  8B55 F8     mov edx,dword ptr ss:\[ebp-8\]       ;
0047D372   |.  8BC3        mov eax,ebx                          ;
0047D374   |.  59          pop ecx
0047D375   |.  E8 C6020000 call OTP.0047D640                    ;这儿就是验证的地方
0047D37A   |.  84C0        test al,al
0047D37C   |.  74 17       je short OTP.0047D395
0047D37E   |.  A1 E4314800 mov eax,dword ptr ds:\[4831E4\]
0047D383   |.  8B00        mov eax,dword ptr ds:\[eax\]
0047D385   |.  B2 01       mov dl,1
0047D387   |.  8B08        mov ecx,dword ptr ds:\[eax\]
0047D389   |.  FF51 64     call dword ptr ds:\[ecx+64\]
0047D38C   |.  8BC3        mov eax,ebx
0047D38E <>|.  E8 694DFEFF call OTP.004620FC                    ;  ->Forms.TCustomForm.Close(TCustomForm);
0047D393   |.  EB 15       jmp short OTP.0047D3AA
0047D395   |>  6A 00       push 0                               ; /Arg1 = 00000000
0047D397   |.  66:8B0D D4D>mov cx,word ptr ds:\[47D3D4\]        ; |
0047D39E   |.  B2 01       mov dl,1                             ; |
0047D3A0   |.  B8 E0D34700 mov eax,OTP.0047D3E0                 ; |ASCII "Incorrect registration, please try again."
0047D3A5   |.  E8 4213FCFF call OTP.0043E6EC                    ; \\OTP.0043E6EC
0047D3AA   |>  33C0        xor eax,eax                          ; 注册不正确的提示
0047D3AC   |.  5A          pop edx
0047D3AD   |.  59          pop ecx
0047D3AE   |.  59          pop ecx
0047D3AF   |.  64:8910     mov dword ptr fs:\[eax\],edx
0047D3B2   |.  68 CCD34700 push OTP.0047D3CC
0047D3B7   |>  8D45 E4     lea eax,dword ptr ss:\[ebp-1C\]
0047D3BA   |.  BA 07000000 mov edx,7
0047D3BF <>|.  E8 EC70F8FF call OTP.004044B0                    ;  ->System.@LStrArrayClr(void;void;Integer);
0047D3C4   \\.  C3          retn

* * *

```

下面来看看 `call OTP.0047D640` 中的代码:



### 代码段 1

```
0047D640   /$  55           push ebp
0047D641   |.  8BEC         mov ebp,esp
0047D643   |.  83C4 C4      add esp,-3C
0047D646   |.  53           push ebx
0047D647   |.  56           push esi
0047D648   |.  33DB         xor ebx,ebx
0047D64A   |.  895D C4      mov dword ptr ss:[ebp-3C],ebx
0047D64D   |.  895D C8      mov dword ptr ss:[ebp-38],ebx
0047D650   |.  895D CC      mov dword ptr ss:[ebp-34],ebx
0047D653   |.  895D D0      mov dword ptr ss:[ebp-30],ebx
0047D656   |.  895D D4      mov dword ptr ss:[ebp-2C],ebx
0047D659   |.  895D DC      mov dword ptr ss:[ebp-24],ebx
0047D65C   |.  895D D8      mov dword ptr ss:[ebp-28],ebx
0047D65F   |.  894D F8      mov dword ptr ss:[ebp-8],ecx
0047D662   |.  8955 FC      mov dword ptr ss:[ebp-4],edx
0047D665   |.  8BF0         mov esi,eax
0047D667   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0047D66A   |.  E8 BD72F8FF  call OTP.0040492C
0047D66F   |.  8B45 F8      mov eax,dword ptr ss:[ebp-8]
0047D672   |.  E8 B572F8FF  call OTP.0040492C
0047D677   |.  8D45 F0      lea eax,dword ptr ss:[ebp-10]
0047D67A   |.  8B15 94A447> mov edx,dword ptr ds:[47A494]      ;  OTP.0047A498
0047D680   |.  E8 2F78F8FF  call OTP.00404EB4
0047D685   |.  8D45 E8      lea eax,dword ptr ss:[ebp-18]
0047D688   |.  8B15 94A447> mov edx,dword ptr ds:[47A494]      ;  OTP.0047A498
0047D68E   |.  E8 2178F8FF  call OTP.00404EB4
0047D693   |.  8D45 E0      lea eax,dword ptr ss:[ebp-20]
0047D696   |.  8B15 94A447> mov edx,dword ptr ds:[47A494]      ;  OTP.0047A498
0047D69C   |.  E8 1378F8FF  call OTP.00404EB4
0047D6A1   |.  33C0         xor eax,eax
0047D6A3   |.  55           push ebp
0047D6A4   |.  68 C5D74700  push OTP.0047D7C5
0047D6A9   |.  64:FF30      push dword ptr fs:[eax]
0047D6AC   |.  64:8920      mov dword ptr fs:[eax],esp
0047D6AF   |.  8B45 F8      mov eax,dword ptr ss:[ebp-8]
0047D6B2   |.  E8 8D70F8FF  call OTP.00404744
0047D6B7   |.  83F8 05      cmp eax,5                       ;注册码长度要大于 5
0047D6BA   |.  7D 07        jge short OTP.0047D6C3
0047D6BC   |.  33DB         xor ebx,ebx
0047D6BE   |.  E9 C7000000  jmp OTP.0047D78A
0047D6C3   |>  8D4D D4      lea ecx,dword ptr ss:[ebp-2C]
0047D6C6   |.  BA DCD74700  mov edx,OTP.0047D7DC           ;  ASCII "CECDCECCCAC6CDCBCDCCCECFCF"
0047D6CB   |.  8BC6         mov eax,esi
```

\------------------------------------------------------------------

`0047D6CD   |.  E8 46040000 call OTP.0047DB18`

这个 `call` 跟进去看非常像 `FGInt` 中的 `ConvertHexStringToBase256String` 这个过程，但是细看却发现不完全一样。

### 代码段 2

```
0047DB18   /$  55           push ebp
0047DB19   |.  8BEC         mov ebp,esp
0047DB1B   |.  83C4 F8      add esp,-8
0047DB1E   |.  53           push ebx
0047DB1F   |.  56           push esi
0047DB20   |.  57           push edi
0047DB21   |.  33DB         xor ebx,ebx
0047DB23   |.  895D F8      mov dword ptr ss:[ebp-8],ebx
0047DB26   |.  8BF9         mov edi,ecx
0047DB28   |.  8955 FC      mov dword ptr ss:[ebp-4],edx
0047DB2B   |.  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0047DB2E   |.  E8 F96DF8FF  call OTP.0040492C
0047DB33   |.  33C0         xor eax,eax
0047DB35   |.  55           push ebp
0047DB36   |.  68 E7DB4700  push OTP.0047DBE7
0047DB3B   |.  64:FF30      push dword ptr fs:[eax]
0047DB3E   |.  64:8920      mov dword ptr fs:[eax],esp
0047DB41   |.  8BC7         mov eax,edi
0047DB43   |.  E8 4469F8FF  call OTP.0040448C              ;  LStrClr
0047DB48   |.  BB 01000000  mov ebx,1
0047DB4D   |>  8B45 FC      /mov eax,dword ptr ss:[ebp-4]
0047DB50   |.  8A4418 FF    |mov al,byte ptr ds:[eax+ebx-1]
0047DB54   |.  3C 39        |cmp al,39
0047DB56   |.  76 15        |jbe short OTP.0047DB6D
0047DB58   |.  8B55 FC      |mov edx,dword ptr ss:[ebp-4]
0047DB5B   |.  25 FF000000  |and eax,0FF
0047DB60   |.  83E8 41      |sub eax,41
0047DB63   |.  83C0 0A      |add eax,0A
0047DB66   |.  8BF0         |mov esi,eax
0047DB68   |.  C1E6 04      |shl esi,4
0047DB6B   |.  EB 10        |jmp short OTP.0047DB7D
0047DB6D   |>  8B55 FC      |mov edx,dword ptr ss:[ebp-4]
0047DB70   |.  25 FF000000  |and eax,0FF
0047DB75   |.  83E8 30      |sub eax,30
0047DB78   |.  8BF0         |mov esi,eax
0047DB7A   |.  C1E6 04      |shl esi,4
0047DB7D   |>  43           |inc ebx
0047DB7E   |.  8B45 FC      |mov eax,dword ptr ss:[ebp-4]
0047DB81   |.  8A4418 FF    |mov al,byte ptr ds:[eax+ebx-1]
0047DB85   |.  3C 39        |cmp al,39
0047DB87   |.  76 12        |jbe short OTP.0047DB9B
0047DB89   |.  8B55 FC      |mov edx,dword ptr ss:[ebp-4]
0047DB8C   |.  25 FF000000  |and eax,0FF
0047DB91   |.  03F0         |add esi,eax
0047DB93   |.  83EE 41      |sub esi,41
0047DB96   |.  83C6 0A      |add esi,0A
0047DB99   |.  EB 0D        |jmp short OTP.0047DBA8
0047DB9B   |>  8B55 FC      |mov edx,dword ptr ss:[ebp-4]
0047DB9E   |.  25 FF000000  |and eax,0FF
0047DBA3   |.  03F0         |add esi,eax
0047DBA5   |.  83EE 30      |sub esi,30
0047DBA8   |>  43           |inc ebx
0047DBA9   |.  8D45 F8      |lea eax,dword ptr ss:[ebp-8]
0047DBAC   |.  8BD6         |mov edx,esi
0047DBAE   |.  80F2 FF      |xor dl,0FF
```

\-------------------------------------------------------------------------

主要是上面这个地方不同，多了一个 `xor`，因此在做注册机的时候也要多一个 `xor` 才对。

\-------------------------------------------------------------------------

### 代码段 3

```
0047DBB1   |.  E8 B66AF8FF  |call OTP.0040466C             ;  LStrFromChar
0047DBB6   |.  8B55 F8      |mov edx,dword ptr ss:[ebp-8]
0047DBB9   |.  8BC7         |mov eax,edi
0047DBBB   |.  E8 8C6BF8FF  |call OTP.0040474C             ;  LStrCat
0047DBC0   |.  8B45 FC      |mov eax,dword ptr ss:[ebp-4]
0047DBC3   |.  E8 7C6BF8FF  |call OTP.00404744             ;  LStrLen
0047DBC8   |.  3BD8         |cmp ebx,eax
0047DBCA   |.^ 7E 81        \jle short OTP.0047DB4D
0047DBCC   |.  33C0         xor eax,eax
0047DBCE   |.  5A           pop edx
0047DBCF   |.  59           pop ecx
0047DBD0   |.  59           pop ecx
0047DBD1   |.  64:8910      mov dword ptr fs:[eax],edx
0047DBD4   |.  68 EEDB4700  push OTP.0047DBEE
0047DBD9   |>  8D45 F8      lea eax,dword ptr ss:[ebp-8]
0047DBDC   |.  BA 02000000  mov edx,2
0047DBE1   |.  E8 CA68F8FF  call OTP.004044B0
0047DBE6   \\.  C3          retn
-----------------------------------------------------------------
0047D6D2   |.  8B45 D4      mov eax,dword ptr ss:[ebp-2C]    ;转换后的 Base10String '1213592423100'
0047D6D5   |.  8D55 E0      lea edx,dword ptr ss:[ebp-20]    ;这是上面初始化的 FGInt
0047D6D8   |.  E8 DBD0FFFF  call OTP.0047A7B8              ;Base10StringToFGInt,以下相同。
0047D6DD   |.  8D4D D0      lea ecx,dword ptr ss:[ebp-30]
0047D6E0   |.  BA 00D84700  mov edx,OTP.0047D800           ;  ASCII "CEC9CCCBCFC9C7C7CDCBC8CFC6"
0047D6E5   |.  8BC6         mov eax,esi
0047D6E7   |.  E8 2C040000  call OTP.0047DB18
0047D6EC   |.  8B45 D0      mov eax,dword ptr ss:[ebp-30]    ;1634068824709
0047D6EF   |.  8D55 F0      lea edx,dword ptr ss:[ebp-10]
0047D6F2   |.  E8 C1D0FFFF  call OTP.0047A7B8
0047D6F7   |.  8D4D CC      lea ecx,dword ptr ss:[ebp-34]
0047D6FA   |.  BA 24D84700  mov edx,OTP.0047D824           ;  ASCII "CAC9CFCDCFCAC9CDCDCDCACBCCCBCECBCBC8C8CEC8CFC9C7C8CDCDCDCDC7CACCCDCDC9CCC8C7CDCDC7CCCDCEC8CDC8C7CEC9C7CCC9CFCACDCBCFCFC7CAC9C7CEC7CDC9CBCFCFCBCCCAC9CDCDCFCBCDCBC9CECBC9CBC9CAC7C8C7CBCECCCECCCDCBC9CFC7C7CCCACACFCE"
0047D6FF   |.  8BC6         mov eax,esi
0047D701   |.  E8 12040000  call OTP.0047DB18
0047D706   |.  8B45 CC      mov eax,dword ptr ss:[ebp-34]
```

\---------------------------------------------------------------

这个时候，得到了 3 个十进制字符串，分别是：

`1213592423100`

`1634068824709`



`5602056222543414477170687222285322637822832172781683605240085681826400435622042461464658784131324608835501`

根据后面的分析可知，它们分别对应于 Elgamal Signature Scheme 的 3 个参数。

上面的程序可以表示如下：

```
str_y:='1213592423100';
Base10StringToFGInt(str_y,FGInt_y);
str_p:='1634068824709';
Base10StringToFGInt(str_p,FGInt_p);
str_g:='5602056222543414477170687222285322637822832172781683605240085681826400435622042461464658784131324608835501';
Base10StringToFGInt(str_g,FGInt_g);
```

实际上对于 `str_g` 我们可以用一个简单的运算找到与其等价的大数从而将其缩小。

在 Elgamal Signature 中有如下等式:

`Base^Exponent = Power (mod Modulus)`

即 `g^x = y mod p`，也可以写成 `y = g^x mod p`。

那么如果觉得上面的 `str_g` 太冗长的话，我们可以做如下的运算：

`FGIntMod(FGInt_g,FGInt_p,FGInt_base)`，我们得到 `FGInt_base=456996142523`。

也就是说用 `FGInt_g` 先去模 `FGInt_p` 得到小于 `FGInt_p` 的 `FGInt_base`。

根据模运算的性质，我们可以知道 `FGInt_base` 与 `FGInt_g` 是等价的。

---

```assembly
0047D709   |.  8D55 E8     lea edx,dword ptr ss:[ebp-18]
0047D70C   |.  E8 A7D0FFFF call OTP.0047A7B8
0047D711   |.  8D45 DC     lea eax,dword ptr ss:[ebp-24]
0047D714   |.  E8 736DF8FF call OTP.0040448C
0047D719   |.  BB 01000000 mov ebx,1
0047D71E   |>  8D45 C8     /lea eax,dword ptr ss:[ebp-38]
0047D721   |.  8B55 F8     |mov edx,dword ptr ss:[ebp-8]
0047D724   |.  8A541A FF   |mov dl,byte ptr ds:[edx+ebx-1]
0047D728   |.  E8 3F6FF8FF |call OTP.0040466C                   ;LStrFromChar
0047D72D   |.  8B55 C8     |mov edx,dword ptr ss:[ebp-38]
0047D730   |.  8D45 DC     |lea eax,dword ptr ss:[ebp-24]
0047D733   |.  E8 1470F8FF |call OTP.0040474C                   ;LStrCat
0047D738   |.  43          |inc ebx
0047D739   |.  8B45 F8     |mov eax,dword ptr ss:[ebp-8]
0047D73C   |.  807C18 FF 2>|cmp byte ptr ds:[eax+ebx-1],2D
0047D741   |.^ 75 DB       \jnz short OTP.0047D71E              ;取'-'前面的字符串
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
0047D76B   |.  8D45 F0     lea eax,dword ptr ss:[ebp-10]        ; FGInt_p
```

---

`[ebp-10]` 所指向的是刚刚初始化后的大数，在这里不妨总结一下 `FGInt` 的大数格式，并与其它几种大数库的格式进行一些简单的比较。

在内存中 `[ebp-10]` 的数据是这样的。

`0012F5E0 01 54 E4 00 B4 73 E5 00 T?磗?萞?_X`

注意到第一个 byte 是 `01`，这是 `FGInt` 的符号位，后面一个 dword 是 `00E573B4`。

这是指向真正的大数数据地址的 pointer。

跟到 `00E573B4` 处看看：

`00E573B4 02 00 00 00 00 00 00 00 85 82 17 76 00 00 00 00 .......厒 v....`
`00E573C4 F8 02 00 00 00 00 00 00 ?......(`

其中的 `02 00 00 00` 即 `00000002`，这是大数的长度。`FGInt` 是用 2³¹ 进制（即 2 的 31 次方）来表示大数的，而 `miracl` 是用 2³² 进制，`freelip` 是用 2³⁰ 进制，在相互转换的时候要注意了。

可以看看 `FGInt` 的 Delphi 定义：

```pascal
TSign = (negative, positive);
TFGInt = Record
  Sign : TSign;
  Number : Array Of LongWord;
```

呵呵，与我们的判断正好相符。

兄弟们可能会注意到在两个数据中间会有一个值为 0 的 dword，可能这是在老版本的 `FGInt` 中的情况。在使用主页的最新 `FGInt` 在 Delphi7 下编译后，这中间是没有值为 0 的 dword 的，即是如下格式：

`00D53FE4 02 00 00 00 85 82 17 76 F8 02 00 00 00 00 00 00 ...厒 v?......`

在上面，程序取了'-'前面的字符串，分别作为 Elgamal 签名的字符串，即 `str_a`, `str_b`。

至此，我们得到了用于 `ElGamalVerify` 的全部参数：

```pascal
Procedure ElGamalVerify(Var g, y, p : TFGInt; a, b, M : String; Var ok : Boolean);
```

---



```
代码:

0047D76E   |.  50           push eax
0047D76F   |.  8B45 DC     mov eax,dword ptr ss:[ebp-24]       ;str_a
0047D772   |.  50           push eax
0047D773   |.  8B45 D8     mov eax,dword ptr ss:[ebp-28]       ;str_b
0047D776   |.  50           push eax
0047D777   |.  8B45 FC     mov eax,dword ptr ss:[ebp-4]        ; str_M
0047D77A   |.  50           push eax
0047D77B   |.  8D4D E0     lea ecx,dword ptr ss:[ebp-20]       ; FGInt_y
0047D77E   |.  8D55 E8     lea edx,dword ptr ss:[ebp-18]       ; FGInt_g
0047D781   |.  8BC6         mov eax,esi
0047D783   |.  E8 70040000 call OTP.0047DBF8
-------------------------------------------------------
0047DBF8   /$  55           push ebp
0047DBF9   |.  8BEC         mov ebp,esp
0047DBFB   |.  83C4 DC     add esp,-24
0047DBFE   |.  53           push ebx
0047DBFF   |.  56           push esi
0047DC00   |.  57           push edi
0047DC01   |.  33DB         xor ebx,ebx
0047DC03   |.  895D E0     mov dword ptr ss:[ebp-20],ebx
0047DC06   |.  895D DC     mov dword ptr ss:[ebp-24],ebx
0047DC09   |.  8B75 14     mov esi,dword ptr ss:[ebp+14]
0047DC0C   |.  8D7D E8     lea edi,dword ptr ss:[ebp-18]
0047DC0F   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC10   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC11   |.  8BF1         mov esi,ecx
0047DC13   |.  8D7D F0     lea edi,dword ptr ss:[ebp-10]
0047DC16   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC17   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC18   |.  8BF2         mov esi,edx
0047DC1A   |.  8D7D F8     lea edi,dword ptr ss:[ebp-8]
0047DC1D   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC1E   |.  A5           movs dword ptr es:[edi],dword ptr >
0047DC1F   |.  8BD8         mov ebx,eax
0047DC21   |.  8D45 F8     lea eax,dword ptr ss:[ebp-8]
0047DC24   |.  8B15 94A447>mov edx,dword ptr ds:[47A494]       ;  OTP.0047A498
0047DC2A   |.  E8 8174F8FF call OTP.004050B0
0047DC2F   |.  8D45 F0     lea eax,dword ptr ss:[ebp-10]
0047DC32   |.  8B15 94A447>mov edx,dword ptr ds:[47A494]       ;  OTP.0047A498
0047DC38   |.  E8 7374F8FF call OTP.004050B0
0047DC3D   |.  8D45 E8     lea eax,dword ptr ss:[ebp-18]
0047DC40   |.  8B15 94A447>mov edx,dword ptr ds:[47A494]       ;  OTP.0047A498
0047DC46   |.  E8 6574F8FF call OTP.004050B0
0047DC4B   |.  8B45 10     mov eax,dword ptr ss:[ebp+10]
0047DC4E   |.  E8 D96CF8FF call OTP.0040492C
0047DC53   |.  8B45 0C     mov eax,dword ptr ss:[ebp+C]
0047DC56   |.  E8 D16CF8FF call OTP.0040492C
0047DC5B   |.  8B45 08     mov eax,dword ptr ss:[ebp+8]
0047DC5E   |.  E8 C96CF8FF call OTP.0040492C
0047DC63   |.  33C0         xor eax,eax
0047DC65   |.  55           push ebp
0047DC66   |.  68 E7DC4700 push OTP.0047DCE7
0047DC6B   |.  64:FF30      push dword ptr fs:[eax]
0047DC6E   |.  64:8920      mov dword ptr fs:[eax],esp
0047DC71   |.  8D4D E0     lea ecx,dword ptr ss:[ebp-20]
0047DC74   |.  8B55 10     mov edx,dword ptr ss:[ebp+10]
0047DC77   |.  8BC3         mov eax,ebx
0047DC79   |.  E8 9AFEFFFF call OTP.0047DB18
0047DC7E   |.  8D4D DC     lea ecx,dword ptr ss:[ebp-24]
0047DC81   |.  8B55 0C     mov edx,dword ptr ss:[ebp+C]
0047DC84   |.  8BC3         mov eax,ebx
0047DC86   |.  E8 8DFEFFFF call OTP.0047DB18
0047DC8B   |.  8B45 E0     mov eax,dword ptr ss:[ebp-20]       ;  str_a
0047DC8E   |.  50           push eax
0047DC8F   |.  8B45 DC     mov eax,dword ptr ss:[ebp-24]       ;  str_b
0047DC92   |.  50           push eax
0047DC93   |.  8B45 08     mov eax,dword ptr ss:[ebp+8]        ;  str_M
0047DC96   |.  50           push eax
0047DC97   |.  8D45 E7     lea eax,dword ptr ss:[ebp-19]       ;  flag
0047DC9A   |.  50           push eax
0047DC9B   |.  8D4D E8     lea ecx,dword ptr ss:[ebp-18]       ;  FGInt_p
0047DC9E   |.  8D55 F0     lea edx,dword ptr ss:[ebp-10]       ;  FGInt_y
0047DCA1   |.  8D45 F8     lea eax,dword ptr ss:[ebp-8]        ;  FGInt_g
0047DCA4   |.  E8 F3EFFFFF call OTP.0047CC9C                   ;  ElGamalVerify
0047DCA9   |.  8A5D E7     mov bl,byte ptr ss:[ebp-19]         ;  这个是 flag
----------------------------------------------------------------
0047D788   |.  8BD8         mov ebx,eax
0047D78A   |>  33C0         xor eax,eax
0047D78C   |.  5A           pop edx
0047D78D   |.  59           pop ecx
0047D78E   |.  59           pop ecx
0047D78F   |.  64:8910      mov dword ptr fs:[eax],edx
0047D792   |.  68 CCD74700 push OTP.0047D7CC
0047D797   |>  8D45 C4     lea eax,dword ptr ss:[ebp-3C]
0047D79A   |.  BA 07000000 mov edx,7
0047D79F   |.  E8 0C6DF8FF call OTP.004044B0
0047D7A4   |.  8D45 E0     lea eax,dword ptr ss:[ebp-20]
0047D7A7   |.  8B15 94A447>mov edx,dword ptr ds:[47A494]       ;  OTP.0047A498
0047D7AD   |.  B9 03000000 mov ecx,3
0047D7B2   |.  E8 0178F8FF call OTP.00404FB8
0047D7B7   |.  8D45 F8     lea eax,dword ptr ss:[ebp-8]
0047D7BA   |.  BA 02000000 mov edx,2
0047D7BF   |.  E8 EC6CF8FF call OTP.004044B0
0047D7C4   \.  C3           retn
```

到现在验证的流程清楚了

即 `ElgamalVerify(FGInt_g,FGInt_y,FGInt_p,str_a,str_b,str_M,flag);`

对于 `ElGamal`，任务就是要求出 `Base^Exponent = Power (mod Modulus)` 中的 `Exponent`，即 `y = g^x mod p` 中的 `x`，现在 `base`，`power` 和 `modulus` 都已经知道了

而且还可以用上面求出的 `FGInt_base` 来代替 `p`。

这是一个 `DLP(Discrete Logarithm Problem,离散对数问题)`，可以使用 `DLPTool` 来求出 `x`。

`DLPTool` 可以在这里下载 `http://www.pediy.com/tools/Cryptography.htm`

另外，还可以使用在线 `Discrete logarithm calculator(离散对数计算器)` 即 `http://www.alpertron.com.ar/DILOG.HTM`，在这里我们使用 `Discrete Logarithm calculator`，其中

`Base` 即可以输入 `5602056222543414477170687222285322637822832172781683605240085681826400435622042461464658784131324608835501` 也可以输入 `456996142523`，显然第二个更小一些

`Power` 输入 `str_y`，即 `1213592423100`

`Mod` 输入 `str_p`，即 `1634068824709`

然后点击 `Find Discrete Logarithm`，秒杀，呵呵

我们得到 `Exp=586292948503`，即所求的 `x`

`ElGamal` 在签名的时候，要生成一个随机数 `k`，使其与 `p-1` 互素，

`p-1=1634068824708`，将其分解得到其素数因子

```
PRIME FACTOR: 2
PRIME FACTOR: 2
PRIME FACTOR: 3
PRIME FACTOR: 29
PRIME FACTOR: 41
PRIME FACTOR: 114526831
```

因此在生成随机数的时候不要选择这几个数的任何一个或几个的乘积就可以了，比如 `2x2=4`，那么 `4` 也不能作为 `k` 的值

在这里我们选择 `1`，方便些.

现在可以做注册机了，代码如下:

```
Base10StringToFGInt('1',FGInt_k);
str_x:='586292948503';
Base10StringToFGInt(str_x,FGInt_x);
str_y:='1213592423100';
Base10StringToFGInt(str_y,FGInt_y);
str_p:='1634068824709';
Base10StringToFGInt(str_p,FGInt_p);
str_base:='456996142523';
Base10StringToFGInt(str_base,FGInt_base);
ElGamalSign(str_M,FGInt_p,FGInt_base,FGInt_x,FGInt_k,str256_a,str256_b);
For i:=1 To length(str256_a) Do
    str_a[i]:=chr(ord(str256_a[i]) xor $FF);
For i:=1 To length(str256_b) Do
    str_b[i]:=chr(ord(str256_b[i]) xor $FF);
ConvertBase256StringToHexString(str256_a,strhex_a);
ConvertBase256StringToHexString(str256_b,strhex_b);
str_sn:=strhex_a+'-'+strhex_b;
```

`This is the end. That's all.`

`cnbragon` `[iPB]` `[RCT]`

> 引用:

* * *

* *最初由 ijia 发布*

**请教一下楼主，用 Elgamal 加密算法加密时能象 RSA 一样用私有密匙加密，然后用公共密匙来解密吗？**

不可以的。

对于 `RSA`，有

`ed= 1 mod (p-1)(q-1)`

`e` 和 `d` 是有对称性的，而且加解密用的公式类似 `m=c^e mod n` 和 `c=m^d mod n`，所以可以这么用。

但是对于 `ElGamal` 的加密：

`a=g^k mod p`

`b=y^kM mod p`

解密是:

`M=b/a^x mod p`

先不看 `mod` 运算，则

`M=b/a^x=(y^kM)/((g^k)^x)=(y^kM)/((g^x)^k)=(g^kM)/(y^k)=M`

如果把 `x` 和 `y` 简单的互换的话是恢复不了正确的信息的。

`4dd466d0e1e0cc123afbe504da2462ca`
