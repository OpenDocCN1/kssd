
# 给 .Net 程序打内存补丁(1)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-307.htm`](https://www.pediy.com/kssd/pediy08/pediy8-307.htm)

给 .Net 程序打内存补丁(1)

by: tankaiha[NE365]

2006.8.23

any problem: visit `http://vxer.cn`

附件下载

内存补丁在破解中的作用不多说了，Win32 平台下的内存补丁技术大家也都很熟悉，这里主要讲 .Net 平台下可执行程序的内存补丁。换一个说法，叫动态地改变正在执行的 .Net 可执行程序的指令（或数据）。传统的技术在 .Net 下不能用了吗？也不是，但是给 `JIT` 即时编译 `MSIL` 代码生成的 `asm` 代码打补丁，难度有点大。我们需要的，是直接在 `MSIL` 的基础上进行修改。此技术我也在学习中，学一点写一点，有错误请大家指正。

前置知识及参考文献（文献都可在 `google` 中搜到，`MSDN` 中还有很多相关文档）：

1. `MSIL` 基本知识
2. [Rewrite MSIL Code on the Fly with the .NET Framework Profiling API (Aleksandr Mikunov)](http://msdn.microsoft.com/msdnmag/issues/03/09/NETProfilingAPI/)
3. [Modifying IL at runtime (Julien Couvreur's)](http://blog.monstuff.com/archives/000058.html)
4. [The .NET Profiling API and the DNProfiler Tool (Matt Pietrek)](http://msdn.microsoft.com/msdnmag/issues/01/12/hood/default.aspx)
5. .Net FrameWork SDK 中的相关文档

如果有可能，在阅读本文之前先阅读上面的文献，这样会轻松一些。

## 一、.Net Profiling API

什么是 .Net Profiling API，从字面上看就是这种 `API` 可以提供 .Net 运行情况的概况。而实际情况是它提供的功能远远不只如此，动态修改 `MSIL` 代码就是用的这些 `API`。如果你已经看了上面的参考文献，应该已经知道它是干吗的。

通俗的说，`Profiling API` 是 .Net 平台提供的底层接口之一（还有 `Debug` 和 `Metadata` 相关的，待续），通过该接口提供的方法，我们可以得到（及控制）以下过程的相关信息：

- Application 的开始／结束
- Assembly 的载入／卸载
- Methods 的开始／结束
- Module 的载入／卸载
- Class 的类的载入/卸载
- 线程
- 与 `COM` 接口的互操作
- 托管／非托管代码的即时编译
- 等等……

基本上 .Net 的核心操作都可以通过 `Profiling API` 来得到信息。

## 二、使用 Profiling API

这里结合一个例子说，因为该接口太复杂，不可能一下学完全部的功能。我们结合这篇文章（在 .NET Framework 2.0 中，没有任何代码能够逃避 Profiling API 的分析）中的例子，在其基础上修改，加入我们需要的功能。修改的方法应用了参考文献 2 的方法。

先去网上搜索上面的文章，把它的附件下来，解压。有四个目录，我们关心的就是 `Profiler` 目录中的文件。

`Profiling API` 是通过编译成 `dll` 文件，注册为系统的 `com` 服务而实现的。（对 `com` 我不熟悉，不多说），因此，你见到的 `profiler` 的例子通常都是三个文件：`ProfilerCallback.h`，`ProfilerCallback.cpp`，`Profiler.cpp`。前两个是实现 `Profiler` 的核心功能代码，最后一个只是实现了 `dll` 的基本构架。我们需要的就是在 `ProfilerCallback.cpp` 添加代码实现功能。

首先找到 `CProfilerCallback::JITCompilationStarted`，顾名思义，当 `JIT` 引擎开始编译 `MSIL` 代码的时候，会执行这里的代码。`CProfilerCallback` 是什么？看一下定义：

```cpp
class CProfilerCallback : public ICorProfilerCallback2
```

它就是作者实现的类，继承自 .Net 的核心接口 `ICorProfilerCallback2`。这里的 `2` 应该代表是针对 .Net 2.0 的。（我试过，.net v1.1 和 v2.0 下分别生成的 `Profiler`，对不同版本的执行程序有时存在兼容性问题。）`ICorProfilerCallback2` 接口很复杂，但琐碎的工作都由作者做好了，我们只是使用现成的类。

## 三、修改的目标

目标程序是一个简单的程序，如图：

![](img/50403f17d2176b944dddbf6567393406.jpg)

代码如下（在 `VS2005` 中编译，运行于 .Net FrameWork 2.0）：

```csharp
namespace tmp
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }

        private void button1_Click(object sender, EventArgs e)
        {
            if(textBox1.Text!="tankaiha")
            {
                MessageBox.Show("Wrong!");
            }
            else
            {
                MessageBox.Show("You get it");
            }
        }
    }
}
```

也就是如果文本框中输入了“tankaiha”，则显示 `You get it`。输入其它则显示错误。用 `ildasm` 可以查看这段代码的 `IL`：

```asm
IL_0000:  ldarg.0
IL_0001:  ldfld      class [System.Windows.Forms]System.Windows.Forms.TextBox tmp.Form1::textBox1
IL_0006:  callvirt   instance string [System.Windows.Forms]System.Windows.Forms.Control::get_Text()
IL_000b:  ldstr      "tankaiha"
IL_0010:  call       bool [mscorlib]System.String::op_Inequality(string,
                                                                string)
IL_0015:  brfalse.s IL_0023
IL_0017:  ldstr      "Wrong!"
IL_001c:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string)
IL_0021:  pop
IL_0022:  ret
IL_0023:  ldstr      "You get it"
IL_0028:  call       valuetype [System.Windows.Forms]System.Windows.Forms.DialogResult [System.Windows.Forms]System.Windows.Forms.MessageBox::Show(string)
IL_002d:  pop
IL_002e:  ret
```

我们的目标，把 `IL_0015` 处的 `brfalse.s`（`0x2C`）改为 `brtrue.s`（`0x2D`）。

## 四、修改 ProfilerCallback.cpp

下面要跟着源码看了。

`GetFullMethodName()`：取得即将编译的方法的名称。我们需要调用该方法来比较是不是到了 `tmp.Form1.button1_Click`。

```cpp
GetFullMethodName (functionId, wszMethod, NAME_BUFFER_SIZE);
if (lstrcmpW(wszMethod,wszTarget)!=0)
{
  goto exit;
}
```

假如比较结果 `OK`，我们已经进入了 `button1_Click`，则接着获得该方法的详细信息。

```cpp
hr = m_pICorProfilerInfo->GetFunctionInfo(functionId, &classId, &moduleId, &tkMethod );
if (FAILED(hr)) 
{ goto exit; }
hr = m_pICorProfilerInfo->GetILFunctionBody(moduleId, tkMethod, &pMethodHeader, &iMethodSize);
if (FAILED(hr)) 
{ goto exit; }
IMAGE_COR_ILMETHOD* pMethod = (IMAGE_COR_ILMETHOD*)pMethodHeader;
```

.Net 中的方法分为 `fat` 和 `tiny`，具体区别见参考文献。因此先判断到底是 `fat` 还是 `tiny`，然后分别处理。我们是入门，只考虑最简单的情况。

```cpp
if(IsTinyHeader(pMethod)) 
  {
    COR_ILMETHOD_TINY* tinyImage = (COR_ILMETHOD_TINY*)&pMethod->Tiny;
    //Handle Tiny method
    codeBytes = tinyImage->GetCode();
    ULONG codeSize = tinyImage->GetCodeSize();
    
  }
  else 
  {
    COR_ILMETHOD_FAT* fatImage = (COR_ILMETHOD_FAT*)&pMethod->Fat;
    codeBytes = fatImage->GetCode();
    ULONG codeSize = fatImage->GetCodeSize();
  }
```

在取得了相应的代码块信息后，开始修改。首先分配新的代码块，将老的复制过去，再修改相应的字节，最后将新的代码块分配给 `Click` 方法。

```cpp
//这里开始修改代码
  IMethodMalloc* pIMethodMalloc = NULL;
  IMAGE_COR_ILMETHOD* pNewMethod = NULL;
  hr = m_pICorProfilerInfo->GetILFunctionBodyAllocator(moduleId, &pIMethodMalloc);
  if (FAILED(hr)) 
  { goto exit; }


  pNewMethod = (IMAGE_COR_ILMETHOD*) pIMethodMalloc->Alloc(iMethodSize);
  if (pNewMethod == NULL)
  { goto exit; }
  memcpy((void*)pNewMethod, (void*)pMethod, iMethodSize);


  if(IsTinyHeader(pNewMethod)) 
  {
    COR_ILMETHOD_TINY* newtinyImage = (COR_ILMETHOD_TINY*)&pNewMethod->Tiny;
    codeBytes = newtinyImage->GetCode();
    ULONG codeSize = newtinyImage->GetCodeSize();
    codeBytes[21]=0x2D;//就是这里修改
  }
  else 
  {
    COR_ILMETHOD_FAT* newfatImage = (COR_ILMETHOD_FAT*)&pNewMethod->Fat;
    codeBytes = newfatImage->GetCode();
    ULONG codeSize = newfatImage->GetCodeSize();
    codeBytes[21]=0x2D;
  }

   hr = m_pICorProfilerInfo->SetILFunctionBody(moduleId, tkMethod, (LPCBYTE) pNewMethod);
   if (FAILED(hr)) 
   { goto exit; }

   pIMethodMalloc->Release();
```

这就是主要添加的代码。源文件中还有很多 `LogEntry()`的代码，这是调试时可以显示调试信息，用 `DebugTrack` 就可以看到了。

源代码中还有几处要修改。一处是 `CProfilerCallback::GetEventMask()`。把源代码屏蔽掉，然后 `m_dwEventMask=COR_PRF_MONITOR_JIT_COMPILATION`，这样就可以只接收 `JIT` 编译的相关信息了。第二处是有一个 `sleep` 及 `int 3`，这是原作者方便调试的，删除之。具体修改后的 `ProfilerCallback.cpp` 见附件。

## 五、实测

编译生成了 `Profiler.dll` 后怎么用呢？主要是以下 5 步（注意，在同一个 `cmd` 窗口中操作）：

1. `regsvr32 Profiler.dll`（注册服务）
2. 设置环境变量，主要是两个
   ```
   SET COR_PROFILER={18884ADE-B15B-4af8-BE6C-FE5117BA4B32}
   SET COR_ENABLE_PROFILING=1
   ```
3. 运行 `tmp.exe`（我们的试验程序）。点击 `check` 后，你会看到结果。
4. 关闭 `Profiler`，`set Cor_Enable_Profiling=0x0`。
5. `regsvr32 /u Profiler.dll`。

我打开了所有的调试信息输出，随便输入一些字符串，运行结果如图：

![](img/8c039861b91dccb955ed5150a7837001.jpg)

红圈处清楚地看到，`2C` 已经被改为 `2D` 了。

你还可以继续作试验，多次点击 `check`，已经不再显示 `JIT` 信息了，因为 .net 已经将这些代码存储在缓冲区中了，而且不管你输入什么，都显示 `you get it`。

## 六、结语

这篇文章讲得太少了，很多具体的信息都在相应的参考文献中，如果光看这篇文章会摸不着头脑的。本文的主要目的就是告诉你怎么使用现成的代码，修改为自已的 `Profiler`，更深的东东还要继续摸索。

To be continued

`4dd466d0e1e0cc123afbe504da2462ca`
