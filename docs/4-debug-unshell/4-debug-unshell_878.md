

# 纯脆灌水

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-461.htm`](https://www.pediy.com/kssd/pediy09/pediy09-461.htm)

> 我刚看到原来论坛有大虾已经把 PE-LOCK1.0X  
> 本来不想发这篇文章，想想既然写了 就发了算了  
>   
> 这个软件是个国内有名的木马，不敢说它的名字，为了逆向学习它的内容  
> （不是为了搞破坏） 我就试着脱了一下它的壳  
>   
>   
> 找 OEP：  
> 我用的是比较原始的方法，在最后一次异常后，单步跟，最后找到  
> fake oep: 005c694d  
> 补上偷的部分在：  
> 005c692d:                  
> push ebp  
> mov ebp,esp  
> add esp,-10  
> push ebx  
> push esi  
> jmp  005c694d  
> 根据后面的单步可以判断是 DELPHI   
>   
> 修补 IAT：  
> crtl+f2  
> 在\[5D4328\]下内存写入断点  
> 边 SHIFT+F9 边看 找出哪个异常是用来添充 IAT 的  
> 第一次中断在第 7 个异常  但只是中间解码  
> 去掉内存断点  跳出循环  再下一次  
> 边 SHIFT+F9 边看 到倒数第 2 个循环被断下  
> 去掉断点  单步  
> 这时还不是  重复刚才的操作  (可以根据观察内容判断)  
> 一直来到  
> 00374661   mov  \[ecx\],ebx  ;ebx=HOOK 的地址  
> 呵呵  就在这下手吧  观察到此时 EAX=要 HOOK 的 API 地址  
> 用脚本把这一句改为：  
> mov  \[ecx\],eax  
> 我的方法比较笨(这也没办法,主要是人笨),把 HOOK 的地址改了  
> 注意这有校验,你改完，要再改回来  
> 校验在：  
> 003741c4:      test eax,eax  
>                      je 00375c12  
>                      mov edx,eax  
>                      mov esi,edi  
>                      mov al,\[esi\]   ;esi=ASCII"kernel32.dll"  
>                      inc esi  
>                      test al,al  
>                      jnz 00374293  
>                      mov eax,\[esi+1\]  
>                      mov ecx,6b3  
>                      sub eax,\[ebp+ecx\*4+3b14\]  ;eax=像密文似的东东(这是那个校验和)  
>                      rol eax,cl  
>                      dec ecx  
>                      jnz 00374351  
>                      mov cl,\[esi\]   ;esi=008c000d  
>                      add esi,5  
>                      push ecx  
>                      push eax  
>                      push edx       ;edx=7c800000  
>                      push dword ptr\[ebp\]  ;ebp=0037062d  
>                      call 00370005    ;这是处理 DLL 的地方  
> 说说壳 HOOK 的办法，总共有 8 种加花指令方式(这的各种方式不是独立的，还有各种组合)  
> 我跟了第一个 API 的  
> 处理 2:  
> mov al,68(push 的机器码)  
> stosb   (edi=hook 地址)  
> push edi  
> stosd     
> mov dl,3  
> 跳回处理的地方因为加了 dl=3 所以跳到处理 3  
> 处理 3:  
> mov ah,1  
> stosw  
> mov al,bl  
> stosb  
> mov al,0c3 (retn)  
> mov ah,dl  
> stosw  
> pop eax(eax=处理 2 中没处理好的地方)  
> mov \[eax\],edi  
> test al,1  
> je 0037524c  
> 以上是第一个 API 的 HOOK 地址的前奏-----花指令  
> 然后就要 STOLEN API 了  用到如下算法  
> 用来确定一条 OPCODE 的长度:  
> 0037501d:  mov eax,\[esi\]  ;esi=api 地址  
>            mov edx,eax  
>            cmp al,0cc    
>            jnz 00375054  
> 0037508d   cmp al,0eb(判断是否是短 JMP)跳转 1  
>            je  00375149  
>            and ah,0f0(将 AH 低 4 位清零)  
>            cmp ax,800f(判断是否为长 je jne 之类)  
>            je  00375149  
>            and al,0fe(将 AL 最低位清，为了与 JMP 区别)  
>            cmp al,0e8(判断是否为长 CALL)  
>            je  00375149            
>            cmp al,0c2(判断是否为 retn+参数)  
>            and al,0fch(将 AL 低 2 位清零)  
>            cmp al,0e0(判断是否为  
>            je  00375149  
>            and al,0f0(将 AL 低 4 位清零)  
>            cmp al,70h(判断是否为  
>            je  00375149  
>            xchg eax,edx(将变完型的取回)  
>            and ax,38ff(将 ah 的高 6 位清零，将 AH 的低 3 位清零)  
>            cmp ax,10ff(判断是否为 call \[\*\*\*\*\*\*\*\*\]形式)  
>            je  00375149  
>            cmp ax,20ff(没查到是什么)  
>            je  00375149  
>            push esi(最普通的 OPCODE)  
>            call 00375310  
> 如果是最普通的代码就 STOLEN，前面的为特殊的，STOLEN 完就不再 STOLEN 了  
> 普通处理的代码，我跟了，是用一个大的数组存相应的数据  
> 而那些数据决定了代码的走向  
> 表如下：  
> 因为我对机器码和汇编的对应关系不熟，就先不现丑分析了  
>   
> 0037531A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037532A  00 80 00 00 00 80 00 00 00 00 00 00 00 00 00 00  .?..?.........  
> 0037533A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037534A  00 80 00 00 00 80 00 00 00 00 00 00 00 00 00 00  .?..?.........  
> 0037535A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037536A  00 80 00 00 00 80 00 00 00 00 00 00 00 00 00 00  .?..?.........  
> 0037537A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037538A  00 80 00 00 00 80 00 00 00 00 00 00 00 00 00 00  .?..?.........  
> 0037539A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 003753AA  00 80 00 00 00 80 00 00 08 00 00 00 00 00 00 00  .?..?........  
> 003753BA  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 003753CA  00 80 00 00 00 80 00 00 08 00 00 00 00 00 00 00  .?..?........  
> 003753DA  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 003753EA  00 80 00 00 00 80 00 00 08 00 00 00 00 00 00 00  .?..?........  
> 003753FA  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037540A  00 80 00 00 00 80 00 00 08 00 00 00 00 00 00 00  .?..?........  
> 0037541A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037542A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037543A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037544A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037545A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037546A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037547A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037548A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037549A  00 00 00 00 00 00 00 00 00 40 00 00 00 40 00 00  .........@...@..  
> 003754AA  08 00 00 00 08 00 00 00 08 10 00 00 18 00 00 00  ...........  
> 003754BA  00 20 00 00 00 60 00 00 00 01 00 00 00 41 00 00  . ...\`......A..  
> 003754CA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 003754DA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003754EA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003754FA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 0037550A  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 0037551A  00 41 00 00 00 60 00 00 00 41 00 00 00 41 00 00  .A...\`...A...A..  
> 0037552A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037553A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037554A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037555A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037556A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037557A  00 00 00 00 00 00 00 00 02 20 00 00 00 00 00 00  ........ ......  
> 0037558A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037559A  20 00 00 00 20 00 00 00 20 00 00 00 20 00 00 00   ... ... ... ...  
> 003755AA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 003755BA  00 01 00 00 00 20 00 00 00 00 00 00 00 00 00 00  .... ..........  
> 003755CA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 003755DA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003755EA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003755FA  00 20 00 00 00 20 00 00 00 20 00 00 00 20 00 00  . ... ... ... ..  
> 0037560A  00 20 00 00 00 20 00 00 00 20 00 00 00 20 00 00  . ... ... ... ..  
> 0037561A  00 41 00 00 00 41 00 00 00 02 00 00 00 00 00 00  .A...A.........  
> 0037562A  00 40 00 00 00 40 00 00 00 41 00 00 00 60 00 00  .@...@...A...\`..  
> 0037563A  00 03 00 00 00 00 00 00 00 02 00 00 00 00 00 00  ..............  
> 0037564A  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037565A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037566A  00 01 00 00 00 01 00 00 00 00 00 00 00 00 00 00  ..............  
> 0037567A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037568A  00 40 00 00 00 40 00 00 00 40 00 00 00 40 00 00  .@...@...@...@..  
> 0037569A  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003756AA  00 01 00 00 00 01 00 00 00 01 00 00 00 01 00 00  ............  
> 003756BA  00 20 00 00 00 20 00 00 02 20 00 00 00 01 00 00  . ... .. .....  
> 003756CA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 003756DA  08 00 00 00 00 00 00 00 08 00 00 00 08 00 00 00  .............  
> 003756EA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 003756FA  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
> 0037570A  00 00 00 00 00 00 00 00 00 40 00 00 00 40 00 00  .........@...@..  
>   
> 表结束  
>   
> 如：  
> 取第一个字节，如果是 0C  就将其乘 4，在表中找到那个对应的数字，然后将其送给 DL 中，再判断来选择流程  
>   
> 脱壳后的修复:  
> 要改的 396c08 为写入的地址  
> 00396838  mov byte ptr\[esi\],dl  ;esi=00396c08  
> 还是脚本，很简单  
> 代码还要用到壳中的内容,所以只要把 VIRTUALALLOC 申请的那部分补在原程序中就行了  
>   
> DUMP 后就可以了  
> 资源靠那个工具修，而减肥就得看看 CcDebuger 兄的那篇文章了  
>   
> 脚本在附件中，我的脚本有局限性，代码写的也很差，因为是第一个脚本  
> 大家就凑合着看吧 

> 昨天脚本没贴上   
> 这个脚本有局限性，主要是我在动态申请的空间下了断  
> 这个得随时变一下  
> var count  
> run   
> mov count,0  
> abl:  
> cmp count,13  
> je abl0  
> esto  
> inc count  
> jmp abl  
> abl0:  
> coe  
> abl1:  
> bp 00390005  
> esto  
> bc 00390005  
> bp 00394661  
> run    
> bc 00394661  
> asm 00394661,"mov \[ecx\],eax"  
> sti  
> asm 00394661,"mov \[ecx\],ebx"  
> cmp ecx,005d4ca0  
> je  done  
> jmp abl1  
> done:  
> bp 7c809ab4  
> run  
> cmp eax,10c0000  
> je abl3  
> jmp abl2  
> abl3:  
> bc 7c809ab4  
> bp 003982f7  
> run  
> bc 003982f7  
> mov edi,005cd5c0  
> bp 005c694d  
> esto  
> esto  
> bc 005c694d  
> mov eip,005d9030  
> asm  005d9030,"pushad"  
> asm  005d9031,"lea esi,\[00396c08\]"  
> asm  005d9037,"lea edi,\[005d9050\]"  
> asm  005d903d,"mov ecx,2000"  
> asm  005d9043,"rep movsb"  
> asm  005d9045,"popad"  
> asm  005d9046,"jmp 005c694d"  
> bp 00406d89  
> run  
> bc 00406d89  
> pause  
>   
> 还得补充两句，在用 ImportREC 修复时要把每个 DLL 后清零  
> 还要手动 DUMP，DUMP 时还要注意先修整 PE 校验和，然后就可以了 

> 下面这个脚本是我写的  
> 只修复了 ITA 和显示 要转存的 壳转移区段  
> 也就是 转移的代码我还没让他 写回来，还有就是 Stolen Code 还没完善！  
> 这个是我以前分析的果果  
>   
> 00DD7890    6A 04           push    4  
> 00DD7892    68 00300000     push    3000  
> 00DD7897    68 28220000     push    2228  
> 00DD789C    6A 00           push    0  
> 00DD789E    FF55 04         call    dword ptr \[ebp+4\]                   ; 获取申请到的空间地址  
> 00DD78A1    97              xchg    eax, edi                            ; 把申请到的地址 给 EDI  
> 00DD78A2    E8 E1FFFFFF     call    00DD7888  
> 00DD78A7    8D75 FB         lea     esi, dword ptr \[ebp-5\]  
> 00DD78AA    81EE 45040000   sub     esi, 445  
> 00DD78B0    56              push    esi  
> 00DD78B1    BB 53000000     mov     ebx, 53  
> 00DD78B6    3017            xor     byte ptr \[edi\], dl                  ; 将要写壳中代码的地址 与 上一次壳跳向源程序 "JMP XXX" 的操作数 异或  
> 00DD78B8    47              inc     edi                                 ; 指向下一个 壳转移 指针  
> 00DD78B9    8B16            mov     edx, dword ptr \[esi\]                ; 修改 JMP 到 壳中代码地址  
> 00DD78BB    83C6 04         add     esi, 4                              ; 指向  壳代码 将要 从 某地方拷贝过来的数据长度  
> 00DD78BE    C602 E9         mov     byte ptr \[edx\], 0E9                 ; 可以理解代码段的写入 JMP 指令  
> 00DD78C1    8BC7            mov     eax, edi                            ; 将要写转移壳代码地址 -> EAX  
> 00DD78C3    2BC2            sub     eax, edx                            ; 求 原代码 到 壳中代码的 偏移  
> 00DD78C5    83E8 05         sub     eax, 5                              ; 减去 "JMP XXX" 长度(纠正去壳地址)  
> 00DD78C8    8942 01         mov     dword ptr \[edx+1\], eax              ; 在 JMP 指令后面写入 操作数  
> 00DD78CB    8A06            mov     al, byte ptr \[esi\]                  ; 将要复制的长度 给 al  
> 00DD78CD    46              inc     esi                                 ; 指向要拷贝的数据  
> 00DD78CE    0FB6C8          movzx   ecx, al                             ; 将要 拷贝到壳代码的数据总长度  
> 00DD78D1    83E0 03         and     eax, 3  
> 00DD78D4    C1E9 02         shr     ecx, 2                              ; ecx 右移 2 位  = int(ecx/2²)  
> 00DD78D7    F3:A5           rep     movs dword ptr es:\[edi\], dword ptr >; 将 指向的复制数据 写到 壳转移代码  
> 00DD78D9    8BC8            mov     ecx, eax                            ; 上面类似于 取操作码  
> 00DD78DB    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr \[e>; 类似取 操作数  
> 00DD78DD    8A06            mov     al, byte ptr \[esi\]                  ; 取 上 面 代码的长度  
> 00DD78DF    46              inc     esi                                 ; 指向下一次 要复制 到 壳中的代码  
> 00DD78E0    03D0            add     edx, eax                            ; 准备在壳转移代码中 写 JMP 返回的地址  
> 00DD78E2    C607 E9         mov     byte ptr \[edi\], 0E9                 ; 写 JMP 指令  
> 00DD78E5    2BD7            sub     edx, edi                            ; 求 壳代码 到 原代码的 偏移  
> 00DD78E7    83EA 05         sub     edx, 5                              ; 减去 "JMP XXX" 长度(纠正返回地址)  
> 00DD78EA    8957 01         mov     dword ptr \[edi+1\], edx              ; 在 JMP 指令后面写入 操作数  
> 00DD78ED    83C7 05         add     edi, 5                              ; EDI 指向 下一个 写壳中代码的指针  
> 00DD78F0    4B              dec     ebx                                 ; 统计 壳转移了 多少次这样的代码  
> 00DD78F1  ^ 75 C3           jnz     short 00DD78B6  
> 00DD78F3    5F              pop     edi  
> 00DD78F4    8D4D 66         lea     ecx, dword ptr \[ebp+66\]  
> 00DD78F7    2BCF            sub     ecx, edi  
> 00DD78F9    F3:AA           rep     stos byte ptr es:\[edi\]  
> 00DD78FB    61              popad  
> 00DD78FC    C3              retn  
>   
>  我一开始想 从这里 直接把壳转移的信息 写到原程序的!  
> 后来 看了一些 很多 壳里面的代码是  
>   
> Lea XXXX  
> Xxxxx  
> 给 EAX,赋值的   
> 所以 要写回源程序的 代码要 智能一点,直接 mov eax,xxxx  
> 但是 好像好难,所以我只能从  
> 补区段里面获取 这个!再补区段  
> 最近,头比较乱,所以说话语无伦次.  
>   
> 呵呵，关注一下大牛 是如何还原代码的！  
>   
>   
>   
> /\*  
> //////////////////////////////////////////////////  
> PeLock 1.X UnPacK And Fix ITA  
>   
> //////////////////////////////////////////////////  
> \*/  
>   
> CheckVersion:  
> //检查 OllyScripts 插件版本  
>   CMP $VERSION, "1.51"      
>   JAE Start      //OllyScripts 插件版不小于 1.51   
>   MSG "OllyScripts 插件版本小于 1.51，请去 WwW.PeDIY.CoM 下载新版本"  
>   JMP stop  
>   
>   
>   
> stop:  
> //结束脚本  
>   CMT eip, "已经停止 脚本"  
>   RET  
>   
>   
> Start:  
> //数据初始化  
>   
>         
>   
>   
>   //获取 代码段相关信息  
>   VAR cdbs  
>   VAR cdss  
>   GMI eip, codeBASE  
>   MOV cdbs, $RESULT  
>       //获取 代码段基址  
>   GMI eip, codesize  
>   MOV cdss, $RESULT  
>       //获取 代码段大小  
>   
>   
>   //定义变量  
>   VAR temp  
>         //临时变量  
>   VAR CRC\_Add  
>         //CRC 验证前一个异常地址  
>   VAR CRC\_Code\_Add  
>         //转向 CRC 补丁地址  
>   VAR CRC\_Patch\_Add  
>         //CRC 补丁地址  
>   VAR Fix\_ITA\_Add  
>         //修复 ITA 的代码地址  
>   
>   //设置 OD  
>   BPMC      //清楚 内存断点  
>   BPHWCALL    //清除所有硬件断电  
>   DBH      //隐藏 OD  
>   CMT eip, "开始执行 脚本"  //批注  
>   JMP Run\_Script    //执行 脚本  
>   
>   
>   
>   
>   
> Run\_Script:  
> //执行脚本  
>   MSGyn "是否执行脚本"  
>   CMP $RESULT,0  
>   JNE Check    //检查壳是否为 PeLock  
>   JMP stop  
>   
>   
> Check:  
> //检查 PELock 1.0x -> Bartosz Wojcik 特征指纹  
>   
>   FIND eip-5c,#4C6F61644C6962726172794100005669727475616C416C6C6F63004B45#  
>   CMP $RESULT,0  
>   JNE Seach\_CRC  
>   MSG "好像不是 PELock 1.0x -> Bartosz Wojcik 吧？"  
>   JMP stop  
>   
>   
>   
>   
> Seach\_CRC:  
> // 查找 CRC 验证 前一个异常的地址  
>   ESTO  
>   FIND eip,#8909#  
>         //查找特征代码 "MOV dword ptr \[ecx\], ecx"  
>   MOV CRC\_Add,$RESULT  
>   CMP CRC\_Add,eip  
>   JE Seach\_CRC\_Code  
>         //是否执行到 特征代码这里  
>   JMP Seach\_CRC  
>   
>   
>   
>   
>   
> Seach\_CRC\_Code:  
> //查找 CRC 代码地址  
>   FIND eip,#2B848D143B0000#  
>         //查找特征代码 "SUB EAX,DWORD PTR SS:\[EBP+ECX\*4+3B14\]"  
>   MOV CRC\_Code\_Add,$RESULT  
>   CMP CRC\_Code\_Add,0  
>   JNE Seach\_Fix\_ITA\_Add  
>         //是否检查到 特征代码  
>   MSG "没有找到 CRC 代码效验地址"  
>   JMP stop  
>   
>   
> Seach\_Fix\_ITA\_Add:  
> //查找 修复 ITA 的代码地址  
>   FIND eip,#8919EB#  
>         //查找特征代码 "MOV DWORD PTR DS:\[ECX\],EBX"  
>   MOV Fix\_ITA\_Add,$RESULT  
>   CMP Fix\_ITA\_Add,0  
>   JNE Come\_CRC\_Code  
>         //是否检查到 特征代码  
>   MSG "没有找到 ITA 代码效验地址"  
>   JMP stop  
>   
>   
>   
> Come\_CRC\_Code:  
> //自行到 进行 CRC 验证的代码  
>   
>   bp CRC\_Code\_Add  
>   ESTO  
>   CMP CRC\_Code\_Add,eip  
>   JE Seach\_CRC\_Patch\_Space  
>         //判断是否执行到 CRC 验证代码段  
>   JMP Come\_CRC\_Code  
>   
>   
> Seach\_CRC\_Patch\_Space:  
> //查找可用写 CRC 补丁的空间  
>   GMEMI eip,MEMORYBASE   
>   MOV CRC\_Patch\_Add,$RESULT  
>   GMEMI eip,MEMORYSIZE  
>   ADD CRC\_Patch\_Add,$RESULT  
>   SUB CRC\_Patch\_Add,100  
>         //CRC\_PATCH 代码地址为 当前执行段末尾-100  
>   CMP CRC\_Patch\_Add,0  
>   JNE CRC\_Patch\_Code  
>   MSG "没有找到 CRC 补丁的空间"  
>         //检验是否找 CRC 补丁 补丁 地址  
>   JMP stop  
>   
>   
> CRC\_Patch\_Code:  
> //CRC 补丁代码  
>   
>   bc CRC\_Code\_Add  
>   MOV   
>   
> \[CRC\_Patch\_Add\],#81F948010000742481F985000000741681F98400000074082B848D143B0000C32D2B848D14C32D3B0000D3C32D8919EB03C30000#  
>         //CRC 补丁  
>   MSG "CRC 补丁成功"  
>   MSG CRC\_Patch\_Add  
>   
>   JMP FIX\_CRC\_Enter\_Point  
>   
>   
>   
> FIX\_CRC\_Enter\_Point:  
> //修改 CRC 的入口  
>   EVAL "call {CRC\_Patch\_Add}"  
>   ASM eip,$RESULT  
>         //修改 当前代码为 CALL CRC 补丁地址  
>   MOV temp,CRC\_Code\_Add  
>   ADD temp,5  
>   MOV \[temp\],#9090#  
>         //把后面的两个字节用 NOP 覆盖  
>   CMT eip,"修改 CRC 的入口"  
>   MSG "成功修改 CRC 的入口"  
>   JMP Seach\_Fix\_ITA  
>   
>   
> Seach\_Fix\_ITA:  
> //查找修复 ITA 的地址  
>   bp Fix\_ITA\_Add  
>   ESTO  
>   CMP eip,Fix\_ITA\_Add  
>   JNE Seach\_Fix\_ITA  
>         //运行到 Fix\_ITA 代码处  
>   JMP Fix\_ITA  
>   
>   
> Fix\_ITA:  
> //修复 ITA  
>   bc Fix\_ITA\_Add  
>   ASM Fix\_ITA\_Add,"MOV DWORD PTR DS:\[ECX\],EAX"  
>         //修改 "MOV DWORD PTR DS:\[ECX\],EBX" 为 "MOV DWORD PTR DS:\[ECX\],EAX"  
>   CMT Fix\_ITA\_Add,"修复 ITA 地址"  
>   MOV temp,cdbs  
>   ADD temp,cdss  
>         //获取 代码段范围 存放再 temp 中  
>   JMP Cater  
>   
>   
> Cater:  
> //异常处理过程  
>   BPWM cdbs,cdss  
>         //代码段 内存写断点  
>   ESTO  
>   COE Cater  
>   BPMC  
>   ret  
>   JMP Come\_Code\_Section  
>   
>   
> Come\_Code\_Section:  
> //设法通过 代码段 内存访问端点 使得 程序执行到 代码段  
>   
>   BPRM cdbs,cdss  
>         //代码段 内存访问断点  
>   ESTO  
>   CMP eip,temp  
>         //验证程序是否自行到 代码段  
>   JA Come\_Code\_Section  
>   CMP eip,cdbs  
>   JA YesCome  
>   JMP Come\_Code\_Section  
>     
>   
>   
> YesCome:  
>   #LOG   
>   LOG "ok"  
>   AN eip  
>   MSG "是否执行自动修复 Stolen Code(还没完善)"  
>   
>   RET 

 4dd466d0e1e0cc123afbe504da2462ca