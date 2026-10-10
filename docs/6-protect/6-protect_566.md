

# 软件保护壳专题 - 代码替换引擎的构建

> 原文：[`www.pediy.com/kssd/pediy10/96644.html`](https://www.pediy.com/kssd/pediy10/96644.html)

代码替换引擎的构建  
<目录>  
0.<什么是替换>  
1.<替换的步骤>  
2.<代码流程图的构建>  
3.<与乱序的结合>  
4.<开始替换>  
5.<替换的 BUG>  
6.<鸡肋 or NOT~~~>  
  
正文 Loading...  
  
<正文>  
0.什么是替换  
   代码替换每个使用过壳的人应该都不会陌生,把指定内存替换成用户指定的代码。但是你尝试过编写  
自动替换吗？把正常代码的地方搬运到其他地点。然后自动在原先处加上花指令。这样且不说达到了变形  
的效果。也使 Cracker 在 F8 与 F7 之间徘徊。如果没有一个完美的 Anti-Debuger 的技巧。我们在 R3 下所用做  
到的极限就是消耗 Cracker 的耐心。  
  
1.替换的步骤  
   0.构建代码流程图  
   1.找到两个 JMP/CALL/JCC 之间的代码段，这个代码段符合以下要求,长度要大于等于 5 个字节(以便放置  
   跳转指令)这段指令没有其他跳转跳入(如果能排除动态跳转最好,但是不太可能,这也是产生不能修复的  
   BUG 的原因)  
   1-x.找到的这个代码段可以不忽略有其他跳转跳入  
   2.添加一个新节以便储存这些被替换的代码。  
   3.遍历这个结构,把要替换的代码段搬运到新的空间,在新的空间末尾添加一个跳入原先代码段结束的  
   地址。  
   4.将原先的代码块的首 5 个字节添加一个 JMP 跳入到搬运后的地址。并填充同样长度的花指令。  
   5.将这个代码块除了首 5 个字节其余字节以花指令填充。  
   5-x.将这个代码块其余的字节,如果大于等于 5 个字节长度,添加 JMP 指令以便指向与这个 JMP 次序相等的  
   原先指令的位置。  
   原先代码序列:     现在代码序列:       搬运后的地址:  
   A                 JMP N\_A             N\_A  
   B                 JMP N\_B             N\_B  
   C                 JMP N\_C             N\_C  
   D                 JMP N\_D             N\_D  
   JMP 类指令-J       JMP 类指令-J         JMP JMP 类指令-J  
   替换后成以上结构  
   这个替换的意义在于防止动态跳转跳入到替换后的代码块中间的位置。由于没有适当的跳入到正确的  
   指令位置引发的错误。不过还是不能完全防止此类错误发生因为每块代码不可能完全是以 5 字节指令对齐  
   也许会出现以下状况  
   替换后的代码序列:  
   JMP N\_A  
   JMP N\_B  
   JMP N\_C  
   BUG\_JMP:  
   .... 不足 5 字节的任意数据 ....  
   JMP 类指令-J  
   如果在代码段的其他位置以动态跳转的形式跳入到 BUG\_JMP 就会发生错误。(偏移类我们可以通过过滤  
   的手段来筛选掉)。  
   6.检查搬运空间的长度是否还足够放置更多的代码块。  
     
要说明的是也可以加入随机手段来控制来进行替换。  
  
2.代码流程图的构建  
   这份代码在专题的 《代码乱序引擎的构建》中已经给出。由于替换引擎也需要它的支持。代码就到上一节  
   看吧。  
  
3.与乱序的结合  
   代码替换生成的全是偏移类跳转为什么不和上一节中的代码乱序结合呢？这样的组合产生的东西是很奇妙的  
   在加上随机率的变化。一定会让 Cracker 很头疼。  
  
4.开始替换  
   原理就是以上所写的,这里直接给出一份比较稳定的代码。  

代码:

```
// 先给出一个替换的结构。记录这个结构的目的还有一个就是为了更复杂的变换。
const DWORD CodeReplace\_MixSize = 0x05;//代码替换最小指令块长度,一个 32 位跳转指令的长度
typedef struct \_CODEREPLACE\_NODE
{
  struct \_CODEREPLACE\_NODE \*pNext;//下一个节点
  DWORD dwSize;//代码块长度
  DWORD dwMemoryAddress;//当前内存地址
  LPBYTE pFileAddress;//当前文件地址
  DWORD dwReplaceMemoryAddress;//替换后的内存地址
  LPBYTE pReplaceFileAddress;//替换后的文件地址
  DWORD dwFinalMemoryAddress;//最终跳转的内存地址
  LPBYTE pFinalFileAddress;//最终跳转的文件地址
  DWORD dwFinalOffset;//最终跳转的偏移
} CODEREPLACE\_NODE, \*PCODEREPLACE\_NODE;

// let's go
// 函数里传递的 PCODE\_FLOW pCodeMixFlow 的目的是为了和代码乱序结合,也为了在结合过程中产生的流程改变
// 防止 BUG 跳转而传递的
PCODEREPLACE\_NODE DrawCodeReplace(CONST LPBYTE pMem, CONST LPBYTE pStore, PCODEREPLACE\_CONFIGURE pCodeReplaceConfigure, PJUNKCODE\_BASE pJunkCodeBase, PCODE\_FLOW pCodeMixFlow)
{
  CxPeDiy PeDiy;
  DWORD dwImageBase = PeDiy.GetNtHeader(pMem)->OptionalHeader.ImageBase;
  // 生成流程图
  LPBYTE pCodeReplaceFileStart = pMem + PeDiy.Rva2Raw(pMem, (DWORD)(pCodeReplaceConfigure->dwStartAddress - dwImageBase));
  PCODE\_FLOW pCodeFlow = DrawCodeFlow(pMem, pCodeReplaceFileStart, pCodeReplaceConfigure->dwSize);
  if (pCodeFlow == NULL)
    return NULL;
  PCODE\_FLOW\_NODE pCurrNode = pCodeFlow->pCodeFlow;
  LPBYTE pCurrStore = pStore, pCodeFileAddress = pCodeReplaceFileStart;
  DWORD dwRemain = pCodeReplaceConfigure->dwSpaceSize;
  PCODEREPLACE\_NODE pCodeReplaceHeader = NULL, \*pCurrCodeReplaceNode = &pCodeReplaceHeader;

  // 如果开启智能替换则生成整体代码图
  PCODE\_FLOW pIntelligentFlow = NULL;
  PCODE\_FLOW\_NODE pCurrIntelligentNode = NULL;
  if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
  {
    LPBYTE pIntelligentStart = pMem + PeDiy.GetEntryPointSection(pMem)->PointerToRawData;
    DWORD dwIntelligentSize = PeDiy.GetEntryPointSection(pMem)->Misc.VirtualSize;
    pIntelligentFlow = DrawCodeFlow(pMem, pIntelligentStart, dwIntelligentSize);
    pCurrIntelligentNode = pIntelligentFlow->pCodeFlow;
  }

  // 替换的必要条件
  // 起始的指令到它最近的 JMP 指令之间,加上这条指令本身的长度最小要等于 5
  
  // 设定初始值
  // 过滤起始的指令也为 JMP 偏移,循环的目的是为了过滤连续的 JMP 类指令
  while ((pCurrNode != NULL) && (pCodeFileAddress == pCurrNode->pFileAddress))
  {
    pCodeFileAddress += (pCurrNode->dwInsLen);
    pCurrNode = pCurrNode->pNext;
  }

  // 首先找出每个能够替换的代码块
  DWORD dwBlockSize = 0;
  DWORD dwBlockSizeMin = pCodeReplaceConfigure->dwBlockSizeMin;
  while ((pCurrNode != NULL) && (dwRemain > 0))
  {
    // 根据概率筛选,如果概率没满足则节点移动
    if (RandomRoll(pCodeReplaceConfigure->RandomArray) == FALSE)
    {
      pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
      pCurrNode = pCurrNode->pNext;
      continue;
    }

    // 这里代码块的长度首先要最小等于 6,最大+5 个字节小于剩余尺寸
    while (pCurrNode != NULL)
    {
      // 验证代码块长度是否合乎标准,如果长度合格则直接跳出
      dwBlockSize = pCurrNode->pFileAddress - pCodeFileAddress;
      // 这里代码块的长度暂时取最小 5 个字节,由于有连续两个 JMP 类指令的影响,或许会出现错误
      // 所以这里最小的代码块要大于 5 字节
      if ((((dwBlockSize + 0x05) <= dwRemain) && (dwBlockSize > dwBlockSizeMin)))
      {
        // 满足条件直接跳出这个循环,找到合适的代码块
        break;
      }
      // 计算并继续验证
      pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
      pCurrNode = pCurrNode->pNext;
    }

    // 流程图是否到达末尾,如果到达直接结束
    if (pCurrNode == NULL)
    {
      if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
        FreeCodeFlow(&pIntelligentFlow);
      FreeCodeFlow(&pCodeFlow);
      return pCodeReplaceHeader;
    }

    // 是否开启智能替换,如果开启则进行更进一步的地址筛选
    // 到了这里这段代码代码已经没有向外跳的代码了,智能筛选
    // 筛选出没有任何地址向里跳的.但是不保证使用动态的指针
    // 跳转,例如:call dword ptr \[XXXX\].XXXX 的值在代码块中
    if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
    {
      DWORD dwStartAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCodeFileAddress - pMem));
      DWORD dwEndAddress = dwStartAddress + dwBlockSize;
      PCODE\_FLOW\_NODE pCheckGoodNode = pCurrIntelligentNode;
      DWORD dwGotoAddress = 0;
      BOOL bGot = FALSE;
      // 遍历流程图,验证找到的代码块是否没有指令跳入
      while (pCheckGoodNode != NULL)
      {
        // 验证地址
        dwGotoAddress = pCheckGoodNode->dwGotoMemoryAddress;
        if ((dwGotoAddress > dwStartAddress) && (dwGotoAddress < dwEndAddress))
        {
          // 找到了一个忽略此次选择,进行重新选择
          bGot = TRUE;
          break;
        }
        pCheckGoodNode = pCheckGoodNode->pNext;
      }/\* end while \*/

      // 遍历第 2 组(代码混淆构建的流程图)
      PCODE\_FLOW\_NODE pCheckGoooNodeCodeMix = NULL;
      if (pCodeMixFlow != NULL) pCheckGoooNodeCodeMix = pCodeMixFlow->pCodeFlow;
      while (pCheckGoooNodeCodeMix != NULL)
      {
        // 验证地址
        dwGotoAddress = pCheckGoooNodeCodeMix->dwGotoMemoryAddress;
        if ((dwGotoAddress > dwStartAddress) && (dwGotoAddress < dwEndAddress))
        {
          // 找到了一个忽略此次选择,进行重新选择
          bGot = TRUE;
          break;
        }
        pCheckGoooNodeCodeMix = pCheckGoooNodeCodeMix->pNext;
      }

      // 如果不符合
      if (bGot == TRUE)
      {
        // 重新进行筛选
        pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
        pCurrNode = pCurrNode->pNext;
        continue;
      }/\* end if \*/
    }

    // 找到合适的代码块
    // 分配空间
    \*pCurrCodeReplaceNode = new CODEREPLACE\_NODE;
    ZeroMemory((\*pCurrCodeReplaceNode), sizeof(CODEREPLACE\_NODE));
    (\*pCurrCodeReplaceNode)->dwSize = dwBlockSize;
    (\*pCurrCodeReplaceNode)->pFileAddress = pCodeFileAddress;
    DWORD dwRaw = PeDiy.Raw2Rva(pMem, (DWORD)(pCodeFileAddress - pMem));
    if (dwRaw == 0)
    {
      int i = i + 1;
    }
    (\*pCurrCodeReplaceNode)->dwMemoryAddress = dwImageBase + dwRaw;
    (\*pCurrCodeReplaceNode)->pFinalFileAddress = pCurrNode->pFileAddress;
    (\*pCurrCodeReplaceNode)->dwFinalMemoryAddress = pCurrNode->dwMemoryAddress;
    (\*pCurrCodeReplaceNode)->pReplaceFileAddress = pCurrStore;
    (\*pCurrCodeReplaceNode)->dwReplaceMemoryAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCurrStore - pMem));
    DWORD dwFinalOffset = (\*pCurrCodeReplaceNode)->dwReplaceMemoryAddress + dwBlockSize;
    dwFinalOffset -= ((\*pCurrCodeReplaceNode)->dwFinalMemoryAddress);
    dwFinalOffset--;
    dwFinalOffset = ~dwFinalOffset;
    dwFinalOffset -= 0x05;//加上 JMP 指令的长度
    (\*pCurrCodeReplaceNode)->dwFinalOffset = dwFinalOffset;
    BYTE JmpCodes\[\] = {"\\xE9\\xFF\\xFF\\xFF\\xFF"};
    \*(DWORD \*)&(JmpCodes\[1\]) = dwFinalOffset;
    // 填充
    memcpy(pCurrStore, (\*pCurrCodeReplaceNode)->pFileAddress, dwBlockSize);
    memcpy(pCurrStore + dwBlockSize, JmpCodes, 0x05);
    // 计算偏移并覆盖原指令
    DWORD dwReplaceOffset = (\*pCurrCodeReplaceNode)->dwReplaceMemoryAddress - (\*pCurrCodeReplaceNode)->dwMemoryAddress - 0x05;
    \*(DWORD \*)&(JmpCodes\[1\]) = dwReplaceOffset;
    memcpy((\*pCurrCodeReplaceNode)->pFileAddress, JmpCodes, 0x05);
    // 产生花指令
    DWORD dwThunkCodeSize = dwBlockSize - 0x05;
    LPBYTE pThunkCode = new BYTE \[dwThunkCodeSize\];
    if (pThunkCode == NULL)
    {
      // 恢复
      memcpy((\*pCurrCodeReplaceNode)->pFileAddress, pCurrStore, dwBlockSize);
      delete (\*pCurrCodeReplaceNode);
      (\*pCurrCodeReplaceNode) = NULL;
      FreeCodeFlow(&pCodeFlow);
      return pCodeReplaceHeader;
    }
    ZeroMemory(pThunkCode, dwThunkCodeSize);
    GenerateThunkCodeBySize(pJunkCodeBase, pThunkCode, dwThunkCodeSize);
    memcpy((\*pCurrCodeReplaceNode)->pFileAddress + 0x05, pThunkCode, dwThunkCodeSize);//跳过第一个 JMP 指令
    delete \[\] pThunkCode;
    pCurrStore += (dwBlockSize + 0x05);//0x05 表示 JMP 指令的长度
    dwRemain -= (dwBlockSize + 0x05);


    // 是否需要添加代码替换保险,增大变形率,增加稳定性,但是也有可能造成错误
    if (pCodeReplaceConfigure->bCodeReplaceFinalJmp == TRUE)
    {
      // 利用反汇编器将替换后指令的长度计算出来
      ud\_t ud\_obj;
      ud\_init(&ud\_obj);
      ud\_set\_mode(&ud\_obj, 32);
      ud\_set\_syntax(&ud\_obj, UD\_SYN\_INTEL);
      ud\_set\_input\_buffer(&ud\_obj, (\*pCurrCodeReplaceNode)->pReplaceFileAddress, (\*pCurrCodeReplaceNode)->dwSize);
      // 设定开始的文件与内存指针
      LPBYTE pFileAddress = (\*pCurrCodeReplaceNode)->pFileAddress + 0x05;
      DWORD dwMemoryAddress = (\*pCurrCodeReplaceNode)->dwMemoryAddress + 0x05;
      ud\_disassemble(&ud\_obj);// 反汇编
      DWORD dwRemainReplaceSize = (\*pCurrCodeReplaceNode)->dwSize - 0x05;//剩余空间按照原代码块位置计算,并以 JMP 指令长度递减
      // 如果这个长度小于 5 则跳过
      if (dwRemainReplaceSize >= 0x05)
      {
        DWORD dwRealMemoryAddress = (\*pCurrCodeReplaceNode)->dwReplaceMemoryAddress + ud\_obj.inp\_ctr;
        LPBYTE pReadFileAddress = (\*pCurrCodeReplaceNode)->pReplaceFileAddress + ud\_obj.inp\_ctr;
        DWORD dwOffsetOther = 0;
        while (dwRemainReplaceSize >= 0x05)
        {
          dwOffsetOther = dwRealMemoryAddress - dwMemoryAddress - 0x05;
          \*(DWORD \*)&(JmpCodes\[1\]) = dwOffsetOther;
          memcpy(pFileAddress, JmpCodes, 0x05);
          pFileAddress += 0x05;
          dwMemoryAddress += 0x05;
          pReadFileAddress += ud\_disassemble(&ud\_obj);
          dwRealMemoryAddress += ud\_disassemble(&ud\_obj);
          dwRemainReplaceSize -= 0x05;
        }/\* end while \*/

        // 查看是否需要补齐花指令
        if (dwRemainReplaceSize != 0)
        {
          // 补齐花指令
          DWORD dwThunkCodeOtherSize = dwRemainReplaceSize;
          LPBYTE pThunkCodeOther = new BYTE \[dwThunkCodeOtherSize\];
          if (pThunkCodeOther != NULL)
          {
            ZeroMemory(pThunkCodeOther, dwThunkCodeOtherSize);
            GenerateThunkCodeBySize(pJunkCodeBase, pThunkCodeOther, dwThunkCodeOtherSize);
            memcpy(pFileAddress, pThunkCodeOther, dwThunkCodeOtherSize);
            delete \[\] pThunkCodeOther;
          }
        }
      }/\* end if \*/
    }/\* end if \*/
    pCurrCodeReplaceNode = &((\*pCurrCodeReplaceNode)->pNext);
    pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
    pCurrNode = pCurrNode->pNext;
  }/\* end while \*/

  // 释放流程图空间
  if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
    FreeCodeFlow(&pIntelligentFlow);
  FreeCodeFlow(&pCodeFlow);
  return pCodeReplaceHeader;
}

```

5.替换的 BUG  
  替换后的硬伤就在于动态跳转到 BUG\_JMP 的位置,每段代码块不能以 5 字节对齐这些。这里有一个补丁的  
  方式就是采用重定位节。不过不能保证 EXE 里都有重定位节。如果是 DLL 或者编译 EXE 时增加了重定位信息  
  那么可以顺利补丁此类 BUG。  
  还可以建立一个智能的筛选方案，就是在筛选代码块是只选以 5 字节对齐的代码,不过就算是这样也无法  
  完全弥补动态跳转带来的影响，它有可能跳转到 JMP 之类中间的地址。  
  
6.鸡肋 or NOT~~~  
  由于有致命的硬伤，所以自动替换带来的后果就是不一定适合每个程序。稳定性不太高。这里有个  
  介于中间的解决方案。当自动替换后如果发生错误。记录下代码块的位置。将它排除在外。显然  
  以上代码里没有做这个功能，兴趣高的朋友自行添加吧。这个功能我测试了几个程序成功率有一少半。  
  加了 10 个程序 4 个程序可以正常运行。这里显然和所用编译器和编写代码有关。不过对于  
  Anti-Anti-Virus 此类软件，这样的替换对于特征码改变来说是种不错的选择。![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")