

# VMProtect 虚拟机小窥

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-602.htm`](https://www.pediy.com/kssd/pediy07/pediy7-602.htm)

最近太颓废了，写点东西发泄一下。  
  
究竟它是不是一个壳我也说不清楚，我不是很会用，随便加了个 notepad 看看。  
  
P.S. 我不是很会起名字，也没有检查那些指令是不是准确，不过这些可能会动态改变，看看就行了。 :D  
  
  
  
vmp\_pcode\_engine:  
    pushf      ; 这些代码看起来是保存符号和寄存器的,  
          ; 但若往后跟几步便会发现这里顺序把  
          ;  
          ; EFLAG  
          ; EAX  
          ; ECX  
          ; EDX  
          ; EBX  
          ; ESP  
          ; EBP  
          ; ESI  
          ; EDI  
          ;  
          ; 入栈,  随后用伪指令弹出构造自身 CONTEXT 环境  
          ;  
    pusha  
    push  0    ; 再压一个 0 进堆栈  
  
    mov  esi, \[esp+28h\]  ; -> 当前代码块伪指令表  
          ;  
          ; 00408B54  D6 82 AC 61  79 29 4B 03 15 C8 E4 9A  B0 61 81 34  謧琣 y)K 蠕毎 a?  
          ; 00408B64  50 FF 23 D0  AC 04 73 CF BF FF 59 31  0B 1E D0 0C  P�#鞋 s 峡�Y1  
?  
          ; 00408B74  69 D8 AF 42  1B AA 87 0E E9 74 54 D5  B2 39 1B 98  i 丿 B 獓閠 T 詹 9  
          ;  
    mov  edi, offset vmp\_banner\_and\_vcontext ; "\[ VMProtect v 1.01 (C) PolyTech |  http"...  
    cld  
    mov  ebx, esi  ; 指向 PCODE  
  
next\_opcode:  
  
    lodsb      ; 取一个伪指令  
    add  al, bl    ; 纯粹是解码, 用了会慢一些  
    add  bl, al  
    movzx  eax, al  
    jmp  ds:pcode\_parse\_table\[eax\*4\] ; 执行表中对应索引的指令, 与 VB 手法一样  
          ;  
          ; 我给一些指令起了名字后这段表的意思是  
          ;  
          ; v\_odi    00     ; 用数据区的 CONTEXT 保存当前环境  
          ; v\_odi    09  
          ; v\_odi    04  
          ; v\_odi    0C  
          ; v\_odi    07  
          ; v\_odi    0A  
          ; v\_odi    05  
          ; v\_odi    07  
          ; v\_odi    03  
          ; v\_odi    01  
          ; v\_oax  
          ; v\_pd    97      ; 一个加密后的地址  
          ; v\_pd    00      ; 密钥  
          ; v\_na2        ;\\按照 na2 指令解码地址并入栈  
          ; v\_pda        ;/  
          ; v\_pdi    01      ; 把保存的 CONTEXT 映射回 CPU  
          ; v\_pdi    03  
          ; v\_pdi    07  
          ; ....  
          ; v\_pdi    00  
          ; v\_exit      ; 返回  
          ;  
  
  
  
v\_pdi:          ; DATA XREF: .vmp0:0040895Do  
    lodsb      ; Push Dword in  data area by Index  
    add  al, bl    ; 操作数也要解码才行  
    add  bl, al  
    push  dword ptr \[edi+eax\*4\]  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
v\_pb:          ; DATA XREF: .vmp0:00408759o  
    lodsd      ; Push Byte (as  DWORD, Lo used only)  
    add  al, bl  
    add  bl, al  
    push  eax  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
loc\_408034:        ; DATA XREF: .vmp0:004088BDo  
          ; .vmp0:00408769o ...  
    push  sp  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
loc\_408038:        ; DATA XREF: .vmp0:004089FDo  
    pop  eax  
    push  dword ptr gs:\[eax\]  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
loc\_40803E:        ; DATA XREF: .vmp0:00408995o  
    pop  eax  
    push  dword ptr fs:\[eax\]  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
v\_pda:          ; DATA XREF: .vmp0:00408755o  
    pop  eax    ; Pop address and Push Dword in  address  
    push  dword ptr \[eax\]  
    jmp  short next\_opcode ; 取一个伪指令  
  
  
  
    ; 省略一些  
  
v\_na2:          ; DATA XREF: .vmp0:004089D1o  
          ; .vmp0:00408A75o  
    pop  eax    ; not &  and then save DWORD  
    not  eax  
    not  dword ptr \[esp\]  
    and  \[esp\], eax  
    jmp  next\_opcode  ; 取一个伪指令  
  
  
loc\_4082C0:        ; DATA XREF: .vmp0:00408791o  
          ; .vmp0:004089C5o  
    pop  eax  
    not  eax  
    not  dword ptr \[esp\]  
    and  \[esp\], eax  
    pushfw  
    jmp  next\_opcode  ; 取一个伪指令  
  
    ; 省略一些  
  
    ; 每个 DWORD 对应一条指令解析器  
  
pcode\_parse\_table dd offset unk\_40829C  ; DATA XREF: execute\_vmp\_engine-B6Fr  
    dd offset unk\_40864D  
    dd offset unk\_4085E5  
    dd offset unk\_408699  
    dd offset unk\_40838D  
    dd offset unk\_408364  
    dd offset unk\_4082D0  
    dd offset unk\_408373  
    dd offset unk\_4086F4  
    dd offset unk\_4085BD  
    dd offset unk\_40853A  
    dd offset loc\_408144  
    dd offset unk\_408465  
    dd offset unk\_408458  
    dd offset unk\_408437  
    dd offset unk\_408531  
    dd offset unk\_408504  
    dd offset unk\_408269  
    dd offset unk\_408347  
    dd offset v\_pda    ; Pop address and Push Dword in  address  
    dd offset v\_pb    ; Push Byte (as  DWORD, Lo used only)  
    dd offset unk\_408261  
    dd offset unk\_4085F5  
    dd offset loc\_408187  
    dd offset loc\_408034+1  
    dd offset loc\_408066  
    dd offset loc\_40807A  
    dd offset unk\_40828F  
    dd offset unk\_4081EB  
    dd offset unk\_40860D  
    dd offset loc\_4080A2  
    dd offset unk\_408594  
    dd offset unk\_408692  
    dd offset loc\_408066  
    dd offset loc\_4082C0  
    dd offset loc\_4080CB  
    dd offset unk\_4086A7  
    dd offset unk\_40855E  
    dd offset unk\_4084E0  
    dd offset loc\_408088  
    dd offset unk\_40840B  
    dd offset loc\_408098  
    dd offset v\_odi    ; pOp dword on stack to  Dword in data area by Index  
    dd offset unk\_4082EE  
    dd offset unk\_4086D1  
    dd offset unk\_408528  
    dd offset unk\_4081A2  
    dd offset unk\_4084AA  
    dd offset loc\_408034+1  
    dd offset unk\_408498  
    dd offset unk\_408199  
    dd offset unk\_4086D8  
    dd offset unk\_408692  
    dd offset loc\_40812E  
    dd offset unk\_40850D  
    dd offset loc\_4080AC  
    dd offset unk\_408570  
    dd offset unk\_4084BC  
    dd offset unk\_40848F  
    dd offset loc\_408070  
    dd offset loc\_40817F  
    dd offset loc\_40805B  
    dd offset unk\_4083A8  
    dd offset unk\_4086DF  
    dd offset unk\_4083D4  
    dd offset unk\_408486  
    dd offset unk\_4082DE  
    dd offset unk\_408253  
    dd offset loc\_40816D  
    dd offset unk\_40865D  
    dd offset unk\_40824B  
    dd offset unk\_40853A  
    dd offset unk\_4084D7  
    dd offset unk\_40858B  
    dd offset unk\_40866D  
    dd offset unk\_40867D  
    dd offset unk\_4086FB  
    dd offset unk\_4084F2  
    dd offset unk\_408477  
    dd offset unk\_408190  
    dd offset unk\_408594  
    dd offset loc\_408270  
    dd offset loc\_4080E5  
    dd offset loc\_408166+1  
    dd offset unk\_40863D  
    dd offset loc\_4080FB  
    dd offset unk\_4084B3  
    dd offset unk\_4086E6  
    dd offset loc\_4080F0  
    dd offset unk\_40841D  
    dd offset unk\_408449  
    dd offset unk\_408449  
    dd offset unk\_4084A1  
    dd offset unk\_4083B8  
    dd offset loc\_4080BB  
    dd offset unk\_408240  
    dd offset unk\_408253  
    dd offset unk\_408555  
    dd offset loc\_40805B  
    dd offset unk\_4085C5  
    dd offset unk\_4084D7  
    dd offset unk\_4085B5  
    dd offset unk\_4082A6  
    dd offset unk\_4086D1  
    dd offset unk\_408364  
    dd offset unk\_4083FF  
    dd offset unk\_408645  
    dd offset unk\_4085ED  
    dd offset unk\_408253  
    dd offset loc\_408034  
    dd offset loc\_408176  
    dd offset loc\_4080CB  
    dd offset unk\_4084BC  
    dd offset loc\_4080F0  
    dd offset loc\_4080CB  
    dd offset unk\_4085A6  
    dd offset unk\_408531  
    dd offset unk\_4086CA  
    dd offset loc\_408105  
    dd offset v\_oax    ; pop ax  
    dd offset aXf\_p    ; "Xf.?  
    dd offset unk\_408684  
    dd offset v\_oax    ; pop ax  
    dd offset loc\_40814B  
    dd offset unk\_4082EE  
    dd offset unk\_4085FD  
    dd offset loc\_4080F0  
    dd offset unk\_4084C5  
    dd offset loc\_40813C  
    dd offset unk\_40851F  
    dd offset unk\_4086A0  
    dd offset unk\_40862D  
    dd offset unk\_4083F2  
    dd offset unk\_40859D  
    dd offset unk\_408240  
    dd offset aXfP    ; "Xf&?  
    dd offset unk\_408567  
    dd offset unk\_40854C  
    dd offset unk\_408528  
    dd offset unk\_408543  
    dd offset unk\_40860D  
    dd offset loc\_4086C3  
    dd offset unk\_40820A  
    dd offset loc\_4080AC  
    dd offset unk\_4085CD  
    dd offset unk\_4081A2  
    dd offset unk\_4086BC  
    dd offset loc\_408135  
    dd offset unk\_4082FF  
    dd offset v\_pdi    ; Push Dword in  data area by Index  
    dd offset unk\_408357  
    dd offset loc\_408166  
    dd offset unk\_40839A  
    dd offset unk\_408605  
    dd offset unk\_4085D5  
    dd offset unk\_408528  
    dd offset unk\_4084E9  
    dd offset unk\_408199  
    dd offset unk\_40842A  
    dd offset unk\_408702  
    dd offset unk\_4085AC  
    dd offset unk\_4085DD  
    dd offset unk\_408582  
    dd offset loc\_40803E  
    dd offset unk\_408665  
    dd offset unk\_4085BD  
    dd offset unk\_408655  
    dd offset loc\_408055  
    dd offset unk\_4084FB  
    dd offset aXfep    ; "Xfe?  
    dd offset loc\_408270  
    dd offset unk\_4083F2  
    dd offset unk\_408382  
    dd offset loc\_40804F  
    dd offset loc\_4080D7  
    dd offset loc\_4082C0  
    dd offset unk\_408321  
    dd offset unk\_408516  
    dd offset v\_na2    ; not &  and then save DWORD  
    dd offset loc\_40808E  
    dd offset unk\_4081FA  
    dd offset unk\_408235  
    dd offset unk\_4081FA  
    dd offset unk\_408498  
    dd offset unk\_408339  
    dd offset unk\_408675  
    dd offset loc\_4080D7  
    dd offset loc\_40808E  
    dd offset loc\_408135  
    dd offset loc\_408038  
    dd offset unk\_408220  
    dd offset unk\_4086B5  
    dd offset loc\_408126  
    dd offset aXf6p    ; "Xf6?  
    dd offset unk\_408615  
    dd offset loc\_40805B  
    dd offset unk\_40825A  
    dd offset unk\_40820A  
    dd offset loc\_408081  
    dd offset unk\_408635  
    dd offset unk\_4083E0  
    dd offset unk\_4085A6  
    dd offset unk\_408579  
    dd offset unk\_408312  
    dd offset loc\_408049  
    dd offset unk\_4084CE  
    dd offset unk\_408284  
    dd offset loc\_408066  
    dd offset unk\_408625  
    dd offset unk\_4086A0  
    dd offset loc\_408110  
    dd offset unk\_4086B5  
    dd offset unk\_408655  
    dd offset unk\_40855E  
    dd offset loc\_4080F0  
    dd offset unk\_40862D  
    dd offset unk\_408570  
    dd offset unk\_40828F  
    dd offset aXfdp    ; "Xfd?  
    dd offset v\_na2    ; not &  and then save DWORD  
    dd offset loc\_4080D7  
    dd offset unk\_408215  
    dd offset v\_oax+1  ; pop ax  
    dd offset unk\_408625  
    dd offset unk\_408699  
    dd offset unk\_40861D  
    dd offset loc\_408270  
    dd offset loc\_40812E  
    dd offset unk\_4086CA  
    dd offset unk\_408486  
    dd offset unk\_40832C  
    dd offset unk\_4085E5  
    dd offset unk\_40867D  
    dd offset unk\_40868B  
    dd offset unk\_4086DF  
    dd offset unk\_4081C4  
    dd offset unk\_408582  
    dd offset unk\_40824B  
    dd offset v\_exit  
    dd offset unk\_408357  
    dd offset unk\_4083C5  
    dd offset unk\_40822A  
    dd offset unk\_4082FF  
    dd offset unk\_408B09  
    dd offset unk\_4086AE  
    dd offset unk\_40853A  
    dd offset unk\_4084E0  
    dd offset unk\_4086BC  
    dd offset loc\_4080A2  
    dd offset loc\_408135  
    dd offset unk\_4085BD  
    dd offset unk\_408279  
    dd offset loc\_40811B  
    dd offset unk\_4086ED  
    dd offset unk\_408635  
    dd offset loc\_40817F  
unk\_408B09  db  58h  ; X    ; DATA XREF: .vmp0:00408AD5o  
    db  61h  ; a  
    db  9Dh  ; ?  
    db 0CBh  ; ?  
  
v\_exit:          ; DATA XREF: .vmp0:00408AC1o  
    pop  eax  
    popa  
    popf  
    retn  
  
    db  8Dh  ; ?  
    db  40h  ; @  
    db    0  
  
    ; 这里就是数据区  
  
vmp\_banner\_and\_vcontext  db '\[ VMProtect v 1.01 (C) PolyTech |  http://www.polytech.ural.'  
          ; DATA XREF: execute\_vmp\_engine-B7Fo  
    db 'ru \]謧琣 y)K'  
    db    3  
    db  15h  
    db 0C8h  ; ?  
    db 0E4h  ; ?  
    db  9Ah  ; ?  
    db 0B0h  ; ?  
    db  61h  ; a  
    db  81h  ; ?  
    db  34h  ; 4  
    db  50h  ; P  
    db 0FFh  
    db  23h  ; #  
    db 0D0h  ; ?  
    db 0ACh  ; ?  
    db    4  
    db  73h  ; s  
    db 0CFh  ; ?  
    db 0BFh  ; ?  
    db 0FFh  
    db  59h  ; Y  
    db  31h  ; 1  
    db  0Bh  
    db  1Eh  
    db 0D0h  ; ?  
    db  0Ch  
    db  69h  ; i  
    db 0D8h  ; ?  
    db 0AFh  ; ?  
    db  42h  ; B  
    db  1Bh  
    db 0AAh  ; ?  
    db  87h  ; ?  
    db  0Eh  
    db 0E9h  ; ?  
    db  74h  ; t  
    db  54h  ; T  
    db 0D5h  ; ?  
    db 0B2h  ; ?  
    db  39h  ; 9  
    db  1Bh  
    db  98h  ; ?  
    db  72h  ; r  
    db 0FFh  
    db 0DEh  ; ?  
    db  61h  ; a  
    db  37h  ; 7  
    db  25h  ; %  
  
; 圹圹圹圹圹圹圹?S U B  R O U T  I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?  
  
  
execute\_vmp\_engine proc  near    ; CODE XREF: vmp\_stolen\_code\_entryj  
  
; FUNCTION CHUNK AT 00408000 SIZE 00000022 BYTES  
  
    push  (offset  vmp\_banner\_and\_vcontext+40h)  
    jmp  vmp\_pcode\_engine  
  
execute\_vmp\_engine endp 

 4dd466d0e1e0cc123afbe504da2462ca