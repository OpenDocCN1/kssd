
# [原创]aPLib压缩引擎的使用（Win32汇编版本）

最近刚看完《加密与解密》第三版，想用asm写一个属于自己的壳，来论坛上逛了逛之后发现安全编程板块已经有了个软件壳保护的专题，也是用asm写的![](https://bbs.kanxue.com/view/img/face/16.gif)![](https://bbs.kanxue.com/view/img/face/16.gif)……

于是便在这里长期蹲点学习……

貌似壳保护专题里面没有对区块进行压缩，《加》第三版中提到可以利用aPLib引擎对区块进行压缩，但是书中是C版本的，于是我就弄了个汇编版本的，希望这个汇编版本的代码对那些希望用汇编写壳的同志们有所帮助……

最近也在一直看壳保护专题里面玩命贴出的代码，疑惑还是不少，但最近那里好像很久没更新了……正在学习壳的同志来一起研究研究啊……![](https://bbs.kanxue.com/view/img/face/27.gif)

## 资源脚本

```rc
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#include        <resource.h>

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

#define IDB_BMP                1

#define IDD_DIALOG1 101

#define IDI_ICON1 102

#define IDC_BUTTON1 1000

#define IDC_EDIT1 1001

#define IDC_BUTTON2 1002

#define IDC_STATIC 1003

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

IDI_ICON1    ICON        "ap.ico"

IDB_BMP        BITMAP        "aPack.bmp"

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

IDD_DIALOG1 DIALOG DISCARDABLE 50, 50, 198, 70

STYLE DS_MODALFRAME | WS_POPUP | WS_CAPTION | WS_SYSMENU

CAPTION "aP压缩机"

FONT 10, "System"

BEGIN

DEFPUSHBUTTON "压缩文件",IDOK,3,38,62,14

PUSHBUTTON "退出",IDCANCEL,132,38,62,14

PUSHBUTTON "解压文件",IDC_BUTTON1,67,38,62,14

CONTROL "",IDC_STATIC,"Static",SS_BITMAP | SS_CENTERIMAGE,2,3,

193,33

LTEXT "文件名",IDC_STATIC,3,56,25,10

EDITTEXT IDC_EDIT1,28,54,148,12,ES_AUTOHSCROLL

PUSHBUTTON "...",IDC_BUTTON2,178,54,16,12

END

//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
```

利用aPLib提供的lib文件，和inc文件。主要就用`aP_max_packed_size`获得压缩长度，`aP_workmem_size`获得需要分配的空间，`aPsafe_get_orig_size`得到原始文件大小这几个函数就可以了，可以到[53cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2T1M7$3g2F1M7$3!0X3N6s2N6S2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.](https://bbs.kanxue.com/elink@62eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2T1M7$3g2F1M7$3!0X3N6s2N6S2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1j5`.)（aPLib的网站下载压缩引擎，里面也有这些函数的具体使用方法）。被压缩过的文件中会有“23PA”作为压缩标记，压缩的时候先搜索文件中是否有这个标记可以判别文件是否曾经被压缩过，然后在采取相应动作。

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

        .386

        .model flat, stdcall

        option casemap :none

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; Include 文件定义

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include        windows.inc

include        user32.inc

includelib    user32.lib

include        kernel32.inc

includelib    kernel32.lib

include        comdlg32.inc

includelib    comdlg32.lib

include        aplib.inc

includelib    aplib.lib

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; Equ 等值定义

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

IDB_BMP        equ            1

IDD_DIALOG1    equ 101

IDI_ICON1    equ 102

IDC_BUTTON1    equ 1000

IDC_EDIT1     equ 1001

IDC_BUTTON2     equ 1002

IDC_STATIC     equ 1003

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 数据段

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

        .data?

hInstance    dd    ?

hWinMain    dd    ?

hWinEdit    dd    ?

hBmp        dd    ?

szFileName    db    MAX_PATH dup (?)

        .const

szOpenFile    db    'Files',0,'*.exe;*.dll;*.txt;*.doc',0

        db    'All Files(*.*)',0,'*.*',0,0

szText        db    '创建文件出错！',0

szCaption    db    '提示',0

szText2        db    '文件已压缩过',0

szText3        db    '压缩成功！',0

szText4        db    '文件名不能为空！',0

szText5        db    '文件未压缩过，无需解压！',0

szText6        db    '解压过程出错！',0

szText7        db    '解压成功！',0

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 代码段

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

        .code

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_OpenFile    proc

        local    @stOF:OPENFILENAME

;***********************************************************************************

; 打开文件

;***********************************************************************************

        invoke    RtlZeroMemory,addr @stOF,sizeof @stOF

        mov    @stOF.lStructSize,sizeof @stOF

        push    hWinMain

        pop    @stOF.hwndOwner

        mov    @stOF.lpstrFilter,offset szOpenFile

        mov    @stOF.lpstrFile,offset szFileName

        mov    @stOF.nMaxFile,MAX_PATH

        mov    @stOF.Flags,OFN_PATHMUSTEXIST or OFN_FILEMUSTEXIST

        invoke    GetOpenFileName,addr @stOF

        .if    ! eax

            jmp    @F

        .endif

        invoke    SetDlgItemText,hWinMain,IDC_EDIT1,addr szFileName

@@:

        ret

_OpenFile    endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_PackFile    proc

        local    @szBuffer[256]:byte

        local    @hFile,@length,@lpSource,@lpDest,@lpWorkMem,@lpPackLen,@dwTemp

;***********************************************************************************

; 文件压缩

;***********************************************************************************

        .if    szFileName[0] == 0

            invoke    MessageBox,NULL,offset szText4,offset szCaption,MB_OK

            jmp    @F

        .endif

        invoke    CreateFile,addr szFileName,GENERIC_READ,FILE_SHARE_READ,NULL,OPEN_EXISTING \

            ,FILE_ATTRIBUTE_NORMAL,NULL

        .if    eax ==    INVALID_HANDLE_VALUE

            invoke    MessageBox,NULL,offset szText,offset szCaption,MB_OK

            jmp    @F

        .endif

        mov    @hFile,eax

        invoke    GetFileSize,eax,NULL

        mov    @length,eax

        invoke    GlobalAlloc,GPTR,@length

        mov    @lpSource,eax

        invoke    ReadFile,@hFile,@lpSource,@length,addr @dwTemp,NULL

        invoke    CloseHandle,@hFile

        mov    esi,@lpSource

        lodsd

        cmp    eax,"23PA"

        jne    Compress

        invoke    MessageBox,NULL,offset szText2,offset szCaption,MB_OK

        invoke    GlobalFree,@lpSource

        jmp    @F

Compress:

        invoke    aP_max_packed_size,@length

        invoke    GlobalAlloc,GPTR,eax

        mov    @lpDest,eax

        invoke    aP_workmem_size,@length

        invoke    GlobalAlloc,GPTR,eax

        mov    @lpWorkMem,eax

        invoke    aPsafe_pack,@lpSource,@lpDest,@length,@lpWorkMem,NULL,NULL

        mov    @lpPackLen,eax

        .if    eax == 0

        jmp    FreeMem

        .endif

        invoke    CreateFile,addr szFileName,GENERIC_WRITE,FILE_SHARE_WRITE,NULL,CREATE_ALWAYS \

            ,FILE_ATTRIBUTE_NORMAL,NULL

        .if    eax ==    INVALID_HANDLE_VALUE

            invoke    MessageBox,NULL,offset szText,offset szCaption,MB_OK

            jmp    @F

        .endif

        mov    @hFile,eax

        invoke    WriteFile,@hFile,@lpDest,@lpPackLen,addr @dwTemp,NULL

        invoke    CloseHandle,@hFile

        invoke    MessageBox,NULL,offset szText3,offset szCaption,MB_OK

FreeMem:

        invoke    GlobalFree,@lpSource

        invoke    GlobalFree,@lpDest

        invoke    GlobalFree,@lpWorkMem

@@:

        ret

_PackFile    endp

;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_DepackFile    proc

        local    @szBuffer[256]:byte

        local    @hFile,@length,@lpSource,@lpDest,@lpWorkMem,@lpPackLen,@OriLen,@dwTemp,@DePackLen

;***********************************************************************************

; 文件解压

;***********************************************************************************

        .if    szFileName[0] == 0

            invoke    MessageBox,NULL,offset szText4,offset szCaption,MB_OK

            jmp    @F

        .endif

        invoke    CreateFile,addr szFileName,GENERIC_READ,FILE_SHARE_READ,NULL,OPEN_EXISTING \

            ,FILE_ATTRIBUTE_NORMAL,NULL
```
