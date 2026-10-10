

# aPLib 压缩引擎的使用（Win32 汇编版本）

> 原文：[`www.pediy.com/kssd/pediy10/94815.html`](https://www.pediy.com/kssd/pediy10/94815.html)

最近刚看完《加密与解密》第三版，想用 asm 写一个属于自己的壳，来论坛上逛了逛之后发现安全编程板块已经有了个软件壳保护的专题，也是用 asm 写的![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")……  
     于是便在这里长期蹲点学习……  
     貌似壳保护专题里面没有对区块进行压缩，《加》第三版中提到可以利用 aPLib 引擎对区块进行压缩，但是书中是 C 版本的，于是我就弄了个汇编版本的，希望这个汇编版本的代码对那些希望用汇编写壳的同志们有所帮助……  
最近也在一直看壳保护专题里面玩命贴出的代码，疑惑还是不少，但最近那里好像很久没更新了……正在学习壳的同志来一起研究研究啊……![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")  
   
资源脚本  
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
#include        <resource.h>  
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
#define IDB\_BMP                1  
#define IDD\_DIALOG1 101  
#define IDI\_ICON1 102  
#define IDC\_BUTTON1 1000  
#define IDC\_EDIT1 1001  
#define IDC\_BUTTON2 1002  
#define IDC\_STATIC 1003  
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
IDI\_ICON1    ICON        "ap.ico"  
IDB\_BMP        BITMAP        "aPack.bmp"  
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
IDD\_DIALOG1 DIALOG DISCARDABLE 50, 50, 198, 70  
STYLE DS\_MODALFRAME | WS\_POPUP | WS\_CAPTION | WS\_SYSMENU  
CAPTION "aP 压缩机"  
FONT 10, "System"  
BEGIN  
DEFPUSHBUTTON "压缩文件",IDOK,3,38,62,14  
PUSHBUTTON "退出",IDCANCEL,132,38,62,14  
PUSHBUTTON "解压文件",IDC\_BUTTON1,67,38,62,14  
CONTROL "",IDC\_STATIC,"Static",SS\_BITMAP | SS\_CENTERIMAGE,2,3,  
193,33  
LTEXT "文件名",IDC\_STATIC,3,56,25,10  
EDITTEXT IDC\_EDIT1,28,54,148,12,ES\_AUTOHSCROLL  
PUSHBUTTON "...",IDC\_BUTTON2,178,54,16,12  
END  
//>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
   
   
   
 利用 aPLib 提供的 lib 文件，和 inc 文件。主要就用 aP\_max\_packed\_size 获得压缩长度，aP\_workmem\_size 获得需要分配的空间，aPsafe\_get\_orig\_size 得到原始文件大小这几个函数就可以了，可以到[`www.ibsensoftware.com/`](http://www.ibsensoftware.com/)  （aPLib 的网站下载压缩引擎，里面也有这些函数的具体使用方法）。被压缩过的文件中会有“23PA”作为压缩标记，压缩的时候先搜索文件中是否有这个标记可以判别文件是否曾经被压缩过，然后在采取相应动作。  
   
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
        .386  
        .model flat, stdcall  
        option casemap :none  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; Include 文件定义  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
include        windows.inc  
include        user32.inc  
includelib    user32.lib  
include        kernel32.inc  
includelib    kernel32.lib  
include        comdlg32.inc  
includelib    comdlg32.lib  
include        aplib.inc  
includelib    aplib.lib  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; Equ 等值定义  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
IDB\_BMP        equ            1  
IDD\_DIALOG1    equ 101  
IDI\_ICON1    equ 102  
IDC\_BUTTON1    equ 1000  
IDC\_EDIT1     equ 1001  
IDC\_BUTTON2     equ 1002  
IDC\_STATIC     equ 1003  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 数据段  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
        .data?  
hInstance    dd    ?  
hWinMain    dd    ?  
hWinEdit    dd    ?  
hBmp        dd    ?  
szFileName    db    MAX\_PATH dup (?)  
   
        .const  
szOpenFile    db    'Files',0,'\*.exe;\*.dll;\*.txt;\*.doc',0  
        db    'All Files(\*.\*)',0,'\*.\*',0,0  
szText        db    '创建文件出错！',0  
szCaption    db    '提示',0  
szText2        db    '文件已压缩过',0  
szText3        db    '压缩成功！',0  
szText4        db    '文件名不能为空！',0  
szText5        db    '文件未压缩过，无需解压！',0  
szText6        db    '解压过程出错！',0  
szText7        db    '解压成功！',0  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
; 代码段  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
   
        .code  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_OpenFile    proc  
        local    @stOF:OPENFILENAME  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 打开文件  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
        invoke    RtlZeroMemory,addr @stOF,sizeof @stOF  
        mov    @stOF.lStructSize,sizeof @stOF  
        push    hWinMain  
        pop    @stOF.hwndOwner  
        mov    @stOF.lpstrFilter,offset szOpenFile  
        mov    @stOF.lpstrFile,offset szFileName  
        mov    @stOF.nMaxFile,MAX\_PATH  
        mov    @stOF.Flags,OFN\_PATHMUSTEXIST or OFN\_FILEMUSTEXIST  
        invoke    GetOpenFileName,addr @stOF  
        .if    ! eax  
            jmp    @F  
        .endif  
        invoke    SetDlgItemText,hWinMain,IDC\_EDIT1,addr szFileName  
@@:          
        ret  
\_OpenFile    endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_PackFile    proc  
        local    @szBuffer\[256\]:byte  
        local    @hFile,@length,@lpSource,@lpDest,@lpWorkMem,@lpPackLen,@dwTemp  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 文件压缩  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
        .if    szFileName\[0\] == 0  
            invoke    MessageBox,NULL,offset szText4,offset szCaption,MB\_OK  
            jmp    @F  
        .endif  
        invoke    CreateFile,addr szFileName,GENERIC\_READ,FILE\_SHARE\_READ,NULL,OPEN\_EXISTING \\  
            ,FILE\_ATTRIBUTE\_NORMAL,NULL  
        .if    eax ==    INVALID\_HANDLE\_VALUE  
            invoke    MessageBox,NULL,offset szText,offset szCaption,MB\_OK  
            jmp    @F  
        .endif  
        mov    @hFile,eax  
        invoke    GetFileSize,eax,NULL  
        mov    @length,eax  
        invoke    GlobalAlloc,GPTR,@length  
        mov    @lpSource,eax  
        invoke    ReadFile,@hFile,@lpSource,@length,addr @dwTemp,NULL  
        invoke    CloseHandle,@hFile  
        mov    esi,@lpSource  
        lodsd  
        cmp    eax,"23PA"  
        jne    Compress  
        invoke    MessageBox,NULL,offset szText2,offset szCaption,MB\_OK  
        invoke    GlobalFree,@lpSource  
        jmp    @F  
Compress:  
        invoke    aP\_max\_packed\_size,@length  
        invoke    GlobalAlloc,GPTR,eax  
        mov    @lpDest,eax  
        invoke    aP\_workmem\_size,@length  
        invoke    GlobalAlloc,GPTR,eax  
        mov    @lpWorkMem,eax  
        invoke    aPsafe\_pack,@lpSource,@lpDest,@length,@lpWorkMem,NULL,NULL  
        mov    @lpPackLen,eax  
        .if    eax == 0  
        jmp    FreeMem  
        .endif  
        invoke    CreateFile,addr szFileName,GENERIC\_WRITE,FILE\_SHARE\_WRITE,NULL,CREATE\_ALWAYS \\  
            ,FILE\_ATTRIBUTE\_NORMAL,NULL  
        .if    eax ==    INVALID\_HANDLE\_VALUE  
            invoke    MessageBox,NULL,offset szText,offset szCaption,MB\_OK  
            jmp    @F  
        .endif  
        mov    @hFile,eax  
        invoke    WriteFile,@hFile,@lpDest,@lpPackLen,addr @dwTemp,NULL  
        invoke    CloseHandle,@hFile  
        invoke    MessageBox,NULL,offset szText3,offset szCaption,MB\_OK          
FreeMem:  
        invoke    GlobalFree,@lpSource  
        invoke    GlobalFree,@lpDest  
        invoke    GlobalFree,@lpWorkMem  
@@:          
        ret  
\_PackFile    endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_DepackFile    proc      
        local    @szBuffer\[256\]:byte  
        local    @hFile,@length,@lpSource,@lpDest,@lpWorkMem,@lpPackLen,@OriLen,@dwTemp,@DePackLen  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 文件解压  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
        .if    szFileName\[0\] == 0  
            invoke    MessageBox,NULL,offset szText4,offset szCaption,MB\_OK  
            jmp    @F  
        .endif  
        invoke    CreateFile,addr szFileName,GENERIC\_READ,FILE\_SHARE\_READ,NULL,OPEN\_EXISTING \\  
            ,FILE\_ATTRIBUTE\_NORMAL,NULL  
        .if    eax ==    INVALID\_HANDLE\_VALUE  
            invoke    MessageBox,NULL,offset szText,offset szCaption,MB\_OK  
            jmp    @F  
        .endif  
        mov    @hFile,eax  
        invoke    GetFileSize,eax,NULL  
        mov    @length,eax  
        invoke    GlobalAlloc,GPTR,@length  
        mov    @lpSource,eax  
        invoke    ReadFile,@hFile,@lpSource,@length,addr @dwTemp,NULL  
        invoke    CloseHandle,@hFile  
        invoke    aPsafe\_get\_orig\_size,@lpSource  
        mov    @OriLen,eax  
        test    eax,eax  
        jnz    DePack  
        invoke    MessageBox,NULL,offset szText5,offset szCaption,MB\_OK  
        invoke    GlobalFree,@lpSource  
        jmp    @F  
DePack:  
        invoke    GlobalAlloc,GPTR,@OriLen  
        mov    @lpDest,eax  
        invoke    aPsafe\_depack,@lpSource,@length,@lpDest,@OriLen  
        mov    @DePackLen,eax  
        .if    eax == 0  
        invoke    MessageBox,NULL,offset szText6,offset szCaption,MB\_OK  
        jmp    FreeMem2  
        .endif  
        invoke    CreateFile,addr szFileName,GENERIC\_WRITE,FILE\_SHARE\_WRITE,NULL,CREATE\_ALWAYS \\  
            ,FILE\_ATTRIBUTE\_NORMAL,NULL  
        .if    eax ==    INVALID\_HANDLE\_VALUE  
            invoke    MessageBox,NULL,offset szText,offset szCaption,MB\_OK  
            jmp    FreeMem2  
        .endif  
        mov    @hFile,eax  
        invoke    WriteFile,@hFile,@lpDest,@OriLen,addr @dwTemp,NULL  
        invoke    CloseHandle,@hFile  
        invoke    MessageBox,NULL,offset szText7,offset szCaption,MB\_OK  
FreeMem2:  
        invoke    GlobalFree,@lpSource  
        invoke    GlobalFree,@lpDest  
@@:  
        ret  
\_DepackFile    endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
\_ProcDlgMain    proc    uses ebx edi esi hWnd,wMsg,wParam,lParam  
        mov    eax,wMsg  
        .if    eax == WM\_CLOSE  
            invoke    EndDialog,hWnd,NULL  
        .elseif    eax == WM\_INITDIALOG  
            push    hWnd  
            pop    hWinMain  
            invoke    LoadIcon,hInstance,IDI\_ICON1  
            invoke    SendMessage,hWinMain,WM\_SETICON,ICON\_BIG,eax  
            invoke    LoadBitmap,hInstance,IDB\_BMP  
            mov    hBmp,eax  
            invoke    SendDlgItemMessage,hWnd,IDC\_STATIC,STM\_SETIMAGE,IMAGE\_BITMAP,eax  
        .elseif eax == WM\_COMMAND  
            mov    eax,wParam  
            .if    ax ==    IDC\_BUTTON2  
                call    \_OpenFile  
            .elseif ax ==    IDOK  
                call    \_PackFile  
            .elseif ax ==    IDC\_BUTTON1  
                call    \_DepackFile  
            .elseif    ax ==    IDCANCEL  
                invoke    EndDialog,hWnd,NULL  
            .endif  
        .else  
            mov    eax,FALSE  
            ret  
        .endif  
        mov    eax,TRUE  
        ret  
   
\_ProcDlgMain    endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
start:  
        invoke    GetModuleHandle,NULL  
        mov    hInstance,eax  
        invoke    DialogBoxParam,hInstance,IDD\_DIALOG1,NULL,offset \_ProcDlgMain,NULL  
        invoke    ExitProcess,NULL  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
        end    start  
   
   
   
     程序很简单，写得不好请见谅，第一次发帖，还望高手们不要严厉批评给我太大打击![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || aplib2.rar  
 |