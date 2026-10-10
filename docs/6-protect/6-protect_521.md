

# Writing Your Own Packer （部分）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-792.htm`](https://www.pediy.com/kssd/pediy08/pediy8-792.htm)

Writing Your Own Packer - by BigBoote   
\--------------------------------------------------------------------------------  
\[译者\]4nil 于 Jan-2-05  
  
  
Writing Your Own Packer - by BigBoote   
  
\--------------------------------------------------------------------------------  
   
  
Intro   
  
Why write your own packer when there are so many existing ones to choose from? Well, aside from making your executables smaller,  
 packing is a good way to quickly and easily obfuscate your work. Existing well-know packers either have an explicit 'unpack' function, or there are readily available procdump scripts for generating an   
unpacked version.  
市面上已经有这么多的加壳工具可供选择为什么还要自己写呢？答案是，除了让你的软件更加小，加壳还有一个好处就是快而简单的使自己的软件得到保护。现有的知名的加壳工具基本上都含有“脱壳”功能，或者可以得到加壳脚本而写出一个脱壳的版本。  
  
Since this document has quickly exploded in length I'm going to break it up into separate installments. In this installment I  
 will cover the qualitative aspects of producing a packer. I'll discuss what you're getting into and how the packer is   
structured in general. I'll briefly discuss some pitfalls, and I'll give some links to technical information you will need  
 to be familiar with before going into the next installments.  
因为这个文档比较大，我将其分为几个环节。在这个部分里我会描述关于制作加壳工具的质量问题。我会讨论关于如何开始以及一个壳的基本结构。主要涉及一些问题，而且我会提供一些关于技术信息的链接，在你进入第二环节前使你对写壳有个更深入的了解。  
  
In the next two installments I'll go into details of how to implement the components of the packer and how I usually go about producing them.  
在接下来的 2 个环节里我会详细讲述如何完善你的壳和平常如何加在你的程序里。  
  
  
What You're Getting Into  
如何开始  
  
It's not really hard, per se, but it is rather tedious code. Lots of pointer manipulation and translation to keep track of. Aside from  
 that, if you can write code to add and subtract integers and do file IO, you've got all the skill needed! As mentioned, it is tedious   
code so you will probably do well to not attempt this coding on a hangover; trust me, I know.  
说实话其实不难，只是无聊的写代码。很多的指针操作和转换需要明白。把那个放一边，假如你会写代码，会加减整数，还有会操作文件 IO，我想你已经掌握了所有的需要的技术!像先前提到的，这个只是无聊的代码编写，你可以做的很好，相信我，我知道。  
  
FYI, the last packer I produced was fairly full-functioned (exes and dlls, several compression algorithms with debug capability and advanced support such as TLS (critical for Delphi  
 apps)) and it weighed in at about 3700 lines for the packer tool and about 1000 lines for the decompression stub it embeds in the target.  
 That's somewhere around 70 printed pages of code. So, not a huge app, but not a tiny one either. The first one I produced took about 1.5  
 weeks to produce including research and bug fixing. Subsequent ones took far less since I had already done the hard part, which is  
 figuring out how. Hopefully this document will save you that time as well!  
FYI，我做的最新的一个功能相当完备的加壳机（EXEs 和 DLLs, 几种压缩算法，具有调试功能，高级技术的使用入 TLS（对于 Delphi 的程序可能有点错误）），该程序差不多有 3700 行代码用于加壳工具，还有 1000 行用于嵌入目标程序的解压块。打印了 70 页纸。也就是说，不是很大的程序，但也不是一个小的。我第一个开发的差不多花了一个半星期（包括开发和调试）。接下来的几个花的时间就少了，因为我已经将困难的部分做好了。希望这个文档同样可以给你省下时间。  
  
You do not have to use assembler for the most part. If you can part with supporting some esoteric features, you won't have to use it at all.  
 All of that is relevant for the decompression stub only anyway. The packer can be in Logo or Object-Oriented COBOL if you like.  
你大部分时间不需要使用汇编工具。假如你不开发一些深奥的功能，你根本没必要用。所有的都只和嵌入的解压程序段有关，假如你喜欢，你可以使用 LOGO 或者 Object-Oriented COBOL。  
  
OK, enough of the blahblahblah, on to technical stuff....  
好，废话不多说了，开始我们的技术资料。  
  
Big Picture  
总览  
  
Simple. Executable is analyzed, transformed, and an extra piece of code is attached which gets invoked instead of the original program. This  
 piece is called a 'stub' and decompresses the image to its original location. Then it jumps to that original location. But you know this already.  
简单，程序经过分析，转换，然后将一段代码插入程序代替原程序首先被调用，这个被称为‘stub’,解压源程序到原地址，然后它再跳到原地址。这个你已经知道。  
  
Sounds simple, but there are pitfalls that await you. Some of these include:  
听起来简单，但是还有一些问题在等待你，部分如下:  
  
\* Support for simplified Thread Local Storage, which is key in supporting Delphi applications  
支持单一的线程局部存储器(Thread Local Storage),支持 Delphi 程序的关键。  
\* Support for code relocation fixups in dlls if you care about packing dlls. Recall ActiveX controls are dlls too, as are other common things   
you might be interested in packing  
支持 DLLs 加壳,要对代码的偏移修正。同样 ActiveX 也是 DLLs, 假如你感兴趣的话还有其他一般性的东西你需要考虑。  
\* Support for some stuff that must be available even in the compressed form. This includes some of your resources and export names in dlls  
支持压缩形式的数据，包括一些资源和 DLLs 里的输出名。  
\* Dealing with bound imports  
处理被限制的输入。  
\* Support for stripping out directory entries that will confuse the Windows loader since the decompression won't have happened and they will  
 point to nothing useful, like the IAT and debug info  
支持剥除可能使 Windows 载入器出错的目录项，使解压呈现可以执行，防止代码指向无用的数据，比如图象注释带还有调试信息。  
\* Support for doing relocation fixups manually on your decompression stub since it will certainly be in a different memory location than where  
 the linker thought it would be when it was compiled  
支持对解压程序段手工进行偏移矫正，因为被编译后的文件在连接过程中可能内存地址会有出入。  
\* Dealing with differences in interpretation of the PE spec between different vendor's linkers. Borland linkers interpret aspects of the spec   
differently from Microsoft's so you need to be ready for that.  
对不同的连接器必须有不同的 PE 格式解释。Borland 连接器和 Microsoft 的对于格式的解释是不一样的，你必须有准备。  
\* Working around bugs in Microsoft code. There is an infamous one relating to OLE and the resource section. Many packers do not accommodate this  
 and this is important for ActiveX support.  
处理 Microsoft 代码里的 bug。一个很有名的是关于对象链接和嵌入以及资源段。很多加壳工具不能很好的处理这个，而且这个对于支持 ActiveX 是非常重要的。  
  
First Step  
第一步  
  
  
OK, enough of the horror stories. The first step is to get painfully familiar with the file format of executables. This is called the 'Portable  
 Executable' format, or PE for short. I will discuss it briefly here. You will need more detail in reality. Rather than attempting to duplicate  
 that, here are some references you will find helpful:  
讲的恐怖了些。第一步很痛苦，就是要熟悉可执行文件格式（PE，Portable Executable）。我会简单讨论一下，但你需要更多具体的细节。为了避  
免直接抄袭，这里有一些参考资料可能对你有帮助：  
  
The Portable Executable File Format from Top to Bottom   
[`mup.anticrack.de/Randy%20Kat...%20Format.html/`](http://mup.anticrack.de/Randy%20Kath%20-%20PE%20Format.html/)  
  
a good and readable discussion, but not totally accurate when it comes to the import section.   
Dead wrong in implying that these sections always exist -- they easily can not exist. Still, a good read.  
不错而且易理解的讨论，但在涉及输入段（import section）的时候不是很确切。有个致命错误就是认为这些段是必需存在的--事实上他们可以不存  
在。但还是不错的文章。  
  
  
An In-Depth Look into the Win32 Portable Executable File Format pts 1 and 2   
[`www.msdnaa.net/Resources/Display.aspx?ResID=1083`](http://www.msdnaa.net/Resources/Display.aspx?ResID=1083)  
[`www.msdnaa.net/Resources/display.aspx?ResID=1323`](http://www.msdnaa.net/Resources/display.aspx?ResID=1323)  
  
great article, weak on discussion of resource section  
非常不错的文章，在对资源段（resource section）的讨论上比较弱。  
  
  
Microsoft Portable Executable and Common Object File Format Specification   
[`www.microsoft.com/whdc/hwdev...are/pecoff.mspx`](http://www.microsoft.com/whdc/hwdev/hardware/pecoff.mspx)  
  
horse's mouth. Dry. Accurate.  
枯燥无味，但是准确。  
  
  
Next Step  
第二步  
  
OK, after you've gotten familiar with those, we can start to write some code. I'm going to save that for the next installments  
 (probably two). They will detail:  
在你了解了以上内容后，我们可以开始写一些代码了。接下来的部分（可能有两个）我们会具体讲解：  
  
\* Making the Unpacker Stub  
制作脱壳程序段  
The stub has several responsibilities aside from the obvious decompression. It also has to perform duties normally done by the   
Windows loader.  
这个程序段除了解压功能，还要充当 Windows 载入器的功能。  
  
\* Making the Packer Application  
制作加壳程序  
The packer application does all the hard work. This makes since when you realize the stub is supposed to do as little as possible   
to have a minimum impact on runtime.  
加壳程序的制作是最困难的部分。因为你必须考虑到加壳以后对原程序在运行时间上的影响要最小。  
  
I'll try to keep code examples to a minimum but there may be some reference to structure members when describing what's going on   
and maybe a snippet or two where code is clearer than human language. Most of the important structures can be found in WINNT.H for those who wish to read ahead   
我会试着保留一些最精简的代码，但是当描述程序的执行过程时可能涉及到结构成员（structure member）而且有些地方是非常好理解的。大多数重要的的结构可以在 WINNT.H 里找到，你可以去看一下。  
  
   
  
  
  
Continuo...  
继续。。  
  
This series is about creating exe packers for Windows 32-bit files.  
这个系列是关于对 Windows 32 位程序进行加壳的程序的制作。  
  
Last installment I mentioned some of the big-picture aspects of creating an exe packer. In this installment I am going to talk about a particular part of the packer, the  
 decompression stub. This is the simpler part. In the next installment(s) I'll talk about the packer application itself.   
Again, this isn't going to be source for a packer, but I might do a straightforward one and publish it as an addendum to this  
 series if folks are interested in having some working source as a starting point.  
上一部分我提到了关于制作加壳程序主要方面。这一部分我将讨论加壳程序的一个具体部分：解压程序段。这个是比较简单的部分。  
在下面几部分里面我会讨论加壳程序主体的制作。同样在附录部分我会直接给你一个程序假如你想将这个作为你自己程序的加壳器，或者做一些改进。  
  
Intro  
介绍  
The decompression stub has several responsibilities:  
解压程序段必须具备以下功能：  
  
\* Find the packed data  
定位已加壳的数据  
\* Restore data contents  
恢复数据内容  
\* Perform relocation fixups  
对偏移的修正  
\* Resolve all imports since the Windows loader couldn't do it  
决定所有输入，因为 Windows 载入器无法使用。  
\* Perform thread local storage duties since the Windows loader couldn't do it  
充当线程局部存储器（thread local storage）的功能，因为 Windows 载入器无法使用。  
\* Boink over to the original program  
要指向原程序  
\* You may also have to handle being reentered if you are packing a dll  
假如给 DLLs 加壳还需要重新加入句柄。  
  
Oh, and it also has to run. So lets start with that...  
而且它需要运行，所以让我们从这里开始吧。。  
  
  
A Stub That Runs  
可以运行的程序段  
  
It's useful to remember that your decompression stub is actually a parasite onto a program that was never expecting for it to be  
 there. As such, you should try to minimize your impact on the runtime environment in your packer. I had mentioned before that you   
could make a packer in Logo or Object-Oriented COBOL, and that really was only partially true. You can make the packer application  
 that way fer sure -- and you might even  
 be able to make the unpacker that way sometimes -- but you will really be much happier with C/C++/ASM for the stub part. I personally  
 like C++. Anyway, it will be smaller. If you don't care about the size, still using stuff like Delphi or VB for the stub would be  
 problematic because it hoists in subtle stuff like TLS and runtimes, and they don't have primitives needed to thunk over to the   
original program. Plus it can hose COM stuff that the original app isn't expecting. So let's assume the unpacker will be in the  
 lower-level languages I spoke of and take solace that this is pretty straightforward code, and that the packer still can be in whatever.  
你必须牢记你的解压程序对于原程序来说就像寄生虫一样，它原本不该在那里，所以，你要将它对程序运行时间的影响降至最低。我曾提到过  
你可以用 Logo 或者面向对象的 COBOL，那只是相对正确的。你当然可以那样做你的加壳程序--而且你甚至可以在同样条件下做出解壳程序--但  
是，假如你用 C/C++/ASM 来编写你的程序段会更令人高兴。我个人喜欢 C++，它做的会比较小。假如你不介意程序大小，仍旧使用 Delphi 或 VB  
可能会有一些问题，因为它在敏感信息（像线程局部存储器 thread local storage）和运行时间处理上不是很好，而且无法跳至原程序。加上  
它可能会使原程序丢失一些 COM 信息。因此用我提到的低级语言来编写解壳程序，这样就可以得到非常简洁的代码，而且无论怎样加壳程序都是可用的。  
  
Since the stub is a parasite, and since it will have to be located in a spot at the original application's convenience, we will have to  
 be relocating it dynamically in the packer application. To help with this we will make the stub with relocation records. These are   
usually used for dlls when they can't be loaded at their preferred address. We will make use of them when binding the stub to the   
original application.  
因为你的程序段就像寄生虫，而且它的插入会给原程序带来不便。我们必须在加壳程序里动态对其进行重新定位。因此我们会使该程序  
段包含偏移信息。这个原本是用于 DLLs 的，当一个 DLL 不能被载入适当的地址。我们会使用这一技术来将程序段绑定到原程序。  
  
If you're an avid ASM coder, many things are more straightforward since you can take care to produce position-independent code.  
 This won't necessarily free you from all relocation concerns, however. The decompression library of choice may well not be  
 position independent. Also, and references to global data will need to be fixed up.  
假如你想用汇编，很多事情看来可能更直接，因为你要侧重编写自由定位的代码。你仍旧需要关心所有的偏移，解压信息库的选择最好  
不是自由定位的，还有公用数据的地址也需要修正。 

 4dd466d0e1e0cc123afbe504da2462ca