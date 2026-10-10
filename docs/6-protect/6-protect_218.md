

# Dump The filter of flexlm

> 原文：[`www.pediy.com/kssd/pediy10/68402.html`](https://www.pediy.com/kssd/pediy10/68402.html)

发个 Flexlm filter 辅助程序， 下面是代码，原理就不说了，很多地方都介绍过了。  
感谢 Nolan Blender 和 Hex-Rays 作者。  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
Dump the filter\_gen  matrix of  flexlm  
code by Phydora  
  
I am not able to use the idc script, so I have to use my way to get the filter\_gen. I'm a Fresh man.  
  
tools: IDA disassembler and Hex-Rays decompiler  
借用他人的话说，Hex-Rays 这工具很强大，很邪恶！  
  
Use IDA disassmble the program that use flexlm,  
search  the instruction   " push   4Ah ", usually there are  only little.  
you will find out it quickly, the address of  the flexlm filter is above the instruction  "push 4Ah"    
for example :  
  
push    offset sub\_XXXXXXXX     //filter   
push    4Ah  
  
when you have the address of the filter,you cane use the Hex-Rays to decompiler it.  
and chang some names of the  variable,   
  
see: http://www.woodmann.com/crackz/Tutorials/Nbufilt.htm  
  
you must find num 0-19 and bit 0-7 to  channge their name to numx or bitx.then break the c code of the filter to two parts.  
one look like:  
  
  
 if ( idx == num10 )  
  {  
    if ( (\_BYTE)bit80 & v5 )  
      LOBYTE(c) = (\_BYTE)bit02 | (\_BYTE)c;  
  }  
  ...  
  
copy the c code above into the percoder()  
  
the other:  
  if ( idx == num2 && (\_BYTE)bit08 & (unsigned \_\_int8)((\_BYTE)c ^ ec) )  
    goto LABEL\_558;  
  if ( idx == num9 && (\_BYTE)bit01 & (unsigned \_\_int8)((\_BYTE)c ^ ec) )  
  {  
    v6 = (\_BYTE)c ^ 0x4D;  
LABEL\_1080:  
    \*inchar = v6;  
    return;  
  }  
  ...  
copy the c code above into the xorcode()  
  
complie this file and run, you wil get the filter matrix.  
  
the  percoder and xorcode contain the filter of some program  now.  
  
Thanks to Nolan Blender  
  
Enjoy it!  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
  
  
#include "stdio.h"  
  
static int num3 = 3;  
static int bit0 = 0x2801;  
static int num10 = 10;  
static int num2 = 2;  
static int num9 = 9;  
static int num1 = 1;  
static int num15 = 15;  
static int num0 = 0;  
static int num16 = 16;  
static int num4 = 4;  
static int num6 = 6;  
static int num20 = 20;  
static int num13 = 13;  
static int num5 = 5;  
static int num8 = 8;  
static int num12 = 12;  
static int bit1 = 0x102;  
static int num7 = 7;  
static int bit5 = 0xa720;  
static int bit3 = 0x3108;  
static int num18 = 18;  
static int bit6 = 0x740;  
static int bit7 = 0xcc80;  
static int num17 = 17;  
static int bit2 = 0x2704;  
static int num14 = 14;  
static int bit4 = 0x3210;  
static int num11 = 11;  
static int num19 = 19;  
  
  
  
 unsigned char c;  
  
 char b;  
  
 unsigned char in\_c;  
 //int  in\_c;  
 int idx;  
  
 char test;  
 char \*inchar;  
 const unsigned char expchar  = 0xff;  
  
void percoder(void);  
void xorcoder(void);  
  
void  main(void)  
{  
  int per \[20\]\[8\];  
  int xorval\[20\];  
  int i, j, k;  
    
  for (k = 0; k < 20; k++)  
  {  
    idx = k;  
  
  for( i = 0; i<8; i++)  
  {  
    in\_c = 1 << i;  
    c = 0;  
        percoder();  
  for( j = 0; j<8; j++)  
  {  
  
    if((1 << j) == c)  
    {  
      per\[k\]\[j\] = i;  
  
      break;  
    }  
  }  
  
  
  }  
  }  
  
  
  inchar = &test;  
  for (i = 0; i < 20; i++)  
  {  
    idx = i;  
    xorval\[i\] = 0;  
  
     b = 0;  
     for(j =0; j < 8; j++)  
     {  
       c = ~b;  
       xorcoder();  
       b=(~c)^b;  
     }  
     xorval\[i\] = b;  
  }  
  
  
  
FILE \*pf;  
pf = fopen("haha.txt", "w");  
fprintf(pf, "permute\_t tab1\[\] = {\\n");  
for(i = 0; i < 19; i++)  
{  
    fprintf(pf, "{");  
  for(j =0; j < 7; j++)   
    fprintf(pf, "%d ,", per\[i\]\[j\]);  
    fprintf(pf, "%d },   /\* idx = %d \*/ \\n", per\[i\]\[7\], i );  
}  
  
    fprintf(pf, "{");  
  for(j =0; j < 7; j++)   
    fprintf(pf, "%d ,", per\[19\]\[j\]);  
    fprintf(pf, "%d }    /\* idx = 19 \*/\\n", per\[19\]\[7\]);  
  
fprintf(pf, "};\\n");  
  
  
fprintf(pf, " \\n int xorvals\[\]  = \\n   { ");  
for(i = 0; i < 19; i++)  
fprintf(pf, "0x%x, ",xorval\[i\]);  
fprintf(pf, "0x%x };", xorval\[i\]);  
  
  
fclose(pf);  
  
}  
  
#define LOBYTE(x)  x  
  
#define \_BYTE  char   
  
  
//change the below  
void percoder()  
{  
  
  if ( idx == num16 )  
  {  
    if ( (\_BYTE)bit7 & in\_c )  
      LOBYTE(c) = bit1;  
  }  
...  
 if ( idx == num10 )  
  {  
    if ( (\_BYTE)bit7 & in\_c )  
      LOBYTE(c) = (\_BYTE)bit1 | (\_BYTE)c;  
  }  
}  
  
void xorcoder()  
  
{  
    
   unsigned \_\_int8 v6; // al@485  
  int v7; // eax@490  
  int v8; // eax@506  
  int v9; // eax@512  
  int v10; // eax@526  
  int v11; // eax@544  
  int v12; // eax@550  
  int v13; // eax@560  
  int v14; // eax@569  
  int v15; // eax@594  
  int v16; // eax@599  
  int v17; // eax@605  
  int v18; // eax@610  
  int v19; // eax@615  
  int v20; // eax@620  
  int v21; // eax@626  
  int v22; // eax@638  
  int v23; // eax@651  
  int v24; // eax@656  
  int v25; // eax@675  
  int v26; // eax@686  
  int v27; // eax@706  
  int v28; // eax@715  
  int v29; // eax@736  
  int v30; // eax@744  
  int v31; // eax@766  
  int v32; // eax@775  
  int v33; // eax@786  
  int v34; // eax@801  
  int v35; // eax@810  
  int v36; // eax@826  
  int v37; // eax@832  
  int v38; // eax@840  
  int v39; // eax@849  
  int v40; // eax@863  
  int v41; // eax@874  
  int v42; // eax@880  
  int v43; // eax@888  
  int v44; // eax@894  
  int v45; // eax@915  
  int v46; // eax@927  
  int v47; // eax@935  
  int v48; // eax@946  
  int v49; // eax@960  
  int in\_c0; // eax@972  
  int in\_c1; // eax@978  
  int in\_c2; // eax@983  
  int in\_c3; // eax@997  
  int in\_c4; // eax@1003  
  int in\_c5; // eax@1021  
  int in\_c6; // eax@1036  
  int in\_c7; // eax@1051  
  int in\_c8; // eax@1057  
  int in\_c9; // eax@1069  
if ( idx == num2 && (\_BYTE)bit3 & (unsigned \_\_int8)((\_BYTE)c ^ expchar) )  
    goto LABEL\_558;  
  if ( idx == num9 && (\_BYTE)bit0 & (unsigned \_\_int8)((\_BYTE)c ^ expchar) )  
  {  
    v6 = (\_BYTE)c ^ 0x4D;  
LABEL\_1080:  
    \*inchar = v6;  
    return;  
  }  
...  
if ( idx == num14 && (\_BYTE)bit3 & (unsigned \_\_int8)((\_BYTE)c ^ expchar) )  
    {  
LABEL\_1085:  
      v6 = (\_BYTE)c ^ 0x94;  
      goto LABEL\_1080;  
    }  
  }  
}

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || main.rar  
 |

我当时处理 filter 的时候累得很，而目标软件又是 linux 平台上的，gdb 我不大会用，所以就在 windows 下做这项工作。  
filter 的弱点是它不可能在 xor 和 permute 之间乱序，仅在 xor 和 permute 内部乱序，所以很容易将 xor 和 permuter 从 code 级分开，在 c code 中会看得更清楚一些，然后依据它们各自的特点构造特定的输入，便可从输出中扫描得到 filter。  
我在上面的程序中改 variable 的名字和使用 Hex-Rays tools 是折衷的，不必要的，  
是可以做出来 filter for windows 的自动提取工具的，我懒了，遇到解 filter 的情况也不多，就没去写。  
1)将 filter.exe 读入内存后暴力搜索，对 sub\_XXXXXXXX 做少许的一些字节特征检查，基本就能确定 filter.  
push    offset sub\_XXXXXXXX     //filter   
push    4Ah  
2)做一些字节模式的检查分开 xor 和 permute，它们的字节特征区别非常明显。  
3)修补一下内存中的 filter.exe 文件，先分离出 xor，但在进入 perpermute 前使函数返回，构造特定输入，修改 IP 对 filter.exe 文件的 filter 数据执行(不要开 windows 的 DEP)便可 dump 出 xor. dump permute 方法与 dump xor 类似。