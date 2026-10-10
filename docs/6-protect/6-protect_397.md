

# 庆祝看雪论坛恢复，发个 anti-od1.1 的 bin，薄码

> 原文：[`www.pediy.com/kssd/pediy10/79930.html`](https://www.pediy.com/kssd/pediy10/79930.html)

这里有些人还没忽悠过

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || anti.rar  
 |

昨天刚刚看到了 shoooo 发的 anti-od1.1。出于个人喜好便下来学习学习。在此感谢 shoooo 又给我一个学习的机会。  
  
现在就把我的学习成果与各位分享。前辈们也请多多指教，指出不足谢谢。  
  
用 OD 载入 anti 文件后果然 OD 挂掉了。  
来看看出错的地址  
  
004A4EE5    3A5B 17                     cmp bl,byte ptr ds:\[ebx+17\]  
  
大家会发现 EBX 的值是 0D0A0D0A.再看看 EBP 指向的栈空间是一堆■■■■的数据。  
看来 EBP 是给被改掉了。  
  
用二进制工具打开 anti 文件找到导入表的文件偏移。会发现 ntdll 后面跟了一串很长的 0D0A0D0A 最后以 00 结束。  
ZwSetInformationProcess 这个函数名也是如此。似乎他两就是罪魁祸首了。呵呵。  
  
接下来就看看哪里把栈给溢出了。  
  
从新加载 OD。  
  
0045F7E6    E8 89CBFFFF                 call <Ollydbg.sub\_45C374>下断点。  
  
OD 加载文件后会断在 0045F7E6 处。F7 跟进。  
由于函数代码比较长就不全部贴出来了。  
  
0045D120    6A 03                             push 3  
0045D122    68 04020000                 push 204         ;指定读取的长度   
0045D127    8B4D CC                         mov ecx,dword ptr ss:\[ebp-34\]  
0045D12A    038D 3CF9FFFF               add ecx,dword ptr ss:\[ebp-6C4\]  
0045D130    51                                   push ecx  
0045D131    8D85 78FDFFFF               lea eax,dword ptr ss:\[ebp-288\]  
0045D137    50                                   push eax  
0045D138    E8 CF410000                  call <Ollydbg.\_Readmemory>                            
0045D13D    83C4 10                          add esp,10  
0045D140    85C0                               test eax,eax  
0045D142    75 4E                              jnz short <Ollydbg.loc\_45D192>    
这里读取 Dll 的名称。  
  
  
0045D192 >  8D85 78FAFFFF               lea eax,dword ptr ss:\[ebp-588\]  
0045D198    50                                    push eax  
0045D199    8D95 78FCFFFF                lea edx,dword ptr ss:\[ebp-388\]  
0045D19F    52                                    push edx  
0045D1A0    6A 00                               push 0  
0045D1A2    6A 00                               push 0  
0045D1A4    8D8D 78FDFFFF               lea ecx,dword ptr ss:\[ebp-288\]  
0045D1AA    51                                    push ecx  
0045D1AB    E8 0C800400                  call <Ollydbg.j\_\_\_fnsplit>  
把后缀.dll 去掉。  
  
  
  
0045D4F2 >  8B55 B8                     mov edx,dword ptr ss:\[ebp-48\]  
0045D4F5    2955 C4                     sub dword ptr ss:\[ebp-3C\],edx  
0045D4F8    8B4D C4                     mov ecx,dword ptr ss:\[ebp-3C\]  
0045D4FB    3B4D A4                     cmp ecx,dword ptr ss:\[ebp-5C\]  
0045D4FE    73 42                         jnb short <Ollydbg.loc\_45D542>  
0045D500    8B45 88                     mov eax,dword ptr ss:\[ebp-78\]  
0045D503    0345 C4                     add eax,dword ptr ss:\[ebp-3C\]  
0045D506    83C0 02                     add eax,2                                         ;得到函数名称  
0045D509    50                              push eax  
0045D50A    8D95 78FCFFFF          lea edx,dword ptr ss:\[ebp-388\]  
0045D510    52                              push edx  
0045D511    E8 BE610400             call <Ollydbg.\_strlen>  
0045D516    59                              pop ecx  
0045D517    8D95 78FDFFFF          lea edx,dword ptr ss:\[ebp-288\]  
0045D51D    B9 00010000             mov ecx,100  
0045D522    2BC8                          sub ecx,eax  
0045D524    8D85 78FCFFFF          lea eax,dword ptr ss:\[ebp-388\]  
0045D52A    83E9 02                     sub ecx,2  
0045D52D    51                              push ecx  
0045D52E    50                              push eax  
0045D52F    68 92C04B00             push Ollydbg.004BC092  
0045D534    52                              push edx  
0045D535    E8 F2960400             call <Ollydbg.\_sprintf>                       ;这里格式化  
0045D53A    83C4 14                     add esp,14  
0045D53D    E9 AA000000             jmp <Ollydbg.loc\_45D5EC>  
  
上面的代码就是格式化字符串，把处理过的 DLL 名和函数名格式化成 USER32.MessageBoxA 的形式。  
当格式化 ntdll 和 ZwSetInformationProcess。注意看栈的变化。我是盯着 EBP  
的地址。果然当格式完后。EBP 被覆盖了。 这里溢出的原因是 DLL 名做了处理，而函数名没有做处理或者说处理不好。导致了在调用 sprintf 函数格式化的时候把栈的溢出了。  
  
问题找到了，花了不少时间。心里挺高兴的，就试着想把这个漏洞补补看。  
  
  
心想既然栈溢出了，无非是栈空间给那两个超长的字符串给弄的。那把缓存弄得足够大，能装的下应该就可以了吧。  
来到函数入口  
0045C374 >  55                          push ebp  
0045C375    8BEC                       mov ebp,esp  
0045C377    81C4 30F9FFFF       add esp,-6D0  
把 ESP-6D0 改大吧。喜欢算准确点也行。我没算 直接改成 esp - 1500 够了吧。  
空间是够大了。还要改缓存的地址。  
原来是 \[ebp-288\]  
以下是我修改的几个地方  
  
0045D517    8D95 78FDFFFF               lea edx,dword ptr ss:\[ebp-1200\]   
  
0045D131    8D85 00EEFFFF               lea eax,dword ptr ss:\[ebp-1200\]  
  
0045D1A4    8D8D 00EEFFFF               lea ecx,dword ptr ss:\[ebp-1200\]  
  
0045D5EC    8D85 00EEFFFF               lea eax,dword ptr ss:\[ebp-1200\]  
  
修改完后再运行试试。哈哈。可以运行了吧。  
  
这个解决方法还有明显缺陷 如果把函数名改更长，还是可以溢出的。  
  
经过和同学讨论觉比较好的解决方法是，把函数名和 DLL 名称  
的总长度求出来，然后动态分配内存。这样就能比较好解决内存溢出的问题。  
而在调试过程中确实发现了比较好的时机获取到 DLL 名和函数名。  
0045D511    E8 BE610400                 call <Ollydbg.\_strlen>  
当运行到这里的时候，栈顶的 2 个值正好是 DLL 名称和函数名称的地址。这里我们就可以改到自己流程求出他们的长度，分配足够  
的内存。  
  
到此，整个分析和修改完成。欢迎各位朋友批评指正，再次感谢 shoooo 给一次了锻炼的机会。也希望能多向前辈们学习，共同进步。  
  
有表述不清楚的地方请多多见谅。分析不足的地方和错误的地方也请各位朋友指正，谢谢。  
  
  
  
                    武汉科锐学员 :dachongGG  
                        
  
                      2009.1.09  
          

我先说说我对它的认识过程（我是个大菜鸟，高手莫笑话偶啊）:  
OD 载入 anti-od1.1，结果还没到入口点 OD 就被 FUCK 了，猜想是输入表的问题。  
拿 loadpe 打开,输入表查看,点 L 看到 ntdll 后面很多换行（兴奋）。  
Name:                0x000040A8  ("ntdll  
  
点 H 打开 40A8 处看一下，后面一大堆 0D0A，0D0A 是回车换行的意思，过一大堆 0D0A 后有 00 结束。到这里我就简单的以为是系统认识 0D0A，OD 不认识造成的错误。然后就放一边了。今天看到有人发个分析的帖子，给出了解决方法结果 shoooo 说只治标，到这里我想到我以前的想法错了。  
看来不动手光靠猜想是不行的。  
  
loadpe 再次打开，我把 0D0A 全改成 3333 就是 ntdl333333333333。。。结果程序一样运行，看来这个输入表项系统填充 IAT 的时候没用上，明明在输入表里，系统怎么没去填充呢？只好点 ntdll 这个输入表项仔细看了，点 H 从 OriginalFirstThunk,FirstThunk 依次看。最后发现问题在 FirstThunk，只要 FirstThunk 指向的 IAT 结构为空，这个表项就不填充。我把 409C 处的值取值随便改成非 0，饭岛爱就运行不起来了（除非 ntdll,ZwSetInformationProcess 改成 00 结束）。我又试了 Kernel32.dllFirstThunk 指向的 IAT 结构后面的值，只要第一个不为 0，整个 IAT 都能正常填充。  
到这里问题就明朗了，shoooo 利用 IAT 结构构造了一个没用的输入表项，只要 IAT 结构为空，这个输入表项的名字和函数可以随便取，反正运行的时候系统不管它。但是我们的 OD 就不行了，它会读这个乱起的名字，如果恶意构造个名字，后果是很严重的。  
  
OK 我们动手补丁下我们的 OD 让它先判断 IAT 结构，为空就不管这项读下个表项。  
  
罗嗦了半天，呵呵。复制一份 OD，用复制的 OD 打开目标 OD 程序，F9，目标程序载入 antiod,  
004A4EE5    3A5B 17                     cmp bl,byte ptr ds:\[ebx+17\]处出错  
在 4A4ED8 处下断重新载入程序，再次载入 antiod 记下异常时的最后一次断下的返回值，45DAED  
看到这个函数太长了，我贴下主要代码  
0045D0E6   > /6A 03         push    3  
0045D0E8   . |6A 14         push    14  
0045D0EA   . |8B45 CC       mov     eax, dword ptr \[ebp-34\]  
0045D0ED   . |0345 BC       add     eax, dword ptr \[ebp-44\]  
0045D0F0   . |0345 FC       add     eax, dword ptr \[ebp-4\]  
0045D0F3   . |50            push    eax  
0045D0F4   . |8D95 30F9FFFF lea     edx, dword ptr \[ebp-6D0\]  
0045D0FA   . |52            push    edx                                            
0045D0FB   . |E8 0C420000   call    \_Readmemory              ////此处为读一项 IMAGE\_IMPORT\_DESCRIPTOR  
0045D100   . |83C4 10       add     esp, 10  
0045D103   . |85C0          test    eax, eax  
0045D105   . |75 0C         jnz     short 0045D113  
0045D107   . |C745 F4 02000>mov     dword ptr \[ebp-C\], 2  
0045D10E   . |E9 1C050000   jmp     0045D62F  
0045D113   > |83BD 3CF9FFFF>cmp     dword ptr \[ebp-6C4\], 0  
0045D11A   . |0F84 0F050000 je      0045D62F  
0045D120   . |6A 03         push    3  
0045D122   . |68 04020000   push    204  
0045D127   . |8B4D CC       mov     ecx, dword ptr \[ebp-34\]  
0045D12A   . |038D 3CF9FFFF add     ecx, dword ptr \[ebp-6C4\]  
0045D130   . |51            push    ecx                                             ///NameRVA  
0045D131   . |8D85 78FDFFFF lea     eax, dword ptr \[ebp-288\]  
0045D137   . |50            push    eax  
0045D138   . |E8 CF410000   call    \_Readmemory           //读 Name 项  
0045D13D   . |83C4 10       add     esp, 10  
0045D140   . |85C0          test    eax, eax  
0045D142   . |75 4E         jnz     short 0045D192  
  
我们需要在读完 IMAGE\_IMPORT\_DESCRIPTOR 还没有读 Name 的时候 patch 判断 FirstThunk  
就在 0045D11A 处补丁。我开始在代码段找了一块 0 串，结果改了不让保存，郁闷！我就自己加了一个区段 VA：1AA000  
45D11A 处改为  
0045D11A   .- E9 E1CE1400   jmp     005AA000  
0045D11F      90                     nop  
5AA000 处代码写：  
005AA000  - 0F84 2936EBFF   je      0045D62F  
005AA006    60                  pushad  
005AA007    6A 00            push    0  
005AA009    6A 04            push    4  
005AA00B    8B4D CC         mov     ecx, dword ptr \[ebp-34\]  
005AA00E    038D 40F9FFFF   add     ecx, dword ptr \[ebp-6C0\]   
005AA014    68 50A05A00     push    005AA050  
005AA019    51              push    ecx                             /////FirstThunk 的值 409C  
005AA01A    A1 685A4D00     mov     eax, dword ptr \[4D5A68\]  
005AA01F    50              push    eax  
005AA020    E8 4951F0FF     call    <jmp.&KERNEL32.ReadProcessMemory>  
005AA025    A1 50A05A00     mov     eax, dword ptr \[5AA050\]  
005AA02A    85C0            test    eax, eax  
005AA02C    61              popad  
005AA02D  - 0F85 ED30EBFF   jnz     0045D120        ///不为 0 继续执行  
005AA033  - E9 DC35EBFF     jmp     0045D614          ///为 0 已读大小加上，再读下一条 IMAGE\_IMPORT\_DESCRIPTOR  
005AA038    90              nop  
  
然后保存。用这个修改的 OllyICE.exe 去打开 antiod 试试没问题，加载别的程序也没出错。  
  
  
加点 OD 对这个 anti 溢出的说明:  
  
有个朋友发帖分析了，我只稍微提下  
  
问题在这个格式化函数处，它把目标格式化成 USER32.MessageBoxA 的形式。用格式化成 USER32.MessageBoxA 作例子说明。  
0045D51D   .  B9 00010000   mov     ecx, 100                            //////作者的期望是格式化的结果在 256 个字节以内。  
0045D522   .  2BC8          sub     ecx, eax                            ///先减去 USER32 的长度，6 个大小  
0045D524   .  8D85 78FCFFFF lea     eax, dword ptr \[ebp-388\]  
0045D52A   .  83E9 02       sub     ecx, 2                              //再减去'.'+结束 00 2 个大小  
0045D52D   .  51            push    ecx                                 ///ecx 就是剩余空间大小  
0045D52E   .  50            push    eax  
0045D52F   .  68 92C04B00   push    004BC092                                      ;  ASCII "%s.%.\*s"  
0045D534   .  52            push    edx                                            /////USER32    这里实际大小只读了 255 个字节，既 DLL 名如果大于 255 个字节，256 位写 00    
0045D535   .  E8 F2960400   call    004A6C2C  
问题就在这里了，如果 USER32 名字大于等于 255，ecx 值将为-1，我们看看它的内部处理  
  
004A6C2C  /$  55            push    ebp  
004A6C2D  |.  8BEC          mov     ebp, esp  
004A6C2F  |.  8B45 08       mov     eax, dword ptr \[ebp+8\]  
004A6C32  |.  8D4D 08       lea     ecx, dword ptr \[ebp+8\]  
004A6C35  |.  C600 00       mov     byte ptr \[eax\], 0  
004A6C38  |.  8D45 10       lea     eax, dword ptr \[ebp+10\]  
004A6C3B  |.  50            push    eax  
004A6C3C  |.  6A 00         push    0  
004A6C3E  |.  6A 00         push    0  
004A6C40  |.  8B55 0C       mov     edx, dword ptr \[ebp+C\]  
004A6C43  |.  52            push    edx  
004A6C44  |.  51            push    ecx  
004A6C45  |.  68 046C4A00   push    004A6C04  
004A6C4A  |.  E8 6D020000   call    004A6EBC  
004A6C4F  |.  83C4 18       add     esp, 18  
004A6C52  |.  5D            pop     ebp  
  
  
格式化都在 4A6EBC，这个函数太长了，只贴关键点说明下。  
         
  
用一个临时变量，我们用 mark 标记，用 IDA 的标记功能比较爽，这里还贴 OD 的吧。  
  
  
根据参数;  ASCII "%s.%.\*s"给 mark 赋值  
走 USER32 时 mark = -1;  
004A6F5B  |.  33D2          |xor     edx, edx  
004A6F5D  |.  8955 F0       |mov     dword ptr \[ebp-10\], edx  
004A6F60  |.  83CA FF       |or      edx, FFFFFFFF  
004A6F63  |.  C645 F7 00    |mov     byte ptr \[ebp-9\], 0  
004A6F67  |.  8955 F8       |mov     dword ptr \[ebp-8\], edx                  ///mark  
  
走 MessageBoxA 时 mark 值为传入的参数 ecx 剩余空间大小, 中间有个处理参数为 0  null 出去的；  
  
关键在这里了：  
004A74A8  |> \\837D F8 00    |cmp     dword ptr \[ebp-8\], 0             
004A74AC  |.  7C 05         |jl      short 004A74B3  
004A74AE  |.  8B45 F8       |mov     eax, dword ptr \[ebp-8\]  
004A74B1  |.  EB 05         |jmp     short 004A74B8  
004A74B3  |>  B8 FFFFFF7F   |mov     eax, 7FFFFFFF  
004A74B8  |>  8B55 E8       |mov     edx, dword ptr \[ebp-18\]  
004A74BB  |.  33C9          |xor     ecx, ecx  
004A74BD  |.  894D C0       |mov     dword ptr \[ebp-40\], ecx  
004A74C0  |.  EB 05         |jmp     short 004A74C7  
004A74C2  |>  48            |/dec     eax  
004A74C3  |.  FF45 C0       ||inc     dword ptr \[ebp-40\]  
004A74C6  |.  42            ||inc     edx  
004A74C7  |>  85C0          | test    eax, eax  
004A74C9  |.  0F84 CF000000 ||je      004A759E  
004A74CF  |.  803A 00       ||cmp     byte ptr \[edx\], 0  
004A74D2  |.^ 75 EE         |\\jnz     short 004A74C2  
004A74D4  |.  E9 C5000000   |jmp     004A759E  
  
///mark < 0 就记下 dword ptr \[ebp-18\]字符串的长度保存在 dword ptr \ebp-40\中，后面用这个长度格式化，不小于 0 取 mark 值和 dword ptr \[ebp-18\]字符串的长度的最小值。  
如果 DLL 名字大于等于 255 字符，分配的栈空间是按格式化 256 字符长分配的, 函数名超长的再格式化自然溢出了。如果 DLL 名字小于 255 字节，.后的函数名取前面字节填充 256 字节剩余空间。 这样顶多出现函数名不全的情况，不会溢出。  
  
  
window DLL 文件名好象只能取到 240 多位（我随便重命名了一个文件，数了下）。 保证程序能运行的情况下用我上面的 patch 方法不会出现溢出的情况  
  
实际上这样 patch 以后也只能保证加载的时候不溢出. 如果程序在运行的时候把 IMAGE\_IMPORT\_DESCRIPTOR 改的面目全非,程序运行的时候 IAT 填充完了,不影响程序运行.OD 这时附加即使不崩溃也读的内容乱七八糟.  读输入表 0045D0FB   . |E8 0C420000   call    \_Readmemory              ////此处为读一项 IMAGE\_IMPORT\_DESCRIPTOR 的这句所在函数过程需要说明下,它的作用如下  
在 OD 的 CPU 窗口里 call    dword ptr \[402198\] ////显示成 00401770  call    dword ptr \<&MSVCRT.\_\_p\_\_commode>;  针对它的解决方法是 NOP 掉读原始输入表的过程,但是这样似乎不太好. 我的方法就是用海风的插件,哈哈!!!  海风的插件真是好东西,支持海风!!!!!!  
  
  
附上我修改的 OllyICE.exe

上传的附件
| ![ || OllyICE.rar \[附件下载：http://bbs.pediy.com/showthread.php?t=80224 \]   
 |