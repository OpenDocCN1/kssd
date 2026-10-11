
# 处女贴 ring3 改 `KernelCallbackTable` 防键盘钩子

> 原文：[`www.pediy.com/kssd/pediy11/105107.html`](https://www.pediy.com/kssd/pediy11/105107.html)

```
__declspec(naked) test()  
{  
  _asm  
  {  
    ret  
    jmp oldaddr  
  }  
}  

DWORD * KernelCallbackTable = NULL;  

_asm  
{  
     push eax  
     mov eax,dword ptr fs:[0x18]  
     mov eax,dword ptr ds:[eax+0x30]  
     mov eax,dword ptr ds:[eax+0x2C]  
     mov KernelCallbackTable, eax  
     pop eax  
}  

DWORD old = 0;  

if(VirtualProtect(&KernelCallbackTable[40], sizeof(PVOID),PAGE_EXECUTE_READWRITE, &old))  
{  
  oldaddr = KernelCallbackTable[40];  
  KernelCallbackTable[40] = (DWORD)test;  
}  
```

前几天研究 `windows` 的消息机制时偶尔发现。

通过修改 `KernelCallbackTable` 内供 `ring3` 回调用 `ring0` 的函数分派表实现
