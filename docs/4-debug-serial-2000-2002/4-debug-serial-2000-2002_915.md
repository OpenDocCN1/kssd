
# 单词跑马灯

> 原文：[`www.pediy.com/kssd/pediy04/kanxue232.htm`](https://www.pediy.com/kssd/pediy04/kanxue232.htm)

## 软件注册算法
1. `(硬盘 ID(Hex)+806H) xor 1E3733H`，转为 10 进制 = 注册 ID
2. `(111A7BH-(注册 ID mod 3E8H)) xor 注册 ID`，转为 10 进制 = 序列号

## MASM32 注册机代码
以下是 MASM32 注册机：

```asm
;kg-pmd.asm==============================================
.386                          ;代码类型？
.model flat,stdcall           ;编译模式？
option casemap:none           ;大小写敏感
include hd.h                  ;头文件
_WinMain Proto :DWORD,:DWORD,:DWORD,:DWORD
Calculation Proto
.const
DLG_MAIN  equ     1000       ;与资源文件中的 ID 值定义要相符
IDC_UN    equ     1001
IDC_SN    equ     1002
ID_GEN    equ     1003
ID_EXIT   equ     1004
.data                           ;数据段定义，静态变量
MsgCap       db "单词跑马灯 Ver 1.0 注册机  by 5,555",0
MsgName      db "请输入您的机器码！",0
hInstance    dd 0
ID_Hex       dd 0
UserLen      dd 0
 VName      db 256 dup (0)
 FSName     db 256 dup (0)
 Drive      db "C:\\",0
 VID        dd 0
.data?                        ;数据段定义，动态变量
UserID      db 80 dup (?)
SN          db 80 dup (?)
.code
start:
invoke GetModuleHandle,NULL              ;取得应用程序的句柄
mov hInstance,eax
invoke DialogBoxParam,hInstance,DLG_MAIN,NULL,offset _WinMain,NULL ;调用 API 函数显示对话框，进入消息循环(_WinMain 函数处理这个循环)，参数含义自己查
invoke ExitProcess,NULL      ;退出消息循环后，结束进程(返回 NULL 值？)
_WinMain  proc     uses ebx edi esi, \  ;过程用到的寄存器，编译时自动 PUSH&POP。
           hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD ;几个系统参数
           mov    eax,wMsg                                ;取消息
           .if    eax == WM_CLOSE                         ;处理关闭窗口、初始化窗口及窗口按钮按下的动作,这里只列了三类
               invoke    EndDialog,hWnd,NULL
           .elseif eax == WM_INITDIALOG
               call GetID                                  ;预先取得跑马灯的用户 ID
               invoke SetDlgItemText, hWnd, IDC_UN, addr UserID ;显示在对话框中的“机器码”后
               invoke SetWindowText,hWnd,ADDR MsgCap            ;将 MsgCap 字串写到对话框标题栏上(原来 MsgCap 字串我是加密过的，用 UEDIT 不能改。这样即使用 exescope 修改了资源中的标题栏，标题信息也不会变：)
               invoke SendDlgItemMessage,hWnd,IDC_SN,EM_SETREADONLY,TRUE,NULL ;发送消息使注册码框只读
           .elseif eax == WM_COMMAND                       ;处理命令
               mov eax,wParam
               .IF lParam!=0                             ;？？？
                   .if eax==ID_GEN
                       invoke GetDlgItemText, hWnd, IDC_UN, addr UserID, 80  ;从 ID 为 IDC_UN(机器码)的文本框中取得字符串并放入 UserID 中
                       .if  eax!=NULL                                    ;如果并非没有输入，计算
                           mov UserLen,eax
                           invoke Calculation
                           invoke SetDlgItemText, hWnd, IDC_SN, addr SN      ;显示 SN 的内容到 ID 号 IDC_SN 的框中
                       .else
                           invoke MessageBox,NULL,addr MsgName,addr MsgCap,MB_OK or MB_ICONEXCLAMATION
                       .endif
                   .elseif eax==ID_EXIT
                       invoke SendMessage,hWnd,WM_CLOSE,NULL,NULL
                   .endif
               .ENDIF
           .else
               mov eax,FALSE       ;如果没有处理过消息，返回 False 值(放入任务队列，下次循环继续处理？)
               ret
           .endif
           mov eax,TRUE            ;否则返回 True
           ret
_WinMain  ENDP       
Calculation Proc
           pushad                              ;保存所有的寄存器值，懒人的方法：)。以下就不用解释了吧。
           mov ecx,UserLen
           lea ebp,UserID
           call Dec2Hex
           mov ID_Hex,eax
           mov ebp,3e8h
           cdq
           div ebp
           mov eax,111a7bh
           sub eax,edx
           xor eax,ID_Hex
           lea ebp,SN
           call Num2Dec
           popad
           ret
Calculation Endp
GetID       Proc
           pushad
           invoke  GetVolumeInformation, addr Drive, addr VName, 255, addr VID, 0, 0, addr FSName, 255
           mov eax,VID
           add eax,806h
           xor eax,1e3733h
           mov ID_Hex,eax
           lea ebp,UserID
           call Num2Dec
           popad
           ret
GetID       Endp
;===数字转换成 10 进制字符串,对 0 也同样适用
Num2Dec Proc  uses ebx edx edi  ;入口：eax:Num to convert, ebp-->buffer 出口：ecx:返回转换后的 ASC 长度
        xor edi,edi
        mov ebx,10
@Num2Dec_Lop1:
        cdq
        div ebx
        push edx
        inc edi
        test eax,eax
        jnz @Num2Dec_Lop1
        mov ecx,edi
@Num2Dec_Lop2:
        pop eax
        add al,'0'
        mov byte ptr [ebp],al
        inc ebp
        dec edi
        jnz @Num2Dec_Lop2
        mov byte ptr [ebp],0       ;加上结尾的 0
        ret
Num2Dec Endp
;===10 进制字符串转换成 16 进制数字,对 0 也同样适用
Dec2Hex Proc  uses ebx edx ecx edi    ;入口：ecx:字符串长度 ebp-->0 结尾的 10 进制字符串 出口：eax:返回值
        xor ebx,ebx
        mov edi,1
@Dec2Hex_Lop_1:
        xor eax,eax
        mov al,[ebp+ecx-1]
        sub al,'0'
        imul edi
        add ebx,eax
        imul edi,10
        loop @Dec2Hex_Lop_1
        mov eax,ebx
        ret
Dec2Hex Endp
        End start
```

## 资源文件
```rc
;kg-pmd.rc       
#include <e:\masm32\include\resource.h>
#define DLG_MAIN  1000
#define IDC_UN    1001
#define IDC_SN    1002
#define ID_GEN    1003
#define ID_EXIT   1004
DLG_MAIN DIALOG 64,53,240,52
STYLE DS_MODALFRAME | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
CAPTION "单词跑马灯 Ver 1.0 注册机  by 5,555"
FONT 9, "宋体"
{
DEFPUSHBUTTON "注册", ID_GEN, 180, 8, 50, 14
PUSHBUTTON "退出", ID_EXIT, 180, 26, 50, 14
EDITTEXT IDC_UN, 52, 10, 100, 12, WS_BORDER | WS_TABSTOP
EDITTEXT IDC_SN, 52, 29, 100, 13, WS_BORDER | WS_TABSTOP
LTEXT "机器码:", -1, 8, 12, 40, 10
LTEXT "注册码:", -1, 8, 30, 41, 9
}
```

## 头文件
```h
;hd.h********************************************
      include \masm32\include\windows.inc
      include \masm32\include\user32.inc
      include \masm32\include\kernel32.inc
      include \masm32\include\gdi32.inc
      include \masm32\include\comctl32.inc
      include \masm32\include\comdlg32.inc
      include \masm32\include\shell32.inc
      include \masm32\include\advapi32.inc
      includelib \masm32\lib\user32.lib
      includelib \masm32\lib\kernel32.lib
      includelib \masm32\lib\gdi32.lib
      includelib \masm32\lib\comctl32.lib
      includelib \masm32\lib\comdlg32.lib
      includelib \masm32\lib\shell32.lib
      includelib \masm32\lib\advapi32.lib
```
