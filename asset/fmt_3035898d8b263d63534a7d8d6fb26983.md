
# 小小的 `anti`

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-632.htm`](https://www.pediy.com/kssd/pediy08/pediy8-632.htm)

附件下载：`anti1.rar`

```c
#include <windows.h>

#pragma comment(linker, "/subsystem:windows")

#pragma comment(linker, "/entry:mydllmain")

#pragma comment(linker, "/filealign:0x200")

void anti()

{

  __asm

  {

    push 0

    push 0

    push 11h

    push -2

    mov eax, 0C7h

    mov edx, esp

    int 2Eh

  }

  __try

  {

    __asm

    {

      lea eax, back

      push 0

      push eax

      mov eax, 0E5h

      mov edx, esp

      __emit 0x0F

      __emit 0x34

    }

  }

  __except (1)

  {

    __asm mov edi, edi

  }

back:

  __asm add esp, 14h

  return ;

}

DWORD __declspec(naked) Is98()

{

  __asm

  {

    xor eax, eax

    mov ax, cs

    shr eax, 8

    retn

  }

}

BOOL APIENTRY mydllmain( HANDLE hModule,

                         DWORD  ul_reason_for_call,

                         LPVOID lpReserved

                     )

{

    switch (ul_reason_for_call)

  {

    case DLL_PROCESS_ATTACH:

      __asm push esi

      if (Is98() == 0)

        anti();

      __asm pop esi

      break;

    }

    return TRUE;

}
```

`4dd466d0e1e0cc123afbe504da2462ca`
