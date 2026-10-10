

# FLEXlm9.2 的破解教学三

> 原文：[`www.pediy.com/kssd/pediy06/pediy6932.htm`](https://www.pediy.com/kssd/pediy06/pediy6932.htm)

翻译第三篇 On Software Reverse Engineering - 3  
            Reverse Engineering, FLEXlm, IMSL  
  
揭示加密钥和加密种子  
  
   从实际的观点出发我们现在可以止步不前了，因为有了许可的拷贝我们可以自由的使用目标程序了。但是，软件破解的最终目的是反向（逆向）工程的所有相关算法并重新创建它们，那就好像我们是原始作者一样。在我们的实例中，最低限度的要求是揭示 VNI 的 5 个 vendor 密钥和 3 个随机种子。不用说，这要求我们更仔细的阅读 FLEXLM SDK 源代码，因此我们简单的回顾一下它是怎样组织的。下面的列表显示了包含最关键文件的文件夹。  
  
src        lmgr.lib 或 lmgr9a.dll 的源文件  
app        lmgras.lib 的源文件  
server     lmgrs.lib 的源文件  
master     lmgrd.exe 的源文件  
utils      实用工具的源文件  
machind    机器独立文件(源 & 头)  
i86\_n3     对于 x86 平台的最终的二进制文件  
h          头文件  
certicom   来自 Certicom 公司的库和头文件  
ulite      FLEXLM 的 ultralite 版本  
    
    此外，文件和函数根据其角色被指定了不同的前缀。例如，目录 app, server 和 master 以 ls\_ 开始，而在 utils 中是以 lm 为前缀。  
  
l\_     许可证, 用于内部函数  
lc\_    许可证客户端, 用于客户端 API  
ls\_    许可证服务器, 用于服务器 API  
lm\_    许可证管理器, 用于实用工具和一般的程序（原为材料）  
  
    现在，我们回到文献\[2\]和\[3\]中关于 Vendor 怎样实用 FLEXLM SDK 的指导中。基本地，Vendor 需要将许多 FLEXLM 二进制与其自己的产品结合并创建若干实用工具－一些作为内部使用，一些用于最终用户发布。在这些 Vendor－所产生的文件之间有很亲密的联系，下面是其关系表：  
  
文件                              由...产生                              #include  
lmrand1.exe                       lmrand1.c + lmgr.lib  
lmcode.c, lsrvend.c               lmrand1.exe + lsvendor.c               lm\_code.h  
lmappfil.c, lmkeyfil.c            lmrand1.exe  
lmnewgen.exe                      lmnewgen.c + lmcode.c + lmgr.lib       lm\_code.h  
lm\_new.c                          lmnewgen.exe                           seeds & pubkey  
lmseeds.h                         lmnewgen.exe  
lmprikey.h, lmpubkey.h            lmnewgen.exe  
lmcrypt.exe                       lmcrypt.c + lmgr.lib                   lm\_code.h,  
                                                                         lmseeds.h,  
                                                                         lmprikey.h  
makekey.exe                       makekey.c + lmgr.lib                   lm\_code.h,  
                                                                         lmseeds.h,   
                                                                         lmprikey.h  
vendor application               vendor code + lm\_new.obj + lmgr.lib +   seeds & pubkey  
  
                                 libsb.lib + libcrvs.lib  
vendor daemon                    lsvendor.obj + lm\_new.obj + lmgr.lib +  lm\_code.h,   
                                                                         lsserver.h,   
                                 lmgras.lib + lmgrs.lib + libsb.lib +   seeds & pubkey  
                                 libcrvs.lib  
  
    注意，并非上述的所有文件都对我们是重要的。lmappfil.c 和 lmkeyfil.c 是 Vendor 确定的滤波器，它们是附加的安全测试但通常未用。类似的，lmprikey.h 和 lmpubkey.h 仅用于 CRO 密钥，在我们的例子中可以忽略。通过编辑 lsrvend.c 或 lsvendor.c，Vendor 可定制后台程序，但很少有人那样做。  
有一件值得讨论的事情是 Certicom 产品，也即 libsb.lib 和 libcrvs.lib 。它们几乎涵盖了从 SHA 到 DSA 的整个加密领域，而且 FLEXLM 采用的 API 几乎为 ECC（椭圆曲线加密技术）和 RGN（随机数字产生）。ECC 用于在 CRO 中的公开/专用密钥对，但在我们的目标程序中关闭了；而 RGN 用于 Vendor 种子的转换，我们不可忽略。  
正如先前声明的，Vendor 选择了 3 个随机种子（LM\_SEED1, LM\_SEED2, LM\_SEED3），但并不直接使用它们。实际上，基于前 3 个，lmnewgen.c!main()®l\_prikey.c!l\_genrand()®libsb.lib!sb\_rngFIPS186Session()产生了 4 个新种子\[8\]（ENCRYPTION\_SEED1, … ENCRYPTION\_SEED4）输出到 lmseeds.h。种子 3 和 4 专门用于公开/专用密钥对，因此我们仅需要考虑种子 1 和 2。  
    尽管 FLEXLM 在 l\_rand.h 和 lm\_rand3.c 中有其自己的 RGN，但它在此仍然使用了在 DSS 标准中确定的算法(c.f. \[8\])。该算法是决定性的（没有读取时间、寄存器值等），因此对给定的 LM\_SEED，那么 ENCRYTION\_SEED（的值）将是固定的。它也是一个单向的函数，这意味着在实践中不可能从已知的 ENCRYPTION\_SEED 来解决 LM\_SEED。因为是 ENCRYPTION\_SEED 被编程到 Vendor 发布版本中，而 LM\_SEED 除了在 l\_genrand()中出现外别处都不出现，因此我们最好恢复前者。但是，恰巧出于相同的原因，我们假定恢复 LM\_SEED 是同样的好。  
    在 FLEXLM 的内部基本上是 C/S 模型，结合 Vendor 应用程序的是客户端，Vendor 后台程序（这里是 vni.exe ）是服务器。注意，Vendor 后台程序不同于 FLEXLM 许可证管理器 lmgrd.exe，后者仅重定向客户端请求到相应的做实际工作的 Vendor 后台程序。令一个因素是许可证文件格式。FLEXLM 提供了丰富的许可选项，在计数与不计数上是很重要的。不计数的许可证对于（用户）数量的检查没有限制并且不需要 Vendor 后台程序－其有效性仅在 Vendor 软件同时作为客户端和服务器时发生。计数的许可证通常用于网络（浮点许可证必须被计数），在大公司通常可以看到这样的情景：中心节点运行服务器，所有工作站连接到它上面进行许可证的输入输出检查。因此，我们可以推断 Vendor 后台程序的主要任务是计数、管理和调节受限许可证的使用。  
    看我们的许可证，它是 “permanent uncounted HOSTID=ANY”, 它表明没有任何条件限制。这一定是每个破解者的梦想（无后台程序处理也使追踪更容易）。实际上在许可证文件中，“SERVER"和"DAEMON"行是不必要的，两个“FEATURE"足以保证 cmath.exe 能无限制的运行。后面我们将有主题讨论许可证类型。  
现在，我们已有了具有 Vendor 密钥和加密种子的 cmath.exe 作为客户段和服务器。有个好消息是 CRO 不起作用，因此仅要一套密钥。如果我们挖掘出它们，我们将可以用 lmcrypt.exe 或 makekey.exe 制作任何许可证文件，与 Visual Numerics 的方式一样。在我们着手进行时，我们总结了在密钥文件中的一些密钥。  
  
lm\_code.h: /\* \\machind\\lm\_code.h and \\h\\lm\_code.h must be identical \*/  
VENDOR\_KEY1, VENDOR\_KEY2, VENDOR\_KEY3, VENDOR\_KEY4, VENDOR\_KEY5  
LM\_SEED1, LM\_SEED2, LM\_SEED3  
lmcode.c:  
#include "lm\_code.h"  
#include "lmclient.h"  
VENDORCODE vendorkeys\[\] = { /\* 错误的名字，需使用 vendorcodes\[\]代替\*/  
{ VENDORCODE\_7,  
ENCRYPTION\_SEED1 ^ VENDOR\_KEY5, ENCRYPTION\_SEED2 ^ VENDOR\_KEY5,  
VENDOR\_KEY1, VENDOR\_KEY2, VENDOR\_KEY3, VENDOR\_KEY4,  
FLEXLM\_VERSION, FLEXLM\_REVISION, FLEXLM\_PATCH,  
LM\_BEHAVIOR\_CURRENT, {CRO\_KEY1, CRO\_KEY2},  
LM\_STRENGTH, LM\_SIGN\_LEVEL, 0  
},  
};  
lmseeds.h:   
ENCRYPTION\_SEED1, ENCRYPTION\_SEED2, ENCRYPTION\_SEED3, ENCRYPTION\_SEED4  
lm\_new.c:   
x = 0x3d73db2e; /\* 在 lmnewgen.c 中产生的随机数 key5\_uniqx \*/  
VENDORCODE.data /\* 由 key5()模糊 \*/  
VENDORCODE.keys /\* 由 l\_xorname()模糊 \*/  
l\_getattr.c:  
#define VENDORMAGIC\_V7 0x08BC0EF8 /\* 不重要, 仅用于 xor（与或）取消\*/  
lmclient.h:  
#define L\_NEW\_JOB l\_n36\_buf  
l\_privat.h:  
#define L\_UNIQ\_KEY5\_FUNC l\_n36\_buff  
#define L\_SET\_KEY5\_FUNC l\_x77\_buf  
#define SEEDS\_XOR mem\_ptr2\_bytes /\* 这里 seeds 隐藏在 job \*/  
#define SEEDS\_XOR\_NUM 12  
   
    此外，这里有三个数字被嵌入到二进制中（比如 VENDORMAGIC\_V7），但好像从来没有使用（至少在 cmath.exe）中。也许它们仅用于某个版本或某个许可证类型，当我们访问这些函数时，我们将会看到它们。  
Besides, there are three numbers that are hard-coded into the binaries (just like VENDORMAGIC\_V7) but seems never used (at least in the case of cmath.exe). Maybe they are  solely  for certain versions or license types, we’ll see that when we visit those functions.  
   
lm\_ckout.c!l\_sg():  
x = 0x6f7330b8; /\* v8.x \*/  
lmnewgen.c!VKEY5():  
x = 0x6f7330b8; /\* v8.x \*/  
l\_key.c!l\_zinit():  
z = crokey\_flag ? 0x62586954 : 0x72346B53; /\* v9.x \*/

 4dd466d0e1e0cc123afbe504da2462ca