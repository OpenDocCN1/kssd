
# 文件句柄分配的原理探测（一）---初步探测

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-145.htm`](https://www.pediy.com/kssd/pediy07/pediy7-145.htm)

```
/****...****/
//下面这段代码是 VC6.0 调试器反汇编后的结果
//原本是想随便写个小程序观察反汇编代码
//问题的出现:当用 OD 调试(debug 文件夹的 fread\_fwrite.exe)的时候,提示打开文件失败
//提出问题:为什么同一个程序在 VC 和 OD 的调试下会出现不同结果?
//如果读者愿意和我一起度过这个旅途的话,请不要忘记在程序文件夹下建立这个文件"zhang.pdf"(文件随便即可);最好源程序和这里一样,调试时方便对照.
/****...****/

1:    // fread\_fwrite.cpp : Defines the entry point for the console application.
2:    //
3:
4:    #include "stdafx.h"
5:    #include "iostream.h"
6:    #include "stdio.h"
7:    #include <windows.h>
8:
9:    int main(int argc, char* argv[])
10:   {
00401040   push        ebp
00401041   mov         ebp,esp
00401043   sub         esp,60h
00401046   push        ebx
00401047   push        esi
00401048   push        edi
00401049   lea         edi,[ebp-60h]
0040104C   mov         ecx,18h
00401051   mov         eax,0CCCCCCCCh
00401056   rep stos    dword ptr [edi]
11:       FILE *f\_open=fopen("zhang.pdf","rb");
00401058   push        offset string "rb" (00428064)
0040105D   push        offset string "zhang.pdf" (00428058)
00401062   call        fopen (00404800)
00401067   add         esp,8
0040106A   mov         dword ptr [ebp-4],eax
12:       FILE *f\_write=fopen("test3.pdf","wb+");
0040106D   push        offset string "wb+" (00428054)
00401072   push        offset string "test3.pdf" (00428048)
00401077   call        fopen (00404800)
0040107C   add         esp,8
0040107F   mov         dword ptr [ebp-8],eax
13:
14:       if(f\_open==NULL)
00401082   cmp         dword ptr [ebp-4],0
00401086   jne         main+0B1h (004010f1)
15:       {
16:           cout<<"OPEN--WRONG!"<<endl;
00401088   push        offset @ILT+10(endl) (0040100f)
0040108D   push        offset string "OPEN--WRONG!" (00428038)
00401092   mov         ecx,offset cout (0042b9c0)
00401097   call        ostream::operator<< (00401720)
0040109C   mov         ecx,eax
0040109E   call        @ILT+0(ostream::operator<< ) (00401005)
17:           getchar();
004010A3   mov         eax,[__iob+4 (0042ade4)]
004010A8   sub         eax,1
004010AB   mov         [__iob+4 (0042ade4)],eax
004010B0   cmp         dword ptr [__iob+4 (0042ade4)],0
004010B7   jl          main+9Ah (004010da)
004010B9   mov         ecx,dword ptr [__iob (0042ade0)]
004010BF   movsx       edx,byte ptr [ecx]
004010C2   and         edx,0FFh
004010C8   mov         dword ptr [ebp-1Ch],edx
004010CB   mov         eax,[__iob (0042ade0)]
004010D0   add         eax,1
004010D3   mov         [__iob (0042ade0)],eax
004010D8   jmp         main+0AAh (004010ea)
004010DA   push        offset __iob (0042ade0)
004010DF   call        _filbuf (004043e0)
004010E4   add         esp,4
004010E7   mov         dword ptr [ebp-1Ch],eax
18:           exit(0);
004010EA   push        0
004010EC   call        exit (00404250)
19:       }
20:
21:       if(f\_write==NULL)
004010F1   cmp         dword ptr [ebp-8],0
004010F5   jne         main+123h (00401163)
22:       {
23:           cout<<"WRITE--WRONG!"<<endl;
004010F7   push        offset @ILT+10(endl) (0040100f) //可看作特征字串
004010FC   push        offset string "WRITE--WRONG!" (00428028)
00401101   mov         ecx,offset cout (0042b9c0)) //可看作特征字串
00401106   call        ostream::operator<< (00401720) //输出经过 push 的字串
0040110B   mov         ecx,eax //以每一个 push 为一个输出单位,且指针后移
0040110D   call        @ILT+0(ostream::operator<< ) (00401005)
             //可以这么说,在 cout 流中有几个输出项会对应几个 push 项和 call ostream 项
24:           getchar();
00401112   mov         ecx,dword ptr [__iob+4 (0042ade4)]
00401118   sub         ecx,1
0040111B   mov         dword ptr [__iob+4 (0042ade4)],ecx
00401121   cmp         dword ptr [__iob+4 (0042ade4)],0
00401128   jl          main+10Ch (0040114c)
0040112A   mov         edx,dword ptr [__iob (0042ade0)]
00401130   movsx       eax,byte ptr [edx]
00401133   and         eax,0FFh
00401138   mov         dword ptr [ebp-20h],eax
0040113B   mov         ecx,dword ptr [__iob (0042ade0)]
00401141   add         ecx,1
00401144   mov         dword ptr [__iob (0042ade0)],ecx
0040114A   jmp         main+11Ch (0040115c)
0040114C   push        offset __iob (0042ade0)
00401151   call        _filbuf (004043e0)
00401156   add         esp,4
00401159   mov         dword ptr [ebp-20h],eax
25:           exit(0);
0040115C   push        0
0040115E   call        exit (00404250)
26:       }
27:
28:       fseek(f\_open,0,2);//用 OD 调试时会在这里出现错误，提示 exception:str!=NULL;
00401163   push        2
00401165   push        0
00401167   mov         edx,dword ptr [ebp-4]
0040116A   push        edx
0040116B   call        fseek (004040e0)
00401170   add         esp,0Ch
29:
30:       long length=ftell(f\_open);
00401173   mov         eax,dword ptr [ebp-4]
00401176   push        eax
00401177   call        ftell (00403e50)
0040117C   add         esp,4
0040117F   mov         dword ptr [ebp-0Ch],eax
31:       cout<<"length:"<<length<<endl;
00401182   push        offset @ILT+10(endl) (0040100f)
00401187   mov         ecx,dword ptr [ebp-0Ch]
0040118A   push        ecx
0040118B   push        offset string "length:" (0042801c)
00401190   mov         ecx,offset cout (0042b9c0)
00401195   call        ostream::operator<< (00401720)
0040119A   mov         ecx,eax
0040119C   call        ostream::operator<< (004013f0)
004011A1   mov         ecx,eax
004011A3   call        @ILT+0(ostream::operator<< ) (00401005)
32:       rewind(f\_open);
004011A8   mov         edx,dword ptr [ebp-4]
004011AB   push        edx
004011AC   call        rewind (00403d80)
004011B1   add         esp,4
33:
34:       char *buffer=new char[length];
004011B4   mov         eax,dword ptr [ebp-0Ch]
004011B7   push        eax
004011B8   call        operator new (00403d60)
004011BD   add         esp,4
004011C0   mov         dword ptr [ebp-14h],eax
004011C3   mov         ecx,dword ptr [ebp-14h]
004011C6   mov         dword ptr [ebp-10h],ecx
35:       fread(buffer,length,1,f\_open);
004011C9   mov         edx,dword ptr [ebp-4]
004011CC   push        edx
004011CD   push        1
004011CF   mov         eax,dword ptr [ebp-0Ch]
004011D2   push        eax
004011D3   mov         ecx,dword ptr [ebp-10h]
004011D6   push        ecx
004011D7   call        fread (00403b90)
004011DC   add         esp,10h
36:       fwrite(buffer,length,1,f\_write);
004011DF   mov         edx,dword ptr [ebp-8]
004011E2   push        edx
004011E3   push        1
004011E5   mov         eax,dword ptr [ebp-0Ch]
004011E8   push        eax
004011E9   mov         ecx,dword ptr [ebp-10h]
004011EC   push        ecx
004011ED   call        fwrite (00403970)
004011F2   add         esp,10h
37:
38:       delete buffer;
004011F5   mov         edx,dword ptr [ebp-10h]
004011F8   mov         dword ptr [ebp-18h],edx
004011FB   mov         eax,dword ptr [ebp-18h]
004011FE   push        eax
004011FF   call        operator delete (004038e0)
00401204   add         esp,4
39:       fclose(f\_open);
00401207   mov         ecx,dword ptr [ebp-4]
0040120A   push        ecx
0040120B   call        fclose (00403800)
00401210   add         esp,4
40:       fclose (f\_write);
00401213   mov         edx,dword ptr [ebp-8]
```



`00401216` `push` `edx`

`00401217` `call` `fclose` (`00403800`)

`0040121C` `add` `esp`, `4`

`41:`

`42:` `return` `0`;

`0040121F` `xor` `eax`, `eax`

`43:` `}`

`00401221` `pop` `edi`

`00401222` `pop` `esi`

`00401223` `pop` `ebx`

`00401224` `add` `esp`, `60h`

`00401227` `cmp` `ebp`, `esp`

`00401229` `call` `__chkesp` (`00404820`)

`0040122E` `mov` `esp`, `ebp`

`00401230` `pop` `ebp`

`00401231` `ret`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

/*来自 `00401062` `call` `fopen` (`00404800`)的调用

`fopen`:

`00404800` `push` `ebp`

`00404801` `mov` `ebp`, `esp`

`00404803` `push` `40h` //shflag

`00404805` `mov` `eax`, `dword ptr [mode]`

`00404808` `push` `eax` //指向打开文件的模式"rb"

`00404809` `mov` `ecx`, `dword ptr [file]`

`0040480C` `push` `ecx` //指向文件名"zhang.pdf"

`0040480D` `call` `_fsopen` (`00404710`)

`00404812` `add` `esp`, `0Ch`

`00404815` `pop` `ebp`

-------------------------------------------------------

//函数原型: `FILE *_fsopen(const char *filename, const char *mode, int shflag);` `shflag`: Type of sharing allowed

//Each of these functions returns a pointer to the stream. A NULL pointer value indicates an error.

`_fsopen` function opens the file specified by `filename` as a stream and prepares the file for subsequent shared reading or writing, as defined by the `mode` and `shflag` arguments. (来自 MSDN)

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

/*来自 `0040480D` `call` `_fsopen` (`00404710`)的调用

`_fsopen`:

`00404710` `push` `ebp`

`00404711` `mov` `ebp`, `esp`

`00404713` `sub` `esp`, `8`

`00404716` `push` `ebx`

`00404717` `push` `esi`

`00404718` `push` `edi`

------------------------------------------------------------

//接下来有 4 段类似于此段的代码,我的判断是检测要打开的文件名和打开的类型

`00404719` `cmp` `dword ptr [file]`, `0` //判断文件名是否为空

`0040471D` `jne` `_fsopen+2Dh` (`0040473d`) //文件名不为空跳转

`0040471F` `push` `offset string "file != NULL"` (`00428204`)

`00404724` `push` `0`

`00404726` `push` `35h`

`00404728` `push` `offset string "fopen.c"` (`004281fc`)

`0040472D` `push` `2`

`0040472F` `call` `_CrtDbgReport` (`00408800`)

`00404734` `add` `esp`, `14h`

`00404737` `cmp` `eax`, `1`

`0040473A` `jne` `_fsopen+2Dh` (`0040473d`)

`0040473C` `int` `3`

`0040473D` `xor` `eax`, `eax`

`0040473F` `test` `eax`, `eax`

`00404741` `jne` `_fsopen+9` (`00404719`)

/* `_CrtDbgReport(int reportType, const char *filename, int linenumber, const char *moduleName, const char *format [, argument] ...);`

Generates a report with a debugging message and sends the report to three possible destinations (debug version only). 具体参见 MSDN*/

----------------------------------------------------------

`00404743` `mov` `ecx`, `dword ptr [file]`

`00404746` `movsx` `edx`, `byte ptr [ecx]`

`00404749` `test` `edx`, `edx`

`0040474B` `jne` `_fsopen+5Bh` (`0040476b`)

`0040474D` `push` `offset string "*file != _T('\\0')"` (`004281e8`)

`00404752` `push` `0`

`00404754` `push` `36h`

`00404756` `push` `offset string "fopen.c"` (`004281fc`)

`0040475B` `push` `2`

`0040475D` `call` `_CrtDbgReport` (`00408800`)

`00404762` `add` `esp`, `14h`

`00404765` `cmp` `eax`, `1`

`00404768` `jne` `_fsopen+5Bh` (`0040476b`)

`0040476A` `int` `3`

`0040476B` `xor` `eax`, `eax`

`0040476D` `test` `eax`, `eax`

`0040476F` `jne` `_fsopen+33h` (`00404743`)

-----------------------------------------------------------------

`00404771` `cmp` `dword ptr [mode]`, `0`

`00404775` `jne` `_fsopen+85h` (`00404795`)

`00404777` `push` `offset string "mode != NULL"` (`004281d8`)

`0040477C` `push` `0`

`0040477E` `push` `37h`

`00404780` `push` `offset string "fopen.c"` (`004281fc`)

`00404785` `push` `2`

`00404787` `call` `_CrtDbgReport` (`00408800`)

`0040478C` `add` `esp`, `14h`

`0040478F` `cmp` `eax`, `1`

`00404792` `jne` `_fsopen+85h` (`00404795`)

`00404794` `int` `3`

`00404795` `xor` `ecx`, `ecx`

`00404797` `test` `ecx`, `ecx`

`00404799` `jne` `_fsopen+61h` (`00404771`)

----------------------------------------------------------

`0040479B` `mov` `edx`, `dword ptr [mode]`

`0040479E` `movsx` `eax`, `byte ptr [edx]`

`004047A1` `test` `eax`, `eax`

`004047A3` `jne` `_fsopen+0B3h` (`004047c3`)

`004047A5` `push` `offset string "*mode != _T('\\0')"` (`004281c4`)

`004047AA` `push` `0`

`004047AC` `push` `38h`

`004047AE` `push` `offset string "fopen.c"` (`004281fc`)

`004047B3` `push` `2`

`004047B5` `call` `_CrtDbgReport` (`00408800`)

`004047BA` `add` `esp`, `14h`

`004047BD` `cmp` `eax`, `1`

`004047C0` `jne` `_fsopen+0B3h` (`004047c3`)

`004047C2` `int` `3`

`004047C3` `xor` `ecx`, `ecx`

`004047C5` `test` `ecx`, `ecx`

`004047C7` `jne` `_fsopen+8Bh` (`0040479b`)

----------------------------------------------------------

`004047C9` `call` `_getstream` (`00409630`)

//Retrieves the current buffered stream. 具体参见 MSDN

//关键!!!此时 `EAX`=`0x0042AE40`,注意后面代码中出现这个值!!!

//就我个人观点 `OD` 对 `VC` 程序中函数的识别效果不是很好,就像在这里 `OD` 仅仅给出 `CALL fread_fw.0040479B`,如果贸然跟进必然会弄的一头雾水.

`004047CE` `mov` `dword ptr [stream]`, `eax` //从后面的代码可以推断这里的几句代码是位分配文件的句柄做准备

`004047D1` `cmp` `dword ptr [stream]`, `0`

`004047D5` `jne` `_fsopen+0CBh` (`004047db`)

`004047D7` `xor` `eax`, `eax`

`004047D9` `jmp` `_fsopen+0E9h` (`004047f9`)

`004047DB` `mov` `edx`, `dword ptr [stream]`

`004047DE` `push` `edx` //流指针入栈(`0x42AE40`)

`004047DF` `mov` `eax`, `dword ptr [shflag]`

`004047E2` `push` `eax` //不知是什么标志?? (`40`)

`004047E3` `mov` `ecx`, `dword ptr [mode]`

`004047E6` `push` `ecx` //指向打开文件的模式"rb"

`004047E7` `mov` `edx`, `dword ptr [file]`

`004047EA` `push` `edx` //指向文件名"zhang.pdf"

`004047EB` `call` `_openfile` (`004092a0`)

//此处 `OD` 给出的是 `CALL fread_fw.004092A0`; Be careful!!

`004047F0` `add` `esp`, `10h`

//当程序执行到这里,看到 `_openfile` 的返回值为 `0`; 判定问题是出在 `_openfile` 中

//重新载入调试,进入 `_openfile()` 函数体

`004047F3` `mov` `dword ptr [retval]`, `eax`

`004047F6` `mov` `eax`, `dword ptr [retval]`

`004047F9` `pop` `edi`

`004047FA` `pop` `esi`

`004047FB` `pop` `ebx`

`004047FC` `mov` `esp`, `ebp`

`004047FE` `pop` `ebp`

`004047FF` `ret`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

/*来自 `004047EB` `call` `_openfile` (`004092a0`)的调用

`_openfile`:

`004092A0` `push` `ebp`

`004092A1` `mov` `ebp`, `esp`

`004092A3` `sub` `esp`, `24h`

`004092A6` `push` `ebx`

`004092A7` `push` `esi`

`004092A8` `push` `edi`

`004092A9` `mov` `eax`, `[__commode (0042bbc0)]`

`004092AE` `mov` `dword ptr [streamflag]`, `eax`

`004092B1` `mov` `dword ptr [commodeset]`, `0`

`004092B8` `mov` `dword ptr [scanset]`, `0`

//这里又是三段类似的变量检测

`004092BF` `cmp` `dword ptr [filename]`, `0`

`004092C3` `jne` `_openfile+43h` (`004092e3`)

`004092C5` `push` `offset string "filename != NULL"` (`00428d0c`)

`004092CA` `push` `0`

`004092CC` `push` `47h`

`004092CE` `push` `offset string "_open.c"` (`00428d04`)

`004092D3` `push` `2`

`004092D5` `call` `_CrtDbgReport` (`00408800`)

`004092DA` `add` `esp`, `14h`

`004092DD` `cmp` `eax`, `1`

`004092E0` `jne` `_openfile+43h` (`004092e3`)

`004092E2` `int` `3`

`004092E3` `xor` `ecx`, `ecx`

`004092E5` `test` `ecx`, `ecx`

`004092E7` `jne` `_openfile+1Fh` (`004092bf`)

`004092E9` `cmp` `dword ptr [mode]`, `0`

`004092ED` `jne` `_openfile+6Dh` (`0040930d`)



```
004092EF   push        offset string "mode != NULL" (004281d8)  

004092F4   push        0  

004092F6   push        48h  

004092F8   push        offset string "\_open.c" (00428d04)  

004092FD   push        2  

004092FF   call        \_CrtDbgReport (00408800)  

00409304   add         esp,14h  

00409307   cmp         eax,1  

0040930A   jne         \_openfile+6Dh (0040930d)  

0040930C   int         3  

0040930D   xor         edx,edx  

0040930F   test        edx,edx  

00409311   jne         \_openfile+49h (004092e9)  

00409313   cmp         dword ptr [str],0  

00409317   jne         \_openfile+97h (00409337)  

00409319   push        offset string "str != NULL" (00428154)  

0040931E   push        0  

00409320   push        49h  

00409322   push        offset string "\_open.c" (00428d04)  

00409327   push        2  

00409329   call        \_CrtDbgReport (00408800)  

0040932E   add         esp,14h  

00409331   cmp         eax,1  

00409334   jne         \_openfile+97h (00409337)  

00409336   int         3  

00409337   xor         eax,eax  

00409339   test        eax,eax  

0040933B   jne         \_openfile+73h (00409313)  

0040933D   mov         ecx,dword ptr [mode]  

00409340   mov         dl,byte ptr [ecx]  

00409342   mov         byte ptr [ebp-20h],dl  

00409345   cmp         byte ptr [ebp-20h],61h  

00409349   je          \_openfile+0DCh (0040937c)  

0040934B   cmp         byte ptr [ebp-20h],72h  

0040934F   je          \_openfile+0B9h (00409359)  

00409351   cmp         byte ptr [ebp-20h],77h  

00409355   je          \_openfile+0CAh (0040936a)  

00409357   jmp         \_openfile+0EEh (0040938e)  

00409359   mov         dword ptr [modeflag],0  

00409360   mov         eax,dword ptr [streamflag]  

00409363   or          al,1  

00409365   mov         dword ptr [streamflag],eax  

00409368   jmp         \_openfile+0F5h (00409395)  

0040936A   mov         dword ptr [modeflag],301h  

00409371   mov         ecx,dword ptr [streamflag]  

00409374   or          ecx,2  

00409377   mov         dword ptr [streamflag],ecx  

0040937A   jmp         \_openfile+0F5h (00409395)  

0040937C   mov         dword ptr [modeflag],109h  

00409383   mov         edx,dword ptr [streamflag]  

00409386   or          edx,2  

00409389   mov         dword ptr [streamflag],edx  

0040938C   jmp         \_openfile+0F5h (00409395)  

0040938E   xor         eax,eax  

00409390   jmp         $L1486+0A0h (004095b6)  

00409395   mov         dword ptr [whileflag],1  
//这里开始进入迷茫区,是一个大的循环.(不要放弃!!核心部分即将出现!! )  

0040939C   mov         eax,dword ptr [mode]  

0040939F   add         eax,1  

004093A2   mov         dword ptr [mode],eax  

004093A5   mov         ecx,dword ptr [mode]  

004093A8   movsx       edx,byte ptr [ecx]  

004093AB   test        edx,edx  

004093AD   je          $L1486+29h (0040953f)  

004093B3   cmp         dword ptr [whileflag],0  

004093B7   je          $L1486+29h (0040953f)  

004093BD   mov         eax,dword ptr [mode]  

004093C0   movsx       ecx,byte ptr [eax]  

004093C3   mov         dword ptr [ebp-24h],ecx  

004093C6   mov         edx,dword ptr [ebp-24h]  

004093C9   sub         edx,2Bh  

004093CC   mov         dword ptr [ebp-24h],edx  

004093CF   cmp         dword ptr [ebp-24h],49h  

004093D3   ja          $L1486+1Dh (00409533)  

004093D9   mov         ecx,dword ptr [ebp-24h]  

004093DC   xor         eax,eax  

004093DE   mov         al,byte ptr  (004095e5)[ecx]  

004093E4   jmp         dword ptr [eax*4+4095BDh]  

$L1462:  

004093EB   mov         edx,dword ptr [modeflag]  

004093EE   and         edx,2  

004093F1   test        edx,edx  

004093F3   je          $L1462+13h (004093fe)  

004093F5   mov         dword ptr [whileflag],0  

004093FC   jmp         $L1462+35h (00409420)  

004093FE   mov         eax,dword ptr [modeflag]  

00409401   or          al,2  

00409403   mov         dword ptr [modeflag],eax  

00409406   mov         ecx,dword ptr [modeflag]  

00409409   and         ecx,0FEh  

0040940C   mov         dword ptr [modeflag],ecx  

0040940F   mov         edx,dword ptr [streamflag]  

00409412   or          dl,80h  

00409415   mov         dword ptr [streamflag],edx  

00409418   mov         eax,dword ptr [streamflag]  

0040941B   and         al,0FCh  

0040941D   mov         dword ptr [streamflag],eax  

00409420   jmp         $L1486+24h (0040953a)  

$L1465:  

00409425   mov         ecx,dword ptr [modeflag]  

00409428   and         ecx,0C000h  

0040942E   test        ecx,ecx  

00409430   je          $L1465+16h (0040943b)  

00409432   mov         dword ptr [whileflag],0  

00409439   jmp         $L1465+1Fh (00409444)  

0040943B   mov         edx,dword ptr [modeflag]  

0040943E   or          dh,80h  

00409441   mov         dword ptr [modeflag],edx  

00409444   jmp         $L1486+24h (0040953a)  

$L1468:  

00409449   mov         eax,dword ptr [modeflag]  

0040944C   and         eax,0C000h  

00409451   test        eax,eax  

00409453   je          $L1468+15h (0040945e)  

00409455   mov         dword ptr [whileflag],0  

0040945C   jmp         $L1468+1Eh (00409467)  

0040945E   mov         ecx,dword ptr [modeflag]  

00409461   or          ch,40h  

00409464   mov         dword ptr [modeflag],ecx  

00409467   jmp         $L1486+24h (0040953a)  

$L1471:  

0040946C   cmp         dword ptr [commodeset],0  

00409470   je          $L1471+0Fh (0040947b)  

00409472   mov         dword ptr [whileflag],0  

00409479   jmp         $L1471+1Fh (0040948b)  

0040947B   mov         dword ptr [commodeset],1  

00409482   mov         edx,dword ptr [streamflag]  

00409485   or          dh,40h  

00409488   mov         dword ptr [streamflag],edx  

0040948B   jmp         $L1486+24h (0040953a)  

$L1474:  

00409490   cmp         dword ptr [commodeset],0  

00409494   je          $L1474+0Fh (0040949f)  

00409496   mov         dword ptr [whileflag],0  

0040949D   jmp         $L1474+1Fh (004094af)  

0040949F   mov         dword ptr [commodeset],1  

004094A6   mov         eax,dword ptr [streamflag]  

004094A9   and         ah,0BFh  

004094AC   mov         dword ptr [streamflag],eax  

004094AF   jmp         $L1486+24h (0040953a)  

$L1477:  

004094B4   cmp         dword ptr [scanset],0  

004094B8   je          $L1477+0Fh (004094c3)  

004094BA   mov         dword ptr [whileflag],0  

004094C1   jmp         $L1477+1Fh (004094d3)  

004094C3   mov         dword ptr [scanset],1  

004094CA   mov         ecx,dword ptr [modeflag]  

004094CD   or          ecx,20h  

004094D0   mov         dword ptr [modeflag],ecx  

004094D3   jmp         $L1486+24h (0040953a)  

$L1480:  

004094D5   cmp         dword ptr [scanset],0  

004094D9   je          $L1480+0Fh (004094e4)  

004094DB   mov         dword ptr [whileflag],0  

004094E2   jmp         $L1480+1Fh (004094f4)  

004094E4   mov         dword ptr [scanset],1  

004094EB   mov         edx,dword ptr [modeflag]  

004094EE   or          edx,10h  

004094F1   mov         dword ptr [modeflag],edx  

004094F4   jmp         $L1486+24h (0040953a)  

$L1483:  

004094F6   mov         eax,dword ptr [modeflag]  

004094F9   and         eax,1000h  

004094FE   test        eax,eax  

00409500   je          $L1483+15h (0040950b)  

00409502   mov         dword ptr [whileflag],0  

00409509   jmp         $L1483+1Eh (00409514)  

0040950B   mov         ecx,dword ptr [modeflag]  

0040950E   or          ch,10h  

00409511   mov         dword ptr [modeflag],ecx  

00409514   jmp         $L1486+24h (0040953a)  

$L1486:  

00409516   mov         edx,dword ptr [modeflag]  

00409519   and         edx,40h  

0040951C   test        edx,edx  

0040951E   je          $L1486+13h (00409529)  

00409520   mov         dword ptr [whileflag],0  

00409527   jmp         $L1486+1Bh (00409531)  
```



### 汇编代码段

```
00409529 mov eax,dword ptr [modeflag]
0040952C or al,40h
0040952E mov dword ptr [modeflag],eax
00409531 jmp $L1486+24h (0040953a)
00409533 mov dword ptr [whileflag],0
0040953A jmp _openfile+0FCh (0040939c) //大循环到此为止
0040953F push 1A4h
00409544 mov ecx,dword ptr [shflag]
00409547 push ecx
00409548 mov edx,dword ptr [modeflag]
0040954B push edx
0040954C mov eax,dword ptr [filename]
0040954F push eax
00409550 call _sopen (0040f6e0)

//int _sopen( const char *filename, int oflag, int shflag [, int pmode ] );

//Each of these functions returns a file handle for the opened file. A return value of –1 indicates an error.具体资料参见 MSDN

//此处关键所在.由于我调试时是 VC 和 OD 两个调试器同时交叉前进的,到这里就产生不同.

//从理论上这个函数应该返回一个句柄.VC 调试器返回 EAX=3(不同机子可能会不同),而 OD 返回的是 EAX=0XFFFFFFFF.这里产生一个问题系统是如何分配句柄???

00409555 add esp,10h
00409558 mov dword ptr [filedes],eax
0040955B cmp dword ptr [filedes],0//在此处判断 _sopen 产生的文件句柄
0040955F jge $L1486+4Fh (00409565)//从这里开始 VC 和 OD 有了截然不同的命运
00409561 xor eax,eax

//OD 的命运:EAX 清零,决定了 f_open=fopen("zhang.pdf","rb")失败的返回标志为 EAX=0;

00409563 jmp $L1486+0A0h (004095b6) //跳转到返回代码处

//1.dword ptr [__cflush (0042ba50)]----->ECX--->+1--->dword ptr [__cflush (0042ba50)];

00409565 mov ecx,dword ptr [__cflush (0042ba50)] //VC 的命运(成功)
0040956B add ecx,1
0040956E mov dword ptr [__cflush (0042ba50)],ecx

//2.dword ptr [str]---->EDX---->dword ptr [stream]---->EAX;流文件中的指针重定位

//这里又产生问题:流 stream 具体是如何工作的,下面的 str 又代表什么样的结构

00409574 mov edx,dword ptr [str]
00409577 mov dword ptr [stream],edx
0040957A mov eax,dword ptr [stream]

//3.dword ptr [streamflag]--->ECX--->dword ptr [EAX+0Ch];设置标志位(DWORD)

0040957D mov ecx,dword ptr [streamflag]
00409580 mov dword ptr [eax+0Ch],ecx

//在 stream 中设置_sopen 返回的 handle 值(不知该称为句柄还是 ID,应该是 ID 吧)

00409583 mov edx,dword ptr [stream]
00409586 mov dword ptr [edx+4],0
0040958D mov eax,dword ptr [stream]
00409590 mov dword ptr [eax],0
00409596 mov ecx,dword ptr [stream]
00409599 mov dword ptr [ecx+8],0
004095A0 mov edx,dword ptr [stream]
004095A3 mov dword ptr [edx+1Ch],0
004095AA mov eax,dword ptr [stream]
004095AD mov ecx,dword ptr [filedes]
004095B0 mov dword ptr [eax+10h],ecx
004095B3 mov eax,dword ptr [stream] //用 EAX 保存返回值,即文件句柄

//上面这段代码处理的后的内存堆栈情况(红色标记表示 streamflag;蓝色标记表示 filedes):

0042AE40 00 00 00 00 00 00 ......
0042AE46 00 00 00 00 00 00 ......
0042AE4C 01 00 00 00 03 00 ......
0042AE52 00 00 00 00 00 00 ......
0042AE58 00 00 00 00 00 00 ......
0042AE5E 00 00 00 00 00 00 ......
0042AE64 00 00 00 00 00 00

streamflag:01 00 00 00
filedes:03 00 00 00

----------------------------------------------------

004095B6 pop edi
004095B7 pop esi
004095B8 pop ebx
004095B9 mov esp,ebp
004095BB pop ebp
004095BC ret
```

--------------------------------------------------

### 总结

#### 分析流程

`fopen()` --> `_fsopen()` --> `_openfile()` --> `_sopen()`

#### 遗留下来的问题

1. `str` 的结构和 `stream` 的工作原理。
2. `_sopen()` 函数体的分析和 `_fsopen()` 模块中的迷茫区。
3. 至于系统是如何分配句柄的，我想读者可以根据 `_getstream` 及其代码的上下部分可以了解个大概。需要注意一点的是句柄和指针一样本质都是地址。

最后回到本篇开始时提出的问题：为什么同一个程序在 `VC` 和 `OD` 的调试下会出现不同结果？

经过上面的分析，我想答案已经很明确。

`_sopen()` 返回的 `file handle` 不同，`OD` 的返回的结果表明要打开的文件资源（`"zhang.pdf"`）没有被找到而且程序已经给这个文件准备好要分配的句柄（`0x0042AE40`），而事实上这个文件确实存在。哎~~，这不是很明显的事，用 `0D` 调试程序相当于直接执行 `debug` 文件夹的 `fread_fwrite.exe`，要不试试会出现什么问题---呵呵，当然是 `"OPEN--WRONG!"`

其实 `debug` 文件夹中的可执行文件只检测该文件夹中是否存在资源文件，如果把 `"zhang.pdf"` 放到 `debug` 文件夹下，程序就能正常处理。

其实这应该很早就想到，但我的脑子是一条筋，转不过弯，每办法!!

`(11 个小时+一份外买+一包泡面+一杯水+饮料 N 瓶)-->一次漫长的旅途==我喜欢`

为了保证完整性，有些额外的反汇编代码没有去掉，我想这样方便大家学习。

如果大家对这个话题有兴趣，请各抒己见，共同探讨遗留下来的问题。

`fopen()` --> `_fsopen()` --> `_openfile()` --> `_sopen()`

### 补充

`-->kernel32.CreateFile() --> ntdll.NtCreateFile() --> int 2e (EAX = 0x25) --> ntoskrnl.ZwCreateFile --> win32k.sys`

`4dd466d0e1e0cc123afbe504da2462ca`
