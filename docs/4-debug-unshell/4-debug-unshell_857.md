
# PECompact 2.x 完美脱壳法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-495.htm`](https://www.pediy.com/kssd/pediy07/pediy7-495.htm)

不好意思拿出来。。。

碰到这么个程序，顺手研究了一下，好像还没人写过。。。

【灌水】PECompact 2.x 完美脱壳法

【水龙头】`cyclotron[BCG][DFCG][FCG][OCN]`

【目标】`GameThrust v1.1`，天空网下载

【加壳方式】PECompact 2.x -> Jeremy Collake

OllyDbg 忽略内存访问异常，载入主程序：

```
00401000 >   B8 1CBA5000     MOV     EAX,0050BA1C
00401005    50              PUSH    EAX
00401006    64:FF35 0000000>PUSH    DWORD PTR FS:[0]
0040100D    64:8925 0000000>MOV     DWORD PTR FS:[0],ESP
00401014    33C0            XOR     EAX,EAX
00401016    8908            MOV     DWORD PTR DS:[EAX],ECX
00401018    50              PUSH    EAX
00401019    45              INC     EBP
0040101A    43              INC     EBX
0040101B    6F              OUTS    DX,DWORD PTR ES:[EDI]          ; I/O command
0040101C    6D              INS     DWORD PTR ES:[EDI],DX          ; I/O command
```

bp `VirtualAlloc` 运行

返回到用户代码：

Ctrl+F 搜索命令：`call edi`

```
0050BAC1    8B4B 0C         MOV     ECX,DWORD PTR DS:[EBX+C]
0050BAC4    894E 14         MOV     DWORD PTR DS:[ESI+14],ECX
0050BAC7    FFD7            CALL    EDI       ；跟进去
0050BAC9    8985 23120010   MOV     DWORD PTR SS:[EBP+10001223],EAX
0050BACF    8BF0            MOV     ESI,EAX
0050BAD1    59              POP     ECX
0050BAD2    5A              POP     EDX
```

进入 `call edi`，来到：

```
01240258    53              PUSH    EBX                       ; GameThru.0050B9F8
01240259    57              PUSH    EDI
0124025A    56              PUSH    ESI
0124025B    55              PUSH    EBP
0124025C    E8 00000000     CALL    01240261
01240261    5D              POP     EBP
01240262    81ED 30120010   SUB     EBP,10001230
01240268    8DB5 27120010   LEA     ESI,DWORD PTR SS:[EBP+10001227]
0124026E    8B46 FC         MOV     EAX,DWORD PTR DS:[ESI-4]
```

Ctrl+F 搜索命令：`mov ecx,[esi+34]`

```
01240333    90              NOP
01240334    90              NOP
01240335    8B4E 34         MOV     ECX,DWORD PTR DS:[ESI+34]
01240338    85C9            TEST    ECX,ECX     ; ecx=8D000 保存原始 Import，把它改成零
0124033A    0F84 89000000   JE      012403C9    ; 这里强制跳过去，跟 tE 一样
01240340    034E 08         ADD     ECX,DWORD PTR DS:[ESI+8]
01240343    51              PUSH    ECX
01240344    56              PUSH    ESI
```

bp `VirtualFree`，中断两次以后返回（thx2fly）：

```
0050BADC    57              PUSH    EDI
0050BADD    FF11            CALL    DWORD PTR DS:[ECX]
0050BADF    8BC6            MOV     EAX,ESI                     ; GameThru.00487228
0050BAE1    5A              POP     EDX
0050BAE2    5E              POP     ESI
0050BAE3    5F              POP     EDI
0050BAE4    59              POP     ECX
0050BAE5    5B              POP     EBX
0050BAE6    5D              POP     EBP
0050BAE7    FFE0            JMP     EAX       ; 飞向光明之颠^o^
```

完全 dump 下来，修正输入表为 `8D000`，收工。

## OS 自动脚本

```
////////////////////////////////////////////
////    PECompact V2.X 完美脱壳脚本
////    cyclotron [BCG][DFCG][FCG][OCN]
////////////////////////////////////////////

msg  "请忽略内存访问异常!"
var  dwImport
gpa  "VirtualAlloc","kernel32.dll" 
bp  $RESULT
run
bc  $RESULT   
rtu
findop  eip,#FFD7#  //call edi
bp  $RESULT
run
bc  $RESULT
sti
findop  eip,#8B4E34#  //mov ecx,[esi+34h]
bp  $RESULT
run
bc  $RESULT
sti
mov  dwImport,ecx
mov  ecx,0
gpa  "VirtualFree","kernel32.dll" 
bp  $RESULT
run
run
bc  $RESULT
rtu
findop  eip,#FFE0#  //jmp eax
bp  $RESULT
run
bc  $RESULT
msg  dwImport
msg  "请转储文件！"
```

Regards,

cyclotron

05.2.6

一点补充 `![](img/83bdd63c29485f7768335d7ba60e67a5.jpg)`

第一次 bp `VirtualAlloc` 断点，当中断第二次是，再返回到程序代码中，可以直接来到下面这里，向下一看，就可以找到处理ＩＡＴ的地方了：

```
003F0C22    E8 89030000       call 003F0FB0   ;![](img/1a550ccb7245757838a4f08fc220b787.jpg) 
003F0C27    85C0              test eax, eax
003F0C29    0F85 A6000000     jnz 003F0CD5
003F0C2F    56                push esi
003F0C30    E8 D7020000       call 003F0F0C
003F0C35    56                push esi
003F0C36    E8 DF010000       call 003F0E1A
003F0C3B    90                nop
003F0C3C    90                nop
003F0C3D    90                nop
003F0C3E    90                nop
003F0C3F    90                nop
003F0C40    90                nop
003F0C41    90                nop
003F0C42    90                nop
003F0C43    90                nop
003F0C44    90                nop
003F0C45    90                nop
003F0C46    90                nop
003F0C47    90                nop
003F0C48    90                nop
003F0C49    8B4E 34           mov ecx, dword ptr ds:[esi+34]  ;![](img/b16bec246e75299c87cf57e1fb808ac9.jpg) 
003F0C4C    85C9              test ecx, ecx
003F0C4E    0F84 89000000     je 003F0CDD     ;********
```

最后 bp `VirtualFree` 中断三次后才能直接返回上述的地方

（不知是不是只有我的机子是这样）中断两次返回后来到

```
003F0D14    8B46 0C           mov eax, dword ptr ds:[esi+C]
003F0D17    03C7              add eax, edi
003F0D19    5D                pop ebp
003F0D1A    5E                pop esi
003F0D1B    5F                pop edi
003F0D1C    5B                pop ebx
003F0D1D    C3                retn
```

`F8` 执行完这几句后才来到:

```
00590145    8985 23120010     mov dword ptr ss:[ebp+10001223], eax      ; SuperRam.0048DDE8
0059014B    8BF0              mov esi, eax
0059014D    59                pop ecx
0059014E    5A                pop edx
0059014F    03CA              add ecx, edx
00590151    68 00800000       push 8000
00590156    6A 00             push 0
00590158    57                push edi
00590159    FF11              call dword ptr ds:[ecx]
0059015B    8BC6              mov eax, esi :诺是三次中断后，会直接返回到这
0059015D    5A                pop edx
0059015E    5E                pop esi
0059015F    5F                pop edi
00590160    59                pop ecx
00590161    5B                pop ebx
00590162    5D                pop ebp
00590163    FFE0              jmp eax
```

4dd466d0e1e0cc123afbe504da2462ca
