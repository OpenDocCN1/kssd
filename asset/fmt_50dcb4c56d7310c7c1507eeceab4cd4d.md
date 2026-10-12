
# 005A7C2E 调用 `vkckmini.00404E5C`

```
005A7C2E      call vkckmini.00404E5C  
005A7C33      push dword ptr ss:[ebp-20]  
005A7C36      mov eax,dword ptr ss:[ebp-8]  
005A7C39      mov edx,5  
005A7C3E      call vkckmini.00404FF4  
005A7C43      inc ebx  
005A7C44      dec esi  
005A7C45      jnz vkckmini.005A7AA7  
```

附上注册机源码，以供参考。不当之处，请各位指正！

---

# 汇编注册机

## 编译命令

```
;-------------------------------------------------------------------------------
;ml /c /coff keygen,asm
;rc keygen.rc
;link /subsystem:windows keygen.obj keygen.res
;-------------------------------------------------------------------------------
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;>>                keygen.asm                   >>
;>>            Coded by qfejj                   >>
;>>           Email ：qfejj@163.com             >>
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

## 源码

```
.586
.model flat,stdcall
option casemap:none
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include    windows.inc
include    kernel32.inc
include    user32.inc
include    gdi32.inc
include    comctl32.inc
includelib kernel32.lib
includelib user32.lib
includelib gdi32.lib
includelib comctl32.lib
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
MANIFEST                equ 24
IDD_MAINDLG             equ 1000
IDOK                    equ 1
IDC_EDT1                equ 1004
IDC_EDT2                equ 1005
IDC_EDT3                equ 1006
IDC_GRP1                equ 1007
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Windowproc   proto :HWND,:UINT,:WPARAM,:LPARAM
GetRegistKey proto
Color macro Red,Green,Blue                              ;宏,方便后面颜色设置
    xor eax,eax
    mov ah,Blue
    shl eax,8
    mov ah,Green
    mov al,Red   
endm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.data
MsgBoxText     db '用户名怎么可以为空呢?',0
MsgBoxCaption db 'Warning',0
String         db '======Keygen For VKSOFT======      ',0dh,0ah
               db '   This Keygen Is Designed By qfejj',0dh,0ah
               db '   Contact Email: qfejj@163.com     ',0dh,0ah  
               db '=============================      ',0dh,0ah,0   
SerialString   db 30h,31h,32h,33h,34h,35h,36h,37h,38h,39h
String1        db 31h,42h,32h,44h,33h,46h,34h,48h,35h,4Ah,36h,4Ch,37h,4Eh,38h,50h
String2        db 39h,52h,38h,54h,37h,56h,36h,58h,35h,5Ah,34h,31h,57h,33h,46h,35h
String3        db 59h,37h,52h,39h,51h,31h,43h,32h,45h,33h,47h,34h,49h,35h,4Bh,36h
String4        db 4Dh,37h,42h,38h,51h,37h,53h,59h,55h,35h,57h,38h,59h,39h
.data?
hInstance       HINSTANCE       ?
hWnd            HINSTANCE       ?
hDc    DWORD    ?
hMDc1   DWORD    ?
hMDc2   DWORD    ?
hBrush  DWORD    ?
TextOutX  DWORD    ?
TextOutY  DWORD    ?
TextOutCY  DWORD    ?
LogFont   LOGFONT   <?>
;-------------------------------------------------
NameBuffer     db 20 dup(?)
SerialBuffer   db 16 dup(?)
KeyBuffer      db 24 dup(?)
ConvertString  db 64 dup(?)
KeyOut         db 30 dup(?) 
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code
start:
  invoke InitCommonControls
  invoke GetModuleHandle,0
  mov hInstance,eax
  invoke DialogBoxParam,hInstance,IDD_MAINDLG,0,offset Windowproc,0
  invoke ExitProcess,0
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
Windowproc PROC hWin:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM
LOCAL  hBitMapMDc:HDC
LOCAL  ps:PAINTSTRUCT
LOCAL  rect:RECT
LOCAL  TextSizeCX:dword
LOCAL  TextSizeCY:dword
  mov eax,uMsg
.if  eax == WM_PAINT
  invoke  BeginPaint,hWin,addr ps
  mov  hDc,eax
  invoke  BitBlt,hDc,0h,0h,120h,40h,hMDc1,0,0,0cc0020h 
  invoke  EndPaint,hWin,addr ps
.elseif  eax == WM_INITDIALOG
  invoke  SetTimer,hWin,200h,45h,NULL
  invoke  GetDC,hWin
  mov  hDc,eax
  invoke  CreateCompatibleDC,hDc                   
  mov  hMDc1,eax
  invoke  CreateCompatibleBitmap,hDc,120h,40h          
  mov  hBitMapMDc,eax
  invoke  SelectObject,hMDc1,hBitMapMDc           
  Color   0,0,0                     
  invoke  CreateSolidBrush,eax
  mov  hBrush,eax
  mov  rect.top,0h               
  mov  rect.left,0h
  mov  rect.right,120h
  mov  rect.bottom,100h
  invoke  FillRect,hMDc1,addr rect,hBrush           
  invoke  DeleteObject,hBrush
  invoke  CreateCompatibleDC,hDc                   
  mov  hMDc2,eax
  Color   0,255,0                    
  invoke  SetTextColor,hMDc2,eax
  invoke  SetBkMode,hMDc2,TRANSPARENT             
  invoke  CreateFontIndirect,addr LogFont      
  invoke  SelectObject,hMDc2,eax
  invoke  CreateCompatibleBitmap,hDc,120h,80h          
  invoke  SelectObject,hMDc2,eax
  Color   0,0,0                    
  invoke  CreateSolidBrush,eax
  mov  hBrush,eax
  mov  rect.top,0h                  
  mov  rect.left,0h
  mov  rect.right,120h
  mov  rect.bottom,100h
  invoke  FillRect,hMDc2,addr rect,hBrush         
  invoke  DeleteObject,hBrush
  mov  TextOutX,25h
  mov  TextOutY,30h
  lea  esi,String
  mov  al,byte ptr [esi]
  .while  al
    .if  al == 0dh
      add TextOutY,10h            
      mov TextOutX,25h            
      add esi,2h
    .else
      invoke  GetTextExtentPoint32,hMDc2,esi,1,addr TextSizeCX  
      invoke  TextOut,hMDc2,TextOutX,TextOutY,esi,1h
      mov eax,TextSizeCX
      add TextOutX,eax
       inc esi
    .endif
    mov  al,byte ptr [esi]
  .endw
  invoke  ReleaseDC,hWin,hDc
.elseif  eax == WM_TIMER
  invoke  GetDC,hWin
  mov  hDc,eax
  invoke  BitBlt,hMDc1,0h,8h,120h,40h,hMDc2,0,TextOutCY,0cc0020h
  inc  TextOutCY
  .if  TextOutCY == 70h
    mov  TextOutCY,0
  .endif
  invoke  BitBlt,hDc,0h,0h,120h,38h,hMDc1,0,0,0cc0020h     
  invoke  ReleaseDC,hWin,hDc
.elseif eax == WM_CLOSE
        invoke EndDialog,hWin,0
.elseif eax == WM_COMMAND
  mov eax,wParam
  .if eax ==IDOK
      push hWin
      pop hWnd
      invoke GetDlgItemText,hWin,IDC_EDT1,addr NameBuffer,20
      .if eax==0
          invoke MessageBox,hWin,addr MsgBoxText,addr MsgBoxCaption,MB_OK
      .else
          invoke GetRegistKey                          ;注册码计算函数
          invoke SetDlgItemText,hWin,IDC_EDT2,addr SerialBuffer
          invoke SetDlgItemText,hWin,IDC_EDT3,addr KeyOut
      .endif
  .endif
.else
  mov eax,0
  ret
.endif
mov eax,1
ret
Windowproc endp
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
GetRegistKey proc
LOCAL CmpTime:dword
  invoke lstrcat,offset ConvertString,offset String1
  invoke lstrcat,offset ConvertString,offset String2
  invoke lstrcat,offset ConvertString,offset String3
  invoke lstrcat,offset ConvertString,offset String4
;--------------
; 计算序列号
;--------------
  xor edi,edi
GetCode1:
  mov dl,byte ptr[NameBuffer+edi]
  inc edx
  add edx,edx
  and edx,7
  mov al,byte ptr[SerialString+edx]      ;取单个序列号
  mov byte ptr[SerialBuffer+edi],al     
  inc edi
  cmp edi,8
  jl GetCode1
  xor edi,edi
GetCode2:
  mov dl,byte ptr[NameBuffer+edi]
  add edx,7h
  and edx,7
  mov al,byte ptr[SerialString+edx]      ;取单个序列号
  mov byte ptr[SerialBuffer+7+edi],al     
  inc edi
  cmp edi,8
  jl GetCode2
;---------------
; 计算注册码
;---------------
  xor edi,edi
  mov CmpTime,0
@1:
  mov edx,CmpTime
  mov eax,CmpTime
  add eax,3
  mov eax,edx
  inc eax
```
