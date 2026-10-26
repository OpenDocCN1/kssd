
```
00AF5450    66:8105 0803000>ADD     WORD PTR [308],0C008
00AF5459    3E:E3 42        JECXZ   SHORT 00AF549E                   ; 多余的前缀
00AF545C    CD20 83EC2583   VxDCall 8325EC83
00AF5462    C429            LES     EBP,[ECX]                        ; 修正的段位寄存器
00AF5464    8B45 FC         MOV     EAX,[EBP-4]
00AF5467    C745 FC FFFFFFF>MOV     DWORD PTR [EBP-4],-1
00AF546E    8945 F8         MOV     [EBP-8],EAX
00AF5471    8D45 F0         LEA     EAX,[EBP-10]
00AF5474    64:A3 00000000  MOV     FS:[0],EAX
00AF547A    F2:             PREFIX REPNE:                            ; 多余的前缀
00AF547B    EB 01           JMP     SHORT 00AF547E
00AF547D    9A 36EB02CD 206>CALL    FAR 6820:CD02EB36                ; 远距呼叫
00AF5484    A6              CMPS    BYTE PTR [ESI],BYTE PTR ES:[EDI]
00AF5485    73 42           JNB     SHORT 00AF54C9

//这里好多花指令啊!在花指令堆里面有一个跳转!会一下子跳到下面这里!

00AF79AF    68 A6734200     PUSH    4273A6

//上面的花指令堆跳到了这里!

00AF79B4    68 DC75AF00     PUSH    0AF75DC
00AF79B9    C3              RETN

//这个地方返回!!!!!
```

---

```
00AF50B0   /EB 01           JMP     SHORT 00AF50B3

//返回到这里!

//注意了!

00AF75DF    51              PUSH    ECX
00AF75E0    57              PUSH    EDI
00AF75E1    9C              PUSHFD
00AF75E2    FC              CLD
00AF75E3    BF 2076AF00     MOV     EDI,0AF7620
00AF75E8    B9 5E140000     MOV     ECX,145E
00AF75ED    F3:AA           REP     STOS BYTE PTR ES:[EDI]
00AF75EF    9D              POPFD
00AF75F0    5F              POP     EDI                              ; Christma.004273A6
00AF75F1    59              POP     ECX                              ; Christma.004273A6
00AF75F2    C3              RETN

//返回咯!
```

---

```
00AF75F3    55              PUSH    EBP
00AF75F4    8BEC            MOV     EBP,ESP
00AF75F6    53              PUSH    EBX
00AF75F7    56              PUSH    ESI
00AF75F8    8B75 0C         MOV     ESI,[EBP+C]
00AF75FB    8B5D 08         MOV     EBX,[EBP+8]
```

---

//注意这里!这里其实就是入口的代码,一会要复制回去的所以现在要把他保存起来!

---

```
004273A6    C3              RETN

//00AF75F2返回到这里!继续返回至4247C9!(这里不是OEP!)OEP在上面就已经进行了,而且现在不知道哪去咯!

004247BD    0000            ADD     [EAX],AL
004247BF    0000            ADD     [EAX],AL
004247C1    0000            ADD     [EAX],AL
004247C3    00E8            ADD     AL,CH
004247C5    90              NOP
004247C6    90              NOP
004247C7    90              NOP
004247C8    90              NOP

//注意这里,上面的入口代码给消灭掉了!那么我们把00AF75F3复制过来就OK了!
//我这个初中生在这里很难用语言来表达,希望读者能看懂和消化!

004247C9    BF 94000000     MOV     EDI,94
004247CE    8BC7            MOV     EAX,EDI
004247D0    E8 1BF2FFFF     CALL    004239F0
004247D5    8965 E8         MOV     [EBP-18],ESP
004247D8    8BF4            MOV     ESI,ESP
004247DA    893E            MOV     [ESI],EDI
004247DC    56              PUSH    ESI
004247DD    FF15 D0204300   CALL    [4320D0]
```

---

把代码都复制好后就用LordPE把他DUMP出来!

然后运行ImportREC1.6把他修复好,其中有一个指针要多找几次才能找到(用上插件,我机子就全不的指针都找到了!)

关于怎么用ImportREC1.6我就不说了,多多看看FLY的文章就知道ImportREC1.6怎么用的了!

脱壳后差看是Microsoft Visual C++ 6.0写的!

197K->532K(我不知道怎么优化,惭愧ING....)

说真的,觉得这个可好容易脱,才半小时,连文章都写好了!

睡觉咯~~~~~祝大家圣诞快了,算法高手快破了这个,做个主厕机出来给Crack们一起庆祝圣诞Ing..........
