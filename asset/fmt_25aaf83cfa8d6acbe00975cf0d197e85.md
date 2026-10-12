
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
