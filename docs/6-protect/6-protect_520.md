

# 加壳要细心,我给 bambam 调错纪实

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-557.htm`](https://www.pediy.com/kssd/pediy08/pediy8-557.htm)

给程序加壳就是给程序动手术,一点也马虎不得,稍微一不留神就出问题.  
本人一直学习使用 c/c++做壳,所以 bambam 就是最好的参考(源码下载中有所有源码).  
  
学习的过程就是读别人的代码,确实原理上不难,但实际一操作还挺麻烦的.这里讲一个给 bambam 调错的亲身经历吧.  
  
  
背景:  
bambam 是 bedrock 根据 bigboot 的文章做的一个壳,代码写的很不错了.但他有的地方处理的不太合理.TLS 是这个壳一个亮点,他使用了 tls 代理技术,巧妙的处理了 tls,这个东西还是很实用的,尽管 99%的程序不用他.borland 编译的程序都有 tls,但多数都是 0,所以其实也没用.但如果这 16 个字节处理不好也会有问题.  
  
下面是 bambam 的代码,  
inline void FinaliseTlsStuff()  
{  
  PIMAGE\_DOS\_HEADER pDosHdr;  
  PIMAGE\_NT\_HEADERS pNtHdr;  
  DWORD dwKatSup;  
  LONG lJmp;  
  
  pDosHdr = (PIMAGE\_DOS\_HEADER)dwLoadAddress;  
  lJmp = pDosHdr->e\_lfanew;  
  dwKatSup = (DWORD)pDosHdr;  
  dwKatSup += lJmp;  
  
  pNtHdr = (PIMAGE\_NT\_HEADERS)dwKatSup;  
  
  if(pNtHdr->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_TLS\].VirtualAddress != 0)  
  {  
        gev.tlsOriginal.AddressOfIndex = (DWORD\*)dwTlsSlotIndex;  
  
    void\* pvTlsData;  
      
    \_\_asm  
    {  
      mov ecx, DWORD PTR dwTlsSlotIndex;  
      mov edx, DWORD PTR fs:\[02ch\]  
      mov ecx, DWORD PTR \[edx+ecx\*4\]  
      mov pvTlsData, ecx  
    }  
  
    int size = gev.tlsOriginal.EndAddressOfRawData - gev.tlsOriginal.StartAddressOfRawData;  
  
    CopyMemory(pvTlsData, (void\*)gev.tlsOriginal.StartAddressOfRawData, size);  
    FillMemory((void\*)gev.tlsOriginal.EndAddressOfRawData, gev.tlsOriginal.SizeOfZeroFill, 0);  
  }  
  fSafeToCallback = true;  
  if(fDelayedTlsCallback)  
  {  
    TlsCallback(tlsDllHandle, tlsReason, tlsReserved);  
  }  
}  
  
看上去这个代码写的很漂亮,而且运行起来也没问题,至少在 windows9x-2003 都可以很好的工作.但其实里面有个 bug.  
  
发现 bug:  
bug 发现的很偶然,因为一般运行的时候都没问题.但有一次我在 stub 中开了一个线程,又在这个线程中开了第二个线程,这样的壳在 win2000 系列的平台是没问题的,但在 win98 下会报错.而且只有 delphi 的程序报错.  
  
起初我以为是 stub 中线程的错误,就反覆修改,但还是同样的报错.就这样让我郁闷了很长时间.我觉的问题和 tls 有关,但就是找不到问题.  
  
注意,win98 下 tls 也是先启动的,所以可以确认 win98 是有 tls 支持的.  
  
我甚至 pediy 一个 exe,没有 tls,他就不报错了.所以现在可以确定 bug 就在这段代码中.说实话,找这个 bug 很郁闷,因为很多的地方很难猜测系统运行的过程,因为系统 loader 报的错,就是 olly 能做的也很有限了.  
  
只能从代码入手,一点点的看,突然,你可以看到有一个地方似乎有点古怪,gev.tlsOriginal.AddressOfIndex = (DWORD\*)dwTlsSlotIndex; 实际上语法是没有问题的,但 dwTlsSlotIndex 是变量,能这么搞吗?当然可以这样做,其他的地方这样赋值很多,但这里似乎是错误的,应该改为 gev.tlsOriginal.AddressOfIndex = (DWORD)&dwTlsSlotIndex;, 这样 win98 下就 ok 了.  
  
这个问题修改是成功了,但我心理还是有些疑问,为什么 win2000,xp 就没事呢?  
  
总结:  
打壳就是动手术,一点都不能马虎,否则出了问题不好解决,主要是症状不好调试.这个文章写了我真实的经历,献给那些自己动手做壳的人吧. 

exetools 论坛里面说了一个 bug:  
\---------------------------------------------  
First bug in stub:

Code:

hMod = LoadLibrary((LPCSTR)dwTemp);

if(hMod != INVALID\_HANDLE\_VALUE)

LoadLibrary return NULL in case of error, not INVALID\_HANDLE\_VALUE (-1)

\---------------------------------------------  

我找到一几个 bug：  
1.在对 stub 进行重定位处理时，有行代码错行了。

2.在 转换 stub 中 thunk 值时，搜索地址的方法有错漏。  
  
3\. 没有对 pe 文件中的多余数据(pe section 中未标明的)进行保留处理。

4\. 没有对 stub 的 VOffset 和 ROffset 进行对齐处理

5\. 对 stub 的输入表起始地址处理有错 

 4dd466d0e1e0cc123afbe504da2462ca