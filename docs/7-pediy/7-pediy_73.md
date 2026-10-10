

# tetris 4000&tetris 5000crack

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-257.htm`](https://www.pediy.com/kssd/pediy08/pediy8-257.htm)

【文章标题】: tetris 4000&tetris 5000crack  
【文章作者】: dogkarl  
【作者邮箱】: xiangpizhu126@yahoo.com.cn  
【软件名称】: tetris 4000&tetris 5000  
【软件大小】: 10.5M&4.45M  
【下载地址】: [`www.softsea.net/soft/114964.htm`](http://www.softsea.net/soft/114964.htm)  
【加壳方式】: SoftWrap  
【编写语言】: C  
【使用工具】: Peid0.94,OllyIce,dup2.0  
【操作平台】: winXP  
【软件介绍】: 很好玩的俄罗斯方块，画面绚丽，而且是 3D 的。  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
    
  暑假来临，奉母亲大人之命，寻找俄罗斯方块，结果找到两个很 pretty 的俄罗斯方块。然而寻注册而不可得，只好自己解决。  
    
  首先介绍 tetris 4000,这个是用 softwarp5.1.2 加壳的，参考了 SoftWrap 6.1.1 Loader By: Gabri3l / ARTeam   
  [`cracking.accessroot.com`](http://cracking.accessroot.com) 一文的思路，因此本文也可以作为该文的中文翻译补充版。  
    
  1.例行的第一步，用 peid 0.94 扫描文件，首先会显示 FSG v1.10 (Eng) -> dulek/xt \*，但是在 hardcore scan 之下，显出了庐山真面目  
  Softwrap (encrypted) main stub / XLok，但是 peid 没有给出具体的版本号。在第一个窗口中按下 buy,可以看到版本号是 5.1.2。  
    
  2.开始破解每个软件之前，我们都要理好思路，定好目标。  
    对于 tetris4000,由于它被一个加密壳加密，最好的方法是完全脱掉它。但是我们今天用的思路却是用  内存补丁对加密壳实施爆破。要暴破  
  ，关键就是要找到壳中关键的判断分支。  
    
  3.有了思路现在开始工作。打开 OllyIce,加载 Tetris4000.exe,然后在选项-〉调试设置中忽略以下异常：  
     INT 3 异常  
     单步异常  
     整数除零  
     无效或异常指令  
    
  4.按下运行程序按钮，我们来到第一个对话框处。（try 按钮是不是灰的无关紧要）softwrap 会提醒你只有三次机会等等。  
    
  5.按下暂停程序按钮，然后在“察看”菜单中选择“调用堆栈”命令。你会看到大致如下的信息：  
    
  调用堆栈：    主线程  
  地址       堆栈       函数过程                              调用来自                        
  0012F4FC   77D19418   包含 ntdll.KiFastSystemCallRet           USER32.77D19416               
  0012F500   77D2E2A2   USER32.WaitMessage                    USER32.77D2E29D                 
  0012F534   77D261C6   USER32.77D2E113                       USER32.77D261C1                 
  0012F55C   77D26208   USER32.77D26110                       USER32.77D26203                 
  0012F57C   77D3B168   USER32.DialogBoxIndirectParamAorW     USER32.77D3B163       此行就是用来创建你看到的对话框的          
  0012F5A8   00F31276   softwrap.00FAA5EC                     softwrap.00F31270     此函数调用了 DialogBoxIndirectParamAorW        
  0012F5C0   00F316A4   softwrap.00F31251                     softwrap.00F3169F               
  0012F5DC   00F31CE7   softwrap.00F31502                     softwrap.00F31CE2     -〉关键的跳转就是在这个函数里          
  0012FE24   00F219CC   ? softwrap.00F316A9                   softwrap.00F219C7               
  0012FE58   004A364C   包含 softwrap.00F219CC                   Tetris40.004A364A             
  0012FF9C   004A1C00   ? Tetris40.004A3598                   Tetris40.004A1BFB  
    
  6.现在我们来分析：softwrap.00FAA5EC 这个函数调用了 DialogBoxIndirectParamAorW 这个函数来创建对话框，双击它，我们看到以下的代码：  
  00FAA5EC    6A 00           push    0  
  00FAA5EE    9C              pushfd  
  00FAA5EF    50              push    eax  
  00FAA5F0    53              push    ebx  
  00FAA5F1    8B5C24 10       mov     ebx, \[esp+10\]  
  00FAA5F5    53              push    ebx  
  00FAA5F6    83EB 06         sub     ebx, 6  
  00FAA5F9    68 911F0000     push    1F91  
  00FAA5FE    68 0000FD00     push    0FD0000  
  00FAA603    C3              retn      //这个 retn 实际上调用了 softwrap 中的一段代码  
  00FAA604    B8 B41E90E0     mov     eax, E0901EB4  
  00FAA609    64:B1 1C        mov     cl, 1C  
  00FAA60C    895C24 0C       mov     \[esp+C\], ebx  
  00FAA610    5B              pop     ebx  
  00FAA611    58              pop     eax  
  00FAA612    9D              popfd  
  00FAA613    C3              retn  
    
  7.看来这里不会有关键的跳转了。经过一层层向上查找后（为了节省时间，我直接给出关键跳转的位置。我怎么找到的？主要就是尝试，另外  
  如果你有一些经验的话，看到 dec eax 和一串 je xxxxx 捆在一起，这明显就是 c 中的 switch-case 语句的遗迹）。  
    双击 0012F5DC   00F31CE7   softwrap.00F31502 这一行，向下滚动，找到以下代码：  
  00D31522    E8 5CD7FFFF     call    00D2EC83  
  00D31527    0FBEC0          movsx   eax, al  
  00D3152A    33DB            xor     ebx, ebx  
  00D3152C    48              dec     eax  
  00D3152D    881D B88BD600   mov     \[D68BB8\], bl  
  00D31533    881D B98BD600   mov     \[D68BB9\], bl  
  00D31539    74 2F           je      short 00D3156A  //If eax==1  
  00D3153B    48              dec     eax  
  00D3153C    0F84 5D010000   je      00D3169F      //If eax==2  
  00D31542    48              dec     eax  
  00D31543    74 1B           je      short 00D31560  //If eax==3  
  00D31545    48              dec     eax  
  00D31546    0F85 58010000   jnz     00D316A4  
  00D3154C    C705 9C8DD600 01000000     mov     dword ptr \[D68D9C\], 1  //If eax==4  
    
  很好，这里是关键的判断处。下一步，究竟 eax 是多少时我们的程序可以运行呢？由于这是在进入我们现在这个对话框之前就执行过的代码，  
  现在在这里下断点是没有用的。  
    
  8.很好，那么让我们按下重新开始程序按钮。在代码行上右击，在菜单中选择“转到->表达式”，在对话框中输入：“00D31527”，程序提示  
  ：指定地址无内存。这是因为 softwrap.dll 还没有加载。看来得想个办法。  
    
  9.在选项-〉调试设置中选择“事件”，钩选"中断于新模块"。现在按下运行程序按钮，有几个模块在“executable modules”中显示出来，可  
  惜没有 softwrap.dll。多次按下运行程序按钮，直到 softwrap.dll 被加载而在“executable modules”中高亮显示出来为止。现在在选项-〉调  
  试设置中选择“事件”，取消钩选"中断于新模块"。  
    
  10.现在在代码行上右击，在菜单中选择“转到->表达式”，在对话框中输入：“00D31527”，你会看到：  
  00D31527    B5 4E           mov     ch, 4E  
  怎么和先前的不一样？因为 softwrap.dll 还没有解压。  
    
  11.在 00D31522 这一行右击，在菜单中选择“断点->内存访问”。  
    
  12.执行程序，开始你会中断到解压的代码中，多按几次运行程序按钮，直到你来到  
  00D31527    0FBEC0          movsx   eax, al  
  现在可以自由修改 eax 的值，看看到底各个分支到什么地方。  
    
  13. 为了节省时间，下面给出各个分支的注释  
  00D31527    0FBEC0          movsx   eax, al  
  00D3152A    33DB            xor     ebx, ebx  
  00D3152C    48              dec     eax  
  00D3152D    881D B88BD600   mov     \[D68BB8\], bl  
  00D31533    881D B98BD600   mov     \[D68BB9\], bl  
  00D31539    74 2F           je      short 00D3156A  //If eax==1 ，程序报告 xlok 服务器错误  
  00D3153B    48              dec     eax  
  00D3153C    0F84 5D010000   je      00D3169F      //If eax==2 ，程序会把我们送到那个你一直看到的对话框那里  
  00D31542    48              dec     eax  
  00D31543    74 1B           je      short 00D31560  //If eax==3 ，程序会威胁你说如果你再实施破解，就让你永远用不成这个软件；-）  
  00D31545    48              dec     eax  
  00D31546    0F85 58010000   jnz     00D316A4  
  00D3154C    C705 9C8DD600 01000000     mov     dword ptr \[D68D9C\], 1  //If eax==4，这个分支就是我梦想要的！直接进游戏！  
  很好，看来我们希望无论如何都去执行分支 4。如果你先前注意的话，一般 eax 的值都是 02.  
    
  14.那么我们需要把分支 02 的 je 00D3169F 改成 je 00D3154C .在 OllyIce 中修改后，我们知道要把 00D3153C 处的 0F 84 5D 01 00 00 修改为 74   
  0E 90 90 90 90。  
    
  15.现在问题出现了。由于文件自解压，自校验，我们不能直接修改文件，因此需要创建一个内存补丁，在程序自解压，自校验之后，但是运行  
  到 00D3153C 之前修改指令。不过幸运的是，早有人写好了专门用于这个问题的软件。  
    
  16. 现在打开 dup2.0。首先新建方案，命名为 tetris4000，然后点“添加”，在对话框里选“偏移量补丁”，选择你创建的“偏移量补丁”，  
  点编辑，把目标文件设为 tetris4000.exe, 然后选择虚拟地址模式，因为我们要在内存中打补丁。  
    
  17.现在在"添加与编辑"对话框中，输入偏移量（00D3153C）,原始字节（0f）,  新的字节（74）。继续输入，直到看起来是这个样子：  
  Virtual Address  原始字节 新的字节  
      00D3153C        0F       74   
      00D3153D        84       0E  
      00D3153E        5D       90  
      00D3153F        01       90  
      00D31540        00       90  
      00D31541        00       90  
    
  18.现在只剩下一个问题了,加载器怎么把握好时机,在程序自解压，自校验之后，但是运行到 00D3153C 之前修改指令呢? 看到右下角的"启用内  
  存校验"选项了吗?这就是解决方法.加载器会监视一个内存地址,当这个内存地址的一个双字被修改为我们事先设定的一个值时,就修改指令.  
    
  19.现在的任务就是去寻找这样一个地址了.到哪里去找呢? 在选项-〉调试设置中选择“事件”，钩选"中断于新模块"。多次按下运行程序按钮  
  ，直到 softwrap.dll 被加载而在“executable modules”中高亮显示出来为止。现在在选项-〉调试设置中选择“事件”，取消钩选"中断于新  
  模块"。在代码行上右击，在菜单中选择“转到->表达式”，在对话框中输入：“00D31527”，在 00D31522 这一行右击，在菜单中选择“断点->  
  内存访问”。  
    
  20.执行程序，开始你会中断到解压的代码中，多按几次运行程序按钮，直到你来到  
  00D31527    0FBEC0          movsx   eax, al  
    
  21。现在察看调用堆栈，如下：  
  调用堆栈：    主线程  
  地址       堆栈       函数过程                              调用来自                        
  0012F5DC   00D31CE7   softwrap.00D31502                     softwrap.00D31CE2  //这是我们所在的地方  
  0012FE24   00D219CC   ? softwrap.00D316A9                   softwrap.00D219C7  //这三个函数都可以作为寻找的目标。  
  0012FE58   004A364C   包含 softwrap.00D219CC                   Tetris40.004A364A//我在这里找到的  
  0012FF9C   004A1C00   ? Tetris40.004A3598                   Tetris40.004A1BFB  
    
  我们要在其中找到一个改变内存中双字的指令。不过我为了节省时间，直接给出结果：  
  双击 0012FE58   004A364C   包含 softwrap.00D219CC 这一行，看到：  
  00D21941 >  55              push    ebp  
  00D21942    8BEC            mov     ebp, esp  
  00D21944    83EC 10         sub     esp, 10  
  00D21947    8B45 10         mov     eax, \[ebp+10\]  
  00D2194A    FF05 647ED600   inc     dword ptr \[D67E64\] <-注意这里  
  00D21950    833D 647ED600 0>cmp     dword ptr \[D67E64\], 2  
  00D21957    53              push    ebx  
  00D21958    56              push    esi  
  00D21959    57              push    edi  
  00D2195A    A3 788ED600     mov     \[D68E78\], eax  
  00D2195F    7C 56           jl      short 00D219B7  
  00D21961    E8 1B260200     call    00D43F81  
  00D21966    8BF0            mov     esi, eax  
  00D21968    C1E6 10         shl     esi, 10  
  00D2196B    E8 11260200     call    00D43F81  
  00D21970    6A 10           push    10  
  00D21972    03F0            add     esi, eax  
  00D21974    33DB            xor     ebx, ebx  
  00D21976    8D45 F0         lea     eax, \[ebp-10\]  
  00D21979    53              push    ebx  
  00D2197A    50              push    eax  
  00D2197B    33FF            xor     edi, edi  
  00D2197D    E8 CE1A0200     call    00D43450  
  00D21982    8D45 F0         lea     eax, \[ebp-10\]  
  00D21985    50              push    eax  
  00D21986    FF75 08         push    dword ptr \[ebp+8\]  
  00D21989    56              push    esi  
  00D2198A    E8 62FDFFFF     call    00D216F1  
  00D2198F    83C4 18         add     esp, 18  
  00D21992    83F8 FF         cmp     eax, -1  
  00D21995    74 1C           je      short 00D219B3  
  00D21997    6A 01           push    1  
  00D21999    8D45 F0         lea     eax, \[ebp-10\]  
  00D2199C    50              push    eax  
  00D2199D    FF75 0C         push    dword ptr \[ebp+C\]  
  00D219A0    E8 0DFFFFFF     call    00D218B2  
  00D219A5    83C4 0C         add     esp, 0C  
  00D219A8    83F8 FF         cmp     eax, -1  
  00D219AB    75 04           jnz     short 00D219B1  
  00D219AD    0BF8            or      edi, eax  
  00D219AF    EB 02           jmp     short 00D219B3  
  00D219B1    8BFE            mov     edi, esi  
  00D219B3    8BC7            mov     eax, edi  
  00D219B5    EB 67           jmp     short 00D21A1E  
  00D219B7    6A 01           push    1  
  00D219B9    68 D923D500     push    00D523D9  
  00D219BE    33DB            xor     ebx, ebx  
  00D219C0    53              push    ebx  
  00D219C1    FF35 687ED600   push    dword ptr \[D67E68\]               ; softwrap.00D20000  
  00D219C7    E8 DDFC0000     call    00D316A9                         ；在这里调用了下层函数  
  00D219CC    E8 B0250200     call    00D43F81  
  00D219D1    8BF0            mov     esi, eax  
  00D219D3    C1E6 10         shl     esi, 10  
  00D219D6    E8 A6250200     call    00D43F81  
  00D219DB    6A 10           push    10  
  00D219DD    03F0            add     esi, eax  
  00D219DF    E8 D11A0200     call    00D434B5  
  00D219E4    6A 10           push    10  
  00D219E6    8BF8            mov     edi, eax  
  00D219E8    53              push    ebx  
  00D219E9    57              push    edi  
  00D219EA    E8 611A0200     call    00D43450  
  00D219EF    57              push    edi  
  00D219F0    FF75 08         push    dword ptr \[ebp+8\]  
  00D219F3    56              push    esi  
  00D219F4    E8 F8FCFFFF     call    00D216F1  
  00D219F9    83C4 1C         add     esp, 1C  
  00D219FC    83F8 FF         cmp     eax, -1  
  00D219FF    74 14           je      short 00D21A15  
  00D21A01    57              push    edi  
  00D21A02    FF75 0C         push    dword ptr \[ebp+C\]  
  00D21A05    E8 16FEFFFF     call    00D21820  
  00D21A0A    83CB FF         or      ebx, FFFFFFFF  
  00D21A0D    3BC3            cmp     eax, ebx  
  00D21A0F    59              pop     ecx  
  00D21A10    59              pop     ecx  
  00D21A11    74 02           je      short 00D21A15  
  00D21A13    8BDE            mov     ebx, esi  
  00D21A15    57              push    edi  
  00D21A16    E8 951A0200     call    00D434B0  
  00D21A1B    59              pop     ecx  
  00D21A1C    8BC3            mov     eax, ebx  
  00D21A1E    5F              pop     edi  
  00D21A1F    5E              pop     esi  
  00D21A20    5B              pop     ebx  
  00D21A21    C9              leave  
  00D21A22    C3              retn  
    
  我选择了 0D2194A    FF05 647ED600   inc     dword ptr \[D67E64\]这条指令，注意，如果你另外找到了很好的数值地址，一样可以用。经过  
  跟踪，我知道他会从 00000000 变为 00000001。  
    
  22.现在在 dup2.0 中，在内存检验中，钩选“启用内存检验”，在“内存地址”一栏中，填写 D67E64，在“内存值”一栏中，填写 00000001。现  
  在保存项目，并创建加载器。  
    
  23.点击加载器，tetris4000 顺利运行！Bingo!  
    
  至于 tetris5000,由于是 asprotect 1.xx-2.xx 的壳，轻松剥掉就行了。然后把"game directory"\\data\\scripts\\general\\init.txt 中的  
  nag\_level = 5 改为 nag\_level = 500 就行了。Have fun!  
     
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2006 年 07 月 19 日 上午 02:15:16 

 4dd466d0e1e0cc123afbe504da2462ca