
# AVI joiner 破解解析

> 原文：[`www.pediy.com/kssd/pediy10/103790.html`](https://www.pediy.com/kssd/pediy10/103790.html)

总的来说，这次破解的难度不大，时间也要的不长，如果说吧学习收集资料的过程都算上的话，也不会超过 `5hours`。说得直接点就是，不会比 `pediy` 的大牛们出的"题目难"。

三天前的下午 4 点左右，也就是 `12/23/2009`，一个同事发给我一个关于破解的文章，我粗略的看了下，就理清楚了其破解的思路。

虽然我看懂了，但我却没有真正实实在在的进行过软件的破解。由于也接触过一些相关的知识，也想看下自己这些年有没有对实际挑战的能力，就进行了一次尝试性的破解工作。（`heihei`，提前透露下：我成功了！）

工欲善其事，必先利其器！不需要有太多的想法，首先要决定的就是用那个调试器（`debugger`）。对于高手们来说，可能还会想下用那个 `debugger` 会更快一些，但对我来说就不存在着个问题了（苦涩的笑笑），拿出不知道什么时候收藏的 `ollydbg1.10` 汉化版。

而后又到 `http://www.ollydbg.de` 上下载最新的 `ollydbg2.02`，这时候可能就有人要问我了，你不都有了 `1.10` 的汉化版，又跑去下载 `2.02` 的英文版做什么？

我在这里解答下：那是因为我压根就没有打算用 `1.10` 来对 `AVIjoiner` 进行破解，原因有二，我希望我的工具尽可能的犀利。另一个原因是，虽然我不在意中文版还是英文版，但我还是会先用下中文版，以求对整个软件有个个基本的认识（好久没用，都忘了）。

在粗略的跑了跑 `1.10`，对软件的基本操作有个大概的了解后，我就启动了 `ollydbg2.02` 来对 `AVIjoiner` 的破解。

## step 1

在注册的窗口中随意的输入 `username` 和 `registration code`，如：

`username`: `numax`
`registration code`: `123456`

在经过四五次的组合后，我得出了一个最基本的结论：

1. `username` 的长度必须 >= 2
2. `registration code` 的长度必须 >= 8

呵呵，至此，就可以开始正式的调试破解了。

## step 2

在 `step 1` 中，我们可以看到有这样类似的错误：

```
Please input correct User Name
```

或者

```
Please input correct Registration Code!
```

![`bbs.pediy.com/picture.php?albumid=143&pictureid=339`](http://bbs.pediy.com/picture.php?albumid=143&pictureid=339)

搜寻类似的字符，并在此处这两处下了断点再说。（笨法子，没什么经验也不想动脑筋。比这更让我无奈的是，就算我动脑筋，我也想不出什么更好的法子来。）

好了，在 `ollydbg` 中加载 `AVIjoiner` 后，

在 `username` 中输入：`numax`，

在 `registration code` 输入：`12345678`

### 代码分析

`CPU Disasm`

```
Address   Hex dump           Command                                      Comments

00405C70  / .  83EC 0C       SUB ESP,0C
00405C73  | .  55            PUSH EBP
00405C74  | .  56            PUSH ESI
00405C75  | .  57            PUSH EDI
00405C76  | .  BF 01000000   MOV EDI,1
00405C7B  | .  57            PUSH EDI                  ;  /Arg1 => 1
00405C7C  | .  8BF1          MOV ESI,ECX               ;  |
00405C7E  | .  E8 04A00100   CALL 0041FC87             ;  \AVIJoiner.0041FC87
00405C83  | .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00405C86  | .  8B68 F4       MOV EBP,DWORD PTR DS:[EAX-0C]
00405C89  | .  83FD 02       CMP EBP,2
00405C8C  | .  7D 15         JGE SHORT 00405CA3
00405C8E  | .  6A 00         PUSH 0                    ;  /Arg3 = 0
00405C90  | .  6A 00         PUSH 0                    ;  |Arg2 = 0
00405C92  | .  68 E4D24200   PUSH OFFSET AVIJoiner.0042D2E4    ;  |Arg1 = ASCII "Please input correct User Name!"
00405C97  | .  E8 B4FD0100   CALL 00425A50             ;  \AVIJoiner.00425A50
00405C9C  | .  5F            POP EDI
00405C9D  | .  5E            POP ESI
00405C9E  | .  5D            POP EBP
00405C9F  | .  83C4 0C       ADD ESP,0C
00405CA2  | .  C3            RETN
00405CA3  |>  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]
00405CA6  | .  8379 F4 08    CMP DWORD PTR DS:[ECX-0C],8
00405CAA  | .  7D 15         JGE SHORT 00405CC1
00405CAC  | .  6A 00         PUSH 0                    ;  /Arg3 = 0
00405CAE  | .  6A 00         PUSH 0                    ;  |Arg2 = 0
00405CB0  | .  68 BCD24200   PUSH OFFSET AVIJoiner.0042D2BC    ;  |Arg1 = ASCII "Please input correct Registration Code!"
00405CB5  | .  E8 96FD0100   CALL 00425A50             ;  \AVIJoiner.00425A50
00405CBA  | .  5F            POP EDI
00405CBB  | .  5E            POP ESI
00405CBC  | .  5D            POP EBP
00405CBD  | .  83C4 0C       ADD ESP,0C
00405CC0  | .  C3            RETN
00405CC1  |>  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]          ; ASCII "numax"
00405CC4  | .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-0C]          ; numax: [eax-0c],the length of username
00405CC7  | .  85C9          TEST ECX,ECX                          ; numax: check the length of username
00405CC9  | .  7D 0A         JGE SHORT 00405CD5
00405CCB  | .  68 57000780   PUSH 80070057
00405CD0  | .  E8 5BB7FFFF   CALL 00401430
00405CD5  |>  8A10          MOV DL,BYTE PTR DS:[EAX]              ; numax: first character 'n'
00405CD7  | .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00405CDA  | .  3978 F4       CMP DWORD PTR DS:[EAX-0C],EDI
00405CDD  | .  7D 0A         JGE SHORT 00405CE9
00405CDF  | .  68 57000780   PUSH 80070057
00405CE4  | .  E8 47B7FFFF   CALL 00401430
00405CE9  |>  8A40 01       MOV AL,BYTE PTR DS:[EAX+1]            ; numax: second character 'u'
00405CEC  | .  884424 0E     MOV BYTE PTR SS:[ESP+0E],AL
00405CF0  | .  8B46 70       MOV EAX,DWORD PTR DS:[ESI+70]
00405CF3  | .  8B48 F4       MOV ECX,DWORD PTR DS:[EAX-0C]
00405CF6  | .  85C9          TEST ECX,ECX
00405CF8  | .  7D 0A         JGE SHORT 00405D04
00405CFA  | .  68 57000780   PUSH 80070057
00405CFF  | .  E8 2CB7FFFF   CALL 00401430
00405D04  |>  8B4E 70       MOV ECX,DWORD PTR DS:[ESI+70]
00405D07  | .  53            PUSH EBX
00405D08  | .  8A18          MOV BL,BYTE PTR DS:[EAX]
00405D0A  | .  3979 F4       CMP DWORD PTR DS:[ECX-0C],EDI
00405D0D  | .  7D 0A         JGE SHORT 00405D19
00405D0F  | .  68 57000780   PUSH 80070057
00405D14  | .  E8 17B7FFFF   CALL 00401430
00405D19  |>  0FB6C2        MOVZX EAX,DL                     ; numax: start address, simple calculating related to username
00405D1C  | .  83C8 41       OR EAX,00000041
00405D1F  | .  99            CDQ
00405D20  | .  BF 0A000000   MOV EDI,0A
00405D25  | .  F7FF          IDIV EDI
00405D27  | .  0FB64424 12   MOVZX EAX,BYTE PTR SS:[ESP+12]
00405D2C  | .  83C8 56       OR EAX,00000056
00405D2F  | .  885424 16     MOV BYTE PTR SS:[ESP+16],DL
00405D33  | .  99            CDQ
00405D34  | .  F7FF          IDIV EDI
00405D36  | .  0FB6C3        MOVZX EAX,BL
00405D39  | .  83C8 49       OR EAX,00000049
00405D3C  | .  885424 12     MOV BYTE PTR SS:[ESP+12],DL
00405D40  | .  99            CDQ
00405D41  | .  F7FF          IDIV EDI
00405D43  | .  0FB641 01     MOVZX EAX,BYTE PTR DS:[ECX+1]
00405D47  | .  83C8 4A       OR EAX,0000004A
00405D4A  | .  8BCF          MOV ECX,EDI
00405D4C  | .  885424 17     MOV BYTE PTR SS:[ESP+17],DL
00405D50  | .  99            CDQ
00405D51  | .  F7F9          IDIV ECX
00405D53  | .  33C0          XOR EAX,EAX
00405D55  | .  33C9          XOR ECX,ECX
00405D57  | .  85ED          TEST EBP,EBP
00405D59  | .  885424 18     MOV BYTE PTR SS:[ESP+18],DL
00405D5D  | .  7E 20         JLE SHORT 00405D7F
00405D5F  | .  90            NOP
00405D60  |>  85C9          /TEST ECX,ECX
00405D62  | .  0F8C D2000000 |JL 00405E3A
00405D68  | .  8B7E 70       |MOV EDI,DWORD PTR DS:[ESI+70]
00405D6B  | .  3B4F F4       |CMP ECX,DWORD PTR DS:[EDI-0C]
00405D6E  | .  0F8F C6000000 |JG 00405E3A
00405D74  | .  0FB6140F      |MOVZX EDX,BYTE PTR DS:[ECX+EDI]
00405D78  | .  03C2          |ADD EAX,EDX
00405D7A  | .  41            |INC ECX
00405D7B  | .  3BCD          |CMP ECX,EBP
00405D7D  | .^ 7C E1         \JL SHORT 00405D60
00405D7F  |>  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]
00405D82  | .  8B51 F4       MOV EDX,DWORD PTR DS:[ECX-0C]
00405D85  | .  85D2          TEST EDX,EDX                   ; numax: end address, simple calculating related to username
00405D87  | .  7D 0A         JGE SHORT 00405D93
00405D89  | .  68 57000780   PUSH 80070057
00405D8E  | .  E8 9DB6FFFF   CALL 00401430
00405D93  |>  8A11          MOV DL,BYTE PTR DS:[ECX]                 ; numax: end address, simple calculating related to regcode
00405D95  | .  8B4E 74       MOV ECX,DWORD PTR DS:[ESI+74]
00405D98  | .  8379 F4 01    CMP DWORD PTR DS:[ECX-0C],1
00405D9C  | .  885424 19     MOV BYTE PTR SS:[ESP+19],DL
00405DA0  | .  7D 0A         JGE SHORT 00405DAC
00405DA2  | .  68 57000780   PUSH 80070057
00405DA7  | .  E8 84B6FFFF   CALL 00401430
00405DAC  |>  8A49 01       MOV CL,BYTE PTR DS:[ECX+1]
```



```
00405DAF  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405DB2  |.  884C24 13     MOV BYTE PTR SS:[ESP+13],CL  

00405DB6  |.  837F F4 02    CMP DWORD PTR DS:[EDI-0C],2  

00405DBA  |.  7D 0A         JGE SHORT 00405DC6  

00405DBC  |.  68 57000780   PUSH 80070057  

00405DC1  |.  E8 6AB6FFFF   CALL 00401430  

00405DC6  |>  8A4F 02       MOV CL,BYTE PTR DS:[EDI+2]  

00405DC9  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405DCC  |.  884C24 14     MOV BYTE PTR SS:[ESP+14],CL  

00405DD0  |.  837F F4 03    CMP DWORD PTR DS:[EDI-0C],3  

00405DD4  |.  7D 0A         JGE SHORT 00405DE0  

00405DD6  |.  68 57000780   PUSH 80070057  

00405DDB  |.  E8 50B6FFFF   CALL 00401430  

00405DE0  |>  8A4F 03       MOV CL,BYTE PTR DS:[EDI+3]  

00405DE3  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405DE6  |.  884C24 15     MOV BYTE PTR SS:[ESP+15],CL  

00405DEA  |.  837F F4 04    CMP DWORD PTR DS:[EDI-0C],4  

00405DEE  |.  7D 0A         JGE SHORT 00405DFA  

00405DF0  |.  68 57000780   PUSH 80070057  

00405DF5  |.  E8 36B6FFFF   CALL 00401430  

00405DFA  |>  8A4F 04       MOV CL,BYTE PTR DS:[EDI+4]  

00405DFD  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405E00  |.  837F F4 05    CMP DWORD PTR DS:[EDI-0C],5  

00405E04  |.  7D 0A         JGE SHORT 00405E10  

00405E06  |.  68 57000780   PUSH 80070057  

00405E0B  |.  E8 20B6FFFF   CALL 00401430  

00405E10  |>  8A5F 05       MOV BL,BYTE PTR DS:[EDI+5]  

00405E13  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405E16  |.  885C24 1A     MOV BYTE PTR SS:[ESP+1A],BL  

00405E1A  |.  837F F4 06    CMP DWORD PTR DS:[EDI-0C],6  

00405E1E  |.  7D 0A         JGE SHORT 00405E2A  

00405E20  |.  68 57000780   PUSH 80070057  

00405E25  |.  E8 06B6FFFF   CALL 00401430  

00405E2A  |>  8A5F 06       MOV BL,BYTE PTR DS:[EDI+6]  

00405E2D  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405E30  |.  885C24 1B     MOV BYTE PTR SS:[ESP+1B],BL  

00405E34  |.  837F F4 07    CMP DWORD PTR DS:[EDI-0C],7  

00405E38  |.  7D 0A         JGE SHORT 00405E44                ; numax: end address, simple calculating related to regcode  

00405E3A  |>  68 57000780   PUSH 80070057  

00405E3F  |.  E8 ECB5FFFF   CALL 00401430  

00405E44  |>  8A5F 07       MOV BL,BYTE PTR DS:[EDI+7]        ; numax: start address, checking regcode and username  

00405E47  |.  0FB67C24 16   MOVZX EDI,BYTE PTR SS:[ESP+16]  

00405E4C  |.  0FB6D2        MOVZX EDX,DL  

00405E4F  |.  83EA 30       SUB EDX,30  

00405E52  |.  3BFA          CMP EDI,EDX                         ; edi, regcode first character         

00405E54  |.  75 48         JNE SHORT 00405E9E  

00405E56  |.  0FB65424 13   MOVZX EDX,BYTE PTR SS:[ESP+13]   

00405E5B  |.  0FB67C24 12   MOVZX EDI,BYTE PTR SS:[ESP+12]  

00405E60  |.  83EA 30       SUB EDX,30  

00405E63  |.  3BFA          CMP EDI,EDX                         ; edi, regcode second character       

00405E65  |.  75 37         JNE SHORT 00405E9E  

00405E67  |.  0FB65424 14   MOVZX EDX,BYTE PTR SS:[ESP+14]  

00405E6C  |.  0FB67C24 17   MOVZX EDI,BYTE PTR SS:[ESP+17]  

00405E71  |.  83EA 30       SUB EDX,30  

00405E74  |.  3BFA          CMP EDI,EDX                         ; edi, regcode third character  

00405E76  |.  75 26         JNE SHORT 00405E9E  

00405E78  |.  0FB65424 15   MOVZX EDX,BYTE PTR SS:[ESP+15]  

00405E7D  |.  0FB67C24 18   MOVZX EDI,BYTE PTR SS:[ESP+18]  

00405E82  |.  83EA 30       SUB EDX,30  

00405E85  |.  3BFA          CMP EDI,EDX                         ; edi, regcode fourth character  

00405E87  |.  75 15         JNE SHORT 00405E9E   

00405E89  |.  99            CDQ  

00405E8A  |.  BF 0A000000   MOV EDI,0A  

00405E8F  |.  F7FF          IDIV EDI  

00405E91  |.  0FB6C2        MOVZX EAX,DL  

00405E94  |.  0FB6D1        MOVZX EDX,CL  

00405E97  |.  83EA 30       SUB EDX,30                          ; edx, regcode fifth character  

00405E9A  |.  3BC2          CMP EAX,EDX                         ; numax: end address, checking regcode and username  

00405E9C  |.  74 38         JE SHORT 00405ED6  

00405E9E  |>  807C24 19 39  CMP BYTE PTR SS:[ESP+19],39         ; numax: built-in regcode: "93023147"  

00405EA3  |.  0F85 83000000 JNE 00405F2C  

00405EA9  |.  807C24 13 33  CMP BYTE PTR SS:[ESP+13],33  

00405EAE  |.  75 7C         JNE SHORT 00405F2C  

00405EB0  |.  807C24 14 30  CMP BYTE PTR SS:[ESP+14],30  

00405EB5  |.  75 75         JNE SHORT 00405F2C  

00405EB7  |.  807C24 15 32  CMP BYTE PTR SS:[ESP+15],32  

00405EBC  |.  75 6E         JNE SHORT 00405F2C  

00405EBE  |.  80F9 33       CMP CL,33  

00405EC1  |.  75 69         JNE SHORT 00405F2C  

00405EC3  |.  807C24 1A 31  CMP BYTE PTR SS:[ESP+1A],31  

00405EC8  |.  75 62         JNE SHORT 00405F2C  

00405ECA  |.  807C24 1B 34  CMP BYTE PTR SS:[ESP+1B],34  

00405ECF  |.  75 5B         JNE SHORT 00405F2C  

00405ED1  |.  80FB 37       CMP BL,37                             ;  

00405ED4  |.  75 56         JNE SHORT 00405F2C  

00405ED6  |>  6A 00         PUSH 0                                ; /Arg3 = 0  

00405ED8  |.  6A 00         PUSH 0                                ; |Arg2 = 0  

00405EDA  |.  68 A0D24200   PUSH OFFSET AVIJoiner.0042D2A0        ; |Arg1 = ASCII "Registration has succeeded!"  

00405EDF  |.  E8 6CFB0100   CALL 00425A50                         ; \\AVIJoiner.00425A50  

00405EE4  |.  8B7E 70       MOV EDI,DWORD PTR DS:[ESI+70]  

00405EE7  |.  E8 49210200   CALL 00428035                         ; [AVIJoiner.00428035  

00405EEC  |.  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]  

00405EEF  |.  57            PUSH EDI                                ; /Arg3  

00405EF0  |.  68 2CCF4200   PUSH OFFSET AVIJoiner.0042CF2C        ; |Arg2 = ASCII "username"  

00405EF5  |.  68 24CF4200   PUSH OFFSET AVIJoiner.0042CF24        ; |Arg1 = ASCII "Option"  

00405EFA  |.  8BC8          MOV ECX,EAX                             ; |  

00405EFC  |.  E8 8FFC0100   CALL 00425B90                         ; \\AVIJoiner.00425B90  

00405F01  |.  8B7E 74       MOV EDI,DWORD PTR DS:[ESI+74]  

00405F04  |.  E8 2C210200   CALL 00428035                         ; [AVIJoiner.00428035  

00405F09  |.  8B40 04       MOV EAX,DWORD PTR DS:[EAX+4]  

00405F0C  |.  57            PUSH EDI                                ; /Arg3  

00405F0D  |.  68 10CF4200   PUSH OFFSET AVIJoiner.0042CF10        ; |Arg2 = ASCII "registration\_code"  

00405F12  |.  68 24CF4200   PUSH OFFSET AVIJoiner.0042CF24        ; |Arg1 = ASCII "Option"  

00405F17  |.  8BC8          MOV ECX,EAX                             ; |  

00405F19  |.  E8 72FC0100   CALL 00425B90                         ; \\AVIJoiner.00425B90  

00405F1E  |.  5B            POP EBX  

00405F1F  |.  5F            POP EDI  

00405F20  |.  8BCE          MOV ECX,ESI  

00405F22  |.  5E            POP ESI  

00405F23  |.  5D            POP EBP  

00405F24  |.  83C4 0C       ADD ESP,0C  

00405F27  |.  E9 53870100   JMP 0041E67F  

00405F2C  |>  6A 00         PUSH 0                                ; /Arg3 = 0  

00405F2E  |.  6A 00         PUSH 0                                ; |Arg2 = 0  

00405F30  |.  68 88D24200   PUSH OFFSET AVIJoiner.0042D288        ; |Arg1 = ASCII "Registration failed!"  

00405F35  |.  E8 16FB0100   CALL 00425A50                         ; \\AVIJoiner.00425A50  

00405F3A  |.  5B            POP EBX  

00405F3B  |.  5F            POP EDI  

00405F3C  |.  5E            POP ESI  

00405F3D  |.  5D            POP EBP  

00405F3E  |.  83C4 0C       ADD ESP,0C  

00405F41  \\.  C3            RETN  
```

我简单的用文字描述下这段代码的意思，并按照功能不同将其分为以下几个部分：

1. 简单检查 `username`，`registration code`：
   - 如果 `username` 的长度小于 2，将提示：`Please input correct User Name`
   - 如果 `registration code` 的长度小于 8，将提示：`Please input correct Registration Code`

2. 基于 `username` 进行计算，代码中我用



`numax`: start address, simple calculating related to `username`

`numax`: end address, simple calculating related to `username`

进行了明确的标明.

仔细阅读下这段代码，我发现大多都是进行除 10 取余的计算，并保存余数.在该段的结束部分，由计算了 `username` 字串 ascii 的码的总和.

3: 基于 `registration` 进行计算,代码中我用

`numax`: start address, simple calculating related to `regcode`

`numax`: end address, simple calculating related to `regcode`

进行了明确的标明.

个人认为，这段基本上就是废码，将以个个字符存入栈后在比较 `registration code` 的长度。客观地来说，十分类似花指令.

4: 基于第二段所得的余数及 `username` 字串 ascii 的码的和,与 `registration code` 进行比较.共五伦，如果都相等，认为 `username` 与 `registration code` 有效.

5：注册成功或失败的提示框.

`step 3`:

获得注册码.

对于想研究算法的人来说呢，可以粗略的看下 `step 2` 种提到的代码的第二部分:“基于 `username` 进行计算"，但对于破解的人来说嘛，呵呵，只要找到,`step 2` 种提到的第四部分就好了.

![`bbs.pediy.com/picture.php?albumid=143&pictureid=340`](http://bbs.pediy.com/picture.php?albumid=143&pictureid=340)

这次成功的破解对我来说可是意义重大,以前所学虽然没有得到全面的认证，但却至少也从一点反应出我曾经的努力并没有白费.
