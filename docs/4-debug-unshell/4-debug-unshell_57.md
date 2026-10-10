

# QuietRIATT：通过 HOOK DLL 函数重建 IAT

> 原文：[`www.pediy.com/kssd/pediy12/107092.html`](https://www.pediy.com/kssd/pediy12/107092.html)

QuietRIATT：通过 HOOK DLL 函数重建 IAT  
  
作者：Jason Raber (jraber@rri-usa.org)  
Brian Krumheuer (bkrumheuer@rri-usa.org)  
译者：riusksk (泉哥：[`riusksk.blogbus.com`](http://riusksk.blogbus.com))  
  
前言  
对于逆向工程师而言，重建一份庞大的输入地址表（Import Address Table）可能是个很耗时又很单调的过程。当 IAT 完全被 hash 或者 munge（译注：进行大面积改动）时，目前的一些 IAT 重建工具可能无法解析出相关的输入函数，这时我们可以采用其它方法来手工重建输入表。随着输入表大小的不同，重建它可能得花耗数天，甚至数周。当然，如果手工重建的话，还可能会产生一些错误。本文讲到的 QuietRIATT 是作为 IDA Pro 插件来使用的，当目前的一些 IAT 工具无法重建 IAT 时，就可以使用它来自动重建 IAT。这样不但可以大大地缩短所花费的时间，而且可以免除一些手工修复的错误。  
  
详细资料  
QuietRIATT 不像目前的一些 IAT 重建工具那样，它并非简单地读取进程的 IAT，而是超出这些工具所含有的各种追踪方法。QuietRIATT 所与众不同之处，在于它通过 HOOK DLL（Kernel32.dll，User32dll 等等）函数的方法，为进程中所使用的每个 DLL 函数创建一份函数名与返回地址列表，这份列表用于创建一份新的 IAT。当其它工具重建失败时，我们就可以使用这种方法来重建 IAT。除非恶意软件检测到 DLL 函数的踪迹，进而导致进程终止运行。  
  QuietRIATT 是为了方便逆向分析恶意软件而设计的，其中主要是针对那些通过修改或重定向 IAT 以防止 ImpREC 重建 IAT 的恶意软件。通常，像 ImpREC 这样的工具都是使用自动追踪的方式来自动修复 IAT，但如果当 IAT 被完全重定向，或者当使用哈希函数来解析外部引用地址时，这些工具将无法修复 IAT，而 QuietRIATT 就是用于处理这些情况的程序。  
QuietRIATT 主要有三个处理过程：  
1)  作为 Microsoft Detours 的修改版来 hook DLL 函数。当附加进程时，它将记录 DLL 函数的创建时间，并生成一个文本文件，以记录函数名及返回地址。  
2)  QuietRIATT 分析这一文本文件，并将这些信息注释到 IDA Pro 数据库中。通过这一数据库，它又创建了另一包含 IAT 详细信息的文本文件。  
3)  将这一新文件加载到 ImpREC 中，以便为原始程序的 dump 添加新的 IAT。  
  
…………  
具体内容参见附件：

上传的附件
| ![](img/b7e61e3627e1e92d7b948dd8fdd24fa6.jpg) || IAT.doc  
 |