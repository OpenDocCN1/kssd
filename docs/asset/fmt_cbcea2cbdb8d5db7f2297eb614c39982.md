
## 注册方式
序列号+`KeyFile`

## 保护方式
花指令,自校检,进程检测,`API` 断点检测

## 加壳方式
`UPX v0.94-1.90`

## 加密算法
`MD5`+`AES-256`(Rijndael)+`ZLib`+`RC4`

## 软件限制
功能限制

## 破解声明
初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

----------------------------------------------------------------

## 破解分析

有关该软件中的 `AES-256`(Rijndael)和 `ZLib` 算法的分析，请参见上文。

本文主要分析该软件中采用的作者自定义算法和 `RC4` 算法。

因为因为本文算法较少，再加上我也是初涉密码学，所以尽可能写详细一点，自己分析起来方便，大家看着也方便，呵呵。

在继续闯关之前，先简单介绍一下 `RC4` 算法：

`RC4` 加密算法是大名鼎鼎的 `RSA` 三人组中的头号人物 `Ron Rivest` 在 1987 年设计的密钥长度可变的流加密算法簇。之所以称其为簇，是由于其核心部分的 `S-Box` 长度可为任意，但一般为 256 字节。该算法的速度可以达到 `DES` 加密的 10 倍左右。

在开始的 7 年中该算法享有专利，直到 1994 年，有人匿名发布了它的源码，此后它就不再是一个商业秘密了。

`RC4` 属于对称算法中的序列密码，按字节逐个对明文加密。

（以下代码均用类 `C` 语言描述）

先看 `S-Box` 的初始化，共分为 2 步：

1. 线性填充 `S-Box`(`S[i]`)。代码如下：

```c
for(i=0;i<256;i++)
  S[i]=i;
```

2. 根据密钥(`K[i]`)扰乱 `S-Box`。代码如下：

```c
for(i=j=0;i<256;i++)
{
  j=(j+S[i]+K[i])%256;
  交换 S[i]和 S[j];
}
```

`RC4` 的加密/解密也很简单，也分为 2 步：

1. 产生随机字节 `k`，代码如下：

```c
i=j=0;
while(明文未结束)
{
  i=(i+1)%256;
  j=(j+s[i])%256;
  交换 S[i]和 S[j];
  t=(S[i]+S[j])%256;
  k=S[t];
}
```

2. 将字节 `k` 与明文异或。

因为是异或运算，所以加密/解密算法相同。

下面言归正传，继续闯关。

### 第 11 关：检测 KeyFile 黑名单

话说 `Ptero` 闯过 10 关后，发现程序注册菜单消失，自以为注册成功，正沾沾自喜中，忽又发现被程序作者骗了，制作出的 `chm` 还是显示未注册版本。

于是又打开 `OD`、`Dede`、`IDA` 等等，继续分析，终于找到了隐藏的 11 关入口：

```asm
00503DF3    8D95 58FEFFFF  lea edx, [ebp-1A8]
00503DF9    8B45 F0        mov eax, [ebp-10]    ; KeyFile 文件名
00503DFC    E8 0763FCFF    call ****CHM.004CA108  ; 计算 KeyFile 文件的 MD5 值
00503E01    8B85 58FEFFFF  mov eax, [ebp-1A8]    ; MD5 值
00503E07    8B55 EC        mov edx, [ebp-14]
00503E0A    E8 25B5FCFF    call ****CHM.004CF334
00503E0F    84C0           test al, al
00503E11    0F84 3F010000  je ****CHM.00503F56    ; 这里跳的话的 Game Over 啦
```

`004CF334` 这个 `Call`，读取了 3 个 `MD5` 值，与 `KeyFile` 的 `MD5` 比较。估计是检测流传出去的 `KeyFile` 的黑名单吧。如果相等的话，嘿嘿，就删除 `KeyFile`，删除注册信息。

我自己构造了一个 `KeyFile`，肯定不在黑名单里啦。所以直接 `pass`，呵呵。

之后来到这里：

```asm
00503FBC    8B85 64FDFFFF  mov eax, [ebp-29C]    ; 用户名
00503FC2    8B4D F0        mov ecx, [ebp-10]    ; KeyFile 文件名
00503FC5    5A             pop edx
00503FC6    E8 BD65FDFF    call ****CHM.004DA588  ; 解密 KeyFile，详细算法参见第 7、8、9 关
00503FCB    8B85 70FDFFFF  mov eax, [ebp-290]
00503FD1    50             push eax      ; KeyFile 解密后的字串
00503FD2    8D85 58FDFFFF  lea eax, [ebp-2A8]
00503FD8    8D95 F3FEFFFF  lea edx, [ebp-10D]
00503FDE    E8 390FF0FF    call ****CHM.00404F1C  ; System.@LStrFromString(String;String;ShortString;ShortString);
00503FE3    8B85 58FDFFFF  mov eax, [ebp-2A8]    ; 加密后的注册码
00503FE9    8D95 5CFDFFFF  lea edx, [ebp-2A4]
00503FEF    E8 2C24F3FF    call ****CHM.00436420  ; 解密
00503FF4    8B85 5CFDFFFF  mov eax, [ebp-2A4]    ; 注册码
00503FFA    50             push eax
00503FFB    8D85 50FDFFFF  lea eax, [ebp-2B0]
00504001    8D95 8EFEFFFF  lea edx, [ebp-172]
00504007    E8 100FF0FF    call ****CHM.00404F1C  ; System.@LStrFromString(String;String;ShortString;ShortString);
0050400C    8B85 50FDFFFF  mov eax, [ebp-2B0]    ; 加密后的用户名
00504012    8D95 54FDFFFF  lea edx, [ebp-2AC]
00504018    E8 0324F3FF    call ****CHM.00436420  ; 解密
0050401D    8B95 54FDFFFF  mov edx, [ebp-2AC]    ; 用户名
00504023    59             pop ecx      ; 注册码
00504024    58             pop eax      ; KeyFile 解密后的字串
00504025    E8 1667FDFF    call ****CHM.004DA740  ; 12、13 关
0050402A    84C0           test al, al
0050402C    74 10          je short ****CHM.0050403E
0050402E    EB 06          jmp short ****CHM.00504036
```

`004DA740` 就是关键 `Call` 啦，`F7` 跟进：

### 第 12 关：KeyFile 的压缩变换

取 `KeyFile` 的第 110 位，记为 `a`，并转换成数字：

```asm
004DA7E2    8D45 D8        lea eax, [ebp-28]
004DA7E5    50             push eax
004DA7E6    B9 01000000    mov ecx, 1
004DA7EB    BA 6E000000    mov edx, 6E
004DA7F0    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA7F3    E8 FCA9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
004DA7F8    8B45 D8        mov eax, [ebp-28]
004DA7FB    E8 E0F6F2FF    call ****CHM.00409EE0  ; SysUtils.StrToInt(AnsiString):Integer;
004DA800    8BD8           mov ebx, eax
004DA802    EB 04          jmp short ****CHM.004DA808
```

取 `KeyFile` 的第 1-109 位：

```asm
004DA83A    8D45 E0        lea eax, [ebp-20]
004DA83D    50             push eax
004DA83E    B9 6D000000    mov ecx, 6D
004DA843    BA 01000000    mov edx, 1
004DA848    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA84B    E8 A4A9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
004DA850    EB 04          jmp short ****CHM.004DA856
```

取 `KeyFile` 的(`a+130`)-(`a+143`)位：

```asm
004DA86A    8D45 DC        lea eax, [ebp-24]
004DA86D    50             push eax
004DA86E    8D93 8D000000  lea edx, [ebx+8D]
004DA874    B9 04000000    mov ecx, 4
004DA879    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA87C    E8 73A9F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
```

转换成 10 进制，记为 `b`：

```asm
004DA8B9    33D2           xor edx, edx
004DA8BB    8B45 DC        mov eax, [ebp-24]
004DA8BE    E8 59F6F2FF    call ****CHM.00409F1C  ; SysUtils.StrToIntDef(AnsiString;Integer):Integer;
004DA8C3    8BF0           mov esi, eax
```

`b` 是下面字符串的长度。接着取紧随其后的 `b` 个字符，记为 `Str1`：

```asm
004DA8E1    8D45 E4        lea eax, [ebp-1C]
004DA8E4    50             push eax
004DA8E5    8D93 91000000  lea edx, [ebx+91]    ; a+144
004DA8EB    8BCE           mov ecx, esi      ; b
004DA8ED    8B45 FC        mov eax, [ebp-4]    ; KeyFile 解密后的字串
004DA8F0    E8 FFA8F2FF    call ****CHM.004051F4  ; System.@LStrCopy;
```

计算 `KeyFile` 的第 1-109 位的 `Hash` 值：

```asm
004DA94D    8D4D D0        lea ecx, [ebp-30]
004DA950    33D2           xor edx, edx
004DA952    8B45 E0        mov eax, [ebp-20]    ; 字符串前 110 位
004DA955    E8 EE56FFFF    call ****CHM.004D0048  ; 自定义的 Hash 算法，详细算法参见第 6 关
004DA95A    8B45 D0        mov eax, [ebp-30]    ; KeyFile 的第 1-109 位的 Hash 字符串
004DA95D    50             push eax
```

注意：这里计算出了一个 `Hash` 字串，下一关要用到。

下面是本关的重头戏，压缩变换：

```asm
004DA95E    8D55 CC        lea edx, [ebp-34]
004DA961    8B45 E4        mov eax, [ebp-1C]    ; 取出的 b 个字节的字符串 Str1
004DA964    E8 A74CFFFF    call ****CHM.004CF610  ; 进行压缩变换，F7 跟进可看到算法
```

`F7` 跟进：

---------------------------------------------------------------- `004CF610` ----------------------------------------------------------------

```asm
004CF610    55             push ebp
…………
…………
…………
004CF64A    8D55 F4        lea edx, [ebp-C]
004CF64D    8B45 FC        mov eax, [ebp-4]
004CF650    E8 D3000000    call ****CHM.004CF728  ; 3 字节→1 字节压缩变换并逆序，F7 跟进可看到算法
```

下面是一个循环：

```asm
004CF680    46             inc esi      ; 变换后字符串长度，作循环计数器
004CF681    33FF           xor edi, edi
004CF683    8B45 F4        mov eax, [ebp-C]    ; 变换后的字符串
004CF686    8A5C38 FF      mov bl, [eax+edi-1]
004CF68A    8B45 F4        mov eax, [ebp-C]
004CF68D    E8 0259F3FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;
004CF692    2BC7           sub eax, edi
004CF694    8B55 F4        mov edx, [ebp-C]
004CF697    8A0402         mov al, [edx+eax]
004CF69A    50             push eax
004CF69B    8D45 F4        lea eax, [ebp-C]
004CF69E    E8 495BF3FF    call ****CHM.004051EC
004CF6A3    5A             pop edx
004CF6A4    885438 FF      mov [eax+edi-1], dl
004CF6A8    8B45 F4        mov eax, [ebp-C]
004CF6AB    E8 E458F3FF    call ****CHM.00404F94  ; System.@LStrLen(String):Integer;
004CF6B0    2BC7           sub eax, edi
004CF6B2    50             push eax
004CF6B3    8D45 F4        lea eax, [ebp-C]
004CF6B6    E8 315BF3FF    call ****CHM.004051EC
004CF6BB    5A             pop edx
```
