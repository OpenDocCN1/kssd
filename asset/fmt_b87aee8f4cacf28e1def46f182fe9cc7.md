
0167:00411CC6  PUSH  65 <mark>常数 65 压栈</mark>

0167:00411CC8  MOV  ECX,\[EBP+08\] <mark>\[EBP+08\]为对话框句柄</mark>

0167:00411CCB  PUSH  ECX

0167:00411CCC  PUSH  0042009C <mark>地址 0042009C 指向字符串“.\\EXEMEUE.DAT”</mark>

0167:00411CD1  LEA  EDX,\[EBP+FFFFDCA4\] <mark>\[EBP+FFFFDCA4\]为解密后的 KEY FILE 数据</mark>

0167:00411CD7  PUSH  EDX

0167:00411CD8  MOV  EAX,\[EBP+FFFFDC9C\] <mark>\[EBP+FFFFDC9C\]为前面验证程序的返回值</mark>

0167:00411CDE  PUSH  EAX

0167:00411CDF  CALL  0041A240 <mark>继续验证 KEY FILE“\.EXEMEUE.DAT”，作用和 98 系列的 CALL 0040C780 完全一样</mark>

0167:00411CE4  TEST  EAX,EAX

0167:00411CE6  JZ  00411DA9

0167:00411CEC  PUSH  00

0167:00411CEE  LEA  ECX,\[EBP\-50\]

0167:00411CF1  PUSH  ECX

0167:00411CF2  CALL  \[KERNEL32!\_lopen\] <mark>打开 KEY FILE 文件“c:\\EXEMEUE.DAT”</mark>

0167:00411CF8  MOV  \[EBP\-00A8\],EAX

0167:00411CFE  PUSH  00

0167:00411D00  PUSH  0042009C

0167:00411D05  CALL  \[KERNEL32!\_lcreat\] <mark>在 EXESPYME 的当前目录创建文件“\.EXEMEUE.DAT”</mark>

0167:00411D0B  MOV  \[EBP\-00AC\],EAX

0167:00411D11  CMP  DWORD PTR \[EBP\-00AC\],-01

0167:00411D18  JNZ  00411D3F

0167:00411D1A  PUSH  10

0167:00411D1C  PUSH  004251F0

0167:00411D21  PUSH  004251F8

0167:00411D26  MOV  EDX,\[EBP+08\]

0167:00411D29  PUSH  EDX

0167:00411D2A  CALL  \[USER32!MessageBoxA\]

0167:00411D30  MOV  EAX,\[EBP\-00A8\]

0167:00411D36  PUSH  EAX

0167:00411D37  CALL  \[KERNEL32!\_lclose\]

0167:00411D3D  JMP  00411DA7

0167:00411D3F  PUSH  50

0167:00411D41  LEA  ECX,\[EBP\-00A0\]

0167:00411D47  PUSH  ECX

0167:00411D48  MOV  EDX,\[EBP\-00A8\]

0167:00411D4E  PUSH  EDX

0167:00411D4F  CALL  \[KERNEL32!\_hread\] <mark>读文件“c:\\EXEMEUE.DAT”</mark>

0167:00411D55  MOV  \[EBP+FFFFDC98\],EAX

0167:00411D5B  CMP  DWORD PTR \[EBP+FFFFDC98\],00

0167:00411D62  JLE  00411D81

0167:00411D64  MOV  EAX,\[EBP+FFFFDC98\]

0167:00411D6A  PUSH  EAX

0167:00411D6B  LEA  ECX,\[EBP\-00A0\]

0167:00411D71  PUSH  ECX

0167:00411D72  MOV  EDX,\[EBP\-00AC\]

0167:00411D78  PUSH  EDX

0167:00411D79  CALL  \[KERNEL32!\_hwrite\] <mark>写文件“\.EXEMEUE.DAT”</mark>

0167:00411D7F  JMP  00411D3F

0167:00411D81  MOV  EAX,\[EBP\-00AC\]

0167:00411D87  PUSH  EAX

0167:00411D88  CALL  \[KERNEL32!\_lclose\] <mark>关闭文件“\.EXEMEUE.DAT”</mark>

0167:00411D8E  MOV  ECX,\[EBP\-00A8\]

0167:00411D94  PUSH  ECX

0167:00411D95  CALL  \[KERNEL32!\_lclose\] <mark>关闭文件“c:\\EXEMEUE.DAT”</mark>

0167:00411D9B  PUSH  01

0167:00411D9D  MOV  EDX,\[EBP+08\]

0167:00411DA0  PUSH  EDX

0167:00411DA1  CALL  \[USER32!EndDialog\]

0167:00411DA7  JMP  00411DD5

0167:00411DA9  PUSH  00

0167:00411DAB  PUSH  0042009C

0167:00411DB0  CALL  \[KERNEL32!\_lcreat\]

0167:00411DB6  MOV  \[EBP\-00AC\],EAX

0167:00411DBC  MOV  EAX,\[EBP\-00A8\]

0167:00411DC2  PUSH  EAX

0167:00411DC3  CALL  \[KERNEL32!\_lclose\]

0167:00411DC9  PUSH  00

0167:00411DCB  MOV  ECX,\[EBP+08\]

0167:00411DCE  PUSH  ECX

0167:00411DCF  CALL  \[USER32!EndDialog\]

0167:00411DD5  JMP  00411DED

0167:00411DD7  PUSH  10

0167:00411DD9  PUSH  00425220

0167:00411DDE  PUSH  00425228

0167:00411DE3  MOV  EDX,\[EBP+08\]

0167:00411DE6  PUSH  EDX

0167:00411DE7  CALL  \[USER32!MessageBoxA\]

0167:00411DED  JMP  00411DFB

0167:00411DEF  PUSH  00

0167:00411DF1  MOV  EAX,\[EBP+08\]

0167:00411DF4  PUSH  EAX

0167:00411DF5  CALL  \[USER32!EndDialog\]

0167:00411DFB  JMP  00411E0F

0167:00411DFD  PUSH  00

0167:00411DFF  MOV  ECX,\[EBP+08\]

0167:00411E02  PUSH  ECX

0167:00411E03  CALL  \[USER32!EndDialog\]

0167:00411E09  JMP  00411E0F

0167:00411E0B  XOR  EAX,EAX

0167:00411E0D  JMP  00411E14

0167:00411E0F  MOV  EAX,00000001

0167:00411E14  MOV  ESP,EBP

0167:00411E16  POP  EBP

0167:00411E17  RET  0010

...

22. 上面的程序跟 98 系列是一样的，只不过验证“.DAT” KEY FILE 时其验证字符串多了个前缀“\.”（98 系列为“SET98UR.DAT”, ME 系列为“\.EXEMEUE.DAT”)，程序后面同样是将 KEY FILE“EXEMEUE.DAT”拷贝到 EXESPYME 的当前目录下；

23. 好了，到这里我就不再往后讲了，因为这篇文章主要是讨论 98 系列跟 ME 系列的不同之处（主要是针对 KEY FILE“EXEMEKEY.DLL”），从这里以后的程序虽然跟 98 系列的不完全一样（因为 ME 系列是“SPY2”型的嘛！），但是对于有效的注册 KEY FILE 来说（REG_TYPE="REGI"），程序功能作用完全一致，也就是说从这里以后可以继续沿用 98 系列的注册机算法，只需要将相应的字符串信息换掉即可，后面程序的具体细节自己去看吧，这篇文章的目的已经达到，我要休息啦，真的好累，啊啊啊。。。！！！

24. 注意：附件所带的注册机 C 语言源程序中 KEY FILE“EXEMEKEY.DLL”的产生方法是先用汇编生成“EXEMEKEY.DLL”，然后将整个“EXEMEKEY.DLL”文件的数据放在源程序中，并根据程序需要修改相应部分的数据（对应产生的 KEY FILE“EXEMEUR.DAT”），另外程序中“EXEMEKEY.DLL”对应的汇编源程序和上面讲解的汇编源程序有一点不同，所以希望大家不要奇怪为什么源程序中的程序结构和破解文件中不一致；

25. 附录：下面放上程序中用到的几个相关 API 函数的原型：

```
HINSTANCE LoadLibrary(   
    LPCTSTR lpLibFileName     // address of filename  of executable module   
 );   

FARPROC GetProcAddress(   
    HMODULE hModule,    // handle to DLL module    
    LPCSTR lpProcName     // name of function   
 );   

BOOL FreeLibrary(   
    HMODULE hLibModule     // handle to loaded library  module   
 );
```

***

标题:注册机 (33 千字)

- 作 者：:娃娃[CCG]
- 时 间：2001-11-19 17:47:43

详细信息:

/* key_file Generator for ExeSpyME V5.0 Author: ddcrack  Written by: 2001/10/08 */

/* #define Program_name  "ExeSpy Millennium Edition for Windows ME/98/95"*/ /* addr: ebp+0x018 */

  #define key_file_name  "EXEMEUR.DAT"   /* addr: ebp+0x060 */

  #define dll_file_name  "EXEMEKEY.DLL"  /* generated dll file name*/

  #define dll_data_memo_addr 0x00429394  /* the location of dll data in system memory */

                                 /* got by finding "00000298" in W32Dasm or tracing in softice */

  #define Version        0x00000005       /* addr: ebp+0x000 */

/* #define Username       input by user    /* addr: ebp+0x160 */

/* #define SN_Header       "PD"             /* addr: ebp+0x26C */

/* #define SN_Tail         "ES"             /* addr: ebp+0x28C */

  #define VersionEx       0x00000001       /* addr: ebp+0x260 */

/* #define User_ID         generated by random addr: ebp+0x264 */

  #define Reg_Type       0x52454749       /* addr: ebp+0x268 */

  /* Reg_Type: 0x52454749 --> "REGI"

               0x53485050 --> "SHPP"

         0x4556414C --> "EVAL" */

  #define Free_year     2025             /* free update date addr: ebp+0x05C */

  #define Free_month     12                 /* Free_year=year+7CD, max year=0x3F*/

  #define Free_day       12                 /* year<=0x1F=31, so Free_year<=2028*/

  #define Free_hour     12

  #define Free_minute   12

  #define Free_second   12

  #define addr_004256EC ***********       /* function unknown, can be any value */

  #define addr_004256E8 0x00429FA0        /* one of XOR factors, can be any value */

  #define addr_004256E4 0x00000008        /* one of XOR factors, can be any value */

  #define addr_004256E0 0x006BE048        /* one of XOR factors, can be any value */


#include <stdio.h>

#include <string.h>

#include <conio.h>

#include <stdlib.h>

#include <time.h>


void main(void)

{

    unsigned char key_file[]=

    {'S','P','Y','2',0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

    0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,

         0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,/*encoded data start addr: ebp+0x00*/

    0x00,0x00,0x00,0x00,
