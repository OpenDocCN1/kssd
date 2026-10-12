
# 管道应用之捕获控制台程序信息

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-615.htm`](https://www.pediy.com/kssd/pediy09/pediy09-615.htm)

前几天问了朋友关于怎样捕捉控制台信息的问题，原来是用了管道技术，而且我这儿还有这方面的资料，只是我平时没注意罢了，真是.......

下面就简单介绍一下管道，说白了就是进程或网络间通信，有两种管道，即有名管道和匿名管道。匿名管道就是没有名字的管道了，也就是说在使用它们时不需要知道其名字。而有名管道正好相反，在使用前必须知道其名字。

也可以根据管道的特性来分类，即是单向的还是双向的。单向管道，数据只能沿一个方向移动，从一端流向另一端，而双向管道数据可以在两端间自由交换。匿名管道通常是单向的而有名管道通常是双向的。有名管道常用于一个服务器联络多个客户端的网络环境。

网络的我不了解，先说说进程的罢，我这人表达能力不行，就举个例子吧，比如说我们现在要捕捉 `ping` 程序的输出信息用管道就很容易实现，我们首先在程序中创建一个管道(通过 `CreatePipe`)这样我们就得到了一个管道的写端口和一个读端口，然后我们通过 `CreateProcess` 创建控制台子进程，其中要用到 `STARTUPINFO` 这个结构，这个结构中就有控制台的三个标准句柄，(输入,输出,错误)我们要做的就是把上面 `CreatePipe` 中得到的写端口送给这里的输出端口，这样控制台应用程序的输出就会被我们偷偷的捕捉了，而它不知道这一切，它还是以为它是向控制台做标准输出，是不是挺爽的。说了这么多的废话下面是一个捕获 `PING` 程序输出信息的小程序，大家将就着看看吧!别扔砖啊!!
`[](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")`

```c
#include "windows.h"

#include "resource.h"

HWND hInst;

LRESULT CALLBACK PING(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{
  DWORD hWrite,hRead;
  STARTUPINFO startupinfo;
  PROCESS_INFORMATION pinfo;
  SECURITY_ATTRIBUTES sat;
  TCHAR  szBuffer[1024],ipBuffer[20];
  int bytesRead;
  HWND  hwndEdit;
  switch (message)
  {
    case WM_INITDIALOG:
      
        SendMessage(hDlg,WM_SETICON,ICON_BIG,LoadIcon(hInst,ICO_PING));
        return TRUE;

    case WM_COMMAND:
      switch (LOWORD(wParam)) 
      { 
      case IDC_EXIT: 
        EndDialog(hDlg, LOWORD(wParam));
        break; 
      case IDOK: 
        RtlZeroMemory(ipBuffer,20);
        
        strcpy(ipBuffer,"ping ");
        
        GetDlgItemText(hDlg,IDC_IPADDRESS,ipBuffer+5,20);
        
        hwndEdit=GetDlgItem(hDlg,IDC_OUTINFOR);
        
        sat.nLength=sizeof(SECURITY_ATTRIBUTES);
        
        sat.bInheritHandle=TRUE;
        
        sat.lpSecurityDescriptor=NULL;
        
        if(CreatePipe(&hRead,&hWrite,&sat,0)==0)    //创建匿名管道
          MessageBox(hDlg,TEXT("创建管道失败"),TEXT("PING"),MB_OK); 
        else
        {
          startupinfo.cb=sizeof(STARTUPINFO);
          
          GetStartupInfo(&startupinfo);
          
          startupinfo.hStdOutput=hWrite;     //用管道的写端代替控制台程序的输出端以便得到输出的信息
          
          startupinfo.hStdError=hWrite;
          
          startupinfo.dwFlags=STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES ;
          
          startupinfo.wShowWindow=SW_HIDE;        //隐藏控制台程序窗口
          
          if(CreateProcess(NULL, ipBuffer,NULL,NULL,TRUE,NULL,NULL,NULL,&startupinfo,&pinfo)==NULL)
            MessageBox(hDlg,TEXT("创建进程失败"),TEXT("PING"),MB_OK);
          
          else
          {
            CloseHandle(hWrite);        //关关闭写端,因为写端已经给了控制台程序,不能存在两个写端
            
            while(TRUE)
            {
                      RtlZeroMemory(szBuffer,1024); 
                      if(ReadFile(hRead,szBuffer,1023,&bytesRead,NULL)==NULL)  //注意 ReadFile 的第一个参数正是读端的句柄
                          break;
                      SendMessage(hwndEdit,EM_SETSEL,-1,0); 
                      SendMessage(hwndEdit,EM_REPLACESEL,FALSE,szBuffer);    //循环读入信息直到没有信息可读
                  }
          
          }
          
          CloseHandle(hWrite);
        }
        
        break;
      }
      
      return TRUE;
  }
    return FALSE;
}

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
  hInst=hInstance;
  DialogBoxParam(hInstance,(LPCTSTR)DLG_MAIN,NULL,(DLGPROC)PING,NULL);
  return 0;
}
```

每一次在捕捉控制台输出之前，应该先把命令回显的地方清空比较好点 `[](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")`

丢个汇编版本滴:

```asm
.386

.model flat, stdcall

option casemap :none

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; Include 文件定义

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

include    windows.inc

include    user32.inc

includelib  user32.lib

include    kernel32.inc

includelib  kernel32.lib

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; Equ 等值定义

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

ICO_MAIN  equ    1000h  ;图标

DLG_CMD  equ    911     

IDC_CmdShell  equ    9011

IDC_CreateCmdShell  equ    9012

IDC_JieGuo  equ    9013

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 数据段

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.data?

hInstance  dd    ?

szBuff db 1024 dup(?)

sat SECURITY_ATTRIBUTES<?>

startupinfo STARTUPINFO<?> 

pinfo PROCESS_INFORMATION<?>

lpBuff db 1024 dup(?)

bytesRead DWORD ?

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

; 代码段

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

_ProcDlgCmd  proc  uses ebx edi esi hWnd,wMsg,wParam,lParam
  local hRead,hWrite,hwndEdit
   mov  eax,wMsg
   
   .if  eax == WM_CLOSE
     invoke  EndDialog,hWnd,NULL
   .elseif  eax == WM_INITDIALOG
     invoke  LoadIcon,hInstance,ICO_MAIN
     
     invoke  SendMessage,hWnd,WM_SETICON,ICON_BIG,eax
   .elseif  eax == WM_COMMAND
     mov  eax,wParam
     
     .if  ax == IDC_CreateCmdShell
       invoke RtlZeroMemory,addr szBuff,sizeof szBuff
       
       invoke GetDlgItemText,hWnd,IDC_CmdShell,addr szBuff,sizeof szBuff
       
       invoke GetDlgItem,hWnd,IDC_JieGuo
       
       mov hwndEdit,eax
       
       mov sat.nLength,sizeof SECURITY_ATTRIBUTES
       
       mov sat.bInheritHandle,TRUE
       
       mov sat.lpSecurityDescriptor,NULL
       
       invoke CreatePipe,addr hRead,addr hWrite,addr sat,0
       
       .if eax==NULL
         mov eax,FALSE
         
         ret
       
       .endif
       
       mov startupinfo.cb,sizeof STARTUPINFO
       
       invoke GetStartupInfo,addr startupinfo
       
       push hWrite
       
       pop startupinfo.hStdOutput ;用管道的写端代替控制台程序的输出端以便得到输出的信息
       
       push hWrite
       
       pop startupinfo.hStdError
       
       mov startupinfo.dwFlags,STARTF_USESHOWWINDOW or STARTF_USESTDHANDLES
       
       mov startupinfo.wShowWindow,SW_HIDE;
       
       invoke CreateProcess,NULL,addr szBuff,NULL,NULL,TRUE,NULL,NULL,NULL,addr startupinfo,addr pinfo
       
       .if eax==NULL
         mov eax,FALSE
         
         ret
       
       .endif
       
       invoke CloseHandle,hWrite
       
       .while TRUE
         invoke RtlZeroMemory,addr lpBuff,sizeof lpBuff
         
         invoke ReadFile,hRead,addr lpBuff,1023,addr bytesRead,NULL
         
         .if eax==NULL
           ret
         
         .endif
       
       invoke SendMessage,hwndEdit,EM_SETSEL,-1,0  
                      invoke SendMessage,hwndEdit,EM_REPLACESEL,FALSE,addr lpBuff
       
       .endw
       
       invoke CloseHandle,hWrite
       
       .endif
     .else
       mov  eax,FALSE
       
       ret
     .endif
   
   mov  eax,TRUE
   
   ret


_ProcDlgCmd  endp

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

start:
   invoke  GetModuleHandle,NULL
   
   mov  hInstance,eax
   
   invoke  DialogBoxParam,hInstance,DLG_CMD,NULL,offset _ProcDlgCmd,NULL
   
   invoke  ExitProcess,NULL

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

   end  start
```

4dd466d0e1e0cc123afbe504da2462ca
