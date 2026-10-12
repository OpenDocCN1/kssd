
```
代码:
00406088    6A 00            PUSH 0
0040608A    EB 01            JMP SHORT masm.0040608D
0040608C    90               NOP
0040608D    68 97604000      PUSH masm.00406097
00406092  - E9 2FB0FFFF      JMP masm.004010C6
00406097    A3 00304000      MOV DWORD PTR DS:\[403000\],EAX
0040609C    EB 01            JMP SHORT masm.0040609F
0040609E    90               NOP
0040609F    68 A9604000      PUSH masm.004060A9
004060A4  - E9 35B0FFFF      JMP masm.004010DE                 ; JMP to COMCTL32.InitCommonControls
004060A9    6A 00            PUSH 0
004060AB    EB 01            JMP SHORT masm.004060AE
004060AD    90               NOP
004060AE    68 962628E1      PUSH E1282696
004060B3    810424 98E9171F ADD DWORD PTR SS:\[ESP\],1F17E998
004060BA    6A 00            PUSH 0
004060BC    EB 01            JMP SHORT masm.004060BF
004060BE    90               NOP
004060BF    6A 65            PUSH 65
004060C1    EB 01            JMP SHORT masm.004060C4
004060C3    90               NOP
004060C4    FF35 00304000    PUSH DWORD PTR DS:\[403000\]
004060CA    EB 01            JMP SHORT masm.004060CD
004060CC    90               NOP
004060CD    68 D7604000      PUSH masm.004060D7
004060D2  - E9 F5AFFFFF      JMP masm.004010CC
004060D7    6A 00            PUSH 0
004060D9    EB 01            JMP SHORT masm.004060DC
004060DB    90               NOP
004060DC    68 E6604000      PUSH masm.004060E6
004060E1  - E9 DAAFFFFF      JMP masm.004010C0
```

这里具体的代码我就不在多讲了，自己看也看得懂吧。

好了，到了这里也算是到了终点站吧，收工！

Greetz:

Fly, Jingulong, yock, tDasm, David, ahao, vcasm, UFO(brother), alan(sister), all of my friends and you!

By `loveboom[DFCG][FCG]`

Email: `bmd2chen@tom.com`

点击下载：附件！`pespin.rar`

```
/*

/////////////////////////////////////////////////////////////////////////////////////////////

// PESpin v0.7 OEP finder

// Author: hacnho/VCT2k4

// Email : hacnho@hotmail.com

// Website: nhandan.info/hacnho

// OS : WinXP Pro, OllyDbg 1.10 Final, OllyScript v0.85

// You can download my unpackme at nhandan.info/hacnho/tuts/unpackme_PESpin_07.zip

// for test this script.

// Note: Not work with Delphi (OllyDBG will be crash when loading).

//////////////////////////////////////////////////////////////////////////////////////////////

*/

MSG "Do not clear memory breakpoint! Thanx for using my script!"

var temp

sti

sti

findop eip, #61E9#

bphws esp, "r"

mov temp, esp

run

eob Break1

Break1:

run

esto

esto

esto

esto

eob Break2

Break2:

esto

esto

esto

esto

esto

eob exit

jmp exit

exit:

log eip

cmt eip, "Script by hacnho/VCT2k4"

MSG "Please press Shift+F9 one times! When you trace to a jump (E9 XXXXXXXX). Press Enter, you'll still on OEP! Ctrl+A for analyze."

ret
```

呵呵，那个作者的只是找 `oep`，不会修复 `iat` 的。

注意，脚本不要在 `ollyscript` `v0.9` 上试，试了也通不过的，因为 `v0.9` 有 `BUG` 的。

`pespin` `v0.7` 脱壳脚本

```
/*

//////////////////////////////////////////////////

    PESpin v0.7 Stolen Code Finder v0.1

    Author:    loveboom

    Email : bmd2chen@tom.com

    OS    : WinXP sp1, Ollydbg 1.1, OllyScript v0.85

    Date  : 2004-7-10

        Action: Auto fix IAT, Removed Junkcode, fix oep code(if target is vb's program)

    Config: Ignore other exceptions except 'Invalid or privileged instruction'

    Note  : If you have one or more question, email me please, thank you!

//////////////////////////////////////////////////

*/

var addr

var isvbapp            //target is a vb application??

var espval             //esp value

var iatstart           //iat start address

var cbase

var csize

var isasm

var isvc

gmi eip, CODEBASE

mov cbase, $RESULT

gmi eip, CODESIZE

mov csize, $RESULT

start:

   msgyn "Setting:Ignore other exceptions except 'Invalid or privileged instruction',Continue?"

   cmp $RESULT, 0

   je lblret

lbl1:

   dbh                  //Hide your debugger

   run

   esto

   esto

lbl2:

   gpa "LoadLibraryA", "kernel32.dll"    //GetProcess

   mov addr, $RESULT

   bp addr

   esto

lbl3:

   bc addr

   rtu

   cmp eip, 70000000    //System is wixp and target is a Vb program ?

   jb lblnext

   sto

   rtu

lblnext:

   mov isvbapp, \[ebx\]

   findop eip, #830A00#    //find 'OR \[EDX\],0'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   go addr

   mov iatstart, edx

   rtr

   sto

checkvb:

   mov espval, esp

   add espval, 4

   cmp isvbapp, 4256534M    //is that a vb application??

   jne lbl4

   msgyn "Target like a vb program, go vb module??"

   cmp $RESULT, 0

   je  lbl4

   mov isvbapp, 1

   jmp lblbpesp

lbl4:

   find eip, #8944241C61#    //found 'mov ss:\[esp+1c\],eax'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   mov \[addr\], #36890290#    //Replace to 'mov ss:\[edx\],eax'

   find eip, #8902EB#        //found 'mov ds:\[edx\],eax'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   fill addr, 2, 90        //Replace to 'NOP'

   findop eip, #7635#        //found 'JBE SHORT Address'

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   mov \[addr\], #EB#        //Replace 'JMP SHORT Address'

   find eip, #83C704#        //Target is a MASM program?

   cmp $RESULT, 0

   je lblabort

   mov isasm, $RESULT

   bp isasm

   find eip, #EB06??8907#    //Target is a VC/Delphi program?

   cmp $RESULT, 0

   je lblabort

   mov addr, $RESULT

   add  addr, 3

   mov isvc, addr

   bp isvc

   run

lbleob:

   cmp eip, isasm        //If target is a vc/delphi program then goto vc/delphi module

   jne lblvc

   bc isasm

   mov \[isasm\], #66C747FFFF25#

   add isasm, 6

   mov \[isasm\], #895701EB05#

lblbpesp:

   bphws espval, "r"        //set a hardware breakpoint esp+4

   cmp isvbapp, 1

   je lblvb

   run

lbl5:

   bphwc espval

   cmp isvc, 1            //if Target is  a VC/Delphi program then clear code

   jne lblnext1

   fill filladdr, 30, 00

lblnext1:

   repl eip, #EB01??#, #909090#, FF

   msg "Junk Code has been Removed!"

   cmt eip, "Stolen code,"

lblend:

   msg "Script by loveboom[DFCG][FCG], Thank you for using my script!"

lblret:

   ret

lblabort:            //if error then abort script

   msg "Error, Script aborted! Maybe target is not protect by PESPIN v0.7 or your config error!"

   ret

//////////////////////////////////////

//

//  Vb module

/////////////////////////////////////

lblvb:

   run

lblvb1:

   bphwc espval

   bprm cbase, csize

   run

lblvb2:

   bpmc

lblvbfixoep:

   mov addr, eip

   add addr, 6

   //eval "OEP is: {addr}"

   //cmt addr, $RESULT

   mov \[addr\], 68

   add addr, 1

   mov espval, esp

   add espval, 4

   mov \[addr\], \[espval\]

   add addr, 4

   mov \[addr\], #E8F0FFFFFF#

   add addr, 5

   //eval "Target's IAT start address is {iatstart}"

   //cmt addr, $RESULT

   jmp lblend

////////////////////////////

//

// VC/Delphi module

//

///////////////////////////

lblvc:

var addr1

var addr2

var addrval

var filladdr

bc isvc                    //Clear Break point

mov addr1, isvc

mov \[addr1\], #E9#

mov addr2, cbase

add addr2, csize

sub addr2, 2c

add addr1, 1

mov addrval, addr2

sub addrval, addr1

sub addrval, 4

mov \[addr1\], addrval

mov filladdr, addr2

mov \[addr2\], #609CBB#

add addr2, 3

mov \[addr2\], cbase

add addr2, 4

mov \[addr2\], #B9#

add addr2, 1

mov \[addr2\], csize

add addr2, 4

mov \[addr2\], #8B133BD7750C#

add addr2, 6

mov \[addr2\], #8B17891383C3#

add addr2, 6
```
