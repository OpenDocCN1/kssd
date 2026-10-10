

# 发布个自己写的 tElock 0.98 脱壳脚本，OEP finder+fixed IAT!

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-788.htm`](https://www.pediy.com/kssd/pediy07/pediy7-788.htm)

在 Win2003+Sp1 下也测试通过   
/\*  
//////////////////////////////////////  
tElock 0.98 -> tE! Unpacker  
Author:Wer412  
Email:Wer412@vip.163.com  
OS:Winxp+Sp2,OllyDbg1.10,OllyScript v0.92  
Date:17/11/2005  
Config:Uncheck "Invald or privaleged instruction" and "ALL FPU exceptions" the boxes on the Exceptions tab in Debugging Options except the topmost one.  
//////////////////////////////////////  
\*/  
var addr  
var count  
var cbase  
var csize  
mov count,0  
gmi eip,CODEBASE  
mov cbase,$RESULT  
gmi eip,CODESIZE  
mov csize,$RESULT  
msgyn "请先设置除"Invald or privaleged instruction" and "ALL FPU exceptions"以外的所有忽略，您确信要继续么？"  
cmp $RESULT,0  
je end  
run  
  
label:  
inc count  
cmp count,2  
je label2  
bprm cbase,csize  
esto  
bpmc  
find eip,#833F008D7F04#  
cmp $RESULT,0  
je label2  
mov addr,$RESULT  
add addr,9  
repl addr,#0F84#,#0F85#,2  
  
label2:  
run  
bprm cbase,csize  
esto  
  
bpmc  
cmt eip,"OEP Found,Please dumped it!"  
msg "Script by Wer412,Thank you for using ...."  
  
end:  
ret

附件:notepad+script.rar 

 4dd466d0e1e0cc123afbe504da2462ca