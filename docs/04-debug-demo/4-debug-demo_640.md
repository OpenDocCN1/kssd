
# VRV2000 9.E

> 原文：[`www.pediy.com/kssd/pediy03/forum754.htm`](https://www.pediy.com/kssd/pediy03/forum754.htm)

嗨，大家好，我是飞刀浪子。“什么飞刀浪子，听都没听说过。”我~!@#$#$#$#$$%&^&\*\*&^$倒，(一个小时后)从桌下爬起。

好，听好了，我乃中原人士，今年该 21 了，进的时男卫生间。好，废话少说，切入正题！今天我讲的是 `VRV2000 9.E(2000.11.22)` 版破解全攻略。

作案工具：`TR`，`HIEW`。先用 `TR` 载 `VRV2000`。
`BPINT 13 AH=2`

```
      .
      .
      .
```

2779:30F7 `80E1C0` AND `CL`,C0
2779:30FA `024E0E` ADD `CL`,`[BP+000E]`
2779:30FD `8A6E0C` MOV `CH`,`[BP+000C]`
2779:3100 `8A760A` MOV `DH`,`[BP+000A]`
2779:3103 `8A5608` MOV `DL`,`[BP+0008]`
2779:3106 `CD13` INT 13；在此处拦下 `CX=5011`，`DX=0100`
2779:3108 `807E0608` CMP `[BYTE BP+0006]`,08
2779:310C `750A` JNE 3118
2779:310E `C45E12` LES `BX`,`[BP+0012]`
2779:3111 `26890F` MOV `[ES:BX]`,`CX`
2779:3114 `26895702` MOV `[ES:BX+0002]`,`DX`
2779:3118 `8AC4` MOV `AL`,`AH`；!注
2779:311A `B400` MOV `AH`,00；!意
2779:311C `5D` POP `BP`
2779:311D `CB` RETF；返回

3616:1612 `6A02` PUSH 02
3616:1614 `9AE4307927` CALL FAR `2779:30E4`；读盘过程,读 1 面 50H 道 11H 扇区
3616:1619 `83C410` ADD `SP`,10；返回到此
3616:161C `0BC0` OR `AX`,`AX`；必须为 0，因它为额外的正常扇区
3616:161E `7520` JNE 1640
3616:1620 `1E` PUSH `DS`
3616:1621 `6818B8` PUSH `B818`
3616:1624 `6A01` PUSH 01
3616:1626 `6A12` PUSH 12
3616:1628 `6A50` PUSH 50
3616:162A `6A01` PUSH 01
3616:162C `6A00` PUSH 00
3616:162E `6A02` PUSH 02
3616:1630 `9AE4307927` CALL FAR `2779:30E4`；读盘过程,读 1 面 50H 道 12H 扇区
3616:1635 `83C410` ADD `SP`,10
3616:1638 `0BC0` OR `AX`,`AX`；此扇区为坏区或没有，所以应该不为 0
3616:163A `7404` JE 1640
3616:163C `B001` MOV `AL`,01；此处将 `AL=1` 就 OK 了
3616:163E `EB08` JMP 1648
3616:1640 `46` INC `SI`
3616:1641 `83FE02` CMP `SI`,02
3616:1644 `7CBE` JL 1604
3616:1646 `B000` MOV `AL`,00
3616:1648 `5E` POP `SI`
3616:1649 `CB` RETF

避开此两处判断点，就 OK 了。不过，`VRV` 会说内存不够，这一次它说的是真的。好，我们搜索代码，咦，找不到，看来被压过了。

`UNP` 试试看，还是不行。那就手脱吧。
`TR` 载入 `VRV2000`
`EXE1` `RELOAD` `BPREG` `G` `G`

；此时出现 `MOV DX`，`XXXX`
；`MOV [CS：0291]`，`DX`
；`MOV AH`，30
；`INT 21`

`WEXE1` `EXE2` `RELOAD` `G`；直到刚才哪个地方
`WEXE2` `Q` `MAKEXE`

好了，当前目录下就有了一个 `MEM.EXE` 的程序，找到刚才的判断点改掉，就 OK 了。不行，不行，怎么用了第一次就不能用第二次，原来它还回写了自己。

再次 `TR` 设 `BPINT 21 AH=3D`。通过堆栈 `STACK` 看它是什么地方 `CALL` 此处，有好几个耶，一个一个 `U` 看看，

分析一下前后代码，可疑就把这个 `CALL` 干掉试试看（最好是用 `JMP` 飞过去，但注意平衡堆栈）。

（通过 `STACK` 看堆栈有时是很有用的，早期的 `VRV` 追到后面的 `INT 13` 会死机，我就是通过堆栈来解的）

另外，若用 `PKLITE` 将解压过后的程序压一偏也可以不用改。下面我们讲一下用 `DEBUG` 脱壳，虽然有些烦，但可以增加功力啦。
`DEBUG VRV2000.EXE-U`

3D96:001C `FC` CLD
3D96:001D `2E806C1310` SUB `[BYTE CS:SI+0013]`,10
3D96:0022 `73E8` JAE 000C
3D96:0024 `6619ED` SBB `EBP`,`EBP`
3D96:0027 `AF` SCASW
3D96:0028 `AD` LODSW
3D96:0029 `0E` PUSH `CS`
3D96:002A `0E` PUSH `CS`
3D96:002B `0E` PUSH `CS`
3D96:002C `06` PUSH `ES`
3D96:002D `1F` POP `DS`
3D96:002E `07` POP `ES`
3D96:002F `16` PUSH `SS`
3D96:0030 `680500` PUSH 0005
3D96:0033 `6631DB` XOR `EBX`,`EBX`
3D96:0036 `CB` RETF-G36-T-P912E:0004 `A4` MOVSB

912E:0005 `E86D00` CALL 0075；解码过程
912E:0008 `72FA` JB 0004；循环
912E:000A `6631C0` XOR `EAX`,`EAX`
912E:000D `40` INC `AX`
912E:000E `E86400` CALL 0075
912E:0011 `6611C0` ADC `EAX`,`EAX`
912E:0014 `E85E00` CALL 0075
912E:0017 `73F5` JAE 000E
912E:0019 `6683E803` SUB `EAX`,03
912E:001D `720D` JB 002C
912E:001F `66C1E008` SHL `EAX`,08
912E:0023 `AC` LODSB
912E:0024 `6683F0FF` XOR `EAX`,-01
912E:0028 `7469` JE 0093
912E:002A `6695` XCHG `EAX`,`EBP`
912E:002C `E84100` CALL 0070
912E:002F `11C9` ADC `CX`,`CX`
912E:0031 `7508` JNE 003B
912E:0033 `41` INC `CX`
912E:0034 `E83900` CALL 0070
912E:0037 `73FB` JAE 0034
912E:0039 `41` INC `CX`
912E:003A `41` INC `CX`
912E:003B `6681FD00F3FFFF` CMP `EBP`,`FFFFF300`
912E:0042 `81D10100` ADC `CX`,0001
912E:0046 `89FA` MOV `DX`,`DI`
912E:0048 `8CC0` MOV `AX`,`ES`
912E:004A `C1EA04` SHR `DX`,04
912E:004D `83E70F` AND `DI`,0F
912E:0050 `01D0` ADD `AX`,`DX`
912E:0052 `1E` PUSH `DS`



# 汇编代码片段

```
912E:0053 8EC0                MOV     ES,AX               
912E:0055 56                PUSH    SI                
912E:0056 66C1E004           SHL     EAX,04              
912E:005A 01F8                ADD     AX,DI               
912E:005C 6601E8             ADD     EAX,EBP             
912E:005F 89C6                MOV     SI,AX               
912E:0061 66C1E804           SHR     EAX,04              
912E:0065 83E60F             AND     SI,0F               
912E:0068 8ED8                MOV     DS,AX               
912E:006A F3                REPZ                
912E:006B A4                MOVSB               
912E:006C 5E                POP     SI                
912E:006D 1F                POP     DS                
912E:006E EB95                JMP     0005      ;继续循环              
912E:0070 E80200             CALL    0075                
912E:0073 11C9                ADC     CX,CX               
912E:0075 6601DB             ADD     EBX,EBX             
912E:0078 7518                JNE     0092                
912E:007A B7B0                MOV     BH,B0               
912E:007C 39DE                CMP     SI,BX               
912E:007E 7609                JBE     0089                
912E:0080 29DE                SUB     SI,BX               
912E:0082 8CDB                MOV     BX,DS               
912E:0084 80C70B             ADD     BH,0B               
912E:0087 8EDB                MOV     DS,BX               
912E:0089 668B1C             MOV     EBX,[SI]              
912E:008C 83EEFC             SUB     SI,-04                
912E:008F 6611DB             ADC     EBX,EBX             
912E:0092 C3                RET                 
912E:0093 59                POP     CX                
912E:0094 8CC0                MOV     AX,ES     ;!!!!(ES-CX) SHL 4 + DI=文件长 设为 X              
912E:0096 80EC04             SUB     AH,04               
912E:0099 8ED8                MOV     DS,AX               
912E:009B 2BBDFE3F           SUB     DI,[DI+3FFE]          
912E:009F 8DB50040           LEA     SI,[DI+4000]          
912E:00A3 5D                POP     BP        ;!!!BP 值指向了程序的起始段              
912E:00A4 31DB                XOR     BX,BX               
912E:00A6 8EC1                MOV     ES,CX               
912E:00A8 AC                LODSB             ; !!!重定位开始              
912E:00A9 25FF00             AND     AX,00FF               
912E:00AC 7405                JE      00B3                
912E:00AE 3CEF                CMP     AL,EF               
912E:00B0 7710                JA      00C2      
912E:00B2 A8AD                TEST    AL,AD               
912E:00B4 01C3                ADD     BX,AX               
912E:00B6 7305                JAE     00BD                
912E:00B8 80C510             ADD     CH,10               
912E:00BB 8EC1                MOV     ES,CX       ；!!!                
912E:00BD 26012F             ADD     [ES:BX],BP    ;!!重定位                
912E:00C0 EBE6                JMP     00A8                
912E:00C2 88C4                MOV     AH,AL               
912E:00C4 AC                LODSB                 
912E:00C5 25FF0F             AND     AX,0FFF               
912E:00C8 75EA                JNE     00B4                
912E:00CA AD                LODSW                 
912E:00CB 01C1                ADD     CX,AX               
912E:00CD 73D5                JAE     00A4                
912E:00CF 07                POP     ES                
912E:00D0 AD                LODSW             ;取原 SP ，记下 AX 等一下会用到。              
912E:00D1 012C                ADD     [SI],BP     ;定位 SS,记下 SI 的值              
912E:00D3 8E14                MOV     SS,[SI]               
912E:00D5 94                XCHG    AX,SP               
912E:00D6 06                PUSH    ES                
912E:00D7 1F                POP     DS                
912E:00D8 EA0000963D         JMP FAR   3D96:0000   ;解码完毕,进入程序-gd8-t-nvrv.dat-RBX   
```

# 后续处理与说明

`X` 的高位 ----> `X` 看前面 `RCXX` 的低位 `WCS:0` ；回写程序的执行代码 `Q`；下面写重定位数据 `DEBUG VRV2000.EXE-G36-T`

\-`GA8-R` `DI` ；将 `DI` 指向一个不影响我们工作的任意地址 `500-FES:500 L 1000 FF` ；为了方便找最后一个重定位

；将 `BB` 处的两字节 `NOP` 掉，将 `BD` 处的三字节改为 `CALL 200` 然后在 `200` 处写我们的指令。`-A200 MOV AX,BX STOSW`

`MOV AX,CX SUB AX,BP` ；`BP` 值指向了程序的起始段 `STOSW RET-G D8`

；搜索 `ES:500` 后什么地方出现连续的 `FF`，找到的第一个 `FF` 的地址 `-500` 假设结果为 `Y`

；将 `Y/4` 的结果记下，等一下要用。`-NRELOCAL.DAT-RCXY` ；若 `Y` 不是 `16` 的整倍数，将它补为 `16` 的整倍数。`-RBX0`

\-`WES:500` ；写 `RELOCAL.DAT`-`HEADER.DAT`-`FDS:100 L 20 0`-`RCX` -20

\-`WDS:100` ；写 `HEADER.DAT`-`QCOPy /B HEADER.DAT+RELOCAL.DAT+VRV.DAT VRV.EXE`

此时，我们已经得到了一个程序，但他还不能执行，下面修补档头，用 `HIEW` 打开

`4D 5A XX XX XX XX XX XX XX XX 00 00 FF FF XX XX XX XX`

---- ---- ---- ---- ---- ----

|\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_ `VRV.EXE` 的长度除以 `200H` 的余数

|\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_ 商，若余数不为 0，还应再加 1

|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_ 刚才 `Y/4` 的结果

|\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_ (`HEADER.DAT`+`RELOCAL.DAT`)长度/`10H`

|\_\_\_\_\_|\_\_\_\_\_\_\_\_\_\_\_ 刚才记的 `SS` 值

|\_\_\_\_\_\_\_\_\_\_\_ 刚才记的 `SP` 值

`00 00 XX XX 00 00 20 00 00 00 00 00 00 00` -----

|\_\_\_\_\_\_\_\_ `HEADER.DAT`+`RELOCAL.DAT` 的长度注:上面都是十六进制，且是字方式，所以请注意存储的反向问题。

好了，大功告成，亲个嘴儿。注：若我们将任意盘格式化一个正常的 1 面 `50H` 道 `11H` 扇区，就可以做它的加密盘。

若有原版加密盘，可将其所读资料写入我们自做的加密盘中，这样更好。 不过，我解的加密盘都没有原盘，所以只有这样啦。

飞刀浪子 留 `4dd466d0e1e0cc123afbe504da2462ca`
