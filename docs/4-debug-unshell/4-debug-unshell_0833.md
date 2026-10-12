
# Hying's pelock v0.7x 脚本

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-554.htm`](https://www.pediy.com/kssd/pediy07/pediy7-554.htm)

使用前的申明：这个脚本只能用于`v0.7x`，因为我也是黑箱操作的，我并不知道具体是`0.7`多少，只知道最近出来的那几个`v0.7x`没有用，对阿达连连看之类的有用。试过`xxx2.52`没有用。因为`osc`插件本身出了点小问题，所以有时得到的结果并不理想，如果你对这个脚本有什么好的方法，欢迎后面跟贴，谢谢!!

```
/*

//////////////////////////////////////////////////

  Hying'pelock unpack script(only for v0.7x) v0.1

  Author:  loveboom

  Email : loveboom#163.com

  OS    : WinXP sp1,Ollydbg 1.1,OllyScript v0.92

  Date  : 2005-3-20

        Action: 停在 Stolen Code 处

  Config: Ignore all exceptions

  Note  : If you have one or more question, email me please,thank you!

//////////////////////////////////////////////////

*/

var addr
var GMHaddr
var jtoaddr
var count
var patchiataddr
var patchiatsize
var cbase
var csize
var siataddr
var dllname
var tmpval

#log

start:

  msgyn "设置:忽略全部异常,继续吗?"
  cmp $RESULT,1
  je lbl1
  ret

lbl1:

  dbh
  gmi eip,CODEBASE
  mov cbase,$RESULT
  gmi eip,CODESIZE
  mov csize,$RESULT
  gpa "CreateFileA","kernel32.dll"
  mov addr,$RESULT
  find addr,#C21C00#   //查找返回处
  mov addr,$RESULT
  bp addr
  esto

lbl2:

  bc addr
  gpa "GetModuleHandleA","kernel32.dll"
  mov GMHaddr,$RESULT
  bprm $RESULT,FF
  esto
  bpmc

lbl3:

/*
查找命令
  MOV BYTE PTR DS:\[EDI\],68
  MOV DWORD PTR DS:\[EDI+1\],ESI
  MOV BYTE PTR DS:\[EDI+5\],0C3
  ADD EDI,6
  MOV DWORD PTR SS:\[ESP-4\],EDI
*/

  find eip,#C60768897701C64705C383C706897C24FC#
  cmp $RESULT,0
  je lblabort
  mov addr,eip
  mov jtoaddr,$RESULT
  fill eip,1,e9
  sub jtoaddr,eip
  sub jtoaddr,5
  inc addr
  mov [addr],jtoaddr   //改成 push api ret 的方式

lblcanti1:

  gpa "ZwSetInformationThread","ntdll.dll"
  cmp $RESULT,0
  je lbleros
  asm $RESULT,"ret 10"

lblgetvinfo:

  gpa "VirtualAlloc","kernel32.dll"
  bp $RESULT
  mov count,5

lblloop1:

  cmp count,0
  je lblloginfo
  dec count
  esto
  jmp lblloop1

lblloginfo:

  bc $RESULT
  mov patchiatsize,esp
  add patchiatsize,8
  mov patchiatsize,[patchiatsize]
  rtu
  mov patchiataddr,eax

lblcp1:

  gpa "lstrcmpA","kernel32.dll"  
  mov addr,$RESULT
  fill addr,1,b8    //让壳检测为没有特殊函数
  inc addr
  mov [addr],1
  add addr,4
  asm addr,"ret 8"
  bp addr
  esto

lbl4:

  bc addr
  rtu

/*
59490F85????????E9????????E80A
  POP ECX
  DEC ECX
  JNZ @B
  JMP Next_DLL
  CALL xxxxxx
*/

  find eip,#59490F85????????E9????????E80A#
  cmp $RESULT,0
  je lblabort
  mov addr,$RESULT
  add addr,d
  bp addr
  esto

lbl5:

  bc addr
  go GMHaddr
  rtu
  mov eax,0       //让壳认为没有 ntdll.dll 文件
  gpa "SetThreadPriority","kernel32.dll"
  bp $RESULT

lbl6: 

  esto
  esto
  esto

lbl7:

  bc $RESULT
  rtu
  sto

/*
  POPAD
  PUSH EAX
  PUSH EDX
  PUSH ECX
*/

  find eip,#61505251#
  cmp $RESULT,0
  je lblabort
  go $RESULT

/*
  CMP EAX,40000
  JBE SHORT 003764BE
  ADD ESP,0C
  RETN
*/

  repl eip,#3D00000400760483C40CC3#,#3D00000400EB0483C40CC3#,500
  bprm cbase,csize
  eob lbl8
  ti

lbl8:

  bpmc
  cmt eip,"现在你可以打开 Trace 窗口尝试找回壳所抽代码."
  msgyn "是否让脚本尝试修复 iat?(尝试修复时必须手工输入保存 iat 的起始地址.一般可用最后一个 section),这将需要几分钟时间."
  cmp $RESULT,0
  je lblend
  ask "请写 iat 所要保存的起始地址:"
  cmp $RESULT,0
  je lblend
  mov siataddr,$RESULT
  add patchiatsize,patchiataddr
  mov addr,patchiataddr

lblfixiatloop:

  find addr,#FF35????????813424????????C3#
  cmp $RESULT,0
  je lblexitloop
  mov addr,$RESULT
  add addr,d
  mov [addr],#83c404c3#
  jmp lblfixiatloop

lblexitloop:

  mov addr,cbase
  log patchiatsize
  log patchiataddr

lblfixloop1:

  find addr,#90e9#
  cmp $RESULT,0
  jne lble9fix
  find addr, #90E8#
  cmp $RESULT,0
  jne lble8fix
  ret

lblend:

  msg "Script finished,Script by loveboom\[DFCG\]\[FCG\]\[US\],Thank for using my script!"
  ret

lbleros:

  msg "本脚本只能在 Winnnt 系统下运行!" //其实这里没有用的，因为没有 ntdll.dll 时脚本插件就会报错
  ret

lblabort:

  msg "脚本只能用于 v0.7x.:-(!"
  ret

lble9fix:

   mov addr,$RESULT
   mov jtoaddr,addr
   add addr,2
   mov tmpval,[addr]
   add tmpval,jtoaddr
   add tmpval,6
   log tmpval
   cmp tmpval,patchiataddr
   jb lblfixloop1
   cmp tmpval,patchiatsize
   ja lblfixloop1
   dec addr
   fill addr,1,0e8
   mov eip,addr
   cob
   sto
   mov addr,esp
   sub addr,8
   mov addr,[addr]
   inc addr
   mov addr,[addr]
   gn addr
   cmp $RESULT,0 
   je lblfixloop1
   cmp dllname,$RESULT_1
   je lble9sub1
   mov dllname,$RESULT_1
   add siataddr,4

lble9sub1:

  mov [siataddr],addr
  mov tmpval,jtoaddr
  fill tmpval,1,ff
  inc tmpval
  fill tmpval,1,25
  inc tmpval
  mov [tmpval],siataddr
  mov addr,tmpval
  add addr,4
  add siataddr,4
  jmp lblfixloop1

lble8fix:

   mov addr,$RESULT
   mov jtoaddr,addr
   add addr,2
   mov tmpval,[addr]
   add tmpval,jtoaddr
   add tmpval,6
   cmp tmpval,patchiataddr
   jb lblfixloop1
   cmp tmpval,patchiatsize
   ja lblfixloop1
   dec addr
   mov eip,addr
   cob
   sto
   mov addr,esp
   sub addr,8
   mov addr,[addr]
   inc addr
   mov addr,[addr]
   gn addr
   cmp $RESULT,0 
   je lblfixloop1
   cmp dllname,$RESULT_1
   je lble8sub1
   mov dllname,$RESULT_1
   add siataddr,4

lble8sub1:

  mov [siataddr],addr
  mov tmpval,jtoaddr
  fill tmpval,1,ff
  inc tmpval
  fill tmpval,1,15
  inc tmpval
  mov [tmpval],siataddr
  mov addr,tmpval
  add addr,4
  add siataddr,4
  jmp lblfixloop1
```
