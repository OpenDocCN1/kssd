

# 自己动手用汇编写 PE 文件头信息查看工具

> 原文：[`www.pediy.com/kssd/pediy10/77544.html`](https://www.pediy.com/kssd/pediy10/77544.html)

编程语言：MASAM32 汇编  
编程环境：MASM32  
所需知识：汇编语言，PE 文件结构（http://bbs.pediy.com/showthread.php?s=&threadid=22892 这里有）  
  
首先是写有界面的程序，需要定义资源文件  
不定义资源文件，若在编程过程中生成控件，则是相当麻烦的  
程序最终界面如下  
![](img/fa67978ecfa1335b33e70a4c3144140b.jpg)    ![](img/0edfb366daf3b70f8fbcf47d898a0d15.jpg)  
所以定义如下资源，贴出代码  
#include    <resource.h>  
  
#define  ICO\_MAIN  2000  
#define  DLG\_MAIN  2000  
#define  IDC\_INFO  2001  
#define  IDM\_MAIN  4000  
#define  IDM\_OPEN  4001  
#define  IDM\_EXIT  4002  
  
ICO\_MAIN  ICON  "Main.ico"  
  
DLG\_MAIN  DIALOG  100,100,250,140  
STYLE    DS\_MODALFRAME | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU  
CAPTION  "PE 文件基本信息"  
MENU    IDM\_MAIN  
FONT    9,"宋体"  
{  
CONTROL  "",IDC\_INFO,"RichEdit20A",196 | ES\_WANTRETURN | WS\_CHILD | ES\_READONLY | WS\_VISIBLE |WS\_BORDER | WS\_VSCROLL | WS\_TABSTOP,0,0,249,140  
  
}  
  
IDM\_MAIN  menu  discardable  
{  
  popup  "文件(&F)"  
  BEGIN  
    menuitem  "打开文件(&O)...",  IDM\_OPEN  
    menuitem  separator  
    menuitem  "退出(&X)",    IDM\_EXIT  
  END  
}  
  
将这些代码放进 peinfo.rc 文件中，用 rc.exe 进行编译生成 peinfo.res 文件  
  
下来写主程序文件，也贴出代码，必要的地方有注释  
      .386  
      .model  flat,stdcall  
      option casemap:none  
  
include    windows.inc  
include    user32.inc  
include    kernel32.inc  
include    comdlg32.inc  
includelib  user32.lib  
includelib  kernel32.lib  
includelib  comdlg32.lib  
  
ICO\_MAIN  equ    2000  
DLG\_MAIN  equ    2000  
IDC\_INFO  equ    2001  
IDM\_MAIN  equ    4000  
IDM\_OPEN  equ    4001  
IDM\_EXIT  equ    4002  
  
      .data?  
hInstance  dd    ?  
hRichEdit  dd    ?  
hWinMain  dd    ?  
hWinEdit  dd    ?  
szFileName  db    MAX\_PATH dup (?)  
  
      .const  
szDllEdit  db    'RichEd20.dll',0  
szClassEdit  db    'RichEdit20A',0  
szFont    db    '宋体',0  
szExtPe    db    'PE Files',0,'\*.exe;\*dll;\*.scr;\*fon;\*.drv',0  
      db    'All Files(\*.\*)',0,'\*.\*',0,0  
szErr    db    '文件格式错误！',0  
szErrFormat  db    '这个文件不是 PE 格式的文件！',0  
  
szMsg    db    '文件名:%s',0dh,0ah  
      db    '--------------------------------------------',0dh,0ah  
      db    '运行平台：                   0x%04X',0dh,0ah  
      db    '节区数量：                   %d',0dh,0ah  
      db    '文件标记：                   0x%04X',0dh,0ah  
      db    '建议装入地址：               0x%08X',0dh,0ah,0  
szMsgSection  db    '---------------------------------------------',0dh,0ah  
        db    '节区名称   节区大小 虚拟地址 Raw\_ 尺寸  Raw\_ 偏移  节区属性',0dh,0ah  
        db    '----------------------------------------------',0dh,0ah  
szFmtSection  db    '%s  %08X  %08X  %08X  %08X  %08X',0dh,0ah,0,0  
            
;///////////////////////////////////////////////////////////////////////////  
        
      .code  
  
\_AppendInfo  proc  \_lpsz  
      local  @stCR:CHARRANGE  
  
      pushad  
      invoke  GetWindowTextLength,hWinEdit  
      mov    @stCR.cpMin,eax  
      mov    @stCR.cpMax,eax  
      invoke  SendMessage,hWinEdit,EM\_EXSETSEL,0,addr @stCR  
      invoke  SendMessage,hWinEdit,EM\_REPLACESEL,FALSE,\_lpsz  
      popad  
      ret  
\_AppendInfo  endp  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
\_ProcessPeFile  proc  \_lpFile,\_lpPeHead,\_dwSize  
        local  @szBuffer\[1024\]:byte,@szSectionName\[16\]:byte  
  
        pushad  
        mov    edi,\_lpPeHead  
        assume  edi:ptr IMAGE\_NT\_HEADERS  
  
        movzx  ecx,\[edi\].FileHeader.Machine  
        movzx  edx,\[edi\].FileHeader.NumberOfSections  
        movzx  ebx,\[edi\].FileHeader.Characteristics  
        invoke  wsprintf,addr @szBuffer,addr szMsg,\\  
            addr szFileName,ecx,edx,ebx,\[edi\].OptionalHeader.ImageBase  
        invoke  SetWindowText,hWinEdit,addr @szBuffer  
        ;----------------------------------------------  
        ;循环显示每个节区的信息  
        ;----------------------------------------------  
        invoke  \_AppendInfo,addr szMsgSection  
        movzx  ecx,\[edi\].FileHeader.NumberOfSections  
        add    edi,sizeof  IMAGE\_NT\_HEADERS  
        assume  edi:ptr  IMAGE\_SECTION\_HEADER  
        .repeat  
            push  ecx  
        ;-----------------------------------------------------------------  
        ;获取节的名称，由于节名名称不一定是以 0 结尾的，所以要进行处理  
        ;-----------------------------------------------------------------  
            invoke  RtlZeroMemory,addr @szSectionName,sizeof @szSectionName  
            push  esi  
            push  edi  
            mov    ecx,8  
            mov    esi,edi  
            lea    edi,@szSectionName  
            cld  
            @@:  
            lodsb  
            .if    !al  
                mov  al,' '  
            .endif  
            stosb  
            loop  @B  
            pop    edi  
            pop    esi  
        ;---------------------------------------------------------------------  
            invoke  wsprintf,addr @szBuffer,addr szFmtSection,\\  
                addr @szSectionName,\[edi\].Misc.VirtualSize,\\  
                \[edi\].VirtualAddress,\[edi\].SizeOfRawData,\\  
                \[edi\].PointerToRawData,\[edi\].Characteristics  
            invoke  \_AppendInfo,addr @szBuffer  
            add    edi,sizeof IMAGE\_SECTION\_HEADER  
            pop    ecx  
            .untilcxz  
            assume  edi:nothing  
            popad  
            ret  
  
\_ProcessPeFile  endp  
;/////////////////////////////////////////////////////////////////////////////////  
\_Init    proc    
      local  @stCf:CHARFORMAT  
  
      invoke  GetDlgItem,hWinMain,IDC\_INFO  
      mov    hWinEdit,eax  
      invoke  LoadIcon,hInstance,ICO\_MAIN  
      invoke  SendMessage,hWinMain,WM\_SETICON,ICON\_BIG,eax  
      invoke  SendMessage,hWinEdit,EM\_SETTEXTMODE,TM\_PLAINTEXT,0  
      invoke  RtlZeroMemory,addr @stCf,sizeof @stCf  
      mov    @stCf.cbSize,sizeof  @stCf  
      mov    @stCf.yHeight,9 \* 20  
      mov    @stCf.dwMask,CFM\_FACE or CFM\_SIZE or CFM\_BOLD  
      invoke  lstrcpy,addr @stCf.szFaceName,addr szFont  
      invoke  SendMessage,hWinEdit,EM\_SETCHARFORMAT,0,addr @stCf  
      invoke  SendMessage,hWinEdit,EM\_EXLIMITTEXT,0,-1  
      ret  
\_Init    endp  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
\_Handler  proc  C \_lpExceptionRecord,\_lpSEH,\_lpContext,\_lpDispatcherContext  
      pushad  
      mov    esi,\_lpExceptionRecord  
      mov    edi,\_lpContext  
      assume  esi:ptr  EXCEPTION\_RECORD,edi:ptr CONTEXT  
      mov    eax,\_lpSEH  
      push  \[eax + 0ch\]  
      pop    \[edi\].regEbp  
      push  \[eax + 8\]  
      pop    \[edi\].regEip  
      push  eax  
      pop    \[edi\].regEsp  
      assume  esi:nothing,edi:nothing  
      popad  
      mov    eax,ExceptionContinueExecution  
      ret  
\_Handler  endp  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
\_OpenFile  proc  
      local  @stOF:OPENFILENAME  
      local  @hFile,@dwFileSize,@hMapFile,@lpMemory  
      invoke  RtlZeroMemory,addr @stOF,sizeof @stOF  
      mov    @stOF.lStructSize,sizeof @stOF  
      push  hWinMain  
      pop    @stOF.hwndOwner  
      mov    @stOF.lpstrFilter,offset szExtPe  
      mov    @stOF.lpstrFile,offset szFileName  
      mov    @stOF.nMaxFile,MAX\_PATH  
      mov    @stOF.Flags,OFN\_PATHMUSTEXIST or OFN\_FILEMUSTEXIST  
      invoke  GetOpenFileName,addr @stOF  
      .if    !eax  
          jmp    @F  
      .endif  
;----------------------------------------------------------  
;打开文件并建立 Mapping  
;----------------------------------------------------------  
      invoke  CreateFile,addr szFileName,GENERIC\_READ,\\  
          FILE\_SHARE\_READ or FILE\_SHARE\_WRITE,NULL,\\  
          OPEN\_EXISTING,FILE\_ATTRIBUTE\_ARCHIVE,NULL  
      .if    eax != INVALID\_HANDLE\_VALUE  
          mov    @hFile,eax  
          invoke  GetFileSize,eax,NULL  
          mov    @dwFileSize,eax  
          .if    eax  
              invoke  CreateFileMapping,@hFile,\\  
                  NULL,PAGE\_READONLY,0,0,NULL  
              .if    eax  
                  mov    @hMapFile,eax  
                  invoke  MapViewOfFile,eax,\\  
                      FILE\_MAP\_READ,0,0,0  
                  .if    eax  
                      mov  @lpMemory,eax  
                    ;-------------------------------------  
                    ;创建用于错误处理的 SEH 结构  
                    ;-------------------------------------  
                      assume  fs:nothing  
                      push  ebp  
                      push  offset \_ErrFormat  
                      push  offset \_Handler  
                      push  fs:\[0\]  
                      mov    fs:\[0\],esp  
                      ;-------------------------  
                      ;检测 PE 文件是否有效  
                      ;-------------------------  
                      mov    esi,@lpMemory  
                      assume  esi:ptr  IMAGE\_DOS\_HEADER  
                      .if    \[esi\].e\_magic != IMAGE\_DOS\_SIGNATURE  
                          jmp    \_ErrFormat  
                      .endif  
                      add    esi,\[esi\].e\_lfanew  
                      assume  esi:ptr IMAGE\_NT\_HEADERS  
                      .if    \[esi\].Signature != IMAGE\_NT\_SIGNATURE  
                          jmp \_ErrFormat  
                      .endif  
                      invoke  \_ProcessPeFile,@lpMemory,esi,@dwFileSize  
                      jmp    \_ErrorExit  
                  \_ErrFormat:  invoke  MessageBox,hWinMain,addr szErrFormat,NULL,MB\_OK  
                  \_ErrorExit:  
                        pop    fs:\[0\]  
                        add    esp,0ch  
                        invoke  UnmapViewOfFile,@lpMemory  
                  .endif  
                  invoke  CloseHandle,@hMapFile  
              .endif  
              invoke  CloseHandle,@hMapFile  
            .endif  
        .endif  
@@:  
        ret  
\_OpenFile  endp  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
\_ProcDlgMain  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam  
        mov    eax,wMsg  
        .if    eax == WM\_CLOSE  
            invoke  EndDialog,hWnd,NULL  
        .elseif  eax == WM\_INITDIALOG  
            push  hWnd  
            pop    hWinMain  
            call  \_Init  
        .elseif  eax == WM\_COMMAND  
            mov    eax,wParam  
            .if    ax == IDM\_OPEN  
                call  \_OpenFile  
                ;invoke  MessageBox,NULL,NULL,NULL,NULL  
            .elseif  ax == IDM\_EXIT  
                invoke  EndDialog,hWnd,NULL  
            .endif  
        .else  
            mov    eax,FALSE  
            ret  
        .endif  
        mov    eax,TRUE  
        ret  
\_ProcDlgMain  endp  
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;  
start:  
    invoke  LoadLibrary,offset szDllEdit  
    mov    hRichEdit,eax  
    invoke  GetModuleHandle,NULL  
    mov    hInstance,eax  
    invoke  DialogBoxParam,hInstance,DLG\_MAIN,NULL,offset \_ProcDlgMain,NULL  
    invoke  FreeLibrary,hRichEdit  
    invoke  ExitProcess,NULL  
    end    start  
代码在这里这里显示的确够乱的，我会打包上去，供大家下载的  
把 peinfo.asm 编译成 peinfo.obj，再与 peinfo.res 连接，就大功告成了，一个查看 pe 文件头信息的工具就出来了

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PEinfo.rar  
 |