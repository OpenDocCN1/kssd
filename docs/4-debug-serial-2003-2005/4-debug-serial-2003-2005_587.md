
# 翻转黑白 2.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6515.htm`](https://www.pediy.com/kssd/pediy06/pediy6515.htm)

一个黑白棋的游戏，反正作者要 10 块注册费，不注册也能玩，但好象有限制。

工具：手术刀（`DEDE`）+ 镊子（`OllyDBG`）

1. 先用 `DEDE` 开刀，在 `Form3` 的 `Button1Click` 里发现一个好玩的东西：

```
0045F7A8   E88BFDFFFF                 call    0045F538
0045F7AD   84C0                       test    al, al
0045F7AF   746F                       jz      0045F820
0045F7B1   6A00                       push    $00
0045F7B3   668B0D84F84500             mov     cx, word ptr [$0045F884]
0045F7BA   B202                       mov     dl, $02
* Possible String Reference to: '注册成功！谢谢您的支持！'
```

`0045F7A8` 的 `call` 就是计算注册码。

2. 用“草原猎豹”做用户名，`DebugYou` 做试练码，用 `OllyDbg` 在 `0045F7A8` 下断，跟进，如下：

```
0045F538  /$  55                    PUSH EBP
0045F539  |.  8BEC                  MOV EBP,ESP
0045F53B  |.  83C4 E0               ADD ESP,-20
0045F53E  |.  53                    PUSH EBX
0045F53F  |.  56                    PUSH ESI
0045F540  |.  57                    PUSH EDI
0045F541  |.  33C9                  XOR ECX,ECX
0045F543  |.  894D E4               MOV DWORD PTR SS:[EBP-1C],ECX
0045F546  |.  894D E0               MOV DWORD PTR SS:[EBP-20],ECX
0045F549  |.  8955 F8               MOV DWORD PTR SS:[EBP-8],EDX
0045F54C  |.  8945 FC               MOV DWORD PTR SS:[EBP-4],EAX
0045F54F  |.  8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]            ;  用户名
0045F552  |.  E8 D146FAFF           CALL Othello2.00403C28
0045F557  |.  8B45 F8               MOV EAX,DWORD PTR SS:[EBP-8]            ;  试练码
0045F55A  |.  E8 C946FAFF           CALL Othello2.00403C28
0045F55F  |.  33C0                  XOR EAX,EAX
0045F561  |.  55                    PUSH EBP
0045F562  |.  68 D9F64500           PUSH Othello2.0045F6D9
0045F567  |.  64:FF30              PUSH DWORD PTR FS:[EAX]
0045F56A  |.  64:8920              MOV DWORD PTR FS:[EAX],ESP
0045F56D  |.  33DB                  XOR EBX,EBX
0045F56F  |.  8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]            ;  用户名
0045F572  |.  E8 FD44FAFF           CALL Othello2.00403A74                  ;  取用户名长度
0045F577  |.  8BF0                  MOV ESI,EAX
0045F579  |.  8B45 F8               MOV EAX,DWORD PTR SS:[EBP-8]            ;  试练码
0045F57C  |.  E8 F344FAFF           CALL Othello2.00403A74                  ;  取试练码长度
0045F581  |.  0FAFF0                IMUL ESI,EAX                            ;  试练码长度*用户名长度
0045F584  |.  85F6                  TEST ESI,ESI      ;结果是 0 就跳去死
0045F586  |.  0F84 25010000         JE Othello2.0045F6B1
0045F58C  |.  C745 F0 00000>        MOV DWORD PTR SS:[EBP-10],0
0045F593  |.  C745 F4 00000>        MOV DWORD PTR SS:[EBP-C],0
0045F59A  |.  C745 E8 00000>        MOV DWORD PTR SS:[EBP-18],0
0045F5A1  |.  C745 EC 00000>        MOV DWORD PTR SS:[EBP-14],0
0045F5A8  |.  8D45 E4               LEA EAX,DWORD PTR SS:[EBP-1C]
0045F5AB  |.  BA F4F64500           MOV EDX,Othello2.0045F6F4               ;  ASCII "HelloCracker"
0045F5B0  |.  E8 DB42FAFF           CALL Othello2.00403890
0045F5B5  |.  8B45 FC               MOV EAX,DWORD PTR SS:[EBP-4]            ;  用户名
0045F5B8  |.  E8 B744FAFF           CALL Othello2.00403A74                  ;  取用户名长度
0045F5BD  |.  8BF0                  MOV ESI,EAX
0045F5BF  |.  85F6                  TEST ESI,ESI
0045F5C1  |.  7E 23                 JLE SHORT Othello2.0045F5E6     ;结果是 0 就跳去死
0045F5C3  |.  BF 01000000           MOV EDI,1
0045F5C8  |>  8B45 FC              /MOV EAX,DWORD PTR SS:[EBP-4]            ;  eax->用户名
0045F5CB  |.  8A4438 FF             |MOV AL,BYTE PTR DS:[EAX+EDI-1]          ;  该循环取得用户名所有字符的 ASCII 码的和
0045F5CF  |.  25 FF000000           |AND EAX,0FF
0045F5D4  |.  33D2                  |XOR EDX,EDX
0045F5D6  |.  0345 F0               |ADD EAX,DWORD PTR SS:[EBP-10]           ;
0045F5D9  |.  1355 F4               |ADC EDX,DWORD PTR SS:[EBP-C]
0045F5DC  |.  8945 F0               |MOV DWORD PTR SS:[EBP-10],EAX           ;  ebp-10 -》用户名所有字符的 ASCII 码的和
0045F5DF  |.  8955 F4               |MOV DWORD PTR SS:[EBP-C],EDX
0045F5E2  |.  47                     |INC EDI
0045F5E3  |.  4E                     |DEC ESI
0045F5E4  |.^ 75 E2                  \JNZ SHORT Othello2.0045F5C8
0045F5E6  |>  6A 00                  PUSH 0
0045F5E8  |.  68 D3070300           PUSH 307D3
0045F5ED  |.  8B45 F0               MOV EAX,DWORD PTR SS:[EBP-10]            ;  用户名所有字符的 ASCII 码的和
0045F5F0  |.  8B55 F4               MOV EDX,DWORD PTR SS:[EBP-C]
0045F5F3  |.  E8 FC60FAFF           CALL Othello2.004056F4    ;  用户名所有字符的 ASCII 码的和 * 198611
0045F5F8  |.  8945 F0               MOV DWORD PTR SS:[EBP-10],EAX
0045F5FB  |.  8955 F4               MOV DWORD PTR SS:[EBP-C],EDX
0045F5FE  |.  8B45 E4               MOV EAX,DWORD PTR SS:[EBP-1C]
0045F601  |.  E8 6E44FAFF           CALL Othello2.00403A74                  ;  取字串'HelloCracker'的长度
0045F606  |.  8BF0                  MOV ESI,EAX
0045F608  |.  85F6                  TEST ESI,ESI      ;结果是 0 就跳去死,神经过敏
0045F60A  |.  7E 23                 JLE SHORT Othello2.0045F62F
0045F60C  |.  BF 01000000           MOV EDI,1
0045F611  |>  8B45 E4              /MOV EAX,DWORD PTR SS:[EBP-1C]
0045F614  |.  8A4438 FF             |MOV AL,BYTE PTR DS:[EAX+EDI-1]
0045F618  |.  25 FF000000           |AND EAX,0FF
0045F61D  |.  33D2                  |XOR EDX,EDX
0045F61F  |.  0345 E8               |ADD EAX,DWORD PTR SS:[EBP-18]
0045F622  |.  1355 EC               |ADC EDX,DWORD PTR SS:[EBP-14]
0045F625  |.  8945 E8               |MOV DWORD PTR SS:[EBP-18],EAX
0045F628  |.  8955 EC               |MOV DWORD PTR SS:[EBP-14],EDX
0045F62B  |.  47                     |INC EDI
0045F62C  |.  4E                     |DEC ESI
0045F62D  |.^ 75 E2                  \JNZ SHORT Othello2.0045F611     ;  该循环取得字串'HelloCracker'所有字符的 ASCII 码的和
0045F62F  |>  6A 00                  PUSH 0
0045F631  |.  68 C6230D00           PUSH 0D23C6
0045F636  |.  8B45 E8               MOV EAX,DWORD PTR SS:[EBP-18]
0045F639  |.  8B55 EC               MOV EDX,DWORD PTR SS:[EBP-14]
0045F63C  |.  E8 B360FAFF           CALL Othello2.004056F4                  ;  'HelloCracker'所有字符的 ASCII 码的和 * 861126
0045F641  |.  8945 E8               MOV DWORD PTR SS:[EBP-18],EAX
0045F644  |.  8955 EC               MOV DWORD PTR SS:[EBP-14],EDX
0045F647  |.  8B45 F0               MOV EAX,DWORD PTR SS:[EBP-10]
0045F64A  |.  8B55 F4               MOV EDX,DWORD PTR SS:[EBP-C]
0045F64D  |.  3345 E8               XOR EAX,DWORD PTR SS:[EBP-18]            ; （'HelloCracker'所有字符的 ASCII 码的和 * 861126）xor (用户名所有字符的 ASCII 码的和 * 198611)
0045F650  |.  3355 EC               XOR EDX,DWORD PTR SS:[EBP-14]
0045F653  |.  8945 F0               MOV DWORD PTR SS:[EBP-10],EAX
0045F656  |.  8955 F4               MOV DWORD PTR SS:[EBP-C],EDX
0045F659  |.  FF75 F4               PUSH DWORD PTR SS:[EBP-C]                ; /Arg2
0045F65C  |.  FF75 F0               PUSH DWORD PTR SS:[EBP-10]               ; |Arg1
0045F65F  |.  8D45 E0               LEA EAX,DWORD PTR SS:[EBP-20]            ; |
0045F662  |.  E8 6181FAFF           CALL Othello2.004077C8                  ; \\根据判断和猜测，应该是 inttostr 函数，把上面运算结果转换成字符串
0045F667  |.  8B45 E0               MOV EAX,DWORD PTR SS:[EBP-20]
0045F66A  |.  E8 0544FAFF           CALL Othello2.00403A74                  ;  得到该字符串长度
0045F66F  |.  8BF0                  MOV ESI,EAX
0045F671  |.  85F6                  TEST ESI,ESI      ;结果是 0 就跳去死
0045F673  |.  7E 3A                 JLE SHORT Othello2.0045F6AF
0045F675  |.  BF 01000000           MOV EDI,1
0045F67A  |>  8D45 E0              /LEA EAX,DWORD PTR SS:[EBP-20]
0045F67D  |.  E8 C245FAFF           |CALL Othello2.00403C44
0045F682  |.  8B55 E0               |MOV EDX,DWORD PTR SS:[EBP-20]
0045F685  |.  0FB6543A FF           |MOVZX EDX,BYTE PTR DS:[EDX+EDI-1]      ;  取上面 xor 计算出来的字符串的每个字符
0045F68A  |.  8B4D E4               |MOV ECX,DWORD PTR SS:[EBP-1C]
0045F68D  |.  0FB64C39 FF           |MOVZX ECX,BYTE PTR DS:[ECX+EDI-1]      ;  取'HelloCracker'的字符
0045F692  |.  03D1                  |ADD EDX,ECX
0045F694  |.  83EA 30               |SUB EDX,30
0045F697  |.  885438 FF             |MOV BYTE PTR DS:[EAX+EDI-1],DL
0045F69B  |.  8B45 E0               |MOV EAX,DWORD PTR SS:[EBP-20]
0045F69E  |.  8A4438 FF             |MOV AL,BYTE PTR DS:[EAX+EDI-1]
0045F6A2  |.  8B55 F8               |MOV EDX,DWORD PTR SS:[EBP-8]
```



0045F6A5  |.  3A443A FF     |CMP AL,BYTE PTR DS:[EDX+EDI-1]  ;试练码每个字符和真注册码每个字符比较

0045F6A9  |.  75 06         |JNZ SHORT Othello2.0045F6B1  ;有哪个字符不等就跳去死

0045F6AB  |.  47          |INC EDI

0045F6AC  |.  4E          |DEC ESI

0045F6AD  |.^ 75 CB        \\JNZ SHORT Othello2.0045F67A  ;该循环把上面 xor 计算出来的字符串的每个字符和'HelloCracker'的每个字符依次相加

0045F6AF  |>  B3 01        MOV BL,1        ;再减去 30H，组成注册码的字符串

0045F6B1  |>  33C0        XOR EAX,EAX         

0045F6B3  |.  5A          POP EDX

0045F6B4  |.  59          POP ECX

0045F6B5  |.  59          POP ECX

0045F6B6  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX

0045F6B9  |.  68 E0F64500   PUSH Othello2.0045F6E0

0045F6BE  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]

0045F6C1  |.  BA 02000000   MOV EDX,2

0045F6C6  |.  E8 5141FAFF   CALL Othello2.0040381C

0045F6CB  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]

0045F6CE  |.  BA 02000000   MOV EDX,2

0045F6D3  |.  E8 4441FAFF   CALL Othello2.0040381C

0045F6D8  \\.  C3          RETN

算法如下：

1. 把用户名每个字符 Ascii 码相加，得到 `sum_username`
2. 把`HelloCracker`的每个字符 Ascii 码相加，得到 `sum_hellocracker`
3. 计算 `IntToStr( (sum_username * 198611) xor (sum_hellocracker * 861126) ![](img/e45fac309594d4f9580d902b0ed40a73.jpg)`，得到字符串 `strTemp`
4. 依次从 `strTemp` 中和`HelloCracker`中取出字符，其 Ascii 码相加，再减去 `30H`，组成注册码，`strTemp` 取完为止。
5. 注册机已写成，把界面修理的 Cool 一点的时候即可发布。

注册码是：`OnuooLvae`

这里面有不少参数和算法好象在绕圈，但好象实际上没有参加注册码的运算？

`call 004056F4` 里的代码，`push` 来 `push` 去，`mul` 来 `mul` 去的，却只是一个简单的两数相乘的运算？

ps:作者的生日好象是 1986 年 11 月 26 日 ;-)

![](img/620448a77d884bd6b2c4a7cfef437501.jpg) 呵呵，最近太忙了，今天才看到大哥们的贴子。

写这个破东东纯粹是胡闹，要是真有人给我寄钱来，我还不知道敢不敢收呢 ![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)

4dd466d0e1e0cc123afbe504da2462ca
