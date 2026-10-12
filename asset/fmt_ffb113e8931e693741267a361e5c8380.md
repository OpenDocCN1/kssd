
# 用 LuaDec 逆回 LUA 脚本内容

> 原文：[`www.pediy.com/kssd/pediy10/90963.html`](https://www.pediy.com/kssd/pediy10/90963.html)

在收集 LUA 资料进行学习时，其中有个叫 `LuaDec` 的，能把 LUA 编译出来的代码解回去(逆回脚本内容)，官方网站是：

`http://luadec.luaforge.net/`

它最后支持 LUA5.0 版本的代码,不过 LUA 也是开源的，个人可修改源码，这很难应对所有版本。

`LuaDec` 的示例，只是用 `luaL_loadfile` 把脚本文件载入，然后就进行解码了，没有在 LUA 中运作时，LUA 执行脚本的哪部分内容。没有这样的例子。

本人不会写文章，直接贴源码及分析时的经验。![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

主要 HOOK 上 `luaV_execute` 以及 `luaV_execute` 函数里边的 `luaD_poscall` 地方

如果是 HOOK 别人的程序,需要注意对 LUA 源码进行修改，达到和别人的 LUA 进行同步。

我接触过 2~3 个程序，目前发现其比较相同处

1，枚举型 OpCode 的指令顺序要对应

2，`lua_TObject` 的结构大小，以及其成员 `value`，`tt` 的上下位置

3，LUA5.1 版本时，`Proto` 结构的变化

4，LUA5.1 版本是，`LClosure` 的结构变化

![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")LUA 源码好多宏，好多 union 类型，看着头疼，最后是用 OD 跟踪看内存，再拿源码对比得出些经验的

![](img/67ce31534a4c20e91a6b42102a214c5e.jpg)

代码:

```cpp
// TestLuaDec.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include "windows.h"
#include "stdlib.h"


DWORD dwSubLuaRet = 0;
DWORD dwLuaRet = 0;
BOOL bMainLuaStart = FALSE;
lua_State* luaHookHandle = NULL;
DWORD fnluaV_execute = NULL;
DWORD fnluaD_precall = NULL;
void hookluaEx()
{
  __try{
    char* code = lua_Dec(luaHookHandle, -1);
    if (code)
    {
      printf("主函数开始\\n");
      printf(code);
      printf("主函数结束\\n\\n");
      free(code);
    }
    
  }__except (EXCEPTION_EXECUTE_HANDLER) {
    printf("lua fail");
    return;
  }
}
__declspec(naked) void hooklua(DWORD lua)
{
  __asm
  {
    push ebp
    mov ebp, esp
    push eax
    mov eax, lua
    mov luaHookHandle, eax
    pop eax
    push lua
    call fnluaV_execute
    mov dwLuaRet, eax
    call hookluaEx
    add esp, 4
    mov eax,dwLuaRet
    pop ebp
    retn
  }
}
void hookSubluaEx()
{
  __try{
    char* code = lua_DecSub(luaHookHandle, -1);
    if (code)
    {
      printf("子函数开始\\n");
      printf(code);
      printf("子函数结束\\n\\n");
      free(code);
    }
    
  }__except (EXCEPTION_EXECUTE_HANDLER) {
    printf("lua fail");
    return;
  }
}
__declspec(naked) void hookSublua(DWORD lua, DWORD lua2)
{
  __asm
  {
    push ebp
    mov ebp, esp
    push lua2
    push lua
    call fnluaD_precall
    mov dwSubLuaRet, eax
    call hookSubluaEx
    add esp, 8
    mov eax,dwSubLuaRet
    pop ebp
    retn
  }
}

void HookLua()
{
  HMODULE ImageBase = GetModuleHandle(NULL);
  
  PIMAGE_DOS_HEADER dos_stub = (PIMAGE_DOS_HEADER)ImageBase;
  PIMAGE_NT_HEADERS PEHead = (PIMAGE_NT_HEADERS)((DWORD)ImageBase + dos_stub->e_lfanew);

  DWORD f_luaV_execute = 0x0000E13D;
  BOOL  b_luaV_execute = FALSE;
  DWORD f_luaD_precall = 0xFED8BD83;
  BOOL  b_luaD_precall = FALSE;
  
  DWORD i = 0;
  DWORD OldProtect, tOld;
  for (int NOS=0; NOS<PEHead->FileHeader.NumberOfSections; NOS++)
  {
    PIMAGE_SECTION_HEADER pSecHeader = (PIMAGE_SECTION_HEADER)((DWORD)PEHead + NOS * sizeof(IMAGE_SECTION_HEADER)+sizeof(IMAGE_NT_HEADERS));
    if (IMAGE_SCN_CNT_CODE & pSecHeader->Characteristics)
    {
      char* pFile = (char*)((DWORD)ImageBase+pSecHeader->VirtualAddress);
      for (i=0; i<pSecHeader->SizeOfRawData-0x10; i++)
      {
        if (b_luaD_precall && b_luaD_precall)
          break;
        if (b_luaV_execute == FALSE && !memcmp(&pFile[i], &f_luaV_execute, 4))
        {
          if (*(PBYTE)(pFile+i+0x32) == 0xE8)
          {
            b_luaV_execute = TRUE;
            DWORD t = (DWORD)ImageBase + i + pSecHeader->VirtualAddress + 0x32;
            VirtualProtect((void*)t, 5, PAGE_EXECUTE_READWRITE, &OldProtect);
            fnluaV_execute = t + 5 +  *(PDWORD)(t+1);
            (*(PDWORD)(t+1)) = (DWORD)hooklua - t - 5;
            VirtualProtect((void*)i, 5, OldProtect, &tOld);
          }
        }
        if (b_luaD_precall == FALSE && !memcmp(&pFile[i], &f_luaD_precall, 4))
        {
          if (*(PBYTE)(pFile+i-0x0E) == 0xE8)
          {
            b_luaD_precall = TRUE;
            DWORD t = (DWORD)ImageBase + i + pSecHeader->VirtualAddress - 0x0E;
            VirtualProtect((void*)t, 5, PAGE_EXECUTE_READWRITE, &OldProtect);
            fnluaD_precall = t + 5 + *(PDWORD)(t+1);
            (*(PDWORD)(t+1)) = (DWORD)hookSublua - t - 5;
            VirtualProtect((void*)i, 5, OldProtect, &tOld);
          }
        }
      }
    }
  }
}
int main(int argc, char* argv[])
{
#ifdef _DEBUG //代码特征在 DEBUG 版本有效
  HookLua();
#endif

  lua_State* L;
  L=lua_open();
  luaopen_base(L);
  luaopen_io(L);
  luaopen_math(L);
  luaopen_string(L);
  luaopen_table(L); 
  
  lua_dofile(L, "test.lua");
  system("pause");
  return 0;
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `TestLuaDec.rar` |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `LuaBuild5.rar` |
