
# Graphic Workshop Professional v2.0a

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-269.htm`](https://www.pediy.com/kssd/pediy07/pediy7-269.htm)

**【破解作者】** qfejj

**【作者邮箱】** qfejj@163.com

**【使用工具】** `OllyDbg v1.10`; `PEID v0.93`; `RadASM 2.1.0`

**【破解平台】** `Windows XP`

**【软件名称】** `Graphic Workshop Professional v2.0a` (patch98)

**【下载地址】** `http://crc.onlinedown.net:82/files/GraphicWorkshop.exe`

**【软件简介】** 按照它自己的介绍，是一个专业的图片管理软件包，可以浏览图片（废话），转换图片格式，打印，建立可搜索图片数据库，帮助用户完成各种复杂的图形处理。我认识它才两天，不知道真的假的。不过那段提示用户注册的语音，倒是令人耳目一新。

**【软件大小】** 8.05MB

**【加壳方式】** 无壳

**【破解声明】** 最近闷的慌，昨天去华军拖了这个软件破了玩玩，纯粹兴趣使然。

---

## 破解内容

注册码格式 `xxxxx-xx-xxxxx-xx`，软件注册码和安装程序注册码相通，我就通过调试安装程序获得软件的注册码。调试后发现软件处理用户名和注册码的大致过程如下：

1.  **第 1 步**：用户名变换，第二段注册码变换，两者结果比较，通过则继续第 2 步，否则提示出错。
2.  **第 2 步**：第一段和第三段注册码一起变换，第四段注册码变换，两者结果比较，通过则完成注册，否则提示出错。

下面详细分析该软件处理用户名和注册码所使用的具体算法。

`OD` 载入目标程序，`bpx GetWindowTextA`，运行程序，输入注册码，确定后 `OD` 中断在每个 `GetWindowTextA`，关键代码分析如下：

```assembly
0040415A      push eax

0040415B      call < jmp.&USER32.GetWindowTextA> ; <--取用户名保存到 eax 所指地址 4148b8

...(省略部分)

00404177      push eax

00404178      call < jmp.&USER32.GetWindowTextA> ; <--取第 1 段注册码保存到缓冲区
0040417D      lea eax,dword ptr ss:[ebp-810]
00404183      push eax
00404184      lea edx,dword ptr ss:[ebp-40C]
0040418A      push edx
0040418B      call GraphicW.0040A110          ; <--从缓冲区复制第 1 段注册码到 eax 所指的地址 12CCC0
00404190      add esp,8
00404193      push GraphicW.0041628F

00404198      lea ecx,dword ptr ss:[ebp-40C]

0040419E      push ecx

0040419F      call GraphicW.0040A110          ; <--在第 1 段注册码后添加短横线
...(省略部分)

004041BE      push eax

004041BF      call < jmp.&USER32.GetWindowTextA> ; <--取第 2 段注册码保存到缓冲区
004041C4      lea edx,dword ptr ss:[ebp-810]
004041CA      push edx

004041CB      lea ecx,dword ptr ss:[ebp-40C]

004041D1      push ecx

004041D2      call GraphicW.0040A110          ; <--从缓冲区复制第 2 段注册码到 eax 所指的地址 12CCC6
004041D7      add esp,8
004041DA      push GraphicW.00416291

004041DF      lea eax,dword ptr ss:[ebp-40C]

004041E5      push eax

004041E6      call GraphicW.0040A110          ; <--在第 2 段注册码后添加短横线
...(省略部分)

00404205      push eax

00404206      call < jmp.&USER32.GetWindowTextA> ; <--取第 3 段注册码保存到缓冲区
0040420B      lea ecx,dword ptr ss:[ebp-810]
00404211      push ecx

00404212      lea eax,dword ptr ss:[ebp-40C]

00404218      push eax

00404219      call GraphicW.0040A110          ; <--缓冲区复制第 3 段注册码到 eax 所指的地址 12CCC9
0040421E      add esp,8
00404221      push GraphicW.00416293

00404226      lea edx,dword ptr ss:[ebp-40C]

0040422C      push edx

0040422D      call GraphicW.0040A110          ; <--在第 3 段注册码后添加短横线
...(省略部分)

0040424C      push eax

0040424D      call < jmp.&USER32.GetWindowTextA> ; <--取第 4 段注册码保存到缓冲区
00404252      lea eax,dword ptr ss:[ebp-810]
00404258      push eax

00404259      lea edx,dword ptr ss:[ebp-40C]

0040425F      push edx

00404260      call GraphicW.0040A110          ; <--从缓冲区复制第 4 段注册码到 eax 所指的地址 12CCCF
00404265      add esp,8
00404268      lea ecx,dword ptr ss:[ebp-40C]
0040426E      push ecx

0040426F      push GraphicW.00414CBA

00404274      call GraphicW.0040A1A0          ; <--把所有注册码复制到地址 414CBA
```

上面的代码可以不看，关键的在下面。

### (1) 这是软件处理用户名的具体过程

```assembly
004042A7      xor eax,eax
004042A9      mov al,byte ptr ds:[edi+4148B8]    ;  <--用户名每一位字母送入 al 开始处理,4148b8 保存着用户名
004042AF      push eax

004042B0      call GraphicW.0040F9E0

004042B5      pop ecx

004042B6      mov edx,edi
004042B8      and edx,7             ;因为下面[41308C]指向的数组中元素有限,以此更改元素地址，循环使用
004042BB      xor ecx,ecx
004042BD      mov cl,byte ptr ds:[edx+41308C]    ;取出一个数组元素，调试后得到该数组为[80,40，20，10，08，04，02，01]
004042C3      xor eax,eax             ;<--用户名 ASSII 值与上面取出的元素值异或
004042C5      add dword ptr ss:[ebp-8],eax       ;<--保存异或结果，累计
004042C8      inc edi               ;<--edi 增 1，用于定位用户名下一个用户名字符
004042C9      cmp byte ptr ds:[edi+4148B8],0     ;<--比较用户名是否已经取完
004042D0      jnz short GraphicW.004042A7
004042D2      mov eax,dword ptr ss:[ebp-8]       ;<--取出前面累计结果，进行下一步处理
004042D5      mov ecx,64
004042DA      cdq               ;<--将 EAX 中的数符号扩展为 EDX:EAX 中的 64 位数,EDX 存放高 32 位
004042DB      idiv ecx             ;<--处理后的用户名除以 64，AX 中的符号位扩到 DX 中(关键)
004042DD      mov edi,edx               ;保存最终处理结果到 edi
004042DF      test edi,edi
004042E1      jnz short GraphicW.004042E4
```

### (2) 这是软件处理第 1 段和第 3 段注册码的具体过程

阅读时参考用户名的处理过程:

```assembly
0040430B     mov al,byte ptr ds:[edi+414CBA]    ;<--第 1 段注册码每一位字母送入 al 开始处理

00404311     mov edx,edi
00404313     and edx,7

00404316     xor al,byte ptr ds:[edx+41308C]    ;数组，[80,40，20，10，08，04，02，01]
0040431C     xor ecx,ecx
0040431E     mov cl,al
00404320     add dword ptr ss:[ebp-8],ecx       ;<--保存处理结果，累计
00404323     inc edi
00404324     cmp byte ptr ds:[edi+414CBA],0     ;<--第 1 段注册码是否取完
0040432B     je short GraphicW.00404332
0040432D     cmp edi,5

00404330     jl short GraphicW.0040430B
00404332     xor edi,edi
00404334     jmp short GraphicW.0040434F
00404336     mov al,byte ptr ds:[edi+414CC3]    ;<--第 3 段注册码每一位字母送入 al 开始处理

0040433C     mov edx,edi
0040433E     and edx,7
00404341     xor al,byte ptr ds:[edx+41308C]    ;数组，[80,40，20，10，08，04，02，01]
00404347     xor ecx,ecx
00404349     mov cl,al
0040434B     add dword ptr ss:[ebp-8],ecx       ;<--保存处理结果，累计
0040434E     inc edi
0040434F     cmp byte ptr ds:[edi+414CC3],0     ;<--第 3 段注册码是否取完
00404356     je short GraphicW.0040435D
00404358     cmp edi,5
0040435B     jl short GraphicW.00404336
0040435D     mov eax,dword ptr ss:[ebp-8]       ;<--取出第 1 段和第 3 段注册码处理的累计结果
00404360     mov ecx,64
00404365     cdq               ;将 EAX 中的数符号扩展为 EDX:EAX 中的 64 位数,EDX 存放高 32 位
00404366     idiv ecx             ;<--处理后的注册码除以 64，AX 中的符号位扩到 DX 中(关键)
00404368     mov edi,edx               ;<--保存最终处理结果到 edi
0040436A     test edi,edi
0040436C     jnz short GraphicW.0040436F
```

### (3) 处理第 2 段和第 4 段注册码用的是同一个函数

具体分析如下:

```assembly
0040FBCC     push ebp
0040FBCD     mov ebp,esp
0040FBCF     push ebx

0040FBD0     push esi

0040FBD1     mov edx,dword ptr ss:[ebp+8]       ;<--指向第 2(4)段注册码地址
0040FBD4     xor ecx,ecx
0040FBD6     mov al,byte ptr ds:[edx]         ;<--第 2 段注册码第 1 位字母送入 al 开始处理
0040FBD8     inc edx
0040FBD9     movsx ebx,al            ;<--扩展到 ebx
0040FBDC     test byte ptr ds:[ebx+418A0D],1

0040FBE3     jnz short GraphicW.0040FBD6

0040FBE5     cmp al,2B             ;  +
0040FBE7     je short GraphicW.0040FBED
```



# 汇编代码分析

## 代码段

```
0040FBE9     cmp al,2D                   ;  -  

0040FBEB     jnz short GraphicW.0040FBFC  

0040FBED     cmp al,2D  

0040FBEF     sete al  

0040FBF2     and eax,1  

0040FBF5     mov esi,eax  

0040FBF7     mov al,byte ptr ds:\[edx\]  

0040FBF9     inc edx  

0040FBFA     jmp short GraphicW.0040FC10  

0040FBFC     xor esi,esi                 

0040FBFE     jmp short GraphicW.0040FC10  

0040FC00     movsx eax,al                ;---真正的处理过程，后面会以圆括弧内数字为顺序进行分析  

0040FC03     add ecx,ecx                 ;<--第一次 0+0=0，第二次(3)b+b=c  

0040FC05     lea ecx,dword ptr ds:\[ecx+ecx\*4\]   ;<--第一次 0+0\*4=0,第二次(4)c+c\*4=d  

0040FC08     add ecx,eax                 ;<--第一次(1)ecx=注册码\[1\]+ecx=a，第二次(5)注册码\[2\]+d=e  

0040FC0A     mov al,byte ptr ds:\[edx\]   ;<--第二(四)段注册码第 2 位字母送入 al 开始处理  

0040FC0C     add ecx,-30                 ;<--第一次(2)ecx-30=a-30=b，第二次(6)eax-30=e-30=f   

0040FC0F     inc edx     

0040FC10     cmp al,30                   ;  30 是数字 0 的 ASCII 值，表示这里需要数字  

0040FC12     jl short GraphicW.0040FC18     

0040FC14     cmp al,39                   ;  39 是数字 9 的 ASCII 值  

0040FC16     jle short GraphicW.0040FC00       

0040FC18     test esi,esi  

0040FC1A     je short GraphicW.0040FC22  

0040FC1C     mov eax,ecx  

0040FC1E     neg eax  

0040FC20     jmp short GraphicW.0040FC24  

0040FC22     mov eax,ecx                 ; <--(7)保存注册码的处理结果  

0040FC24     pop esi  

0040FC25     pop ebx  

0040FC26     pop ebp  

0040FC27     retn  
```

## 第 2(4) 段注册码处理算法详细分析

下面是第 2(4)段注册码处理算法的详细分析，咱们先来个约定:

注册码\[1\]表示第一个数字的 ASCII 值，注册码\[2\]表示第二个数字的 ASCII 值，我输入的第 2(4)段注册码是 78，

所以有注册码\[1\]=37,注册码\[2\]=38,算法如下:

```
(1)注册码\[1\]+ecx=a 

   ;ecx=0,结果 a=注册码\[1\]--------------------------------------------------------37  

(2)a-30=b           

   ;b=注册码\[1\]-30---------------------------------------------------------------7  

(3)b+b=c             

   ;c=注册码\[1\]-30+注册码\[1\]-30--------------------------------------------------E(十进制 14)  

(4)c+c\*4=d          

   ;d=注册码\[1\]-30+注册码\[1\]-30+{c=注册码\[1\]-30+注册码\[1\]-30}\*4------------------46  

(5)注册码\[2\]+d=e     

   ;e=注册码\[2\]+注册码\[1\]-30+注册码\[1\]-30+{注册码\[1\]-30+注册码\[1\]-30}\*4----------7E(十进制 126)  

(6)e-30=f            

   ;f=注册码\[2\]+注册码\[1\]-30+注册码\[1\]-30+{注册码\[1\]-30+注册码\[1\]-30}\*4-30-------4E  

(7)mov eax,f         

   ;保存注册码处理结果到 eax，前面已经分析过，用户名处理后的结果保存在 esi   
```

综上，得到 `eax=f=注册码[2]+注册码[1]-30+注册码[1]-30+{注册码[1]-30+注册码[1]-30}*4-30`

整理一下，即

```
============================= 
eax=注册码\[2\]+注册码\[1\]\*A-210 
=============================          
```

注意是 16 进制的，不要弄错了;到此，所有的算法分析完毕，下面说说注册机的编写。

## 注册机编写思路

既然知道了算法，下一步当然是写注册机了。通过上面的分析，我们可以按照软件的变换方法处理用户名和第 1、3 段注册码，得到处理结果(软件使用 `esi` 保存)，再由处理结果(`esi`)反推出第 2、4 注册码。也可以先得到 2、4 段注册码处理结果，再以此计算用户名，不过这样好像不妥，因为不容易得到自己喜欢的用户名。既然这样，我还是用第一种方法吧。那么，如何依靠 `esi` 计算出注册码呢?我们来看，由于注册码必须是数字，所以正确的注册码必须满足下面的关系：

```
    esi+210-39 < 注册码[2]+注册码[1]*A < esi+210-30      ;<--39 和 30 分别是 9 和 0 的 ASCII 值

    即      esi+1D7 < 注册码[2]+注册码[1]*A < esi+1E0
```

既然可以确定范围,那反推注册码也就容易多了,这里只有两个数，我就用穷举来实现了。另外，第一、三、四段注册码和用户名无关，只要满足一定关系就可以，我就拿用户名换算了一下作为第一，三段的注册码了，最后以此求解第四段注册码。如果哪位哥们有更好的求解方法，还望赐教。

## 注册机源码

```
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

;>>  Graphic  Workshop  v2.0a(pach98)  注册机  >>>

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

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

IDDIALOG            equ 10

IDOK                equ 1

IDABOUT             equ 2

IDREGNAME           equ 4

IDREGCODE           equ 3

IDC_STC1            equ 5

IDC_STC2            equ 6

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

WinDialog proto :HWND,:UINT,:WPARAM,:LPARAM

CalCode   proto

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

RGB macro red,green,blue

  xor eax,eax

  mov ah,blue

  shl eax,8

  mov ah,green

  mov al,red

endm

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.data 

MsgBoxText    db 'Graphic  Workshop  v2.0a  注册机',0dh

           db '                编写:qfejj',0dh

           db '      Email:qfejj@163.com',0         

MsgBoxCaption db 'About',0

MsgBoxTextErr db '用户名不能为空',0

MsgBoxWarning db '错误',0

String        db 'Keygen For Everyone  ',0

FontName      db 'script',0

xorBuffer     db 1 dup (80h,40h,20h,10h,08h,04h,02h,01h)

CodeBase      db 1 dup (30h,31h,32h,33h,34h,35h,36h,37h,38h,39h)

.data?

hInstance   HINSTANCE ?

hButton     HINSTANCE ?

hNameEdit   HINSTANCE ?

NameBuffer  db 33 dup(?)

CodeBuffer  db 20 dup(?)

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code

start:

  invoke InitCommonControls

  invoke GetModuleHandle,NULL

  mov hInstance,eax

  invoke DialogBoxParam,hInstance,IDDIALOG,NULL,offset WinDialog,0

  invoke ExitProcess,0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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

          mov byte ptr\[CodeBuffer+5\],dl

          mov byte ptr\[CodeBuffer+8\],dl

          mov byte ptr\[CodeBuffer+14\],dl

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

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CalCode proc

LOCAL   ResultName:dword

LOCAL   ResultCode:word

LOCAL   TempResult:dword

LOCAL   MulResult:byte

;--------------

;用户名处理过程

;--------------

  xor edi,edi

  mov dword ptr\[TempResult\],0

  xor eax,eax

ConvertName:

  mov al,byte ptr\[NameBuffer+edi\]

  push eax
```



```assembly
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[NameBuffer+edi],0
jnz ConvertName
mov eax,dword ptr [TempResult]
mov ecx,64h
cdq
idiv ecx
mov dword ptr [ResultName],edx ;保存用户名处理结果

;-----------------------------------
;获得第 2 段注册码,我用穷举来实现
;-----------------------------------

xor edi,edi

@1:
mov al,byte ptr [CodeBase+edi]
mov dl,0Ah
mul dx
mov byte ptr [MulResult],al
xor edx,edx

@2:
mov al,byte ptr [CodeBase+edx]
mov cl,byte ptr [MulResult]
add al,cl
mov ecx,dword ptr [ResultName]
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
mov al,byte ptr [CodeBase+edi]
mov byte ptr [CodeBuffer+6],al ;保存第二段第一位值
mov al,byte ptr [CodeBase+edx]
mov byte ptr [CodeBuffer+7],al ;保存第二段第一位值

;--------------------------------------
;获得第 1、3 段注册码,我以变换用户名实现
;--------------------------------------

xor edi,edi

GetCode:
mov dl,byte ptr [NameBuffer+edi]
add edx,edx
and edx,7
mov al,byte ptr [CodeBase+edx] ;取单个注册码
mov byte ptr [CodeBuffer+edi],al ;填入第一段注册码
add edx,5
and edx,7
mov al,byte ptr [CodeBase+edx] ;取单个注册码
mov byte ptr [CodeBuffer+9+edi],al ;填入第三段注册码
inc edi
cmp edi,5
jl GetCode

;-----------------
;处理 1、3 段注册码
;-----------------

xor edi,edi
mov dword ptr [TempResult],0
xor eax,eax

ConvertCode1:
mov al,byte ptr [CodeBuffer+edi]
push eax
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[CodeBuffer+edi],0
jnz ConvertCode1

xor edi,edi
xor eax,eax

ConvertCode2:
mov al,byte ptr [CodeBuffer+9+edi]
push eax
mov dl,al
pop ecx
mov edx,edi
and edx,7
xor ecx,ecx
mov cl,byte ptr [xorBuffer+edx]
xor eax,ecx
add dword ptr [TempResult], eax
inc edi
cmp byte ptr ds:[CodeBuffer+9+edi],0
jnz ConvertCode2

mov eax,dword ptr [TempResult]
mov ecx,64h
cdq
idiv ecx
mov word ptr [ResultCode],dx

;-----------------------------------
;获得第 4 段注册码,我用穷举来实现
;-----------------------------------

xor edi,edi

@3:
mov al,byte ptr [CodeBase+edi]
mov dl,0Ah
mul dx
mov byte ptr [MulResult],al
xor edx,edx

@4:
mov al,byte ptr [CodeBase+edx]
mov cl,byte ptr [MulResult]
add al,cl
mov cx,word ptr [ResultCode]
add cx,210h
cmp al,cl
je SaveCode2
inc edx
cmp edx,9
jl @4
inc edi
cmp edi,9
jl @3

SaveCode2:
mov al,byte ptr [CodeBase+edi]
mov byte ptr [CodeBuffer+15],al ;保存第四段第一位值
mov al,byte ptr [CodeBase+edx]
mov byte ptr [CodeBuffer+16],al ;保存第四段第二位值

;-------------------
;为注册码添加分隔线
;-------------------

mov byte ptr [CodeBuffer+5],2Dh
mov byte ptr [CodeBuffer+8],2Dh
mov byte ptr [CodeBuffer+14],2Dh
ret

CalCode endp
end start
```

--------------------------------------------------------------------------------

### 资源文件

```rc
;>>>>>>>>>>>>>>>>>
;>>> rsrc.rc >>>>>
;>>>>>>>>>>>>>>>>>

#define IDDIALOG 10
#define IDREGNAME 4
#define IDREGCODE 3
#define IDC_STC1 5
#define IDC_STC2 6
#define IDOK 1
#define IDABOUT 2

IDDIALOG DIALOGEX 6,5,196,104
CAPTION "GraphicWorkshop v2.0a 注册机"
FONT 8,"MS Sans Serif"
STYLE 0x10CA0800
EXSTYLE 0x00000000
BEGIN
  CONTROL "",IDREGNAME,"Edit",0x50810000,76,38,106,13,0x00000000
  CONTROL "",IDREGCODE,"Edit",0x50810800,76,60,106,13,0x00000000
  CONTROL "Registration Name",IDC_STC1,"Static",0x50000201,12,38,60,13,0x00000000
  CONTROL "Registration Code",IDC_STC2,"Static",0x50000201,12,60,58,11,0x00000000
  CONTROL "计算",IDOK,"Button",0x50010000,40,81,43,15,0x00000000
  CONTROL "关于",IDABOUT,"Button",0x50010000,114,81,43,15,0x00000000
END
```

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
