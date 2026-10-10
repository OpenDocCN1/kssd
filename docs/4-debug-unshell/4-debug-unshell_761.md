

# 脱壳学习日记

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-652.htm`](https://www.pediy.com/kssd/pediy08/pediy8-652.htm)

最近对一句话认识很深，知识就象一个圆，你知道得越少，那么你不懂的就越少。  
  
前阵子想着开发一个变形代码还原的工具，找了很多资料，学得越多，新的问题就越多，而且一个比一个难，终于到了我无法解决的程度。只好先放下，继续好好学习逆向了。  
  
开一个帖，写上现在和以后脱壳的学习过程，希望有一天可以不需要参考高手们相同壳的分析，写出自己的原创分析。  
  
假如你也准备或者正在学习脱壳，那么我在每篇脱文最前面的心得体会或者对你有帮助。  
  
11 月 20 日  
  
2 楼 MSLRHv0.31a 主程序  
3 楼 MSLRHv0.32a 主程序   
6 楼 pe-armor0.74 

  

这东西比较适合练耐性还有就是复习一下各种 ANTI DEBUG，还有就是测试一下你 OD 的抗 ANTI 能力，假如过不了某写 ANTI，就得去找插件或者自己动手了。这里涉及的 ANTI 有时间校验、ISDEBUG 标记、检测 Drx、检测 INT3、ZwQueryInformationProcess、OUTPUTDEBUGSTRING 文件自校验和通过 CREATEFILE ANTI DUMP，最后是一个压缩壳。  
  
原程序可在下面连接下载：  
http://www.pediy.com/tools/PACK/Protectors/MSLRH/MSLRHv0.31a.rar  
  
这里是已脱壳的程序  
http://www.nxer.cn/709394/attachment/1163338396\_0.rar  
  
由于有大量时间检查，于是便找来 ULTRAEDIT 把 RDTSC 直接替换成 XOR EAX，EAX 了。运行程序发现出错，有自校验，只好细心跟了，跟了大半天，终于到了自校验的位置，由于不是简单的对比加跳转，必须还原改动过的值。只好去论坛找文章看看有没有更好的方法对付 RDTSC。查到前辈写的脱这个的文章，知道了 DEJUNK 插件，学习了一下，写了下面脚本：  
  
ActivePatList1  = \_mslrh01  
  
\[CODE\_mslrh01\]  
  
S = 0f31  
  
R = 33c0  
  
后面有 2 种解码，分别是 XOR 11 和 XOR 15，所以对应 3 种情况，替换了 825 处。  
  
看了前辈的文章之后，对一些之前不明白的 ANTI 也较为清楚了，下面按流程一步一步列出来。  
  
00456AA0    810424 6F130000 add dword ptr ss:\[esp\],136F  
00456AA7    64:FF35 0000000>push dword ptr fs:\[0\]  
00456AAE    64:8925 0000000>mov dword ptr fs:\[0\],esp  
00456AB5    EB 05           jmp short \[MSLRH\].00456ABC  
  
装了 SEH 之后便是  
  
0045745C    33C0            xor eax,eax  
0045745E    0FB600          movzx eax,byte ptr ds:\[eax\]  
  
内存访问错误，进入 SEH 后继续。大量相同规律的花指令，程序没有乱序，方向总是向下，有意义的代码没怎么花，时间校验的 cmp eax,0fff 也没花。这是第一次跟这个壳的时候总结的经验，所以可以向下寻找没有花的地方，然后在最接近的 cmp eax,0fff 直接跳过去，这是第一次跟这个程序时总结出来的经验。  
  
004587B6    8B4424 0C       mov eax,dword ptr ss:\[esp+C\]  
004587BA    33C9            xor ecx,ecx  
004587BC    3348 04         xor ecx,dword ptr ds:\[eax+4\]  
004587BF    3348 08         xor ecx,dword ptr ds:\[eax+8\]  
004587C2    3348 0C         xor ecx,dword ptr ds:\[eax+C\]  
004587C5    3348 10         xor ecx,dword ptr ds:\[eax+10\]  
004587C8    8B6424 08       mov esp,dword ptr ss:\[esp+8\]  
004587CC    64:8F05 0000000>pop dword ptr fs:\[0\]  
  
这里检查 dr0 到 dr3 是否为 0，没有下硬断，则 ecx 为 0  
  
00459191    51              push ecx  
00459192    33C9            xor ecx,ecx  
00459194    E8 00000000     call \[MSLRH\].00459199  
00459199    5F              pop edi  
0045919A    81C7 C4090000   add edi,9C4  
004591A0    5A              pop edx  
004591A1    83C2 15         add edx,15  
004591A4    0FB60439        movzx eax,byte ptr ds:\[ecx+edi\]  
004591A8    33C2            xor eax,edx  
004591AA    880439          mov byte ptr ds:\[ecx+edi\],al  
004591AD    41              inc ecx  
004591AE    81F9 93000000   cmp ecx,93  
004591B4  ^ 72 EE           jb short \[MSLRH\].004591A4  
004591B6    EB 05           jmp short \[MSLRH\].004591BD  
  
这段是对 00459b5d-00459bf0 进行解码（简单的 XOR）。假如曾经下过硬断，那么这里的 KEY 就不对了。正确的是 15。解码前要恢复为清除时间校验而覆盖的数据，解码之后再此清除时间校验，后同。  
  
00459B5D    8B5C24 20       mov ebx,dword ptr ss:\[esp+20\]  
00459B61    66:BB 0000      mov bx,0  
00459B65    0FB703          movzx eax,word ptr ds:\[ebx\]  
00459B68    2D 4D5A0000     sub eax,5A4D  
00459B6D    74 08           je short 复件\_\[MS.00459B77       
00459B6F    81EB 00000100   sub ebx,10000                                 
00459B75  ^ EB EE           jmp short 复件\_\[MS.00459B65  
  
通过在堆栈中返回系统的地址找 KERNEL32 的基址。Billy Belceb 病毒编写教程里有提到过这种方法。现在 EBX 是 KERNEL32 的基址  
  
00459B77    8BFB            mov edi,ebx  
00459B79    037B 3C         add edi,dword ptr ds:\[ebx+3C\]  
00459B7C    83C7 78         add edi,78  
00459B7F    8B3F            mov edi,dword ptr ds:\[edi\]  
00459B81    03FB            add edi,ebx  
00459B83    57              push edi  
  
EDI 输出表地址  
  
00459B84    83C7 20         add edi,20  
00459B87    8B3F            mov edi,dword ptr ds:\[edi\]  
00459B89    03FB            add edi,ebx  
00459B8B    33C0            xor eax,eax  
00459B8B    33C0            xor eax,eax  
00459B8D    40              inc eax  
00459B8E    8B0F            mov ecx,dword ptr ds:\[edi\]  
00459B90    03CB            add ecx,ebx  
00459B92    83C7 04         add edi,4  
00459B95    8139 47657450   cmp dword ptr ds:\[ecx\],50746547  
00459B9B  ^ 75 F0           jnz short 复件\_\[MS.00459B8D  
00459B9D    8179 04 726F634>cmp dword ptr ds:\[ecx+4\],41636F72  
00459BA4  ^ 75 E7           jnz short 复件\_\[MS.00459B8D  
  
历遍输出表的 AddressOfNames，对比头 8 个字符识别出 GetProcAddress  
  
00459BA6    6BC0 02         imul eax,eax,2  
00459BA9    5F              pop edi  
00459BAA    57              push edi  
00459BAB    83C7 24         add edi,24  
00459BAE    8B3F            mov edi,dword ptr ds:\[edi\]  
00459BB0    03FB            add edi,ebx  
00459BB2    03F8            add edi,eax  
00459BB4    66:8B07         mov ax,word ptr ds:\[edi\]  
00459BB7    6BC0 04         imul eax,eax,4  
00459BBA    5F              pop edi  
00459BBB    83C7 1C         add edi,1C  
00459BBE    8B3F            mov edi,dword ptr ds:\[edi\]  
00459BC0    03FB            add edi,ebx  
00459BC2    03F8            add edi,eax  
00459BC4    8B7F FC         mov edi,dword ptr ds:\[edi-4\]  
00459BC7    03FB            add edi,ebx  
00459BC9    803F CC         cmp byte ptr ds:\[edi\],0CC  
00459BCC   /75 09           jnz short 复件\_\[MS.00459BD7     
  
取得 GetProcAddress 在内存中的位置还不忘检查其入口有没有断点。。。  
  
00459BD7    E8 00000000     call 复件\_\[MS.00459BDC  
00459BDC    58              pop eax  
00459BDD    2D EC3A0000     sub eax,3AEC  
00459BE2    B0 00           mov al,0  
00459BE4    05 00200100     add eax,12000  
00459BE9    8BF0            mov esi,eax  
00459BEB    891E            mov dword ptr ds:\[esi\],ebx  
00459BED    897E 10         mov dword ptr ds:\[esi+10\],edi  
  
将 KERNEL32 的基址存入 EP，GetProcAddress 则放在 EP+10  
  
00459BF7    5F              pop edi  
00459BF8    81C7 C4090000   add edi,9C4  
00459BFE    0FB60439        movzx eax,byte ptr ds:\[ecx+edi\]  
00459C02    83F0 15         xor eax,15  
00459C05    880439          mov byte ptr ds:\[ecx+edi\],al  
00459C08    41              inc ecx  
00459C09    81F9 3F0C0000   cmp ecx,0C3F  
00459C0F  ^ 72 ED           jb short \[MSLRH\].00459BFE  
00459C11    EB 05           jmp short \[MSLRH\].00459C18  
  
对 0045a5bb-0045b1fa 解码，同样是 XOR 15。  
  
0045A5D0    E8 00000000     call 复件\_\[MS.0045A5D5  
0045A5D5    832C24 18       sub dword ptr ss:\[esp\],18  
0045A5D9    FF36            push dword ptr ds:\[esi\]  
0045A5DB    FF56 10         call dword ptr ds:\[esi+10\]                    ; KERNEL32.GetProcAddress  
  
0012FF98   77E60000  KERNEL32.77E60000  
0012FF9C   0045A5BD  ASCII "OutputDebugStringA"  
  
指令间穿插字符。。  
  
0045A5DE    8946 14         mov dword ptr ds:\[esi+14\],eax                 ; KERNEL32.OutputDebugStringA  
  
存进 EP+14 的位置。似乎 EP 要被重写为输入表了  
  
下一个 API 地址的载入方法对于我来说比较新鲜。  
  
0045A5EB    E8 10000000     call 复件\_\[MS.0045A600  
0045A600    FF36            push dword ptr ds:\[esi\]  
0045A602    FF56 10         call dword ptr ds:\[esi+10\]  
0045A605    8946 18         mov dword ptr ds:\[esi+18\],eax                 ; KERNEL32.GetCommandLineA  
  
0045A5EB 处 CALL 的返回地址竟然是  
  
0045A5F0   47 65 74 43 6F 6D 6D 61 6E 64 4C 69 6E 65 41 00  GetCommandLineA.  
  
把 OD 的堆栈参考骗过去了，只提示是返回地址，没有字符参考。不过跟进 KERNEL32 的领域就能显示字符参考了。而且后面将地址保存的时候也有注释，这个技巧意义不大。后面全是类似的技巧和函数的载入，就忽略了。  
  
0045B1C9    8CC9            mov cx,cs  
0045B1CB    32C9            xor cl,cl  
0045B1CD    83F9 00         cmp ecx,0  
0045B1D0    75 28           jnz short 复件\_\[MS.0045B1FA  
  
上面的代码似乎是检查操作系统的，就不会有后面 PEB 的检查了。据说如果是 WIN9X 就会跳走  
  
0045B1D2    64:FF35 3000000>push dword ptr fs:\[30\]  
0045B1D9    58              pop eax  
0045B1DA    0FB648 02       movzx ecx,byte ptr ds:\[eax+2\]  
0045B1DE    884E 0C         mov byte ptr ds:\[esi+C\],cl  
  
将 ISDEBUG 标记写入 0046800C。当然我有插件。  
  
0045B1E1    8B40 0C         mov eax,dword ptr ds:\[eax+C\]  
0045B1E4    8B40 0C         mov eax,dword ptr ds:\[eax+C\]  
0045B1E7    8D58 20         lea ebx,dword ptr ds:\[eax+20\]  
0045B1EA    8D48 18         lea ecx,dword ptr ds:\[eax+18\]  
0045B1ED    8103 C8000000   add dword ptr ds:\[ebx\],0C8  
0045B1F3    B8 00000000     mov eax,0  
0045B1F8    0101            add dword ptr ds:\[ecx\],eax  
0045B1FA    33C9            xor ecx,ecx  
0045B1FC    E8 00000000     call 复件\_\[MS.0045B201  
  
改写 PEB 中程序在内存中的基址和大小信息，大小+0c8，基址+0。有意义？  
  
0045B1FC    E8 00000000     call 复件\_\[MS.0045B201  
0045B201    5F              pop edi                                       ; 复件\_\[MS.0045B201  
0045B202    81C7 C1090000   add edi,9C1  
0045B208    0FB60439        movzx eax,byte ptr ds:\[ecx+edi\]  
0045B20C    83F0 11         xor eax,11  
0045B20F    880439          mov byte ptr ds:\[ecx+edi\],al  
0045B212    41              inc ecx  
0045B213    81F9 521D0000   cmp ecx,1D52  
0045B219  ^ 72 ED           jb short 复件\_\[MS.0045B208  
0045B21B    EB 05           jmp short 复件\_\[MS.0045B222  
  
0045bbc2-0045d914 解码，这次是 XOR 11  
  
0045C56C    E8 00000000     call \[MSLRH\].0045C571  
0045C571    810424 CA090000 add dword ptr ss:\[esp\],9CA  
0045C578    64:FF35 0000000>push dword ptr fs:\[0\]  
0045C57F    64:8925 0000000>mov dword ptr fs:\[0\],esp  
0045C586    33DB            xor ebx,ebx  
0045C588    8B1B            mov ebx,dword ptr ds:\[ebx\]  
  
安装完 SEH 就立刻产生内存访问错误进入了。  
  
0045D8DF    8B4424 0C       mov eax,dword ptr ss:\[esp+C\]  
0045D8E3    33C9            xor ecx,ecx  
0045D8E5    3348 04         xor ecx,dword ptr ds:\[eax+4\]  
0045D8E8    3348 08         xor ecx,dword ptr ds:\[eax+8\]  
0045D8EB    3348 0C         xor ecx,dword ptr ds:\[eax+C\]  
0045D8EE    3348 10         xor ecx,dword ptr ds:\[eax+10\]  
  
继续检查 DRX  
  
0045DA76    884E 0D         mov byte ptr ds:\[esi+D\],cl  
  
这里将检查结果存在 0046800D 了。  
  
0045E42A    FF56 14         call dword ptr ds:\[esi+14\]                    ; KERNEL32.OutputDebugStringA  
  
0045EDD4    FF56 18         call dword ptr ds:\[esi+18\]                    ; KERNEL32.GetCommandLineA  
  
0045EDD7    40              inc eax  
0045EDD8    33C9            xor ecx,ecx  
0045EDDA    41              inc ecx  
0045EDDB    803C01 00       cmp byte ptr ds:\[ecx+eax\],0  
0045EDDF    74 0C           je short 复件\_\[MS.0045EDED  
0045EDE1    803C01 22       cmp byte ptr ds:\[ecx+eax\],22  
0045EDE5  ^ 75 F3           jnz short 复件\_\[MS.0045EDDA  
0045EDE7    C60401 00       mov byte ptr ds:\[ecx+eax\],0  
0045EDEB  ^ EB ED           jmp short 复件\_\[MS.0045EDDA  
0045EDED    6A 00           push 0  
0045EDEF    6A 00           push 0  
0045EDF1    6A 03           push 3  
0045EDF3    6A 00           push 0  
0045EDF5    6A 00           push 0  
0045EDF7    68 00000080     push 80000000  
0045EDFC    50              push eax  
0045EDFD    FF56 1C         call dword ptr ds:\[esi+1C\]  
  
一段计算字符长度未经优化的代码。然后是 CREATEFILE 打开自己,这样就 DUMP 不了了。修改 EIP 直接跳过  
  
0045F7A7    837E 40 00      cmp dword ptr ds:\[esi+40\],0  
0045F7AB    74 24           je short \[MSLRH\].0045F7D1  
0045F7AD    FF56 24         call dword ptr ds:\[esi+24\]  
0045F7B0    50              push eax  
0045F7B1    6A 00           push 0  
0045F7B3    68 00040000     push 400  
0045F7B8    FF56 28         call dword ptr ds:\[esi+28\]  
0045F7BB    8BDC            mov ebx,esp  
0045F7BD    83EB 04         sub ebx,4  
0045F7C0    6A 00           push 0  
0045F7C2    6A 00           push 0  
0045F7C4    6A 04           push 4  
0045F7C6    53              push ebx  
0045F7C7    6A 07           push 7  
0045F7C9    50              push eax  
0045F7CA    FF56 40         call dword ptr ds:\[esi+40\]  
0045F7CD    58              pop eax  
0045F7CE    8846 0E         mov byte ptr ds:\[esi+E\],al  
  
比较 ntdll.是 ZwQueryInformationProcess 否获取成功，获取成功就用该 API 检查是否被 DEBUG，将结果写入 0046800E  
  
00460178    8CC9            mov cx,cs  
0046017A    32C9            xor cl,cl  
0046017C    83F9 00         cmp ecx,0  
0046017F    0F84 A1130000   je \[MSLRH\].00461526  
00460185    8B46 38         mov eax,dword ptr ds:\[esi+38\]  
00460188    8078 01 4C      cmp byte ptr ds:\[eax+1\],4C  
0046018C    0F85 94130000   jnz \[MSLRH\].00461526  
  
检查系统，前半部分检查是不是 WIN9X，如果是则通过  
  
SetUnhandledExceptionFilter  
  
在 KERNEL32 中的偏移来确定具体是哪一个系统？我这里是 2K，跳走了  
  
00461ED2    58              pop eax                                       ; \[MSLRH\].00461ED2  
00461ED3    2D E2BD0000     sub eax,0BDE2  
00461ED8    B0 00           mov al,0  
00461EDA    05 00200100     add eax,12000  
00461EDF    8BF0            mov esi,eax  
00461EE1    807E 0C 00      cmp byte ptr ds:\[esi+C\],0  
00461EE5    74 51           je short \[MSLRH\].00461F38  
00461F38    807E 0D 00      cmp byte ptr ds:\[esi+D\],0  
00461F3C  ^ 0F85 AF41FFFF   jnz \[MSLRH\].004560F1  
00461F42    74 04           je short \[MSLRH\].00461F48  
00461F44    75 02           jnz short \[MSLRH\].00461F48  
  
00461F4B    807E 0E 00      cmp byte ptr ds:\[esi+E\],0  
00461F4F  ^ 0F85 9C41FFFF   jnz \[MSLRH\].004560F1  
00461F55    E8 0A000000     call \[MSLRH\].00461F64  
  
00461F6C    807E 0F 00      cmp byte ptr ds:\[esi+F\],0  
00461F70  ^ 0F85 7B41FFFF   jnz \[MSLRH\].004560F1  
00461F76    50              push eax  
00461F77    E8 02000000     call \[MSLRH\].00461F7E  
  
0046800F 不知道是什么的校验值，估计是针对某操作系统的。后面开始自校验了，清除所有断点，还原花指令。  
  
00461F92    59              pop ecx  
  
00461F9D    83E9 05         sub ecx,5  
  
00461FAA    33DB            xor ebx,ebx  
  
00461FB6    B8 9CBE0000     mov eax,0BE9C  
  
00461FC5    8BF9            mov edi,ecx                                   ; \[MSLRH\].00461F8D  
  
00461FD1    2BF8            sub edi,eax  
  
00461FDD    0FB607          movzx eax,byte ptr ds:\[edi\]  
  
00461FEA    03D8            add ebx,eax  
  
00461FF6    47              inc edi                                       ; \[MSLRH\].004560F1  
  
00462001    3BF9            cmp edi,ecx                                   ; \[MSLRH\].00461F8D  
  
0046200D  ^\\72 CE           jb short \[MSLRH\].00461FDD  
  
计算 004560f1 到 00461f8d 的校验和（从花指令入口处到进入自校验之前的代码）  
  
0046200F    BF 00704400     mov edi,\[MSLRH\].00447000  
00462014    B9 00BC0000     mov ecx,0BC00  
00462023    0FB607          movzx eax,byte ptr ds:\[edi\]  
00462030    02DF            add bl,bh  
00462032    32DF            xor bl,bh  
00462034    32C3            xor al,bl  
00462040    8807            mov byte ptr ds:\[edi\],al  
0046204C    47              inc edi  
00462057    49              dec ecx  
00462062  ^\\75 B5           jnz short \[MSLRH\].00462019  
  
对 00447000 开始大小为 BC00 的区域解码，KEY 就是刚才的校验和。  
  
00462064    E8 00000000     call \[MSLRH\].00462069  
00462069    59              pop ecx  
0046206A    2959 16         sub dword ptr ds:\[ecx+16\],ebx  
0046206D    61              popad  
0046206E    60              pushad  
0046206F    BE 00704400     mov esi,\[MSLRH\].00447000  
00462074    8DBE 00A0FBFF   lea edi,dword ptr ds:\[esi+FFFBA000\]  
0046207A    57              push edi  
0046207B    83CD FF         or ebp,FFFFFFFF  
0046207E    68 0FDE9F00     push 9FDE0F  
00462083    C3              retn  
  
这里的结果仍然跟校验和关联  
  
0046206A    2959 16         sub dword ptr ds:\[ecx+16\],ebx  
  
之后，最后两条指令便变成  
  
0046207E    68 D0284500     push \[MSLRH\].004528D0  
00462083    C3              retn  
  
跳到下一部分,据前辈的文章说，就是 UPX 核心了。  
  
004528d8-004529a1 是将 00447000 解压到 00401000。  
  
004529e0-00452a1a 是填写 IAT（在 00450fe7 获得 API 名字，将地址填进 00415028，就是说没加密）。  
  
用 OLLYDUMP 下来就完成脱壳了 

同一个壳的不同版本，由于有了前一个版本的经验，这次跟起来很轻松，显然经验是大多数新人所欠缺的，只能将勤补拙了。另外一个问题就是，当你遇到你所不知道的 ANTI 时，你会怎么办？到论坛请教高手？还是尝试自己弄清楚？我只想说，探索精神很重要。有的时候，象这里的例子，其实完全可以自己解决的。此外通过脱这个壳，有的 API 的运用，其实可以很巧妙，例如本例子中的 SetUnhandledExceptionFilter，无论你怎么 BP，都是断不下来的。  
  
源程序下载 http://emadicius.rvlcnsecurity.com/programas/MSLRHv0.32a.zip  
  
已脱壳的程序  
http://www.nxer.cn/709394/attachment/1163728509\_0.rar  
  
用 OD 加载程序时报错。现在用 WINDBG 载入 OD，bp messageboxa。用 OD 载入 MSLRHv0.32a。断下。  
  
0:000> kb  
ChildEBP RetAddr  Args to Child                
WARNING: Stack unwind information not available. Following frames may be wrong.  
0012c5c8 00454059 00120106 0012c5e0 004bb410 USER32!MessageBoxA  
0012cbe4 0045dce7 004bc1ea 063f0050 005bd670 image00400000!Error+0x3d  
0012d2d0 0045f7eb 063f0000 76bc1f1c 76bc2075 image00400000!Createlistwindow+0x2af7  
00000000 00000000 00000000 00000000 00000000 image00400000!Decoderange+0x180b  
  
在反汇编窗口输入 00454059  
  
0045401c 55              push    ebp  
0045401d 8bec            mov     ebp,esp  
0045401f 81c4fcf9ffff    add     esp,0FFFFF9FCh  
00454025 8d450c          lea     eax,\[ebp+0Ch\]  
00454028 50              push    eax  
00454029 8b5508          mov     edx,dword ptr \[ebp+8\]  
0045402c 52              push    edx  
0045402d 8d8dfcf9ffff    lea     ecx,\[ebp-604h\]  
00454033 51              push    ecx  
00454034 e81b2c0500      call    image00400000!Createpatchwindow+0x79f4 (004a6c54)  
00454039 83c40c          add     esp,0Ch  
0045403c 8d85fcf9ffff    lea     eax,\[ebp-604h\]  
00454042 8b157c3b4d00    mov     edx,dword ptr \[image00400000!\_\_CPPdebugHook+0x23a54 (004d3b7c)\]  
00454048 6810200000      push    2010h  
0045404d 6810b44b00      push    offset image00400000!\_\_CPPdebugHook+0xb2e8 (004bb410)  
00454052 50              push    eax  
00454053 52              push    edx  
00454054 e8bdb40500      call    image00400000!Createpatchwindow+0x102b6 (004af516)  
00454059 6a00            push    0  ；MESSAGEBOX 的返回地址  
0045405b e878dafdff      call    image00400000!Animate (00431ad8)  
00454060 59              pop     ecx  
00454061 8be5            mov     esp,ebp  
00454063 5d              pop     ebp  
00454064 c3              ret  
  
在 00454064 下断，返回 OD 按确定。  
  
0045dcd1 eb30            jmp     image00400000!Createlistwindow+0x2b13 (0045dd03)  
0045dcd3 837df402        cmp     dword ptr \[ebp-0Ch\],2  
0045dcd7 7513            jne     image00400000!Createlistwindow+0x2afc (0045dcec)  
0045dcd9 8d4e50          lea     ecx,\[esi+50h\]  
0045dcdc 51              push    ecx  
0045dcdd 68eac14b00      push    offset image00400000!\_\_CPPdebugHook+0xc0c2 (004bc1ea)  
0045dce2 e83563ffff      call    image00400000!Error (0045401c)  
0045dce7 83c408          add     esp,8  ；返回到这里  
0045dcea eb17            jmp     image00400000!Createlistwindow+0x2b13 (0045dd03)  
  
这里比较 EBP-C 是否为 2，是则弹出错误窗口。  
  
0:000> ? ebp-0c  
Evaluate expression: 1233604 = 0012d2c4  
  
重新载入 OD，下条件断点  
  
0:000> ba w4 0012d2c4 "r @$t0=poi(0012d2c4);j(@$t0!=2)'g'"  
  
OD 载入 MSLRHv0.32a。  
  
0045c651 7527            jne     image00400000!Createlistwindow+0x148a (0045c67a)  
0045c653 0fb79534faffff  movzx   edx,word ptr \[ebp-5CCh\]  
0045c65a 81c280000000    add     edx,80h  
0045c660 8b8da0f9ffff    mov     ecx,dword ptr \[ebp-660h\]  
0045c666 c1e103          shl     ecx,3  
0045c669 81c1e0000000    add     ecx,0E0h  
0045c66f 3bd1            cmp     edx,ecx  
0045c671 7407            je      image00400000!Createlistwindow+0x148a (0045c67a)  
0045c673 c745f402000000  mov     dword ptr \[ebp-0Ch\],2  
0045c67a 837df400        cmp     dword ptr \[ebp-0Ch\],0 ss:0023:0012d2c4=00000002  ；断在这里  
0045c67e 7524            jne     image00400000!Createlistwindow+0x14b4 (0045c6a4)  
  
这里对比 ebp-5CC 和 ebp-660。一定条件 ebp-0Ch=2。  
  
0:000> db ebp-660  
0012cc70  ff 00 00 00 00 00 00 00-00 00 00 00 f8 19 05 00  ................  
  
重新载入 OD，下条件断点  
  
0:000> ba w4 0012cc70 "r @$t0=poi(0012cc70);j(@$t0!=ff)'g'"  
  
OD 载入 MSLRHv0.32a。  
  
eax=0012cc14 ebx=004caf4c ecx=00000020 edx=000000e0 esi=01b028a8 edi=0012cc74  
eip=004a3547 esp=0012cb90 ebp=0012cb98 iopl=0         nv up ei pl nz ac po nc  
cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00200212  
image00400000!Createpatchwindow+0x42e7:  
004a3547 f3a5            rep movs dword ptr es:\[edi\],dword ptr \[esi\] es:0023:0012cc74=00160106 ds:0023:01b028a8=00000000  
  
在内存窗口中输入 ESI  
  
01b02908 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ........................  
01b02920 00 00 00 00 00 00 00 00 2e 65 64 61 74 61 00 00 00 10 04 00 00 10 00 00  .........edata..........  
01b02938 00 00 00 00 00 04 00 00 00 00 00 00 00 00 00 00 00 00 00 00 80 00 00 e0  ........................  
01b02950 2e 65 64 61 74 61 00 00 00 d0 00 00 00 20 04 00 00 c8 00 00 00 04 00 00  .edata....... ..........  
01b02968 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 c0 00 00 00 00 03 00 00 00  ............@...........  
01b02980 00 30 00 00 00 f0 04 00 00 2c 00 00 00 cc 00 00 00 00 00 00 00 00 00 00  .0.......,..............  
01b02998 00 00 00 00 40 00 00 c0 00 00 00 00 00 00 00 00 00 50 00 00 00 20 05 00  ....@............P... ..  
01b029b0 00 50 00 00 00 f8 00 00 00 00 00 00 00 00 00 00 00 00 00 00 20 00 00 e0  .P.................. ...  
01b029c8 2e 74 65 78 74 00 00 00 00 10 00 00 00 70 05 00 00 00 00 00 00 48 01 00  .text........p.......H..  
  
象 PE 头。向上移看看，果然  
  
01b02830 50 45 00 00 4c 01 05 00 fe df 9c 42 00 00 00 00 00 00 00 00 e0 00 0f 01  PE..L......B............  
01b02848 0b 01 05 0c 00 30 01 00 00 90 00 00 00 10 04 00 00 20 05 00 00 20 05 00  .....0........... ... ..  
01b02860 00 f0 04 00 00 00 40 00 00 10 00 00 00 02 00 00 04 00 00 00 04 00 00 00  ......@.................  
01b02878 04 00 00 00 00 00 00 00 00 80 05 00 00 10 00 00 00 00 00 00 02 00 00 00  ........................  
01b02890 00 00 10 00 00 10 00 00 00 00 10 00 00 10 00 00 00 00 00 00 ff 00 00 00  ........................  
01b028a8 00 00 00 00 00 00 00 00 f8 19 05 00 88 01 00 00 00 f0 04 00 f8 29 00 00  .....................)..  
  
既然是 PE，用 LORDPE 打开 MSLRHv0.32a。  
  
可选头部大小 00e0  
RVA 数及大小  FF  
  
点 RVA 数及大小旁边的减号  
  
可选头部大小 0850  
RVA 数及大小  FE  
  
再点加  
  
可选头部大小 0858  
RVA 数及大小  FF  
  
这两项是相关联的，可见 OD 中 EBP-660 和 EBP-5CC 就是这两项了，校验不正确就报错。程序在系统中能正确运行。RVA 数及大小错，只影响 OD 加载。  
  
0045c671 7407            je      image00400000!Createlistwindow+0x148a (0045c67a)  
  
为了以后对该 ANTI 免疫，将上面的 JE 改成 JMP。  
  
现在可以用 OD 分析了。该壳跟上一版差不多，又是 RDTSC 海，不过限制宽了，从 FF 变成 FFFF，最多只能防单步，有等于没有。同样是插入宏的方式加花，方向向下，看到有意义的地方直接下断 F9 过去就好了。相同的地方就不说了，说说不同的地方，不过不代表不需要处理 CreateFile。  
  
00453118    64:FF35 0000000>push dword ptr fs:\[0\]  
0045311F    64:8925 0000000>mov dword ptr fs:\[0\],esp  
00453126    33C0            xor eax,eax  
00453128    CD 01           int 1  
0045312A    40              inc eax  
0045312B    40              inc eax  
  
此处在 INT 1 后下断就可以了。然后是用 CREATEMUTEX 建立互斥对象，存在则继续，不存在则新建一个进程，自己退出。修改 EIP 直接跳到后面就可以了。再来是 FindWindow，也查 IMPORTRECT 窗口，最好也自己动手把窗口标题改了。  
  
SetUnhandledExceptionFilter 用得有点诡异。  
  
00455216    8B46 38         mov eax,dword ptr ds:\[esi+38\]  ；\[esi+38\]=SetUnhandledExceptionFilter  
00455219    8B40 0B         mov eax,dword ptr ds:\[eax+B\]  
0045521C    8908            mov dword ptr ds:\[eax\],ecx     ；ECX 是程序自定义的 SEH  
  
我们来看看为什么可以这么用  
  
77E678A7 >/$  8B4C24 04     mov ecx,dword ptr ss:\[esp+4\]  
77E678AB  |.  A1 4C04EC77   mov eax,dword ptr ds:\[77EC044C\]  
77E678B0  |.  890D 4C04EC77 mov dword ptr ds:\[77EC044C\],ecx  
77E678B6  \\.  C2 0400       retn 4  
  
SetUnhandledExceptionFilter 入口点+B 的地方刚好是 4C04EC77。当然也可以+5。  
  
0045549A    33DB            xor ebx,ebx  
0045549C    F7F3            div ebx  
  
除 0 错误，进入 SEH 例程,之后便来到最后的自校验了。  
  
该版本跟前一个版本一样，将一些 DEBUG 的校验放到最后,校验值的存放位置 45700c 到 45700F，处理 CREATEMUTEX 和 CreateFile 后，直接在该处下内存断点就能到最后的自校验了，仍然是 UPX 内核。 

 4dd466d0e1e0cc123afbe504da2462ca