
# 简单获取外网 IP 地址

> 原文：[`www.pediy.com/kssd/pediy14/144216.html`](https://www.pediy.com/kssd/pediy14/144216.html)

```cpp
#include "stdafx.h"  

#include "获取外网 IP 地址.h"  

#include <WININET.H>  

#pragma   comment(lib,"Wininet.lib")  

#ifdef _DEBUG  

#define new DEBUG_NEW  

#undef THIS_FILE  

static char THIS_FILE[] = __FILE__;  

#endif  

/////////////////////////////////////////////////////////////////////////////  

// The one and only application object  

CWinApp theApp;  

using namespace std;  

//////////////////////////////////////////////  

char * getip()  

{  

  static char Add[256];  

  HINTERNET hit=InternetOpen("ie",INTERNET_OPEN_TYPE_PRECONFIG, NULL, NULL, 0);  

  if(hit)  

  {  

  HINTERNET hIurl = InternetOpenUrl(hit,"http://www.baidu.com/s?wd=ip",NULL,-1,0,0);   

  if(hIurl)  

  {  

    char buf[100000];  

    memset(buf,0,100000);  

    DWORD dwR=0;  

    if(InternetReadFile(hIurl,buf,100000,&dwR))  

    {  

      char * s=buf;  

      char * s1=strstr(s,"本机 IP: <strong>");  

      s1=s1+21;  

      char * s2=strstr(s1,"</strong>   ");  

      int count=s2-s1;  

      strncpy(Add,s1,count);  

    }  

    else  

    {  

      strcpy(Add,"");  

    }  

  }  

  else  

  {  

    strcpy(Add,"");  

  }  

  }  

  else  

  {  

    strcpy(Add,"");  

  }  

  return Add;  

}  

int _tmain(int argc, TCHAR* argv[], TCHAR* envp[])  

{  

  int nRetCode = 0;  

  CString strHello=getip();  

  cout << (LPCTSTR)strHello << endl;  

  return nRetCode;  

}
```
