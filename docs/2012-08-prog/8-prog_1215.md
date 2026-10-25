
# Win32 调试 API 的另类应用

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-20.htm`](https://www.pediy.com/kssd/pediy07/pediy7-20.htm)

俺写一个虚拟 CPU，可出错不容易找到，就想到这个办法。让一个程序单步执行，每执行一步就与虚拟 CPU 执行的结果相比较，不同的地方就是我的错。

下面是我的代码，高手别笑！

```cpp
//比较程序执行的每一步，找到不同的地方

void CDebugDlg::Run(HCurrentThread &currentThread)

{

  DEBUG_EVENT debug_event;

  LPEXCEPTION_DEBUG_INFO pdebug_info;

  STARTUPINFO starupInfo;

  BYTE codebuf[30];

  CONTEXT context;

  HANDLE hTread;

  HANDLE hProc;

  PE32Loader * w32;

  STOP_CONDITION cond;

  DWORD rip,readsize;

  HFile file;

  if(!file.OpenForRead(m_path))

    return;

  if(!(w32= new PE32Loader))

    return;

  w32->Attach(&file);

  w32->LoadModule(m_vm);

  m_vm.SetMemory(w32);

  rip=w32->m_pe.GetEntry();

  memset(codebuf,0,sizeof(codebuf));

  ::GetStartupInfo(&starupInfo);

  if(::CreateProcess(m_path,"",NULL,NULL,FALSE,DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS,NULL,NULL,&starupInfo,&pi))

  {

    for(;;)

    {

      if(WaitForDebugEvent(&debug_event,INFINITE))

      {

        switch(debug_event.dwDebugEventCode)

        {

        case CREATE_PROCESS_DEBUG_EVENT://

          //保存线程和进程句柄

          hTread=debug_event.u.CreateProcessInfo.hThread;

          hProc=debug_event.u.CreateProcessInfo.hProcess;

          //把程序入口指令改为单步中断 int 3

          readsize=0;

          ::ReadProcessMemory(hProc,(LPCVOID)rip,codebuf,1,&readsize);//保存原入口指令字节

          codebuf[10]=0xcc;//中断 int 3 的机器码

          readsize=0;

          ::WriteProcessMemory(hProc,(LPVOID)rip,&codebuf[10],1,&readsize);

          ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId,DBG_CONTINUE);

          break;

        case EXCEPTION_DEBUG_EVENT:

          pdebug_info=&debug_event.u.Exception;

          switch(pdebug_info->ExceptionRecord.ExceptionCode)

          {

          case EXCEPTION_BREAKPOINT:

            context.ContextFlags=CONTEXT_CONTROL|CONTEXT_INTEGER;

            ::GetThreadContext(hTread ,&context);

            if(context.Eip==rip+1)

            {

              //如果是修改过的入口则恢复为原来的指令重新执行。

              readsize=0;

              ::ReadProcessMemory(hProc,(LPCVOID)rip,&codebuf[10],10,&readsize);

              if(codebuf[10]==0xcc)

              {

                context.EFlags|=0x100;//开始单步

                context.Eip=rip;//重新执行

                //设置虚拟 CPU 的寄存器内容

                m_vm.m_cpu.rReg[AMD_RIP_INDEX].qword=context.Eip;

                m_vm.m_cpu.rReg[AMD_RAX_INDEX].qword=context.Eax;

                m_vm.m_cpu.rReg[AMD_RBX_INDEX].qword=context.Ebx;

                m_vm.m_cpu.rReg[AMD_RCX_INDEX].qword=context.Ecx;

                m_vm.m_cpu.rReg[AMD_RDX_INDEX].qword=context.Edx;

                m_vm.m_cpu.rReg[AMD_RSI_INDEX].qword=context.Esi;

                m_vm.m_cpu.rReg[AMD_RDI_INDEX].qword=context.Edi;

                m_vm.m_cpu.rReg[AMD_RBP_INDEX].qword=context.Ebp;

                //恢复为原来的指令。

                readsize=0;

                ::WriteProcessMemory(hProc,(LPVOID)rip,&codebuf,1,&readsize);

                ::SetThreadContext(hTread,&context);

              }

            }

            ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId,DBG_CONTINUE);

            break;

          case EXCEPTION_SINGLE_STEP:

            context.ContextFlags=CONTEXT_CONTROL|CONTEXT_INTEGER;

            ::GetThreadContext(hTread ,&context);

            cond.type=STOP_STEP;

            cond.cond_info.dwords[0]=m_vm.GetCurrentSetpCount()+1;

            m_vm.Run(&cond);

            if(!CmpCpu(context))

            {

              MessageBox("EXCEPTION_SINGLE_STEP");

            }

            context.EFlags|=0x100;//开始单步

            ::SetThreadContext(hTread,&context);

            ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId,DBG_CONTINUE);

            break;

          }

          break;

        case EXIT_PROCESS_DEBUG_EVENT:

          return;

          break;

        case RIP_EVENT:

          ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId,DBG_TERMINATE_PROCESS);

          break;

        default:

          ContinueDebugEvent(debug_event.dwProcessId,debug_event.dwThreadId,DBG_EXCEPTION_NOT_HANDLED);

        }

      }

    }

  }

}

比较 CPU 寄存器内容是否相同

BOOL CDebugDlg::CmpCpu(CONTEXT &context)

{

  if(m_vm.m_cpu.rReg[AMD_RAX_INDEX].dwords[0]!=context.Eax||

    m_vm.m_cpu.rReg[AMD_RBX_INDEX].dwords[0]!=context.Ebx||

    m_vm.m_cpu.rReg[AMD_RCX_INDEX].dwords[0]!=context.Ecx||

    m_vm.m_cpu.rReg[AMD_RDX_INDEX].dwords[0]!=context.Edx||

    m_vm.m_cpu.rReg[AMD_RSI_INDEX].dwords[0]!=context.Esi||

    m_vm.m_cpu.rReg[AMD_RDI_INDEX].dwords[0]!=context.Edi||

    m_vm.m_cpu.rReg[AMD_RBP_INDEX].dwords[0]!=context.Ebp||

    m_vm.m_cpu.rReg[AMD_RIP_INDEX].dwords[0]!=context.Eip)

    return FALSE;

  return TRUE;

}
```

俺写的虚拟 CPU 是 `hdasm64`，请大家下载试用：

本身是一个 Win32 程序（`PE32` 格式）在下载的包里有 64 位的 `PE` 和 `ELF` 文件各一个，32 位 `ELF` 文件一个，供大家研究！

支持实模式、保护模式、64 位模式三种模式指令集的动态反汇编. 支持 DOS 系统 `COM` 和 `EXE` 可执行文件格式，支持 32 位和 64 位 `PE` 文件格式（`windows` 可执行文件），支持 32 位和 64 位 `ELF` 文件格式（`Linux` 可执行文件）,共计六种文件格式。支持部分指令的虚拟执行调试。

下载页面：

[`pay500.com/s/s56504.htm`](http://pay500.com/s/s56504.htm)

`4dd466d0e1e0cc123afbe504da2462ca`
