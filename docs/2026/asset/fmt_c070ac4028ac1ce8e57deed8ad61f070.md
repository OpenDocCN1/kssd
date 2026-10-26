
```cpp
// dllmain.cpp : 定义 DLL 应用程序的入口点。

//请注意：我删除了sub_100010B0函数实现代码(通过特征码得到函数地址，类似于OD搜索二进制功能)，感兴趣的可以自己去实现。

#include "stdafx.h"

#include "include/detours.h"

#include "include/detver.h"

#pragma comment(lib,"lib.X86\\detours.lib")

char *szMatchCode[] = {"E8????????B0018B4C242864890D00000000595E5B8BE55DC21400|1|2",

           "6A006A016A03E8????????83C4108D4317|7|1"};//只有两段特征码和修改皮肤有关，so我只保留了2段。我自己的函数通配符是AA

void CDECL  DebugPrintf(TCHAR * szFormat, ...){



  TCHAR   szBuffer [1024] ;

  va_list pArgList ;



  // The va_start macro (defined in STDARG.H) is usually equivalent to:

  // pArgList = (char *) &szFormat + sizeof (szFormat) ;



  va_start (pArgList, szFormat) ;



  // The last argument to wvsprintf points to the arguments



  vsnprintf (szBuffer, sizeof (szBuffer) / sizeof (TCHAR), 

    szFormat, pArgList) ;



  // The va_end macro just zeroes out pArgList for no good reason



  va_end (pArgList) ;



  OutputDebugString(szBuffer);

}

static HANDLE  (WINAPI* OLD_CreateMutexA)(

                    __in          LPSECURITY_ATTRIBUTES lpMutexAttributes,

                    __in          BOOL bInitialOwner,

                    __in          LPCTSTR lpName)=CreateMutexA;  

DWORD dw_Pid = 0;



ULONG InterestAddr[2] = {0};

BOOL MatchGetAddr()

{

  DWORD dwOffect = 0;

  int dType = 0;

  char szMatchItem[MAX_PATH];

  ULONG uTmpAddr;

  int  index= 0;

  for(int i = 0;i<2;i++)

  {

    index =0;

    sscanf(szMatchCode[i], "%[^|]|%x|%d", szMatchItem, &dwOffect, &dType);



    uTmpAddr = sub_100010B0(0,(char*)0x00401000,0x2000000,szMatchItem);



    DebugPrintf("Match Code %s dwOffect %x dType %d Result = %08x",szMatchCode[i],dwOffect,dType,uTmpAddr);



    if(uTmpAddr == 0) continue;

    if (dType == 1)

    {

      InterestAddr[i] = uTmpAddr + dwOffect;

    }

    if (dType == 2)

    {

      InterestAddr[i] = (uTmpAddr + dwOffect + *(DWORD *)(uTmpAddr + dwOffect) + 4);

    }

  }

  DebugPrintf("InterestAddr = %08x %08x",InterestAddr[0],InterestAddr[1]);

  return 1;

};



DWORD SAVEOLD1;

WORD SAVEOLD2;

ULONG uRetAddr;

DWORD MySeat;

int SkinReplace(int par1,int par2,void* lp)

{

  if ( lp && !IsBadReadPtr(lp, 1u) )

  {

    if ( *(BYTE *)lp )

    {

      if ( *(BYTE *)lp == 0x65 )

      {

        if ( par1 >= 0x12 )

        {

          DebugPrintf("ReplaceSkin bType = 0, dwId = %x\n", *(DWORD *)((char *)lp + 1));

          if ( dw_Pid == *(DWORD *)((char *)lp + 1) )

          {

            if ( lp != (void *)-17 )

            {

              DebugPrintf("My::ReplaceSkin bType = 0x65, dwFilter = 0x%X, SkinName = %s\n", par1, (char *)lp + 17);

              if ( !*(DWORD *)((char *)lp + 9) )

              {

                *(DWORD*)((char *)lp + 9) = 1;//全部换为第一个皮肤

                return 1;

              }

            }

          }

        }

      }

    }

    else                                        // 这都无从证实。全部和他写的一样就行了。

    {

      if ( par1 >= 0xC )

      {

        if ( dw_Pid == *((DWORD *)lp + 2) )

          MySeat = *((DWORD *)lp + 1);

        DebugPrintf(

          "My::ReplaceSkin bType = 0, dwFilter = 0x%X, g_dwCamp = 0x%X, g_dwCamp+4 = 0x%X\n",

          par1,

          MySeat,

          *((DWORD *)lp + 2));

      }

    }

  }

  return 1;



//#第一个HOOK处理完毕

}

//照抄。。。

int _stdcall HOOK00_1(int par1, void *lp, int flag, int par2)

{

  signed int result; // eax@3



  if ( flag == 3 && par1 )                    

    result = SkinReplace(par1, par2, lp);     

  else

    result = 1;

  return result;



}

DWORD OriCallMachineCode = 0;

DWORD OriCallAddr = 0;



void _stdcall HOOKHERO(DWORD _ebx)

{

  DWORD mayheroname; // esi@1

  int GameSeat; // edi@2



  mayheroname = _ebx + 0x97;

  if ( _ebx != 0xFFFFFF69 )

  {

    GameSeat = *(DWORD *)(_ebx + 9);

    DebugPrintf("HOOKHERO dwCamp=%x,g_dwCamp=%x", *(DWORD *)(_ebx + 9), MySeat);

    if ( GameSeat == MySeat )

    {

      DebugPrintf("HOOKHERO ReplaceSkin bType = 0x48, Camp = 0x%X, pszSkinName = %p, SkinName = %s\n",

        GameSeat,

        mayheroname,

        mayheroname);

      if ( !*(DWORD *)(_ebx + 0x13))

      {

        *(DWORD*)(_ebx + 0x13) = 1;//换成第一个皮肤

        DWORD flOldProtect;

        if ( VirtualProtect((PVOID)InterestAddr[1], 4, PAGE_EXECUTE_READWRITE, &flOldProtect) )

        {                                       // 这里只hook一次。。。应该是初始化游戏时的一次。jmphook 就是游戏每次显示的皮肤。也许

          *(DWORD *)InterestAddr[1] = OriCallMachineCode;

          VirtualProtect((PVOID)InterestAddr[1], 4, flOldProtect, &flOldProtect);

          DebugPrintf("Unhook success!");

        }

        else

        {

          DebugPrintf("Unhook failed!");

        }

      }



    }

  }

}

_declspec(naked) void HookCall11()

{

  __asm 

  {

    pushad

      pushfd

      push    ebx

      call    HOOKHERO

      popfd

      popad

      push    OriCallAddr

      retn



  }

}

_declspec(naked) void HOOK00()

{

  __asm {

  pushad

  pushfd

  mov     eax, [ebp+10h]

  push    eax            

  mov     eax, [ebp+0Ch]

  push    eax             

  mov     eax, [ebp+18h]

  push    eax             

  mov     eax, [ebp+14h]

  push    eax             

  call    HOOK00_1

  test    eax, eax

  jz      RET1

  popfd

  popad

  MOV     EAX, DWORD PTR FS:[0] //原指令

  jmp     uRetAddr

RET1:

  popfd

  popad

  mov     esp, ebp

  pop     ebp

  retn    14h

  }

}

void Hook()

{

  ULONG uHookAddr = InterestAddr[0];

  HANDLE handle=GetCurrentProcess();

  char MyJMP[5]={0};

  MyJMP[0]=(char)0xe9;

  ULONG uTempAddr=(ULONG)MyJMP;

  uRetAddr = uHookAddr + 6; 

  ULONG uSkillJmp=(ULONG)HOOK00-uHookAddr-5;

  __asm

  {

    mov eax,uSkillJmp

      mov ebx, uTempAddr

      add ebx ,1

      mov [ebx],eax

      mov ecx,[ebx]

  }

  WriteProcessMemory(handle,(LPVOID)(uHookAddr),(LPVOID)MyJMP,5,NULL);

}

HANDLE  WINAPI NEW_CreateMutexA(

                __in          LPSECURITY_ATTRIBUTES lpMutexAttributes,

                __in          BOOL bInitialOwner,

                __in          LPCTSTR lpName) 

{  



  //修改输入参数，调用原函数  

  DebugPrintf("Enter MutexHook");

  HANDLE ret=OLD_CreateMutexA(lpMutexAttributes,

    bInitialOwner,

    lpName);  

  if(!InterestAddr[0])//Mark

  {

    MatchGetAddr();//得到特征码对应的地址

    if(InterestAddr[0]){

      InterestAddr[0] = InterestAddr[0]+13;//jmp HOOK的地址

      SAVEOLD1 = *(DWORD*)InterestAddr[0];

      SAVEOLD2 = *(WORD*)(InterestAddr[0]+4);//这里保存之前的6个字节。不保存也行。。。反正又不用恢复HOOK

      Hook();//HOOK InterestAddr[0]这个地址



      OriCallMachineCode = *(DWORD *)InterestAddr[1];

      OriCallAddr = (DWORD)((char *)InterestAddr[1] + *(DWORD *)InterestAddr[1] + 4);

      DWORD flOldProtect;

      if ( VirtualProtect((PVOID)InterestAddr[1], 4, PAGE_EXECUTE_READWRITE, &flOldProtect) )

      {

        *(DWORD *)InterestAddr[1] = (DWORD)((char *)HookCall11- InterestAddr[1] - 4);

        VirtualProtect((PVOID)InterestAddr[1], 4, flOldProtect, &flOldProtect);// 修改call

        DebugPrintf("Hook success!");

      }

      else

      {

        DebugPrintf("Hook failed!");

      }

      //Detours恢复HOOK 

      DetourTransactionBegin();

      DetourUpdateThread(GetCurrentThread());



      DetourDetach(&(PVOID&)OLD_CreateMutexA,NEW_CreateMutexA);



      DetourTransactionCommit();

    }

  }



  return ret;  

}  

BOOL HookMutex()

{

  DetourRestoreAfterWith();  

  DetourTransactionBegin();  

  DetourUpdateThread(GetCurrentThread());  



  DetourAttach(&(PVOID&)OLD_CreateMutexA,NEW_CreateMutexA);  



  DetourTransactionCommit();  



  return TRUE;

}

BOOL APIENTRY DllMain( HMODULE hModule,

                       DWORD  ul_reason_for_call,

                       LPVOID lpReserved

           )

{

  char * szCommand;

  switch (ul_reason_for_call)

  {

  case DLL_PROCESS_ATTACH:

    DisableThreadLibraryCalls(hModule);

    DebugPrintf("Dll 已加载");

    szCommand = GetCommandLine();

    for (int i = strlen(szCommand)-1;i>0;i--)

    {

      if ( *(szCommand+i) == '=' &&  

         *(szCommand+i-1) == '=')

      {

        DebugPrintf("Find == %s",szCommand+i+1);

        dw_Pid = strtoul(szCommand+i+1,0,0);

        DebugPrintf("dw_Pid = %x",dw_Pid);

        break;

      }

    }

    HookMutex();

    break;



  case DLL_THREAD_ATTACH:

  case DLL_THREAD_DETACH:

  case DLL_PROCESS_DETACH:

    break;

  }

  return TRUE;

}
```
