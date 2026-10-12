
ZeroMemory(&pi, sizeof(pi));

ZeroMemory(&si, sizeof(si));

si.cb = sizeof(si);

LPVOID breakAddr = (LPVOID)0x004D940D;  //这是断点地址，就是用工具生成注册机时要填的那个地址

BYTE firstByte = 0x58;      //同样是生成注册机时要填的那个第一字节

BYTE bp = 0xCC;

DWORD real;

BYTE buf[64];

int nCount = 0;

DWORD way;

//用调试方式创建需要破解的程序进程，此处进程为 Cxjk.exe

if(CreateProcess("Cxjk.exe", NULL, NULL, NULL, FALSE,
  DEBUG_ONLY_THIS_PROCESS | DEBUG_PROCESS, NULL, NULL, &si, &pi))
{
  //调试循环体
  while(TRUE)
  {
    //等待调试事件
    if(WaitForDebugEvent(&devent, INFINITE))
    {
      way = DBG_EXCEPTION_NOT_HANDLED;
      switch(devent.dwDebugEventCode)
      {
        case EXIT_PROCESS_DEBUG_EVENT:  //进程退出事件
          //return;
          break;
        case EXCEPTION_DEBUG_EVENT:     //异常事件
          {
            if(devent.u.Exception.ExceptionRecord.ExceptionCode == EXCEPTION_BREAKPOINT)
            {
              nCount++;
              //创建进程后会产生第一次异常中断，此时可以设置断点
              if(nCount == 1)
              {
                //读取第一个字节进行效验
                ReadProcessMemory(pi.hProcess, breakAddr, buf, 1, &real);
                if(buf[0] == firstByte)                  
                {
                  //设置断点
                  DWORD oldProtect;
                  VirtualProtectEx(pi.hProcess, breakAddr, 1, PAGE_READWRITE, &oldProtect);
                  WriteProcessMemory(pi.hProcess, breakAddr, &bp, 1, &real);
                  VirtualProtectEx(pi.hProcess, breakAddr, 1, oldProtect, NULL);
                }
                way = DBG_CONTINUE;
              }
              //如果是在断点处暂停，就可以读取内存中的注册码
              else if(devent.u.Exception.ExceptionRecord.ExceptionAddress == breakAddr)
              {
                GetThreadContext(pi.hThread, &Regs);
                ReadProcessMemory(pi.hProcess, (char *)Regs.Edx, buf, 64, &real);  //从寄存器 Edx 中读取注册码
                MessageBox(NULL, (char *)buf, "注册机 By 冷夜 http://nightx.info/     ", 0);
                //去除断点
                DWORD oldProtect;
                VirtualProtectEx(pi.hProcess, breakAddr, 1, PAGE_READWRITE, &oldProtect);
                WriteProcessMemory(pi.hProcess, breakAddr, &firstByte, 1, NULL);
                VirtualProtectEx(pi.hProcess, breakAddr, 1, oldProtect, NULL);
                //回到断点处继续执行
                Regs.Eip = (DWORD)breakAddr;
                SetThreadContext(pi.hThread, &Regs);
                way = DBG_CONTINUE;
              }
            }
            break;
          }
        }
        ContinueDebugEvent(pi.dwProcessId, pi.dwThreadId, way);
      }
  }
}
else
  MessageBox(NULL, "请将注册机和要破解程序放在同一目录", NULL, 0);

---

声明一下，代码不是我写的，是东拼西凑借鉴程序员的代码得到的。呵呵。运行一下。也成功。

用户名:`冷夜` 注册码;是 `29ACF5B22A1B6323A3A72075DB4447407E1C5D9FBAE52D13`

---

【经验总结】

软件还是很简单的。重要的只是思路而已。希望文章能对初学者有所帮助。

---

【版权声明】: 本文原创于冷夜/nightx, 转载请注明作者并保持文章的完整, 谢谢!

2011 年 08 月 05 日 08:26:48
