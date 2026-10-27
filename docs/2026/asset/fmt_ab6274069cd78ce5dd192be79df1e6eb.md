
```
/*
 * 预读分析阶段,在分区处理是时进行,数据与代码区域
 * 范围的界定,纯数据区域返回TRUE,反之为FALSE
 */

INLINE  BOOL PredictBlockEnd(LPBYTE pMem, LPBYTE pCurr, DWORD dwSize, DWORD *pOutSize, PANALYZE_CONFIGURE pAnalyzeConfigure)

{
  BOOL bBlock = FALSE;

  DWORD dwOffset = 0;

  ud_t ud_obj;

  ud_init(&ud_obj);

  ud_set_mode(&ud_obj, 32);

  ud_set_syntax(&ud_obj, UD_SYN_INTEL);

  ud_set_input_buffer(&ud_obj, pCurr, dwSize);

  while (ud_disassemble(&ud_obj) != 0)

  {

    enum ud_mnemonic_code mnemonic = ud_obj.mnemonic;

    if ((mnemonic == UD_Inop) ||

      (mnemonic == UD_Iint3) ||

      ((mnemonic == UD_Iadd) &&

      (ud_obj.inp_ctr == 2) &&

      (*(WORD *)&(ud_obj.inp_sess) == 0)))

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

    dwOffset += ud_insn_len(&ud_obj);

  }



  // 这里做深度鉴别

  bBlock = DeepAnalyzeBlock(pMem, pCurr, dwSize, pAnalyzeConfigure);

  *pOutSize = dwOffset;

  return bBlock;

}



/*

 * 进行分区处理

 * 以NOP,INT3,0,无效指令编码字符进行分区标志

 * 遇到NOP或者INT3,0,无效指令编码则将扫描过的区域

 * 作为一个区块摘出,继续查询,如果其后的数据有很长

 * 一段为连续的 NOP,INT3,0字节则将此视做绝对无效区域,

 * 在扫描过程中,如果遇到无效指令直接忽略当作这个区域

 * 的一部分如果遇到有效指令,则进入预分析阶段,尝试向前

 * 分析如果分析的指令长度小于最小代码长度则直接忽略.如果

 * 大于则将此区域继续分析,往返以上流程.直到完毕

 */

INLINE PPROCEDURE FormatBlock(LPBYTE pMem, LPBYTE pStart, DWORD dwSize, PANALYZE_CONFIGURE pAnalyzeConfigure)

{

  DWORD dwImageBase = GetNtHeader(pMem)->OptionalHeader.ImageBase;

  PPROCEDURE pBlock, *pCurrBlockPoint = NULL;

  pCurrBlockPoint = &pBlock;

  DWORD dwBlockEndSignCount = 0, dwOffset = 0;



  ud_t ud_obj;

  ud_init(&ud_obj);

  ud_set_mode(&ud_obj, 32);

  ud_set_syntax(&ud_obj, UD_SYN_INTEL);

  ud_set_input_buffer(&ud_obj, pStart, dwSize);



  while (ud_disassemble(&ud_obj) != 0)

  {

    enum ud_mnemonic_code mnemonic = ud_obj.mnemonic;

    if ((mnemonic == UD_Inop) ||

      (mnemonic == UD_Iint3) ||

      ((mnemonic == UD_Iadd) &&

      (ud_obj.inp_ctr == 2) &&

      (*(WORD *)&(ud_obj.inp_sess) == 0)))

    {

      dwBlockEndSignCount += ud_insn_len(&ud_obj);

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

           * 判断上一组区块标记是否为0如果不为则分配空间存储它

           */

          if (dwBlockEndSignCount != 0)

          {

            (*pCurrBlockPoint) = __new__(PROCEDURE, 1);

            memset((*pCurrBlockPoint), 0, sizeof(PROCEDURE));

            (*pCurrBlockPoint)->bBlock = TRUE;//代码区域

            (*pCurrBlockPoint)->pFileStartAddress = pCurr - dwBlockEndSignCount;

            (*pCurrBlockPoint)->dwMemoryStartAddress = dwImageBase + Raw2Rva(pMem, (DWORD)((*pCurrBlockPoint)->pFileStartAddress - pMem));

            (*pCurrBlockPoint)->dwSize = dwBlockEndSignCount;

            pCurrBlockPoint = &((*pCurrBlockPoint)->pNext);

            dwBlockEndSignCount = 0;//清空标志记录计数

          }

          // 设定此段为代码区域

          (*pCurrBlockPoint) = __new__(PROCEDURE, 1);

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

        ud_set_input_buffer(&ud_obj, pCurr, dwRemainSize);

        continue;

      }

      else

      {

        // 进入到这里就是进入到分析的末尾

        (*pCurrBlockPoint) = __new__(PROCEDURE, 1);

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

    dwOffset += ud_insn_len(&ud_obj);

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

    (*pCurrBlockPoint) = __new__(PROCEDURE, 1);

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

  PANALYZE_CONFIGURE pAnalyzeConfigure = &(pParents->AnalyzeConfigure);

  DWORD dwImageBase = GetNtHeader(pMem)->OptionalHeader.ImageBase;

  // 如果此块的长度小于用户指定长度

  if (dwSize < pAnalyzeConfigure->bCodeMixSize)

  {

    PPROCEDURE pBlock = __new__(PROCEDURE, 1);

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

       * 调用Procedure2Procedure过后形成的函数链中存在两种函数,一种是函数存在在已有的

       * 未知区域中,一种是在做第一次扫描时分析到的函数

       */

      DWORD dwCount = Procedure2Procedure(pMem, pProcedureList, pProcedure);

    }

    else

    {

      // 直接连接到主链

      PPROCEDURE *pCurrMainPoint = &pProcedureList;

      while (*pCurrMainPoint != NULL) pCurrMainPoint = &((*pCurrMainPoint)->pNext);

      *pCurrMainPoint = __new__(PROCEDURE, 1);

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



  g_pAnalyzeDataDispatcher = MakeAnalyzeDataFromInstructionDispatcher();



_new_analyze:

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

      goto _handler;

    }

    pCurrProcedure = pCurrProcedure->pNext;

  }



  // 判断是否到达末尾

  if (dwRemainSize != 0)

  {

    dwBlockSize = dwRemainSize;

    goto _handler;

  }



  // 释放派遣表

  DestroyDispatcherList(&g_pAnalyzeDataDispatcher);

  return pProcedureList;



// 这里为处理函数所在

_handler:

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

  __PrintProcedureListCount__(pProcedureList);

  goto _new_analyze;//重新进入新的一次分析

}
```
