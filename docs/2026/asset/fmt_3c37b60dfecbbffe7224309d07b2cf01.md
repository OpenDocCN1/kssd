
在做这些尝试时，对传入已修改`wRecordType`的伪造`RR_Record`对象，某些检查会阻止`RR_Free`的调用；

# 6）任意地址读

其实我们通过覆盖`DNS_Timout`对象的`pFreeFunction`指针，已经拥有任意代码执行的能力了！

在`dns!Timeout_CleanDelayedFreeList`函数中，会依次调用`CoolingDelayedFreeList`列表中每一个`Tiemout`对象的`pFreeFunction`指向的函数地址；

`CoolingDelayedFreeList`列表中的保存着即将被释放的`DNS_Timeout`对象；

幸运的是，`Tiemout`对象中包含一个可以传给`pFreeFunction`参数的字段；

下图是`dns!Timeout_CleanDelayedFreeList`函数伪代码：

![图片描述](upload/attach/202104/749942_2SD7NNK4TFQ5FJU.png)

我们可以等`Timeout`对象申请后通过覆盖`Timtout`对象的这些字段，来触发漏洞；

新版本的`dns.exe`编译时带了CFG，目前公开可以绕过CFG的方法：通过覆盖栈上的返回地址，然后执行ROP代码；

虽然目前我们没有找到稳定的方法来将数据写到栈上；但是可以在`dns.exe`中找一个对CFG有效的函数地址作为我们的读写能力；

`dns!NsecDnsRecordConvert`这个函数比较核实，它只有输入一个参数；

下图为`NsecDnsRecordConvert`函数可以接收的参数结构：

![图片描述](upload/attach/202104/749942_RY74M9D875WS46G.png)

在函数内申请了一块内存，并且调用了`Dns_StringCopy`函数；这就是我们的任意地址读Primitive；

因为我们可以控制传入的函数参数和参数内容，我们可以将`pDnsString`指向我们想要读的地址；

在`DNS_StringCopy`函数内会申请一段内存，并将`pDnsString`指向的内存数据拷贝进去

下图为`NsecDnsRecordConvert`函数伪代码：

![图片描述](upload/attach/202104/749942_UAEK6SXBSS8KMY3.png)

因为我们可以控制`wSize`，我们可以控制上图中`Rpc_AllocateRecord`申请的内存大小；

我们可以控制大小，让它申请内存时，申请到我们布置好的堆坑上；等内存拷贝完，我们通过内存泄露来获取读取的数据；

![图片描述](upload/attach/202104/749942_RMEXE9WYDEZZURS.png)

我们读取的地址应该在`dns.exe`范围内，并且数据中包含一个`msvcrt.dll`的地址；

这里选择的是`dns!_imp_exit` 该地址范围内包含一个`msvcrt!exit`的函数地址，通过泄露该地址，我们就可以绕过`msvcrt`的ASLR；

拿到`msvcrt`的基址后，加固定偏移，就可以计算出`msvcrt!system`函数的地址；

需要注意的是：`Dns_StringCopy`希望拷贝源是一个NULL结尾的字符串；如果地址的最低有效字节是0x00，则计算出的字符串大小是1，并且该地址不会被复制；

在案例中的`msvcrt`文件，这个问题并不会影响，但是作者并没有测试所有版本的`msvcrt`；

# 7）远程代码执行

一切就绪，我们可以再次触发申请一个`Timeout`对象，然后覆盖`pFreeFunction`字段为`msvcrt!system`函数地址，以及`pFreeFuncParam`为一个堆内存地址，

该地址上保存着要执行的payload 命令；

为了获取反弹的shell，最简单的办法是使用`mshta.exe`来执行一个远程HTA文件。 不过这个exp还有是存在更多扩展性和利用思路；

english version：4b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4M7X3q4H3L8s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3M7r3!0K6N6q4)9J5c8X3q4F1j5i4c8G2L8i4W2Q4x3X3c8G2k6W2)9J5k6r3q4F1i4K6u0V1k6i4S2H3L8r3!0A6N6q4)9J5k6s2u0U0k6g2)9J5k6s2N6A6N6r3S2Q4x3X3c8U0N6X3g2Q4x3X3b7J5x3o6t1H3i4K6u0V1x3e0x3#2x3q4)9J5k6s2y4A6k6%4u0W2k6l9`.`.

# 1）漏洞介绍

`CVE-2020-1350`是`DNS.exe`在处理畸形DNS Sig消息时，由于对数据包的字段校验不严格，导致了整型溢出。

DNS SIG 消息中包含对DNS记录集合的数字签名，DNS记录集合就是一种名字相同，或者类型相同的DNS消息集合；

攻击者可以配置一个恶意的域名`TestDnsRce.com`，该域名的DNS指向的DNS服务器作为本次攻击的目标；

1. 当客户端查询`evildomain.com`的DNS记录；
2. 目标DNS服务器向根域名服务器查询`evildomain.com`的NS记录；
3. 根域名服务器告诉目标DNS，`evildomain.com`的权威DNS服务器是3.3.3.3，并该记录缓存起来；
4. 客户端向目标服务器查询`evildomain.com`的Sig记录；
5. 目标服务器将请求转发给权威DNS服务器;
6. 权威DNS服务器返回畸形Sig查询结果;

目标DNS服务器处理`SigQuery`的畸形消息时触发漏洞第一现场；

参考下图：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_92KMKYWSDU57JRU.png)

漏洞在`dns!SigWireRead`函数中，该函数用来缓存从另一台DNS服务器返回的DNS Sig记录

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_3CTKMDFXVSZNE9V.png)

根据上图漏洞函数中的第11行，`RR_AllocateEx`传入了一个16位无符号整型，经过计算后作为申请内存的大小参数。

假如传入的Sig查询记录消息包，经过计算后使大小超出0xFFFF，就会触发整型溢出漏洞；

# 2）触发漏洞

由于整个消息包大小是有限制的，不能通过类似响应一个数据很大Signature的消息包来触发该漏洞；

也不能通过UDP来触发该漏洞，因为DNS消息通过UDP传输的大小也是有限制的，上限为512或4096字节，这取决于目标服务器是否支持DNS扩展机制（EDNSO）；

如果使用DNS截断，就会先发送一个UDP请求，如果响应消息被截断，`TC`字段被设置，就会尝试用TCP协议重新请求，TCP协议传输DNS消息最大限制是64KB(0xFFFF)，仍然不够触发漏洞；

## 2.1）DNS名称压缩

在使用TCP协议传输时，通过修改DNS消息中被压缩的名称，可以导致`SigName.Length`变大， 而且不改变整个DNS消息包的大小；

下图为DNS 消息中使用名称压缩的数据包：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_937M9JX38JKNTMW.png)

红色框 - DNS消息开始至偏移0xC的数据

绿色框 - Flag标记，0xC0 代表DNS名称位置相对偏移在DNS消息中；

蓝色框 - DNS名称的偏移，0x0C，表示从DNS消息头开始，偏移0x0C字节才是DNS名称的开始位置；

黄色框 - 简单编码后的DNS名称

DNS名称被使用以下方式进行编码，域名中的每个“点”作为分隔符，替换成了每个点后面的字符个数，并以NULL结尾；

例如：`e63K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3N6G2L8$3N6D9k6g2)9J5k6h3y4G2L8b7`.`.

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_3D2398HX22FYBJ5.png)

编码后：`3www6google3com`

在上面捕获的DNS流量数据包中，因为请求的域名是以 “9.xxx"开头的，所以偏移指向0x1；

因此，如果我们将偏移0xC改成0xD，域名开始的偏移就会指向0x39，表示域名的下一部分有0x39个字节；从而导致下一部分名称数据会扩展到数据包中的签名部分；

如之前在`dns!SigWireRead`函数中所见，传入`RR_AllocateEx`参数的大小是经过计算的， （`signatureLength + SIgName.Length +0x14`）；

DNS名称最大长度为0xFF字节，从伪造的DNS名称中多出来的大小足以触发整型溢出漏洞，并且整包数据大小不会超过65KB的限制；

# 3）漏洞利用

该漏洞利用最耗时的是了解如何正确操作堆布局的方法；下面部分重点介绍如何利用堆布局避免程序Crash并控制堆内存的释放和分配；

## 3.1）WinDNS堆管理器

首先需要了解一下WinDNS服务是如何管理堆内存的；

WinDNS服务有它自己的堆内存池，如果申请的Buffer大小超出0xA0字节，WinDNS将使用Windows自带堆管理器（`HeapAlloc`）；

否则，WinDNS使用自己的内存池，按照申请的内存大小分类，分别是：0x50，0x68，0x88，0xA0； 每种大小，使用单项链表关联起来；

如果链表中没有可用的WinDNS内存，则使用Windows原生堆来申请一个大内存块（memory chunk）；然后按照大小分别划分给WinDNS不同的单项链表中；

例如，要分配的内存大小是：0x50，0x68， 0x88，0xA0； 则对应的内存块大小是：0xFF0, 0xFD8, 0xFF0, 0xFA0,

每个内存块可分配的WinDNS内存数量：

0xFF0/0x50 = 0x33

0xFD8/0x68 = 0x27

0xFF0/0x88 = 0x1E

0xFA0/0xA0= 0x19

下图为`dns!Mem_Alloc`伪代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_FAGZGEXAST5UWUZ.png)

当WinDNS的一块内存被释放时，并不会直接释放内存，而是将内存块再次添加对大小的链表中；减少实际内存分配消耗；

WinDNS内存的分配和释放遵循最基本的Last In First Out （LIFO）规则， 最后一个释放的，将会再下一次申请时被使用；

下图为`dns!Mem_Free`伪代码：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_TVKS4CV39GF5EKM.png)

## 3.2）WinDNS Buffer 结构

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_BF3B3RQUCRAXQXY.png)

了解WinDNS内存管理机制，让我们在后面构造堆布局时更方便；

## 3.3）如何避免堆拷贝溢出时程序Crash

编写POC时，遇到的第一个问题就是通过`memcpy`拷贝Signature数据到申请的堆内存时，发生了访问违规；

我们必须报证拷贝溢出部分的内存被拷贝到有效的内存地址上；

在观察堆布局时发现，Windows把我们使用原始堆申请的WinDNS内存块，放在名为"Internal"的堆Segments上；

Internal Heap Sigments的大小为：0x41fd0-0x41ff0，经过观察，这些Heap Segments，只会被WinDNS用来申请内存块；

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_TWTA4NNDYN6MYCF.png)

如果我们能报证被覆盖的内存大小低于0xA0，就能报证被覆盖的内存块一定是Heap Segments这里的其中一个内存块；

## 3.4）构造堆坑

如果我们可以释放这些连续Heap Segments中的一块内存，然后重新申请到刚刚被释放的内存，然后再进行溢出我们可以使溢出的数据覆盖到有效的内存地址，这也是最常用的堆溢出利用技术；

在这种场景下，我们可以让客户端发送一个Sig请求给DNS服务器，服务器可以在返回的数据包中设置短`TTL`，来控制内存释放；

同样的，如果我们想让一块内存长时间存在，则可以修改为长`TTL`； 每两分钟就会释放一次`TTL`过期的内存

`TTL`（Time To Live）通常用秒来表示，用来表示DNS查询缓存记录的有效时间，不可能一直有效，当过期后，就会丢弃缓存数据，并向上层权威名称服务器获取最新数据；

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_UKZZ5BW79TRKB7C.png)

构造堆坑的经典步骤如下：

1. 使VictimDNS服务器向EvilDNS服务器发送大量subdomain查询请求；
2. VictimDNS服务器得到EvilDNS服务器响应后，会将数据缓存在堆内存中（Heap Spray）;
3. EvilDNS服务器响应subdomain查询请求时，设置一个为短`TTL`，剩余全部为长`TTL`；
4. 等待被设置短`TTL`的DNS响应包内存被释放，WinDNS每两分钟释放一次过期的数据包；
5. 再次请求subdomain查询，EvilDNS服务器会响应一个畸形DNS包，触发整型溢出；
6. 因为内存是LIFO分配规则，上一步请求的DNS缓存记录，将会使用第4步释放的堆坑内存；

下图为使用堆喷避免crash

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_YDRUZJZCS7Q6SZ7.png)

现在可以稳定避免在拷贝数据时发生Crash了，但由于覆盖了堆上一些其它对象导致不定时Crash；

下图为 覆盖了`CacheTreeNode`后发生Crash

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_ABJXAXD6923R6RB.png)

通过在Windbg中观察什么到底是什么类型内存，挨着被我们覆盖的内存；

下图为 在Windbg中跟踪内存申请

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_DMPTW6JQDMR5X46.png)

可以看到在我们溢出的Buffer附近，两块新的WinDNS内存块被申请了，大小分别是0xFF0和0xFA0，（后者是由于堆喷，记录大小是0xA0导致的）

那0xFF0 大小的内存块是干啥的？这个内存块的划分大小是0x88，用来保存DNS记录缓存的二进制Tree对象；被我们覆盖的正是缓存树对象；

当遍历树时就会发生Crash；

此时思路已经很清楚了，还记得被覆盖的内存是在Heap Segments中的，该内存只能被WinDNS用来管理内存块；

也就是被覆盖的对象大小<= 0xA0 , 并且是WinDNS内存块管理的可用大小0x55、0x68、0x88、0xA0 这些大小；

我们知道WinDNS管理的内存被释放后，不会还给Windows，而是添加到相应大小的链表中；

我们可以强制分配大量大小为0x88的内存，并释放这些内存，一旦释放后，这些内存就会被添加到WinDNS的`FreeList`链表；

避免从Windows中分配新的堆块，然后向WinDNS中申请大量不会释放的内存；确保我们覆盖的内存位于新的HeapSegments中

这样就不会覆盖到堆中一些重要对象；

下图为 堆梳理，避免溢出时覆盖重要的对象

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_YSVVA8R3CSG64BW.png)

# 4）覆盖堆上的对象

通过前面的布局，我们已经可以为我们溢出对象在堆上构造一个坑了，通过这个坑，我们可以覆盖被我们用来堆喷的DNS缓存记录对象；

## 4.1）了解内存环境

因为我们使用堆喷申请了很多内存块，这些内存被连续添加`FreeList`中，代表它们以连续的顺序进行分配。

所以我们进行DNS SIG记录查询的顺序，就是它们在堆上出现的顺序，我们必须知道溢出时，我们会覆盖哪些内存；

下图为 使用一个伪造的`Record`对象覆盖内存

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_3ZQADT2EQDHDXZX.png)

## 4.2）`RR_Record` 结构

首先来看看我们用来堆喷的对象结构，缓存的WinDNS `Record`：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/749942_6ZE6EQE2NAFC2JT.png)

了解这些结构和`WINDNS_BUFF`结构，将会更方便构造`RR_Record`对象；

## 4.3）控制内存释放

前面，我们通过控制`Record`对象的`TTL`字段，等待`TTL`时间到期后，每过2分钟就会触发一轮释放；

每次都要等2分钟，不是我们没有耐心，而是这会影响到我们控制重新申请内存；如果可以立即释放内存就好了;

当一个`RR_Record`对象从缓存中响应给对应的NS查询时，首先会检查`dwTTL` 和 `dwTimeStamp`字段是否已经过期；

因为每2分钟才清理一次过期的缓存记录对象，可能会发生要查询的缓存记录对象已经过期了的场景；

我们可以在伪造一个`RR_Record`对象时，将`dwTTL`字段和`dwTimeStamp`字段设置为0，然后查询对应的subdomain就会导致这块伪造的对象内存被立即释放；

## 4.4）控制内存申请

掌握了前面的控制WinDNS内存释放， 现在控制内存申请其实会方便了，因为WinDNS内存申请基于LIFO；

一旦我们释放了一块内存，我们下一次申请同样大小的内存时，一定是我们前面释放的那块内存；

因为我们同样可以控制`WINDNS_BUFF`结构，我们可以伪造原始Buffer大小，这样我们就可以控制WinDNS从内存块中返回给我们内存的大小了；
