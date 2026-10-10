

# 结构化异常处理\_ 手动实现

> 原文：[`www.pediy.com/kssd/pediy10/78001.html`](https://www.pediy.com/kssd/pediy10/78001.html)

结构化异常处理  
  
由于 windows 系统可以在运行态安装一个异常处理回调函数，当应用程序的异常发生时候，回调过程就被系统触发，由应用程序来处理这些异常，实现对异常的修补  
  
  
但是这个异常回调是怎么实现的呢？  
  

代码:

```
ASSUME FS :NOTHING


ifndef SehStruct
  SehStruct EQU 1
  .DATA
  SEH sSEH <>
  
endif

.CODE

MOV SEH.SaveEip , ContinueAddr

MOV SEH.OrgEbp, EBP

PUSH OFFSET SehHandler

PUSH FS:\[0\]

MOV SEH.OrgEsp, ESP
MOV FS:\[0\],ESP

;去除回调

POP FS:\[0\]

ADD ESP,4

```

  
代码:

```
.386

.model flat,stdcall
option casemap:none

include \\masm32\\include\\user32.inc
includelib \\masm32\\lib\\user32.lib

include \\masm32\\include\\windows.inc

;结构定义

sSEH STRUCT
  OrgEsp  DD ?
  OrgEbp  DD ?
  SaveEip DD ?
sSEH ends

;宏定义

InstSehFrame MACRO ContinueAddr
  ASSUME FS : NOTHING
  
  IFNDEF SehStruct
    SehStruct EQU 1
    .DATA
    SEH sSEH <>
  ENDIF
.CODE
mov SEH.SaveEip, ContinueAddr
mov SEH.OrgEbp,EBP

push offset SehHandler

push FS:\[0\]
mov SEH.OrgEsp,ESP
mov FS:\[0\],ESP
endm

KillSehFrame MACRO

pop  fs:\[0\]
add esp,4

endm


;过程定义

.CODE

SehHandler PROC c pExcept:DWORD ,pFrame:DWORD ,pContext:DWORD,PDispatch:DWORD
  
  
  mov eax,pContext
  ASSUME eax:ptr context
  
  push seh.SaveEip
  POP \[eax\].regEip
  PUSH SEH.OrgEsp
  pop \[eax\],regEsp
  push SEH.OrgEbp
  pop \[eax\],regEbp
  
  mov eax,ExceptionContinueExecution
  

  ret

SehHandler endp

.DATA
szGood DB "SEH succeed",0
szCap DB "OK",0

.CODE
main:
InstSehFrame <OFFSET SavePlace1>

;产生异常的代码 1
xor eax,eax
xchg DWORD ptr \[eax\],eax

SavePlace1:

KillSehFrame
InstSehFrame <OFFSET SavePlace2>

;产生异常 2
xor ebx,ebx
xor edx,edx
mov eax,2
div ebx

SavePlace2:
KillSehFrame

invoke MessageBox,0,offset szGood,offset szCap,MB\_OK
RET

end main

InstSehFrame 安装异常回调函数
KillSehFrame 去除自定义的异常处理回调函数

```

我们知道在 windows 下，如果 windows 执行了 int3 的指令的话，就会触发一个断点异常。  
在上面我们实现了异常的自我处理，然后接下来如何处理呢？  
  

代码:

```
main:

  InstSehFrame offset Error\_Handler
  
  invoke GetModuleHandle,NULL
  
  mov hInstance,eax
  invoke DialogBoxParam,hInstance,offset szDlgName,0,WndProc,0
  
  KillSehFrame
  invoke ExitProcess,0
  
WndProc proc hWnd:HWND ,uMsg:UINT ,wParam:WPARAM,lParam:LPARAM

.if uMsg == WM\_CLOSE
  invoke EndDialog,hWnd,0
  
.elseif uMsg == WM\_INITDIALOG
  mov eax,hWnd
  mov \[mbp.hwndOwner\],eax
  invoke LoadIcon,hInstance,IDI\_LC
  invoke SendMessage,hWnd,WM\_SETICON,ICON\_SMALL,eax
  ;GetModuleHandle 获得 api 入口地址
  invoke GetModuleHandle,addr szLibUser
  invoke GetProcAddress,eax,addr szProcMsgBox
  mov \[dwAddress\],eax
  
  ;保存 hook 前信息
  invoke lstrcpy,addr szText,addr szMsgAbout
  
.elseif uMsg == WM\_COMMAND
  mov eax,wParam
  mov edx,eax
  shr edx,16
  movzx eax,ax
  .if edx == BN\_CLICKED
    .if eax == IDC\_BUTTON\_EXIT || eax == IDCANCEL
      invoke EndDialog,hWnd,NULL
    .elseif eax ==IDC\_BUTTON\_ABOUT || eax ==IDOK
      invoke MessageBox,hWnd,offset szMsgAbout,offset szCaption,MB\_OK
    .elseif eax == IDC\_CHECKBUTTON\_HOOK
      ;memory read write exectuable
      invoke VirtualProtect,\[dwAddress\],1,PAGE\_EXECUTE\_READWRITE,addr dwOldProtect
      invoke IsDlgButtonChecked ,hWnd,IDC\_CHECKBUTTON\_HOOK
      mov edx,\[dwAddress\]
      test eax,eax
      .if zero?
        ;uninstall hook
        mov cl,\[bOldByte\]
        ;bOldByte = API
        invoke lstrcpy,addr szMsgAbout,addr szText
        ;
      .else
        ;re install hook
        mov cl,byte ptr \[edx\]
        mov byte ptr \[edx\] ,0CCh
        mov \[bOldByte\],cl
        invoke lstrcat,addr szMsgAbout,addr szMyText
      .endif 
    .endif
  .endif
.else
  mov eax,FALSE
  ret
  
.endif

mov eax,TRUE

ret

WndProc endp


;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;处理异常             \*
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

Error\_Handler proc uses ecx lpExceptRecord:DWORD,lpFrame:DWORD,lpContext:DWORD,lpDispatch:DWORD
  
  ;处储并改变 sethook 函数返回值
  
  mov eax ,\[lpContext\]
  mov eax,\[eax\]\[CONTEXT.regEsp\]
  mov ecx,\[eax\]
  
  mov \[eax\],offset SetHook
  mov \[dwRetAddr\],ecx
  
  mov eax,\[dwAddress\]
  mov cl,\[bOldByte\]
  mov byte ptr \[eax\],cl
  
  ;continue next
  mov eax,ExceptionContinueExecution
  
  ret

Error\_Handler endp


;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
;set api hook
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*


SetHook proc uses ecx
  
  mov eax,\[dwAddress\]
  mov cl,\[eax\]
  mov byte ptr \[eax\],0CCh
  ;int3
  mov \[bOldByte\],cl
  jmp \[dwRetAddr\]
  ;jump the address after api hook
  
  ;ret

SetHook endp

end main

```