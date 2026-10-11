
# 汇编代码分析

```
0053C19B  |. 75 20          |JNZ SHORT EsoftPro.0053C1BD  

0053C19D  |. 8A4424 05      |MOV AL,BYTE PTR SS:[ESP+5]  

0053C1A1  |. C1E0 04        |SHL EAX,4  

0053C1A4  |. 81E2 FF000000  |AND EDX,0FF  

0053C1AA  |. C1EA 02        |SHR EDX,2  

0053C1AD  |. 0AC2           |OR AL,DL  

0053C1AF  |. 8B5424 08      |MOV EDX,DWORD PTR SS:[ESP+8]  

0053C1B3  |. 88442A 01      |MOV BYTE PTR DS:[EDX+EBP+1],AL  

0053C1B7  |. 8D45 02        |LEA EAX,DWORD PTR SS:[EBP+2]  

0053C1BA  |. 45             |INC EBP  

0053C1BB  |. EB 38          |JMP SHORT EsoftPro.0053C1F5  

0053C1BD  |> 80FA FF        |CMP DL,0FF  

0053C1C0  |. 74 33          |JE SHORT EsoftPro.0053C1F5  

0053C1C2  |. 8A4424 05      |MOV AL,BYTE PTR SS:[ESP+5]  

0053C1C6  |. C1E0 04        |SHL EAX,4  

0053C1C9  |. 81E2 FF000000 |AND EDX,0FF  

0053C1CF  |. C1EA 02        |SHR EDX,2  

0053C1D2  |. 0AC2           |OR AL,DL  

0053C1D4  |. 8B5424 08      |MOV EDX,DWORD PTR SS:[ESP+8]  

0053C1D8  |. 88442A 01      |MOV BYTE PTR DS:[EDX+EBP+1],AL  

0053C1DC  |. 8B4424 08      |MOV EAX,DWORD PTR SS:[ESP+8]  

0053C1E0  |. 8A5424 06      |MOV DL,BYTE PTR SS:[ESP+6]  

0053C1E4  |. C1E2 06        |SHL EDX,6  

0053C1E7  |. 0A5424 07      |OR DL,BYTE PTR SS:[ESP+7]  

0053C1EB  |. 885428 02      |MOV BYTE PTR DS:[EAX+EBP+2],DL  

0053C1EF  |. 8D45 03        |LEA EAX,DWORD PTR SS:[EBP+3]  

0053C1F2  |. 83C5 02        |ADD EBP,2  

0053C1F5  |> 45             |INC EBP  

0053C1F6  |. FF4C24 0C      |DEC DWORD PTR SS:[ESP+C]  

0053C1FA  |.⁰F85 02FFFFFF \\JNZ EsoftPro.0053C102  

0053C200  |> 83C4 10        |ADD ESP,10  

0053C203  |. 5D             |POP EBP  

0053C204  |. 5F             |POP EDI  

0053C205  |. 5E             |POP ESI  

0053C206  |\\. C3             |RETN  

3,  
```

## 进一步的变换

```
(0053CD10   . FF93 80000000 CALL DWORD PTR DS:[EBX+80])

00540CBC  /. 55              |PUSH EBP  
00540CBD  |. 8BEC            |MOV EBP,ESP  
00540CBF  |. 83C4 E8         |ADD ESP,-18  
00540CC2  |. 53              |PUSH EBX  
00540CC3  |. 56              |PUSH ESI  
00540CC4  |. 57              |PUSH EDI  
00540CC5  |. 894D FC         |MOV DWORD PTR SS:[EBP-4],ECX  
00540CC8  |. 8BD8            |MOV EBX,EAX  
00540CCA  |. 807B 30 00      |CMP BYTE PTR DS:[EBX+30],0  
00540CCE  |. 75 16           |JNZ SHORT EsoftPro.00540CE6  
00540CD0  |. B9 440D5400     |MOV ECX,EsoftPro.00540D44             ;  ASCII "Cipher not initialized"  
00540CD5  |. B2 01           |MOV DL,1  
00540CD7  |. A1 08C65300     |MOV EAX,DWORD PTR DS:[53C608]  
00540CDC  |. E8 A7C4ECFF     |CALL EsoftPro.0040D188  
00540CE1  |. E8 AA35ECFF     |CALL EsoftPro.00404290  
00540CE6  |> 8BF2            |MOV ESI,EDX  
00540CE8  |. 8B45 FC         |MOV EAX,DWORD PTR SS:[EBP-4]  
00540CEB  |. 8945 F8         |MOV DWORD PTR SS:[EBP-8],EAX  
00540CEE  |. 8B45 08         |MOV EAX,DWORD PTR SS:[EBP+8]  
00540CF1  |. 85C0            |TEST EAX,EAX  
00540CF3  |. 76 3E           |JBE SHORT EsoftPro.00540D33  
00540CF5  |. 8945 E8         |MOV DWORD PTR SS:[EBP-18],EAX  
00540CF8  |> 8A06            |MOV AL,BYTE PTR DS:[ESI]  
00540CFA  |. 8845 F7         |MOV BYTE PTR SS:[EBP-9],AL  
00540CFD  |. 8D4D EF         |LEA ECX,DWORD PTR SS:[EBP-11]  
00540D00  |. 8D53 40         |LEA EDX,DWORD PTR DS:[EBX+40]  
00540D03  |. 8BC3            |MOV EAX,EBX  
00540D05  |. 8B38            |MOV EDI,DWORD PTR DS:[EAX]  
00540D07  |. FF57 6C         |CALL DWORD PTR DS:[EDI+6C]               blowfish_en  
00540D0A  |. 8A06            |MOV AL,BYTE PTR DS:[ESI]  
00540D0C  |. 3245 EF         |XOR AL,BYTE PTR SS:[EBP-11]  
00540D0F  |. 8B55 F8         |MOV EDX,DWORD PTR SS:[EBP-8]  
00540D12  |. 8802            |MOV BYTE PTR DS:[EDX],AL  
00540D14  |. 8D53 40         |LEA EDX,DWORD PTR DS:[EBX+40]  
00540D17  |. 8D43 41         |LEA EAX,DWORD PTR DS:[EBX+41]  
00540D1A  |. B9 07000000     |MOV ECX,7  
00540D1F  |. E8 681CECFF     |CALL EsoftPro.0040298C  
00540D24  |. 8A45 F7         |MOV AL,BYTE PTR SS:[EBP-9]  
00540D27  |. 8843 47         |MOV BYTE PTR DS:[EBX+47],AL  
00540D2A  |. 46              |INC ESI  
00540D2B  |. FF45 F8         |INC DWORD PTR SS:[EBP-8]  
00540D2E  |. FF4D E8         |DEC DWORD PTR SS:[EBP-18]  
00540D31  |.⁷⁵ C5            |JNZ SHORT EsoftPro.00540CF8  
00540D33  |> 5F              |POP EDI  
00540D34  |. 5E              |POP ESI  
00540D35  |. 5B              |POP EBX  
00540D36  |. 8BE5            |MOV ESP,EBP  
00540D38  |. 5D              |POP EBP  
00540D39  |\\. C2 0400         |RETN 4  
```

分析这段代码知道变换过程是这样的：

设 `basedecode` 后得到的字符串为

`"1234"`

先对 `hex` 串 `'FA6A1510698C8467'` 运用 `blowfish` 加密得到 `hex` 串:

`'4D72933486AEF1F0'`

则 `ascii('1')⁰x4d=0x7c`，`0x7c` 就是变换后的结果。

然后把 `ascii('1')` 添加到 `'FA6A1510698C8467'` 后面，得到 `'FA6A1510698C846731'`

再对 `'6A1510698C846731'` 运用 `blowfish` 加密。

加密结果的的头两位再与 `ascii('2')` 异或，得到变换结果的第 2 位，

如此循环处理 `len(base6 解码后的字符串)` 次，依次连接变换后的结果，得到最后用来与特定字串

`"'作者的名字，软件信息，网站等等'+'网卡 mac'+'硬盘序列号'"` 比较的字串.

变换部分清楚了，那剩下的就是找到 `blowfish` 计算的 `key` 了.

通过对 `hex` 串 `'FA6A1510698C8467'` 地址设置内存断点跟踪，来到这儿：

```
005408CD  |. 8D4B 38        |LEA ECX,DWORD PTR DS:[EBX+38]  
005408D0  |. 8D53 38        |LEA EDX,DWORD PTR DS:[EBX+38]  
005408D3  |. 8BC3           |MOV EAX,EBX  
005408D5  |. 8B30           |MOV ESI,DWORD PTR DS:[EAX]  
005408D7  |. FF56 6C        |CALL DWORD PTR DS:[ESI+6C]          blowfish_en  
005408DA  |. 8BC3           |MOV EAX,EBX  
005408DC  |. 8B10           |MOV EDX,DWORD PTR DS:[EAX]  
005408DE  |. FF52 48        |CALL DWORD PTR DS:[EDX+48]  
005408E1  |. EB 16          |JMP SHORT EsoftPro.005408F9  
```

原来 `'FA6A1510698C8467'` 是用 `blowfish` 加密 64 位全零串得到的. 那剩下的就是找出 `key` 了。

对 `blowfish` 的 `pbox` 数据地址设置内存访问断点，来到这里：

```
005408A4  |. 8B4D FC        |MOV ECX,DWORD PTR SS:[EBP-4]       ; |  
005408A7  |. 8BC3           |MOV EAX,EBX                         ; |  
005408A9  |. E8 1EC0FFFF    |CALL EsoftPro.0053C8CC              ; \\检查密钥长度  
005408AE  |. 8BD7           |MOV EDX,EDI  
005408B0  |. 8B4D FC        |MOV ECX,DWORD PTR SS:[EBP-4]  
005408B3  |. 8BC3           |MOV EAX,EBX  
005408B5  |. 8B38           |MOV EDI,DWORD PTR DS:[EAX]  
005408B7  |. FF57 5C        |CALL DWORD PTR DS:[EDI+5C]          ;  密钥初始化  
005408BA  |. 85F6           |TEST ESI,ESI  
005408BC  |. 75 25          |JNZ SHORT EsoftPro.005408E3  
005408BE  |. 8D43 38        |LEA EAX,DWORD PTR DS:[EBX+38]  
005408C1  |. 33C9           |XOR ECX,ECX  
005408C3  |. BA 08000000    |MOV EDX,8  
005408C8  |. E8 EF28ECFF    |CALL EsoftPro.004031BC  
005408CD  |. 8D4B 38        |LEA ECX,DWORD PTR DS:[EBX+38]  
005408D0  |. 8D53 38        |LEA EDX,DWORD PTR DS:[EBX+38]  
```

得到 `key` 是:

`'BA0E57F2906C60C62BBB8C7EEBB53B0E'`

(`in hex`)

继续对这个 `key` 的地址设置内存断点，分析得到：

```
005406B1    . 8B53 38      |MOV EDX,DWORD PTR DS:[EBX+38]  
005406B4    . 8910         |MOV DWORD PTR DS:[EAX],EDX  
005406B6    . 8D83 8C000000 |LEA EAX,DWORD PTR DS:[EBX+8C]  
005406BC    . 8B53 34      |MOV EDX,DWORD PTR DS:[EBX+34]  
005406BF    . 8910         |MOV DWORD PTR DS:[EAX],EDX  
005406C1    . 8BC3         |MOV EAX,EBX  
005406C3    . E8 20F5FFFF  |CALL EsoftPro.0053FBE8           ;  md5  
005406C8    . 8BD6         |MOV EDX,ESI  
005406CA    . 8D43 40      |LEA EAX,DWORD PTR DS:[EBX+40]  
005406CD    . B9 10000000  |MOV ECX,10  
005406D2    . E8 B522ECFF  |CALL EsoftPro.0040298C  
```

加密 `key` 是通过对字符串 `'作者的名字’` 做 `md5` 计算得到的。

如果有兴趣，也可以看看机器码的计算过程，可以发现，

机器码是对 `'ghq1978'` 进行 `tiger` 计算，得到 `key`:

`'534E72C523DDF7A89079166D71563965547BB63B0B4FCDD4'`

(`in hex`)

然后用这个 `key`，字符串 `"'ghq1978'+'网卡 mac'+'硬盘序列号'"`

进行和 3 处一样的变换，把结果用 `base64` 编码输出就是软件显示的机器码.

整个授权文件的验证过程烦了点，主要是那些循环太讨厌了。

但看清楚了过程，要写注册机还是比较简单的。

水平有限，分析得不对的地方，敬请诸位大侠赐教！

`by ikki`

`4dd466d0e1e0cc123afbe504da2462ca`
