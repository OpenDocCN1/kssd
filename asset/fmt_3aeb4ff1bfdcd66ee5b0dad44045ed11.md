
```
0072144A  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
0072144D  |.  50            PUSH EAX
0072144E  |.  8B4D 0C       MOV ECX,DWORD PTR SS:[EBP+C]
00721451  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00721454  |.  8BC6          MOV EAX,ESI
00721456  |.  E8 EDFBFFFF   CALL dumped\_.00721048            ;  算法关键 CALL!跟进。。
0072145B  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]     ;  真码出现!
0072145E  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]     ;  假码出现!
00721461  |.  E8 F237CEFF   CALL dumped\_.00404C58            ;  对比 CALL
00721466  |.  75 02         JNZ SHORT dumped\_.0072146A       ;  传说中经典的对比语句!^\_^~!!
```

\-------------------------------------------------------------------------------------------------------

跟进 `00721456`

```
00721048  /$  55            PUSH EBP
00721049  |.  8BEC          MOV EBP,ESP
0072104B  |.  51            PUSH ECX
0072104C  |.  B9 11000000   MOV ECX,11                        ;  ECX=11
00721051  |>  6A 00         /PUSH 0
00721053  |.  6A 00         |PUSH 0
00721055  |.  49            |DEC ECX
00721056  |.^ 75 F9         \\JNZ SHORT dumped\_.00721051
00721058  |.  51            PUSH ECX
00721059  |.  874D FC       XCHG DWORD PTR SS:[EBP-4],ECX     ;  0012EFBC 和 ECX 内容交换
0072105C  |.  53            PUSH EBX
0072105D  |.  56            PUSH ESI
0072105E  |.  894D F8       MOV DWORD PTR SS:[EBP-8],ECX      ;  将 ECX=fcrjzmd(用户名)保存到 0012EFB8 里!
00721061  |.  8955 FC       MOV DWORD PTR SS:[EBP-4],EDX      ;  将 EDX=MRG254KBDBG5BP(机器码)保存到 0012EFBC 里!
00721064  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
00721067  |.  E8 903CCEFF   CALL dumped\_.00404CFC            ;  获取 fcrjzmd 用户名
0072106C  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
0072106F  |.  E8 883CCEFF   CALL dumped\_.00404CFC
00721074  |.  33C0          XOR EAX,EAX                        ;  EAX=0
00721076  |.  55            PUSH EBP
00721077  |.  68 83137200   PUSH dumped\_.00721383
0072107C  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
0072107F  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
00721082  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
00721085  |.  E8 823ACEFF   CALL dumped\_.00404B0C            ;  获取机器码的字符长度!
0072108A  |.  8BF0          MOV ESI,EAX                        ;  ESI=EAX=E(我机器码字符长度)
0072108C  |.  85F6          TEST ESI,ESI                        ;  测试 ESI
0072108E  |.  7E 24         JLE SHORT dumped\_.007210B4       ;  小于或等于则跳转!
00721090  |.  BB 01000000   MOV EBX,1                          ;  EBX=1
00721095  |>  8D55 E8       /LEA EDX,DWORD PTR SS:[EBP-18]     ;  将 0012EFA8 送入 EDX
00721098  |.  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]      ;  EAX=MRG254KBDBG5BP(机器码)
0072109B  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出机器码的 ASCII 码。
007210A0  |.  E8 178BCEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210A5  |.  8B55 E8       |MOV EDX,DWORD PTR SS:[EBP-18]    ;  转换后的 10 进制字符保存到 EDX
007210A8  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]    ;  将 0012EFB0 送入 EAX
007210AB  |.  E8 643ACEFF   |CALL dumped\_.00404B14           ;  这个 CALL 作用就是把转换好的 10 进制字符串合并起来，并保存在 0012EFB0 里!
007210B0  |.  43            |INC EBX                           ;  EBX+1=计数器
007210B1  |.  4E            |DEC ESI                           ;  ESI-1
007210B2  |.^ 75 E1         \\JNZ SHORT dumped\_.00721095     ;  取完为止!
007210B4  |>  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
007210B7  |.  E8 503ACEFF   CALL dumped\_.00404B0C            ;  获取用户名字符长度!
007210BC  |.  8BF0          MOV ESI,EAX                        ;  ESI=EAX=7(用户名长度)
007210BE  |.  85F6          TEST ESI,ESI                        ;  测试 ESI
007210C0  |.  7E 3F         JLE SHORT dumped\_.00721101       ;  小于或等于则跳转!
007210C2  |.  BB 01000000   MOV EBX,1                          ;  EBX=1
007210C7  |>  8D55 E4       /LEA EDX,DWORD PTR SS:[EBP-1C]     ;  将 0012EFA4 送入 EDX
007210CA  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]      ;  EAX=fcrjzmd(用户名)
007210CD  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出用户名的 ASCII 码!
007210D2  |.  E8 E58ACEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210D7  |.  8B55 E4       |MOV EDX,DWORD PTR SS:[EBP-1C]    ;  转换后的 10 进制字符保存到 EDX
007210DA  |.  8D45 F0       |LEA EAX,DWORD PTR SS:[EBP-10]    ;  将 0012EFB0 送入 EAX
007210DD  |.  E8 323ACEFF   |CALL dumped\_.00404B14           ;  这个 CALL 作用就是把上面机器码转换好的 10 进制字符串合并起来，并保存在 0012EFB0 里!
007210E2  |.  8D55 E0       |LEA EDX,DWORD PTR SS:[EBP-20]    ;  将 0012EFA0 送入 EDX
007210E5  |.  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]     ;  EAX=fcrjzmd(用户名)
007210E8  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1];  依次逐个取出用户名的 ASCII 码!
007210ED  |.  E8 CA8ACEFF   |CALL dumped\_.00409BBC           ;  这个 CALL 作用是把上面依次取出 ASCII 码转换成 10 进制!
007210F2  |.  8B55 E0       |MOV EDX,DWORD PTR SS:[EBP-20]    ;  转换后的 10 进制字符保存到 EDX
007210F5  |.  8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]    ;  将 0012EFAC 送入 EAX
007210F8  |.  E8 173ACEFF   |CALL dumped\_.00404B14           ;  ***这个 CALL 作用就是把用户名转换好的 10 进制字符串合并起来，并保存在
007210FD  |.  43            |INC EBX                           ;  EBX+1=计数器
007210FE  |.  4E            |DEC ESI                           ;  ESI-1
007210FF  |.^ 75 C6         \\JNZ SHORT dumped\_.007210C7     ;  取完为止!
00721101  |>  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]      ;  将 0012EFB4 送入 EAX
00721104  |.  50            PUSH EAX
00721105  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]     ;  EAX=778271505352756668667153668010299114106122109100(
00721108  |.  E8 FF39CEFF   CALL dumped\_.00404B0C            ;  获取常数 30
0072110D  |.  8BD0          MOV EDX,EAX                        ;  EDX=EAX=30
0072110F  |.  83EA 0A       SUB EDX,0A                          ;  EDX=EDX-0A=26
00721112  |.  B9 0A000000   MOV ECX,0A                          ;  ECX=0A
00721117  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]     ;  EAX=778271505352756668667153668010299114106122109100(
0072111A  |.  E8 4D3CCEFF   CALL dumped\_.00404D6C            ;  取机器码和用户名 10 进制合并字符串的 0612210910
0072111F  |.  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]      ;  EAX=1
00721122  |.  83E8 01       SUB EAX,1                          ;  EAX=EAX-1;  Switch (cases 0..1)
00721125  |.  72 0B         JB SHORT dumped\_.00721132        ;  小于跳转!
00721127  |.  0F84 D0000000 JE dumped\_.007211FD             ;  相等则跳!
0072112D  |.  E9 33020000   JMP dumped\_.00721365
00721132  |>  68 9C137200   PUSH dumped\_.0072139C            ;  ASCII "JLH-";  Case 0 of switch 00721122
00721137  |.  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
0072113A  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0072113D  |.  8A52 01       MOV DL,BYTE PTR DS:[EDX+1]
00721140  |.  E8 EF38CEFF   CALL dumped\_.00404A34
00721145  |.  FF75 DC       PUSH DWORD PTR SS:[EBP-24]
00721148  |.  8D45 D8       LEA EAX,DWORD PTR SS:[EBP-28]
0072114B  |.  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
0072114E  |.  8A52 03       MOV DL,BYTE PTR DS:[EDX+3]
00721151  |.  E8 DE38CEFF   CALL dumped\_.00404A34
```
