
# VMProtect 虚拟机小窥

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-602.htm`](https://www.pediy.com/kssd/pediy07/pediy7-602.htm)

最近太颓废了，写点东西发泄一下。

究竟它是不是一个壳我也说不清楚，我不是很会用，随便加了个 `notepad` 看看。

P.S. 我不是很会起名字，也没有检查那些指令是不是准确，不过这些可能会动态改变，看看就行了。 :D

`vmp_pcode_engine`:

```
    pushf       ; 这些代码看起来是保存符号和寄存器的,
                ; 但若往后跟几步便会发现这里顺序把
                ;
                ; EFLAG
                ; EAX
                ; ECX
                ; EDX
                ; EBX
                ; ESP
                ; EBP
                ; ESI
                ; EDI
                ;
                ; 入栈,  随后用伪指令弹出构造自身 CONTEXT 环境
                ;
    pusha
    push  0     ; 再压一个 0 进堆栈
    mov  esi, [esp+28h]  ; -> 当前代码块伪指令表
                ;
                ; 00408B54  D6 82 AC 61  79 29 4B 03 15 C8 E4 9A  B0 61 81 34  謧琣 y)K 蠕毎 a?
                ; 00408B64  50 FF 23 D0  AC 04 73 CF BF FF 59 31  0B 1E D0 0C  P�#鞋 s 峡�Y1
                ?
                ; 00408B74  69 D8 AF 42  1B AA 87 0E E9 74 54 D5  B2 39 1B 98  i 丿 B 獓閠 T 詹 9
                ;
    mov  edi, offset vmp_banner_and_vcontext ; "\[ VMProtect v 1.01 (C) PolyTech |  http"...
    cld
    mov  ebx, esi  ; 指向 PCODE

next_opcode:
    lodsb      ; 取一个伪指令
    add  al, bl    ; 纯粹是解码, 用了会慢一些
    add  bl, al
    movzx  eax, al
    jmp  ds:pcode_parse_table[eax*4] ; 执行表中对应索引的指令, 与 VB 手法一样
                ;
                ; 我给一些指令起了名字后这段表的意思是
                ;
                ; v_odi    00     ; 用数据区的 CONTEXT 保存当前环境
                ; v_odi    09
                ; v_odi    04
                ; v_odi    0C
                ; v_odi    07
                ; v_odi    0A
                ; v_odi    05
                ; v_odi    07
                ; v_odi    03
                ; v_odi    01
                ; v_oax
                ; v_pd     97     ; 一个加密后的地址
                ; v_pd     00     ; 密钥
                ; v_na2        ;\\按照 na2 指令解码地址并入栈
                ; v_pda        ;/
                ; v_pdi    01     ; 把保存的 CONTEXT 映射回 CPU
                ; v_pdi    03
                ; v_pdi    07
                ; ....
                ; v_pdi    00
                ; v_exit     ; 返回
                ;
v_pdi:          ; DATA XREF: .vmp0:0040895Do
    lodsb      ; Push Dword in  data area by Index
    add  al, bl    ; 操作数也要解码才行
    add  bl, al
    push  dword ptr [edi+eax*4]
    jmp  short next_opcode ; 取一个伪指令
v_pb:          ; DATA XREF: .vmp0:00408759o
    lodsd      ; Push Byte (as  DWORD, Lo used only)
    add  al, bl
    add  bl, al
    push  eax
    jmp  short next_opcode ; 取一个伪指令
loc_408034:       ; DATA XREF: .vmp0:004088BDo
                ; .vmp0:00408769o ...
    push  sp
    jmp  short next_opcode ; 取一个伪指令
loc_408038:       ; DATA XREF: .vmp0:004089FDo
    pop  eax
    push  dword ptr gs:[eax]
    jmp  short next_opcode ; 取一个伪指令
loc_40803E:       ; DATA XREF: .vmp0:00408995o
    pop  eax
    push  dword ptr fs:[eax]
    jmp  short next_opcode ; 取一个伪指令
v_pda:          ; DATA XREF: .vmp0:00408755o
    pop  eax    ; Pop address and Push Dword in  address
    push  dword ptr [eax]
    jmp  short next_opcode ; 取一个伪指令
    ; 省略一些
v_na2:          ; DATA XREF: .vmp0:004089D1o
                ; .vmp0:00408A75o
    pop  eax    ; not &  and then save DWORD
    not  eax
    not  dword ptr [esp]
    and  [esp], eax
    jmp  next_opcode  ; 取一个伪指令
loc_4082C0:       ; DATA XREF: .vmp0:00408791o
                ; .vmp0:004089C5o
    pop  eax
    not  eax
    not  dword ptr [esp]
    and  [esp], eax
    pushfw
    jmp  next_opcode  ; 取一个伪指令
    ; 省略一些
    ; 每个 DWORD 对应一条指令解析器
pcode_parse_table dd offset unk_40829C  ; DATA XREF: execute_vmp_engine-B6Fr
    dd offset unk_40864D
    dd offset unk_4085E5
    dd offset unk_408699
    dd offset unk_40838D
    dd offset unk_408364
    dd offset unk_4082D0
    dd offset unk_408373
    dd offset unk_4086F4
    dd offset unk_4085BD
    dd offset unk_40853A
    dd offset loc_408144
    dd offset unk_408465
    dd offset unk_408458
    dd offset unk_408437
    dd offset unk_408531
    dd offset unk_408504
    dd offset unk_408269
    dd offset unk_408347
    dd offset v_pda    ; Pop address and Push Dword in  address
    dd offset v_pb     ; Push Byte (as  DWORD, Lo used only)
    dd offset unk_408261
    dd offset unk_4085F5
    dd offset loc_408187
    dd offset loc_408034+1
    dd offset loc_408066
    dd offset loc_40807A
    dd offset unk_40828F
    dd offset unk_4081EB
    dd offset unk_40860D
    dd offset loc_4080A2
    dd offset unk_408594
    dd offset unk_408692
    dd offset loc_408066
    dd offset loc_4082C0
    dd offset loc_4080CB
    dd offset unk_4086A7
    dd offset unk_40855E
    dd offset unk_4084E0
    dd offset loc_408088
    dd offset unk_40840B
    dd offset loc_408098
    dd offset v_odi    ; pOp dword on stack to  Dword in data area by Index
    dd offset unk_4082EE
    dd offset unk_4086D1
    dd offset unk_408528
    dd offset unk_4081A2
    dd offset unk_4084AA
    dd offset loc_408034+1
    dd offset unk_408498
    dd offset unk_408199
    dd offset unk_4086D8
    dd offset unk_408692
    dd offset loc_40812E
    dd offset unk_40850D
    dd offset loc_4080AC
    dd offset unk_408570
    dd offset unk_4084BC
    dd offset unk_40848F
    dd offset loc_408070
    dd offset loc_40817F
    dd offset loc_40805B
    dd offset unk_4083A8
    dd offset unk_4086DF
    dd offset unk_4083D4
    dd offset unk_408486
    dd offset unk_4082DE
    dd offset unk_408253
    dd offset loc_40816D
    dd offset unk_40865D
    dd offset unk_40824B
    dd offset unk_40853A
    dd offset unk_4084D7
    dd offset unk_40858B
    dd offset unk_40866D
    dd offset unk_40867D
    dd offset unk_4086FB
    dd offset unk_4084F2
    dd offset unk_408477
    dd offset unk_408190
    dd offset unk_408594
    dd offset loc_408270
    dd offset loc_4080E5
    dd offset loc_408166+1
    dd offset unk_40863D
    dd offset loc_4080FB
    dd offset unk_4084B3
    dd offset unk_4086E6
    dd offset loc_4080F0
    dd offset unk_40841D
    dd offset unk_408449
    dd offset unk_408449
    dd offset unk_4084A1
    dd offset unk_4083B8
    dd offset loc_4080BB
    dd offset unk_408240
    dd offset unk_408253
    dd offset unk_408555
    dd offset loc_40805B
    dd offset unk_4085C5
    dd offset unk_4084D7
    dd offset unk_4085B5
    dd offset unk_4082A6
    dd offset unk_4086D1
    dd offset unk_408364
    dd offset unk_4083FF
    dd offset unk_408645
    dd offset unk_4085ED
    dd offset unk_408253
    dd offset loc_408034
    dd offset loc_408176
    dd offset loc_4080CB
    dd offset unk_4084BC
    dd offset loc_4080F0
    dd offset loc_4080CB
    dd offset unk_4085A6
    dd offset unk_408531
    dd offset unk_4086CA
    dd offset loc_408105
    dd offset v_oax    ; pop ax
    dd offset aXf_p    ; "Xf.?
    dd offset unk_408684
    dd offset v_oax    ; pop ax
    dd offset loc_40814B
    dd offset unk_4082EE
    dd offset unk_4085FD
    dd offset loc_4080F0
    dd offset unk_4084C5
    dd offset loc_40813C
    dd offset unk_40851F
    dd offset unk_4086A0
    dd offset unk_40862D
    dd offset unk_4083F2
    dd offset unk_40859D
    dd offset unk_408240
    dd offset aXfP     ; "Xf&?
    dd offset unk_408567
    dd offset unk_40854C
    dd offset unk_408528
    dd offset unk_408543
    dd offset unk_40860D
    dd offset loc_4086C3
    dd offset unk_40820A
```



```
dd offset loc_4080AC
dd offset unk_4085CD
dd offset unk_4081A2
dd offset unk_4086BC
dd offset loc_408135
dd offset unk_4082FF
dd offset v_pdi    ; Push Dword in  data area by Index
dd offset unk_408357
dd offset loc_408166
dd offset unk_40839A
dd offset unk_408605
dd offset unk_4085D5
dd offset unk_408528
dd offset unk_4084E9
dd offset unk_408199
dd offset unk_40842A
dd offset unk_408702
dd offset unk_4085AC
dd offset unk_4085DD
dd offset unk_408582
dd offset loc_40803E
dd offset unk_408665
dd offset unk_4085BD
dd offset unk_408655
dd offset loc_408055
dd offset unk_4084FB
dd offset aXfep    ; "Xfe?
dd offset loc_408270
dd offset unk_4083F2
dd offset unk_408382
dd offset loc_40804F
dd offset loc_4080D7
dd offset loc_4082C0
dd offset unk_408321
dd offset unk_408516
dd offset v_na2    ; not &  and then save DWORD
dd offset loc_40808E
dd offset unk_4081FA
dd offset unk_408235
dd offset unk_4081FA
dd offset unk_408498
dd offset unk_408339
dd offset unk_408675
dd offset loc_4080D7
dd offset loc_40808E
dd offset loc_408135
dd offset loc_408038
dd offset unk_408220
dd offset unk_4086B5
dd offset loc_408126
dd offset aXf6p    ; "Xf6?
dd offset unk_408615
dd offset loc_40805B
dd offset unk_40825A
dd offset unk_40820A
dd offset loc_408081
dd offset unk_408635
dd offset unk_4083E0
dd offset unk_4085A6
dd offset unk_408579
dd offset unk_408312
dd offset loc_408049
dd offset unk_4084CE
dd offset unk_408284
dd offset loc_408066
dd offset unk_408625
dd offset unk_4086A0
dd offset loc_408110
dd offset unk_4086B5
dd offset unk_408655
dd offset unk_40855E
dd offset loc_4080F0
dd offset unk_40862D
dd offset unk_408570
dd offset unk_40828F
dd offset aXfdp    ; "Xfd?
dd offset v_na2    ; not &  and then save DWORD
dd offset loc_4080D7
dd offset unk_408215
dd offset v_oax+1  ; pop ax
dd offset unk_408625
dd offset unk_408699
dd offset unk_40861D
dd offset loc_408270
dd offset loc_40812E
dd offset unk_4086CA
dd offset unk_408486
dd offset unk_40832C
dd offset unk_4085E5
dd offset unk_40867D
dd offset unk_40868B
dd offset unk_4086DF
dd offset unk_4081C4
dd offset unk_408582
dd offset unk_40824B
dd offset v_exit
dd offset unk_408357
dd offset unk_4083C5
dd offset unk_40822A
dd offset unk_4082FF
dd offset unk_408B09
dd offset unk_4086AE
dd offset unk_40853A
dd offset unk_4084E0
dd offset unk_4086BC
dd offset loc_4080A2
dd offset loc_408135
dd offset unk_4085BD
dd offset unk_408279
dd offset loc_40811B
dd offset unk_4086ED
dd offset unk_408635
dd offset loc_40817F

unk_408B09  db  58h  ; X    ; DATA XREF: .vmp0:00408AD5o
  db  61h  ; a
  db  9Dh  ; ?
  db 0CBh  ; ?

v_exit:         ; DATA XREF: .vmp0:00408AC1o
  pop  eax
  popa
  popf
  retn
  db  8Dh  ; ?
  db  40h  ; @
  db    0
  ; 这里就是数据区

vmp_banner_and_vcontext  db '\[ VMProtect v 1.01 (C) PolyTech |  http://www.polytech.ural.'
          ; DATA XREF: execute_vmp_engine-B7Fo
  db 'ru ]謧琣 y)K'
  db    3
  db  15h
  db 0C8h  ; ?
  db 0E4h  ; ?
  db  9Ah  ; ?
  db 0B0h  ; ?
  db  61h  ; a
  db  81h  ; ?
  db  34h  ; 4
  db  50h  ; P
  db 0FFh
  db  23h  ; #
  db 0D0h  ; ?
  db 0ACh  ; ?
  db    4
  db  73h  ; s
  db 0CFh  ; ?
  db 0BFh  ; ?
  db 0FFh
  db  59h  ; Y
  db  31h  ; 1
  db  0Bh
  db  1Eh
  db 0D0h  ; ?
  db  0Ch
  db  69h  ; i
  db 0D8h  ; ?
  db 0AFh  ; ?
  db  42h  ; B
  db  1Bh
  db 0AAh  ; ?
  db  87h  ; ?
  db  0Eh
  db 0E9h  ; ?
  db  74h  ; t
  db  54h  ; T
  db 0D5h  ; ?
  db 0B2h  ; ?
  db  39h  ; 9
  db  1Bh
  db  98h  ; ?
  db  72h  ; r
  db 0FFh
  db 0DEh  ; ?
  db  61h  ; a
  db  37h  ; 7
  db  25h  ; %

; 圹圹圹圹圹圹圹?S U B  R O U T  I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?

execute_vmp_engine proc  near    ; CODE XREF: vmp_stolen_code_entryj
; FUNCTION CHUNK AT 00408000 SIZE 00000022 BYTES
  push  (offset  vmp_banner_and_vcontext+40h)
  jmp  vmp_pcode_engine

execute_vmp_engine endp 
```
`4dd466d0e1e0cc123afbe504da2462ca`
