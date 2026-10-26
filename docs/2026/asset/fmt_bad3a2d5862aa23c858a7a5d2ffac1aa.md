
# Encryptpe v22005314service脱壳示例

这篇文章答应某人N久了，今天趁老板出差了，偷点时间写下来，但愿某人今天已经搞定了![](https://bbs.kanxue.com/view/img/face/27.gif)，考虑到某些原因因此没有选择普通加密方式的脱壳做为文章。

[附件:EPEv22005314.rar](https://bbs.kanxue.com/bbs.pediy.com/upload/2005/8/files/epev22005314.rar)

**【目标】**：win98’s notepad  
**【工具】**：Olydbg1.1(diy版)、LORDPE、ImportREC1.6F  
**【任务】**：去除注册信息，去除目标壳  
**【操作平台】**：Windows 2003 server  
**【作者】**：LOVEBOOM[DFCG][FCG][US]  
**【相关链接】**：ceeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2F1j5%4u0&6M7s2c8H3k6g2)9J5k6h3y4G2L8b7`.`.

**【简要说明】**：这篇文章想了很久，也说了很久，今天还是写下来了，这篇文章很久以前就答应过某人，给他看看，拖了这么久，真是不好意思，但愿他看到这篇文章的时候已经搞定了这个壳:-)。这个壳对现在很多人来说是比较简单的了，因此，我就不太细写了，如果你是那种基本的东西或基本的OD调试都不会的初学朋友，建议你们先自己看看基础知识，再学脱壳，我不希望，也不建议任何新人不会走就学跑。

## 详细过程

先设置下OD：忽略全部异常，隐藏你的OD。先运行一次目标程序，用进程工具杀掉程序，然后用OD载入目标进入EP处：

```
0040D000 >  60               PUSHAD                                   ; EP
0040D001    9C               PUSHFD
0040D002    64:FF35 00000000 PUSH DWORD PTR FS:[0]
```

入壳处和2003的是一样的。

现在按F9运行几秒钟，然后按F12暂停，暂停后打开memory窗口，在下面位置下断：

下断的位置处就可以很直接的分出epe的版本。下断后，F9继续运行。这样很快在下面停下：

```
71236732    46               INC ESI
71236733    8807             MOV BYTE PTR DS:[EDI],AL                 ; 继续运行后在这里断下
71236735    47               INC EDI
```

停下后，直接向下找到跳去OEP处，因为壳是用`upx`加的，所以比较好找:-)。

```
712368B7    61               POPAD
712368B8  ^ E9 23EFFAFF      JMP 711E57E0                             ; 这里跳去oep
```

找到地址后，直接F4运行到`712368b3`处。再按F8一次：

```
711E57E0    55               PUSH EBP                                 ; 壳代码OEP
711E57E1    8BEC             MOV EBP,ESP
711E57E3    83C4 C4          ADD ESP,-3C
```

到这里观察一下就可以看出壳是用delphi写的，注意这里是壳代码的OEP不是目标程序的OEP。到了OEP后，用OD的查找功能查找如下命令：

```
MOV WORD PTR DS:[EAX],25FF
```

可以找到下以两个地方：

```
711E4FEB    66:C700 FF25     MOV WORD PTR DS:[EAX],25FF               ; 查找点1
711E4FF0    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]
……
711E502A    66:C700 FF25     MOV WORD PTR DS:[EAX],25FF               ; 查找点2
711E502F    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]
711E5032    8950 02          MOV DWORD PTR DS:[EAX+2],EDX
```

找到后先在`711e4feb`处下内存访问断点，现在查找命令集：

```
XOR ECX,DWORD PTR SS:[EBP-4]
MOV DWORD PTR DS:[EAX],ECX
MOV DWORD PTR SS:[EBP-8],EDX
```

找到以下位置：

```
711E37C5    334D FC          XOR ECX,DWORD PTR SS:[EBP-4]             ; 这里IAT填充＊＊＊＊
711E37C8    8908             MOV DWORD PTR DS:[EAX],ECX
711E37CA    8955 F8          MOV DWORD PTR SS:[EBP-8],EDX
711E37CD    85FF             TEST EDI,EDI
711E37CF    74 29            JE SHORT 711E37FA
```

找到后在`711E37C5`处也下断内存访问断点。好了，现在按F9运行，停一会，发现以下两个地方会访问以上两个地方，也就是CRC　检测处：

```
711E3870  /$  53             PUSH EBX          CRC检测一
711E3871  |.  56             PUSH ESI
711E3872  |.  33F6           XOR ESI,ESI
711E3874  |.  8B15 D4F61D71  MOV EDX,DWORD PTR DS:[711DF6D4]          ;  V2200531.711DF720
711E387A  |.  EB 09          JMP SHORT 711E3885
711E387C  |>  33DB           /XOR EBX,EBX
711E387E  |.  8A1A           |MOV BL,BYTE PTR DS:[EDX]                ;  这里中断
711E3880  |.  03F3           |ADD ESI,EBX
711E3882  |.  41             |INC ECX
711E3883  |.  8BD1           |MOV EDX,ECX
711E3885  |>  BB 70381E71     MOV EBX,711E3870                        ;  Entry address
711E388A  |.  8BCA           |MOV ECX,EDX
711E388C  |.  3BD9           |CMP EBX,ECX
711E388E  |.^ 77 EC          \JA SHORT 711E387C
711E3890  |.  84C0           TEST AL,AL
711E3892  |.  74 07          JE SHORT 711E389B
711E3894  |.  A1 88DB1F71    MOV EAX,DWORD PTR DS:[711FDB88]
711E3899  |.  8930           MOV DWORD PTR DS:[EAX],ESI
711E389B  |>  A1 88DB1F71    MOV EAX,DWORD PTR DS:[711FDB88]
711E38A0  |.  3B30           CMP ESI,DWORD PTR DS:[EAX]
711E38A2  |.  0F95C0         SETNE AL                                 ;  修改这里＊＊＊＊
711E38A5  |.  5E             POP ESI
711E38A6  |.  5B             POP EBX
711E38A7  \.  C3             RETN
……

711E540C  /$  53             PUSH EBX                                 ;  CRC检测点二
711E540D  |.  56             PUSH ESI
711E540E  |.  57             PUSH EDI
711E540F  |.  8BD8           MOV EBX,EAX
711E5411  |.  33F6           XOR ESI,ESI
711E5413  |.  BF 343B1E71    MOV EDI,EncryptPE_Init
711E5418  |.  EB 24          JMP SHORT 711E543E
711E541A  |>  84DB           /TEST BL,BL
711E541C  |.  74 19          |JE SHORT 711E5437
711E541E  |.  81FF 343B1E71  |CMP EDI,EncryptPE_Init
711E5424  |.  75 11          |JNZ SHORT 711E5437
711E5426  |.  E8 D9FEFFFF    |CALL 711E5304
711E542B  |.  84C0           |TEST AL,AL
711E542D  |.  75 08          |JNZ SHORT 711E5437
711E542F  |.  A1 9CDB1F71    |MOV EAX,DWORD PTR DS:[711FDB9C]
711E5434  |.  C600 01        |MOV BYTE PTR DS:[EAX],1
711E5437  |>  33C0           |XOR EAX,EAX
711E5439  |.  8A07           |MOV AL,BYTE PTR DS:[EDI]                ;  这里一次访问中断
711E543B  |.  03F0           |ADD ESI,EAX
711E543D  |.  47             |INC EDI
711E543E  |>  B8 0C541E71     MOV EAX,711E540C                        ;  Entry address
711E5443  |.  3BC7           |CMP EAX,EDI
711E5445  |.^ 77 D3          \JA SHORT 711E541A
711E5447  |.  84DB           TEST BL,BL
711E5449  |.  74 07          JE SHORT 711E5452
711E544B  |.  A1 0CDF1F71    MOV EAX,DWORD PTR DS:[711FDF0C]
711E5450  |.  8930           MOV DWORD PTR DS:[EAX],ESI
711E5452  |>  A1 0CDF1F71    MOV EAX,DWORD PTR DS:[711FDF0C]
711E5457  |.  3B30           CMP ESI,DWORD PTR DS:[EAX]
711E5459  |.  0F95C0         SETNE AL                                 ;  修改这里＊＊＊＊
711E545C  |.  5F             POP EDI
711E545D  |.  5E             POP ESI
711E545E  |.  5B             POP EBX
711E545F  \.  C3             RETN
```

把那两个CRC检测处改为`mov al,0`就可以跳过检测。修改好那两个check crc点后，直接在一个crc处理处设置断点：

```
711E5459    B0 00            MOV AL,0      ;我这里下的断
711E545B    90               NOP
711E545C    5F               POP EDI
```

断下后的CTRL＋F9返回：

```
711E4E6B    33C0             XOR EAX,EAX
711E4E6D    E8 9A050000      CALL 711E540C
711E4E72    84C0             TEST AL,AL                               ; 返回到这里
711E4E74    75 0B            JNZ SHORT 711E4E81                       ; 这里不能跳的，因为我里面已经改成mov al,0所以这里不会跳的　
711E4E76    33C0             XOR EAX,EAX
711E4E78    E8 2FBBFCFF      CALL 711B09AC
711E4E7D    84C0             TEST AL,AL
711E4E7F    74 08            JE SHORT 711E4E89
```

经过踪发现壳的处理，也算是这个壳里最为精彩的片段了：

```
711E36B5    334D FC         XOR ECX,DWORD PTR SS:[EBP-4]
IAT处理:
711E36B8    8908            MOV DWORD PTR DS:[EAX],ECX
711E36BA    8955 F8         MOV DWORD PTR SS:[EBP-8],EDX
711E36BD    85FF            TEST EDI,EDI
711E36BF    74 29           JE SHORT 711E36EA
```

修改成：

```
711E36B5    8B4D FC         MOV ECX,DWORD PTR SS:[EBP-4]         ;改为传入正确的API
711E36B8    8908            MOV DWORD PTR DS:[EAX],ECX
711E36BA    894D F8         MOV DWORD PTR SS:[EBP-8],ECX    ;改为传入正确的API
……
711E4FEB    66:C700 FF25     MOV WORD PTR DS:[EAX],25FF               ; 这里要patch＊＊＊＊
711E4FF0    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]
711E4FF3    8950 02          MOV DWORD PTR DS:[EAX+2],EDX
711E4FF6    8BD0             MOV EDX,EAX
711E4FF8    8B5D E8          MOV EBX,DWORD PTR SS:[EBP-18]
711E4FFB    83C3 06          ADD EBX,6
711E4FFE    8B45 E0          MOV EAX,DWORD PTR SS:[EBP-20]
711E5001    03C0             ADD EAX,EAX
711E5003    03D8             ADD EBX,EAX
711E5005    8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]
711E5008    8B00             MOV EAX,DWORD PTR DS:[EAX]
711E500A    0FB70B           MOVZX ECX,WORD PTR DS:[EBX]
711E500D    81E9 00300000    SUB ECX,3000
711E5013    03C1             ADD EAX,ECX
711E5015    0345 DC          ADD EAX,DWORD PTR SS:[EBP-24]
711E5018    48               DEC EAX
711E5019    2BD0             SUB EDX,EAX
711E501B    83EA 04          SUB EDX,4
711E501E    8910             MOV DWORD PTR DS:[EAX],EDX               ; 这里壳写入加密后的地址patch＊＊＊＊
711E5020    E9 68010000      JMP 711E518D
711E5025    E8 EA7DFCFF      CALL 711ACE14
711E502A    66:C700 FF25     MOV WORD PTR DS:[EAX],25FF               ; 这里同上＊＊＊＊
711E502F    8B55 FC          MOV EDX,DWORD PTR SS:[EBP-4]
```
