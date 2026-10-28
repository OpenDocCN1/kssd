
```asm
                call dword ptr [RegCloseKey]    //此处不能使用call RegCloseKey，只能进行间接寻址调用

A10002300:
                mov     eax, edi
                pop     edi
                pop     esi
                add     esp, 0x10
                retn    0x0c

        }
```

全部源代码如下：

```cpp
// PCInfoAPI.cpp : Defines the entry point for the DLL application.
//
#include "stdafx.h"

#define PCInfoAPI_EXPORTS

#ifdef PCInfoAPI_EXPORTS

#define PCInfoAPI_API extern "C" __declspec(dllexport)

#else

#define PCInfoAPI_API __declspec(dllimport)

#endif

BOOL APIENTRY DllMain( HANDLE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                                         )
{
    return TRUE;
}

char *key1="Software\\abc\\def";
char *key2="InstallCode";

PCInfoAPI_API  int __stdcall RegistryCode(int a1,int a2,const char * key)
{
        //还原堆栈,VC为函数默认生成了保护寄存器的代码
        #ifdef _DEBUG //DEBUG模式和RELEASE模式生成的保护寄存器的代码不一样
                __asm
                {
                        pop edi
                        pop esi
                        pop ebx
                        pop ebp
                }
        #else
                __asm
                {
                        pop edi
                        pop esi
                        pop ebp
                }
        #endif
        __asm
        {
                sub     esp, 0x10
                lea     eax, dword ptr [esp]
                push    esi
                push    edi
                xor     edi, edi
                push    edi
                push    eax
                push    edi
                push    0x0F003F
                push    edi
                push    edi
                push    edi
                push    key1
                push    0x80000002
                call dword ptr [RegCreateKeyExA]
                test    eax, eax
                jnz     short A10002300
                mov     esi, dword ptr [esp+0x24]
                //mov     esi,  key
                mov     eax, key2
                mov     ecx, [eax]
                mov     edx, [eax+4]
                mov     eax, [eax+8]
                test    esi, esi
                mov     dword ptr [esp+0x0C], ecx
                mov     dword ptr [esp+0x10], edx
                mov     dword ptr [esp+0x14], eax
                je      short A10002300
                push    esi
                call dword ptr [lstrlenA]
                inc     eax
                lea     ecx, dword ptr [esp+0x0C]
                mov     edx, dword ptr [esp+0x08]
                push    eax
                push    esi
                push    1
                push    edi
                push    ecx
                push    edx
                call dword ptr [RegSetValueExA]
                test    eax, eax
                jnz     short A100022F5
                mov     edi, 1
A100022F5:
                mov     eax, dword ptr [esp+0x08]
                push    eax
                call dword ptr [RegCloseKey]
A10002300:
                mov     eax, edi
                pop     edi
                pop     esi
                add     esp, 0x10
                retn    0x0c
        }
}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（28.69kb，1441次下载）
- [2.jpg](javascript:void(0);)（56.30kb，1419次下载）
- [3.jpg](javascript:void(0);)（123.15kb，1417次下载）
