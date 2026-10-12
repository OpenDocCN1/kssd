
# 发个利用 `EnumJobs` 监控打印机的源码

> 原文：[`www.pediy.com/kssd/pediy10/92239.html`](https://www.pediy.com/kssd/pediy10/92239.html)

办公室里换了个打印机，没两天墨盒完了，很生气，决定监控，网上找了一些，要注册。还有些不能监控网络共享打印文档，决心自己写个简单的，当初想用 `HOOK`，失败，因为我发现打印机共享时，驱动的文件夹都共享了，可见不行，只好用 `EnumJobs`，采用定时器监控。

装那个机子都可监控，只要机子开着，不过最好装在打印报务器上。

代码:

```cpp
#include <windows.h>   
#include <iostream>
#include "resource.h"

int oldID[10];
typedef struct _PRINTPROCESSOROPENDATA
{
  PDEVMODE  pDevmode;
  LPWSTR  pDatatype;
  LPWSTR  pParameters;
  LPWSTR  pDocumentName;
  DWORD   JobId;
  LPWSTR  pOutputFile;
  LPWSTR  pPrinterName;
}PRINTPROCESSOROPENDATA,*PPRINTPROCESSOROPENDATA;


char * getinic(LPCSTR lb,LPCSTR name)
{
  char FilePath[MAX_PATH];    
  GetModuleFileName(NULL,FilePath,sizeof(FilePath));    
  char drive[_MAX_DRIVE];    
  char dir[_MAX_DIR];    
  char fname[_MAX_FNAME];    
  char ext[_MAX_EXT];    
  _splitpath(FilePath,drive,dir,fname,ext);    
  wsprintf(FilePath,"%s%s",drive,dir);
  strcat(FilePath,"\\date.ini");
  char dd[256];
  GetPrivateProfileString(lb,name,NULL,dd,256,FilePath);
  return dd;
}

int GetdocPages(LPDEVMODE pDevMode,LPSTR pwszDocName)
{
  HINSTANCE hdll=LoadLibrary("gdi32.dll");
  typedef HANDLE(*UGetSpoolFileHandle)(LPSTR pwszPrinterName,LPDEVMODE pDevmode,LPSTR pwszDocName);
  UGetSpoolFileHandle GetSpoolFileHandle=(UGetSpoolFileHandle)GetProcAddress(hdll,"GdiGetSpoolFileHandle");
  HANDLE hPrint=GetSpoolFileHandle(getinic("date","Printer"),pDevMode,pwszDocName);
  typedef DWORD(*UGdiGetPageCount)(HANDLE  SpoolFileHandle);
  UGdiGetPageCount GetPageCount=(UGdiGetPageCount)GetProcAddress(hdll,"GdiGetPageCount");
  int i=GetPageCount(hPrint);
  return i;
}

VOID CALLBACK TimerProc ( HWND hwnd, UINT message, UINT iTimerID, DWORD dwTime)
{
  HANDLE hPrinter; 
  //int ret=OpenPrinter("HP LaserJet M1005",&hPrinter,NULL);
  int ret=OpenPrinter(getinic("date","Printer"),&hPrinter,NULL);
  DWORD   pcbNeeded=0;   //所有打印字节数    
  DWORD   pcReturned=0;   //打印任务数    
  ret=EnumJobs(hPrinter,0,127,2,0,0,&pcbNeeded,&pcReturned);   //必要的一步，先取出打印机里任务的字节数，然后决定 jobs1 变量大小。//--------------------------①    
  int actNeed=0;   
  actNeed=pcbNeeded;    
    JOB_INFO_2 *jobs1=NULL;
  jobs1=new JOB_INFO_2[actNeed];
  ret=EnumJobs(hPrinter,0,127,2,(LPBYTE)jobs1,actNeed,&pcbNeeded,&pcReturned);



  for(int i=0;i<pcReturned;i++)
  {
  bool isfine=false;
  for(int n=0;n<oldID[0];n++)
  {
    if(jobs1[i].JobId==oldID[n+1]) isfine=true;
  }
  if(isfine) break;
  FILE *fp;
  if((fp=fopen(getinic("date","outfile"),"r"))==NULL)
  {
    fp=fopen(getinic("date","outfile"),"a");
    fputs("<table width=\"700\" border=\"0\" align=\"center\" cellspacing=\"0\">\n",fp);
    fputs("<tr><th><font size=\"+4\" color=\"#0000CC\" face=\"华文新魏\">打印机监控记录</font></th>\n",fp);
    fputs("</tr></table>\n",fp);
    fputs("<table width=\"700\" border=\"1\" align=\"center\" cellspacing=\"0\" bordercolor=\"#99CC33\"><tr>\n",fp);
    fputs("<th width=\"35\" bgcolor=\"#99CC33\">序号</th>\n",fp);
    fputs("<th width=\"273\" bgcolor=\"#99CC33\">文档名</th>\n",fp);
    fputs("<th width=\"73\" bgcolor=\"#99CC33\">用户</th>\n",fp);
    fputs("<th width=\"164\" bgcolor=\"#99CC33\">计算机</th>\n",fp);
    fputs("<th width=\"42\" bgcolor=\"#99CC33\">纸张</th>\n",fp);
    fputs("<th width=\"41\" bgcolor=\"#99CC33\">页数</th>\n",fp);
    fputs("<th width=\"42\" bgcolor=\"#99CC33\">份数</th></tr>\n",fp);

    fclose(fp);

  }else
  {
  fclose(fp);
  fp=fopen(getinic("date","outfile"),"a");
  fputs("<tr><td>",fp);
  char tmpBuf[260];
  _ultoa(jobs1[i].JobId,tmpBuf,10);
  fputs(tmpBuf,fp);
  fputs("</td><td>\n",fp);
  fputs(jobs1[i].pDocument,fp);

  fputs("</td><td>\n",fp);
  fputs(jobs1[i].pUserName,fp);

  fputs("</td><td>\n",fp);
  fputs(jobs1[i].pMachineName,fp);

  fputs("</td><td>\n",fp);
  if(jobs1[i].pDevMode->dmPaperSize==DMPAPER_A4)  fputs("A4",fp);
  if(jobs1[i].pDevMode->dmPaperSize==DMPAPER_B5)  fputs("B5",fp);
  if(jobs1[i].pDevMode->dmPaperSize==263)  fputs("16K",fp);
  fputs("</td><td>\n",fp);
  _ultoa(jobs1[i].TotalPages,tmpBuf,10);
  fputs(tmpBuf,fp);
  fputs("</td><td>\n",fp);
  //int pp=GetdocPages(jobs1[i].pDevMode,jobs1[i].pDocument);
  //_ultoa(pp,tmpBuf,10);
  _ultoa(jobs1[i].pDevMode->dmCopies,tmpBuf,10);
  fputs(tmpBuf,fp);
  fputs("</td></tr>\n",fp);
  fputs("\n",fp);
  fclose(fp);
  }
  }
  for(int i=0;i<pcReturned;i++)
  {
  oldID[i+1]=jobs1[i].JobId;
  }
  oldID[0]=pcReturned;
  delete   jobs1;    

}

void showmenu(HWND hdwnd)
{
  POINT point;//定义一个指针变量
  ::GetCursorPos(&point);
  SetForegroundWindow(hdwnd);//让菜单自动消失
  HMENU htaskmenu=LoadMenu(GetModuleHandle(NULL),MAKEINTRESOURCE(IDR_MENU1));//读入菜单，获得菜单句柄。
    htaskmenu=GetSubMenu(htaskmenu,0);//设置为右键点击弹出菜单
  TrackPopupMenuEx(htaskmenu,TPM_VERTICAL|TPM_LEFTALIGN,point.x,point.y,hdwnd,NULL);//在单击的位置上显示弹出菜单
}

void taskico(HWND hdwnd,bool ifshow)
{
NOTIFYICONDATA nid;
nid.cbSize=sizeof(NOTIFYICONDATA);
strcpy(nid.szTip,"打印机监控精灵");
nid.hWnd=hdwnd;
nid.uID=3;
nid.uCallbackMessage=WM_USER+10;
HICON hIcon = LoadIcon(GetModuleHandle(NULL),(LPCTSTR)IDI_ICON1);    
nid.hIcon=hIcon;
nid.uFlags=NIF_MESSAGE|NIF_ICON|NIF_TIP;
if(ifshow) Shell_NotifyIcon(NIM_ADD,&nid);    //在任务栏中加入一个图标
else Shell_NotifyIcon(NIM_DELETE,&nid);
}

void autorun()
{
LPCSTR str;
HKEY hRegKey;
BOOL bResult;
str="Software\\Microsoft\\Windows\\CurrentVersion\\Run";
if(RegOpenKey(HKEY_LOCAL_MACHINE, str, &hRegKey) != ERROR_SUCCESS) 
bResult=FALSE;
else
{
  char FilePath[MAX_PATH];    
  GetModuleFileName(NULL,FilePath,sizeof(FilePath));    
  char drive[_MAX_DRIVE];    
  char dir[_MAX_DIR];    
  char fname[_MAX_FNAME];    
  char ext[_MAX_EXT];    
  _splitpath(FilePath,drive,dir,fname,ext);    
::RegSetValueEx( hRegKey,fname,0,REG_SZ,(CONST BYTE *)FilePath,255);
}
}

LRESULT CALLBACK DBFunc(HWND hdwnd,UINT message,WPARAM wParam,LPARAM lParam) 
    { 
    switch(message){
  case WM_CREATE:
    autorun();
    taskico(hdwnd,true);
    oldID[0]=0;
    SetTimer(hdwnd,1,1000,TimerProc);
    break;
  case WM_USER+10:
    {
    UINT uMsg=lParam;//用户对任务栏图标进行的是什么操作
    switch(uMsg)
    {
    case WM_RBUTTONDOWN://如果是单击右键
    showmenu(hdwnd);
    break;
    }
    }
    break;
    case WM_COMMAND: 
    switch(LOWORD(wParam)){ 
    case ID_Menu:
    taskico(hdwnd,false);
    PostQuitMessage(0);
  break;
    case ID_MENU_40002:
    ShellExecute(hdwnd,"open",getinic("date","outfile"),NULL,NULL,SW_SHOW);
  break;
  }
  break;
    }
return DefWindowProc(hdwnd,message,wParam,lParam);
}


int APIENTRY WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow) 
{


static TCHAR szAppName[]=TEXT ("UUUUUU");
HWND hwnd;
MSG msg;
WNDCLASS wndclass;
wndclass.style =0;
wndclass.lpfnWndProc =DBFunc;
wndclass.cbClsExtra =0;
wndclass.cbWndExtra =0;
wndclass.hInstance =hInstance;
wndclass.hIcon =0;
wndclass.hCursor =0;
wndclass.hbrBackground =0;
wndclass.lpszMenuName =NULL;
wndclass.lpszClassName =szAppName;
if (!RegisterClass (&wndclass))
{
MessageBox (NULL,TEXT("Program requires Windows NT!"),szAppName, MB_ICONERROR);
return 0;
}
hwnd = CreateWindow (szAppName, NULL,WS_DISABLED,0, 0,0, 0,NULL, NULL, hInstance, NULL);
while (GetMessage(&msg, NULL, 0, 0))
{
TranslateMessage (&msg);
DispatchMessage (&msg);
}
return msg.wParam;
} 

```

其实有问题，就是天杀的 `word` 份数总是 1,除非打 `sp3`，水的很，后来有人说解析 `SPL`，要使用 `GdiGetPageCount`，但找不到 `DDK`，麻烦。写的乱莫笑！

下面是搞好的，注意修改 `ini` 中的内容

上传的附件

| `![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)` | | `Release.rar` |
| :--- | :--- | :--- |
