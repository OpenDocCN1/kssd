
```
0044C955    59              POP     ECX

0044C956    59              POP     ECX

0044C957    50              PUSH    EAX

0044C958    E8 13590100     CALL    winimage.00462270

0044C95D    85C0            TEST    EAX,EAX

0044C95F    59              POP     ECX

0044C960    59              POP     ECX

0044C961    75 05           JNZ     SHORT winimage.0044C968

0044C963    33C0            XOR     EAX,EAX

0044C965    40              INC     EAX

0044C966    EB 35           JMP     SHORT winimage.0044C99D

0044C968    8D85 00FFFFFF   LEA     EAX,DWORD PTR SS:[EBP-100]

0044C96E    50              PUSH    EAX

0044C96F    81C6 05201124   ADD     ESI,24112005                               ; ESI=762C78+24112005=24874C7D

0044C975    8D85 00FEFFFF   LEA     EAX,DWORD PTR SS:[EBP-200]

0044C97B    56              PUSH    ESI

0044C97C    50              PUSH    EAX

0044C97D    E8 91FDFFFF     CALL    winimage.0044C713                          ; 12、注册码运算(24B74C7D)

0044C982    59              POP     ECX

0044C983    59              POP     ECX

0044C984    50              PUSH    EAX

0044C985    E8 E6580100     CALL    winimage.00462270

0044C98A    85C0            TEST    EAX,EAX

0044C98C    59              POP     ECX

0044C98D    59              POP     ECX

0044C98E    75 0B           JNZ     SHORT winimage.0044C99B

0044C990    33C0            XOR     EAX,EAX

0044C992    40              INC     EAX

0044C993    85FF            TEST    EDI,EDI

0044C995    74 06           JE      SHORT winimage.0044C99D

0044C997    8907            MOV     DWORD PTR DS:[EDI],EAX

0044C999    EB 02           JMP     SHORT winimage.0044C99D

0044C99B    33C0            XOR     EAX,EAX

0044C99D    5E              POP     ESI

0044C99E    C9              LEAVE

0044C99F    C3              RETN
```

================================================================================

1~12都调用相同的`CALL`来运算的。在这里我们就先看一下`00447133`里面到底有什么东东了!!跟进。。。

```
0044C713    55              PUSH    EBP

0044C714    8BEC            MOV     EBP,ESP

0044C716    83EC 10         SUB     ESP,10

0044C719    56              PUSH    ESI

0044C71A    FF75 0C         PUSH    DWORD PTR SS:[EBP+C]

0044C71D    8B75 08         MOV     ESI,DWORD PTR SS:[EBP+8]

0044C720    8D45 F0         LEA     EAX,DWORD PTR SS:[EBP-10]

0044C723    68 14DB4700     PUSH    winimage.0047DB14                          ; ASCII "%lX"

0044C728    50              PUSH    EAX

0044C729    FF15 14354700   CALL    DWORD PTR DS:[<&USER32.wsprintfA>]         ; USER32.wsprintfA

0044C72F    8A45 F0         MOV     AL,BYTE PTR SS:[EBP-10]

0044C732    83C4 0C         ADD     ESP,0C

0044C735    84C0            TEST    AL,AL

0044C737    74 1D           JE      SHORT winimage.0044C756

0044C739    8D4D F0         LEA     ECX,DWORD PTR SS:[EBP-10]

0044C73C    2BCE            SUB     ECX,ESI

0044C73E    3C 38           CMP     AL,38                                      ; AL和38做比较

0044C740    75 04           JNZ     SHORT winimage.0044C746                    ; 不等转移

0044C742    04 0A           ADD     AL,0A                                      ; 相等则AL加A

0044C744    EB 06           JMP     SHORT winimage.0044C74C                    ; 无条件转移

0044C746    3C 42           CMP     AL,42                                      ; AL和42做比较

0044C748    75 02           JNZ     SHORT winimage.0044C74C                    ; 不等转移

0044C74A    04 F6           ADD     AL,0F6                                     ; 相等则AL和F6相加

0044C74C    8806            MOV     BYTE PTR DS:[ESI],AL                       ; 保存AL到0012E954

0044C74E    46              INC     ESI                                        ; ESI=ESI+1

0044C74F    8A0431          MOV     AL,BYTE PTR DS:[ECX+ESI]

0044C752    84C0            TEST    AL,AL

0044C754  ^ 75 E8           JNZ     SHORT winimage.0044C73E

0044C756    8B45 08         MOV     EAX,DWORD PTR SS:[EBP+8]

0044C759    C606 00         MOV     BYTE PTR DS:[ESI],0

0044C75C    5E              POP     ESI

0044C75D    C9              LEAVE

0044C75E    C3              RETN
```

=================================================================================

从上面这不难看出这个`CALL`的作用是将运算出的注册码逐个和38(8)、42(B)做比较，和38相等的则和A相加，和42相等的则和F6相加取低位`AL`的值。

再从`044C783`往下看我看到相同算法有12处之多，

`762C78`

`762C78+14051948=147B45C0`

`762C78+17061954=177C45CC`

`762C78+10051981=107B45F9`

`762C78+4011995=477460D`

`762C78+2061997=27C460F`

`762C78+12091999=127F4611`

`762C78+16062004=167C4C7C`

`762C78+21042002=217A4C7A`

`762C78+13062004=137C4C7C`

`762C78+9112005=9874C7D`

`762C78+24112005=24874C7D`

通过和38、42比较相加就得出真码了。

仔细看了一下`762C78`后面加的这一串数字全部竟是日期来的，而且这个12组加得的值都是注册码来的。晕哦。前面11组为标准的注册码，最一组为专业版的注册码。

通过下面的这个跳转来判断是否为专为版或是标准版的。

================================================================================

```
0044C961    75 05           JNZ     SHORT winimage.0044C968                    ; 不等则转移为专业版，相等则转移标准版!!
0044C963    33C0            XOR     EAX,EAX
0044C965    40              INC     EAX
0044C966    EB 35           JMP     SHORT winimage.0044C99D                    ; 转移到标准版。
```

注册成功注册表信息如下：

`[HKEY\_CURRENT_USER\Software\WinImage]`

`"NameRegistered"="fcrjzmd"`

`"CodeRegistered"="24B74C7D"`

标准版注册码：

`762C7B`、`147845C0`、`177C45CC`、`107845F9`、`477460D`、`27C460F`、`127F4611`、`167C4C7C`、`217A4C7A`、`137C4C7C`、`9874C7D`

专业版注册码：

`24874C7D`

结后言：

总的来说这个软件算法很简单适合我等初学者学习算法，哎!夜深了一口气写完破文有点凌乱哦真是不好意思啦，眼悃了脑袋有点乱了想不出那么多，总结了过于简单文风不好希望大侠们不要见怪了。我要睡了现在3点钟了，谢谢您能坚持看完我这个菜文。

fcrjzmd

3:01 2006-1-6

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
