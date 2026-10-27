
```
0056A80C  |.  FF75 EC       |PUSH DWORD PTR SS:[EBP-14]
0056A80F  |.  8D45 E8       |LEA EAX,DWORD PTR SS:[EBP-18]
0056A812  |.  8A16          |MOV DL,BYTE PTR DS:[ESI]                    ;  再取一遍
0056A814  |.  80E2 0F       |AND DL,0F                                   ;  保留后1位数用于查表
0056A817  |.  81E2 FF000000 |AND EDX,0FF
0056A81D  |.  8A92 BCAD5800 |MOV DL,BYTE PTR DS:[EDX+58ADBC]             ;  查表 (0123456789abcdef?)
0056A823  |.  E8 80A7E9FF   |CALL Arial_CD.00404FA8
0056A828  |.  FF75 E8       |PUSH DWORD PTR SS:[EBP-18]
0056A82B  |.  8BC7          |MOV EAX,EDI
0056A82D  |.  BA 03000000   |MOV EDX,3
0056A832  |.  E8 09A9E9FF   |CALL Arial_CD.00405140                      ;  保存查表结果
0056A837  |.  46            |INC ESI
0056A838  |.  FECB          |DEC BL                                      ;  BL 计数器
0056A83A  |.^ 75 B6         \JNZ SHORT Arial_CD.0056A7F2                 ;  循环
0056A83C  |.  33C0          XOR EAX,EAX
0056A83E  |.  5A            POP EDX
0056A83F  |.  59            POP ECX
0056A840  |.  59            POP ECX
0056A841  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
0056A844  |.  68 5EA85600   PUSH Arial_CD.0056A85E
0056A849  |>  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
0056A84C  |.  BA 02000000   MOV EDX,2
0056A851  |.  E8 96A5E9FF   CALL Arial_CD.00404DEC
0056A856  \.  C3            RETN                                         ;  转到 0056A85E
0056A857   .^ E9 D49EE9FF   JMP Arial_CD.00404730
0056A85C   .^ EB EB         JMP SHORT Arial_CD.0056A849
0056A85E   .  5F            POP EDI
0056A85F   .  5E            POP ESI
0056A860   .  5B            POP EBX
0056A861   .  8BE5          MOV ESP,EBP
0056A863   .  5D            POP EBP
0056A864   .  C3            RETN                                         ;  返回到 005879EC
```

```
D5A3B4AE27CEA82776E769AE8F0CCC29 -- 查表 --> d5a3b4ae27cea82776e769ae8f0ccc29
```

终于跟完了，一层一层的`Call`转得头晕,有些地方还没有明白，跟书上讲的一样，搞得这么复杂最后却用明码比较。用《加密与解密》光盘上的`MD5calculator.exe`计算用户名，再将计算结果里的字母由大写改成小写，就可得到注册码。

- 我的用户名：`wzwgp`
- 注册码：`d5a3b4ae27cea82776e769ae8f0ccc29`

注册信息保存在注册表：`HKEY_USERS\S-1-5-21-1123561945-492894223-1060284298-1002\Software\Arial CD Ripper`项下的`username`。

破文写得象老太太的裹脚布，感谢你看完。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
