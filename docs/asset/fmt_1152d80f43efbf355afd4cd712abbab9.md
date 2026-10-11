
`111100 111011 111010 111001 111000 110111 110110 110101 110101 110110 110111 111000 111001 111010 111011 111100 111100 111011`

这样注册码前半部分 `876543211234567887` 转换后为：

堆栈 `ss:\[0012F924\]=00FC7E70`, `(ASCII "111100111011111010111001111000110111110110110101110101110110110111111000111001111010111011111100111100111011")`

分组，`8bit` 为 1 组，转换成十六进制：

```
004D9063     8D85 F0FBFFFF      lea eax,dword ptr ss:\[ebp-410\]

004D9069     50                 push eax

004D906A     B9 08000000        mov ecx,8

004D906F     BA 01000000        mov edx,1

004D9074     8B45 F8            mov eax,dword ptr ss:\[ebp-8\]

004D9077     E8 8CB8F2FF        call VideoCap.00404908                 ; 取 8bit

004D907C     8B95 F0FBFFFF      mov edx,dword ptr ss:\[ebp-410\]

004D9082     8D45 F7            lea eax,dword ptr ss:\[ebp-9\]

004D9085     E8 2AFAFFFF        call VideoCap.004D8AB4                 ; 转换成十六进制

004D908A     8D85 ECFBFFFF      lea eax,dword ptr ss:\[ebp-414\]

004D9090     8A55 F7            mov dl,byte ptr ss:\[ebp-9\]           ; 值

004D9093     E8 1CB5F2FF        call VideoCap.004045B4

004D9098     8B95 ECFBFFFF      mov edx,dword ptr ss:\[ebp-414\]

004D909E     8BC7               mov eax,edi

004D90A0     E8 0BB6F2FF        call VideoCap.004046B0                 ; 连接

004D90A5     8D45 F8            lea eax,dword ptr ss:\[ebp-8\]

004D90A8     B9 08000000        mov ecx,8

004D90AD     BA 01000000        mov edx,1

004D90B2     E8 91B8F2FF        call VideoCap.00404948

004D90B7     4B                 dec ebx

004D90B8   ^ 75 A9              jnz short VideoCap.004D9063
```

`11110011 10111110 10111001 11100011 01111101 10110101 11010111 01101101 11111000 11100111 10101110 11111100 11110011 1011`

后面 4bit 舍弃，得到：

`00FC7EDC  F3 BE B9 E3 7D B5 D7 6D F8 E7 AE FC F3           缶广}底 m  ??`

再转换成 2 进制（`HR 00FC7EDC`）：

```
004D9163     8BC6                 mov eax,esi

004D9165     8B55 FC              mov edx,dword ptr ss:\[ebp-4\]

004D9168     0FB6543A FF          movzx edx,byte ptr ds:\[edx+edi-1\]

004D916D     8B9495 FCFBFFFF      mov edx,dword ptr ss:\[ebp+edx*4-404\]   ; 转换成 2 进制，断在此

004D9174     E8 37B5F2FF          call VideoCap.004046B0

004D9179     47                   inc edi

004D917A     4B                   dec ebx

004D917B   ^ 75 E6               jnz short VideoCap.004D9163
```

堆栈 `ss:\[0012F8CC\]=00FC7B18`, `(ASCII "11110011101111101011100111100011011111011011010111010111011011011111100011100111101011101111110011110011")`

分组，后面 `63（H）＝99（D）` 位为 1 组，前面剩下的为 1 组：

`M1=11110=1E(H)`

`M2=011101111101011100111100011011111011011010111010111011011011111100011100111101011101111110011110011`

＝`3BEB9E37DB5D76DF8E7AEFCF3(H)`

因为注册码前部分为 18 位，×6＝108bit，舍弃后 4bit＝104bit，分成 2 组的话，第 1 组只能 5bit，第 2 组 99bit。

加密：

```
004DBD3E     8D45 F0              lea eax,dword ptr ss:\[ebp-10\]

004DBD41     E8 DED8FFFF          call VideoCap.004D9624

004DBD46     8D45 AC              lea eax,dword ptr ss:\[ebp-54\]

004DBD49     E8 7E86F2FF          call VideoCap.004043CC

004DBD4E     8D55 AC              lea edx,dword ptr ss:\[ebp-54\]

004DBD51     8D45 E8              lea eax,dword ptr ss:\[ebp-18\]

004DBD54     E8 03E5FFFF          call VideoCap.004DA25C

004DBD59     EB 10               jmp short VideoCap.004DBD6B

004DBD6B     8B45 AC              mov eax,dword ptr ss:\[ebp-54\]         ; 加密值
```

第 1 组加密值为：

堆栈 `ss:\[0012F8C4\]=00FC383C`, `(ASCII "100110100000010101000101001000011111000000011100010010111010000110100101000111010100001010001011000")`

即：`4D02A290F80E25D0D28EA1458`

然后前面补 0，第 1 组补不补都一样。

验证一下：

`N=729181748E63BE5BA59406B8F`

`E=68FF4D2042822BBAF2449D1B1`

`M=1E`

`C=4D02A290F80E25D0D28EA1458`

的确如此，看来这次加密只是把 E 换了而已，挺大的。

第 2 组加密值为：

堆栈 `ss:\[0012F8C4\]=00FC4FE4`, `(ASCII "101100010011101110100000111000100110111001011010101010101001110001001101110110011110101011100111")`

即：`B13BA0E26E5AAA9C4DD9EAE7`

补 0 到 98 的倍数（前面添了 2 个 0）：

堆栈 `ss:\[0012F8C4\]=00FC4FE4`, `(ASCII "00101100010011101110100000111000100110111001011010101010101001110001001101110110011110101011100111")`

注意：第 1 组前面补不补 0 没关系，但是第 2 组加密值一定要补，不补的话会影响到下面连接值

2 组加密值的 2 进制流连接起来：

堆栈 `ss:\[0012F8C8\]=00FC7C68`, `(ASCII "000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010011010000001010100010100100001111100000001110001001011101000011010010100011101010000101000101100000101100010011101110100000111000100110111001011010101010101001110001001101110110011110101011100111".)`

即：`1340A8A43E03897434A3A85160B13BA0E26E5AAA9C4DD9EAE7`

找第 1 个 111，找到 111 后取后面的值（十进制 163bit）：

```
004DBDB1     8D45 B0              lea eax,dword ptr ss:\[ebp-50\]

004DBDB4     B9 01000000          mov ecx,1

004DBDB9     BA 01000000          mov edx,1

004DBDBE     E8 858BF2FF          call VideoCap.00404948

004DBDC3     8D45 A0              lea eax,dword ptr ss:\[ebp-60\]

004DBDC6     50                   push eax

004DBDC7     B9 03000000          mov ecx,3

004DBDCC     BA 01000000          mov edx,1

004DBDD1     8B45 B0              mov eax,dword ptr ss:\[ebp-50\]

004DBDD4     E8 2F8BF2FF          call VideoCap.00404908

004DBDD9     8B45 A0              mov eax,dword ptr ss:\[ebp-60\]

004DBDDC     BA 74BE4D00          mov edx,VideoCap.004DBE74             ; ASCII "111"

004DBDE1     E8 0E8AF2FF          call VideoCap.004047F4

004DBDE6     74 0D                je short VideoCap.004DBDF5

004DBDE8     8B45 B0              mov eax,dword ptr ss:\[ebp-50\]

004DBDEB     E8 B888F2FF          call VideoCap.004046A8

004DBDF0     83F8 03              cmp eax,3

004DBDF3   ^ 7F BC               jg short VideoCap.004DBDB1

004DBDF8     B9 03000000          mov ecx,3

004DBDFD     BA 01000000          mov edx,1

004DBE02     E8 418BF2FF          call VideoCap.00404948                 ; 取后面的值

004DBE07     8B55 08              mov edx,dword ptr ss:\[ebp+8\]
```

加密值（补了 n 个 0）为：

堆栈 `ss:\[0012F8C8\]=00FC7C68`, `(ASCII "000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000010011010000001010100010100100001111100000001110001001011101000011010010100011101010000101000101100000101100010011101110100000111000100110111001011010101010101001110001001101110110011110101011100111".)`

即：`1340A8A43E03897434A3A85160B13BA0E26E5AAA9C4DD9EAE7`

111 后面的值(163BIT)：

`1100000001110001001011101000011010010100011101010000101000101100000101100010011101110100000111000100110111001011010101010101001110001001101110110011110101011100111`

即：`603897434A3A85160B13BA0E26E5AAA9C4DD9EAE7（H）`

然后 8bit 为 1 组，转换成十六进制，不够的前面补 0：

```
004DBE0A     8B45 B0              mov eax,dword ptr ss:\[ebp-50\]

004DBE0D     E8 9ED3FFFF          call VideoCap.004D91B0

004DBE12     8D45 B8              lea eax,dword ptr ss:\[ebp-48\]

004DBE15     E8 0AD8FFFF          call VideoCap.004D9624
```

163/8，余数 3，前面补 5 个 0：

`00000110 00000011 10001001 01110100 00110100 10100011 10101000 01010001 01100000 10110001 00111011 10100000 11100010 01101110 01011010 10101010 10011100 01001101 11011001 11101010 11100111`

转换后的值：

`00FC38E4  06 03 89 74 34 A3 A8 51 60 B1 3B A0 E2 6E 5A AA   塼 4（Q\_?_ 鈔 Z`

`00FC38F4  9C 4D D9 EA E7 00                                   淢訇?..(`

返回到：

```
004DC059     E8 22FAFFFF          call VideoCap.004DBA80

004DC05E     8B45 08              mov eax,dword ptr ss:\[ebp+8\]

004DC061     8B55 F0              mov edx,dword ptr ss:\[ebp-10\]         ; 值？
```

再返回就完成第 1 组注册码的加密运算过程。

#### 6.对最后加密运算值的运算

```
004DC354     8B45 CC              mov eax,dword ptr ss:\[ebp-34\]         ; 运算值

004DC357     8D55 D0              lea edx,dword ptr ss:\[ebp-30\]

004DC35A     E8 85040000          call VideoCap.004DC7E4                 ; 取出数字，并连接

004DC35F     8B45 D0              mov eax,dword ptr ss:\[ebp-30\]         ; 值

004DC362     E8 C1CDF2FF          call VideoCap.00409128                 ; 转换成十六进制

004DC367     8945 F0              mov dword ptr ss:\[ebp-10\],eax        ; 保存
```

`HR EAX:`
