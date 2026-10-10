

# QQ 聊天霸王 V2.06 脱壳破解一条龙

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-622.htm`](https://www.pediy.com/kssd/pediy07/pediy7-622.htm)

【破文标题】：QQ 聊天霸王 V2.06 脱壳破解一条龙  
  
【破文作者】：KuNgBiM\[DFCG\]  
  
【作者邮箱】：gb\_1227@163.com  
  
【软件名称】：QQ 聊天霸王 V2.06  
  
【加密保护】：ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov  
  
【编译语言】：Microsoft Visual Basic 5.0 / 6.0  
  
【调试环境】：WinXP、PEiD、Ollydbg、LordPE、ImportREC  
  
【破解日期】：2005-08-15  
  
【破解目的】：推广使用 Ollydbg 手动脱壳  
  
【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
—————————————————————————————————  
【脱壳过程】：   
  
一、脱壳  
  
  
侦壳脱壳：用 PEiD 查壳，ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov 加壳。  
  
设置 Ollydbg 忽略除了“内存访问异常”之外的所有其它异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  
  
Ollydbg 载入主程序：  
            
00401000 >  68 01E04100         push QQ 聊天霸.0041E001           // OD 载入后停在这里  
00401005    E8 01000000         call QQ 聊天霸.0040100B  
0040100A    C3                  retn  
0040100B    C3                  retn  
0040100C    DE4F A4             fimul word ptr ds:\[edi\-5C\]  
0040100F    BB D5CE976E         mov ebx,6E97CED5  
00401014    E7 BB               out 0BB,eax  
  
F9 运行，程序会中断在异常处，Shift+F9 通过异常：  
  
00D8311C    3100                xor dword ptr ds:\[eax\],eax       // 第一次异常，Shift+F9  
00D8311E    EB 01               jmp short 00D83121  
00D83120    68 648F0500         push 58F64  
00D83125    0000                add byte ptr ds:\[eax\],al  
00D83127    00EB                add bl,ch  
00D83129    02E8                add ch,al  
00D8312B    0158 68             add dword ptr ds:\[eax+68\],ebx  
00D8312E    AC                  lods byte ptr ds:\[esi\]  
00D8312F    A9 D7006858         test eax,586800D7  
00D83134    31D8                xor eax,ebx  
00D83136    0068 5C             add byte ptr ds:\[eax+5C\],ch  
00D83139    27                  daa  
00D8313A    D800                fadd dword ptr ds:\[eax\]  
00D8313C    68 5824D800         push 0D82458  
00D83141    68 501ED800         push 0D81E50  
00D83146    68 6018D800         push 0D81860  
00D8314B    68 042BD800         push 0D82B04  
00D83150    C3                  retn  
00D83151    5D                  pop ebp  
00D83152    C2 0400             retn 4  
  
00D82B79    3100                xor dword ptr ds:\[eax\],eax       // 第二次异常，Shift+F9  
00D82B7B    EB 01               jmp short 00D82B7E  
00D82B7D    68 648F0500         push 58F64  
00D82B82    0000                add byte ptr ds:\[eax\],al  
00D82B84    00EB                add bl,ch  
00D82B86    02E8                add ch,al  
00D82B88    0158 E8             add dword ptr ds:\[eax\-18\],ebx  
00D82B8B  ^ 71 E4               jno short 00D82B71  
00D82B8D    FFFF                ???                                               ; 未知命令  
00D82B8F    E8 25000000         call 00D82BB9  
00D82B94    8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00D82B98    8380 B8000000 02    add dword ptr ds:\[eax+B8\],2  
00D82B9F    51                  push ecx  
00D82BA0    31C9                xor ecx,ecx  
00D82BA2    8948 04             mov dword ptr ds:\[eax+4\],ecx  
00D82BA5    8948 08             mov dword ptr ds:\[eax+8\],ecx  
00D82BA8    8948 0C             mov dword ptr ds:\[eax+C\],ecx  
00D82BAB    8948 10             mov dword ptr ds:\[eax+10\],ecx  
00D82BAE    C740 18 55010000    mov dword ptr ds:\[eax+18\],155  
00D82BB5    59                  pop ecx  
00D82BB6    31C0                xor eax,eax  
00D82BB8    C3                  retn  
  
00D82BC1    3100                xor dword ptr ds:\[eax\],eax       // 第三次异常，Shift+F9  
00D82BC3    64:8F05 00000000    pop dword ptr fs:\[0\]  
00D82BCA    58                  pop eax  
00D82BCB    E8 88FAFEFF         call 00D72658  
00D82BD0    A1 6C55D800         mov eax,dword ptr ds:\[D8556C\]  
00D82BD5    8B00                mov eax,dword ptr ds:\[eax\]  
00D82BD7    83E8 0D             sub eax,0D  
00D82BDA    8B15 6C55D800       mov edx,dword ptr ds:\[D8556C\]  
00D82BE0    8B12                mov edx,dword ptr ds:\[edx\]  
00D82BE2    8B12                mov edx,dword ptr ds:\[edx\]  
  
00D82EBE    3100                xor dword ptr ds:\[eax\],eax       // 第四次异常，Shift+F9  
00D82EC0    EB 01               jmp short 00D82EC3  
00D82EC2    68 648F0500         push 58F64  
00D82EC7    0000                add byte ptr ds:\[eax\],al  
00D82EC9    00EB                add bl,ch  
00D82ECB    02E8                add ch,al  
00D82ECD    0158 E8             add dword ptr ds:\[eax\-18\],ebx  
00D82ED0  ^ E0 E7               loopdne short 00D82EB9  
00D82ED2    FFFF                ???                                               ; 未知命令  
00D82ED4    33C0                xor eax,eax  
00D82ED6    5A                  pop edx  
00D82ED7    59                  pop ecx  
00D82ED8    59                  pop ecx  
00D82ED9    64:8910             mov dword ptr fs:\[eax\],edx  
00D82EDC    68 F42ED800         push 0D82EF4  
00D82EE1    8D85 F8FEFFFF       lea eax,dword ptr ss:\[ebp\-108\]  
00D82EE7    E8 5405FFFF         call 00D73440  
00D82EEC    C3                  retn  
  
00D818EA    3100                xor dword ptr ds:\[eax\],eax       // 第五次异常，Shift+F9  
00D818EC    EB 01               jmp short 00D818EF  
00D818EE    68 648F0500         push 58F64  
00D818F3    0000                add byte ptr ds:\[eax\],al  
00D818F5    00EB                add bl,ch  
00D818F7    02E8                add ch,al  
00D818F9    0158 A1             add dword ptr ds:\[eax\-5F\],ebx  
00D818FC    6C                  ins byte ptr es:\[edi\],dx  
00D818FD    55                  push ebp  
00D818FE    D800                fadd dword ptr ds:\[eax\]  
00D81900    8B00                mov eax,dword ptr ds:\[eax\]  
00D81902    8D70 24             lea esi,dword ptr ds:\[eax+24\]  
00D81905    B2 01               mov dl,1  
00D81907    A1 3CBCD700         mov eax,dword ptr ds:\[D7BC3C\]  
00D8190C    E8 F710FFFF         call 00D72A08  
00D81911    8BD8                mov ebx,eax  
00D81913    EB 51               jmp short 00D81966  
  
00D819C1    3100                xor dword ptr ds:\[eax\],eax       // 第六次异常，Shift+F9  
00D819C3    EB 01               jmp short 00D819C6  
00D819C5    68 648F0500         push 58F64  
00D819CA    0000                add byte ptr ds:\[eax\],al  
00D819CC    00EB                add bl,ch  
00D819CE    02E8                add ch,al  
00D819D0    0158 5A             add dword ptr ds:\[eax+5A\],ebx  
00D819D3    5D                  pop ebp  
00D819D4    5F                  pop edi  
00D819D5    5E                  pop esi  
00D819D6    5B                  pop ebx  
00D819D7    C3                  retn  
  
00D81EDB    3100                xor dword ptr ds:\[eax\],eax       // 第七次异常，Shift+F9  
00D81EDD    EB 01               jmp short 00D81EE0  
00D81EDF    68 648F0500         push 58F64  
00D81EE4    0000                add byte ptr ds:\[eax\],al  
00D81EE6    00EB                add bl,ch  
00D81EE8    02E8                add ch,al  
00D81EEA    0158 EB             add dword ptr ds:\[eax\-15\],ebx  
00D81EED    6A E8               push -18  
00D81EEF    1906                sbb dword ptr ds:\[esi\],eax  
00D81EF1    FFFF                ???                                               ; 未知命令  
00D81EF3    8BF0                mov esi,eax  
00D81EF5    8B03                mov eax,dword ptr ds:\[ebx\]  
00D81EF7    0345 EC             add eax,dword ptr ss:\[ebp\-14\]  
00D81EFA    8945 FC             mov dword ptr ss:\[ebp\-4\],eax  
00D81EFD    8B4B 04             mov ecx,dword ptr ds:\[ebx+4\]  
  
00D81FBB    3100                xor dword ptr ds:\[eax\],eax       // 第八次异常，Shift+F9  
00D81FBD    EB 01               jmp short 00D81FC0  
00D81FBF    68 648F0500         push 58F64  
00D81FC4    0000                add byte ptr ds:\[eax\],al  
00D81FC6    00EB                add bl,ch  
00D81FC8    02E8                add ch,al  
00D81FCA    0158 5F             add dword ptr ds:\[eax+5F\],ebx  
00D81FCD    5E                  pop esi  
00D81FCE    5B                  pop ebx  
00D81FCF    8BE5                mov esp,ebp  
00D81FD1    5D                  pop ebp  
00D81FD2    C3                  retn  
00D81FD3    00FF                add bh,bh  
00D81FD5    FFFF                ???                                               ; 未知命令  
  
00D824AE    3100                xor dword ptr ds:\[eax\],eax       // 第九次异常，Shift+F9  
00D824B0    EB 01               jmp short 00D824B3  
00D824B2    68 648F0500         push 58F64  
00D824B7    0000                add byte ptr ds:\[eax\],al  
00D824B9    00EB                add bl,ch  
00D824BB    02E8                add ch,al  
00D824BD    0158 A1             add dword ptr ds:\[eax\-5F\],ebx  
00D824C0    6C                  ins byte ptr es:\[edi\],dx  
00D824C1    55                  push ebp  
00D824C2    D800                fadd dword ptr ds:\[eax\]  
00D824C4    8B00                mov eax,dword ptr ds:\[eax\]  
00D824C6    8B00                mov eax,dword ptr ds:\[eax\]  
00D824C8    8B15 6055D800       mov edx,dword ptr ds:\[D85560\]  
00D824CE    8902                mov dword ptr ds:\[edx\],eax  
00D824D0    B2 01               mov dl,1  
00D824D2    A1 E812D800         mov eax,dword ptr ds:\[D812E8\]  
  
00D82684    3100                xor dword ptr ds:\[eax\],eax       // 第十次异常，Shift+F9  
00D82686    EB 01               jmp short 00D82689  
00D82688    68 648F0500         push 58F64  
00D8268D    0000                add byte ptr ds:\[eax\],al  
00D8268F    00EB                add bl,ch  
00D82691    02E8                add ch,al  
00D82693    0158 8B             add dword ptr ds:\[eax\-75\],ebx  
00D82696    5D                  pop ebp  
00D82697    E4 85               in al,85  
00D82699    DB                  ???                                               ; 未知命令  
00D8269A    75 09               jnz short 00D826A5  
00D8269C    33C0                xor eax,eax  
00D8269E    A3 786DD800         mov dword ptr ds:\[D86D78\],eax  
00D826A3    EB 3C               jmp short 00D826E1  
00D826A5    A1 6C55D800         mov eax,dword ptr ds:\[D8556C\]  
  
00D831D4    3100                xor dword ptr ds:\[eax\],eax       // 第十一次异常，Shift+F9  
00D831D6    EB 01               jmp short 00D831D9  
00D831D8    68 648F0500         push 58F64  
00D831DD    0000                add byte ptr ds:\[eax\],al  
00D831DF    00EB                add bl,ch  
00D831E1    02E8                add ch,al  
00D831E3    0158 8D             add dword ptr ds:\[eax\-73\],ebx  
00D831E6    45                  inc ebp  
00D831E7    F8                  clc  
00D831E8    E8 0BE5FFFF         call 00D816F8  
00D831ED    8B45 F8             mov eax,dword ptr ss:\[ebp\-8\]  
00D831F0    E8 A304FFFF         call 00D73698  
00D831F5    50                  push eax  
00D831F6    8B45 F8             mov eax,dword ptr ss:\[ebp\-8\]  
00D831F9    E8 5E06FFFF         call 00D7385C  
  
00D83262    3100                xor dword ptr ds:\[eax\],eax       // 第十二次异常，Shift+F9  
00D83264    EB 01               jmp short 00D83267  
00D83266    68 648F0500         push 58F64  
00D8326B    0000                add byte ptr ds:\[eax\],al  
00D8326D    00EB                add bl,ch  
00D8326F    02E8                add ch,al  
00D83271    0158 B2             add dword ptr ds:\[eax\-4E\],ebx  
00D83274    01A1 E812D800       add dword ptr ds:\[ecx+D812E8\],esp  
00D8327A    E8 C5E0FFFF         call 00D81344  
00D8327F    8945 FC             mov dword ptr ss:\[ebp\-4\],eax  
  
00D83542    3100                xor dword ptr ds:\[eax\],eax       // 第十三次异常，Shift+F9  
00D83544    EB 01               jmp short 00D83547  
00D83546    68 648F0500         push 58F64  
00D8354B    0000                add byte ptr ds:\[eax\],al  
00D8354D    00EB                add bl,ch  
00D8354F    02E8                add ch,al  
00D83551    0158 A1             add dword ptr ds:\[eax\-5F\],ebx  
00D83554    44                  inc esp  
00D83555    54                  push esp  
00D83556    D800                fadd dword ptr ds:\[eax\]  
00D83558    8038 00             cmp byte ptr ds:\[eax\],0  
00D8355B    0F84 18010000       je 00D83679  
00D83561    33C0                xor eax,eax  
  
00D83812    3100                xor dword ptr ds:\[eax\],eax       // 第十四次异常，Shift+F9  
00D83814    EB 01               jmp short 00D83817  
00D83816    68 648F0500         push 58F64  
00D8381B    0000                add byte ptr ds:\[eax\],al  
00D8381D    00EB                add bl,ch  
00D8381F    02E8                add ch,al  
00D83821    0158 A1             add dword ptr ds:\[eax\-5F\],ebx  
00D83824    D054D8 00           rcl byte ptr ds:\[eax+ebx\*8\],1  
00D83828    8078 30 00          cmp byte ptr ds:\[eax+30\],0  
00D8382C    75 0B               jnz short 00D83839  
00D8382E    A1 D054D800         mov eax,dword ptr ds:\[D854D0\]  
00D83833    8078 31 00          cmp byte ptr ds:\[eax+31\],0  
00D83837    74 20               je short 00D83859  
00D83839    A1 CC55D800         mov eax,dword ptr ds:\[D855CC\]  
00D8383E    8378 24 00          cmp dword ptr ds:\[eax+24\],0  
  
00D838CD    3100                xor dword ptr ds:\[eax\],eax       // 第十五次异常，Shift+F9  
00D838CF    EB 01               jmp short 00D838D2  
00D838D1    68 648F0500         push 58F64  
00D838D6    0000                add byte ptr ds:\[eax\],al  
00D838D8    00EB                add bl,ch  
00D838DA    02E8                add ch,al  
00D838DC    0158 33             add dword ptr ds:\[eax+33\],ebx  
00D838DF    C05A 59 59          rcr byte ptr ds:\[edx+59\],59  
00D838E3    64:8910             mov dword ptr fs:\[eax\],edx  
00D838E6    68 0339D800         push 0D83903  
00D838EB    8D45 EC             lea eax,dword ptr ss:\[ebp\-14\]  
00D838EE    E8 4DFBFEFF         call 00D73440  
00D838F3    8D45 F8             lea eax,dword ptr ss:\[ebp\-8\]  
00D838F6    E8 45FBFEFF         call 00D73440  
00D838FB    C3                  retn  
  
00D82AA6    3100                xor dword ptr ds:\[eax\],eax       // 最后一次异常  
00D82AA8    64:8F05 00000000    pop dword ptr fs:\[0\]  
00D82AAF    58                  pop eax  
00D82AB0    833D 786DD800 00    cmp dword ptr ds:\[D86D78\],0  
00D82AB7    74 14               je short 00D82ACD  
00D82AB9    6A 0C               push 0C  
00D82ABB    B9 786DD800         mov ecx,0D86D78  
00D82AC0    8D45 F8             lea eax,dword ptr ss:\[ebp\-8\]  
00D82AC3    BA 04000000         mov edx,4  
00D82AC8    E8 63E1FFFF         call 00D80C30  
00D82ACD    FF75 FC             push dword ptr ss:\[ebp\-4\]  
00D82AD0    FF75 F8             push dword ptr ss:\[ebp\-8\]  
00D82AD3    8B45 F4             mov eax,dword ptr ss:\[ebp\-C\]  
00D82AD6    8338 00             cmp dword ptr ds:\[eax\],0  
00D82AD9    74 02               je short 00D82ADD  
00D82ADB    FF30                push dword ptr ds:\[eax\]  
00D82ADD    FF75 F0             push dword ptr ss:\[ebp\-10\]  
00D82AE0    FF75 EC             push dword ptr ss:\[ebp\-14\]  
00D82AE3    C3                  retn                            // F2 直接在这下断！ Shift+F9  
  
中断后取消断点，F7 返回到：  
  
00D91CCC   /E9 54080000         jmp 00D92525                    // 返回到这里，Alt + M 打开内存映射  
00D91CD1   |DA71 97             fidiv dword ptr ds:\[ecx\-69\]  
00D91CD4   |BE 0708F3C5         mov esi,C5F30807  
00D91CD9   |71 51               jno short 00D91D2C  
00D91CDB   |6BBD 2EBEC2B4 A2    imul edi,dword ptr ss:\[ebp+B4C2BE2E\],-5E  
00D91CE2   |8ACC                mov cl,ah  
00D91CE4   |C3                  retn  
00D91CE5   |DAB0 F1112243       fidiv dword ptr ds:\[eax+432211F1\]  
00D91CEB   |8562 94             test dword ptr ds:\[edx\-6C\],esp  
00D91CEE   |E5 AE               in eax,0AE  
00D91CF0   |7B A9               jpo short 00D91C9B  
00D91CF2   |2C A8               sub al,0A8  
00D91CF4   |8826                mov byte ptr ds:\[esi\],ah  
00D91CF6   |D9ED                fldln2  
00D91CF8   |0A7488 1F           or dh,byte ptr ds:\[eax+ecx\*4+1F\]  
00D91CFC   |AB                  stos dword ptr es:\[edi\]  
00D91CFD   |2E:C1A0 4CB7E3D9 23 shl dword ptr cs:\[eax+D9E3B74C\],23  
00D91D05   |E7 B4               out 0B4,eax  
00D91D07  ^|79 B4               jns short 00D91CBD  
00D91D09   |003406              add byte ptr ds:\[esi+eax\],dh  
  
———————————————————————————————————  
内存映射，项目 21  
 地址=00401000             // 在此处 F2 设置访问中断，F9 运行，直达 OEP！  
 大小=0001A000 (106496.)  
 物主=QQ 聊天霸 00400000  
 区段=  
 包含=code  
 类型=Imag 01001002  
 访问=R  
 初始访问=RWE  
———————————————————————————————————  
  
00401630    68 101D4000         push QQ 聊天霸.00401D10          // 中断在此！在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程  
00401635    E8 EEFFFFFF         call QQ 聊天霸.00401628  
0040163A    0000                add byte ptr ds:\[eax\],al  
0040163C    0000                add byte ptr ds:\[eax\],al  
0040163E    0000                add byte ptr ds:\[eax\],al  
00401640    3000                xor byte ptr ds:\[eax\],al  
00401642    0000                add byte ptr ds:\[eax\],al  
00401644    40                  inc eax  
00401645    0000                add byte ptr ds:\[eax\],al  
00401647    0000                add byte ptr ds:\[eax\],al  
00401649    0000                add byte ptr ds:\[eax\],al  
0040164B    006CBF E7           add byte ptr ds:\[edi+edi\*4-19\],ch  
0040164F    43                  inc ebx  
00401650    C9                  leave  
00401651    47                  inc edi  
00401652    D911                fst dword ptr ds:\[ecx\]  
00401654    889400 E04C3D87     mov byte ptr ds:\[eax+eax+873D4CE0\],dl  
  
运行 ImportREC 1.6，选择这个进程。把 OEP 改为 00001630，点击“IAT AutoSearch”，然后点击“Get Imports”，函数指针无效。  
再点“Show Invalid”,在无效函数上，鼠标右键选择“Trace Level 1(Disasm)”自动检测修复，这下指针有效了，Fix Dump !!  
正常运行！看来这次 ASProtect 对这个输入表没有动太多的手脚。Microsoft Visual Basic 5.0 / 6.0 编写。  
  
用 LordPE 删除 data、adata 共 2 个区段，然后再用 LordPE“重建 PE”，程序就精简了不少。93.5 KB -> 188 KB -> 113 KB  
  
—————————————————————————————————   
二、破解   
  
  
脱壳后的程序在启动时还是有“延时 NAG 提示框”，去掉 NAG 岂不很顺眼？  
  
分析一下，这个 NAG 应该是程序的注册验证所致，所以。。。“直捣黄龙”  
  
下断：bpx \_\_vbaStrCmp  
  
F9 运行，断下：  
  
00417B08    FF15 BCE04200       call dword ptr ds:\[<&msvbvm60.#406>\]        // 这里中断，F8 单步  ; msvbvm60.\_\_vbaStrCmp  
00417B0E    85C0                test eax,eax                                // eax 为 0 则跳  
00417B10    0F84 08070000       je dumped\_.0041821E                         // 不能条，所以 nop 掉！★验证爆破点★  
00417B16    8D95 C4FEFFFF       lea edx,dword ptr ss:\[ebp\-13C\]  
  
代码修改完后保存文件，运行！OK，启动注册框没了，主要的功能限制也“烟消云散”了！破解成功！  
  
—————————————————————————————————   
三、总结  
  
  
其实遇到 ASProtect 这类世界名壳来说，也不必畏惧，只要你有耐心，一定会和我们一样，做得更好！  
  
【完美爆破点】  
  
00417B10    0F84 08070000       je dumped\_.0041821E           // nop 掉！  
  
\------------------------------------------------------------------  
  
★★★★★★ 纪念中华人民共和国 抗日战争胜利 60 周年 ★★★★★★  
  
\--------------------------------------------------------------------------  
  
版权所有（C）2005 KuNgBiM\[DFCG\]         Copyright (C) 2005 KuNgBiM\[DFCG\]  
  
\--------------------------------------------------------------------------  
          UnPacked\_Cracked By KuNgBiM\[DFCG\]  
  
                     2005-08-15     
  
                     15:10:15 PM 

 4dd466d0e1e0cc123afbe504da2462ca