
所以有`注册码[1]`=37,`注册码[2]`=38,算法如下:

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

(1)注册码[1]+ecx=a

   ;ecx=0,结果a=注册码[1]--------------------------------------------------------37

(2)a-30=b

   ;b=注册码[1]-30---------------------------------------------------------------7

(3)b+b=c

   ;c=注册码[1]-30+注册码[1]-30--------------------------------------------------E(十进制14)

(4)c+c*4=d

   ;d=注册码[1]-30+注册码[1]-30+{c=注册码[1]-30+注册码[1]-30}*4------------------46

(5)注册码[2]+d=e

   ;e=注册码[2]+注册码[1]-30+注册码[1]-30+{注册码[1]-30+注册码[1]-30}*4----------7E(十进制126)

(6)e-30=f

   ;f=注册码[2]+注册码[1]-30+注册码[1]-30+{注册码[1]-30+注册码[1]-30}*4-30-------4E

(7)mov eax,f

   ;保存注册码处理结果到eax，前面已经分析过，用户名处理后的结果保存在esi

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

综上，得到`eax=f=注册码[2]+注册码[1]-30+注册码[1]-30+{注册码[1]-30+注册码[1]-30}*4-30`

整理一下，即

```
=============================

eax=注册码[2]+注册码[1]*A-210

=============================
```

注意是16进制的，不要弄错了;到此，所有的算法分析完毕，下面说说注册机的编写。

既然知道了算法，下一步当然是写注册机了。通过上面的分析，我们可以按照软件的变换方法处理用户名和第1、3段注册码，得到处理结果(软件使用`esi`保存)，再由处理结果(`esi`)反推出第2、4注册码。也可以先得到2、4段注册码处理结果，再以此计算用户名，不过这样好像不妥，因为不容易得到自己喜欢的用户名。既然这样，我还是用第一种方法吧。那么，如何依靠`esi`计算出注册码呢?我们来看，由于注册码必须是数字，所以正确的注册码必须满足下面的关系：

```text
esi+210-39 < 注册码[2]+注册码[1]*A < esi+210-30        ;<--39和30分别是9和0的ASCII值
```

即 `esi+1D7 < 注册码[2]+注册码[1]*A < esi+1E0`

既然可以确定范围,那反推注册码也就容易多了,这里只有两个数，我就用穷举来实现了。另外，第一、三、四段注册码和用户名无关，只要满足一定关系就可以，我就拿用户名换算了一下作为第一，三段的注册码了，最后以此求解第四段注册码。如果哪位哥们有更好的求解方法，还望赐教。

---

## 注册机源码

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>> Graphic Workshop v2.0a(pach98) 注册机 >>>

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.586

.model flat,stdcall

option casemap:none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include windows.inc

include kernel32.inc

include user32.inc

include comctl32.inc

include gdi32.inc

includelib kernel32.lib

includelib user32.lib

includelib comctl32.lib

includelib gdi32.lib

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

IDDIALOG                equ 10

IDOK                    equ 1

IDABOUT                 equ 2

IDREGNAME               equ 4

IDREGCODE               equ 3

IDC_STC1                equ 5

IDC_STC2                equ 6

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

WinDialog proto :HWND,:UINT,:WPARAM,:LPARAM

CalCode   proto

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

RGB macro red,green,blue

        xor eax,eax

        mov ah,blue

        shl eax,8

        mov ah,green

        mov al,red

endm

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.data

MsgBoxText    db 'Graphic Workshop v2.0a注册机',0dh

              db '              编写:qfejj',0dh

              db '       Email:qfejj@163.com',0

MsgBoxCaption db 'About',0

MsgBoxTextErr db '用户名不能为空',0

MsgBoxWarning db '错误',0

String        db 'Keygen For Everyone ',0

FontName      db 'script',0

xorBuffer     db 1 dup (80h,40h,20h,10h,08h,04h,02h,01h)

CodeBase      db 1 dup (30h,31h,32h,33h,34h,35h,36h,37h,38h,39h)

.data?

hInstance   HINSTANCE ?

hButton     HINSTANCE ?

hNameEdit   HINSTANCE ?

NameBuffer  db 33 dup(?)

CodeBuffer  db 20 dup(?)

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code

start:

  invoke InitCommonControls

  invoke GetModuleHandle,NULL

  mov hInstance,eax

  invoke DialogBoxParam,hInstance,IDDIALOG,NULL,offset WinDialog,0

  invoke ExitProcess,0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

WinDialog proc hWin,uMsg,wParam,lParam

LOCAL pc:PAINTSTRUCT

LOCAL hdc:HDC

LOCAL hfont:HFONT

  mov eax,uMsg

  .if eax==WM_CLOSE

      invoke EndDialog,hWin,NULL

  .elseif eax==WM_PAINT

      invoke BeginPaint,hWin,addr pc

      mov hdc,eax

      invoke CreateFont,50,14,0,0,900,0,0,0,GB2312_CHARSET,\

                        OUT_OUTLINE_PRECIS,CLIP_DEFAULT_PRECIS,\

                        DEFAULT_QUALITY,FF_MODERN,addr FontName

      invoke SelectObject, hdc, eax

      mov    hfont,eax

      RGB    0,255,0

      invoke SetTextColor,hdc,eax

      RGB    165,61,107

      invoke SetBkColor,hdc,eax

      invoke TextOut,hdc,0,0,addr String,sizeof String

      invoke SelectObject,hdc, hfont

      invoke EndPaint,hWin,addr pc

  .elseif eax==WM_INITDIALOG

      invoke GetDlgItem,hWin,IDREGNAME

      mov hNameEdit,eax

      invoke GetDlgItem,hWin,IDOK

      mov hButton,eax

      invoke GetWindowTextLength,hNameEdit

      invoke EnableWindow,hButton,FALSE

  .elseif eax==WM_COMMAND

      invoke GetWindowTextLength,hNameEdit

      .if eax<6

         invoke EnableWindow,hButton,0

      .else

               invoke EnableWindow,hButton,1

      .endif

      mov eax,wParam

      .if eax==IDOK

          invoke GetDlgItemText,hWin,IDREGNAME,addr NameBuffer,sizeof NameBuffer

          .if eax==0

              invoke MessageBox,hWin,addr MsgBoxTextErr,addr MsgBoxWarning,MB_ICONWARNING

          .else

              invoke CalCode     ;计算注册码的函数

              invoke SetDlgItemText,hWin,IDREGCODE,addr CodeBuffer

              xor edx,edx

              mov byte ptr[CodeBuffer+5],dl

              mov byte ptr[CodeBuffer+8],dl

              mov byte ptr[CodeBuffer+14],dl

          .endif

      .elseif eax==IDABOUT

          invoke MessageBox,hWin,addr MsgBoxText,addr MsgBoxCaption,MB_OK

      .endif

  .else

        mov eax,FALSE

        ret

  .endif

  mov eax,TRUE

  ret

WinDialog endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CalCode proc

LOCAL   ResultName:dword

LOCAL   ResultCode:word

LOCAL   TempResult:dword

LOCAL   MulResult:byte

;--------------

;用户名处理过程

;--------------

  xor edi,edi

  mov dword ptr[TempResult],0

  xor eax,eax

ConvertName:

  mov al,byte ptr[NameBuffer+edi]

  push eax

  mov dl,al

  pop ecx

  mov edx,edi

  and edx,7

  xor ecx,ecx

  mov cl,byte ptr[xorBuffer+edx]

  xor eax,ecx

  add dword ptr[TempResult], eax

  inc edi

  cmp byte ptr ds:[NameBuffer+edi],0

  jnz ConvertName

  mov eax,dword ptr[TempResult]

  mov ecx,64h

  cdq

  idiv ecx

  mov dword ptr[ResultName],edx       ;保存用户名处理结果

;-----------------------------------

;获得第2 段注册码,我用穷举来实现

;-----------------------------------

  xor edi,edi

@1:

  mov al,byte ptr[CodeBase+edi]

  mov dl,0Ah

  mul dx

  mov byte ptr[MulResult],al

  xor edx,edx

@2:

  mov al,byte ptr[CodeBase+edx]

  mov cl,byte ptr[MulResult]

  add al,cl

  mov ecx,dword ptr[ResultName]

  add ecx,210h

  cmp al,cl

  je SaveCode1

  inc edx

  cmp edx,9

  jl @2

  inc edi

  cmp edi,9

  jl @1

SaveCode1:

  mov al,byte ptr[CodeBase+edi]

  mov byte ptr[CodeBuffer+6],al       ;保存第二段第一位值

  mov al,byte ptr[CodeBase+edx]

  mov byte ptr [CodeBuffer+7],al      ;保存第二段第一位值

;--------------------------------------

;获得第1、3 段注册码,我以变换用户名实现

;--------------------------------------

  xor edi,edi

GetCode:

  mov dl,byte ptr[NameBuffer+edi]

  add edx,edx

  and edx,7

  mov al,byte ptr[CodeBase+edx]      ;取单个注册码

  mov byte ptr[CodeBuffer+edi],al    ;填入第一段注册码

  add edx,5

  and edx,7

  mov al,byte ptr[CodeBase+edx]      ;取单个注册码

  mov byte ptr[CodeBuffer+9+edi],al  ;填入第三段注册码

  inc edi

  cmp edi,5

  jl GetCode

;-----------------

;处理1、3 段注册码

;-----------------

  xor edi,edi

  mov dword ptr[TempResult],0

  xor eax,eax

ConvertCode1:

  mov al,byte ptr[CodeBuffer+edi]

  push eax

  mov dl,al

  pop ecx

  mov edx,edi

  and edx,7

  xor ecx,ecx

  mov cl,byte ptr[xorBuffer+edx]

  xor eax,ecx

  add dword ptr[TempResult], eax

  inc edi

  cmp byte ptr ds:[CodeBuffer+edi],0

  jnz ConvertCode1

  xor edi,edi

  xor eax,eax

ConvertCode2:

  mov al,byte ptr[CodeBuffer+9+edi]

  push eax

  mov dl,al

  pop ecx

  mov edx,edi

  and edx,7

  xor ecx,ecx

  mov cl,byte ptr[xorBuffer+edx]

  xor eax,ecx

  add dword ptr[TempResult], eax

  inc edi

  cmp byte ptr ds:[CodeBuffer+9+edi],0

  jnz ConvertCode2

  mov eax,dword ptr[TempResult]

  mov ecx,64h

  cdq
```
