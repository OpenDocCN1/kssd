
# [原创]为了理解反汇编引擎而写的X86/X64反汇编引擎

自己学习逆向一段时间了，发现很不得法。就想自己写个东西来巩固一下。翻了论坛，发现很多人都写了自己的反汇编引擎。自己翻看了获益良多。但是发现没有人详细的写过X64指令的解析过程。所以自己就打算写一个X64的反汇编引擎。后来研究过后发现，其实X86和X64的解析差不多的，就把X86和X64一起合起来写了。为了写一个反汇编引擎发现自己也是够拼了。自己翻译了Intel手册的几个章节。后来发现intel手册的一个很大缺点，就是X64的说明很少。例如X64下的ModR/M、SIB解析表根本没有。我就去翻看了AMD的文档，发现了宝贝。AMD不愧是X64的发明人，手册把X64的各种资料都包含了进去。自己很是兴奋，就把介绍指令概述的章节给翻译了。能够把X86/X64讲明白的，我就服AMD的文档。他用一张图把怎么解析指令给概括了。就是下图

![](https://bbs.kanxue.com/upload/attach/202002/620577_XZ5J52VBUCQB2KC.png)

原本我在看完Intel文档之后各种思路被这个图串联起来了。这里不能不说一声AMD YES！还有就是要感谢大神@mik，是大神的文档帮我省了很多时间。我写代码的过程中无意中发现了大神的文档。是从X度文库下来下的，花了我一个月的会员啊，为了一篇文档。不过大神的网站上不了了。非常遗憾。附上大神在看雪的主页[@mik](https://bbs.pediy.com/user-202613.htm)。

我在写的过程之中发现X86/X64指令的解析过程很麻烦，有些前面部分的解析和后面的字节有关，后面的字节的解析也可能和前面的字节有关。指令各部分长度的不一致。写得我心烦。接下来我会先分析Intel/AMD手册说明解析一条指令跟什么有关系。让然后再逐个分析解析指令的各个部分的详细过程。

> 手册里对解析指令最重要的就是OPCODE TABLES。一个字节是一条指令里的什么部分都能从OPCODE TABLES里面查出来。

## OpcodeMap里面的包含的元素

（1）Opcode：例如PUSH（注意缩写还有就是简写）一对多的放在类型说明里面

（2）Addressing Method + Operand Type：例如Gv, Ev 一对多的单独作为一个类型

（3）Register Codes：例如rBX/r11 一对多的单独作为一个类型

（4）Superscripts：例如1A、i64 视情况而定，也是比较少。不适合做单独的类型

（5）Opcode Extensions Group Number：例如Grp 2（intel的前面可能会有对该组的描述性文字如Immediate）

（6）Prefix：例如：Operand Size (Prefix)

（7）REX Prefix：例如：REX.X 类型说明

（8）描述性文字：例如：word, double-word or quad-word register with rAX

（9）距离：例如：near 类型说明，或者其他

（10）说明位于其他表：例如：3-byte escape，VEX+2byte

（11）空白：直接类型说明，为0，与通用的一样，不过后面的opcode没有name

（12）异常代码：例如：UD0 太少不放进去结构体了

加粗的都是关键，其他的只是说明注解。

## 那么应该怎么使用OPCODE TABLES

建议大家去看Intel文档的Volume 2 (2A, 2B, 2C & 2D):Instruction Set Reference, A-Z中的Appendix A Opcode Map面的A.1 USING OPCODE TABLES，我已经做了翻译（见附件）。我概括如下

1、看位于哪个OPCODE表

2、取出指令，寻址类型还有操作数类型

3、注意符合寄存器表示方式的解析与操作数类型相关

4、写死在opcode里面的寄存器需要表示，这个是单独的。跟第5点不是存在一样的表示

5、根据寻址类型还有操作数类型确定后面是否有ModR/M字节、SIB字节、偏移量字节（1、2、4、8字节）、立即数字节（1、2、4、8字节）。

6、依次解码opcode还有操作数1-3

我的代码是首先构造一个和OPCODE表`const OPCODE_STRUCT Primary_opcodemap[0x10][0x10]`，这个表是由结构体数组组成的，其实就是AMD文档里的OPCODE TABLES的基本结构

```
typedef struct OPCODE_STRUCT
{
	const char *opcode;
	int  AddressingMethod1st;
	int  OperandType1st;
	int  AddressingMethod2nd;
	int  OperandType2nd;
	int  AddressingMethod3rd;
	int  OperandType3rd;
	int  SelfType;     //该指令字节码的类型
}OPCODE_STRUCT, *POPCODE_STRUCT;
```

因为OPCODE TABLES里包含各种各种类型的元素，有些OPCODE有多种解析方式。所以在解析之前需要做处理。

```
	//解析具体的opcode，先返回opcode字符串，还有就是处理操作数寻址方式还有操作数大小，方便后面统一处理
	//开始先对三个操作数的寻址方式和操作数类型做默认的处理
	decode_struct.AddressingMethod[0] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].AddressingMethod1st;
	decode_struct.OperandType[0] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].OperandType1st;
	decode_struct.AddressingMethod[1] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].AddressingMethod2nd;
	decode_struct.OperandType[1] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].OperandType2nd;
	decode_struct.AddressingMethod[2] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].AddressingMethod3rd;
	decode_struct.OperandType[2] = popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].OperandType3rd;
	switch (popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].SelfType)
	{
	case 0://通用的类型opcode解析
		Result = Result + popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].opcode;
		//还是在这里处理寻址方式和寻址类型然后返回相关信息，接下来再根据相关信息去解析出后面的字节（等指针移动之后）。
		//通用的类型通用类型的直接获取opcode字符串，然后获取三个操作数的寻址方式还有操作数类型
		break;
	case PUPO:
		if (REX_B_Flag)
		{
			decode_struct.AddressingMethod[0] = decode_struct.AddressingMethod[1];
			decode_struct.AddressingMethod[1] = UNCON;
		}
		else
			decode_struct.AddressingMethod[1] = UNCON;
		Result = Result + Result + popcodemap[(int)HighNibble(*POpcode)][(int)LowNibble(*POpcode)].opcode;
		break;
//篇幅有限，只是贴一部分代码
```

先根据`SelfType`还有具体的情况确认opcode，然后再确定寻址方式还有寻址类型（需要做统一的处理）。

```
	//先判断opcode后面的字节有没有ModRM字节
	for (int icount = 0; icount <3;icount++)
	{
		switch (decode_struct.AddressingMethod[icount])
		{
		case C:
		case D:
		case G:
		case P:
		case S:
		case V:
		case E:
		case M:
		case N:
		case Q:
		case R:
		case U:
		case W:
			ModRM_Flag = true;
			if (0x4== RM(*(POpcode + 1)) && 0x3 != MOD(*(POpcode + 1)))//当ModRM.r/m == 100（0x4）且Mod ≠11b时，存在SIB字节
				SIB_Flag = true;
			if (0x0 == MOD(*(POpcode + 1)) && 0x5 == RM(*(POpcode + 1)))//当ModRM.r/m == 101（0x5）且MOD == 0时，disp为4字节
				idispcount = 4;
			if (0x1 == MOD(*(POpcode + 1)))//当ModRM.MOD == 1时，disp为1字节
				idispcount = 1;
			if (0x2 == MOD(*(POpcode + 1)))//当ModRM.MOD == 2时，disp为4字节
				idispcount = 4;
			if (true == SIB_Flag && 0x5 == BASE(*(POpcode + 2)) && 0x0 == MOD(*(POpcode + 1)))//有SIB且base域为5，当ModRM.MOD == 0时，disp为4字节
				idispcount = 4;
			if (true == SIB_Flag && 0x5 == BASE(*(POpcode + 2)) && 0x1 == MOD(*(POpcode + 1)))//有SIB且base域为5，当ModRM.MOD == 1时，disp为1字节
				idispcount = 1;
			if (true == SIB_Flag && 0x5 == BASE(*(POpcode + 2)) && 0x2 == MOD(*(POpcode + 1)))//有SIB且base域为5，当ModRM.MOD == 2时，disp为4字节
				idispcount = 4;
			break;
		default:
			break;
		}
	}
```

确定一条指令包含什么部分：

先确定有没有ModRM字节-->然后确定有没有SIB字节

|

I-->确定有没有Disp字节（有几字节？）3的寻址方式还有类型确认后面的有什么字节

再确定有没有immediate字节（有几字节）这个可以到最后解码immediate字节的时候再做。

## 一、先解码寄存器操作数

![](https://bbs.kanxue.com/upload/attach/202002/620577_AN6YGX8PRBHBB4G.jpg)

![](https://bbs.kanxue.com/upload/attach/202002/620577_E95R9678K5JXB5E.jpg)

```
//General Registers in Legacy and Compatibility Modes OR General Registers in 64-Bit Mode Without REX
const char *GPRs32[0x3][0x8] =
{
  {"AL", "CL", "DL", "BL", "AH", "CH", "DH", "BH" },//8bit
  {"AX", "CX", "DX", "BX", "SP", "BP", "SI", "DI" },//16bit
  {"EAX","ECX","EDX","EBX","ESP","EBP","ESI","EDI"} //32bit
};

//General Registers in 64-Bit Mode With REX
const char *GPRs64[0x4][0x10] =
{
  {"AL", "CL", "DL", "BL", "SPL", "BPL", "SIL", "DIL", "R8B", "R9B", "R10B", "R11B", "R12B", "R13B", "R14B", "R15B"},//8bit
  {"AX", "CX", "DX", "BX", "SP" , "BP" , "SI" , "DI" , "R8W", "R9W", "R10W", "R11W", "R12W", "R13W", "R14W", "R15W"},//16bit
  {"EAX","ECX","EDX","EBX","ESP", "EBP", "ESI", "EDI", "R8D", "R9D", "R10D", "R11D", "R12D", "R13D", "R14D", "R15D"},//32bit
  {"RAX","RCX","RDX","RBX","RSP", "RBP", "RSI", "RDI", "R8" , "R9" , "R10" , "R11" , "R12",  "R13" , "R14" , "R15" } //64bit
};
		switch (decode_struct.AddressingMethod[icount])
		{
		case rAX:
		case rCX:
		case rDX:
		case rBX:
		case rSP:
		case rBP:
		case rSI:
		case rDI:
		case r8:
		case r9:
		case r10:
		case r11:
		case r12:
		case r13:
		case r14:
		case r15://解析直接出现的寄存器操作数
			if (UNCON != decode_struct.OperandType[icount])
				Result = Result + GPRs64[decode_struct.OperandType[icount]][decode_struct.AddressingMethod[icount]];
			else
			{
				if (MODE32 == Mode)
				{
					if (OPSizeFlag)
					    Result = Result + GPRs32[_16BIT][decode_struct.AddressingMethod[icount]];
					else
						Result = Result + GPRs32[_32BIT][decode_struct.AddressingMethod[icount]];
				}
				else
				{
					if (REX_W_Flag || REX_B_Flag)
						Result = Result + GPRs64[_64BIT][decode_struct.AddressingMethod[icount]];
					else
					{
						if (OPSizeFlag)
							Result = Result + GPRs64[_16BIT][decode_struct.AddressingMethod[icount]];
						else
							Result = Result + GPRs64[_32BIT][decode_struct.AddressingMethod[icount]];
					}
				}
			}
			break;
```

寄存器的解码

1、根据当前的模式还有就是有没有操作数大小覆盖前缀来确定（没有大小描述的情况）

2、直接解码返回（有大小描述的情况）

## 二、解码Mod.reg域
