

# 软件保护壳技术专题 - 变形引擎的构建

> 原文：[`www.pediy.com/kssd/pediy10/95879.html`](https://www.pediy.com/kssd/pediy10/95879.html)

变形引擎的构建  
<YY>  
    好久没有更新文章了。今年工作太忙，确实没有时间来发贴。有些朋友问我，专题会不会成为“太监”  
按照我耐磨的性格，应该不会。  
<目录>  
0.哪些部分需要变形?  
1.聊一聊花指令的构建  
  1.1 产生随机寄存器  
  1.2 产生随机的清 0  
  1.3 产生随机的 JMP  
  1.4 产生异常  
  1.5 利用 MODRM/SIB  
2.构建解密头  
3.另一种变形  
4.又是另一种变形  
5.强调些东西  
<内容>  
0.哪些部分需要变形  
壳的变形技术主要来自于病毒。病毒变形主要目的为了躲避杀毒软件的查杀。壳的变形也无非是这个。当然还有  
目的是为了延长破解时间。消耗 Cracker 的精力。  
现在我们来看一个经典的变形结构。也确认哪部分是用来变形的  
壳加载段  
\----------  
解密头  
壳主体被加密的部分  
密钥储存  
\----------  
大多数情况下。壳加载器的结构就是这三个部分。解密头对壳主体进行解密后再跳入到壳主体进行运行  
这里就可以很明显的看出 壳主体是很容易进行变形的。只要每次加密的密钥不一样最后的队列肯定不一样  
现在解密头是固定的。所以这里是最需要进行变形的。  
以上这个结构绝对是经典多态的结构。再复杂的变形也是从以上这个简单的结构开始的。下面看一段简单的 CODE  

代码:

```
call delta
delta:
pop esi
add esi, offset Packet - offset delta
mov edi, esi
mov ecx, offset Decoder - offset Packet
mov eax, esi
add eax, offset Decoder - offset Packet
mov bl, byte ptr \[eax\]
@@:
lodsb
xor al, bl
stosb
loop @B
;; 这里解密后直接进去壳主体
Packet:
;........壳主体
Decoder:
db 099h

```

以上就是简单的 ASM 代码了。这里的算法是 XOR 当然比较简单,当然在解密之前要首先保证代码段是可写属性  
现在我们开始变形解密头要如何变形呢？这里可以看出其实总共解密头也没几个字节,提出两种通用方式  
一个就是添加花指令构建将花指令丢到解密头每条指令中间。一个就是等价的代码替换,寄存器的替换。  
总之解密头是我们自己编写的。我们可以精心的去构建一段代码。并将此按照我们自己的规则进行模板化  
每当生成新的壳解密头时进行填充。当然也可以用一些复杂的规则去扩大这个结构。自定义的模板规则。复杂  
的花指令生成，等等。产生解密模板的时候可以适当的运用一些编译原理的技术。首先产生一个自己可以理解的  
中间指令，然后再翻译成 X86 代码。这样的好处是可以无限的扩充模板的形势。变形力度很大。在病毒上可能实现要麻烦  
些。毕竟病毒要考虑到自身体积的大小。但是壳却不用考虑那么多。  
  
另外一点要说明的是做变形的时候不要拘泥于此格式。可以尽量发挥想象来做。例如把解密分段，或者将解密先写入到栈中  
在执行。总之达到恶心 Cracker 的目的就成。  
  
1.聊一聊花指令的构建  
花指令的构建方法有很多种,但是原理都是相同的在不影响运行上下文的情况下，产生一些无效的指令。同样下文以例子展出  
如果自己做的,尽量发挥想象力。不用拘泥于这篇文章或者某篇文章。  
1.1 产生随机寄存器  
每条指令对寄存器的操作的编码都有一些规则，具体是什么规则可以读下 INTEL 的编码规则。这个专题里也有一篇是讲反汇编引擎的  
不过为了不产生歧义还是推荐去看 INTEL 的手册。这里也不讲 IA32 的编码规则了。直接来看些具体的例子$R 表示寄存器,IMM(X)表示  
立即数  
例如 MOV $R, IMM32,形如类似的移动语句。OPCODE 是 B8 IMM32 这样的。B8 表示 MOV EAX, 可以在 OD 里查看 B9 为 MOV ECX,这里有个排序  
从 EAX 开始为 EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI 这样的顺序。如果翻看 MODRM/SIB 规则可以看到这个排序规则，至于为什么要这样排  
只能去问 INTEL 公司了。INTEL 手册里也没说明。  
总之 MOV $R, IMM32 可以变成为 B8 B9 BA BB .... 按照以上那个排序选取寄存器。其他指令也一样例如  
POP $R 是从 58 开始的.所以每次选取之需要很简单的一个随机模 8 就可以了。 但是这里要排除一个意外就是 esp。ESP 控制着堆栈指针稍微有些不  
细心就会造成程序运行错误。为了保险还是把它 T 出去好了。不过不是非要排除 ESP。如果你的花指令产生器构建的够精密,模板够好。  
也可以把 ESP 的变换也加进来。为了稳定，按照我谨慎的性格还是将它排除在外。SHOW 段代码。C++的。  
代码:

```
const DWORD Registry\_Eax = 0x00;
const DWORD Registry\_Ecx = 0x01;
const DWORD Registry\_Edx = 0x02;
const DWORD Registry\_Ebx = 0x03;
const DWORD Registry\_Esp = 0x04;
const DWORD Registry\_Ebp = 0x05;
const DWORD Registry\_Esi = 0x06;
const DWORD Registry\_Edi = 0x07;
const DWORD Registry\_MaxCount = 0x08;
while (bR0 == Registry\_Esp) bR0 = (BYTE)(lrand() % Registry\_MaxCount);
// bR0 是选取的寄存器,lrand 是随机函数

```

1.2 产生随机的清 0  
在编写代码过程中请 0 操作是常用操作。sub $R,$R ... xor $R,$R 等都是常用的清 0 方式  
我们把这些方式记录下来并保存为我们的模板,如果你不喜欢单指令的。也可以做一些复杂些  
的模板 mov $R, IMM32; sub $R, IMM32-1; dec $R;都是可以的。以下展示段简单的 C++代码  
代码:

```
const DWORD Clr0\_Mov = 0x00;
const DWORD Clr0\_Xor = 0x01;
const DWORD Clr0\_Sub = 0x02;
DWORD MakeClearZero(LPBYTE pClearZero, DWORD dwBufSize, BYTE bRegistry)
{
  // mov $R, 0
  const BYTE MoveZeroBase\[\] = {"\\xB8\\x00\\x00\\x00\\x00"};
  // xor $R, $R
  const BYTE XorZeroBase\[\] = {"\\x33\\xC0"};
  // sub $R, $R
  const BYTE SubZeroBase\[\] = {"\\x2B\\xC0"};
  int nRand = 0;
  if (dwBufSize >= 5)
  {
    lsrand((unsigned)time(NULL));
    nRand = lrand() % 3;
    if (nRand == Clr0\_Mov)
    {
      memcpy(pClearZero, MoveZeroBase, 0x05);
      while (bRegistry == Registry\_Esp)
        bRegistry = (BYTE)(lrand() % Registry\_MaxCount);
      bRegistry += 0xB8;
      \*(BYTE \*)pClearZero = bRegistry;
      return 0x05;
    }
    else if (nRand == Clr0\_Xor)
    {
      memcpy(pClearZero, XorZeroBase, 0x02);
      while (bRegistry == Registry\_Esp)
        bRegistry = (BYTE)(lrand() % Registry\_MaxCount);
      bRegistry \*= 0x09;
      bRegistry += 0xC0;
      \*(BYTE \*)(pClearZero + 1) = bRegistry;
      return 0x02;
    }
    else
    {
      memcpy(pClearZero, SubZeroBase, 0x02);
      while (bRegistry == Registry\_Esp)
        bRegistry = (BYTE)(lrand() % Registry\_MaxCount);
      bRegistry \*= 0x09;
      bRegistry += 0xC0;
      \*(BYTE \*)(pClearZero + 1) = bRegistry;
      return 0x02;
    }
  }
  else if (dwBufSize >= 2)
  {
    lsrand((unsigned)time(NULL));
    if (nRand == Clr0\_Xor)
    {
      memcpy(pClearZero, XorZeroBase, 0x02);
      while (bRegistry == Registry\_Esp)
        bRegistry = (BYTE)(lrand() % Registry\_MaxCount);
      bRegistry \*= 0x09;
      bRegistry += 0xC0;
      \*(BYTE \*)(pClearZero + 1) = bRegistry;
      return 0x02;
    }
    else
    {
      memcpy(pClearZero, SubZeroBase, 0x02);
      while (bRegistry == Registry\_Esp)
        bRegistry = (BYTE)(lrand() % Registry\_MaxCount);
      bRegistry \*= 0x09;
      bRegistry += 0xC0;
      \*(BYTE \*)(pClearZero + 1) = bRegistry;
      return 0x02;
    }
  }
  return 0;
}

```

可以发挥想象自行编写。包括外部模板导入等等。  
1.3 产生随机的 JMP  
这里就不废话了。有了上面的基础,直接看实例代码吧  
代码:

```
DWORD GenerateDummyJmp(PJUNKCODE\_BASE pJunkCodeBase, LPBYTE pStart, DWORD dwJunkCodeSize)
{
  const DWORD dwMinSize = 0x05;
  if (dwJunkCodeSize < dwMinSize) return 0;
  LPBYTE pEnd = pStart + dwJunkCodeSize;
  // 只能产生向后跳的指令
  BYTE JmpCodes\[\] = {"\\xE9\\xFF\\xFF\\xFF\\xFF"};
  DWORD dwOffset = pEnd - pStart - 0x05;
  \*(DWORD \*)&JmpCodes\[1\] = dwOffset;
  memcpy(pStart, JmpCodes, 0x05);
  // 生成花指令
  DWORD dwRet = GenerateJunkCode(pJunkCodeBase, pStart + 0x05, dwOffset);
  return (dwRet + 0x05);
}
// pJunkCodeBase 是 垃圾指令库,GenerateJunkCode 是产生垃圾指令。进行填充可以自行构建

```

1.4 产生异常  
直接 C 代码  
代码:

```
// 这个其中有些函数是产生 push pop 的原理都一样.有兴趣的自己动手 DIY 就 OK 了。原理很简单没什么好写的了
DWORD NaTaSha::GenerateDummySehJmp(PJUNKCODE\_BASE pJunkCodeBase, LPBYTE pHeader, DWORD dwHeaderSize)
{
  // 基本型
  // push $R0        ; 1byte
  // push $R1        ; 1byte
  // call delta      ; 5bytes
  // delta:
  // pop $R0        ; 1byte
  // add $R0, dwOffset  ; 6bytes
  // push $R0        ; 1byte
  // xor $R1, $R1      ; 2bytes
  // push fs:\[$R1\]    ; 3bytes
  // mov fs:\[$R1\], esp  ; 3bytes
  // 触发异常
  // 垃圾代码
  // 执行代码
  // pop fs:\[$R1\]      ; 3bytes
  // add esp, 04h      ; 3bytes
  // pop $R1        ; 1byte
  // pop $R0        ; 1byte
  const DWORD dwSehTotal = 0x80;
  if (dwHeaderSize < dwSehTotal) return 0;

  LPBYTE pHandler = NULL, pCurr = pHeader;
  // 模拟 SEH 跳转的空间最小长度为 128 个字节
  const DWORD dwResumeStack = 0x06;//恢复异常处理堆栈长度
  DWORD dwRemain = dwHeaderSize, dwSize = 0;
  // 首先随机选出两个寄存器,为了避免意外,过滤掉 esp 寄存器
  lsrand((unsigned)time(NULL));
  // R0 作为重定位与触发异常,R1 作为清 0
  BYTE bR0 = (BYTE)(lrand() % Registry\_MaxCount), bR1 = (BYTE)(lrand() % Registry\_MaxCount);
  while (bR0 == Registry\_Esp) bR0 = (BYTE)(lrand() % Registry\_MaxCount);
  while (bR1 == Registry\_Esp) bR1 = (BYTE)(lrand() % Registry\_MaxCount);
  
  DWORD dwRet = 0;
  // 保存寄存器
  dwRet = MakePushValue(bR0);
  \*pCurr = (BYTE)dwRet;
  dwSize++;
  dwRemain--;
  pCurr++;

  dwRet = MakePushValue(bR1);
  \*pCurr = (BYTE)dwRet;
  dwSize++;
  dwRemain--;
  pCurr++;

  // 重定位
  LPBYTE pCall0 = pCurr;
  dwRet = MakeRelocateCode(pCurr, dwRemain, bR0, 0);
  dwSize += dwRet;
  dwRemain -= dwRet;
  pCurr += dwRet;
  DWORD dwCall0Size = dwRet;

  // 压入异常句柄
  dwRet = MakePushValue(bR0);
  \*pCurr = (BYTE)dwRet;
  dwSize++;
  dwRemain--;
  pCurr++;

  // 清 0
  dwRet = MakeClearZero(pCurr, dwRemain, bR1);
  dwSize += dwRet;
  dwRemain -= dwRet;
  pCurr += dwRet;

  // 建立异常堆栈
  const BYTE PushFsR\[\] = {"\\x64\\xFF\\x30"};//3 字节
  const BYTE MovEspR\[\] = {"\\x64\\x89\\x20"};//3 字节
  // push dword ptr fs:\[$R\]
  memcpy(pCurr, PushFsR, 0x03);
  \*(pCurr + 0x02) = bR1 + 0x30;
  dwSize += 0x03;
  dwRemain -= 0x03;
  pCurr += 0x03;

  // mov dword ptr fs:\[$R\], esp
  memcpy(pCurr, MovEspR, 0x03);
  \*(pCurr + 0x02) = bR1 + 0x20;
  dwSize += 0x03;
  dwRemain -= 0x03;
  pCurr += 0x03;

  // 触发异常
  dwRet = MakeExpCode(pCurr, dwRemain, bR1);
  dwSize += dwRet;
  dwRemain -= dwRet;
  pCurr += dwRet;

  // 确定是否产生垃圾代码
  lsrand((unsigned)time(NULL));
  int nRand = lrand() % 2;
  if (nRand == 0)// 不产生垃圾代码
    goto GenerateExpHandler;

  // 垃圾代码
  DWORD dwJunkCodeSize = dwRemain / 2;
  nRand = lrand() % dwJunkCodeSize;
  dwJunkCodeSize = nRand + 1;//随机长度范围在剩余长度的一半之内
  dwRet = GenerateJunkCode(pJunkCodeBase, pCurr, dwJunkCodeSize);
  dwSize += dwRet;
  dwRemain -= dwRet;
  pCurr += dwRet;

  // 异常执行函数
GenerateExpHandler:
  DWORD dwExpCodeSize = dwRemain - dwResumeStack;
  dwRet = GenerateJunkCode(pJunkCodeBase, pCurr, dwExpCodeSize);
  dwSize += dwRet;
  dwRemain -= dwRet;
  DWORD dwOffset = (DWORD)(pCurr - pCall0);
  // 0x04 为偏移长度
  \*(DWORD \*)(pCall0 + dwCall0Size - 0x04) = dwOffset;
  pCurr += dwRet;

  // 恢复异常堆栈
  const BYTE PopFsR\[\] = {"\\x64\\x8F\\x00"};
  const BYTE AddEsp4\[\] = {"\\x83\\xC4\\x04"};
  // pop dword ptr fs:\[$R\]
  memcpy(pCurr, PopFsR, 0x03);
  \*(pCurr + 0x02) = bR1;
  dwSize += 0x03;
  dwRemain -= 0x03;
  pCurr += 0x03;

  // add esp, 04h
  memcpy(pCurr, AddEsp4, 0x03);
  dwSize += 0x03;
  dwRemain -= 0x03;
  pCurr += 0x03;
  return dwSize;
}

```

1.5 利用 MODRM/SIB  
说说 MODRM/SIB 的邪恶之处。如果你不知道什么是 MODRM/SIB 去看看那篇反汇编引擎的构建吧或者直接去都 INTEL 手册。  
TMD 这个壳里运用了很多 M/S 这个结构构造的花指令。这种构造很恶心尤其是联合上 JMP 指令。每次你要想在 OD 里浏览代码  
整体样子。都必须要去修复这段代码。相比脱过壳的朋友都遇上过这种。现在看一个基本型.其余复杂的形式靠自己想象了。  
再次郑重想做好壳。要时刻保持丰富的想象力还有一颗“邪恶”的心。所谓是人至\*则无敌。壳至\*则无 Cracker...  
\------  
00401010:  
JMP 004013E ;这里就跳入到了 dword ptr \[4 \* ebx + 0xFFFFEEEE\]这部分  
其余指令  
004013D  
mov eax, dword ptr \[4 \* ebx + 0xFFFFEEEE\]  
  
这里发挥一下想象力吧。dword ptr \[4 \* ebx + 0xFFFFEEEE\] 这部分是可是有足足 6 个字节的空间可以发挥想象力  
但是通常只能利用到 SIB 位。因为 MODRM 位产生不了。当 Mod == 00 01 10 RM == 100 时,那么它的下一个字节就是 SIB 了。  
通常 JMP 指令是个不错的选择。因为最长的 JMP 也就 5 个字节刚好满足。既有了花指令的作用，又可以对抗反汇编。一个不错的想法。  
这的代码太长就不 SHOW 了自己发挥一下。不难利用。  
  
2.构建解密头  
看了以上的 CODE，聪明的人应该早就想到如何去构建解密头了。和花指令构建一样。只需要按照事先定义好的模板随机选取适当的寄存器  
填写花指令模板。重新修订 JMP/CALL/JCC 三种指令的偏移就好了。如果看到这里你还是没明白，那么先去补充一下基础知识吧。  
  
3.另一种变形  
这种变形主要目的是迷糊 Cracker.名字我不知道怎么取，就叫另一种变形好了。让我们看一下这种变形的结构  
首先看一下正常的代码段  
正常情况  
\----------  
指令段 1  
CALL C\_A  
指令段 2  
C\_A:  
指令段 3  
\*\*\*\*\*\*\*\*\*\*  
壳段的某区域  
\----------  
壳指令段 1  
P\_A:  
壳指令段 2  
  
  
非正常情况  
\----------  
指令段 1  
CALL P\_A  
指令段 2  
C\_A:  
指令段 3  
\*\*\*\*\*\*\*\*\*\*  
壳段的某区域  
\----------  
壳指令段 1  
P\_A:  
花指令  
JMP C\_A:  
壳指令段 2  
  
从上面的两种情况可以看出来这种变形了吧。其实这种变形主要是为了对抗一些流程图软件，例如“万恶”的 IDA。“万恶”的 F5  
原理十分简单将属于偏移的 CALL/JMP/JCC 指令修改偏移首先跳到壳的某个区域,壳的这个区域可以做很多事情了。自动生成的花指令，  
让 Cracker 可以看出来的反调试代码。或者跳入到自己的虚拟机内。执行完毕后 JMP 到原来正常代码跳入到的地方。这个技巧的运用要  
注意的是 8 位的跳转尽量不要去动。16 位的偏移最好计算一下是否偏移够长度。32 位的则大胆的去做 HOOK 吧。  
  
4.又是另一种变形  
这种变形我也不知道是什么名字，不过姑且认为是一个替换吧。见结构  
正常情况  
\----------  
指令段 1  
CALL C\_A  
指令段 2  
C\_A:  
指令段 3  
JMP C\_B  
指令段 4  
C\_B:  
\*\*\*\*\*\*\*\*\*\*  
壳段的某区域  
\----------  
壳指令段 1  
P\_A:  
壳指令段 2  
  
看看做过替换后的结构  
非正常情况  
\----------  
指令段 1  
CALL C\_A  
JMP P\_A  
花指令  
C\_A:  
指令段 3  
JMP C\_B  
指令段 4  
C\_B:  
\*\*\*\*\*\*\*\*\*\*  
壳段的某区域  
\----------  
壳指令段 1  
P\_A:  
指令段 2  
JMP C\_A  
壳指令段 2  
  
这里可以看出我们将两个 JMP 指令中间的指令段 2 搬运到了壳的某个地方。并将中间的指令用花指令进行填充。这样极大程度的改变了特征码  
在指令执行完毕后又跳回了指令执行完毕的地址。这个搬运比较复杂。条件也有诸多限制。例如如果指令段 1 中有代码跳入到指令段 2 中就会出现错误  
这个问题有一个非完美的解决方案。例如将指令段 2 内的每条指令的偏移计算得出。并利用 JMP 指令进行填充。填充后的结构如下  
指令段 2  
\----------  
JMP P\_A(指令段第一条指令)  
JMP P\_A+N(指令段第二条指令)  
JMP P\_A+N+N(指令段第三条指令)  
.....  
以此类推,保证每当跳入到替换后的代码区域可以正常的跳入到原本的指令继续执行。不过也会出现错误。就是空间的限制。指令段 2 的整体空间  
在现实情况下不一定是以 5 字节对齐的。所以不可能每条指令都有一个入口 JMP.最坏的情况的是在某个地方有一个动态跳转。例如 CALL DWORD PTR \[EAX\]  
这种形式。正好跳入到一个不以 5 字节对齐替换段的末尾地址。这会引起不稳定。当然还有一些其他解决方案。在生成之前首先生成整个 PE 文件的流程图  
并进行分析。哪些是可以进行替换的哪些是不可以的。长度问题。偏移问题。是否有指令跳入到这片区域执行，这片区域是否有指令跳出。  
我们为什么要截取两个跳转指令直接的代码段进行替换呢？很简单就是大多数 JMP 类指令都是以偏移做标准的。移动后的代码偏移量不变。跳转地址取发生了  
改变。当然如果你有绝对好的设计。也可以考虑修复此偏移在替换后的情况下。我为了稳定在自己壳中，还是忽略了此项设计。原理就是这些。  
越是复杂的壳越没有良好的兼容性。都是这样的“龌龊”技巧造成的。掌握了基本 PE 结构，基本壳原理后。发挥自己的想象力，让 Crakcer 郁闷去吧。  
  
5.强调些东西  
很多花指令生成的花指令都太明显了。可以一眼看出来是花指令，有些居然还出现浮点指令和 MMX 指令等。说实在的耗无意义。  
花指令构建越是像真实指令越好。如果你在构造花指令模板的时候。可以直接构建一些正常代码的模板。例如一套很复杂的数学运行工作的函数。  
花指令主要起迷惑作用。垃圾指令只适合病毒对抗杀毒软件。不适合长期与 Crakcer 做斗争的壳。  
  
这里还要说的是，专题不会成为太监，但是真的不能保证更新速度。我的时间不是非常充裕。希望大家见谅。![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")