

# 耐心的体验手脱 SVK1.43 加壳记事本（两种方法）

> 原文：[`www.pediy.com/kssd/pediy10/86198.html`](https://www.pediy.com/kssd/pediy10/86198.html)

【文章标题】: 耐心的体验手脱 SVK1.43 加壳记事本（两种方法）  
【文章作者】: zyqex  
【软件名称】: 加壳 NotePad98  
【下载地址】: 附件  
【加壳方式】: SVK1.43  
【使用工具】: ollyice  
【操作平台】: xp  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  【前语】  
    
   唔~又是记事本！！呵呵，没办法，水平菜，最近忙着找工作，好歹脱了四五天，又晾了一个周，   还是拿出来给部分人  
  分享一下（高手就飘过吧）。我将用两种方法演示：1.常规方法  2.以壳解壳  
    
  【Begin】  
    
  方法一：  
    
   一.先大致介绍一下壳用到的技术：  
   1.加密 iat  
   2.stolen code (oep 处)  
   3.replace code (mov 类的)  
   4.还有一个替换 api 的，不知该归为什么（我首次接触 stolen code 等概念，失误之处敬请诸位赐教）  
  二.  
    1.peid 查看，SVKP 1.3x -> Pavol Cerven（其实是 1.43，我用论坛里的那个 svk1.43 的版本）  
    od 载入，开始的时候提示打开文件错误，不用管点确定就是，Alt+M 看看，只看得到一个区段  
    （至少在我的机器上是这样）。  
   地址=00400000  
   大小=0002D000 (184320.)  
   属主=NOTEPAD\_ 00400000 (自身)  
   区段=  
   包含=PE 文件头  
   类型=Imag 01001002  
   访问=R  
   初始访问=RWE  
   2.设置一下 od:  
    olly advance1.26 beta 12 插件里勾选 IsDebuggerPresent 及 Anti-RdTSC 下的 Enable  
    异常设置，只勾选“忽略在 kernel32 中的内存访问异常”（忽略所有异常，按下面的做法我的 od 会死，所以就麻烦一点吧）  
  下面的跟 chasgone《某外挂 SVKP 1.4x -> Pavol Cerven 脱壳》的部分类似，我就直接引用一下了（呵呵，应该不会犯错吧）  
  【引用】  
   搞定 iat                        //避过 IAT 加密的处理，这样修复时就容易多了  
  od 载入   下断点 bp GetModuleHandleA+5            //到达处理 IAT 的地方，随意跟踪过壳的不会不知道  
     
  shift＋f9 断下，取消断点，ctrl＋f9 返回主程序。  
    
  ctrl＋f 搜索特征码  
    
  cmp dword ptr ds:\[ebx\],251097CC  
    
  把这下面的所有的比较和跳转全部 nop 掉  
    
  然后在搜索特征码  
  mov dword ptr ds:\[edi\],eax           //此处处理甚妙，改过后 iat 里写入的直接就是调用的函数的地址了  
  popad  
    
  改成：  
  popad  
  mov dword ptr ds:\[edi\],eax  
    
  【引用结束】  
  接着上面  
  0B755B4F    8907            mov     dword ptr \[edi\], eax        //即上面要修改的地方  
  0B755B51    61              popad  
  0B755B52    8385 43010200 0>add     dword ptr \[ebp+20143\], 4  
  0B755B59  ^ E9 ADFBFFFF     jmp     0B75570B  
  0B755B5E    83C6 14         add     esi, 14  
  0B755B61    8B95 871C0300   mov     edx, dword ptr \[ebp+31C87\]  
  0B755B67    C685 32550200 0>mov     byte ptr \[ebp+25532\], 0  
  0B755B6E    C685 33550200 0>mov     byte ptr \[ebp+25533\], 0  
  0B755B75  ^ E9 9DFAFFFF     jmp     0B755617  
  0B755B7A    33C0            xor     eax, eax  
  0B755B7C    64:8F00         pop     dword ptr fs:\[eax\]  
  0B755B7F    83C4 04         add     esp, 4  
  0B755B82    C3              retn       //在这里下断（这时 iat 就处理完了,下断后 F9 运行）  
    
  断下后，取消 0B755B82 处断点。  
  接着 Alt+M 在那个主程序唯一的区段上 F2 下断，F9 运行  
  断在：  
  0013FF17    8B89 78010000   mov     ecx, dword ptr \[ecx+178\]  
  0013FF1D    8BAD 42210000   mov     ebp, dword ptr \[ebp+2142\]  
  f8 步过 0013FF17 处后（不步过待会下断运行后会原地停留），Alt+M 在那个主程序唯一的区段上 F2 下断，F9 运行  
  断在：  
  0B7DF5F4    FF15 E4634000   call    dword ptr \[4063E4\]           //其实这两句是被偷的完整的两句代码  
  0B7DF5FA    89C6            mov     esi, eax                       
  F7 步进一下（F8 或 F9 就跑飞），Alt+M 在那个主程序唯一的区段上 F2 下断，F9 运行  
  断在：  
  004010DD    3C 22           cmp     al, 22                   //foep 处  
  004010DF    75 1B           jnz     short 004010FC  
  004010E1    56              push    esi  
  004010E2    FF15 AC644000   call    dword ptr \[4064AC\]  
  很明显的一个大跨段的跳跃了^\_^  
  向上翻看：  
  004010C7    000D 0A000090   add     byte ptr \[9000000A\], cl  
  004010CD    90              nop  
  004010CE    90              nop  
  004010CF    90              nop  
  004010D0    90              nop  
  004010D1    90              nop  
  004010D2    90              nop  
  004010D3    90              nop  
  004010D4    90              nop  
  004010D5    90              nop  
  004010D6    90              nop  
  004010D7    90              nop  
  004010D8    90              nop  
  004010D9    90              nop  
  004010DA    90              nop  
  004010DB    90              nop  
  004010DC    90              nop  
  004010CC--004010DC 被偷了。  
  拿起 importrec  填 oep=10dd，自动搜索 iat  全部有效 保存为树文件 后面有用  
  好了，现在我们来搞定 stolen code  
  对于 sloen code 的处理有（又一篇文章里的引用^\_^）  
  【引用】  
  找回抽掉的代码的方法一般有：  
  1、TC 跟踪得到  
  2、tDasm 老兄介绍的还原 stolen code 的分析法  
  3、FLY 老兄介绍的 cdi 猜测法，这个我还是未明白  
  4、blowfish 老兄介绍的建立 stack 法，这个方法非常好。  
  【引用结束】  
  关于 2：看雪精华里搜 tDasm 相关文章吧，没搜到合适的  
  关于 3：很遗憾我没搜索到（发了个求助帖，好像没人知道，知道的补充一下吧。）  
  关于 4：看雪精华 6 里搜《重建 stolen bytes 的一般方法》（建议去好好的看看，非常不错的一篇文章）  
  ----------------------------------------------------------------------------------------------  
  下面我大致介绍一下我自己找的过程中用的方法（可能有些愚笨，见谅！），上面的 1、2、4 的混用。  
    
  先到 foep 后记下寄存器的值（我们可以注意到 foep 开始以后的代码里用到了寄存器 eax,esi,  
  所以在跟踪 stolen code 时要注意这里个寄存器的变化。），以及堆栈内 ebp 到 esp 的内容。  
    
  先看看代码的入口特征，vc 的，大致的跟踪一遍，看程序在什么地方跳向 foep 处，这个地方以上的一些代码一般就是  
  模拟被偷代码的功能部分了，在跟踪过程中我们注意什么时候 ebp 变为到 foep 后 ebp 的值，注意寄存器的变化，我自己  
  琢磨的 tc 跟踪法应该是这样使用：重载程序，按二的步骤到达地址 0B755B82 处，然后在 foep(4010dd)处下断，Ctrl+F11  
  (步进跟踪)，停下后看记录。  
    
  引用 blowfish 一句话：“重建 stolen bytes 的目的，是为了让局部变量、全局变量、寄存器的初值在执行到 fake OEP 时  
  和未加壳前的保持一致，并非要精确还原这些 stolen 的指令，只要功能相同即可。 ” 加壳程序要恢复被偷处的功能也  
  是同样的道理。（不清处见谅，水平太菜了！亲自跟上几遍也许就明白了^\_^）  
    
  依照这样一个思路，跟踪到下面的代码(只列出了关键部分)：  
  0B7E0095   主  sub     esp, 44  FL=A, ESP=0013FF7C              //被偷代码  
  0B7E0098   主  push    eax  ESP=0013FF78                    //被偷代码（注意到 esp 的值）  
  0B7E0099   主  add     esi, 8B8A3F66  FL=S, ESI=8B9DFB7A  
  0B7E009F   主  jmp     0B7DE2B3  
  ……  
  0B7DF5ED   主  sub     dword ptr \[esp\], eax  FL=P  
  0B7DF5F0   主  pop     esi  ESP=0013FF74, ESI=0013BC14  
  0B7DF5F1   主  mov     edi, edi  
  0B7DF5F3   主  pop     eax  EAX=0B7DE159, ESP=0013FF78  
  0B7DF5F4   主  call    dword ptr \[4063E4\]  ESP=0013FF74      //被偷代码  
  0B7DF5FA   主  mov     esi, eax  ESI=00152398             //被偷代码（注意 esi 值）  
  0B7DF5FC   主  mov     al, byte ptr \[eax\]  EAX=00152322     //被偷代码  
  0B7DF5FE   主  push    74B5D177  ESP=0013FF74  
  0B7DF603   主  add     eax, 8B8A3F66  FL=PS, EAX=8B9F6288  
  0B7DF608   主  push    eax  ESP=0013FF70  
  0B7DF609   主  mov     eax, 8B8A3F66  EAX=8B8A3F66  
  0B7DF60E   主  sub     dword ptr \[esp\], eax  FL=P  
  0B7DF611   主  jmp     0B7DE83A  
  0B7DE83A   主  mov     eax, dword ptr \[esp\]  EAX=00152322  
  0B7DE83D   主  mov     eax, 8B8A3F66  EAX=8B8A3F66  
  0B7DE842   主  add     dword ptr \[esp+4\], eax  FL=CP  
  0B7DE846   主  pop     eax  EAX=00152322, ESP=0013FF74  
  0B7DE847   主  add     esp, 4  FL=P, ESP=0013FF78  
  0B7DE84A   主  jmp     dword ptr \[esp-4\]                //这里跳向 foep  
    
  c 程序的入口为 push ebp     mov ebp,esp 这两句在上面的代码里被变形了，罗列一下 stolen code:  
  push ebp  
  mov ebp,esp  
  sub esp,44  
  push eax  
  call dword ptr ds:\[4063E4\]                     
  mov esi,eax  
  mov al,byte ptr ds:\[eax\]  
  补到 00401010cc 处并保存到可执行文件，然后重新用 od 打开，运行到 oep 处后，用 LoadPe dump 程序，改  
  入口为 004010CC。  
  当然使用 blowfish 介绍的建立 stack 法会容易一些，有兴趣的可以自己试试。  
  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  三.修复被替换的 api  
  上面的步骤完成后，如果使用步骤二保存的树文件修复 iat，运行程序没反应。用 od 加入看看，很多调用 MessageBox  
  和 ExitProcess 的地方。  
    
  现在我们知道了了 foep--004010dd,od 载入“加壳记事本”， bp GetModuleHandleA+5 断下后直接在 foep 处下断。  
  断下后，我们看到程序里很多 call    dword ptr \[4064AC\]和 call    dword ptr \[4063A0\]。  
  随便找一个调用的地方 f7 进去看看（可以在下面的语句设断，然后 Ctr+F11），查看记录：  
    
  会发现如下的关键地方：  
  \*\*\*\*\*\*\*对于 call    dword ptr \[4064AC\]\*\*\*\*\*\*\*\*  
  00F2A078    9C              pushfd               //004064ac  
  00F2A079    55              push    ebp  
  00F2A07A    E8 00000000     call    00F2A07F  
  --------------------------------------------------------------------  
  0013FE64    833C31 00       cmp     dword ptr \[ecx+esi\], 0             //关键循环 1  
  0013FE68    74 64           je      short 0013FECE              
  0013FE6A    50              push    eax  
  0013FE6B    807D 36 01      cmp     byte ptr \[ebp+36\], 1  
  0013FE6F    75 0C           jnz     short 0013FE7D  
  0013FE71    2B85 3E010000   sub     eax, dword ptr \[ebp+13E\]  
  0013FE77    0385 42010000   add     eax, dword ptr \[ebp+142\]  
  0013FE7D    390431          cmp     dword ptr \[ecx+esi\], eax         ; NOTEPAD.004010E4  
  0013FE80    58              pop     eax  
  0013FE81    74 07           je      short 0013FE8A  
  0013FE83    83E9 09         sub     ecx, 9  
  0013FE86    74 46           je      short 0013FECE  
  0013FE88  ^ EB DA           jmp     short 0013FE64  
    
  eax=004010E4 (NOTEPAD.004010E4)  
  ds:\[00F2CB14\]=0040353F (NOTEPAD.0040353F)  
  ------------------------------------------------------------------------  
  0013FED9    833C31 00       cmp     dword ptr \[ecx+esi\], 0           ; 关键循环 2  
  0013FEDD    74 48           je      short 0013FF27                    
  0013FEDF    50              push    eax  
  0013FEE0    807D 36 01      cmp     byte ptr \[ebp+36\], 1  
  0013FEE4    75 0C           jnz     short 0013FEF2  
  0013FEE6    2B85 3E010000   sub     eax, dword ptr \[ebp+13E\]  
  0013FEEC    0385 42010000   add     eax, dword ptr \[ebp+142\]  
  0013FEF2    390431          cmp     dword ptr \[ecx+esi\], eax       //eax 存储的是需要被修改的地方的地址  
  0013FEF5    58              pop     eax  
  0013FEF6    74 07           je      short 0013FEFF  
  0013FEF8    83E9 08         sub     ecx, 8  
  0013FEFB    74 2A           je      short 0013FF27  
  0013FEFD  ^ EB DA           jmp     short 0013FED9  
  --------------------------------------  
  0013FF24    9D              popfd  
  0013FF25  - FF20            jmp     dword ptr \[eax\]          跳向正确函数  
  其实上面代码中两个循环的  ecx+esi 指向两张表（跟踪时可到数据窗口看看，不清楚的建议看看  
  《某外挂 SVKP 1.4x -> Pavol Cerven 脱壳》chasgone 的文章，就不细说了，大同小异，处理方法稍有区别）。  
    
  表 1：  
     0B7DCAFD  00 00 00 00 00 00 00 00 00 35 DC 64 40 00 （70 37  .........5@.p7  
     0B7DCB0D  40 00） 35 DC 64 40 00 （3F 35 40 00） 3D E4 64 40 00  @.5@.?5@.=@.  
     0B7DCB1D  （5D 2E 40 00） 00 00 00 00 00 43 3A 5C 44 6F 63 75  \].@......C:\\Docu  
     （）里的表示壳会修改的地址，其前面的四个字节就是修改后的值  
     比如说：E4 64 40 00 （5D 2E 40 00）  
     这里()里的地址 00402e5d 处的内容将会被修改为 004064e4(这个值指向被调用的正确的函数地址)  
     即：待修复的地址为\[0B7DCB1D-9\*n\],修改为的值是\[0B7DCB1D-9\*n-4\],n=0,1,2,3……  
    
  表 2：  
  0B7DB72E                                      F4 64 40 00              @.  
  0B7DB73E  8F 4F 40 00 F4 64 40 00 1A 4F 40 00 F4 64 40 00  @.@.O@.@.  
  0B7DB74E  3C 4E 40 00 F4 64 40 00 A7 4D 40 00 F4 64 40 00  <N@.@.@.@.  
  0B7DB75E  E1 4C 40 00 F4 64 40 00 56 4C 40 00 DC 64 40 00  @.@.VL@.@.  
  0B7DB76E  24 4C 40 00 DC 64 40 00 0D 4C 40 00 DC 64 40 00  $L@.@..L@.@.  
  0B7DB77E  FB 4B 40 00 DC 64 40 00 E8 4B 40 00 DC 64 40 00  @.@.@.@.  
  ……    
  0B7DBCEE  41 12 40 00 EC 64 40 00 27 12 40 00 F4 64 40 00  A@.@.'@.@.  
  0B7DBCFE  1B 11 40 00 F4 64 40 00 03 11 40 00 F4 64 40 00  @.@.@.@.  
  0B7DBD0E  E4 10 40 00                                      ?@..  
       
  表 2 和表 1 类似，比如说  
  0B7DBCFE  1B 11 40 00 F4 64 40 00 03 11 40 00 F4 64 40 00  @.@.@.@.  
  0B7DBD0E  E4 10 40 00  
  0B7DBD0E 地址处是要修复的地址，0B7DBD0A 地址处是修改为的值，接下来 0B7DBD06 处的值又是待修复的地址……  
  即：待修复的地址为\[0B7DBD0E-4\*2n\],修改为的值是\[0B7DBD0E-4\*（2n+1)\],n=0,1,2,3……  
    
  另外对于 call    dword ptr \[4063A0\]的处理也是一样的，不多说了，直接给出另两个表  
  表 3：  
  0B7BA122  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
  0B7BA132  3D BC 63 40 00 （3A 34 40 00） 1D A4 63 40 00 （F0 2F  =@.:4@.@.?  
  0B7BA142  40 00） 35 D8 63 40 00 （E0 2F 40 00） 35 D0 63 40 00  @.5@.?@.5@.  
  0B7BA152  （5A 19 40 00） 00 00 00 00 9C 55 E8 00 00 00 00 5D  Z@.....?...\]  
  表 4  
  0B7B8EE3  00 00 00 00 00 00 00 00 00 00 00 00 A4 63 40 00  ............@.  
  0B7B8EF3  62 4F 40 00 A4 63 40 00 49 4F 40 00 A4 63 40 00  bO@.@.IO@.@.  
  0B7B8F03  F0 4E 40 00 A4 63 40 00 77 4E 40 00 A4 63 40 00  @.@.wN@.@.  
  0B7B8F13  5D 4E 40 00 A4 63 40 00 12 4E 40 00 94 63 40 00  \]N@.@.N@.@.  
  ……  
  0B7B9303  4C 16 40 00 D0 63 40 00 A7 14 40 00 D0 63 40 00  L@.@.?@.@.  
  0B7B9313  66 14 40 00 D8 63 40 00 0C 14 40 00 DC 63 40 00  f@.@..@.@.  
  0B7B9323  BA 11 40 00 D4 63 40 00 8E 11 40 00 D4 63 40 00  ?@.@.?@.@.  
  0B7B9333  79 11 40 00 9C 63 40 00 4E 11 40 00 98 63 40 00  y@.@.N@.@.  
  0B7B9343  33 11 40 00   
    
  \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  这样很容易就可以写出手动的修复代码来：  
  0B800000    9C              pushfd                                   //表 2  
  0B800001    60              pushad  
  0B800002    BE 3EB77D0B     mov     esi, 0B7DB73E  
  0B800007    8B06            mov     eax, dword ptr \[esi\]  
  0B800009    8B5E FC         mov     ebx, dword ptr \[esi-4\]  
  0B80000C    8918            mov     dword ptr \[eax\], ebx  
  0B80000E    83C6 08         add     esi, 8  
  0B800011    81FE 0EBD7D0B   cmp     esi, 0B7DBD0E  
  0B800017  ^ 7E EE           jle     short 0B800007                  //此处跳转地址以分配的为准  
  ----------------------------------------------------------  
  0B800019    BE F38E7B0B     mov     esi, 0B7B8EF3                        //表 1  
  0B80001E    8B06            mov     eax, dword ptr \[esi\]  
  0B800020    8B5E FC         mov     ebx, dword ptr \[esi-4\]  
  0B800023    8918            mov     dword ptr \[eax\], ebx  
  0B800025    83C6 08         add     esi, 8  
  0B800028    81FE 43937B0B   cmp     esi, 0B7B9343  
  0B80002E  ^ 7E EE           jle     short 0B80001E                //此处跳转地址以分配的为准  
  -----------------------------------------------------------  
  0B800030    BE 0BCB7D0B     mov     esi, 0B7DCB0B  
  0B800035    8B06            mov     eax, dword ptr \[esi\]  
  0B800037    8B5E FC         mov     ebx, dword ptr \[esi-4\]  
  0B80003A    8918            mov     dword ptr \[eax\], ebx  
  0B80003C    83C6 09         add     esi, 9  
  0B80003F    81FE 1DCB7D0B   cmp     esi, 0B7DCB1D  
  0B800045  ^ 7E EE           jle     short 0B800035                   //此处跳转地址以分配的为准  
  -----------------------------------------------------------  
  0B800047    BE 37A17B0B     mov     esi, 0B7BA137  
  0B80004C    8B06            mov     eax, dword ptr \[esi\]  
  0B80004E    8B5E FC         mov     ebx, dword ptr \[esi-4\]  
  0B800051    8918            mov     dword ptr \[eax\], ebx  
  0B800053    83C6 09         add     esi, 9  
  0B800056    81FE 52A17B0B   cmp     esi, 0B7BA152  
  0B80005C  - 0F8E EAFF2FFD   jle     0B80004C                       //此处跳转地址以分配的为准  
  0B800062    61              popad  
  0B800063    9D              popfd  
    
  上面的代码就是修复代码，修复的时候用插件分配一块内存，然后写入以上代码，我的是 0B800000 开始。  
  打开步骤二 dump 的程序，分配一块内存，写入上面的代码，然后新建在代码开始处 eip，执行完后让它跳回 oep.  
  接着再 dump 一下程序，运行 dump 的程序，用 import rec 选择进程（注意是 dump 后程序的进程），然后载入  
  步骤二保存的的树文件，填入 oep 4010cc,fix dump.  
    
  四.修复 replace code (mov 类)  
   到了这里程序还是不能运行，其实是因为表 1 里地址处的 mov 指令被替换成了 call 指令。发生的地点就是表 1、3  
  （）里需要修复的地址-2 的地方。罗列一下，即地址：  
  0040343a-2，00402ff0-2，00402fe0-2，0040195a-2，00402E5D-2，0040353F-2，00403770-2  
  修复方法：  
  比如说：  
  0040343a-2 处，od 载入步骤三得到的程序，然后 Ctrl+G 输入 0040343a-2 到：        
  00403438       FF15 BC634000       call dword ptr ds:\[<&kernel32.lstrcat>\]      ;  kernel32.lstrcatA  
  0040343E   |.  FFD7                call edi  
  在 00403438 处按空格键（Space），出现 call dword ptr ds:\[4063BC\]，将其改为 mov edi,dword ptr ds:\[4063BC\]  
  其他地方的处理方法类似，mov 后面的寄存器为上面 Ctrl+G 后地址下面的 call 所用的寄存器，就像上面的  
  call dword ptr ds:\[<&kernel32.lstrcat>\]下面的 call edi。  
  就 7 个地方，手动修改不是很麻烦。  
  修改完后保存到可执行文件。  
  运行程序，这样就 OK 了。^\_^ ^\_^  
  最后优化一下，我优化的结果是和原记事本一样大，这个就不说了，请参看其他文章或者《加密与解密 3》有关章节。  
  -------------------------------------------------------------------------------------------------------  
  方法二：（待续，写了快一下午，休息一下^\_^）  
    
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2009 年 04 月 13 日 19:05:46

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || NOTEPAD\_98.rar\[解压密码：PEDIY\]  
 |

方法二：  
  呵呵，回来了，继续吧！  
  -----------------------------------------------------------------------------------  
  以壳解壳：  
  注意，这里我是在找回了 stolen 的情况下进行的，本来想连 stolen 的地方也用壳自身的代码解决，无奈还得用个跳转语句来实现跳向那段代码，程序还膨胀到 800K，晕，因为方法一得出了 stolen code 所以就直接贴上去，而将重点放在 iat 的处理上。  
  1.od 载入加壳记事本，bp GetModuleHandleA+5，F9 运行断下后，直接在 oep 处下断，运行断下    ，然后 lodpe dump 程序，记得要验证一下 pe,重新载入 dump 出的程序，来到 4010cc,把方法一找到的 stolen  code 补上去，注意要将 004063e4 地址处的数据改为 7c812f2d (kernel32GetCommandLineA 的地址),保存到可执行文件。  
  2.区域脱壳：  
  跟踪一下壳里面调用函数的地方，可以知道它跳向地址 0b7XXXXX 处，只要程序包含了这部分代码，程序自然就能跳向正确的函数。  
  od 载入加壳记事本，bp GetModuleHandleA+5 运行断下后，用 lodpe 区域脱壳：右键->区域转存  
，选择地址 0B7B0000-0B7E9000,默认保存为 Region0B7B0000-0B7E9000。然后，用 lodpe 的 pe 编辑器载入步骤 1 dump 出的程序，点击【区段】 ，然后右键->从磁盘载入段，选择文件 Region0B7B0000-0B7E9000.dum,右键->编辑区段，改虚拟地址为 0B7B0000-00400000（基址）=0B3B0000，改 oep 为 10cc（记得点击保存）  
 3.载入步骤 2 得到的程序，运行，出现异常：  
 地址=0013FDD5  
 消息=访问违规: 正在写入到 \[0B7F2324\]  
  
0013FDC2    61              popad  
0013FDC3    8BC4            mov     eax, esp  
0013FDC5    8BA5 AACE0200   mov     esp, dword ptr \[ebp+2CEAA\]  
0013FDCB    81AD AACE0200 0>sub     dword ptr \[ebp+2CEAA\], 100  
0013FDD5    50              push    eax  
分析一下，原来 0013FDCB 处 esp 被改为 0B7F2324 了（尝试修改内存访问属性失败），幸好还有一招，改\[ebp+2CEAA\]处的 0B7F2324 为 0013d500(当前堆栈里的一个地址，这个地址的附近不要有数据)，保存到可执行文件。运行一下程序，可以执行（还没完啦！）  
4.修正  
点菜单【新建】、【打开】等都会让程序崩溃，汗~。od 载入吧，运行，提示一个地址(7xxxxxxx,没记下来太忙了，见谅！)访问异常。好吧，现在我们 alt+e 看看 7xxxxxxx 处，可看到没有这个地址处的模块调用，od 载入加壳记事本，alt+e 可看到  
Executable modules, 条目 5  
 基址=76320000  
 大小=00047000 (290816.)  
 入口=76321AB8 comdlg32.<模块入口点>  
 名称=comdlg32 (系统)  
 文件版本=6.00.2900.2180 (xpsp\_sp2\_rtm.04  
 路径=C:\\WINDOWS\\system32\\comdlg32.dll  
原来是没有加载 comdlg32.dll~  
好，写点小代码实现吧。  
我是在壳的入口地址 0040c000 处写下：  
push 0040c100    ；这个地址写入字符串“comdlg32.dll”，不要忘记标记字符串结束的 0  
call 7c801d77    ；LoadLibrary 的地址    
jmp 004010cc  
保存到可执行文件，lodpe 改入口为:0040c000  
运行一下，ok 了。（脱出来 400k,两种方法很鲜明的对比！）  
  
【结束】  
\----------------------------------------------------------------------------------  
【结语】  
 最近太忙了，写得有些仓促，错误之处请指正！