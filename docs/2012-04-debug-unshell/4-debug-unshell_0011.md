
# 如何用程序判定一个 PE 文件是否加壳

> 原文：[`www.pediy.com/kssd/pediy11/115515.html`](https://www.pediy.com/kssd/pediy11/115515.html)

由于工作原因，时间比较少上论坛来泡泡了。最近由于某些原因时间变得稍微宽裕了些，有空整理整理这几年的一些资料和文档。

感觉这个方向论坛上讨论得并不多，是这个方向没有什么使用价值呢，还是说太简单了大家都不愿意研究？总而言之，我个人感觉这个方向还是一个挺有趣的，所以发上来共享一下。

这篇文章涉及的东西，虽然不是很难，但可能并不适合于新手。尽管我已经尽力去解释一些东西，但由于水平有限难免会存在一些错误。还请各位大侠不吝赐教啊！

下面是 doc 和代码的打包下载

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || judgepack.zip |

## 前言

开始切入正题前，让我先解释一下这篇文章的结构。

一个很平凡的题目（命名的确是很头疼的问题，以此命名的原因是希望能在搜索引擎提高命中率：D），但却不好回答。

我在 07 年底遇到了到这样的需求：设计一个函数判定 PE 文件是否加壳，加了什么壳。这个需求的必要性不言自明。

当我拿到需求并且与需求方确认后将需求分解为以下 2 个层次：

1. 初级需求是函数的返回值是 BOOL 型，返回 True 或者 False 表示是否加壳。
2. 高级需求是函数的返回值必须能返回表示壳的种类，返回壳 ID 或者壳名称。

关于手动脱壳的分析员来说判断一个 PE 文件是否加壳方法太多，而且通常非常容易。但是如果将这些方法一一列举出来后，我发现用程序来模拟这些人工的动作有 3 个问题：

1. 大部分方法，实现起来比较复杂。
2. 有些方法误判率非常高。
3. 大部分方法，判定的壳种类非常有限。

所以本文的正文部分结构上分成二个部分，分别尝试去回答上面的两个问题。第一章解决第 1 个小问题，其中又分成两个小节使用两种技术来分别实现；第二章解决第 2 个小问题。希望你能按照顺序阅读全文。

在开始旅行前，有一些共识需要强调一下。Pack 的英文直译过来应该是“被压缩”的意思。而中文里“壳”的意思更多的是强调“被保护”（“壳”的历史不是这篇文章的重点）这里要说明的是，尽管按照中文的习惯，我更多的使用“加壳”这个词语，但本文试图解决的问题 -- 检测一个 PE 文件是否“被压缩”，而不是是否“被保护”。例如：一些自解压的 WinRar 程序也会被判定为加壳。

## 第一章、你加壳了吗？

### 第一部分、信息熵(Entropy)

引用: In information theory, entropy is a measure of the uncertainty associated with a random variable. The term by itself in this context usually refers to the Shannon entropy, which quantifies, in the sense of an expected value, the information contained in a message, usually in units such as bits. Equivalently, the Shannon entropy is a measure of the average information content one is missing when one does not know the value of the random variable. The concept was introduced by Claude E. Shannon in his 1948 paper "A Mathematical Theory of Communication".

![](img/ce88134dcbc50b641f56eb7b72a9eaed.jpg)

引用自：`http://en.wikipedia.org/wiki/Information_entropy`

一开始是否被上面这恐怖的定义吓到了？按照国内理科类教课书和论文的惯例----将公式推导和定义放在一开始，目的就是吓人。潜台词是接下来内容可信度很高很牛 B。

如果你真的被吓到了，那么我的目的也就达到了。但是我的潜台词是这个理论的确很牛 B,但是我们完全不需要理会它也可以。

“信息熵”是通信领域的一个概念。以我的理解就是在一段的数据中其所携带的信息量。应用在判定 PE 是否加壳时，“信息熵”可以这么理解，计算一个 PE 文件的信息量或者其中某段（Section）的信息量。当这个熵的值超过一定阈值时，则该 PE 或某段被加壳（Packed）。说得更准确（更通俗）些，就是压缩数据往往携带着更多的信息量，PE 文件携带信息量得多的就意味着可能被加壳了。

在 PEiD 中就有这样的功能：

![](img/410c7b2e303d3a9fbc8edb00672c1158.jpg)

(该图片引用自小喂的帖子 `http://bbs.pediy.com/showthread.php?t=39443`)

计算一个节或者一个 PE 文件的熵，可以从论坛中小喂的帖子下载到源代码。

小喂的代码的核心算法就是一个函数（一句话）：

![](img/9e03ea0b29ce21c08f90d0d1b04f975b.jpg)

对于判定是否加壳的 Entropy 阈值定义如下：

* 加壳(Packed): `Entropy > 6.75`
* 可能加壳(Maybe Packed): `6.75 > Entropy > 6.5`
* 无壳(No Packed): `6.5 > Entropy`

![](img/86dc52a4487fb113b7e4887ac84410cb.jpg)

ps.关于这个定义的来源我一直没弄明白。这是 order-0 entropy 的定义如此，还是经验所得？

到这里基本的理论已经介绍完毕了，这章结束了吗？还没有！

当你将这套代码做测试的时候，你会发现它的误判和漏判率非常的高。当我发现这个情况的时候，我尝试使用 PEiD 来对比，结果让我大吃一惊。PEiD 计算的 Entropy 和用小喂代码计算的 Entropy 竟然完全不一样!(貌似计算一个 Section 的 Entropy 是一样的)而且 PEiD 判定的命中率非常之高。

我非常想知道 PEiD 是如何做到的，于是通过 google 我来到 PEiD 的官方论坛。在论坛中有一个帖子引起了我的注意。

![](img/447f5d4034c63f19cbc1cae034b5efe9.jpg)

`http://www.peid.info/forum/viewtopic.php?t=42`

巫术（voodoo）？！啊哈,别开玩笑了。对于客户端程序来说可没有秘密可言！：D

既然人家作者不愿意说，那我只好踏上 PEiD 的逆向分析之路。

很抱歉的是由于逆向是在 07 年时完成的。现在已经找不到自己当时逆向的一些注释或者资料。所以这个部分就略过吧 -- 我想你也不是很感兴趣。

下面进入结论阶段，让我们看看 PEiD 都做了哪些工作。

1. 重新组织需要计算的数据
   i．以下数据不列入计算熵的范围：导出表数据、导入表数据、资源数据、重定向数据。
   ii. 尾部全 0 的数据不列入计算熵的范围。
   iii. PE 头不列入计算熵的范围。
2. 分别计算每一部分数据的熵 `E` 和该部分数据大小 `S`。
3. 以下列公式得到整个 PE 文件的熵 `Entropy = ∑Ei * Si / ∑Si (i = 1,2…n)`。

经过上面的条件限制和方法，我计算出来的 Entropy 与 PEiD 的就基本上一样了。尝试使用 100 个加壳的样本和 100 个没有加壳的样本进行测试得到下面的数据：

1. 当定义 `Entropy > 6.5` 为 Packed 时，误报率是 20%左右；遗漏样本在 10%以下
2. 当定义 `Entropy > 6.75` 为 Packed 时，误报率是 10%左右；遗漏样本在 15%左右

实际应用当中，针对具体的需求我还对代码和阈值做了一些微调。不过很抱歉这部分代码不能共享出来。

### 第二部分、节表的秘密

在小喂的帖子中提到了猜测 Process Explorer 对文件的判定也使用的是 Entropy 的方法，其实不然。这一章就是要揭开 Process Explorer 对文件加壳判定的秘密。

先来看看 Process Explorer 在对加壳与否界面上的展示。

![](img/012a82d8fcdd81b4dd77b74a95a7d9fb.jpg)

Process Explorer 默认用紫色高亮显示被加壳的镜像，从图中我们知道 Totalcmd 被判定是 Packed Images。

Sysinternal 自从"弃明投暗"纳入微软帐下后：D，源代码就不再公开。无奈之下还是只能选择逆向 Process Explorer（版本号 v11.12）这条路。其实逆向完 Process Explorer 后，我发现远没有想象中的复杂。下面将一些逆向的过程展示出来。

1. 判定的函数这里开始，`CreateFile` 打开要判定的文件。

   ![](img/4ade131a5207abdc5dc7b30dd9ae6ce0.jpg)

2. 校验一下 PE 文件头，然后判定是 win32 还是 win64

   ![](img/9c2a9646b82253744b032eb9264b2f71.jpg)

3. 下面进入了核心算法，一共有 10 条规则进行判定镜像是否加壳

   ![](img/8f6fd44721c8291a9ff801cf3cc4cab6.jpg)

4. 主循环结束，通过一个标志位决定了是否上色：）

   ![](img/43c77a4cf07f5a00831ce3f780e650a6.jpg)

我原来打算总结一下这 10 条规则，但是这次我决定偷懒了。还是让代码说明一切吧！

在下载这个文档的同时，你应该能下载我的测试代码（`procexp_judge_pack.cpp`），代码中没有实现对 win64 的 PE 文件的判定，又偷懒了:D。

代码:

```cpp
BOOL _IsPack_32(PBYTE pBuf, UINT uSize)
{
  BOOL bRet = FALSE;

  PIMAGE_DOS_HEADER     pDosHdr  = NULL;
  PIMAGE_NT_HEADERS     pNTHdr   = NULL;
  PIMAGE_FILE_HEADER    pFileHdr = NULL;
  PIMAGE_SECTION_HEADER pSecHdr  = NULL;
  UINT      i        = 0;
  
  pDosHdr  = (PIMAGE_DOS_HEADER)pBuf;
  pNTHdr   = (PIMAGE_NT_HEADERS)((PBYTE)pDosHdr + pDosHdr->e_lfanew);
  pFileHdr = &pNTHdr->FileHeader;
  pSecHdr  = (PIMAGE_SECTION_HEADER)((PBYTE)pNTHdr + sizeof(IMAGE_NT_HEADERS32));

  for(i = 0; i < pFileHdr->NumberOfSections; i++)
  {
    if (IsBadReadPtr(pSecHdr, sizeof(IMAGE_SECTION_HEADER)))
      break;

    // Rule1
    if (pSecHdr[i].Characteristics & IMAGE_SCN_CNT_CODE && 0x1000 < pSecHdr[i].Misc.VirtualSize && pSecHdr[i].SizeOfRawData <= pSecHdr[i].Misc.VirtualSize - 0x1000)
      goto Exit1;

    // Rule2
    if (!memcmp(pSecHdr[i].Name, szSecNameText, sizeof(szSecNameText)) && 0x1000 < pSecHdr[i].Misc.VirtualSize && pSecHdr[i].SizeOfRawData < pSecHdr[i].Misc.VirtualSize - 0x1000)
      goto Exit1;

    // Rule3
    if (0 == *pSecHdr[i].Name && pSecHdr[i].SizeOfRawData < pSecHdr[i].Misc.VirtualSize)
      goto Exit1;

    // Rule4
    if (0 != pSecHdr[i].SizeOfRawData)
      continue;

    // Rule5
    if (0x1000 > pSecHdr[i].Misc.VirtualSize)
      continue;

    // Rule6
    if (!memcmp(pSecHdr[i].Name, szSecNameTextBSS, sizeof(szSecNameTextBSS)))
      continue;

    // Rule7
    if (!memcmp(pSecHdr[i].Name, szSecNameTLS, sizeof(szSecNameTLS)))
      continue;
    
    // Rule8
    if (!memcmp(pSecHdr[i].Name, szSecNameBSS, sizeof(szSecNameBSS)))
      continue;

    // Rule9
    if (!memcmp(pSecHdr[i].Name, szSecNameData, sizeof(szSecNameData)))
      continue;

    // Rule10
    if (pSecHdr[i].Characteristics & (IMAGE_SCN_CNT_INITIALIZED_DATA | IMAGE_SCN_CNT_UNINITIALIZED_DATA))
      goto Exit1;
  }
  goto Exit0;
Exit1:
  bRet = TRUE;
Exit0:
  return bRet;
}
```

由于我没有对此方法做样本的误报和漏报测试，只是简单的验证了一下代码的正确性，所以我无法肯定的指出哪种方法（Entropy vs Section）更优秀。

个人认为，Entropy 的方式有强大的理论背景支持（尽管我也不太懂那是为什么），可能更准确些；而 Section 的方式貌似没有很强的理论背景支持（谁知道这些规则有什么道理，也许这只有微软和 Matt Pietrek 等人才比较清楚吧！），很有可能会产生较多的误判和漏判 。

## 第二章、你加了什么壳

这章本不应该放在最后，因为我所采用的方法是绝大多数人都会想到的方法。的确，对于确定壳的种类没有比特征码更简单明了的方法了。

PEiD 中对于壳特征开放了 `userdb.txt`，广大分析员将自己辛勤的劳动成果共享于网络。要用代码去实现壳的分类，你需要做的就是：

1. 甄选这些特征。
2. 修改一个字符匹配算法（支持通配符）。

ps.值得一提的是 PEiD 自己还维护了一个内部特征，我们不得而知:(

![](img/7830e9c61551ef6219e166f220fd4c2d.jpg)

这就够了吗？

用人工维护特征这种方法，向来就具备以下几个弱点：

1. 滞后性
   特征只能针对已知的壳，那些最新的壳，你需要时间去发现和收集
2. 局限性
   你不可能提取出所有壳的特征，判定程序只能无奈的返回 Unknown
3. 误判
   人永远无法保证 100%的正确

我曾经试图想解决三个问题，可是目前还没有更好的解决方案。即便如此让机器代理人来提取特征仍然是有可能的。设计一个类似神经网络的系统，让壳样本不断的去训练你的特征。当所有的流程无需人工参与的时候，虽然滞后性无法避免但可以缩短时间；虽然局限性无法根除，但可以降低比率；误判无法保证 100%，那就保证 99.999999%。

好吧，我承认。这一章我有点扯...你们还是无视吧！
