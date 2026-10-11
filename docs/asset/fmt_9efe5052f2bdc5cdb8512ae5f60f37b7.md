
`0012FF92    834424 08 0E       add     dword ptr ss:[esp+8],0E               ；3736f4`  

`0012FF97    68 E6363700       push    3736E6`  

`0012FF9C    C2 1000           retn    10`  

`003736F4    8985 180F4100     mov     dword ptr ss:[ebp+410F18],eax        ; 00400000-－》38262d，准备盖掉罪状`  

`00373711    6A 04             push    4`  

`00373713    68 00100000       push    1000                                  ; 申请缓冲区大小 1000`  

`00373718    68 00100000       push    1000`  

`0037371D    6A 00             push    0`  

`0037371F    FF95 140F4100     call    near dword ptr ss:[ebp+410F14]      ; VirtualAlloc 申请缓冲区（390000)`  

又申请内存，乖乖，还没有走两步，就又开始申请内存玩是么名堂了？？？？  

`00373725    8985 54184100     mov     dword ptr ss:[ebp+411854],eax        ; 将申请的缓冲区段地址入 382f69`  

`0037372B    68 003C070A       push    0A073C00                              ;`  

`00373730    68 9FFC0D75       push    750DFC9F                              ;干吗又推这样的东西入堆栈,看着 75 头`  

`                                                                                     ;就明白要为下面跑堆栈里面跑代码.`  

`00373735    50               push    eax                                     ;390000`  

`003737A3    FF30             push    dword ptr ds:[eax]                       ; 将 37373f 内的内容送到堆栈，从这里开始`  

`                                                                                     ;到 3737ad，将把代码送入堆栈，真精彩的代码`  

其中花指令带压代码入栈方式,真的很漂亮,跟着都让人感觉心旷神怡.  

`003737AA    89E0             mov     eax,esp                                   ; 将堆栈地址入 eax,准备入堆栈执行`  

`003737AD    FFD0             call    near eax                                  ; 跳入堆栈进行代码执行`  

`0012FF44    873424          xchg    dword ptr ss:[esp],esi            ; 将堆栈和 ESI 的内容互换`  

`0012FF47    8B36            mov     esi,dword ptr ds:[esi]            ; 将[ESI]的内容赋值给 ESI`  

`0012FF49    81F6 EBFF71C8   xor     esi,C871FFEB                      ;将 ESI 和 C871ffeb 比较`  

`0012FF4F    75 19           jnz     short 0012FF6A                    ;不等则跳,这里不能跳.`  

`0012FF51    8B7424 50       mov     esi,dword ptr ss:[esp+50]       ;`  

`0012FF55    56              push    esi                               ; 取 ESP+50 的内容重新压栈`  

`0012FF56    8B36            mov     esi,dword ptr ds:[esi]            ;`  

`0012FF58    81F6 EBFF71E8   xor     esi,E871FFEB                      ; 将 ESP+50 的内容和 E871FFEB 比较`  

`0012FF5E    75 09           jnz     short 0012FF69                    ;不等则跳,这里不能跳`  

`0012FF60    5E              pop     esi                               ;`  

`0012FF61    83C6 4C         add     esi,4C`  

`0012FF64    897424 48       mov     dword ptr ss:[esp+48],esi       ; 将从堆栈的返回代码段地址压栈`  

`0012FF68    8D7424 58       lea     esi,dword ptr ss:[esp+58]       ; 将 ESP+58 地址送 ESI`  

`0012FF6C    51              push    ecx                               ;`  

`0012FF6D    B9 02000000     mov     ecx,2                             ;要运算出 2 个指令`  

`0012FF72    8136 EBFF7874   xor     dword ptr ds:[esi],7478FFEB    ; 作这个运算真不简单,记得刚才我说的 75 头的数值压栈 0012FF78 83EE FC sub esi,-4 ;的话么,这里把指令变成真正的指令.`  

`0012FF78    83EE FC         sub     esi,-4                            ;下一个数据`  

`0012FF7B    49              dec     ecx                               ;计数器`  

`0012FF7C    75 F4           jnz     short 0012FF72                    ;`  

`0012FF7E    59              pop     ecx                               ;`  

`0012FF7F    8D7424 58       lea     esi,dword ptr ss:[esp+58]       ; 将[ESP+58]内容送入 ESI`  

`0012FF83    FFD6            call    near esi                          ; 到 12ff98`  

`0012FF98    74 03           je      short 0012FF9D`  

`0012FF9D    C3              retn                                      ; 返回到 12ff85`  

`0012FF85    5E              pop     esi                               ; 准备去调用 KERNEL32.dll`  

`0012FF86    F3:             prefix rep:`  

`0012FF87    68 87373700     push    373787`  

`0012FF8C    C2 5000         retn    50                                ; 返回代码段`  

大家看了这段如何?真的很精彩,在堆栈里面执行代码还不忘记边执行边解码出来,我等菜鸟佩服的五体投地.  

后面将持续的运用这样的方式.  

`00373787    8B0424          mov     eax,dword ptr ss:[esp]            ; 将缓冲区（390000)段地址入 EAX`  

`0037378A    83EC F8         sub     esp,-8                            ; 堆栈指针＋8(12ff9c）`  

`0037387F    5E              pop     esi                               ; 取 370000 段地址`  

`00374478    56              push    esi`  

`00374479    8DB5 C1214000   lea     esi,dword ptr ss:[ebp+4021C1]  ; 将地址（3738d6)送 ESI`  

`0037447F    68 03EA17B3     push    B317EA03                          ;`  

`003744F2    FF33            push    dword ptr ds:[ebx]                ;又开始要往堆栈里面压代码`  

`003744F9    89E3            mov     ebx,esp                           ;要往堆栈里面走了 :(`  

`003744FC    FFD3            call    near ebx                          ;去堆栈了,真受不了,好好程序段不用`  

`                                                                                     ;喜欢跑堆栈里面玩,以后我们都跑女厕所`  

`003745CE    8BFE            mov     edi,esi                   ; 3738d6 开始，446 大小准备解码，送 370000`  

`003745D0    B9 46040000     mov     ecx,446`  

`003745D5    AC              lods    byte ptr ds:[esi]               ; 把取出的数据放到 al 中`  

`003745D6    9C              pushfd`  

`003745D7    6A 03           push    3`  

`003745EA    83C4 04         add     esp,4                          ;ESP+4`  

`003745F1    FF0C24          dec     dword ptr ss:[esp]             ;3-1`  

`003745EA    83C4 04         add     esp,4                          ;ESP+4`  

`003745F1    FF0C24          dec     dword ptr ss:[esp]             ;2-1`  

`003745EA    83C4 04         add     esp,4                          ;ESP+4`  

`003745F1    FF0C24          dec     dword ptr ss:[esp]             ;1-1`  

其实上面就是将[ESP]中的内容=-1,这是迷惑人的,算花指令,真正也就下面的解码方式  

`00374603    32C1            xor     al,cl                             ;al=8`  

`0037461C    C0C0 04         rol     al,4                              ;al=80`  

`0037464C    AA              stos    byte ptr es:[edi]                 ;把计算后的结果放回去`  

`0037464D    E2 86           loopd   short 003745D5                    ;循环解码`  

一直在解码,真是藏在深闺不露面呀,不行我跑女厕所看你去!!!!!!!!!!!!!!  

后面开始很危险了,这里  

`0037464F    5E              pop     esi                         ; 00373885`  

`00374650    33C0            xor     eax,eax`  

`00374652    64:FF30         push    dword ptr fs:[eax]            ; 又在建 SEH 了,准备异常了`  

`00374655    64:8920         mov     dword ptr fs:[eax],esp`  

`00374658    0F0B            ud2`  

看句柄跟着跑呀  

`0012FF9C /0012FFE0`  

`0012FFA0 |00373885  返回到 00373885 来自 00374478`  

`0012FFA4 |7C930738  ntdll.7C930738`  

`00373885    E8 04000000      call    0037388E                       ;`  

`003738CC    FF81 B8000000    inc     dword ptr ds:[ecx+B8]          ;`  

`003738D2    3D 03000080      cmp     eax,80000003                   ;呵呵这里比这个大家明白了么?`  

`00373B24    8161 14 F00FFFFF and     dword ptr ds:[ecx+14],FFFF0FF0 ; ffff0ff0`  

`003747B7    AC               lods    byte ptr ds:[esi]               ; 2d`  

`003747B8    32C1            xor     al,cl`  

`003747BA    04 4D           add     al,4D`  
