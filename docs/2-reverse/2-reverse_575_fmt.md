
# 高级文件格式分析 (原创)

> 原文：[`www.pediy.com/kssd/pediy10/102498.html`](https://www.pediy.com/kssd/pediy10/102498.html)

（一个全日文小游戏） 图片文件（扩展名 `DET`）分析及解密（原创）

可以向我索取 `EXE` 文件和 `IDB` 文件。---------- 本人 QQ：330759606 ≮风々吉它≯

重绘时，一定会调用到图片。再往回找，从而发现源头

## 首先查找绘图函数

得到 `StretchDIBits`

```
int __stdcall StretchDIBits(HDC,int,int,int,int,int,int,int,int,const void *,const BITMAPINFO *,UINT,DWORD)
```

```
.idata:00430040                 extrn StretchDIBits:dword
```

；IDA 中的流程缩略图关了，重打开的办法：`View-Toolbars-Navigation-Graph overview`

## 代码段分析

```
.text:0040AC3F                 mov     ebp, [esi]
.text:0040AC41                 mov     ebx, dword_43C208
.text:0040AC47                 mov     esi, [esi+0Ch]
.text:0040AC4A                 push    SRCCOPY         ; DWORD
.text:0040AC4F                 push    0               ; UINT
.text:0040AC51                 push    edi             ; BITMAPINFO *
.text:0040AC52                 mov     edx, ebx
.text:0040AC54                 push    ebp             ; CONST VOID *lpBits
.text:0040AC55                 sub     esi, ebx
.text:0040AC55
.text:0040AC57
.text:0040AC57 loc_40AC57:                        ; CODE XREF: DrawBitMap+ADj
.text:0040AC57                 mov     eax, dword_43CB8C
.text:0040AC5C                 mov     ecx, dword_43CB80
.text:0040AC62                 sub     edx, eax
.text:0040AC64                 lea     eax, [edx+1]
.text:0040AC67                 mov     edx, dword_43CB88
.text:0040AC6D                 push    eax             ; int
.text:0040AC6E                 sub     edx, ecx
.text:0040AC70                 inc     edx
.text:0040AC71                 push    edx             ; int
.text:0040AC72                 dec     esi
.text:0040AC73                 push    esi             ; int
.text:0040AC74                 push    ecx             ; int
.text:0040AC75                 push    eax             ; int
.text:0040AC76                 mov     eax, dword_43CB8C
.text:0040AC7B                 push    edx             ; int
.text:0040AC7C                 push    eax             ; int
.text:0040AC7D                 push    ecx             ; int
.text:0040AC7E                 mov     ecx, [esp+68h+arg_0]
.text:0040AC82                 push    ecx             ; HDC
.text:0040AC83                 call    ds:StretchDIBits               //拉伸图片
.text:0040AC89                 push    edi
.text:0040AC8A                 call    Delete
```

查看 `lpBits` 的内容，记下里面的数据

跟踪到 `lpBits` 来源处：

`WinProc` (消息是 `WM_PAINT`) -> `SubDraw` -> `DrawBitMap` -> `StretchDIBits`

### `WinPRoc` 调用链

```
call    ds:GetDC
mov     edx, dword_4CBDD0   ;背景数据
mov     esi, eax
push    esi         ; hdc
push    edx         ; lpBit
call    SubDraw
mov     eax, hWnd
push    esi         ; hDC
push    eax         ; hWnd
call    ds:ReleaseDC
```

发现从绘图出发，很难找到解密入口，改成从文件读取时查找解密入口

## 改变思路，从文件读取中查找，从而直接找到解密处

首先查找文件读取函数

中断 `CreateFile` 函数，直到是 `BMP` 文件（这个要通过 `OD` 动态分析）

然后再从 `OD` 的堆栈窗口中找到上级函数

### Call stack of main thread

```
Address    Stack      Procedure / arguments                    Called from                     Frame
0012EF50   0041E16B   Includes kernel32.CreateFileA           .0041E169                   0012F62C
0012EF54   0012EF74     FileName = "pic\\mlogo.bmp"
0012EF58   80000000     Access = GENERIC_READ
0012EF5C   00000000     ShareMode = 0
0012EF60   00000000     pSecurity = NULL
0012EF64   00000003     Mode = OPEN_EXISTING
0012EF68   00000080     Attributes = NORMAL
0012EF6C   00000000     hTemplateFile = NULL
0012F07C   0040C17F   .0041E120                   .0040C17A
0012F0B4   00405D48   .0040C170                   .00405D43
```

然后在 `0041E169` 处中断

```
0041E14C   .  8B35 C0004300 mov     esi, dword ptr [<&KERNEL32.Creat>;  kernel32.CreateFileA
0041E152   .  6A 00        push    0                                        ; /hTemplateFile = NULL
0041E154   .  68 80000000  push    80                                       ; |Attributes = NORMAL
0041E159   .  6A 03        push    3                                        ; |Mode = OPEN_EXISTING
0041E15B   .  6A 00        push    0                                        ; |pSecurity = NULL
0041E15D   .  6A 00        push    0                                        ; |ShareMode = 0
0041E15F   .  68 00000080  push    80000000                                 ; |Access = GENERIC_READ
0041E164   .  8D4424 1C    lea     eax, dword ptr [esp+1C]                 ; |
0041E168   .  50          push    eax                                      ; |FileName
0041E169   .  FFD6        call    esi   ; CreateFile                    ; \\CreateFileA
```

先是 `pic\\mlog.bmp`，然后是 `bg\\mlog.bmp`。然后是 `parts\\mlog.bmp`

经过测试，发现这个函数，每次打开这些文件都失败。

这说明，解出来的 `BMP` 并不放在文件中。

查看目录下的小文件含有的信息，试着找到与 `pic\\mlog.bmp` 相关的内容。

发现目录下有以 `pic`, `bg`, `parts` 开头的文件。

逐个查看，发现，`NME` 扩展名中存的是 `BMP` 文件名。

`parts.nme` : (内容)

```
event.bmp hr.bmp logo_00.bmp logo_s_00.bmp mes.bmp mlogo.bmp name.bmp pph1_00.bmp pph2.bmp ppS1.bmp test04.bmp
test04c.bmp test04d.bmp test04e.bmp c_mink_00.bmp b201.bmp nowonsale02.bmp nowonsale01.bmp ?
```

所以肯定会打开 `parts.nme`

重启动程序，再次跟踪 `CreateFile` 直到出现 `parts.nme`

发现先打开 `parts.atm`, 再打开 `parts.nme`. 在之前是打开 `Pic` 和 `bg` 开头的。

所以认为 `ATM` 中存放的是配置，而 `NME` 是名称

## 从 OD 的堆栈窗口中找到上级函数

### Call stack of main thread

```
Address    Stack      Procedure / arguments                    Called from                     Frame
0012F05C   0040C33C   Includes kernel32.CreateFileA           .0040C33A
0012F060   0043CFB0     FileName = "parts.nme"
0012F064   80000000     Access = GENERIC_READ
0012F068   00000000     ShareMode = 0
0012F06C   00000000     pSecurity = NULL
0012F070   00000003     Mode = OPEN_EXISTING
0012F074   00000080     Attributes = NORMAL
0012F078   00000000     hTemplateFile = NULL
0012F0B4   00405D48   .0040C170                   .00405D43
```

在 `40c33A` 处中断：

```
0040C2FA   > \B8 67666666  mov     eax, 66666667
0040C2FF   .  F7EB         imul    ebx
0040C301   .  C1FA 03      sar     edx, 3
0040C304   .  8BCA         mov     ecx, edx
0040C306   .  68 B8D04300  push    0043D0B8                     ;  ASCII "parts"
0040C30B   .  C1E9 1F      shr     ecx, 1F
0040C30E   .  03D1         add     edx, ecx
0040C310   .  68 34984300  push    00439834                     ;  ASCII "%s.nme"
0040C315   .  68 B0CF4300  push    0043CFB0                     ;  ASCII "parts.nme"
0040C31A   .  895424 2C    mov     dword ptr [esp+2C], edx
0040C31E   .  FFD7         call    edi   ；wsprintf
0040C320   .  83C4 0C      add     esp, 0C
0040C323   .  6A 00        push    0
0040C325   .  68 80000000  push    80
0040C32A   .  6A 03        push    3
0040C32C   .  6A 00        push    0
0040C32E   .  6A 00        push    0
0040C330   .  68 00000080  push    80000000
0040C335   .  68 B0CF4300  push    0043CFB0                     ;  ASCII "parts.nme"
0040C33A   .  FFD5         call    ebp  ;createfile
```

然后分析堆栈

### Call stack of main thread

```
Address    Stack      Procedure / arguments                    Called from                     Frame
0012F0B4   00405D48   .0040C170                   .00405D43
```

`Frame_ack->GetBmpInfo`

发现这个来自一个线程

好像是用于进行帧处理的

然后分析下如何对这个文件操作的

```
0040C347   .  52          push    edx
0040C348   .  C74424 18 000>mov     dword ptr [esp+18], 0
0040C350   .  E8 3Bfdffff  call    0040C090     ；分析文件                   ;  DEncrypt
0040C355   .  83C4 04      add     esp, 4
0040C358   .  85C0         test    eax, eax    ;成功与否
0040C35A   .  7D 68        jge     short 0040C3C4
0040C35C   .  8B5C24 30    mov     ebx, dword ptr [esp+30]
0040C360   .  53          push    ebx
0040C361   .  68 B0CF4300  push    0043CFB0                     ;  ASCII "parts.nme"
0040C366   .  E8 A5fdffff  call    0040C110     ;err_show
```



```
0040C36B   .  83C4 08       add     esp, 8  
0040C36E   .  83FB 03       cmp     ebx, 3  
0040C371   .  7D 51        jge     short 0040C3C4  

跟踪 call    0040C090  

0040C090  /$  83EC 08       sub     esp, 8  
0040C093  |.  53           push    ebx  
0040C094  |.  55           push    ebp  
0040C095  |.  8B6C24 14    mov     ebp, dword ptr [esp+14]  
0040C099  |.  57           push    edi  
0040C09A  |.  6A 00        push    0                      ; /pFileSizeHigh = NULL  
0040C09C  |.  56           push    esi                    ; |hFile  
0040C09D  |.  FF15 08014300 call    dword ptr [<&KERNEL32.GetFileSiz>; \GetFileSize  
0040C0A3  |.  8BF8         mov     edi, eax  
0040C0A5  |.  8D5F FC      lea     ebx, dword ptr [edi-4]  
0040C0A8  |.  53           push    ebx  
0040C0A9  |.  C74424 1C 000>mov     dword ptr [esp+1C], 0  
0040C0B1  |.  E8 B75D0100  call    00421E6D               ;   new  
0040C0B6  |.  83C4 04      add     esp, 4  
0040C0B9  |.  6A 00        push    0                      ; /pOverlapped = NULL  
0040C0BB  |.  8D4C24 10    lea     ecx, dword ptr [esp+10]          ; |  
0040C0BF  |.  51           push    ecx                    ; |pBytesRead  
0040C0C0  |.  53           push    ebx                    ; |BytesToRead  
0040C0C1  |.  8B1D 10014300 mov     ebx, dword ptr [<&KERNEL32.ReadF>; |kernel32.ReadFile  
0040C0C7  |.  50           push    eax                    ; |Buffer 文件内容  
0040C0C8  |.  56           push    esi                    ; |hFile  
0040C0C9  |.  8945 00      mov     dword ptr [ebp], eax     ; |  
0040C0CC  |.  FFD3         call    ebx                    ; \ReadFile  
0040C0CE  |.  6A 00        push    0                      ; /pOverlapped = NULL  
0040C0D0  |.  8D5424 10    lea     edx, dword ptr [esp+10]          ; |  
0040C0D4  |.  52           push    edx                    ; |pBytesRead  
0040C0D5  |.  6A 04        push    4                      ; |BytesToRead = 4  
0040C0D7  |.  8D4424 1C    lea     eax, dword ptr [esp+1C]          ; |  
0040C0DB  |.  50           push    eax                    ; |BUFFER 解密号  
0040C0DC  |.  56           push    esi                    ; |hFile  
0040C0DD  |.  FFD3         call    ebx                    ; \ReadFile  
0040C0DF  |.  8B4D 00      mov     ecx, dword ptr [ebp]     ;  new address  
0040C0E2  |.  51           push    ecx  
0040C0E3  |.  8D57 FC      lea     edx, dword ptr [edi-4]  
0040C0E6  |.  8D4C24 1C    lea     ecx, dword ptr [esp+1C]  
0040C0EA  |.  E8 21FFFFFF  call    0040C010               ;  通过文件内容算出加密号  
0040C0EF  |.  8B4424 14    mov     eax, dword ptr [esp+14]  
0040C0F3  |.  8B4C24 1C    mov     ecx, dword ptr [esp+1C]  
0040C0F7  |.  83C4 04      add     esp, 4  
0040C0FA  |.  3BC1         cmp     eax, ecx               ;  比较两个加密号  
0040C0FC  |.  5F           pop     edi  
0040C0FD  |.  5D           pop     ebp  
0040C0FE  |.  5B           pop     ebx  
0040C0FF  |.  74 07        je      short 0040C108          
0040C101  |.  83C8 FF      or      eax, FFFFFFFFF           ；出错  
0040C104  |.  83C4 08      add     esp, 8  
0040C107  |.  C3           retn  
0040C108  |>  8BC2         mov     eax, edx               ;加密号位置  
0040C10A  |.  83C4 08      add     esp, 8  
0040C10D  \\.  C3           retn  
```

分析后得出：

取出文件内容直到倒数第 4 个字节。最后 4 个字节再独立取出来。

我们认为这 4 个字节用于文件校验。称为加密号

接下来应该对 `NME` 文件进行处理，我们猜想可能是查 `mlog.bmp` 对应的数据

```
0040C3F0   >  8B53 F8      mov     edx, dword ptr [ebx-8]  
0040C3F3   . |035424 14    add     edx, dword ptr [esp+14]          ;  取下一个 OBJ  
0040C3F7   . |68 A8CE4300  push    0043CEA8               ;  ASCII "mlogo.bmp"  
0040C3FC   . |52           push    edx                    ;  NME 中的文件名  
0040C3FD   . |E8 EF5F0100  call    004223F1               ;  Find Obj  
0040C402   . |83C4 08      add     esp, 8  
0040C405   . |85C0         test    eax, eax               ;  0:found  
0040C407   . |0F85 B9000000 jnz     0040C4C6  
0040C40D   . |68 B8D04300  push    0043D0B8               ;  ASCII "parts"  
0040C412   . |68 3C984300  push    0043983C               ;  ASCII "%s.det"  
0040C417   . |68 B0CF4300  push    0043CFB0               ;  ASCII "parts.nme"  
0040C41C   . |FFD7         call    edi                    ； wspintf  
0040C41E   . |83C4 0C      add     esp, 0C  
0040C421   > |6A 00        push    0  
0040C423   . |68 80000000  push    80  
0040C428   . |6A 03        push    3  
0040C42A   . |6A 00        push    0  
0040C42C   . |6A 00        push    0  
0040C42E   . |68 00000080  push    80000000  
0040C433   . |68 B0CF4300  push    0043CFB0               ;  ASCII "parts.nme"  
0040C438   . |FFD5         call    ebp                    ; open this file  
0040C43A   . |6A 00        push    0                      ; /Origin = FILE\_BEGIN  
0040C43C   . |8BF8         mov     edi, eax               ; |  
0040C43E   . |8B43 FC      mov     eax, dword ptr [ebx-4]     ; |  
0040C441   . |6A 00        push    0                      ; |pOffsetHi = NULL  
0040C443   . |50           push    eax                    ; |OffsetLo  
0040C444   . |57           push    edi                    ; |hFile  
0040C445   . |FF15 0C014300 call    dword ptr [<&KERNEL32.SetFilePoi>; \SetFilePointer  
0040C44B   . |8B33         mov     esi, dword ptr [ebx]  
0040C44D   . |56           push    esi  
0040C44E   . |E8 1A5A0100  call    00421E6D               ; new  
0040C453   . |83C4 04      add     esp, 4  
0040C456   . |6A 00        push    0                      ; /pOverlapped = NULL  
0040C458   . |8D4C24 2C    lea     ecx, dword ptr [esp+2C]          ; |  
0040C45C   . |51           push    ecx                    ; |pBytesRead  
0040C45D   . |56           push    esi                    ; |BytesToRead  
0040C45E   . |8BE8         mov     ebp, eax               ; |  
0040C460   . |55           push    ebp                    ; |Buffer  
0040C461   . |57           push    edi                    ; |hFile  
0040C462   . |FF15 10014300 call    dword ptr [<&KERNEL32.ReadFile>] ; \ReadFile  
0040C468   . |57           push    edi                    ; /hObject  
0040C469   . |FF15 D0004300 call    dword ptr [<&KERNEL32.CloseHandl>; \CloseHandle  
```

查到 `mlog.bmp` 时，除了返回 `TRUE`，还需要一些相关信息。这些信息可能记录在内存中。或一个对象中

要么是 `OBJ` 对象。要么是全局变量。所以需要跟踪 `call    004223F1`

发现内部复杂，找不到有用的东西。暂时不管它。

我们又发现在不断的查找过程当中有个值在累加。是 `EBX`

分析后面的代码发现：

```
0040C4C6   > \8B4424 18    mov     eax, dword ptr [esp+18]  
0040C4CA   .  8B4C24 20    mov     ecx, dword ptr [esp+20]  
0040C4CE   .  40           inc     eax  
0040C4CF   .  83C3 14      add     ebx, 14  
0040C4D2   .  3BC1         cmp     eax, ecx  
0040C4D4   .  894424 18    mov     dword ptr [esp+18], eax  
0040C4D8   .^ 0F8C 12FFFFFF jl      0040C3F0  
```

这说明信息并不是通过 `call 4223f1` 来得到的，而只是简单的一一对应方式。

`mblog.bmp` 是 `parts.nme` 中的第 6 个，按 0 算，就是 `5*14h=64h`; `14h` 一定是 `BMP` 的信息结构。

`64h+8h` 是在 `parts.atm` 中的位置。也就是说，`ATM` 文件中存放的是 `DEC` 文件中资源的定位信息。

`parts.atm` 中头 8 个字节(值为 0)可能有另用。

我们猜想接下来就是查看 `parts.det`，并得到 `mlog.bmp` 对应的资源数据。

```
0040C40D   .  68 B8D04300   push    0043D0B8               ;  ASCII "parts"  
0040C412   .  68 3C984300   push    0043983C               ;  ASCII "%s.det"  
0040C417   .  68 B0CF4300   push    0043CFB0               ;  ASCII "parts.det"  
0040C41C   .  FFD7          call    edi  
0040C41E   .  83C4 0C       add     esp, 0C  
```



### 汇编代码分析

```
0040C421   >  6A 00             push    0  
0040C423   .  68 80000000       push    80  
0040C428   .  6A 03             push    3  
0040C42A   .  6A 00             push    0  
0040C42C   .  6A 00             push    0  
0040C42E   .  68 00000080       push    80000000  
0040C433   .  68 B0CF4300       push    0043CFB0                 ;  ASCII "parts.det"  
0040C438   .  FFD5             call    ebp                      ; CreateFile  
0040C43A   .  6A 00             push    0                        ; /Origin = FILE_BEGIN  
0040C43C   .  8BF8             mov     edi, eax                 ; hFile  
0040C43E   .  8B43 FC          mov     eax, dword ptr [ebx-4]     ; 定位点 28f9d  
0040C441   .  6A 00             push    0                        ; |pOffsetHi = NULL  
0040C443   .  50                push    eax                      ; |OffsetLo  
0040C444   .  57                push    edi                      ; |hFile  
0040C445   .  FF15 0C014300    call    dword ptr [<&KERNEL32.SetFilePointer>] ; SetFilePointer  
0040C44B   .  8B33             mov     esi, dword ptr [ebx]       ; SizeOfFile 7179  
0040C44D   .  56                push    esi  
0040C44E   .  E8 1A5A0100      call    00421E6D                 ; new  
0040C453   .  83C4 04          add     esp, 4  
0040C456   .  6A 00             push    0                        ; /pOverlapped = NULL  
0040C458   .  8D4C24 2C        lea     ecx, dword ptr [esp+2C]    ; |  
0040C45C   .  51                push    ecx                      ; |pBytesRead  
0040C45D   .  56                push    esi                      ; |BytesToRead  
0040C45E   .  8BE8             mov     ebp, eax                 ; |  
0040C460   .  55                push    ebp                      ; |Buffer  
0040C461   .  57                push    edi                      ; |hFile  
0040C462   .  FF15 10014300    call    dword ptr [<&KERNEL32.ReadFile>] ; ReadFile  
0040C468   .  57                push    edi                      ; /hObject  
0040C469   .  FF15 D0004300    call    dword ptr [<&KERNEL32.CloseHandle>] ; CloseHandle  
```

得到的 `BUFFER` 中就是 `BMP` 对应的加密数据。

试着构造下资源的结构：

```c
struct BmpVerData //加密的 BMP 数据
{
    DWORD  SizeOfBmpFile;
    DWORD  OffsetOfDet;     //资源偏移
    DWORD  Un_1;            //表示未知，以后再改
    DWORD  Un_2;
    DWORD  Un_3;
};
```

### 三、资源分析

继续看下面的代码

```
.text:0040C46F                     push    ebp  
.text:0040C470                     mov     edx, esi  
.text:0040C472                     lea     ecx, [esp+50h+@@pr_Key] ; 用来校验  
.text:0040C476                     mov     [esp+50h+@@pr_Key], 0  
.text:0040C47E                     call    40c010                 ;CRC_Verify  
.text:0040C47E  

.text:0040C483                     mov     edx, [esp+50h+@@pr_Key]  
.text:0040C487                     mov     eax, [ebx+4]  
.text:0040C48A                     add     esp, 4  
.text:0040C48D                     cmp     edx, eax  
.text:0040C48F                     jz      short CleanMem  
.text:0040C48F  

.text:0040C491  

.text:0040C491 err:  
.text:0040C491                     push    ebp  
.text:0040C492                     call    Delete  
.text:0040C492  

.text:0040C497                     mov     esi, [esp+50h+@@if_use]  
.text:0040C49B                     push    esi  
.text:0040C49C                     push    offset FileName  
.text:0040C4A1                     call    Err_Show  
```

这些已加中文名称的 `CALL` 都是 `OD` 动态分析出来的

`40c010` 前面已看到过了，是用来校验的

所以可以断定 `40c491` 处一定是错误显示。

我们跳过，看 `CleanMem`:

```
.text:0040C500 CleanMem:                       ; CODE XREF: GetBmpInfo+31Fj  
.text:0040C500                     mov     eax, [esp+4Ch+@@NumberOfBmp]  
.text:0040C504                     mov     ecx, [esp+4Ch+@@ObjForDeCode]  
.text:0040C508                     mov     ebx, [esp+4Ch+var_C]  
.text:0040C50C                     lea     eax, [eax+eax*4]     ;一个资源数据的大小  
.text:0040C50F                     mov     eax, [ecx+eax*4+10h] ;10h 处为数据的起始。前 10h 内容为文件头 7113  
                                                    ;eax：数据大小。  
                                                    ;  
.text:0040C513                     push    esi  
.text:0040C514                     push    ebp  
.text:0040C515                     call    UnEntry  
.text:0040C515  

.text:0040C51A                     push    ebp  
.text:0040C51B                     mov     esi, eax  
.text:0040C51D                     call    Delete  
.text:0040C51D  

.text:0040C522                     mov     edx, [esp+58h+@@ObjForDeCode]  
.text:0040C526                     push    edx  
.text:0040C527                     call    Delete  
.text:0040C527  

.text:0040C52C                     mov     eax, [esp+5Ch+var_38]  
.text:0040C530                     push    eax                      ;NME 数据删除  
.text:0040C531                     call    Delete  
.text:0040C531  

.text:0040C536                     add     esp, 14h  
.text:0040C539                     pop     ebp  
.text:0040C53A                     pop     edi  
.text:0040C53B                     mov     eax, esi  
.text:0040C53D                     pop     esi  
.text:0040C53E                     pop     ebx  
.text:0040C53F                     add     esp, 28h  
.text:0040C542                     retn  
```

后面有很多相同的 `CALL DELETE`，我们在执行后发现，`PUSH` 入的地址处数据无效，所以认为是 `DELETE`

有理由认为我们已经到了解密代码处了。因为再后面就把不用的对象给释放了

跟踪 `call UnEntry`

```
0040BF80  /$  57                push    edi  
0040BF81  |.  50                push    eax                       ; 7113  
0040BF82  |.  E8 E65E0100       call    00421E6D                   ;  new  
0040BF87  |.  8B4C24 10         mov     ecx, dword ptr [esp+10]  
0040BF8B  |.  8903             mov     dword ptr [ebx], eax  
0040BF8D  |.  83C4 04          add     esp, 4  
0040BF90  |.  33FF             xor     edi, edi  
0040BF92  |.  33C0             xor     eax, eax  
0040BF94  |.  85C9             test    ecx, ecx  
0040BF96  |.  7E 6B            jle     short 0040C003  
0040BF98  |.  55                push    ebp  
0040BF99  |.  56                push    esi  
0040BF9A  |.  8D9B 00000000     lea     ebx, dword ptr [ebx]  
0040BFA0  |>  8B5424 10         /mov     edx, dword ptr [esp+10]  
0040BFA4  |.  8A0C17           |mov     cl, byte ptr [edi+edx]  
0040BFA7  |.  80F9 FF          |cmp     cl, 0FF                   ;FF 时会重复前面的数据  
0040BFAA  |.  75 46            |jnz     short 0040BFF2  
0040BFAC  |.  8A4C17 01        |mov     cl, byte ptr [edi+edx+1]  
0040BFB0  |.  47               |inc     edi  
0040BFB1  |.  80F9 FF          |cmp     cl, 0FF                   ;  
0040BFB4  |.  75 08            |jnz     short 0040BFBE  
0040BFB6  |.  8B0B             |mov     ecx, dword ptr [ebx]  
0040BFB8  |.  C60408 FF       |mov     byte ptr [eax+ecx], 0FF  
0040BFBC  |.  EB 39            |jmp     short 0040BFF7  
0040BFBE  |>  0FB6C9          |movzx   ecx, cl  
0040BFC1  |.  8BD1             |mov     edx, ecx  
0040BFC3  |.  C1EA 02          |shr     edx, 2  
0040BFC6  |.  42               |inc     edx                       ;偏移量  
0040BFC7  |.  81E1 03000080    |and     ecx, 80000003  
0040BFCD  |.  79 05            |jns     short 0040BFD4  
0040BFCF  |.  49               |dec     ecx  
0040BFD0  |.  83C9 FC          |or      ecx, FFFFFFFC  
0040BFD3  |.  41               |inc     ecx  
0040BFD4  |>  83C1 03          |add     ecx, 3  
0040BFD7  |.  85C9             test    ecx, ecx  
0040BFD9  |.  7E 1D            |jle     short 0040BFF8  
0040BFDB  |.  8BE9             |mov     ebp, ecx  
0040BFDD  |.  8D49 00          |lea     ecx, dword ptr [ecx]  
0040BFE0  |>  8B33             |/mov     esi, dword ptr [ebx]  
0040BFE2  |.  8BCE             ||mov     ecx, esi  
0040BFE4  |.  2BCA             ||sub     ecx, edx  
0040BFE6  |.  8A0C01           ||mov     cl, byte ptr [ecx+eax]  
0040BFE9  |.  880C06           ||mov     byte ptr [esi+eax], cl  
0040BFEC  |.  40               ||inc     eax  
0040BFED  |.  4D               ||dec     ebp  
0040BFEE  |.^ 75 F0            |\jnz     short 0040BFE0  
0040BFF0  |.  EB 06            |jmp     short 0040BFF8  
0040BFF2  |>  8B13             |mov     edx, dword ptr [ebx]  
```



# 逆向分析与解密逻辑

## 汇编指令流

0040BFF4  |.  880C10        |mov     byte ptr [eax+edx], cl  
0040BFF7  |>  40            |inc     eax  
0040BFF8  |>  8B4C24 14     |mov     ecx, dword ptr [esp+14]  
0040BFFC  |.  47            |inc     edi  
0040BFFD  |.  3BF9          |cmp     edi, ecx  
0040BFFF  |.^ 7C 9F         |jl      short 0040BFA0  
0040C001  |.  5E            |pop     esi  
0040C002  |.  5D            |pop     ebp  
0040C003  |>  5F            |pop     edi  
0040C004  |\. C3            |retn  

## 一级解密反 C 代码

分析后给出这一级解密的反 C 代码：

```c
int k=0;
for(i=0;i<DetSize;i++)
{
   if(buf[i]==0xFF)
   {
     k=i;
     i++;
     if(buf[i]!=0xff)
    {
         PreOffset=buf[i]>>2;
     PreOffset++;
     preNum=buf[i] & 0x80000003;
     if(preNum<0)
     {
         preNum--;
       preNum |=0xfffffffc;
       preNum++;
     }
     preNum+=3;
     if(preNum>0)
     {
           for(;preNum>0;preNum--)
       {
             nbuf[j]=nbuf[j-PreOffset];
       j++;
       }
       j--;
     }
     else  
     {
       j--;
     }
    }
    else
    {
        nbuf[j]=0xff;
    }
   }
   else
   {
     nbuf[j]=buf[i];
   }
j++;
}
```

紧接着发现这只是一级解密，数据还是不可直接读取。

## 二级解密分析

我们猜想数据还会继续解密，但是在它的上级函数中直接返回，发现位于 `FindBmp` 中。

```asm
.text:004183E0 FindBmp           proc near             ; CODE XREF: Frame_ack+3EBp
.text:004183E0
.text:004183E0 @@ptr_BmpInfo   = dword ptr -10Ch
.text:004183E0 var_108         = byte ptr -108h
.text:004183E0 var_4           = dword ptr -4
.text:004183E0 arg_0           = dword ptr 4
.text:004183E0 arg_4           = dword ptr 8
.text:004183E0 arg_8           = dword ptr 0Ch
.text:004183E0
.text:004183E0                 sub     esp, 10Ch
.text:004183E6                 mov     eax, G_Key
.text:004183EB                 xor     eax, [esp+10Ch]
.text:004183F2                 push    ebx
.text:004183F3                 push    ebp
.text:004183F4                 mov     ebp, [esp+114h+arg_8]
.text:004183FB                 push    esi
.text:004183FC                 mov     esi, ecx        ; bmp type
.text:004183FE                 push    esi
.text:004183FF                 mov     [esp+11Ch+var_4], eax
.text:00418406                 mov     ebx, edx
.text:00418408                 lea     eax, [esp+11Ch+var_108]
.text:0041840C                 push    offset s_S_bmp_0 ; "%s.bmp"
.text:00418411                 push    eax             ; LPSTR
.text:00418412                 mov     [ebx+4A4h], ebp
.text:00418418                 mov     [ebx+4A0h], edi
.text:0041841E                 call    ds:wsprintfA
.text:00418424                 lea     ecx, [esp+124h+@@ptr_BmpInfo]
.text:00418428                 push    ecx
.text:00418429                 lea     edx, [esp+128h+var_108]
.text:0041842D                 push    edx
.text:0041842E                 mov     [esp+12Ch+@@ptr_BmpInfo], 0
.text:00418436                 call    GetBmpInfo                 ;回到这里
.text:00418436
.text:0041843B                 add     esp, 14h        ; BmpInfo 还是加密的
.text:0041843E                 test    eax, eax        ; 数据的大小
.text:00418440                 jg      short BmpDecode
.text:00418440
.text:00418442
.text:00418442 NotFound:
.text:00418442                 push    esi
.text:00418443                 push    offset s_Gtg@gcglbuS_0 ; "%s"
.text:00418448                 push    offset byte_4C29D0 ; LPSTR
.text:0041844D                 call    ds:wsprintfA
.text:00418453                 push    offset s_KgvUVNuv ; ""
.text:00418458                 mov     ebx, offset byte_4C29D0
.text:0041845D                 call    sub_41BBC0
.text:0041845D
.text:00418462                 add     esp, 10h
.text:00418465                 jmp     short loc_4184E6
.text:00418465
.text:00418467 ; ---------------------------------------------------------------------------
.text:00418467
.text:00418467 BmpDecode:                 ; CODE XREF: FindBmp+60j
.text:00418467                 mov     esi, [esp+118h+@@ptr_BmpInfo]
.text:0041846B                 test    esi, esi
.text:0041846D                 jz      short loc_4184E6
.text:0041846D
.text:0041846F                 mov     cl, [esi]
.text:00418471                 cmp     cl, 'F'
.text:00418474                 jnz     short loc_4184B9
.text:00418474
.text:00418476                 mov     al, [esi+1]
.text:00418479                 and     al, '_'
.text:0041847B                 cmp     al, 'D'
.text:0041847D                 jnz     short loc_41849A
.text:0041847D
.text:0041847F
.text:0041847F FD_Type:                              ；bmp 类型
.text:0041847F                 mov     eax, [esp+118h+arg_4]
.text:00418486                 mov     ecx, [esp+118h+arg_0]
.text:0041848D                 push    edi
.text:0041848E                 push    ebp
.text:0041848F                 push    eax
.text:00418490                 push    ecx
.text:00418491                 mov     eax, esi                              ;资源对象
.text:00418493                 call    BmpDecode
.text:00418493
.text:00418498                 jmp     short loc_4184DD
.text:00418498
.text:0041849A ; ---------------------------------------------------------------------------
.text:0041849A
.text:0041849A loc_41849A:                 ; CODE XREF: FindBmp+9Dj
.text:0041849A                 cmp     al, 'E'
.text:0041849C                 jnz     short loc_4184B9
.text:0041849C
.text:0041849E                 mov     edx, [esp+118h+arg_4]
.text:004184A5                 mov     eax, [esp+118h+arg_0]
.text:004184AC                 push    edi
.text:004184AD                 push    ebp
.text:004184AE                 push    edx
.text:004184AF                 push    eax
.text:004184B0                 mov     eax, esi
.text:004184B2                 call    sub_417B70
.text:004184B2
.text:004184B7                 jmp     short loc_4184DD
.text:004184B7
.text:004184B9 ; ---------------------------------------------------------------------------
.text:004184B9
.text:004184B9 loc_4184B9:                 ; CODE XREF: FindBmp+94j
.text:004184B9                             ; FindBmp+BCj
.text:004184B9                 cmp     cl, 'B'
.text:004184BC                 jnz     short loc_4184DD
.text:004184BC
.text:004184BE                 cmp     byte ptr [esi+1], 'M'
.text:004184C2                 jnz     short loc_4184DD
.text:004184C2
.text:004184C4                 mov     ecx, [esp+118h+arg_4]
.text:004184CB                 mov     edx, [esp+118h+arg_0]
.text:004184D2                 push    edi
.text:004184D3                 push    ebp
.text:004184D4                 push    esi
.text:004184D5                 push    ecx
.text:004184D6                 push    edx
.text:004184D7                 push    ebx
.text:004184D8                 call    sub_417EC0
.text:004184D8
.text:004184DD
.text:004184DD loc_4184DD:                 ; CODE XREF: FindBmp+B8j
.text:004184DD                             ; FindBmp+D7j
.text:004184DD                             ; FindBmp+DCj
.text:004184DD                             ; FindBmp+E2j
.text:004184DD                 push    esi
.text:004184DE                 call    Delete
.text:004184DE
.text:004184E3                 add     esp, 4
.text:004184E3
.text:004184E6
.text:004184E6 loc_4184E6:                 ; CODE XREF: FindBmp+85j
```



### FindBmp 尾部与二级解码逻辑

```
.text:004184E6 ; FindBmp+8Dj  
.text:004184E6 mov ecx, [esp+118h+var_4]  
.text:004184ED xor ecx, [esp+118h]  
.text:004184F4 pop esi  
.text:004184F5 pop ebp  
.text:004184F6 pop ebx  
.text:004184F7 call Err_Check ; 校对  
.text:004184FC add esp, 10Ch  
.text:00418502 retn 0Ch  
.text:00418502 FindBmp endp  
```

分析以上代码得出：

接下去会根据加密文件类型 ID（`BMP` 为 `FD`），来具体的二级解码。

要得到真正的位图数据，只能进入 `call BmpDecode`。

`BmpObj` 中已存在相关信息。

### BmpDecode 参数准备

```
.text:00416E50 sub esp, 24h  
.text:00416E53 push ebp  
.text:00416E54 push esi  
.text:00416E55 mov esi, eax ; BmpObj  
.text:00416E57 movzx ebp, word ptr [esi+4] ; BmpWidth  
.text:00416E5B movsx eax, byte ptr [esi+2] ; BmpCell 调色板面数  
.text:00416E5F push edi  
.text:00416E60 movzx edi, word ptr [esi+6] ; BmpHeight  
.text:00416E64 mov ecx, edi  
.text:00416E66 imul ecx, ebp  
.text:00416E69 lea edx, ds:40h[ecx*4]  
.text:00416E70 push edx  
.text:00416E71 mov [esp+34h+@@BmpSize], eax ; 对齐后的 BMP 大小 1d4c40h  
.text:00416E75 call New  
.text:00416E7A push eax ; 就是 BMP 位图对象  
.text:00416E7B mov [esp+38h+@@BmpObj], eax  
.text:00416E7F mov eax, [esp+38h+@@BmpSize]  
.text:00416E83 push esi  
.text:00416E84 push eax  
.text:00416E85 push edi  
.text:00416E86 push ebp  
.text:00416E87 call UnEntryBmpObj ; 填充 BMP 位图对象  
.text:00416E8C mov eax, [esp+48h+arg_0]  
.text:00416E90 xor edx, edx  
.text:00416E92 add esp, 18h  
.text:00416E95 cmp eax, edx  
.text:00416E97 mov [esp+30h+var_24], edx  
.text:00416E9B mov [esp+30h+var_1C], edx  
.text:00416E9F mov [esp+30h+var_20], ebp  
.text:00416EA3 mov [esp+30h+var_18], edi  
.text:00416EA7 jge short loc_416EB1  
```

一般情况不要跟踪到 `CALL` 中，经过执行 `call UnEntryBmpObj` 发现参数 `BMP` 位图对象的内容已被填充。

所以把这个 `CALL` 取名为 `UnEntryBmpObj`。

### UnEntryBmpObj 内部分析

跟踪这个 `call UnEntryBmpObj`：

发现这个函数太过复杂。

为了直接找到位图的填充代码，直接在 `BMP` 位图对象的内容处下硬件断点：

找到下面的代码：

```
loc_416D75: ; CODE XREF: UnEntryBmpObj+411j  
.text:00416D75 movzx edi, byte ptr [esi]  
.text:00416D78 movzx edx, byte ptr [esi+1]  
.text:00416D7C movzx ecx, byte ptr [esi+2]  
.text:00416D80 shl edi, 8  
.text:00416D83 add edi, edx  
.text:00416D85 shl edi, 8  
.text:00416D88 add edi, ecx  
.text:00416D8A mov cl, byte_4C2248[eax]  
.text:00416D90 shl edi, 8  
.text:00416D93 xor edi, 80000080h  
.text:00416D99 shr edi, cl  
.text:00416D9B shl eax, 10h  
.text:00416D9E mov edx, edi  
.text:00416DA0 shr edx, 8  
.text:00416DA3 xor edx, eax  
.text:00416DA5 mov eax, [esp+104h+var_F4]  
.text:00416DA9 mov [eax], edx ; 位图数据填充处  
.text:00416DAB add eax, 4  
.text:00416DAE and edi, 0FFh  
.text:00416DB4 add esi, 3  
.text:00416DB7 loc_416DB7: ; CODE XREF: UnEntryBmpObj+5F3j  
.text:00416DB7 mov [esp+104h+var_F4], eax  
.text:00416DBB mov [esp+104h+var_F0], edi  
.text:00416DBF loc_416DBF: ; CODE XREF: UnEntryBmpObj+3BCj  
.text:00416DBF ; UnEntryBmpObj+3E2j  
.text:00416DBF mov ebx, [esp+104h+var_F4]  
.text:00416DC3 loc_416DC3: ; CODE XREF: UnEntryBmpObj+3D7j  
.text:00416DC3 ; UnEntryBmpObj+409j  
.text:00416DC3 cmp ebx, [esp+104h+var_E8]  
.text:00416DC7 jb loc_4168C0  
```

#### 数据压缩与查找表分析

```
00416B42 . 8B5C24 10 mov ebx, dword ptr [esp+10]  
00416B46 > 8B10 mov edx, dword ptr [eax]  
00416B48 . 8913 mov dword ptr [ebx], edx ; 数据相同的情况  
```

也就是说数据被压缩了。

只有 `esi` 是对源数据定位的。

```
00416CE5 > /0FB606 movzx eax, byte ptr [esi]  
00416CE8 . |0FB61445 F016>movzx edx, byte ptr [eax*2+4C16F0]  
00416CF0 . |0FB63C45 F116>movzx edi, byte ptr [eax*2+4C16F1]  
00416CF8 . |46 inc esi  
```

所以，源数据中存在的是 `4C16F0` 的偏移。实际数据在 `4c16f0` 中的。

所以要查看 `4c16f0` 中的数据是何时写入的，这是张信息表。

查看这张表发现是普通的数据表，从 `0001h` 到 `00FFh` 的奇数表。

所以 `BMP` 数据是被压缩和加密处理过的。加密的方式是通过表来实现的。

由于这个函数特复杂，需进一步分析。

### 生成表 G_UnDeTable

```
.text:004167A0 @@CreateNewTable: ; CODE XREF: UnEntryBmpObj+39j  
.text:004167A0 cmp eax, 1  
.text:004167A3 jnz short loc_4167AB  
.text:004167A5 mov dword ptr [esp+104h+@@UnDeTable_1+4], ecx ; 26  
.text:004167A9 jmp short loc_4167B6  
.text:004167AB ; ---------------------------------------------------------------------------  
.text:004167AB loc_4167AB: ; CODE XREF: UnEntryBmpObj+23j  
.text:004167AB mov esi, ds:G_UnDeTable[eax*4]  
.text:004167B2 mov dword ptr [esp+esi*4+104h+@@UnDeTable_1], eax ; 数组,4*26h  
```

生成表 `G_UnDeTable[4*26h]`

```
0012EEA0 02 00 00 00 26 00 00 00 ...&...  
0012EEA8 00 00 00 00 03 00 00 00 .......  
0012EEB0 04 00 00 00 05 00 00 00 ......  
0012EEB8 14 00 00 00 16 00 00 00 ......  
0012EEC0 15 00 00 00 17 00 00 00 ......  
0012EEC8 06 00 00 00 07 00 00 00 ......  
0012EED0 08 00 00 00 09 00 00 00 .......  
0012EED8 0A 00 00 00 0B 00 00 00 .......  
0012EEE0 18 00 00 00 19 00 00 00 ......  
0012EEE8 1A 00 00 00 1B 00 00 00 ......  
0012EEF0 1C 00 00 00 1D 00 00 00 ......  
0012EEF8 0C 00 00 00 0D 00 00 00 ........  
0012EF00 0E 00 00 00 0F 00 00 00 ......  
0012EF08 10 00 00 00 11 00 00 00 ......  
0012EF10 12 00 00 00 13 00 00 00 ......  
0012EF18 1E 00 00 00 1F 00 00 00 ......  
0012EF20 20 00 00 00 21 00 00 00 ...!...  
0012EF28 22 00 00 00 23 00 00 00 "...#...  
0012EF30 24 00 00 00 25 00 00 00 $...%...  
```

### 表引用与维度计算

```
.text:004167C0 loc_4167C0: ; CODE XREF: UnEntryBmpObj+101j  
.text:004167C0 mov ecx, ds:G_UnTable_1[eax]  
.text:004167C6 imul ecx, edx ; edx:BmpWidth  
.text:004167C9 add ecx, ds:G_UnTable_0[eax] ; 0  
.text:004167CF mov dword ptr [esp+eax+104h+@@NT1_Dim], ecx  
.text:004167D3 mov ecx, ds:(G_UnTable_1+4)[eax] ; 1  
.text:004167D9 mov ebp, ds:(G_UnTable_0+4)[eax]  
.text:004167DF imul ecx, edx  
.text:004167E2 add ecx, ebp  
.text:004167E4 mov edi, ds:(G_UnTable_0+8)[eax] ; 2  
```



```
.text:004167EA  mov dword ptr [esp+eax+104h+@@NT1_Dim+4], ecx ; 1

.text:004167EE  mov ecx, ds:(G_UnTable_1+8)[eax]

.text:004167F4  imul ecx, edx

.text:004167F7  add ecx, edi

.text:004167F9  mov esi, ds:(G_UnTable_0+0Ch)[eax] ; 3

.text:004167FF  mov dword ptr [esp+eax+104h+@@NT1_Dim+8], ecx ; 2

.text:00416803  mov ecx, ds:(G_UnTable_1+0Ch)[eax]

.text:00416809  imul ecx, edx

.text:0041680C  mov ebp, ds:(G_UnTable_0+10h)[eax] ; 4

.text:00416812  add ecx, esi

.text:00416814  mov dword ptr [esp+eax+104h+@@NT1_Dim+0Ch], ecx ; 3

.text:00416818  mov ecx, ds:(G_UnTable_1+10h)[eax]

.text:0041681E  imul ecx, edx

.text:00416821  mov edi, ds:(G_UnTable_0+14h)[eax] ; 5

.text:00416827  add ecx, ebp

.text:00416829  mov dword ptr [esp+eax+104h+@@NT1_Dim+10h], ecx ; 4

.text:0041682D  mov ecx, ds:(G_UnTable_1+14h)[eax]

.text:00416833  imul ecx, edx

.text:00416836  mov esi, ds:(G_UnTable_0+18h)[eax] ; 6

.text:0041683C  add ecx, edi

.text:0041683E  mov dword ptr [esp+eax+104h+@@NT1_Dim+14h], ecx ; 5

.text:00416842  mov ecx, ds:(G_UnTable_1+18h)[eax]

.text:00416848  mov ebp, ds:(G_UnTable_0+1Ch)[eax] ; 7

.text:0041684E  imul ecx, edx

.text:00416851  add ecx, esi

.text:00416853  mov dword ptr [esp+eax+104h+@@NT1_Dim+18h], ecx ; 6

.text:00416857  mov ecx, ds:(G_UnTable_1+1Ch)[eax]

.text:0041685D  mov edi, ds:(G_UnTable_0+20h)[eax] ; 8

.text:00416863  imul ecx, edx

.text:00416866  add ecx, ebp

.text:00416868  mov dword ptr [esp+eax+104h+@@NT1_Dim+1Ch], ecx ; 7

.text:0041686C  mov ecx, ds:(G_UnTable_1+20h)[eax]

.text:00416872  imul ecx, edx

.text:00416875  add ecx, edi

.text:00416877  mov dword ptr [esp+eax+104h+@@NT1_Dim+20h], ecx ; 8

.text:0041687B  add eax, 24h

.text:0041687E  cmp eax, 48h ;48H 大小

.text:00416881  jl loc_4167C0
```

这时出现了两张表。`EDX` 都是 `BmpWidth`；

分析后，得出：

```
for(i=0;i<9;i++)

{

@@NT1_Dim[i]=(G_UnTable_1[i] * BmpWidth)+G_UnTable_0[i];

}
```

产生的表为：`NT1_Dim[4*12h]`（`BmpWidth=320h`）

```
0012EE58  FF FF FF FF E0 FC FF FF   帱

0012EE60  E1 FC FF FF DF FC FF FF   狳唿

0012EE68  C0 F9 FF FF FE FF FF FF   砾?

0012EE70  A0 F6 FF FF FD FF FF FF   ?

0012EE78  80 F3 FF FF FC FF FF FF   ??

0012EE80  60 F0 FF FF FB FF FF FF   `??

0012EE88  40 ED FF FF FA FF FF FF   @??

0012EE90  20 EA FF FF F9 FF FF FF    ??

0012EE98  00 E7 FF FF F8 FF FF FF   .??
```

```
.text:00416887  imul edx, [esp+104h+BmpHeight]     ;BmpWidth*BmpHeight

.text:0041688F  mov esi, [esp+104h+BmpObj]

.text:00416896  lea eax, [ebx+edx*4]           ;到文件 TAIL

.text:00416899  add esi, 10h

.text:0041689C  cmp ebx, eax                 ;相同表示图片 0 字节

.text:0041689E  mov edi, 80h

.text:004168A3  mov [esp+104h+@@BmpObj_Offset10], esi

.text:004168A7  mov [esp+104h+var_F0], edi

.text:004168AB  mov [esp+104h+@@UdForBmp], ebx

.text:004168AF  mov [esp+104h+@@BmpSize], edx

.text:004168B3  mov [esp+104h+@@BmpTail], eax

.text:004168B7  jnb loc_416DD1                ;跳到最后
```

接下来，又出现了两张新表。一张表有 `100h*2`（分奇数表和偶数表）

偶数表:`G_BmpInfoTable1[200h]`

```
004C14F0  01 00 01 02 01 04 01 06 01 08 01 0A 01 0C 01 0E   ?Ё?????

004C1500  01 10 01 12 01 14 01 16 01 18 01 1A 01 1C 01 1E   ????????

004C1510  01 20 01 22 01 24 01 26 01 28 01 2A 01 2C 01 2E   ????????

004C1520  01 30 01 32 01 34 01 36 01 38 01 3A 01 3C 01 3E   、???????

004C1530  01 40 01 42 01 44 01 46 01 48 01 4A 01 4C 01 4E   ???????丁

004C1540  01 50 01 52 01 54 01 56 01 58 01 5A 01 5C 01 5E   刁吁嘁威封币

004C1550  01 60 01 62 01 64 01 66 01 68 01 6A 01 6C 01 6E   态搁

004C1560  01 70 01 72 01 74 01 76 01 78 01 7A 01 7C 01 7E   瘁码

004C1570  01 80 01 82 01 84 01 86 01 88 01 8A 01 8C 01 8E   老舁萁谁

004C1580  01 90 01 92 01 94 01 96 01 98 01 9A 01 9C 01 9E   送阁

004C1590  01 A0 01 A2 01 A4 01 A6 01 A8 01 AA 01 AC 01 AE   ????????

004C15A0  01 B0 01 B2 01 B4 01 B6 01 B8 01 BA 01 BC 01 BE   ????????

004C15B0  01 C0 01 C2 01 C4 01 C6 01 C8 01 CA 01 CC 01 CE   ????????

004C15C0  01 D0 01 D2 01 D4 01 D6 01 D8 01 DA 01 DC 01 DE   ????????

004C15D0  01 E0 01 E2 01 E4 01 E6 01 E8 01 EA 01 EC 01 EE   ???

004C15E0  01 F0 01 F2 01 F4 01 F6 01 F8 01 FA 01 FC 01 FE   ?????度??

004C15F0  01 00 02 04 02 08 02 0C 02 10 02 14 02 18 02 1C   ???????

004C1600  02 20 02 24 02 28 02 2C 02 30 02 34 02 38 02 3C   ??????。???

004C1610  02 40 02 44 02 48 02 4C 02 50 02 54 02 58 02 5C   ??????堂

004C1620  02 60 02 64 02 68 02 6C 02 70 02 74 02 78 02 7C   怂搂砂

004C1630  02 80 02 84 02 88 02 8C 02 90 02 94 02 98 02 9C   谂适

004C1640  02 A0 02 A4 02 A8 02 AC 02 B0 02 B4 02 B8 02 BC   ????????

004C1650  02 C0 02 C4 02 C8 02 CC 02 D0 02 D4 02 D8 02 DC   ????????

004C1660  02 E0 02 E4 02 E8 02 EC 02 F0 02 F4 02 F8 02 FC   ??????

004C1670  02 00 03 08 03 10 03 18 03 20 03 28 03 30 03 38   ?????〃?

004C1680  03 40 03 48 03 50 03 58 03 60 03 68 03 70 03 78   ??怃

004C1690  03 80 03 88 03 90 03 98 03 A0 03 A8 03 B0 03 B8   考蠃逃????

004C16A0  03 C0 03 C8 03 D0 03 D8 03 E0 03 E8 03 F0 03 F8   ??????

004C16B0  03 00 04 10 04 20 04 30 04 40 04 50 04 60 04 70   ?????怄

004C16C0  04 80 04 90 04 A0 04 B0 04 C0 04 D0 04 E0 04 F0   耄逄?????

004C16D0  04 00 05 20 05 40 05 60 05 80 05 A0 05 C0 05 E0   ??怅者??

004C16E0  05 00 06 40 06 80 06 C0 06 00 07 80 07 00 08 00   ?耆?
```

奇数表:`G_BmpInfoTable2[200h]`

```
004C16F0  00 01 00 03 00 05 00 07 00 09 00 0B 00 0D 00 0F  ā???????

004C1700  00 11 00 13 00 15 00 17 00 19 00 1B 00 1D 00 1F  ????????

004C1710  00 21 00 23 00 25 00 27 00 29 00 2B 00 2D 00 2F  ??─?????

004C1720  00 31 00 33 00 35 00 37 00 39 00 3B 00 3D 00 3F  ????????

004C1730  00 41 00 43 00 45 00 47 00 49 00 4B 00 4D 00 4F  ???????

004C1740  00 51 00 53 00 55 00 57 00 59 00 5B 00 5D 00 5F  匀开

004C1750  00 61 00 63 00 65 00 67 00 69 00 6B 00 6D 00 6F  愀攀最

004C1760  00 71 00 73 00 75 00 77 00 79 00 7B 00 7D 00 7F  缀

004C1770  00 81 00 83 00 85 00 87 00 89 00 8B 00 8D 00 8F  蜀

004C1780  00 91 00 93 00 95 00 97 00 99 00 9B 00 9D 00 9F  销需

004C1790  00 A1 00 A3 00 A5 00 A7 00 A9 00 AB 00 AD 00 AF  ????????

004C17A0  00 B1 00 B3 00 B5 00 B7 00 B9 00 BB 00 BD 00 BF  ????????

004C17B0  00 C1 00 C3 00 C5 00 C7 00 C9 00 CB 00 CD 00 CF  ????????

004C17C0  00 D1 00 D3 00 D5 00 D7 00 D9 00 DB 00 DD 00 DF  ????????

004C17D0  00 E1 00 E3 00 E5 00 E7 00 E9 00 EB 00 ED 00 EF  ????

004C17E0  00 F1 00 F3 00 F5 00 F7 00 F9 00 FB 00 FD 00 FF  ???????

004C17F0  01 02 01 06 01 0A 01 0E 01 12 01 16 01 1A 01 1E  ????????

004C1800  01 22 01 26 01 2A 01 2E 01 32 01 36 01 3A 01 3E  ????????

004C1810  01 42 01 46 01 4A 01 4E 01 52 01 56 01 5A 01 5E  ???丁刁嘁威币

004C1820  01 62 01 66 01 6A 01 6E 01 72 01 76 01 7A 01 7E  瘁

004C1830  01 82 01 86 01 8A 01 8E 01 92 01 96 01 9A 01 9E  舁阁

004C1840  01 A2 01 A6 01 AA 01 AE 01 B2 01 B6 01 BA 01 BE  ????????

004C1850  01 C2 01 C6 01 CA 01 CE 01 D2 01 D6 01 DA 01 DE  ????????
```



## 资源数据段

```
004C1860  01 E2 01 E6 01 EA 01 EE 01 F2 01 F6 01 FA 01 FE  ???度?

004C1870  02 04 02 0C 02 14 02 1C 02 24 02 2C 02 34 02 3C  ????????

004C1880  02 44 02 4C 02 54 02 5C 02 64 02 6C 02 74 02 7C  ??搂

004C1890  02 84 02 8C 02 94 02 9C 02 A4 02 AC 02 B4 02 BC  谂????

004C18A0  02 C4 02 CC 02 D4 02 DC 02 E4 02 EC 02 F4 02 FC  ???????

004C18B0  03 08 03 18 03 28 03 38 03 48 03 58 03 68 03 78  ?????

004C18C0  03 88 03 98 03 A8 03 B8 03 C8 03 D8 03 E8 03 F8  蠃?????

004C18D0  04 10 04 30 04 50 04 70 04 90 04 B0 04 D0 04 F0  ??逄???

004C18E0  05 20 05 60 05 A0 05 E0 06 40 06 C0 07 80 08 00  ?怅???
```

## 资源读取与解密逻辑

这里是资源的读取和解密。

寄存器状态：`edi:80h`

```
.text:004168BD             lea     ecx, [ecx+0]
```

### 标签 `loc_4168C0`

代码交叉引用：`UnEntryBmpObj+647j`

```
.text:004168C0 loc_4168C0:

.text:004168C0             movzx   eax, G_BmpInfoTable1+1[edi*2]  ; member1

.text:004168C8             test    eax, eax

.text:004168CA             movzx   ecx, G_BmpInfoTable1[edi*2]    ; member0

.text:004168D2             jnz     short loc_4168EE              ；不跳转说明还需查 G_BmpInfoTable2
```

寄存器状态：`eax:G_BmpInfoTable1.member1;`

如果不是第一次和最后一次，`4168d4` 的代码不执行（意味着 `BmpOBj` 指针的位置不变）。

寄存器状态：`ecx:偶数表中的 member0 G_BmpInfoTable1[k].member0`

```
.text:004168D4             movzx   eax, byte ptr [esi]           ; Bmp 数据(BmpObj+10h)

.text:004168D7             movzx   edx, G_BmpInfoTable2[eax*2]   ; member0

.text:004168DF             movzx   eax, G_BmpInfoTable2+1[eax*2] ; member1

.text:004168E7             inc     esi                            ; 取 BmpOBj 下个数据

.text:004168E8             add     ecx, edx

.text:004168EA             test    eax, eax

.text:004168EC             jz      short loc_4168D4
```

寄存器状态：`eax:G_BmpInfoTable2.member1;`

这说明 `BmpObj` 前 10 个字节都是文件头。而且数据是以 `BYTE` 类型存放的。

分析认为，如果 `G_BmpInfoTable1.member0+G_BmpInfoTable2.member0=0` 时。说明需继续取 `*Res`。

继续代码：

又碰到了张表：`G_UnDeForTable[100h]`：`100h` 大小 ；用来比较用

### 反混淆表数据

```
004C2248  00 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2258  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2268  02 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2278  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2288  01 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2298  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22A8  02 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22B8  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22C8  00 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22D8  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22E8  02 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C22F8  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2308  01 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2318  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2328  02 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????

004C2338  03 07 06 07 05 07 06 07 04 07 06 07 05 07 06 07  ????????
```

### 标签 `loc_4168EE`

代码交叉引用：`UnEntryBmpObj+152j`

```
.text:004168EE loc_4168EE:

.text:004168EE             movzx   edi, G_UnDeForTable[eax]

.text:004168F5             mov     edx, ecx

.text:004168F7             sub     edx, edi

.text:004168F9             add     eax, 100h

.text:004168FE             test    edx, edx               ;if(edx<edi)跳

.text:00416900             jle     short loc_416935       ;跳表示不用 RES 来修正
```

**分析**：
* `edi`: `G_UnDeForTable[G_BmpInfoTable2[i].member1]`
* `edx`: `G_BmpInfoTable1.member0+G_BmpInfoTable2.member0`
* `k`: `100h`
* `cl`: `G_BmpInfoTable1[k].member0+G_BmpInfoTable2[k].member0`
* `ax`: `G_BmpInfoTable2[k].member1+100h`

### 标签 `loc_416935`

代码交叉引用：`UnEntryBmpObj+180j`

```
.text:00416935 loc_416935:

.text:00416935             shl     eax, cl                ;*4

.text:00416937             mov     ecx, eax               ;(ax>>cl)

.text:00416939             shr     ecx, 8                 ;(ax>>cl)/100h

.text:0041693C             mov     ebp, dword ptr [esp+ecx*4+104h+@@UnDeTable_1+4]  ;ecx>1h,否则出错

.text:00416940             and     eax, 0FFh              ; 只取最后一字节,这样就不会超过 ff

.text:00416945             cmp     ebp, 14h               ; 4*6h 才是 14h

.text:00416948             mov     [esp+104h+@@TmpNum], ebp

.text:0041694C             jl      loc_416B8E             ; 小于第[6]的继续跳
```

### 标签 `loc_416B8E`

代码交叉引用：`UnEntryBmpObj+1CCj`

```
.text:00416B8E loc_416B8E:

.text:00416B8E             cmp     ebp, 2                 ; 和 2 再比较,就只[0]  0 时是单独的数据

.text:00416B91             jl      loc_416D75             ; 小于的继续跳
```

寄存器状态：`eax: FFh && G_BmpInfoTable2[k].member1+100h`
赋值：`i=eax;`

### 标签 `loc_416D75`

代码交叉引用：`UnEntryBmpObj+411j`

```
.text:00416D75 loc_416D75:

.text:00416D75             movzx   edi, byte ptr [esi]    ; 当前资源位 Res[i]

.text:00416D78             movzx   edx, byte ptr [esi+1]  ; 当前资源位的下一位 Res[i+1]

.text:00416D7C             movzx   ecx, byte ptr [esi+2]  ; 当前资源位的二位 Res[i+2]

.text:00416D80             shl     edi, 8                 ; *100h

.text:00416D83             add     edi, edx               ; Res[i]*100h+Res[i+1]

.text:00416D85             shl     edi, 8                 ; (Res[i]*100h+Res[i+1])*100h

.text:00416D88             add     edi, ecx               ; (Res[i]*100h+Res[i+1])*100h+Res[i+2]

.text:00416D8A             mov     cl, G_UnDeForTable[eax]

.text:00416D90             shl     edi, 8                 ; (Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h

.text:00416D93             xor     edi, 80000080h         ; ((Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h)xor 80000080h

.text:00416D99             shr     edi, cl                ; (((Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h)xor 80000080h)<<G_UnDeForTable[j]

.text:00416D9B             shl     eax, 10h               ; j*10eh

.text:00416D9E             mov     edx, edi

.text:00416DA0             shr     edx, 8                 ;((((Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h)xor 80000080h)<<G_UnDeForTable[j])/100h

.text:00416DA3             xor     edx, eax               ;(((((Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h)xor 80000080h)<<G_UnDeForTable[j])/100h) xor j

.text:00416DA5             mov     eax, [esp+104h+@@UdForBmp]

.text:00416DA9             mov     [eax], edx             ; Fill ;填充到图片数据中。

.text:00416DAB             add     eax, 4                 ;指向图像数据的下一个

.text:00416DAE             and     edi, 0FFh

.text:00416DB4             add     esi, 3
```

寄存器状态：`edi:FFh && (((Res[i]*100h+Res[i+1])*100h+Res[i+2]*100h)xor 80000080h)<<G_UnDeForTable[j]`

### 标签 `loc_416DB7`

代码交叉引用：`UnEntryBmpObj+5F3j`

```
.text:00416DB7 loc_416DB7:

.text:00416DB7             mov     [esp+104h+@@ptr_ImageData], eax

.text:00416DBB             mov     [esp+104h+@@TmpNum], edi
```

### 标签 `loc_416DBF`

代码交叉引用：`UnEntryBmpObj+3BCj`, `UnEntryBmpObj+3E2j`

```
.text:00416DBF loc_416DBF:

.text:00416DBF             mov     ebx, [esp+104h+@@ptr_ImageData]    ;当前图像数据位置
```

### 标签 `loc_416DC3`

代码交叉引用：`UnEntryBmpObj+3D7j`, `UnEntryBmpObj+409j`

```
.text:00416DC3 loc_416DC3:
```



# 汇编分析与 C 语言转换

## 核心逻辑说明

当 `EBP` 值为 `26` 时，重复前面的一段数据。

### 双重复与色偏分支 (`loc_416B1D`, `loc_416B70`)

**汇编代码：**

```
.loc_416B1D:                 ;大量重复
lea     edi, [edi+edi+1]     ;双重复
mov     ecx, edx

loc_416B70:                  ;色偏分支
mov     ebp, [ebx+edx*4]     ; edx:取前面的 ImageData 在 NT1_Dim 表中查找差距(这是用来对整行的数据进行复制)
sub     ebp, [eax+edx*4]
add     eax, 4
add     ebp, [eax-4]         ; EBP 修正色偏
add     ebx, 4
dec     ecx                  ; 循环次数
mov     [ebx-4], ebp         ; FILL
jnz     short loc_416B70
```

**对应的 C 代码：**

```c
SubData=NT_Dim[(Od_value-0x14)];
for(int i=Offset_Value;i>0;i--)
{
  *(Image)=*(Image+SubData)-*(OffsetImage+SubData)+*(OffsetImage); //填充 IMAGE 数据
  Image++;
  OffsetImage++;
}
```

### 填充前面的整段数据，不带色偏 (`loc_416B46`)

**汇编代码：**

```
loc_416B46:          ; 填充前面的整段数据，不带色偏
mov     edx, [eax]
mov     [ebx], edx   ; Fill
add     ebx, 4
add     eax, 4       ; ecx:段大小
dec     ecx          ; 多次填充前面的一段数据
jnz     short loc_416B46
```

### 不重复，不带色差 (`loc_416D75`)

**汇编代码：**

```
loc_416D75:                 ;ebp<=2 不重复，不带色差的
movzx   edi, byte ptr [esi]     ;同时取多个 Res 数据
movzx   edx, byte ptr [esi+1]
movzx   ecx, byte ptr [esi+2]
shl     edi, 8
add     edi, edx
shl     edi, 8
add     edi, ecx
mov     cl, G_UnDeForTable[eax]
shl     edi, 8
xor     edi, 80000080h
shr     edi, cl
shl     eax, 10h
mov     edx, edi
shr     edx, 8
xor     edx, eax
mov     eax, [esp+104h+@@ptr_ImageData]
mov     [eax], edx       ; Fill
add     eax, 4
and     edi, 0FFh
add     esi, 3
```

**对应的 C 代码：**

```c
if(Od_value<2)
{
  //无修正,无重复,直接组合
  ImageData=(((Res[0]<<8)+Res[1])<<8)+Res[2];
  ImageData<<=8;
  ImageData^=0x80000080;
  ImageData>>=G_UnDeForTable[FindCode];
  TmpSave=FindCode;                    //临时值
  FindCode=ImageData & 0xFF;           //取出下次用的 FindCode,用于查表定位
  ImageData>>=8;
  ImageData^=(TmpSave<<0x10);
  *Image=ImageData;                    //填充 IMAGE 数据
  Image++;
  Res+=3;
}
```

### 不重复数据，可能带色差 (`loc_416D48`)

**汇编代码：**

```
loc_416D48:             ;14>ebp>2 不重复数据。可能带色差
shl     edi, cl        ;分别处理不同的 Res 数据。
mov     ecx, edi
shr     ecx, 8
mov     eax, ecx
and     eax, 1
neg     eax
add     ecx, 0FFFFFFFEh
xor     eax, ecx
cdq
sub     eax, edx
mov     ecx, eax
mov     eax, [esp+104h+@@ptr_ImageData]
sar     ecx, 1
add     ecx, ebx
and     edi, 0FFh
mov     [eax], ecx     ; Fill
add     eax, 4
jmp     short loc_416DB7
```

**对应的 C 代码：**

```c
Exbt<<=Bt_num;
Old_Offv=(-((Exbt>>8) &1)^((Exbt>>8)-2));  //色差
if(Old_Offv>=0)
{adjsig=0;}                                   //临时值
else
{adjsig=-1;}
  Offset_Color+=(Old_Offv-adjsig)>>1;        //保留符号位
  *Image=Offset_Color;                       //填充 IMAGE 数据
FindCode=Exbt & 0xFF;
Image++;
```

## 特殊代码讲解

### `G_UnDeForTable` 相关逻辑

**汇编代码：**

```
.text:004168EE                 movzx   edi, G_UnDeForTable[eax]
.text:004168F5                 mov     edx, ecx
.text:004168F7                 sub     edx, edi
.text:004168F9                 add     eax, 100h                ;为下一轮作准备
.text:004168FE                 test    edx, edx
.text:00416900                 jle     short loc_416935

//如果 edx>0 时，就表示要取*Res
.text:00416902                 mov     ecx, edi
.text:00416904                 shl     eax, cl
.text:00416906                 movzx   ecx, byte ptr [esi]
.text:00416909                 and     eax, 0FFFFFF00h
.text:0041690E                 dec     edx
.text:0041690F                 add     eax, ecx
.text:00416911                 inc     esi
.text:00416912                 cmp     edx, 8
.text:00416915                 jl      short loc_41692F

//可能会了出现大量的重复
.text:00416917                 mov     ecx, edx
.text:00416919                 shr     ecx, 3
.text:0041691C                 mov     edi, ecx
.text:0041691E                 neg     edi
.text:00416920                 lea     edx, [edx+edi*8]

loc_416923:                     ; CODE XREF: UnEntryBmpObj+1ADj
.text:00416923                 movzx   edi, byte ptr [esi]
.text:00416926                 shl     eax, 8
.text:00416929                 add     eax, edi
.text:0041692B                 inc     esi
.text:0041692C                 dec     ecx
.text:0041692D                 jnz     short loc_416923

loc_41692F:                     ; CODE XREF: UnEntryBmpObj+195j
.text:0041692F                 lea     eax, [eax+eax+1]    ;加倍重复
.text:00416933                 mov     ecx, edx
```

**对应的 C 代码：**

```c
Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;
if(Bt_num>Ud)
{
    Exbt<<=Ud;
    Exbt=(Exbt & 0xFFFFFF00)+(*Res);
    prevId--;
    Res++;
    if(prevId>=8)
    {
        OffsetPrevId=prevId>>3;
        prevId+=(0-OffsetPrevId)*8;
        for(i=OffsetPrevId;i>0;i--)
        {
            Exbt <<=8;          ;重复数*100h 或更大
            Exbt+=*Res;
            Res++;
        }
    }
    Exbt*=2;
    Exbt+=1;
    Bt_num=prevId;
}
```

### 表查询与修正

- `G_BmpInfoTable1s` 查到为 `0` 时，则要用新的 `*Res`。
- 有多种修正方法。
- `G_BmpInfoTable1.member0=0` 时，用 `Res_G_BmpInfoTable2` 来定位查 `G_BmpInfoTable2`。还是 `0`，就下一个，还在表中查，用 `Res` 数据来查。并且 `Res++`，直到不是 `0`。
- `G_BmpInfoTable(1,2).member0=G_BmpInfoTable1.member0+G_BmpInfoTable2.member0`；`Res++`
- `G_BmpInfoTable(1,2).member1` 用来定位 `G_UnDeForTable`。
- `G_BmpInfoTable(1,2).member0` 用来比较 `G_UnDeForTable`。

### `cdq` 指令说明

`cdq`: 测试 `EDX` 是不是和 `EAX` 的进位标识一样，类似于 `a?0:-1`。

## 动态跟踪过程参考

以下是动态跟踪中的过程（不是很详细，仅提供参考，详细分析请参考反后的 C 代码）：

### 经一级解密的 BMP 部分数据（还是加密的）

```
00C05000  8F FE FD 64 00 88 01 04  .?    资源中的数据用来定位 G_BmpInfoTable2
00C05008  44 49 12 D4 54 C9 46 90  DI
00C05010  90 14 D1 05 49 11 12 14  ??I
00C05018  95 37 A4 CA 80 51 41 12  ?なQA
00C05020  D4 54 D5 41 4D 21 38 6D  M!8m
00C05028  01 2D 40 4B 52 54 E1 84  -@KRT
00C05030  C5 45 4D D4 D9 4A A4 80  M 再 J
00C05038  05 21 4E 08 4C A1 2D 4D  !NL?M
00C05040  64 05 30 84 A5 32 53 15  d02S
00C05048  05 2D 4A 94 44 B5 38 54  -J?T
00C05050  14 D2 13 58 41 4F C1 A5  ?XAO 隶
```

### 解析步骤

1. `8F` 取出 `FEFD64` -> `FEFD6480`->`07EFD648` ->`07EFD6`  + `48`  `8F` 是第一个数据
2. `cl(>>1(G_UnDeForTable))`
3. `F8`->`F80000`->`FEEFD6` (`8 xor 7=F`)
4. 第一次填充完毕。
5. `90`->`190`->`320(<<1)`->`3` + `20`
6. `//20*2->01 40(G_BmpInfoTable1) 和 G_UnDeForTable 比 (01=01)`
7. `20*2->140(添加 1)->280(<<1)->2` + `80`
8. `80*2->01 00(G_BmpInfoTable1)`
9. `2->0(取最后一位）->-1`
10. `RES:00->00 01 ->07(G_UnDeForTable)`
11. `01->101(添加 1)`
12. `2->0(cdq)`
13. `101->202(<<1)`
14. `202->2 + 02`
15. `2->0(-2)`

///////////////// 校正下。用于重复前面的某些数据

16. `02*2->01 04(G_BmpInfoTable1)->05(G_UnDeForTable)`
17. `01 04->104(添加 1)->208(<<cl)->2 + 08(用于下轮 1 的查找)`
18. `2->1(-1) //和 0 比`
19. 当前 `G_UnDeTable` 中的定位和 `26` 比 （`26` 可能是种标志） `ebp=0`
20. `08*2->01 10(G_BmpInfoTable1)->03(G_UnDeForTable)`
21. `01 10->110(添加 1)->220(<<cl)->2 + 20(用于下轮 2 的查找)`
22. `G_UnDeTable` 用来比的,实现不同的分支 (现在用 `ebp=2-2` 定位)
23. 用 `ebp` 定位 `NT1_Dim`;
24. `20*2->01 40(G_BmpInfoTable1) 和 G_UnDeForTable 比 (01=01)`
25. `01 40->140(添加 1)->280(<<cl)->2 + 80(用于下轮 3 的查找)`
26. `2->0(取最后一位）->0(~)`
27. `2->0(-2)`

///////////////// 校正下。有时定位到 `IMage--`；用于重复前面的某些数据

28. `80*2->01 00(G_BmpInfoTable1) 查 G_BmpInfoTable2`
29. `88*2(Res)->01 22(G_BmpInfoTable2)->06(G_UnDeForTable)`
30. `01 22->122(添加 1)->488(<<cl)->4 + 88(用于下轮 4 的查找)`
31. `4->0(取最后一位)->0(~)(edx)`
32. `4->2(-2)->2(xor edx)->100(<<7)   ebx+100;ebx:上个 ImageData 07EFD6+100=07F0D6`
33. `88*2->02 20(G_BmpInfoTable1)->02(G_UnDeForTable)(edx)`
34. `02 20->120(添加 1)->480(<<cl)->4 + 80(查毕,用于下个 ImageData)`



# 逆向工程分析总结

## 汇编指令逻辑推导
*   `4 -> 0 (取最后一位) -> 0 (~) (eax)`
*   `4 -> 2 (-2) -> 2 (xor eax) -> 0 (cdq)`
*   `2 -> 1 (sar 1) ebx + 1`
*   `07FFD6 + 1 = 07F0D7` // 填充 `Image`
*   `80 * 2 -> 01 00 -> Res(01) -> 00 03 (G_BmpInfoTable1) -> 07 (G_UnDeForTable) Res++`
*   `00 03 -> 103 (添加 1) -> 206 (<< cl) -> 2 + 06` `ebp = 2 - 2 G_UnDeTable(0): 2`

## 解密流程概要
首先从 `.NME` 文件中找到图片的名称，并记录下序号。然后查找相应的 `.ATM` 文件，获取相关信息（包括 `DET` 文件中对应的位置和大小）。需验证加密号，以免数据已被修改。

随后进行一级解密（重复某些前面的数据）。根据文件前 `10H` 的文件头（包含宽度、高度、位数）算出图片大小，进行解压缩，即二级解密。

## 反汇编分析总结
1.  通过 `IDA` 分析出函数的整体架构。
2.  遇到 `CALL` 时别马上跟踪，先分析参数，再动态看 `CALL` 后的变化。
3.  做好注释，并把一些常数改成具体的 `API` 常数的表示。
4.  加强静态分析能力，不要依赖于动态分析。
5.  寄存器也可能是变量或参数，在分析时要注意。
6.  尽可能减少变量的个数，变量多了会大大增加分析难度。
7.  利用 `OD` 的堆栈和 `IDA` 的图表交叉来分析流程。
8.  熟悉各种 `CALL` 调用的方式，如 `STDCALL` 还是 `CDECL`、`FASTCALL` 等。
9.  特别要注意 `CDQ`、`SAR` 等汇编指令，这表明这是个有符号数。
10. 熟悉反汇编工具 `OD` 和 `IDA`，以及能编写简单的脚本。
11. 熟悉 `VC`、`BC`、`DELPH` 等常用编译器的风格。
12. 确保每个环节都不出差错，否则到后面时不知道错在哪里。
13. 思考如果是自己写的话，会如何写这个程序，从而找到突破的地方。

## 二级解密详细流程
1.  创建所需要的 5 张表。
2.  初始化 `FindCode` 为 `0x80`，用它查找 `G_BmpInfoTable1`；得到序号和值。如果值为 `0`，取出加密数据中的值。
3.  再用这个值拿来查表 `G_BmpInfoTable2`。如果还是 `0`，循环查表 `G_BmpInfoTable2` 直到不是 `0`。加密数据的内容主要就是用来查表的。
4.  再用这个值查表 `G_UnDeForTable`。得到的值和原来的序号比，如果小于就说明要修正。
5.  这个值同时也是分支号：
    *   `0` 时：表示只填充一次，且直接对加密数据运算得到图素。
    *   `0x26` 时：重复填充前面的一段图素。
    *   `2 - 0x14` 时：修正色差，并填充一次。
    *   `0x14 - 0x26` 时：重复 `N` 行的图素。

详细内容请看下面的反 `C` 代码，代码在下面的帖子中。

## C 语言逆向实现

下面试着反成 `C` (测试成功)：

并用 `SDK` 来写：

### MyBmp.h

```c
struct Record {
    char id;
    char value;
};

struct BmpRecord {
    int id;
    char str[16];
};

struct BmpHeader {
    BYTE UNKNOW1;
    BYTE UNKNOW2;
    BYTE BitCount;
    BYTE UNKNOW4;
    WORD BmpWidth;
    WORD BmpHeight;
};

// typedef char BYTE;

Record G_BmpInfoTable1[0x100];
Record G_BmpInfoTable2[0x100];
BYTE G_UnDeForTable[0x100];
DWORD G_UnDeTable[0x26];
DWORD NT_Dim[0x12];
DWORD *Image;
DWORD *Tail;
DWORD ImageData;
DWORD OffsetImageData;
DWORD *OffsetImage;
BYTE *Res;
DWORD BmpWidth;
BmpHeader *Bmp_Head;
BmpHeader G_BmpHead;
DWORD *BmpData = 0;

DWORD G_UnTable[0x26] = {
    0x2, 0x1, 0x0, 0x3,
    0x4, 0x5, 0x0A, 0x0B,
    0x0C, 0x0D, 0x0E, 0x0F,
    0x16, 0x17, 0x18, 0x19,
    0x1A, 0x1B, 0x1C, 0x1D,
    0x6, 0x8, 0x7, 0x9,
    0x10, 0x11, 0x12, 0x13,
    0x14, 0x15, 0x1E, 0x1F,
    0x20, 0x21, 0x22, 0x23,
    0x24, 0x25
};

DWORD G_UnTable_0[0x12] = {
    -1, 0, 1, -1,
    0, -2, 0, -3,
    0, -4, 0, -5, 0, -6, 0, -7, 0, -8
};

DWORD G_UnTable_1[0x12] = {
    0, -1, -1, -1, -2, 0, -3, 0,
    -4, 0, -5, 0, -6, 0, -7, 0, -8, 0
};

DWORD G_UnTable_2[0x12] = {
    -1, 0, 1, -1, -2, -2, -2, -1, 0, 1, 2, 2, -3, -3, -3, -3,
    -2, -1
};
```

### MapReader.cpp

```cpp
// MapReader.cpp : Defines the entry point for the application.
//
#include "stdafx.h"
#include "MyBmp.h"
#include "resource.h"

#define MAX_LOADSTRING 100

// Global Variables:
HINSTANCE hInst;                 // current instance
TCHAR szTitle[MAX_LOADSTRING];             // The title bar text
TCHAR szWindowClass[MAX_LOADSTRING];             // The title bar text

// Forward declarations of functions included in this code module:
ATOM MyRegisterClass(HINSTANCE hInstance);
BOOL InitInstance(HINSTANCE, int);
LRESULT CALLBACK WndProc(HWND, UINT, WPARAM, LPARAM);
LRESULT CALLBACK About(HWND, UINT, WPARAM, LPARAM);
HRESULT FileRead();
void LookBmpData(LPVOID lp, LPVOID nlp, int id);
void InitTable(DWORD nWidth);
DWORD* UnDecode(BYTE* lp, int BmpWidth);
void DrawBmp(HWND hwnd, DWORD* lp);

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR lpCmdLine,
                     int nCmdShow)
{
   // TODO: Place code here.
   MSG msg;
   HACCEL hAccelTable;

   // Initialize global strings
   LoadString(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
   LoadString(hInstance, IDC_MAPREADER, szWindowClass, MAX_LOADSTRING);

   MyRegisterClass(hInstance);

   // Perform application initialization:
   if (!InitInstance(hInstance, nCmdShow))
   {
      return FALSE;
   }

   hAccelTable = LoadAccelerators(hInstance, (LPCTSTR)IDC_MAPREADER);
   FileRead();

   // Main message loop:
   while (GetMessage(&msg, NULL, 0, 0))
   {
      if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
      {
         TranslateMessage(&msg);
         DispatchMessage(&msg);
      }
   }
   return msg.wParam;
}

//
//  FUNCTION: MyRegisterClass()
//
//  PURPOSE: Registers the window class.
//
//  COMMENTS:
//
//    This function and its usage is only necessary if you want this code
//    to be compatible with Win32 systems prior to the 'RegisterClassEx'
//    function that was added to Windows 95. It is important to call this function
//    so that the application will get 'well formed' small icons associated
//    with it.
//
ATOM MyRegisterClass(HINSTANCE hInstance)
{
   WNDCLASSEX wcex;

   wcex.cbSize = sizeof(WNDCLASSEX);
   wcex.style      = CS_HREDRAW | CS_VREDRAW;
   wcex.lpfnWndProc   = (WNDPROC)WndProc;
   wcex.cbClsExtra    = 0;
   wcex.cbWndExtra    = 0;
   wcex.hInstance    = hInstance;
   wcex.hIcon      = LoadIcon(hInstance, (LPCTSTR)IDI_MAPREADER);
   wcex.hCursor    = LoadCursor(NULL, IDC_ARROW);
   wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1);
   wcex.lpszMenuName  = (LPCSTR)IDC_MAPREADER;
   wcex.lpszClassName  = szWindowClass;
   wcex.hIconSm    = LoadIcon(wcex.hInstance, (LPCTSTR)IDI_SMALL);

   return RegisterClassEx(&wcex);
}

//
//   FUNCTION: InitInstance(HANDLE, int)
//
//   PURPOSE: Saves instance handle and creates main window
//
//   COMMENTS:
//
//        In this function, we save the instance handle in a global variable and
//        create and display the main program window.
//
BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
    HWND hWnd;
    hInst = hInstance; // Store instance handle in our global variable
    hWnd = CreateWindow(szWindowClass, szTitle, WS_OVERLAPPEDWINDOW,
       CW_USEDEFAULT, 0, CW_USEDEFAULT, 0, NULL, NULL, hInstance, NULL);
    if (!hWnd)
    {
       return FALSE;
    }

    ShowWindow(hWnd, nCmdShow);
    UpdateWindow(hWnd);
    return TRUE;
}

//
//  FUNCTION: WndProc(HWND, unsigned, WORD, LONG)
//
//  PURPOSE:  Processes messages for the main window.
//
//  WM_COMMAND  - process the application menu
//  WM_PAINT  - Paint the main window
//  WM_DESTROY  - post a quit message and return
//
//
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
   int wmId, wmEvent;
   PAINTSTRUCT ps;
   HDC hdc;
   TCHAR szHello[MAX_LOADSTRING];

   LoadString(hInst, IDS_HELLO, szHello, MAX_LOADSTRING);

   switch (message)
   {
      case WM_COMMAND:
         wmId    = LOWORD(wParam);
         wmEvent = HIWORD(wParam);
         // Parse the menu selections:
         switch (wmId)
         {
            case IDM_ABOUT:
               DialogBox(hInst, (LPCTSTR)IDD_ABOUTBOX, hWnd, (DLGPROC)About);
               break;
            case IDM_EXIT:
               delete BmpData;
               DestroyWindow(hWnd);
               break;
            case IDC_TEST:
                     DrawBmp(hWnd, BmpData);
               break;
            default:
               return DefWindowProc(hWnd, message, wParam, lParam);
         }
         break;
      case WM_PAINT:
         hdc = BeginPaint(hWnd, &ps);
         // TODO: Add any drawing code here...
         RECT rt;
         GetClientRect(hWnd, &rt);
         DrawText(hdc, szHello, strlen(szHello), &rt, DT_CENTER);
         EndPaint(hWnd, &ps);
         break;
      case WM_DESTROY:
         PostQuitMessage(0);
         break;
      default:
         return DefWindowProc(hWnd, message, wParam, lParam);
   }
   return 0;
}
```



```
return DefWindowProc(hWnd, message, wParam, lParam);
}
return 0;
}

// Mesage handler for about box.

LRESULT CALLBACK About(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{
switch (message)
{
case WM_INITDIALOG:
return TRUE;
case WM_COMMAND:
if (LOWORD(wParam) == IDOK || LOWORD(wParam) == IDCANCEL)
{
EndDialog(hDlg, LOWORD(wParam));
return TRUE;
}
break;
}
return FALSE;
}

HRESULT FileRead()
{
int BmpId=0x5;
BmpRecord br[200];
HANDLE hFile;
DWORD dwSize,dwByRead;
hFile=CreateFile("parts.nme",GENERIC_READ,0,0,OPEN_EXISTING,0,0);
if(INVALID_HANDLE_VALUE==hFile) return E_FAIL;
dwSize=GetFileSize(hFile,NULL);
dwSize-=4;
const DWORD nSize=dwSize;
BYTE *buf=new BYTE[dwSize];
ReadFile(hFile,buf,dwSize,&dwByRead,NULL);
memset(br,0,200*sizeof(BmpRecord));
int j=0;
for(int i=0;i<200;i++)
{
br[i].id=i;
int k=0;
while(buf[j]!=0)
{
br[i].str[k]=buf[j];
j++;
k++;
}
br[i].str[k]=0;
j++;
}
delete buf;
CloseHandle(hFile);
hFile=CreateFile("parts.atm",GENERIC_READ,0,0,OPEN_EXISTING,0,0);
if(INVALID_HANDLE_VALUE==hFile) return E_FAIL;
dwSize=GetFileSize(hFile,NULL);
dwSize-=4;
buf=new BYTE[dwSize];
ReadFile(hFile,buf,dwSize,&dwByRead,NULL);
DWORD DetOffset,DetSize;
DetOffset=*(DWORD*)(buf+4+0x14*BmpId);
DetSize=*(DWORD*)(buf+8+0x14*BmpId);
DWORD uObjSize;
uObjSize=*(DWORD*)(buf+((BmpId+BmpId*4)*4)+16);
delete buf;
CloseHandle(hFile);
hFile=CreateFile("parts.det",GENERIC_READ,0,0,OPEN_EXISTING,0,0);
if(INVALID_HANDLE_VALUE==hFile) return E_FAIL;
buf=new BYTE[DetSize];
SetFilePointer(hFile,DetOffset,0,FILE_BEGIN);
ReadFile(hFile,buf,DetSize,&dwByRead,NULL);
BYTE* nbuf=new BYTE [uObjSize];
memset(nbuf,0,uObjSize);
DWORD PreOffset;
long preNum;
j=0;
int k=0;
for(i=0;i<DetSize;i++)
{
if(buf[i]==0xFF)
{
k=i;
i++;
if(buf[i]!=0xff)
{
PreOffset=buf[i]>>2;
PreOffset++;
preNum=buf[i] & 0x80000003;
if(preNum<0)
{
preNum--;
preNum |=0xfffffffc;
preNum++;
}
preNum+=3;
if(preNum>0)
{
for(;preNum>0;preNum--)
{
nbuf[j]=nbuf[j-PreOffset];
j++;
}
j--;
}
else
{
j--;
}
}
else
{
nbuf[j]=0xff;
}
}
else
{
nbuf[j]=buf[i];
}
j++;
}
delete buf;
CloseHandle(hFile);
Bmp_Head=(BmpHeader*)nbuf;
InitTable(Bmp_Head->BmpWidth);
BmpData=UnDecode(nbuf,Bmp_Head->BmpWidth);
memcpy(&G_BmpHead,Bmp_Head,0x10);
delete nbuf;

/*
dwByRead=0x1d4c00;
DetSize=0x1d4c00;
hFile=CreateFile("bmp.mem",GENERIC_READ|GENERIC_WRITE,0,0,CREATE_ALWAYS,0,0);
WriteFile(hFile,BmpData,DetSize,&dwByRead,NULL);
CloseHandle(hFile);
*/
return S_OK;
}

DWORD* UnDecode(BYTE* lp,int BmpWidth)
{
DWORD prevId;
DWORD oldId;
DWORD OffsetPrevId;
long adjsig;
int i;
long Old_Offv;
BYTE Bt_num;
DWORD TmpSave;
BYTE Bt_value;
BYTE Ud;
DWORD Offset_Value;
DWORD Offset_Color;
DWORD Nt_value;
DWORD Od_value;
BYTE FindCode;
DWORD Exbt;
DWORD SubData;
int BmpSize=(Bmp_Head->BmpHeight)*(Bmp_Head->BmpWidth)+0x10;
Image=new DWORD[BmpSize];
memset(Image,0,BmpSize*4);
ImageData=*Image;
Res=(lp+0x10);
Tail=Image+BmpSize-0x10;
FindCode=0x80;
OffsetImageData=ImageData;
do
{
//if (Image==(DWORD*)(0xfb0040+0x77048)) _asm{int 3};
OffsetImage=Image;
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num =G_BmpInfoTable1[FindCode].id;
if(!Bt_value)
{
do
{
Bt_value=G_BmpInfoTable2[*Res].value;
Bt_num +=G_BmpInfoTable2[*Res].id;
Res++;
}while(!Bt_value);
}
Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;
if(Bt_num>Ud)
{
Exbt<<=Ud;
Exbt=(Exbt & 0xFFFFFF00)+(*Res);
prevId--;
Res++;
if(prevId>=8)
{
OffsetPrevId=prevId>>3;
prevId+=(0-OffsetPrevId)*8;
for(i=OffsetPrevId;i>0;i--)
{
Exbt <<=8;
Exbt+=*Res;
Res++;
}
}
Exbt*=2;
Exbt+=1;
Bt_num=prevId;
}
Exbt<<=Bt_num;
Od_value=(BYTE)G_UnDeTable[(Exbt>>8)-2];
FindCode=(BYTE)(Exbt & 0xFF);
if(Od_value<0x14)
{
if(Od_value<2)
{
//无修正,无重复,直接组合
ImageData=(((Res[0]<<8)+Res[1])<<8)+Res[2];
ImageData<<=8;
ImageData^=0x80000080;
ImageData>>=G_UnDeForTable[FindCode];
TmpSave=FindCode;
FindCode=ImageData & 0xFF;
ImageData>>=8;
ImageData^=(TmpSave<<0x10);
*Image=ImageData; //填充 IMAGE 数据
Image++;
Res+=3;
}
else
{
Nt_value=NT_Dim[Od_value-2];
Offset_Color=*(Image+Nt_value); //用于修正
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num =G_BmpInfoTable1[FindCode].id;
if(!Bt_value)
{
do
{
Bt_value=G_BmpInfoTable2[*Res].value;
Bt_num +=G_BmpInfoTable2[*Res].id;
Res++;
}while(!Bt_value);
}
Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;
if(Bt_num>Ud)
{
Exbt<<=Ud;
Exbt=(Exbt & 0xFFFFFF00)+(*Res);
prevId--;
Res++;
if(prevId>=8)
{
OffsetPrevId=prevId>>3;
prevId+=(0-OffsetPrevId)*8;
for(i=OffsetPrevId;i>0;i--)
{
Exbt <<=8;
Exbt+=*Res;
Res++;
}
}
Exbt*=2;
Exbt+=1;
Bt_num=prevId;
}
Exbt<<=Bt_num;
Offset_Color+=((0-((Exbt>>8)& 1) ^ ((Exbt>>8)-2))<<0xF);
FindCode=(BYTE)(Exbt & 0xFF);
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num=G_BmpInfoTable1[FindCode].id;
if(!Bt_value)
{
do
{
Bt_value=G_BmpInfoTable2[*Res].value;
Bt_num +=G_BmpInfoTable2[*Res].id;
Res++;
}while(!Bt_value);
}
Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;
if(Bt_num>Ud)
{
Exbt<<=Ud;
Exbt=(Exbt & 0xFFFFFF00)+(*Res);
prevId--;
Res++;
if(prevId>=8)
{
OffsetPrevId=prevId>>3;
prevId+=(0-OffsetPrevId)*8;
for(i=OffsetPrevId;i>0;i--)
{
Exbt <<=8;
Exbt+=*Res;
Res++;
}
}
Exbt*=2;
Exbt+=1;
Bt_num=prevId;
}
Exbt<<=Bt_num;
Offset_Color+=((0-((Exbt>>8)& 1) ^ ((Exbt>>8)-2))<<0x7); //色差
FindCode=(BYTE)(Exbt & 0xFF);
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num=G_BmpInfoTable1[FindCode].id;
if(!Bt_value)
{
do
{
Bt_value=G_BmpInfoTable2[*Res].value;
Bt_num +=G_BmpInfoTable2[*Res].id;
Res++;
}while(!Bt_value);
}
Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;
if(Bt_num>Ud)
{
Exbt<<=Ud;
Exbt=(Exbt & 0xFFFFFF00)+(*Res);
prevId--;
Res++;
```



```
if(prevId>=8)
{
    OffsetPrevId=prevId>>3;
    prevId+=(0-OffsetPrevId)*8;
    for(i=OffsetPrevId;i>0;i--)
    {
        Exbt <<=8;
        Exbt+=*Res;
        Res++;
    }
}
Exbt*=2;
Exbt+=1;
Bt_num=prevId;
}

Exbt<<=Bt_num;
Old_Offv=(-((Exbt>>8) &1)^((Exbt>>8)-2));
if(Old_Offv>=0)
{adjsig=0;}
else
{adjsig=-1;}

Offset_Color+=(Old_Offv-adjsig)>>1; //保留符号位
*Image=Offset_Color;                 //填充 IMAGE 数据
FindCode=Exbt & 0xFF;
Image++;
}

}
else
{
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num=G_BmpInfoTable1[FindCode].id;

  if(!Bt_value)
  {
    do
    {
      Bt_value=G_BmpInfoTable2[*Res].value;
      Bt_num+=G_BmpInfoTable2[*Res].id;
      Res++;
    }while(!Bt_value);
  }

Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;
prevId=Bt_num-Ud;

if(Bt_num>Ud)
  {
      Exbt<<=Ud;
  Exbt=(Exbt & 0xFFFFFF00)+(*Res);
  prevId--;
  Res++;

    if(prevId>=8)
    {
    OffsetPrevId=prevId>>3;
        prevId+=(0-OffsetPrevId)*8;
        for(i=OffsetPrevId;i>0;i--)
    {
        Exbt <<=8;
        Exbt+=*Res;
        Res++;
    }
    }

  Exbt*=2;
  Exbt+=1;
  Bt_num=prevId;
  }

Exbt<<=Bt_num;
oldId=Exbt>>8;
Offset_Value=((Exbt>>8)& 1)-1;
FindCode=(BYTE)(Exbt & 0xFF);
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num=G_BmpInfoTable1[FindCode].id;

  if(!Bt_value)
  {
    do
    {
      Bt_value=G_BmpInfoTable2[*Res].value;
      Bt_num+=G_BmpInfoTable2[*Res].id;
      Res++;
    }while(!Bt_value);
  }

Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;

prevId=Bt_num-Ud;
//SubData=Bt_value-Ud;

  if(Bt_num>Ud)
  {
  //416a35
   Exbt<<=Ud;
   Exbt=(Exbt & 0xFFFFFF00)+(*Res);
   prevId--;
   Res++;

     if(prevId>=8)
    {
    OffsetPrevId=prevId>>3;
        prevId+=(0-OffsetPrevId)*8;
        for(i=OffsetPrevId;i>0;i--)
    {
        Exbt <<=8;
        Exbt+=*Res;
        Res++;
    }
    }

  Exbt*=2;
  Exbt+=1;
  Bt_num=prevId;
  }

/*
DWROD CalcTmp;
if(Bt_num>=0)
{CalcTmp=0;}
esle{CalcTmp=-1}
*/
//Offset_Value^=(((Exbt<<Bt_num)>>8)-2)-(((((Exbt>>8)>>1)-CalceTmp)+Offset_Value)*BmpWidth);
Exbt<<=Bt_num;
if(oldId>=0)
{TmpSave=0;}
else
{TmpSave=-1;}
Old_Offv=(((oldId-TmpSave)>>1)+Offset_Value)*BmpWidth;
Offset_Value^=(Exbt>>8)-2;
Offset_Value-=Old_Offv;
//Offset_Value^=(((Exbt>>8)-2)-((((Exbt>>8)>>1)+Offset_Value)*BmpWidth));
OffsetImage+=Offset_Value;

FindCode=(BYTE)(Exbt & 0xFF);
Bt_value=G_BmpInfoTable1[FindCode].value;
Bt_num=G_BmpInfoTable1[FindCode].id;

  if(!Bt_value)
  {
    do
    {
      Bt_value=G_BmpInfoTable2[*Res].value;
      Bt_num+=G_BmpInfoTable2[*Res].id;
      Res++;
    }while(!Bt_value);
  }

Ud=G_UnDeForTable[Bt_value];
Exbt=Bt_value;
Exbt+=0x100;

prevId=Bt_num-Ud;

if(Bt_num>Ud)
  {
      Exbt<<=Ud;
  Exbt=(Exbt & 0xFFFFFF00)+(*Res);
  prevId--;
  Res++;

    if(prevId>=8)
    {
    OffsetPrevId=prevId>>3;
        prevId+=(0-OffsetPrevId)*8;
        for(i=OffsetPrevId;i>0;i--)
    {
        Exbt <<=8;
        Exbt+=*Res;
        Res++;
    }
    }

  Exbt*=2;
  Exbt+=1;
  Bt_num=prevId;
  }

Exbt<<=Bt_num;
Offset_Value=(Exbt>>8);
Offset_Value--;
FindCode=(BYTE)(Exbt & 0xFF);
if(Od_value==0x26)
  {
   if(Offset_Value>0)
   {
     for(i=Offset_Value;i>0;i--)
     {
       *Image=*OffsetImage;                //填充 IMAGE 数据
       Image++;
       OffsetImage++;
     }
   }
  }
  else
  {
   if(Offset_Value>0)
   {
   SubData=NT_Dim[(Od_value-0x14)];
     for(int i=Offset_Value;i>0;i--)
     {
      *(Image)=*(Image+SubData)-*(OffsetImage+SubData)+*(OffsetImage); //填充 IMAGE 数据
      Image++;
      OffsetImage++;
     }
   }
  }
}

}while(Image<Tail);                              ;数据未读完
Image-=(BmpSize-0x10);
return Image;
}

void InitTable(DWORD nWidth)
{
char a,b,x;
a=b=x=0;
char i;

  for(BYTE y=0;y<=0xFF;y++)
  {
   i=y;
   a=i;
   b=0;
   y=i;

  if(i)
  {
     if(i<0)
   {
      do
     {
        a <<=1;
     b++;
     }while(a<0);
   }

     if((a<<=1)==0) b--;
  }

    b++;

     G_BmpInfoTable1[y].id=b;
     G_BmpInfoTable1[y].value=a;
    a=i;
    b=0;
   if(i)
   {
   if(i<0)
   {
       do
     {
         a <<=1;
     b++;
     }while(a<0);
   }
   a<<=1;
   x=a;
   }

     a=1<<b;
    a +=x;
     G_BmpInfoTable2[y].id=b;
     G_BmpInfoTable2[y].value=a;
    a=0;
    b=i;
    if(i & 0x7f)
    {
         do
     {
         b<<=1;
     a++;
     }while(b & 0x7f);

    }
    G_UnDeForTable[y]=a;
  if(y==0xff) break;
  }

 DWORD TmpNum;
 for(i=0;i<0x26;i++)
 {
   TmpNum=G_UnTable[i];
   G_UnDeTable[TmpNum]=i;

 }

G_UnDeTable[1]=0x26;

for(i=0;i<18;i++)
{
NT_Dim[i]=(G_UnTable_1[i] * nWidth)+G_UnTable_0[i];
}
}

void   DrawBmp(HWND hwnd,DWORD* lp)    //仅用于测试
{
HDC hdc;
hdc=GetDC(hwnd);
LPBITMAPINFO pBit = (LPBITMAPINFO) new BYTE[ sizeof(BITMAPINFO) + 8 ];
//pBit->bmiHeader.biWidth
DWORD tmp[5];

tmp[0]=0x28;
tmp[1]=0x320;
tmp[2]=0x258;
tmp[3]=0x00200001;
tmp[4]=0;
pBit=(BITMAPINFO*)tmp;

StretchDIBits(hdc,0,0,0x320,0x258,0,0,0x320,0x258,lp,pBit,DIB_RGB_COLORS,SRCCOPY);

ReleaseDC(hwnd,hdc);
}
```

以上代码经测试，可完美的读出图片。
