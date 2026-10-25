
# 图像合成的利器 2003 v4

> 原文：`www.pediy.com/kssd/pediy06/pediy6790.htm`

`Software:h*y*2003 v4.06`

图像合成的利器

`Tools:IDA Pro v4.5、TRW 2000`

`Cracker:lq7972[bruceyu13@sina.com]`

`Notes:学习。以前用过它的老版本，今天看到新点的就装上了；由于上次 PJ 的资料丢了，只好从头来～这个软件实属国产精品，有条件就注册用 Z 版；如果你只是为了省钱，那么不用看这篇文字了`

记得老版本中的注册机制是非常典型的真假码对比跳转，断点`bpx hmemcpy`很容易找到注册码比较的地方～实际上拦住转到程序领空后`d ecx`再`Alt+↑↓`就见得到了，可见程序是事先把注册码计算好，再通过打开注册界面获得用户输入，最后比较

我关心的是它算法～

`TRW`载入，断点`bpx SendMessage`（其它如`ShowWindow`等等也行，但`CreateWindow`就～），按`F5`，打开注册窗口，拦住后`pmodule`，来到程序界面，按`ESC`键退出注册窗口，再次中断；现在我们看看在哪里？

```
snyped:004119B8          mov     [esp+0C0h+var_18], eax

snyped:004119BF          call    ?DoModal@CDialog@@UAEHXZ  CDialog::DoModal(void)

snyped:004119C4          cmp     eax, 1

snyped:004119C7          jnz     short loc_4119EC
```

`004119BF`处的`Call`就是了。好，基于前面的分析我们从这里往前不远应该能找到注册算法或调用注册算法的`Call`－－果然，上面就有这么一句

```
snyped:004118E1          call    ds:GetComputerNameA
```

看来正在接近中。。。

```
snyped:004118B0          push    offset asc_4404E4  "%x"

snyped:004118B5          push    eax

snyped:004118B6          mov     byte ptr [esp+0CCh+var_4], bl

snyped:004118BD          call    ?Format@CString@@QAAXPBDZZ  CString::Format(char const *,...)
```

激动中。。。

```
snyped:00411831          lea     edx, [esp+100h+var_90]

snyped:00411835          push    offset aCCCCCCCCCCCCCC  "%c%c%c%c-%c%c%c%c-%c%c%c%c-%c%c%c%c"

snyped:0041183A          push    edx

snyped:0041183B          call    ?Format@CString@@QAAXPBDZZ  CString::Format(char const *,...)

snyped:00411840          mov     eax, [esp+108h+var_A4]
```

到这里，`"%c%c%c%c-%c%c%c%c-%c%c%c%c-%c%c%c%c"`这个东西应就是注册码的格式了！注册算法就在这里！

再次激动。。。

顶上就是注册算法

`snyped:004116B1`

```
snyped:004116B1 loc_4116B1:                          CODE XREF: sub_410FD0+5DDj

snyped:004116B1          mov     eax, [edi+0D0h] ;这个值？(387F9FFh)

snyped:004116B7          cmp     eax, ebx

snyped:004116B9          jnz     short loc_41171A

;。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。

 loc_41171A:                          CODE XREF: sub_410FD0+6E9j

snyped:0041171A          add     eax, 0BCC09h ;加上 0BCC09h

snyped:0041171F          push    eax

snyped:00411720          lea     eax, [esp+0C4h+var_A8]

snyped:00411724          push    offset a8d       "%8d"

snyped:00411729          push    eax

snyped:0041172A          call    ?Format@CString@@QAAXPBDZZ  CString::Format(char const *,...) ;这个是 Hex2Dec

snyped:0041172F          mov     eax, [esp+0CCh+var_A8] ;Num

snyped:00411733          add     esp, 0Ch
```

;从这里正式开始。。。

```
snyped:00411736          mov     dl, [eax+1]

snyped:00411739          mov     cl, [eax]

snyped:0041173B          movsx   eax, byte ptr [eax+2]

snyped:0041173F          movsx   edx, dl

snyped:00411742          movsx   ecx, cl

snyped:00411745          add     eax, edx

snyped:00411747          lea     eax, [eax+ecx-90h]

snyped:0041174E          mov     ecx, 7

snyped:00411753          cdq

snyped:00411754          idiv    ecx

snyped:00411756          lea     ecx, [esp+0C0h+var_A8]

snyped:0041175A          add     dl, 30h
```

;歇一下，喘口气

```
snyped:0041175D          push    edx

snyped:0041175E          push    3

snyped:00411760          call    ?SetAt@CString@@QAEXHD@Z  CString::SetAt(int,char)
```

;这个是结果替换 `Num` 第 4 位

```
snyped:00411765          mov     eax, [esp+0C0h+var_A8]
```

;开始～

```
snyped:00411769          mov     dl, [eax+5]

snyped:0041176C          mov     cl, [eax+4]

snyped:0041176F          movsx   eax, byte ptr [eax+6]

snyped:00411773          movsx   edx, dl

snyped:00411776          movsx   ecx, cl

snyped:00411779          add     eax, edx

snyped:0041177B          lea     eax, [eax+ecx-90h]

snyped:00411782          mov     ecx, 7

snyped:00411787          cdq

snyped:00411788          idiv    ecx

snyped:0041178A          add     dl, 30h
```

;再歇歇

```
snyped:0041178D          push    edx

snyped:0041178E          push    ecx

snyped:0041178F          lea     ecx, [esp+0C8h+var_A8]

snyped:00411793          call    ?SetAt@CString@@QAEXHD@Z  CString::SetAt(int,char)
```

;这个是结果替换 `Num` 第 8 位

```
snyped:00411798          mov     eax, [esp+0C0h+var_A8]
```

;Come on～

```
snyped:0041179C          mov     cl, [eax+2]

snyped:0041179F          mov     bl, [eax+1]

snyped:004117A2          mov     dl, [eax+7]

snyped:004117A5          mov     [esp+0C0h+var_AD], cl

snyped:004117A9          mov     cl, [eax+3]

snyped:004117AC          mov     [esp+0C0h+var_A9], bl

snyped:004117B0          mov     bl, [eax+4]

snyped:004117B3          mov     [esp+0C0h+var_AB], cl

snyped:004117B7          mov     cl, [eax+6]

snyped:004117BA          mov     [esp+0C0h+var_AA], bl

snyped:004117BE          mov     bl, [eax]

snyped:004117C0          mov     al, [eax+5]

snyped:004117C3          mov     [esp+0C0h+var_AC], al
```

;计算完

;最后处理并格式化输出，要注意前后变量（变量没有`r/s`）的移送：

```
snyped:004117C7          movsx   eax, dl ;从第 16 位到第 1 位

snyped:004117CA          push    eax

snyped:004117CB          movsx   eax, [esp+0C4h+var_AD]

snyped:004117D0          add     eax, 15h

snyped:004117D3          push    eax

snyped:004117D4          movsx   eax, [esp+0C8h+var_AB]

snyped:004117D9          push    eax

snyped:004117DA          movsx   eax, cl

snyped:004117DD          add     eax, 17h

snyped:004117E0          push    eax

snyped:004117E1          movsx   eax, [esp+0D0h+var_A9]

snyped:004117E6          push    eax

snyped:004117E7          movsx   eax, [esp+0D4h+var_AA]

snyped:004117EC          add     eax, 11h

snyped:004117EF          push    eax

snyped:004117F0          movsx   eax, bl

snyped:004117F3          push    eax

snyped:004117F4          movsx   eax, [esp+0DCh+var_AC]

snyped:004117F9          add     eax, 11h

snyped:004117FC          push    eax

snyped:004117FD          movsx   eax, cl

snyped:00411800          push    eax

snyped:00411801          movsx   eax, [esp+0E4h+var_AB]

snyped:00411806          add     eax, 13h

snyped:00411809          push    eax

snyped:0041180A          movsx   eax, [esp+0E8h+var_AA]

snyped:0041180F          push    eax

snyped:00411810          movsx   edx, dl

snyped:00411813          movsx   eax, [esp+0ECh+var_AC]

snyped:00411818          add     edx, 12h

snyped:0041181B          push    edx

snyped:0041181C          push    eax

snyped:0041181D          movsx   edx, [esp+0F4h+var_AD]

snyped:00411822          movsx   eax, cl

snyped:00411825          movsx   ecx, bl

snyped:00411828          add     edx, 1Bh

snyped:0041182B          add     ecx, 11h

snyped:0041182E          push    edx

snyped:0041182F          push    eax

snyped:00411830          push    ecx

snyped:00411831          lea     edx, [esp+100h+var_90]

snyped:00411835          push    offset aCCCCCCCCCCCCCC  "%c%c%c%c-%c%c%c%c-%c%c%c%c-%c%c%c%c"

snyped:0041183A          push    edx
```



# 注册机

`snyped:0041183B` `call ?Format@CString@@QAAXPBDZZ` `CString::Format(char const*,...)`；`ECX` 指向生成的注册码

很简单吧？

写注册机，还是用 `MASM32`，方便，照抄就行:)

## 【注册机】

```
;//////////////////////////////////////////////////////////////////////////////////////////////

;今天有点事儿要办，晚上再写吧

标题:hy2003 v 4.0.6 注册机（MASM32） (6 千字)

发信人:lq7972

时间:2003-10-01 17:44:12

详细信息:

hy2003 v 4.0.6 注册机（MASM32）

Notes:到今天才写它，因为终于要放几天假啦；啦啦啦啦～

[补遗]上面对那个值(004116B1 mov eax, [edi+0D0h])没有弄明白：原以为是软件的固定值；到今天写注册机才知道它需要也可以从软件编号反算出来，见注册机源码--其它的主要是注意通过对这个值进行不停的或前或后取(并计算)出注册码
```

### `KeyGen.asm`

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; The KeyGen by lq7972,with MASM32 V8

; E-mail:bruceyu13@sina.com

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.386

.model flat,stdcall

option casemap:none

;Include 文件定义

include windows.inc

include user32.inc

includelib user32.lib

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

szName db 11 dup (?)

szSN db 20 dup (?)

Reg1 dd 4 dup (?)

Reg2 dd 4 dup (?)

Reg3 dd 4 dup (?)

Reg4 dd 4 dup (?)

Temp dd 8 dup (?)

.data

hInstance dd 0

szErr db '请输入！',0

szCaption db '错误！',0 

RegFmt db '%4s-%4s-%4s-%4s',0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code

;****************************************************************************************

_RegFmt proc

local @Temp1:byte,@Temp2:byte,@Temp3:byte,@Temp4:byte,@Temp5:byte

pushad

lea eax,offset Temp

mov cl,[eax+2]

mov bl,[eax+1]

mov dl,[eax+7]

mov byte ptr @Temp1,cl

mov cl,[eax+3]

mov byte ptr @Temp2,bl

mov bl,[eax+4]

mov byte ptr @Temp3,cl

mov cl,[eax+6]

mov byte ptr @Temp4,bl

mov bl,[eax]

mov al,[eax+5]

mov byte ptr @Temp5,al

mov byte ptr [Reg4+3],dl

movsx eax,byte ptr [@Temp1]

add eax,15h

mov byte ptr [Reg4+2],al

movsx eax,byte ptr [@Temp3]

mov byte ptr [Reg4+1],al

movsx eax,cl

add eax,17h

mov byte ptr [Reg4],al

movsx eax,byte ptr [@Temp2]

mov byte ptr [Reg3+3],al

movsx eax,byte ptr [@Temp4]

add eax,11h

mov byte ptr [Reg3+2],al 

mov byte ptr [Reg3+1],bl

movsx eax,byte ptr [@Temp5]

add eax,11h

mov byte ptr [Reg3],al 

mov byte ptr [Reg2+3],cl

movsx eax,byte ptr [@Temp3]

add eax,13h

mov byte ptr [Reg2+2],al

movsx eax,byte ptr [@Temp4]

mov byte ptr [Reg2+1],al

movsx edx,dl

movsx eax,byte ptr [@Temp5]

add edx,12h

mov byte ptr [Reg2],dl

mov byte ptr [Reg1+3],al

movsx edx,byte ptr [@Temp1]

movsx eax,cl

movsx ecx,bl

add edx,1Bh

add ecx,11h

mov byte ptr [Reg1+2],dl

mov byte ptr [Reg1+1],al

mov byte ptr [Reg1],cl

invoke wsprintf,addr szSN,addr RegFmt,addr Reg1,addr Reg2,addr Reg3,addr Reg4

popad

ret

_RegFmt endp

;****************************************************************************************

_RegCodCalc proc

pushad

invoke htodw,addr [szName+3] ;MASM32.LIB 转换函数，从软件编号反算生成注册码的关键值

sub eax,0B4E9Dh

add eax,0BCC09h

invoke dwtoa,eax,addr Temp

lea eax,offset Temp

xor edx,edx

xor ecx,ecx

mov dl,[eax+1]

mov cl,[eax]

movsx eax,byte ptr [eax+2]

movsx edx,dl

movsx ecx,cl

add eax,edx

lea eax,[eax+ecx-90h]

mov ecx,7

cdq

idiv ecx

add dl,30h

mov byte ptr [Temp+3],dl

lea eax,offset [Temp]

mov dl,[eax+5]

mov cl,[eax+4]

movsx eax,byte ptr [eax+6]

movsx edx,dl

movsx ecx,cl

add eax,edx

lea eax,[eax+ecx-90h]

mov ecx,7

cdq

idiv ecx

add dl,30h

mov byte ptr [Temp+7],dl

invoke _RegFmt

popad

ret

_RegCodCalc endp

;****************************************************************************************

_ProcDlgMain proc uses ebx edi esi ebp hWnd,wMsg,wParam,lParam

mov eax,wMsg

.if eax == WM_CLOSE 

invoke EndDialog,hWnd,NULL

.elseif eax == WM_COMMAND 

mov eax,wParam 

.if eax == IDOK

invoke RtlZeroMemory,offset szName,512

invoke GetDlgItemText,hWnd,EditName,offset szName,11

.if eax != NULL

invoke _RegCodCalc

invoke SetDlgItemText,hWnd,EditSN,offset szSN

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

### `KeyGen.rc`

```rc
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

#include   

#define ICO_MAIN 0x1000

#define DLG_MAIN 1

#define EDITName 10

#define EDITSN 11

//ICO_MAIN ICON "01.ico"

DLG_MAIN DIALOG 100,150,250,60

STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU | WS_THICKFRAME

CAPTION "幻影 2003 v4.06 注册机" 

FONT 9,"宋体" 

{

 CONTROL "Name:" ,-1,"Static",SS_LEFT,10,13,40,17

 CONTROL "Code:" ,-2,"Static",SS_CENTER,10,40,20,17

 CONTROL "" ,10,"Edit",ES_LEFT,30,13,150,10

 CONTROL "" ,11,"Edit",ES_LEFT,30,40,150,10

 DEFPUSHBUTTON "GENERATE",IDOK,200,11,40,15

 PUSHBUTTON "EXIT",IDCANCEL,200,36,41,14

}
```

### `makefile`

```make
;<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

NAME = KeyGen 

OBJS = $(NAME).obj

RES = $(NAME).res

LINK_FLAG = /subsystem:windows

ML_FLAG = /c /coff

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
