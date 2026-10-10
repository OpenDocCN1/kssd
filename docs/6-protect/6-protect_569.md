

# 软件保护壳技术专题 - 使用数学运算式随机生成花指令

> 原文：[`www.pediy.com/kssd/pediy10/103229.html`](https://www.pediy.com/kssd/pediy10/103229.html)

![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")  
花指令应该是壳中最常用的防护手法。自然生成随机花指令也是重要的手段之一。在这个专题中的有一篇是讨论变形引擎的构建。里面列出了一些根据模板产生花指令的,随机主要出现在寄存器与几个固定模板。  
这篇文章探讨了使用数学计算公式来构建花指令。这样生成的花指令更像一个有用的算法。并且不用担心更新模板的问题。随机数学公式的就是最好的模板。  
有两个步骤：  
1.使用随机手段生成一个计算公式的模板  
例如:  
EXP = ~(7612CB \* 123 - 98BB) ^ (111BBC \\ 98BAC mod 5678)  
2.对 EXP 进行汇编  
这个操作其实也有一定模板话的程序，不过话说每个编译器生成代码的时候都有自已的一套模板。当然这里的算一个小型汇编器。如果将它做扩展应该可以生成很多有趣的代码。  
  
![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")  
生成表达式算法:  
首先我将表达式分为主动表达式,被动表达式,子表达式  
每个主动表达式(操作符号的前者)允许拥有一个被动表达式或者子表达式,或者没有  
被动表达式(操作符号的后者)允许拥有一个子表达式或者没有,但是它不允许再有被动表达式  
子表达式用于扩展当前表达式  
最后形式是一个 2 叉树的结构  
                                          exp  
                    exp1               op               exp2  
      exp1\_1   op   exp1\_2       exp2\_1     op     exp2\_2  
结构如下  

代码:

```
// 数学计算表达式节点
typedef struct \_MATHEXP
{
  struct \_MATHEXP \*pPassivityExp;//被动表达式
  struct \_MATHEXP \*pSubExp;//子表达式

  BOOL bNot;//是否进行 NOT 操作
  MATH\_OPT Opt;//操作符
  DWORD dwVal;//操作数
} MATHEXP, \*PMATHEXP;

```

使用这个树再进行汇编操作。  
以下是代码，代码比较全。不过目前形式还比较单调。如果愿意可以发挥自己的想象将他扩充。  
对于 2 叉树的操作全部采用了深度优先算法。并且方向是从右到左。这和计算式的优先级有关系  
生成表达式与汇编操作的优先级为:被动表达式 > 子表达式 > 主动表达式  
随机算法这里没有给出。  
如果有需要的朋友，可以自己编写随机值算法。随机算法一定要生成的平均，否则在生成表达式时可以造成无限生成。  
![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")  
代码:

```
/\*
 \* 只产生一层根表达式模型,不决定子表达式是什么
 \* 只决定运算符与分配主副表达式结构的内存
 \* 生成后,如果有子表达式则无常量值,无子表达式则
 \* 一定有常量值
 \* 被动表达式不能再有被动表达式,只能拥有子表达式
 \*/
#define \_\_CreateMathExp\_\_(pMathExp)\\
  (pMathExp) = \_\_new\_\_(MATHEXP, 1);\\
  memset((pMathExp), 0, sizeof(MATHEXP));

#define \_\_GenerateMathOpt\_\_()        (MATH\_OPT)(GenerateRandomNumber() % MAX\_MATH\_OPT + 1)
#define \_\_GenerateMathOptWithOutNot\_\_()    (MATH\_OPT)(GenerateRandomNumber() % (MAX\_MATH\_OPT-1) + 1)
#define \_\_HasPassMathExp\_\_()        (RandRoll(g\_MathExpGenRollArray))
#define \_\_HasSubMathExp\_\_()          (RandRoll(g\_MathExpGenRollArray))
PMATHEXP  GenerateMathRootExp()
{
  PMATHEXP pMathExp = NULL;
  \_\_CreateMathExp\_\_(pMathExp);

  // 是否存在被动表达式
  if (\_\_HasPassMathExp\_\_() == TRUE)//有被动表达式
  {
    /\*
     \* 随机选定一个操作符
     \* 如果是 NOT 操作符设定后
     \* 重新生成操作符
     \*/
    MATH\_OPT Opt = \_\_GenerateMathOpt\_\_();
    if (Opt == MATH\_NOT)
    {
      pMathExp->bNot = TRUE;
      Opt = \_\_GenerateMathOptWithOutNot\_\_();
      pMathExp->Opt = Opt;
    }
    else
    {
      pMathExp->Opt = Opt;
    }
    // 进入生成被动表达式
    \_\_CreateMathExp\_\_(pMathExp->pPassivityExp);
    // 被动表达式是否需要 NOT 操作符
    Opt = \_\_GenerateMathOpt\_\_();
    if (Opt == MATH\_NOT)
    {
      pMathExp->pPassivityExp->bNot = TRUE;
    }
  }
  else//没有被动表达式
  {
    MATH\_OPT Opt = \_\_GenerateMathOpt\_\_();
    if (Opt == MATH\_NOT)
    {
      pMathExp->bNot = TRUE;
    }
  }
  return pMathExp;
}

/\*
 \* 产生子数学计算表达式链
 \* 这里采用宽度优先算法,先生成子表达式
 \*/
PMATHEXP  GenerateMathExp()
{
  PMATHEXP pMathExp = GenerateMathRootExp();//产生一个根
  PMATHEXP pPassMathExp = pMathExp->pPassivityExp;
  /\*
   \* 决定主表达式是否有子表达式
   \* 如果没有子表达式则随机选定一个常数
   \*/
  if (\_\_HasSubMathExp\_\_() == TRUE)
  {
    pMathExp->pSubExp = GenerateMathExp();
  }
  else
  {
    pMathExp->dwVal = GenerateRandomNumber();//随机选定一个常数
  }

  // 生成被动表达式的子表达式
  if (pPassMathExp != NULL)
  {
    if (\_\_HasSubMathExp\_\_() == TRUE)
    {
      pPassMathExp->pSubExp = GenerateMathExp();
    }
    else
    {
      pPassMathExp->dwVal = GenerateRandomNumber();//随机选定一个常数
    }
  }
  return pMathExp;
}

/\*
 \* 删除节点采用深度优先算法
 \*/
VOID  ReleaseMathExp(PMATHEXP \*pMathExpPoint)
{
  PMATHEXP pCurr = \*pMathExpPoint, pPassMathExp = pCurr->pPassivityExp;
  if (pCurr->pSubExp == NULL)
  {
    \_\_delete\_\_(pCurr);
  }
  else
  {
    ReleaseMathExp(&(pCurr->pSubExp));
  }

  // 删除被动表达式
  if (pPassMathExp != NULL)
  {
    ReleaseMathExp(&pPassMathExp);
  }

  \*pMathExpPoint = NULL;
  return;
}

INLINE ASM\_BLOCK\_TYPE  ReturnAsmBlockType(PMATHEXP pMathExp)
{
  if (pMathExp->bNot == TRUE)
  {
    if (pMathExp->pPassivityExp != NULL)
    {
      if (pMathExp->pSubExp != NULL)
      {
        return ASM\_BLOCK\_TYPE\_NOT\_PASSIV\_SUB;
      }
      else
      {
        return ASM\_BLOCK\_TYPE\_NOT\_PASSIV;
      }
    }
    else
    {
      if (pMathExp->pSubExp != NULL)
      {
        return ASM\_BLOCK\_TYPE\_NOT\_SUB;
      }
      else
      {
        return ASM\_BLOCK\_TYPE\_NOT;
      }
    }
  }/\* end if \*/
  else
  {
    if (pMathExp->pPassivityExp != NULL)
    {
      if (pMathExp->pSubExp != NULL)
      {
        return ASM\_BLOCK\_TYPE\_PASSIV\_SUB;
      }
      else
      {
        return ASM\_BLOCK\_TYPE\_PASSIV;
      }
    }
    else
    {
      if (pMathExp->pSubExp != NULL)
      {
        return ASM\_BLOCK\_TYPE\_SUB;
      }
      else
      {
        return ASM\_BLOCK\_TYPE\_CONST;
      }
    }
  }/\* end else \*/

  return ASM\_BLOCK\_TYPE\_NONE;
}

LPBYTE  AsmBlock(PMATHEXP pMathExp, DWORD \*pdwOutBufferSize);
INLINE LPBYTE  AsmAdd(DWORD \*pdwBufferSize)
{
  /\*
   \* add eax, ecx
   \*/
  BYTE AddEaxEcx\[3\] = "\\x03\\xC1";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, AddEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmSub(DWORD \*pdwBufferSize)
{
  /\*
   \* Sub eax, ecx
   \*/
  BYTE SubEaxEcx\[3\] = "\\x2B\\xC1";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, SubEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmMul(DWORD \*pdwBufferSize)
{
  /\*
   \* mul eax, ecx
   \*/
  BYTE MulEaxEcx\[3\] = "\\xF7\\xE1";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, MulEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmDiv(DWORD \*pdwBufferSize)
{
  /\*
   \* push edx
   \* div eax, ecx
   \* pop edx
   \*/
  BYTE PushEdx = 0x52;
  BYTE DivEaxEcx\[3\] = "\\xF7\\xF1";
  BYTE PopEdx = 0x5A;

  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 4);
  
  memcpy(pAsmBuffer, &PushEdx, 1);
  memcpy(pAsmBuffer + 1, DivEaxEcx, 2);
  memcpy(pAsmBuffer + 1 + 2, &PopEdx, 1);

  \*pdwBufferSize = 1 + 2 + 1;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmMod(DWORD \*pdwBufferSize)
{
  /\*
   \* push edx
   \* div eax, ecx
   \* mov eax, edx
   \* pop edx
   \*/
  BYTE PushEdx = 0x52;
  BYTE DivEaxEcx\[3\] = "\\xF7\\xF1";
  BYTE MovEaxEdx\[3\] = "\\x8B\\xD0";
  BYTE PopEdx = 0x5A;

  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 6);
  
  memcpy(pAsmBuffer, &PushEdx, 1);
  memcpy(pAsmBuffer + 1, DivEaxEcx, 2);
  memcpy(pAsmBuffer + 1 + 2, MovEaxEdx, 2);
  memcpy(pAsmBuffer + 1 + 2 + 2, &PopEdx, 1);

  \*pdwBufferSize = 1 + 2 + 2 + 1;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmOr(DWORD \*pdwBufferSize)
{
  /\*
   \* or eax, ecx
   \*/
  BYTE OrEaxEcx\[3\] = "\\x09\\xC8";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, OrEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmAnd(DWORD \*pdwBufferSize)
{
  /\*
   \* and eax, ecx
   \*/
  BYTE AndEaxEcx\[3\] = "\\x21\\xC8";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, AndEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmXor(DWORD \*pdwBufferSize)
{
  /\*
   \* xor eax, ecx
   \*/
  BYTE XorEaxEcx\[3\] = "\\x33\\xC1";
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, 2);
  
  memcpy(pAsmBuffer, XorEaxEcx, 2);
  \*pdwBufferSize = 2;
  return pAsmBuffer;
}

INLINE LPBYTE  AsmOpt(MATH\_OPT Opt, DWORD \*pdwBufferSize)
{
  DWORD dwBufferSize = 0;
  LPBYTE pAsmBuffer = NULL;
  switch (Opt)
  {
  case MATH\_ADD:
    {
      pAsmBuffer = AsmAdd(&dwBufferSize);
    }break;
  case MATH\_SUB:
    {
      pAsmBuffer = AsmSub(&dwBufferSize);
    }break;
  case MATH\_MUL:
    {
      pAsmBuffer = AsmMul(&dwBufferSize);
    }break;
  case MATH\_DIV:
    {
      pAsmBuffer = AsmDiv(&dwBufferSize);
    }break;
  case MATH\_MOD:
    {
      pAsmBuffer = AsmMod(&dwBufferSize);
    }break;
  case MATH\_OR:
    {
      pAsmBuffer = AsmOr(&dwBufferSize);
    }break;
  case MATH\_AND:
    {
      pAsmBuffer = AsmAnd(&dwBufferSize);
    }break;
  case MATH\_XOR:
    {
      pAsmBuffer = AsmXor(&dwBufferSize);
    }break;
  }

  \*pdwBufferSize = dwBufferSize;
  return pAsmBuffer;
}

/\*
 \* 只有常量
 \*/
LPBYTE  AsmBlockOnConst(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* mov eax, 表达式的值
   \*/
  DWORD dwVal = pMathExp->dwVal;
  LPBYTE pBuffer = \_\_new\_\_(BYTE, 5);
  BYTE MovEaxConst\[6\] = "\\xB8\\x00\\x00\\x00\\x00";
  \*(DWORD \*)(MovEaxConst + 1) = dwVal;
  memcpy(pBuffer, MovEaxConst, 5);
  \*pdwBufferSize = 5;
  return pBuffer;
}

/\*
 \* 有子表达式
 \*/
LPBYTE  AsmBlockOnSub(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  DWORD dwAsmBufferSize = 0;
  LPBYTE pAsmBuffer = NULL;
  pAsmBuffer = AsmBlock(pMathExp->pSubExp, &dwAsmBufferSize);
  return pAsmBuffer;
}

/\*
 \* 有被动表达式
 \*/
LPBYTE  AsmBlockOnPassiv(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* ... 计算被动表达式的值
   \* mov ecx, eax;ecx 为被动表达式的值
   \* mov eax, 主动表达式的值
   \* opt eax, ecx
   \*/
  // 先求进行被动表达式的汇编
  DWORD dwPassivBufferSize = 0;
  LPBYTE pPassivBuffer = AsmBlock(pMathExp->pPassivityExp, &dwPassivBufferSize);
  // mov ecx, eax | 8BC8
  BYTE MovEcxEax\[3\] = "\\x8B\\xC8";
  // mov eax, XXXXXXXX
  DWORD dwVal = pMathExp->dwVal;
  BYTE MovEaxConst\[6\] = "\\xB8\\x00\\x00\\x00\\x00";
  \*(DWORD \*)(MovEaxConst + 1) = dwVal;
  // opt eax, ecx
  DWORD dwOptSize = 0;
  LPBYTE pOptBuffer = AsmOpt(pMathExp->Opt, &dwOptSize);

  // 合成汇编代码
  DWORD dwAsmSize = dwPassivBufferSize + 2 + 5 + dwOptSize;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, pPassivBuffer, dwPassivBufferSize);
  memcpy(pAsmBuffer + dwPassivBufferSize, MovEcxEax, 2);
  memcpy(pAsmBuffer + dwPassivBufferSize + 2, MovEaxConst, 5);
  memcpy(pAsmBuffer + dwPassivBufferSize + 2 + 5, pOptBuffer, dwOptSize);
  \_\_delete\_\_(pOptBuffer);
  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 有被动表达式
 \* 有子表达式
 \*/
LPBYTE  AsmBlockOnPassivSub(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* ...计算被动表达式的值
   \* push eax
   \* ...计算子表达式的值
   \* pop ecx;ecx 为先前压入栈的被动表达式的结果
   \* opt eax, ecx
   \*/
  // 计算被动表达式的值
  DWORD dwPassivAsmBufferSize = 0;
  LPBYTE pPassivAsmBuffer = AsmBlock(pMathExp->pPassivityExp, &dwPassivAsmBufferSize);
  // push eax
  BYTE PushEax = 0x50;
  // 计算子表达式的值
  DWORD dwSubAsmBufferSize = 0;
  LPBYTE pSubAsmBuffer = AsmBlock(pMathExp->pSubExp, &dwSubAsmBufferSize);
  // pop ecx
  BYTE PopEcx = 0x59;
  // opt eax, ecx
  DWORD dwOptSize = 0;
  LPBYTE pOptBuffer = AsmOpt(pMathExp->Opt, &dwOptSize);

  // 合成
  DWORD dwAsmSize = dwPassivAsmBufferSize + 1 + dwSubAsmBufferSize + 1 + dwOptSize;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, pPassivAsmBuffer, dwPassivAsmBufferSize);
  memcpy(pAsmBuffer + dwPassivAsmBufferSize, &PushEax, 1);
  memcpy(pAsmBuffer + dwPassivAsmBufferSize + 1, pSubAsmBuffer, dwSubAsmBufferSize);
  memcpy(pAsmBuffer + dwPassivAsmBufferSize+ 1 + dwSubAsmBufferSize, &PopEcx, 1);
  memcpy(pAsmBuffer + dwPassivAsmBufferSize + 1 + dwSubAsmBufferSize + 1, pOptBuffer, dwOptSize);
  \_\_delete\_\_(pPassivAsmBuffer);
  \_\_delete\_\_(pSubAsmBuffer);
  \_\_delete\_\_(pOptBuffer);
  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 有 NOT 操作符
 \*/
LPBYTE  AsmBlockOnNot(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* mov eax, 表达式的值
   \* not eax
   \*/
  // mov eax, XXXXXXXX
  DWORD dwVal = pMathExp->dwVal;
  BYTE MovEaxConst\[6\] = "\\xB8\\x00\\x00\\x00\\x00";
  \*(DWORD \*)(MovEaxConst + 1) = dwVal;
  // not eax
  BYTE NotEax\[3\] = "\\xF7\\xD0";

  // 合成
  DWORD dwAsmSize = 5 + 2;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, MovEaxConst, 5);
  memcpy(pAsmBuffer + 5, NotEax, 2);

  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 有 NOT 操作符
 \* 有子表达式
 \*/
LPBYTE  AsmBlockOnNotSub(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* ...计算子表达式的值
   \* not eax
   \*/
  DWORD dwSubBufferSize = 0;
  LPBYTE pSubAsmBuffer = AsmBlockOnSub(pMathExp, &dwSubBufferSize);
  // not eax
  BYTE NotEax\[3\] = "\\xF7\\xD0";

  // 合成
  DWORD dwAsmSize = dwSubBufferSize + 2;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, pSubAsmBuffer, dwSubBufferSize);
  memcpy(pAsmBuffer + dwSubBufferSize, NotEax, 2);
  \_\_delete\_\_(pSubAsmBuffer);
  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 有 NOT 操作符
 \* 有被动表达式
 \*/
LPBYTE  AsmBlockOnNotPassiv(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
   \* ...计算被动表达式的值
   \* not eax
   \*/
  // 被动表达式
  DWORD dwPassivBufferSize = 0;
  LPBYTE pPassivBuffer = AsmBlockOnPassiv(pMathExp, &dwPassivBufferSize);
  // not eax
  BYTE NotEax\[3\] = "\\xF7\\xD0";

  // 合成
  DWORD dwAsmSize = dwPassivBufferSize + 2;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, pPassivBuffer, dwPassivBufferSize);
  memcpy(pAsmBuffer + dwPassivBufferSize, NotEax, 2);
  \_\_delete\_\_(pPassivBuffer);
  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 有 NOT 操作符
 \* 有被动表达式
 \* 有子表达式
 \*/
LPBYTE  AsmBlockOnNotPassivSub(PMATHEXP pMathExp, DWORD \*pdwBufferSize)
{
  /\*
     \* ...计算被动表达式与子表达式的值
   \* not eax
   \*/
  DWORD dwPassivSubAsmBufferSize = 0;
  LPBYTE pPassivSubAsmBuffer = AsmBlockOnPassivSub(pMathExp, &dwPassivSubAsmBufferSize);
  // not eax
  BYTE NotEax\[3\] = "\\xF7\\xD0";

  // 合成
  DWORD dwAsmSize = dwPassivSubAsmBufferSize + 2;
  LPBYTE pAsmBuffer = \_\_new\_\_(BYTE, dwAsmSize);
  memcpy(pAsmBuffer, pPassivSubAsmBuffer, dwPassivSubAsmBufferSize);
  memcpy(pAsmBuffer + dwPassivSubAsmBufferSize, NotEax, 2);
  \_\_delete\_\_(pPassivSubAsmBuffer);
  \*pdwBufferSize = dwAsmSize;
  return pAsmBuffer;
}

/\*
 \* 汇编优先级
 \* 被动表达式 > 子表达式 > 主动表达式
 \*/
LPBYTE  AsmBlock(PMATHEXP pMathExp, DWORD \*pdwOutBufferSize)
{
  LPBYTE pAsmBlock = NULL;
  DWORD dwAsmBufferSize = 0;

  ASM\_BLOCK\_TYPE AsmBlockType = ReturnAsmBlockType(pMathExp);
  switch (AsmBlockType)
  {
  case ASM\_BLOCK\_TYPE\_NOT\_PASSIV\_SUB:
    {
      pAsmBlock = AsmBlockOnNotPassivSub(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_NOT\_PASSIV:
    {
      pAsmBlock = AsmBlockOnNotPassiv(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_NOT\_SUB:
    {
      pAsmBlock = AsmBlockOnNotSub(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_NOT:
    {
      pAsmBlock = AsmBlockOnNot(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_PASSIV\_SUB:
    {
      pAsmBlock = AsmBlockOnPassivSub(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_PASSIV:
    {
      pAsmBlock = AsmBlockOnPassiv(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_SUB:
    {
      pAsmBlock = AsmBlockOnSub(pMathExp, &dwAsmBufferSize);
    }break;
  case ASM\_BLOCK\_TYPE\_CONST:
    {
      pAsmBlock = AsmBlockOnConst(pMathExp, &dwAsmBufferSize);
    }break;
  }

  \*pdwOutBufferSize = dwAsmBufferSize;
  return pAsmBlock;
}

LPBYTE  AssemblerMathExp(PMATHEXP pMathExp, DWORD \*pdwOutBufferSize)
{
  DWORD dwAsmBufferSize = 0;
  LPBYTE pAsmBuffer = AsmBlock(pMathExp, &dwAsmBufferSize);
  \*pdwOutBufferSize = dwAsmBufferSize;

  return pAsmBuffer;
}

LPBYTE  MathExpGen(DWORD \*pOutBufferSize)
{
  // 产生一个随机值队列
  GenerateRollArray(g\_MathExpGenRollArray, DEF\_GEN\_MATHEXP\_SEED);
  PMATHEXP pMathExp = GenerateMathExp();
  /\*
   \* 进行汇编,利用深度优先算法
   \* 先移动到图中的最底层进行汇编(最底层的运算优先级最高)
   \*/
  DWORD dwAsmSize = 0;
  LPBYTE pAsmBuffer = AssemblerMathExp(pMathExp, &dwAsmSize);
  \*pOutBufferSize = dwAsmSize;

  // 释放表达式内存
  ReleaseMathExp(&pMathExp);
  return pAsmBuffer;
}

```

如果在做些随机的函数模板生成与跳转指令的模板与上面的形式结合。应该可以不难作出完全随机的类真花指令生成器。不过以上在进行汇编的时候，也只采用 eax,ecx 寄存器.在做除法于取余操作时使用 edx 寄存器。有兴趣的朋友可以这些寄存器可以扩充。或者采用那个万用逻辑闸来扩充这套指令。在做逻辑运算时使用 NAND 闸或者 NOR 闸进行逻辑算术的扩充并再次生成。  
  
![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")![](img/1e00a4c5ba6d273521879fcbd5363999.jpg "碌眼")![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")![](img/1e3c9433a1c444ac212c55a1ce70452a.jpg "狂怒")![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/32b12ba0f4c0e6a8843aaa06ba62fbdd.jpg "皱眉")![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")