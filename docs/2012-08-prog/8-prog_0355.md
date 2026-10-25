
# WIN95 中代码不用 VXD 获得 0 级特权的方法

> 原文：[`www.pediy.com/kssd/pediy04/kanxue177.htm`](https://www.pediy.com/kssd/pediy04/kanxue177.htm)

WIN95 中代码不用 VXD 获得 0 级特权的方法

袁哥

作过 WIN95 的程序的程序员可能都知道 WIN95 提供了 VXD 驱动接口，以方便第三方软件人员编写驱动程序，WIN95 把他们（VXD）放在 CPU 的 0 级特权下执行，可以直接访问硬件，还有执行一些特权指令。而我们往往就需要程序能直接访问硬件或者执行特权指令，但又不方便再增加一个文件（VXD 与程序文件是分开的），还有一个原因是 VXD 的编写不熟，那就希望在自己的程序里能获得 CPU0 级的特权。WIN95 是没给应用程序这特权的，应用程序是在 CPU3 级下执行.怎么办呢？

让我们想一想 CPU 硬件上到底是怎样区分代码是在几级执行的？原来是段描述符里，那我们自然想到自己能不能修改段描述符呢？段描述符是放在 GDT，LDT(应用程序的一般是放在 LDT 里)。关键就得找到 LDT,看怎样想办法能获得改写他的权利。那 LDT 在哪里呢？我们再想一想，WIN95 肯定得访问他吧。分配段什么的，就得在上面建立段描述符。我们来找一找 WIN95 的 API 里有这样的函数吗？

很高兴找到了 `AllocDstoCsAlias` 等。那我们来看一看这 API 怎么访问的。我们编一小程序调用这 API，再用 TDW 调试软件跟踪。真高兴，这 API 是在 CPU3 级执行，但这处理过程完全在里面，原来 WIN95 的 LDT 本声有一个 3 级的数据段指向他，还可写的呢。3 级，那不我也可以读写了吗，我也可以自己建立段描述符了呀，还有调用门呀，可以给自己开几个后门呀。

下面就是用 16 位的 BORLAND C 编写的获得这数据段的程序段：

```
        LPSTR LibFileName="krnl386.exe\0";
    LPSTR FunName="AllocCStoDSAlias\0";
    FARPROC FunAddress;
    HANDLE LibHandle;
    WORD segFun ;
    LibHandle=LoadLibrary(LibFileName);
    FunAddress=GetProcAddress(LibHandle,FunName);
    segFun=HIWORD(FunAddress);
    _DX=segFun;
    asm {
      push ds
          mov ds,dx
      mov dx,word ptr [0x6]  //指向 windows95 的局部描述符的数据段，
                              // window3.2 mov ds,word ptr [0x32]
                              //你可以分析各个版本的不同，加上版本等。
      pop ds
        }
```

现在的 `DX` 就是这数据段，你可以在这上面建立 0 级代码段还有调用门，你最好是先用 API 分配一个段描述符，再在上面修改，用完后释放，要不你建立的系统再改写那就完了。

相信现在你可以自由发挥了。什么打狗棒呀，端口读写的拦截，磁盘解密呀都轻轻松松了！还有你可以访问系统本生，象动态翻译软件的显示 API 的拦截你可以在系统内部做到，避免在显示 API 的开始用 JMP 的弊病。还有你编程得小心了，要不系统会死翘翘的哟！

WINNT 的处理是通过中断门进入了 0 级，所以指向 LDT 的数据段是 0 级的，但大家也可以去分析分析 WINNT，想想别的办法（相应 WIN95 里的 VXD，在 WINNT 里有 SYS，一般在目录 `%SYSTEM%\system32\drivers\` 里的 `.SYS` 文件就是）。

4dd466d0e1e0cc123afbe504da2462ca
