

# VMP1.20 的 Opcode 识别脚本

> 原文：[`www.pediy.com/kssd/pediy10/58462.html`](https://www.pediy.com/kssd/pediy10/58462.html)

主题:VMP1.20 的 Opcode 识别脚本  
   
作者:DarkBull  
   
邮箱:DarkBull#126.com  
   
前言:前一阵学习了 wangdell 的＜VMProtect1.2x 总结＞,受益非浅.于是照着写了个 OD 脚本,大部分 Opcode 名称与 wangdell 的命名相同,个别不同的纯属个人习惯,仅供参考.由于水平有限,接触的 VMP 程序程序不多,难免有错误的地方,请高手多多指教.  
   
// VMProtect1.2x Find OPcode Script  
// Writen By DarkBull  
data:  
    var codebase  
    var codesize  
    var init  
    var loopep  
    var optbl  
    var opaddr  
    var opcode  
    var opcnt  
    var findcnt  
    var loopcnt  
    var temp  
code:  
    gmi eip,CODEBASE  
    mov codebase,$RESULT  
    gmi eip,CODESIZE  
    mov codesize,$RESULT  
findinit:  
    find codebase,#??????????????????68000000008B742428#  
    cmp $RESULT,0  
    jne vminit  
input1:  
    mov $RESULT,0  
    ask "Plese Input VMP Base Address:"  
    cmp $RESULT,0  
    je input1  
    find $RESULT,#??????????????????68000000008B742428#  
    cmp $RESULT,0  
    je error  
vminit:  
    mov temp,$RESULT  
    lbl $RESULT,"VM\_Init"  
    mov init,$RESULT  
findep:  
    find temp,#033424#  
    cmp $RESULT,0  
    je error  
    add $RESULT,3  
    mov temp,$RESULT  
    lbl $RESULT,"VM\_LoopEP"  
    mov loopep,$RESULT  
findoptb:  
    find temp,#FF??85#  
    cmp $RESULT,0  
    je error  
    add $RESULT,3  
    mov optbl,\[$RESULT\]  
    lbl optbl,"VM\_OP\_Table"  
// \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
findop:  
    mov opaddr,\[optbl\]  
    gn opaddr  
    cmp $RESULT,0  
    jne findnext  
    inc findcnt  
    mov opcode,\[opaddr\],1  
    cmp opcode,0F  
    je op2b  
    cmp opcode,54  
    je VM\_PUSH\_ESP  
    cmp opcode,58  
    jb find@1  
    cmp opcode,5A  
    ja find@1  
    jmp popreg  
find@1:  
    cmp opcode,5B  
    je ret1  
    cmp opcode,5C  
    je VM\_POP\_ESP  
    cmp opcode,5D  
    jb find@2  
    cmp opcode,5F  
    ja find@2  
    jmp ret1  
find@2:  
    cmp opcode,66  
    je prefix  
    cmp opcode,88  
    je lodsb1  
    cmp opcode,89  
    je lodsdw1  
    cmp opcode,8A  
    je lodsb1  
    cmp opcode,8B  
    je lodsdw1  
    cmp opcode,09B  
    je VM\_WAIT  
    cmp opcode,09D  
    je ret1  
    cmp opcode,0AC  
    je lodsb1  
    cmp opcode,0AD  
    je lodsdw1  
    cmp opcode,0C7  
    je ret2  
    cmp opcode,0D8  
    jb find@3  
    cmp opcode,0DF  
    ja find@3  
    jmp fpu  
find@3:  
    cmp opcode,0F7  
    je norw  
    jmp error  
op2b:  
    mov opcode,\[opaddr+1\],1  
    cmp opcode,20  
    je spr1  
    cmp opcode,21  
    je spr2  
    cmp opcode,0B6  
    je lodsb1  
    cmp opcode,0B7  
    je lodsw1  
    jmp error  
spr1:  
    mov opcode,\[opaddr+2\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_PUSH\_CR0  
    cmp opcode,1  
    je VM\_PUSH\_CR1  
    cmp opcode,2  
    je VM\_PUSH\_CR2  
    cmp opcode,3  
    je VM\_PUSH\_CR3  
    cmp opcode,4  
    je VM\_PUSH\_CR4  
    cmp opcode,5  
    je VM\_PUSH\_CR5  
    cmp opcode,6  
    je VM\_PUSH\_CR6  
    cmp opcode,7  
    je VM\_PUSH\_CR7  
    jmp error  
spr2:  
    mov opcode,\[opaddr+2\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_PUSH\_DR0  
    cmp opcode,1  
    je VM\_PUSH\_DR1  
    cmp opcode,2  
    je VM\_PUSH\_DR2  
    cmp opcode,3  
    je VM\_PUSH\_DR3  
    cmp opcode,4  
    je VM\_PUSH\_DR4  
    cmp opcode,5  
    je VM\_PUSH\_DR5  
    cmp opcode,6  
    je VM\_PUSH\_DR6  
    cmp opcode,7  
    je VM\_PUSH\_DR7  
    jmp error  
popreg:  
    mov opcode,\[opaddr+1\],1  
    cmp opcode,01  
    je adddw  
    cmp opcode,0F  
    je pop2b  
    cmp opcode,26  
    je mov1  
    cmp opcode,28  
    je popsub  
    cmp opcode,2E  
    je mov1  
    cmp opcode,36  
    je mov1  
    cmp opcode,3E  
    je mov1  
    cmp opcode,58  
    jb pop@1  
    cmp opcode,5A  
    ja pop@1  
    jmp pop2  
pop@1:  
    cmp opcode,5B  
    jb pop@2  
    cmp opcode,5F  
    ja pop@2  
    jmp ret1  
pop@2:  
    cmp opcode,64  
    je mov1  
    cmp opcode,65  
    je mov1  
    cmp opcode,66  
    je poppref  
    cmp opcode,88  
    je popmovb  
    cmp opcode,89  
    je popmovdw  
    cmp opcode,8A  
    je popmovb  
    cmp opcode,8B  
    je popmovdw  
    cmp opcode,8F  
    je VM\_MOV\_DS1\_S2  
    cmp opcode,9D  
    je ret1  
    cmp opcode,0E9  
    je gr1  
    cmp opcode,0F7  
    je nordw  
    cmp opcode,0FF  
    je VM\_MOV\_S1\_DS1  
    jmp error  
adddw:  
    mov opcode,\[opaddr+2\],2  
    cmp opcode,2404  
    je add@1  
    cmp opcode,240C  
    je add@1  
    cmp opcode,2414  
    je add@1  
    jmp error  
add@1:  
    mov opcode,\[opaddr+4\],2  
    cmp opcode,9C66  
    je VM\_ADD\_F  
    jmp VM\_ADD  
pop2b:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,22  
    je spr3  
    cmp opcode,23  
    je spr4  
    jmp error  
spr3:  
    mov opcode,\[opaddr+3\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_POP\_CR0  
    cmp opcode,1  
    je VM\_POP\_CR1  
    cmp opcode,2  
    je VM\_POP\_CR2  
    cmp opcode,3  
    je VM\_POP\_CR3  
    cmp opcode,4  
    je VM\_POP\_CR4  
    cmp opcode,5  
    je VM\_POP\_CR5  
    cmp opcode,6  
    je VM\_POP\_CR6  
    cmp opcode,7  
    je VM\_POP\_CR7  
    jmp error  
spr4:  
    mov opcode,\[opaddr+3\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_POP\_DR0  
    cmp opcode,1  
    je VM\_POP\_DR1  
    cmp opcode,2  
    je VM\_POP\_DR2  
    cmp opcode,3  
    je VM\_POP\_DR3  
    cmp opcode,4  
    je VM\_POP\_DR4  
    cmp opcode,5  
    je VM\_POP\_DR5  
    cmp opcode,6  
    je VM\_POP\_DR6  
    cmp opcode,7  
    je VM\_POP\_DR7  
    jmp error  
mov1:  
    mov opcode,\[opaddr+1\],2  
    cmp opcode,08A26  
    je VM\_MOVB\_S1\_ES1  
    cmp opcode,08A2E  
    je VM\_MOVB\_S1\_CS1  
    cmp opcode,08A36  
    je VM\_MOVB\_S1\_SS1  
    cmp opcode,08A3E  
    je VM\_MOVB\_S1\_DS1  
    cmp opcode,08A64  
    je VM\_MOVB\_S1\_FS1  
    cmp opcode,08A65  
    je VM\_MOVB\_S1\_GS1  
    cmp opcode,8F26  
    je VM\_MOV\_ES1\_S2  
    cmp opcode,8F2E  
    je VM\_MOV\_CS1\_S2  
    cmp opcode,8F36  
    je VM\_MOV\_SS1\_S2  
    cmp opcode,8F3E  
    je VM\_MOV\_DS1\_S2  
    cmp opcode,8F64  
    je VM\_MOV\_FS1\_S2  
    cmp opcode,8F65  
    je VM\_MOV\_GS1\_S2  
    cmp opcode,0FF26  
    je VM\_MOV\_S1\_ES1  
    cmp opcode,0FF2E  
    je VM\_MOV\_S1\_CS1  
    cmp opcode,0FF36  
    je VM\_MOV\_S1\_SS1  
    cmp opcode,0FF3E  
    je VM\_MOV\_S1\_DS1  
    cmp opcode,0FF64  
    je VM\_MOV\_S1\_FS1  
    cmp opcode,0FF65  
    je VM\_MOV\_S1\_GS1  
    jmp error  
popsub:  
   mov opcode,\[opaddr+3\],2  
    cmp opcode,08A26  
    je VM\_MOVB\_S1\_ES1  
    cmp opcode,08A2E  
    je VM\_MOVB\_S1\_CS1  
    cmp opcode,08A36  
    je VM\_MOVB\_S1\_SS1  
    cmp opcode,08A3E  
    je VM\_MOVB\_S1\_DS1  
    cmp opcode,08A64  
    je VM\_MOVB\_S1\_FS1  
    cmp opcode,08A65  
    je VM\_MOVB\_S1\_GS1  
    jmp error  
pop2:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,58  
    jb pop2@1  
    cmp opcode,5A  
    ja pop2@1  
    jmp pop3  
pop2@1:  
    cmp opcode,5B  
    jb pop2@2  
    cmp opcode,5F  
    ja pop2@2  
    jmp ret1  
pop2@2:  
    cmp opcode,66  
    je pop2pr  
    cmp opcode,9D  
    je ret1  
    cmp opcode,0F7  
    je muldw  
    jmp error  
pop3:  
    mov opcode,\[opaddr+3\],1  
    cmp opcode,5B  
    jb pop3@1  
    cmp opcode,5F  
    ja pop3@1  
    jmp ret1  
pop3@1:  
    cmp opcode,9D  
    je ret1  
    cmp opcode,0F7  
    je divdw  
    jmp error  
divdw:  
    mov opcode,\[opaddr+4\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,6  
    je VM\_DIV  
    cmp opcode,7  
    je VM\_IDIV  
    jmp error  
pop2pr:  
    mov opcode,\[opaddr+3\],1  
    cmp opcode,58  
    jb error  
    cmp opcode,5A  
    ja error  
        mov opcode,\[opaddr+4\],2  
        cmp opcode,0A50F  
        je VM\_SHLD\_F  
        cmp opcode,0AD0F  
        je VM\_SHRD\_F  
        jmp error  
muldw:  
    mov opcode,\[opaddr+3\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,4  
    je VM\_MUL\_F  
    cmp opcode,5  
    je VM\_IMUL\_F  
    jmp error  
poppref:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,26  
    je mov2  
    cmp opcode,2E  
    je mov2  
    cmp opcode,36  
    je mov2  
    cmp opcode,3E  
    je mov2  
    cmp opcode,58  
    jb popr@1  
    cmp opcode,5A  
    ja popr@1  
    jmp poprpo  
popr@1:  
    cmp opcode,64  
    je mov2  
    cmp opcode,65  
    je mov2  
    cmp opcode,8F  
    je VM\_MOVW\_DS1\_S2  
    cmp opcode,0FF  
    je VM\_MOVW\_S1\_DS1  
    jmp error  
mov2:  
    mov opcode,\[opaddr+2\],2  
    cmp opcode,8F26  
    je VM\_MOVW\_ES1\_S2  
    cmp opcode,8F2E  
    je VM\_MOVW\_CS1\_S2  
    cmp opcode,8F36  
    je VM\_MOVW\_SS1\_S2  
    cmp opcode,8F3E  
    je VM\_MOVW\_DS1\_S2  
    cmp opcode,8F64  
    je VM\_MOVW\_FS1\_S2  
    cmp opcode,8F65  
    je VM\_MOVW\_GS1\_S2  
    cmp opcode,0FF26  
    je VM\_MOVW\_S1\_ES1  
    cmp opcode,0FF2E  
    je VM\_MOVW\_S1\_CS1  
    cmp opcode,0FF36  
    je VM\_MOVW\_S1\_SS1  
    cmp opcode,0FF3E  
    je VM\_MOVW\_S1\_DS1  
    cmp opcode,0FF64  
    je VM\_MOVW\_S1\_FS1  
    cmp opcode,0FF65  
    je VM\_MOVW\_S1\_GS1  
    mov opcode,\[opaddr+2\],3  
    cmp opcode,0B60F26  
    je VM\_MOVB\_S1\_ES1  
    cmp opcode,0B60F2E  
    je VM\_MOVB\_S1\_CS1  
    cmp opcode,0B60F36  
    je VM\_MOVB\_S1\_SS1  
    cmp opcode,0B60F3E  
    je VM\_MOVB\_S1\_DS1  
    cmp opcode,0B60F64  
    je VM\_MOVB\_S1\_FS1  
    cmp opcode,0B60F65  
    je VM\_MOVB\_S1\_GS1  
    jmp error  
poprpo:  
    mov opcode,\[opaddr+3\],1  
    cmp opcode,26  
    je mov3  
    cmp opcode,2E  
    je mov3  
    cmp opcode,36  
    je mov3  
    cmp opcode,3E  
    je mov3  
    cmp opcode,64  
    je mov3  
    cmp opcode,65  
    je mov3  
    cmp opcode,88  
    je VM\_MOVB\_DS1\_S2  
    cmp opcode,0D3  
    je shdw  
    jmp error  
mov3:  
    mov opcode,\[opaddr+3\],2  
    cmp opcode,8826  
    je VM\_MOVB\_ES1\_S2  
    cmp opcode,882E  
    je VM\_MOVB\_CS1\_S2  
    cmp opcode,8836  
    je VM\_MOVB\_SS1\_S2  
    cmp opcode,883E  
    je VM\_MOVB\_DS1\_S2  
    cmp opcode,8864  
    je VM\_MOVB\_FS1\_S2  
    cmp opcode,8865  
    je VM\_MOVB\_GS1\_S2  
    jmp error  
shdw:  
    mov opcode,\[opaddr+4\],1  
    cmp opcode,0E0  
    je shldw  
    cmp opcode,0E2  
    je shldw  
    cmp opcode,0E8  
    je shrdw  
    cmp opcode,0EA  
    je shrdw  
    jmp error  
shldw:  
    mov opcode,\[opaddr+6\],2  
    cmp opcode,9C66  
    je VM\_SHL\_F  
    jmp VM\_SHL  
shrdw:  
    mov opcode,\[opaddr+6\],2  
    cmp opcode,9C66  
    je VM\_SHR\_F  
    jmp VM\_SHR  
popmovb:  
    mov opcode,\[opaddr+3\],2  
    cmp opcode,5066  
    je VM\_MOVB\_S1\_DS1  
    jmp error  
popmovdw:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,0C4  
    je VM\_POP\_ESP  
    cmp opcode,0CC  
    je VM\_POP\_ESP  
    cmp opcode,0D4  
    je VM\_POP\_ESP  
    cmp opcode,0C6  
    je VM\_JMP  
    cmp opcode,0CE  
    je VM\_JMP  
    cmp opcode,0D6  
    je VM\_JMP  
    jmp error  
gr1:  
    mov opcode,\[opaddr\],1  
    cmp opcode,58  
    je VM\_POP\_EAX  
    cmp opcode,59  
    je VM\_POP\_ECX  
    cmp opcode,5A  
    je VM\_POP\_EDX  
    jmp error  
nordw:  
    mov opcode,\[opaddr+9\],2  
    cmp opcode,9C66  
    je VM\_NOR\_F  
    jmp VM\_NOR  
prefix:  
    mov opcode,\[opaddr+1\],1  
    cmp opcode,6  
    je VM\_PUSH\_ES  
    cmp opcode,7  
    je VM\_POP\_ES  
    cmp opcode,0E  
    je VM\_PUSH\_CS  
    cmp opcode,0F  
    je sr1  
    cmp opcode,16  
    je VM\_PUSH\_SS  
    cmp opcode,17  
    je VM\_POP\_SS  
    cmp opcode,1E  
    je VM\_PUSH\_DS  
    cmp opcode,1F  
    je VM\_POP\_DS  
    cmp opcode,54  
    je VM\_PUSH\_SP  
    cmp opcode,58  
    jb pref@1  
    cmp opcode,5A  
    ja pref@1  
    jmp prefpop  
pref@1:  
    cmp opcode,5C  
    je VM\_POP\_SP  
    cmp opcode,89  
    je lodsw1  
    cmp opcode,8B  
    je lodsw1  
    cmp opcode,8C  
    je sr3  
    cmp opcode,0AD  
    je lodsw1  
    jmp error  
sr1:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,0A0  
    je VM\_PUSH\_FS  
    cmp opcode,0A1  
    je VM\_POP\_FS  
    cmp opcode,0A8  
    je VM\_PUSH\_GS  
    cmp opcode,0A9  
    je VM\_POP\_GS  
    jmp error  
prefpop:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,0  
    je addb  
    cmp opcode,66  
    je prpopr  
    cmp opcode,0E9  
    je gr2  
    jmp error  
addb:  
    mov opcode,\[opaddr+5\],2  
    cmp opcode,9C66  
    je VM\_ADDB\_F  
    jmp VM\_ADDB  
prpopr:  
    mov opcode,\[opaddr+3\],1  
    cmp opcode,1  
    je addw  
    cmp opcode,58  
    jb ppop@1  
    cmp opcode,5A  
    ja ppop@1  
    jmp prpo2  
ppop@1:  
    cmp opcode,89  
    je popsp  
    cmp opcode,8E  
    je sr2  
    jmp error  
addw:  
    mov opcode,\[opaddr+6\],2  
    cmp opcode,9C66  
    je VM\_ADDW\_F  
    jmp VM\_ADDW  
prpo2:  
    mov opcode,\[opaddr+4\],1  
    cmp opcode,66  
    je prpo2pr  
    cmp opcode,0D2  
    je shb  
    cmp opcode,0F6  
    je prpo2ex  
prpo2pr:  
    mov opcode,\[opaddr+5\],1  
    cmp opcode,58  
    jb pp2p@1  
    cmp opcode,5A  
    ja pp2p@1  
    jmp prpo3  
pp2p@1:  
    cmp opcode,0D3  
    je shw  
    cmp opcode,0F7  
    je mulw  
    jmp error  
prpo3:  
    mov opcode,\[opaddr+6\],2  
    cmp opcode,0F766  
    je divw  
    jmp error  
divw:  
    mov opcode,\[opaddr+8\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,6  
    je VM\_DIVW  
    cmp opcode,7  
    je VM\_IDIVW  
    jmp error  
shw:  
    mov opcode,\[opaddr+6\],1  
    cmp opcode,0E0  
    je shlwcode  
    cmp opcode,0E2  
    je shlwcode  
    cmp opcode,0E8  
    je shrwcode  
    cmp opcode,0EA  
    je shrwcode  
shlwcode:  
    mov opcode,\[opaddr+9\],2  
    cmp opcode,9C66  
    je VM\_SHLW\_F  
    jmp VM\_SHLW  
shrwcode:  
    mov opcode,\[opaddr+9\],2  
    cmp opcode,9C66  
    je VM\_SHRW\_F  
    jmp VM\_SHRW  
mulw:  
    mov opcode,\[opaddr+6\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,4  
    je VM\_MULW\_F  
    cmp opcode,5  
    je VM\_IMULW\_F  
    jmp error  
shb:  
    mov opcode,\[opaddr+5\],1  
    cmp opcode,0E0  
    je shlb  
    cmp opcode,0E2  
    je shlb  
    cmp opcode,0E8  
    je shrb  
    cmp opcode,0EA  
    je shrb  
    jmp error  
shlb:  
    mov opcode,\[opaddr+8\],2  
    cmp opcode,9C66  
    je VM\_SHLB\_F  
    jmp VM\_SHLB  
shrb:  
    mov opcode,\[opaddr+8\],2  
    cmp opcode,9C66  
    je VM\_SHRB\_F  
    jmp VM\_SHRB  
prpo2ex:  
    mov opcode,\[opaddr+5\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,2  
    je norb  
    cmp opcode,4  
    je VM\_MULB\_F  
    cmp opcode,5  
    je VM\_IMULB\_F  
    cmp opcode,6  
    je VM\_DIVB  
    cmp opcode,7  
    je VM\_IDIVB  
    jmp error  
norb:  
    mov opcode,\[opaddr+0C\],2  
    cmp opcode,9C66  
    je VM\_NORB\_F  
    jmp VM\_NORB  
popsp:  
    mov opcode,\[opaddr+4\],1  
    cmp opcode,0C4  
    je VM\_POP\_SP  
    cmp opcode,0CC  
    je VM\_POP\_SP  
     cmp opcode,0D4  
    je VM\_POP\_SP  
    jmp error  
sr2:  
    mov opcode,\[opaddr+4\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_POP\_ES  
    cmp opcode,1  
    je VM\_POP\_CS  
    cmp opcode,2  
    je VM\_POP\_SS  
    cmp opcode,3  
    je VM\_POP\_DS  
    cmp opcode,4  
    je VM\_POP\_FS  
    cmp opcode,5  
    je VM\_POP\_GS  
    jmp error  
gr2:  
    mov opcode,\[opaddr+1\],1  
    cmp opcode,58  
    je VM\_POP\_AX  
    cmp opcode,59  
    je VM\_POP\_CX  
    cmp opcode,5A  
    je VM\_POP\_DX  
    jmp error  
lodsw1:  
    mov opcode,\[opaddr+2\],1  
    cmp opcode,0C4  
    je VM\_PUSH\_SP  
    cmp opcode,0CC  
    je VM\_PUSH\_SP  
    cmp opcode,0D4  
    je VM\_PUSH\_SP  
    mov $RESULT,opaddr  
    mov loopcnt,2  
lodsw@1:  
    find $RESULT,#E9#  
    cmp $RESULT,0  
    je error  
    mov temp,loopep  
    sub temp,$RESULT  
    sub temp,5  
    cmp temp,\[$RESULT+1\]  
    jne lodsw@1  
lodsw@2:  
    preop $RESULT  
    mov opcode,\[$RESULT\],1  
    cmp opcode,50  
    je VM\_PUSHD\_IMM16  
    cmp opcode,51  
    je VM\_PUSHD\_IMM16  
    cmp opcode,52  
    je VM\_PUSHD\_IMM16  
    mov opcode,\[$RESULT\],2  
    cmp opcode,5066  
    je VM\_PUSH\_IMM16  
    cmp opcode,5166  
    je VM\_PUSH\_IMM16  
    cmp opcode,5266  
    je VM\_PUSH\_IMM16  
    sub loopcnt,1  
    cmp loopcnt,0  
    jne lodsw@2  
    jmp error  
sr3:  
    mov opcode,\[opaddr+2\],1  
    shr opcode,3  
    and opcode,7  
    cmp opcode,0  
    je VM\_PUSH\_ES  
    cmp opcode,1  
    je VM\_PUSH\_CS  
    cmp opcode,2  
    je VM\_PUSH\_SS  
    cmp opcode,3  
    je VM\_PUSH\_DS  
    cmp opcode,4  
    je VM\_PUSH\_FS  
    cmp opcode,5  
    je VM\_PUSH\_GS  
    jmp error  
lodsb1:  
    mov $RESULT,opaddr  
    mov loopcnt,2  
lodsb@1:  
    find $RESULT,#E9#  
    cmp $RESULT,0  
    je error  
    mov temp,loopep  
    sub temp,$RESULT  
    sub temp,5  
    cmp temp,\[$RESULT+1\]  
    jne lodsb@1  
lodsb@2:  
    preop $RESULT  
    mov opcode,\[$RESULT\],1  
    cmp opcode,50  
    jb lodsb@3  
    cmp opcode,52  
    ja lodsb@3  
        preop $RESULT  
        mov opcode,\[$RESULT\],3  
        cmp opcode,87048B  
        je VM\_PUSH\_CTX  
        cmp opcode,870C8B  
        je VM\_PUSH\_CTX  
        cmp opcode,87148B  
        je VM\_PUSH\_CTX  
        jne VM\_PUSHD\_IMM8  
lodsb@3:  
    cmp opcode,66  
    jne lodsb@5  
        mov opcode,\[$RESULT+1\],1  
        cmp opcode,50  
        jb lodsb@4  
        cmp opcode,52  
        ja lodsb@4  
            preop $RESULT  
            mov opcode,\[$RESULT\],3  
            cmp opcode,87048A  
            je VM\_PUSH\_CTXB0  
            cmp opcode,870C8A  
            je VM\_PUSH\_CTXB0  
            cmp opcode,87148A  
            je VM\_PUSH\_CTXB0  
            mov opcode,\[$RESULT\],4  
            cmp opcode,187448A  
            je VM\_PUSH\_CTXB1  
            cmp opcode,1874C8A  
            je VM\_PUSH\_CTXB1  
            cmp opcode,187548A  
            je VM\_PUSH\_CTXB1  
            cmp opcode,87048B66  
            je VM\_PUSH\_CTXW  
            cmp opcode,870C8B66  
            je VM\_PUSH\_CTXW  
            cmp opcode,87148B66  
            je VM\_PUSH\_CTXW  
            jmp VM\_PUSH\_IMM8  
lodsb@4:  
        mov opcode,\[$RESULT+1\],3  
        cmp opcode,870489  
        je VM\_POP\_CTXW  
        cmp opcode,870C89  
        je VM\_POP\_CTXW  
        cmp opcode,871489  
        je VM\_POP\_CTXW  
        cmp opcode,87048F  
        je VM\_POP\_CTXW  
        cmp opcode,8734FF  
        je VM\_PUSH\_CTXW  
        jmp error  
lodsb@5:  
    mov opcode,\[$RESULT\],3  
    cmp opcode,870488  
    je VM\_POP\_CTXB0  
    cmp opcode,870C88  
    je VM\_POP\_CTXB0  
    cmp opcode,871488  
    je VM\_POP\_CTXB0  
    cmp opcode,8734FF  
    je VM\_PUSH\_CTX  
    cmp opcode,87048F  
    je VM\_POP\_CTX  
    cmp opcode,870489  
    je VM\_POP\_CTX  
    cmp opcode,870C89  
    je VM\_POP\_CTX  
    cmp opcode,871489  
    je VM\_POP\_CTX  
    mov opcode,\[$RESULT\],4  
    cmp opcode,1874488  
    je VM\_POP\_CTXB1  
    cmp opcode,1874C88  
    je VM\_POP\_CTXB1  
    cmp opcode,1875488  
    je VM\_POP\_CTXB1  
    sub loopcnt,1  
    cmp loopcnt,0  
    jne lodsb@2  
    jmp error  
lodsdw1:  
    mov opcode,\[opaddr+1\],1  
    cmp opcode,0E0  
    je VM\_PUSH\_ESP  
    cmp opcode,0E1  
    je VM\_PUSH\_ESP  
    cmp opcode,0E2  
    je VM\_PUSH\_ESP  
    mov $RESULT,opaddr  
    mov loopcnt,2  
lodsdw@1:  
    find $RESULT,#E9#  
    cmp $RESULT,0  
    je error  
    mov temp,loopep  
    sub temp,$RESULT  
    sub temp,5  
    cmp temp,\[$RESULT+1\]  
    jne lodsdw@1  
lodsdw@2:  
    preop $RESULT  
    mov opcode,\[$RESULT\],1  
    cmp opcode,50  
    je VM\_PUSH\_IMM32  
    cmp opcode,51  
    je VM\_PUSH\_IMM32  
    cmp opcode,52  
    je VM\_PUSH\_IMM32  
    sub loopcnt,1  
    cmp loopcnt,0  
    jne lodsdw@2  
    jmp error  
ret1:  
    mov opcode,\[opaddr+0A\],1  
    jmp ret@1  
ret2:  
    mov opcode,\[opaddr+11\],1  
ret@1:  
    cmp opcode,0C3  
    je VM\_RETN  
    cmp opcode,0CB  
    je VM\_RETF  
    jmp error  
fpu:  
    mov opcode,\[opaddr\],2  
    cmp opcode,0F0D9  
    je VM\_F2XM1  
    cmp opcode,0E1D9  
    je VM\_FABS  
    cmp opcode,04D8  
    je VM\_FADD  
    cmp opcode,04DC  
    je VM\_FADDQ  
    cmp opcode,0E0D9  
    je VM\_FCHS  
    cmp opcode,0E2DB  
    je VM\_FCLEX  
    cmp opcode,1CD8  
    je VM\_FCOMP  
    cmp opcode,1CDC  
    je VM\_FCOMPQ  
    cmp opcode,0FFD9  
    je VM\_FCOS  
    cmp opcode,0F6D9  
    je VM\_FDECSTP  
    cmp opcode,34D8  
    je VM\_FDIV  
    cmp opcode,34DC  
    je VM\_FDIVQ  
    cmp opcode,04DB  
    je VM\_FILD  
    cmp opcode,2CDF  
    je VM\_FILDQ  
    cmp opcode,0F7D9  
    je VM\_FINCSTP  
    cmp opcode,0E3DB  
    je VM\_FINIT  
    cmp opcode,1CDB  
    je VM\_FISTP  
    cmp opcode,3CDF  
    je VM\_FISTPQ  
    cmp opcode,1CDF  
    je VM\_FISTPW  
    cmp opcode,24DA  
    je VM\_FISUB  
    cmp opcode,24DE  
    je VM\_FISUBW  
    cmp opcode,04D9  
    je VM\_FLD  
    cmp opcode,04DD  
    je VM\_FLDQ  
    cmp opcode,2CDB  
    je VM\_FLDT  
    cmp opcode,0E8D9  
    je VM\_FLD1  
    cmp opcode,2CD9  
    je VM\_FLDCW  
    cmp opcode,0ECD9  
    je VM\_FLDLG2  
    cmp opcode,0EDD9  
    je VM\_FLDLN2  
    cmp opcode,0EBD9  
    je VM\_FLDPI  
    cmp opcode,0EED9  
    je VM\_FLDZ  
    cmp opcode,0CD8  
    je VM\_FMUL  
    cmp opcode,0CDC  
    je VM\_FMULQ  
    cmp opcode,0F3D9  
    je VM\_FPATAN  
    cmp opcode,0F8D9  
    je VM\_FPREM  
    cmp opcode,0F5D9  
    je VM\_FPREM1  
    cmp opcode,0F2D9  
    je VM\_FPTAN  
    cmp opcode,0FCD9  
    je VM\_FRNDINT  
    cmp opcode,0FED9  
    je VM\_FSIN  
    cmp opcode,0FAD9  
    je VM\_FSQRT  
    cmp opcode,14D9  
    je VM\_FST  
     cmp opcode,14DD  
    je VM\_FSTQ  
    cmp opcode,3CD9  
    je VM\_FSTCW  
    cmp opcode,1CD9  
    je VM\_FSTP  
    cmp opcode,1CDD  
    je VM\_FSTPQ  
    cmp opcode,3CDB  
    je VM\_FSTPT  
    cmp opcode,0E0DF  
    je VM\_FSTSW  
    cmp opcode,24D8  
    je VM\_FSUB  
    cmp opcode,24DC  
    je VM\_FSUBQ  
    cmp opcode,2CD8  
    je VM\_FSUBR  
    cmp opcode,2CDC  
    je VM\_FSUBRQ  
    cmp opcode,0E4D9  
    je VM\_FTST  
    cmp opcode,0F1D9  
    je VM\_FYL2X  
    jmp error  
norw:  
    mov opcode,\[opaddr+9\],2  
    cmp opcode,9C66  
    je VM\_NORW\_F  
    jmp VM\_NORW  
// \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
VM\_ADDB:  
    lbl opaddr,"VM\_ADDB"  
    jmp findnext  
VM\_ADDB\_F:  
    lbl opaddr,"VM\_ADDB\_F"  
    jmp findnext  
VM\_ADDW:  
    lbl opaddr,"VM\_ADDW"  
    jmp findnext  
VM\_ADDW\_F:  
    lbl opaddr,"VM\_ADDW\_F"  
    jmp findnext  
VM\_ADD:  
    lbl opaddr,"VM\_ADD"  
    jmp findnext  
VM\_ADD\_F:  
    lbl opaddr,"VM\_ADD\_F"  
    jmp findnext  
VM\_MULB\_F:  
    lbl opaddr,"VM\_MULB\_F"  
    jmp findnext  
VM\_MULW\_F:  
    lbl opaddr,"VM\_MULW\_F"  
    jmp findnext  
VM\_MUL\_F:  
    lbl opaddr,"VM\_MUL\_F"  
    jmp findnext  
VM\_IMULB\_F:  
    lbl opaddr,"VM\_IMULB\_F"  
    jmp findnext  
VM\_IMULW\_F:  
    lbl opaddr,"VM\_IMULW\_F"  
    jmp findnext  
VM\_IMUL\_F:  
    lbl opaddr,"VM\_IMUL\_F"  
    jmp findnext  
VM\_DIVB:  
    lbl opaddr,"VM\_DIVB"  
    jmp findnext  
VM\_DIVW:  
    lbl opaddr,"VM\_DIVW"  
    jmp findnext  
VM\_DIV:  
    lbl opaddr,"VM\_DIV"  
    jmp findnext  
VM\_IDIVB:  
    lbl opaddr,"VM\_IDIVB"  
    jmp findnext  
VM\_IDIVW:  
    lbl opaddr,"VM\_IDIVW"  
    jmp findnext  
VM\_IDIV:  
    lbl opaddr,"VM\_IDIV"  
    jmp findnext  
VM\_SHLB:  
    lbl opaddr,"VM\_SHLB"  
    jmp findnext  
VM\_SHLB\_F:  
    lbl opaddr,"VM\_SHLB\_F"  
    jmp findnext  
VM\_SHLW:  
    lbl opaddr,"VM\_SHLW"  
    jmp findnext  
VM\_SHLW\_F:  
    lbl opaddr,"VM\_SHLW\_F"  
    jmp findnext  
VM\_SHL:  
    lbl opaddr,"VM\_SHL"  
    jmp findnext  
VM\_SHL\_F:  
    lbl opaddr,"VM\_SHL\_F"  
    jmp findnext  
VM\_SHLD\_F:  
    lbl opaddr,"VM\_SHLD\_F"  
    jmp findnext  
VM\_SHRB:  
    lbl opaddr,"VM\_SHRB"  
    jmp findnext  
VM\_SHRB\_F:  
    lbl opaddr,"VM\_SHRB\_F"  
    jmp findnext  
VM\_SHRW:  
    lbl opaddr,"VM\_SHRW"  
    jmp findnext  
VM\_SHRW\_F:  
    lbl opaddr,"VM\_SHRW\_F"  
    jmp findnext  
VM\_SHR:  
    lbl opaddr,"VM\_SHR"  
    jmp findnext  
VM\_SHR\_F:  
    lbl opaddr,"VM\_SHR\_F"  
    jmp findnext  
VM\_SHRD\_F:  
    lbl opaddr,"VM\_SHRD\_F"  
    jmp findnext  
VM\_MOVB\_S1\_CS1:  
    lbl opaddr,"VM\_MOVB\_S1\_CS1"  
    jmp findnext  
VM\_MOVB\_S1\_DS1:  
    lbl opaddr,"VM\_MOVB\_S1\_DS1"  
    jmp findnext  
VM\_MOVB\_S1\_ES1:  
    lbl opaddr,"VM\_MOVB\_S1\_ES1"  
    jmp findnext  
VM\_MOVB\_S1\_FS1:  
    lbl opaddr,"VM\_MOVB\_S1\_FS1"  
    jmp findnext  
VM\_MOVB\_S1\_GS1:  
    lbl opaddr,"VM\_MOVB\_S1\_GS1"  
    jmp findnext  
VM\_MOVB\_S1\_SS1:  
    lbl opaddr,"VM\_MOVB\_S1\_SS1"  
    jmp findnext  
VM\_MOVW\_S1\_CS1:  
    lbl opaddr,"VM\_MOVW\_S1\_CS1"  
    jmp findnext  
VM\_MOVW\_S1\_DS1:  
    lbl opaddr,"VM\_MOVW\_S1\_DS1"  
    jmp findnext  
VM\_MOVW\_S1\_ES1:  
    lbl opaddr,"VM\_MOVW\_S1\_ES1"  
    jmp findnext  
VM\_MOVW\_S1\_FS1:  
    lbl opaddr,"VM\_MOVW\_S1\_FS1"  
    jmp findnext  
VM\_MOVW\_S1\_GS1:  
    lbl opaddr,"VM\_MOVW\_S1\_GS1"  
    jmp findnext  
VM\_MOVW\_S1\_SS1:  
    lbl opaddr,"VM\_MOVW\_S1\_SS1"  
    jmp findnext  
VM\_MOV\_S1\_CS1:  
    lbl opaddr,"VM\_MOV\_S1\_CS1"  
    jmp findnext  
VM\_MOV\_S1\_DS1:  
    lbl opaddr,"VM\_MOV\_S1\_DS1"  
    jmp findnext  
VM\_MOV\_S1\_ES1:  
    lbl opaddr,"VM\_MOV\_S1\_ES1"  
    jmp findnext  
VM\_MOV\_S1\_FS1:  
    lbl opaddr,"VM\_MOV\_S1\_FS1"  
    jmp findnext  
VM\_MOV\_S1\_GS1:  
    lbl opaddr,"VM\_MOV\_S1\_GS1"  
    jmp findnext  
VM\_MOV\_S1\_SS1:  
    lbl opaddr,"VM\_MOV\_S1\_SS1"  
    jmp findnext  
VM\_MOVB\_CS1\_S2:  
    lbl opaddr,"VM\_MOVB\_CS1\_S2"  
    jmp findnext  
VM\_MOVB\_DS1\_S2:  
    lbl opaddr,"VM\_MOVB\_DS1\_S2"  
    jmp findnext  
VM\_MOVB\_ES1\_S2:  
    lbl opaddr,"VM\_MOVB\_ES1\_S2"  
    jmp findnext  
VM\_MOVB\_FS1\_S2:  
    lbl opaddr,"VM\_MOVB\_FS1\_S2"  
    jmp findnext  
VM\_MOVB\_GS1\_S2:  
    lbl opaddr,"VM\_MOVB\_GS1\_S2"  
    jmp findnext  
VM\_MOVB\_SS1\_S2:  
    lbl opaddr,"VM\_MOVB\_SS1\_S2"  
    jmp findnext  
VM\_MOVW\_CS1\_S2:  
    lbl opaddr,"VM\_MOVW\_CS1\_S2"  
    jmp findnext  
VM\_MOVW\_DS1\_S2:  
    lbl opaddr,"VM\_MOVW\_DS1\_S2"  
    jmp findnext  
VM\_MOVW\_ES1\_S2:  
    lbl opaddr,"VM\_MOVW\_ES1\_S2"  
    jmp findnext  
VM\_MOVW\_FS1\_S2:  
    lbl opaddr,"VM\_MOVW\_FS1\_S2"  
    jmp findnext  
VM\_MOVW\_GS1\_S2:  
    lbl opaddr,"VM\_MOVW\_GS1\_S2"  
    jmp findnext  
VM\_MOVW\_SS1\_S2:  
    lbl opaddr,"VM\_MOVW\_SS1\_S2"  
    jmp findnext  
VM\_MOV\_CS1\_S2:  
    lbl opaddr,"VM\_MOV\_CS1\_S2"  
    jmp findnext  
VM\_MOV\_DS1\_S2:  
    lbl opaddr,"VM\_MOV\_DS1\_S2"  
    jmp findnext  
VM\_MOV\_ES1\_S2:  
    lbl opaddr,"VM\_MOV\_ES1\_S2"  
    jmp findnext  
VM\_MOV\_FS1\_S2:  
    lbl opaddr,"VM\_MOV\_FS1\_S2"  
    jmp findnext  
VM\_MOV\_GS1\_S2:  
    lbl opaddr,"VM\_MOV\_GS1\_S2"  
    jmp findnext  
VM\_MOV\_SS1\_S2:  
    lbl opaddr,"VM\_MOV\_SS1\_S2"  
    jmp findnext  
VM\_PUSH\_SP:  
    lbl opaddr,"VM\_PUSH\_SP"  
    jmp findnext  
VM\_PUSH\_ESP:  
    lbl opaddr,"VM\_PUSH\_ESP"  
    jmp findnext  
VM\_POP\_SP:  
    lbl opaddr,"VM\_POP\_SP"  
    jmp findnext  
VM\_POP\_ESP:  
    lbl opaddr,"VM\_POP\_ESP"  
    jmp findnext  
VM\_POP\_AX:  
    lbl opaddr,"VM\_POP\_AX"  
    jmp findnext  
VM\_POP\_EAX:  
    lbl opaddr,"VM\_POP\_EAX"  
    jmp findnext  
VM\_POP\_CX:  
    lbl opaddr,"VM\_POP\_CX"  
    jmp findnext  
VM\_POP\_ECX:  
    lbl opaddr,"VM\_POP\_ECX"  
    jmp findnext  
VM\_POP\_DX:  
    lbl opaddr,"VM\_POP\_DX"  
    jmp findnext  
VM\_POP\_EDX:  
    lbl opaddr,"VM\_POP\_EDX"  
    jmp findnext  
VM\_PUSH\_CR0:  
    lbl opaddr,"VM\_PUSH\_CR0"  
    jmp findnext  
VM\_PUSH\_CR1:  
    lbl opaddr,"VM\_PUSH\_CR1"  
    jmp findnext  
VM\_PUSH\_CR2:  
    lbl opaddr,"VM\_PUSH\_CR2"  
    jmp findnext  
VM\_PUSH\_CR3:  
    lbl opaddr,"VM\_PUSH\_CR3"  
    jmp findnext  
VM\_PUSH\_CR4:  
    lbl opaddr,"VM\_PUSH\_CR4"  
    jmp findnext  
VM\_PUSH\_CR5:  
    lbl opaddr,"VM\_PUSH\_CR5"  
    jmp findnext  
VM\_PUSH\_CR6:  
    lbl opaddr,"VM\_PUSH\_CR6"  
    jmp findnext  
VM\_PUSH\_CR7:  
    lbl opaddr,"VM\_PUSH\_CR7"  
    jmp findnext  
VM\_PUSH\_DR0:  
    lbl opaddr,"VM\_PUSH\_DR0"  
    jmp findnext  
VM\_PUSH\_DR1:  
    lbl opaddr,"VM\_PUSH\_DR1"  
    jmp findnext  
VM\_PUSH\_DR2:  
    lbl opaddr,"VM\_PUSH\_DR2"  
    jmp findnext  
VM\_PUSH\_DR3:  
    lbl opaddr,"VM\_PUSH\_DR3"  
    jmp findnext  
VM\_PUSH\_DR4:  
    lbl opaddr,"VM\_PUSH\_DR4"  
    jmp findnext  
VM\_PUSH\_DR5:  
    lbl opaddr,"VM\_PUSH\_DR5"  
    jmp findnext  
VM\_PUSH\_DR6:  
    lbl opaddr,"VM\_PUSH\_DR6"  
    jmp findnext  
VM\_PUSH\_DR7:  
    lbl opaddr,"VM\_PUSH\_DR7"  
    jmp findnext  
VM\_POP\_CR0:  
    lbl opaddr,"VM\_POP\_CR0"  
    jmp findnext  
VM\_POP\_CR1:  
    lbl opaddr,"VM\_POP\_CR1"  
    jmp findnext  
VM\_POP\_CR2:  
    lbl opaddr,"VM\_POP\_CR2"  
    jmp findnext  
VM\_POP\_CR3:  
    lbl opaddr,"VM\_POP\_CR3"  
    jmp findnext  
VM\_POP\_CR4:  
    lbl opaddr,"VM\_POP\_CR4"  
    jmp findnext  
VM\_POP\_CR5:  
    lbl opaddr,"VM\_POP\_CR5"  
    jmp findnext  
VM\_POP\_CR6:  
    lbl opaddr,"VM\_POP\_CR6"  
    jmp findnext  
VM\_POP\_CR7:  
    lbl opaddr,"VM\_POP\_CR7"  
    jmp findnext  
VM\_POP\_DR0:  
    lbl opaddr,"VM\_POP\_DR0"  
    jmp findnext  
VM\_POP\_DR1:  
    lbl opaddr,"VM\_POP\_DR1"  
    jmp findnext  
VM\_POP\_DR2:  
    lbl opaddr,"VM\_POP\_DR2"  
    jmp findnext  
VM\_POP\_DR3:  
    lbl opaddr,"VM\_POP\_DR3"  
    jmp findnext  
VM\_POP\_DR4:  
    lbl opaddr,"VM\_POP\_DR4"  
    jmp findnext  
VM\_POP\_DR5:  
    lbl opaddr,"VM\_POP\_DR5"  
    jmp findnext  
VM\_POP\_DR6:  
    lbl opaddr,"VM\_POP\_DR6"  
    jmp findnext  
VM\_POP\_DR7:  
    lbl opaddr,"VM\_POP\_DR7"  
    jmp findnext  
VM\_PUSH\_CS:  
    lbl opaddr,"VM\_PUSH\_CS"  
    jmp findnext  
VM\_PUSH\_DS:  
    lbl opaddr,"VM\_PUSH\_DS"  
    jmp findnext  
VM\_PUSH\_ES:  
    lbl opaddr,"VM\_PUSH\_ES"  
    jmp findnext  
VM\_PUSH\_FS:  
    lbl opaddr,"VM\_PUSH\_FS"  
    jmp findnext  
VM\_PUSH\_GS:  
    lbl opaddr,"VM\_PUSH\_GS"  
    jmp findnext  
VM\_PUSH\_SS:  
    lbl opaddr,"VM\_PUSH\_SS"  
    jmp findnext  
VM\_POP\_DS:  
    lbl opaddr,"VM\_POP\_DS"  
    jmp findnext  
VM\_POP\_ES:  
    lbl opaddr,"VM\_POP\_ES"  
    jmp findnext  
VM\_POP\_FS:  
    lbl opaddr,"VM\_POP\_FS"  
    jmp findnext  
VM\_POP\_GS:  
    lbl opaddr,"VM\_POP\_GS"  
    jmp findnext  
VM\_POP\_SS:  
    lbl opaddr,"VM\_POP\_SS"  
    jmp findnext  
VM\_PUSH\_CTXB0:  
    lbl opaddr,"VM\_PUSH\_CTXB0"  
    jmp findnext  
VM\_PUSH\_CTXB1:  
    lbl opaddr,"VM\_PUSH\_CTXB1"  
    jmp findnext  
VM\_PUSH\_CTXW:  
    lbl opaddr,"VM\_PUSH\_CTXW"  
    jmp findnext  
VM\_PUSH\_CTX:  
    lbl opaddr,"VM\_PUSH\_CTX"  
    jmp findnext  
VM\_POP\_CTXB0:  
    lbl opaddr,"VM\_POP\_CTXB0"  
    jmp findnext  
VM\_POP\_CTXB1:  
    lbl opaddr,"VM\_POP\_CTXB1"  
    jmp findnext  
VM\_POP\_CTXW:  
    lbl opaddr,"VM\_POP\_CTXW"  
    jmp findnext  
VM\_POP\_CTX:  
    lbl opaddr,"VM\_POP\_CTX"  
    jmp findnext  
VM\_PUSH\_IMM8:  
    lbl opaddr,"VM\_PUSH\_IMM8"  
    jmp findnext  
VM\_PUSH\_IMM16:  
    lbl opaddr,"VM\_PUSH\_IMM16"  
    jmp findnext  
VM\_PUSH\_IMM32:  
    lbl opaddr,"VM\_PUSH\_IMM32"  
    jmp findnext  
VM\_PUSHD\_IMM8:  
    lbl opaddr,"VM\_PUSHD\_IMM8"  
    jmp findnext  
VM\_PUSHD\_IMM16:  
    lbl opaddr,"VM\_PUSHD\_IMM16"  
    jmp findnext  
VM\_NORB:  
    lbl opaddr,"VM\_NORB"  
    jmp findnext  
VM\_NORB\_F:  
    lbl opaddr,"VM\_NORB\_F"  
    jmp findnext  
VM\_NORW:  
    lbl opaddr,"VM\_NORW"  
    jmp findnext  
VM\_NORW\_F:  
    lbl opaddr,"VM\_NORW\_F"  
    jmp findnext  
VM\_NOR:  
    lbl opaddr,"VM\_NOR"  
    jmp findnext  
VM\_NOR\_F:  
    lbl opaddr,"VM\_NOR\_F"  
    jmp findnext  
VM\_WAIT:  
    lbl opaddr,"VM\_WAIT"  
    jmp findnext  
VM\_JMP:  
    lbl opaddr,"VM\_JMP"  
    jmp findnext  
VM\_RETN:  
    lbl opaddr,"VM\_RETN"  
    jmp findnext  
VM\_RETF:  
    lbl opaddr,"VM\_RETF"  
    jmp findnext  
VM\_F2XM1:  
    lbl opaddr,"VM\_F2XM1"  
    jmp findnext  
VM\_FABS:  
    lbl opaddr,"VM\_FABS"  
    jmp findnext  
VM\_FADD:  
    lbl opaddr,"VM\_FADD"  
    jmp findnext  
VM\_FADDQ:  
    lbl opaddr,"VM\_FADDQ"  
    jmp findnext  
VM\_FCHS:  
    lbl opaddr,"VM\_FCHS"  
    jmp findnext  
VM\_FCLEX:  
    lbl opaddr,"VM\_FCLEX"  
    jmp findnext  
VM\_FCOMP:  
    lbl opaddr,"VM\_FCOMP"  
    jmp findnext  
VM\_FCOMPQ:  
    lbl opaddr,"VM\_FCOMPQ"  
    jmp findnext  
VM\_FCOS:  
    lbl opaddr,"VM\_FCOS"  
    jmp findnext  
VM\_FDECSTP:  
    lbl opaddr,"VM\_FDECSTP"  
    jmp findnext  
VM\_FDIV:  
    lbl opaddr,"VM\_FDIV"  
    jmp findnext  
VM\_FDIVQ:  
    lbl opaddr,"VM\_FDIVQ"  
    jmp findnext  
VM\_FILD:  
    lbl opaddr,"VM\_FILD"  
    jmp findnext  
VM\_FILDQ:  
    lbl opaddr,"VM\_FILDQ"  
    jmp findnext  
VM\_FINCSTP:  
    lbl opaddr,"VM\_FINCSTP"  
    jmp findnext  
VM\_FINIT:  
    lbl opaddr,"VM\_FINIT"  
    jmp findnext  
VM\_FISTP:  
    lbl opaddr,"VM\_FISTP"  
    jmp findnext  
VM\_FISTPQ:  
    lbl opaddr,"VM\_FISTPQ"  
    jmp findnext  
VM\_FISTPW:  
    lbl opaddr,"VM\_FISTPW"  
    jmp findnext  
VM\_FISUB:  
    lbl opaddr,"VM\_FISUB"  
    jmp findnext  
VM\_FISUBW:  
    lbl opaddr,"VM\_FISUBW"  
    jmp findnext  
VM\_FLD:  
    lbl opaddr,"VM\_FLD"  
    jmp findnext  
VM\_FLDQ:  
    lbl opaddr,"VM\_FLDQ"  
    jmp findnext  
VM\_FLDT:  
    lbl opaddr,"VM\_FLDT"  
    jmp findnext  
VM\_FLD1:  
    lbl opaddr,"VM\_FLD1"  
    jmp findnext  
VM\_FLDCW:  
    lbl opaddr,"VM\_FLDCW"  
    jmp findnext  
VM\_FLDLG2:  
    lbl opaddr,"VM\_FLDLG2"  
    jmp findnext  
VM\_FLDLN2:  
    lbl opaddr,"VM\_FLDLN2"  
    jmp findnext  
VM\_FLDPI:  
    lbl opaddr,"VM\_FLDPI"  
    jmp findnext  
VM\_FLDZ:  
    lbl opaddr,"VM\_FLDZ"  
    jmp findnext  
VM\_FMUL:  
    lbl opaddr,"VM\_FMUL"  
    jmp findnext  
VM\_FMULQ:  
    lbl opaddr,"VM\_FMULQ"  
    jmp findnext  
VM\_FPATAN:  
    lbl opaddr,"VM\_FPATAN"  
    jmp findnext  
VM\_FPREM:  
    lbl opaddr,"VM\_FPREM"  
    jmp findnext  
VM\_FPREM1:  
    lbl opaddr,"VM\_FPREM1"  
    jmp findnext  
VM\_FPTAN:  
    lbl opaddr,"VM\_FPTAN"  
    jmp findnext  
VM\_FRNDINT:  
    lbl opaddr,"VM\_FRNDINT"  
    jmp findnext  
VM\_FSIN:  
    lbl opaddr,"VM\_FSIN"  
    jmp findnext  
VM\_FSQRT:  
    lbl opaddr,"VM\_FSQRT"  
    jmp findnext  
VM\_FST:  
    lbl opaddr,"VM\_FST"  
    jmp findnext  
VM\_FSTQ:  
    lbl opaddr,"VM\_FSTQ"  
    jmp findnext  
VM\_FSTCW:  
    lbl opaddr,"VM\_FSTCW"  
    jmp findnext  
VM\_FSTP:  
    lbl opaddr,"VM\_FSTP"  
    jmp findnext  
VM\_FSTPQ:  
    lbl opaddr,"VM\_FSTPQ"  
    jmp findnext  
VM\_FSTPT:  
    lbl opaddr,"VM\_FSTPT"  
    jmp findnext  
VM\_FSTSW:  
    lbl opaddr,"VM\_FSTSW"  
    jmp findnext  
VM\_FSUB:  
    lbl opaddr,"VM\_FSUB"  
    jmp findnext  
VM\_FSUBQ:  
    lbl opaddr,"VM\_FSUBQ"  
    jmp findnext  
VM\_FSUBR:  
    lbl opaddr,"VM\_FSUBR"  
    jmp findnext  
VM\_FSUBRQ:  
    lbl opaddr,"VM\_FSUBRQ"  
    jmp findnext  
VM\_FTST:  
    lbl opaddr,"VM\_FTST"  
    jmp findnext  
VM\_FYL2X:  
    lbl opaddr,"VM\_FYL2X"  
    jmp findnext  
// \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
findnext:  
    add optbl,4  
    inc opcnt  
    cmp opcnt,100  
    jne findop  
    itoa findcnt,10.  
    eval "Find {$RESULT} VM Opcode."  
    log $RESULT,""  
    msg $RESULT  
    jmp end  
error:  
    msg "出错了!@#$%^&\*"  
    pause  
end:  
    ret

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || vmp120.rar \[解压密码：PEDIY\]  
 |