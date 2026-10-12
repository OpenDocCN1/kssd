
# Script: `ACPROTECT` 1.22

> 原文：[`www.pediy.com/kssd/pediy06/pediy6468.htm`](https://www.pediy.com/kssd/pediy06/pediy6468.htm)

```
/*

//////////////////////////////////////////////////

  UltraProtect 1.x/ACprotect 1.22 OEP Finder(Default mode,not for Delphi)

  Author:  loveboom

  Email : bmd2chen@tom.com

  OS    : Win2kADV sp2,OllyDbg 1.1b,OllyScript v0.62

  Date  : 2004-3-28

  Config: Hide ollydbg(IsDebuggerPresent),Exceptions:uncheck "INT3 breaks".

  Note  : If you have one or more question, email me please,thank you!

//////////////////////////////////////////////////

*/

var bpaddr   //Break Point Address

start:

run

lbl1:

findop eip,#C3#

eob lbl2

bp $RESULT

esto

lbl2:

bc $RESULT

sto

mov bpaddr,esp

eob lbl3

bphws bpaddr,"r"

run

lbl3:

bphwc bpaddr

sto

sto

end:

cmt eip,"OEP found!please dumped it!"

msg "Script by loveboom\[DFCG\],Thank you for using my script!"

ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
