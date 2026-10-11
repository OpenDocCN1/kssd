
# ring 3 级 32 位 x86 cpu 仿真

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-491.htm`](https://www.pediy.com/kssd/pediy09/pediy09-491.htm)

> 【 标题 】 ring 3 级 32 位 x86 cpu 仿真

> 【 作者 】 `linxer`

> 【 QQ 】 `3568599`

> 【 声明 】 俺系初级选手，高手略过。失误之处敬请诸位大侠赐教！

> 题外话：本文系本人最近编码的总结，目前初步完成了 ring 3 级别 x86 cpu 的仿真，但由于这个东西还跟很多模块关联在一起，这些模块我都还没有写，因此，我写的这个虚拟的 ring 3 x86 cpu 的代码也还没有进行功能性等测试，目前只是能编译过去而已。本文旨在说明如何仿真，不在于 show 代码，不过为了说明问题，会引用一些代码，这些代码来源于那些未测试的代码，有意往下看的，请抱着一颗发掘代码 bug 的心，抱歉！

> 前段时间已经发过一篇关于如何识别 x86 机器码的文章，机器码识别出来了，接下来就是要交给仿真的 cpu 执行，这里给出一个相对简单的 cpu 的实现。

> 首先说下，这里为什么只仿真 ring 3 级别的一些功能？这是由于仿真的 cpu 上不能安装 OS 决定的，如果要用到一些 ring 0 的功能那怎么办呢，就只能仔细分析这个 ring 0 的功能，通过其它方式仿真出来了。

> 这个仿真的 cpu 包含以下三个部分：cpu 环境，寻址系统和指令解析系统。

## 一. cpu 环境

### 必要的宏定义

```
#define u8 unsigned char
#define s8 char

#define u16 unsigned short int
#define s16 short int

#define u32 unsigned int
#define s32 int
```

1. 8 个普通寄存器在 x86 上按如下顺序索引

```
typedef enum tagCommonRegIndex
{
  EAX = 0,
  ECX,
  EDX,
  EBX,
  ESP,
  EBP,
  ESI,
  EDI
}CommonRegIndex;
```

2. 6 个段寄存器在 x86 上按如下顺序索引

```
typedef enum tagSegmentRegIndex
{
  ES = 0,
  CS,
  SS,
  DS,
  FS,
  GS
}SegmentRegIndex;
```

3. 定义标志寄存器有用位的索引

这里并没有按照 `intel` CPU 的格式来定义，主要是出于效率考虑，因为程序执行过程中有大量的跳转语句，这些语句都要用到条件位，用这种方式比用一个 `unsigned long` 来定义标志寄存器，每次可以节省一个`&`操作，由于在仿真的 cpu 上执行程序，效果是个大问题，因此这里在效率问题上也是“寸土必争”的；另外，我还把常用的条件位放在一起了，这也与 x86 cpu 不同，这里只要是为了减少 `cache miss` 情况，不过应该收效甚微的

这种定义方式就注定了标志寄存器操作指令的特殊性，幸好与标志寄存器相关的指令只要仿真四条(`pushf/sahf/popf/lahf`)

```
typedef enum tagFlagReg
{
  CF = 0,
  ZF,
  SF,
  OF,
  DF,
  PF,
  AF,
  TF,
  IF
}FlagReg;
```

4. 普通 32 位寄存器结构声明

用这个结构可以比较方便的访问普通寄存器里包含的"小寄存器"，比如说 `eax` 中的 `ax/ah/al`

```
typedef union tagCommonReg
{
  s32 nAll;
  s16 _x;
  struct tag8CommonReg
  {
    s8 _l;
    s8 _h;
  }_8;
}CommonReg, *PCommonReg;
```

5. cpu 环境

```
typedef struct tagCPUEnvernment
{
  CommonReg  commonReg[8];  //8 个普通 32 位寄存器
  u32    eip;    //eip 寄存器
  CommonReg  segReg[6];  //段寄存器，这里其实用 s16 就可以啦，段寄存器虽然有 48bit，但是我们可见的只有 16bit
  u8    flagReg[9];  //要用到的 9 个标志寄存器位
}CPUEnvernment, *PCPUEnvernment;
```

为了以下的应用，这里定义一个 cpu 环境变量：

```
CPUEnvernment g_cpu;
```

## 二. 寻址系统

这里不含寄存器的寻址，因为寄存器的寻址可以很轻松搞定，一般都在 `opcode` 和 `ModR/M` 包含了。这里特指内存寻址。

对 32 位的 cpu 来说，含有 16 位寻址和 32 位寻址，具体用的是那种寻址方式，由每条指令的寻址大小前缀码指定，不过现在好像都是用 32 位寻址啦。

### 1. 16 位寻址

要内存寻址的指令，指令一定会有 `modR/M` 字节，这个字节会标识该如何寻址，在 16 位寻址的指令中，是没有 `sib` 字节的，这样使得 16 位寻址仿真起来也相对简单些。比如，当 `mod=00,R/M=000` 的时候，它表示`[bx + si]`，我们可以由`g_cpu.commonReg[EBX]._x + g_cpu.commonReg[ESI]._x`得到程序执行过程中用到的虚拟地址。具体情况可以参考 `Intel` 官方手册第二卷的 `Intel Instruction Set Reference(A-M)`部分的 36 页。

### 2. 32 位寻址

32 位寻址比 16 位寻址要复杂的多，要 32 内存寻址的指令中一定会有 `modR/M` 字节，可能会有 `sib` 字节。

在 `R/M` 不为 `100` 的时候，表示指令中不含 `sib` 字节，它的情况跟 16 位寻址没有什么差别，比如，当 `mod=00,R/M=000` 的时候，它表示`[eax]`，我们可以由`g_cpu.commonReg[EAX].nAll`得到程序执行过程中用到的虚拟地址。具体情况可以参考 `Intel` 官方手册第二卷的 `Intel Instruction Set Reference(A-M)`部分的 37 页。

在 `R/M=100` 的时候，那么指令就含有 `sib` 字节了，`sib` 字节主要用来支持一些象数组样的寻址，比如说 `sib=01 000 001`，由它表示的是`[ecx + eax * 2]`，我们可以由`g_cpu.commonReg[ECX].nAll + g_cpu.commonReg[EAX].nAll * 2`得到 `sib` 字节表示的虚拟地址，然后在和 `mod` 字段配合，加上一定的偏移，就可以得到最终的虚拟地址了。具体情况可以参考 `Intel` 官方手册第二卷的 `Intel Instruction Set Reference(A-M)`部分的 38 页。

### 3. 地址转换

由上面得到的地址都是虚拟地址，那怎么获得这个虚拟地址实际表示的操作数呢，即怎样获得它在真实 cpu 上的虚拟地址呢，这个转换应该可以说比较简单的，在程序在仿真 cpu 上执行前，我们要有个 `PE load` 的过程，在这个过程中，我们可以知道，这个 `pe` 文件的 `ImageBase` 跟真实加载到内存中的起始地址的差额，用这个差额就可以完成这种地址转换了。

## 三. 指令解析系统

写这个模块是最枯燥的，不过这也取决于你要虚拟多少条指令，基本上是每个指令要有一个专门的解析函数，因此工作量是挺大的，体力活！

对指令的解析可以用两种方法来做到：

### 1. 关键地方内嵌汇编

因为对条件位的设置比较麻烦，我们可以用真实的 cpu 来搞定这些工作，这样轻松简洁，出错概率也减小不少，拿 `opcode=0x3c` 来说，它是 `cmp` 指令，它的一个参作数在 `eax` 中，第二个参作数是一个立即数

```
s32 cmp_rac_imm_fun()
{
  s32 lFirst;
  s32 lSecond;
  s16 sFirst;
  s16 sSecond;
  s8 cFirst;
  s8 cSecond;

  u8 cCF = 0; //这里目前只仿真了 4 个条件位，初始默认为 0
  u8 cOF = 0;
  u8 cSF = 0;
  u8 cZF = 0;

  if(操作数大小是 4) //将最可能执行到的条件放前面，以提高效率
  {
    lFirst = g_cpu.commonReg[EAX].nAll;
    lSecond = 立即数;
    _asm
    {
      mov esi, lFirst;
      cmp esi, lSecond;
      jnc cmp_rac_imm_cf_1;
      mov cCF, 1;
 cmp_rac_imm_cf_1:
      jno cmp_rac_imm_of_1;
      mov cOF, 1;
 cmp_rac_imm_of_1:
      jne cmp_rac_imm_zf_1;
      mov cZF, 1;
 cmp_rac_imm_zf_1:
      jns cmp_rac_imm_sf_1;
      mov cSF, 1;
 cmp_rac_imm_sf_1:
    }
  }
  else if(操作数大小是 2)
  {
    sFirst = g_cpu.commonReg[EAX]._x;
    sSecond = 立即数;
    _asm
    {
      mov si, sFirst;
      cmp si, sSecond;
      jnc cmp_rac_imm_cf_2;
      mov cCF, 1;
 cmp_rac_imm_cf_2:
      jno cmp_rac_imm_of_2;
      mov cOF, 1;
 cmp_rac_imm_of_2:
      jne cmp_rac_imm_zf_2;
      mov cZF, 1;
 cmp_rac_imm_zf_2:
      jns cmp_rac_imm_sf_2;
      mov cSF, 1;
 cmp_rac_imm_sf_2:
    }
  }
  else if(操作数大小是 1)
  {
    cFirst = g_cpu.commonReg[EAX]._8._l;
    cSecond = 立即数;
    _asm
    {
      mov dh, cFirst;
      cmp dh, cSecond;
      jnc cmp_rac_imm_cf_3;
      mov cCF, 1;
 cmp_rac_imm_cf_3:
      jno cmp_rac_imm_of_3;
      mov cOF, 1;
 cmp_rac_imm_of_3:
      jne cmp_rac_imm_zf_3;
      mov cZF, 1;
 cmp_rac_imm_zf_3:
      jns cmp_rac_imm_sf_3;
      mov cSF, 1;
 cmp_rac_imm_sf_3:
    }
  }

  g_cpu.flagReg[ZF] = cZF; //修正仿真 cpu 中的某些条件位
  g_cpu.flagReg[OF] = cOF;
  g_cpu.flagReg[CF] = cCF;
  g_cpu.flagReg[SF] = cSF;

  g_cpu.eip += 指定大小; //eip 指向下条待执行指令

  return 0;
}
```

当然这种方法对某些指令的仿真是行不通的。

### 2. 完全手工模拟

还以 `opcode=0x3c` 为例，这里简单点说明，略过很多情况

```
s32 cmp_rac_imm_fun()
{
  s32 lFirst;
  s16 sFirst;
  s8 cFirst;

  //这里为了说明问题，阿拉就不模拟对 OF,CF 的影响了，我怕麻烦

  if(操作数大小是 4) //将最可能执行到的条件放前面，以提高效率
  {
    lFirst = g_cpu.commonReg[EAX].nAll - 立即数;
    g_cpu.flagReg[ZF] = (lFirst == 0);
    g_cpu.flagReg[SF] = (lFirst < 0);
  }
  else if(操作数大小是 2)
  {
    sFirst = g_cpu.commonReg[EAX]._x - 立即数;
    g_cpu.flagReg[ZF] = (sFirst == 0);
    g_cpu.flagReg[SF] = (sFirst < 0);
  }
  else if(操作数大小是 1)
  {
    cFirst = g_cpu.commonReg[EAX]._8._l - 立即数;
    g_cpu.flagReg[ZF] = (cFirst == 0);
    g_cpu.flagReg[SF] = (cFirst < 0);
  }

  g_cpu.eip += 指定大小; //eip 指向下条待执行指令

  return 0;
}
```

通过以上两个方法，应该是可以搞定所有 ring 3 级别的指令的解析的。

下面说下，如何将识别出来的机器码跟该机器码对应的解析函数关联起来：我们在 x86 机器码的识别过程，可以给每个识别出来的 `opcode` 一个 `id`，而其相映的解析函数也用这个 `id`，然后将所有解析函数放入函数指针数组中，并通过这个 `id` 索引，这样就可以从 `opcode` 快速找到相映的解析函数了。

就写到这里吧，这个仿真 cpu 的功能还很简单，还有待加强......

> 个人建议，在 x86 平台上仿真 x86，没有必要识别所有指令，建立数组，标记带 `modr\m byte` 和 `string` 类型的 `opcode`。重定向寻址指令就行了

> 牛人写代码就是快，我构思 `VM` 比你早，但是你的框架写出来了，我还在想实现思路![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif")

> 不过我的是用在加密上的，而不是解密![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

>  引用: *最初由 **笨笨雄**发布* ![查看帖子](img/showthread.php?p=295581#post295581) 个人建议，在 x86 平台上仿真 x86，没有必要识别所有指令，建立数组，标记带 `modr\m byte` 和 `string` 类型的 `opcode`。重定向寻址指令就行了

> 雄大哥，不知道能否说得详细点呢，觉得你这个思路很好，阿拉想了解下，寻址重定向前后应该做些什么呢？



> 由于阿拉资质有限，目前还没有想明白，你这个思路能不能在我这里用，如果能用的话，又该怎么用...... 

> 首先是解释 `modr\m byte`，如果是不带寄存器的寻址，直接改地址就好了  

> 如果是带寄存器寻址的，可以在代码运行缓冲区中改写这条指令。举一个简单的例子吧  

> 假设有以下代码  

```
mov eax,00400000
xor ecx,ecx
mov edx,dword ptr [eax+4*ecx]
```

> 前两条指令不带 `modr\m byte` 的，在计算完指令长度之后，可以将一条完整的指令复制到代码执行缓冲区中。第三条指令，由于寻址是不确定的。因此在代码执行缓冲区中写入一条 `jmp` 到自己虚拟机里的。也就是缓冲区被写成下面的样子：  

```
mov eax,00400000
xor ecx,ecx
jmp VM_reAddr_proc
```

> 然后按照实际情况的 `eax` 和 `ecx` 便可以知道确切的值了。计算实际加载地址与默认地址的差值，然后加上去不就好了。如果你担心程序自己申请内存，并对跟内存寻址的情况。那么你在模拟分配内存函数的时候，将申请地址的返回值，也加上那个差值。便可以用统一的算法解决所有寻址问题了。计算出程序原本需要访问地址，生成  

```
mov reg32,dword ptr [xxxxxxxx]
```

> 注意环境保护，你可以用真实 CPU 的寄存器。切换堆栈也就 `mov esp,dword ptr [xxxxxxx]`。  

> 在最初构思的时候，我是想如何提高虚拟机速率的。不过最近发现，编译器使用了大量带 `modr\m byte` 的代码。这样的方法不见得能提高多少性能。当然了，这跟我的方向不同，没深入想过 

`4dd466d0e1e0cc123afbe504da2462ca`
