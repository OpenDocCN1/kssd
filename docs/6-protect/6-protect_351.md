

# 小小的 anti

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-632.htm`](https://www.pediy.com/kssd/pediy08/pediy8-632.htm)

附件下载：anti1.rar

#include <windows.h>  
  
#pragma comment(linker, "/subsystem:windows")  
#pragma comment(linker, "/entry:mydllmain")  
#pragma comment(linker, "/filealign:0x200")  
  
  
void anti()  
{  
  \_\_asm  
  {  
    push 0  
    push 0  
    push 11h  
    push -2  
    mov eax, 0C7h  
    mov edx, esp  
    int 2Eh  
  }  
  
  \_\_try  
  {  
    \_\_asm  
    {  
      lea eax, back  
      push 0  
      push eax  
      mov eax, 0E5h  
      mov edx, esp  
      \_\_emit 0x0F  
      \_\_emit 0x34  
    }  
  }  
  
  \_\_except (1)  
  {  
    \_\_asm mov edi, edi  
  }  
  
back:  
  \_\_asm add esp, 14h  
  return ;  
}  
  
  
DWORD \_\_declspec(naked) Is98()  
{  
  \_\_asm  
  {  
    xor eax, eax  
    mov ax, cs  
    shr eax, 8  
    retn   
  }  
}  
  
BOOL APIENTRY mydllmain( HANDLE hModule,   
                       DWORD  ul\_reason\_for\_call,   
                       LPVOID lpReserved  
           )  
{  
    switch (ul\_reason\_for\_call)  
  {  
    case DLL\_PROCESS\_ATTACH:  
      \_\_asm push esi  
      if (Is98() == 0)  
        anti();  
      \_\_asm pop esi  
      break;  
    }  
    return TRUE;  
} 

 4dd466d0e1e0cc123afbe504da2462ca