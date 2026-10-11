
## 汇编代码

```asm
0047695E  |.  8A46 06        MOV AL,BYTE PTR DS:[ESI+6]  
00476961  |.  8845 F0        MOV BYTE PTR SS:[EBP-10],AL     ;  保存，上面的四个保存，保存了右 28 位 D0  
00476964  |.  BF 10000000    MOV EDI,10  
00476969  |.  BB 40055C00    MOV EBX,OFFSET <unpacked.移位表>    ;  取移位表地址  
0047696E  |.  8B75 FC        MOV ESI,DWORD PTR SS:[EBP-4]     ;  下面注释中,i 取 1～16  
00476971  |>  8D45 F1        /LEA EAX,DWORD PTR SS:[EBP-F]    ;  左 28 位 C(i-1)  
00476974  |.  8A0B          |MOV CL,BYTE PTR DS:[EBX]        ;  取移位表数据  
00476976  |.  BA 03000000   |MOV EDX,3  
0047697B  |.  E8 00FFFFFF   |CALL unpacked.00476880           ;  C(i-1)左移后生成 Ci  
00476980  |.  8D45 ED       |LEA EAX,DWORD PTR SS:[EBP-13]   ;  右 28 位 D(i-1)  
00476983  |.  8A0B          |MOV CL,BYTE PTR DS:[EBX]        ;  取移位表数据  
00476985  |.  BA 03000000   |MOV EDX,3  
0047698A  |.  E8 F1FEFFFF   |CALL unpacked.00476880           ;  D(i-1)移位生成 Di  
0047698F  |.  8A55 F1       |MOV DL,BYTE PTR SS:[EBP-F]  
00476992  |.  C1E2 04       |SHL EDX,4  
00476995  |.  8A45 F2       |MOV AL,BYTE PTR SS:[EBP-E]  
00476998  |.  33C9          |XOR ECX,ECX  
0047699A  |.  8AC8          |MOV CL,AL  
0047699C  |.  C1E9 04       |SHR ECX,4  
0047699F  |.  0AD1          |OR DL,CL  
004769A1  |.  8855 E6       |MOV BYTE PTR SS:[EBP-1A],DL     ;  保存  
004769A4  |.  C1E0 04       |SHL EAX,4  
004769A7  |.  33D2          |XOR EDX,EDX  
004769A9  |.  8A55 F3       |MOV DL,BYTE PTR SS:[EBP-D]  
004769AC  |.  C1EA 04       |SHR EDX,4  
004769AF  |.  0AC2          |OR AL,DL  
004769B1  |.  8845 E7       |MOV BYTE PTR SS:[EBP-19],AL     ;  保存  
004769B4  |.  8A55 F3       |MOV DL,BYTE PTR SS:[EBP-D]  
004769B7  |.  C1E2 04       |SHL EDX,4  
004769BA  |.  8A45 F4       |MOV AL,BYTE PTR SS:[EBP-C]  
004769BD  |.  33C9          |XOR ECX,ECX  
004769BF  |.  8AC8          |MOV CL,AL  
004769C1  |.  C1E9 04       |SHR ECX,4  
004769C4  |.  0AD1          |OR DL,CL  
004769C6  |.  8855 E8       |MOV BYTE PTR SS:[EBP-18],DL     ;  保存  
004769C9  |.  C1E0 04       |SHL EAX,4  
004769CC  |.  0A45 ED       |OR AL,BYTE PTR SS:[EBP-13]  
004769CF  |.  8845 E9       |MOV BYTE PTR SS:[EBP-17],AL     ;  保存  
004769D2  |.  8A45 EE       |MOV AL,BYTE PTR SS:[EBP-12]  
004769D5  |.  8845 EA       |MOV BYTE PTR SS:[EBP-16],AL     ;  保存  
004769D8  |.  8A45 EF       |MOV AL,BYTE PTR SS:[EBP-11]  
004769DB  |.  8845 EB       |MOV BYTE PTR SS:[EBP-15],AL     ;  保存  
004769DE  |.  8A45 F0       |MOV AL,BYTE PTR SS:[EBP-10]  
004769E1  |.  8845 EC       |MOV BYTE PTR SS:[EBP-14],AL     ;  保存，上面 7 个保存，将 Ci 和 Di 合并成 56 位数据  
004769E4  |.  6A 05          |PUSH 5                      ; /Arg1 = 00000005  
004769E6  |.  8BCE          |MOV ECX,ESI                 ; |  
004769E8  |.  8D45 E6       |LEA EAX,DWORD PTR SS:[EBP-1A]  ; |  
004769EB  |.  BA 06000000   |MOV EDX,6                   ; |  
004769F0  |.  E8 F3FDFFFF   |CALL unpacked.004767E8        ; \\缩位变换 B，将 56 位数决变换成 48 位的 Ki  
004769F5  |.  83C6 06       |ADD ESI,6  
004769F8  |.  43            |INC EBX  
004769F9  |.  4F            |DEC EDI  
004769FA  |.^ 0F85 71FFFFFF \\JNZ unpacked.00476971           ;  循环 16 轮,生成 16 个子密钥  
00476A00  |.  8B7D D8       MOV EDI,DWORD PTR SS:[EBP-28]  
00476A03  |.  8B75 DC       MOV ESI,DWORD PTR SS:[EBP-24]  
00476A06  |.  8B5D E0       MOV EBX,DWORD PTR SS:[EBP-20]  
00476A09  |.  8BE5          MOV ESP,EBP  
00476A0B  |.  5D            POP EBP  
00476A0C  \\.  C2 0400       RETN 4  
```

### 小结

『缩位变换 A』输入 `64` 位密钥，输出 `56` 位数据

```
57 49 41 33 25 17 09  
01 58 50 42 34 26 18  
10 02 59 51 43 35 27  
19 11 03 60 52 44 36  
63 55 47 39 31 23 15  
07 62 54 46 38 30 22  
14 06 61 53 45 37 29  
21 13 05 28 20 12 04
```

`56` 位数据分为两个 `28` 位，进入 `16` 轮循环

移位表：`1 1 2 2 2 2 2 2 1 2 2 2 2 2 2 1`

『缩位变换 B』输入 `56` 位，输出 `48` 位 `Ki`

```
14 17 11 24 01 05  
03 28 15 06 21 10  
23 19 12 04 26 08  
16 07 27 20 13 02  
41 52 31 37 47 55  
30 40 51 45 33 48  
44 49 39 56 34 53  
46 42 50 36 29 32
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

----------------------------------------------------------------------------------------------

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享:)

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

----------------------------------------------------------------------------------------------

文章写于 2005-8-25 17:55:37

`4dd466d0e1e0cc123afbe504da2462ca`
