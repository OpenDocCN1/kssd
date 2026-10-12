
# 自认为巧妙的 ring3 HOOK 模板，欢迎大牛指点

> 原文：[`www.pediy.com/kssd/pediy12/132633.html`](https://www.pediy.com/kssd/pediy12/132633.html)

```c
typedef struct _PUSH_CONTEXT { //represent pushad and pushfd on the stack  
  DWORD dwEflags;  
  DWORD dwEDI;  
  DWORD dwESI;  
  DWORD dwEBP;  
  DWORD dwESP;  
  DWORD dwEBX;  
  DWORD dwEDX;  
  DWORD dwECX;  
  DWORD dwEAX;  
} PUSH_CONTEXT, *PPUSH_CONTEXT;  

void __cdecl get_data(LPVOID lpParam, const PUSH_CONTEXT context) //context 参数必须定义成传值，刚好和 pushad,pushfd 相对应  

{  
  //hook 后取数据的函数可以用 C 写了，呵呵  
  return ;  
}  

/*  
 * 挂钩任意内存地址,并执行指定的函数,函数参数为 lpParam  
 * 函数原型为 VOID __cdecl get_data(LPVOID lpParam, const PUSH_CONTEXT context);  
 * 要 hook 的地方不能有相对转移指令,如 call,jmp 等  
 * hook 后执行 lpHookProc 时堆栈增加 0x2C 字节  
 * LPVOID lpHookAddress 要挂钩的地址  
 * DWORD dwHookLen 挂钩时要改写的长度  
 * LPVOID lpHookProc 挂钩后要执行的函数的地址  
 * LPVOID lpParam 挂钩后要执行的函数的参数,  
 * 次参数和 CreateThread 函数中线程函数的参数差不多  
 */  

BOOL CreateHook(LPVOID lpHookAddress, DWORD dwHookLen, LPVOID lpHookProc,LPVOID lpParam)  

{  
  DWORD dwOldProtect;  
  LPVOID lpHookThunk;  
  DWORD dwHookThunkStart;  
  DWORD dwParamStart;  
  DWORD dwHookThunkLen;  
  UCHAR jmpCode[5] = {0x90,0x90,0x90,0x90,0x90};  
  __asm  
  {  
    lea eax,ASM_HOOKTHUNK_START  
    mov dword ptr[dwHookThunkStart],eax  
    lea edx,ASM_HOOKTHUNK_END  
    sub edx,eax  
    mov dword ptr[dwHookThunkLen],edx  
    lea edx,param  
    sub edx,eax  
    mov dword ptr[dwParamStart],edx  
    jmp HookStart  
  ASM_HOOKTHUNK_START:  
    pushad  
    pushfd  
    call L1  
  param:  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
    _emit 0x00  
  L1:  
    pop eax  
    push dword ptr[eax]  
    call dword ptr[eax+0x04]  
    add esp,0x04  
    popfd  
    popad  
  ASM_HOOKTHUNK_END:  
  }  

HookStart:  
  if(!VirtualProtect(lpHookAddress, dwHookLen, PAGE_EXECUTE_READWRITE,&dwOldProtect))  
  {  
#ifdef _DEBUG  
    OutputDebugStringEx("CreateHook() can't change protect at address:0x%08X",lpHookAddress);  
#endif  
    return FALSE;  
  }  
  lpHookThunk = VirtualAlloc(NULL,dwHookThunkLen  
          + dwHookLen  
          + 5,  
          MEM_COMMIT,  
          PAGE_EXECUTE_READWRITE);  
  if(lpHookThunk == NULL)  
  {  
    #ifdef _DEBUG  
      OutputDebugString("VirtualAlloc() failed in CreateHook()");  
    #endif  
    return FALSE;  
  }  
  memcpy(lpHookThunk,(void *)dwHookThunkStart,dwHookThunkLen); //copy hook thunk code  
  *(LPVOID*)((DWORD)lpHookThunk + dwParamStart) = lpParam;  
  *(LPVOID*)((DWORD)lpHookThunk + dwParamStart + 0x04) = lpHookProc;  
  memcpy((LPVOID)((DWORD)lpHookThunk+dwHookThunkLen),lpHookAddress,dwHookLen); //copy old address's code  
  jmpCode[0] = 0xE9;  
  *(DWORD*)(&jmpCode[1]) = (DWORD)lpHookAddress + dwHookLen  
    -  ((DWORD)lpHookThunk+dwHookThunkLen+dwHookLen+5);  
  memcpy((LPVOID)((DWORD)lpHookThunk+dwHookThunkLen+dwHookLen),jmpCode,5); // make the code jump to old address  
  jmpCode[0] = 0xE9;  
  *(DWORD*)(&jmpCode[1]) =  
  (DWORD)lpHookThunk - (DWORD)lpHookAddress - 5;  
  __asm  
  {  
    mov ecx,0x05  
    lea esi,jmpCode  
    mov edi,dword ptr[lpHookAddress]  
    rep movsb //create hook code  
  }  
  if(!VirtualProtect(lpHookAddress,dwHookLen,dwOldProtect,&dwOldProtect))  
  {  
#ifdef _DEBUG  
    OutputDebugStringEx("CreateHook() can't restore old protect at address:0x%08X",lpHookAddress);  
#endif  
  };  
  return TRUE;  

}  

typedef struct _MY_STRUCT  
{  
  char buf[256];//???  
} MY_STRUCT;  

void main()  
{  
  MY_STRUCT my_data;  
  CreateHook((LPVOID)target_addr, 0x05/*??*/, (LPVOID)get_data, &my_data);  

}
```
