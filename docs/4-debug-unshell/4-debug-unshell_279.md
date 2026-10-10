

# 老生常谈 Armadillo CopyMem-II+Debug-Blocker 保护方式脱壳

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-552.htm`](https://www.pediy.com/kssd/pediy09/pediy09-552.htm)

> 算一算时间很久没有写文章了，时间过的可真快，这次提起笔写写，一方面总结一下，另一方  
> 面证实一下自己还不算老吧。十一长假过了大半，培着老婆天天在家里，也算是打发一下时间吧。  
> 写些什么呢，自己很清楚，这些年来一直在看雪和一衰烟雨论坛混，可是自己在脱壳方面还是壳盲。  
> 最后决定还是谈谈 Armadillo CopyMem-II+Debug-Blocker 保护方式脱壳，都是一些总结性的，都是  
> 轻车熟路的套路，没什么创新，大牛们就别浪费时间了，小鸟们倒是可以一看，也许会有些收获。  
>   
>     这里我总结了一下方法：手工方法和工具辅助方法，再加上我的一点点"创新"。先简单说一下  
> 这种保护方式特点：父进程调试子进程，在子进程里负责 IAT 的加密，子进程到达 OEP 后触发异常，  
> 父进程负责子进程代码解密，然后再按照一定的算法加密回去，防止子进程被 Dump。现有的脱壳方  
> 式先 dump 子进程，然后再单独恢复 IAT，这是因为 IAT 的加密在子进程里,OD 无法调试子进程。OD 是 3  
> 环的调试器，我们用 0 环的调试器就可以调试子进程了。当时我用 softice+iceext 调试子进程时也费  
> 了一番周折，毕竟穿山甲反 ICE 还是很厉害的。现在我用国货：Syser，试了一段时间发现 Syser 稳定  
> 性还算不错，至少在我的机器是这样的，而且穿山甲对其没有任何的防护，这也许是 Syser 在脱壳方  
> 面的第一篇文章吧!让我们开始吧!我们先用新版的 Armadillo 给 98notepad 加壳作为试练品。在调试  
> 过程中发现 OD 和 Syser 在硬件断点上打架，可在 OD"调试选项">"调试"中去掉"使用硬件断点单步执行  
> 或跟踪代码"复选框。  
>   
>   一、手工篇  
>         (一)方法 1  
>       
>   该方法也许是最早的方法了，国人某牛人首创，该方法简单易行，老少皆宜。  
>   先 bp WaitForDebugEvent，shift+f9,断下后 ALT+F9 返回，bc WaitFordebugEvent 去掉断点。  
>   "查找">"所有常数">FFFFFFF8, 双击该行 "00434FAF  or eax, FFFFFFF8",向上来到:  
>       
> /\*434F68\*/  cmp     dword ptr \[ebp-A60\], 0  //这里下断点  
> /\*434F6F\*/  jl      0043521A  
> /\*434F75\*/  mov     ecx, dword ptr \[ebp-A60\]  
> /\*434F7B\*/  cmp     ecx, dword ptr \[474414\]  
> /\*434F81\*/  jge     0043521A  
>    
>    我们在 434F68 处 F2 下断点.启动 Syser,Ctrl+F12 激活,>bpm outputdebugstringa x,下硬件断点,  
> 我们利用这个断点来断下子进程.在 OD 里 shift+f9 运行,中断在 Syser 里,按 x 运行,这里中断 4 次,前 2  
> 次是父进程产生的,后 2 次是子进程产生的,第 4 次中断后,>bd 0 禁用该断点,>bpm getmodulehandlea   
> x 下硬件断点,我们开始找魔法跳.按 x 运行,我们观察堆栈窗口,当显示"VirtualAlloc"时,再按 x 运  
> 行一次,断下后,>bd 1 禁用该断点. >g \*esp,运行到子进程空间,往下找到魔法跳:  
>   
> 00d57ec6   jnz 00d57ecd   //魔法跳  
> 00d57ec8   jmp 00d57dbc  
>     
> \>a 00d57ec6 nop   //nop 掉魔法跳  
> \>a 00d57ec7 nop   //nop 掉魔法跳  
>   
> \>bpm createthread x //下断点在子进程中找 OEP  
> \>x 运行,断下后>bd 2,F6 运行到返回,F8 单步一下,F6 运行到返回,F8 单步一下,往下找到:  
>   
> 00d8f1b4 call ecx  //选中,F7 运行到该行,此时 ecx=004010cc=OEP,F8 单步一下,此时在 OEP 处还没  
> 解码,触发异常中断在 OD 里,我们可以在 OD 里给子进程进行解码了.  
>   
> 在 00435033 处进行 patch,先给\[ebp-A60\]置 0  
> /\*435033\*/  add     dword ptr \[ebp-A60\], 1  //\[ebp-A60\]指向当前解码快,置 0 从第一块解码  
> /\*43503A\*/  mov     dword ptr \[474418\], 1   //\[474418\]指向父进程再次加密的解码快临界值,  
> 置 1 为了欺骗父进程永不进行加密.  
> /\*435044\*/  jmp     00434F68  
>   
> bp WriteProcessMemory,下断点,shift+f9,断下后,  
>     
> 0013DD18   00437C40  /CALL 到 WriteProcessMemory 来自 notepad.00437C3A  
> 0013DD1C   00000078  |hProcess = 00000078 (window)  
> 0013DD20   00401000  |Address = 401000  
> 0013DD24   00BA5EF8  |Buffer = 00BA5EF8  
> 0013DD28   00001000  |BytesToWrite = 1000 (4096.)  
> 0013DD2C   0013DD64  \\pBytesWritten = 0013DD64  
>   
> 从堆栈可以看到开始给子进程 OEP 处解码了,d 00ba5ef5+(004010cc-00401000),修改该处为 EB,FE,  
> 让子进程进入循环.bc WriteProcessMemory,bc 434F68,bp 0043521A,shift+f9,子进程全部解  
> 码,dump 子进程,修复 IAT,保存,将 OEP 改回原来的 55,8B.  
>   
>   (二)方法 2  
>     
> 方法 2 是国外教程里的经典方法,找到那个加密 call,nop 掉,patch,让父进程强行解码.方法一和  
> 方法二比较了一下,发现实质是一样的,一个在外围,一个在内部.那个加密 call 也很容易找到,"查找  
> ">"所有常数">FFFFFFF8 双击该行"004371CF  or edx, FFFFFFF8",向下找到  
> /\*43721B\*/  call    004372A0  //这就是加密的 CALL  
> 后面的过程就留给你了,子进程的 IAT 处理是一样的.  
>   
>   二、工具辅助篇  
>     
>   手工脱壳尽管复杂些，但可以加深理解，理解之后我们可以利用工具加快脱壳的步伐。  
>     
>   (一)方法 1  
>     
>   用 Armadillo COPYMEMEII DUMP 的一个 LOADPE 小插件，这个小工具的确方便，老大们说的很清楚  
> 了。  
>   先用方法二定位加密 call,nop 掉，bp WriteProcessMemory，用 Syser 处理子进程 IAT 加密，中断  
> 到 OD 后，让子进程进入循环，用 LOADPE 小插件 dump,修复 IAT,保存,将 OEP 改回原来的 55,8B.  
>     
>   (二)方法 2  
>     
>   用 deroko/ARTeam 提供的 Armadillo CopyMem II dumper 工具可以非常简单的 dump。  
>   dumper2.EXE notepad.exe 运行，用 Syser 处理子进程 IAT 加密，成功后提示子进程的 PID，OEP，  
> 并且自动 dump 和恢复了 OEP 原始值，我们只需修复 IAT,保存即可。  
>   
>   (三)方法 3  
>     
>   用 AvAtAr//TEAM RESURREC 提供的 Armadillo Process Detach v1.3 Final，也非常简单的 dump。  
>   选择 CopyMem-II 选项,把目标拖进去，也是用 用 Syser 处理子进程 IAT 加密，成功后提示：  
>       
> Filename: notepad.EXE  
> Parent process iD:  \[00000464\]  
> Processing...  
> \[PROTECTiON SYSTEM\]  
> Professional Edition  
> \[PROTECTiON OPTiONS\]  
> Debug-Blocker protection detected  
> CopyMem-II protection detected  
> Memory-Patching Protections enabled  
> \[CHiLD iNFO\]  
> Crypto call found:  \[0043721B\]  
> Child process iD:   \[000007F0\]  
> Entry point:        \[004010CC\]  
> Original bytes:     \[558BEC83\]  
> Detached successfully :)  
>   
>   此时用 OD 附加，f9,f12,恢复 OEP 原始数据，dump,修复 IAT,保存。  
>     
>   总算写完了，断断续续用了一天时间，希望您看完之后有些收获，我就知足了，最后再次感谢  
> 看雪和一衰烟雨论坛，感谢大家对我的支持和帮助。  
>   
>   
>                                        daxia2002\[DFCG\]于 2007-10-5 

 4dd466d0e1e0cc123afbe504da2462ca