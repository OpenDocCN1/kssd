
# 扫雷自动化原理

那么我们如果加上 `( 30 + 18 / 2 )` 就可以点在第一个雷的中央。

## 自动扫雷的实现

```c
//------------------------------------------
//  AutoMineSweep.exe
//
//  本程序针对 32 位 Vista 系统中的扫雷编写
//
//  对于 64 位系统没有经过测试
//
//  rdsnow\[BCG\]\[PYG\]\[D.4s\]
//
//  13:43 2007/2/25
//
//------------------------------------------

#include <windows.h>

int WINAPI WinMain ( HINSTANCE  hInstance ,
                    HINSTANCE  hPrevInstance ,
                    PSTR    szCmdLine ,
                    int    iCmdShow )
{
  //寻找扫雷窗口，若无，则试着运行扫雷

  HWND hWnd = ::FindWindow ( NULL ,TEXT("扫雷"));
  TCHAR szPath\[80\] ;
  if ( !hWnd ){
    ::GetEnvironmentVariableW ( TEXT("ProgramFiles") ,szPath ,80 );
    lstrcat(szPath ,TEXT("\\\\Microsoft Games\\\\Minesweeper\\\\"));
    if ( ::ShellExecute( NULL ,TEXT("open") ,TEXT("Minesweeper.exe") ,
      NULL ,szPath ,SW\_SHOW ) > (HINSTANCE)32){
        ::Sleep ( 5000 );
        hWnd = ::FindWindow ( NULL ,TEXT("扫雷"));
    }
    else{
      MessageBox ( NULL ,TEXT("没有找到扫雷程序") ,NULL ,MB\_OK | MB\_ICONWARNING);
      return 0 ;
    }
  }

  //打开扫雷进程

  DWORD dwProcessId ;
  GetWindowThreadProcessId ( hWnd ,&dwProcessId );
  HANDLE hProcess = ::OpenProcess ( PROCESS\_ALL\_ACCESS ,FALSE ,dwProcessId );
  if ( !hProcess ){
    MessageBox ( NULL ,TEXT("打开进程失败") ,NULL ,MB\_OK | MB\_ICONWARNING);
    return 0 ;
  }

  //在进程内存中寻找程序入口点

  unsigned char byteCode = 0 ;
  DWORD dwCode ,dwCodeAddress = 0x382A ;
  while ( true ){
    ::ReadProcessMemory ( hProcess ,(LPCVOID)dwCodeAddress ,&byteCode ,1 ,NULL );
    if ( byteCode == 0xE8 ){
      ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwCodeAddress+1) ,&dwCode ,4 ,NULL );
      if ( dwCode == 0x43A ){
        ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwCodeAddress+6) ,&dwCode ,4 ,NULL );
        if ( dwCode == 0xFFFFFD4A )
          break;
        else dwCodeAddress += 0x10000 ;
      }
      else dwCodeAddress += 0x10000 ;
    }
    else dwCodeAddress += 0x10000 ;
  }
  dwCodeAddress &= 0xFFFF0000 ;

  //读取子控件 static 的句柄

  HANDLE hStatic;
  ::ReadProcessMemory ( hProcess ,(LPCVOID)( dwCodeAddress+0x53CE4 ) ,&hStatic ,4 ,NULL);

  //取得子控件 static 的窗口矩形

  RECT stWndRect ;
  ::GetWindowRect ( (HWND)hStatic , &stWndRect );

  //模拟鼠标点击第一个格子

  bool isMine\[30\]\[24\];
  POINT stCurrentPoint ,stNewPoint;
  ::GetCursorPos ( &stCurrentPoint );
  stNewPoint.x = stWndRect.left + 39;
  stNewPoint.y = stWndRect.top + 41;
  ::SetCursorPos ( stNewPoint.x ,stNewPoint.y );
  ::SetWindowPos ( hWnd ,HWND\_TOPMOST ,0 ,0 ,0 ,0 ,SWP\_NOSIZE | SWP\_NOMOVE );
  ::SendMessage ( hWnd ,WM\_LBUTTONDOWN ,NULL ,MAKELPARAM( stNewPoint.x ,stNewPoint.y ));
  ::SendMessage ( hWnd ,WM\_LBUTTONUP ,NULL ,MAKELPARAM( stNewPoint.x ,stNewPoint.y ));

  //读出雷所在的位置，并关闭进程句柄

  DWORD dwESI ,dwTotal ,dwLine ,dwColumn ,dwAddress ,dwMine;
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwCodeAddress + 0x531B8 ) ,&dwESI ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwESI+0xC) ,&dwESI ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwESI+4) ,&dwTotal ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwESI+8) ,&dwLine ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwESI+0xC) ,&dwColumn ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwESI+0x44) ,&dwAddress ,4 ,NULL);
  ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwAddress+0xC) ,&dwAddress ,4 ,NULL);
  unsigned char i,j;
  for( i=0 ;i<dwColumn ;i++){
    ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwAddress+4\*i) ,&dwMine ,4 ,NULL);
    ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwMine+0xC) ,&dwMine ,4 ,NULL);
    for ( j=0 ;j<dwLine ;j++ )
    ::ReadProcessMemory ( hProcess ,(LPCVOID)(dwMine+j) ,(LPVOID)&isMine\[i\]\[j\] ,1 ,NULL);
  }
  ::CloseHandle(hProcess);

  //模拟扫雷鼠标动作

  for ( i=0 ;i<dwLine ;i++ ){
    for ( j=0 ;j<dwColumn ;j++ ){
      ::SetCursorPos ( stNewPoint.x +18\*j ,stNewPoint.y +18\*i );
      if ( isMine\[j\]\[i\] ){
        //::Sleep( 500 );
        ::SendMessage ( hWnd ,WM\_RBUTTONDOWN ,NULL ,MAKELPARAM( stNewPoint.x +18\*j ,stNewPoint.y +18\*i ));
        ::SendMessage ( hWnd ,WM\_RBUTTONUP ,NULL ,MAKELPARAM( stNewPoint.x +18\*j ,stNewPoint.y +18\*i ));
      }
      else {
        ::SendMessage ( hWnd ,WM\_LBUTTONDOWN ,NULL ,MAKELPARAM( stNewPoint.x +18\*j ,stNewPoint.y +18\*i ));
        ::SendMessage ( hWnd ,WM\_LBUTTONUP ,NULL ,MAKELPARAM( stNewPoint.x +18\*j ,stNewPoint.y +18\*i ));
      }
    }
  }
  ::SetWindowPos ( hWnd ,HWND\_NOTOPMOST ,0 ,0 ,0 ,0 ,SWP\_NOSIZE | SWP\_NOMOVE );
  ::SetCursorPos ( stCurrentPoint.x ,stCurrentPoint.y );
  
  return 0 ;
}
```

实际操作时一定要注意在扫雷过程中尽量不要移动鼠标。会导致扫雷失败。

## 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2007 年 02 月 26 日 22:19:55
```
4dd466d0e1e0cc123afbe504da2462ca
```
