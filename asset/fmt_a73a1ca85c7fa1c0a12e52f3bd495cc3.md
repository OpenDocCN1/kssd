
_ 这里是对原始文档的排版处理结果 _

### 程序代码

```
_OnCBSelChange endp

;****************************************************************
;根据给定的QQ号，打开临时聊天对话框
;****************************************************************
_OpenQQWnd proc
    LOCAL    posBuffer[255]:byte

    invoke GetDlgItemText,hWndDlg,IDC_QQNUM ,addr lpBuf,20  ;取用户输入到文本
    invoke wsprintf,addr posBuffer,addr lptemp,addr lpBuf ;连接文本串
    invoke ShellExecute,NULL,NULL,addr posBuffer,NULL,NULL,SW_HIDE  ;执行 IE 命令
    
    ret

_OpenQQWnd endp

;****************************************************************
;窗口主调函数，判断并处理各种消息
;****************************************************************
DlgProc proc hWin:HWND,uMsg:UINT,wParam:WPARAM,lParam:LPARAM

    mov    eax,uMsg
    .if eax==WM_INITDIALOG
        mov    eax,hWin
        mov    hWndDlg,eax
        call    _initDlg
    .elseif eax==WM_COMMAND
        mov eax,wParam 
            .if lParam!=0  
                mov edx,eax
                shr edx,16  
                 .if dx==BN_CLICKED ;按钮消息处理
                    .if ax==IDC_SEND ;发送
                    call _SendMsg
                    .elseif ax==IDC_SAVE  ;保存
                    call _SaveMsg
                 .elseif ax==IDC_AUTOSEND  ;自动项开关
                    call _AutoSend
                 .elseif ax==IDC_OPENQQWND ;打开临时QQ聊天对话框
                    call _OpenQQWnd
                       .endif
                 .elseif dx==EN_CHANGE  ;文本框内容改变
                       .if ax==IDC_MESSAGE
                    invoke GetDlgItem,hWndDlg,IDC_SAVE
                    invoke EnableWindow,eax,TRUE
                       .endif
                 .elseif dx==CBN_SELCHANGE  ;手动改变列表选项
                    call _OnCBSelChange
                .endif  
            .endif  
              
    .elseif eax==WM_CLOSE
        invoke KillTimer,hWndDlg,ID_TIMER
        invoke EndDialog,hWin,0
    .elseif eax==WM_TIMER  ;时钟消息
        invoke _Timer,wParam
    .else
        mov    eax,FALSE
        ret
    .endif
    mov    eax,TRUE
    ret

DlgProc endp

end start
```

xxx感谢 `4stOne` 的指正，我改过的内容放在 24 楼了，有兴趣的给测试一下。

新的群发器可以让你随便指定一个QQ号，然后打开临时聊天窗口，自动发消息给他,编译后的 `exe` 文件在27楼。

> 引用: 最初由 **`4st0ne`**发布 能发下 `inc` 和 `rc` 文件吗？

**`QQ` 自动群发器. `Inc` 文件：**

```
include windows.inc
include kernel32.inc
include user32.inc
include Comctl32.inc
include shell32.inc

includelib kernel32.lib
includelib user32.lib
includelib Comctl32.lib
includelib shell32.lib

DlgProc      PROTO  :HWND,:UINT,:WPARAM,:LPARAM
_OnCBSelChange    PROTO
_EnumWindowsProc  PROTO   :HWND,:HANDLE

.const

IDD_MAINDIALOG      equ 101
IDC_FILEMSG      equ 1001
IDC_MESSAGE      equ 1002
IDC_TIME      equ 1005
IDC_AUTOSEND      equ 1006
IDC_RBN2      equ 1007
IDC_RBN3      equ 1008
IDC_SAVE      equ 1009
IDC_SEND      equ 1010
;

ID_TIMER      equ 8888
;#########################################################################

.data?
hWndDlg        dd ?
lpBuf        db 21 dup(?)
dwRandom      dd ?

;#########################################################################与
.data
lpString   DB  '未找到消息文件：'
lpFileName   DB  'FileMsg.txt',0
lpWRJ     DB  '程序编者：王仁军',0
lpQQWindow  DB  '#32770',0
lpQQDlg    DB  'AfxWnd42',0
lpQQMsg    DB  'RICHEDIT',0

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
```

**`QQ` 自动群发器. `Rc` 文件：**

```
#include "Res/QQ 自动群发器 Dlg.rc"
#define MY_ICON 10
MY_ICON ICON "QQ 群发器.ico"

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
```

**`QQ` 自动群发器. `dlg` 文件：**

```
#define IDD_MAINDIALOG 101
#define IDC_GRP1 1003
#define IDC_FILEMSG 1001
#define IDC_MESSAGE 1002
#define IDC_TIME 1005
#define IDC_AUTOSEND 1006
#define IDC_RBN3 1008
#define IDC_SAVE 1009
#define IDC_SEND 1010
#define IDC_STC1 1011
#define IDC_RBN2 1007
#define IDC_RBN1 1004
IDD_MAINDIALOG DIALOGEX 150,110,191,145
CAPTION "QQ 自动群发器"
FONT 12,"宋体",400,0
STYLE 0x14CA0800
EXSTYLE 0x00000000
BEGIN
  CONTROL "",IDC_FILEMSG,"ComboBox",0x50210043,3,1,185,164,0x00000000
  CONTROL "我可以自动把消息发给多个人",IDC_MESSAGE,"Edit",0x50211004,3,15,183,99,0x00020000
  CONTROL "",IDC_GRP1,"Button",0x50000007,3,112,80,30,0x00000000
  CONTROL "6",IDC_TIME,"Edit",0x50010001,6,118,17,9,0x00020000
  CONTROL "秒自动发一次",IDC_AUTOSEND,"Button",0x50014C23,22,118,57,9,0x00020000
  CONTROL "固定",IDC_RBN1,"Button",0x50030009,4,129,25,11,0x00000000
  CONTROL "循环",IDC_RBN2,"Button",0x50010009,30,129,25,11,0x00000000
  CONTROL "随机",IDC_RBN3,"Button",0x50010009,57,129,25,11,0x00000000
  CONTROL "保存[&F]",IDC_SAVE,"Button",0x58010000,85,117,48,15,0x00000000
  CONTROL "发送[&S]",IDC_SEND,"Button",0x50010001,136,117,48,15,0x00000000
  CONTROL "Email：673759017@qq.com",IDC_STC1,"Static",0x50000000,88,132,96,11,0x00000000
END
```

### 修改后的代码

```
;****************************************************************
;找到窗口，判断是否是QQ的聊天窗口，是则给他发送文字，不是就让 EnumWindows 继续找
;****************************************************************
_EnumWindowsProc proc  hWndQQ:HWND,@pMem:DWORD

  invoke  GetWindowText,hWndQQ,addr lpBuf,21  ;取窗口标题文字
  mov  ecx,eax
  mov  ax,0ebd3h  ;"与"的内码倒置
  lea  edx,lpBuf
  .while  ax!=word ptr [edx]  ;标题中有“与”这个字吗？
     or  ecx,ecx
     je  FINDNEXTWND
     dec  ecx
     inc  edx
  .endw

  invoke  GetDlgItem,hWndQQ,0h;开始获取输入框句柄
  invoke  GetDlgItem,eax,0h
  invoke  GetDlgItem,eax,37Eh
  invoke  SendMessage,eax,EM_REPLACESEL,0,@pMem ;向 QQ 输入框粘贴字符
  invoke  GetDlgItem,hWndQQ,0h ;开始获取发送按钮句柄
  invoke  GetDlgItem,eax,1h
  invoke  SendMessage,eax,BM_CLICK,0,0;点击发送按钮
FINDNEXTWND:
  mov  eax,TRUE  ;告诉 EnumWindows 函数，我要继续找下一个窗口，挨个找，一个都不能少
  ret
  
_EnumWindowsProc endp
```

`4dd466d0e1e0cc123afbe504da2462ca`
