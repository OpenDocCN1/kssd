
# anti vmware

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-120.htm`](https://www.pediy.com/kssd/pediy09/pediy09-120.htm)

```c
#include <windows.h>  
#pragma comment(linker, "/entry:main")  

int main()  
{  
  __asm  
  {  
    mov    dx, 1  
  l1:  
    verr  dx        //其中的 dx, 可以为其他。只要 dx & 4 == TRUE, 那么就会在 vmware 中发生异常，而外部没有发现这种现象  
    shl    dx, 1  
    jmp    l1  
  }  
  return 0;  
} 

bool VMWareTest()  
{  
  BYTE PortValue1,PortValue2;  
  __try  
  {  
    __asm  
    {  
      pushad   
      pushfd  
      xor ebx,ebx  
      mov ecx,0xa   
      mov eax, 'VMXh'      ; EAX=magic  
      mov dx, 'VX'      ; DX=magic  
      in  eax, dx        ; specially processed io cmd  
      cmp ebx, 'VMXh'      ; also eax/ecx modified (maybe vmw/os ver?)  
      jne local_001  
      mov gInVMWARE,1  
  local_001:  
      popfd  
      popad  
    }  
  }  
  __except(EXCEPTION_EXECUTE_HANDLER)  
  {  
    gInVMWARE=false;  
  }  
  return gInVMWARE;  
}  

bool VirtualPCTest()  
{  
  __try  
  {  
    __asm  
    {  
      pushad       
      mov  ebx, 0 // Flag  
      mov  eax, 1 // VPC function number  
      __emit 0Fh  
      __emit 3Fh  
      __emit 07h  
      __emit 0Bh  
      test ebx, ebx  
      jnz local_001  
      mov gInVirtualPC,1  
  local_001:  
      popad  
    }  
  }  
  __except(EXCEPTION_EXECUTE_HANDLER)  
  {  
    gInVirtualPC=false;  
  }  
  if(gInVirtualPC)  
    DbgPrint("Syser : Host machine is VirtualPC !\n");  
  return gInVirtualPC;  
}
```

> 楼主提的指令是 `vmware` 虚拟 `cpu` 不支持的,所以会异常退出.
> 现在 `007` 里都包含此指令,其实可以直接硬断 `nop` 掉,就可以继续在 `vmware` 里跑了.

`4dd466d0e1e0cc123afbe504da2462ca`
