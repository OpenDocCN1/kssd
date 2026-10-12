
```c
    strcat( modulePath, moduleName );
    
    // 检查待注入的模块是否存在
    WIN32_FIND_DATA WFD;
    if ( FindFirstFile( modulePath, &WFD ) == INVALID_HANDLE_VALUE )
    {
        char msg[512];
        strcpy( msg, "文件 " );
        strcat( msg, moduleName );
        strcat( msg, " 不存在!" );
        
        MessageBox( NULL, msg, appName, MB_OK | MB_ICONINFORMATION );
        
        PostMessage( appHwnd, WM_CLOSE, 0, 0 );
    }
    
    // 监视目标进程, 注入模块
    while ( true )
    {
        int interval = 0;
        int returnValue = 0;
        
        if ( InjectModule( exeName, modulePath ) == true )
        {
            interval = 1000;
            
            if ( strcmp( trayIcon.szTip, running ) != 0 )
            {      
                strcpy( trayIcon.szTip, running );
                
                Shell_NotifyIcon( NIM_MODIFY, &trayIcon );  
            }
        }
        else
        {
            interval = 10;
            
            if ( strcmp( trayIcon.szTip, monitor ) != 0 )
            {
                strcpy( trayIcon.szTip, monitor );
                
                Shell_NotifyIcon( NIM_MODIFY, &trayIcon );  
            }
        }
        
        Sleep( interval );
    }
    
    return 0;
}


LRESULT CALLBACK WinProc( HWND hwnd, UINT message, WPARAM wParam, LPARAM lParam )
{
    switch ( message )
    {
    case WM_TRAY:
    
        if ( (UINT)lParam == WM_RBUTTONUP )
            PostMessage( appHwnd, WM_CLOSE, 0, 0 );   
        
        break;
        
    case WM_CLOSE:        
        
        Shell_NotifyIcon( NIM_DELETE, &trayIcon );
        
        PostMessage( appHwnd, WM_QUIT, 0, 0 );  
        
        break;
        
    default:
        return DefWindowProc( hwnd, message, wParam, lParam );
    }
    
    return 0;
}


// 一个简单的例子 : Module.dll ////////////////////////////////////////////////
//
// 文件 : Module.cpp
// 
// 作者 : 刘泽围
//
// 日期 : 2007 年 01 月 17 日
//
// 描述 : 被我们用来注入的模块
///////////////////////////////////////////////////////////////////////////////
#include <windows.h>
#pragma comment( lib, "user32.lib" )
#pragma comment( linker, "/base:0x20000000" ) // 将 Dll 加载到一个非默认的地址可以防止
                                              // 首选地址加载失败对程序性能产生的影响
int WINAPI DllMain( HANDLE module, DWORD reason, LPVOID reserved )
{
    if ( reason == DLL_PROCESS_ATTACH )
        MessageBox( NULL, "Hello Pediy!", "Module.dll", MB_OK );
    
    return true;
}
```

Have fun![](img/9e827a3de2ad55eed0f95082425861b2.jpg "微笑")

`4dd466d0e1e0cc123afbe504da2462ca`
