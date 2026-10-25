
# Obsidium 1.2 Unpack script v0.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6900.htm`](https://www.pediy.com/kssd/pediy06/pediy6900.htm)

本想用 OM 写，可是发了一下午时间都没有写出来:-(,我要补课了:(.哪位有兴趣就帮忙转换一下吧.

脚本解决了，水生涯那篇关于 NOTEPAD 的脱文在 winxp sp1,sp2 下调试不成功的问题.当然这个问题的解决得助于 yock 的，没有他我可能要发很多时间的说.

脚本没有处理"特别照顾"的 API，跟一下也不难吧

,晕哦，把后面调试的东西也加进去了,编辑了一下

```
 Jahren    
//////////////////////////////////////////////////  
  Obsidium 1.2 Unpack script v0.1  
  Author:  loveboom  
  Email : bmd2chen@tom.com  
  OS    : WinXP sp2,Ollydbg 1.1,OllyScript v0.92  
  Date  : 2004-11-30  
           Action: Remove Junk code,Found stolen code  
  Config: Ignore other exceptions except 'Memory access   violation'and patch 'SetUnhandledExceptionFilter'  
  Note  : If you have one or more question, email me please,thank you!  
//////////////////////////////////////////////////  
*/
var espval  
var bbase  
var addr  
var bsize  
var goaddr  
var goaddr1  
var tmpaddr  
Start:  
   msgyn "Setting:Ignore other exceptions except'Memory access violation'and and patch 'SetUnhandledExceptionFilter',Continue?"  
   cmp $RESULT,0  
   jne lbl1  
   ret  
lbl1:  
   dbh  
   sti  
   sti  
   mov espval,esp  
   gpa "CheckRemoteDebuggerPresent","Kernel32.dll"  
   cmp $RESULT,0  
   je lblestot  
   mov addr,$RESULT      //Patch 'CheckRemoteDebuggerPresent'  
   mov [addr],#33C0C20800#    //'xor eax,eax,retn 8'  
lblestot:  
   run  
   esto  
   esto  
lbl2:  
   gpa "VirtualAllocEx","kernel32.dll"  
   cmp $RESULT,0  
   je lblerros  
   bp $RESULT  
   esto  
lbl3:  
   bc $RESULT  
   mov bsize,edx      //Get VirtualAlloc size    
   rtu  
   mov bbase,eax       //Get VirtualAlloc Base  
   run  
lblbp:  
   esto  
   esto  
   esto  
   esto  
   esto  
   bprm bbase,bsize      //Set a Memory break point  
   esto  
lbl4:  
   bpmc  
   rtr  
   sto  
lblfind:  
   find eip,#66F706200074#    //found 'test word ptr [esi],20]'  
   cmp $RESULT,0  
   je lblabort  
   mov addr,$RESULT  
   mov [addr],#66F706080075#    //Replace to 'test word ptr [esi],08'  
   find addr,#83C60883C704#    //found 'add esi,8,add edi,4'  
   cmp $RESULT,0  
   je lblabort  
   mov goaddr,$RESULT  
   mov goaddr1,goaddr  
   find addr,#FF50??85C074#    //found 'CALL DWORD PTR DS:[EAX+??]'  
   cmp $RESULT,0  
   je lblabort  
   mov addr,$RESULT  
   add addr,6  
   sub goaddr,addr  
   inc addr  
   mov tmpaddr,[addr]  
   dec goaddr  
   dec addr  
   mov [addr],goaddr  
   inc addr  
   mov [addr],tmpaddr  
   add addr,1  
   find addr,#FF50??85C074#  
   cmp $RESULT,0  
   je lblabort  
   mov addr,$RESULT  
   add addr,6  
   sub goaddr1,addr  
   inc addr  
   mov tmpaddr,[addr]  
   dec goaddr1  
   dec addr  
   mov [addr],goaddr1  
   inc addr  
   mov [addr],tmpaddr  
   run  
lblesto:  
   findop eip,#FFE1#      //Found command 'JMP ECX'  
   bp $RESULT  
   esto  
lbl5:  
   bc $RESULT  
   bphws espval,"r"  
   log espval  
   run  
lblClearJunkCode:    //Clear Junk code  
   bphwc espval  
   repl eip,#EB01??#,#909090#,FF  
   repl eip,#EB02????#,#90909090#,FF  
   repl eip,#EB03??????#,#9090909090#,FF  
   repl eip,#EB04????????#,#909090909090#,FF  
lblLogCode:  
   find eip,#9C#  
   bp $RESULT  
   ti  
lbl6:  
   bc $RESULT  
   run  
   bprm bbase,bsize  
   run  
lbl7:  
   bpmc  
   dbs  
   cmt eip,"Now,press ALT+V+N open trace window,you will found stolen code!"  
lblend:  
   msg "Script by loveboom[DFCG[FCG],Thank you for using my script!"  
   ret  
lblerros:  
   msg "Sorry script require OS:WINDOWS 2x/xp!"  
   ret    
lblabort:  
   msg "Error,Script aborted!,Meybe target is not protect by OBSIDIUM 1.2 or Target is a vb program."  
   ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
