
# VMP1.20 的 Opcode 识别脚本

> 原文：[`www.pediy.com/kssd/pediy10/58462.html`](https://www.pediy.com/kssd/pediy10/58462.html)

主题：`VMP1.20` 的 `Opcode` 识别脚本  
作者：`DarkBull`  
邮箱：`DarkBull#126.com`  

前言：前一阵学习了 `wangdell` 的＜`VMProtect1.2x` 总结＞，受益非浅。于是照着写了个 `OD` 脚本，大部分 `Opcode` 名称与 `wangdell` 的命名相同，个别不同的纯属个人习惯，仅供参考。由于水平有限，接触的 `VMP` 程序程序不多，难免有错误的地方，请高手多多指教。

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

    lbl $RESULT,"VM\_Init"  

    mov init,$RESULT  

findep:  

    find temp,#033424#  

    cmp $RESULT,0  

    je error  

    add $RESULT,3  

    mov temp,$RESULT  

    lbl $RESULT,"VM\_LoopEP"  

    mov loopep,$RESULT  

findoptb:  

    find temp,#FF??85#  

    cmp $RESULT,0  

    je error  

    add $RESULT,3  

    mov optbl,\[$RESULT\]  

    lbl optbl,"VM\_OP\_Table"  

// *****************************************************************************************  

findop:  

    mov opaddr,\[optbl\]  

    gn opaddr  

    cmp $RESULT,0  

    jne findnext  

    inc findcnt  

    mov opcode,\[opaddr\],1  

    cmp opcode,0F  

    je op2b  

    cmp opcode,54  

    je VM\_PUSH\_ESP  

    cmp opcode,58  

    jb find@1  

    cmp opcode,5A  

    ja find@1  

    jmp popreg  

find@1:  

    cmp opcode,5B  

    je ret1  

    cmp opcode,5C  

    je VM\_POP\_ESP  

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

    je VM\_WAIT  

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

    mov opcode,\[opaddr+1\],1  

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

    mov opcode,\[opaddr+2\],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,0  

    je VM\_PUSH\_CR0  

    cmp opcode,1  

    je VM\_PUSH\_CR1  

    cmp opcode,2  

    je VM\_PUSH\_CR2  

    cmp opcode,3  

    je VM\_PUSH\_CR3  

    cmp opcode,4  

    je VM\_PUSH\_CR4  

    cmp opcode,5  

    je VM\_PUSH\_CR5  

    cmp opcode,6  

    je VM\_PUSH\_CR6  

    cmp opcode,7  

    je VM\_PUSH\_CR7  

    jmp error  

spr2:  

    mov opcode,\[opaddr+2\],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,0  

    je VM\_PUSH\_DR0  

    cmp opcode,1  

    je VM\_PUSH\_DR1  

    cmp opcode,2  

    je VM\_PUSH\_DR2  

    cmp opcode,3  

    je VM\_PUSH\_DR3  

    cmp opcode,4  

    je VM\_PUSH\_DR4  

    cmp opcode,5  

    je VM\_PUSH\_DR5  

    cmp opcode,6  

    je VM\_PUSH\_DR6  

    cmp opcode,7  

    je VM\_PUSH\_DR7  

    jmp error  

popreg:  

    mov opcode,\[opaddr+1\],1  

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

    je VM\_MOV\_DS1\_S2  

    cmp opcode,9D  

    je ret1  

    cmp opcode,0E9  

    je gr1  

    cmp opcode,0F7  

    je nordw  

    cmp opcode,0FF  

    je VM\_MOV\_S1\_DS1  

    jmp error  

adddw:  

    mov opcode,\[opaddr+2\],2  

    cmp opcode,2404  

    je add@1  

    cmp opcode,240C  

    je add@1  

    cmp opcode,2414  

    je add@1  

    jmp error  

add@1:  

    mov opcode,\[opaddr+4\],2  

    cmp opcode,9C66  

    je VM\_ADD\_F  

    jmp VM\_ADD  

pop2b:  

    mov opcode,\[opaddr+2\],1  

    cmp opcode,22  

    je spr3  

    cmp opcode,23  

    je spr4  

    jmp error  

spr3:  

    mov opcode,\[opaddr+3\],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,0  

    je VM\_POP\_CR0  

    cmp opcode,1  

    je VM\_POP\_CR1  

    cmp opcode,2  

    je VM\_POP\_CR2  

    cmp opcode,3  

    je VM\_POP\_CR3  

    cmp opcode,4  

    je VM\_POP\_CR4  

    cmp opcode,5  

    je VM\_POP\_CR5  

    cmp opcode,6  

    je VM\_POP\_CR6  

    cmp opcode,7  

    je VM\_POP\_CR7  

    jmp error  

spr4:  

    mov opcode,\[opaddr+3\],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,0  

    je VM\_POP\_DR0  

    cmp opcode,1  

    je VM\_POP\_DR1  

    cmp opcode,2  

    je VM\_POP\_DR2  

    cmp opcode,3  

    je VM\_POP\_DR3  

    cmp opcode,4  

    je VM\_POP\_DR4  

    cmp opcode,5  

    je VM\_POP\_DR5  

    cmp opcode,6  

    je VM\_POP\_DR6  

    cmp opcode,7  

    je VM\_POP\_DR7  

    jmp error  

mov1:  

    mov opcode,\[opaddr+1\],2  

    cmp opcode,08A26  

    je VM\_MOVB\_S1\_ES1  

    cmp opcode,08A2E  

    je VM\_MOVB\_S1\_CS1  

    cmp opcode,08A36  

    je VM\_MOVB\_S1\_SS1  

    cmp opcode,08A3E  

    je VM\_MOVB\_S1\_DS1  

    cmp opcode,08A64  

    je VM\_MOVB\_S1\_FS1  

    cmp opcode,08A65  

    je VM\_MOVB\_S1\_GS1  

    cmp opcode,8F26  

    je VM\_MOV\_ES1\_S2  

    cmp opcode,8F2E  

    je VM\_MOV\_CS1\_S2  

    cmp opcode,8F36  

    je VM\_MOV\_SS1\_S2  

    cmp opcode,8F3E  

    je VM\_MOV\_DS1\_S2  

    cmp opcode,8F64  

    je VM\_MOV\_FS1\_S2  

    cmp opcode,8F65  

    je VM\_MOV\_GS1\_S2  

    cmp opcode,0FF26  

    je VM\_MOV\_S1\_ES1  

    cmp opcode,0FF2E  

    je VM\_MOV\_S1\_CS1  

    cmp opcode,0FF36  

    je VM\_MOV\_S1\_SS1  

    cmp opcode,0FF3E  

    je VM\_MOV\_S1\_DS1  

    cmp opcode,0FF64  

    je VM\_MOV\_S1\_FS1  

    cmp opcode,0FF65  

    je VM\_MOV\_S1\_GS1  

    jmp error  

popsub:  

    mov opcode,\[opaddr+3\],2  

    cmp opcode,08A26  

    je VM\_MOVB\_S1\_ES1  

    cmp opcode,08A2E  

    je VM\_MOVB\_S1\_CS1  

    cmp opcode,08A36  

    je VM\_MOVB\_S1\_SS1  

    cmp opcode,08A3E  

    je VM\_MOVB\_S1\_DS1  

    cmp opcode,08A64  

    je VM\_MOVB\_S1\_FS1  

    cmp opcode,08A65  

    je VM\_MOVB\_S1\_GS1  

    jmp error  

pop2:  

    mov opcode,\[opaddr+2\],1  

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

    mov opcode,\[opaddr+3\],1  

    cmp opcode,5B  

    jb pop3@1  
```



```assembly
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

    cmp opcode,6  

    je VM_DIV  

    cmp opcode,7  

    je VM_IDIV  

    jmp error  

pop2pr:  

    mov opcode,[opaddr+3],1  

    cmp opcode,58  

    jb error  

    cmp opcode,5A  

    ja error  

        mov opcode,[opaddr+4],2  

        cmp opcode,0A50F  

        je VM_SHLD_F  

        cmp opcode,0AD0F  

        je VM_SHRD_F  

        jmp error  

muldw:  

    mov opcode,[opaddr+3],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,4  

    je VM_MUL_F  

    cmp opcode,5  

    je VM_IMUL_F  

    jmp error  

poppref:  

    mov opcode,[opaddr+2],1  

    cmp opcode,26  

    je mov2  

    cmp opcode,2E  

    je mov2  

    cmp opcode,36  

    je mov2  

    cmp opcode,3E  

    je mov2  

    cmp opcode,58  

    jb popr@1  

    cmp opcode,5A  

    ja popr@1  

    jmp poprpo  

popr@1:  

    cmp opcode,64  

    je mov2  

    cmp opcode,65  

    je mov2  

    cmp opcode,8F  

    je VM_MOVW_DS1_S2  

    cmp opcode,0FF  

    je VM_MOVW_S1_DS1  

    jmp error  

mov2:  

    mov opcode,[opaddr+2],2  

    cmp opcode,8F26  

    je VM_MOVW_ES1_S2  

    cmp opcode,8F2E  

    je VM_MOVW_CS1_S2  

    cmp opcode,8F36  

    je VM_MOVW_SS1_S2  

    cmp opcode,8F3E  

    je VM_MOVW_DS1_S2  

    cmp opcode,8F64  

    je VM_MOVW_FS1_S2  

    cmp opcode,8F65  

    je VM_MOVW_GS1_S2  

    cmp opcode,0FF26  

    je VM_MOVW_S1_ES1  

    cmp opcode,0FF2E  

    je VM_MOVW_S1_CS1  

    cmp opcode,0FF36  

    je VM_MOVW_S1_SS1  

    cmp opcode,0FF3E  

    je VM_MOVW_S1_DS1  

    cmp opcode,0FF64  

    je VM_MOVW_S1_FS1  

    cmp opcode,0FF65  

    je VM_MOVW_S1_GS1  

    mov opcode,[opaddr+2],3  

    cmp opcode,0B60F26  

    je VM_MOVB_S1_ES1  

    cmp opcode,0B60F2E  

    je VM_MOVB_S1_CS1  

    cmp opcode,0B60F36  

    je VM_MOVB_S1_SS1  

    cmp opcode,0B60F3E  

    je VM_MOVB_S1_DS1  

    cmp opcode,0B60F64  

    je VM_MOVB_S1_FS1  

    cmp opcode,0B60F65  

    je VM_MOVB_S1_GS1  

    jmp error  

poprpo:  

    mov opcode,[opaddr+3],1  

    cmp opcode,26  

    je mov3  

    cmp opcode,2E  

    je mov3  

    cmp opcode,36  

    je mov3  

    cmp opcode,3E  

    je mov3  

    cmp opcode,64  

    je mov3  

    cmp opcode,65  

    je mov3  

    cmp opcode,88  

    je VM_MOVB_DS1_S2  

    cmp opcode,0D3  

    je shdw  

    jmp error  

mov3:  

    mov opcode,[opaddr+3],2  

    cmp opcode,8826  

    je VM_MOVB_ES1_S2  

    cmp opcode,882E  

    je VM_MOVB_CS1_S2  

    cmp opcode,8836  

    je VM_MOVB_SS1_S2  

    cmp opcode,883E  

    je VM_MOVB_DS1_S2  

    cmp opcode,8864  

    je VM_MOVB_FS1_S2  

    cmp opcode,8865  

    je VM_MOVB_GS1_S2  

    jmp error  

shdw:  

    mov opcode,[opaddr+4],1  

    cmp opcode,0E0  

    je shldw  

    cmp opcode,0E2  

    je shldw  

    cmp opcode,0E8  

    je shrdw  

    cmp opcode,0EA  

    je shrdw  

    jmp error  

shldw:  

    mov opcode,[opaddr+6],2  

    cmp opcode,9C66  

    je VM_SHL_F  

    jmp VM_SHL  

shrdw:  

    mov opcode,[opaddr+6],2  

    cmp opcode,9C66  

    je VM_SHR_F  

    jmp VM_SHR  

popmovb:  

    mov opcode,[opaddr+3],2  

    cmp opcode,5066  

    je VM_MOVB_S1_DS1  

    jmp error  

popmovdw:  

    mov opcode,[opaddr+2],1  

    cmp opcode,0C4  

    je VM_POP_ESP  

    cmp opcode,0CC  

    je VM_POP_ESP  

    cmp opcode,0D4  

    je VM_POP_ESP  

    cmp opcode,0C6  

    je VM_JMP  

    cmp opcode,0CE  

    je VM_JMP  

    cmp opcode,0D6  

    je VM_JMP  

    jmp error  

gr1:  

    mov opcode,[opaddr],1  

    cmp opcode,58  

    je VM_POP_EAX  

    cmp opcode,59  

    je VM_POP_ECX  

    cmp opcode,5A  

    je VM_POP_EDX  

    jmp error  

nordw:  

    mov opcode,[opaddr+9],2  

    cmp opcode,9C66  

    je VM_NOR_F  

    jmp VM_NOR  

prefix:  

    mov opcode,[opaddr+1],1  

    cmp opcode,6  

    je VM_PUSH_ES  

    cmp opcode,7  

    je VM_POP_ES  

    cmp opcode,0E  

    je VM_PUSH_CS  

    cmp opcode,0F  

    je sr1  

    cmp opcode,16  

    je VM_PUSH_SS  

    cmp opcode,17  

    je VM_POP_SS  

    cmp opcode,1E  

    je VM_PUSH_DS  

    cmp opcode,1F  

    je VM_POP_DS  

    cmp opcode,54  

    je VM_PUSH_SP  

    cmp opcode,58  

    jb pref@1  

    cmp opcode,5A  

    ja pref@1  

    jmp prefpop  

pref@1:  

    cmp opcode,5C  

    je VM_POP_SP  

    cmp opcode,89  

    je lodsw1  

    cmp opcode,8B  

    je lodsw1  

    cmp opcode,8C  

    je sr3  

    cmp opcode,0AD  

    je lodsw1  

    jmp error  

sr1:  

    mov opcode,[opaddr+2],1  

    cmp opcode,0A0  

    je VM_PUSH_FS  

    cmp opcode,0A1  

    je VM_POP_FS  

    cmp opcode,0A8  

    je VM_PUSH_GS  

    cmp opcode,0A9  

    je VM_POP_GS  

    jmp error  

prefpop:  

    mov opcode,[opaddr+2],1  

    cmp opcode,0  

    je addb  

    cmp opcode,66  

    je prpopr  

    cmp opcode,0E9  

    je gr2  

    jmp error  

addb:  

    mov opcode,[opaddr+5],2  

    cmp opcode,9C66  

    je VM_ADDB_F  

    jmp VM_ADDB  

prpopr:  

    mov opcode,[opaddr+3],1  

    cmp opcode,1  

    je addw  

    cmp opcode,58  

    jb ppop@1  

    cmp opcode,5A  

    ja ppop@1  

    jmp prpo2  

ppop@1:  

    cmp opcode,89  

    je popsp  

    cmp opcode,8E  

    je sr2  

    jmp error  

addw:  

    mov opcode,[opaddr+6],2  

    cmp opcode,9C66  

    je VM_ADDW_F  

    jmp VM_ADDW  

prpo2:  

    mov opcode,[opaddr+4],1  

    cmp opcode,66  

    je prpo2pr  

    cmp opcode,0D2  

    je shb  

    cmp opcode,0F6  

    je prpo2ex  

prpo2pr:  

    mov opcode,[opaddr+5],1  

    cmp opcode,58  

    jb pp2p@1  

    cmp opcode,5A  

    ja pp2p@1  

    jmp prpo3  

pp2p@1:  

    cmp opcode,0D3  

    je shw  

    cmp opcode,0F7  

    je mulw  

    jmp error  

prpo3:  

    mov opcode,[opaddr+6],2  

    cmp opcode,0F766  

    je divw  

    jmp error  

divw:  

    mov opcode,[opaddr+8],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,6  

    je VM_DIVW  

    cmp opcode,7  

    je VM_IDIVW  

    jmp error  

shw:  

    mov opcode,[opaddr+6],1  

    cmp opcode,0E0  

    je shlwcode  

    cmp opcode,0E2  

    je shlwcode  

    cmp opcode,0E8  

    je shrwcode  

    cmp opcode,0EA  

    je shrwcode  

shlwcode:  

    mov opcode,[opaddr+9],2  

    cmp opcode,9C66  

    je VM_SHLW_F  

    jmp VM_SHLW  

shrwcode:  

    mov opcode,[opaddr+9],2  

    cmp opcode,9C66  

    je VM_SHRW_F  

    jmp VM_SHRW  

mulw:  

    mov opcode,[opaddr+6],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,4  

    je VM_MULW_F  

    cmp opcode,5  

    je VM_IMULW_F  

    jmp error  

shb:  

    mov opcode,[opaddr+5],1  

    cmp opcode,0E0  

    je shlb  

    cmp opcode,0E2  

    je shlb  

    cmp opcode,0E8  

    je shrb  

    cmp opcode,0EA  

    je shrb  

    jmp error  

shlb:  

    mov opcode,[opaddr+8],2  

    cmp opcode,9C66  

    je VM_SHLB_F  

    jmp VM_SHLB  

shrb:  

    mov opcode,[opaddr+8],2  

    cmp opcode,9C66  

    je VM_SHRB_F  

    jmp VM_SHRB  

prpo2ex:  

    mov opcode,[opaddr+5],1  

    shr opcode,3  

    and opcode,7  

    cmp opcode,2  

    je norb  

    cmp opcode,4  

    je VM_MULB_F  

    cmp opcode,5  

    je VM_IMULB_F  

    cmp opcode,6  

    je VM_DIVB  

    cmp opcode,7  

    je VM_IDIVB  

    jmp error  

norb:  

    mov opcode,[opaddr+0C],2  

    cmp opcode,9C66  

    je VM_NORB_F  

    jmp VM_NORB  

popsp:  

    mov opcode,[opaddr+4],1  
```



```
cmp opcode,0C4  

je VM_POP_SP  

cmp opcode,0CC  

je VM_POP_SP  

cmp opcode,0D4  

je VM_POP_SP  

jmp error  

sr2:  

mov opcode,[opaddr+4],1  

shr opcode,3  

and opcode,7  

cmp opcode,0  

je VM_POP_ES  

cmp opcode,1  

je VM_POP_CS  

cmp opcode,2  

je VM_POP_SS  

cmp opcode,3  

je VM_POP_DS  

cmp opcode,4  

je VM_POP_FS  

cmp opcode,5  

je VM_POP_GS  

jmp error  

gr2:  

mov opcode,[opaddr+1],1  

cmp opcode,58  

je VM_POP_AX  

cmp opcode,59  

je VM_POP_CX  

cmp opcode,5A  

je VM_POP_DX  

jmp error  

lodsw1:  

mov opcode,[opaddr+2],1  

cmp opcode,0C4  

je VM_PUSH_SP  

cmp opcode,0CC  

je VM_PUSH_SP  

cmp opcode,0D4  

je VM_PUSH_SP  

mov $RESULT,opaddr  

mov loopcnt,2  

lodsw@1:  

find $RESULT,#E9#  

cmp $RESULT,0  

je error  

mov temp,loopep  

sub temp,$RESULT  

sub temp,5  

cmp temp,[$RESULT+1]  

jne lodsw@1  

lodsw@2:  

preop $RESULT  

mov opcode,[$RESULT],1  

cmp opcode,50  

je VM_PUSHD_IMM16  

cmp opcode,51  

je VM_PUSHD_IMM16  

cmp opcode,52  

je VM_PUSHD_IMM16  

mov opcode,[$RESULT],2  

cmp opcode,5066  

je VM_PUSH_IMM16  

cmp opcode,5166  

je VM_PUSH_IMM16  

cmp opcode,5266  

je VM_PUSH_IMM16  

sub loopcnt,1  

cmp loopcnt,0  

jne lodsw@2  

jmp error  

sr3:  

mov opcode,[opaddr+2],1  

shr opcode,3  

and opcode,7  

cmp opcode,0  

je VM_PUSH_ES  

cmp opcode,1  

je VM_PUSH_CS  

cmp opcode,2  

je VM_PUSH_SS  

cmp opcode,3  

je VM_PUSH_DS  

cmp opcode,4  

je VM_PUSH_FS  

cmp opcode,5  

je VM_PUSH_GS  

jmp error  

lodsb1:  

mov $RESULT,opaddr  

mov loopcnt,2  

lodsb@1:  

find $RESULT,#E9#  

cmp $RESULT,0  

je error  

mov temp,loopep  

sub temp,$RESULT  

sub temp,5  

cmp temp,[$RESULT+1]  

jne lodsb@1  

lodsb@2:  

preop $RESULT  

mov opcode,[$RESULT],1  

cmp opcode,50  

jb lodsb@3  

cmp opcode,52  

ja lodsb@3  

    preop $RESULT  

    mov opcode,[$RESULT],3  

    cmp opcode,87048B  

    je VM_PUSH_CTX  

    cmp opcode,870C8B  

    je VM_PUSH_CTX  

    cmp opcode,87148B  

    je VM_PUSH_CTX  

    jne VM_PUSHD_IMM8  

lodsb@3:  

cmp opcode,66  

jne lodsb@5  

    mov opcode,[$RESULT+1],1  

    cmp opcode,50  

    jb lodsb@4  

    cmp opcode,52  

    ja lodsb@4  

        preop $RESULT  

        mov opcode,[$RESULT],3  

        cmp opcode,87048A  

        je VM_PUSH_CTXB0  

        cmp opcode,870C8A  

        je VM_PUSH_CTXB0  

        cmp opcode,87148A  

        je VM_PUSH_CTXB0  

        mov opcode,[$RESULT],4  

        cmp opcode,187448A  

        je VM_PUSH_CTXB1  

        cmp opcode,1874C8A  

        je VM_PUSH_CTXB1  

        cmp opcode,187548A  

        je VM_PUSH_CTXB1  

        cmp opcode,87048B66  

        je VM_PUSH_CTXW  

        cmp opcode,870C8B66  

        je VM_PUSH_CTXW  

        cmp opcode,87148B66  

        je VM_PUSH_CTXW  

        jmp VM_PUSH_IMM8  

lodsb@4:  

    mov opcode,[$RESULT+1],3  

    cmp opcode,870489  

    je VM_POP_CTXW  

    cmp opcode,870C89  

    je VM_POP_CTXW  

    cmp opcode,871489  

    je VM_POP_CTXW  

    cmp opcode,87048F  

    je VM_POP_CTXW  

    cmp opcode,8734FF  

    je VM_PUSH_CTXW  

    jmp error  

lodsb@5:  

mov opcode,[$RESULT],3  

cmp opcode,870488  

je VM_POP_CTXB0  

cmp opcode,870C88  

je VM_POP_CTXB0  

cmp opcode,871488  

je VM_POP_CTXB0  

cmp opcode,8734FF  

je VM_PUSH_CTX  

cmp opcode,87048F  

je VM_POP_CTX  

cmp opcode,870489  

je VM_POP_CTX  

cmp opcode,870C89  

je VM_POP_CTX  

cmp opcode,871489  

je VM_POP_CTX  

mov opcode,[$RESULT],4  

cmp opcode,1874488  

je VM_POP_CTXB1  

cmp opcode,1874C88  

je VM_POP_CTXB1  

cmp opcode,1875488  

je VM_POP_CTXB1  

sub loopcnt,1  

cmp loopcnt,0  

jne lodsb@2  

jmp error  

lodsdw1:  

mov opcode,[opaddr+1],1  

cmp opcode,0E0  

je VM_PUSH_ESP  

cmp opcode,0E1  

je VM_PUSH_ESP  

cmp opcode,0E2  

je VM_PUSH_ESP  

mov $RESULT,opaddr  

mov loopcnt,2  

lodsdw@1:  

find $RESULT,#E9#  

cmp $RESULT,0  

je error  

mov temp,loopep  

sub temp,$RESULT  

sub temp,5  

cmp temp,[$RESULT+1]  

jne lodsdw@1  

lodsdw@2:  

preop $RESULT  

mov opcode,[$RESULT],1  

cmp opcode,50  

je VM_PUSH_IMM32  

cmp opcode,51  

je VM_PUSH_IMM32  

cmp opcode,52  

je VM_PUSH_IMM32  

sub loopcnt,1  

cmp loopcnt,0  

jne lodsdw@2  

jmp error  

ret1:  

mov opcode,[opaddr+0A],1  

jmp ret@1  

ret2:  

mov opcode,[opaddr+11],1  

ret@1:  

cmp opcode,0C3  

je VM_RETN  

cmp opcode,0CB  

je VM_RETF  

jmp error  

fpu:  

mov opcode,[opaddr],2  

cmp opcode,0F0D9  

je VM_F2XM1  

cmp opcode,0E1D9  

je VM_FABS  

cmp opcode,04D8  

je VM_FADD  

cmp opcode,04DC  

je VM_FADDQ  

cmp opcode,0E0D9  

je VM_FCHS  

cmp opcode,0E2DB  

je VM_FCLEX  

cmp opcode,1CD8  

je VM_FCOMP  

cmp opcode,1CDC  

je VM_FCOMPQ  

cmp opcode,0FFD9  

je VM_FCOS  

cmp opcode,0F6D9  

je VM_FDECSTP  

cmp opcode,34D8  

je VM_FDIV  

cmp opcode,34DC  

je VM_FDIVQ  

cmp opcode,04DB  

je VM_FILD  

cmp opcode,2CDF  

je VM_FILDQ  

cmp opcode,0F7D9  

je VM_FINCSTP  

cmp opcode,0E3DB  

je VM_FINIT  

cmp opcode,1CDB  

je VM_FISTP  

cmp opcode,3CDF  

je VM_FISTPQ  

cmp opcode,1CDF  

je VM_FISTPW  

cmp opcode,24DA  

je VM_FISUB  

cmp opcode,24DE  

je VM_FISUBW  

cmp opcode,04D9  

je VM_FLD  

cmp opcode,04DD  

je VM_FLDQ  

cmp opcode,2CDB  

je VM_FLDT  

cmp opcode,0E8D9  

je VM_FLD1  

cmp opcode,2CD9  

je VM_FLDCW  

cmp opcode,0ECD9  

je VM_FLDLG2  

cmp opcode,0EDD9  

je VM_FLDLN2  

cmp opcode,0EBD9  

je VM_FLDPI  

cmp opcode,0EED9  

je VM_FLDZ  

cmp opcode,0CD8  

je VM_FMUL  

cmp opcode,0CDC  

je VM_FMULQ  

cmp opcode,0F3D9  

je VM_FPATAN  

cmp opcode,0F8D9  

je VM_FPREM  

cmp opcode,0F5D9  

je VM_FPREM1  

cmp opcode,0F2D9  

je VM_FPTAN  

cmp opcode,0FCD9  

je VM_FRNDINT  

cmp opcode,0FED9  

je VM_FSIN  

cmp opcode,0FAD9  

je VM_FSQRT  

cmp opcode,14D9  

je VM_FST  

cmp opcode,14DD  

je VM_FSTQ  

cmp opcode,3CD9  

je VM_FSTCW  

cmp opcode,1CD9  

je VM_FSTP  

cmp opcode,1CDD  

je VM_FSTPQ  

cmp opcode,3CDB  

je VM_FSTPT  

cmp opcode,0E0DF  

je VM_FSTSW  

cmp opcode,24D8  

je VM_FSUB  

cmp opcode,24DC  

je VM_FSUBQ  

cmp opcode,2CD8  

je VM_FSUBR  

cmp opcode,2CDC  

je VM_FSUBRQ  

cmp opcode,0E4D9  

je VM_FTST  

cmp opcode,0F1D9  

je VM_FYL2X  

jmp error  

norw:  

mov opcode,[opaddr+9],2  

cmp opcode,9C66  

je VM_NORW_F  

jmp VM_NORW  
```



```
// ********************************************************************

VM_ADDB:
    lbl opaddr,"VM_ADDB"
    jmp findnext

VM_ADDB_F:
    lbl opaddr,"VM_ADDB_F"
    jmp findnext

VM_ADDW:
    lbl opaddr,"VM_ADDW"
    jmp findnext

VM_ADDW_F:
    lbl opaddr,"VM_ADDW_F"
    jmp findnext

VM_ADD:
    lbl opaddr,"VM_ADD"
    jmp findnext

VM_ADD_F:
    lbl opaddr,"VM_ADD_F"
    jmp findnext

VM_MULB_F:
    lbl opaddr,"VM_MULB_F"
    jmp findnext

VM_MULW_F:
    lbl opaddr,"VM_MULW_F"
    jmp findnext

VM_MUL_F:
    lbl opaddr,"VM_MUL_F"
    jmp findnext

VM_IMULB_F:
    lbl opaddr,"VM_IMULB_F"
    jmp findnext

VM_IMULW_F:
    lbl opaddr,"VM_IMULW_F"
    jmp findnext

VM_IMUL_F:
    lbl opaddr,"VM_IMUL_F"
    jmp findnext

VM_DIVB:
    lbl opaddr,"VM_DIVB"
    jmp findnext

VM_DIVW:
    lbl opaddr,"VM_DIVW"
    jmp findnext

VM_DIV:
    lbl opaddr,"VM_DIV"
    jmp findnext

VM_IDIVB:
    lbl opaddr,"VM_IDIVB"
    jmp findnext

VM_IDIVW:
    lbl opaddr,"VM_IDIVW"
    jmp findnext

VM_IDIV:
    lbl opaddr,"VM_IDIV"
    jmp findnext

VM_SHLB:
    lbl opaddr,"VM_SHLB"
    jmp findnext

VM_SHLB_F:
    lbl opaddr,"VM_SHLB_F"
    jmp findnext

VM_SHLW:
    lbl opaddr,"VM_SHLW"
    jmp findnext

VM_SHLW_F:
    lbl opaddr,"VM_SHLW_F"
    jmp findnext

VM_SHL:
    lbl opaddr,"VM_SHL"
    jmp findnext

VM_SHL_F:
    lbl opaddr,"VM_SHL_F"
    jmp findnext

VM_SHLD_F:
    lbl opaddr,"VM_SHLD_F"
    jmp findnext

VM_SHRB:
    lbl opaddr,"VM_SHRB"
    jmp findnext

VM_SHRB_F:
    lbl opaddr,"VM_SHRB_F"
    jmp findnext

VM_SHRW:
    lbl opaddr,"VM_SHRW"
    jmp findnext

VM_SHRW_F:
    lbl opaddr,"VM_SHRW_F"
    jmp findnext

VM_SHR:
    lbl opaddr,"VM_SHR"
    jmp findnext

VM_SHR_F:
    lbl opaddr,"VM_SHR_F"
    jmp findnext

VM_SHRD_F:
    lbl opaddr,"VM_SHRD_F"
    jmp findnext

VM_MOVB_S1_CS1:
    lbl opaddr,"VM_MOVB_S1_CS1"
    jmp findnext

VM_MOVB_S1_DS1:
    lbl opaddr,"VM_MOVB_S1_DS1"
    jmp findnext

VM_MOVB_S1_ES1:
    lbl opaddr,"VM_MOVB_S1_ES1"
    jmp findnext

VM_MOVB_S1_FS1:
    lbl opaddr,"VM_MOVB_S1_FS1"
    jmp findnext

VM_MOVB_S1_GS1:
    lbl opaddr,"VM_MOVB_S1_GS1"
    jmp findnext

VM_MOVB_S1_SS1:
    lbl opaddr,"VM_MOVB_S1_SS1"
    jmp findnext

VM_MOVW_S1_CS1:
    lbl opaddr,"VM_MOVW_S1_CS1"
    jmp findnext

VM_MOVW_S1_DS1:
    lbl opaddr,"VM_MOVW_S1_DS1"
    jmp findnext

VM_MOVW_S1_ES1:
    lbl opaddr,"VM_MOVW_S1_ES1"
    jmp findnext

VM_MOVW_S1_FS1:
    lbl opaddr,"VM_MOVW_S1_FS1"
    jmp findnext

VM_MOVW_S1_GS1:
    lbl opaddr,"VM_MOVW_S1_GS1"
    jmp findnext

VM_MOVW_S1_SS1:
    lbl opaddr,"VM_MOVW_S1_SS1"
    jmp findnext

VM_MOV_S1_CS1:
    lbl opaddr,"VM_MOV_S1_CS1"
    jmp findnext

VM_MOV_S1_DS1:
    lbl opaddr,"VM_MOV_S1_DS1"
    jmp findnext

VM_MOV_S1_ES1:
    lbl opaddr,"VM_MOV_S1_ES1"
    jmp findnext

VM_MOV_S1_FS1:
    lbl opaddr,"VM_MOV_S1_FS1"
    jmp findnext

VM_MOV_S1_GS1:
    lbl opaddr,"VM_MOV_S1_GS1"
    jmp findnext

VM_MOV_S1_SS1:
    lbl opaddr,"VM_MOV_S1_SS1"
    jmp findnext

VM_MOVB_CS1_S2:
    lbl opaddr,"VM_MOVB_CS1_S2"
    jmp findnext

VM_MOVB_DS1_S2:
    lbl opaddr,"VM_MOVB_DS1_S2"
    jmp findnext

VM_MOVB_ES1_S2:
    lbl opaddr,"VM_MOVB_ES1_S2"
    jmp findnext

VM_MOVB_FS1_S2:
    lbl opaddr,"VM_MOVB_FS1_S2"
    jmp findnext

VM_MOVB_GS1_S2:
    lbl opaddr,"VM_MOVB_GS1_S2"
    jmp findnext

VM_MOVB_SS1_S2:
    lbl opaddr,"VM_MOVB_SS1_S2"
    jmp findnext

VM_MOVW_CS1_S2:
    lbl opaddr,"VM_MOVW_CS1_S2"
    jmp findnext

VM_MOVW_DS1_S2:
    lbl opaddr,"VM_MOVW_DS1_S2"
    jmp findnext

VM_MOVW_ES1_S2:
    lbl opaddr,"VM_MOVW_ES1_S2"
    jmp findnext

VM_MOVW_FS1_S2:
    lbl opaddr,"VM_MOVW_FS1_S2"
    jmp findnext

VM_MOVW_GS1_S2:
    lbl opaddr,"VM_MOVW_GS1_S2"
    jmp findnext

VM_MOVW_SS1_S2:
    lbl opaddr,"VM_MOVW_SS1_S2"
    jmp findnext

VM_MOV_CS1_S2:
    lbl opaddr,"VM_MOV_CS1_S2"
    jmp findnext

VM_MOV_DS1_S2:
    lbl opaddr,"VM_MOV_DS1_S2"
    jmp findnext

VM_MOV_ES1_S2:
    lbl opaddr,"VM_MOV_ES1_S2"
    jmp findnext

VM_MOV_FS1_S2:
    lbl opaddr,"VM_MOV_FS1_S2"
    jmp findnext

VM_MOV_GS1_S2:
    lbl opaddr,"VM_MOV_GS1_S2"
    jmp findnext

VM_MOV_SS1_S2:
    lbl opaddr,"VM_MOV_SS1_S2"
    jmp findnext

VM_PUSH_SP:
    lbl opaddr,"VM_PUSH_SP"
    jmp findnext

VM_PUSH_ESP:
    lbl opaddr,"VM_PUSH_ESP"
    jmp findnext

VM_POP_SP:
    lbl opaddr,"VM_POP_SP"
    jmp findnext

VM_POP_ESP:
    lbl opaddr,"VM_POP_ESP"
    jmp findnext

VM_POP_AX:
    lbl opaddr,"VM_POP_AX"
    jmp findnext

VM_POP_EAX:
    lbl opaddr,"VM_POP_EAX"
    jmp findnext

VM_POP_CX:
    lbl opaddr,"VM_POP_CX"
    jmp findnext

VM_POP_ECX:
    lbl opaddr,"VM_POP_ECX"
    jmp findnext

VM_POP_DX:
    lbl opaddr,"VM_POP_DX"
    jmp findnext

VM_POP_EDX:
    lbl opaddr,"VM_POP_EDX"
    jmp findnext

VM_PUSH_CR0:
    lbl opaddr,"VM_PUSH_CR0"
    jmp findnext

VM_PUSH_CR1:
    lbl opaddr,"VM_PUSH_CR1"
    jmp findnext

VM_PUSH_CR2:
    lbl opaddr,"VM_PUSH_CR2"
    jmp findnext

VM_PUSH_CR3:
    lbl opaddr,"VM_PUSH_CR3"
    jmp findnext

VM_PUSH_CR4:
    lbl opaddr,"VM_PUSH_CR4"
    jmp findnext

VM_PUSH_CR5:
    lbl opaddr,"VM_PUSH_CR5"
    jmp findnext

VM_PUSH_CR6:
    lbl opaddr,"VM_PUSH_CR6"
    jmp findnext

VM_PUSH_CR7:
    lbl opaddr,"VM_PUSH_CR7"
    jmp findnext

VM_PUSH_DR0:
    lbl opaddr,"VM_PUSH_DR0"
    jmp findnext

VM_PUSH_DR1:
    lbl opaddr,"VM_PUSH_DR1"
    jmp findnext

VM_PUSH_DR2:
    lbl opaddr,"VM_PUSH_DR2"
    jmp findnext

VM_PUSH_DR3:
    lbl opaddr,"VM_PUSH_DR3"
    jmp findnext

VM_PUSH_DR4:
    lbl opaddr,"VM_PUSH_DR4"
    jmp findnext

VM_PUSH_DR5:
    lbl opaddr,"VM_PUSH_DR5"
    jmp findnext

VM_PUSH_DR6:
    lbl opaddr,"VM_PUSH_DR6"
    jmp findnext

VM_PUSH_DR7:
    lbl opaddr,"VM_PUSH_DR7"
    jmp findnext

VM_POP_CR0:
    lbl opaddr,"VM_POP_CR0"
    jmp findnext

VM_POP_CR1:
    lbl opaddr,"VM_POP_CR1"
    jmp findnext

VM_POP_CR2:
    lbl opaddr,"VM_POP_CR2"
    jmp findnext

VM_POP_CR3:
    lbl opaddr,"VM_POP_CR3"
    jmp findnext

VM_POP_CR4:
    lbl opaddr,"VM_POP_CR4"
    jmp findnext

VM_POP_CR5:
    lbl opaddr,"VM_POP_CR5"
    jmp findnext

VM_POP_CR6:
    lbl opaddr,"VM_POP_CR6"
    jmp findnext

VM_POP_CR7:
    lbl opaddr,"VM_POP_CR7"
    jmp findnext

VM_POP_DR0:
    lbl opaddr,"VM_POP_DR0"
    jmp findnext

VM_POP_DR1:
    lbl opaddr,"VM_POP_DR1"
    jmp findnext

VM_POP_DR2:
    lbl opaddr,"VM_POP_DR2"
    jmp findnext

VM_POP_DR3:
    lbl opaddr,"VM_POP_DR3"
    jmp findnext

VM_POP_DR4:
    lbl opaddr,"VM_POP_DR4"
    jmp findnext

VM_POP_DR5:
    lbl opaddr,"VM_POP_DR5"
    jmp findnext
```



```asm
    jmp findnext  

VM_POP_DR6:  

    lbl opaddr,"VM_POP_DR6"  

    jmp findnext  

VM_POP_DR7:  

    lbl opaddr,"VM_POP_DR7"  

    jmp findnext  

VM_PUSH_CS:  

    lbl opaddr,"VM_PUSH_CS"  

    jmp findnext  

VM_PUSH_DS:  

    lbl opaddr,"VM_PUSH_DS"  

    jmp findnext  

VM_PUSH_ES:  

    lbl opaddr,"VM_PUSH_ES"  

    jmp findnext  

VM_PUSH_FS:  

    lbl opaddr,"VM_PUSH_FS"  

    jmp findnext  

VM_PUSH_GS:  

    lbl opaddr,"VM_PUSH_GS"  

    jmp findnext  

VM_PUSH_SS:  

    lbl opaddr,"VM_PUSH_SS"  

    jmp findnext  

VM_POP_DS:  

    lbl opaddr,"VM_POP_DS"  

    jmp findnext  

VM_POP_ES:  

    lbl opaddr,"VM_POP_ES"  

    jmp findnext  

VM_POP_FS:  

    lbl opaddr,"VM_POP_FS"  

    jmp findnext  

VM_POP_GS:  

    lbl opaddr,"VM_POP_GS"  

    jmp findnext  

VM_POP_SS:  

    lbl opaddr,"VM_POP_SS"  

    jmp findnext  

VM_PUSH_CTXB0:  

    lbl opaddr,"VM_PUSH_CTXB0"  

    jmp findnext  

VM_PUSH_CTXB1:  

    lbl opaddr,"VM_PUSH_CTXB1"  

    jmp findnext  

VM_PUSH_CTXW:  

    lbl opaddr,"VM_PUSH_CTXW"  

    jmp findnext  

VM_PUSH_CTX:  

    lbl opaddr,"VM_PUSH_CTX"  

    jmp findnext  

VM_POP_CTXB0:  

    lbl opaddr,"VM_POP_CTXB0"  

    jmp findnext  

VM_POP_CTXB1:  

    lbl opaddr,"VM_POP_CTXB1"  

    jmp findnext  

VM_POP_CTXW:  

    lbl opaddr,"VM_POP_CTXW"  

    jmp findnext  

VM_POP_CTX:  

    lbl opaddr,"VM_POP_CTX"  

    jmp findnext  

VM_PUSH_IMM8:  

    lbl opaddr,"VM_PUSH_IMM8"  

    jmp findnext  

VM_PUSH_IMM16:  

    lbl opaddr,"VM_PUSH_IMM16"  

    jmp findnext  

VM_PUSH_IMM32:  

    lbl opaddr,"VM_PUSH_IMM32"  

    jmp findnext  

VM_PUSHD_IMM8:  

    lbl opaddr,"VM_PUSHD_IMM8"  

    jmp findnext  

VM_PUSHD_IMM16:  

    lbl opaddr,"VM_PUSHD_IMM16"  

    jmp findnext  

VM_NORB:  

    lbl opaddr,"VM_NORB"  

    jmp findnext  

VM_NORB_F:  

    lbl opaddr,"VM_NORB_F"  

    jmp findnext  

VM_NORW:  

    lbl opaddr,"VM_NORW"  

    jmp findnext  

VM_NORW_F:  

    lbl opaddr,"VM_NORW_F"  

    jmp findnext  

VM_NOR:  

    lbl opaddr,"VM_NOR"  

    jmp findnext  

VM_NOR_F:  

    lbl opaddr,"VM_NOR_F"  

    jmp findnext  

VM_WAIT:  

    lbl opaddr,"VM_WAIT"  

    jmp findnext  

VM_JMP:  

    lbl opaddr,"VM_JMP"  

    jmp findnext  

VM_RETN:  

    lbl opaddr,"VM_RETN"  

    jmp findnext  

VM_RETF:  

    lbl opaddr,"VM_RETF"  

    jmp findnext  

VM_F2XM1:  

    lbl opaddr,"VM_F2XM1"  

    jmp findnext  

VM_FABS:  

    lbl opaddr,"VM_FABS"  

    jmp findnext  

VM_FADD:  

    lbl opaddr,"VM_FADD"  

    jmp findnext  

VM_FADDQ:  

    lbl opaddr,"VM_FADDQ"  

    jmp findnext  

VM_FCHS:  

    lbl opaddr,"VM_FCHS"  

    jmp findnext  

VM_FCLEX:  

    lbl opaddr,"VM_FCLEX"  

    jmp findnext  

VM_FCOMP:  

    lbl opaddr,"VM_FCOMP"  

    jmp findnext  

VM_FCOMPQ:  

    lbl opaddr,"VM_FCOMPQ"  

    jmp findnext  

VM_FCOS:  

    lbl opaddr,"VM_FCOS"  

    jmp findnext  

VM_FDECSTP:  

    lbl opaddr,"VM_FDECSTP"  

    jmp findnext  

VM_FDIV:  

    lbl opaddr,"VM_FDIV"  

    jmp findnext  

VM_FDIVQ:  

    lbl opaddr,"VM_FDIVQ"  

    jmp findnext  

VM_FILD:  

    lbl opaddr,"VM_FILD"  

    jmp findnext  

VM_FILDQ:  

    lbl opaddr,"VM_FILDQ"  

    jmp findnext  

VM_FINCSTP:  

    lbl opaddr,"VM_FINCSTP"  

    jmp findnext  

VM_FINIT:  

    lbl opaddr,"VM_FINIT"  

    jmp findnext  

VM_FISTP:  

    lbl opaddr,"VM_FISTP"  

    jmp findnext  

VM_FISTPQ:  

    lbl opaddr,"VM_FISTPQ"  

    jmp findnext  

VM_FISTPW:  

    lbl opaddr,"VM_FISTPW"  

    jmp findnext  

VM_FISUB:  

    lbl opaddr,"VM_FISUB"  

    jmp findnext  

VM_FISUBW:  

    lbl opaddr,"VM_FISUBW"  

    jmp findnext  

VM_FLD:  

    lbl opaddr,"VM_FLD"  

    jmp findnext  

VM_FLDQ:  

    lbl opaddr,"VM_FLDQ"  

    jmp findnext  

VM_FLD T:  

    lbl opaddr,"VM_FLD T"  

    jmp findnext  

VM_FLD1:  

    lbl opaddr,"VM_FLD1"  

    jmp findnext  

VM_FLD CW:  

    lbl opaddr,"VM_FLD CW"  

    jmp findnext  

VM_FLD LG2:  

    lbl opaddr,"VM_FLD LG2"  

    jmp findnext  

VM_FLD LN2:  

    lbl opaddr,"VM_FLD LN2"  

    jmp findnext  

VM_FLD PI:  

    lbl opaddr,"VM_FLD PI"  

    jmp findnext  

VM_FLD Z:  

    lbl opaddr,"VM_FLD Z"  

    jmp findnext  

VM_F M U L:  

    lbl opaddr,"VM_F M U L"  

    jmp findnext  

VM_F M U L Q:  

    lbl opaddr,"VM_F M U L Q"  

    jmp findnext  

VM_F P A T A N:  

    lbl opaddr,"VM_F P A T A N"  

    jmp findnext  

VM_F P R E M:  

    lbl opaddr,"VM_F P R E M"  

    jmp findnext  

VM_F P R E M 1:  

    lbl opaddr,"VM_F P R E M 1"  

    jmp findnext  

VM_F P T A N:  

    lbl opaddr,"VM_F P T A N"  

    jmp findnext  

VM_F R N D I N T:  

    lbl opaddr,"VM_F R N D I N T"  

    jmp findnext  

VM_F S I N:  

    lbl opaddr,"VM_F S I N"  

    jmp findnext  

VM_F S Q R T:  

    lbl opaddr,"VM_F S Q R T"  

    jmp findnext  

VM_F S T:  

    lbl opaddr,"VM_F S T"  

    jmp findnext  

VM_F S T Q:  

    lbl opaddr,"VM_F S T Q"  

    jmp findnext  

VM_F S T C W:  

    lbl opaddr,"VM_F S T C W"  

    jmp findnext  

VM_F S T P:  

    lbl opaddr,"VM_F S T P"  

    jmp findnext  

VM_F S T P Q:  

    lbl opaddr,"VM_F S T P Q"  

    jmp findnext  

VM_F S T P T:  

    lbl opaddr,"VM_F S T P T"  

    jmp findnext  

VM_F S T S W:  

    lbl opaddr,"VM_F S T S W"  

    jmp findnext  

VM_F S U B:  

    lbl opaddr,"VM_F S U B"  

    jmp findnext  

VM_F S U B Q:  

    lbl opaddr,"VM_F S U B Q"  

    jmp findnext  

VM_F S U B R:  

    lbl opaddr,"VM_F S U B R"  

    jmp findnext  

VM_F S U B R Q:  

    lbl opaddr,"VM_F S U B R Q"  

    jmp findnext  

VM_F T S T:  

    lbl opaddr,"VM_F T S T"  

    jmp findnext  

VM_F Y L 2 X:  

    lbl opaddr,"VM_F Y L 2 X"  

    jmp findnext  

// ******************************************************************************************************************************************  

findnext:  

    add optbl,4  

    inc opcnt  

    cmp opcnt,100  

    jne findop  

    itoa findcnt,10.  

    eval "Find {$RESULT} VM Opcode."  

    log $RESULT,""  

    msg $RESULT  

    jmp end  

error:  

    msg "出错了!@#$%^&*"  

    pause  

end:  

    ret
```

### 附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `vmp120.rar` \[解压密码：PEDIY\]  

 |
