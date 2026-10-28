
# [原创]IE锁主页，小玩具

仅限微软的IE。每次点击主页都有效

注：需注入IE进程

`bin:`be1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8Q4x3X3f1I4x3e0W2Y4i4K6u0W2j5$3!0E0i4K6u0r3k6W2)9J5c8V1k6r3b7@1x3%4b7@1x3J5x3o6j5&6b7V1f1#2x3V1c8Q4x3X3g2Z5N6r3#2D9

```c
BOOL APIENTRY DllMain( HANDLE hModule,

                       DWORD  ul\_reason\_for\_call,

                       LPVOID lpReserved

                     )

{

        g\_module    =    hModule;

        if (DLL\_PROCESS\_ATTACH    ==    ul\_reason\_for\_call)

        {

            char szbuffer[1024]={0};

            GetModuleFileName(NULL, szbuffer, 1021);

            OutputDebugString(szbuffer);

            if (strstr(szbuffer, "explore"))

            {

                g\_module    =    hModule;

                if (GetModuleHandle("ieframe.dll"))

                {

                    HEInitHook(&g\_HookGetStdLocation, "ieframe.dll", (char*)150, myGetStdLocation);

                    HEStartHook(&g\_HookGetStdLocation);

                }

                else

                {

                    HEInitHook(&g\_HookGetStdLocation, "shdocvw.dll", (char*)150, myGetStdLocation);

                    HEStartHook(&g\_HookGetStdLocation);

                }

            }

        }

        if (DLL\_PROCESS\_DETACH==ul\_reason\_for\_call)

        {

            HEStopHook(&g\_HookGetStdLocation);

        }

    return TRUE;

}
```

========================================================================

```c
int WINAPI myGetStdLocation( WCHAR *pUrl, int k,int j

                            )

{

    PGetStdLocation    pJmp    =    (PGetStdLocation)g\_HookGetStdLocation.Stub;

    int ret = pJmp(pUrl, k, j);

    WCHAR szbuffer[1024];

    wsprintfW(szbuffer, L"strlen %d, len1 %d, len2 %d, URL:%s\r\n", wcslen(pUrl), k,j, pUrl);

    MessageBoxW(NULL,szbuffer,L"test",NULL);

    wcscpy(pUrl, L"9b0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0");

    wsprintfW(szbuffer, L"strlen %d, len1 %d, len2 %d, URL:%s\r\n", wcslen(pUrl), k,j, pUrl);

    MessageBoxW(NULL,szbuffer,L"test",NULL);

    return ret;

}
```

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
