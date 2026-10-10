

# 关于反汇编引擎的问题！

> 原文：[`www.pediy.com/kssd/pediy10/77856.html`](https://www.pediy.com/kssd/pediy10/77856.html)

最近在做对程序自动跟踪调试的工作。在用的反汇编引擎：libdasm1.5  
但是在处理 CALL 语句时出现问题：  
例：  
00401000   .  E8 0B000000   call    00401010  
00401005   . /E9 16000000   jmp     00401020  
在网上查了下，说是 0x401010 = 0x00401005 + 0x0B，  
但是用 libdasm 反汇编出的语句是  
call 0x15  
怎么会这样呢？  
另：  
如何通过 CALL XXXX 判断 XXXX 是用户自己定义的函数，还是系统调用呢？  
初步想是通过代码段地址范围来判断，但是动态载入的 DLL 呢，各位有什么其他好办法吗？  
谢谢！  
问题太简单，可能也有点偏，请各位提点思路，不胜感激！ 

反汇编成 call 0x0B 还是可以理解的，因为 E8 和 E9 的参数都是 16/32 为的相对偏移地址，符号扩展后加到 EIP（也就是下条指令的地址）上去就是新的 EIP，也就是翻译出来给我们看的地址。  
call 0x15 没有道理啊！你确定的确是被翻译成了 call 0x15？又没有什么运行的例子什么的？发出来一起研究研究？  
  
Call xxx 的对 xxx 的判断，你可以试试这个方法（菜鸟愚见，望高手毋笑）？  
1、首先判断该地址是否在 pe 文件的.txt/.code 代码映射后的空间范围类，是则表示跳转后的地址应该是程序自身。  
2、如果不是 1，读取 pe 文件的所有 import 的函数的地址，逐个比较。  
3、如果还不是，读取实例所有模块，比较地址。 

引用: *最初由 **egogg**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=543951#post543951) 反汇编成 call 0x0B 还是可以理解的，因为 E8 和 E9 的参数都是 16/32 为的相对偏移地址，符号扩展后加到 EIP（也就是下条指令的地址）上去就是新的 EIP，也就是翻译出来给我们看的地址。  
call 0x15 没有道理啊！你确定的确是被翻译成了 call 0x15？又没有什么运行的例子什么的？发出来一起... 

十分感谢 egogg。经你提醒，我试验了下所用的反汇编引擎，以下和 OD 做对比：  
OD：  
00401000   .  E8 0B000000   call    00401010  
00401017  |.  E8 D4B50000   call    0040C5F0  
00401025   .  E8 27090000   call    00401951  
用 libdasm 的结果：  
call 0x15  
call 0xb5de  
call 0x931  
反汇编的结果与相对偏移都相差 A，应该是这里的问题吧。  
对反汇编引擎实在不了解，刚搜到《打造自己的反汇编引擎》系列，感谢精彩分享。  
不知道怎么传附件，所用 libdasm 反汇编引擎地址如下：  
http://www.nologin.org/main.pl?action=codeView&codeId=49  
搞不懂当初为什么会用这个引擎，害人啊![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  
不知道除了 OD 的 dasm 外，还有什么常用的反汇编引擎呢？  
  
关于 CALL 操作数的问题，感谢你的三点建议，只是第二点是不是会非常耗时呢？每个 CALL 都要去遍历 import 表？我想只要操作数地址在程序 CODE 段或用户自己加载的 DLL 段，那就应该是用户函数了，但是如何区分系统的 DLL 还是用户自己加载的 DLL 呢？  
  
再次感谢 egogg 的及时答复和精彩建议！ 

引用: *最初由 **westboy**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=544009#post544009) 十分感谢 egogg。经你提醒，我试验了下所用的反汇编引擎，以下和 OD 做对比：  
OD：  
00401000   .  E8 0B000000   call    00401010  
00401017  |.  E8 D4B50000   call    0040C5F0  
00401025   . ... 

od 的反汇编引擎就挺好的啊，你当初为什么不就用 od 的呢？其他的反汇编引擎看雪老大的书上都列了，具体哪些我现在手头上没有书。他没有列的我知道的还有 nasm 的反汇编引擎，pvdasm。  
  
至于判断跳转指令的问题，做指令分析的话，这些遍历可能是不可避免的，具体实现我也不知道，ida 的文档中应该有相应的解决方案。还有就是，不一定每一个 call 都需要比较，只需要比较 far 调用，像 E8 这样的短跳转指令，正常应用都应该是程序自己代码的程序调用，远指令调用（指令以 9A、FF 开头）才需要比较地址范围。这部分我也就使曾经想过也不知道对不对，而且由于水平有限还没有试过，你可以试试，看看效果。  
  
我大致检查了一下，libdasm1.5 中并没有任何错误（关于 call near 这部分），应该是你在使用的过程中有不小心的地方，函数入口有一个 offset，这里 offset 的值被初始化成了 0xA，如果你不用这个值话，应该把这个值赋 0（这时你得到的就是 call 0x0B）或者变成当前指令的偏移地址（线性地址，这时你得到的就是 call 00401010）。你可以试一下。 

显示格式不同，  
  
显示格式是：　call $+offset        //这个$　指本条指令边界  
  
  
  
call 0x15              // call 00401010　对于$ 来说是偏移 0x15  
call 0xb5de          // call 0040c5f0    对于$ 来说是偏移 0xb5de  
call 0x931            // call 00401951  对于 $ 来说是偏移 0x931  
  
编码是一样的，只是显示不同而已 

引用: *最初由 **egogg**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=544028#post544028) od 的反汇编引擎就挺好的啊，你当初为什么不就用 od 的呢？其他的反汇编引擎看雪老大的书上都列了，具体哪些我现在手头上没有书。他没有列的我知道的还有 nasm 的反汇编引擎，pvdasm。  
  
至于判断跳转指令的问题，做指令分析的话，这些遍历可能是不可避免的，具体实现我也不知道，ida 的文档中应该有相应的... 

非常感谢 egogg 和 mik 的答复，问题解决了。确实是 offset 的问题，只要传入当前指令的偏移地址就可以得到正确的反汇编指令，是我大脑短路了，应该仔细看看 sample![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")  
由于水量太菜，当时用 OD 的 dasm 的时候总是由于版本问题，一堆错误，就偷懒换了。  
  
再次感谢两位！ 

再弱问一下，那我是不是可以通过 call 指令的 near 或 far 来区分后面的操作数呢？  
看了 egogg 推荐的 coder table，发现 CALL 指令对应的代码只有：E8，FF，9A（这个还没见过）  
那我如果要区分 CALL XXXX 中 XXXX 的范围，  
例如：  
if（E8）  
{  
     if(min<XXXX<max)  
         blablabla;  
}  
else if(FF)  
{  
      if(min<DS:\[XXXX\]<max)  
          blablabla  
}  
else  
{  
     （9A？？？？）![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") ![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")   
}  
 想法太菜，各位见笑！ 

引用: *最初由 **westboy**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=544389#post544389) 再弱问一下，那我是不是可以通过 call 指令的 near 或 far 来区分后面的操作数呢？  
看了 egogg 推荐的 coder table，发现 CALL 指令对应的代码只有：E8，FF，9A（这个还没见过）  
那我如果要区分 CALL XXXX 中 XXXX 的范围，  
例如：  
if（E8）  
{  
     if(... 

我认为，根据 libdasm 的特点，不用比较 near far 什么的也行，只要是该跳转指令（INSTRUCTION\_TYPE\_CALL 或者 JMP），取串中的地址比较就行了。 

near 调用时，CALL 的操作数确实就是跳转地址，但是当是 far 时，或者需要取 DS：\[XXXX\]，或者类似：CALL EDI，再取 EDI 的值，有点麻烦。  
您说的 libdasm 的特点我不太明白指的是什么？ 

引用: *最初由 **westboy**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=544923#post544923) near 调用时，CALL 的操作数确实就是跳转地址，但是当是 far 时，或者需要取 DS：\[XXXX\]，或者类似：CALL EDI，再取 EDI 的值，有点麻烦。  
您说的 libdasm 的特点我不太明白指的是什么？ 

我是看到:   
代码:

```
typedef struct \_INSTRUCTION {
  int length;    // Instruction length
  enum Instruction type;  // Instruction type
  enum Mode mode;    // Addressing mode
  BYTE opcode;    // Actual opcode
  BYTE modrm;    // MODRM byte
  BYTE sib;    // SIB byte
  int modrm\_offset;  // MODRM byte offset
  int extindex;    // Extension table index
  int fpuindex;    // FPU table index
  int dispbytes;    // Displacement bytes (0 = no displacement)
  int immbytes;    // Immediate bytes (0 = no immediate)
  int sectionbytes;  // Section prefix bytes (0 = no section prefix)
  OPERAND op1;    // First operand (if any)
  OPERAND op2;    // Second operand (if any)
  OPERAND op3;    // Additional operand (if any)
  PINST ptr;    // Pointer to instruction table
  int flags;    // Instruction flags
  short eflags\_affected;  // Process eflags affected
  short eflags\_used;      // Processor eflags used by this instruction
  int iop\_written;  // mask of affected implied registers (written)
  int iop\_read;    // mask of affected implied registers (read)
} INSTRUCTION, \*PINSTRUCTION;

```

这个结构所说的。不需要判断 near，还是 far 调用。  
  
enum Instruction type;  // Instruction type  
  
可以根据这个来判断是不是 INSTRUCTION\_TYPE\_CALL/JMP，如果是则进行判断。由于跳转指令一般只有一个操作数，所以再根据 op1 来判断：  
代码:

```
typedef struct \_OPERAND {
  enum Operand type;  // Operand type (register, memory, etc)
  int reg;    // Register (if any)
  int basereg;    // Base register (if any)
  int indexreg;    // Index register (if any)
  int scale;    // Scale (if any)
  int dispbytes;    // Displacement bytes (0 = no displacement)
  int dispoffset;    // Displacement value offset
  int immbytes;    // Immediate bytes (0 = no immediate)
  int immoffset;    // Immediate value offset
  int sectionbytes;  // Section prefix bytes (0 = no section prefix)
  WORD section;    // Section prefix value
  DWORD displacement;  // Displacement value
  DWORD immediate;  // Immediate value
  int flags;    // Operand flags
} OPERAND, \*POPERAND;

```

根据  
  
enum Operand type;  
这个成员判断操作数的类型，直接根据操作数结构体中的各个成员来从相应的地方取值判断。  
  
我想说的就是，可以根据这些以有的信息来判断，而不是解析最后生成的字符串。解析字符串也挺好的，但是还是感觉这种方法相对更合适一点。  
  
总之，实践的人（比如说你自己）才有最发言权，我只是说说我的想法而已，能实现功能的方法才是好方法。 

呵呵，非常感谢 egogg。  
明白一些您的意思，利用引擎已得到信息确实可以减少很多工作。  
感谢您的精彩建议。![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")