
# 软件保护壳技术专题 - 识别函数与数据区域

> 原文：[`www.pediy.com/kssd/pediy10/102977.html`](https://www.pediy.com/kssd/pediy10/102977.html)

![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬") ![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心") ![](img/1e00a4c5ba6d273521879fcbd5363999.jpg "碌眼") ![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色") ![](img/32b12ba0f4c0e6a8843aaa06ba62fbdd.jpg "皱眉") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷") ![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑") ![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif") ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑") ![](img/1e3c9433a1c444ac212c55a1ce70452a.jpg "狂怒") ![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

做壳很容易，有点做病毒知识的人都可以编写出壳来。但强度就有所差别了。

什么才是凶猛的壳？

我觉的是这样的 `猛壳 = 识别 + 重定位`。识别用于让壳更了解要保护的程序。重定位为了可以保证保护程序的正常运行。如果能把识别与重定位做好。那么很多新的玩意就都可以玩了。如果你能把一个壳做成一个 `2` 进制文件的 `2` 次编译器。这个编译器的输入是以编译好的程序。输出是进行重编译的程序。那么这个壳因该相当的犀利。不过事与愿违。要完美的解决以上技术问题。有相当大的难度。我们也许不能把这个程序进行这样的重编译。但是局部的代码因该还是可以的。例如 `VMP` 不正是这样的产物吗？ 写篇文章把我最近搞的一个函数识别的小算法奉献给朋友们。

### 算法流程

#### 0. 单函数识别

从一个地址开始,在未知长度的情况下,识别出这个函数的长度。

流程算法如下:

1. 从当前位置开始进行反汇编
2. 记录所有跳转流程指令
3. 遇到 `ret` 或者 `retn` 指令后,回溯整个跳转流程链表
4. 如果找到 `jcc` 则判断是否跳转后的地址是否大于当前的指令地址
   - 大于,则跳转到流程,进入预分析结尾流程
   - 无大于的情况,此函数分析完毕,返回

**预分析结尾:**

这里鉴别函数帧也可能造成不稳定,有些函数在 `ret` 之后还有一些指令在这些指令之后必定有一个 `JMP` 跳转到最后一个 `ret` 或者之前利用此来进行预分析结尾

这里出现种情况:

1. 找到上述那个最后的 `JMP` 则将此视为结尾
2. 如果找到 `nop` 视为结尾(用户可以指定此选项,默认存在)
3. 如果找到 `int3` 视为结尾(用户可以指定此选项,默认存在)
4. 如果找到视为结尾(用户可以指定此选项,默认存在)

#### 1. 交叉引用

从一个函数开始,检查它的对外跳转的语句以便达到另一个函数,如此循环识别出代码段中的所有被直接引用的函数体。

识别交叉引用采用三遍分析,并在分析过程中记录每天指令,并设置是否可以 `HOOK`(用于代码乱序)

流程算法如下:

**第一遍分析**

1. 依次对所有指令进行分析,设定指令是否可以交换,并进行初步 `HOOK` 分析刺探,将通过偏移的的跳转指令进行识别,并鉴别 `HOOK` 成功率,并填充这条指令对标志寄存器的影响和通用寄存器的影响
2. 通过单函数鉴别 (上述已经阐述了其算法)函数确立入口函数的长度
3. 再次遍历入口函数,通过分析入口函数的 `CALL` 指令,分析出其他函数
4. 循环,2 过程直到把映射中所有的函数都鉴别出来
5. 进行链表排序(地址由低到高)
6. 将链表与映射内存由高到低依次进行对比,找出未找到的代码块,并使用,1,2 方法进行鉴别将无法分析的代码识为数据

**第二遍分析**

1. 遍历函数链
2. 在识别过程中,鉴别出一个函数开辟的局部变量所需的空间,具体监视 `sub esp, XXX` 指令
3. 在遍历过程中如果发现有指令直接对全局变量进行引用,记录其访问地址
4. 如果遇到跳转指令,则设置它目标地址指令的跳转引用链表

**第三遍分析**

1. 遍历函数链
2. 找到所有动态跳转指令,并对它进行反馈分析,直到找出动态跳转寄存器值的合成结果,并进行运算

这里反馈的结束算法是:

查看当前跳转指令,是否由引用指令

如果有则遍历这个引用链

如果没有则向上分析条指令,如果没有存在可以影响寄存器的 `mov`, `lea` 指令,则直接视此指令为不可 `HOOK` 的指令,如果动态跳转出现使用两个寄存器做内存跳转的直接视为不可 `HOOK` 的指令。

当然这里有些步骤是可以合并的。在实际 `CODE` 中我将上面三遍合成为两遍原则。

#### 2. 识别代码段的数据与代码

在理论上代码与数据是不可分的,通过分析将代码与数据进行分离。这也是难度最大的地方。

这里给出一些代码。

很多朋友要问为什么要再次鉴别这些东西。从交叉引用就可以识别为什么还要继续在重新识别一遍

原因很简单：为了可以保证所有函数和无用区域都作为信息传递给壳。有些动态调用的从交叉引用很难分析得出。在引用链上没有。所以要采取这个方案重新把他找出来。还有那些函数与函数之间的区域，这些区域你没感觉到对于做壳来说很珍贵吗？发挥你想象力去利用这些空白区域。。。

下面算法没给出深度分析的算法。那个可以自己总结规则了。现在我的那个代码已经过 `W` 行了。太多了规则了。 规则越细，可以保证识别程度越准确。

代码:



```
/*
 * 预读分析阶段,在分区处理是时进行,数据与代码区域
 * 范围的界定,纯数据区域返回 TRUE,反之为 FALSE
 */
INLINE  BOOL PredictBlockEnd(LPBYTE pMem, LPBYTE pCurr, DWORD dwSize, DWORD *pOutSize, PANALYZE\_CONFIGURE pAnalyzeConfigure)
{
  BOOL bBlock = FALSE;
  DWORD dwOffset = 0;
  ud\_t ud\_obj;
  ud\_init(&ud\_obj);
  ud\_set\_mode(&ud\_obj, 32);
  ud\_set\_syntax(&ud\_obj, UD\_SYN\_INTEL);
  ud\_set\_input\_buffer(&ud\_obj, pCurr, dwSize);
  while (ud\_disassemble(&ud\_obj) != 0)
  {
    enum ud\_mnemonic\_code mnemonic = ud\_obj.mnemonic;
    if ((mnemonic == UD\_Inop) ||
      (mnemonic == UD\_Iint3) ||
      ((mnemonic == UD\_Iadd) &&
      (ud\_obj.inp\_ctr == 2) &&
      (*(WORD *)&(ud\_obj.inp\_sess) == 0)))
    {
      /*
       * 到达结束条件
       * 检查是否到达了用户定义代码的最小范围,如果没到直接视为数据
       * 如果大于等于则进入深入鉴别
       */
      if (dwOffset < pAnalyzeConfigure->bCodeMixSize)
      {
        bBlock = TRUE;
      }
      else
      {
        // 进入深度分析
        bBlock = DeepAnalyzeBlock(pMem, pCurr, dwOffset, pAnalyzeConfigure);
      }
      *pOutSize = dwOffset;
      return bBlock;
    }/* end if */
    dwOffset += ud\_insn\_len(&ud\_obj);
  }

  // 这里做深度鉴别
  bBlock = DeepAnalyzeBlock(pMem, pCurr, dwSize, pAnalyzeConfigure);
  *pOutSize = dwOffset;
  return bBlock;
}

/*
 * 进行分区处理
 * 以 NOP,INT3,0,无效指令编码字符进行分区标志
 * 遇到 NOP 或者 INT3,0,无效指令编码则将扫描过的区域
 * 作为一个区块摘出,继续查询,如果其后的数据有很长
 * 一段为连续的 NOP,INT3,0 字节则将此视做绝对无效区域,
 * 在扫描过程中,如果遇到无效指令直接忽略当作这个区域
 * 的一部分如果遇到有效指令,则进入预分析阶段,尝试向前
 * 分析如果分析的指令长度小于最小代码长度则直接忽略.如果
 * 大于则将此区域继续分析,往返以上流程.直到完毕
 */
INLINE PPROCEDURE FormatBlock(LPBYTE pMem, LPBYTE pStart, DWORD dwSize, PANALYZE\_CONFIGURE pAnalyzeConfigure)
{
  DWORD dwImageBase = GetNtHeader(pMem)->OptionalHeader.ImageBase;
  PPROCEDURE pBlock, *pCurrBlockPoint = NULL;
  pCurrBlockPoint = &pBlock;
  DWORD dwBlockEndSignCount = 0, dwOffset = 0;

  ud\_t ud\_obj;
  ud\_init(&ud\_obj);
  ud\_set\_mode(&ud\_obj, 32);
  ud\_set\_syntax(&ud\_obj, UD\_SYN\_INTEL);
  ud\_set\_input\_buffer(&ud\_obj, pStart, dwSize);

  while (ud\_disassemble(&ud\_obj) != 0)
  {
    enum ud\_mnemonic\_code mnemonic = ud\_obj.mnemonic;
    if ((mnemonic == UD\_Inop) ||
      (mnemonic == UD\_Iint3) ||
      ((mnemonic == UD\_Iadd) &&
      (ud\_obj.inp\_ctr == 2) &&
      (*(WORD *)&(ud\_obj.inp\_sess) == 0)))
    {
      dwBlockEndSignCount += ud\_insn\_len(&ud\_obj);
    }
    else
    {
      /*
       * 遇到有效指令了
       * 现在该进入预读分析阶段,这里还有一个判断
       * 标志是剩余长度,如果其后的长度小于最小代码
       * 长度直接将这段代码视作为数据
       */
      DWORD dwRemainSize = dwSize - dwOffset;
      LPBYTE pCurr = pStart + dwOffset;
      if (dwRemainSize >= pAnalyzeConfigure->bCodeMixSize)
      {
        /*
         * 进入预读分析阶段
         * 在这里开始判断,如果预读处理分析的结果为单纯的数据块则
         * 与先前分析的区域进行合并
         * 否则单独为前面分析的块分配内存
         */
        DWORD dwOutSize = 0;
        if (PredictBlockEnd(pMem, pCurr, dwRemainSize, &dwOutSize, pAnalyzeConfigure) == TRUE)
        {
          // 进行合并
          dwBlockEndSignCount += dwOutSize;
        }
        else
        {
          /*
           * 判断上一组区块标记是否为 0 如果不为则分配空间存储它
           */
          if (dwBlockEndSignCount != 0)
          {
            (*pCurrBlockPoint) = \_\_new\_\_(PROCEDURE, 1);
            memset((*pCurrBlockPoint), 0, sizeof(PROCEDURE));
            (*pCurrBlockPoint)->bBlock = TRUE;//代码区域
            (*pCurrBlockPoint)->pFileStartAddress = pCurr - dwBlockEndSignCount;
            (*pCurrBlockPoint)->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)((*pCurrBlockPoint)->pFileStartAddress - pMem));
            (*pCurrBlockPoint)->dwSize = dwBlockEndSignCount;
            pCurrBlockPoint = &((*pCurrBlockPoint)->pNext);
            dwBlockEndSignCount = 0;//清空标志记录计数
          }
          // 设定此段为代码区域
          (*pCurrBlockPoint) = \_\_new\_\_(PROCEDURE, 1);
          memset((*pCurrBlockPoint), 0, sizeof(PROCEDURE));
          (*pCurrBlockPoint)->bBlock = FALSE;//代码区域
          (*pCurrBlockPoint)->pFileStartAddress = pCurr;
          (*pCurrBlockPoint)->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)(pCurr - pMem));
          (*pCurrBlockPoint)->dwSize = dwOutSize;
          pCurrBlockPoint = &((*pCurrBlockPoint)->pNext);
        }

        // 重新设定反汇编指针
        pCurr += dwOutSize;
        dwOffset += dwOutSize;
        dwRemainSize -= dwOutSize;//剩余长度
        ud\_set\_input\_buffer(&ud\_obj, pCurr, dwRemainSize);
        continue;
      }
      else
      {
        // 进入到这里就是进入到分析的末尾
        (*pCurrBlockPoint) = \_\_new\_\_(PROCEDURE, 1);
        memset((*pCurrBlockPoint), 0, sizeof(PROCEDURE));
        (*pCurrBlockPoint)->bBlock = TRUE;//表示有可能是数据
        dwBlockEndSignCount += dwRemainSize;
        (*pCurrBlockPoint)->dwSize = dwBlockEndSignCount;
        (*pCurrBlockPoint)->pFileStartAddress = pCurr;
        (*pCurrBlockPoint)->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)(pCurr - pMem));
        pCurrBlockPoint = &((*pCurrBlockPoint)->pNext);
        dwBlockEndSignCount = 0;//清空标志记录计数
      }
    }
    dwOffset += ud\_insn\_len(&ud\_obj);
  }

  /*
   * 处理组后一个纯区域块
   * 两种情况造成这种现象
   * 1) 整个分析的数据都是纯数据区
   * 2) 被分析的数据最后一个区域是纯数据区
   */
  if (dwBlockEndSignCount != 0)
  {
    LPBYTE pBlockZoon = NULL;
    // 第一种情况
    if (dwBlockEndSignCount == dwSize)
    {
      pBlockZoon = pStart;
    }
    else
    {
      pBlockZoon = pStart + dwSize - dwBlockEndSignCount;
    }
    (*pCurrBlockPoint) = \_\_new\_\_(PROCEDURE, 1);
    memset((*pCurrBlockPoint), 0, sizeof(PROCEDURE));
    (*pCurrBlockPoint)->bBlock = TRUE;//表示有可能是数据
    (*pCurrBlockPoint)->dwSize = dwBlockEndSignCount;
    (*pCurrBlockPoint)->pFileStartAddress = pBlockZoon;
    (*pCurrBlockPoint)->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)(pBlockZoon - pMem));
    pCurrBlockPoint = &((*pCurrBlockPoint)->pNext);
    dwBlockEndSignCount = 0;//清空标志记录计数
  }

  return pBlock;
}

/*
 * 进行预分析,识别出数据
 * 如果分析出不是数据
 */   
INLINE PPROCEDURE AnalyzeData(LPBYTE pMem, LPBYTE pCurr, DWORD dwSize, PPROCEDURE pProcedureList, PPROGRAM pParents)
{
  PANALYZE\_CONFIGURE pAnalyzeConfigure = &(pParents->AnalyzeConfigure);
  DWORD dwImageBase = GetNtHeader(pMem)->OptionalHeader.ImageBase;
  // 如果此块的长度小于用户指定长度
  if (dwSize < pAnalyzeConfigure->bCodeMixSize)
  {
    PPROCEDURE pBlock = \_\_new\_\_(PROCEDURE, 1);
    memset(pBlock, 0, sizeof(PROCEDURE));
    pBlock->bBlock = TRUE;
    pBlock->pFileStartAddress = pCurr;
    pBlock->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)(pCurr - pMem));
    pBlock->dwSize = dwSize;
    pBlock->pNext = NULL;

    PPROCEDURE *pCurrMainPoint = &pProcedureList;
    while (*pCurrMainPoint != NULL) pCurrMainPoint = &((*pCurrMainPoint)->pNext);
    *pCurrMainPoint = pBlock;
    return pProcedureList;
  }

  /*
   * 开始对这个块进行有效的分区
   * 分区完毕后对链中的代码块进行函数帧分析
   * 初次分析
   */
  PPROCEDURE pFormatBlockList = FormatBlock(pMem, pCurr, dwSize, pAnalyzeConfigure);

  /*
   * 遍历此链表进行分析
   * 如果遇到代码块则进入并开始代码分析
   */
  PPROCEDURE pCurrBlock = pFormatBlockList;

  while (pCurrBlock != NULL)
  {
    if (pCurrBlock->bBlock == FALSE)//为代码块
    {
      // 进行分析
      PPROCEDURE pProcedure = GetProcFrame(pMem, pCurrBlock->pFileStartAddress, pCurrBlock->dwSize, pParents);
      /*
       * 调用 Procedure2Procedure 过后形成的函数链中存在两种函数,一种是函数存在在已有的
       * 未知区域中,一种是在做第一次扫描时分析到的函数
       */
      DWORD dwCount = Procedure2Procedure(pMem, pProcedureList, pProcedure);
    }
    else
    {
      // 直接连接到主链
      PPROCEDURE *pCurrMainPoint = &pProcedureList;
      while (*pCurrMainPoint != NULL) pCurrMainPoint = &((*pCurrMainPoint)->pNext);
      *pCurrMainPoint = \_\_new\_\_(PROCEDURE, 1);
      memcpy(*pCurrMainPoint, pCurrBlock, sizeof(PROCEDURE));
      (*pCurrMainPoint)->pNext = NULL;
    }
    pCurrBlock = pCurrBlock->pNext;
  }

  // 销毁分析链
  ReleaseProcedureList(&pFormatBlockList);
  return pProcedureList;
}

/*
 * 找寻所有不在链上的内存区域,并归结到链上
 */
INLINE PPROCEDURE AnalyzeProcedurePass1DiscoveryUnknowZoon(LPBYTE pMem, PPROCEDURE pProcedureList, PPROGRAM pParents)
{
  PPROCEDURE pCurrProcedure = pProcedureList;
  LPBYTE pStart = pMem + GetEntryPointSection(pMem)->PointerToRawData, pCurr = pStart;
  DWORD dwCodeSize = GetEntryPointSection(pMem)->Misc.VirtualSize;
  PPROCEDURE pUnknowProcedureList = NULL;
  LPBYTE pNow = NULL;
  DWORD dwCurrSize = 0, dwBlockSize = 0, dwRemainSize = 0;

  g\_pAnalyzeDataDispatcher = MakeAnalyzeDataFromInstructionDispatcher();

\_new\_analyze:
  while (pCurrProcedure != NULL)
  {
    pNow = pCurrProcedure->pFileStartAddress;
    dwCurrSize = pCurrProcedure->dwSize;

    if (pCurr == pNow)//如果相当则直接略过
    {
      pCurr += dwCurrSize;
      dwRemainSize -= dwCurrSize;
    }
    else if (pCurr < pNow)//分析这个代码块
    {
      dwBlockSize = (DWORD)(pNow - pCurr);
      dwRemainSize -= dwBlockSize;
      goto \_handler;
    }
    pCurrProcedure = pCurrProcedure->pNext;
  }

  // 判断是否到达末尾
  if (dwRemainSize != 0)
  {
    dwBlockSize = dwRemainSize;
    goto \_handler;
  }

  // 释放派遣表
  DestroyDispatcherList(&g\_pAnalyzeDataDispatcher);
  return pProcedureList;

// 这里为处理函数所在
\_handler:
  /*
   * 这里的处理和前一次处理有个区别就是不知道当前这个区域是
   * 数据还是代码,所有首先要做数据鉴别
   */
  pUnknowProcedureList = AnalyzeData(pMem, pCurr, dwBlockSize, pProcedureList, pParents);
  pProcedureList = SortProcedureList(pProcedureList);//排序
  // 重新设定所有初始值,然后重新来过
  pCurr = pStart;
  dwRemainSize = dwCodeSize;//重新设定长度
  pUnknowProcedureList = NULL;
  pCurrProcedure = pProcedureList;
  \_\_PrintProcedureListCount\_\_(pProcedureList);
  goto \_new\_analyze;//重新进入新的一次分析
}
```



### 3. YY

目前这种算法做规则很稳定。但是也很麻烦。不仅代码量大，而且要不断地更新规则。很麻烦。目前我正在通过人工神经网络去模糊识别代码和数据。已经有一个比较不错的方案。准备作为毕业的论文来搞。如果论文发表成功，我再翻译成中文奉献给论坛的朋友们。

三种流程如图:

`![](img/10630ab4b1a735788bfe3d94f41133b3.jpg)`

`![](img/63525d991087d269a3f01f8a3f2443cc.jpg)`

`![](img/09ea0c52d41d74ed3f91ebfafffbd2ac.jpg)`
