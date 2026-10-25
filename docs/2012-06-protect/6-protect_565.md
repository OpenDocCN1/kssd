
# 软件保护壳专题 - 代码乱序引擎的构建

> 原文：[`www.pediy.com/kssd/pediy10/96640.html`](https://www.pediy.com/kssd/pediy10/96640.html)

## 目录

0. `<什么是乱序>`
1. `<乱序的步骤>`
2. `<代码流程图的构建>`
3. `<开始乱序>`
4. `<YY>`

## 正文

### 0. 什么是乱序

乱序就是打乱原来的流程。

很多朋友逆向程序时，会把一个程序直接丢到 `IDA` 里会出现一副流程图。我们现在要做的就是把这幅流程图打乱。但是并不影响原先流程的逻辑。乱序的原理其实比较简单。说白了就是做 `HOOK` 然后填充花指令。最后在跳转会原先的地址。

### 1. 乱序的步骤

0. 把正常代码流程的 `CALL`/`JMP`/`JCC` 的指令，全部都找出来。
1. 对上述进行的位置，进行筛选，如果是 8 位跳转直接过滤，如果是 16 位跳转计算偏移空间是否足够跳转到花指令的偏移。如果是 32 位的情况直接发通行证。
2. 添加一个新节或者利用其它感染方式增加一块可用的区域做存储花指令用。
3. 遍历 0 和 1 生存的结构。当遇到有偏移时。记录原先要跳转的地方。将原先的偏移替换成花指令空间的位置。最后在花指令末尾添加一个跳入到原先跳转地址的偏移（一般都是向上跳，新增的区域一般处于最后的内存空间）。
4. 验证花指令的剩余空间是否足够，以便判断是否停止 `HOOK`。

当然你也可以加入一些随机判断，某条跳转是否进行乱序。有些则不用。

### 2. 代码流程图的构建

原理很简单，利用反汇编引擎，开始遍历要乱序的地址和长度。一步一步的找出所有合适的跳转类型指令。

代码如下

```
代码:
```



```cpp
// 这个是流程图的结构,因为用不到动态转载,所以只记录偏移性质的跳转
typedef struct _CODE_FLOW_NODE
{
  struct _CODE_FLOW_NODE *pNext;//下一个节点
  BOOL bGoDown;//是否向下跳
  DWORD dwBits;//跳转范围
  DWORD dwType;//指令类型
  BOOL bFar;//是否是远跳
  DWORD dwMemoryAddress;//当前内存地址
  LPBYTE pFileAddress;//当前文件地址
  DWORD dwGotoMemoryAddress;//跳转后的内存地址
  LPBYTE pGotoFileAddress;//跳转后的文件地址
  DWORD dwInsLen;//指令长度
  union
  {
    BYTE bOffset;
    WORD wOffset;
    DWORD dwOffset;
  };//偏移
} CODE_FLOW_NODE, *PCODE_FLOW_NODE;

// 下面的代码利用了 udis86 反汇编引擎。个人觉的不错。尤其是直接做反汇编器使用时
// CxPeDiy 是定义的一个操作 PE 文件的集合类。做一些 PE 结构操作。自己动手吧。
// 代码不算复杂。如果你有个大屏幕的话看起来会简单些。如果像我一样在一台 T61 上
// 工作。那实在有些不便了

PCODE_FLOW DrawCodeFlow(CONST LPBYTE pMem, CONST LPBYTE pStart, DWORD dwSize)
{
  PCODE_FLOW pCodeFlow = new CODE_FLOW;
  if (pCodeFlow == NULL)
    return NULL;
  ZeroMemory(pCodeFlow, sizeof(CODE_FLOW));

  CxPeDiy PeDiy;
  // 获取一些基本信息
  DWORD dwImageBase = PeDiy.GetNtHeader(pMem)->OptionalHeader.ImageBase;

  // 初始化反汇编引擎
  ud_t ud_obj;
  ud_init(&ud_obj);
  ud_set_input_buffer(&ud_obj, pStart, dwSize);
  ud_set_mode(&ud_obj, 32);
  ud_set_syntax(&ud_obj, UD_SYN_INTEL);
  if (pCodeFlow->ErrDisassembleAddress.Init(0x1000) == FALSE)
    return NULL;

  PCODE_FLOW_NODE pCodeFlowHeader = NULL, *pCodeFlowNode = &pCodeFlowHeader;

  LPBYTE pCurr = pStart;
  while (ud_disassemble(&ud_obj) != 0)
  {
    // printf("\t%s\n", ud_insn_asm(&ud_obj));
    if (ud_obj.mnemonic == UD_Iinvalid)
    {
      // 反汇编出现错误
      pCodeFlow->ErrDisassembleAddress.PutIntoQueue((QUEUE_ELEMENT)pCurr);
    }
    else
    {
      // 判断是否是跳转地址
      switch (ud_obj.mnemonic)
      {
      case UD_Ijo:
      case UD_Ijno:
      case UD_Ijb:
      case UD_Ijae:
      case UD_Ijz:
      case UD_Ijnz:
      case UD_Ijbe:
      case UD_Ija:
      case UD_Ijs:
      case UD_Ijns:
      case UD_Ijp:
      case UD_Ijnp:
      case UD_Ijl:
      case UD_Ijge:
      case UD_Ijle:
      case UD_Ijg:
      case UD_Ijcxz:
      case UD_Ijecxz:
      case UD_Ijrcxz:
        {
          // 分配节点内存
          *pCodeFlowNode = new CODE_FLOW_NODE;
          (*pCodeFlowNode)->bFar = FALSE;
          DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
          (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
          (*pCodeFlowNode)->pFileAddress = pCurr;
          (*pCodeFlowNode)->dwType = JmpIns_Type_Jcc;
          (*pCodeFlowNode)->pNext = NULL;

          // 判断是否有前缀
          if (ud_obj.pfx_opr == 0x66)
          {
            // 16 位
            (*pCodeFlowNode)->dwBits = Jmp_Bit_16;
            (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
            WORD wOffset = ud_obj.operand[0].lval.uword;
            (*pCodeFlowNode)->wOffset = wOffset;
            if (wOffset >= 0x8000)
            {
              (*pCodeFlowNode)->bGoDown = FALSE;
              wOffset = ~wOffset;
              wOffset++;
              wOffset -= ud_obj.inp_ctr;
              (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - wOffset;
              DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
              (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
            }
            else
            {
              (*pCodeFlowNode)->bGoDown = TRUE;
              wOffset += ud_obj.inp_ctr;
              (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + wOffset;
              DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
              (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
            }/* end else */
          }
          else
          {
            if (ud_obj.inp_sess[0] == 0x0F)
            {
              // 32 位
              (*pCodeFlowNode)->dwBits = Jmp_Bit_32;
              (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
              DWORD dwOffset = ud_obj.operand[0].lval.udword;
              (*pCodeFlowNode)->dwOffset = dwOffset;
              if (dwOffset >= 0x80000000)
              {
                (*pCodeFlowNode)->bGoDown = FALSE;
                dwOffset = ~dwOffset;
                dwOffset++;
                dwOffset -= ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - dwOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }
              else
              {
                (*pCodeFlowNode)->bGoDown = TRUE;
                dwOffset += ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + dwOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }/* end else */
            }
            else
            {
              // 8 位
              (*pCodeFlowNode)->dwBits = Jmp_Bit_8;
              (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
              BYTE bOffset = ud_obj.operand[0].lval.ubyte;
              (*pCodeFlowNode)->bOffset = bOffset;
              if (bOffset >= 0x80)
              {
                (*pCodeFlowNode)->bGoDown = FALSE;
                bOffset = ~bOffset;
                bOffset++;
                bOffset -= ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - bOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }
              else
              {
                (*pCodeFlowNode)->bGoDown = TRUE;
                bOffset += ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + bOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }/* end else */
            }
          }/* end else */
          pCodeFlowNode = &((*pCodeFlowNode)->pNext);
        }break;
      case UD_Ijmp:
        {          
          if (ud_obj.inp_sess[0] == 0xEB)
          {
            // 8 位
            // 分配节点内存
            *pCodeFlowNode = new CODE_FLOW_NODE;
            (*pCodeFlowNode)->bFar = FALSE;
            DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
            (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
            (*pCodeFlowNode)->pFileAddress = pCurr;
            (*pCodeFlowNode)->dwType = JmpIns_Type_Jmp;
            (*pCodeFlowNode)->dwBits = Jmp_Bit_8;
            (*pCodeFlowNode)->pNext = NULL;
            BYTE bOffset = ud_obj.operand[0].lval.ubyte;
            (*pCodeFlowNode)->bOffset = bOffset;
            (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
            if (bOffset >= 0x80)
            {
              (*pCodeFlowNode)->bGoDown = FALSE;
              bOffset = ~bOffset;
              bOffset++;
              bOffset -= ud_obj.inp_ctr;
              (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - bOffset;
              DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
              (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
            }
            else
            {
              (*pCodeFlowNode)->bGoDown = TRUE;
              bOffset += ud_obj.inp_ctr;
              (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + bOffset;
              DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
              (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
            }/* end else */
            pCodeFlowNode = &((*pCodeFlowNode)->pNext);
          }
          else
          {
            if ((ud_obj.pfx_opr == 0x66) || (ud_obj.pfx_adr == 0x67))
            {
              if (ud_obj.inp_sess[1] == 0xE9)
              {
                // 16 位
                // 分配节点内存
                *pCodeFlowNode = new CODE_FLOW_NODE;
                (*pCodeFlowNode)->bFar = FALSE;
                DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
                (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
                (*pCodeFlowNode)->pFileAddress = pCurr;
                (*pCodeFlowNode)->dwType = JmpIns_Type_Jmp;
                (*pCodeFlowNode)->dwBits = Jmp_Bit_16;
                (*pCodeFlowNode)->pNext = NULL;
                WORD wOffset = ud_obj.operand[0].lval.uword;
                (*pCodeFlowNode)->wOffset = wOffset;
                (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
                if (wOffset >= 0x8000)
                {
                  (*pCodeFlowNode)->bGoDown = FALSE;
                  wOffset = ~wOffset;
                  wOffset++;
                  wOffset -= ud_obj.inp_ctr;
                  (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - wOffset;
                  DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                  (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
                }
                else
                {
                  (*pCodeFlowNode)->bGoDown = TRUE;
                  wOffset += ud_obj.inp_ctr;
                  (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + wOffset;
                  DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                  (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
                }/* end else */
                pCodeFlowNode = &((*pCodeFlowNode)->pNext);
              }/* end if */
            }
            else
            {
              if (ud_obj.inp_sess[0] == 0xE9)
              {
                // 32 位
                // 分配节点内存
                *pCodeFlowNode = new CODE_FLOW_NODE;
                (*pCodeFlowNode)->bFar = FALSE;
                DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
                (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
                (*pCodeFlowNode)->pFileAddress = pCurr;
                (*pCodeFlowNode)->dwBits = Jmp_Bit_32;
                (*pCodeFlowNode)->dwType = JmpIns_Type_Jmp;
                (*pCodeFlowNode)->pNext = NULL;
                DWORD dwOffset = ud_obj.operand[0].lval.udword;
                (*pCodeFlowNode)->dwOffset = dwOffset;
                (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
                if (dwOffset >= 0x80000000)
                {
                  (*pCodeFlowNode)->bGoDown = FALSE;
                  dwOffset = ~dwOffset;
                  dwOffset++;
                  dwOffset -= ud_obj.inp_ctr;
                  (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - dwOffset;
                  DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                  (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
                }
                else
                {
                  (*pCodeFlowNode)->bGoDown = TRUE;
                  dwOffset += ud_obj.inp_ctr;
                  (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + dwOffset;
                  DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                  (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
                }/* end else */
                pCodeFlowNode = &((*pCodeFlowNode)->pNext);
              }
            }/* end else */
          }/* end else */
        }break;
      case UD_Icall:
        {
          if ((ud_obj.pfx_opr == 0x66) || (ud_obj.pfx_adr == 0x67))
          {
            if (ud_obj.inp_sess[1] == 0xE8)
            {
              // 16 位
              // 分配节点内存
              *pCodeFlowNode = new CODE_FLOW_NODE;
              (*pCodeFlowNode)->bFar = FALSE;
              DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
              (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
              (*pCodeFlowNode)->pFileAddress = pCurr;
              (*pCodeFlowNode)->pNext = NULL;
              (*pCodeFlowNode)->dwType = JmpIns_Type_Call;
              (*pCodeFlowNode)->dwBits = Jmp_Bit_16;
              WORD wOffset = ud_obj.operand[0].lval.uword;
              (*pCodeFlowNode)->wOffset = wOffset;
              (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
              if (wOffset >= 0x8000)
              {
                (*pCodeFlowNode)->bGoDown = FALSE;
                wOffset = ~wOffset;
                wOffset++;
                wOffset -= ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - wOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }
              else
              {
                (*pCodeFlowNode)->bGoDown = TRUE;
                wOffset += ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + wOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }/* end else */
              pCodeFlowNode = &((*pCodeFlowNode)->pNext);
            }/* end if */
          }
          else
          {
            if (ud_obj.inp_sess[0] == 0xE8)
            {
              // 32 位
              // 分配节点内存
              *pCodeFlowNode = new CODE_FLOW_NODE;
              (*pCodeFlowNode)->bFar = FALSE;
              DWORD dwRVA = PeDiy.Raw2Rva(pMem, (DWORD)(pCurr - pMem));
              (*pCodeFlowNode)->dwMemoryAddress = dwImageBase + dwRVA;
              (*pCodeFlowNode)->pFileAddress = pCurr;
              (*pCodeFlowNode)->dwType = JmpIns_Type_Call;
              (*pCodeFlowNode)->dwBits = Jmp_Bit_32;
              (*pCodeFlowNode)->pNext = NULL;
              DWORD dwOffset = ud_obj.operand[0].lval.udword;
              (*pCodeFlowNode)->dwOffset = dwOffset;
              (*pCodeFlowNode)->dwInsLen = ud_obj.inp_ctr;
              if (dwOffset >= 0x80000000)
              {
                (*pCodeFlowNode)->bGoDown = FALSE;
                dwOffset = ~dwOffset;
                dwOffset++;
                dwOffset -= ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress - dwOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }
              else
              {
                (*pCodeFlowNode)->bGoDown = TRUE;
                dwOffset += ud_obj.inp_ctr;
                (*pCodeFlowNode)->dwGotoMemoryAddress = (*pCodeFlowNode)->dwMemoryAddress + dwOffset;
                DWORD dwRaw = PeDiy.Rva2Raw(pMem, (DWORD)((*pCodeFlowNode)->dwGotoMemoryAddress - dwImageBase));
                (*pCodeFlowNode)->pGotoFileAddress = pMem + dwRaw;
              }/* end else */
              pCodeFlowNode = &((*pCodeFlowNode)->pNext);
            }/* end if */             
          }/* end else */
        }break;
      }/* end switch */
    }/* end else */
    pCurr += ud_obj.inp_ctr;
  }/* end if */
  pCodeFlow->pCodeFlow = pCodeFlowHeader;
  return pCodeFlow;
}
```



### 3. 开始乱序

现在有了以上这个结构，我们就可以开始乱序了。乱序同样也需要一个结构，用来记录要跳转的地点、原先的偏移等等，以便以后有需要做更复杂变换的时候使用。

代码:



```c
// 就是这个结构了。记录乱序后的情况，先记录下来。以后想做更复杂的变化就需要了。
typedef struct _CODEMIX_NODE
{
  struct _CODEMIX_NODE *pNext;//下一个节点
  DWORD dwMemoryAddress;//当前内存地址
  LPBYTE pFileAddress;//当前文件地址
  DWORD dwOrigGotoMemoryAddress;//原跳转后的内存地址
  LPBYTE pOrigGotoFileAddress;//原跳转后的文件地址
  DWORD dwGotoMemoryAddress;//当前跳转后的内存地址
  LPBYTE pGotoFileAddress;//当前跳转后的文件地址
  DWORD dwFinalMemoryAddress;//最终跳转的内存地址
  LPBYTE pFinalFileAddress;//最终跳转的文件地址
  DWORD dwBits;//跳转范围
  union
  {
    WORD wOffset;
    DWORD dwOffset;//跳入花指令的偏移
  };
  DWORD dwFinalOffset;//最终跳转的偏移
} CODEMIX_NODE, *PCODEMIX_NODE;

// 这里我也不说什么了。代码是最说明问题的。
// 其中的 PTHUNKCODE_CONFIGURE 是产生花指令用到的。自己构建吧。思路可以参照专题的变形引擎的构建。^\_^

PCODEMIX_NODE DrawCodeMixFlow(CONST LPBYTE pMem, CONST LPBYTE pStore, PTHUNKCODE_CONFIGURE pThunkConfigure, PCODEMIX_CONFIGURE pCodeMixConfigure, PJUNKCODE_BASE pJunkCodeBase)
{
  CxPeDiy PeDiy;
  DWORD dwImageBase = PeDiy.GetNtHeader(pMem)->OptionalHeader.ImageBase;
  // 生成流程图
  LPBYTE pCodeMixFileStart = pMem + PeDiy.Rva2Raw(pMem, (DWORD)(pCodeMixConfigure->dwStartAddress - dwImageBase));
  PCODE_FLOW pCodeFlow = DrawCodeFlow(pMem, pCodeMixFileStart, pCodeMixConfigure->dwSize);
  if (pCodeFlow == NULL)
    return NULL;
  PCODE_FLOW_NODE pCurrNode = pCodeFlow->pCodeFlow;
  LPBYTE pCurrStore = pStore;
  DWORD dwRemainSize = pCodeMixConfigure->dwSpaceSize;

  PCODEMIX_NODE pCodeMixHeader = NULL, *pCurrCodeMixNode = &pCodeMixHeader;

  while ((pCurrNode != NULL) && (dwRemainSize >= 0x05))//剩余字节至少要等于一个 JMP 指令的字节
  {
    // 计算是否这个跳转可以 Mix
    BOOL bCanMix = FALSE;
    if (pCurrNode->dwBits == Jmp_Bit_16)
    {
      // 计算
      DWORD dwStoreMemoryAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCurrStore - pMem));//储存空间的内存地址
      DWORD dwDifference = dwStoreMemoryAddress - pCurrNode->dwMemoryAddress;
      if (dwDifference <= 0xFFFF) bCanMix = TRUE;
      if (bCanMix == TRUE)
      {
        // 查看生成率是否进行 MIX
        if (RandomRoll(pCodeMixConfigure->RandomArray) == TRUE)
        {
          // 分配内存
          *pCurrCodeMixNode = new CODEMIX_NODE;
          if ((*pCurrCodeMixNode) == NULL);//出错
          ZeroMemory((*pCurrCodeMixNode), sizeof(CODEMIX_NODE));
          (*pCurrCodeMixNode)->dwBits = Jmp_Bit_16;
          (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress = pCurrNode->dwGotoMemoryAddress;
          (*pCurrCodeMixNode)->pOrigGotoFileAddress = pCurrNode->pGotoFileAddress;
          (*pCurrCodeMixNode)->dwMemoryAddress = pCurrNode->dwMemoryAddress;
          (*pCurrCodeMixNode)->pFileAddress = pCurrNode->pFileAddress;
          (*pCurrCodeMixNode)->pGotoFileAddress = pCurrStore;
          (*pCurrCodeMixNode)->dwGotoMemoryAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCurrStore - pMem));;
          // 计算偏移,跳转地址 - 原地址 - 指令长度
          (*pCurrCodeMixNode)->wOffset = (WORD)((*pCurrCodeMixNode)->dwGotoMemoryAddress - (*pCurrCodeMixNode)->dwMemoryAddress - pCurrNode->dwInsLen);
          // 重新设置偏移
          *(WORD *)(pCurrNode->pFileAddress + pCurrNode->dwInsLen - 0x02) = (*pCurrCodeMixNode)->wOffset;
          // 由花指令生成器生成一块花指令,由花指令最大长度 + JMP 指令长度(5)
          DWORD dwThunkCodeSize = 0, dwThunkCodeBufSize = pThunkConfigure->dwMaxSize + 0x05;
          LPBYTE pThunkCode = new BYTE [dwThunkCodeBufSize];
          if (pThunkCode == NULL)
          {
            delete (*pCurrCodeMixNode);
            (*pCurrCodeMixNode) = NULL;
            break;
          }
          ZeroMemory(pThunkCode, dwThunkCodeBufSize);

          // 检查花指令形成的最小长度与剩余空间长度
          if (pThunkConfigure->dwMinSize < dwRemainSize)
          {
            dwThunkCodeSize = GenerateThunkCode(pJunkCodeBase, pThunkCode, dwThunkCodeBufSize, pThunkConfigure);
            while ((dwThunkCodeSize + 0x05) > dwRemainSize) dwThunkCodeSize = GenerateThunkCode(pJunkCodeBase, pThunkCode, dwThunkCodeBufSize, pThunkConfigure);
          }
          else
          {
            lsrand((unsigned)time(NULL));
            dwThunkCodeSize = (DWORD)(lrand() % (dwRemainSize + 1 - 0x05));//减去最后 JMP 的大小
            dwThunkCodeSize = GenerateThunkCodeBySizeByRandom(pJunkCodeBase, pThunkCode, dwThunkCodeSize, pThunkConfigure->RandomArray);
          }

          // 在花指令末尾添加跳入真正的地址,E9 XX XX XX XX 5 个字节
          BYTE JmpCodes[] = {"\xE9\xFF\xFF\xFF\xFF"};
          // 计算跳转的偏移, 由于是向上跳, 计算偏移 = (原地址 - 跳转地址 - 1) 求反
          DWORD dwJmpOffset = 0, dwSourceAddress = (*pCurrCodeMixNode)->dwGotoMemoryAddress + dwThunkCodeSize;//得到花指令末尾的内存地址
          if (dwSourceAddress > (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress)
          {
            dwJmpOffset = dwSourceAddress - (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress;
            dwJmpOffset--;
            dwJmpOffset = ~dwJmpOffset;
            dwJmpOffset -= 0x05;//减去最后 JMP 指令的 5 个字节得到偏移;
          }
          else
          {
            dwJmpOffset = (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress - dwSourceAddress - 0x05;//减去最后的 JMP 指令的 5 个字节得到偏移
          }
          (*pCurrCodeMixNode)->dwFinalOffset = dwJmpOffset;
          *(DWORD *)&(JmpCodes[1]) = dwJmpOffset;
          memcpy(pThunkCode + dwThunkCodeSize, JmpCodes, 0x05);
          dwThunkCodeSize += 0x05;//加上末尾的 JMP 指令长度
          memcpy(pCurrStore, pThunkCode, dwThunkCodeSize);
          pCurrStore += dwThunkCodeSize;
          dwRemainSize -= dwThunkCodeSize;
          delete [] pThunkCode;
          (*pCurrCodeMixNode)->pNext = NULL;
          pCurrCodeMixNode = &((*pCurrCodeMixNode)->pNext);
        }/* end if */
      }/* end if */
    }
    else if (pCurrNode->dwBits == Jmp_Bit_32)
    {
      // 如果是 32 位直接就是允许 Mix
      bCanMix = TRUE;
      if (bCanMix == TRUE)
      {
        // 查看生成率是否进行 MIX
        if (RandomRoll(pCodeMixConfigure->RandomArray) == TRUE)
        {
          // 分配内存
          *pCurrCodeMixNode = new CODEMIX_NODE;
          if ((*pCurrCodeMixNode) == NULL);//出错
          ZeroMemory((*pCurrCodeMixNode), sizeof(CODEMIX_NODE));
          (*pCurrCodeMixNode)->dwBits = Jmp_Bit_32;
          (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress = pCurrNode->dwGotoMemoryAddress;
          (*pCurrCodeMixNode)->pOrigGotoFileAddress = pCurrNode->pGotoFileAddress;
          (*pCurrCodeMixNode)->dwMemoryAddress = pCurrNode->dwMemoryAddress;
          (*pCurrCodeMixNode)->pFileAddress = pCurrNode->pFileAddress;
          (*pCurrCodeMixNode)->pGotoFileAddress = pCurrStore;
          (*pCurrCodeMixNode)->dwGotoMemoryAddress = dwImageBase + PeDiy.Raw2Rva(pMem, (DWORD)(pCurrStore - pMem));
          // 计算偏移,跳转地址 - 原地址 - 指令长度
          (*pCurrCodeMixNode)->dwOffset = (DWORD)((*pCurrCodeMixNode)->dwGotoMemoryAddress - (*pCurrCodeMixNode)->dwMemoryAddress - pCurrNode->dwInsLen);
          // 重新设置偏移
          *(DWORD *)(pCurrNode->pFileAddress + pCurrNode->dwInsLen - 0x04) = (*pCurrCodeMixNode)->dwOffset;
          // 由花指令生成器生成一块花指令,由花指令最大长度 + JMP 指令长度(5)
          DWORD dwThunkCodeBufSize = pThunkConfigure->dwMaxSize + 0x05;
          LPBYTE pThunkCode = new BYTE [dwThunkCodeBufSize];
          if (pThunkCode == NULL)
          {
            delete (*pCurrCodeMixNode);
            (*pCurrCodeMixNode) = NULL;
            break;
          }
          // 检查花指令形成的最小长度与剩余空间长度
          DWORD dwThunkCodeSize;
          if (pThunkConfigure->dwMinSize < dwRemainSize)
          {
            dwThunkCodeSize = GenerateThunkCode(pJunkCodeBase, pThunkCode, dwThunkCodeBufSize, pThunkConfigure);
            while ((dwThunkCodeSize + 0x05) > dwRemainSize) dwThunkCodeSize = GenerateThunkCode(pJunkCodeBase, pThunkCode, dwThunkCodeBufSize, pThunkConfigure);
          }
          else
          {
            lsrand((unsigned)time(NULL));
            dwThunkCodeSize = (DWORD)(lrand() % (dwRemainSize + 1 - 0x05));//减去最后 JMP 的大小
            dwThunkCodeSize = GenerateThunkCodeBySizeByRandom(pJunkCodeBase, pThunkCode, dwThunkCodeSize, pThunkConfigure->RandomArray);
          }

          // 在花指令末尾添加跳入真正的地址,E9 XX XX XX XX 5 个字节
          BYTE JmpCodes[] = {"\xE9\xFF\xFF\xFF\xFF"};
          // 计算跳转的偏移, 由于是向上跳, 计算偏移 = (原地址 - 跳转地址 - 1) 求反
          DWORD dwJmpOffset = 0, dwSourceAddress = (*pCurrCodeMixNode)->dwGotoMemoryAddress + dwThunkCodeSize;//得到花指令末尾的内存地址
          if (dwSourceAddress > (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress)
          {
            dwJmpOffset = dwSourceAddress - (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress;
            dwJmpOffset--;
            dwJmpOffset = ~dwJmpOffset;
            dwJmpOffset -= 0x05;//减去最后 JMP 指令的 5 个字节得到偏移;
          }
          else
          {
            dwJmpOffset = (*pCurrCodeMixNode)->dwOrigGotoMemoryAddress - dwSourceAddress - 0x05;//减去最后的 JMP 指令的 5 个字节得到偏移
          }
          (*pCurrCodeMixNode)->dwFinalOffset = dwJmpOffset;
          *(DWORD *)&(JmpCodes[1]) = dwJmpOffset;
          memcpy(pThunkCode + dwThunkCodeSize, JmpCodes, 0x05);
          dwThunkCodeSize += 0x05;//加上 JMP 指令的长度
          memcpy(pCurrStore, pThunkCode, dwThunkCodeSize);
          pCurrStore += dwThunkCodeSize;
          dwRemainSize -= dwThunkCodeSize;
          delete [] pThunkCode;
          (*pCurrCodeMixNode)->pNext = NULL;
          pCurrCodeMixNode = &((*pCurrCodeMixNode)->pNext);
        }/* end if */
      }/* end if */
    }
    pCurrNode = pCurrNode->pNext;
  }
  FreeCodeFlow(&pCodeFlow);
  return pCodeMixHeader;
}
```



### 4. YY

这篇文章写不太多。去掉代码没几行文字了。不过代码还算完整。喜欢做壳的但又不太了解这个技术的朋友。可以参照这个代码向自己的壳里添加这个功能。如果做好的话是比较猥琐的一个功能。这个技巧也不是不能破除。但是可以让`Cracker`花费更多的精力。希望大家喜欢。。。

`![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")`
