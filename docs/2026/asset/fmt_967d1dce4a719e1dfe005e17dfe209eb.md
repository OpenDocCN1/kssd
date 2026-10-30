
```
00411A9C  17 FC 82 19 BE 1C B8 60 21 69 1D 93 30 31 37 4B  鼈?竊!i?17K
00411AAC  08 BA A3 4F E3 F3 6A 33 41 47 95 EC 21 99 29 BF  海O泱j3AG曥!?
00411ABC  75 C5 53 E8 58 39 4F 6B C1 12 B6 73 CC 31 88 35  u臩鑈9Ok?秙??
00411ACC  E9 24 A5 F5 75 ED 75 1C 16 6A 1E E6 07 9A A9 36  ?ヵu韚j?毄6
00411ADC  A1 61 62 47 5B 39 F4 77 D0 F6 72 AF 3A 0A 6E 56  bG[9魒婿r?.nV
00411AEC  12 FA 2B A3 86 31 B8 42 12 1D 03 62 F6 74 DB 09  ?1窧b鰐?
00411AFC  B0 2B B9 94 BD F4 AA 67 CC 31 B0 0F 24 01 64 0F  ?箶紧猤??$d
00411B0C  70 31 67 21 58 C6 5A 9B 7F 32 6E F8 0C 80 34 EC  p1g!X芞?2n?€4
00411B1C  69 69 79 32 30 68 BC 06 A8 0A 82 83 6D 53 6E 73  iiy20h??們mSns
00411B2C  CA 91 0D A6                                      蕬.?m
```

目的：

```
00EC0000  17 FC 82 19 BE 1C B8 60 21 69 1D 93 30 31 37 4B  鼈?竊!i?17K
00EC0010  08 BA A3 4F E3 F3 6A 33 41 47 95 EC 21 99 29 BF  海O泱j3AG曥!?
00EC0020  75 C5 53 E8 58 39 4F 6B C1 12 B6 73 CC 31 88 35  u臩鑈9Ok?秙??
00EC0030  E9 24 A5 F5 75 ED 75 1C 16 6A 1E E6 07 9A A9 36  ?ヵu韚j?毄6
00EC0040  A1 61 62 47 5B 39 F4 77 D0 F6 72 AF 3A 0A 6E 56  bG[9魒婿r?.nV
00EC0050  12 FA 2B A3 86 31 B8 42 12 1D 03 62 F6 74 DB 09  ?1窧b鰐?
00EC0060  B0 2B B9 94 BD F4 AA 67 CC 31 B0 0F 24 01 64 0F  ?箶紧猤??$d
00EC0070  70 31 67 21 58 C6 5A 9B 7F 32 6E F8 0C 80 34 EC  p1g!X芞?2n?€4
00EC0080  69 69 79 32 30 68 BC 06 A8 0A 82 83 6D 53 6E 73  iiy20h??們mSns
00EC0090  CA 91 0D A6                                      蕬.?.
```

SMC解码，异或：

```
0041187A   > /81FE 94000000         CMP ESI,0x94                                       ;  0x94 循环条件 （148）
00411880   . |7D 27                 JGE X7-不问少.004118A9
00411882   . |8BC6                  MOV EAX,ESI
00411884   . |33D2                  XOR EDX,EDX
00411886   . |F771 34               DIV DWORD PTR DS:[ECX+0x34]                        ;  % 0x14
00411889   . |46                    INC ESI                                            ;  +1
0041188A   . |8975 E0               MOV DWORD PTR SS:[EBP-0x20],ESI                    ;  计数器
0041188D   . |8D41 24               LEA EAX,DWORD PTR DS:[ECX+0x24]
00411890   . |8378 14 10            CMP DWORD PTR DS:[EAX+0x14],0x10
00411894   . |72 02                 JB X7-不问少.00411898
00411896   . |8B00                  MOV EAX,DWORD PTR DS:[EAX]                         ;  取input
00411898   > |8A0410                MOV AL,BYTE PTR DS:[EAX+EDX]                       ;  逐位取input
0041189B   . |8B55 EC               MOV EDX,DWORD PTR SS:[EBP-0x14]
0041189E   . |3002                  XOR BYTE PTR DS:[EDX],AL                           ;  异或
004118A0   . |42                    INC EDX                                            ;  +1
004118A1   . |8955 EC               MOV DWORD PTR SS:[EBP-0x14],EDX
004118A4   . |8955 DC               MOV DWORD PTR SS:[EBP-0x24],EDX
004118A7   .^\EB D1                 JMP X7-不问少.0041187A
004118A9   >  8B41 34               MOV EAX,DWORD PTR DS:[ECX+0x34]

004118B8   .  8B09                  MOV ECX,DWORD PTR DS:[ECX]               ;  input
004118BA   >  8BC6                  MOV EAX,ESI
004118BC   .  33D2                  XOR EDX,EDX
004118BE   .  F775 E4               DIV DWORD PTR SS:[EBP-0x1C]              ;  0x94 % 0x14 = 0x8
004118C1   .  8A040A                MOV AL,BYTE PTR DS:[EDX+ECX]             ;  取input第8位
004118C4   .  8B4D EC               MOV ECX,DWORD PTR SS:[EBP-0x14]
004118C7   .  3001                  XOR BYTE PTR DS:[ECX],AL                 ;  用input第8位解密
004118C9   .  8D4F 60               LEA ECX,DWORD PTR DS:[EDI+0x60]
004118CC   .  BA 9C1A4100           MOV EDX,7-不问少.00411A9C
004118D1   .  8BC2                  MOV EAX,EDX
004118D3   .  2BC1                  SUB EAX,ECX
004118D5   .  83C0 60               ADD EAX,0x60
004118D8   .  0141 01               ADD DWORD PTR DS:[ECX+0x1],EAX
004118DB   .  8D8F 8B000000         LEA ECX,DWORD PTR DS:[EDI+0x8B]
004118E1   .  8B41 01               MOV EAX,DWORD PTR DS:[ECX+0x1]
004118E4   .  2BC1                  SUB EAX,ECX
004118E6   .  81C2 8B000000         ADD EDX,0x8B
004118EC   .  03C2                  ADD EAX,EDX
004118EE   .  8941 01               MOV DWORD PTR DS:[ECX+0x1],EAX           ;  解密
```

## 六、算法破解

1、根据40位校验值按“4的倍数+余数”，求出40个字符`Array4`，然后转`Array3`，再进行20位字符串，逐位穷举，限定合法字符

Python：

```
#!/usr/bin/env python3

# -*- coding: utf-8 -*-

# Ctf.pediy.com ctf2017 crackme 7 keygen

# python 3.6.0 32bit

# written by 爱琴海

# 2017/06/14 

def Main():    
    #跟破文对应，穷举20位合规字符注册码
    Array2 = [0x89,0xBC,0x95,0xFC,0xFB,0xBA,0xED,0x9A,0xBB,0xAE,0xFE,0x99,0xA2,0x98,0xB9,0xF9,0x9F,0x84,0x9C,0xFD,
    0x83,0xAD,0xB6,0xA9,0xA5,0xF5,0x8C,0xA7,0x9E,0x96,0x8A,0xF4,0x85,0xBE,0xA8,0x8F,0x86,0xAF,0x88,0x9D,0x87,0xBF,
    0xFF,0xA1,0x8B,0x81,0xA0,0xAB,0x8E,0xBD,0xB5,0xAA,0x82,0x94,0xA4,0x8D,0xA3,0xF8,0xB4,0xFA,0x9B,0xA6,0xB8,0x80]
    Array3 = [0xA3,0x0B,0x1B,0x82,0x8A,0xBB,0x4A,0x9B,0xA9,0xBB,0x93,0xAA,0xA3,0x6B,0x82,0xAA,0x02,0x4A,0xB2,0x43]
    for n in range (0x0,len(Array3)):
    #穷举20位输入
        for sn in range(0x30,0x7B):
        #字符串范围缩小一些，还可以继续缩小，速度够快，想想没必要了，如果显示结果非字母数字则忽略即可
            sn1 = sn ^ 0xCC
            if sn1 in Array2:
                Start1 = Array2.index(sn1)
                Element_new = Search(Array2,Start1,n)
                if Array3[n] == (((Element_new ^ 0xCC) << 0x3) | ((Element_new ^ 0xCC) >> 0x5)) & 0xFF:
                    print ('第 %d 位' % (n+1),chr(sn))
    return

def Search(Array,Start,Count):
    #Start、Count下标均从0开始，跟上层调用保持一致
    #Count为0-19，也就是数组下标
    #Start为上层匹配Array2的下标
    for i in range(0,Count+1):
        n = Start // 5 + 5
        Start = (Start + n) % len(Array)
    return Array[Start]

if __name__ == '__main__':    
    #调用算法程序
    Main()
```

穷举合法字符注册码如下：第1、3、11、19位只有唯一解，而其他位都存在好几个解，为了进一步筛选注册码，需要分析SMC解码

```
第 1 位 B
第 2 位 j
第 2 位 w
第 3 位 n
第 4 位 d
第 4 位 s
第 5 位 A
第 5 位 Y
第 5 位 l
第 6 位 b
第 6 位 t
第 7 位 P
第 7 位 i
第 8 位 H
第 8 位 e
第 9 位 c
第 9 位 d
第 9 位 s
第 10 位 P
第 10 位 i
第 11 位 y
第 12 位 2
第 12 位 5
第 13 位 0
第 13 位 g
第 13 位 o
第 14 位 1
第 14 位 a
第 15 位 4
第 15 位 7
第 15 位 B
第 16 位 K
第 16 位 r
第 17 位 G
第 17 位 J
第 17 位 X
第 18 位 9
第 18 位 D
第 18 位 I
第 19 位 O
第 20 位 F
第 20 位 k
```

2、根据SMC解码，结合对反汇编代码的认知进行推理，在待选注册码范围中人工筛选出最终答案：（作者你也太狠了吧，呵呵）
