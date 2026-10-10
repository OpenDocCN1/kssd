

# 逆向 Microsoft.Windows.Server.2003.x64.Edition.VOL.Keymaker.Only-ZWT

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-66.htm`](https://www.pediy.com/kssd/pediy09/pediy09-66.htm)

> 【原创】逆向 Microsoft.Windows.Server.2003.x64.Edition.VOL.FIXED.Keymaker.Only-ZWT  
>   
> 一直以来对 Windows 的注册算法很感兴趣，谁叫它那么有名呢，呵呵。不过并不是一开始就选  
> 择 ZWT 的这个 keygen 作为逆向的目标的。我最早见到的 windows keygen 是由 YAG 在 2004 年发布的  
> Microsoft.Windows.XP.2003.Enterprise.Server.Keygen-YAG，在该包中包含有 2 个 keygen 和  
> 一份文档。这两个 keygen 在网上流行很广，不过由于它是基于 OpenSSL 的大数库开发，而我又  
> 对那个库不熟，所以一直也没什么想法。倒是那份文档，让我对 windows 的注册算法有了初步  
> 认识，使我总想自己动手实现这套算法。后来见到 ZWT 的这个 keygen：  
> Microsoft.Windows.Server.2003.x64.Edition.VOL.FIXED.Keymaker.Only-ZWT。拿它动手有  
> 三个原因：它没有加壳，它是用 Visual C++开发的，它使用的是 miracl 大数库。  
>   
> 写作本文的主要目的，其实是为了介绍实现算法的过程，算法才是重点，而不是逆向。为什么  
> 呢？因为我觉得仅仅通过逆向一个 keygen 而找到算法的正确实现太难了，我就深受折磨。只有  
> 结合相关文档资料理解算法，再通过逆向和动态跟踪调试来验证才能正确实现算法。所以逆向  
> 在我所做的工作中其实只是小部分，含有算法的学习和理解，以及动态跟踪和调试。  
>   
> 所有的这些工作都是去年暑假完成的，现在才有时间给整理出来。  
>   
> 1、关于 Windows 注册和验证算法  
> 请参考以下三篇文档：  
> MSKey4in1 Read Me.doc  
> Microsoft 的 25 位 CDKey 里有什么.TXT  
> 恢复原版 XP.TXT  
> 网上有下载，附件中也有。  
> 文档中算法描述基本都是对的，不过也有一些很明显的错误，需要自己多注意。  
>   
> 2、逆向 ZWT 的 keygen  
> IDA 加载，载入 miracl.sig。  
> IDA 基本不能识别 miracl 的 API 调用，但在 string 窗口中可以看到很多熟悉的字符串，可断定  
> 确实使用了 miracl 库。推测是采用 c++ wrapper 的方式调用 miracl 库函数。  
> 采用 BlackEye 提出的方法手动识别主要的 miracl 函数。  
> 由于附件中提供了源码，这里不再给出反汇编分析的结果。  
> 对照文档和汇编代码可整理出算法的流程，并写出主要的 C++代码。  
>   
> 3、实现 keyGenerator 和 KeyVerifier  
> 在逆向的基础上进一步细化，可得到我们自己的 keyGenerator。  
> 为了验证所逆向的 keygen 的正确性，同时还写了一个 KeyVerifier，以实现验证算法。  
> 从文档中就可以看出来，ECC 算法的验证算法比注册（签名）算法更容易实现。因而开发一个  
> KeyVerifier 既是必要的，也是可行的。  
>   
> 附件中的代码使用 c++ wrapper 的方式调用 miracl 库函数。用 c++ wrapper 方式能够发挥 c++语  
> 言的优点，使得代码更加精炼，也更不容易出错。同时也方便将自己写的 keyGenerator 的反  
> 汇编代码与逆向分析得到的结果进行对比分析。  
>   
> 调试注意事项：  
> 求解二次方程所用到的函数 sqroot()或 sqrt()，其失败的几率是相当高的，必须要对其返回的  
> 结果进行分析。这个地方让我多花了一整天的时间。  
>   
> 附件中包含了本文所用到的文档和相关的源代码。  
> 所有资料均来自于 Internet，请勿用于非法用途，否则后果自负！  
>   
>   
> 在逆向 ZWT 的 keygen 的过程中，也顺便找到其中 3 处错误：  
>   
> .text:0040285D                 call    \_rand  
> .text:00402862                 cdq  
> .text:00402863                 mov     ecx, 400  
> .text:00402868                 idiv    ecx             ; rand()%400  
> .text:0040286A                 push    0  
>   
> 错误 1：调用伪随机数生成函数 rand()之前没有调用 srand()初始化随机数种子。以致每次运  
> 行 keygen 时 rand()返回相同的随机数序列：0x29,0x4823,...  
> 错误 2：这里对随机数取 400 的模。但我在别的地方看到的是取 0x400 的模。  
> 错误 3：每生成一个 key 都调用一次 irand((unsigned)time(NULL))，使得随机数与时间紧密相  
> 关，随机性很差。  
>   
> 这些错误在附件代码中已经得到更正。  
>   
> 附件中的代码也好像有问题，就是用 ZWT 的 keygen 生成的部分 key 可能通不过验证，我调试了  
> 半天也没有找到原因，只好暂时先搁置,看哪天灵感来了再搞定它。 

 4dd466d0e1e0cc123afbe504da2462ca