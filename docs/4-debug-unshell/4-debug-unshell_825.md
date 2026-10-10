

# 某人最新 0.7x 完整分析（一）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-683.htm`](https://www.pediy.com/kssd/pediy07/pediy7-683.htm)

【破解作者】 hnhuqiong  
【作者邮箱】 3KKK@  
【作者主页】 3KKK  
【使用工具】 FLYODBG  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 XXXXXXXXXXXXX  
【软件简介】 我们看的不是软件，是它的壳  
【加壳方式】 XXXX 0.7X  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
004670D1 >  60                 pushad                              ; 从这里到 0ed 是典型的伪造头  
004670D2    E8 00000000        call    jl.004670D7                 ;     假 CALL，  
004670D7    5D                 pop     ebp                         ;  
004670D8    81ED D7000000      sub     ebp,0D7                     ; 取段地址  
004670DE    8DB5 EE000000      lea     esi,dword ptr ss:\[ebp+EE\]   ;   
004670E4    55                 push    ebp                         ; 段地址入栈(467000)  
004670E5    56                 push    esi                         ; 将段地址＋EE 地址入栈，为后面   
                                                                     放解码数据，我这里是 4670EE         
004670E6    81C5 89110000      add     ebp,1189  
004670EC    55                 push    ebp                         ; 返回地址入栈（468199)，也就是  
                                                                     跳这个 CALL 其实就是一个 jmp  
004670ED    C3                 retn  
  
此壳大量的运用了假 call，当然对于大侠级别的是小菜，但是通篇用这样的  
call，如果不运用花指令予以打掉，简直是自虐。这个壳运用花指令对于  
ANTI－DEBUG 可以说是到了极限，没有一处是可以清净的进行分析。  
  
00468189    25 6FA78540        and     eax,4085A76F                    
0046818E    8D3C4B             lea     edi,dword ptr ds:\[ebx+ecx\*2>; 垃圾数据，也可以  
                                                             说这个是垃圾代码迷惑人的  
00468191    8D92 C57FEB02      lea     edx,dword ptr ds:\[edx+2EB7F>; 垃圾数据  
00468197    8D8E BD5D714B      lea     ecx,dword ptr ds:\[esi+4B715>; 垃圾数据  
0046819D    4F                 dec     edi                         ; 垃圾数据  
0046819E    8D340A             lea     esi,dword ptr ds:\[edx+ecx\]  ;   
004681A1    BD F9227641        mov     ebp,417622F9                ; 将常数 417622f9 给 EBP  
004681A6    8B1C24             mov     ebx,dword ptr ss:\[esp\]      ; 4670ee  
004681A9    8D92 A3703444      lea     edx,dword ptr ds:\[edx+44347>; 垃圾数据  
004681AF    BF DF542B32        mov     edi,322B54DF                ; 将常数 322b54df 给 DI   
004681B4    8D8E D1559117      lea     ecx,dword ptr ds:\[esi+17915>; 垃圾数据  
004681BA    812B 2EB0826E      sub     dword ptr ds:\[ebx\],6E82B02E ; 计算的结果是  
                                                                    （458B5DEB)送 CS＋EE  
004681C0    8D85 05AC7078      lea     eax,dword ptr ss:\[ebp+7870A>垃圾数据  
004681C6    8D8E 9B64774D      lea     ecx,dword ptr ds:\[esi+4D776>垃圾数据  
004681CC    25 70542946        and     eax,46295470                ; 垃圾数据  
004681D1    4B                 dec     ebx                         ;  
004681D2    83EB FB            sub     ebx,-5                      ; ebx==4670f2  
004681D5    8D340A             lea     esi,dword ptr ds:\[edx+ecx\]  ;  
004681D8    45                 inc     ebp                         ; jl.00468189  
004681D9    25 EC6A7325        and     eax,25736AEC                ;  
004681DE    25 B21AFC7A        and     eax,7AFC1AB2                 垃圾数据  
004681E3    812B BB08217B      sub     dword ptr ds:\[ebx\],7B2108BB ; 计算的结果是(74C00B00)=>  
                                                                     送 CS ＋F2 处  
004681E9    81C5 C6D65E6F      add     ebp,6F5ED6C6                 ；垃圾数据  
004681EF    51                 push    ecx                         ；  
004681F0    59                 pop     ecx                         ; 来回推 ECX 入栈出栈，垃圾  
004681F1    B8 DC199E70        mov     eax,709E19DC                 ；垃圾数据  
004681F6    51                 push    ecx  
004681F7    59                 pop     ecx                         ; 来回推 ECX 入栈出栈，垃圾  
004681F8    8D3C4B             lea     edi,dword ptr ds:\[ebx+ecx\*2>  
004681FB    45                 inc     ebp                         ;   
004681FC    4B                 dec     ebx  
004681FD    83EB FB            sub     ebx,-5                      ; ebx=4670f6  
00468200    BF ECAE113B        mov     edi,3B11AEEC                ；垃圾数据  
00468205    8D340A             lea     esi,dword ptr ds:\[edx+ecx\]  ；垃圾数据  
00468208    8DA8 003C3F24      lea     ebp,dword ptr ds:\[eax+243F3>  
0046820E    8133 05602914      xor     dword ptr ds:\[ebx\],14296005 ; 计算的结果是（5DACD4FC）  
                                                                   ；送 CS ＋F2 处  
00468214    83EB FC            sub     ebx,-4                      ; ebx=4670fa  
00468217    B9 163CA035        mov     ecx,35A03C16                 ；垃圾代码  
0046821C    3BC1               cmp     eax,ecx                      ；垃圾代码  
0046821E    B9 4B53F626        mov     ecx,26F6534B                 ；垃圾代码  
00468223    B8 3A70A84D        mov     eax,4DA8703A                 ；垃圾代码  
00468228    8B03               mov     eax,dword ptr ds:\[ebx\]      ；  
0046822A    F7D0               not     eax                         ;   
0046822C    8903               mov     dword ptr ds:\[ebx\],eax      ; 将 45ff0c 送到 CS +FA  
0046822E    8DA8 E97B625B      lea     ebp,dword ptr ds:\[eax+5B627>；垃圾代码  
00468234    45                 inc     ebp                         ; 垃圾代码  
00468235    BA FA48F310        mov     edx,10F348FA                ；垃圾代码  
0046823A    81CA 3657683A      or      edx,3A685736                ；垃圾代码  
00468240    45                 inc     ebp                         ; 垃圾代码  
00468241    8D85 82F18C3B      lea     eax,dword ptr ss:\[ebp+3B8CF>；垃圾代码  
00468247    81D1 4C6B002B      adc     ecx,2B006B4C                ；垃圾代码   
0046824D    4B                 dec     ebx                         ；  
0046824E    83EB FB            sub     ebx,-5                      ; ebx＝＝4670fe  
00468251    81C5 940D9345      add     ebp,45930D94                ；垃圾代码  
00468257    8B03               mov     eax,dword ptr ds:\[ebx\]      ；   
00468259    F7D0               not     eax                            
0046825B    8903               mov     dword ptr ds:\[ebx\],eax      ; 计算的结果是  
                                                                    （2424488b)送 CS＋FE  
0046825D    4F                 dec     edi                         ; 垃圾代码  
0046825E    8D85 0DE82853      lea     eax,dword ptr ss:\[ebp+5328E>；垃圾代码  
00468264    B9 574BFF50        mov     ecx,50FF4B57                ；垃圾代码  
00468269    81C5 2A6F5147      add     ebp,47516F2A                ；垃圾代码  
0046826F    4B                 dec     ebx                         ；  
00468270    83EB FB            sub     ebx,-5                      ；  
00468273    25 214D0151        and     eax,51014D21                ; eax＝＝467102  
00468278    BA 89E6420C        mov     edx,0C42E689                ；垃圾代码  
0046827D    3BC1               cmp     eax,ecx                      ；垃圾代码  
0046827F    8133 9A1DD82A      xor     dword ptr ds:\[ebx\],2AD81D9A ; 计算的结果是  
                                                                    （8D044589)送 CS＋102                                                                      
00468285    8D85 33DF6172      lea     eax,dword ptr ss:\[ebp+7261D>  垃圾代码  
0046828B    81F7 78C19C1D      xor     edi,1D9CC178                  垃圾代码  
00468291    8D85 FB3FEA56      lea     eax,dword ptr ss:\[ebp+56EA3>  垃圾代码  
00468297    83EB FC            sub     ebx,-4                        ; ebx=CS+106  
0046829A    E8 02000000        call    jl.004682A1                   ; 虚跳 CALL，花指令  
  
004682A1    5E                 pop     esi                               ; 保护现场  
004682A2    81CE 4792BB22      or      esi,22BB9247  
004682A8    81C5 1BAB0657      add     ebp,5706AB1B                      ; 垃圾代码  
004682AE    8133 47154C49      xor     dword ptr ds:\[ebx\],494C1547       ; 计算的结果是（0455ff65）  
                                                                         ，送 CS+10A  
004682B4    8DA8 959D1C65      lea     ebp,dword ptr ds:\[eax+651C9D95\]   ; 垃圾代码  
004682BA    83EB FC            sub     ebx,-4                            ; CS+10A  
004682BD    8B03               mov     eax,dword ptr ds:\[ebx\]            ；   
004682BF    F7D0               not     eax                               ；  
004682C1    8903               mov     dword ptr ds:\[ebx\],eax            ; 计算的结果是（55ff5600）  
                                                                        ；送 CS+10A  
004682C3    8D340A             lea     esi,dword ptr ds:\[edx+ecx\]        ; 垃圾代码  
004682C6    BA 926B7F56        mov     edx,567F6B92                      ; 垃圾代码  
004682CB    BF 7B62BA5E        mov     edi,5EBA627B                      ; 垃圾代码  
004682D0    81C5 3C34D76D      add     ebp,6DD7343C                      ; 垃圾代码  
004682D6    B9 DEAFC704        mov     ecx,4C7AFDE                       ; 垃圾代码  
004682DB    4B                 dec     ebx  
004682DC    83EB FB            sub     ebx,-5                            ; CS+10E  
004682DF    3BC1               cmp     eax,ecx                           ; 垃圾代码  
004682E1    25 A981CB14        and     eax,14CB81A9                      ; 垃圾代码  
004682E6    812B 75F8C422      sub     dword ptr ds:\[ebx\],22C4F875       ; 计算的结果是（1c758d73）  
                                                                         ，送 CS+10E  
004682EC    8DA8 E3F5B334      lea     ebp,dword ptr ds:\[eax+34B3F5E3\]   ; 垃圾代码  
004682F2    BA B802F97D        mov     edx,7DF902B8                      ; 垃圾代码  
004682F7    B9 D7711D1D        mov     ecx,1D1D71D7                      ; 垃圾代码  
004682FC    4B                 dec     ebx  
004682FD    83EB FB            sub     ebx,-5                            ; CS+112  
00468300    8DA8 1A4E0E7B      lea     ebp,dword ptr ds:\[eax+7B0E4E1A\]   ; 垃圾代码  
00468306    B8 71037F33        mov     eax,337F0371                      ; 垃圾代码  
0046830B    3BC1               cmp     eax,ecx                           ; 垃圾代码  
0046830D    8D92 51A10971      lea     edx,dword ptr ds:\[edx+7109A151\]   ; 垃圾代码  
00468313    51                 push    ecx                               ; 垃圾代码  
00468314    59                 pop     ecx                               ; CS+EE，垃圾，清空前面假 CALL 入的堆栈  
00468315    81F7 40079F60      xor     edi,609F0740                      ; 垃圾代码  
0046831B    812B 9B1E105D      sub     dword ptr ds:\[ebx\],5D101E9B       ; 计算的结果是（55ff5056），送 CS+112  
00468321    BF AF102C2A        mov     edi,2A2C10AF                      ; 垃圾代码  
00468326    8D8E BDEC0965      lea     ecx,dword ptr ds:\[esi+6509ECBD\]   ; 垃圾代码  
0046832C    F7C2 A16EF328      test    edx,28F36EA1                      ; 垃圾代码  
00468332    B8 4A11E162        mov     eax,62E1114A                      ; 垃圾代码  
00468337    25 E2686047        and     eax,476068E2                      ; 垃圾代码  
0046833C    83EB FC            sub     ebx,-4                            ; CS+116  
0046833F    B9 E49E9030        mov     ecx,30909EE4                      ; 垃圾代码  
00468344    8103 DF92742D      add     dword ptr ds:\[ebx\],2D7492DF       ; 计算的结果是（2c458974），送 CS+116  
0046834A    43                 inc     ebx  
0046834B    43                 inc     ebx  
0046834C    43                 inc     ebx  
0046834D    43                 inc     ebx                               ; CS+11A  
0046834E    BF 58506F78        mov     edi,786F5058                      ; 垃圾代码  
00468353    81F7 DD57125B      xor     edi,5B1257DD                      ; 垃圾代码  
00468359    3BC1               cmp     eax,ecx                           ; 垃圾代码  
0046835B    81F7 B6D6FF2B      xor     edi,2BFFD6B6                      ; 垃圾代码  
00468361    81CE FF4FA918      or      esi,18A94FFF                      ; 垃圾代码  
00468367    8D85 D1616900      lea     eax,dword ptr ss:\[ebp+6961D1\]     ; 垃圾代码  
0046836D    4F                 dec     edi                               ; 垃圾代码  
0046836E    8103 33A7550A      add     dword ptr ds:\[ebx\],0A55A733       ; 计算的结果是（0068046a），送 CS+11a  
00468374    81F7 2A21832A      xor     edi,2A83212A                      ; 垃圾代码  
0046837A    8D85 308D2C48      lea     eax,dword ptr ss:\[ebp+482C8D30\]   ; 垃圾代码  
00468380    3BC1               cmp     eax,ecx                           ; 垃圾代码  
00468382    B9 ECB5265A        mov     ecx,5A26B5EC                      ; 垃圾代码  
00468387    81CA 67E14F73      or      edx,734FE167                      ; 垃圾代码  
0046838D    4B                 dec     ebx  
0046838E    83EB FB            sub     ebx,-5                            ; CS+11E  
00468391    8D85 90B22541      lea     eax,dword ptr ss:\[ebp+4125B290\]   ; 垃圾代码  
00468397    8DA8 EE62926D      lea     ebp,dword ptr ds:\[eax+6D9262EE\]   ; 垃圾代码  
0046839D    25 BA289002        and     eax,29028BA                       ; 垃圾代码  
004683A2    BE 2C267320        mov     esi,2073262C                      ; 垃圾代码  
004683A7    812B 75F43E3B      sub     dword ptr ds:\[ebx\],3B3EF475       ; 计算的结果是（FF000010），送 CS+11E  
004683AD    25 56AC092D        and     eax,2D09AC56                      ; 垃圾代码  
004683B2    8D340A             lea     esi,dword ptr ds:\[edx+ecx\]        ; 垃圾代码  
004683B5    3BC1               cmp     eax,ecx                           ; 垃圾代码  
004683B7    8D8E 31CE7824      lea     ecx,dword ptr ds:\[esi+2478CE31\]   ; 垃圾代码  
004683BD    B9 04EA210A        mov     ecx,0A21EA04                      ; 垃圾代码  
004683C2    B8 65563C31        mov     eax,313C5665                      ; 垃圾代码  
004683C7    E8 02000000        call    jl.004683CE                       ; 虚跳 CALL，花指令  
  
004683CE    5E                 pop     esi                               ; 无意义  
004683CF    4B                 dec     ebx  
004683D0    83EB FB            sub     ebx,-5                            ; CS +122  
004683D3    E8 02000000        call    jl.004683DA                       ; 虚 call 花指令,无意义  
  
                                .............  
                          一路都是这样解码＋无意义的垃圾代码  
                                .............  
  
0046947A    FF0424             inc     dword ptr ss:\[esp\]                ; 解码结束，返回地址为 4670ee  
                                                                          ;开始从解码后的代码处执行代码  
                                                                           ；以后看见这个壳入手就看第一个  
                                                                           CALL 中 ESI 值，直接跳 ESI 这里往下分析  
0046947D    C3                 retn  
  
小结：这样一段是开始，一上来就可以看出来，这个壳利用动态生产代码＋大量的垃圾  
      代码来迷惑跟踪的分析，入手就给很多普通的 DEBUG 者一个下马威，这才是万里长征的  
      开始，刚刚打好行囊，我们准备上路了 ：）  
        
004670EE   /EB 5D              jmp     short jl.0046714D  
004670F0   |8B45 00            mov     eax,dword ptr ss:\[ebp\]  
004670F3   |0BC0               or      eax,eax                           ;   
004670F5   |74 04              je      short jl.004670FB  
004670F7   |55                 push    ebp  
004670F8   |FF65 0C            jmp     near dword ptr ss:\[ebp+C\]  
004670FB   |FF45 00            inc     dword ptr ss:\[ebp\]  
004670FE   |8B4424 24          mov     eax,dword ptr ss:\[esp+24\]         ;    
00467102   |8945 04            mov     dword ptr ss:\[ebp+4\],eax          ;    
00467105   |8DB5 84000000      lea     esi,dword ptr ss:\[ebp+84\]         ; kernel32.dll,把 dll 的名称保存地址放到 esi 中  
0046710B   |56                 push    esi  
0046710C   |FF55 78            call    near dword ptr ss:\[ebp+78\]        ; kernel32.GetModuleHandleA  
0046710F   |8D75 1C            lea     esi,dword ptr ss:\[ebp+1C\]         ; 把 VirtualAlloc 名称所在的地址传到 esi 中  
00467112   |56                 push    esi  
00467113   |50                 push    eax                               ;    
00467114   |FF55 74            call    near dword ptr ss:\[ebp+74\]        ;  kernel32.GetProcAddress  
00467117   |8945 2C            mov     dword ptr ss:\[ebp+2C\],eax         ; VirtuAlloc 地址保存  
0046711A   |6A 04              push    4  
0046711C   |68 00100000        push    1000  
00467121   |FF75 10            push    dword ptr ss:\[ebp+10\]             ; 申请空间大小 17951  
00467124   |6A 00              push    0  
00467126   |FF55 2C            call    near dword ptr ss:\[ebp+2C\]        ; 申请空间,VirtualAlloc  
00467129 > |50                 push    eax                               ; 申请地址入栈(370000)  
0046712A   |8945 0C            mov     dword ptr ss:\[ebp+C\],eax          ; 申请地址  
0046712D   |8B5D 08            mov     ebx,dword ptr ss:\[ebp+8\]          ; 解码代码相对偏移 2499  
00467130   |03DD               add     ebx,ebp                           ; 把 RVA 转换成 VA（469499)  
00467132   |50                 push    eax                               ; 解码后放入申请后的地址（370000)  
00467133   |53                 push    ebx                               ; 源码首地址入栈（469499)  
00467134   |E8 18000000        call    jl.00467151                       ; 去解码，这里这个壳常常更换这个压缩引擎  
00467139   |5A                 pop     edx                               ;    
0046713A   |52                 push    edx                               ; 确认缓冲区（370000）地址段入栈  
0046713B   |55                 push    ebp                               ; 代码段地址 467000 入栈  
0046713C   |8D85 DE000000      lea     eax,dword ptr ss:\[ebp+DE\]         ; 4670de 地址入 eax  
00467142   |C600 EB            mov     byte ptr ds:\[eax\],0EB             ; 改代码  
00467145   |C640 01 10         mov     byte ptr ds:\[eax+1\],10            ; 回头把 4670DE 处修改为(jmp 4670f0)，为后面取 API 地址作准备  
00467149   |8B45 30            mov     eax,dword ptr ss:\[ebp+30\]  
0046714C   |8945 74            mov     dword ptr ss:\[ebp+74\],eax         ; 抹掉 GetProcAddress 的地址(输入表)  
0046714F  - FFE2               jmp     near edx                          ; 去申请的空间(370000)执行解码后的壳代码  
  
  
今天就写到这里，这个壳上来就给人二次解码，你想想，高手就是高手，代码里面你能看见的有多少东西？  
而且现在才刚刚开始，温柔的一刀。  
\--------------------------------------------------------------------------------  
【破解总结】  
  
  
第一章 垃圾代码＋二次解码，你怎么舍得我难过  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 hnhuqiong  
【作者邮箱】 3KKK@  
【作者主页】 3KKK  
【使用工具】 FLYODBG  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 xxxxxxxxxxxxxxxx  
【下载地址】 xxxxxxxxxxxx  
【软件简介】 对壳感兴趣，软件就不提了  
【加壳方式】 XXX 0.7X   
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
00370000    B9 04000000        mov     ecx,4                             ; 大量的花指令＋INT 3 一直到 3732ae  
00370005    E8 1F000000        call    00370029  
0037000A  ^ EB FA              jmp     short 00370006  
0037000C    E8 16000000        call    00370027  
00370011    E9 EBF80000        jmp     0037F901  
00370016    58                 pop     eax                               ;   
00370017    EB 09              jmp     short 00370022  
00370019    0F25               ???                                       ; 未知命令  
0037001B    E8 F2FFFFFF        call    00370012  
00370020    0FB9               ???                                       ; 未知命令  
  
上手分析：？？？？？？？？怎么到处都是乱七八糟的东西！！！！！  
          你进入了一个水中花，镜中月的地方，到处的花指令，我一边分析一边高声的  
          唱着谭校长的《水中花》，世界上要是有个地方到处有这样的花花草草，我们是要爱护的 ：）  
          从头来，看仔细了。OD 插件中你最好备好武器（AJUNK ，花指令去除器），我们走！！  
  
00370000    B9 04000000        mov     ecx,4                         ; 可以说，这个壳的花指令运用的让人惊叹  
00370005    E8 1F000000        call    00370029                      ；这壳代码处一开始开始让你看的几乎没有心情  
0037000A    90                 nop                                   ；玩，你会发现这里的有一个特点，就是花指令的循环  
0037000B    90                 nop                                   ；中，反复玩的就是 call，jmp，JXX，  
0037000C    E8 16000000        call    00370027                      ；反复的玩寄存器，堆栈，就是不操作内存，当你枯燥  
00370011    90                 nop                                   ；的在 F7 的时候，突然插入有用的指令，让人防不胜防  
00370012  ^ EB F8              jmp     short 0037000C                ；  
00370014    90                 nop                                   ；天道殷勤，你只要多玩花指令，玩的滚瓜烂熟，那么  
00370015    90                 nop                                   ；你学会收集花指令的模式，其实，它们也简单，跟踪者  
00370016    58                 pop     eax                           ；毕竟只是重复前人的劳动，设计花指令的人让人肃然起敬  
00370017    EB 09              jmp     short 00370022                ；要知道花指令设计上比你去跟踪要难上很多很多，设计过程中  
00370019    90                 nop                                   ；会出现多少次的程序崩溃！，一套完美的花指令是多少个日夜加上  
0037001A    90                 nop                                   ；多少严密的逻辑设计出来的，向花指令开发者致敬！！！  
0037001B    E8 F2FFFFFF        call    00370012                      ；  
00370020    90                 nop                                   ；  
00370021    90                 nop                                   ；  
00370022    49                 dec     ecx                           ；破花指令有一个特点就是花指令不会大范围的跳转，因为  
00370023  ^ 75 F1              jnz     short 00370016                ；它需要用花指令来欺骗 DEBUG 或者静态分析软件，如果大范围  
00370025    EB 05              jmp     short 0037002C                ；跳，那么设计的复杂程度将急剧上升。  
00370027    90                 nop                                   ；  
00370028    90                 nop                                   ；而且花指令有一个特点就是往指令里面跳，那么我们要感谢  
00370029    90                 nop                                   ；od 插件 AJUNK 的作者，一个特别特别好用的花指令工具，可惜最后  
0037002A    90                 nop                                   ；一个版本 BUG 严重，UNDO 有严重的 BUG 。  
0037002B    90                 nop                                   ；  
0037002C    B9 04000000        mov     ecx,4                         ；  
00370031    E8 1F000000        call    00370055                      ；左边从 370000 到 3700c0 是去掉花指令后的正常状态的代码，怎么样  
00370036    90                 nop                                   ；看的清清楚楚吧？？NOP 掉的指令不会影响任何程序状态，你所需要的  
00370037    90                 nop                                   ；是把这些代码弄明白了，用二进制 copy 下来，对比没有 NOP 掉的原程序  
00370038    E8 16000000        call    00370053                      ；我们将再次运用一个花指令工具－－－花指令去除器，好了，这个程序的  
0037003D    90                 nop                                   ；的花指令将被你收入囊中！！！  
0037003E  ^ EB F8              jmp     short 00370038                ；当你收集了这个壳足够多的花指令，那么！  
00370040    90                 nop                                   ；当你观看这个壳的时候，发现它在脱掉一层层的面纱展现在你的面前！！！  
00370041    90                 nop                                   ；  
00370042    58                 pop     eax                           ；   
00370043    EB 09              jmp     short 0037004E                ；  
00370045    90                 nop                                   ；  
00370046    90                 nop                                   ；  
00370047    E8 F2FFFFFF        call    0037003E                      ；  
0037004C    90                 nop                                   ；  
0037004D    90                 nop                                   ；  
0037004E    49                 dec     ecx  
0037004F  ^ 75 F1              jnz     short 00370042  
00370051    EB 05              jmp     short 00370058  
00370053  ^ EB F9              jmp     short 0037004E  
00370055  ^ EB F0              jmp     short 00370047  
00370057    90                 nop  
00370058    B9 04000000        mov     ecx,4  
0037005D    E8 1F000000        call    00370081  
00370062    90                 nop  
00370063    90                 nop  
00370064    E8 16000000        call    0037007F  
00370069    90                 nop  
0037006A  ^ EB F8              jmp     short 00370064  
0037006C    90                 nop  
0037006D    90                 nop  
0037006E    58                 pop     eax  
0037006F    EB 09              jmp     short 0037007A  
00370071    90                 nop  
00370072    90                 nop  
00370073    E8 F2FFFFFF        call    0037006A  
00370078    90                 nop  
00370079    90                 nop  
0037007A    49                 dec     ecx  
0037007B  ^ 75 F1              jnz     short 0037006E  
0037007D    EB 05              jmp     short 00370084  
0037007F  ^ EB F9              jmp     short 0037007A  
00370081  ^ EB F0              jmp     short 00370073  
00370083    90                 nop  
00370084    E8 03000000        call    0037008C  
00370089    90                 nop  
0037008A    90                 nop  
0037008B    90                 nop  
0037008C    58                 pop     eax  
0037008D    EB 01              jmp     short 00370090  
0037008F    90                 nop  
00370090    83C0 07            add     eax,7  
00370093    50                 push    eax  
00370094    C3                 retn  
00370097    E8 24000000        call    003700C0  
  
  
运行前的寄存器状态（EIP ＝370000)                 花指令运行后的寄存器状态（EIP ＝3700c0)                  
  
EAX 0035FE6C                                      EAX 00370097  
ECX 00000002                                      ECX 00000000  
EDX 00370000                                      EDX 00370000  
EBX 00469499 jl.00469499                          EBX 00469499 jl.00469499  
ESP 0012FF9C                                      ESP 0012FF94  
EBP 00467000 jl.00467000                          EBP 00467000 jl.00467000  
ESI 0046701C ASCII "VirtualAlloc"                 ESI 0046701C ASCII "VirtualAlloc"  
EDI 570B6113                                      EDI 570B6113  
EIP 00370000                                      EIP 003700C0  
C 0  ES 0023 32bit 0(FFFFFFFF)                    C 0  ES 0023 32bit 0(FFFFFFFF)  
P 1  CS 001B 32bit 0(FFFFFFFF)                    P 0  CS 001B 32bit 0(FFFFFFFF)  
A 0  SS 0023 32bit 0(FFFFFFFF)                    A 0  SS 0023 32bit 0(FFFFFFFF)  
Z 1  DS 0023 32bit 0(FFFFFFFF)                    Z 0  DS 0023 32bit 0(FFFFFFFF)  
S 0  FS 003B 32bit 7FFDF000(FFF)                  S 0  FS 003B 32bit 7FFDF000(FFF)  
T 0  GS 0000 NULL                                 T 0  GS 0000 NULL  
D 0                                               D 0  
O 0  LastErr ERROR\_SUCCESS (00000000)             O 0  LastErr ERROR\_SUCCESS (00000000)  
EFL 00000246 (NO,NB,E,BE,NS,PE,GE,LE)             EFL 00000202 (NO,NB,NE,A,NS,PO,GE,G)  
DR0 00000000                                      DR0 00000000  
DR1 00000000                                      DR1 00000000  
DR2 00000000                                      DR2 00000000  
DR3 00000000                                      DR3 00000000  
DR6 00000000                                      DR6 00000000  
DR7 00000000                                      DR7 00000000  
          
这里以后，我将不再将这个壳的花指令模式展现出来，为大家清楚的看到这个程序。  
           
                                       
003700C0    64:FF35 00000000   push    dword ptr fs:\[0\]            ；开始了 SEH ，这个壳大量的运用了 SEH ，  
003700C7    EB 12              jmp     short 003700DB              ’熟练程度几乎是我遇见壳中最强的，可以说  
003700DB    64:8925 00000000   mov     dword ptr fs:\[0\],esp        ；SEH 这里体现了作者的的写壳机巧和这个壳  
                            .....                                  ；树立了在密界极高的待遇。  
                            .....  
0037011A    66:BF 4D4A         mov     di,4A4D  
0037011E    CC                 int3                                ;; 这里准备 INT 3 异常了，你会发现下面去掉花指令后  
                                                                   &；-----------------------------------------   
                 （观察堆栈）                                      &；0012FF94  /0012FFE0  指针到下一个 SEH 记录  
                                                                   &；0012FF98  |003700E9  SE 句柄  
                                                                   &； 0012FF9C  |00467000  jl.00467000  
                                                                   &；-----------------------------------------  
0037011F    90                 nop                                 ；怎么和 3700CO 如此像？是的，这里就是 SEH 的出口  
00370120    90                 nop  
00370121    64:8F05 00000000   pop     dword ptr fs:\[0\]             ; 你在这里下断（F2)，F4 一把带过，祝贺你，你过了  
00370128    58                 pop     eax                          ; 第一个陷阱。其实，老老实实的看堆栈，用 SEH 的句炳  
                                                                    ；你会发现兜了一圈，回到了这里。呵呵，玩多 SEH 了  
                                                                    ；你会发现有时候有技巧的，过 SEH 过去就过去了，没有过去                                           
                                                                    ；在栽跟头地方重新仔细找问题，马上会解决的。  
  
  
小结:这里的 SEH 其实没有特别的，基本没有有用的代码，就是让 DEBUGER 和它比耐心，找到 SEH 的出口，直接奔过去，不和它纠缠。  
     其实你要是喜欢，可以把 370000 到 7372AE 一大段全 NOP 掉,并不影响程序。爽吧？  
     如果大量花指令的我用.............代替了。  
  
003732AE    E8 00000000        call    003732B3                          ; 真正开始面对了有分量的地方了，天黑  
003732B3    5A                 pop     edx                               ; 风高杀人夜，弟兄们要小心了。  
003732B4    81EA 9E1B4000      sub     edx,401B9E                        ；  
003732BA    EB 23              jmp     short 003732DF                    ；一堆花指令  
003732DF    50                 push    eax                               ‘  
003732E0    E8 EFFFFFFF        call    003732D4                          ；  
003732D4    58                 pop     eax                               ;   
003732D5    9C                 pushfd                                    ；  
003732D6    05 E6FFFFFF        add     eax,-1A                           ；  
003732DB    9D                 popfd                                     ；  
003732DC >^ FFE0               jmp     near eax                          ;   
    
.............................  
0037331B    8D75 74            lea     esi,dword ptr ss:\[ebp+74\]         ; 取 GetProcAddress 地址给 ESI  
............................  
00373348    8DBA 080F4100      lea     edi,dword ptr ds:\[edx+410F08\]     ; 将 38261d 地址传给 EDI  
.............................  
00373378    F3:A5              rep     movs dword ptr es:\[edi\],dword ptr>; GetModuleHandleA 和 LoadLibraryA 地址传送到 38261d 中  
  
\-------------------------------------------------------------------------;  
0038261D  6C FE 35 00 29 B5 80 7C 77 1D 80 7C 00 00 00 00  l?.)祤|w€|....;大家看看它准备干什么  ：）    
\-------------------------------------------------------------------------;  
  
003733A4    8B45 04            mov     eax,dword ptr ss:\[ebp+4\]          ; ntdll.7C930738  
.............................  
003733D0    8982 180F4100      mov     dword ptr ds:\[edx+410F18\],eax     ; ntdll.7C930738 传入 38262d  
.............................  
00373400    8D85 51010000      lea     eax,dword ptr ss:\[ebp+151\]         ;   
.............................  
00373431    8982 840F4100      mov     dword ptr ds:\[edx+410F84\],eax      ; 将 467151 送 382699  
.............................  
00373462    8B45 2C            mov     eax,dword ptr ss:\[ebp+2C\]          ; kernel32.VirtualAlloc  
.............................                                             ;看见这个大家应该知道它要干什么了吧？？  
0037348D    8982 140F4100      mov     dword ptr ds:\[edx+410F14\],eax            ; 将 VirtualAlloc 地址传送到 382629  
.............................  
003734E4    8B85 280F4100      mov     eax,dword ptr ss:\[ebp+410F28\]            ; 38263d 地址入 eax=0  
.............................  
003736CD    0BC0               or      eax,eax  
003736CF    75 40              jnz     short 00373711                           ; 必须跳,这里如果 F7 跟踪将会不跳  
003736D1    6A 00              push    0  
003736D3    FF95 0C0F4100      call    near dword ptr ss:\[ebp+410F0C\]           ; kernel32.GetModuleHandleA  
  
呵呵，把个地址搬过来搬过去的，原来是要弄个 call GetModuleHandleA，真累。别急，才刚刚开始呢  
  
003736E7    68 24080E68        push    680E0824                                ；压这些到堆栈干什么？  
003736EC    68 90908344        push    44839090                                ；？？？  
003736F1  - FFE4               jmp     near esp                                ；BT ，原来要跑堆栈里面执行代码了  
－－－－－－－－－－－－－堆栈里面不好玩也。就下面这样几句，也跑堆栈里面  
0012FF92    834424 08 0E       add     dword ptr ss:\[esp+8\],0E                 ；3736f4  
0012FF97    68 E6363700        push    3736E6  
0012FF9C    C2 1000            retn    10  
...............................  
003736F4    8985 180F4100      mov     dword ptr ss:\[ebp+410F18\],eax            ; 00400000-－》38262d，准备盖掉罪状  
................................  
00373711    6A 04              push    4  
00373713    68 00100000        push    1000                                     ; 申请缓冲区大小 1000  
00373718    68 00100000        push    1000  
0037371D    6A 00              push    0  
0037371F    FF95 140F4100      call    near dword ptr ss:\[ebp+410F14\]           ; VirtualAlloc 申请缓冲区（390000)  
又申请内存，乖乖，还没有走两步，就又开始申请内存玩是么名堂了？？？？  
00373725    8985 54184100      mov     dword ptr ss:\[ebp+411854\],eax            ; 将申请的缓冲区段地址入 382f69  
0037372B    68 003C070A        push    0A073C00                                 ;  
00373730    68 9FFC0D75        push    750DFC9F                                 ;干吗又推这样的东西入堆栈,看着 75 头  
                                                                                ;就明白要为下面跑堆栈里面跑代码.  
00373735    50                 push    eax                                      ;390000  
...............................................  
003737A3    FF30               push    dword ptr ds:\[eax\]                       ; 将 37373f 内的内容送到堆栈，从这里开始  
                                                                                ;到 3737ad，将把代码送入堆栈，真精彩的代码  
  
..........................................  
其中花指令带压代码入栈方式,真的很漂亮,跟着都让人感觉心旷神怡.  
...........................................  
003737AA    89E0               mov     eax,esp                                  ; 将堆栈地址入 eax,准备入堆栈执行  
003737AD    FFD0               call    near eax                                 ; 跳入堆栈进行代码执行  
  
\---------------------------------------------------------------------------------------  
0012FF44    873424             xchg    dword ptr ss:\[esp\],esi         ; 将堆栈和 ESI 的内容互换  
0012FF47    8B36               mov     esi,dword ptr ds:\[esi\]         ; 将\[ESI\]的内容赋值给 ESI  
0012FF49    81F6 EBFF71C8      xor     esi,C871FFEB                   ;将 ESI 和 C871ffeb 比较  
0012FF4F    75 19              jnz     short 0012FF6A                 ;不等则跳,这里不能跳.  
0012FF51    8B7424 50          mov     esi,dword ptr ss:\[esp+50\]      ;   
0012FF55    56                 push    esi                            ; 取 ESP+50 的内容重新压栈  
0012FF56    8B36               mov     esi,dword ptr ds:\[esi\]         ;   
0012FF58    81F6 EBFF71E8      xor     esi,E871FFEB                   ; 将 ESP+50 的内容和 E871FFEB 比较  
0012FF5E    75 09              jnz     short 0012FF69                 ;不等则跳,这里不能跳  
0012FF60    5E                 pop     esi                            ;   
0012FF61    83C6 4C            add     esi,4C  
0012FF64    897424 48          mov     dword ptr ss:\[esp+48\],esi      ; 将从堆栈的返回代码段地址压栈  
0012FF68    8D7424 58          lea     esi,dword ptr ss:\[esp+58\]      ; 将 ESP+58 地址送 ESI  
0012FF6C    51                 push    ecx                            ;   
0012FF6D    B9 02000000        mov     ecx,2                          ;要运算出 2 个指令  
0012FF72    8136 EBFF7874      xor     dword ptr ds:\[esi\],7478FFEB    ; 作这个运算真不简单,记得刚才我说的 75 头的数值压栈 0012FF78    83EE FC            sub     esi,-4                         ;的话么,这里把指令变成真正的指令.  
0012FF78    83EE FC            sub     esi,-4                         ;下一个数据  
0012FF7B    49                 dec     ecx                            ;计数器  
0012FF7C  ^ 75 F4              jnz     short 0012FF72                 ;  
0012FF7E    59                 pop     ecx                            ;  
0012FF7F    8D7424 58          lea     esi,dword ptr ss:\[esp+58\]      ; 将\[ESP+58\]内容送入 ESI  
0012FF83    FFD6               call    near esi                       ; 到 12ff98  
0012FF98   /74 03              je      short 0012FF9D  
0012FF9D    C3                 retn                                   ; 返回到 12ff85  
0012FF85    5E                 pop     esi                            ; 准备去调用 KERNEL32.dll  
0012FF86    F3:                prefix rep:  
0012FF87    68 87373700        push    373787  
0012FF8C    C2 5000            retn    50                             ; 返回代码段  
\-------------------------------------------------------------------------------------------  
大家看了这段如何?真的很精彩,在堆栈里面执行代码还不忘记边执行边解码出来,我等菜鸟佩服的五体投地.  
后面将持续的运用这样的方式.  
  
00373787    8B0424             mov     eax,dword ptr ss:\[esp\]         ; 将缓冲区（390000)段地址入 EAX  
0037378A    83EC F8            sub     esp,-8                         ; 堆栈指针＋8(12ff9c）  
............  
0037387F    5E                 pop     esi                            ; 取 370000 段地址  
00374478    56                 push    esi  
00374479    8DB5 C1214000      lea     esi,dword ptr ss:\[ebp+4021C1\]  ; 将地址（3738d6)送 ESI  
0037447F    68 03EA17B3        push    B317EA03                       ;  
..............  
003744F2    FF33               push    dword ptr ds:\[ebx\]              ;又开始要往堆栈里面压代码  
...................  
003744F9    89E3               mov     ebx,esp                          ;要往堆栈里面走了 :(  
003744FC    FFD3               call    near ebx                        ;去堆栈了,真受不了,好好程序段不用  
                                                                       ;喜欢跑堆栈里面玩,以后我们都跑女厕所  
      这里我就不跑了,和上面一样的形势.  
  
003745CE    8BFE               mov     edi,esi                     ; 3738d6 开始，446 大小准备解码，送 370000  
003745D0    B9 46040000        mov     ecx,446  
003745D5    AC                 lods    byte ptr ds:\[esi\]           ; 把取出的数据放到 al 中  
.....................................................  
003745D6    9C                 pushfd  
003745D7    6A 03              push    3  
003745EA    83C4 04            add     esp,4                        ;ESP+4  
......  
003745F1    FF0C24             dec     dword ptr ss:\[esp\]           ;3-1  
........  
003745EA    83C4 04            add     esp,4                        ;ESP+4  
........  
003745F1    FF0C24             dec     dword ptr ss:\[esp\]           ;2-1  
........  
003745EA    83C4 04            add     esp,4                        ;ESP+4  
........  
003745F1    FF0C24             dec     dword ptr ss:\[esp\]           ;1-1  
........  
........................................................  
其实上面就是将\[ESP\]中的内容=-1,这是迷惑人的,算花指令,真正也就下面的解码方式  
00374603    32C1               xor     al,cl                         ;al=8  
0037461C    C0C0 04            rol     al,4                          ;al=80  
0037464C    AA                 stos    byte ptr es:\[edi\]             ;把计算后的结果放回去  
0037464D  ^\\E2 86              loopd   short 003745D5                ;循环解码  
  
\-----------------------------------------------------------------------  
一直在解码,真是藏在深闺不露面呀,不行我跑女厕所看你去!!!!!!!!!!!!!!  
  
后面开始很危险了,这里  
  
0037464F    5E                 pop     esi                         ; 00373885  
00374650    33C0               xor     eax,eax  
00374652    64:FF30            push    dword ptr fs:\[eax\]          ; 又在建 SEH 了,准备异常了  
00374655    64:8920            mov     dword ptr fs:\[eax\],esp  
00374658    0F0B               ud2                                 ;异常  
\------------------------看句柄跟着跑呀  
0012FF9C  /0012FFE0  
0012FFA0  |00373885  返回到 00373885 来自 00374478  
0012FFA4  |7C930738  ntdll.7C930738  
\-------------------------  
00373885    E8 04000000        call    0037388E                           ;   
...........  
003738CC    FF81 B8000000      inc     dword ptr ds:\[ecx+B8\]              ;   
003738D2    3D 03000080        cmp     eax,80000003                       ;呵呵这里比这个大家明白了么?  
  
00373B24    8161 14 F00FFFFF   and     dword ptr ds:\[ecx+14\],FFFF0FF0     ; ffff0ff0  
003747B7    AC                 lods    byte ptr ds:\[esi\]                  ; 2d  
003747B8    32C1               xor     al,cl  
003747BA    04 4D              add     al,4D  
003747BC    C0C0 03            rol     al,3  
003747BF    AA                 stos    byte ptr es:\[edi\]                  ;又在解码  
003747C0  ^\\E2 F5              loopd   short 003747B7  
...........................  
0037497A    AC                 lods    byte ptr ds:\[esi\]  
0037497B    03D8               add     ebx,eax                             ; :)  
0037497D  ^ E2 FB              loopd   short 0037497A  
0037497F    8BC3               mov     eax,ebx  
00374981    F8                 clc                                       ;异常  
...........................  
  
这里是我需要套用一个前辈的一个分析,的确,刚刚过来的时候,这里感觉有点诡异,一边建 SEH  
一边解码,走的时候,我恐惧感不断地加强,在这里曾经苦闷过,伤心过,终于看见了一点光,一点点,  
这里特别感谢前辈 Windows,强烈鄙视王子,不说清楚就往下跑,这里摔得人不下成千上万,我终于爬过来  
了,这里特别贴上 Windows 的分析  
  
(★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
;在这个版本的壳中,最巧妙的就是它的 4 个关键 SEH. 接下来我们遇到了第一个.  
;这个 seh 对反跟踪效果非常的好. 它首先利用一个 int3 中断进入 seh, 设置 4 个单步断点于 dr 寄存器中.   
;然后分别在四个单步中断处完成解码的相应操作,;最后经过一个除 0 异常进入 seh,设置好 dr 寄存器的值后,拆除 seh.  
;    我们如果在 4 个单步断点之间进行软硬件中断调试,是不太可行的.   
;    a.软件: 很多代码没有解密,无法对其进行软件中断  
;    b.硬件: dr 寄存器被占用,根本无法使用硬件中断  
;所以,我们唯一的办法只有在它最后一次进入 seh 之后的代码上下软件断点"CMP EAX,CD000094". 幸好,这段代码部分没有加密,  
不然就根本无法跟踪下去了.;然后,出第一个 seh 中断. 这个时候,我们必须把 dr 寄存器中的值记录下来,因为在第二个 seh 中要  
用到. 除非,你在第一个和第二个 seh 之间不进行  
★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★  
  
这里我倒是要说,不可能它不解密,否则它自己也要陷入死循环.  
不得不佩服这个壳如此地漂亮的下陷阱,和在它自己在陷阱中解码继续前进,我也在前进了,不断前进  
我耳边开始响起来了那个<义勇军进行曲>  
  
  
003742E0    3D 940000CD        cmp     eax,CD000094                       ;F2 下断点这里,我们起飞!!!!!!!!  
  
(●★003742E5    23ACAB 121617A9 AND EBP,DWORD PTR DS:\[EBX+EBP\*4+A9171612> )★●);这里是没有解码时候走不下去的                                                             )                      
003742E5   /0F85 89010000      jnz     00374474                            ;芝麻开门了,嘻嘻  
003742EB   |8322 00            and     dword ptr ds:\[edx\],0                ;把走过来的路堵死,真厉害,步步为营  
  
00374350    C741 04 2301FF0F   mov     dword ptr ds:\[ecx+4\],0FFF0123        ;DR0  
.......  
00374382    C741 08 6745FF0F   mov     dword ptr ds:\[ecx+8\],0FFF4567        ;DR1  
.......  
003743B3    C741 0C AB89FF0F   mov     dword ptr ds:\[ecx+C\],0FFF89AB        ;DR2  
.......  
003743E5    C741 10 EFCDFF0F   mov     dword ptr ds:\[ecx+10\],0FFFCDEF       ;DR3  
.......  
00374415    8161 14 F00FFFFF   and     dword ptr ds:\[ecx+14\],FFFF0FF0       ;DR6  
.......  
00374447    C741 18 55010000   mov     dword ptr ds:\[ecx+18\],155            ;DR7  
.........  
  
EFL 00000206 (NO,NB,NE,A,NS,PE,GE,G)  
DR0 0FFF0123  
DR1 0FFF4567  
DR2 0FFF89AB  
DR3 0FFFCDEF  
DR6 FFFF0FF0  
DR7 00000555  
  
记下它们,后面有用  :)  
  
\---------我完整的把这个跨越写下来,看见没有,中间的 INT 3 就是让无数英雄卡脑袋,无数 MM 飞  
哭鼻子的地方,真黑哟,CX=51F,多少个 INT 3 哟,我耳边响起来了,  
\-----------------------有多少爱可以从来 ,  
51F(我要飞????),BT!!!!!!!!!我就是不飞,我飞多高摔多重 :)  
  
0037549E    FF35 00000000      push    dword ptr ds:\[0\]  
003754A4    64:8925 00000000   mov     dword ptr fs:\[0\],esp  
003754AB    8DB5 D1424000      lea     esi,dword ptr ss:\[ebp+4042D1\]  
003754B1    B9 1F050000        mov     ecx,51F  
003754B6    8A0431             mov     al,byte ptr ds:\[ecx+esi\]  
003754B9    CC                 int3  
003754BA    90                 nop  
003754BB    880431             mov     byte ptr ds:\[ecx+esi\],al  
003754BE  ^ E2 F6              loopd   short 003754B6  
003754C0    64:8F05 00000000   pop     dword ptr fs:\[0\]                   ; 我们下 F2 这里,这下飞高点  
  
同志们,这一节我写到这里,希望给看雪拉长点好看得东西,有钱的捧个钱场,有人的捧个人场,  
看雪是我家,卫生靠大家!  
  
◇◎◎●◎●◎◎●○●◎●★★№☆№☆№☆№№§●◎●＿↑→←△▲↑＿▲＠什么乱七八糟的东西  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 hnhuqiong  
【作者邮箱】 3KKK@  
【作者主页】 3KKK  
【使用工具】 FLYODBG  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 xxxxxxxxxxxxxx  
【下载地址】 xxxxxxxxxxxxxxxx  
【软件简介】 我们只是关心它的壳，不是软件  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
第三节：PUSH  API  RET，想说爱你不容易  
  
003754C7    58                 pop     eax         ;ax=375142  
003754C8    68 00FE90C7        push    C790FE00    ;压入堆栈的代码  
003754CD    52                 push    edx         ；  
..............  
00375542    89E2               mov     edx,esp     ;又去堆栈  
00375545    FFD2               call    near edx  
...............这里都是在重复着入堆栈执行的把戏   
  
  
00375617    8D05 8B0D4100      lea     eax,dword ptr ds:\[410D8B\]          ; 取地址=410d8b  
0037561D    03C5               add     eax,ebp                            ; 以 EBP(fff71715)作常数,和 EAX 做加=3824a0,这里就是  
0037561F    8985 040F4100      mov     dword ptr ss:\[ebp+410F04\],eax      ; 把结果送入 382619  
\----------我们来看看这里放什么东西?后面怎么有点面熟??  
\----------哟,7c80b529,7c801d77,7c809a81 三个数据怎么跑这里  
\----------躲着??我们下刀看看什么东西??  
\----------7c80b529->ModuleHandleA  
\----------7c801d77->LoadLibraryA  
\----------7c809a81->VirtualAlloc  
\----------哈哈,我看见一只绣花鞋!!!!!!!!!!!!!!!!原来你跑这里躲起来了  
\----------真是踏破铁鞋什么来着???结果看见绣花鞋    :)  
\---------------------------------------------------------  
00382619  A0 24 38 00 6C FE 35 00 29 B5 80 7C 77 1D 80 7C  ?8.l?.)祤|w€|   
00382629  81 9A 80 7C 00 00 40 00 00 00 00 00 00 00 00 00  仛€|..@.........  
\-----------------------------------------------------------  
00375625    8DB5 40104100      lea     esi,dword ptr ss:\[ebp+411040\]      ; 取 KERNEL32.DLL 地址  
........  
00375635    8D85 493F4000      lea     eax,dword ptr ss:\[ebp+403F49\]      ; 取???地址=37565E  
.......  
0037564C    8B85 0C0F4100      mov     eax,dword ptr ss:\[ebp+410F0C\]      ; 取 kernel32.GetModuleHandleA 地址  
.......  
00375658   /E9 0EC90000        jmp     00381F6B                             
◆◆◆◆前面一堆指令就是取出绣花鞋子,然后大范围的跳肯定有明堂,我们看看它准备干什么???  
  
.......  
00381F6B    6A 00              push    0  
00381F6D    50                 push    eax                                ; GetModuleHandleA 入栈  
00381F6E    8B85 54184100      mov     eax,dword ptr ss:\[ebp+411854\]      ;取缓冲区地址  
00381F74    68 00FE2FC7        push    C72FFE00  
00381F79    50                 push    eax                                ;缓冲区地址入栈  
.......                                 
00381FE7    FF30               push    dword ptr ds:\[eax\]                 ; eax=381F83 的内容(eb0050c2)入栈  
又准备跑女厕所里面去了,又在压代码入栈,这里就不解释了,记住上面它做了什么准备工作  
就行,不要看的脑袋都大,我分析起来,比大家还要累  :)说白了,这个跑堆栈执行就是为了  
迷惑人,让人感觉受不了,还好,我受的了,拿着手术刀慢慢看着呢,古人有语苞丁解牛,  
我们就学古人.我们玩这个壳是学习来得,嘻嘻  
它从堆栈出来后,进入了一个庞大的判断代码,要干什么呢???  
...........  
\-------------------------------             -----------------------------------------------------------  
003820D2    8B7C24 24          mov     edi,dword ptr ss:\[esp+24\]  
003820D6    8B7424 28          mov     esi,dword ptr ss:\[esp+28\]          ; 取 GetModuleHandleA  
003820DA    66:8B06            mov     ax,word ptr ds:\[esi\]               ; 取\[SI\]内容=8BFF(这里就是 MOV EDI,EDI 这个指令)  
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆  
??????????????????????????????????????????????????????????????????????????  
我们来看看\[SI\]中的是什么?也就是下面的内容,哦,是 GetModuleHandleA 处的代码,它要取这个内容干什么?  
7C80B529 >  8BFF            mov     edi,edi  
7C80B52B    55              push    ebp  
7C80B52C    8BEC            mov     ebp,esp  
7C80B52E    837D 08 00      cmp     dword ptr ss:\[ebp+8\],0  
??????????????????????????????????????????????????????????????????????????  
  
                         我先把下面的一段代码移动到这里给大家先有个思想准备  
                         其中\*号是它的运算后的东西送缓冲区代码,而&号内的是  
                         缓冲区 390000 内的执行代码  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
(这里,我们看看这个很特别的指令,在 Pentium 以上的 CPU 中，提供了一条机器指令 RDTSC（Read Time Stamp Counter）  
来读取这个时间戳的数字，并将其保存在 EDX:EAX 寄存器对中,这个壳大量的运用了这个反调试的指令,因为要是连贯  
运行,那么,几个机器周期将很短很短的时间,但是,当你拿着 DEBUG 一步一步看得时候,那么过去了多少时间????ANTI-Debug  
这个是一个非常有效的方式来检查是否被跟踪了,要避开它很简单,就是运行后,你把 EAX 给清成一个小数字,你平安了!!!!!)  
  
00382471    0F31               rdtsc  
00382473    8BC8               mov     ecx,eax                     ; 把取到的时间数送 ECX,别问我为什么不取 EDX  :)  
00382475    C1E8 03            shr     eax,3                       ;  
00382478    83E0 03            and     eax,3                       ;   
                                                                   ;这里是它取基准值  
0038247B    C607 EB            mov     byte ptr ds:\[edi\],0EB       ; 把 EB 送缓冲区+2(3900002)的地方,  
0038247E    47                 inc     edi  
0038247F    890F               mov     dword ptr ds:\[edi\],ecx      ; 把时间值送缓冲区(390003)  
00382481    8807               mov     byte ptr ds:\[edi\],al        ; 把计算后的 al=1 送缓冲区(390003),我们到下面  
                                                                   ;看看整个缓冲区  
00382483    40                 inc     eax                         ; ax+1=2  
00382484    03F8               add     edi,eax                     ; DI+2  
00382486  ^\\E9 4FFCFFFF        jmp     003820DA                    ;回头去比较代码的地方了,  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
这里是缓冲区 390000 的东西,它跑这个缓冲区干什么?其实,是执行被修改掉的 7c80b529 的代码  
也就是一个偷梁换柱的概念,它不让好好的执行 7c80b529(GetModuleHandleA)处的代码,而是偷换  
掉后执行自己的,这样,不会弄得系统崩溃,也很好的隐藏了它的绣花鞋.小娘子不知道到底  
是一双马脚还是三寸金莲  :)反正我知道这个小娘子喜欢乱上厕所  :)  
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
00390000    57                 push    edi  
00390001    5F                 pop     edi                                      ; 0037565E  
00390002    EB 01              jmp     short 00390005  
00390004    90                 nop  
00390005    55                 push    ebp  
00390006    EB 00              jmp     short 00390008  
00390008    54                 push    esp  
00390009    5D                 pop     ebp                                      ; 0037565E  
0039000A    EB 00              jmp     short 0039000C  
0039000C    68 2EB5807C        push    7C80B52E  
00390011    C3                 retn  
&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&  
  
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆  
言归正传,我们继续刚才的-----------处,也就是准备开始处开始分析  
003820DD    3C 50              cmp     al,50                              ; 将取出的内容低位进行比较,为什么和 50 比?不是 8B  
                                                                          ;么?这里就是取时间后,要判断的东西,见我提到上面  
003820DF    72 0D              jb      short 003820EE                     ; 去的代码.  
003820E1    3C 57              cmp     al,57                              ;57 这里是判断是否是 pop(55)  
003820E3    77 09              ja      short 003820EE                     ;大于则跳,这里到跳下面了,  
003820E5    8807               mov     byte ptr ds:\[edi\],al               ;将 pop 放入缓冲区  
003820E7    46                 inc     esi                                ;   
003820E8    47                 inc     edi                                ;将 esi 和 EDI 分别+1  
003820E9    E9 83030000        jmp     00382471                           ;去进行时间检查的地方,呵呵,在定位要修改的地方  
003820EE    3C 6A              cmp     al,6A                              ;和 6A(其实 6A 是 PUSH)比,怎么好像和缓冲区的内容  
                                                                          ;相对应?  
003820F0    75 15              jnz     short 00382107                     ;如果相等,那么说明修改过了  :)  
003820F2    46                 inc     esi                                ;   
003820F3    46                 inc     esi                                ; 将 ESI+2  
003820F4    C607 68            mov     byte ptr ds:\[edi\],68               ; 将 68(PUSH)送入缓冲区  
003820F7    47                 inc     edi                                ;  
003820F8    8AC4               mov     al,ah                              ;  
003820FA    0FBEC8             movsx   ecx,al                             ;  
003820FD    890F               mov     dword ptr ds:\[edi\],ecx             ;把要修改的东西的内容继续入缓冲区  
003820FF    83C7 04            add     edi,4                              ;edi+4  
00382102    E9 6A030000        jmp     00382471                           ;去时间判断地方  
00382107    3C 68              cmp     al,68                              ;68,和 68(其实就是 PUSH)比  
00382109    75 0C              jnz     short 00382117                     ;刚才比较送进去的东西,要继续比较,真明白为什么这个程序  
                                                                          ;的兼容性特别好了,每一步都查看.  
0038210B    B9 05000000        mov     ecx,5                              ;cx=5  
00382110    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>;送 5 个数据到缓冲区  
00382112    E9 5A030000        jmp     00382471                           ;去时间判断地方  
00382117    3C A1              cmp     al,0A1                             ;a1,和 A1(其实就是 mov)比  
00382119    75 2A              jnz     short 00382145  
0038211B    C607 E8            mov     byte ptr ds:\[edi\],0E8              ;把 E8(call)送入缓冲区  
0038211E    47                 inc     edi                                ;EDI+1  
0038211F    C707 04000000      mov     dword ptr ds:\[edi\],4               ;把 04 送入缓冲区  
00382125    83C7 04            add     edi,4                              ;EDI+4  
00382128    8B4E 01            mov     ecx,dword ptr ds:\[esi+1\]           ;将  
0038212B    890F               mov     dword ptr ds:\[edi\],ecx             ;送绣花鞋的东西到缓冲区  :)  
0038212D    83C7 04            add     edi,4                              ;edi+4  
00382130    C607 58            mov     byte ptr ds:\[edi\],58               ;将 58 送缓冲区(  
00382133    47                 inc     edi                                ;edi+1  
00382134    C707 8B008B00      mov     dword ptr ds:\[edi\],8B008B          ;送 8b008b 到缓冲区  
0038213A    83C7 04            add     edi,4                              ;edi+4  
0038213D    83C6 05            add     esi,5                              ;ESI+5  
00382140    E9 2C030000        jmp     00382471                           ;呵呵又跑时间判断里面了,  
00382145    3C 8B              cmp     al,8B                            ;8b,(其实就是 mov edi,edi)比  
00382147    0F85 67010000      jnz     003822B4  
0038214D    80FC 40            cmp     ah,40                            ;看高位和 40 比较,(其实是 mov eax,dword ptr ds:\[eax+55\]);  
00382150    0F83 98000000      jnb     003821EE  
00382156    8AC4               mov     al,ah                            ;把 AH 送 AL  
00382158    24 07              and     al,7                             ;and al,7   
0038215A    3C 04              cmp     al,4                             ;和 4 比  
0038215C    75 4F              jnz     short 003821AD  
0038215E    8A4E 02            mov     cl,byte ptr ds:\[esi+2\]           ;取 ESI+2 内容  
00382161    80E1 07            and     cl,7                             ;and cl,7  
00382164    80F9 05            cmp     cl,5                             ;比较 cl  
00382167    75 26              jnz     short 0038218F  
00382169    C607 FF            mov     byte ptr ds:\[edi\],0FF            ;将 FF 送入缓冲区  
0038216C    04 30              add     al,30                            ;al+30  
0038216E    47                 inc     edi                              ;EDI+1  
0038216F    8807               mov     byte ptr ds:\[edi\],al             ;结果送入缓冲区  
00382171    47                 inc     edi                                  
00382172    8A46 02            mov     al,byte ptr ds:\[esi+2\]  
00382175    8807               mov     byte ptr ds:\[edi\],al  
00382177    47                 inc     edi  
00382178    8B4E 03            mov     ecx,dword ptr ds:\[esi+3\]  
0038217B    890F               mov     dword ptr ds:\[edi\],ecx  
0038217D    83C7 04            add     edi,4  
00382180    8AC4               mov     al,ah  
00382182    C0E8 03            shr     al,3  
00382185    04 58              add     al,58  
00382187    8807               mov     byte ptr ds:\[edi\],al  
00382189    47                 inc     edi  
0038218A    83C6 07            add     esi,7  
0038218D    EB 55              jmp     short 003821E4  
0038218F    C607 FF            mov     byte ptr ds:\[edi\],0FF  
00382192    04 30              add     al,30  
00382194    47                 inc     edi  
00382195    8807               mov     byte ptr ds:\[edi\],al  
00382197    47                 inc     edi  
00382198    8A46 02            mov     al,byte ptr ds:\[esi+2\]  
0038219B    8807               mov     byte ptr ds:\[edi\],al  
0038219D    47                 inc     edi  
0038219E    8AC4               mov     al,ah  
003821A0    C0E8 03            shr     al,3  
003821A3    04 58              add     al,58  
003821A5    8807               mov     byte ptr ds:\[edi\],al  
003821A7    47                 inc     edi  
003821A8    83C6 03            add     esi,3  
003821AB    EB 37              jmp     short 003821E4  
003821AD    3C 05              cmp     al,5  
003821AF    75 1D              jnz     short 003821CE  
003821B1    8B4E 02            mov     ecx,dword ptr ds:\[esi+2\]  
003821B4    66:C707 FF35       mov     word ptr ds:\[edi\],35FF  
003821B9    894F 02            mov     dword ptr ds:\[edi+2\],ecx  
003821BC    83C7 06            add     edi,6  
003821BF    8AC4               mov     al,ah  
003821C1    C0E8 03            shr     al,3  
003821C4    04 58              add     al,58  
003821C6    8807               mov     byte ptr ds:\[edi\],al  
003821C8    47                 inc     edi  
003821C9    83C6 06            add     esi,6  
003821CC    EB 16              jmp     short 003821E4  
003821CE    C607 FF            mov     byte ptr ds:\[edi\],0FF  
003821D1    47                 inc     edi  
003821D2    04 30              add     al,30  
003821D4    8807               mov     byte ptr ds:\[edi\],al  
003821D6    47                 inc     edi  
003821D7    8AC4               mov     al,ah  
003821D9    C0E8 03            shr     al,3  
003821DC    04 58              add     al,58  
003821DE    8807               mov     byte ptr ds:\[edi\],al  
003821E0    47                 inc     edi  
003821E1    83C6 02            add     esi,2  
003821E4    E9 88020000        jmp     00382471  
003821E9    E9 C6000000        jmp     003822B4  
003821EE    80FC 80            cmp     ah,80                            ;和 80 比(其实是 mov  eax,dword ptr ds:\[eax+83EC8B55\])  
003821F1    73 50              jnb     short 00382243  
003821F3    80EC 40            sub     ah,40  
003821F6    8AC4               mov     al,ah  
003821F8    24 07              and     al,7  
003821FA    3C 04              cmp     al,4  
003821FC    75 22              jnz     short 00382220  
003821FE    C607 FF            mov     byte ptr ds:\[edi\],0FF  
00382201    04 70              add     al,70  
00382203    47                 inc     edi  
00382204    8807               mov     byte ptr ds:\[edi\],al  
00382206    47                 inc     edi  
00382207    66:8B4E 02         mov     cx,word ptr ds:\[esi+2\]  
0038220B    66:890F            mov     word ptr ds:\[edi\],cx  
0038220E    83C7 02            add     edi,2  
00382211    8AC4               mov     al,ah  
00382213    C0E8 03            shr     al,3  
00382216    04 58              add     al,58  
00382218    8807               mov     byte ptr ds:\[edi\],al  
0038221A    47                 inc     edi  
0038221B    83C6 04            add     esi,4  
0038221E    EB 1C              jmp     short 0038223C  
00382220    C607 FF            mov     byte ptr ds:\[edi\],0FF  
00382223    04 70              add     al,70  
00382225    47                 inc     edi  
00382226    8807               mov     byte ptr ds:\[edi\],al  
00382228    47                 inc     edi  
00382229    8A46 02            mov     al,byte ptr ds:\[esi+2\]  
0038222C    8807               mov     byte ptr ds:\[edi\],al  
0038222E    47                 inc     edi  
0038222F    8AC4               mov     al,ah  
00382231    C0E8 03            shr     al,3  
00382234    04 58              add     al,58  
00382236    8807               mov     byte ptr ds:\[edi\],al  
00382238    47                 inc     edi  
00382239    83C6 03            add     esi,3  
0038223C    E9 30020000        jmp     00382471  
00382241    EB 71              jmp     short 003822B4  
00382243    80FC C0            cmp     ah,0C0                                   ;其实就是 mov eax,eax  
00382246    73 4F              jnb     short 00382297  
00382248    80EC 80            sub     ah,80  
0038224B    8AC4               mov     al,ah  
0038224D    24 07              and     al,7  
0038224F    3C 04              cmp     al,4  
00382251    75 1F              jnz     short 00382272  
00382253    C607 FF            mov     byte ptr ds:\[edi\],0FF  
00382256    04 B0              add     al,0B0  
00382258    47                 inc     edi  
00382259    8807               mov     byte ptr ds:\[edi\],al  
0038225B    47                 inc     edi  
0038225C    B9 05000000        mov     ecx,5  
00382261    83C6 02            add     esi,2  
00382264    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
00382266    8AC4               mov     al,ah  
00382268    C0E8 03            shr     al,3  
0038226B    04 58              add     al,58  
0038226D    8807               mov     byte ptr ds:\[edi\],al  
0038226F    47                 inc     edi  
00382270    EB 1E              jmp     short 00382290  
00382272    C607 FF            mov     byte ptr ds:\[edi\],0FF  
00382275    04 B0              add     al,0B0  
00382277    47                 inc     edi  
00382278    8807               mov     byte ptr ds:\[edi\],al  
0038227A    47                 inc     edi  
0038227B    8B4E 02            mov     ecx,dword ptr ds:\[esi+2\]  
0038227E    890F               mov     dword ptr ds:\[edi\],ecx  
00382280    83C7 04            add     edi,4  
00382283    8AC4               mov     al,ah  
00382285    C0E8 03            shr     al,3  
00382288    04 58              add     al,58  
0038228A    8807               mov     byte ptr ds:\[edi\],al  
0038228C    47                 inc     edi  
0038228D    83C6 06            add     esi,6  
00382290    E9 DC010000        jmp     00382471  
00382295    EB 1D              jmp     short 003822B4  
00382297    80EC C0            sub     ah,0C0                              ;ah-c0=3f  
0038229A    8AC4               mov     al,ah                               ;al=3f  
0038229C    24 07              and     al,7                                ;and al,7=7  
0038229E    04 50              add     al,50                               ;al+50=57  
003822A0    C0EC 03            shr     ah,3                                ;shr ah,3=7    
003822A3    80C4 58            add     ah,58                               ;and ah,58=5F,这样计算后的结果是 5F57   
003822A6    66:8907            mov     word ptr ds:\[edi\],ax                ;5F57 也就是(PUSH EDI,POP EDI),改掉 7080b529 处的代码,  
                                                                                
003822A9    83C7 02            add     edi,2                                ;缓冲区地址+2  
003822AC    83C6 02            add     esi,2                                ;将 ESI+2,为的是跳过刚才该动的地方执行  
003822AF    E9 BD010000        jmp     00382471  
003822B4    3C A3              cmp     al,0A3                               ;00a3 ,其实是 mov dword ptr ds:\[EC8B55FF\],eax  
003822B6    75 0C              jnz     short 003822C4  
003822B8    B9 05000000        mov     ecx,5  
003822BD    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
003822BF    E9 AD010000        jmp     00382471  
003822C4    66:3D 2BD2         cmp     ax,0D22B                             ;d22b,其实就是,shr byte ptr ds:\[ebx\],cl  
003822C8    75 30              jnz     short 003822FA  
003822CA    66:8907            mov     word ptr ds:\[edi\],ax  
003822CD    46                 inc     esi                                ;   
003822CE    46                 inc     esi                                ;   
003822CF    47                 inc     edi  
003822D0    47                 inc     edi  
003822D1    8BDE               mov     ebx,esi                            ;   
003822D3    AC                 lods    byte ptr ds:\[esi\]  
003822D4    EB 01              jmp     short 003822D7  
003822D6    AC                 lods    byte ptr ds:\[esi\]  
003822D7    3C C3              cmp     al,0C3  
003822D9  ^ 75 FB              jnz     short 003822D6  
003822DB    4E                 dec     esi                                ;   
003822DC    C607 68            mov     byte ptr ds:\[edi\],68  
003822DF    8D47 0B            lea     eax,dword ptr ds:\[edi+B\]  
003822E2    8947 01            mov     dword ptr ds:\[edi+1\],eax  
003822E5    C647 05 68         mov     byte ptr ds:\[edi+5\],68  
003822E9    8977 06            mov     dword ptr ds:\[edi+6\],esi           ;   
003822EC    C647 0A C3         mov     byte ptr ds:\[edi+A\],0C3  
003822F0    83C7 0B            add     edi,0B  
003822F3    8BF3               mov     esi,ebx  
003822F5    E9 77010000        jmp     00382471  
003822FA    66:3D FF74         cmp     ax,74FF                             ;74ff 其实就是 JE  
003822FE    75 0C              jnz     short 0038230C  
00382300    B9 04000000        mov     ecx,4  
00382305    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
00382307    E9 65010000        jmp     00382471  
0038230C    66:3D FF75         cmp     ax,75FF                             ;75ff 其实就是 JNE  
00382310    75 1D              jnz     short 0038232F  
00382312    66:C707 FF74       mov     word ptr ds:\[edi\],74FF  
00382317    83C7 02            add     edi,2  
0038231A    0F31               rdtsc  
0038231C    83E0 03            and     eax,3  
0038231F    C1E0 06            shl     eax,6  
00382322    83C0 25            add     eax,25  
00382325    AA                 stos    byte ptr es:\[edi\]  
00382326    83C6 02            add     esi,2  
00382329    A4                 movs    byte ptr es:\[edi\],byte ptr ds:\[esi>  
0038232A    E9 42010000        jmp     00382471  
0038232F    8AC8               mov     cl,al                             ;都不等于地话将做掩饰  
00382331    80E1 F8            and     cl,0F8                            ;and cl,0f8  
00382334    80F9 B0            cmp     cl,0B0                              
00382337    75 0E              jnz     short 00382347  
00382339    66:8907            mov     word ptr ds:\[edi\],ax  
0038233C    83C7 02            add     edi,2  
0038233F    83C6 02            add     esi,2  
00382342    E9 2A010000        jmp     00382471  
00382347    8AC8               mov     cl,al                             ;  
00382349    80E1 F8            and     cl,0F8  
0038234C    80F9 B8            cmp     cl,0B8  
0038234F    75 1B              jnz     short 0038236C  
00382351    8B4E 01            mov     ecx,dword ptr ds:\[esi+1\]  
00382354    C607 68            mov     byte ptr ds:\[edi\],68  
00382357    47                 inc     edi  
00382358    890F               mov     dword ptr ds:\[edi\],ecx  
0038235A    83C7 04            add     edi,4  
0038235D    24 07              and     al,7  
0038235F    04 58              add     al,58  
00382361    8807               mov     byte ptr ds:\[edi\],al  
00382363    47                 inc     edi  
00382364    83C6 05            add     esi,5  
00382367    E9 05010000        jmp     00382471  
0038236C    3C E8              cmp     al,0E8                           ;0e8 其实就是 call  
0038236E    75 25              jnz     short 00382395  
00382370    8D47 0B            lea     eax,dword ptr ds:\[edi+B\]  
00382373    C607 68            mov     byte ptr ds:\[edi\],68  
00382376    8947 01            mov     dword ptr ds:\[edi+1\],eax  
00382379    8D46 05            lea     eax,dword ptr ds:\[esi+5\]  
0038237C    0346 01            add     eax,dword ptr ds:\[esi+1\]  
0038237F    C647 05 68         mov     byte ptr ds:\[edi+5\],68  
00382383    8947 06            mov     dword ptr ds:\[edi+6\],eax  
00382386    C647 0A C3         mov     byte ptr ds:\[edi+A\],0C3  
0038238A    83C6 05            add     esi,5  
0038238D    83C7 0B            add     edi,0B  
00382390    E9 DC000000        jmp     00382471  
00382395    66:3D 64FF         cmp     ax,0FF64                        ;ff64 其实就是 call fs\[x\],SEH 的意思  
00382399    75 25              jnz     short 003823C0  
0038239B    807E 02 32         cmp     byte ptr ds:\[esi+2\],32  
0038239F    75 09              jnz     short 003823AA  
003823A1    B9 03000000        mov     ecx,3  
003823A6    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
003823A8    EB 11              jmp     short 003823BB  
003823AA    807E 02 35         cmp     byte ptr ds:\[esi+2\],35  
003823AE    75 09              jnz     short 003823B9  
003823B0    B9 07000000        mov     ecx,7  
003823B5    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
003823B7    EB 02              jmp     short 003823BB  
003823B9    EB 05              jmp     short 003823C0  
003823BB    E9 B1000000        jmp     00382471  
003823C0    66:3D 6489         cmp     ax,8964                       ;8964    
003823C4    75 25              jnz     short 003823EB  
003823C6    807E 02 22         cmp     byte ptr ds:\[esi+2\],22  
003823CA    75 09              jnz     short 003823D5  
003823CC    B9 03000000        mov     ecx,3  
003823D1    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
003823D3    EB 11              jmp     short 003823E6  
003823D5    807E 02 25         cmp     byte ptr ds:\[esi+2\],25  
003823D9    75 09              jnz     short 003823E4  
003823DB    B9 07000000        mov     ecx,7  
003823E0    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds>  
003823E2    EB 02              jmp     short 003823E6  
003823E4    EB 05              jmp     short 003823EB  
003823E6    E9 86000000        jmp     00382471  
003823EB    3C 83              cmp     al,83                         ;83    
003823ED    75 36              jnz     short 00382425  
003823EF    80FC BF            cmp     ah,0BF  
003823F2    76 2F              jbe     short 00382423  
003823F4    80EC C0            sub     ah,0C0  
003823F7    8AC4               mov     al,ah  
003823F9    24 07              and     al,7  
003823FB    04 50              add     al,50  
003823FD    8807               mov     byte ptr ds:\[edi\],al  
003823FF    47                 inc     edi  
00382400    C607 83            mov     byte ptr ds:\[edi\],83  
00382403    47                 inc     edi  
00382404    80E4 38            and     ah,38  
00382407    80C4 04            add     ah,4  
0038240A    8827               mov     byte ptr ds:\[edi\],ah  
0038240C    47                 inc     edi  
0038240D    C607 24            mov     byte ptr ds:\[edi\],24  
00382410    47                 inc     edi  
00382411    8A66 02            mov     ah,byte ptr ds:\[esi+2\]  
00382414    8827               mov     byte ptr ds:\[edi\],ah  
00382416    47                 inc     edi  
00382417    04 08              add     al,8  
00382419    8807               mov     byte ptr ds:\[edi\],al  
0038241B    47                 inc     edi  
0038241C    83C6 03            add     esi,3  
0038241F    EB 50              jmp     short 00382471  
00382421    EB 02              jmp     short 00382425  
00382423    EB 00              jmp     short 00382425  
00382425    3C CC              cmp     al,0CC                            ;cc? INT 3  
00382427    75 05              jnz     short 0038242E  
00382429    E9 A40E0000        jmp     003832D2  
0038242E    66:3D CD03         cmp     ax,3CD                            ;03cd INT 3  
00382432    75 05              jnz     short 00382439  
00382434    E9 990E0000        jmp     003832D2  
00382439    C607 68            mov     byte ptr ds:\[edi\],68              ;比较了半天,这里才是真命天子,68 入缓冲区  
0038243C    8977 01            mov     dword ptr ds:\[edi+1\],esi          ; 送地址到缓冲区  
                                                                         ;这里形成了这样一句:  
                                                                         ;68 29B5807C push kernel32.GetModuleHandleA  
0038243F    C647 05 C3         mov     byte ptr ds:\[edi+5\],0C3           ;把返回(0c3=ret)送过去  
00382443    83C7 06            add     edi,6                             ;  
00382446    897C24 FC          mov     dword ptr ss:\[esp-4\],edi          ;把 EDI+6 地址压栈  
0038244A    837C24 2C 00       cmp     dword ptr ss:\[esp+2C\],0           ;  
0038244F    74 18              je      short 00382469                    ;  
00382451    8BCE               mov     ecx,esi                            ;   
00382453    2B4C24 28          sub     ecx,dword ptr ss:\[esp+28\]          ;   
00382457    8B7C24 28          mov     edi,dword ptr ss:\[esp+28\]          ;   
0038245B    33C0               xor     eax,eax  
0038245D    51                 push    ecx  
0038245E    C1E9 02            shr     ecx,2  
00382461    F3:AB              rep     stos dword ptr es:\[edi\]  
00382463    59                 pop     ecx                                ;   
00382464    83E1 03            and     ecx,3  
00382467    F3:AA              rep     stos byte ptr es:\[edi\]  
00382469    61                 popad  
0038246A    8B4424 DC          mov     eax,dword ptr ss:\[esp-24\]          ;将起始指针(390000)送入 EAX  
0038246E    C2 0C00            retn    0C                                 ;返回去缓冲区执行  
\----------------------------------                 -------------------------------------------------------  
  
  
这里需要小节一下,  
这里很长,也很漂亮首先,进来,查看 GetModuleHandleA 处的代码是否已经修改,如果没有修改,  
将其修改后,去执行缓冲区中的仿真代码,让 API 调用完全隐匿起来,所以,我们以前下各种 API 断点都无法断下它,  
因为它藏匿的很严也很深,这个就是典型的 PUSH API RETN 方式,为了防止被修改的代码出现不可测因素,在这个  
段中做了大量的判断,我完整的把它 COPY 下来,是因为我崇敬作者,在修改后,不断地完善这个代码,使这片代码不  
断地兼容各种不可测因素,要知道,修改这些 API 会造成大量的不可测因素,也就是为什么很多壳及其不稳定的因素  
,其实,开 390000(大小 1000)这个缓冲区就是为了在缓冲区中尽可能的兼容好各种因素,作者的苦心真是难得.其实,  
这段代码很多 PACKer 应该看看,这样的兼容性的确做到了尽可能的去考虑到各种因素,这也是我为什么一直很静下  
心来慢慢跟踪这个壳的原因.有些东西我写的到处都是现在重新写,真是一个废体力的活,以前跟踪的时候,写的记  
录到处都是,现在放满桌子,再敲回来有点困难,这里我就不一一录入了.请大家原谅.  
  
这里我送上一个 PUSH  API RET 这样的解释如下:  
1)定位 API 的映像基址  
2)以名字查找它，通常将该名字的固定编码(hardcoded)hash 值(hash value) 与每个 API 的 hash 值相比较，直到匹配为止。  
3)模拟 API 的前 K 个指令(K 每次可以随机选取)  
4)跳转到第(K+1)-th 个 API 指令  
  
  
  
为了后面能够清晰的看见我们的跟踪,这里我们弄一下手脚,将  
003820DF    720D      jb   short 003820EE  
这句修改一下,直接奔 00382439,    
如这样就行.   jmp 382439  
看看我们的执行效果:  
00390000    68 29B5807C        push    kernel32.GetModuleHandleA  
00390005    C3                 retn                  ;看多简介的模式!!  
我们去 API 看看,哈哈,很清楚哟,  
0012FF9C   0037565E  /CALL 到 GetModuleHandleA  
0012FFA0   00382755  \\pModule = "KERNEL32.dll"  
  
  
  
这样,既可以不再去重复劳动,也可以在我们的 OD 中将清晰的展现各种 API 了,嘻嘻,大家试验一下,是否  
感觉轻松了很多很多.  
  
明天要上班了,我先整理出这些东西,一有空马上继续把我的废纸堆整理弄给大家.  :)  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 hnhuqiong  
【作者邮箱】 3KKK@  
【作者主页】 3KKK  
【使用工具】 FLYODBG  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 XXXXXXXX  
【下载地址】 XXXXXXXXX  
【软件简介】 不多说了，继续  
【加壳方式】 某人 0.7X  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
孙悟空三打白骨精  
  
这里作者将大量的取出有用的 API 地址  
0037565E    8BF0               mov     esi,eax                            ; 取出的 KERNEL.DLL 的基址给 ESI  
00375660    8985 40104100      mov     dword ptr ss:\[ebp+411040\],eax      ; 将基址送入 382755，  
00375666    8D9D 4E104100      lea     ebx,dword ptr ss:\[ebp+41104E\]      ；将基址的地址送入 ebx  
0037566C    68 00FE98B6        push    B698FE00                           ；又压代码入栈，要去女厕所了  
00375671    50                 push    eax                                ; 基址入栈  
...........去女厕所转了一趟  
0003757BB    6A 21              push    21  
003757BD    59                 pop     ecx                                ; 计数器＝21，这里估计要解码 21 个是么东西？  
                                                                          ;我们静静的等待  
003757BE    8DBD 31154100      lea     edi,dword ptr ss:\[ebp+411531\]      ; 取地址给 edi（382C46)，这里放解码后的东西  
003757C4    53                 push    ebx                                ; 382762,将要解码的源地址送 ebx  
003757C5    8A03               mov     al,byte ptr ds:\[ebx\]               ; 送 382763 的内容给 AL＝bc  
003757C7    EB 06              jmp     short 003757CF  
003757C9    F6D0               not     al                                 ; 这里就是简单的 NOT 解码  
003757CB    AA                 stos    byte ptr es:\[edi\]                  ; 解码后数据送地址 382c46  
003757CC    43                 inc     ebx  
003757CD    8A03               mov     al,byte ptr ds:\[ebx\]  
003757CF    0AC0               or      al,al  
003757D1  ^ 75 F6              jnz     short 003757C9  
003757D3    AA                 stos    byte ptr es:\[edi\]                  ; 解码完毕，哦原来是 CloseHandle  
003757D4    5B                 pop     ebx  
003757D5    8DBD 31154100      lea     edi,dword ptr ss:\[ebp+411531\]      ; 将解码后的地址送 EDI  
003757DB    51                 push    ecx  
003757DC    57                 push    edi  
003757DD    56                 push    esi                                ; 呵呵，好东西压栈了  
..................  
003757E7    8D85 FB404000      lea     eax,dword ptr ss:\[ebp+4040FB\]      ;取地址 375810  
................  
00381F6B    6A 00              push    0                                  ; 0 入栈  
00381F6D    50                 push    eax                                ; 将前面取的数据入栈=3824a0,这里将是一个特别地方  
00381F6E    8B85 54184100      mov     eax,dword ptr ss:\[ebp+411854\]      ; 取缓冲区地址  
00381F74    68 00FE2FC7        push    C72FFE00                           ; 压代码入栈  
.................  
       这里又要去女厕所,然后加上远跳转去干什么呢?好,大家安静听我分解  
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆  
..................  
003824A0    55                 push    ebp  
003824A1    8BEC               mov     ebp,esp                            ; 怎么好熟悉的代码？？？  
003824A3    83C4 F4            add     esp,-0C  
003824A6    60                 pushad                                     ; 保存现场  
003824A7    8B75 08            mov     esi,dword ptr ss:\[ebp+8\]           ; Kernel.dll 基址送 ESI  
003824AA    0BF6               or      esi,esi                            ;  
003824AC    75 0E              jnz     short 003824BC                     ;  
003824AE    64:A1 18000000     mov     eax,dword ptr fs:\[18\]  
003824B4    8B40 30            mov     eax,dword ptr ds:\[eax+30\]  
003824B7    8B40 08            mov     eax,dword ptr ds:\[eax+8\]  
003824BA    8BF0               mov     esi,eax  
003824BC    8BC6               mov     eax,esi                            ;  
003824BE    8BD8               mov     ebx,eax                            ;  
003824C0    8BC8               mov     ecx,eax                            ;   
003824C2    8BD0               mov     edx,eax                            ;  
003824C4    8BF8               mov     edi,eax                            ;将基址送个个寄存器  
003824C6    66:8138 4D5A       cmp     word ptr ds:\[eax\],5A4D             ;查看是否动过手脚  
☆☆☆☆☆☆☆☆☆☆☆☆☆3824a0 中的东西  
003824A0    55                 push    ebp  
003824A1    8BEC               mov     ebp,esp                            ; 怎么好熟悉的代码？？？  
003824A3    83C4 F4            add     esp,-0C  
003824A6    60                 pushad                                     ; 保存现场  
003824A7    8B75 08            mov     esi,dword ptr ss:\[ebp+8\]           ; Kernel.dll 基址送 ESI  
003824AA    0BF6               or      esi,esi  
003824AC    75 0E              jnz     short 003824BC  
003824AE    64:A1 18000000     mov     eax,dword ptr fs:\[18\]  
003824B4    8B40 30            mov     eax,dword ptr ds:\[eax+30\]  
003824B7    8B40 08            mov     eax,dword ptr ds:\[eax+8\]  
003824BA    8BF0               mov     esi,eax                            ;   
003824BC    8BC6               mov     eax,esi                            ;   
003824BE    8BD8               mov     ebx,eax                            ;   
003824C0    8BC8               mov     ecx,eax                            ;   
003824C2    8BD0               mov     edx,eax                            ;   
003824C4    8BF8               mov     edi,eax                            ;   
003824C6    66:8138 4D5A       cmp     word ptr ds:\[eax\],5A4D  
003824CB    74 05              je      short 003824D2  
003824CD    E9 94000000        jmp     00382566  
003824D2    0349 3C            add     ecx,dword ptr ds:\[ecx+3C\]          ; 这里是 7c80000+E8=7c800e8(PE）?？？  
003824D5    8379 78 00         cmp     dword ptr ds:\[ecx+78\],0            ; 7c800160=262c(sub al,26)  
003824D9    75 05              jnz     short 003824E0  
003824DB    E9 86000000        jmp     00382566  
003824E0    0371 78            add     esi,dword ptr ds:\[ecx+78\]          ; =7c800262c  
003824E3    8975 F8            mov     dword ptr ss:\[ebp-8\],esi           ; 送堆栈 12ff88  
003824E6    8BC6               mov     eax,esi                            ; ax=7c80262c  
003824E8    0341 7C            add     eax,dword ptr ds:\[ecx+7C\]  
003824EB    8945 F4            mov     dword ptr ss:\[ebp-C\],eax           ; ax=7c8092a7 送堆栈 12ff84  
003824EE    8B45 08            mov     eax,dword ptr ss:\[ebp+8\]           ; 再次取基址  
003824F1    0346 1C            add     eax,dword ptr ds:\[esi+1C\]  
003824F4    8945 FC            mov     dword ptr ss:\[ebp-4\],eax           ; ax=7c802654 送堆栈 12ff8c  
003824F7    817D 0C 00000100   cmp     dword ptr ss:\[ebp+C\],10000         ; 这里为什么是 1000 比?我们先放心来  
003824FE    76 35              jbe     short 00382535  
00382500    8B4E 18                      mov     ecx,dword ptr ds:\[esi+18\]       ; 计数器，要比较 cx=3b5 次，以后进来这里根据情况的  
00382503    0356 24                      add     edx,dword ptr ds:\[esi+24\]       ; ＝7c8043fc  
00382506    037E 20                      add     edi,dword ptr ds:\[esi+20\]       ; ＝7c803528  
00382509    EB 1E                        jmp     short 00382529  
0038250B    8B07                         mov     eax,dword ptr ds:\[edi\]          ; eax＝4b73,Kernel.dll 的 IAT 首地址  
0038250D    0345 08                      add     eax,dword ptr ss:\[ebp+8\]        ; eax＝7c804b73(ActivateActCtx）计算了半天就是为了取这个的 ASCII 地址  
00382510    FF75 0C                      push    dword ptr ss:\[ebp+C\]            ; 将 Closehandle 的地址入栈  
00382513    50                           push    eax                             ; 将 ActivateActCtx 地址入栈，上面两个东西入栈就是为下面的 call 入栈  
00382514    E8 D1000000                  call    003825EA                        ; 进来就是找要找的东西，去下面######看代码  
00382519    0BC0                         or      eax,eax                         ; kernel32.CloseHandle  
0038251B    75 05                        jnz     short 00382522  
0038251D    0FB702                       movzx   eax,word ptr ds:\[edx\]  
00382520    EB 0B                        jmp     short 0038252D  
00382522    83C7 04                      add     edi,4                           ; EDI ＋4（下一个函数）  
00382525    83C2 02                      add     edx,2                           ; EDX ＋2  
00382528    49                           dec     ecx                             ; 计数器减一  
00382529    0BC9                         or      ecx,ecx  
0038252B  ^ 75 DE                        jnz     short 0038250B                  ; 找不到就继续回去找  
0038252D    0BC9                         or      ecx,ecx                         ; 一直找到自己要找的东西，就下来了  
0038252F    75 11                        jnz     short 00382542  
00382531    EB 33                        jmp     short 00382566  
00382533    EB 0D                        jmp     short 00382542  
00382535    8B45 0C                      mov     eax,dword ptr ss:\[ebp+C\]  
00382538    2B46 10                      sub     eax,dword ptr ds:\[esi+10\]  
0038253B    3B46 14                      cmp     eax,dword ptr ds:\[esi+14\]  
0038253E    76 02                        jbe     short 00382542  
00382540    EB 24                        jmp     short 00382566  
00382542    8B5D FC                      mov     ebx,dword ptr ss:\[ebp-4\]  
00382545    8B0483                       mov     eax,dword ptr ds:\[ebx+eax\*4\]    ; 9b77  
00382548    0345 08                      add     eax,dword ptr ss:\[ebp+8\]        ; 求出 CloseHandle 的入口地址  
0038254B    3B45 F8                      cmp     eax,dword ptr ss:\[ebp-8\]        ; IAT 上限  
0038254E    76 0B                        jbe     short 0038255B  
00382550    3B45 F4                      cmp     eax,dword ptr ss:\[ebp-C\]        ; IAT 下限  
00382553    73 06                        jnb     short 0038255B  
00382555    50                           push    eax  
00382556    E8 12000000                  call    0038256D                        ; 这里我就不进去看了，应该是越限后的处理  
0038255B    8945 FC                      mov     dword ptr ss:\[ebp-4\],eax        ; 将得出的地址入栈  
0038255E    61                           popad                                   ; 干完活了，我们恢复现场上馆子喽  
0038255F    8B45 FC                      mov     eax,dword ptr ss:\[ebp-4\]  
00382562    C9                           leave  
00382563    C2 0800                      retn    8  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
###############################call    003825EA#########################  
003825EA    55                 push    ebp  
003825EB    8BEC               mov     ebp,esp  
003825ED    83C4 FC            add     esp,-4                        ; 又看见老面孔了  
003825F0    60                 pushad  
003825F1    8B75 08            mov     esi,dword ptr ss:\[ebp+8\]      ; ActivateActCtx 的地址  
003825F4    8B7D 0C            mov     edi,dword ptr ss:\[ebp+C\]      ; Closehandle  
003825F7    33C0               xor     eax,eax  
003825F9    8945 FC            mov     dword ptr ss:\[ebp-4\],eax      ; 12ff50=0  
003825FC    AC                 lods    byte ptr ds:\[esi\]             ; 取 ActivateActCtx  
003825FD    3A07               cmp     al,byte ptr ds:\[edi\]          ; 和（Closehandle）作比较，呵呵  
003825FF    74 08              je      short 00382609  
00382601    33C0               xor     eax,eax  
00382603    48                 dec     eax  
00382604    8945 FC            mov     dword ptr ss:\[ebp-4\],eax      ; ffffff 送 12ff50  
00382607    EB 05              jmp     short 0038260E  
00382609    47                 inc     edi                           ; kernel32.7C803528  
0038260A    0AC0               or      al,al  
0038260C  ^ 75 EE              jnz     short 003825FC  
0038260E    61                 popad  
0038260F    8B45 FC            mov     eax,dword ptr ss:\[ebp-4\]      ; 抹掉痕迹（FFFFFF )  
00382612    C9                 leave  
00382613    C2 0800            retn    8  
00382616    33C0               xor     eax,eax  
00382618    C3                 retn  
###############################  
◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆◆  
进去后干什么呢呢?原来是拿着好东西,自己一个人找厨子下小灶呢!!!偷偷摸摸的真不体面  
厨子也狡猾,说,不行,我给你点油吧,你再找人家弄吧.只好又找前台小姐,漂亮 MM 说:请跟我来 ~~~:)  
结果小姐 mm 什么也不干,就带着一个包厢接着一个包厢转悠!!!从纽约帝国大赛楼顶磨磨蹭蹭才带到 YYY 层 XXXX 包厢  
,终于看见有和自己一样的东西了,心理只嘀咕,直接去不就得了,闹个大红脸,只好自己干活了.轻松一下,呵呵,看得累了  
胡诌一个笑话.  
  
00375810    0FB64B FF          movzx   ecx,byte ptr ds:\[ebx-1\]       ;  
00375814    8903               mov     dword ptr ds:\[ebx\],eax        ;  
00375816    03D9               add     ebx,ecx  
00375818    43                 inc     ebx                           ; 这里不一定，是乱序的要注意，看如何取  
00375819    59                 pop     ecx                           ; 计数器  
0037581A  ^ E2 A2              loopd   short 003757BE                ;没有传完 21 个函数就继续  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*21 个函数罗列如下\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
      函数名                            所取函数后所放地址  
 kernel32.CloseHandle                        382763  
 kernel32.CreateFileA                        382770  
 kernel32.CreatFileMapingA                   38277d  
 kernel32.CreateThread                       382791  
 kernel32.DeleteFileA                        38279f  
 kernel32.DeviceIoControl                    3827ac  
 kernel32.ExitProcess                        3827bd  
 kernel32.FindResourceA                      3827ca  
 kernel32.GetCommandLineA                    3827d9  
 kernel32.GetFileSize                        3827ea  
 kernel32.GetCurrentProcess                  3827f7  
 kernel32.GetCurrentProcessId                38280a  
 kernel32.GetCurrentThread                   38281f  
 kernel32.GetModuleHandleA                   382831  
 kernel32.GetModuleFileNameA                 382843  
 kernel32.GetTempPathA                       382857  
 kernel32.GetVersion                         382865  
 kernel32.lstrcmpA                           382871  
 kernel32.LoadResource                       38287b  
 kernel32.MapViewOfFile                      382889  
 kernel32.ReadProcessMemory                  382898  
 kernel32.ResetEvent                         3828ab  
 kernel32.SetEvent                           3828b7  
 ntdll.RtlSetLastWin32Error                  3828c1  
 kernel32.SetThreadPriority                  3828cf  
 kernel32.TerminateThread                    3828e2  
 kernel32.UnmapViewOfFile                    3828f3  
 kernel32.VirtualAllocEx                     382904  
 kernel32.VirtualFree                        382914  
 kernel32.VirtualProtect                     382921  
 kernel32.WaitForSingleObject                382931  
 kernel32.WriteProcessMemory                 382946  
 kernel32.WriteFile  :                       38295a       
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
0037581C    8DB5 900F4100      lea     esi,dword ptr ss:\[ebp+410F90\] ; 将 USER32.dll 字符传送给 ESI  
00375822    68 00FE98C7        push    C798FE00  
00375827    50                 push    eax                           ; WriteFile 函数地址入栈  
00375828    E8 5D000000        call    0037588A                      ; 这里是去调 USER32.dll  
  
  
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!call 37588a  
......  
00375879    8B0424             mov     eax,dword ptr ss:\[esp\]        ; 将 WriteFile 的地址取出来  
...  
00375971    56                 push    esi                           ; 将 User32.dll 字符的地址入栈  
0037597B    8D85 8F424000      lea     eax,dword ptr ss:\[ebp+40428F\] ;   
00375985    50                 push    eax                           ;取地址 3759a4 并入栈  
00375986    E8 04000000        call    0037598F                       ;去 GetModuleHandleA User32.dll  
  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@call 37598f  
00375992    8B85 0C0F4100      mov     eax,dword ptr ss:\[ebp+410F0C\]  ; 取 GetModuleHandleA 地址  
0037599E   /E9 C8C50000        jmp     00381F6B                       ;去 381f68 好像就是去 PUSH API RET  
..............这里就不下去了,见第三章  
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
003759A4    0BC0               or      eax,eax                        ;ax=0,,如果 GetModuleHandleA 失败的话就是 0，  
                                                                      ;说明内存中并没有加载相关的 DLL,而这里返回的是 0???  
                                                                      ;说明内存中没有这个,不信,你打开模块查看一下 :)  
◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎  
可执行模块  
基址         大小    （? 入口                                        名称     （系统）     文件版本                             
00400000     0007D000 (5  004670D1 xxxxx.<ModuleEntryPoint>              xxxxxx                    5.56W9                             
7C800000     0011C000 (1  7C80B436 kernel32.<ModuleEntryPoint>        kernel32 （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
7C920000     00094000 (6  7C933156 ntdll.<ModuleEntryPoint>           ntdll    （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎                                                                        
............  
003759A8    56                 push    esi                            ;user32.dll 字符地址入栈  
003759B2    8D85 C7424000      lea     eax,dword ptr ss:\[ebp+4042C7\]  ;   
003759BC    50                 push    eax                            ;将 3759dc 地址其实就是返回点  
003759BD    E8 04000000        call    003759C6                       ; 实际是 call LoadLibraryA User32.dll  
  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%call 37596c  
003759C9    8B85 100F4100      mov     eax,dword ptr ss:\[ebp+410F10\]  ; kernel32.LoadLibraryA  
003759D5   /E9 91C50000        jmp     00381F6B                        ;去 CALL API,不用追了直接到返回点下断  
  
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%  
003759DC    8BF0               mov     esi,eax                         ; 稀泥哗啦一堆 DLL 给加载了,关联的都给蹂到内存里面了  
                                                                       ;加载后返回地址送 ESI  
  
◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎  
可执行模块  
基址         大小    （? 入口                                        名称     （系统）            文件版本                             
00000000     0007D000 (5  004670D1 xxxxxxxx.<ModuleEntryPoint>       xxxxxx(不说软件名字了)     5.56W9                                
62C20000     00009000 (3  62C22EAD LPK.LpkDllInitialize               LPK      （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
73FA0000     0006B000 (4  73FDAEB6 USP10.<ModuleEntryPoint>           USP10    （系统）         1.0420.2600.2180 (xpsp\_sp2\_rtm.       
76300000     0001D000 (1  763012C0 IMM32.<ModuleEntryPoint>           IMM32    （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
77BE0000     00058000 (3  77BEF2A1 msvcrt.<ModuleEntryPoint>          msvcrt   （系统）         7.0.2600.2180 (xpsp\_sp2\_rtm.040      
77D10000     0008F000 (5  77D1F538 USER32.UserClientDllInitialize     USER32   （系统）         5.1.2600.2622 (xpsp\_sp2\_gdr.050        
77DA0000     000A9000 (6  77DA70D4 ADVAPI32.<ModuleEntryPoint>        ADVAPI32 （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040       
77E50000     00091000 (5  77E56284 RPCRT4.<ModuleEntryPoint>          RPCRT4   （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040       
77EF0000     00046000 (2  77EF63CA GDI32.<ModuleEntryPoint>           GDI32    （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
7C800000     0011C000 (1  7C80B436 kernel32.<ModuleEntryPoint>        kernel32 （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040        
7C920000     00094000 (6  7C933156 ntdll.<ModuleEntryPoint>           ntdll    （系统）         5.1.2600.2180 (xpsp\_sp2\_rtm.040  
◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎◎    
003759E4    6A 0A              push    0A                                                ;又要取 10 个函数  
003759E6    59                 pop     ecx                                               ;把给给 ECX  
003759E7    8DBD 31154100      lea     edi,dword ptr ss:\[ebp+411531\]                     ;传 WriteFile 地址  
003759ED    53                 push    ebx  
003759EE    9C                 pushfd                                                     ;保留现场  
003759EF    6A 03              push    3  
.......  
00375A1B    8A03               mov     al,byte ptr ds:\[ebx\]                               ;al=bc  
00375A1D    EB 06              jmp     short 00375A25                                       
00375A1F    F6D0               not     al                                                 ;net 解码  
00375A21    AA                 stos    byte ptr es:\[edi\]                                  ;  
00375A22    43                 inc     ebx  
00375A23    8A03               mov     al,byte ptr ds:\[ebx\]                               ; 取出源数据  
00375A25    0AC0               or      al,al                                              ;  
00375A27  ^ 75 F6              jnz     short 00375A1F                                     ;  
这些函数怎么看都和 Crack 有关  :)  
◇◇◇◇◇◇◇◇◇◇◇◇◇◇  
"CreateDialogIndirectParamA"  
"DialogBoxIndirectParamA"  
"EnableWindow"  
"EnumWindows"  
"GetForegroundWindow"  
"GetWindowTextA"  
"MessageBoxA"  
"SendMessageA"  
"SetTimer"  
"wsprintfA"  
◇◇◇◇◇◇◇◇◇◇◇◇◇◇  
....................  
00375A54    8B85 040F4100      mov     eax,dword ptr ss:\[ebp+410F04\]                       ;取返回地址=375a67  
00375A60   /E9 06C50000        jmp     00381F6B                                            ;不跟,这里是 CALL API  
\---------  
00375A67    0FB64B FF          movzx   ecx,byte ptr ds:\[ebx-1\]              ;cx=a  
00375A6B    8903               mov     dword ptr ds:\[ebx\],eax               ; USER32.wsprintfA 地址入 38274b  
00375A6D    03D9               add     ebx,ecx                              ;  
00375A6F    43                 inc     ebx  
00375A70    59                 pop     ecx  
00375A71    49                 dec     ecx  
00375A78    8DB5 4F124100      lea     esi,dword ptr ss:\[ebp+41124F\]         ;取 WS2 32.dll 地址  
............  
...............又开始代码压栈了........  
没有什么动作,是防止花指令  
00375BD7    8D85 EB444000      lea     eax,dword ptr ss:\[ebp+4044EB\]         ;取返回地址=375c00,下断这里  
                               jmp     00381F6B                              ;call api ,GetModuleFileNameA.WS\_32.dll  
...........................  
00375C00    0BC0               or      eax,eax                                ;ax=0,返回告诉我们,内存没有加载,下面估计要加载  
  
00375C0E    8D85 23454000      lea     eax,dword ptr ss:\[ebp+404523\]           ;取返回地址=375c38,下断这里  
00375C25    8B85 100F4100      mov     eax,dword ptr ss:\[ebp+410F10\]           ; kernel32.LoadLibraryA,果然要引导加载了  
  
可执行模块，项目 3  
 基址=71A20000  
 大小=00017000 (94208.)  
 入口=71A21273 WS2\_32.<ModuleEntryPoint>  
 名称=WS2\_32   （系统）  
 文件版本=5.1.2600.2180 (xpsp\_sp2\_rtm.040  
 路径=C:\\WINDOWS\\system32\\WS2\_32.DLL  
  
呵呵,加载成功了,  
00375C38    8BF0               mov     esi,eax                              ; WS2\_32.#382 这个是什么,没有符号,不知道什么函数  
00375C3A    8D9D 5B124100      lea     ebx,dword ptr ss:\[ebp+41125B\]        ;  
00375C40    B9 04000000        mov     ecx,4                                ;取这个函数中的 4 个函数  
00375C45    8DBD 31154100      lea     edi,dword ptr ss:\[ebp+411531\]         ;地址=00382C46, (ASCII "wsprintfA")  
00375C4B    53                 push    ebx                                    ;  
00375C4C    8A03               mov     al,byte ptr ds:\[ebx\]                   ;=71a200a8  
☆☆☆☆☆☆☆☆  
"WSASend"  
"WSARecv"  
"send"  
"recv"  
☆☆☆☆☆☆☆☆  
呵呵,怪不得这个壳外挂喜欢用,这些东西都在这里搞明堂,没法拦截包的,嘻嘻  
  
00375C6E    8D85 83454000      lea     eax,dword ptr ss:\[ebp+404583\]          ;取返回地址=375c98,下断这里  
                               jmp     00381F6B                               ;步骤同上,直接过  
...........  
00375C98    0FB64B FF          movzx   ecx,byte ptr ds:\[ebx-1\]              ;cx=5  
00375C9C    8903               mov     dword ptr ds:\[ebx\],eax               ; WS2\_32.recv 送 382988  
00375C9E    03D9               add     ebx,ecx                              ;  
00375CA0    43                 inc     ebx                                  ;  
00375CA1    59                 pop     ecx                                  ;  
00375CA2  ^ E2 A1              loopd   short 00375C45                       ;  
00375CA4    8DB5 78124100      lea     esi,dword ptr ss:\[ebp+411278\]        ;地址=0038298D, (ASCII "advapi32.dll")  
......................  
00375E03    8D85 17474000      lea     eax,dword ptr ss:\[ebp+404717\]         ;取返回地址=375c2c,下断这里  
00375E1A    8B85 0C0F4100      mov     eax,dword ptr ss:\[ebp+410F0C\]        ; kernel32.GetModuleHandleA,要取 advapi32.dll 的句柄了  
                               jmp     00381F6B                               ;步骤同上,直接过  
.....................  
00375E2C    0BC0               or      eax,eax                              ; ADVAPI32.77DA0000,返回句柄了,可以不用加载了  
00375E66    8D9D 86124100      lea     ebx,dword ptr ss:\[ebp+411286\]        ;  
00375E6C    6A 08              push    8                                    ;要取里面的 8 个函数  
00375E6E    59                 pop     ecx                                  ;   
00375E6F    8DBD 31154100      lea     edi,dword ptr ss:\[ebp+411531\]        ;地址=00382C46, (ASCII "recv"  
00375E75    53                 push    ebx                                  ;)  
00375E76    8A03               mov     al,byte ptr ds:\[ebx\]                 ;  
  
....................  
00375E98    8D85 AD474000      lea     eax,dword ptr ss:\[ebp+4047AD\]        ;取返回地址=375ec2,下断这里  
00375EBB   /E9 ABC00000        jmp     00381F6B                             ;步骤同上,直接过  
  
00375EC2    0FB64B FF          movzx   ecx,byte ptr ds:\[ebx-1\]  
00375EC6    8903               mov     dword ptr ds:\[ebx\],eax               ; ADVAPI32.CloseServiceHandle  
00375EC8    03D9               add     ebx,ecx  
00375ECA    43                 inc     ebx  
00375ECB    59                 pop     ecx  
  
☆☆☆☆☆☆☆☆☆☆☆☆8 个函数  
"CloseServiceHandle"  
"ControlService"  
 "CreateServiceA"  
 "DeleteService"  
 "OpenSCManagerA"  
 "OpenServiceA"  
 "QueryServiceStatus"  
 "StartServiceA"  
 ☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
   
 00375ED7    8D85 EB474000      lea     eax,dword ptr ss:\[ebp+4047EB\]    ;取返回地址=375f00,下断这里  
 00375EEE    8B85 EC0F4100      mov     eax,dword ptr ss:\[ebp+410FEC\]    ; USER32.GetForegroundWindow,哦来这个东西了,呵呵,看样子危险要来  
                                jmp     00381F6B                             ;步骤同上,直接过  
   
 ...............  
 00375F08    8985 D0164100      mov     dword ptr ss:\[ebp+4116D0\],eax    ;GetForegroundWindow 返回地址  
 ..............  
 00375F39    6A 01              push    0                                    ; ??????窗口的 Enable Windows??,这里是一个很厉害的  
                                                                             ;的陷阱,它将把你的键盘和屏幕,鼠标都锁死,呵呵,不要问我为什么知道的  
                                                                             ;我给锁过就知道这里了,喀嚓掉它,就是改成 1  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++=  
0012FF98   00375FC3  /CALL 到 EnableWindow  
0012FF9C   00C601C6  |hWnd = 00C601C6 ('flyODBG - xxxxxx.exe',class='fly\*OD\*',wndproc=053B3168)  
0012FFA0   00000001  \\Enable = TRUE                                          <<<<--------看见了么?如果是 0,祝贺你,你中标了  
++++++++++++++++++++++++++++++++++++++++++=++++++++++++++++  
  
  
  
小节,这里很枯燥,没有什么特别要说的就是反复从各个 DLL 把以后需要的函数弄出来,放到一个隐秘的地方,最后一个 ANTI\_DEBUG 有点阴  
呵呵,好了,这个有点郁闷,但是,有了这些,你不可能在这个软件上弄什么明堂,比如外挂的包你是不可能拦截了,为什么上面已经清清楚楚了  
  
真累,想歇息一下,空 2 天再发把,后面都是一些花样,ANTI-DEBUG 的东西,刚刚碰得时候,很恐怖,熟悉了也就没有什么了,  
花指令有点多,呵呵,希望大家能够喜欢我的文章,  
  
  
  
  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

【破解作者】 hnhuqiong  
【作者邮箱】 3KKK@  
【作者主页】 3KKK  
【使用工具】 FLYODBG  
【破解平台】 Win9x/NT/2000/XP  
【软件名称】 xxxxxxxxxx  
【下载地址】 xxxxxxxxxx  
【软件简介】 我们只是关心这个壳  
【加壳方式】 0.7x  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\--------------------------------------------------------------------------------  
【破解内容】  
  
  
第五章之  东邪西毒  
  
刀,又见宝刀  
但,只见刀影不见刀  
  
传说中有个人,有一把宝刀  
一直密而不宣  
江湖之人都为之胆寒  
  
传说中有个地方  
叫看雪山庄  
高手如云而都来去无踪  
  
江湖有人看见韦小宝检了一把刀  :)  
  
  
(从这里开始,你进入了核心,我们来论技术的,不是写小说的)  
00375FC3   /7C 05              jl      short 00375FCA                        ;EnableWindow 返回这里  
00375FC5   |EB 05              jmp     short 00375FCC                        ;那么你安全落地,开始往下走了,  
00375FC7   |90                 nop                                           ;我这里有点罗嗦,的又再次展开花指令模式  
00375FC8   |90                 nop                                           ;是为了说明一下,这里开始 ANTI 的手段不断  
00375FC9   |90                 nop                                           ;加强,而且中间不断的夹杂大量有用的指令  
00375FCA   \\90                 nop                                           ;高手出手常常是见刀不见人,不留神就中了  
00375FCB    90                 nop                                           ;毒,不禁想起了一个武侠中的门派----唐门  
00375FCC    8D85 E1484090      lea     eax,dword ptr ss:\[ebp+904048E1\]       ;唐门暗器独步天下,常常杀人以无形,而现在  
00375FD2    EB 02              jmp     short 00375FD6                        ;开始,我将一一展现这些,孙子云,兵者,诡道也!  
00375FD4    90                 nop                                           ;其实,作者有的时候,不应该在程序上有什么  
00375FD5    90                 nop                                           ;重点防护区,应该全程序平衡,否则,你下手越  
00375FD6    50                 push    eax                                   ;重,说明这个区域有重点的东西要出来,肯定会  
00375FD7    E8 04000000        call    00375FE0                              ;引起人家的注意,一旦小心,那么你的设计就  
00375FDC    90                 nop                                           ;成了此地无银了.   
00375FDD    90                 nop  
00375FDE    90                 nop                                           ;话说多了,375fcc 处取返回地址,并且下面将返回  
00375FDF    90                 nop                                           ;地址入栈 , 返回地址是:375ff6  
00375FE0    83C4 04            add     esp,4  
00375FE3    8B85 0A114100      mov     eax,dword ptr ss:\[ebp+41110A\]         ; GetCurrentThread 地址送 EAX,为 call 传送参数  
00375FE9    EB 04              jmp     short 00375FEF                        ；获取当前正在运行的线程，结果放入 EAX  
00375FEB    90                 nop  
00375FEC    90                 nop  
00375FED    90                 nop  
00375FEE    90                 nop  
00375FEF    E9 77BF0000        jmp     00381F6B                               ;去 PUSH API RET 是 call GetCurrentThread 了                                                                                          ;后面我将不再展开这类 call 模式  
..........................下面是设线程的  
00375FF6    6A 02              push    2                                      ;2 设定线程为极高级别  
00375FF8    50                 push    eax                                    ;GetCurrentThread 返回码 fffffe 入栈，准备下?                                                                                       ；函数来调用。  
00376002    8D85 17494000      lea     eax,dword ptr ss:\[ebp+404917\]          ;又取 call 返回地址:37602c  
0037600D    E8 04000000        call    00376016  
  
00376016    83C4 04            add     esp,4  
00376019    8B85 BA114100      mov     eax,dword ptr ss:\[ebp+4111BA\]         ; 取 SetThreadPriority 地址  
   
 00376025   /E9 41BF0000        jmp     00381F6B                             ;call  
  
..........................ANTI-DEBUG  
0037602C    50                 push    eax                                   ; ＝1  
0037602D    52                 push    edx                                   ; ＝7c92eb94  
0037602E    51                 push    ecx                                   ; ＝12ff7c 以上三个都入栈  
  
00376032    0F31               rdtsc                                         ; 取时间基准值  
00376036    E8 03000000        call    0037603E                              ;   
  
0037603E    83C4 04            add     esp,4                                 ;  
00376041    E8 38000000        call    0037607E                              ;  
  
00376081    68 C2100000        push    10C2                                  ;10C2 入栈  
0037608C    68 24080E68        push    680E0824                              ;代码入栈  
00376091    68 90908344        push    44839090                              ;代码入栈  
00376096  - FFE4               jmp     near esp                              ;去堆栈执行,上趟女厕所  
  
0012FF84    90                 nop  
0012FF85    90                 nop  
0012FF86    834424 08 0E       add     dword ptr ss:\[esp+8\],0E               ; 376099  
0012FF8B    68 99603700        push    376099                                ; 376099  
0012FF90    C2 1000            retn    10  
  
00376098    E8 C3E80300        call    003B4960  
  
0037604F    E8 46000000        call    0037609A  
  
0037609A    E8 03000000        call    003760A2                             ;一路花指令,这个花指令你要是打掉,就                                                                                               ;破坏了整个代码的观看的完整性  
003760A2    58                 pop     eax                                   ; 0012FF7C  
  
003760A6    83C0 07            add     eax,7  
003760A9    50                 push    eax      
003760AA    C3                 retn                                         ;花指令序列,将 call 堆栈平衡,37609a 的 call 平衡  
  
003760AD    C3                 retn                                          ;37604f call 平衡                             
00376060    E8 02000000        call    00376067                              ;看你有多少耐心  
  
00376067    0F31               rdtsc                                         ;再次取时间戳,  
00376069    83C4 04            add     esp,4                                 ;  
0037606C    2BC1               sub     eax,ecx                               ;和基准时间戳计算,看过去多少时间  
0037606E    3D 00000200        cmp     eax,20000                             ;对比计算结果,大于 20000,对不起,有人尾随我上女厕所              
00376073    76 04              jbe     short 00376079                        ;来人啦,抓流氓哟,这里必须改造成 jmp,安全了,                                                                                                         ;给小娘子带上口罩  
00376075    83C4 0C            add     esp,0C                                ;  
00376078    C3                 retn                                          ;  
00376079    59                 pop     ecx                                   ; 将 SetThreadPriority 的结果出堆栈  
0037607A    5A                 pop     edx                                   ;   
0037607B    58                 pop     eax                                   ;   
  
.。。。。。。。。。。。。。。。。  
003760AE    8B85 180F4100      mov     eax,dword ptr ss:\[ebp+410F18\]         ; 将 00400000 取出来  
003760B4    0340 3C            add     eax,dword ptr ds:\[eax+3C\]              ；＝4000e8  
003760B7    68 00FE98C7        push    C798FE00                               ；  
003760BC    50                 push    eax                                    ; 这里入栈估计有明堂  
  
003760BD    E8 5D000000        call    0037611F                               ？  
  
0037612A    FF30               push    dword ptr ds:\[eax\]                     ;又去堆栈了  
  
0037612C    E8 C9000000        call    003761FA                               ；去堆栈执行代码  
  
00376206    8B40 50            mov     eax,dword ptr ds:\[eax+50\]            ;7ccfa  
00376209    3385 2C0F4100      xor     eax,dword ptr ss:\[ebp+410F2C\]          ;eax=35ff6c  
  
00376215    E8 5D000000        call    00376277                               ;去堆栈执行代码  
  
0037635E    8B8D 080F4100      mov     ecx,dword ptr ss:\[ebp+410F08\]          ;ecx=35ff6cc  
00376364    3BC1               cmp     eax,ecx                                ;  
00376366    8DB5 2F144100      lea     esi,dword ptr ss:\[ebp+41142F\]          ;  
0037636C    46                 inc     esi                                   ; =382b45  
0037636D    68 00FE9A28        push    289AFE00                              ;  
00376372    50                 push    eax  
00376373    E8 5D000000        call    003763D5                              ;??????去堆栈  
  
●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●DEBUG 文件检查●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●  
003764BC    6A 09              push    9  
003764BE    59                 pop     ecx                                   ; cx=9,这里一共要判断 9 个 DEBUG 程序  
003764C6    AC                 lods    byte ptr ds:\[esi\]             ; 将 382b45 处的字符送 AL  
  
003764C9    F6D0               not     al                            ; not al 解码  
003764CB    8846 FF            mov     byte ptr ds:\[esi-1\],al        ; 解码后放回  
003764CE    AC                 lods    byte ptr ds:\[esi\]             ; 取 ESI 处的字符  
003764CF    0AC0               or      al,al  
003764D1  ^ 75 F6              jnz     short 003764C9                ;译码完毕后为\\\\.\\NTICE  
  
003764D3    5E                 pop     esi  
003764D4    6A 00              push    0  
003764D6    68 80000000        push    80  
003764DB    6A 03              push    3  
003764DD    6A 00              push    0  
003764DF    6A 03              push    3  
003764E1    68 000000C0        push    C0000000                      ; 以上入堆栈的是 CreateFileA 的传入参数  
003764E6    9C                 pushfd                                ;保留标志寄存器  
003764E7    6A 03              push    3                             ;  
003764EF    E8 06000000        call    003764FA                      ;  
  
0037651D    8D85 314E4000      lea     eax,dword ptr ss:\[ebp+404E31\]         ;返回地址,376546  
00376534    8B85 5B104100      mov     eax,dword ptr ss:\[ebp+41105B\]        ; kernel32.CreateFileA,  
00376540   /E9 26BA0000        jmp     00381F6B                              ;call CreateFileA  
  
  
00376581    0F31               rdtsc  
00376583    83C4 04            add     esp,4  
00376586    2BC1               sub     eax,ecx  
00376588    3D 00000200        cmp     eax,20000  
0037658D    76 04              jbe     short 00376593                       ; 再次进行 rdtsc 的 ANTI-DEBUG  
  
003765C8    83F8 FF            cmp     eax,-1                               ; 判断 SICE 是否存在  
003765CB   /74 05              je      short 003765D2                       ;有就见上帝了 :)  
  
003765D2    56                 push    esi  
003765D3    AC                 lods    byte ptr ds:\[esi\]  
003765D4    EB 06              jmp     short 003765DC  
003765D6    F6D0               not     al                                   ; 习惯真好,把\\\\.\\NTICE 痕迹抹去  
003765D8    8846 FF            mov     byte ptr ds:\[esi-1\],al  
003765DB    AC                 lods    byte ptr ds:\[esi\]  
003765DC    0AC0               or      al,al  
003765DE  ^ 75 F6              jnz     short 003765D6  
  
  -------中间还有点 rdtsc 的 AnTI-debug  
    
0037666C    0BC9               or      ecx,ecx                             ;一直检查 9 个文件  
\\\\.\\NTICE  
\\\\.\\SICE  
\\\\.\\TWX2002  
\\\\.\\filemon  
\\\\.\\regmon  
\\\\.\\FILEVXD  
\\\\.\\REGVXD  
\\\\.\\ICEDUMP  
\\\\.\\BW2K  
0037666E  ^ 0F85 50FEFFFF      jnz     003764C4                            ;对 DEBUG 程序检查完下去  
..●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●  
  
.........  
00376E86    90                 nop  
00376E87    0F31               rdtsc                                       ;再次利用 rdtsc 检查,绕过  
00376E89    8BC8               mov     ecx,eax  
.....  
  
00376F03    E8 0A000000        call    00376F12                             ;  
00376F1B    8D85 2F584000      lea     eax,dword ptr ss:\[ebp+40582F\]        ;返回地址  
  
00376F26    E8 04000000        call    00376F2F                             ;这里进去是取 NTDLL.DLL 的句柄  
.                                                                           ;我这里返回为 0  
00376F4A    E8 5D000000        call    00376FAC                             ;  
003770B9    8D85 CD594000      lea     eax,dword ptr ss:\[ebp+4059CD\]         ;返回 3770e2  
  
.........这里都是重复劳动,一直到下面,是取线程,为 ZwSetInformationThread 做准备  
.........注意一下 rdtsc 的陷阱就行,没有什么好论述的  
  
003772E7    6A 01              push    0                                    ; 这里用了一个狠招,就是 ZwSetInformationThread  
003772E9    6A 00              push    0                                    ;这个函数是用这个函数可以将某个线程的调试端口设为 0，使                                                                            ;得 Win32 调试器无法再收到该线程的调试事件，使调试器无法  
                                                                            ;再调试该线程。这个主要是针对 ring3 调试器的  
003772EB    6A 11              push    11                                   ;所以,很多初学者直接看见自己调试器挂掉成白的,就是被线程  
003772ED    50                 push    eax                                  ; 把调试器和程序脱钩了.这里简单,把 PUSH 0 改成 1 就行了,有的  
003772EE    8D85 E85B4000      lea     eax,dword ptr ss:\[ebp+405BE8\]        ;朋友也可以直接 NOP 掉 call ,呵呵,也行吧,注意现场不乱就行  
003772F4    50                 push    eax                                  ;   
003772F5    8BC7               mov     eax,edi                              ;   
003772F7    E9 6FAC0000        jmp     00381F6B                             ;call ,返回地址是 3772fd  
  
\------------------------------------------------------------------------------  
003772FD    8D1D D4174100      lea     ebx,dword ptr ds:\[4117D4\]            ; 4117d4  
00377303    833C2B 00          cmp     dword ptr ds:\[ebx+ebp\],0             ;  
  
00377362    E8 EEFFFFFF        call    00377355  
  
0037733F    8D042B             lea     eax,dword ptr ds:\[ebx+ebp\]            ;382ee9,  
  
0037736D    8B48 08            mov     ecx,dword ptr ds:\[eax+8\]              ;解码大小 24200  
  
00377398    8B70 04            mov     esi,dword ptr ds:\[eax+4\]              ;偏移地址 1000  
  
003773C8    03B5 180F4100      add     esi,dword ptr ss:\[ebp+410F18\]        ; 解码地址 401000  
  
00377427    8D85 EE144100      lea     eax,dword ptr ss:\[ebp+4114EE\]         ;382c03 解码钥匙地址  
  
003774AB    56                 push    esi                                  ; 00401000 入栈  
  
003774D9    E8 D79F0000        call    003814B5                             ; ,解压.可以不看,直接过+号  
  
++++++++++++++++++++++++++++++++call 3814b5+++++++++++++++++++++++++++++++++  
003814B5    55                 push    ebp                                  ; fff71715  
003814B6    8BEC               mov     ebp,esp                              ; 12ff94=3774de;返回地址  
003814B8    83C4 FC            add     esp,-4  
003814BB    60                 pushad                                       ; 保存现场  
003814BC    8B7D 10            mov     edi,dword ptr ss:\[ebp+10\]            ; 382c03  
003814BF    8B75 08            mov     esi,dword ptr ss:\[ebp+8\]             ; 401000  
003814C2    8B5D 0C            mov     ebx,dword ptr ss:\[ebp+C\]             ; 24200  
003814C5    D1EB               shr     ebx,1                                ; /2=12100 解码以 WORD 为长度  
003814C7    EB 41              jmp     short 0038150A  
003814C9    53                 push    ebx  
003814CA    33DB               xor     ebx,ebx  
003814CC    66:C745 FE 0000    mov     word ptr ss:\[ebp-2\],0  
003814D2    EB 25              jmp     short 003814F9  
003814D4    66:8B16            mov     dx,word ptr ds:\[esi\]                 ; 401000(f6b8)->dx  
003814D7    66:23145F          and     dx,word ptr ds:\[edi+ebx\*2\]           ; dx and 97da=9698  
003814DB    B9 10000000        mov     ecx,10                               ; 循环 10 次  
003814E0    33C0               xor     eax,eax  
003814E2    66:D1E2            shl     dx,1                                 ; shl dx,1  
003814E5    73 01              jnb     short 003814E8  
003814E7    40                 inc     eax  
003814E8  ^ E2 F8              loopd   short 003814E2  
003814EA    66:83E0 01         and     ax,1                                 ; and ax,1  
003814EE    66:8BCB            mov     cx,bx  
003814F1    66:D3E0            shl     ax,cl                                ;shl ax,cl  
003814F4    66:0145 FE         add     word ptr ss:\[ebp-2\],ax  
003814F8    43                 inc     ebx  
003814F9    83FB 10            cmp     ebx,10                               ; 以 20 个字节作为解码钥匙  
(DA 97 F1 53 93 2C 02 EA 4C 43 49 6B 80 4F 3D 74 14 A5 30 24  解码钥匙)  
003814FC  ^ 72 D6              jb      short 003814D4  
003814FE    5B                 pop     ebx                                  ; jl.00401000  
003814FF    66:8B45 FE         mov     ax,word ptr ss:\[ebp-2\]               ; 解码后的数据放 ax  
00381503    66:8906            mov     word ptr ds:\[esi\],ax                  ;解码后的数据放回原地址  
00381506    83C6 02            add     esi,2  
00381509    4B                 dec     ebx  
0038150A    0BDB               or      ebx,ebx  
0038150C  ^ 75 BB              jnz     short 003814C9                       ; 解码如果没有完成就继续转  
0038150E    61                 popad  
0038150F    C9                 leave  
00381510    C2 0C00            retn    0C  
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  
  
0037752A    51                 push    ecx                                     ;24200 解码长度入栈  
00377536    6A 04              push    4                                         
00377561    68 00100000        push    1000                                    ;一看到上面数据,要开缓冲区了  
0037758B    FF342B             push    dword ptr ds:\[ebx+ebp\]                  ;大小 5c000,申请空间  
0037758B    FF342B             push    dword ptr ds:\[ebx+ebp\]  
0037758E    6A 00              push    0  
00377590    8D85 FE634000      lea     eax,dword ptr ss:\[ebp+4063FE\]        ; 377b13  
转堆栈去隐藏一下  
003776E6    8B85 140F4100      mov     eax,dword ptr ss:\[ebp+410F14\]        ; kernel32.VirtualAlloc,呵呵  
..........去申请了,960000  
  
00377B1A    5B                 pop     ebx                                  ;   
00377B47    8BF0               mov     esi,eax                              ;解码后所放地址 960000  
00377B71    8BC3               mov     eax,ebx                              ;   
00377B9E    03C5               add     eax,ebp                              ;  
00377BC9    8B78 04            mov     edi,dword ptr ds:\[eax+4\]             ;1000  偏移地址  
00377BF7    03BD 180F4100      add     edi,dword ptr ss:\[ebp+410F18\]        ; =00401000  得出解码源地址  
00377C8A    8D85 9E654000      lea     eax,dword ptr ss:\[ebp+40659E\]         ;377cb3  
00377CA1    8B85 840F4100      mov     eax,dword ptr ss:\[ebp+410F84\]        ; 467151,这个好像是某个压缩引擎的地址也!  
00377CAD   /E9 B9A20000        jmp     00381F6B                             ;去解码啦,BT,一个位置上 2 次加密  
  
呕呕也也,呕啦啦,呕咧咧,我爬,我上,我纹丝不动,我乾坤大法力,去 960000 看看吧,好东西出来了哟  
  
00377D34    51                 push    ecx                                               ; 5c000  
00377D35    C1E9 02            shr     ecx,2                                             ; /4  
00377D38    F3:A5              rep     movs dword ptr es:\[edi\],dword ptr ds:\[esi\]        ; 960000-->401000,以 DWORD 方式放回  
00377D3A    59                 pop     ecx                                               ; 00960000  
00377D3B    83E1 03            and     ecx,3                                             ;and 3??==0 呀,晕死  
00377D3E    F3:A4              rep     movs byte ptr es:\[edi\],byte ptr ds:\[esi\]          ;传个大头鬼,帮 KKK 抓个 BUG  
  
00377DC0    68 00800000        push    8000                                               ;8000  
00377DF0    6A 00              push    0                                                 ; 0  
00377E20    56                 push    esi                                                ;96000,呵呵,一看就知道要释放内存了  
00377E53    8D85 68674000      lea     eax,dword ptr ss:\[ebp+406768\]                      ;返回地址 377e7d  
00377E6A    8B85 FF114100      mov     eax,dword ptr ss:\[ebp+4111FF\]           ; kernel32.VirtualFree  
00377E76   /E9 F0A00000        jmp     00381F6B                                ;call VirtualFree  
  
00377ED7    83C3 0C            add     ebx,0C                                  ;4117e0  
00377303    833C2B 00          cmp     dword ptr ds:\[ebx+ebp\],0                ; '9330  
0037733F    8D042B             lea     eax,dword ptr ds:\[ebx+ebp\]              ; 382ef5  
0037736D    8B48 08            mov     ecx,dword ptr ds:\[eax+8\]                ;2998 ,解码长度  
00377398    8B70 04            mov     esi,dword ptr ds:\[eax+4\]                ;5dcd0,偏移地址  
003773C8    03B5 180F4100      add     esi,dword ptr ss:\[ebp+410F18\]           ; 00400000+5dcd0  
003773FB    8BFE               mov     edi,esi                                 ; 0045DCD0,解码首地址  
  
0037743C  ^\\7F E9              jg      short 00377427                          ;回去再次解码一次  
00377427    8D85 EE144100      lea     eax,dword ptr ss:\[ebp+4114EE\]           ;又取钥匙地址了,382c03  
...........  
其中又分配缓冲区地址 3f0000,解码首地址在上面 45dcd0,二次解码.  
...........  
00377ED7    83C3 0C            add     ebx,0C  
00377F03    8DB5 BD694000      lea     esi,dword ptr ss:\[ebp+4069BD\]           ;返回地址  
...........又去堆栈玩了一趟  
00377F60    8B0424             mov     eax,dword ptr ss:\[esp\]                   ;1  
00378058    87E6               xchg    esi,esp                                       ; ;3780d2<-->12ffa4  
0037805A    B9 927D0000        mov     ecx,7D92                                      ; 计数器=7d9c  
  
0037805F    58                 pop     eax                                           ; f0b77a74  
00378060   /EB 04              jmp     short 00378066  
00378066   \\F6D0               not     al                                            ; not al  
00378068    E8 03000000        call    00378070  
00378070    83C4 04            add     esp,4  
00378073    50                 push    eax  
00378074    EB 04              jmp     short 0037807A  
0037807A    44                 inc     esp                                           ; inc esp  
0037807B    EB 03              jmp     short 00378080  
00378080  ^ E2 DD              loopd   short 0037805F                                ;最后计算 eax=78858BF8  
  
00378082    87E6               xchg    esi,esp                                       ; 12ffa4<==>37fe64  
00378084    6A 04              push    4                                             ;又要申请内存了  
00378086    68 00100000        push    1000  
0037808B    68 00200000        push    2000  
00378090    6A 00              push    0  
0037809B    8D85 B0694000      lea     eax,dword ptr ss:\[ebp+4069B0\]                 ;3780c5 返回地址  
003780B2    8B85 140F4100      mov     eax,dword ptr ss:\[ebp+410F14\]                 ; kernel32.VirtualAlloc  
003780BE   /E9 A89E0000        jmp     00381F6B                                      ;call VirtualAlloc  
...  
  
003780C5    8985 5C184100      mov     dword ptr ss:\[ebp+41185C\],eax                 ; 分配内存地址为 3f0000  
  
\--------------------------------------------------------------------------------  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

鉴于和作者沟通,由于一款好壳是大量心血的结晶,为防止用心不良者,本人后面文章将简单的论述,不再进行仔细分解,请大家谅解.     ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) ![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) 4dd466d0e1e0cc123afbe504da2462ca