
```
0062155B  |.  33D2          |XOR EDX,EDX                             ;  异或EDX
0062155D  |.  F7F1          |DIV ECX                                 ;  除完取余！！！放入EDX中。
0062155F  |.  8BC2          |MOV EAX,EDX                             ;  把余数放到EAX中。
00621561  |.  33D2          |XOR EDX,EDX                             ;  异或EDX！！！
00621563  |.  290424        |SUB DWORD PTR SS:[ESP],EAX              ;  用预设的075BCD15（123456789）每次减余数。
00621566  |.  195424 04     |SBB DWORD PTR SS:[ESP+4],EDX
0062156A  |.  58            |POP EAX
0062156B  |.  5A            |POP EDX
0062156C  |.  8945 F0       |MOV DWORD PTR SS:[EBP-10],EAX           ;  把EAX的值放到[EBP-10],EAX中。
0062156F  |.  8955 F4       |MOV DWORD PTR SS:[EBP-C],EDX
00621572  |.  4B            |DEC EBX                                 ;  EBX做为循环变量，减1
00621573  |>  8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]           ;  又一次把连接后的机器码字符放到EAX中。
00621576  |.  E8 C538DEFF   |CALL MeiRong.00404E40
0062157B  |.  3BD8          |CMP EBX,EAX                             ;  比较循环变量与连接后机器码的长度
0062157D  |.  7F 04         |JG SHORT MeiRong.00621583
0062157F  |.  85DB          |TEST EBX,EBX
00621581  |.^ 7F B9         \JG SHORT MeiRong.0062153C               ;  跳回重新计算。通过以上的计算，然后转换为十六进制。前面加四个0就是注册码！！！
00621583      8B5E 60       MOV EBX,DWORD PTR DS:[ESI+60]
00621586  |.  85DB          TEST EBX,EBX
00621588  |.  7F 11         JG SHORT MeiRong.0062159B
0062158A  |.  FF75 F4       PUSH DWORD PTR SS:[EBP-C]                ; /Arg2
0062158D  |.  FF75 F0       PUSH DWORD PTR SS:[EBP-10]               ; |Arg1
00621590  |.  8BD7          MOV EDX,EDI                              ; |
00621592  |.  33C0          XOR EAX,EAX                              ; |
00621594  |.  E8 B788DEFF   CALL MeiRong.00409E50                    ; \MeiRong.00409E50
00621599  |.  EB 26         JMP SHORT MeiRong.006215C1               ；这里就出现真正的注册码了！！！
0062159B  |>  FF75 F4       PUSH DWORD PTR SS:[EBP-C]                ; /Arg2
0062159E  |.  FF75 F0       PUSH DWORD PTR SS:[EBP-10]               ; |Arg1
006215A1  |.  8BD7          MOV EDX,EDI                              ; |
006215A3  |.  8BC3          MOV EAX,EBX                              ; |
006215A5      E8 A688DEFF   CALL MeiRong.00409E50
006215AA  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]
006215AC  |.  E8 8F38DEFF   CALL MeiRong.00404E40
006215B1  |.  8BC8          MOV ECX,EAX
006215B3  |.  2B4E 60       SUB ECX,DWORD PTR DS:[ESI+60]
006215B6  |.  8B56 60       MOV EDX,DWORD PTR DS:[ESI+60]
006215B9  |.  42            INC EDX
006215BA  |.  8BC7          MOV EAX,EDI
006215BC  |.  E8 1F3BDEFF   CALL MeiRong.004050E0
006215C1  |>  33C0          XOR EAX,EAX
006215C3  |.  5A            POP EDX
006215C4  |.  59            POP ECX
006215C5  |.  59            POP ECX
006215C6  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
006215C9  |.  68 EB156200   PUSH MeiRong.006215EB
006215CE  |>  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
006215D1  |.  BA 05000000   MOV EDX,5
006215D6  |.  E8 C935DEFF   CALL MeiRong.00404BA4
006215DB  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
006215DE  |.  E8 9D35DEFF   CALL MeiRong.00404B80
006215E3  \.  C3            RETN
```

---

以上结束后就出现明码的注册码！！！

---

## 【破解总结】

1、这个是取的机器码的前六位与一个固定的再连是机器码后六位.

2、先把一个固定的数放到`EAX`中,准备累减.`075BCD15` (`123456789`)

3、从后面取连接后的机器码字符取ASCII值

4、把一个固定数`459(十六)1113`存上.除每一位的ASCII值取余数

6、用`EAX`累加值减去余数. 得到的值以十六进制形式出现,呵呵,前面补上四个`0000`，所谓补`0`其实就是十六进制前面的四个`0`。

## 【版权声明】

技术交流,转载请注明作者！！！！！！！！！！！！！！！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
