

# CrypKey V5.4

> 原文：[`www.pediy.com/kssd/pediy06/pediy6472.htm`](https://www.pediy.com/kssd/pediy06/pediy6472.htm)

CrypKey V5.4 脱壳——ProCalc.exe   
    
    
    
下载地址：  http://www.fcgchina.com/bbs/index.php?mods=upload&action=downFile&forumID=1&postId=14&replyId=1    
软件大小：  207 KB   
  
【软件简介】：老外的某个空气计算小程序   
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、Ollydbg、PEiD、LordPE、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
              
     
    
这个小程序是\[FCG\]的 laoqian 兄提出来的，CrypKey V5.4 保护，没有 Key。   
            
CrypKey 关键是密码学保护，如果有 Key 的话，脱壳很容易，几分钟就搞定了。没有 Key，只好想办法跳过注册验证了。   
  
  
0042883E     E8 00000000         call ProCalc.00428843//进入 OD 后停在这    
00428843     58                  pop eax   
00428844     83E8 05             sub eax,5   
00428847     50                  push eax   
00428848     5F                  pop edi   
00428849     57                  push edi   
0042884A     8BF7                mov esi,edi   
0042884C     81EF 1F260000       sub edi,261F   
00428852     83C6 39             add esi,39   
00428855     BA 00000000         mov edx,0   
0042885A     8BDF                mov ebx,edi   
0042885C     B9 0B000000         mov ecx,0B   
00428861     8B06                mov eax,dword ptr ds:\[esi\]   
00428863     3303                xor eax,dword ptr ds:\[ebx\]   
00428865     8906                mov dword ptr ds:\[esi\],eax   
00428867     83C6 04             add esi,4   
0042886A     83C3 04             add ebx,4   
0042886D     E2 F2               loopd short ProCalc.00428861//解码    
0042886F     42                  inc edx   
00428870     83FA 05             cmp edx,5   
00428873     74 02               je short ProCalc.00428877   
00428875     EB E3               jmp short ProCalc.0042885A   
00428877     58                  pop eax   
00428878     50                  push eax   
00428879     83E8 2D             sub eax,2D   
0042887C     50                  push eax   
0042887D     E8 97010000         call <jmp.&KERNEL32.GetModuleHandleA>   
00428882     5B                  pop ebx   
00428883     53                  push ebx   
00428884     83EB 1C             sub ebx,1C   
00428887     8903                mov dword ptr ds:\[ebx\],eax   
00428889     5B                  pop ebx   
0042888A     53                  push ebx   
0042888B     83EB 49             sub ebx,49   
0042888E     53                  push ebx   
0042888F     83C3 2D             add ebx,2D   
00428892     FF33                push dword ptr ds:\[ebx\]   
00428894     E8 7A010000         call <jmp.&KERNEL32.GetProcAddress>   
00428899     5B                  pop ebx   
0042889A     53                  push ebx   
0042889B     83EB 3C             sub ebx,3C   
0042889E     8903                mov dword ptr ds:\[ebx\],eax   
004288A0     5B                  pop ebx   
004288A1     53                  push ebx   
004288A2     83EB 38             sub ebx,38   
004288A5     53                  push ebx   
004288A6     83EB 04             sub ebx,4   
004288A9     FF13                call dword ptr ds:\[ebx\]   
004288AB     5B                  pop ebx   
004288AC     53                  push ebx   
004288AD     83EB 20             sub ebx,20   
004288B0     8903                mov dword ptr ds:\[ebx\],eax   
004288B2     5B                  pop ebx   
004288B3     53                  push ebx   
004288B4     81EB 040C0000       sub ebx,0C04   
004288BA     53                  push ebx   
004288BB     81C3 E80B0000       add ebx,0BE8   
004288C1     FF33                push dword ptr ds:\[ebx\]   
004288C3     E8 4B010000         call <jmp.&KERNEL32.GetProcAddress>   
004288C8     FFD0                call eax   
004288CA     A9 00000080         test eax,80000000   
004288CF     74 1A               je short ProCalc.004288EB   
004288D1     5B                  pop ebx   
004288D2     53                  push ebx   
004288D3     81EB C90B0000       sub ebx,0BC9   
004288D9     53                  push ebx   
004288DA     81C3 8D0B0000       add ebx,0B8D   
004288E0     FF13                call dword ptr ds:\[ebx\]   
004288E2     83F8 00             cmp eax,0   
004288E5     0F84 D9000000       je ProCalc.004289C4   
004288EB     5B                  pop ebx   
004288EC     53                  push ebx   
004288ED     83EB 56             sub ebx,56   
004288F0     53                  push ebx   
004288F1     83C3 1A             add ebx,1A   
004288F4     FF13                call dword ptr ds:\[ebx\]   
004288F6     83F8 00             cmp eax,0   
004288F9     0F84 BB000000       je ProCalc.004289BA   
004288FF     5B                  pop ebx   
00428900     53                  push ebx   
00428901     83EB 5A             sub ebx,5A   
00428904     8903                mov dword ptr ds:\[ebx\],eax   
00428906     5B                  pop ebx   
00428907     53                  push ebx   
00428908     83EB 6B             sub ebx,6B   
0042890B     53                  push ebx   
0042890C     83C3 4F             add ebx,4F   
0042890F     FF33                push dword ptr ds:\[ebx\]   
00428911     E8 FD000000         call <jmp.&KERNEL32.GetProcAddress>   
00428916     6A 00               push 0   
00428918     FFD0                call eax   
0042891A     5B                  pop ebx   
0042891B     53                  push ebx   
0042891C     83EB 6F             sub ebx,6F   
0042891F     8903                mov dword ptr ds:\[ebx\],eax   
00428921     5B                  pop ebx   
00428922     53                  push ebx   
00428923     81EB DE000000       sub ebx,0DE   
00428929     53                  push ebx   
0042892A     81C3 C2000000       add ebx,0C2   
00428930     FF33                push dword ptr ds:\[ebx\]   
00428932     E8 DC000000         call <jmp.&KERNEL32.GetProcAddress>   
00428937     FFD0                call eax   
00428939     5B                  pop ebx   
0042893A     53                  push ebx   
0042893B     81EB CE000000       sub ebx,0CE   
00428941     8903                mov dword ptr ds:\[ebx\],eax   
00428943     5B                  pop ebx   
00428944     53                  push ebx   
00428945     81EB 80000000       sub ebx,80   
0042894B     53                  push ebx   
0042894C     83C3 26             add ebx,26   
0042894F     FF33                push dword ptr ds:\[ebx\]   
00428951     E8 BD000000         call <jmp.&KERNEL32.GetProcAddress>   
00428956     5B                  pop ebx   
00428957     53                  push ebx   
00428958     8BD3                mov edx,ebx   
0042895A     81EB 3E280000       sub ebx,283E   
00428960     8B0B                mov ecx,dword ptr ds:\[ebx\]   
00428962     2BD1                sub edx,ecx   
00428964     52                  push edx   
00428965     81EB 90D8FFFF       sub ebx,-2770   
0042896B     FF33                push dword ptr ds:\[ebx\]   
0042896D     81EB 70270000       sub ebx,2770   
00428973     53                  push ebx   
00428974     81C3 CF270000       add ebx,27CF   
0042897A     FF33                push dword ptr ds:\[ebx\]   
0042897C     FFD0                call eax; cki32h.SecurityProc//验证注册,还原 OEP 处代码 ★    
0042897E     5B                  pop ebx   
0042897F     53                  push ebx   
00428980     81EB 2E0D0000       sub ebx,0D2E   
00428986     53                  push ebx   
00428987     81C3 D40C0000       add ebx,0CD4   
0042898D     FF33                push dword ptr ds:\[ebx\]   
0042898F     E8 7F000000         call <jmp.&KERNEL32.GetProcAddress>   
00428994     5B                  pop ebx   
00428995     53                  push ebx   
00428996     50                  push eax   
00428997     81EB 040D0000       sub ebx,0D04   
0042899D     53                  push ebx   
0042899E     6A 00               push 0   
004289A0     E8 74000000         call <jmp.&KERNEL32.GetModuleHandleA>   
004289A5     5B                  pop ebx   
004289A6     59                  pop ecx   
004289A7     53                  push ebx   
004289A8     50                  push eax   
004289A9     FFD1                call ecx; cki32h.ResolveImports//还原输入表 ★ ◢附注：肆◣    
004289AB     5B                  pop ebx   
004289AC     8BC3                mov eax,ebx   
004289AE     81EB 3E280000       sub ebx,283E   
004289B4     8B0B                mov ecx,dword ptr ds:\[ebx\]   
004289B6     2BC1                sub eax,ecx//EAX=0042883E - 00026F36=00401908    
004289B8     FFE0                jmp eax ; ProCalc.00401908//飞向光明之巅！    
  
  
————————————————————————   
  
  
004018F0     FF25 CC104000       jmp dword ptr ds:\[4010CC\]  ; MSVBVM60.EVENT\_SINK\_QueryInterface   
004018F6     FF25 84104000       jmp dword ptr ds:\[401084\]  ; MSVBVM60.EVENT\_SINK\_AddRef   
004018FC     FF25 BC104000       jmp dword ptr ds:\[4010BC\]  ; MSVBVM60.EVENT\_SINK\_Release   
00401902     FF25 24114000       jmp dword ptr ds:\[401124\]  ; MSVBVM60.ThunRTMain   
00401908     68 DC2B4000         push ProCalc.00402BDC//用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程    
0040190D     E8 F0FFFFFF         call ProCalc.00401902      ; jmp to MSVBVM60.ThunRTMain   
  
  
运行 ImportREC，选择这个进程。把 OEP 改为 00001908，点 IT AutoSearch，点“Get Import”。   
用 LordPE 删除最后的 Hi, mom!区段。FixDump，再用 FileScan 优化一下，正常运行！   
  
  
—————————————————————————————————   
跟进 0042897C  call eax  ; cki32h.SecurityProc//验证注册    
  
  
003AADBA     E8 8D70FFFF         call cki32h.003A1E4C//跟进 ◢附注：壹◣    
003AADBF     83C4 08             add esp,8   
003AADC2     84C0                test al,al   
003AADC4     74 15               je short cki32h.003AADDB//JMP    
003AADC6     6A 00               push 0   
003AADC8     8D85 E8FEFFFF       lea eax,dword ptr ss:\[ebp\-118\]   
003AADCE     50                  push eax   
003AADCF     E8 7870FFFF         call cki32h.003A1E4C   
003AADD4     83C4 08             add esp,8   
003AADD7     84C0                test al,al   
003AADD9     75 EB               jnz short cki32h.003AADC6   
003AADDB     C645 FB 00          mov byte ptr ss:\[ebp\-5\],0   
003AADDF     E8 1068FFFF         call cki32h.003A15F4//修改②：mov al,01 ★    
003AADE4     84C0                test al,al   
003AADE6     0F84 CB000000       je cki32h.003AAEB7   
003AADEC     6A 00               push 0   
003AADEE     E8 C7DD0000         call cki32h.003B8BBA   
003AADF3     59                  pop ecx   
003AADF4     8945 F4             mov dword ptr ss:\[ebp\-C\],eax//修改③    
003AADF7     837D F4 00          cmp dword ptr ss:\[ebp\-C\],0   
003AADFB     7D 12               jge short cki32h.003AAE0F   
  
修改③  003AADF4 处代码修改为：   
003AADF4     C70590C63C000200000 mov dword ptr ds:\[3CC690\],2   
003AADFE     EB 0F               jmp short cki32h.003AAE0F   
  
  
003AADFD     8B55 F4             mov edx,dword ptr ss:\[ebp\-C\]   
003AAE00     52                  push edx   
003AAE01     6A 01               push 1   
003AAE03     6A 00               push 0   
003AAE05     E8 AEBFFFFF         call cki32h.003A6DB8   
003AAE0A     E8 39FEFFFF         call cki32h.003AAC48   
003AAE0F     E8 C8DD0000         call cki32h.003B8BDC//◢附注：贰◣    
003AAE14     8945 F0             mov dword ptr ss:\[ebp\-10\],eax//修改④：EAX=1 ★    
003AAE17     837D F0 00          cmp dword ptr ss:\[ebp\-10\],0   
003AAE1B     7E 12               jle short cki32h.003AAE2F   
003AAE1D     C645 FB 01          mov byte ptr ss:\[ebp\-5\],1   
003AAE21     E8 5A99FFFF         call cki32h.003A4780   
003AAE26     84C0                test al,al   
003AAE28     75 05               jnz short cki32h.003AAE2F//JMP    
003AAE2A     E8 19FEFFFF         call cki32h.003AAC48   
003AAE2F     6A 01               push 1   
003AAE31     8D4D EC             lea ecx,dword ptr ss:\[ebp\-14\]   
003AAE34     51                  push ecx   
003AAE35     E8 6DDD0000         call cki32h.003B8BA7   
003AAE3A     83C4 08             add esp,8   
003AAE3D     8945 F4             mov dword ptr ss:\[ebp\-C\],eax//修改⑤：eax＝0  ★    
003AAE40     837D F4 00          cmp dword ptr ss:\[ebp\-C\],0   
003AAE44     74 18               je short cki32h.003AAE5E//JMP    
003AAE46     837D F4 00          cmp dword ptr ss:\[ebp\-C\],0   
003AAE4A     7D 0D               jge short cki32h.003AAE59   
003AAE4C     8B45 F4             mov eax,dword ptr ss:\[ebp\-C\]   
003AAE4F     50                  push eax   
003AAE50     6A 01               push 1   
003AAE52     6A 00               push 0   
003AAE54     E8 5FBFFFFF         call cki32h.003A6DB8   
003AAE59     E8 EAFDFFFF         call cki32h.003AAC48//ExitProcess    
003AAE5E     8B55 EC             mov edx,dword ptr ss:\[ebp\-14\]   
003AAE61     52                  push edx   
003AAE62     E8 9D67FFFF         call cki32h.003A1604   
003AAE67     59                  pop ecx   
003AAE68     84C0                test al,al   
003AAE6A     75 32               jnz short cki32h.003AAE9E//JMP    
003AAE6C     6A 00               push 0   
003AAE6E     8B0D 20D63C00       mov ecx,dword ptr ds:\[3CD620\]; cki32h.003C080C   
003AAE74     8B41 68             mov eax,dword ptr ds:\[ecx+68\]   
003AAE77     50                  push eax   
003AAE78     68 017F0000         push 7F01   
003AAE7D     6A 00               push 0   
003AAE7F     E8 A2430100         call cki32h.003BF226          ; jmp to USER32.LoadIconA   
003AAE84     50                  push eax   
003AAE85     8B15 20D63C00       mov edx,dword ptr ds:\[3CD620\] ; cki32h.003C080C   
003AAE8B     8B8A F4010000       mov ecx,dword ptr ds:\[edx+1F4\]   
003AAE91     51                  push ecx   
003AAE92     6A 00               push 0   
003AAE94     E8 C7BFFFFF         call cki32h.003A6E60   
003AAE99     E8 AAFDFFFF         call cki32h.003AAC48   
003AAE9E     A1 1CD63C00         mov eax,dword ptr ds:\[3CD61C\]   
003AAEA3     8B90 AF020000       mov edx,dword ptr ds:\[eax+2AF\]   
003AAEA9     F682 B2060000 40    test byte ptr ds:\[edx+6B2\],40   
003AAEB0     74 05               je short cki32h.003AAEB7//JMP    
003AAEB2     E8 8D82FFFF         call cki32h.003A3144   
003AAEB7     E8 20000000         call cki32h.003AAEDC   
003AAEBC     84C0                test al,al   
003AAEBE     75 05               jnz short cki32h.003AAEC5//JMP    
003AAEC0     E8 83FDFFFF         call cki32h.003AAC48   
003AAEC5     807D FB 00          cmp byte ptr ss:\[ebp\-5\],0   
003AAEC9     75 0A               jnz short cki32h.003AAED5//JMP    
003AAECB     E8 9ADC0000         call cki32h.003B8B6A   
003AAED0     E8 8376FFFF         call cki32h.003A2558   
003AAED5     8BE5                mov esp,ebp   
003AAED7     5D                  pop ebp   
003AAED8     C2 1000             retn 10//返回 0042897E    
  
  
—————————————————————————————————   
◢附注：壹◣ 跟进：003AADBA  call cki32h.003A1E4C   
  
  
003A1E4C     55                  push ebp   
003A1E4D     8BEC                mov ebp,esp   
003A1E4F     51                  push ecx   
003A1E50     E8 03070000         call cki32h.003A2558   
003A1E55     807D 0C 00          cmp byte ptr ss:\[ebp+C\],0   
003A1E59     74 05               je short cki32h.003A1E60   
003A1E5B     E8 70070000         call cki32h.003A25D0   
003A1E60     A1 1CD63C00         mov eax,dword ptr ds:\[3CD61C\]   
003A1E65     33C9                xor ecx,ecx   
003A1E67     8B90 AF020000       mov edx,dword ptr ds:\[eax+2AF\]   
003A1E6D     8A4A 1C             mov cl,byte ptr ds:\[edx+1C\]   
003A1E70     8B15 1CD63C00       mov edx,dword ptr ds:\[3CD61C\]   
003A1E76     51                  push ecx   
003A1E77     81C2 18010000       add edx,118   
003A1E7D     8B45 08             mov eax,dword ptr ss:\[ebp+8\]   
003A1E80     50                  push eax   
003A1E81     68 8E733C00         push cki32h.003C738E ; ASCII "%scki\_%c."   
003A1E86     52                  push edx   
003A1E87     E8 48D40100         call cki32h.003BF2D4  ; jmp to USER32.wsprintfA   
003A1E8C     83C4 10             add esp,10   
003A1E8F     8B0D 1CD63C00       mov ecx,dword ptr ds:\[3CD61C\]   
003A1E95     8B81 AF020000       mov eax,dword ptr ds:\[ecx+2AF\]   
003A1E9B     33D2                xor edx,edx   
003A1E9D     8A50 1C             mov dl,byte ptr ds:\[eax+1C\]   
003A1EA0     83FA 61             cmp edx,61   
003A1EA3     75 39               jnz short cki32h.003A1EDE   
003A1EA5     807D 0C 00          cmp byte ptr ss:\[ebp+C\],0   
003A1EA9     74 1C               je short cki32h.003A1EC7//修改①、JMP 003A1EC2 ★    
003A1EAB     E8 10110000         call cki32h.003A2FC0   
003A1EB0     E8 3F050000         call cki32h.003A23F4   
003A1EB5     48                  dec eax   
003A1EB6     75 05               jnz short cki32h.003A1EBD   
003A1EB8     B0 01               mov al,1   
003A1EBA     59                  pop ecx   
003A1EBB     5D                  pop ebp   
003A1EBC     C3                  retn   
003A1EBD     E8 A2040000         call cki32h.003A2364   
003A1EC2     33C0                xor eax,eax   
003A1EC4     59                  pop ecx   
003A1EC5     5D                  pop ebp   
003A1EC6     C3                  retn   
  
————————————————————————   
◢附注：贰◣ 003AAE0F  call cki32h.003B8BDC 里面的验证：   
  
  
003B9F01     53                  push ebx   
003B9F02     833D 90C63C00 00    cmp dword ptr ds:\[3CC690\],0//修改\[3CC690\]＝2    
003B9F09     75 07               jnz short cki32h.003B9F12//JMP    
003B9F0B     B8 98FFFFFF         mov eax,-68   
003B9F10     5B                  pop ebx   
003B9F11     C3                  retn   
003B9F12     833D 90C63C00 01    cmp dword ptr ds:\[3CC690\],1   
003B9F19     7E 3A               jle short cki32h.003B9F55   
003B9F1B     68 5CCB3C00         push cki32h.003CCB5C              ; ASCII "get\_num\_multi\_users"   
003B9F20     FF35 8CC63C00       push dword ptr ds:\[3CC68C\]   
003B9F26     E8 C94F0000         call cki32h.003BEEF4              ; jmp to kernel32.GetProcAddress   
003B9F2B     8BD8                mov ebx,eax   
003B9F2D     85DB                test ebx,ebx   
003B9F2F     75 07               jnz short cki32h.003B9F38   
003B9F31     B8 97FFFFFF         mov eax,-69   
003B9F36     5B                  pop ebx   
003B9F37     C3                  retn   
  
————————————————————————   
◢附注：叁 ◣还原输入表：   
  
  
003AB361     8B45 10             mov eax,dword ptr ss:\[ebp+10\]   
003AB364     F640 03 80          test byte ptr ds:\[eax+3\],80   
003AB368     74 21               je short cki32h.003AB38B   
003AB36A     8B55 10             mov edx,dword ptr ss:\[ebp+10\]   
003AB36D     0FB70A              movzx ecx,word ptr ds:\[edx\]   
003AB370     51                  push ecx   
003AB371     8B45 08             mov eax,dword ptr ss:\[ebp+8\]   
003AB374     50                  push eax   
003AB375     E8 7A3B0100         call cki32h.003BEEF4; jmp to kernel32.GetProcAddress   
003AB37A     8B55 14             mov edx,dword ptr ss:\[ebp+14\]   
003AB37D     8902                mov dword ptr ds:\[edx\],eax   
003AB37F     8B4D 14             mov ecx,dword ptr ss:\[ebp+14\]   
003AB382     8339 00             cmp dword ptr ds:\[ecx\],0   
003AB385     75 2A               jnz short cki32h.003AB3B1   
003AB387     33C0                xor eax,eax   
003AB389     5D                  pop ebp   
003AB38A     C3                  retn   
003AB38B     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]   
003AB38E     8B4D 10             mov ecx,dword ptr ss:\[ebp+10\]   
003AB391     0311                add edx,dword ptr ds:\[ecx\]   
003AB393     83C2 02             add edx,2   
003AB396     52                  push edx   
003AB397     8B45 08             mov eax,dword ptr ss:\[ebp+8\]   
003AB39A     50                  push eax   
003AB39B     E8 543B0100         call cki32h.003BEEF4 ; jmp to kernel32.GetProcAddress   
003AB3A0     8B55 14             mov edx,dword ptr ss:\[ebp+14\]   
003AB3A3     8902                mov dword ptr ds:\[edx\],eax   
003AB3A5     8B4D 14             mov ecx,dword ptr ss:\[ebp+14\]   
003AB3A8     8339 00             cmp dword ptr ds:\[ecx\],0   
003AB3AB     75 04               jnz short cki32h.003AB3B1   
003AB3AD     33C0                xor eax,eax   
003AB3AF     5D                  pop ebp   
003AB3B0     C3                  retn   
003AB3B1     8345 10 04          add dword ptr ss:\[ebp+10\],4   
003AB3B5     8345 14 04          add dword ptr ss:\[ebp+14\],4   
003AB3B9     8B55 10             mov edx,dword ptr ss:\[ebp+10\]   
003AB3BC     833A 00             cmp dword ptr ds:\[edx\],0   
003AB3BF     75 A0               jnz short cki32h.003AB361   
003AB3C1     B0 01               mov al,1   
003AB3C3     5D                  pop ebp   
003AB3C4     C3                  retn   
  
  
—————————————————————————————————                                      
         ,     \_/    
        /| \_.-~/            \_     ,        青春都一晌   
       ( /~   /              ~-.\_ |   
       \`\\  \_/                   ~ )          忍把浮名    
   \_-~~~-.)  )\_\_/;;,.          \_  //'    
  /'\_,   --~    ~~~-  ,;;\_\_\_(  (.-~~~-.        换了破解轻狂   
 \`~ \_( ,\_..-- (     ,;'' /    ~--   /.\_\`    
  /~~//'   /' \`~         ) /--..\_, )\_  \`~    
  "  \`~"  "      \`"      /~'\`    \`\\~~      
                         "     "   "~'  ""   
  
       
     Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]\[NUKE\]\[DCM\]   
  
                2004-03-25 19:00  
  
 4dd466d0e1e0cc123afbe504da2462ca