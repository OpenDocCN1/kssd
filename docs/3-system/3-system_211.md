

# 旧调重弹－SetUnhandledExceptionFilter 的使用问题

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-759.htm`](https://www.pediy.com/kssd/pediy07/pediy7-759.htm)

旧调重弹－SetUnhandledExceptionFilter 的使用问题  
kongfoo/2005.12.19  
  
. Delphi 中使用 SetUnhandledExceptionFilter 的问题  
  
　　这几天在写一个测试程序，包括使用 SetUnhandledExceptionFilter 来  
反 OD 调试，关于 SetUnhandledExceptionFilter 的反调试原理之前（一年前了哈）  
simonzh2000 等人都有详细讲述，这里就不多讲了，在 Delphi 里面要用这个  
函数却有点问题，先看看代码：  
  
program SetUnhandledExceptionFilterTest;  
uses Windows;  
  
procedure Handler;  
begin  
  MessageBox(Hwnd(0),'Final SEH','.',0);  
  asm  
    mov eax,1  
    retn  
  end;  
end;  
  
begin  
  SetUnhandledExceptionFilter(@Handler);  
  asm  
    xor edx,edx  
    idiv edx  
  end;  
end.  
  
　　很简单的一个程序，但得到的效果却不是预期之中的，在弹出'Final SEH'  
之后又弹出一个 Runtime error。如果在窗体之中的按钮事件里面写这样的测试  
代码的话，效果更加是“离奇”，弹出一个“除 0 错”之后就没事发生了，根本  
没有去 Handler 里面。  
　　究其原因，Delphi 编译出来的程序有自己的一套用 SEH 建立起来的异常处理  
机构，是自动的，在程序启动之初就建立起来了。就连上面这样简单的程序也一  
样，在 SetUnhandledExceptionFilter 之前就建立了一个 SEH 结构。如果要达到预  
期效果：发生异常时去到我们的 Handler 里面，然后就完事了，我们就要先清掉  
Delphi 建立的 SEH。再来看看代码：  
  
program SetUnhandledExceptionFilterTest;  
uses Windows;  
  
procedure Handler;  
begin  
  MessageBox(Hwnd(0),'Final SEH','.',0);  
  asm  
    mov eax,1  
    retn  
  end;  
end;  
  
begin  
  //首先要清掉 SEH  
  asm  
   @Loop:  
    mov eax,fs:\[0\]  
    cmp dword ptr \[eax\],$FFFFFFFF  
    je @SEHCleared  
    mov eax,\[eax\]  
    mov fs:\[0\],eax  
    jmp @Loop  
   @SEHCleared:  
  end;  
  SetUnhandledExceptionFilter(@Handler);  
  asm  
    xor edx,edx  
    idiv edx  
  end;  
end.  
  
　　现在程序按照我们的想法运行了。在窗体的按钮 OnClick 事件中使用也正常。  
SetUnhandledExceptionFilter 在高级语言中应用时应该要注意一下这个问题了。  
特别是用高级语言写壳用到这个反调试技巧时。另外说一说在异常处理函数返回  
值为-1 时的问题。hume 在《SEH in ASM 研究》中的资料说到：  
  
EXCEPTION\_EXECUTE\_HANDLER equ 1 表示我已经处理了异常,可以优雅地结束了   
EXCEPTION\_CONTINUE\_SEARCH equ 0 表示我不处理,其他人来吧,于是 windows 调用默认的处理程序显示一个错误框,并结束   
EXCEPTION\_CONTINUE\_EXECUTION equ -1 表示错误已经被修复,请从异常发生处继续执行   
  
　　网上搜到一些关于 SetUnhandledExceptionFilter 的文章演示的都是返回值  
为 1 的情况，而在反调试当中，这种情况是没用的，因为返回到系统之后程序就  
结束了，有用的是返回为-1 的情况。具体设置和我们平时的 SEH 异常处理函数类  
似，在到达 SetUnhandledExceptionFilter 指定的异常处理函数之后，\[\[esp+4\]+4\]  
处就是 CONTEXT STRUCT。看代码可以更清晰（测试环境：winxp sp2）：  
  
//这里是用 SetUnhandledExceptionFilter 设置的异常处理函数  
@Handler:  
  mov eax,\[esp+4\]  
  mov eax,\[eax+4\]  
  add dword ptr \[eax+0B8\],2 //异常指令 2 个字节长，加 2 指向下一条指令。eax 指向 CONTEXT STRUCT。  
  mov eax,-1  
  retn  
  
//设置代码：  
  push offset @Handler  
  call SetUnhandledExceptionFilter  
  xor edx,edx  
  idiv edx  
  nop //可以在这里下断测试（当然，OD 要 patch UnhandledExceptionFilter）。  
  
　　OK，收工。 

 4dd466d0e1e0cc123afbe504da2462ca