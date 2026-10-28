
# 大富翁6免CD破解

最近想试一下光驱的不定破解,所以找同学借了一张光盘版的游戏,顺便自己爽爽,哪知是小李飞刀,两张碟,Cao,要1.8G,我的硬盘实在太小,本以为最后的C盘能够搞定它(1.2G),哪知....哎....后来换了一个大富翁6,以前看过大众软件它总是排在前面!来试一试!

OK!装好游戏,运行程序,进入游戏,刚刚Loading完,死机!郁闷....
看来机器太差了,赛扬700,128M,重起后,拿出光盘,运行程序,提示要你插入光盘,`W32Dasm`打开程序,有一个及时模式,一个回合模式!我们打开及时模式,我们查找`GetDriveTypeA`,地方如下:

* Reference To: `KERNEL32.GetDriveTypeA`, Ord:0104h

                                  |

:0047C830 8B3590504900            mov esi, dword ptr [00495090]

:0047C836 B343                    mov bl, 43

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0047C87D(C)

|

:0047C838 0FBEC3                  movsx eax, bl

:0047C83B 50                      push eax

* Possible StringData Ref from Data Obj ->"%c:\

附大富翁6免CD补丁：

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;大富翁6免CD补丁,Made By dengkeng ,Link By MASM32 V8
;E-Mail:shellc0de@sohu.com
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

```asm
    .486                       ; create 32 bit code
    .model flat, stdcall       ; 32 bit memory model
    option casemap :none       ; case sensitive
    include masm32includewindows.inc
    include masm32includemasm32.inc
    include masm32includegdi32.inc
    include masm32includeuser32.inc
    include masm32includekernel32.inc
    include masm32includeComctl32.inc
    include masm32includecomdlg32.inc
    include masm32includeshell32.inc
    include masm32includeoleaut32.inc
    includelib masm32libmasm32.lib
    includelib masm32libgdi32.lib
    includelib masm32libuser32.lib
    includelib masm32libkernel32.lib
    includelib masm32libComctl32.lib
    includelib masm32libcomdlg32.lib
    includelib masm32libshell32.lib
    includelib masm32liboleaut32.lib

.data?

hInstance  dd  ?
hFile    dd  ?
szFileName  db  MAX_PATH dup (?)
szName    db  MAX_PATH dup (?)
szBuffer  db  1024 dup (?)
BakPath    db  128  dup (?)

lFileSize  dd  ?
lFileOffset  dd  ?

.const

szCaption  db  'Richman6',0
szFilter  db  'exe(*.exe)',0,'*.exe',0,0
szTitle    db  '打开大富翁6',0
szErrOpen  db  '不能打开文件,请在试一次!',13,10,0
szErrSize  db  '文件可能已经被修改,请重新拷贝一份原文件!',13,10,0
szErrPointer  db  '文件指针设置错误!',13,10,0
szErrWrite  db  '文件不能写入!',13,10,0
szPatchOK  db  '文件成功修补!',13,10
    db  '欢迎使用大富翁6及时模式补丁!',13,10
    db  'Made By dengkeng!',13,10,0
szOpenSccessful db  '文件成功打开,接着将为您修复大富翁6!',13,10,0
HeModel    db  '您所打开的是大富翁6的回合模式!',0
JsModel    db  '您所打开的是大富翁6的及时模式!',0
HeHe    db  'Cao,你总该选点什么吧?',0
BakFile    db  '现在为您备份文件!',0
BakName    db  '.bak',0

lChanges  dd  2h
BytesToWrite  db  74h,26h
lByteWriten  dd  0

szGameName  db  'rich6_t.exe'

.code

_OpenFile  proc
    local  @stOF:OPENFILENAME
    invoke  RtlZeroMemory,addr @stOF,sizeof @stOF
    mov  @stOF.lStructSize,sizeof @stOF
    push  hInstance
    pop  @stOF.hInstance
    mov  @stOF.lpstrTitle,offset szTitle
    mov  @stOF.lpstrFilter,offset szFilter
    mov  @stOF.lpstrFile,offset szFileName ;全路径的文件名
    mov  @stOF.lpstrFileTitle,offset szName ;不包含路径名
    mov  @stOF.nMaxFile,MAX_PATH
    mov  @stOF.Flags,OFN_FILEMUSTEXIST or OFN_PATHMUSTEXIST
    invoke  GetOpenFileName,addr @stOF
    .if  eax
      invoke  MessageBox,hInstance,addr szOpenSccessful,addr szCaption,MB_OK
    .endif
    ret
_OpenFile  endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:
    invoke  GetModuleHandle,NULL
    mov  hInstance,eax
    invoke  _OpenFile
    cmp  eax,FALSE
    mov  ebx,offset HeHe
    jz  openerr
    invoke  lstrcpy,addr BakPath,addr szFileName
    invoke  lstrcat,addr BakPath,addr BakName
    invoke  MessageBox,NULL,addr BakFile,offset szTitle,MB_OK
    invoke  CopyFile,addr szFileName,addr BakPath,FALSE ;备份文件
    invoke  lstrcmp,addr szName,addr szGameName
    cmp  eax,0 ;判断是否为回合模式
    jnz  @1
    invoke  MessageBox,hInstance,addr HeModel,addr szTitle,MB_OK
    mov  edx,802A0h ;对文件的大小及其偏移进行修改,使其成为回合模式
    mov  lFileOffset,edx
    mov  edx,704512
    mov  lFileSize,edx
    jmp  @2
  @1:
    invoke  MessageBox,hInstance,addr JsModel,addr szTitle,MB_OK
    mov  edx,7C850h ;对文件的大小及其偏移进行修改,使其成为及时模式
    mov  lFileOffset,edx
    mov  edx,688128
    mov  lFileSize,edx
  @2:
    invoke  CreateFile,addr szFileName,GENERIC_READ+GENERIC_WRITE,0,0,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,0
    cmp  eax,-1
    mov  ebx,offset szErrOpen
    jz  openerr
    mov  hFile,eax
    invoke  GetFileSize,hFile,NULL
    cmp  eax,lFileSize
    mov  ebx,offset szErrSize
    jnz  err
    invoke  SetFilePointer,hFile,lFileOffset,NULL,FILE_BEGIN
    cmp  eax,-1
    mov  ebx,offset szErrPointer
    jz  err
    invoke  WriteFile,hFile,offset BytesToWrite,lChanges,offset lByteWriten,NULL
    test  eax,eax
    mov  ebx,offset szErrWrite
    jz  err
    mov  ebx,offset szPatchOK
  err:
    invoke  CloseHandle,hFile
  openerr:
    invoke  MessageBox,NULL,ebx,offset szTitle,MB_OK
    invoke  ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    end  start
```

大侠见笑!!!!!!
