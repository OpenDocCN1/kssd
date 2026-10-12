
# 发一个汇编写的“QQ 性别名称任意修改器”

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-598.htm`](https://www.pediy.com/kssd/pediy09/pediy09-598.htm)

> 根据易论坛“近在眼前”提供的易程序源码改编而成，这里表示感谢！

> QQ 性别修改器，可以修改性别为任意字符（不能超过 2 个字节）

## 代码

```asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
;   Programmed by hacker0058, nohacks@126.com
;   Website: http://nohacks.ys168.com
;             QQ 性别修改器 V1.0
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

      .386 
   .model flat,stdcall 
   option casemap:none 
      
      ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
   ;        Include 数据
   ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
   include windows.inc 
   include kernel32.inc 
   include shell32.inc
   include debug.inc
   includelib kernel32.lib 
   include user32.inc 
   includelib user32.lib 
   includelib shell32.lib
      
      ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
   ;        Equ 数据 RC 资源
   ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
         DLG_MAIN        equ         1000
   IDC_EDT             equ         1001
   IDC_IDOK          equ         1002
         
      ;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
      .data 
         .data?
      ls db 10 dup (?)
      sextemp db 10 dup (?)
      hInstance HINSTANCE ?   
      
      .code 
      ;********************************************************************
   _ProcDlgMain        proc        uses ebx edi esi, \
                        hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD
                       
                       
                       
      LOCAL hwnd,hwnd1,hwnd2,hwnd3,hwnd4,hwnd5,idx
   
        mov eax,wMsg
     
        cmp eax,WM_CLOSE
        je  boxClose
        cmp eax,WM_INITDIALOG
        je  boxStart
        cmp eax,WM_COMMAND
        je commoand
    
    retFalse:
          mov        eax,FALSE
          ret      
           
          
    boxClose:
        invoke EndDialog,hWnd,NULL
        jmp retTrue
           
    boxStart:
           
           invoke SetWindowPos,hWnd,HWND_TOPMOST,0,0,0,0, SWP_NOMOVE or SWP_NOSIZE ;窗口置顶
           jmp retTrue
               
    commoand:
        mov eax,wParam
        
    .if eax == IDC_IDOK
           
        invoke  FindWindow,CTEXT("#32770"), CTEXT("QQ2007 设置")
        
        .if eax==0
            
        invoke  FindWindow,CTEXT("#32770"), CTEXT("QQ2006 设置")
            
        .endif
        
           
        .if eax!=0
            
           mov hwnd,eax
           
           invoke  FindWindowEx ,hwnd, 0, CTEXT("#32770"), 0
            
           mov hwnd1,eax
           
           invoke  FindWindowEx ,hwnd1, 0, CTEXT("#32770"), 0
           
           mov hwnd2,eax
           
           invoke    GetDlgItem ,hwnd2, 1356
           
           mov hwnd3,eax
           
           invoke    GetDlgItem ,hwnd2, 1345
           
           mov hwnd4,eax
           
           invoke    GetDlgItem ,hwnd1, 1343
           
           mov hwnd5,eax
           
           invoke SendMessage,hwnd3,32,0,0
           
           invoke GetDlgItemText,hWnd,IDC_EDT ,addr sextemp,10  ;取用户输入到文本
           
           invoke SendMessage,hwnd3,323,0, addr sextemp
           
           mov idx,eax
           
           invoke SendMessage,hwnd3,334,idx,0
           
           invoke  SendMessage ,hwnd4, 13, 10,addr ls
           
           invoke  SendMessage ,hwnd4, 12, 0, addr ls
           
           invoke SendMessage,hwnd5, 245, 0, 0      
           jmp retTrue
           
           
        .else
           
           invoke MessageBox,0,CTEXT("请先打开 QQ 个人设置窗口！"),CTEXT("提示"),MB_OK   
           
        .endif
           
    .endif
           
    retTrue:
           
          mov        eax,TRUE
          ret        
      _ProcDlgMain        endp              

      ;********************************************************************
      start:
   invoke GetModuleHandle,NULL
   mov hInstance,eax
   invoke DialogBoxParam,eax,DLG_MAIN,NULL,offset _ProcDlgMain,0  ;显示对话框
   invoke ExitProcess,NULL
      
      ;********************************************************************
      end        start
```

`4dd466d0e1e0cc123afbe504da2462ca`
