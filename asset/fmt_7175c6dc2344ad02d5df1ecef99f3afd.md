
### 汇编部分 (MASM32)

```masm
include kernel32.inc
includelib kernel32.lib
include gdi32.inc
includelib gdi32.lib
include comdlg32.inc
includelib comdlg32.lib
include masm32.inc
includelib masm32.lib

;Equ 等值定义
ICO_MAIN equ 1000H
DLG_MAIN equ 1
EditName equ 10
EditSN equ 11

;****************************************************************************************

.data?
hInstance dd ?
szSN db 255 dup (?)
szName db 255 dup (?)
regStr db 255 dup (?)
theString db 8 dup (?)
lenString db 8 dup (?)

.data
szErr db '请输入用户名！',0
szCaption db '错误！',0

.const
Num01 dq 64.00000000000000
Num02 dq 90.00000000000000
Num03 dq 97.00000000000000
Num04 dq 123.0000000000000

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code
;****************************************************************************************

_testStr proc uses ebx edi esi ebp, @tempStr
local @tempStr00
local @temp01, @temp02, @temp03, @temp04

invoke atodw, @tempStr
mov dword ptr @tempStr00, eax
xor eax, eax
fild @tempStr00
fcomp Num01
fnstsw ax
test ah,41h
jz @F
mov dword ptr @temp01, 01
jmp a10
@@:
mov dword ptr @temp01, 00
a10:

xor eax, eax
fild @tempStr00
fcomp Num02
fnstsw ax
test ah, 1
jnz @F
mov dword ptr @temp02, 01
jmp a20
@@:
mov dword ptr @temp02, 00
a20:

xor eax, eax
fild @tempStr00
fcomp Num03
fnstsw ax
test ah, 41
jz @F
mov dword ptr @temp03, 01
jmp a30
@@:
mov dword ptr @temp03, 00
a30:

xor eax, eax
fild @tempStr00
fcomp Num04
fnstsw ax
test ah, 01
jnz @F
mov dword ptr @temp04, 01
jmp a40
@@:
mov dword ptr @temp04, 00
a40:

mov ecx, dword ptr @temp01
or ecx, @temp02
neg ecx
sbb ecx, ecx
neg ecx
mov edx, dword ptr @temp03
or edx, @temp04
neg edx
sbb edx, edx
neg edx
and ecx, edx
ret
_testStr endp

;****************************************************************************************
;****************************************************************************************

_regCodCalc proc
local @temp[8], @temp00[8], @temp01, @temp02
pushad
invoke lstrlen, addr szName
mov edi, eax
sub esi, esi
xor ebx, ebx

;计算中间字串变量
.while esi < edi
sub edx, edx
mov dl, byte ptr [szName+esi]
xor dx, 5
mov dword ptr @temp01, edx
invoke dwtoa, @temp01, addr @temp02
;MASM32 库函数
mov eax, dword ptr @temp02
mov dword ptr [regStr+ebx], eax
invoke lstrlen, addr regStr
mov ebx, eax
inc esi
.break .if esi > edi
.continue
.endw

;计算注册码
invoke lstrlen, addr regStr
inc eax
mov ecx, 2
cdq
idiv ecx
mov dword ptr lenString, eax
mov esi, eax
xor ebx, ebx
xor edi, edi
.while edi < esi
invoke RtlZeroMemory, addr @temp, 8
invoke RtlZeroMemory, addr @temp00, 8
movsx eax, byte ptr [regStr+2*edi]
movsx edx, byte ptr [regStr+2*edi+1]
mov byte ptr @temp, al
mov byte ptr @temp00, dl
invoke lstrcat, addr @temp, addr @temp00
invoke _testStr, addr @temp
;子程序，带一个参数，返回在 ecx 中
test ecx, ecx
jnz @F
invoke atodw, addr @temp
;MASM32 库函数
jmp a10
@@:
mov eax, dword ptr @temp
a10:
mov dword ptr [szSN+ebx], eax
invoke lstrlen, addr szSN
mov ebx, eax
inc edi
.break .if edi > esi
.continue
.endw

popad
ret
_regCodCalc endp

;****************************************************************************************

_ProcDlgMain proc uses ebx edi esi ebp, hWnd,wMsg,wParam,lParam
mov eax,wMsg
.if eax==WM_CLOSE
invoke EndDialog,hWnd,NULL
.elseif eax==WM_COMMAND
mov eax,wParam
.if eax == IDOK
invoke RtlZeroMemory,addr szSN,255
invoke GetDlgItemText,hWnd,EditName,offset szName,255
.if eax != NULL
invoke _regCodCalc
invoke SetDlgItemText, hWnd, EditSN, offset szSN
mov eax,FALSE
ret
.else
invoke MessageBox,NULL,offset szErr,offset szCaption,MB_OK
mov eax,FALSE
ret
.endif
.elseif eax == IDCANCEL
invoke EndDialog,hWnd,NULL
.endif
.else
mov eax,FALSE
ret
.endif
mov eax,TRUE
ret
_ProcDlgMain endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:
invoke GetModuleHandle,NULL
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlgMain,NULL
invoke ExitProcess,NULL

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

end start
```

### 资源文件

`KeyGen.rc`

```rc
#include
#define ICO_MAIN 0x1000
#define DLG_MAIN 1
#define EDITName 10
#define EDITSN 11

ICO_MAIN ICON "01.ico"

DLG_MAIN DIALOG 100,150,250,60
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME
CAPTION "豪杰超级解霸 3K～英雄版 注册机"
FONT 9,"宋体"
{
    CONTROL "Name:",-1,"Static",SS_LEFT,10,13,40,17
    CONTROL "SN:",-2,"Static",SS_CENTER,10,40,20,17
    CONTROL "",10,"Edit",ES_LEFT,30,13,150,10
    CONTROL "",11,"Edit",ES_LEFT,30,40,150,10
    DEFPUSHBUTTON "GENERATE",IDOK,200,11,40,15
    PUSHBUTTON "EXIT",IDCANCEL,200,36,41,14
}
```

### 构建脚本

`makefile`

```makefile
NAME = KeyGen
OBJS = $(NAME).obj #需要的目标文件
RES = $(NAME).res #需要的资源文件
LINK_FLAG = /subsystem:windows #连接选项
ML_FLAG = /c /coff #编译选项

$(NAME).exe: $(OBJS) $(RES)
    Link $(LINK_FLAG) $(OBJS) $(RES)

.asm.obj:
    ml $(ML_FLAG) $<

.rc.res:
    rc $<

clean:
    del *.obj
    del *.res
```
