
```
0051F17A  |>  8B5D FC       MOV EBX,DWORD PTR [EBP-4]
0051F17D  |.  33C0          XOR EAX,EAX
```

这里就是将用户名添0后成8字节的倍数，然后分成`n`组，分别用每一组消息修改`inbuff`的前8个字节，然后进行MD5编码，循环`n`次。

---

要写出 keygen，必须跟进`CALL AutoRunP.0051F034`，跟进 MD5 函数

```
0051F1FF  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
0051F202  |.  C745 E4 6745> MOV DWORD PTR [EBP-1C],1234567 ;  传递四个链接变量(已经变形)
0051F209  |.  C745 E8 EFCD> MOV DWORD PTR [EBP-18],89ABCDE>
0051F210  |.  C745 EC DCFE> MOV DWORD PTR [EBP-14],BA98FED>
0051F217  |.  C745 F0 2143> MOV DWORD PTR [EBP-10],7650432>
```

## 变形1

这里已经不是：

```
A ＝ 0x67452301
B ＝ 0xefcdab89
C ＝ 0x98badcfe
D ＝ 0x10325476
```

找到一个变形

```
0051F21E  |.  8B45 F0       MOV EAX,DWORD PTR [EBP-10]
0051F221  |.  8903          MOV DWORD PTR [EBX],EAX        ;  复制四个链接变量的副本
0051F223  |.  8B45 EC       MOV EAX,DWORD PTR [EBP-14]
0051F226  |.  8906          MOV DWORD PTR [ESI],EAX        ;  复制四个链接变量的副本
0051F228  |.  8B45 E8       MOV EAX,DWORD PTR [EBP-18]
0051F22B  |.  8907          MOV DWORD PTR [EDI],EAX        ;  复制四个链接变量的副本
0051F22D  |.  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
0051F230  |.  8945 D4       MOV DWORD PTR [EBP-2C],EAX     ;  复制四个链接变量的副本
0051F233  |.  6A 10         PUSH 10
0051F235  |.  8D45 D0       LEA EAX,DWORD PTR [EBP-30]
0051F238  |.  B9 01000000   MOV ECX,1
0051F23D  |.  8B15 B4F15100 MOV EDX,DWORD PTR [51F1B4]     ;  AutoRunP.0051F1B8
0051F243  |.  E8 7465EEFF   CALL AutoRunP.004057BC
0051F248  |.  83C4 04       ADD ESP,4                      ;  下面将 128 位信息填充成 512 位
0051F24B  |.  8B45 FC       MOV EAX,DWORD PTR [EBP-4]
0051F24E  |.  8B00          MOV EAX,DWORD PTR [EAX]
0051F250  |.  8B55 D0       MOV EDX,DWORD PTR [EBP-30]
0051F253  |.  8902          MOV DWORD PTR [EDX],EAX        ;  填充

……………………（总共有16个填充）

0051F2EF  |.  8B45 FC       MOV EAX,DWORD PTR [EBP-4]
0051F2F2  |.  8B00          MOV EAX,DWORD PTR [EAX]
0051F2F4  |.  8B55 D0       MOV EDX,DWORD PTR [EBP-30]
0051F2F7  |.  8942 3C       MOV DWORD PTR [EDX+3C],EAX     ;  填充
0051F2FA  |.  8B45 D4       MOV EAX,DWORD PTR [EBP-2C]
```

## 变形2

和标准的 MD5 的填充不一样，并不是加个1在消息后然后填充0，最后附上消息长度。

而是将消息分为四个`dword`，逆序、顺序、逆序、顺序填充成 512 位。

比如消息是：

```
0012F104  31 32 33 34 35 36 37 38 39 30 41 42 43 44 45 46  1234 5678 90AB CDEF
```

填充后是：

```
0106CF74  43 44 45 46 39 30 41 42 35 36 37 38 31 32 33 34  CDEF 90AB 5678 1234
0106CF84  31 32 33 34 35 36 37 38 39 30 41 42 43 44 45 46  1234 5678 90AB CDEF
0106CF94  43 44 45 46 39 30 41 42 35 36 37 38 31 32 33 34  CDEF 90AB 5678 1234
0106CFA4  31 32 33 34 35 36 37 38 39 30 41 42 43 44 45 46  1234 5678 90AB CDEF
```

```
//------------- round 1 -------------//

0051F2FD  |.  50            PUSH EAX                       ; /Arg4
0051F2FE  |.  8B45 D0       MOV EAX,DWORD PTR [EBP-30]     ; |
0051F301  |.  8B00          MOV EAX,DWORD PTR [EAX]        ; |
0051F303  |.  50            PUSH EAX                       ; |Arg3
0051F304  |.  6A 07         PUSH 7                         ; |Arg2 = 00000007
0051F306  |.  68 78A46AD7   PUSH D76AA478                  ; |Arg1 = D76AA478
0051F30B  |.  8BC3          MOV EAX,EBX                    ; |
0051F30D  |.  8B0F          MOV ECX,DWORD PTR [EDI]        ; |
0051F30F  |.  8B16          MOV EDX,DWORD PTR [ESI]        ; |
0051F311  |.  E8 CE070000   CALL AutoRunP.0051FAE4         ; \AutoRunP.0051FAE4
0051F316  |.  8B07          MOV EAX,DWORD PTR [EDI]

……………………

0051F4C2  |.  8B17          MOV EDX,DWORD PTR [EDI]        ; |
0051F4C4  |.  E8 1B060000   CALL AutoRunP.0051FAE4         ; \AutoRunP.0051FAE4
0051F4C9  |.  8B45 D4       MOV EAX,DWORD PTR [EBP-2C]

//------------- round 2 -------------//

0051F4CC  |.  50            PUSH EAX                       ; /Arg4
0051F4CD  |.  8B45 D0       MOV EAX,DWORD PTR [EBP-30]     ; |
0051F4D0  |.  8B40 04       MOV EAX,DWORD PTR [EAX+4]      ; |
0051F4D3  |.  50            PUSH EAX                       ; |Arg3
0051F4D4  |.  6A 05         PUSH 5                         ; |Arg2 = 00000005
0051F4D6  |.  68 62251EF6   PUSH F61E2562                  ; |Arg1 = F61E2562
0051F4DB  |.  8BC3          MOV EAX,EBX                    ; |
0051F4DD  |.  8B0F          MOV ECX,DWORD PTR [EDI]        ; |
0051F4DF  |.  8B16          MOV EDX,DWORD PTR [ESI]        ; |
0051F4E1  |.  E8 3A060000   CALL AutoRunP.0051FB20         ; \AutoRunP.0051FB20
0051F4E6  |.  8B07          MOV EAX,DWORD PTR [EDI]

……………………

0051F691  |.  8B17          MOV EDX,DWORD PTR [EDI]        ; |
0051F693  |.  E8 88040000   CALL AutoRunP.0051FB20         ; \AutoRunP.0051FB20
0051F698  |.  8B45 D4       MOV EAX,DWORD PTR [EBP-2C]

//------------- round 3 -------------//

0051F69B  |.  50            PUSH EAX                       ; /Arg4
0051F69C  |.  8B45 D0       MOV EAX,DWORD PTR [EBP-30]     ; |
0051F69F  |.  8B40 14       MOV EAX,DWORD PTR [EAX+14]     ; |
0051F6A2  |.  50            PUSH EAX                       ; |Arg3
0051F6A3  |.  6A 04         PUSH 4                         ; |Arg2 = 00000004
0051F6A5  |.  68 4239FAFF   PUSH FFFA3942                  ; |Arg1 = FFFA3942
0051F6AA  |.  8BC3          MOV EAX,EBX                    ; |
0051F6AC  |.  8B0F          MOV ECX,DWORD PTR [EDI]        ; |
0051F6AE  |.  8B16          MOV EDX,DWORD PTR [ESI]        ; |
0051F6B0  |.  E8 A7040000   CALL AutoRunP.0051FB5C         ; \AutoRunP.0051FB5C
0051F6B5  |.  8B07          MOV EAX,DWORD PTR [EDI]

……………………

0051F860  |.  8B17          MOV EDX,DWORD PTR [EDI]        ; |
0051F862  |.  E8 F5020000   CALL AutoRunP.0051FB5C         ; \AutoRunP.0051FB5C
0051F867  |.  8B45 D4       MOV EAX,DWORD PTR [EBP-2C]

//------------- round 4 -------------//

0051F86A  |.  50            PUSH EAX                       ; /Arg4
0051F86B  |.  8B45 D0       MOV EAX,DWORD PTR [EBP-30]     ; |
0051F86E  |.  8B00          MOV EAX,DWORD PTR [EAX]        ; |
0051F870  |.  50            PUSH EAX                       ; |Arg3
0051F871  |.  6A 06         PUSH 6                         ; |Arg2 = 00000006
0051F873  |.  68 442229F4   PUSH F4292244                  ; |Arg1 = F4292244
0051F878  |.  8BC3          MOV EAX,EBX                    ; |
0051F87A  |.  8B0F          MOV ECX,DWORD PTR [EDI]        ; |
0051F87C  |.  8B16          MOV EDX,DWORD PTR [ESI]        ; |
0051F87E  |.  E8 15030000   CALL AutoRunP.0051FB98         ; \AutoRunP.0051FB98
0051F883  |.  8B07          MOV EAX,DWORD PTR [EDI]

……………………

0051FA2F  |.  8B17          MOV EDX,DWORD PTR [EDI]        ; |
0051FA31  |.  E8 62010000   CALL AutoRunP.0051FB98         ; \AutoRunP.0051FB98
0051FA36  |.  8B45 F0       MOV EAX,DWORD PTR [EBP-10]
0051FA39  |.  0303          ADD EAX,DWORD PTR [EBX]        ;  + d
0051FA3B  |.  8B55 FC       MOV EDX,DWORD PTR [EBP-4]
0051FA3E  |.  8902          MOV DWORD PTR [EDX],EAX        ;  save
0051FA40  |.  8B45 EC       MOV EAX,DWORD PTR [EBP-14]
0051FA43  |.  0306          ADD EAX,DWORD PTR [ESI]        ;  + c
0051FA45  |.  8B55 F8       MOV EDX,DWORD PTR [EBP-8]
0051FA48  |.  8902          MOV DWORD PTR [EDX],EAX        ;  save
0051FA4A  |.  8B45 E8       MOV EAX,DWORD PTR [EBP-18]
0051FA4D  |.  0307          ADD EAX,DWORD PTR [EDI]        ;  + b
0051FA4F  |.  8B55 F4       MOV EDX,DWORD PTR [EBP-C]
0051FA52  |.  8902          MOV DWORD PTR [EDX],EAX        ;  save
0051FA54  |.  8B45 E4       MOV EAX,DWORD PTR [EBP-1C]
0051FA57  |.  0345 D4       ADD EAX,DWORD PTR [EBP-2C]     ;  + a
0051FA5A  |.  8B55 08       MOV EDX,DWORD PTR [EBP+8]
0051FA5D  |.  8902          MOV DWORD PTR [EDX],EAX        ;  save
```

## 变形3

跟进后发现`HASH`函数本身没有变形，但是参与运算的链接变量的顺序变化了。

---

## 破解心得

要得到正确的`serial`和`key`，按照下面的流程：

先对用户名添0后分成`n`组，再反序，分别用每一组消息修改`inbuff`的前8个字节，然后进行 MD5 编码，循环`n`次后，结果的最后 32 位作为`dword`转为十六进制

文本作为`serial`的中间 8 个字符。

然后前面接上`"0018"`，后面接上长度是 4 的任意 16 进制文本，总共 16 个字符作为`serial`。
