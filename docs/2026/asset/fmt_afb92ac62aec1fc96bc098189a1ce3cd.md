
# 为什么内核中不能直接调用`NtReadFile()`呢?

因为一方面`NtReadFile`不是导出函数，另外就是调用这些函数需要系统堆栈上有框架，但是内核中直接调用的话，肯定没有为本次调用的框架存在，可能是其他框架，比如自陷框架、中断异常框架，所以导致直接调用`NtReadFile`出错，可以通过调用`ZwReadFile()`调用

## x86

```
nt!ZwReadFile:

83e874cc b811010000      mov     eax,111h//因为是内核直接调用 所以无须保存各个寄存器

83e874d1 8d542404        lea     edx,[esp+4]//使edx指向堆栈上的参数快

83e874d5 9c              pushfd   // eflags

83e874d6 6a08            push    8//cs

83e874d8 e831130000      call    nt!KiSystemService (83e8880e)//通过调用它来形成框架

83e874dd c22400          ret     24h
```

## x64

```
nt!ZwReadFile:

fffff800`03c6f6e0 488bc4          mov     rax,rsp

fffff800`03c6f6e3 fa              cli

fffff800`03c6f6e4 4883ec10        sub     rsp,10h

fffff800`03c6f6e8 50              push    rax

fffff800`03c6f6e9 9c              pushfq

fffff800`03c6f6ea 6a10            push    10h

fffff800`03c6f6ec 488d05dd310000  lea     rax,[nt!KiServiceLinkage (fffff800`03c728d0)]

fffff800`03c6f6f3 50              push    rax

fffff800`03c6f6f4 b803000000      mov     eax,3

fffff800`03c6f6f9 e902690000 jmp  nt!KiServiceInternal (fffff800`03c76000)//KiServiceLinkage 和KiServiceInternal 都是初始化系统服务的里面会调用KiSystemServiceStart->KiSystemServiceRepeat(选择ssdt 还是shadow ssdt执行调用)->KiSystemServiceExit(退出调用)

fffff800`03c6f6fe 6690            xchg    ax,ax
```

![](https://bbs.kanxue.com/upload/attach/201704/620086_off2hzxvnose124.jpg)

最近身体出问题了，所以进度很慢，希望大家多多支持

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [windows7内核分析之x86&x64第二章系统调用.docx](javascript:void(0);)

  （122.19kb，170次下载）
