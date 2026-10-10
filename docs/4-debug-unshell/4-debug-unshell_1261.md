

# 还原甲壳 2 之宿主\[卡得要死\]

> 原文：[`www.pediy.com/kssd/pediy11/113226.html`](https://www.pediy.com/kssd/pediy11/113226.html)

00401500 >/$  55            push ebp                             //程序载入首先停在这里  
00401501  |.  8BEC          mov ebp,esp  
00401503  |.  53            push ebx  
00401504  |.  56            push esi  
00401505  |.  57            push edi  
00401506  |.  BF 04304000   mov edi,GUI-宿主.00403004  
0040150B  |.  6A 00         push 0x0                                 ; /pModule = NULL  
  
  
004015A0  |.  FF35 24304000 push dword ptr ds:\[0x403024\]             ;  GUI-宿主.00403004  
004015A6  |.  FF15 20304000 call dword ptr ds:\[0x403020\]         //这个 CALL 我们 F7 跟进去看  
004015AC  |.  5F            pop edi                              //停在上面 CALL 时 00403020=00D31020  
004015AD  |.  5E            pop esi                              //我们跟进去还原作者所谓的模拟  
004015AE  |.  B8 01000000   mov eax,0x1  
004015B3  |.  5B            pop ebx  
004015B4  |.  5D            pop ebp  
004015B5  \\.  C2 1000       retn 0x10                            //因为这里执行进去后直接退出  
  
  
00D31020    55              push ebp                             //进入后发现超卡  
00D31021    8BEC            mov ebp,esp                          //由于代码比较多我也就不一句一句复制啦，因为太卡  
00D31023    83EC 60         sub esp,0x60  
00D31026    53              push ebx  
00D31027    56              push esi  
00D31028    57              push edi  
00D31029    8B45 08         mov eax,dword ptr ss:\[ebp+0x8\]  
00D3102C    8B48 08         mov ecx,dword ptr ds:\[eax+0x8\]  
00D3102F    894D F4         mov dword ptr ss:\[ebp-0xC\],ecx  
  
  
00D31103    51              push ecx  
00D31104    8B55 0C         mov edx,dword ptr ss:\[ebp+0xC\]  
00D31107    52              push edx  
00D31108    E8 C32C0000     call 00D33DD0                        //到达这里时我们直接 F7 跟进  
00D3110D    5F              pop edi  
00D3110E    5E              pop esi  
00D3110F    5B              pop ebx  
00D31110    8BE5            mov esp,ebp  
00D31112    5D              pop ebp  
00D31113    C3              retn                                 //同样老规矩因为这里就是返回  
  
  
  
00D33DD0    55              push ebp                            //F7 跟进后我们来到这里  
00D33DD1    8BEC            mov ebp,esp                           
00D33DD3    83EC 50         sub esp,0x50  
00D33DD6    53              push ebx  
00D33DD7    56              push esi  
00D33DD8    57              push edi  
  
00D33E2E    8B45 FC         mov eax,dword ptr ss:\[ebp-0x4\]           ; GUI-宿主.00401000  
00D33E31    A3 DC62D300     mov dword ptr ds:\[0xD362DC\],eax       //注意上面已经把 00401000 传送给了 EAX  
00D33E36    8B65 08         mov esp,dword ptr ss:\[ebp+0x8\]        //而上面又将 EAX 传给了 00D362DC 而下面刚好跳去 00D362DC  
00D33E39    61              popad  
00D33E3A    FF25 DC62D300   jmp dword ptr ds:\[0xD362DC\]           //进入这里后就可以看到开始模拟啦  
00D33E40    5F              pop edi                               //停在上面时可以  
00D33E41    5E              pop esi  
  
  
00401000  /$- FF25 20004000 jmp dword ptr ds:\[0x400020\]           //可以发现 00401000 已经发生了变化啦  
00401006  |?  0000          add byte ptr ds:\[eax\],al  
00401008  |.  0000          add byte ptr ds:\[eax\],al              //这里全部都是空代码  
0040100A  |.  0000          add byte ptr ds:\[eax\],al              //可以肯定开始模拟这段的代码啦  
0040100C  |.  0000          add byte ptr ds:\[eax\],al  
0040100E  |?  0000          add byte ptr ds:\[eax\],al  
00401010  |.  0000          add byte ptr ds:\[eax\],al  
  
  
10D40000    9C              pushfd                                //又进入了一段内存空间  
10D40001    60              pushad  
10D40002    8925 9060D300   mov dword ptr ds:\[0xD36090\],esp  
10D40008    8B25 C062D300   mov esp,dword ptr ds:\[0xD362C0\]  
10D4000E    8BEC            mov ebp,esp  
10D40010    B8 9460D300     mov eax,0xD36094  
10D40015    C700 00000000   mov dword ptr ds:\[eax\],0x0  
10D4001B    B8 CC62D300     mov eax,0xD362CC  
10D40020    C700 00000000   mov dword ptr ds:\[eax\],0x0  
10D40026    B8 1024D300     mov eax,0xD32410                      //这里为 EAX 赋值为 00D32410  
10D4002B    FFE0            jmp eax                               //准备进入解码环节，我个人叫卡机环节  
                     
  
00D32410    55              push ebp  
00D32411    8BEC            mov ebp,esp                           //进入这里后电脑进入半卡死状态  
00D32413    83EC 78         sub esp,0x78                          //由于有部分花指令我也就不复制那么多啦  
00D32416    53              push ebx                              //直接 F8 走我就直接给大家复制关键吧  
00D32417    56              push esi  
00D32418    57              push edi  
00D32419    A1 FC60D300     mov eax,dword ptr ds:\[0xD360FC\]  
  
  
00D32D27    3BC1            cmp eax,ecx  
00D32D29    75 05           jnz short 00D32D30                   //这里的 JNZ 没有跳，我们就别管他，一样按 F8 跟  
00D32D2B  ^ E9 4EFCFFFF     jmp 00D3297E                         //不要乱跳，因为太卡啦，从来几次不划算  
  
  
00D32993    3B05 5060D300   cmp eax,dword ptr ds:\[0xD36050\]          ; //这里比较还有几个字节需要修改  
00D32999    7D 12           jge short 00D329AD  
00D3299B    A1 C862D300     mov eax,dword ptr ds:\[0xD362C8\]          ; //这里读取申请来的临时空间  
00D329A0    0345 DC         add eax,dword ptr ss:\[ebp-0x24\]  
00D329A3    8A0D 0760D300   mov cl,byte ptr ds:\[0xD36007\]            ; //这里将需要修改的指令传送  
00D329A9    8808            mov byte ptr ds:\[eax\],cl                 ; //这里开始修改  
00D329AB  ^ EB DA           jmp short 00D32987  
  
  
00D329AD    A1 C862D300     mov eax,dword ptr ds:\[0xD362C8\]          ; //这里以下开始逐位修改临时空间的代码  
00D329B2    0305 5060D300   add eax,dword ptr ds:\[0xD36050\]  
00D329B8    8A0D 0760D300   mov cl,byte ptr ds:\[0xD36007\]            ; //这里传送当前地址临时空间的指令  
00D329BE    8808            mov byte ptr ds:\[eax\],cl                 ; //CL 每次传送一个字节的指令  
00D329C0    A1 C862D300     mov eax,dword ptr ds:\[0xD362C8\]  
00D329C5    0305 5460D300   add eax,dword ptr ds:\[0xD36054\]          ; //这里计算下一个需要修改的地址  
00D329CB    8A0D 0A60D300   mov cl,byte ptr ds:\[0xD3600A\]            ; //这里传送需要修改的指令  
00D329D1    8808            mov byte ptr ds:\[eax\],cl                 ; //这里开始修改  
  
  
  
  
00D32A7A    8925 C062D300   mov dword ptr ds:\[0xD362C0\],esp  
00D32A80    892D 0461D300   mov dword ptr ds:\[0xD36104\],ebp  
00D32A86    8B25 9060D300   mov esp,dword ptr ds:\[0xD36090\]  
00D32A8C    61              popad  
00D32A8D    9D              popfd  
00D32A8E  - FF25 C862D300   jmp dword ptr ds:\[0xD362C8\]           //这里跳向解码完毕的地方  
00D32A94   /EB 02           jmp short 00D32A98                    //停在上面一句时可以发现 00D362C8\]=10D30000  
00D32A96   |FF25 50EB02FF   jmp dword ptr ds:\[0xFF02EB50\]  
  
10D30000    B8 01000000     mov eax,0x1                           //这里就是我们被模拟后的第一句代码  
10D30005    90              nop  
10D30006    90              nop                                   //这里开始会循环在 10D30000 显示被模拟的代码  
10D30007    90              nop  
10D30008    90              nop  
10D30009  - FF25 1600D310   jmp dword ptr ds:\[0x10D30016\]         //这里跳走开始去模拟第二个指令  
10D3000F    FF25 00000000   jmp dword ptr ds:\[0\]                  //停在上面时 10D30016=00D32A94  
10D30015    00942A D3000000 add byte ptr ds:\[edx+ebp+0xD3\],dl     //也就是停在我们进入前的下面一句代码  
10D3001C    0000            add byte ptr ds:\[eax\],al              //具体请看下面的注释  
10D3001E    0000            add byte ptr ds:\[eax\],al  
10D30020    0000            add byte ptr ds:\[eax\],al  
注释：  
00D32A8E  - FF25 C862D300   jmp dword ptr ds:\[0xD362C8\]           //这里跳向第一句解码完毕的地方  
00D32A94   /EB 02           jmp short 00D32A98                    //这里即是第二句模拟的开始  
  
  
00D32A94   /EB 02            jmp short 00D32A98                    //第二句出来后在这里  
00D32A96   |FF25 50EB02FF    jmp dword ptr ds:\[0xFF02EB50\]         //因为太卡我只复制关键啦  
00D32A9C    25 8BC48948      and eax,0x4889C48B                    //具体有多卡到时候大家自己体验吧  
00D32AA1    F8               clc  
00D32AA2    EB 02            jmp short 00D32AA6  
  
  
  
00D32A86    8B25 9060D300    mov esp,dword ptr ds:\[0xD36090\]  
00D32A8C    61               popad  
00D32A8D    9D               popfd  
00D32A8E  - FF25 C862D300    jmp dword ptr ds:\[0xD362C8\]           //继续同一个地址进入解码  
00D32A94    EB 02            jmp short 00D32A98                    //继续 00D362C8\]=10D30000  
00D32A96    FF25 50EB02FF    jmp dword ptr ds:\[0xFF02EB50\]  
  
10D30000    83F8 01          cmp eax,0x1                           //可以发现我们同一个地址却有不同的语句出现了吧  
10D30003    90               nop                                   //就这样开始依次循环解码我们的第三句吧  
10D30004    90               nop                                   //上面是一句比较下面用猜也知道是判断句  
10D30005    90               nop  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]         //这里继续出去解码和上面一次步骤一样  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
  
  
  
00D32A94   /EB 02            jmp short 00D32A98                    //依然来到了这里开始解码  
00D32A96   |FF25 50EB02FF    jmp dword ptr ds:\[0xFF02EB50\]  
00D32A9C    25 8BC48948      and eax,0x4889C48B  
00D32AA1    F8               clc  
  
  
  
10D30000    6A 04            push 0x4                              //经过了半天解码之后我们继续来到这里  
10D30002    90               nop                                   //这次为我们压入的是 PUSH 4  
10D30003    90               nop  
10D30004    90               nop  
10D30005    90               nop  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
  
  
10D30000    68 00304000      push 0x403000                         //紧接着就是给我们压入 push 0x403000  
10D30005    90               nop                                   //继续 F8 走  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
10D30015    00942A D3000000  add byte ptr ds:\[edx+ebp+0xD3\],dl  
  
  
10D30000    68 00304000      push 0x403000                         //又一次为我们压入了 push 0x403000  
10D30005    90               nop                                   //继续 F8 走  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
10D30015    00942A D3000000  add byte ptr ds:\[edx+ebp+0xD3\],dl  
  
10D30000    6A 00            push 0x0                               //这次为我们压入的是 PUSH 0  
10D30002    90               nop                                    //继续 F8 走  
10D30003    90               nop  
10D30004    90               nop  
10D30005    90               nop  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
10D30015    00942A D3000000  add byte ptr ds:\[edx+ebp+0xD3\],dl  
  
  
00D32913    8B08             mov ecx,dword ptr ds:\[eax\]               ; USER32.MessageBoxA  
00D32915    894D E0          mov dword ptr ss:\[ebp-0x20\],ecx         //程序在上面一句时显示我们的程序开始调用对话框  
00D32918    8D45 FC          lea eax,dword ptr ss:\[ebp-0x4\]          //ds:\[00402008\]=77D50702 (USER32.MessageBoxA)  
00D3291B    50               push eax                                //寄存器和信息窗口都显示我们的程序调用了对话框的地址  
00D3291C    8D4D F8          lea ecx,dword ptr ss:\[ebp-0x8\]          //那么我们继续向下走  
  
  
00D32952    9D               popfd  
00D32953    A1 0061D300      mov eax,dword ptr ds:\[0xD36100\]  
00D32958  - FFE0             jmp eax                                  ; USER32.MessageBoxA  
00D3295A    EB 22            jmp short 00D3297E                       //上面的 JMP EAX 跳向我们的对话框函数  
00D3295C    8B45 FC          mov eax,dword ptr ss:\[ebp-0x4\]           //我们继续走  
   
  
  
10D30000    C3               retn                                     //这时我们程序给我们返回了 RETN  
10D30001    90               nop                                      //但是我们在按 F9 运行一次又给我们返回了 PUSH 4  
10D30002    90               nop                                      //不可能是无限循环的吧  
10D30003    90               nop                                      //经过三次之后对话框选确定，程序为我们返回了 PUSH 0  
10D30004    90               nop                                      //从而我们可以判断得出我们的程序上面是三句调用  
10D30005    90               nop                                      //既然是调用那就不可能是判断句，因为可以实验的  
10D30006    90               nop                                      //具体可以看下面我的注释   
10D30007    90               nop   
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
注释：  
望我们运行程序起来会提示三次对话框，我们都点确定  
而我们进来时程序给了一句 MOV EAX,1 的语句，紧跟着的就是 cmp eax,0x1  
既然那里是一句比较是否为 1 那么不管我们是用任何的判断跳转的语句皆是不可能的  
此时更不可能是 JMP 因为 JMP 是无条件跳转，是不可能出现对话框的  
唯一的调用语句就是 CALL 才可以眺用，那就因为是直接 CALL 我们的第一句也就是在判断句  
的下面一句 PUSH 4，因为下面两句的 PUSH 00403000 全部是用来调用字符串的  
具体的可以在调用到对话框时留意堆栈窗口，我们继续走起程序来  
  
10D30000    6A 00            push 0x0                                //程序在第三次弹出对话框返回之后  
10D30002    90               nop                                     //给我们来了个不一样的地址  
10D30003    90               nop                                     //因为前三次由 RETN 返回后弹出的是 PUSH 4  
10D30004    90               nop                                     //我们继续 F8 跟上吧  
10D30005    90               nop  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
  
  
00D3290D    8945 E4         mov dword ptr ss:\[ebp-0x1C\],eax          //当我们在一次运行程序直接调用退出啦  
00D32910    8B45 E4         mov eax,dword ptr ss:\[ebp-0x1C\]          ; <&KERNEL32.ExitProcess>  
00D32913    8B08            mov ecx,dword ptr ds:\[eax\]               //这里开始显示调用退出  
00D32915    894D E0         mov dword ptr ss:\[ebp-0x20\],ecx          //并且注意 EAX 里的地址是 00402000  
00D32918    8D45 FC         lea eax,dword ptr ss:\[ebp-0x4\]           ss:\[003F0F80\]=00402000 (<&KERNEL32.ExitProcess>  
  
  
00D3293A    A3 0061D300     mov dword ptr ds:\[0xD36100\],eax  
00D3293F    892D 0461D300   mov dword ptr ds:\[0xD36104\],ebp  
00D32945    8925 C062D300   mov dword ptr ds:\[0xD362C0\],esp  
00D3294B    8B25 9060D300   mov esp,dword ptr ds:\[0xD36090\]  
00D32951    61              popad  
00D32952    9D              popfd  
00D32953    A1 0061D300     mov eax,dword ptr ds:\[0xD36100\]           //现在我们来整理一下我们所找回来的代码  
00D32958    FFE0            jmp eax                                   //程序到这里执行退出  
  
注释：  
  
10D30000    B8 01000000     mov eax,0x1   
10D30000    83F8 01          cmp eax,0x1    
10D30000    6A 04            push 0x4   
10D30000    68 00304000      push 0x403000   
10D30000    68 00304000      push 0x403000  
这里是显示对话框啦，而我们的对话框就是 00402800，所以这里应该是 CALL 00402800  
10D30000    6A 00            push 0x0  
10D30000    C3               retn          //这个 RETN 返回调用了 3 次对话框函数  
10D30000    6A 00            push 0x0    
这里是显示退出啦。而我们的推出的函数是由 00402000 调用，所以这里应该是 CALL 00402000  
  
得到如下代码之后我们在进行详细的整理一翻来完成我们这次的卡机之旅吧  
  
10D30000    B8 01000000     mov eax,0x1      //第一句，上面说过这里给 EAX 赋值为 1  
10D30000    83F8 01          cmp eax,0x1     //第二句，这里比较 EAX 是否为 1  
而我们的第三句应该是一句 JE 或者 JNZ 的判断句，好我们继续从信载入程序，我们看看当我们让 EAX 为 0 时程序会有  
什么不一样的操作以继续我们下一步的卡机之旅，我们在 10D30000 下硬件执行断点，从载程序  
  
00401500 >/$  55             push ebp                  //程序从新载入  
00401501  |.  8BEC           mov ebp,esp               //好吧，我们什么都不用管，我们直接把程序运行  
00401503  |.  53             push ebx  
00401504  |.  56             push esi  
00401505  |.  57             push edi  
00401506  |.  BF 04304000    mov edi,GUI-宿主.00403004  
0040150B  |.  6A 00          push 0x0                                 ; /pModule = NULL  
0040150D  |.  893D 24304000  mov dword ptr ds:\[0x403024\],edi          ; |  
00401513  |.  FF15 18204000  call dword ptr ds:\[<&KERNEL32.GetModuleH>; \\GetModuleHandleW  
  
10D30000    83F8 01          cmp eax,0x1               //程序运行两次之后停在这里  
10D30003    90               nop                       //此时我们注意寄存器窗口 EAX 的值是为 1  
10D30004    90               nop                       //我们把 EAX 的值修改为 0 看看  
10D30005    90               nop                       //修改为 0 之后我们开始 F8 跟吧  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
10D30015    00942A D3000000  add byte ptr ds:\[edx+ebp+0xD3\],dl  
  
10D30000    6A 00            push 0x0                   //程序来到了一句 PUSH 0  
10D30002    90               nop                        //我们继续 F8 跟上吧  
10D30003    90               nop  
10D30004    90               nop  
10D30005    90               nop  
10D30006    90               nop  
10D30007    90               nop  
10D30008    90               nop  
10D30009  - FF25 1600D310    jmp dword ptr ds:\[0x10D30016\]  
10D3000F    FF25 00000000    jmp dword ptr ds:\[0\]  
10D30015    00942A D3000000  add byte ptr ds:\[edx+ebp+0xD3\],dl  
  
00D3290D    8945 E4          mov dword ptr ss:\[ebp-0x1C\],eax          ; <&KERNEL32.ExitProcess>  
00D32910    8B45 E4          mov eax,dword ptr ss:\[ebp-0x1C\]        //程序直接调用了退出  
00D32913    8B08             mov ecx,dword ptr ds:\[eax\]             //看来如我们所猜的一样程序果然是判断 EAX 是否为 1  
00D32915    894D E0          mov dword ptr ss:\[ebp-0x20\],ecx        //那么我们的代码到此就算全部完结啦  
00D32918    8D45 FC          lea eax,dword ptr ss:\[ebp-0x4\]         //具体还原代码部分请看注释  
00D3291B    50               push eax  
00D3291C    8D4D F8          lea ecx,dword ptr ss:\[ebp-0x8\]  
  
注释：  
首先我们得知我们的程序在那里还有一句判断句就是判断程序是否为 1  
那么也就是说我们的那段判断句是 JNZ，\[JNZ 是大小于小于则跳\]  
而 JNZ 就是直接跳转到 PUSH 0 而  
我们程序在弹出三次对话框之后也是直接由 RETN 返回到 PUSH 0 紧跟随的就是退出  
下面来开始还原代码真像，OD 从新载入程序在 00401000 处开始汇编代码，当我们  
汇编到对话框代码时发现，我们的对话框函数地址居然是 0040200C，于是有了以下  
感想，大哥我是不是最近熬夜太多啦，眼花啊，从新试一次吧，呵呵  
发现程序在 00401000 处出现了我们的 API 地址，这下可算是正确啦  
  
00401000  /$- FF25 20004000 jmp dword ptr ds:\[0x400020\]         //这个才是我们真实的入口点  
00401006  |?  0000          add byte ptr ds:\[eax\],al            //我们注意一下我们的数据窗口  
00401008  |.  0000          add byte ptr ds:\[eax\],al            //留意下我是不是眼花啦，看错啦  
0040100A  |.  0000          add byte ptr ds:\[eax\],al  
0040100C  |.  0000          add byte ptr ds:\[eax\],al  
0040100E  |?  0000          add byte ptr ds:\[eax\],al  
  
00402000 >7C81CAC2  kernel32.ExitProcess               //终于是出现啦，事实证明我没有眼花  
00402004 >00000000                             //好啦，继续汇编我们的代码  
00402008 >77D50702  USER32.MessageBoxA  
0040200C >00000000  
00402010 >00002054  
00402014 >00000000  
00402018 >00000000  
  
00401000    B8 01000000     mov eax,0x1                   //当代码汇编到这里时应该完结了吧  
00401005    83F8 01         cmp eax,0x1                   //可是一看对话框调用错了 API，可是这里又没有 API 可调  
00401008    75 25           jnz short GUI-宿主.0040102F   //因为数据窗口的确是显示的 MessageBoxA  
0040100A    E8 0C000000     call GUI-宿主.0040101B        //呵呵，于是开始下面的继续  
0040100F    E8 07000000     call GUI-宿主.0040101B  
00401014    E8 02000000     call GUI-宿主.0040101B  
00401019    EB 14           jmp short GUI-宿主.0040102F  
0040101B    6A 04           push 0x4  
0040101D    68 00304000     push GUI-宿主.00403000  
00401022    68 00304000     push GUI-宿主.00403000  
00401027    6A 00           push 0x0  
00401029    E8 DA0F0000     call <&KERNEL32.VirtualAlloc>       
0040102E    C3              retn  
0040102F    6A 00           push 0x0  
00401031    E8 CA0F0000     call <&KERNEL32.ExitProcess>  
  
终于把代码全部还原啦，我们还原好的代码如下  
00401000      B8 01000000   mov eax,0x1  
00401005      83F8 01       cmp eax,0x1  
00401008      75 25         jnz short GUI-宿主.0040102F  
0040100A      E8 0C000000   call GUI-宿主.0040101B  
0040100F      E8 07000000   call GUI-宿主.0040101B  
00401014      E8 02000000   call GUI-宿主.0040101B  
00401019      EB 14         jmp short GUI-宿主.0040102F  
0040101B      6A 04         push 0x4  
0040101D      68 00304000   push GUI-宿主.00403000  
00401022      68 00304000   |push GUI-宿主.00403000  
00401027      6A 00         push 0x0  
00401029      E8 08000000   |call <jmp.&KERNEL32.VirtualAlloc>  
0040102E      C3            retn  
0040102F      6A 00         |push 0x0  
00401031      E8 06000000   call <jmp.&KERNEL32.ExitProcess>  
00401036    - FF25 08204000 jmp dword ptr ds:\<&KERNEL32.VirtualAllo>;  USER32.MessageBoxA  
0040103C    - FF25 00204000 jmp dword ptr ds:\[<&KERNEL32.ExitProcess>;  kernel32.ExitProcess  
00401042      C2 F800       retn 0xF8  
  
至于为什么要定义这两个 API 在这里我想大家都应该知道的，简单的解释就是我们的程序到了这里才是真实的入口点  
前面的只不过是被包裹了的一层壳而已，所以我们在这里要把输入表修正一下，准备脱壳完成卡机之旅啦  
运行我们的 LORDPE 脱壳吧，在运行我们的 ImportREC 修复下输入表，检查下程序一切正常，清理下垃圾代码

上传的附件
| ![ || GUI-宿主.rar   || ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || UnPacked.rar  
 |