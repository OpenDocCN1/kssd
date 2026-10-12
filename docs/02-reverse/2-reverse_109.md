
# UltraProtect 1.x/ACprotect 1.22

> 原文：[`www.pediy.com/kssd/pediy06/pediy6465.htm`](https://www.pediy.com/kssd/pediy06/pediy6465.htm)

只能用于会抽主程序代码的 VB 程序，其它的程序你就不要去试了.试了也没用

```
//////////////////////////////////////////////////
  UltraProtect 1.x/ACprotect 1.22 OEP Finder(For VB only!)
    ( for 'oep obfuscation and anti loader' mode)
  Author:  loveboom
  Email : bmd2chen@tom.com
  OS    : Win2kADV sp2,OllyDbg 1.1b,OllyScript v0.62
  Date  : 2004-3-26
  Config: Hide ollydbg(IsDebuggerPresent),Exceptions:uncheck "INT3 breaks".
  Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
```

```
var addr
var paddr   //push address
var Aaddr  // asm Address
var cbase

gmi eip,CODEBASE
mov cbase,$RESULT

gmi eip,CODESIZE
eob lbl1
run

lbl1:
findop eip,#C3#
eob lbl2
bp $RESULT
esto

lbl2:
bc $RESULT
sto
mov addr,esp
eob lbl3
bphws addr,"r"
run

lbl3:
bphwc addr
eob lbl4
bprm cbase,FF
run

lbl4:
bpmc
mov addr,esp
add addr,4
mov paddr,[addr]
mov Aaddr,eip
add Aaddr,7
mov [Aaddr],paddr
sub Aaddr,1
mov [Aaddr],#68#
mov addr,Aaddr
cmt addr,"OEP is Found"
add addr,5
cmt addr,"Here patch code 'call EIP',and then you can dumped it!"
msg "Script by loveboom[DFCG],Thank you for using my script!"
ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
