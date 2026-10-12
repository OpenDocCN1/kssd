
```cpp
// 先给出一个替换的结构。记录这个结构的目的还有一个就是为了更复杂的变换。
const DWORD CodeReplace_MixSize = 0x05;//代码替换最小指令块长度,一个 32 位跳转指令的长度
typedef struct _CODEREPLACE_NODE
{
  struct _CODEREPLACE_NODE *pNext;//下一个节点
  DWORD dwSize;//代码块长度
  DWORD dwMemoryAddress;//当前内存地址
  LPBYTE pFileAddress;//当前文件地址
  DWORD dwReplaceMemoryAddress;//替换后的内存地址
  LPBYTE pReplaceFileAddress;//替换后的文件地址
  DWORD dwFinalMemoryAddress;//最终跳转的内存地址
  LPBYTE pFinalFileAddress;//最终跳转的文件地址
  DWORD dwFinalOffset;//最终跳转的偏移
} CODEREPLACE_NODE, *PCODEREPLACE_NODE;

// let's go
// 函数里传递的 PCODE_FLOW pCodeMixFlow 的目的是为了和代码乱序结合,也为了在结合过程中产生的流程改变
// 防止 BUG 跳转而传递的
PCODEREPLACE_NODE DrawCodeReplace(CONST LPBYTE pMem, CONST LPBYTE pStore, PCODEREPLACE_CONFIGURE pCodeReplaceConfigure, PJUNKCODE_BASE pJunkCodeBase, PCODE_FLOW pCodeMixFlow)
{
  CxPeDiy PeDiy;
  DWORD dwImageBase = PeDiy.GetNtHeader(pMem)->OptionalHeader.ImageBase;
  // 生成流程图
  LPBYTE pCodeReplaceFileStart = pMem + PeDiy.Rva2Raw(pMem, (DWORD)(pCodeReplaceConfigure->dwStartAddress - dwImageBase));
  PCODE_FLOW pCodeFlow = DrawCodeFlow(pMem, pCodeReplaceFileStart, pCodeReplaceConfigure->dwSize);
  if (pCodeFlow == NULL)
    return NULL;
  PCODE_FLOW_NODE pCurrNode = pCodeFlow->pCodeFlow;
  LPBYTE pCurrStore = pStore, pCodeFileAddress = pCodeReplaceFileStart;
  DWORD dwRemain = pCodeReplaceConfigure->dwSpaceSize;
  PCODEREPLACE_NODE pCodeReplaceHeader = NULL, *pCurrCodeReplaceNode = &pCodeReplaceHeader;

  // 如果开启智能替换则生成整体代码图
  PCODE_FLOW pIntelligentFlow = NULL;
  PCODE_FLOW_NODE pCurrIntelligentNode = NULL;
  if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
  {
    LPBYTE pIntelligentStart = pMem + PeDiy.GetEntryPointSection(pMem)->PointerToRawData;
    DWORD dwIntelligentSize = PeDiy.GetEntryPointSection(pMem)->Misc.VirtualSize;
    pIntelligentFlow = DrawCodeFlow(pMem, pIntelligentStart, dwIntelligentSize);
    pCurrIntelligentNode = pIntelligentFlow->pCodeFlow;
  }

  // 替换的必要条件
  // 起始的指令到它最近的 JMP 指令之间,加上这条指令本身的长度最小要等于 5
  // 设定初始值
  // 过滤起始的指令也为 JMP 偏移,循环的目的是为了过滤连续的 JMP 类指令
  while ((pCurrNode != NULL) && (pCodeFileAddress == pCurrNode->pFileAddress))
  {
    pCodeFileAddress += (pCurrNode->dwInsLen);
    pCurrNode = pCurrNode->pNext;
  }

  // 首先找出每个能够替换的代码块
  DWORD dwBlockSize = 0;
  DWORD dwBlockSizeMin = pCodeReplaceConfigure->dwBlockSizeMin;
  while ((pCurrNode != NULL) && (dwRemain > 0))
  {
    // 根据概率筛选,如果概率没满足则节点移动
    if (RandomRoll(pCodeReplaceConfigure->RandomArray) == FALSE)
    {
      pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
      pCurrNode = pCurrNode->pNext;
      continue;
    }

    // 这里代码块的长度首先要最小等于 6,最大+5 个字节小于剩余尺寸
    while (pCurrNode != NULL)
    {
      // 验证代码块长度是否合乎标准,如果长度合格则直接跳出
      dwBlockSize = pCurrNode->pFileAddress - pCodeFileAddress;
      // 这里代码块的长度暂时取最小 5 个字节,由于有连续两个 JMP 类指令的影响,或许会出现错误
      // 所以这里最小的代码块要大于 5 字节
      if ((((dwBlockSize + 0x05) <= dwRemain) && (dwBlockSize > dwBlockSizeMin)))
      {
        // 满足条件直接跳出这个循环,找到合适的代码块
        break;
      }
      // 计算并继续验证
      pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
      pCurrNode = pCurrNode->pNext;
    }

    // 流程图是否到达末尾,如果到达直接结束
    if (pCurrNode == NULL)
    {
      if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
        FreeCodeFlow(&pIntelligentFlow);
      FreeCodeFlow(&pCodeFlow);
      return pCodeReplaceHeader;
    }

    // 是否开启智能替换,如果开启则进行更进一步的地址筛选
    // 到了这里这段代码代码已经没有向外跳的代码了,智能筛选
    // 筛选出没有任何地址向里跳的.但是不保证使用动态的指针
    // 跳转,例如:call dword ptr [XXXX].XXXX 的值在代码块中
    if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
    {
      DWORD dwStartAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCodeFileAddress - pMem));
      DWORD dwEndAddress = dwStartAddress + dwBlockSize;
      PCODE_FLOW_NODE pCheckGoodNode = pCurrIntelligentNode;
      DWORD dwGotoAddress = 0;
      BOOL bGot = FALSE;
      // 遍历流程图,验证找到的代码块是否没有指令跳入
      while (pCheckGoodNode != NULL)
      {
        // 验证地址
        dwGotoAddress = pCheckGoodNode->dwGotoMemoryAddress;
        if ((dwGotoAddress > dwStartAddress) && (dwGotoAddress < dwEndAddress))
        {
          // 找到了一个忽略此次选择,进行重新选择
          bGot = TRUE;
          break;
        }
        pCheckGoodNode = pCheckGoodNode->pNext;
      }/* end while */

      // 遍历第 2 组(代码混淆构建的流程图)
      PCODE_FLOW_NODE pCheckGoooNodeCodeMix = NULL;
      if (pCodeMixFlow != NULL) pCheckGoooNodeCodeMix = pCodeMixFlow->pCodeFlow;
      while (pCheckGoooNodeCodeMix != NULL)
      {
        // 验证地址
        dwGotoAddress = pCheckGoooNodeCodeMix->dwGotoMemoryAddress;
        if ((dwGotoAddress > dwStartAddress) && (dwGotoAddress < dwEndAddress))
        {
          // 找到了一个忽略此次选择,进行重新选择
          bGot = TRUE;
          break;
        }
        pCheckGoooNodeCodeMix = pCheckGoooNodeCodeMix->pNext;
      }

      // 如果不符合
      if (bGot == TRUE)
      {
        // 重新进行筛选
        pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
        pCurrNode = pCurrNode->pNext;
        continue;
      }/* end if */
    }

    // 找到合适的代码块
    // 分配空间
    *pCurrCodeReplaceNode = new CODEREPLACE_NODE;
    ZeroMemory((*pCurrCodeReplaceNode), sizeof(CODEREPLACE_NODE));
    (*pCurrCodeReplaceNode)->dwSize = dwBlockSize;
    (*pCurrCodeReplaceNode)->pFileAddress = pCodeFileAddress;
    DWORD dwRaw = PeDiy.Raw2Rva(pMem, (DWORD)(pCodeFileAddress - pMem));
    if (dwRaw == 0)
    {
      int i = i + 1;
    }
    (*pCurrCodeReplaceNode)->dwMemoryAddress = dwImageBase + dwRaw;
    (*pCurrCodeReplaceNode)->pFinalFileAddress = pCurrNode->pFileAddress;
    (*pCurrCodeReplaceNode)->dwFinalMemoryAddress = pCurrNode->dwMemoryAddress;
    (*pCurrCodeReplaceNode)->pReplaceFileAddress = pCurrStore;
    (*pCurrCodeReplaceNode)->dwReplaceMemoryAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCurrStore - pMem));
    DWORD dwFinalOffset = (*pCurrCodeReplaceNode)->dwReplaceMemoryAddress + dwBlockSize;
    dwFinalOffset -= ((*pCurrCodeReplaceNode)->dwFinalMemoryAddress);
    dwFinalOffset--;
    dwFinalOffset = ~dwFinalOffset;
    dwFinalOffset -= 0x05;//加上 JMP 指令的长度
    (*pCurrCodeReplaceNode)->dwFinalOffset = dwFinalOffset;
    BYTE JmpCodes[] = {"\xE9\xFF\xFF\xFF\xFF"};
    *(DWORD *)&(JmpCodes[1]) = dwFinalOffset;
    // 填充
    memcpy(pCurrStore, (*pCurrCodeReplaceNode)->pFileAddress, dwBlockSize);
    memcpy(pCurrStore + dwBlockSize, JmpCodes, 0x05);
    // 计算偏移并覆盖原指令
    DWORD dwReplaceOffset = (*pCurrCodeReplaceNode)->dwReplaceMemoryAddress - (*pCurrCodeReplaceNode)->dwMemoryAddress - 0x05;
    *(DWORD *)&(JmpCodes[1]) = dwReplaceOffset;
    memcpy((*pCurrCodeReplaceNode)->pFileAddress, JmpCodes, 0x05);
    // 产生花指令
    DWORD dwThunkCodeSize = dwBlockSize - 0x05;
    LPBYTE pThunkCode = new BYTE [dwThunkCodeSize];
    if (pThunkCode == NULL)
    {
      // 恢复
      memcpy((*pCurrCodeReplaceNode)->pFileAddress, pCurrStore, dwBlockSize);
      delete (*pCurrCodeReplaceNode);
      (*pCurrCodeReplaceNode) = NULL;
      FreeCodeFlow(&pCodeFlow);
      return pCodeReplaceHeader;
    }
    ZeroMemory(pThunkCode, dwThunkCodeSize);
    GenerateThunkCodeBySize(pJunkCodeBase, pThunkCode, dwThunkCodeSize);
    memcpy((*pCurrCodeReplaceNode)->pFileAddress + 0x05, pThunkCode, dwThunkCodeSize);//跳过第一个 JMP 指令
    delete [] pThunkCode;
    pCurrStore += (dwBlockSize + 0x05);//0x05 表示 JMP 指令的长度
    dwRemain -= (dwBlockSize + 0x05);

    // 是否需要添加代码替换保险,增大变形率,增加稳定性,但是也有可能造成错误
    if (pCodeReplaceConfigure->bCodeReplaceFinalJmp == TRUE)
    {
      // 利用反汇编器将替换后指令的长度计算出来
      ud_t ud_obj;
      ud_init(&ud_obj);
      ud_set_mode(&ud_obj, 32);
      ud_set_syntax(&ud_obj, UD_SYN_INTEL);
      ud_set_input_buffer(&ud_obj, (*pCurrCodeReplaceNode)->pReplaceFileAddress, (*pCurrCodeReplaceNode)->dwSize);
      // 设定开始的文件与内存指针
      LPBYTE pFileAddress = (*pCurrCodeReplaceNode)->pFileAddress + 0x05;
      DWORD dwMemoryAddress = (*pCurrCodeReplaceNode)->dwMemoryAddress + 0x05;
      ud_disassemble(&ud_obj);// 反汇编
      DWORD dwRemainReplaceSize = (*pCurrCodeReplaceNode)->dwSize - 0x05;//剩余空间按照原代码块位置计算,并以 JMP 指令长度递减
      // 如果这个长度小于 5 则跳过
      if (dwRemainReplaceSize >= 0x05)
      {
        DWORD dwRealMemoryAddress = (*pCurrCodeReplaceNode)->dwReplaceMemoryAddress + ud_obj.inp_ctr;
        LPBYTE pReadFileAddress = (*pCurrCodeReplaceNode)->pReplaceFileAddress + ud_obj.inp_ctr;
        DWORD dwOffsetOther = 0;
        while (dwRemainReplaceSize >= 0x05)
        {
          dwOffsetOther = dwRealMemoryAddress - dwMemoryAddress - 0x05;
          *(DWORD *)&(JmpCodes[1]) = dwOffsetOther;
          memcpy(pFileAddress, JmpCodes, 0x05);
          pFileAddress += 0x05;
          dwMemoryAddress += 0x05;
          pReadFileAddress += ud_disassemble(&ud_obj);
          dwRealMemoryAddress += ud_disassemble(&ud_obj);
          dwRemainReplaceSize -= 0x05;
        }/* end while */

        // 查看是否需要补齐花指令
        if (dwRemainReplaceSize != 0)
        {
          // 补齐花指令
          DWORD dwThunkCodeOtherSize = dwRemainReplaceSize;
          LPBYTE pThunkCodeOther = new BYTE [dwThunkCodeOtherSize];
          if (pThunkCodeOther != NULL)
          {
            ZeroMemory(pThunkCodeOther, dwThunkCodeOtherSize);
            GenerateThunkCodeBySize(pJunkCodeBase, pThunkCodeOther, dwThunkCodeOtherSize);
            memcpy(pFileAddress, pThunkCodeOther, dwThunkCodeOtherSize);
            delete [] pThunkCodeOther;
          }
        }/* end if */
    }/* end if */
    pCurrCodeReplaceNode = &((*pCurrCodeReplaceNode)->pNext);
    pCodeFileAddress = pCurrNode->pFileAddress + pCurrNode->dwInsLen;
    pCurrNode = pCurrNode->pNext;
  }/* end while */

  // 释放流程图空间
  if (pCodeReplaceConfigure->bCodeReplaceIntelligent == TRUE)
    FreeCodeFlow(&pIntelligentFlow);
  FreeCodeFlow(&pCodeFlow);
  return pCodeReplaceHeader;
}
```
