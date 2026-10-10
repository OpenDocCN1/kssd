

# Armadillo V4.X CopyMem-II 脱壳——魔法转换

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-686.htm`](https://www.pediy.com/kssd/pediy07/pediy7-686.htm)

Armadillo V4.X CopyMem-II 脱壳——魔法转换(Magic Converter) V4.0 正式版  
              
             
               
下载页面：  http://www.skycn.com/soft/3265.html  
软件大小：  2388 KB  
软件语言：  英文  
软件类别：  国产软件 / 共享版 / 图像转换  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2005-04-25 11:48:30  
下载次数：  209670  
推荐等级：  \*\*\*\*  
开 发 商：  http://www.keksoft.com/  
软件介绍：  魔法转换 是一个功能强大的图像批量处理转换软件。它包括图像浏览、批量转换(可自定义脚本)、字符画转换、图像优化、捕捉图像和创建 EXE 可执行文件六大部分。使用它可以让你制定转换脚本，然后根据脚本对图像进行批量处理转换。它还支持 WINDOWS 右键菜单进行转换操作，你还可以自定义 WINDOWS 右键菜单。同时它还能将图像打包生成.EXE 可执行文件，然后可以脱离图像浏览器进行自动播放。魔法转换能支持目前所有流行的图像格式，它的输出格式为：bmp、jpg、gif、png、tif、pcx、wmf、emf、tga、ico、wbmp，还包括 txt、rtf、htm 等字符画格式。   
               
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：WinXP、flyODBG、PEiD、LordPE、ImportREC  
               
—————————————————————————————————   
【脱壳过程】：  
            
       
这里使用修改版 OllyDbg，如果使用原版，需要处理 OutputDebugStringA 问题   
设置 OllyDbg 忽略所有其它异常选项。用 IsDebug 插件去掉 OllyDbg 的调试器标志。  
—————————————————————————————————  
一、寻找 OEP+解码 Dump  
     
              
0061F743    55              push ebp  
//进入 OllyDbg 后暂停在这  
0061F744    8BEC            mov ebp,esp  
0061F746    6A FF           push -1  
0061F748    68 209B6400     push 00649B20  
0061F74D    68 80F46100     push 0061F480  
0061F752    64:A1 00000000  mov eax,dword ptr fs:\[0\]  
0061F758    50              push eax  
0061F759    64:8925 0000000>mov dword ptr fs:\[0\],esp  
0061F760    83EC 58         sub esp,58  
0061F763    53              push ebx  
0061F764    56              push esi  
0061F765    57              push edi  
0061F766    8965 E8         mov dword ptr ss:\[ebp-18\],esp  
0061F769    FF15 88416400   call dword ptr ds:\[<&KERNEL32.GetVersion>\]  
  
BP WaitForDebugEvent  
中断后取消断点，看堆栈：  
0012BCB8   0060F8BF  /CALL 到 WaitForDebugEvent 来自 MAGCT.0060F8B9  
0012BCBC   0012CD90  |pDebugEvent = 0012CD90  
0012BCC0   000003E8  \\Timeout = 1000. ms  
在数据窗口定位到 0012CD90 处，准备看 OEP 值  
接着下断：BP WriteProcessMemory  
  
现在去代码窗口 Ctrl+G：0060F8BF  
Ctrl+F 在当前位置下搜索命令：or eax,0FFFFFFF8  
找到第一处在 0060FE8F 处，在其上 cmp dword ptr ss:\[ebp-A34\],0 处设置断点。  
  
0060FE43    83BD CCF5FFFF 0>cmp dword ptr ss:\[ebp-A34\],0  
//下断，Shift+F9 中断下来  把\[ebp-A34\]＝\[0012CD7C\]=000001B7 清 0  ★  
0060FE4A    0F8C A8020000   jl 006100F8  
0060FE50    8B8D CCF5FFFF   mov ecx,dword ptr ss:\[ebp-A34\]  
0060FE56    3B0D 48AF6400   cmp ecx,dword ptr ds:\[64AF48\]  
//注意\[64AF48\]  ★  
0060FE5C    0F8D 96020000   jge 006100F8  
//解码结束后跳转 006100F8     在 006100F8 处下断  ★  
0060FE62    8B95 40F6FFFF   mov edx,dword ptr ss:\[ebp-9C0\]  
0060FE68    81E2 FF000000   and edx,0FF  
0060FE6E    85D2            test edx,edx  
0060FE70    0F84 AD000000   je 0060FF23  
0060FE76    6A 00           push 0  
0060FE78    8BB5 CCF5FFFF   mov esi,dword ptr ss:\[ebp-A34\]  
0060FE7E    C1E6 04         shl esi,4  
0060FE81    8B85 CCF5FFFF   mov eax,dword ptr ss:\[ebp-A34\]  
0060FE87    25 07000080     and eax,80000007  
0060FE8C    79 05           jns short 0060FE93  
0060FE8E    48              dec eax  
0060FE8F    83C8 F8         or eax,FFFFFFF8  
//找到这里  
0060FE92    40              inc eax  
0060FE93    33C9            xor ecx,ecx  
0060FE95    8A88 1C896400   mov cl,byte ptr ds:\[eax+64891C\]  
0060FE9B    8B95 CCF5FFFF   mov edx,dword ptr ss:\[ebp-A34\]  
0060FEA1    81E2 07000080   and edx,80000007  
0060FEA7    79 05           jns short 0060FEAE  
0060FEA9    4A              dec edx  
0060FEAA    83CA F8         or edx,FFFFFFF8  
0060FEAD    42              inc edx  
0060FEAE    33C0            xor eax,eax  
0060FEB0    8A82 1D896400   mov al,byte ptr ds:\[edx+64891D\]  
0060FEB6    8B3C8D 70436400 mov edi,dword ptr ds:\[ecx\*4+644370\]  
0060FEBD    333C85 70436400 xor edi,dword ptr ds:\[eax\*4+644370\]  
0060FEC4    8B8D CCF5FFFF   mov ecx,dword ptr ss:\[ebp-A34\]  
0060FECA    81E1 07000080   and ecx,80000007  
0060FED0    79 05           jns short 0060FED7  
0060FED2    49              dec ecx  
0060FED3    83C9 F8         or ecx,FFFFFFF8  
0060FED6    41              inc ecx  
0060FED7    33D2            xor edx,edx  
0060FED9    8A91 1E896400   mov dl,byte ptr ds:\[ecx+64891E\]  
0060FEDF    333C95 70436400 xor edi,dword ptr ds:\[edx\*4+644370\]  
0060FEE6    8B85 CCF5FFFF   mov eax,dword ptr ss:\[ebp-A34\]  
0060FEEC    99              cdq  
0060FEED    B9 1C000000     mov ecx,1C  
0060FEF2    F7F9            idiv ecx  
0060FEF4    8BCA            mov ecx,edx  
0060FEF6    D3EF            shr edi,cl  
0060FEF8    83E7 0F         and edi,0F  
0060FEFB    03F7            add esi,edi  
0060FEFD    8B15 2CAF6400   mov edx,dword ptr ds:\[64AF2C\]  
0060FF03    8D04B2          lea eax,dword ptr ds:\[edx+esi\*4\]  
0060FF06    50              push eax  
0060FF07    8B8D CCF5FFFF   mov ecx,dword ptr ss:\[ebp-A34\]  
0060FF0D    51              push ecx  
0060FF0E    E8 2F210000     call 00612042  
0060FF13    83C4 0C         add esp,0C  
0060FF16    25 FF000000     and eax,0FF  
//这里 Patch ★  
0060FF1B    85C0            test eax,eax  
0060FF1D    0F84 D5010000   je 006100F8  
  
Patch 代码：  
0060FF16    FF85 CCF5FFFF   inc dword ptr ss:\[ebp-0A34\]  
0060FF1C    C705 4CAF6400 0>mov dword ptr ds:\[64AF48+4\],1  
0060FF26    E9 18FFFFFF     jmp 0060FE43  
  
Shift+F9，中断在上面 BP WriteProcessMemory 断点处。看数据窗口：  
0012CD90  01 00 00 00 A8 0B 00 00 40 0E 00 00 01 00 00 80  ...?..@....€  
0012CDA0  00 00 00 00 00 00 00 00 BC 83 5B 00 02 00 00 00  ........純\[....  
0012CDB0  00 00 00 00 BC 83 5B 00 BC 83 5B 00 0C AD 4D B2  ....純\[.純\[..璏  
很明显可以看到 OEP=005B83BC  
  
取消 WriteProcessMemory 处断点，继续 Shift+F9，中断在 006100F8 处  
此时子进程代码已经解开，运行 LordPE，完全 Dump 出子进程  
  
  
—————————————————————————————————  
二、搞定输入表  
  
  
可以完美还原 IAT，不过修改的代码麻烦，就不写了。  
再次载入主程序 MAGCT.EXE  
下断：BP DebugActiveProcess  中断后看堆栈：  
0013BCBC   0060F71A  /CALL 到 DebugActiveProcess 来自 MAGCT.0060F714  
0013BCC0   00000DEC  \\ProcessId = DEC  
  
新开一个 OllyDbg，附加进程 ID DEC 的子进程  
F9，再 F12，会暂停在 EP 处  
  
0061F743    EB FE           jmp short 0061F743  
//子进程在 EP 处死循环  恢复原来 EP 处的代码：55 8B  
  
切换单进程。BP OpenMutexA  Shift+F9，中断后看堆栈：  
0013D7AC   0060B321  /CALL 到 OpenMutexA 来自 MAGCT.0060B31B  
0013D7B0   001F0001  |Access = 1F0001  
0013D7B4   00000000  |Inheritable = FALSE  
0013D7B8   0013DDEC  \\MutexName = "DEC::DADD467F84"  ★ 注意 0013DDEC  
  
Ctrl+G：401000  键入以下分离代码  
00401000    60              pushad  
00401001    9C              pushfd  
00401002    68 ECDD1300     push 13DDEC  ★ 堆栈里看到的值  
00401007    33C0            xor eax,eax  
00401009    50              push eax  
0040100A    50              push eax  
0040100B    E8 2FDB407C     call 7C80EB3F ; kernel32.CreateMutexA  
00401010    9D              popfd  
00401011    61              popad  
00401012    E9 04DC407C     jmp 7C80EC1B  ; kernel32.OpenMutexA  
  
60 9C 68 EC DD 13 00 33 C0 50 50 E8 2F DB 40 7C 9D 61 E9 04 DC 40 7C  
  
在 401000 处新建 EIP，F9 运行，再次中断在 OpenMutexA 处。  
返回 401000 处，“撤销选择”，清掉写入的代码。再取消 OpenMutexA 断点。  
  
下断：HE GetModuleHandleA    Shift+F9，注意看堆栈：  
00136884   00E8372E  /CALL 到 GetModuleHandleA 来自 00E83728  
00136888   00E97474  \\pModule = "kernel32.dll"  
0013688C   00E98744  ASCII "VirtualAlloc"  
  
00136884   00E8374B  /CALL 到 GetModuleHandleA 来自 00E83745  
00136888   00E97474  \\pModule = "kernel32.dll"  
0013688C   00E98738  ASCII "VirtualFree"  
  
001365F4   00E6ACC1  /CALL 到 GetModuleHandleA 来自 00E6ACBB  
001365F8   00136738  \\pModule = "kernel32.dll"  
  
堆栈如上变化时,取消 GetModuleHandleA 处断点，Alt+F9 返回  
  
00E6ACBB    FF15 D820E900   call dword ptr ds:\[E920D8\] ; kernel32.GetModuleHandleA  
00E6ACC1    8B0D E4C9E900   mov ecx,dword ptr ds:\[E9C9E4\]  
//返回这里  
00E6ACC7    89040E          mov dword ptr ds:\[esi+ecx\],eax  
00E6ACCA    A1 E4C9E900     mov eax,dword ptr ds:\[E9C9E4\]  
00E6ACCF    391C06          cmp dword ptr ds:\[esi+eax\],ebx  
00E6ACD2    75 16           jnz short 00E6ACEA  
00E6ACD4    8D85 B4FEFFFF   lea eax,dword ptr ss:\[ebp-14C\]  
00E6ACDA    50              push eax  
00E6ACDB    FF15 E020E900   call dword ptr ds:\[E920E0\] ; kernel32.LoadLibraryA  
00E6ACE1    8B0D E4C9E900   mov ecx,dword ptr ds:\[E9C9E4\]  
00E6ACE7    89040E          mov dword ptr ds:\[esi+ecx\],eax  
00E6ACEA    A1 E4C9E900     mov eax,dword ptr ds:\[E9C9E4\]  
00E6ACEF    391C06          cmp dword ptr ds:\[esi+eax\],ebx  
00E6ACF2    0F84 32010000   je 00E6AE2A  
//Magic Jump！ 修改为：jmp 00E6AE2A ★  
  
这个东东在处理输入表的时候还有时间校验，不处理的话会导致某些函数被加密  
下断：BP GetTickCount   中断后取消断点返回  
  
00E88D57    FF15 AC22E900   call dword ptr ds:\[E922AC\] ; kernel32.GetTickCount  
00E88D5D    8985 8CC3FFFF   mov dword ptr ss:\[ebp-3C74\],eax  
//返回这里  
00E88D63    6A 01           push 1  
00E88D65    58              pop eax  
00E88D66    85C0            test eax,eax  
00E88D68    0F84 A8030000   je 00E89116  
  
下面还有一处 GetTickCount 取时间  
  
00E89100    8908            mov dword ptr ds:\[eax\],ecx  
//函数写入。在这里可以看见输入表函数开始地址 005D7208，输入表处理结束后可以计算出大小=8B4  
00E89102    8B85 FCC7FFFF   mov eax,dword ptr ss:\[ebp-3804\]  
00E89108    83C0 04         add eax,4  
00E8910B    8985 FCC7FFFF   mov dword ptr ss:\[ebp-3804\],eax  
00E89111    E9 4DFCFFFF     jmp 00E88D63  
00E89116    FF15 AC22E900   call dword ptr ds:\[E922AC\] ; kernel32.GetTickCount  
00E8911C    2B85 8CC3FFFF   sub eax,dword ptr ss:\[ebp-3C74\]  
00E89122    8B8D 90C3FFFF   mov ecx,dword ptr ss:\[ebp-3C70\]  
00E89128    6BC9 32         imul ecx,ecx,32  
00E8912B    81C1 D0070000   add ecx,7D0  
00E89131    3BC1            cmp eax,ecx  
//时间校验  
00E89133    76 07           jbe short 00E8913C  
//修改为：JMP 00E8913C  ★  
00E89135    C685 20C8FFFF 0>mov byte ptr ss:\[ebp-37E0\],1  
00E8913C    83BD D0C6FFFF 0>cmp dword ptr ss:\[ebp-3930\],0  
00E89143    0F85 8A000000   jnz 00E891D3  
  
00E891B8    83C4 0C         add esp,0C  
00E891BB    8B85 58C8FFFF   mov eax,dword ptr ss:\[ebp-37A8\]  
00E891C1    8985 A49EFFFF   mov dword ptr ss:\[ebp+FFFF9EA4\],eax  
00E891C7    FFB5 A49EFFFF   push dword ptr ss:\[ebp+FFFF9EA4\]  
00E891CD    E8 64820000     call 00E91436 ; jmp to msvcrt.operator delete  
00E891D2    59              pop ecx  
00E891D3    E9 05F7FFFF     jmp 00E888DD  
00E891D8    8B85 DCC6FFFF   mov eax,dword ptr ss:\[ebp-3924\]  
//这里下断，中断后输入表处理完毕  
  
运行 ImportREC，选择这个进程。填入 RVA=001D7208、Size=000008B4，Get Import 剪切掉无效函数，修改 OEP RVA=001B83BC，FixDump，正常运行！   
             
  
—————————————————————————————————                                      
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacKed By :  fly  
               2005-04-27 20:00 

 4dd466d0e1e0cc123afbe504da2462ca