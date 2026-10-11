
# 及时雨 PK 版 7.28

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-254.htm`](https://www.pediy.com/kssd/pediy08/pediy8-254.htm)

及时雨 PK 版 7.28 服务器验证分析过程(现在他们的验证方式早就已经修改了希望不会影响到什么)

外挂一共有 3 次验证其中 2 次是在 `*.exe` 文件中,1 次是在按 F12 呼出外挂的时候进行的!

前 2 次验证就是利用外挂本身的加密解密算法进行验证的!第 3 次验证分为 2 个部分;

---

## 第一次验证

`send1`(入栈长度 3) 实践写入效验数据的位置是 2

```
0012F3F8   0012F410  |Arg1 = 0012F410 
0012F3FC   00000003  |Arg2 = 00000003 
0012F400   0000002E  |Arg3 = 0000002E 
0012F404   000000D9  \Arg4 = 000000D9 
```

加密函数的入栈参数(长度为 3) 加上一个效验和数据长度正好是 4

`0012F410 13 32 77 77 2ww`

返回一个固定的数据包,但是必须通过解密算法的效验,由于他对发送数据用到的加密算法和对返回数据用到的解密算法都在外挂内存中所以可以拿来直接使用.

返回的数据包

```
0012F410  51 C3 0E B0 FC 13 FD 46 3F 54 E8 E2 C4 E2 B1 73  Q?包鼺?T 桠拟眘
0012F420  04 1B C8 4D 57 3F CB 0A F8 5B 9D 1A 00 79 37 5F  萂 W??鳾?.y7_
0012F430  1E D8 EF B2 0C 4B 87 9E 0C F8 81 C3 C1 28 77 D9  仫?K 嚍.鴣昧(w
0012F440  D4 FD DD C4 DF AC 42 21 4E 3C 8A 9B 88 EC BA 2D  札菽攥 B!N<姏堨?
0012F450  A2 B1 E4 63 FE A7 FF 81 90 93 E9 15 0B 8C AB BC  ⒈鋍亹撻尗
0012F460  35 21 6C AD 42 8F 60 04 7F 7A F9 84 DB 05 BF 1F  5!l 瑽廯z 鶆??
0012F470  9C C3 3D 59 29 33 B0 86 4D A2 78 06 C7 43 F8 1D  溍=Y)3 皢 M荂?
0012F480  6C 75 65 EE 08 AA 40 51 F6 C4 35 9D 3C D2 D4 9A  lue?狜 Q 瞿 5?
                                 ^^^^^^^^^^^
```

上面的数据包进行解密运算得到下面的数据:

```
0012F410  55 4F 4B 00 00 00 00 00 00 00 00 00 00 00 00 00  UOK.............
0012F420  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F430  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F440  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F450  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F460  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F470  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F480  00 00 00 00 00 00 00 00 00 00 00 25 C3 00 00 11  ...........%?.
                                   ^^这个数据是效验和的数值
```

是根据数据包前面的数据累加得到的!并且用到了加密算法的 2 个数据:`$2E` 和 `$D9`

外挂是对返回的数据解密之后读取第 `[ESI+1]` 和 `[ESI+2]` 位比较是不是 `'OK'` 如果是 `"登陆"` 按钮就变成可用;

---

## 第二次验证

`send2`(入栈长度 3F) 实践写入效验数据的位置是 3E

```
0012F494   0012F4BC  |Arg1 = 0012F4BC 
0012F498   0000003F  |Arg2 = 0000003F 
0012F49C   0000002E  |Arg3 = 0000002E 
0012F4A0   000000D9  \Arg4 = 000000D9 
```

外挂发送的数据包(加密之前)

```
0012F4BC  15 BC B0 CA B1 D3 EA 00 00 00 00 00 18 F5 12 00  及时雨.....?.
0012F4CC  C4 C6 D1 77 38 F5 12 00 37 3B D1 77 00 E0 FD 7F  钠褀 8?.7;褀.帻z
0012F4DC  38 F5 12 00 67 3B D1 77 04 F5 12 00 4B 3B D1 77  8?.g;褀?.K;褀
0012F4EC  00 02 00 9A 02 00 00 77 D0 C2 16 00 60 F5 12 00  ..?..w 新.`?.
                                   ^^这里是效验值
```

加密之后

```
0012F4BC  11 18 51 84 ED FC 3F B1 EC 73 EB 53 01 8C EC AF  Q 勴?膘 s 隨岇
0012F4CC  70 DC 39 B0 96 E7 4B FE 96 3F 80 F2 55 40 AF 33  p?皷鏚䱷?€騏@?
0012F4DC  C8 93 FC 79 A7 D1 55 76 14 47 9A 32 F3 B5 07 32  葥鼀аUvG?蟮 2
0012F4EC  6F 23 54 38 57 41 AA 63 CE 83 47 D0 FE 1C 9E 00  o#T8WA 猚蝺 G 玄?
```

服务器返回的数据:

```
0012F4BC  04 B2 32 A8 1A BD DC 7E E6 05 20 8A C3 FC 0B A4  ??杰~? 娒?
0012F4CC  12 74 B6 D7 02 13 A4 F1 9C 1A ED 05 ED FA D2 E4  t 蹲ゑ??睑忆
0012F4DC  99 8E 2C AF 5D 21 6F 0E 2E 3C 0B C5 40 72 C8 68 檸,痌!o.<臔 r 萮
0012F4EC  D6 DA 2B 81 DF 79 4A 61 20 A3 A6 F7 DF F8 E4 8F  众+佭 yJa ＆鬟
0012F4FC  8F 72 44 53 30 59 59 51 68 5A 54 6F 35 50 53 59  弐 DS0YYQhZTo5PSY
0012F50C  6A 67 4D 2B 54 71 67 3D 3D 00 00 00 00 00 00 00  jgM+Tqg==.......
0012F51C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F52C  00 00 00 00 00 00 00 00 00 00 00 D8 DA 77 F2 00  ...........刳 w?
```

解密之后

```
0012F4BC  00 4F 4B 00 00 00 00 00 00 00 00 00 00 00 00 00  .OK.............
0012F4CC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F4DC  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F4EC  00 00 00 9A 02 00 00 77 D0 C2 16 00 60 F5 E4 8F  ...?..w 新.`蹁
0012F4FC  8F 2B 37 56 59 74 33 52 38 6F 44 66 76 4D 44 32  ?7VYt3R8oDfvMD2
0012F50C  4F 77 58 4E 77 75 41 3D 3D 00 00 00 00 00 00 00  OwXNwuA==.......
0012F51C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
0012F52C  00 00 00 00 00 00 00 00 00 00 00 D4 A9 76 EC 00  ...........冤 v?
```

外挂是对返回的数据解密之后读取第 `[ESI+1]` 和 `[ESI+2]` 位比较是不是 `'OK'`

### 解密算法

```
00401D50  /$  55          PUSH    EBP
00401D51  |.  8BEC        MOV     EBP,ESP
00401D53  |.  51          PUSH    ECX
00401D54  |.  53          PUSH    EBX
00401D55  |.  56          PUSH    ESI
00401D56  |.  BA BC040000 MOV     EDX,4BC
00401D5B  |.  33C9        XOR     ECX,ECX
00401D5D  |.  33C0        XOR     EAX,EAX
00401D5F  |.  8B75 08     MOV     ESI,[ARG.1]                         ;  jsyz.0041B1EB
00401D62  |>  8A040E      /MOV     AL,[ESI+ECX]
00401D65  |.  8ADE        |MOV     BL,DH
00401D67  |.  32D8        |XOR     BL,AL
00401D69  |.  66:0FB6C0   |MOVZX   AX,AL
00401D6D  |.  03C2        |ADD     EAX,EDX
00401D6F  |.  881C0E      |MOV     [ESI+ECX],BL
00401D72  |.  8D1440      |LEA     EDX,[EAX+EAX*2]
00401D75  |.  C1E2 03     |SHL     EDX,3
00401D78  |.  2BD0        |SUB     EDX,EAX
00401D7A  |.  C1E2 03     |SHL     EDX,3
00401D7D  |.  2BD0        |SUB     EDX,EAX
00401D7F  |.  8D1490      |LEA     EDX,[EAX+EDX*4]
00401D82  |.  C1E2 05     |SHL     EDX,5
00401D85  |.  2BD0        |SUB     EDX,EAX
00401D87  |.  81EA 96350000 |SUB    EDX,3596
00401D8D  |.  41          |INC     ECX
00401D8E  |.  3B4D 0C     |CMP     ECX,[ARG.2]
00401D91  |.^ 75 CF       \JNZ     SHORT 00401D62
00401D93  |.  8B55 0C     MOV     EDX,[ARG.2]
00401D96  |.  33C9        XOR     ECX,ECX
00401D98  |.  33C0        XOR     EAX,EAX
00401D9A  |.  4A          DEC     EDX
00401D9B  |.  8B75 08     MOV     ESI,[ARG.1]                         ;  jsyz.0041B1EB
00401D9E  |>  02040E      /ADD     AL,[ESI+ECX]
00401DA1  |.  41          |INC     ECX
00401DA2  |.  3BCA        |CMP     ECX,EDX
00401DA4  |.^ 75 F8       \JNZ     SHORT 00401D9E
00401DA6  |.  3245 10     XOR     AL,[EBP+10]  ;$2E
00401DA9  |.  8A55 14     MOV     DL,[EBP+14]  ;$D9
00401DAC  |.  F6EA        IMUL    DL
00401DAE  |.  38040E      CMP     [ESI+ECX],AL
00401DB1  |.  74 07       JE      SHORT 00401DBA
00401DB3  |.  B8 01000000 MOV     EAX,1
00401DB8  |.  EB 02       JMP     SHORT 00401DBC
00401DBA  |>  33C0        XOR     EAX,EAX
00401DBC  |>  8945 FC     MOV     [LOCAL.1],EAX
00401DBF  |.  8B45 FC     MOV     EAX,[LOCAL.1]                       ;  jsyz.0041B1EB
00401DC2  |.  5E          POP     ESI
00401DC3  |.  5B          POP     EBX
00401DC4  |.  8BE5        MOV     ESP,EBP
00401DC6  |.  5D          POP     EBP
00401DC7  \.  C3          RETN
```

### 加密算法

```
00401DD0  /$  55          PUSH    EBP
00401DD1  |.  8BEC        MOV     EBP,ESP
00401DD3  |.  53          PUSH    EBX
00401DD4  |.  56          PUSH    ESI
00401DD5  |.  8B55 0C     MOV     EDX,[ARG.2]
00401DD8  |.  33C9        XOR     ECX,ECX
00401DDA  |.  33C0        XOR     EAX,EAX
00401DDC  |.  4A          DEC     EDX
00401DDD  |.  8B75 08     MOV     ESI,[ARG.1]                         ;  jsyz.0041B1EB
00401DE0  |>  02040E      /ADD     AL,[ESI+ECX]
00401DE3  |.  41          |INC     ECX
00401DE4  |.  3BCA        |CMP     ECX,EDX
00401DE6  |.^ 75 F8       \JNZ     SHORT 00401DE0
00401DE8  |.  3245 10     XOR     AL,[EBP+10]  ;$2E
00401DEB  |.  8A55 14     MOV     DL,[EBP+14]  ;$D9
00401DEE  |.  F6EA        IMUL    DL
```



00401DF0   |.   88040E        MOV     [ESI+ECX],AL  

00401DF3   |.   BA BC040000   MOV     EDX,4BC  

00401DF8   |.   33C9          XOR     ECX,ECX  

00401DFA   |.   33C0          XOR     EAX,EAX  

00401DFC   |.   8B75 08       MOV     ESI,[ARG.1]                          ;   jsyz.0041B1EB  

00401DFF   |>   8A040E        /MOV    AL,[ESI+ECX]  

00401E02   |.   8ADE          |MOV    BL,DH  

00401E04   |.   32D8          |XOR    BL,AL  

00401E06   |.   0FB6C3        |MOVZX  EAX,BL  

00401E09   |.   03C2          |ADD    EAX,EDX  

00401E0B   |.   881C0E        |MOV    [ESI+ECX],BL  

00401E0E   |.   8D1440        |LEA    EDX,[EAX+EAX*2]  

00401E11   |.   C1E2 03       |SHL    EDX,3  

00401E14   |.   2BD0          |SUB    EDX,EAX  

00401E16   |.   C1E2 03       |SHL    EDX,3  

00401E19   |.   2BD0          |SUB    EDX,EAX  

00401E1B   |.   8D1490        |LEA    EDX,[EAX+EDX*4]  

00401E1E   |.   C1E2 05       |SHL    EDX,5  

00401E21   |.   2BD0          |SUB    EDX,EAX  

00401E23   |.   81EA 96350000 |SUB    EDX,3596  

00401E29   |.   41            |INC    ECX  

00401E2A   |.   3B4D 0C       |CMP    ECX,[ARG.2]  

00401E2D   |.^ 75 D0          \JNZ    SHORT 00401DFF  

00401E2F   |.   5E            POP    ESI  

00401E30   |.   5B            POP    EBX  

00401E31   |.   5D            POP    EBP  

00401E32   \.   C3            RETN  

======================================================================================

### 第三部分的验证

这部分的验证比较麻烦，前 2 次服务器返回的数据跟外挂发送的数据没有直接的联系，只是进行了封包的效验。

和可以直接模拟封包算法组成封包直接发送就可以搞定，而这次的验证服务器返回的数据其中 3 部分数据跟外挂发送给服务器的数据有直接的关系。

#### 发送部分的处理：

```
.text:100010D4             call    ds:TlsGetValue   ;取得一个数据
.text:100010DA             mov     [ebp+var_9F], eax;保存到要发送的封包缓冲区中
.text:100010E0             call    ds:GetTickCount  ;另外又取得一个数据
.text:100010E6             mov     [ebp+var_9B], eax;保存到要发送的封包缓冲区中
.text:100010EC             push    0                 ; flags
.text:100010EE             push    80h               ; len
.text:100010F3             lea     eax, [ebp+buf]
.text:100010F9             push    eax               ; buf
.text:100010FA             push    [ebp+s]           ; s
.text:100010FD             call    ds:send           ; Send data on a connected socket
.text:10001103             jmp     short loc_10001115
```

首先通过 `TlsGetValue` 获得一个数据保存在发送的封包中 `[ESI+$65]`。

然后通过 `GetTickCount` 获得一个数据保存在发送的封包中 `[ESI+$69]`。

这次发送的封包没有进行封包的加密运算。

#### 数据包示例 (Send)

```
1  10.66.6.71  218.22.13.17  128  Send

0000  8F 69 50 6A 6C 71 32 58 76 30 57 4B 71 46 2B 6B    .iPjlq2Xv0WKqF+k
0010  43 7A 58 6B 52 48 67 3D 3D 00 00 00 00 00 00 00    CzXkRHg==.......
0020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030  00 00 00 00 00 00 00 00 00 00 00 36 F8 4E E3 00    ...........6.N..
0040  00 00 00 00 58 02 00 00 20 03 00 00 58 02 00 00    ....X... ...X...
0050  20 03 00 00 00 00 00 00 00 00 00 00 97 A4 C5 77     ..............w
0060  00 00 00 00 20 8F 9C 0A 00 23 2F 0B 00 00 00 00    .... ....#/.....
                ^^^^^^^^^^^ ^^^^^^^^^^^
0070  00 00 00 00 00 00 00 00 90 BE 70 73 60 08 15 00    ..........ps`...

8F 9C 0A 00             23 2F 0B 00

TlsGetValue=000A9C8F    GetTickCount=000B2F23
```

```
.text:10001137             call    ds:recv        ; Receive data from a socket 返回函数
.text:1000113D             cmp     eax, 80h
.text:10001142             jnz     short loc_1000119F ; 不能跳改 JZ 比较数据包长度是不是$80
.text:10001144             jmp     short loc_10001149
.text:10001146 byte_10001146   db 4Fh, 4Bh, 0          ; DATA XREF: sub_1000111B+2Eo
.text:10001149 loc_10001149:                          ; CODE XREF: sub_1000111B+29j
.text:10001149             push    offset byte_10001146
.text:1000114E             lea     eax, [ebp-103h]
.text:10001154             push    eax
.text:10001155             call    sub_1001B55A
.text:1000115A             or      eax, eax
.text:1000115C

.text:1000115C loc_1000115C:                          ; DATA XREF: sub_1001DB4A+2Co
.text:1000115C             jnz     short loc_1000118D ;
.text:1000115E             mov     byte_1003D25F, 1
.text:10001165             lea     esi, [ebp+var_FB]
.text:1000116B             lea     edi, byte_1003D154;把数据都放到这里
.text:10001171             mov     ecx, 76h
.text:10001176             cld
.text:10001177             rep movsb
```

通过对 `1003D154` 数据区，数据的调用我们可以分析出返回的数据都有什么用途。

#### 返回之后对数据的处理

```
.text:10011221 sub_10011221    proc near                ; CODE XREF: DialogFunc+3Ap
.text:10011221                 push    esi
.text:10011222                 lea     esi, unk_1003D1AD
.text:10011228                 push    1               ; dwTlsIndex
.text:1001122A                 call    ds:TlsGetValue  ;取得跟发送数据相同的值
.text:10011230                 xor     eax, [esi]      ;XOR 返回的数据 1 DATA_1
.text:10011232                 xor     eax, [esi+4]    ;XOR 返回的数据 2 DATA_2
.text:10011235                 mov     ecx, 3Bh        ;最后这里得出固定的数值$999
.text:1001123A                 lea     esi, dword_1004087B
.text:10011240

.text:10011240 loc_10011240:                          ; CODE XREF: sub_10011221+24j
.text:10011240                 xor     [esi], eax      ;用来对外挂的数据进行解密运算
.text:10011242                 add     esi, 4
.text:10011245                 loop    loc_10011240
.text:10011247                 pop     esi
.text:10011248                 retn

.text:10011248 sub_10011221    endp
```

返回的封包开始的地方仍然是 `OK`。

然后对返回的封包读取：

读取封包的 `[ESI+$64]` XOR `[ESI+$64+4]` XOR `TlsGetValue=$999`

必须使数据最后的得数为 `999`。

#### 数据包示例 (Recv)

```
2  218.22.13.17  10.66.6.71  128  Recv

0000  17 4F 4B 00 00 00 00 00 00 01 BC B0 CA B1 D3 EA    .OK.............
0010  50 4B B0 E6 BC B4 C8 D5 C6 F0 C3 E2 B7 D1 BF AA    PK..............
0020  B7 C5 2C C3 E2 B7 D1 D5 CA BA C5 A3 BA BC B0 CA    ..,.............
0030  B1 D3 EA A3 AC C3 DC C2 EB A3 BA 36 36 36 00 00    ...........666..
0040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0050  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060  00 00 36 40 C5 13 20 D5 CF 13 4A AC 2E 00 00 00    ..6@.. ...J.....
                ^^^^^^^^^^^ ^^^^^^^^^^^ ^^^^^^^^^^^
0070  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................

36 40 C5 13       20 D5 CF 13        4A AC 2E 00

DATA1=13C54036    DATA2=13CFD520     DATA3=002EAC4A
```

要求：`DATA_1` XOR `DATA_2` XOR `TlsGetValue=$999`

所以 `DATA_1` XOR `DATA_2` = `TlsGetValue` XOR `$999`。

如果：设 `DATA_1=$11111111`，`DATA_2=$11111111` XOR `$999` XOR `TlsGetValue`。

把 `DATA_1` 和 `DATA_2` 填充到返回的数据包中就解决了第一个验证部分。

下面的部分说明 `DATA3=GetTickCount` XOR `$258369`。

```
.text:100136F7 sub_100136F7    proc near                ; CODE XREF: sub_10037BB2+2Dp
.text:100136F7                 push    ebp
.text:100136F8                 mov     ebp, esp
.text:100136FA                 add     esp, 0FFFFFFECh
.text:100136FD                 call    ds:GetTickCount   ;读取时间
.text:10013703                 mov     edx, dword_1003D1B5;读取固定地址的数据
.text:10013709                 xor     edx, 258369h       ;XOR 一个固定的数值$258369
.text:1001370F                 sub     eax, edx           ;现在读取的时间-发送封包时的时间
.text:10013711                 cmp     eax, 0EA60h        ;如果大于$EA60 就出错
.text:10013716                 jbe     short locret_1001371F
.text:10013718                 mov     byte_1003D25F, 0
.text:1001371F

.text:1001371F locret_1001371F:                        ; CODE XREF: sub_100136F7+1Fj
.text:1001371F                 leave
.text:10013720                 retn

.text:10013720 sub_100136F7    endp
```

要求：

`GetTickCount` - (`DATA_3` XOR `$258369`) < `$0EA60`

所以 `DATA_3=$258369` XOR `GetTickCount(这个数据是发送封包中的数据)`。

这是对封包返回时间的要求。

一共 3 个数据有用我们就可以模拟服务器返回的封包了!

#### 数据包示例 (Send)

```
1  10.66.6.71  218.22.13.17  128  Send
```



# 数据包分析

## 接收数据包 `Recv`

`2   218.22.13.17   10.66.6.71   128   Recv`

```
0000  8F 69 50 6A 6C 71 32 58 76 30 57 4B 71 46 2B 6K    .iPjlq2Xv0WKqF+k
0010  43 7A 58 6B 52 48 67 3D 3D 00 00 00 00 00 00 00    CzXkRHg==.......
0020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0030  00 00 00 00 00 00 00 00 00 00 00 36 F8 4E E3 00    ...........6.N..
0040  00 00 00 00 58 02 00 00 20 03 00 00 58 02 00 00    ....X... ...X...
0050  20 03 00 00 00 00 00 00 00 00 00 00 97 A4 C5 77      ..............w
0060  00 00 00 00 20 8F 9C 0A 00 23 2F 0B 00 00 00 00    .... ...#/.....
0070  00 00 00 00 00 00 00 00 90 BE 70 73 60 08 15 00    ..........ps`.
```

## 发送数据包 `Send`

```
0000  17 4F 4B 00 00 00 00 00 00 01 BC B0 CA B1 D3 EA    .OK.............
0010  50 4B B0 E6 BC B4 C8 D5 C6 F0 C3 E2 B7 D1 BF AA    PK..............
0020  B7 C5 2C C3 E2 B7 D1 D5 CA BA C5 A3 BA BC B0 CA    ..,.............
0030  B1 D3 EA A3 AC C3 DC C2 EB A3 BA 36 36 36 00 00    ...........666..
0040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0050  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0060  00 00 36 40 C5 13 20 D5 CF 13 4A AC 2E 00 00 00    ..6@.. ...J.....
0070  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
```

## 服务器端解密验证逻辑

以下是服务端用于解密验证相关代码：

```pascal
// 返回的数据包结构
type
  PrecvDat = ^TrecvDat;
  TrecvDat = packed Record
    Tou:Byte;
    Str:string[54];
    Dontkonw_0:DWord;
    Dontkonw_1:DWord;
    Dontkonw_2:DWord;
    Dontkonw_3:DWord;
    Dontkonw_4:DWord;
    Dontkonw_5:DWord;
    Dontkonw_6:DWord;
    Dontkonw_7:DWord;
    Dontkonw_8:DWord;
    Dontkonw_9:DWord;
    Dontkonw_10:DWord;
    z:byte;
    DTlsGetValue:DWord;
    DGetTickCount:DWord;
    DontKonw_11:string[18];
  end;

// 模拟发送的数据包结构
type
  PsendDat = ^TsendDat;
  TsendDat = packed Record
    Tou:Byte;
    Str1:array[0..7]of byte;
    Fd:Byte;
    Str2:string[87];
    Data1:DWord;
    Data2:DWord;
    Data3:Dword;
    DontKonw:array[0..17] of byte;
  end;

// 第 1 个外挂发送来的封包进行解密效验计算
function DeRecv_0:boolean;
begin
result:=decode(recvbuf,3,$2e,$d9);
end;

// 对第 2 个外挂发送来的封包进行解密效验计算
function DeRecv_1:boolean;
begin
result:=decode(recvbuf,$3f,$2e,$d9);
end;

// 第 3 个外挂发送来的封包我们需要取得 2 个有用的数据用来计算发送的封包的数据
procedure DeRecv_2();
var
Temp:trecvDat;
begin
  Move(recvbuf,temp,$80);
  TlGValue:=temp.DTlsGetValue;
  TGTCount:=temp.DGetTickCount;
end;

// 模拟发送的第 1 个数据封包
procedure Ensend_0;
begin
FillChar(sendBuf[0],sizeof(sendbuf[0]),0);

sendBuf[0][0]:=$55;
sendBuf[0][1]:=$4f;
sendBuf[0][2]:=$4b;
sendBuf[0][$7b]:=$25;
sendBuf[0][$7c]:=$c3;

EnCode(sendBuf[0],$80,$2e,$d9);
end;

// 模拟发送的第 2 个数据封包
procedure Ensend_1;
begin
FillChar(sendBuf[1],sizeof(sendbuf[1]),0);

sendBuf[1][1]:=$4f; sendBuf[1][2]:=$4b;sendBuf[1][51]:=$9a;sendBuf[1][52]:=$2;

sendBuf[1][55]:=$77;sendBuf[1][56]:=$d0;sendBuf[1][57]:=$c2;sendBuf[1][58]:=$16;

sendBuf[1][60]:=$60;sendBuf[1][61]:=$f5;sendBuf[1][62]:=$e4;sendBuf[1][63]:=$8f;

sendBuf[1][64]:=$8f;sendBuf[1][65]:=$2b;sendBuf[1][66]:=$37;sendBuf[1][67]:=$56;

sendBuf[1][68]:=$59;sendBuf[1][69]:=$74;sendBuf[1][70]:=$33;sendBuf[1][71]:=$52;

sendBuf[1][72]:=$38;sendBuf[1][73]:=$6f;sendBuf[1][74]:=$44;sendBuf[1][75]:=$66;

sendBuf[1][76]:=$76;sendBuf[1][77]:=$4d;sendBuf[1][78]:=$44;sendBuf[1][79]:=$32;

sendBuf[1][80]:=$4f;sendBuf[1][81]:=$77;sendBuf[1][82]:=$58;sendBuf[1][83]:=$4e;

sendBuf[1][84]:=$77;sendBuf[1][85]:=$75;sendBuf[1][86]:=$41;sendBuf[1][87]:=$3d;

sendBuf[1][88]:=$3d;sendBuf[1][123]:=$d4;sendBuf[1][124]:=$a9;sendBuf[1][125]:=$76;

sendBuf[1][126]:=$ec;

EnCode(sendBuf[1],$80,$2e,$d9);
end;

// 模拟发送的第 3 个数据封包
procedure Ensend_2();
var
senddata:TsendDat;
data_1:dword;
  a:word; s:string;
begin
FillChar(sendBuf[2],sizeof(sendbuf[2]),0);

data_1:=$11111111;//这个是我自己设定的数据,可以设为任意值

senddata.Tou:=$17;

FillChar(senddata.Str1,8,0);

senddata.Str1[0]:=$4f;

senddata.Str1[1]:=$4b;//填充数据包头

senddata.Fd:=01;

FillChar(senddata.Str2,88,0);

senddata.Str2:='我是 PowerBoy 我破解的好用吗?呵呵!';

senddata.Data1:=Data_1;                   //设定的 DATA_1 进行填充

senddata.Data2:=data_1 xor TlGValue xor $999;//由 DATA_1 算出 DATA_2 填充

senddata.Data3:=TGTCount xor $258369;      //DATA_3 也是上面数据经过计算之后进行填充

FillChar(senddata.DontKonw ,18,0);          //封包其余部分都为 00

Move(senddata,sendBuf[2],$80);
end;
```

## 加解密汇编实现

```pascal
procedure EnCode(Indata: Pointer; len: integer; Key1 :Byte; Key2 :Byte);stdcall;
asm
          PUSH    EBP

          PUSH    EBX

          PUSH    ESI

          MOV     EBP,ESP

          MOV     EDX,LEN

          XOR     ECX,ECX

          XOR     EAX,EAX

          DEC     EDX

          MOV     ESI,indata

@@loc_00401DE0: ADD     AL,[ESI+ECX]

          INC     ECX

          CMP     ECX,EDX

          JNZ     @@loc_00401DE0

          XOR     AL,key1

          MOV     DL,key2

          IMUL    DL

          MOV     [ESI+ECX],AL

          MOV     EDX,4BCh

          XOR     ECX,ECX

          XOR     EAX,EAX

          MOV     ESI,indata

@@loc_00401DFF: MOV     AL,[ESI+ECX]

          MOV     BL,DH

          XOR     BL,AL

          MOVZX   EAX,BL

          ADD     EAX,EDX

          MOV     [ESI+ECX],BL

          LEA     EDX,[EAX+EAX*2]

          SHL     EDX,3

          SUB     EDX,EAX

          SHL     EDX,3

          SUB     EDX,EAX

          LEA     EDX,[EAX+EDX*4]

          SHL     EDX,5

          SUB     EDX,EAX

          SUB     EDX,3596h

          INC     ECX

          CMP     ECX,len

          JNZ     @@loc_00401DFF

          POP     ESI

          POP     EBX

          POP     EBP
end;

function DeCode(var indata:Tbuff;len:integer;Key1:Byte;Key2:Byte):boolean;
var
t:Pointer;
begin
t:=@indata;

asm
          PUSH    EBP

          MOV     EBP,ESP

          PUSH    ECX

          PUSH    EBX

          PUSH    ESI

          MOV     EDX,4BCh

          XOR     ECX,ECX

          XOR     EAX,EAX

          MOV     ESI,t

@@loc_00401D62: MOV     AL,[ESI+ECX]

          MOV     BL,DH

          XOR     BL,AL

          MOVZX   AX,AL

          ADD     EAX,EDX

          MOV     [ESI+ECX],BL

          LEA     EDX,[EAX+EAX*2]

          SHL     EDX,3

          SUB     EDX,EAX

          SHL     EDX,3

          SUB     EDX,EAX

          LEA     EDX,[EAX+EDX*4]

          SHL     EDX,5

          SUB     EDX,EAX

          SUB     EDX,3596h

          INC     ECX

          CMP     ECX,len

          JNZ     @@loc_00401D62

          MOV     EDX,len

          XOR     ECX,ECX

          XOR     EAX,EAX

          DEC     EDX

          MOV     ESI,t

@@loc_00401D9E: ADD     AL,[ESI+ECX]

          INC     ECX

          CMP     ECX,EDX

          JNZ     @@loc_00401D9E

          XOR     AL,key1

          MOV     DL,key2

          IMUL    DL

          CMP     [ESI+ECX],AL
```



```
JE      @@loc_00401DBA  

MOV     EAX,1  

JMP     @@loc_00401DBC  

@@loc_00401DBA: XOR     EAX,EAX  

@@loc_00401DBC: POP     ESI  

        POP     EBX  

        MOV     ESP,EBP  

        POP     EBP  

end;  

end;  

004020FF     90          NOP  

00402100   .  56          PUSH    ESI  

00402101   .  8BF1        MOV     ESI,ECX                 ;  ntdll.77F5166A  

00402103   .  6A 01       PUSH    1  

00402105   .  E8 B6BE0100 CALL    0041DFC0  

0040210A   .  8D8E 10010000 LEA    ECX,[ESI+110]  

00402110   .  E8 29CA0100 CALL    0041EB3E  

00402115   .  85C0        TEST    EAX,EAX  

00402117   .  75 15       JNZ     SHORT 0040212E  ;这里修改一下就不用输入用户名了  

00402119   .  6A 30       PUSH    30  

0040211B   .  68 84514300 PUSH    00435184  

00402120   .  68 C0514300 PUSH    004351C0  

00402125   .  8BCE        MOV     ECX,ESI  

00402127   .  E8 B3B40100 CALL    0041D5DF  

0040212C   .  5E          POP     ESI                   ;  JSY.00429700  

0040212D   .  C3          RETN  
```

```
.text:10001030 name            = sockaddr ptr -14h  

.text:10001030 s             = dword ptr -4  

.text:10001030 hWnd           = dword ptr  8  

.text:10001030 wMsg           = dword ptr  0Ch  

.text:10001030 arg_8          = dword ptr  10h  

.text:10001030  

.text:10001030            push    ebp  

.text:10001031            mov     ebp, esp  

.text:10001033            add     esp, 0FFFFFFECh  

.text:10001036            push    0             ; protocol  

.text:10001038            push    1             ; type  

.text:1000103A            push    2             ; af  

.text:1000103C            call    ds:socket       ; Create a socket which is bound to a  

.text:1000103C                     ; specific service provider  

.text:10001042            cmp     eax, 0FFFFFFFFh  

.text:10001045            jz      short loc_10001091  

.text:10001047            mov     [ebp+s], eax  

.text:1000104A            push    10h  

.text:1000104C            lea     eax, [ebp+name]  

.text:1000104F            push    eax  

.text:10001050            call    sub_1001AC57  

.text:10001055            mov     eax, [ebp+arg_8]  

.text:10001058            xchg    ah, al  

.text:1000105A            mov     word ptr [ebp+name.sa_data], ax  

.text:1000105E            mov     [ebp+name.sa_family], 2  

.text:10001064            call    sub_10001097  

.text:10001069            mov     dword ptr [ebp+name.sa_data+2], eax//DLL 访问的 IP 保存  

.text:1000106C            push    33h           ; lEvent  

.text:1000106E            push    [ebp+wMsg]     ; wMsg  

.text:10001071            push    [ebp+hWnd]     ; hWnd  

.text:10001074            push    [ebp+s]        ; s  

.text:10001077            call    ds:WSAAsyncSelect ; Request Windows message-based  

.text:10001077                     ; notification of network events  

.text:10001077                     ; for a socket  

.text:1000107D            push    10h           ; namelen  

.text:1000107F            lea     eax, [ebp+name]  

.text:10001082            push    eax           ; name  

.text:10001083            push    [ebp+s]       ; s  

.text:10001086            call    ds:connect  

.text:1000108C            mov     eax, [ebp+s]  

.text:1000108F            jmp     short locret_10001093  
```

`218.22.13.17`-->`DA.16.0D.11`-->`110D16DA`

`127.0.0.1`   -->`7F.0.0.1`   -->`0100007F`

```
.text:10001097 sub_10001097    proc near          ; CODE XREF: sub_10001030+34p  

.text:10001097            cmp     byte_1003D261, 0  

.text:1000109E            jz      short loc_100010AD  

.text:100010A0  

.text:100010A0 loc_100010A0:                   ; DATA XREF: sub_1001DB4A+8o  

.text:100010A0            mov     ah, 11h  

.text:100010A2            mov     al, 0Dh  

.text:100010A4            shl     eax, 10h  

.text:100010A7            mov     ah, 16h  

.text:100010A9            mov     al, 0DAh  

.text:100010AB            jmp     short locret_100010B8  

.text:100010AD ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  

.text:100010AD  

.text:100010AD loc_100010AD:                   ; CODE XREF: sub_10001097+7j  

.text:100010AD                     ; DATA XREF: sub_1001DB4A+1Ao  

.text:100010AD            mov     ah, 11h  

.text:100010AF            mov     al, 0Dh  

.text:100010B1            shl     eax, 10h  

.text:100010B4            mov     ah, 16h  

.text:100010B6            mov     al, 0DAh  

.text:100010B8  

.text:100010B8 locret_100010B8:                ; CODE XREF: sub_10001097+14j  

.text:100010B8            retn 

 4dd466d0e1e0cc123afbe504da2462ca
```
