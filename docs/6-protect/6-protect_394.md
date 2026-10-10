

# 乱弹

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-652.htm`](https://www.pediy.com/kssd/pediy07/pediy7-652.htm)

真得没事做了，灌灌水吧，随意说了。  
  
OllyDbg 有个 bug （也许不能叫 bug），遇到异常会清掉 DRx，  
如果壳用 DRX 值来解码就会出错。这就是为什么不能直接用 OllyDbg 跟完 hying's PE-Armor 而要中途保存 DRx 的原因。  
  
TLS 的 callback 可以在程序进入入口之前执行（甚至在 import 被装入之前？  
）但听说 9x 下不会。Execryptor 就用这个，习惯了 open with ollydbg 的还得改改，麻烦啊。  
  
令 X=\[GetProcessHeap+0Ch\] and 40h,非调试时应该为 0。  
  
SetUnhandledExceptionHandler 会用 ZwQueryProcessInformation 查询 class=7（可能是 11，我记不清了）的调试信息，只要 hook 那个 native api 就行了。  
  
ZwSetInformationThread 可以把调试端口设置为 0，调试器就收不到 debug event 了。  
  
native api 都是这种形式的：  
mov eax, ServiceId  
mov edx, xxxxxxxx  
call \[edx\]  
retn xx  
2k 下大同小异：  
mov eax, ServiceId  
lea edx, \[esp+4\]  
int 2e  
retn xx  
hying 新版会扫描所有代码直到 retn（并且第一条指令不能为 retn），就没法设置断点了，因为它还会复制代码，memory breakpoint 也不好用。不过可以用这种形式：  
mov eax, ServiceId  
mov edx, PtrToMyPage ;指向自己的内存  
call \[edx\]  
retn xx  
执行后就可以肆意妄为了，不怕检测。更狠得可以 hook SSDT（服务描述表）。  
  
有些程序利用调试器的 bug，我们可以自己写个调试器，只具有基本的功能，  
来完成脱壳不受影响。  
  
  
不知道我在说什么呢。现在流行 VM 啊，一个 opcode table 把 debugger 搞得团团转，设计 VM 是体力活，解 VM 更是体力活，可能 reverse 技术发展到一定程度脱壳真地如 loveboom 说的走不远了。  
  
写代码和调试的时候要听音乐，否则人真的会疯掉。  
  
易语言这东西有前途。  
  
一个好壳除了 AntiDebug 不能太软，输入表和代码处理分支要足够多，200 个以上为佳。  
  
有时间应该写个好的 polymorphic engine，把 drx 解码嵌进去。  
  
。。。。。。 

关于 Tls CallBack 应该是 Import 被装入后进入  
进去的时候，能在 Stack 中看到 ep, 出来后再走走就 call ep 了  
  
其实进程在 ExitProcess 退出的时候， 还会进一次这个 CallBack  
大约是在 ExitProcess 中的 LdrShutDownProcess 中进的  
当然如果喜欢用 int2E 或 sysenter 退进程就不会到那个地方了 

 4dd466d0e1e0cc123afbe504da2462ca