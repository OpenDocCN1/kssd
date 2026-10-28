
# [原创]一个Win7X64内核注入32位进程的例子

这是前段时间完成的一个内核注入的代码, 没用采用APC,无线程,无dll,启动时注入。

读取`C:\process.txt`, `process.txt`存放需注入的进程名, 一行一个.

原理是在注册模块回调`\WINDOWS\SysWOW64\KernelBase.dll`的时候修改已经被读入内存的OEP转入shellcode中，运行后再跳回去. 简单易懂. 也没有什么技术含量![](https://bbs.kanxue.com/view/img/face/16.gif)

之所以选择`\WINDOWS\SysWOW64\KernelBase.dll`是因为SysWOW64中的DLL只有32位进程才会加载，这样就不需要判断是否是32位进程了。选择`KernelBase.dll`则是一个模块加载时机问题。其他一些dll加载时机都比较早了. 读取一些信息不太方便, 连PEB都没初始化完![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)。

在修改内存中OEP时为了方便，`X64TrapFrame`都是用的硬编码,都是临时调试出来的。

因为在加载`KernelBase.dll`时, 内存中存放入口地址的指针位于`RSI+0x7B0`,通过直接修改这个值达到目的

```c
//修改读入内存的OEP
PULONGLONG WriteOEP(PEPROCESS curProc, PULONGLONG pBuf)
{
    PEPROCESS proc = curProc;
    PETHREAD pThread = NULL;
    PULONGLONG pX64TrapFrame, pRsi, pRet, pOEP;
    NTSTATUS status;
    //获得KTHREAD
    pRet = ((ULONGLONG)curProc+0x308);
    pThread = *pRet - 0x420;
    //获得KTRAP_FRAME结构
    pX64TrapFrame = (ULONGLONG)pThread + 0x1d8;
    //获得pRsi寄存器
    pRsi = *pX64TrapFrame + 0x150;
    //获得OEP
    pRsi = *pRsi + 0x7B0;
    pOEP = *pRsi;
    *pRsi = pBuf;
    //写入shellcode+0x240
    //mov eax, 0x00000000
    //jmp eax
    RtlCopyMemory( (PUCHAR)pBuf+240, &pOEP, 4 );
    return pOEP;
}
```

自己看吧。可注释掉文件判断测试注入所有32位进程,Win7X64虚拟机测试没问题

近些天无聊在做一个网络防火墙, 搞定后放码,快完了.

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [KernelInject.rar](javascript:void(0);)

  （229.71kb，679次下载）
