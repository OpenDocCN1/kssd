
# .net 下某强壳的不完全分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-600.htm`](https://www.pediy.com/kssd/pediy08/pediy8-600.htm)

`.net` 下某强壳的不完全分析

`tankaiha[ne365]@ http://vxer.cn`

2006-09-12

其实题目有点偏离内容，因为本文并不是对壳的本身进行深入的分析，而是从另一个角度对其保护和破解进行分析。这个壳是啥偶就不说了，呵呵。（壳作者可是我的偶像）下面开始。

## 1、壳的保护方式简介

该壳的网站上已经有很多篇文章介绍过他的方式，但并不深入。（这是当然，因为已经商业化了。）总的来说，`.net` 的运行机制和 `win32` 平台下有很多不同，比如 `.net` 的 `PE` 载入内存后，不是立刻将所有的代码从 `MSIL` 编译至 `native code`，而是每运行一个 `method`，编译一个 `method`。当某个 `method` 编译过后，它在一定时间内的 `native code` 总是存在于内存中，便于下次直接调用。而由于有垃圾收集系统，一段时间不用后，该 `method` 所占用的资源又可能被释放。

该壳便是利用了 `.net` 的这种特性，被保护的程序所有的 `method` 的代码块大小都被置为 `0`，而真正的代码被壳用多种算法加密保存，但每个 `method` 的定义仍在，当 `jit` 引擎调用某个 `method` 时，被挂钩的程序就是转到壳的解密代码中，将源代码解密完再传给 `jit`。其中包括重建方法表的结构，因为原程序的方法表中代码块的大小是 `0`。这些加密和解密代码都在一个本地 `dll` 中，所以该壳的唯一缺点就是被其加密过的程序无法跨平台。其实这一点对共享软件作者来说并不重要，一个程序可以在所有装 `.net framework` 的平台上运行就足够了。

大家都知道 `.net` 中有 `Profiler`，它可以 `JIT` 开始编译时输出 `IL` 代码。如果这样，那不是可以直接用 `Profiler` 将 `IL` 代码输出吗？非也。壳的作者也在网站上说过，他很了解 `profiler` 和 `jit` 之间的通讯，可以很容易的避开 `Profiler` 的监测。下面，先来看看 `profiler` 和 `jit` 是怎么通讯的。

```
代码:

***

TADDR MethodDesc::MakeJitWorker(COR_ILMETHOD_DECODER* ILHeader, DWORD flags)
{
...
...
            if (CORProfilerTrackJITInfo() && !IsNoMetadata())
            {
                {
                   PROFILER_CALL;
                   g_profControlBlock.pProfInterface->JITCompilationStarted((ThreadID) GetThread(),(CodeID) this,TRUE);
                }
                COR_ILMETHOD *pilHeader = GetILHeader();
                new (ILHeader) COR_ILMETHOD_DECODER(pilHeader, GetMDImport(), NULL);
            }
...
...

***
```

红色的代码就是 `JIT` 通过接口通知 `Profiler` 开始 `JITCompilationStarted`，这时我们可以在 `Profiler` 中 dump 内存中的 `il` 代码。如果这时 `IL` 代码还没有解密呢？（事实也是如此）壳完全可以在发送了该通知后，再调用解密代码，这样，光用 `Profiler` 是没法取得程序的源代码的。

例如，对用该壳加密过的某程序用 `Profiler` 在 `JITCompilationStarted` 里 dump 得到的内容如下：

`JITCompilationStarted:`

`Class name is:`

`CFunction name is: x`

`enter fat codeFlags: FFF`

`MaxStack: 9600`

`CodeSize: FFFF07E7`

`LocalVarSigTok: FFFFFFF`

`E707FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`

`FFFF0083E707FFFFFFFFFFFFFFFFFFFFFFFF1A28790000062AFFFF8089E707FFFFFFFFFFFFFFFFFFFFFF8088E707`

`FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFC08BE707FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF80B6E707`

`FFFFFFFFFFFFFFFFFFFFFFFFFFFF03300A002B000000000000002879000006286503000A80ED0B000428F807000A`

`80EE0B000428F700000A80EF0B000420000000802CE52AFFFF80A6E707FFFF1A28790000062AFFFF80A3E707FFFF`

`1A28790000062AFFFF80A0E707FFFFFFFFFFFFFF80A1E707FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`

`FFFFFFFFFFFF40BBE707FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF00B7E707`

`FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFC0B1E707FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF`

`FF........`

不用说了，一堆乱码。下面，我们就开始看怎么 dump 真正的源代码。

## 2、取得解密后源代码的思路

本文的主要内容就是分析怎样取得壳解密过的源代码，而真正完全的脱壳目前还不行，当然，偶觉得也没必要。我们的思路，既然壳挂钩了 `jit` 与 `ee`（`execute engine`）的通讯，我们也可以中断，在 `jit` 即将进行编译的一瞬前，代码总是解密过的吧，我们要找到这个断点。

`rotor` 是源代码是我们了解以上内容的好资料。我们在 `fjit.cpp` 里可以看到这个函数：

```
代码:

***

/*=============================================
*********/
/* jit the method. if successful, return number of bytes jitted, else return 0 */
FJitResult FJit::jitCompile(
                BYTE ** ReturnAddress,
                unsigned * ReturncodeSize
                )

***
```

看名称很像 `jit` 的关键函数，只是 `rotor` 的代码并不是 `windows` 上 `.net framework` 的代码，所以 `jitCompile` 也不是 `framework` 中相对应的函数的名称。想想别的方法。

`mscorjit.dll` 是 `jit` 引擎的核心文件，其中肯定有从 `ee` 读入 `IL` 的函数。用 `ida` 对其进行反编译，当 `ida` 提示从网上下载文件的符号时，点确认。仔细搜索一下，有这样一个函数

`.text:7906E7F4 private: virtual enum  CorJitResult __stdcall CILJit::compileMethod(class ICorJitInfo *, struct CORINFO_METHOD_INFO *, unsigned int, unsigned char * *, unsigned long *) proc near`

`compileMethod`，光看这个名字就知道他的功能是编译 `method`。看一下输入参数，其中第二个参数是 `struct CORINFO_METHOD_INFO *`

在 `rotor` 的源代码中搜索该结构，可以得到如下定义（尽管不知道两个结构是否完全相同，不妨先假设相同，死马当活马医）

```
struct CORINFO_METHOD_INFO

{

    CORINFO_METHOD_HANDLE       ftn;

    CORINFO_MODULE_HANDLE       scope;

    BYTE *                      ILCode;

    unsigned                    ILCodeSize;

    unsigned short              maxStack;

    unsigned short              EHcount;

    CorInfoOptions              options;

    CORINFO_SIG_INFO            args;

    CORINFO_SIG_INFO            locals;

};
```

前两个又是指向结构的指针

```
typedef struct CORINFO_MODULE_STRUCT_*     CORINFO_MODULE_HANDLE;

typedef struct CORINFO_METHOD_STRUCT_*     CORINFO_METHOD_HANDLE;
```

只是我搜遍了本机和网络也没有找到它们的具体定义，看来又是 `UnDocumented` 的。无所谓，关键是第三和第四个成员。`ILCode` 和 `ILCodeSize`，这是不是就是指向了 `IL` 代码的内存地址呢？而指向的代码是否又被解过密呢？如果是，我们就可以在这里进行中断，从而输出解密后的代码。

## 3、调试

既然已经深入到 `mscorjit.dll` 内部，就无需用 `.net` 调试了，直接用最熟悉的 `OllyDbg`。先运行被加密过的程序，然后用 `OD` 附加到该进程上，接着 `bp 7906E7F4`，很快，`OD` 便中断了。看堆栈值

```
0012D750        79E9776F  返回到 mscorwks.79E9776F

0012D754        790AF170  mscorjit.790AF170

0012D758        0012D894

0012D75C        0012D920

0012D760        00107210

0012D764        0012D9D4

0012D768        0012D9AC

0012D76C        61893F16

0012D770        00000000

0012D774        0012D920
```

前两行是 `jit` 中的返回地址，原函数中的 `CORINFO_METHOD_INFO *` 应该从第四行开始，在 `0012D920` 上点右键，选“数据窗口中跟随”，来到该址处。数据窗口中内容如下：

```
0012D920  28 70 97 01 14 2C 05 01 59 56 90 01 25 00 00 00  (p?,YV?%...

0012D930  08 00 00 00 00 00 00 00 20 00 00 00 00 00 00 00   

....... .......
```

根据结构定义，红色字体的 `01905659` 是 `IL` 代码的开始处，而该代码的大小是蓝色显示的 `0x25`。继续，查看 `01905659` 处开始的 `0x25` 个字节，数据如下：

```
01905659  72 CE EB 01 70 72 E5 17 00 70 16 1F 40 28 69 01  r 坞 pr?.p@(i

01905669  00 0A 26 02 7B B9 05 00 04 7E 83 01 00 0A 28 C6  ..&{?.~?..(

01905679  07 00 06 26 2A                                 .&*
```

这时，偶写的小工具 `hex2il` 就派上用场了，对这段数据进行反编译，得到如下代码

```
代码:

***

l_0000  ldstr  <string>7001EBCE
l_0005  ldstr  <string>700017E5
l_0010  ldc.i4.0   
l_0011  ldc.i4.s  <int8> 40
l_0013  call  <method>0A000169
l_0018  pop   
l_0019  ldarg.0   
l_0020  ldfld  <field>040005B9
l_0025  ldsfld  <field>0A000183
l_0030  call  <method>060007C6
l_0035  pop   
l_0036  ret   

***
```

由于是直接编译，没有取得原文件的 `metadata` 信息，所以这里 `<string>`、`<field>` 和 `<method>` 都是直接用数值表示的。怎么看它们的名称呢？没事，用 `ildasm` 打开原程序，dump 后的 `IL` 里会有这些 token 相对应的值：

```
// 7001ebce : (34) L"Thank you for registering XXXXXXXX!"

// 700017e5 : ( 7) L"XXXXXXXX"

.field /*040005B9*/ private class XXXXXXXX.x729558e4a99b7fb0/*02000063*//xdd2eae989d3fc452/*0200014A*/ xa479091352f99870

.method /*060007C6*/ private static pinvokeimpl("user32" lasterr winapi)         

bool  EnumWindows(class CodeLib.x729558e4a99b7fb0/*02000063*//xdd2eae989d3fc452/*0200014A*/ xa479091352f99870, native int x7a5ee4e5d933ebbc) cil managed preservesig
```

这里还有两个没有解析出来 `<method>0A000169` 和 `<field>0A000183`，我暂时也不知道为啥，估计是另外载入的 `assembly` 中调用的。其实 `<method>0A000169` 应该是 `MessageBox.show()`，就是显示“谢谢注册”的窗口。（也可能是 `hex2il` 有 BUG![](img/9e41176b4900ac914e77ca870db1e3dd.jpg)）

到此为止，我们已经将源程序的一个 `method` 的源代码导出了。而且完全可以写一个程序，将该程序执行过的所有 `method` 的源 `IL` 都导出。

## 4、结语

工作还没有完，一是怎么将源程序所有的 `method` 都导出？如果某个 `method` 没有执行的话，那 `jit` 就永远不会对它编译。二是即使全部导出了，还要将其全部合并为一个 `PE` 文件，这不是不可能的，只是工作量非常大。但是，该方法的意义在于，只要得到了关键段（比如注册码比较段的源代码），加上足够的耐心分析，你最终会得到它的注册算法的。

本文当作抛砖引玉，希望有兴趣的继续深入。欢迎和我就该问题以及所有 `.net` 下逆向的话题进行讨论！
