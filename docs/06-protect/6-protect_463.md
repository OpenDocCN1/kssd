
# 【五一礼物】PhantOm 插件笔记~!送给大家五一礼物吧~!

> 原文：[`www.pediy.com/kssd/pediy10/87522.html`](https://www.pediy.com/kssd/pediy10/87522.html)

明天五一了,把笔记发出来给大家学习一下,也算是个礼物吧,多余的话就不说了,直接看附件吧~!

需要说明的是`Delphi`部分,是最初研究的,所以很多地方都不对的,一边看汇编,一边记录,根本没调试,也没测试,只要编译通过就可以了,`Delphi`部分基本全了,只少驱动部分了.驱动我没心思看了~~!

`C`语言函数已经全部通过,也就是说功能代码没什么问题的,实际我是看着`DELPHI`写的`C`代码,后来`DELPHI`就再没修改了,直接用`C`了~!

注意:本次贴出来的是学习笔记,大家研究学习用就好了,别乱飞就是了,希望大家支持一下了,有需要研究和学习的只管下就好了

![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

`DELPHI`:

2 楼贴`C`代码...需要的看看~~!主要部分贴完了~~!

应该没了~~!

^\_^

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || PhantOm.rar  
|

```c
int mgHOD_ModfiyOllyDbgCaption( HWND hWnd )  
{  
  if( g_hInstOllyDbg == NULL )  
  {  
    //GetModuleFileName( NULL , szbuffer , MAX_PATH);  
    //ExtractFilePath  
    g_hInstOllyDbg = GetModuleHandle(NULL);  
    if( g_hInstOllyDbg == NULL ){ return -1; }  
  }  
  if( strcmpi( g_tagNmlInfo.szOllyDbgMainCaption ,_T("") ) == 0 )  
  {  
    strcpy( g_tagNmlInfo.szOllyDbgMainCaption ,_T("Pediy"));  
  }  
  if( strcmpi( g_tagNmlInfo.szOllyDbgCpuCaption ,_T("") ) == 0 )  
  {  
    strcpy( g_tagNmlInfo.szOllyDbgCpuCaption ,_T("T_T"));  
  }  
  //CPU WINDOW:  
  DWORD  dwAddr = (DWORD)g_hInstOllyDbg + 0x0B3963;  
  DWORD  dwReturn = 0;  
  //  
  WriteProcessMemory( GetCurrentProcess(), (LPVOID)dwAddr ,  
    g_tagNmlInfo.szOllyDbgCpuCaption, 0x06,&dwReturn);  
  //Main Window:  
  dwAddr = (DWORD)g_hInstOllyDbg + 0x0B71EE;  
  //  
  WriteProcessMemory( GetCurrentProcess(), (LPVOID)dwAddr ,  
    g_tagNmlInfo.szOllyDbgMainCaption, 0x07,&dwReturn);  
  dwAddr = (DWORD)g_hInstOllyDbg + 0x0B7218;  
  //  
  WriteProcessMemory( GetCurrentProcess(), (LPVOID)dwAddr ,  
    g_tagNmlInfo.szOllyDbgMainCaption, 0x07,&dwReturn);  
  dwAddr = (DWORD)g_hInstOllyDbg + 0x0C1A91;  
  //  
  WriteProcessMemory( GetCurrentProcess(), (LPVOID)dwAddr ,  
    g_tagNmlInfo.szOllyDbgMainCaption, 0x07,&dwReturn);  
  char szbuffer[100] = {0};  
  GetWindowText( g_hWndOllyDbg , szbuffer ,100 );  
  for( int i= 0 ; i < (int)strlen( szbuffer ) ; i++ )  
  {  
    if( g_tagNmlInfo.szOllyDbgMainCaption[i] == 0x00 ){ break; }  
    szbuffer[i] = g_tagNmlInfo.szOllyDbgMainCaption[i];  
  }  
  SetWindowText( g_hWndOllyDbg , szbuffer );  
  return 0;  
}  

int mgHOD_RemoveDebugBreakPoint( HWND hWnd ,DWORD tid )  
{  
  DWORD dwRet  = 0;  
  DWORD dwAddr = (DWORD)g_hInstOllyDbg + 0x034E37;  
  BYTE  buffer = 0x90;  
  //  
  if( *(PDWORD)(dwAddr) != 0x90 &&  *(PDWORD)(dwAddr) == 0x74 )  
  {  
    //je -> nop :  
    WriteProcessMemory( ::GetCurrentProcess() , (LPVOID)dwAddr, &buffer , 0x01 , &dwRet );  
    WriteProcessMemory( ::GetCurrentProcess() , (LPVOID)(dwAddr + 0x01), &buffer , 0x01 , &dwRet );  
    buffer = 0xEB;  
    dwAddr = (DWORD)g_hInstOllyDbg + 0x034E44;  
    WriteProcessMemory( ::GetCurrentProcess() , (LPVOID)dwAddr, &buffer , 0x01 , &dwRet );  
  }  
  return 0;  
}  

int mgHOD_CustomHandleException( HWND hWnd ,DWORD tid )  
{  
  DWORD dwAddr = (DWORD)g_hInstOllyDbg + 0x0AF6DC;  
  if( *(LPBYTE)(dwAddr) == 0x90 )  
  {  
    Addtolist( 0,-1 , _T("Status: Exceptions handler already patched. ") );  
    return 1 ;  
  }  
  if( *(LPBYTE)(dwAddr) != 0x00 )  
  {  
    Addtolist( 0,-1 , _T("Status: Exceptions handler error sign. ") );  
    return -1;  
  }  
  //  
  BYTE  bybuf[0x5E] =  
  {  
    0x90, 0x90, 0xA1, 0x14, 0x57, 0x4D, 0x00, 0x83, 0xF8, 0x08,  
    0x74, 0x33, 0xA1, 0x20, 0x57, 0x4D, 0x00, 0x3D, 0x05, 0x00,  
    0x00, 0xC0, 0x74, 0x27, 0x3D, 0x1E, 0x00, 0x00, 0xC0, 0x74,  
    0x20, 0x3D, 0x01, 0x00, 0x00, 0x80, 0x74, 0x19, 0x3D, 0x1D,  
    0x00, 0x00, 0xC0, 0x74, 0x12, 0x3D, 0x94, 0x00, 0x00, 0xC0,  
    0x74, 0x0B, 0x90, 0x68, 0x14, 0x57, 0x4D, 0x00, 0xE9, 0x54,  
    0x9F, 0xF8, 0xFF, 0xBB, 0x01, 0x00, 0x01, 0x80, 0x53, 0xA1,  
    0x1C, 0x57, 0x4D, 0x00, 0x50, 0x8B, 0x15, 0x18, 0x57, 0x4D,  
    0x00, 0x52, 0xFF, 0x15, 0xB8, 0xD2, 0x50, 0x00, 0xE9, 0x43,  
    0x99, 0xF8, 0xFF, 0x90  
  };  
  //  
  DWORD dwRet = 0 ;  
  //  
  WriteProcessMemory( GetCurrentProcess(), (LPVOID)dwAddr , bybuf , 0x5E , &dwRet );  
  //  
  dwAddr = (DWORD)g_hInstOllyDbg + 0x03966A ;  
  if( *(LPBYTE)(dwAddr) != 0x68 )  
  {  
    Addtolist( 0,-1 , _T("Status: Exceptions handler already patched. ") );  
    return -2;  
  }  
  DWORD dwbuf = 0x07606D;  
  BYTE  bydat = 0xE9;  
  //  
  WriteProcessMemory( GetCurrentProcess() , (LPVOID)dwAddr , &bydat , 0x01 , &dwRet );  
  WriteProcessMemory( GetCurrentProcess() , (LPVOID)( dwAddr + 0x01 ) , &dwbuf , 0x04, &dwRet );  
  //  
  Addtolist( 0,-1 , "Status: Exceptions handler patched. ");  
  return 0;  
}  

int mgHOD_PatchODStringAndFPUBugs( HWND hWnd ,DWORD tid )  
{  
  DWORD dwAddr = (DWORD)g_hInstOllyDbg + 0x0AA2EE;  
  BYTE  bydat = 0xEB;  
  //  
  DWORD dwRet = 0 ;  
  if( *(LPBYTE)(dwAddr) == 0x74 )  
  {  
    WriteProcessMemory( GetCurrentProcess() , (LPVOID)dwAddr , &bydat , 0x01 , &dwRet );  
    Addtolist( 0,-1 , _T("Status: FPU bug patched. "));  
  }  
  //  
  dwAddr = (DWORD)g_hInstOllyDbg + 0x0313A5;  
  bydat  = 0x90;  
  if( *(LPBYTE)(dwAddr) == 0x90 )  
  {  
    Addtolist( 0,-1 , _T("Status: OutputDebugString already patched. "));  
    return 1;  
  }  
  //  
  if( *(LPBYTE)(dwAddr) != 0xE8 )  
  {  
    Addtolist( 0,-1 , _T("Status: OutputDebugString error sign. "));  
    return -1;  
  }  
  //  
  WriteProcessMemory( GetCurrentProcess() , (LPVOID)dwAddr , &bydat , 0x01 , &dwRet );  
  //  
  DWORD dwbuf = 0x90909090;  
  WriteProcessMemory( GetCurrentProcess() , (LPVOID)(dwAddr + 0x01) , &dwbuf , 0x04 , &dwRet );  
  //  
  Addtolist( 0,-1 , _T("Status: OutputDebugString patched. ") );  
  return 0;  
}  

int mgHOD_BlockInput(HWND hWnd)  
{  
  BYTE bydat[0x06]={0x33, 0xC0, 0x40, 0xC2, 0x04, 0x00};  
  //  
  DWORD dwFunAddr = (DWORD)GetProcAddress( GetModuleHandle("USER32.DLL"),"BlockInput");  
  if( dwFunAddr == NULL )  
  {  
    Addtolist( 0,-1 , _T("Unable get original pointer. "));  
    return -1;  
  }  
  BYTE bybuf[0x10] = {0};  
  //  
  if( Readmemory( bybuf , dwFunAddr , 0x0F , MM_SILENT ) != 0x0F )  
  {  
    Addtolist( 0,-1 , _T("Unable read original pointer. "));  
    return -1;  
  }  
  int logon = 0;  
  //  
  if( bybuf[0x09] == 0xCD && bybuf[0x0A] == 0x2E )  
  {  
    logon = 0;  
  }else{  
    logon = 0x0A;  
  }  
  //  
  if( logon == 0)  
  {  
    Addtolist( 0,-1 , _T("Unable detect function. "));  
    return -1;  
  }  
  //  
  if( Writememory( bydat , (dwFunAddr + 0x0A) , 0x06 , MM_SILENT ) != 0x06)  
  {  
    Addtolist( 0,-1 , _T("Unable hook BlockInput. "));  
    return -1;  
  }  
  Addtolist( 0,-1 , _T("Status: BlockInput hooked.  "));  
  return 0;  
}  

//--------------------------------------------------------------------------------------------------------  
//  
//phantom plugin:  

int mgHOD_HideFormPEB1( LPVOID lpAddr )  
{  
  DWORD  dwAddr = (DWORD)lpAddr + 0x30;  
  DWORD dwbuf = 0;  
  DWORD dwRet = 0;  
  BYTE  bybuf = 0;  
  //  
  if( ReadProcessMemory( (HANDLE)Plugingetvalue(VAL_HPROCESS) , (LPVOID)dwAddr , &dwbuf , 0x04, &dwRet )  )  
  {  
    Readmemory( &dwAddr ,( dwbuf + 0x10 ), 0x04 , MM_SILENT );  
    //  
    int n = 0 ;  
    do{  
      Writememory( &bybuf , ( dwAddr + 0x4C + n ) , 0x01 , MM_SILENT );  
      n++;  
    }while( n != 0x1C);  
    return 1;  
  }  
  return 0;  
}  

//--------------------------------------------------------------------------------------------------------  
//  
//phantom plugin:  
```



```c
int      mgHOD_HideFormPEB2( LPVOID lpAddr , int nType )

{
  DWORD  dwAddr = (DWORD)lpAddr + 0x30;
  DWORD dwbuf = 0;
  DWORD dwRet = 0;
  BYTE  bybuf = 0;
  //
  if( ReadProcessMemory( (HANDLE)Plugingetvalue(VAL_HPROCESS) , (LPVOID)dwAddr , &dwbuf , 0x04, &dwRet )  )
  {
    if( nType != 0 && nType != 2 || WriteProcessMemory( (HANDLE)Plugingetvalue(VAL_HPROCESS) ,
      (LPVOID)(dwbuf + 0x02), &bybuf , 0x01 , &dwRet  ) == TRUE )
    {
      bybuf = 0x01;
      if( nType != 1 || WriteProcessMemory( (HANDLE)Plugingetvalue(VAL_HPROCESS) ,
        (LPVOID)(dwbuf + 0x02), &bybuf , 0x01 , &dwRet) == TRUE )
      {
        return 1;
      }
    }
  }
  return 0;
}

int      mgHOD_GetTickCount(HWND hWnd)
{
  HMODULE  hNTDLL = GetModuleHandle( "ntdll.dll" );
  HMODULE hKernel= GetModuleHandle( "KERNEL32.DLL" );
  //
  DWORD dwGetTickCount = (DWORD)::GetProcAddress( hKernel ,"GetTickCount");
  if(  dwGetTickCount == NULL || dwGetTickCount < (DWORD)hKernel)
  {
    Addtolist( 0,-1 , _T("Unable get original pointer."));
    return -1;
  }

  BYTE bybuf1[0x10]={0};
  //
  if( Readmemory( bybuf1 , dwGetTickCount , 0x0F , MM_SILENT ) != 0x0F )
  {
    Addtolist( 0,-1 , _T("Unable read original function."));
    return -1;
  }

  DWORD dwCount = GetTickCount();
  //向调试进程分配空间:
  DWORD  dwNewBuf = (DWORD)mgHOD_VirtualAllocEx() ;
  DWORD  dwAddr   = dwNewBuf + 0x100;
  Writememory( &dwCount , ( dwAddr + 0x04 ) , 0x04 , MM_SILENT );
  BYTE bycode[0x20] ={
    0x0F, 0xAC, 0xD0, 0x18, 0x90, 0x90, 0x90, 0x90, 0x90, 0x90,
    0x90, 0xB9, 0x04, 0x01, 0x17, 0x00, 0x8B, 0x01, 0x83, 0xC0,
    0x01, 0x89, 0x01, 0x50, 0xC1, 0xE8, 0x08, 0x8B, 0xD0, 0x58,
    0xC3, 0x90};
  *(LPDWORD)&bycode[0x0C] =  ( dwAddr + 0x04 );
  //offset 0x70 write:
  Writememory( bycode , ( dwNewBuf + 0x70 ) , 0x1F ,MM_SILENT );
  //
  DWORD  dwA = dwGetTickCount + 0x0A;
  DWORD  dwB = dwNewBuf + 0x70 - dwA - 5 ;
  //
  if( *(LPDWORD)&bybuf1[2] == 0x909090F3 )
  {
    dwA = dwGetTickCount + 0x2E;
  }
  //
  BYTE   bybuf2 = 0xE9;
  if( Writememory( &bybuf2 , dwA , 0x01 , MM_SILENT ) != 0x01 )
  {
    Addtolist( 0,-1 , _T("Unable hook GetTickCount."));
    return -1;
  }
  //
  dwA = dwGetTickCount + 0x0B;
  if( *(LPDWORD)&bybuf1[2] == 0x909090F3 )
  {
    dwA = dwGetTickCount + 0x2F;
    dwB = dwNewBuf + 0x70 - dwA - 4;
    bybuf2 = 0x90;
    Writememory( &bybuf2 , (dwA + 0x04), 0x01 ,MM_SILENT );
    Writememory( &bybuf2 , (dwA + 0x05), 0x01 ,MM_SILENT );
  }
  if( Writememory( &dwB , dwA , 0x04 , MM_SILENT ) != 0x04 )
  {
    Addtolist( 0,-1 , _T("Unable hook GetTickCount."));
    return -1;
  }
  //
  Addtolist( 0, -1 , "Status: GetTickCount hooked .");
  return 0;
}

//--------------------------------------------------------------------------------------------------------
//
//phantom plugin:
int      mgHOD_ProtectDRx(HWND hWnd)
{
  mgHOD_KiUserExceptionDispatcher( hWnd );
  mgHOD_ZwContinue( hWnd );
  return 0;
}

int      mgHOD_KiUserExceptionDispatcher(HWND hWnd)
{
  HMODULE  hNTDLL = GetModuleHandle( "ntdll.dll" );
  //
  DWORD dwApiAddr = (DWORD)::GetProcAddress( hNTDLL ,"KiUserExceptionDispatcher");
  if(  dwApiAddr == NULL || dwApiAddr < (DWORD)hNTDLL )
  {
    Addtolist( 0,-1 , _T("Unable get original pointer."));
    return -1;
  }
  //
  DWORD dwbuf1 = 0 ;
  //
  if( Readmemory( &dwbuf1 , ( dwApiAddr + 0x0A ), 0x04 , MM_SILENT ) != 0x04 )
  {
    Addtolist( 0,-1 , _T("Unable read original function."));
    return -1;
  }
  //向调试进程分配空间:
  DWORD  dwNewBuf = (DWORD)mgHOD_VirtualAllocEx() ;
  DWORD  dwAddr   = dwNewBuf + 0x3C;
  DWORD  dwA = dwAddr - ( dwApiAddr + 0x09 ) - 0x05;
  //write JMP xxxxxxxx:
  BYTE  bybuf = 0xE9;
  if( Writememory( &bybuf , ( dwApiAddr + 0x09 ) , 0x01 , MM_SILENT ) != 0x01 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  if( Writememory( &dwA , ( dwApiAddr + 0x0A ) , 0x04 , MM_SILENT ) != 0x04 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  //
  BYTE  bycode[0x30] ={
      0x60, 0x83, 0xC1, 0x04, 0x8B, 0xF1, 0x8B, 0xD1, 0xBF, 0x00,
      0x00, 0x08, 0x00, 0xB9, 0x18, 0x00, 0x00, 0x00, 0xFC, 0xF3,
      0xA4, 0xBE, 0x18, 0x00, 0x08, 0x00, 0x8B, 0xFA, 0xB9, 0x18,
      0x00, 0x00, 0x00, 0xF3, 0xA4, 0x90, 0x90, 0x61, 0x8B, 0xC0,
      0x90, 0x90};
  //modfiy code:
  *(LPDWORD)&bycode[0x09] = dwNewBuf;
  *(LPDWORD)&bycode[0x16] = dwNewBuf + 0x18;
  dwAddr = dwNewBuf + 0x3C - 0x01 ;
  bybuf = 0x90;
  if( Writememory( &bybuf , dwAddr , 0x01 , MM_SILENT ) != 0x01 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  dwAddr = dwNewBuf + 0x3C;
  if( Writememory( &bycode , dwAddr , 0x26 , MM_SILENT ) != 0x26 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  //write call xxxxxxxx:
  dwAddr += 0x26;
  bybuf = 0xE8;
  if( Writememory( &bybuf , dwAddr , 0x01 , MM_SILENT ) != 0x01 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  dwAddr ++;
  DWORD dwB = dwbuf1 + 0x05 + ( dwApiAddr + 0x09 );
  dwA = dwB - (dwAddr - 0x01) - 0x05 ;
  if( Writememory( &dwA , dwAddr , 0x04 , MM_SILENT ) != 0x04 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  //
  dwAddr += 0x04;
  bybuf = 0xE9;
  if( Writememory( &bybuf , dwAddr , 0x01 , MM_SILENT ) != 0x01 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  //
  dwB = dwApiAddr + 0x09 + 0x05;
  dwA = dwB - dwAddr - 0x05;
  if( Writememory( &dwA , ( dwNewBuf + 0x3C + 0x26 + 0x01 + 0x05 ) , 0x04 , MM_SILENT ) != 0x04 )
  {
    Addtolist( 0,-1 , _T("Unable hook KiUserExceptionDispatcher ."));
    return -1;
  }
  //
  Addtolist( 0, -1 , "Status: KiUserExceptionDispatcher hooked .");
  return 0;
}

int      mgHOD_ZwContinue(HWND hWnd)
{
  HMODULE  hNTDLL = GetModuleHandle( "ntdll.dll" );
  //
  DWORD dwApiAddr = (DWORD)::GetProcAddress( hNTDLL ,"ZwContinue");
  if(  dwApiAddr == NULL || dwApiAddr < (DWORD)hNTDLL )
  {
    Addtolist( 0,-1 , _T("Unable get original pointer."));
    return -1;
  }
  //
  BYTE bybuf1[0x10] = {0};
  if( Readmemory( bybuf1 , dwApiAddr, 0x0F , MM_SILENT ) != 0x0F )
  {
    Addtolist( 0,-1 , _T("Unable read original function."));
    return -1;
  }
  //
  DWORD dwA = 0;
  if( bybuf1[0x09] == 0xCD && bybuf1[0x0A] == 0x2E )
  {
    dwA = 1;
  }else{
    dwA = 0;
  }
  if( dwA == 0xFF )
  {
    Addtolist( 0,-1 , _T("Unable detect function."));
    return -1;
  }
  //向调试进程分配空间:
  DWORD  dwNewBuf = (DWORD)mgHOD_VirtualAllocEx() ;
  //
  BYTE bybuf2 = 0xE9;
  DWORD dwB   = 0;
  if( dwA == 1 )
  {
    if(Writememory( &bybuf2 , (dwApiAddr + 0x09) , 0x01 , MM_SILENT ) != 0x01 )
    {
      Addtolist( 0,-1 , _T("Unable hook ZwContinue."));
      return -1;
    }
    dwB = dwNewBuf + 0x9E - ( dwApiAddr + 0x09 ) - 5 ;
    if( Writememory( &dwB , (dwApiAddr + 0x0A) , 0x04 , MM_SILENT ) != 0x04 )
    {
      Addtolist( 0,-1 , _T("Unable hook ZwContinue."));
      return -1;
    }
  }else{
    if( Writememory( &bybuf2 , (dwApiAddr + 0x0A) , 0x01 , MM_SILENT ) != 0x01 )
    {
      Addtolist( 0,-1 , _T("Unable hook ZwContinue.."));



```
      return -1;  
  }  
  dwB = dwNewBuf + 0x9E - ( dwApiAddr + 0x0A ) - 5 ;  
  if( Writememory( &dwB , (dwApiAddr + 0x0B) , 0x04 , MM_SILENT ) != 0x04 )  
  {  
    Addtolist( 0,-1 , _T("Unable hook ZwContinue."));  
    return -1;  
  }  

  //  
  dwB = dwNewBuf + 0x9E;  
  BYTE  bycode[0x30] ={  
    0x8B, 0x44, 0x24, 0x04, 0x0F, 0xBA, 0x30, 0x04, 0x60, 0x83,  
    0xC0, 0x04, 0x8B, 0xF0, 0xBF, 0x18, 0x00, 0x08, 0x00, 0xB9,  
    0x18, 0x00, 0x00, 0x00, 0xFC, 0xF3, 0xA4, 0x8B, 0xF8, 0xBE,  
    0x00, 0x00, 0x08, 0x00, 0xB9, 0x18, 0x00, 0x00, 0x00, 0xF3,  
    0xA4, 0x61};  
  //  
  *(LPDWORD)&bycode[0x0F] = dwNewBuf + 0x18;  
  *(LPDWORD)&bycode[0x1E] = dwNewBuf;  
  //  
  if( Writememory( &bycode , dwB , 0x2A , MM_SILENT ) != 0x2A )  
  {  
    Addtolist( 0,-1 , _T("Unable hook ZwContinue."));  
    return -1;  
  }  
  if( dwA == 1 )  
  {  
    Writememory( &bybuf1 , ( dwB + 0x2A ) , 0x0E , MM_SILENT );  
  }else{  
    Writememory( &bybuf1 , ( dwB + 0x2A ) , 0x0F , MM_SILENT );  
  }  
  //  
  Addtolist( 0, -1 , "Status: ZwContinue hooked .");  
  return 0;  
}
```
