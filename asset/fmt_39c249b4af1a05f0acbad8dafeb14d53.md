
\---------------------------------------- 004D9184 ----------------------------------------

004D9184    55           push  ebp

……

004D91AD    C645 F7 00   mov byte ptr [ebp-9], 0

004D91B1    33C0         xor eax, eax

004D91B3    8945 F8      mov [ebp-8], eax

004D91B6    BB 01000000  mov ebx, 1

004D91BB    33F6         xor esi, esi

004D91BD    EB 5A        jmp short ****CHM.004D9219

循环，压缩变换：

004D91BF    85F6         test esi, esi

004D91C1    75 18        jnz short ****CHM.004D91DB

004D91C3    33C0         xor eax, eax

004D91C5    8A441F FF    mov al, [edi+ebx-1]

004D91C9    8A80 BCAA5500 mov al, [eax+55AABC]    ; 查表替换

004D91CF    8845 F7      mov [ebp-9], al

004D91D2    C745 F8 020000> mov dword ptr [ebp-8], 2

004D91D9    EB 39        jmp short ****CHM.004D9214

004D91DB    8D45 F0      lea eax, [ebp-10]

004D91DE    8B4D F8      mov ecx, [ebp-8]

004D91E1    33D2         xor edx, edx

004D91E3    8A55 F7      mov dl, [ebp-9]

004D91E6    D3E2         shl edx, cl

004D91E8    81E2 C0000000 and edx, 0C0

004D91EE    33C9         xor ecx, ecx

004D91F0    8A4C1F FF    mov cl, [edi+ebx-1]

004D91F4    0FB689 BCAA550> movzx ecx, byte ptr [ecx+55AABC]  ; 查表替换

004D91FB    0BD1         or edx, ecx

004D91FD    E8 9EBCF2FF  call ****CHM.00404EA0  ; System.@LStrFromChar(String;String;Char);

004D9202    8B55 F0      mov edx, [ebp-10]

004D9205    8B45 FC      mov eax, [ebp-4]

004D9208    E8 8FBDF2FF  call ****CHM.00404F9C  ; System.@LStrCat;

004D920D    8B45 FC      mov eax, [ebp-4]

004D9210    8345 F8 02   add dword ptr [ebp-8], 2

004D9214    46           inc esi

004D9215    83E6 03      and esi, 3

004D9218    43           inc ebx

004D9219    8BC7         mov eax, edi

004D921B    E8 74BDF2FF  call ****CHM.00404F94  ; System.@LStrLen(String):Integer;

004D9220    3BD8         cmp ebx, eax

004D9222    7F 07        jg short ****CHM.004D922B

004D9224    807C1F FF 2E  cmp byte ptr [edi+ebx-1], 2E

004D9229  ^ 75 94        jnz short ****CHM.004D91BF

以下就返回了……

上面这个循环很让人头痛，不知道是作者从哪里找来的算法。我是在 IDA 当中分析的。

这是所查的表，姑且也算一个 `S-Box` 吧，设为 `S[i]`。

表中有效数值从 0-39h：

0055AABC  80 80 80 80 80 80 80 80 80 80 80 80 80 80 80 80  €€€€€€€€€€€€€€€€

0055AACC  80 80 80 80 80 80 80 80 80 80 80 80 80 80 80 80  €€€€€€€€€€€€€€€€

0055AADC  80 80 80 3A 80 3B 3C 80 80 80 3D 3E 80 3F 80 80  €€€:€;<€€€=>€?€€

0055AAEC  80 80 00 01 02 03 04 05 06 07 80 80 80 80 80 80  €€.€€€€€€

0055AAFC  80 08 09 0A 0B 0C 0D 0E 0F 80 10 11 12 13 14 80  €.. ..€€

0055AB0C  15 16 17 18 19 1A 1B 1C 1D 1E 1F 80 80 80 80 80  €€€€€

0055AB1C  80 20 21 22 23 24 25 26 27 28 29 2A 80 2B 2C 2D  € !"#$%&'()*€+,-

0055AB2C  2E 2F 30 31 32 33 34 35 36 37 38 80 80 80 39 80  ./012345678€€€9€

下面将其算法用 C 语言给出：

```c
for(i=0;i<=sizeof(str)&&strIn[i]!=0x2E;i++)
{
  if(!(i%4))
  {
    colume=2;
    key=S[strIn[i]];
  }
  else
  {
    strOut[i-1]=((key<<colume)&0xC0)|S[strIn[i]];
    colume+=2;
  }
}
```

以上算法以 4 字节为一组，第 1 字节为子密钥，将其后的 3 字节解密，如果遇到 2Eh，则停止解密。

下面介绍一下该算法原理。

看算法中的 Magic Number：`0C0h`，为什么要用它呢？

将它转换成 2 进制看看：`1100 0000`。

它实际上就是一个掩码。

子密钥的 15、14 位为 0，0-13 位为下面 3 字节密文的 15、14 位的掩码。

密文的 15、14 位由子密钥和掩码决定，0-13 位由查表得出。

因为子密钥和密文字节都只用到 0-13 位，也就是 0-39h，这就解释了上面 `S-Box` 有效值的范围。

至于加密算法嘛，先将明文按 3 字节分组，然后将 3 个密文字节的 15、14 位组合成密钥，再换成在 `S-Box` 中的位置。密文的 0-13 位也分别换成在 `S-Box` 中的位置。

\--------------------------------------------------------------------------------

### 第 15 关：RC4 解密

既然密文、密钥都有了，下面就是解密啦：

004D90B3    8D95 F2FDFFFF  lea edx, [ebp-20E]    ; RC4 密钥

004D90B9    8D85 F2FBFFFF  lea eax, [ebp-40E]

004D90BF    B9 FF000000    mov ecx, 0FF

004D90C4    E8 67FCFFFF    call ****CHM.004D8D30  ; S-Box 初始化，F7 跟进可看到算法

004D90C9    68 FF000000    push 0FF     ; 密钥长度

004D90CE    8D8D F1FEFFFF  lea ecx, [ebp-10F]    ; 解密后明文地址，覆盖掉密文

004D90D4    8D95 F1FEFFFF  lea edx, [ebp-10F]    ; 上一关解密后字串作明文

004D90DA    8D85 F2FBFFFF  lea eax, [ebp-40E]    ; S-Box

004D90E0    E8 1FFDFFFF    call ****CHM.004D8E04  ; RC4 加密

`RC4` 加密的代码比较长，我就不贴了。其实算法还是很简单的，见本文开头的说明。

下面是对 `S-Box` 的初始化：

\---------------------------------------- 004D8D30 ----------------------------------------

004D8D30    53           push  ebx

004D8D31    56           push  esi

004D8D32    57           push  edi

004D8D33    55           push  ebp

004D8D34    81C4 FC FE FF FF add esp, -104

004D8D3A    8BEA         mov ebp, edx

004D8D3C    890424       mov [esp], eax

004D8D3F    85C9         test ecx, ecx

004D8D41    7E 08        jle short ****CHM.004D8D4B

004D8D43    81F9 00010000 cmp ecx, 100      ; 密钥长度不能大于 256 位，否则就产生异常啦

004D8D49    7E 16        jle short ****CHM.004D8D61

004D8D4B    B9 E08D4D00  mov ecx, ****CHM.004D8DE0    ; ASCII "Invalid key length"

004D8D50    B2 01        mov dl, 1

004D8D52    A1 3C884000  mov eax, [40883C]

004D8D57    E8 0C52F3FF  call ****CHM.0040DF68  ; AxCtrls.TOleStream.Create(TOleStream;boolean;IStream);

004D8D5C    E8 ABB8F2FF  call ****CHM.0040460C  ; System.@RaiseExcept;

下面的循环对 `S-Box` 进行线性填充，生成 0-0FFh 的数列：

004D8D61    33DB         xor ebx, ebx

004D8D63    8B3424       mov esi, [esp]

004D8D66    8D7C24 04    lea edi, [esp+4]

004D8D6A    881E         mov [esi], bl

004D8D6C    8BC3         mov eax, ebx

004D8D6E    99           cdq

004D8D6F    F7F9         idiv ecx

004D8D71    03D5         add edx, ebp

004D8D73    8A02         mov al, [edx]

004D8D75    8807         mov [edi], al

004D8D77    43           inc ebx

004D8D78    47           inc edi

004D8D79    46           inc esi

004D8D7A    81FB 00010000 cmp ebx, 100

004D8D80  ^ 75 E8        jnz short ****CHM.004D8D6A

下面的循环根据密钥(`K[i]`)扰乱 `S-Box`：

004D8D82    33F6         xor esi, esi

004D8D84    BB 00010000  mov ebx, 100

004D8D89    8B0424       mov eax, [esp]    ; S-Box，S[i]

004D8D8C    8D7C24 04    lea edi, [esp+4]    ; 密钥，K[i]

004D8D90    8A10         mov dl, [eax]

004D8D92    33C9         xor ecx, ecx

004D8D94    8ACA         mov cl, dl

004D8D96    03F1         add esi, ecx

004D8D98    33C9         xor ecx, ecx

004D8D9A    8A0F         mov cl, [edi]

004D8D9C    03F1         add esi, ecx

004D8D9E    81E6 FF000000 and esi, 0FF

004D8DA4    8B0C24       mov ecx, [esp]

004D8DA7    8A0C31       mov cl, [ecx+esi]

004D8DAA    8808         mov [eax], cl

004D8DAC    8B0C24       mov ecx, [esp]

004D8DAF    881431       mov [ecx+esi], dl

004D8DB2    47           inc edi

004D8DB3    40           inc eax

004D8DB4    4B           dec ebx

004D8DB5  ^ 75 D9        jnz short ****CHM.004D8D90

以下就返回了……

\--------------------------------------------------------------------------------

### 第 16 关：数字和的验证

还记得 12 关中从 `KeyFile` 读取的那个数字 `b` 吗？它是 `Str1` 的长度。

下面的代码读取 `KeyFile` 的(b+244)-(b+247)字节：

004DA9A2    8D45 E8      lea eax, [ebp-18]

004DA9A5    50           push eax

004DA9A6    8D941E F500000> lea edx, [esi+ebx+F5]

004DA9AD    03D3         add edx, ebx

004DA9AF    B9 04000000  mov ecx, 4

004DA9B4    8B45 FC      mov eax, [ebp-4]    ; KeyFile 解密后的字串

004DA9B7    E8 38A8F2FF  call ****CHM.004051F4  ; System.@LStrCopy;

转换成 10 进制，记为 `c`：

004DA9F4    33D2         xor edx, edx

004DA9F6    8B45 E8      mov eax, [ebp-18]
