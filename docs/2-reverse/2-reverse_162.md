

# 解决 od 格式化 long double 错误

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-817.htm`](https://www.pediy.com/kssd/pediy08/pediy8-817.htm)

//摘自 od 的反汇编引擎中的 asmserv.c 文件  

>  
> 
> ```
> 代码:
> 
> * * *
> 
> // Decodes and prints 80-bit long double ext into string s (at least 32 bytes
> // long). Procedure correctly displays all, even invalid, numbers without
> // arithmetical exceptions. Returns resulting length of the string.
> int Printfloat10(char \*s,long double ext) {
>   int k;
>   char \*e=(char \*)&ext;
>   if (\*(ulong \*)e==0 && \*(ushort \*)(e+4)==0 && \*(ulong \*)(e+6)==0x7FFF8000L)
>   {
>     k=sprintf(s,"+INF 7FFF 80000000 00000000");
>   }
>   else if (\*(ulong \*)e==0 && \*(ushort \*)(e+4)==0 && \*(ulong \*)(e+6)==0xFFFF8000L)
>   {
>     k=sprintf(s,"-INF FFFF 80000000 00000000");
>   }
>   else if ((\*(ulong \*)(e+6) & 0x7FFF8000L)==0x7FFF8000L)
>   {
>     k=sprintf(s,"%cNAN %04X %08lX %08lX",(e\[9\] & 0x80)==0?'+':'-', (int)(\*(ushort \*)(e+8)),\*(ulong \*)(e+4),\*(ulong \*)e);
>   }
>   else if ((\*(ulong \*)(e+6) & 0x7FFF0000L)==0x7FFF0000L)
>   {
>     k=sprintf(s,"%c??? %04X %08lX %08lX",(e\[9\] & 0x80)==0?'+':'-',(int)(\*(ushort \*)(e+8)),\*(ulong \*)(e+4),\*(ulong \*)e);
>   }
>   else if ((\*(ulong \*)(e+6) & 0x7FFF0000L)!=0 && (\*(ulong \*)(e+6) & 0x00008000)==0)
>   {
>     k=sprintf(s,"%cUNORM %04X %08lX %08lX",(e\[9\] & 0x80)==0?'+':'-',(int)(\*(ushort \*)(e+8)),\*(ulong \*)(e+4),\*(ulong \*)e);
>   }
>   else if (\*(ulong \*)e==0 && \*(ushort \*)(e+4)==0 && \*(ulong \*)(e+6)==0x80000000L)
>   {
>     k=sprintf(s,"-0.0");               // Negative floating 0.0
>   }
>   else if (ext==0.0)
>   {
>     k=sprintf(s,"0.0");                // Print 0 with decimal point
>   }
>   else if ((ext>=-1.e10 && ext<-1.0) || (ext>1.0 && ext<=1.e10))
>   {
>        k=sprintf(s,"%#.20Lg",ext);//出错就在这里  
>   }
>   else if ((ext>=-1.0 && ext<=-1.e-5) || (ext>=1.e-5 && ext<=1.0))
>   {
>     k=sprintf(s,"%#.19Lf",ext); 
>   }
>   else
>   {
>        k=sprintf(s,"%#.19Le",ext); //出错就在这里 
>    }
>   
>   return k;
> };
> 
> * * *
> 
> 
> ```
> 
>     

附件下载：Ollycrash.zip  
用原版 OD 打开 ollycrash.EXE 就会崩溃，引起这原因的就是如下语句：

  
0040101E FF FF FF FF FF FF FF FF 3D 40  
  
0040102C fld tbyte ptr \[40101E\] 

对于 403D FFFFFFFF FFFFFFFF 这个值引起的问题，其实不应该算是 ollydbg 的 bug,应该是 borland 编译器 sprintf 的 bug  
  
软件在理论上是没错的，实际上被硬件忽悠了  
在内存里 403D FFFFFFFF FFFFFFFF 这个浮点值的整数部分对应于有符号的 QWORD 最大值：7FFFFFFF FFFFFFFF  
fistp qword ptr \[xxx\]在把浮点转成整数时会自动 4 舍 5 入就相当于 40 3E 80000000 00000000 了  
其整数部分已经超出了有符号 QWORD 的表示范围，在执行 fistp qword ptr \[xxx\]时就会产生异常   
  
从理论上计算 403E 80000000 00000000 应该等于 2 的 63 次方，即 9223372036854775808  
403D FFFFFFFF FFFFFFFF 应该等于 2 的 63 次方减 0.5,即 9223372036854775807.5  
关于自动进位我想应该是在浮点转换成整数时才产生的，在没有转换成整数时应该还是取 9223372036854775807.5  

知道了原因就有很多解决办法，但是要选最好的方法。有的人用改变浮点寄存器的控制寄存器，这个办法不好，影响整个系统的浮点计算精度和浮点异常处理。  
下面的方法应该没有什么明显缺陷。  
004AA2E0   8B4424 04        MOV EAX,DWORD PTR SS:\[ESP+4\]  
004AA2E4   8B5424 08        MOV EDX,DWORD PTR SS:\[ESP+8\]  
004AA2E8   E9 63530000      JMP OLLYDBG.004AF650    //修改此行  
004AA2ED   90               NOP  
004AA2EE   74 06            JE SHORT OLLYDBG.004AA2F6  
004AA2F0   DB2A             FLD TBYTE PTR DS:\[EDX\]  
004AA2F2   DF38             FISTP QWORD PTR DS:\[EAX\]  
004AA2F4   9B               WAIT  
004AA2F5   C3               RETN  
004AA2F6   8B0A             MOV ECX,DWORD PTR DS:\[EDX\]  
004AA2F8   8908             MOV DWORD PTR DS:\[EAX\],ECX  
004AA2FA   8B4A 04          MOV ECX,DWORD PTR DS:\[EDX+4\]  
004AA2FD   8948 04          MOV DWORD PTR DS:\[EAX+4\],ECX  
004AA300   C3               RETN  
修补代码：  
004AF650   66:817A 08 3E40  CMP WORD PTR DS:\[EDX+8\],403E  
004AF656  ⁰F84 9AACFFFF    JE OLLYDBG.004AA2F6  
004AF65C   833A FF          CMP DWORD PTR DS:\[EDX\],-1  
004AF65F  ⁰F85 8BACFFFF    JNZ OLLYDBG.004AA2F0  
004AF665   837A 04 FF       CMP DWORD PTR DS:\[EDX+4\],-1  
004AF669  ⁰F85 81ACFFFF    JNZ OLLYDBG.004AA2F0  
004AF66F   66:817A 08 3D40  CMP WORD PTR DS:\[EDX+8\],403D  
004AF675  ⁰F85 75ACFFFF    JNZ OLLYDBG.004AA2F0  
004AF67B   FF02             INC DWORD PTR DS:\[EDX\]  
004AF67D   FF42 04          INC DWORD PTR DS:\[EDX+4\]  
004AF680   C642 07 80       MOV BYTE PTR DS:\[EDX+7\],80  
004AF684   66:FF42 08       INC WORD PTR DS:\[EDX+8\]  
004AF688  ^E9 69ACFFFF      JMP OLLYDBG.004AA2F6 

 4dd466d0e1e0cc123afbe504da2462ca