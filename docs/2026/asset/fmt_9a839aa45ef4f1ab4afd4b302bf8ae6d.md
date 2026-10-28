
# [原创]Win32平台下感染coff-obj文件方法

这是早些时候提交给Ph4nt0m Webzine 0x05的paper，现在已经发布了，仅供有这方面研究的朋友参考。感染`obj`可以有很多办法，这个只是最简单的一个。

## [0x01] 简介

本文介绍了如何感染win32平台的coff-obj格式的文件。在没有`rebuild all`情况下，被感染的coff-obj文件将被链接器从新合并到新文件当中，病毒代码也就将寄宿于产生的EXE文件中。

## [0x02] 感染思路

可感染obj文件的病毒非常少，主要集中在感染DOS时期的com文件，最早的感染obj文件的病毒是Stormbringer在1993年编写的shifter。charme的blog上有关于这个的详细分析《感染OBJ文件》。在win32平台下貌似还没看到有感染coff-obj格式的病毒，下面将演示如何去感染coff-obj文件的实现思路。

主要的想法，还是要获得coff-obj文件中代码执行的控制流程，但受coff-obj格式限制，使得感染方式不可能像PE文件那样灵活。但仍然可以用很多思路来突破感染这些瓶颈。

| PE文件的主要感染思路 | coff-obj |
| --- | --- |
| 1 感染文件头 | 头部无可操作空间 |
| 2 修改EOP | 没有EOP的概念 |
| 3 添加新节 | 可以尝试 |
| 4 EPO（入口模糊） | 可以尝试 |
| 5 hook 导入表 | 可以尝试 |
| 6 PE捆绑 | 链接器执行会出问题 |
| 7 ... | ... |

这里采用比较稳妥的方式来获得控制流程，就是重新构造一个cof-obj格式的`.text`段来获得控制流程，下面先看一下coff-obj的文件格式。

## [0x03] coff-obj格式

coff-obj文件格式比较清晰，由文件头+可选头+段+数据+重定位+符号组成。对obj文件来说是没有可选头的概念的，所以后面提到的coff-obj专指生成的目标文件格式，简称obj，下面是要构造一个新的`.text`段的示意图，实际段的排列和重定位的数据是混合的，此处仅是为了方便描述，简化处理了。

```text
+--------------+
| FILEHDR      |
+--------------+
| SECHDR1      | -----------\假设此处是原.text段，修改节数据使它指向新的添加数据
+--------------+            |
| SECHDR2      |            |
+--------------+            |
| ...          |            |
+--------------+            |
|  SECHDR N    |            |
+--------------+            |
|  SECDATE     |            |
+--------------+            |
|  LINE        |            |
+--------------+            |
|  Symbol      |            |
+--------------+            |
|  String      |            |
+--------------+            |
| new data     | <----------.
+--------------+
```

下面使用一个简单的C程序`t_obj`来详细说明obj格式。

```c
#include <stdio.h>

int main()
{
    char *title = "hello world!";
    printf(title);
}
```

执行`cmd > dumpbin /all /disasm t_obj.obj`，截取主要部分打印数据：

```text
//-------------------------------- 文件头 -------------------------------
FILE HEADER VALUES

     14C machine (i386)
       4 number of sections *** 重要的结构，这告诉我们这个obj文件有几个段，而我们关心的就是.text段
  4C6DF71D time date stamp
     125 file pointer to symbol table
      10 number of symbols
       0 size of optional header
       0 characteristics

//-------------------------------- .drectve段 -----------------------------
//----该段在obj文件被link过程中会被舍弃掉，主要提供给link的命令参数-------

SECTION HEADER #1

.drectve name
       0 physical address
       0 virtual address
      26 size of raw data
      B4 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
  100A00 flags
         Info
         Remove
         1 byte align

RAW DATA #1

  00000000: 2D 64 65 66 61 75 6C 74 6C 69 62 3A 4C 49 42 43  -defaultlib:LIBC
  00000010: 20 2D 64 65 66 61 75 6C 74 6C 69 62 3A 4F 4C 44  -defaultlib:OLD
  00000020: 4E 41 4D 45 53 20                                NAMES

   Linker Directives
   -----------------
   -defaultlib:LIBC
   -defaultlib:OLDNAMES

//-------------------------------- .text段 -----------------------------

SECTION HEADER #2

   .text name
       0 physical address
       0 virtual address
      10 size of raw data                   -----> 段长度，也就是实际编码长度
      DA file pointer to raw data           -----> .text中代码，相对文件头部的偏移
      EA file pointer to relocation table   -----> 指向.text中需要重定位的数据指针
       0 file pointer to line numbers
       2 number of relocations              -----> .text中代码，需要被修正的重定位数量
       0 number of line numbers
  60501020 flags
         Code
         Communal; sym= _main
         16 byte align
         Execute Read

_main:
  00000000: 68 00 00 00 00     push    offset _main   ---->此处偏移是0x00000000，还没有被重定位
  00000005: E8 00 00 00 00     call    0000000A       ---->此处偏移是0x00000000，还没有被重定位
  0000000A: 59                 pop     ecx
  0000000B: C3                 ret
  0000000C: 90                 nop
  0000000D: 90                 nop
  0000000E: 90                 nop
  0000000F: 90                 nop

RAW DATA #2

  00000000: 68 00 00 00 00 E8 00 00 00 00 59 C3 90 90 90 90  h.........Y.....
```

具体`0x68`，`0xe8`后面的值需要由编译器来绝对。

```text
//--------------------- .text段中的被修正的重定位表 --------------------
//最终代码就是根据该表中的值给链接器提供信息做最后的重定位依据

RELOCATIONS #2

                                                Symbol    Symbol
 Offset    Type              Applied To         Index     Name
 --------  ----------------  -----------------  --------  ------
 00000001  DIR32                      00000000         D  ??_C@_0N@NHHG@hello?5world?$CB?$AA@ (`string")
 00000006  REL32                      00000000         A  _printf
```

这里说一下，重定位的类型有3类：

- `DIR32` ---> 直接重定位，多是字符串一类情况，需要有链接器最终定位具体的虚拟地址值
- `REL32` ---> 相对重定位，当前opcode，相对于要跳转的函数的相对偏移值
- `DIR32NB` ---> 供调试信息使用，与`DIR32`的区别是重定位值不包含可执行文件的默认加载地址

`00000001`、`00000006`表示相对`.text`代码处要修正的位置偏移，也就是`[]`括起来的部分：

```text
00000000: 68 [00] 00 00 00     push        offset _main
00000005: E8 [00] 00 00 00     call        0000000A
```

```text
//-------------------------------- .data段 -----------------------------

SECTION HEADER #3

   .data name
       0 physical address
       0 virtual address
       D size of raw data
      FE file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
C0301040 flags
         Initialized Data
         Communal; sym= "`string"" (??_C@_0N@NHHG@hello?5world?$CB?$AA@)
         4 byte align
         Read Write

RAW DATA #3

  00000000: 68 65 6C 6C 6F 20 77 6F 72 6C 64 21 00           hello world!.
```

数据的位置最后由链接器来定义。

下面看一下`t_obj.obj`在IDA中的显示情况：

```text
.text:00000000                  public _main
.text:00000000                  _main           proc near
.text:00000000 68 10 00 00 00   push    offset ??_C@_0N@NHHG@hello?5world?$CB?$AA@ ; "hello world!"
.text:00000005 E8 16 00 00 00   call    _printf
.text:0000000A 59               pop     ecx
.text:0000000B C3               retn
.text:0000000B                  _main           endp
```

IDA根据重定位表的格式，对`.text`段分别修正为`0x10`、`0x16`两个偏移。同时当你增加`.text` coe大小时，链接器会自动调整`.data`的位置。

最终生成的`t_obj.EXE`文件显示情况：

```text
00401000   /$  68 30704000      push t_obj.00407030  ;  ASCII "hello world!"
00401005   |.  E8 06000000      call t_obj.00401010
0040100A   |.  59               pop ecx
0040100B   \.  C3               retn
```

字符串`"hello world!"`被定位到了`0x00407030`。`printf`函数被定位到相对偏移`0x05 + 0x6 = 0x0b`的位置，也即是虚拟地址为`401010`。

此时我们知道，如果要修改obj文件的`.text`代码是要注意的，要避免修改到重定位的部分，否则你的代码也会被链接器改写。

## [0x04] 修改宿主数据

1. 修改`.text`结构使其指向新加入的病毒代码位置，下面是段的结构体类型：

```c
typedef struct _sec_hdr
{
    char c_name[8];              // 段名
    unsigned long ul_v_size;     // 虚拟大小
    unsigned long ul_v_addr;     // 虚拟地址
    unsigned long ul_sec_size;   // 段长度
    unsigned long ul_sec_off;    // 段数据偏移
    unsigned long ul_rel_off;    // 段重定位表偏移
    unsigned long ul_lno_off;    // 行号表偏移
    unsigned short ul_num_rel;   // 重定位表个数
    unsigned short ul_num_ln;    // 行号表长度
    unsigned long ul_flags;      // 段标识
} sec_hdr;
```

`ul_sec_size` --> 修改为原代码的长度+病毒代码长度+重定位表长度

`ul_sec_off` --> 指向文件末尾

2. 在分析中发现一个问题，如果把原`.text`的代码拷贝到新的空间中而不拷贝重定位数据，会导致链接器无法执行。生成的EXE文件也无法执行，因为那部分数据被病毒代码填充，链接器无法解析。
