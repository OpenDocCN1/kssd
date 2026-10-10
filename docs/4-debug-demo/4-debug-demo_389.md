

# flashget 1.6 的下载线程增加 

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-1.htm`](https://www.pediy.com/kssd/pediy07/pediy7-1.htm)

不是什么好文章，大家随便看看就好了  
用空到我的 blog 里看看  
[`peansen.blog.edu.cn/`](http://peansen.blog.edu.cn/)  
  
  
flashget 1.6 的下载线程增加  
作者：peansen  
日期：2005-1-10  
flashget 确实是一个练习 MFC 破解的好地方  
我想大家对 flashget 一定很熟悉。他的最大下载线程是 10，在这里我想记下我修改下载线程的过程。  
首先我们打开 flashget 工具默认-下载属性，改为文件分成 11 部分下载，点确定，我们看到对话框提示我们数据必须在 1-10 之间，好，我们就从这里入手，用 odbg 载入 flashget，下断点 bp MessageBoxA。重复上述操作。被断下，运行到用户代码并关掉 flashget 跳出的提示框，我们可以看到  
004BF433  |> \\8D85 ECFEFFFF   lea eax,dword ptr ss:\[ebp-114\]  
004BF439  |.  68 04010000     push 104                                  ; /BufSize = 104 (260.)  
004BF43E  |.  50              push eax                                  ; |PathBuffer  
004BF43F  |.  6A 00           push 0                                    ; |hModule = NULL  
004BF441  |.  8DBD ECFEFFFF   lea edi,dword ptr ss:\[ebp-114\]            ; |  
004BF447  |.  FF15 A8F34D00   call near dword ptr ds:\[<&KERNEL32.GetMod>; \\GetModuleFileNameA  
004BF44D  |>  53              push ebx                                  ; /Style  
004BF44E  |.  57              push edi                                  ; |Title  
004BF44F  |.  FF75 08         push dword ptr ss:\[ebp+8\]                 ; |Text  
004BF452  |.  FF75 F4         push dword ptr ss:\[ebp-C\]                 ; |hOwner  
004BF455  |.  FF15 10F74D00   call near dword ptr ds:\[<&USER32.MessageB>; \\MessageBoxA  
004BF45B  |.  85F6            test esi,esi                              ;  flashget.005107DC  
004BF45D  |.  8BF8            mov edi,eax  
现在我们要做的是找出这个函数的调用函数，一层一层往上找。这里要紧跟一个线索：堆栈里的字符串，看看是谁压入这个字符串的。我们跟到这边  
0040CB8F   .  8BC8            mov ecx,eax  
0040CB91   .  E8 AA740000     call flashget.00414040//eax 从这个函数里来----进入  
0040CB96   .  8B4C24 14       mov ecx,dword ptr ss:\[esp+14\]  
0040CB9A   .  50              push eax                                  ; /Arg4 这个 eax=0a（就是限制值）  
0040CB9B   .  6A 01           push 1                                    ; |Arg3 = 00000001  
0040CB9D   .  51              push ecx                                  ; |Arg2  
0040CB9E   .  57              push edi                                  ; |Arg1  
0040CB9F   .  E8 22160B00     call flashget.004BE1C6                    ; \\flashget.004BE1C6  
0040CBA4   .  8B5424 14       mov edx,dword ptr ss:\[esp+14\]  
跟进，下面是其中一部分  
004140A6  |.  C2 0400         retn 4  
004140A9  |>  8B81 38010000   mov eax,dword ptr ds:\[ecx+138\]            ;  Case 2D of switch 00414044  
004140AF  |.  C2 0400         retn 4  
004140B2  |>  8B81 C8010000   mov eax,dword ptr ds:\[ecx+1C8\]   我们的是这个，看他的地址         ;  Case 2 of switch 00414044  
004140B8  |.  C2 0400         retn 4  
004140BB  |>  8B81 CC010000   mov eax,dword ptr ds:\[ecx+1CC\]            ;  Case 3 of switch 00414044  
004140C1  |.  C2 0400         retn 4  
我们看到上面的内存地址：510908  
好了，清楚我们刚才跟踪的所有断点（函数一层层跟踪时设的好多断点）重新载入程序，在内存 510908 处设写入内存断点，F9 我们来到  
004171EF  |.  E8 E0200B00     call flashget.004C92D4  
004171F4  |.  6A 08           push 8  
004171F6  |.  68 908A5000     push flashget.00508A90                    ;  ASCII "ConnType"  
004171FB  |.  68 D48A5000     push flashget.00508AD4                    ;  ASCII "Connection"  
00417200  |.  8BCE            mov ecx,esi  
00417202  |.  8986 E0000000   mov dword ptr ds:\[esi+E0\],eax  
00417208  |.  E8 C7200B00     call flashget.004C92D4  
0041720D  |.  6A 0A           push 0A//最大是 10  
0041720F  |.  68 C08C5000     push flashget.00508CC0                    ;  ASCII "Max Parallel Num"  
00417214  |.  68 AC715000     push flashget.005071AC                    ;  ASCII "General"  
00417219  |.  8BCE            mov ecx,esi  
0041721B  |.  8986 E4000000   mov dword ptr ds:\[esi+E4\],eax  
00417221  |.  E8 AE200B00     call flashget.004C92D4  
00417226  |.  83F8 1E         cmp eax,1E//超过择直接用 30，就是你只改上面是不够的  
00417229  |.  8986 C8010000   mov dword ptr ds:\[esi+1C8\],eax  
0041722F  |.  7E 0A           jle short flashget.0041723B  
00417231  |.  C786 C8010000 1>mov dword ptr ds:\[esi+1C8\],1E  
0041723B  |>  6A 08           push 8  
0041723D  |.  68 B48C5000     push flashget.00508CB4                    ;  ASCII "MaxSimJobs"  
00417242  |.  68 AC715000     push flashget.005071AC                    ;  ASCII "General"  
00417247  |.  8BCE            mov ecx,esi  
上面的代码很容易分析，猜也猜的到。我开始作了这个修改  
0041720D  |.  6A 0A           push 0A-----------〉改为 f0  
0041720F  |.  68 C08C5000     push flashget.00508CC0                    ;  ASCII "Max Parallel Num"  
00417214  |.  68 AC715000     push flashget.005071AC                    ;  ASCII "General"  
00417219  |.  8BCE            mov ecx,esi  
0041721B  |.  8986 E4000000   mov dword ptr ds:\[esi+E4\],eax  
00417221  |.  E8 AE200B00     call flashget.004C92D4  
00417226  |.  83F8 1E         cmp eax,1E-------------〉改为 f0  
运行程序显示文件分成-16，我靠。变成负数了，我管那么多，我也不想再去取掉这个 unsigned 的问题了，直接把 f0 改成 7f（呵呵，在可能的范围内取最大利益。）上面的两处改了就好了。在 1-127 之间，我想够了。我拿了个做实验，用 120，呵呵，结果程序死了。哎，线程不要太多，不然会快的受不了。只要高兴就好。 

 4dd466d0e1e0cc123afbe504da2462ca