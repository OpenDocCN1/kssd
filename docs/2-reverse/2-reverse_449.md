

# W32Dasm8.93 捉虫记

> 原文：[`www.pediy.com/kssd/pediy03/forum155.htm`](https://www.pediy.com/kssd/pediy03/forum155.htm)

 W32DASM8.93 捉虫记    湖北 wuhuashang   
    W32DASM 是调试修改程序的强大工具，作为 SOFTICE 的辅助工具 W32DASM 为 CRACKER 必备。   
    当前许多软件采用注册方式，输入注册号,若正确显示"THANK YOU",否则显示"INCORRECT PASSWORD",用 W32DASM 反编译该软件,然后查找指定的错误信息如"INCORRECT PASSWORD"或其他因没有注册而显示的某些提示信息,假若被找到,对熟练的 CRACKER 而言,可能 10 分钟后便完成破解,而对于没有破解经验的计算机爱好者,只要他有一点汇编语言知识,再看一些别人用 W32DASM 破解的文章,用不了几天,他便可以破解 INTERNET 网上的许多共享软件;但是如果显示的信息并不是英文而是中文如"注册号不正确",您试着也找......,您发现只要是中文信息,W32DASM 是没法对中文反编译的!   
  W32DASM 不支持中文日文韩文的反编译,把它看作 BUG 应该可以被修改,磨刀不误砍柴工!.   
  我写此文章不是教人破解软件的加密信息,而是写出修改此软件的 BUG 的过程,假设反编译的中文软件 DEMO.EXE,输入错误注册号后显示"无效的注册号".   
1.您可以用 HEXEDIT 修改为"ERRORPASS"(若被未知的软件压缩加密呢?),修改后最好注销或重新启动计算机(内存有 ERRORPASS 字串)   
2.启动 W32DASM,打开 DEMO.EXE,在 32DASM 反编译约 1-3 秒钟后 CTRL+D 切入 SOFTICE   
  S DS:0 L FFFFFFFF "ERRORPA"  (不要输入为 ERRORPASS,就是不要完整啦)   
  若没有找到退出 SOFTICE,重新打开 DEMO.EXE,大约 3 秒钟后 CTRL+D 切入 SOFTICE(依次类推),经过一翻斗争   
  在 167:00610175 处发现有"ERRORPASS"字串.   
3.设置断点 BPM 167:610175,退出 SOFTICE,W32DASM 继续编译,马上被 SOFTICE 拦截,分析:W32DASM 能反编译的字  符串的 ASCII 值在 20H-7F 之间,必然有 CMP AX,20 类似的比较,上下观察并没有该语句,退出 SOFTICE,马上又被  SOFTICE 拦截下来,停止在 CS:417241 处:   
  
.0041721A: 8B8530FFFFFF                mov      eax,\[ebp\]\[0FFFFFF30\]          
.00417220: 85C0                        test      eax,eax                        
.00417222: 0F848B000000                je      .0004172B3      
.00417228: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.0041722E: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.00417234: 8D040A                      lea      eax,\[edx\]\[ecx\]                
.00417237: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.0041723D: 0FBE0C10                    movsx    ecx,b,\[eax\]\[edx\]    
  
说明:D DS:EAX+EDX  好家伙!原来正在读入"ERRORPASS",可以猜想,W32DASM 能识别的字符串的 ASCII 码为 20-7D 之间而中文的 ASCII 码>A0 (是负数),改 CS:417244 75 CC ==>90 90 ,中文信息不就可以处理吗?哈哈,别高兴太早,会失败得惨!记录下 CS:417241 83F9207C5C   
.00417241: 83F920                      cmp      ecx,020 ;" "                  
.00417244: 7C5C                        jl      .0004172A2  -------- (1)      
.00417246: 8B83A5726000                mov      eax,\[ebx\]\[0006072A5\]          
.0041724C: 8B93081F6F00                mov      edx,\[ebx\]\[0006F1F08\]          
.00417252: 8D0C10                      lea      ecx,\[eax\]\[edx\]                
.00417255: 8B8530FFFFFF                mov      eax,\[ebp\]\[0FFFFFF30\]          
.0041725B: 0FBE1401                    movsx    edx,b,\[ecx\]\[eax\]              
.0041725F: 83FA7D                      cmp      edx,07D ;"}"                  
.00417262: 7F3E                        jg      .0004172A2      
  
.00417264: 8B8BA5726000                mov      ecx,\[ebx\]\[0006072A5\]          
.0041726A: 8B83081F6F00                mov      eax,\[ebx\]\[0006F1F08\]          
.00417270: 8D1401                      lea      edx,\[ecx\]\[eax\]                
.00417273: 8B8D30FFFFFF                mov      ecx,\[ebp\]\[0FFFFFF30\]          
.00417279: 0FBE440AFF                  movsx    eax,b,\[edx\]\[ecx\]\[-0001\]        
.0041727E: 83F820                      cmp      eax,020 ;" "    
说明:有敏感的比较,共有多少处要修改呢?您不是设置有断点吗?退出 SOFTICE,必然又拦截下来,用笔记录拦截的 CS:IP.   
.00417281: 7C23                        jl      .0004172A6  -------- (2)      
.00417283: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.00417289: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.0041728F: 8D040A                      lea      eax,\[edx\]\[ecx\]                
.00417292: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.00417298: 0FBE4C10FF                  movsx    ecx,b,\[eax\]\[edx\]\[-0001\]        
.0041729D: 83F97D                      cmp      ecx,07D ;"}"                  
.004172A0: 7F04                        jg      .0004172A6      
  
.004172A2: 33C0                        xor      eax,eax                        
.004172A4: EB05                        jmps    .0004172AB    
.004172A6: B801000000                  mov      eax,000000001 ;"    "          
.004172AB: 898574FFFFFF                mov      \[ebp\]\[0FFFFFF74\],eax          
.004172B1: EB4B                        jmps    .0004172FE    
.004172B3: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.004172B9: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.004172BF: 8D040A                      lea      eax,\[edx\]\[ecx\]                
.004172C2: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.004172C8: 0FBE0C10                    movsx    ecx,b,\[eax\]\[edx\]              
.004172CC: 83F920                      cmp      ecx,020 ;" "                  
.004172CF: 7C1E                        jl      .0004172EF  -------- (3)      
.004172D1: 8B83A5726000                mov      eax,\[ebx\]\[0006072A5\]          
.004172D7: 8B93081F6F00                mov      edx,\[ebx\]\[0006F1F08\]          
.004172DD: 8D0C10                      lea      ecx,\[eax\]\[edx\]                
......   
00417392: 0FBE0431                    movsx    eax,b,\[ecx\]\[esi\]              
.00417396: 83F820                      cmp      eax,020 ;" "                  
.00417399: 7C25                        jl      .0004173C0  -------- (4)      
.0041739B: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.004173A1: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.004173A7: 8D040A                      lea      eax,\[edx\]\[ecx\]                
.004173AA: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.004173B0: 8D0C10                      lea      ecx,\[eax\]\[edx\]                
.004173B3: 0FBE0431                    movsx    eax,b,\[ecx\]\[esi\]              
.004173B7: 83F87D                      cmp      eax,07D ;"}"                  
.004173BA: 0F8E68FFFFFF                jle      .000417328      
.004173C0: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.004173C6: 85D2                        test      edx,edx                        
.004173C8: 0F8483000000                je      .000417451    
.004173CE: 8B8BA5726000                mov      ecx,\[ebx\]\[0006072A5\]          
.004173D4: 8B83081F6F00                mov      eax,\[ebx\]\[0006F1F08\]          
.004173DA: 8D1401                      lea      edx,\[ecx\]\[eax\]                
.004173DD: 8B8D30FFFFFF                mov      ecx,\[ebp\]\[0FFFFFF30\]          
.004173E3: 66833C0A20                  cmp      w,\[edx\]\[ecx\],020 ;" "          
此处的比较有点奇怪,为啥是 CMP WORD PTR \[EDX+ECX\],20 而不是 CMP BYTE PTR\[EDX+ECX\],20 呢?而且有好几个地方.   
.004173E8: 7256                        jb      .000417440  -------- (5)      
.004173EA: 8B83A5726000                mov      eax,\[ebx\]\[0006072A5\]          
.004173F0: 8B93081F6F00                mov      edx,\[ebx\]\[0006F1F08\]          
.004173F6: 8D0C10                      lea      ecx,\[eax\]\[edx\]                
  
......   
.004173F9: 8B8530FFFFFF                mov      eax,\[ebp\]\[0FFFFFF30\]          
.004173FF: 66833C017D                  cmp      w,\[ecx\]\[eax\],07D ;"}"          
.00417404: 773A                        ja      .000417440      
.00417406: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.0041740C: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.00417412: 8D040A                      lea      eax,\[edx\]\[ecx\]                
.00417415: 8B9530FFFFFF                mov      edx,\[ebp\]\[0FFFFFF30\]          
.0041741B: 66837C10FF20                cmp      w,\[eax\]\[edx\]\[-0001\],020 ;" "   
多好的比较方式,用此方法可以快速解决某些西文软件不能输入汉字的问题.不是吗?   
  
.00417421: 7221                        jb      .000417444  -------- (6)      
.00417423: 8B8BA5726000                mov      ecx,\[ebx\]\[0006072A5\]          
.00417429: 8B83081F6F00                mov      eax,\[ebx\]\[0006F1F08\]          
.0041742F: 8D1401                      lea      edx,\[ecx\]\[eax\]                
.00417432: 8B8D30FFFFFF                mov      ecx,\[ebp\]\[0FFFFFF30\]          
.00417438: 66837C0AFF7D                cmp      w,\[edx\]\[ecx\]\[-0001\],07D ;"}"   
.0041743E: 7704                        ja      .000417444          
.00417440: 33C0                        xor      eax,eax                        
.00417442: EB05                        jmps    .000417449    
.00417444: B801000000                  mov      eax,000000001 ;"    "          
.00417449: 898570FFFFFF                mov      \[ebp\]\[0FFFFFF70\],eax          
.0041744F: EB47                        jmps    .000417498    
.00417451: 8B93A5726000                mov      edx,\[ebx\]\[0006072A5\]          
.00417457: 8B8B081F6F00                mov      ecx,\[ebx\]\[0006F1F08\]          
.0041745D: 8D040A                      lea      eax,\[edx\]\[ecx\]            
  
......   
00460556: 8A0418                        mov      al,\[eax\]\[ebx\]                  
.00460559: 3C20                        cmp      al,020 ;" "                    
.0046055B: 7204                        jb      .000460561  -------- (7)      
.0046055D: 3C7D                        cmp      al,07D ;"}"                    
.0046055F: 7613                        jbe      .000460574      
.00460561: 68DE3D4C00                  push      0004C3DDE ;" L=?              
.00460566: 8D95C0FEFFFF                lea      edx,\[ebp\]\[0FFFFFEC0\]          
.0046056C: 52                          push      edx                            
.0046056D: E8E8E60400                  call    .0004AEC5A    
.00460572: EB2D                        jmps    .0004605A1      
.00460574: 6A01                        push      001                
我试所有的类似 CMP xxxx,20          
    JB/JL  xxxx    (该行用 90 90 替换)   
经验证发现许多应该显示的字符串也没有显示,惨惨惨!问题出在比较的方式不一样.您可以用 HEXEDIT 打开一个 EXE 文件,所有被显示字符串末尾有一个字节 00H,当读取字符串完毕后读入字符 00H,此时 JB/JL 必然成立,必须正常跳转,不能被改成 NOP,所以改思路:   
    CMP    WORD PTR \[XXXX\],20   
    JB/JL  XXXX  ==>90 90   
    CMP    BYTE PTR\[XXXX\],20  { 对照(3)(4) (7)处} ===>20 改 00   
    JB/JL  XXXX                                  ==>JB 改 JE   
完整记录下要修改处的 CS:IP 或 16 进制字串,用 UNASPACK 脱掉 W32DASM 的外壳,再用 HEXEDIT(必须记录被修改处的 16 进制字串)或 HACKVIEW 修改,最后您去测试,中文信息是不是全显示出来了?   
  
  版权所有，请不要修改原内容   4dd466d0e1e0cc123afbe504da2462ca