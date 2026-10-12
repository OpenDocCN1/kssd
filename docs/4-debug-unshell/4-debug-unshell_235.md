
# 我也来炒现饭,在角落里发现了两个脚本

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-553.htm`](https://www.pediy.com/kssd/pediy07/pediy7-553.htm)

![](img/1a550ccb7245757838a4f08fc220b787.jpg)

```
/*
//////////////////////////////////////////////////
  Armadillo 3.x DLL Unpacking script v0.1
  Author:  loveboom
  Email : loveboom%163.com
  OS    : WinXP sp2,Ollydbg 1.1,OllyScript v0.92
  Date  : 2005-03-07
        Action: Auto fix IAT,find oep
  Config: Ignore all exceptions and ingnore exception: 'C000001E (INVALID LOCK SEQUENCE)'
  Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/
var addr    //addr
var gmaddr    //GetModuleHandleA's address
var fillvalue
var cbase
var csize
var count
var relocaddr
var relocsize

start:
  msgyn "Setting: Ignore all exceptions and ingnore exception: 'C000001E (INVALID LOCK SEQUENCE)',continue?"
  cmp $RESULT,1
  JE lblgetinfo1
  ret

lblgetinfo1:    //获取 code base
  ask "请输入.text 段的起始地址:"
  cmp $RESULT,0
  jne lblsetvalue1
  ret

lblsetvalue1:
  mov cbase,$RESULT

lblgetinfo2:      //获取 CODE SIZE
  ask "请输入.text 段的大小:"
  cmp $RESULT,0
  jne lblsetvalue2
  ret

lblsetvalue2:
  mov csize,$RESULT

LBL1:
  dbh
  mov count,0
  gpa "GetModuleHandleA","kernel32.dll"
  mov gmaddr,$RESULT
  bphws gmaddr,"x"

lbl2:
  esto

lblcmp:
  mov addr,esp
  add addr,8
  mov addr,[addr]
  mov addr,[addr]
  cmp addr,74726956
  jne lbl2
  inc count
  cmp count,2
  jne lbl2
  esto
  rtu

lbl3:
  bphwc gmaddr
  find eip,#0F84#
  cmp $RESULT,0
  je lblabort
  mov addr,$RESULT
  fill addr,1,90
  inc addr
  fill addr,1,e9
  rtr
  sto
  mov count,5

lblloop:
  find eip,#6A00FF35#
  go $RESULT
  findop eip,#7436#
  go $RESULT
  dec count
  cmp count,0
  je lblbreak
  jmp lblloop

lblbreak:
/*
  MOV EAX,DWORD PTR DS:[1080030]
  MOV EAX,DWORD PTR DS:[EAX]
  MOV DWORD PTR SS:[EBP-37D0],EAX          ; eax==重定位开始地址
  MOV EAX,DWORD PTR DS:[1080030]
  ADD EAX,4
  MOV DWORD PTR DS:[1080030],EAX
  MOV EAX,DWORD PTR DS:[1080030]
  MOV EAX,DWORD PTR DS:[EAX]
  MOV DWORD PTR SS:[EBP-3798],EAX          ; EAX==重定位大小
  MOV EAX,DWORD PTR DS:[1080030]
  ADD EAX,4
  MOV DWORD PTR DS:[1080030],EAX
  CMP DWORD PTR SS:[EBP-37D0],0            ; 判断重定位地址是否为空
  JE SHORT 01067CCD
  CMP DWORD PTR SS:[EBP-3798],0            ; 判断重定位大小是否为空
  JE SHORT 01067CCD
*/
  find eip,#A1????????8B008985????????A1????????83C004A3????????A1????????8B008985????????A1????????83C004#
  cmp $RESULT,0
  je lblabort
  go $RESULT
  sto
  sto
  mov relocaddr,eax
  sto
  find eip,#8985#
  go $RESULT
  mov relocsize,eax
  find eip,#74??83BD????????0074#
  cmp $RESULT,0
  je lblabort
  mov addr,$RESULT
  add addr,B
  find addr,#74#
  cmp $RESULT,0
  je lblabort
  fill $RESULT,1,EB
  bprm cbase,csize

lbl4:
  esto

lbl5:
  find eip,#558BEC#
  cmp $RESULT,0
  je lbl4
  cmp $RESULT,eip
  jne lbl4
  bpmc

lblend:
  cmt eip,"程序 oep"
  eval "这个 DLL 文件的重定位地址 VA 是: {relocaddr}.大小为: {relocsize}"
  msg $RESULT
  msg "Script by loveboom[DFCG][FCG][US],thank you for using my script!"
  ret

lblabort:
  msg "Error!Script aborted.Maybe target is not protect by arm 3.x or user aborted!"
  ret

//-------------------------------------------------------------
/*
//////////////////////////////////////////////////
  MSLRH v0.31A unpack script v0.1
  Author:  loveboom
  Email : loveboom%163.com
  OS    : WinXP sp2,Ollydbg 1.1,OllyScript v0.92
  Date  : 2005-03-07
        Action: Auto fix IAT,find oep
  Config: Ignore all exceptions and ingnore exception: 'C000001E (INVALID LOCK SEQUENCE)'
  Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/
var addr
var espval
var cbase
var csize
var peheader
var mbase

start:
  msgyn "Setting:Ignore all exceptions.continue?"
  cmp $RESULT,1
  je lbl1
  ret

lbl1:
  dbh
  mov espval,esp    //Get esp value
  sub espval,4
  gmi eip,MODULEBASE    //Get code section information
  mov mbase,$RESULT
  mov peheader,mbase
  add peheader,3c
  mov addr,[peheader]
  add addr,mbase
  mov peheader,addr    //Get pe header
  add peheader,100    //Get section size
  mov csize,[peheader]
  add peheader,4    //Get section VirutalAddress
  mov cbase,[peheader]
  add cbase,mbase

lbl2:
  gpa "OutputDebugStringA","kernel32.dll"
  cmp $RESULT,0
  je lbl3
  mov addr,$RESULT
  asm addr,"xor eax,eax"  //Patch api function
  add addr,2
  asm addr, "ret 4"

lbl3:
  gpa "CreateFileA","kernel32.dll"
  bp $RESULT
  esto
  bc $RESULT
  rtu

lbl4:      //clear anti-ImportREC
  mov addr,eax
  exec
    push {addr}
    Call CloseHandle
  ende

lbl5:
  gpa "ZwQueryInformationProcess","ntdll.dll"    //Clear Anti-Ring3 debug
  cmp $RESULT,0
  je lbl6
  bp $RESULT
  esto
  bc $RESULT
  rtu
  sto
  mov eax,0

lbl6:
  bprm cbase,csize
  esto
  bpmc
  bphws espval,"r"
  esto
  bphwc espval
  sto

lblend:
  dbs
  cmt eip,"OEP"
  msg "Script by loveboom[DFCG][FCG][US],thank you for using my script!"
  ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
