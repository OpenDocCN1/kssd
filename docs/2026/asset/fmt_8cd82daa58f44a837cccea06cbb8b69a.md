
# [分享]VMP1.20的Opcode识别脚本

主题：VMP1.20的Opcode识别脚本  
作者：DarkBull  
邮箱：DarkBull#126.com  

前言：前一阵学习了wangdell的《VMProtect1.2x总结》，受益非浅。于是照着写了个OD脚本，大部分Opcode名称与wangdell的命名相同，个别不同的纯属个人习惯，仅供参考。由于水平有限，接触的VMP程序程序不多，难免有错误的地方，请高手多多指教。

``` 
// VMProtect1.2x Find OPcode Script

// Writen By DarkBull

data:

    var codebase
    var codesize
    var init
    var loopep
    var optbl
    var opaddr
    var opcode
    var opcnt
    var findcnt
    var loopcnt
    var temp

code:

    gmi eip,CODEBASE
    mov codebase,$RESULT
    gmi eip,CODESIZE
    mov codesize,$RESULT

findinit:

    find codebase,#??????????????????68000000008B742428#
    cmp $RESULT,0
    jne vminit

input1:

    mov $RESULT,0
    ask "Plese Input VMP Base Address:"
    cmp $RESULT,0
    je input1
    find $RESULT,#??????????????????68000000008B742428#
    cmp $RESULT,0
    je error

vminit:

    mov temp,$RESULT
    lbl $RESULT,"VM_Init"
    mov init,$RESULT

findep:

    find temp,#033424#
    cmp $RESULT,0
    je error
    add $RESULT,3
    mov temp,$RESULT
    lbl $RESULT,"VM_LoopEP"
    mov loopep,$RESULT

findoptb:

    find temp,#FF??85#
    cmp $RESULT,0
    je error
    add $RESULT,3
    mov optbl,[$RESULT]
    lbl optbl,"VM_OP_Table"

// ********************************************************************
findop:

    mov opaddr,[optbl]
    gn opaddr
    cmp $RESULT,0
    jne findnext
    inc findcnt
    mov opcode,[opaddr],1
    cmp opcode,0F
    je op2b
    cmp opcode,54
    je VM_PUSH_ESP
    cmp opcode,58
    jb find@1
    cmp opcode,5A
    ja find@1
    jmp popreg

find@1:

    cmp opcode,5B
    je ret1
    cmp opcode,5C
    je VM_POP_ESP
    cmp opcode,5D
    jb find@2
    cmp opcode,5F
    ja find@2
    jmp ret1

find@2:

    cmp opcode,66
    je prefix
    cmp opcode,88
    je lodsb1
    cmp opcode,89
    je lodsdw1
    cmp opcode,8A
    je lodsb1
    cmp opcode,8B
    je lodsdw1
    cmp opcode,09B
    je VM_WAIT
    cmp opcode,09D
    je ret1
    cmp opcode,0AC
    je lodsb1
    cmp opcode,0AD
    je lodsdw1
    cmp opcode,0C7
    je ret2
    cmp opcode,0D8
    jb find@3
    cmp opcode,0DF
    ja find@3
    jmp fpu

find@3:

    cmp opcode,0F7
    je norw
    jmp error

op2b:

    mov opcode,[opaddr+1],1
    cmp opcode,20
    je spr1
    cmp opcode,21
    je spr2
    cmp opcode,0B6
    je lodsb1
    cmp opcode,0B7
    je lodsw1
    jmp error

spr1:

    mov opcode,[opaddr+2],1
    shr opcode,3
    and opcode,7
    cmp opcode,0
    je VM_PUSH_CR0
    cmp opcode,1
    je VM_PUSH_CR1
    cmp opcode,2
    je VM_PUSH_CR2
    cmp opcode,3
    je VM_PUSH_CR3
    cmp opcode,4
    je VM_PUSH_CR4
    cmp opcode,5
    je VM_PUSH_CR5
    cmp opcode,6
    je VM_PUSH_CR6
    cmp opcode,7
    je VM_PUSH_CR7
    jmp error

spr2:

    mov opcode,[opaddr+2],1
    shr opcode,3
    and opcode,7
    cmp opcode,0
    je VM_PUSH_DR0
    cmp opcode,1
    je VM_PUSH_DR1
    cmp opcode,2
    je VM_PUSH_DR2
    cmp opcode,3
    je VM_PUSH_DR3
    cmp opcode,4
    je VM_PUSH_DR4
    cmp opcode,5
    je VM_PUSH_DR5
    cmp opcode,6
    je VM_PUSH_DR6
    cmp opcode,7
    je VM_PUSH_DR7
    jmp error

popreg:

    mov opcode,[opaddr+1],1
    cmp opcode,01
    je adddw
    cmp opcode,0F
    je pop2b
    cmp opcode,26
    je mov1
    cmp opcode,28
    je popsub
    cmp opcode,2E
    je mov1
    cmp opcode,36
    je mov1
    cmp opcode,3E
    je mov1
    cmp opcode,58
    jb pop@1
    cmp opcode,5A
    ja pop@1
    jmp pop2

pop@1:

    cmp opcode,5B
    jb pop@2
    cmp opcode,5F
    ja pop@2
    jmp ret1

pop@2:

    cmp opcode,64
    je mov1
    cmp opcode,65
    je mov1
    cmp opcode,66
    je poppref
    cmp opcode,88
    je popmovb
    cmp opcode,89
    je popmovdw
    cmp opcode,8A
    je popmovb
    cmp opcode,8B
    je popmovdw
    cmp opcode,8F
    je VM_MOV_DS1_S2
    cmp opcode,9D
    je ret1
    cmp opcode,0E9
    je gr1
    cmp opcode,0F7
    je nordw
    cmp opcode,0FF
    je VM_MOV_S1_DS1
    jmp error

adddw:

    mov opcode,[opaddr+2],2
    cmp opcode,2404
    je add@1
    cmp opcode,240C
    je add@1
    cmp opcode,2414
    je add@1
    jmp error

add@1:

    mov opcode,[opaddr+4],2
    cmp opcode,9C66
    je VM_ADD_F
    jmp VM_ADD

pop2b:

    mov opcode,[opaddr+2],1
    cmp opcode,22
    je spr3
    cmp opcode,23
    je spr4
    jmp error

spr3:

    mov opcode,[opaddr+3],1
    shr opcode,3
    and opcode,7
    cmp opcode,0
    je VM_POP_CR0
    cmp opcode,1
    je VM_POP_CR1
    cmp opcode,2
    je VM_POP_CR2
    cmp opcode,3
    je VM_POP_CR3
    cmp opcode,4
    je VM_POP_CR4
    cmp opcode,5
    je VM_POP_CR5
    cmp opcode,6
    je VM_POP_CR6
    cmp opcode,7
    je VM_POP_CR7
    jmp error

spr4:

    mov opcode,[opaddr+3],1
    shr opcode,3
    and opcode,7
    cmp opcode,0
    je VM_POP_DR0
    cmp opcode,1
    je VM_POP_DR1
    cmp opcode,2
    je VM_POP_DR2
    cmp opcode,3
    je VM_POP_DR3
    cmp opcode,4
    je VM_POP_DR4
    cmp opcode,5
    je VM_POP_DR5
    cmp opcode,6
    je VM_POP_DR6
    cmp opcode,7
    je VM_POP_DR7
    jmp error

mov1:

    mov opcode,[opaddr+1],2
    cmp opcode,08A26
    je VM_MOVB_S1_ES1
    cmp opcode,08A2E
    je VM_MOVB_S1_CS1
    cmp opcode,08A36
    je VM_MOVB_S1_SS1
    cmp opcode,08A3E
    je VM_MOVB_S1_DS1
    cmp opcode,08A64
    je VM_MOVB_S1_FS1
    cmp opcode,08A65
    je VM_MOVB_S1_GS1
    cmp opcode,8F26
    je VM_MOV_ES1_S2
    cmp opcode,8F2E
    je VM_MOV_CS1_S2
    cmp opcode,8F36
    je VM_MOV_SS1_S2
    cmp opcode,8F3E
    je VM_MOV_DS1_S2
    cmp opcode,8F64
    je VM_MOV_FS1_S2
    cmp opcode,8F65
    je VM_MOV_GS1_S2
    cmp opcode,0FF26
    je VM_MOV_S1_ES1
    cmp opcode,0FF2E
    je VM_MOV_S1_CS1
    cmp opcode,0FF36
    je VM_MOV_S1_SS1
    cmp opcode,0FF3E
    je VM_MOV_S1_DS1
    cmp opcode,0FF64
    je VM_MOV_S1_FS1
    cmp opcode,0FF65
    je VM_MOV_S1_GS1
    jmp error

popsub:

   mov opcode,[opaddr+3],2
    cmp opcode,08A26
    je VM_MOVB_S1_ES1
    cmp opcode,08A2E
    je VM_MOVB_S1_CS1
    cmp opcode,08A36
    je VM_MOVB_S1_SS1
    cmp opcode,08A3E
    je VM_MOVB_S1_DS1
    cmp opcode,08A64
    je VM_MOVB_S1_FS1
    cmp opcode,08A65
    je VM_MOVB_S1_GS1
    jmp error

pop2:

    mov opcode,[opaddr+2],1
    cmp opcode,58
    jb pop2@1
    cmp opcode,5A
    ja pop2@1
    jmp pop3

pop2@1:

    cmp opcode,5B
    jb pop2@2
    cmp opcode,5F
    ja pop2@2
    jmp ret1

pop2@2:

    cmp opcode,66
    je pop2pr
    cmp opcode,9D
    je ret1
    cmp opcode,0F7
    je muldw
    jmp error

pop3:

    mov opcode,[opaddr+3],1
    cmp opcode,5B
    jb pop3@1
    cmp opcode,5F
    ja pop3@1
    jmp ret1

pop3@1:

    cmp opcode,9D
    je ret1
    cmp opcode,0F7
    je divdw
    jmp error

divdw:

    mov opcode,[opaddr+4],1
    shr opcode,3
    and opcode,7
```
