
# WINDOWS 自带游戏空档接龙的 BUG

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-82.htm`](https://www.pediy.com/kssd/pediy09/pediy09-82.htm)

> `XP` `SP2`,  
> 运行空档接龙游戏(`freecell.exe`)  
> 最大化窗口,然后 `F2` 开始新游戏,然后在窗口右侧中部点击鼠标右键,应该就会使程序崩溃

![](img/8fd22d9a2df758d46e6db173168add09.jpg)  

> 无意中发现,有兴趣的大牛研究一下 

> 看过楼上给出的代码，觉得那句 `je` 不是关键，要不然改成 `jmp` 后，下面那一大堆的东西就没用了。  

> 我感觉是 `je` 上面的那句 `call` 的问题  

> `010033D9` `CALL` `freecell.01002CFC`  

> 这一句是检查鼠标指向位置是否有一张牌，`eax` 返回 `1` 表明有  
> 检查方式是根据鼠标指针坐标来计算的。  
> 跟进大致分析如下：  

```assembly
    01002CFC  /$  55             PUSH    EBP  
    01002CFD  |.  8BEC          MOV     EBP, ESP  
    01002CFF  |.  A1 68780001   MOV     EAX, DWORD PTR DS:[1007868]  
    01002D04  |.  3945 0C       CMP     DWORD PTR SS:[EBP+C], EAX        ;  比较 y,是否在上面空格区  
    01002D07  |.  73 4E         JNB     SHORT freecell.01002D57        ;  不是则跳  
    ...  
    ...  
    01002D57  |>  8B0D 90700001 MOV     ECX, DWORD PTR DS:[1007090]  
    01002D5D  |.  03C1          ADD     EAX, ECX  
    01002D5F  >|. 3945 0C       CMP     DWORD PTR SS:[EBP+C], EAX        ;  检查 y 是否在纸牌上界以下  
    01002D62  |.  72 0B         JB      SHORT freecell.01002D6F        ;  是则不跳  
    01002D64  |.  8B15 70700001 MOV     EDX, DWORD PTR DS:[1007070]  
    01002D6A  |.  3955 08       CMP     DWORD PTR SS:[EBP+8], EDX        ;  检查 x 是否在纸牌左边界右侧  
    01002D6D  |.  73 07         JNB     SHORT freecell.01002D76        ;  是则跳  
    01002D6F  |>  33C0          XOR     EAX, EAX  
    01002D71  |.  E9 AE000000   JMP     freecell.01002E24                ;  右击窗口最左边  
    01002D76  |>  8B45 08       MOV     EAX, DWORD PTR SS:[EBP+8]  
    01002D79  |.  53            PUSH    EBX  
    01002D7A  |.  56            PUSH    ESI  
    01002D7B  |.  8B35 74700001 MOV     ESI, DWORD PTR DS:[1007074]  
    01002D81  |.  2BC2          SUB     EAX, EDX  
    01002D83  |.  2BF2          SUB     ESI, EDX  
    01002D85  |.  33D2          XOR     EDX, EDX  
    01002D87  |.  F7F6          DIV     ESI  
    01002D89  |.  8B75 10       MOV     ESI, DWORD PTR SS:[EBP+10]  
    01002D8C  |.  40            INC     EAX                            ;  所在列数（第一列为 1）  
    01002D8D  |.  8906          MOV     DWORD PTR DS:[ESI], EAX  
    01002D8F  |.  8B0485 6C7000>MOV     EAX, DWORD PTR DS:[EAX*4+100706C]  ;  每列纸牌的左边界  
    01002D96  |.  0305 70830001 ADD     EAX, DWORD PTR DS:[1008370]        ;  列边界+纸牌宽度  
    01002D9C  |.  3945 08       CMP     DWORD PTR SS:[EBP+8], EAX  
    01002D9F  |.  76 04         JBE     SHORT freecell.01002DA5  
    01002DA1  |>  33C0          XOR     EAX, EAX  
    01002DA3  |.  EB 7D         JMP     SHORT freecell.01002E22        ;  在两列纸牌之间，包括窗口最右边  
    01002DA5  |>  A1 68780001   MOV     EAX, DWORD PTR DS:[1007868]  
    01002DAA  |.  03C8          ADD     ECX, EAX                       ;  ecx=纸牌上边界  
    01002DAC  |.  294D 0C       SUB     DWORD PTR SS:[EBP+C], ECX        ;  y = y - 上边界  
    01002DAF  |.  8B45 0C       MOV     EAX, DWORD PTR SS:[EBP+C]        ;  eax 为离上边界的距离  
    01002DB2  |.  33D2          XOR     EDX, EDX  
    01002DB4  |.  F735 80790001 DIV     DWORD PTR DS:[1007980]         ;  除以上下两张牌的间隔距离  
    01002DBA  |.  6A 15         PUSH    15                            ;  15 是程序认为每列可能被摆放的最大牌数  
    01002DBC  |.  5A            POP     EDX  
    01002DBD  |.  3BC2          CMP     EAX, EDX                       ;  eax 为计算后得出的鼠标所在列的层数（第一层为 0）  
    01002DBF  |.  73 02         JNB     SHORT freecell.01002DC3        ;  超过则跳(包括 15)  
    01002DC1  |.  8BD0          MOV     EDX, EAX  
    01002DC3  |>  8B4D 14       MOV     ECX, DWORD PTR SS:[EBP+14]  
    01002DC6  |.  8911          MOV     DWORD PTR DS:[ECX], EDX        ;  所在层  
    01002DC8  |.  8B06          MOV     EAX, DWORD PTR DS:[ESI]        ;  所在列  
    01002DCA  |.  8BD8          MOV     EBX, EAX  
    01002DCC  |.  6BDB 54       IMUL    EBX, EBX, 54                  ;  列 X54  
    01002DCF  |.  83BB 00750001>CMP     DWORD PTR DS:[EBX+1007500], -1      ;  所在列是否还有牌  
    01002DD6  |.^ 74 C9         JE      SHORT freecell.01002DA1        ;  没有则跳  
    01002DD8  |.  83FA 14       CMP     EDX, 14  
    01002DDB  |.  73 16         JNB     SHORT freecell.01002DF3  
    01002DDD  |.  8BD8          MOV     EBX, EAX  
    01002DDF  |.  6BDB 15       IMUL    EBX, EBX, 15  
    01002DE2  |.  03DA          ADD     EBX, EDX                      ;  列 X15+层  
    01002DE4  |.  833C9D 047500>CMP     DWORD PTR DS:[EBX*4+1007504], -1   ;  下一张牌面点数是否有效  
    01002DEC  |.  74 05         JE      SHORT freecell.01002DF3        ;  无效则跳  
    01002DEE  |.  33C0          XOR     EAX, EAX  
    01002DF0  |.  40            INC     EAX  
    01002DF1  |.  EB 2F         JMP     SHORT freecell.01002E22  
    01002DF3  |>  6BC0 15       IMUL    EAX, EAX, 15  
    01002DF6  |.  03C2          ADD     EAX, EDX                      ;  列 X15+层  
    01002DF8  |.  EB 09         JMP     SHORT freecell.01002E03  
    01002DFA  |>  FF09          /DEC     DWORD PTR DS:[ECX]            ;  层数  
    01002DFC  |.  8B06          |MOV     EAX, DWORD PTR DS:[ESI]  
    01002DFE  |.  6BC0 15       |IMUL    EAX, EAX, 15  
    01002E01  |.  0301          |ADD     EAX, DWORD PTR DS:[ECX]  
    01002E03  |>  833C85 007500> CMP     DWORD PTR DS:[EAX*4+1007500], -1  ;  检查牌面点数（当层数为 15 时，因为牌面点数数据是连续的，所以这时指针所指的是右侧一列首张牌）  
    01002E0B  |.^ 74 ED         \JE     SHORT freecell.01002DFA       ;  空牌则再往上找，最终确定该列实际牌数  
    01002E0D  |.  8B01          MOV     EAX, DWORD PTR DS:[ECX]        ;  所在列牌的数量  
    01002E0F  |.  0FAF05 807900>IMUL    EAX, DWORD PTR DS:[1007980]  
    01002E16  |.  0305 68780001 ADD     EAX, DWORD PTR DS:[1007868]    ;  所在列的下边界  
    01002E1C  |.  3B45 0C       CMP     EAX, DWORD PTR SS:[EBP+C]      ;  比较 y，影响 CF，当层数为 15 时，这里就出问题了--CF 为 0  
    01002E1F  |.  1BC0          SBB     EAX, EAX                      ;  带 CF 的减法  
    01002E21  |.  40            INC     EAX  
    01002E22  |>  5E            POP     ESI  
    01002E23  |.  5B            POP     EBX  
    01002E24  |>  5D            POP     EBP  
    01002E25  \\.  C2 1000       RETN    10
```

4dd466d0e1e0cc123afbe504da2462ca
