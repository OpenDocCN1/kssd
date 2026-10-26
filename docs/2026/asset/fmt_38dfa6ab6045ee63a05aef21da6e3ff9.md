
```
004E1E14  /$  8BD0          MOV EDX,EAX                                 

004E1E16  |.  C1EA 18       SHR EDX,18                                     ;取出EAX的第一字节

004E1E19  |.  8B1495 E8F255>MOV EDX,DWORD PTR DS:[EDX*4+55F2E8]            ； 查表得数据

004E1E20  |.  8BC8          MOV ECX,EAX

004E1E22  |.  C1E9 10       SHR ECX,10                                     

004E1E25  |.  81E1 FF000000 AND ECX,0FF                                    ；取出EAX的第二字节

004E1E2B  |.  33148D E8EE55>XOR EDX,DWORD PTR DS:[ECX*4+55EEE8]            ；查表得数据与前数据异或 

004E1E32  |.  8BC8          MOV ECX,EAX

004E1E34  |.  C1E9 08       SHR ECX,8

004E1E37  |.  81E1 FF000000 AND ECX,0FF                                   ；取出EAX的第三字节

004E1E3D  |.  33148D E8EA55>XOR EDX,DWORD PTR DS:[ECX*4+55EAE8]           ；查表得数据与前数据异或  

004E1E44  |.  25 FF000000   AND EAX,0FF                                   ；取出EAX的第四字节

004E1E49  |.  331485 E8E655>XOR EDX,DWORD PTR DS:[EAX*4+55E6E8]           ；查表得数据与前数据异或

004E1E50  |.  8BC2          MOV EAX,EDX                                   ；保存最后结果

004E1E52  \.  C3            RETN
```

观察堆栈发现

0012FB1C   004E1FB1  返回到 ResScope.004E1FB1 来自 ResScope.004E1E54

....

0012FB34   00519438  返回到 ResScope.00519438 来自 ResScope.004E1F8C

而`00519438`就和上面处理注册码的`CALL ResScope.00519298`接上了，这是反推，现在让我们正向从头慢慢看看

```
00519298   $  55            PUSH EBP

00519299   .  8BEC          MOV EBP,ESP

0051929B   .  81C4 64FFFFFF ADD ESP,-9C

。。。。。。

005192E4   >  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]         ;005193bc跳到这儿，循环将0012fb9c数据块清0（大小18h）

005192E7   .  83C0 01       ADD EAX,1

005192EA   .  71 05         JNO SHORT ResScope.005192F1

005192EC   .  E8 33A4EEFF   CALL ResScope.00403724

005192F1   >  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]

005192F4   .  8B12          MOV EDX,DWORD PTR DS:[EDX]

005192F6   .  48            DEC EAX

005192F7   .  85D2          TEST EDX,EDX

005192F9   .  74 05         JE SHORT ResScope.00519300

005192FB   .  3B42 FC       CMP EAX,DWORD PTR DS:[EDX-4]

005192FE   .  72 05         JB SHORT ResScope.00519305

00519300   >  E8 17A4EEFF   CALL ResScope.0040371C

00519305   >  40            INC EAX

00519306   .  8A4402 FF     MOV AL,BYTE PTR DS:[EDX+EAX-1]

0051930A   .  8B55 E4       MOV EDX,DWORD PTR SS:[EBP-1C]

0051930D   .  8802          MOV BYTE PTR DS:[EDX],AL                        ;用0填充数据

0051930F   .  807D FB 00    CMP BYTE PTR SS:[EBP-5],0

00519313   .  0F84 99000000 JE ResScope.005193B2

00519319   .  837D F0 14    CMP DWORD PTR SS:[EBP-10],14

0051931D   .  0F85 8F000000 JNZ ResScope.005193B2

。。。。。

005193B2   >  FF45 F0       INC DWORD PTR SS:[EBP-10]

005193B5   .  FF45 E4       INC DWORD PTR SS:[EBP-1C]

005193B8   .  837D F0 18    CMP DWORD PTR SS:[EBP-10],18

005193BC   .^ 0F85 22FFFFFF JNZ ResScope.005192E4

005193C2   .  68 E05A5700   PUSH ResScope.00575AE0                              ; /Arg1 = 00575AE0

005193C7   .  BA E85A5700   MOV EDX,ResScope.00575AE8                           ; |

005193CC   .  8D85 68FFFFFF LEA EAX,DWORD PTR SS:[EBP-98]                       ; |

005193D2   .  B9 20000000   MOV ECX,20                                          ; |

005193D7   .  E8 3889FCFF   CALL ResScope.004E1D14                              ;得到初始数据，在0012fb64

005193DC   .  C745 F0 01000>MOV DWORD PTR SS:[EBP-10],1

005193E3   >  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]

005193E6   .  83E8 01       SUB EAX,1

005193E9   .  71 05         JNO SHORT ResScope.005193F0

005193EB   .  E8 34A3EEFF   CALL ResScope.00403724

005193F0   >  6BC0 08       IMUL EAX,EAX,8

005193F3   .  71 05         JNO SHORT ResScope.005193FA

005193F5   .  E8 2AA3EEFF   CALL ResScope.00403724

005193FA   >  83F8 17       CMP EAX,17

005193FD   .  76 05         JBE SHORT ResScope.00519404

005193FF   .  E8 18A3EEFF   CALL ResScope.0040371C

00519404   >  8D4C05 B0     LEA ECX,DWORD PTR SS:[EBP+EAX-50]

00519408   .  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]

0051940B   .  83EA 01       SUB EDX,1

0051940E   .  71 05         JNO SHORT ResScope.00519415

00519410   .  E8 0FA3EEFF   CALL ResScope.00403724

00519415   >  6BD2 08       IMUL EDX,EDX,8

00519418   .  71 05         JNO SHORT ResScope.0051941F

0051941A   .  E8 05A3EEFF   CALL ResScope.00403724

0051941F   >  83FA 17       CMP EDX,17

00519422   .  76 05         JBE SHORT ResScope.00519429

00519424   .  E8 F3A2EEFF   CALL ResScope.0040371C

00519429   >  8D5405 B0     LEA EDX,DWORD PTR SS:[EBP+EAX-50]

0051942D   .  8D85 68FFFFFF LEA EAX,DWORD PTR SS:[EBP-98]

00519433   .  E8 548BFCFF   CALL ResScope.004E1F8C                      ;里面调用了上面004E1E89的代码
```

其中`005193D7   .  E8 3889FCFF   CALL ResScope.004E1D14`的代码如下

```
004E1D14  /$  55            PUSH EBP

004E1D15  |.  8BEC          MOV EBP,ESP

004E1D17  |.  83C4 F8       ADD ESP,-8

004E1D1A  |.  53            PUSH EBX

004E1D1B  |.  56            PUSH ESI

004E1D1C  |.  57            PUSH EDI

........

004E1D96  |>  8BC8          /MOV ECX,EAX

004E1D98  |.  C1E1 02       |SHL ECX,2

004E1D9B  |.  0FB64C0B 03   |MOVZX ECX,BYTE PTR DS:[EBX+ECX+3]  ;EBX=00575AE8,可以查看此处内存数据是固定的

004E1DA0  |.  C1E1 18       |SHL ECX,18

004E1DA3  |.  8BF0          |MOV ESI,EAX

004E1DA5  |.  C1E6 02       |SHL ESI,2

004E1DA8  |.  0FB67433 02   |MOVZX ESI,BYTE PTR DS:[EBX+ESI+2]

004E1DAD  |.  C1E6 10       |SHL ESI,10

004E1DB0  |.  0BCE          |OR ECX,ESI

004E1DB2  |.  8BF0          |MOV ESI,EAX

004E1DB4  |.  C1E6 02       |SHL ESI,2

004E1DB7  |.  0FB67433 01   |MOVZX ESI,BYTE PTR DS:[EBX+ESI+1]     

004E1DBC  |.  C1E6 08       |SHL ESI,8

004E1DBF  |.  0BCE          |OR ECX,ESI

004E1DC1  |.  8BF0          |MOV ESI,EAX

004E1DC3  |.  C1E6 02       |SHL ESI,2

004E1DC6  |.  0FB63433      |MOVZX ESI,BYTE PTR DS:[EBX+ESI]

004E1DCA  |.  0BCE          |OR ECX,ESI

004E1DCC  |.  890A          |MOV DWORD PTR DS:[EDX],ECX            ;EDX=0012FB64,

004E1DCE  |.  40            |INC EAX

004E1DCF  |.  83C2 04       |ADD EDX,4

004E1DD2  |.  83F8 08       |CMP EAX,8

004E1DD5  |.^ 75 BF         \JNZ SHORT ResScope.004E1D96

004E1DD7  |.  5F            POP EDI

004E1DD8  |.  5E            POP ESI

004E1DD9  |.  5B            POP EBX

004E1DDA  |.  59            POP ECX

004E1DDB  |.  59            POP ECX

004E1DDC  |.  5D            POP EBP

004E1DDD  \.  C2 0400       RETN 4
```

上面的代码就是将`00575AE8`的数据(`DB9073B0`)放到`0012FB64`，然后`CALL 004E1F8C`里调用`004E1E89`->`004E1E14`,`EAX`的值为`B07390DB`

今天先分析到这儿，有空再继续哈

--------------------------------------------------------------------------------

【版权声明】: 看雪论坛首发，转载请注明作者并保持文章的完整, 谢谢!

--------------------------------------------------------------------------------

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [ResScope.rar](javascript:void(0);)

  （2.18MB，87次下载）
