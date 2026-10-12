
# 文件密使 2.1

> 原文：[`www.pediy.com/kssd/pediy03/forum1150.htm`](https://www.pediy.com/kssd/pediy03/forum1150.htm)

**目标软件**：文件密使 2.1

**破解人**：TAE!\[CCG BCG FCG\]

**保护方法**：注册码

**下载地址**：`http://newhua.infosail.com/down/jiami.exe`

前面的步骤不再敷述，下面是重点！

跟踪到这里：

```
:00415C85 E80E4E0000    Call 0041AA98

:00415C8A 8D8E50160000 lea ecx, dword ptr [esi+00001650]

:00415C90 E82BC3FFFF    call 00411FC0       \\进入

:00415C95 85C0         test eax, eax

:00415C97 752E         jne 00415CC7

:00415C99 6A40         push 00000040

* Possible Reference to Dialog: 

                      |

:00415C9B 6890814200    push 00428190

* Possible StringData Ref from Data Obj ->"注册码错误，请重新输入"

                      |

:00415CA0 68288F4200    push 00428F28

:00415CA5 8BCB         mov ecx, ebx
```

下面是第一段：

```
:0041204A 8A1C0F       mov bl, byte ptr [edi+ecx]

:0041204D 8D340F       lea esi, dword ptr [edi+ecx]

:00412050 8BC3       mov eax, ebx

:00412052 25FF000000   and eax, 000000FF

:00412057 99         cdq

:00412058 F7F9       idiv ecx

:0041205A 8BD0       mov edx, eax

:0041205C 8AC3       mov al, bl

:0041205E F6E9       imul cl

:00412060 02D0       add dl, al

:00412062 41         inc ecx

:00412063 83F920     cmp ecx, 00000020

:00412066 88142E     mov byte ptr [esi+ebp], dl

:00412069 7CDF       jl 0041204A
```

这段程序是算第一个数据串依次放入了 `esi+ebp`，待会说说它的算法！下面会用到。

```
:0041206B 8B6C2410     mov ebp, dword ptr [esp+10]

:0041206F 33FF       xor edi, edi

:00412071 33F6       xor esi, esi

:00412073 8BC6       mov eax, esi

:00412075 33C9       xor ecx, ecx

:00412077 99         cdq

:00412078 83E203     and edx, 00000003

:0041207B C6443C1000 mov [esp+edi+10], 00

:00412080 03C2       add eax, edx

:00412082 C1F802     sar eax, 02

:00412085 8D440414   lea eax, dword ptr [esp+eax+14]
```

再看下面这里：

又一个循环！

```
:00412089 8A1408       mov dl, byte ptr [eax+ecx]

:0041208C 8A5C3C10     mov bl, byte ptr [esp+edi+10]

:00412090 32DA       xor bl, dl

:00412092 41         inc ecx

:00412093 83F908     cmp ecx, 00000008

:00412096 885C3C10   mov byte ptr [esp+edi+10], bl

:0041209A 7CED       jl 00412089                \\内层的小循环
```

将上面得到的数据和用户名连接进行计算！

```
:0041209C 83C620       add esi, 00000020      \\大循环每循环一次 esi＋20

:0041209F 47         inc edi               

:004120A0 81FE80000000 cmp esi, 00000080      \\也就是循环 4 次喽！

:004120A6 7CCB       jl 00412073            \\外面一层大循环！
```

经过以上四次大循环可以得到四个数值！

```
:004120AE 33C9       xor ecx, ecx

:004120B0 8A4C040C   mov cl, byte ptr [esp+eax+0C]

:004120B4 03E9       add ebp, ecx

:004120B6 85C0       test eax, eax

:004120B8 7406       je 004120C0

:004120BA C1E508     shl ebp, 08

:004120BD 48         dec eax

:004120BE 79EE       jns 004120AE
```

这里试讲算出的四个数值连接起来放入 `ebp`，于是这个 `ebp` 中数的 10 进制值就是正确的注册码了！

```
:004120C0 8B542444     mov edx, dword ptr [esp+44]

:004120C4 8D4C2440     lea ecx, dword ptr [esp+40]

:004120C8 2BEA       sub ebp, edx

:004120CA C7442438FFFFFFFF mov [esp+38], FFFFFFFF

:004120D2 F7DD       neg ebp

:004120D4 1BED       sbb ebp, ebp

:004120D6 45         inc ebp

:004120D7 8BF5       mov esi, ebp
```

### 注册算法

姓名以 `1234` 为例，有 4 个字符，用名字的最后一个字符

```
0x34/4=0xD

0x34*4=0xD0

0xD0+0xD=0xDD//数据串中的第一个数
```

然后就是用

```
0xDD/5=0x2C

0xDD*5=0xFF51

0x51＋0x2C＝0x7D//数据串中的第二个数
```

算出 20 个数后，将名字和这 20 个数连接起来形成 `31323334DD7D020E7105322AFBD282EE20F0E0AC871`

```
0x31⁰=0x31

0x31⁰x32=03

03⁰x33=30
```

做 8 次这样的操作以后得到一个数 `0xA8`

外面的大循环做 4 次，于是得到四个数 `0xA8`、`0xE0`、`0x3E`、`0x00`，将四个数反相连接起来得到 `3EE0A8`，

这就是注册码的 16 进制数，那么将它换算成十进制就是正确的注册码了！

一个可用的注册码！

**用户名**：`TAE!\[CCG\]`

**注册码**：`45677`

我的注册机已经写了一大半了，可是有些实现为程序的细节不太清楚，所以请伪装者大哥给写个注册机，

好让我学协！多谢了！！

```c
#include "stdio.h"

main()

{

unsigned char string[32];

int i,d;

unsigned long n;

static g[4];

unsigned char c,e;

printf("*****JiaMi 2.1****\nThis keygen is made by Pretender\nPlease input your name : ");

gets(string);

printf("Your Register code is  : ");

d=strlen(string);

for(i=d-1;i<32;i++)

{c=string[i];

e=c;

e/=(i+1);

c*=(i+1);

c&=0xff;

c+=e;

c&=0xff;

string[i+1]=c;

}

for(i=0;i<8;i++)

{c=string[i];

g[0]^=c;}

for(i=8;i<16;i++)

{c=string[i];

g[1]^=c;}

for(i=16;i<24;i++)

{c=string[i];

g[2]^=c;}

for(i=24;i<32;i++)

{c=string[i];

g[3]^=c;}

n=g[3];

n=n<<8;

n+=g[2];

n=n<<8;

n+=g[1];

n=n<<8;

n+=g[0];

printf("%lu",n);

printf("\n **********************      ---   ---   ---\n *Welcome to WWW.CRACKNOW.COM*      /      /      / --\n **********************      ---   ---   --/\n");

}
```

编译后 3K,哪位懒得编译 QQ 发给你

```
;*******************MASM 6.0 Compile

cls

c:\masm32\bin\ml /c /coff /Cp  cr_wm.asm

rc /v cr_wm.rc

c:\masm32\bin\link /SUBSYSTEM:WINDOWS /LIBPATH:c:\masm32\lib cr_wm.obj cr_wm.res

;*******************

.386

.model flat,stdcall

option casemap:none

include c:\masm32\include\windows.inc

include c:\masm32\include\user32.inc

include c:\masm32\include\kernel32.inc

includelib c:\masm32\lib\user32.lib

includelib c:\masm32\lib\kernel32.lib

_ProcDlg proto :DWORD,:DWORD,:DWORD,:DWORD   

;->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>data seg

      .data

Serial db 4 dup(0)

Username  db 36 dup(0)

Currentpos dd 0

szBuf1  db 100 dup(0)

szBuf2  db "%u",0

MsgMesaage1 db "Must input Username in order to generate Reg Code! ",0

MsgCap     db "By (C)hume,2001",0

MsgMesaage2 db "The Username can't exceed 32 byetes! ",0

      .data?

hInstance  HANDLE ?

      .const

DLG_MAIN equ     1000

IDC_UN    equ     1001

IDC_REG   equ     1002

ID_GEN    equ     1003

ID_EXIT   equ     1004

  ;-->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>code seg

      .code

start: 

   invoke   GetModuleHandle,NULL

   mov    hInstance,eax

   invoke   DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _ProcDlg,0

   invoke   ExitProcess,NULL

_ProcDlg   proc    uses ebx edi esi, \

      hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD

      mov   eax,wMsg

      .if   eax == WM_CLOSE

         invoke   EndDialog,hWnd,NULL

      .elseif   eax == WM_INITDIALOG

                 invoke   SetWindowPos,hWnd,HWND_TOPMOST,0,0,0,0,\

              SWP_NOMOVE or SWP_NOSIZE or SWP_DRAWFRAME

      .elseif   eax == WM_COMMAND

        mov eax,wParam

        .IF lParam!=0

           .if ax==ID_GEN

              invoke GetDlgItem,hWnd,IDC_UN

              invoke SetFocus,eax

              invoke RtlZeroMemory,addr Username,33

              invoke RtlZeroMemory,addr szBuf1,100
```



```asm
invoke GetDlgItemText, hWnd, IDC_UN, addr Username, 32

    .if eax > 32
    invoke MessageBox, NULL, addr MsgMesaage1, addr MsgCap, MB_OK or MB_ICONEXCLAMATION

    .elseif eax != 0
    call Cal
    mov eax, dword ptr Serial
    invoke wsprintf, addr szBuf1, addr szBuf2, eax
    invoke SetDlgItemText, hWnd, IDC_REG, addr szBuf1
    invoke GetDlgItem, hWnd, IDC_REG
    invoke SetFocus, eax
    .else
    invoke MessageBox, NULL, addr MsgMesaage1, addr MsgCap, MB_OK or MB_ICONEXCLAMATION
    .endif

    .elseif ax == ID_EXIT
    invoke SendMessage, hWnd, WM_CLOSE, NULL, NULL
    .endif

.ENDIF

.else
mov eax, FALSE
ret

.endif
mov eax, TRUE
ret
```

`_ProcDlg` `ENDP`

`Cal` `proc`

```asm
pusha
mov al, 0
mov ecx, -1
LEA EDI, Username
repnz scasb
NOT ecx
dec ecx
LEA EDI, Username
dec edi

Fill_Rest:
mov bl, byte ptr [edi+ecx]
lea esi, [edi+ecx]
mov eax, ebx
and eax, 0FFh
cdq
idiv ecx
mov edx, eax
mov al, bl
imul cl
add dl, al
inc ecx
cmp ecx, 20h
mov byte ptr [esi+1], dl
jl Fill_Rest
lea ebp, Serial
xor esi, esi
xor edi, edi
mov eax, esi

Calsn: mov eax, esi
xor ecx, ecx
cdq
mov byte ptr [ebp+edi], 0
sar eax, 2

Xor_Every_8bit:
add ecx, eax
mov dl, byte ptr [ebp+ecx+4] ;取每 8 位第一个字节
and ecx, 7
mov bl, byte ptr [ebp+edi]
xor bl, dl
inc ecx
cmp ecx, 8
mov byte ptr [ebp+edi], bl
jl Xor_Every_8bit
add esi, 20h
inc edi
cmp esi, 80h
jl Calsn
popa
ret
```

`Cal` `endp`

```asm
;-->>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>end all
end start
;************************************************
```

```c
#include <c:\masm32\include\resource.h>

#define DLG_MAIN     1000
#define IDC_UN       1001
#define IDC_REG      1002
#define ID_GEN       1003
#define ID_EXIT      1004

DLG_MAIN DIALOG 31, 56, 262, 52

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU

CAPTION "文件密使.V2.1 Keygen by Hume.E-mail:humewen@263.net"

FONT 8, "MS Sans Serif"

{
DEFPUSHBUTTON "GENERATE", ID_GEN, 191, 6, 50, 14
PUSHBUTTON "EXIT", ID_EXIT, 191, 26, 50, 14
EDITTEXT IDC_UN, 43, 10, 128, 12, WS_BORDER | WS_TABSTOP
EDITTEXT IDC_REG, 43, 29, 128, 13, ES_READONLY | WS_BORDER | WS_TABSTOP
LTEXT "注册名:", -1, 8, 12, 26, 10
LTEXT "注册码:", -1, 8, 30, 25, 9
}
```

4dd466d0e1e0cc123afbe504da2462ca
