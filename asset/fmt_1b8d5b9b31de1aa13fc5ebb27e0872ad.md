
```c
///////////////// vm.h ////////////////

typedef struct _tagVMCONTEXT
{
  DWORD eax;
  DWORD ebx;
  DWORD ecx;
  DWORD edx;
  DWORD edi;
  DWORD esi;
  DWORD ebp;
  DWORD *esp;
  DWORD eflags;
  BYTE arg[5]; //操作数
  DWORD stack[4096];  //堆栈
} VMCONTEXT;

VMCONTEXT vmcontext;
typedef int (* VMHANDLER)(void);

__declspec(naked) int VMStart()
{
  __asm
  {
    push eax
    pop [vmcontext.eax]
    push ebx
    pop [vmcontext.ebx]
    push ecx
    pop [vmcontext.ecx]
    push edx
    pop [vmcontext.edx]
    push edi
    pop [vmcontext.edi]
    push esi
    pop [vmcontext.esi]
    push ebp
    pop [vmcontext.ebp]
    pushfd
    pop [vmcontext.eflags]
    push eax
    mov eax,offset vmcontext.stack
    add eax,0x1000*4-4
    push eax
    pop [vmcontext.esp]
    pop eax
    mov eax,1     //返回值为 1
    ret
  }
}

__declspec(naked) int VMExit()
{
  __asm
  {
    push [vmcontext.eax]
    pop eax
    push [vmcontext.ebx]
    pop ebx
    push [vmcontext.ecx]
    pop ecx
    push [vmcontext.edx]
    pop edx
    push [vmcontext.edi]
    pop edi
    push [vmcontext.esi]
    pop esi
    push [vmcontext.ebp]
    pop ebp
    push [vmcontext.eflags]
    popfd
    mov eax,1   //返回值为 1
    ret
  }
}


int VPushImm32()
{
  DWORD imm=*((DWORD *)vmcontext.arg);
  vmcontext.esp--;
  *vmcontext.esp=imm;
  return 5;  //返回值 = 1 + 用到的操作数个数，下同。
}

int VCallImm32()
{
  DWORD imm =*((DWORD *)vmcontext.arg);
  _asm
  {
    mov esi, esp
    mov esp, [vmcontext.esp]
    call imm
    push eax
    pop [vmcontext.eax]
    mov esp, esi
  }
  return 5;
}

int VMovReg32Mem32()
{
  BYTE index=vmcontext.arg[0];
  DWORD *addr =*(DWORD **>(&vmcontext.arg[1]);
  ((DWORD *)&vmcontext)[index]=*addr;
  return 6;
}

int VCmpReg32Imm32()
{
  BYTE index=vmcontext.arg[0];
  DWORD reg=((DWORD *)&vmcontext)[index];
  DWORD imm=*((DWORD*)(&vmcontext.arg[1]));
  if(reg==imm)
  {
    vmcontext.eflags = vmcontext.eflags | 0x40;  //ZF 标志置 1
  }
  else
  {
    vmcontext.eflags = vmcontext.eflags & 0xFFFFFFBF; //ZF 标志置 0
  }
  return 6;
}

int VJnz()
{
  int iRtn;
  if(vmcontext.eflags & 0x40)
  {
    iRtn = 5;
  }
  else
  {
    iRtn = *((int *)vmcontext.arg);
  }
  return iRtn;
}

int VJmp()
{
  int iRtn=*((int *)vmcontext.arg);
  return iRtn;
}

int VNop()
{
  return 1;
}

int VPop()
{
  vmcontext.esp++;
  return 1;
}

VMHANDLER VMHandlerTable[]= //删减了许多 Handler，这里相当的不完整
{
  (VMHANDLER)NULL, //00,无对应 Handler
  (VMHANDLER)VMStart, //01
  (VMHANDLER)VMExit,  //02

  (VMHANDLER)VNop, //03
  (VMHANDLER)VMovReg32Mem32, //04
  (VMHANDLER)VCmpReg32Imm32, //05

  (VMHANDLER)VPushImm32, //06
  (VMHANDLER)VCallImm32, //07
  (VMHANDLER)VPop, //08

  (VMHANDLER)VJnz,//09
  (VMHANDLER)VJmp,  //0A
};

///////////////// vm.cpp ////////////////

#define _WIN32_WINNT 0x0500
#include "windows.h"
#include "iostream.h"
#include "vm.h"

int DebugMain(int argc, char* argv[]);

DWORD key;
char w[]="wrong";
char r[]="right";

void ShowMessage(char *s)
{
  cout<<s;
}


int main(int argc, char* argv[])
{
  if(!IsDebuggerPresent())
  {
    return DebugMain(argc,argv); 
  }
  cin>>key;
  __asm int 3;
  return 0;
}

int DebugMain(int argc, char* argv[])
{
  char filename[260];
  GetModuleFileName(0,filename,260); //获取自身文件名
  STARTUPINFO  si={0};
  GetStartupInfo(&si);
  PROCESS_INFORMATION  pi={0};
  if(!CreateProcess(filename,NULL,NULL,NULL,FALSE,DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS,NULL,NULL,&si,&pi)) //创建被调试进程
  {
    return 0; 
  }

  BOOL WhileDoFlag=TRUE;
  DEBUG_EVENT DBEvent ;
  DWORD dwState;
  CONTEXT Regs ;
  Regs.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS;

  BYTE vm_code[20][6]=
  {
    0x00,0x00,0x01,0x03,0x03,0x03, //VMStart
    0x04,0x00,0x00,0x00,0x00,0x00, //VMovReg32Mem32
    0x05,0x00,0x78,0x56,0x34,0x12, //VCmpReg32Imm32
    0x09,0x12,0x00,0x00,0x00,0x03, //VJnz
    0x06,0x00,0x00,0x00,0x00,0x03, //VPushImm32 
    0x0A,0x0C,0x00,0x00,0x00,0x03, //VJmp
    0x06,0x00,0x00,0x00,0x00,0x03, //VPushImm32 
    0x07,0x00,0x00,0x00,0x00,0x03, //VCallImm32
    0x08,0x03,0x03,0x03,0x03,0x03, //VPop   
    0x02,0x00,0x00,0x00,0x00,0x00, //VMExit
  }; 
  BYTE *vm_eip=&vm_code[0][0];

  *((DWORD *)(&vm_code[1][2]))=(DWORD)&key;
  *((DWORD *)(&vm_code[4][1]))=(DWORD)r;
  *((DWORD *)(&vm_code[6][1]))=(DWORD)w;
  *((DWORD *)(&vm_code[7][1]))=(DWORD)ShowMessage;

  while (WhileDoFlag) 
  {
    WaitForDebugEvent (&DBEvent, INFINITE);
    dwState = DBG_EXCEPTION_NOT_HANDLED ;
    switch (DBEvent.dwDebugEventCode)
    {
      case CREATE_PROCESS_DEBUG_EVENT:
        dwState = DBG_CONTINUE ;
        break;       

      case EXIT_PROCESS_DEBUG_EVENT :
        WhileDoFlag=FALSE;
        break ;
       
      case EXCEPTION_DEBUG_EVENT:
        switch (DBEvent.u.Exception.ExceptionRecord.ExceptionCode)
        {
          case EXCEPTION_BREAKPOINT: //下面相当于 VM 的调度器
          {
            GetThreadContext(pi.hThread, &Regs);
            if(*vm_eip!=NULL)
            {
              vm_eip+=Regs.Eax;
            }
            else
            {
              vm_eip+=1;
            }
            if(*vm_eip!=NULL)
            {
              DWORD addrCC=(--Regs.Eip); //int 3 指令地址
              Regs.Esp-=4;
              WriteProcessMemory(pi.hProcess,(LPVOID)Regs.Esp,&addrCC,sizeof(addrCC),NULL); //将 int 3 指令地址放入堆栈。
              Regs.Eip=(DWORD)VMHandlerTable[*vm_eip]; //修改 eip 为 handler 地址
              WriteProcessMemory(pi.hProcess,(LPVOID)vmcontext.arg,vm_eip+1,5,NULL); //传入参数
              SetThreadContext(pi.hThread,&Regs);
            }
            dwState = DBG_CONTINUE ;
            break;
          }
        }
        break;
    }     
    ContinueDebugEvent(pi.dwProcessId, pi.dwThreadId, dwState) ;
  }

  CloseHandle(pi.hProcess) ;
  CloseHandle(pi.hThread)  ;
  return 0;
}
```

实际上就是将`VM`的`Handler`留在子进程中执行，而将`VM`的调度器和伪代码放在父进程的执行代码中，在父进程的控制下，实现`vm`在双进程之间的"来回切换"（表达不是很确切），从而增加逆向跟踪的难度。我贴出来的只是一个简单的测试代码，还缺少很多很多东西，感兴趣的朋友可以去实现一个完整的双进程`vm`模型。

ps：参考了`Bughoho`版主的虚拟机源代码。
