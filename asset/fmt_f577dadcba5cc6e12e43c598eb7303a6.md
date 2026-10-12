
```c
hooksocket.InitHookApi(::GetModuleHandle(NULL),0x00376000,0x0568);  

//装载 OpenGL:  

PDWORD pdwAddr=(PDWORD)((DWORD)::GetModuleHandle(NULL)+0x00375A00);  

*pdwAddr=(DWORD)fpnRender;  

pdwAddr  =(PDWORD)((DWORD)::GetModuleHandle(NULL)+0x002B5E69);  

*pdwAddr=(DWORD)((DWORD)::GetModuleHandle(NULL)+0x00375A00);  

}  

return 0;  

}  

//---------------------------------------------------
```

上面的代码是我针对某个游戏做的，由于游戏是基于 OpenGL 的，所以我就把 OpenGL 的 Render 修改下来，来处理我的代码

比如，我们要用到程序内部函数：

```c
//-----------------------------------------------------------
//原程序是如下的调用方法：
00664B75  .  50           push    eax
00664B76  .  8D55 8C      lea     edx, dword ptr [ebp-74]
00664B79  .  52           push    edx
00664B7A  .  A1 181E9E07  mov     eax, dword ptr [79E1E18]
00664B7F  .  83C0 14      add     eax, 14
00664B82  .  50           push    eax
00664B83  .  8B0D 141E9E07 mov     ecx, dword ptr [79E1E14]
00664B89  .  83C1 23      add     ecx, 23
00664B8C  .  51           push    ecx
00664B8D  .  E8 8E7CF2FF  call    0058C820  

//我把它做成 C 函数调用，这样我们就可以直接用这个函数来调用程序内部函数了：
int CMgMuAssistantApp::SetMainText(int x,int y,LPSTR str,int unk1,int unk2,int unk3)
{
  ShowText=(MAINTEXT)0x0058C820;
  ShowText(x,y,str,unk1,unk2,unk3);
  return 0;
}

//---------------------------------------------------------------------
```

下面是我的回调函数，我们发现我们再发送命令给目标程序时，目标程序就乖乖的按我们说的做，让它放那，它就放那，让它返回什么，它就乖乖返回什么，我们可以设置更多的命令来对程序操作：

```c
//--------------------------------------------------------------------
LRESULT CALLBACK fpnWndProc(HWND hDlg,UINT message,WPARAM wParam,LPARAM lParam)
{
  //保存窗口句柄:
  theApp.game.hWnd=hDlg;

  switch(message)
  {
  case WM_INITDIALOG:{break;}
  case WM_PAINT:{break;}
  case WM_CMD:
    {
      if((DWORD)wParam==0x01){PostQuitMessage(0);}
      //返回的是主程序句柄:
      if((DWORD)wParam==0x02){
        //保存主程序句柄:
        theApp.m_hWndMainApp=(HWND)lParam;
        //返回游戏句柄:
        ::SendMessage(theApp.m_hWndMainApp,WM_CMD,0x02,(LPARAM)hDlg);
      }
      if((DWORD)wParam==0x08){
        //连接主程序，这里我创建了 SOCKET 来网络传输数据:
        theApp.m_uPortMainApp=(UINT)lParam;
        pClient=new CClient;
        pClient->Close();
        pClient->Create();
        if(lstrcmp(theApp.m_strAddrMainApp,"")==0)
        {
          //获取主机地址:
          char hostname[MAX_PATH]={0};
          ::gethostname(hostname,MAX_PATH);
          hostent *addr=::gethostbyname(hostname);
          theApp.m_strAddrMainApp=inet_ntoa(*(struct in_addr *)addr->h_addr_list[0]);
        }
        if(pClient->Connect(theApp.m_strAddrMainApp,theApp.m_uPortMainApp)==FALSE)
        {
          ::SendMessage(theApp.m_hWndMainApp,WM_CMD,0x08,(LPARAM)-1);
        }else{::SendMessage(theApp.m_hWndMainApp,WM_CMD,0x08,(LPARAM)1);}
      }
      if((DWORD)wParam==0x10){    
        //修改标题,方便我们调用:
        ::SetWindowText(hDlg,_T("[MG] MU"));
      }
      if((DWORD)wParam==0x11){
        CRect rect;
        //获取窗口矩形:
        ::GetWindowRect(hDlg,&rect);
        //移动窗口:
        ::MoveWindow(hDlg,LOWORD(lParam),HIWORD(lParam),rect.Width(),rect.Height(),TRUE);
      }
      return TRUE; //直接返回,不让游戏回调处理,这个是我们的命令机密不能让程序知道:
    }
  case WM_COPYDATA:  //我们替我们的目标程序处理进程间通信消息，为了方便点:)
    {
      //wParam:窗口句柄
      //lParam:数据
      theApp.OnCopyData(CWnd::FromHandle((HWND)wParam),(COPYDATASTRUCT*)lParam); return TRUE;
    }
  default:{break;}
  }
  //转到游戏的回调函数:
  return fpsWndProc(hDlg,message,wParam,lParam);
}

//-----------------------------------------------------------------------------------------------
```

实际这回是在上回的基础而来的，我们总结一下就知道，当我们完全操作了回调函数时，一个应用程序就几乎成了你手中的蛋糕，想怎么样吃就怎么样吃:)。`KS` 在进程的通信和相互性上没做多少，我们很多时候都要用 `CreateRemoteThread` 这个函数来做 DLL 的加载，实际不用它的方法还是有很多的，只是没有被人发现而已，同过前面的说明，我想大家都会明白了，怎么样把一个程序完全操控，但是回过头来一想，如果我们对一个进程做了完全操作，我们可以再用这个进程操作其它的进程，那么会是烦琐点，但是是件很让人恐怖的事。前面的这个回调函数处理的事情很少，因为，我没把它做的有多强，我不想把它做多强 实际，当一个进程的回调函数交给你来处理，那么我们就可以做很多我们想做的事。当我们将 DLL 放入某个进程时，我们就可以调用它的任意一个函数，来为我们工作，我们只需要做影射，把我们要调用的函数和地址影射到我们的 DLL 中，然后再加工~~~~

:)^^^^

                      -By EasyStudy For PhantomNet 

 4dd466d0e1e0cc123afbe504da2462ca
