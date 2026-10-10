

# Softlocx V5.0.0.6 加壳的 OCX 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6739.htm`](https://www.pediy.com/kssd/pediy06/pediy6739.htm)

【软件简介】：SoftLocx comprises of an OCX control for developers wishing to integrate copy protection directly into supporting development languages. With the OCX, protection is achieved with 4 property settings and one line of code. Softlocx contains functionality to e-commerce enable and/or network protect software, and includes a code wizard for quick generation of protection code in your desired development language.  
                      
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：Win2000、Ollydbg V1.10、PEiD、LordPE、WinHex

【实例下载】：[`bbs.pediy.com/showthread.php?s=&threadid=688`](http://bbs.pediy.com/showthread.php?s=&threadid=6881)1  
               
—————————————————————————————————   
【脱壳过程】：  
            
        
SoftLocx 现在不多见了，PEiD 显示：Crunch/PE Heuristic -> Bit-Arts  
用其加壳的 EXE 文件脱壳不难，加壳的 OCX 和 DLL 文件脱壳多了步寻找重定位表数据。  
下面用 Softlocx V5.0.0.6 主程序自带的 Softlocx5.ocx 来演示一下脱壳过程。  
Ollydbg 无法“正常”中断在 Softlocx5.ocx 的 EP，所以稍微改了一下思路。  
—————————————————————————————————  
一、退出时获得 OEP  
               
               
设置 Ollydbg 忽略所有异常选项。载入 Softlocx5.ocx，F9 运行，DLL 加载完毕。  
现在关闭 Ollydbg DLL Loader 的小窗口，Ollydbg 暂停在壳入口。  
这里和 ASProtect 加壳的 dll 一样，DLL 卸载时会再次中断在外壳入口处！  
  
70079000     55                push ebp  
//暂停在这里  
70079001     E8 00000000       call Softlocx.70079006  
70079006     5D                pop ebp  
70079007     83ED 06           sub ebp,6  
7007900A     8BC5              mov eax,ebp  
7007900C     55                push ebp  
7007900D     60                pushad  
7007900E     89AD 8A340000     mov dword ptr ss:\[ebp+348A\],ebp  
70079014     2B85 65340000     sub eax,dword ptr ss:\[ebp+3465\]  
7007901A     8985 19250000     mov dword ptr ss:\[ebp+2519\],eax  
70079020     55                push ebp  
70079021     BB CD1E0000       mov ebx,1ECD  
70079026     03DD              add ebx,ebp  
70079028     53                push ebx  
70079029     64:67:FF36 0000   push dword ptr fs:\[0\]  
7007902F     64:67:8926 0000   mov dword ptr fs:\[0\],esp  
70079035     80BD 90360000 00  cmp byte ptr ss:\[ebp+3690\],0  
7007903C     75 09             jnz short Softlocx.70079047  
//跳转  
7007903E     C685 90360000 01  mov byte ptr ss:\[ebp+3690\],1  
70079045     EB 15             jmp short Softlocx.7007905C  
70079047     64:67:8F06 0000   pop dword ptr fs:\[0\]  
7007904D     83C4 08           add esp,8  
70079050     61                popad  
70079051     5D                pop ebp  
70079052     8B85 86340000     mov eax,dword ptr ss:\[ebp+3486\]  
70079058     5D                pop ebp  
70079059     FFE0              jmp eax    ; Softlocx.70026848  
//飞向光明之巅！ ^O^  
  
  
—————————————————————————————————  
二、重定位表信息  
  
  
现在下断：BP VirtualFree  
然后 Ctrl+F2 重新载入这个 OCX，直接中断在 VirtualFree 处  
0006FB64   700797A5   /CALL 到 VirtualFree 来自 7007979F  
0006FB68   007A0000   |Address = 007A0000  
0006FB6C   00000000   |Size = 0  
0006FB70   00008000   \\FreeType = MEM\_RELEASE  
暂停断点，Alt+F9 返回。  
  
Ctrl+S 在“整个段块”搜索命令序列：  
sub ecx,8  
shr ecx,1  
这 2 条命令对于所有用 Softlocx V5.0.0.6 加壳的程序来说都是固定的。  
找到在 7007AE1F 处，在其上的 7007ADFE 处下断。F9 运行，中断下来。  
  
7007ADFE     8BB5 6E340000     mov esi,dword ptr ss:\[ebp+346E\]  
//\[ebp+346E\]＝\[7007C46E\]=0006F000   ★ 重定位表 RVA！  
7007AE04     8B95 19250000     mov edx,dword ptr ss:\[ebp+2519\]  
7007AE0A     03F2              add esi,edx  
7007AE0C     8B85 15250000     mov eax,dword ptr ss:\[ebp+2515\]  
7007AE12     2BD0              sub edx,eax  
//检测是否需要重定位处理 ★  
7007AE14     74 6C             je short 7007AE82  
//可以在这里改标志位 Z=1，使其跳转，这样脱壳后就不需要修改基址了。  
//这个 OCX 不需要重定位，但是为了确定重定位表 Size，所以改标志位 Z=0，使其进行处理  
7007AE16     8B06              mov eax,dword ptr ds:\[esi\]  
7007AE18     0BC0              or eax,eax  
7007AE1A     74 66             je short 7007AE82  
//改标志位 Z=0，使其进行处理。处理结束则跳转,在 7007AE82 处下断  
7007AE1C     8B4E 04           mov ecx,dword ptr ds:\[esi+4\]  
7007AE1F     83E9 08           sub ecx,8  
7007AE22     D1E9              shr ecx,1  
7007AE24     8BBD 19250000     mov edi,dword ptr ss:\[ebp+2519\]  
7007AE2A     03F8              add edi,eax  
7007AE2C     83C6 08           add esi,8  
7007AE2F     0FB706            movzx eax,word ptr ds:\[esi\]  
7007AE32     C1C8 0C           ror eax,0C  
7007AE35     FEC8              dec al  
7007AE37     78 42             js short 7007AE7B  
7007AE39     2C 02             sub al,2  
7007AE3B     74 38             je short 7007AE75  
7007AE3D     EB 3C             jmp short 7007AE7B  
7007AE3F     C1E8 14           shr eax,14  
7007AE42     66:011C07         add word ptr ds:\[edi+eax\],bx  
7007AE46     EB 33             jmp short 7007AE7B  
7007AE48     C1E8 14           shr eax,14  
7007AE4B     66:011407         add word ptr ds:\[edi+eax\],dx  
7007AE4F     EB 2A             jmp short 7007AE7B  
7007AE51     52                push edx  
7007AE52     C1E8 14           shr eax,14  
7007AE55     8BD8              mov ebx,eax  
7007AE57     C1E0 10           shl eax,10  
7007AE5A     8A16              mov dl,byte ptr ds:\[esi\]  
7007AE5C     66:81E2 FF0F      and dx,0FFF  
7007AE61     66:8BC2           mov ax,dx  
7007AE64     5A                pop edx  
7007AE65     8D8410 00800000   lea eax,dword ptr ds:\[eax+edx+8000\]  
7007AE6C     89041F            mov dword ptr ds:\[edi+ebx\],eax  
7007AE6F     83C6 02           add esi,2  
7007AE72     49                dec ecx  
7007AE73     EB 06             jmp short 7007AE7B  
7007AE75     C1E8 14           shr eax,14  
7007AE78     011407            add dword ptr ds:\[edi+eax\],edx  
7007AE7B     83C6 02           add esi,2  
7007AE7E     E2 AF             loopd short 7007AE2F  
7007AE80     EB 94             jmp short 7007AE16  
//循环  
7007AE82     60                pushad  
//中断在这里时，ESI=7007533C    ★ 结束地址+2  
7007AE83     6A 00             push 0  
7007AE85     68 80000000       push 80  
7007AE8A     6A 03             push 3  
7007AE8C     6A 00             push 0  
7007AE8E     6A 00             push 0  
7007AE90     68 00000080       push 80000000  
7007AE95     BB E4180000       mov ebx,18E4  
7007AE9A     03DD              add ebx,ebp  
7007AE9C     53                push ebx  
7007AE9D     FF95 3A220000     call dword ptr ss:\[ebp+223A\]  
7007AEA3     83F8 FF           cmp eax,-1  
7007AEA6     74 1E             je short 7007AEC6  
  
重定位表信息：  
RVA=0006F000  
Size=7007533C-7006F000-2=633A  
  
  
—————————————————————————————————  
三、IAT、Dump  
  
  
恢复 VirtualFree 的断点，F9 运行，再次中断在 VirtualFree 处  
0006FB64   7007D118   /CALL 到 VirtualFree 来自 7007D112  
0006FB68   00790000   |Address = 00790000  
0006FB6C   00000000   |Size = 0  
0006FB70   00008000   \\FreeType = MEM\_RELEASE  
  
取消断点，Alt+F9 返回。  
  
7007D112     FF95 32220000     call dword ptr ss:\[ebp+2232\]  
7007D118     8B95 19250000     mov edx,dword ptr ss:\[ebp+2519\]  
//返回这里  
7007D11E     8BB5 72340000     mov esi,dword ptr ss:\[ebp+3472\]  
//\[ebp+3472\]=\[7007C472\]=00058B28  ★ IAT RVA！  
//此时代码解密完毕，输入表是完整的，没有被破坏，所以就在这里用 LordPE 完全 Dump 出这个 OCX！★   
  
7007D124     03F2              add esi,edx  
7007D126     8B46 0C           mov eax,dword ptr ds:\[esi+C\]  
7007D129     0BC0              or eax,eax  
7007D12B     0F84 F5000000     je 7007D226  
7007D131     03C2              add eax,edx  
7007D133     8BD8              mov ebx,eax  
7007D135     50                push eax  
7007D136     FF95 26220000     call dword ptr ss:\[ebp+2226\]  
7007D13C     0BC0              or eax,eax  
7007D13E     75 0F             jnz short 7007D14F  
7007D140     53                push ebx  
7007D141     FF95 2A220000     call dword ptr ss:\[ebp+222A\]  
7007D147     0BC0              or eax,eax  
7007D149     0F84 67010000     je 7007D2B6  
7007D14F     8985 7E340000     mov dword ptr ss:\[ebp+347E\],eax  
7007D155     6A 00             push 0  
7007D157     8F85 82340000     pop dword ptr ss:\[ebp+3482\]  
7007D15D     8B06              mov eax,dword ptr ds:\[esi\]  
7007D15F     0BC0              or eax,eax  
7007D161     8B95 19250000     mov edx,dword ptr ss:\[ebp+2519\]  
7007D167     75 03             jnz short 7007D16C  
7007D169     8B46 10           mov eax,dword ptr ds:\[esi+10\]  
7007D16C     03C2              add eax,edx  
7007D16E     0385 82340000     add eax,dword ptr ss:\[ebp+3482\]  
7007D174     8B18              mov ebx,dword ptr ds:\[eax\]  
7007D176     8B7E 10           mov edi,dword ptr ds:\[esi+10\]  
7007D179     03FA              add edi,edx  
7007D17B     03BD 82340000     add edi,dword ptr ss:\[ebp+3482\]  
7007D181     0BDB              or ebx,ebx  
7007D183     0F84 81000000     je 7007D20A  
7007D189     F7C3 00000080     test ebx,80000000  
7007D18F     75 06             jnz short 7007D197  
7007D191     8D5C13 02         lea ebx,dword ptr ds:\[ebx+edx+2\]  
7007D195     EB 13             jmp short 7007D1AA  
7007D197     81E3 FFFFFF0F     and ebx,0FFFFFFF  
7007D19D     C700 00000000     mov dword ptr ds:\[eax\],0  
7007D1A3     C685 86360000 01  mov byte ptr ss:\[ebp+3686\],1  
7007D1AA     899D 87360000     mov dword ptr ss:\[ebp+3687\],ebx  
7007D1B0     53                push ebx  
7007D1B1     FFB5 7E340000     push dword ptr ss:\[ebp+347E\]  
7007D1B7     FF95 22220000     call dword ptr ss:\[ebp+2222\]  
7007D1BD     0BC0              or eax,eax  
7007D1BF     0F84 F1000000     je 7007D2B6  
7007D1C5     8907              mov dword ptr ds:\[edi\],eax  
7007D1C7     80BD 86360000 01  cmp byte ptr ss:\[ebp+3686\],1  
7007D1CE     74 27             je short 7007D1F7  
7007D1D0     8B9D 87360000     mov ebx,dword ptr ss:\[ebp+3687\]  
7007D1D6     60                pushad  
7007D1D7     B8 00000000       mov eax,0  
7007D1DC     B9 00000000       mov ecx,0  
7007D1E1     8BFB              mov edi,ebx  
7007D1E3     803C0F 00         cmp byte ptr ds:\[edi+ecx\],0  
7007D1E7     74 03             je short 7007D1EC  
7007D1E9     41                inc ecx  
7007D1EA     EB F7             jmp short 7007D1E3  
7007D1EC     C6040F 00         mov byte ptr ds:\[edi+ecx\],0  
7007D1F0     49                dec ecx  
7007D1F1     83F9 00           cmp ecx,0  
7007D1F4     77 F6             ja short 7007D1EC  
7007D1F6     61                popad  
7007D1F7     C685 86360000 00  mov byte ptr ss:\[ebp+3686\],0  
7007D1FE     8385 82340000 04  add dword ptr ss:\[ebp+3482\],4  
7007D205     E9 53FFFFFF       jmp 7007D15D  
7007D20A     B9 13000000       mov ecx,13  
7007D20F     8BFE              mov edi,esi  
7007D211     B8 00000000       mov eax,0  
7007D216     F3:AA             rep stos byte ptr es:\[edi\]  
7007D218     83C6 14           add esi,14  
7007D21B     8B95 19250000     mov edx,dword ptr ss:\[ebp+2519\]  
7007D221     E9 00FFFFFF       jmp 7007D126  
//循环处理输入表  
  
其实下面跳 OEP 也很容易跟踪：  
  
7007D226     80BD 4D190000 01  cmp byte ptr ss:\[ebp+194D\],1  
7007D22D     74 39             je short 7007D268  
7007D268     80BD 4D190000 01  cmp byte ptr ss:\[ebp+194D\],1  
7007D26F     74 29             je short Softlocx.7007D29A  
  
7007D29A     64:67:8F06 0000   pop dword ptr fs:\[0\]  
7007D2A0     83C4 08           add esp,8  
7007D2A3     8B85 19250000     mov eax,dword ptr ss:\[ebp+2519\]  
7007D2A9     0185 86340000     add dword ptr ss:\[ebp+3486\],eax  
7007D2AF     FFA5 603D0000     jmp dword ptr ss:\[ebp+3D60\]; Softlocx.700790E4  
  
700790E4     61                popad  
700790E5     5D                pop ebp  
700790E6     8B85 86340000     mov eax,dword ptr ss:\[ebp+3486\]  
700790EC     5D                pop ebp  
700790ED     FFE0              jmp eax                    ; Softlocx.70026848  
//飞向光明之巅！ ^O^  
  
  
70026848     55                push ebp  
//OEP   ★  
70026849     8BEC              mov ebp,esp  
7002684B     53                push ebx  
7002684C     8B5D 08           mov ebx,dword ptr ss:\[ebp+8\]  
7002684F     56                push esi  
70026850     8B75 0C           mov esi,dword ptr ss:\[ebp+C\]  
70026853     57                push edi  
70026854     8B7D 10           mov edi,dword ptr ss:\[ebp+10\]  
70026857     85F6              test esi,esi  
70026859     75 09             jnz short Softlocx.70026864  
7002685B     833D 48320670 00  cmp dword ptr ds:\[70063248\],0  
70026862     EB 26             jmp short Softlocx.7002688A  
  
  
—————————————————————————————————  
四、PE 修正  
  
  
用 LordPE 修正 dumped\_.dll：  
重定位表 RVA=0006F000、大小＝633A  
修正 IAT RVA=00058B28  
修正 OEP RVA=00026848  
保存之。  
  
如果还想做的完美点，可以删除最后一个壳区段，再把输出表数据挪到空白地方即可。  
OK，脱壳完成啦。  
  
  
—————————————————————————————————                                   
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了破解轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacked By :  fly  
               2004-11-08 18:30

 4dd466d0e1e0cc123afbe504da2462ca