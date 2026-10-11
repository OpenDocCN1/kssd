
# IE 锁主页，小玩具

> 原文：[`www.pediy.com/kssd/pediy12/141293.html`](https://www.pediy.com/kssd/pediy12/141293.html)

仅限微软的 IE。每次点击主页都有效  

注：需注入 IE 进程  

bin:`http://d.119g.com/f/FFCC7CC2069BE52D.html`  

```
BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul_reason_for_call, 
                       LPVOID lpReserved 
                     ) 
{ 
        g_module    =    hModule; 
        if (DLL_PROCESS_ATTACH    ==    ul_reason_for_call) 
        { 
            char szbuffer[1024]={0}; 
            GetModuleFileName(NULL, szbuffer, 1021); 
            OutputDebugString(szbuffer); 
            if (strstr(szbuffer, "explore")) 
            { 
                g_module    =    hModule; 
                if (GetModuleHandle("ieframe.dll")) 
                { 
                    HEInitHook(&g_HookGetStdLocation, "ieframe.dll", (char*)150, myGetStdLocation); 
                    HEStartHook(&g_HookGetStdLocation); 
                } 
                else 
                { 
                    HEInitHook(&g_HookGetStdLocation, "shdocvw.dll", (char*)150, myGetStdLocation); 
                    HEStartHook(&g_HookGetStdLocation); 
                } 
            } 
        } 
        if (DLL_PROCESS_DETACH==ul_reason_for_call) 
        { 
            HEStopHook(&g_HookGetStdLocation); 
        } 
    return TRUE; 
} 

======================================================================== 
int WINAPI myGetStdLocation( WCHAR *pUrl, int k,int j 
                            ) 
{ 
    PGetStdLocation    pJmp    =    (PGetStdLocation)g_HookGetStdLocation.Stub; 
    int ret = pJmp(pUrl, k, j); 
    WCHAR szbuffer[1024]; 
    wsprintfW(szbuffer, L"strlen %d, len1 %d, len2 %d, URL:%s\r\n", wcslen(pUrl), k,j, pUrl); 
    MessageBoxW(NULL,szbuffer,L"test",NULL); 
    wcscpy(pUrl, L"http://www.baidu.com"); 
    wsprintfW(szbuffer, L"strlen %d, len1 %d, len2 %d, URL:%s\r\n", wcslen(pUrl), k,j, pUrl); 
    MessageBoxW(NULL,szbuffer,L"test",NULL); 
    return ret; 
}
```
