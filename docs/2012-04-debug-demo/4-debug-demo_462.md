
# 迅游无限激活破解

> 原文：[`www.pediy.com/kssd/pediy11/125240.html`](https://www.pediy.com/kssd/pediy11/125240.html)

本文以技术交流为目的，下载本代码后请在 24 小时内删除，不得用于任何商业用途。

迅游是目前市场上最好的网络加速工具之一，对于每一台机器，可以免费激活一个有 7 天免费试用时间的账号，一台机器只能激活一个免费账号，所以有很多人经常跑去网吧激活帐号。。。

本文代码可以破除一台机器只能激活一个账号的限制。调试过程很简单，这里只发代码。

代码:

```c
xTiNt// XunYou.cpp : Defines the entry point for the DLL application.
//
//By 打泡泡(DPP)
#include "stdafx.h"
#include <stdio.h>
#include <stdlib.h>

char g_newSerial[256] = {0};

char * getRandomSerial()
{
  srand(GetTickCount());
  memset(g_newSerial, 0, 256);
  DWORD dwTickCount = GetTickCount()*rand();
  for (int i=0; i<2; i++)
  {
    sprintf(g_newSerial+i*8, "%08X", dwTickCount);
  }
  OutputDebugString(g_newSerial);
  return g_newSerial;
}

DWORD dwOldFunc = 0;

VOID __stdcall ModifySerial2(PDWORD ppp)
{
  __try
  {
    DWORD ppp2 = *ppp+4;
    char * pSerial = (char *)*(PDWORD)ppp2;
    OutputDebugString(pSerial);
    memcpy(pSerial, getRandomSerial(), 16);
    OutputDebugString(pSerial);
  }
  __except(1)
  {
    OutputDebugString("except");
  }
}

__declspec(naked) VOID ModifySerial()
{
  __asm
  {
    MOV  EAX,  ESP
    PUSHAD
    ADD   EAX,  4
    PUSH  EAX
    CALL  ModifySerial2
    POPAD
    JMP   DWORD PTR[dwOldFunc]
  }
}

DWORD WINAPI thread_Hook(LPVOID lp)
{
  //0453354C  8354C
  DWORD dwJsBase = (DWORD)GetModuleHandle("js.dll");
  while (!dwJsBase)
  {
    dwJsBase = (DWORD)GetModuleHandle("js.dll");
    Sleep(520);
  }
  DWORD dwHookAddr = dwJsBase + 0x8354C;  //Hook 这里!!
  dwOldFunc = dwJsBase + 0x1CD0;
  while (*(PDWORD)dwHookAddr != 0xF7E77FE8)
  {
    Sleep(520);
  }
  BYTE callBytes[5] = {0xE8, 0x00, 0x00, 0x00, 0x00};
  DWORD tmp = (DWORD)ModifySerial - dwHookAddr - 5;
  memcpy(callBytes+1, &tmp, 4);
  if (!WriteProcessMemory((HANDLE)-1, (PVOID)dwHookAddr, callBytes, 5, &tmp))
  {
    OutputDebugString("[fuckXY]: hook error");
  }
  else
  {
    MessageBoxA(GetForegroundWindow(), "破解成功，本机可以无限激活新账号！", "by FuckXunYou Studio QQ1544205537", 0);
  }
  return 0;
}

BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul_reason_for_call, 
                       LPVOID lpReserved
        )
{
  if (ul_reason_for_call == DLL_PROCESS_ATTACH)
  {
    CreateThread(0, 0, thread_Hook, 0, 0, 0);     
  }
    return TRUE;
}

本代码由 xTiNt 自动着色 http://kbadboy.yeah.net
```
