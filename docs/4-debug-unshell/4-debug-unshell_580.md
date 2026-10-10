

# CrypKey V5.7

> 原文：[`www.pediy.com/kssd/pediy06/pediy7000.htm`](https://www.pediy.com/kssd/pediy06/pediy7000.htm)

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
               
【调试环境】：WinXP、Ollydbg V1.10、PEiD、LordPE、ImportREC 

【实例下载】：点击下载  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
                   
这个壳新加的 3 个区段名为：Have A Nice Day！ 送给大家的祝福 ^O^  
记得是 CryptKey V5.X 加壳程序的检测配置 DLL 文件。  
使用了“CrypKey Stealth”选项加壳。This advanced feature will encrypt and compress your program and add runtime guards against hacking, making your program very difficult to reverse enginee...  
EXE 的脱壳简单，DLL 的脱壳需要找到重定位表的信息。  
  
设置 Ollydbg 忽略的所有异常选项。老规矩：用 IsDebug 1.4 插件去掉 Ollydbg 的调试器标志。  
—————————————————————————————————  
一、前奏  
  
  
003FA700     8B1D 6BA63F00       mov ebx,dword ptr ds:\[3FA66B\]//进入 OD 后暂停在这  
003FA706     83FB 00             cmp ebx,0  
003FA709     75 0A               jnz short CKI32h.003FA715  
003FA70B     E8 3C000000         call CKI32h.003FA74C  
003FA710     E8 FB0A0000         call CKI32h.003FB210  
003FA715     8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
003FA719     50                  push eax  
003FA71A     E8 09020000         call CKI32h.003FA928  
//解压代码，处理重定位     
003FA71F     A1 6BA63F00         mov eax,dword ptr ds:\[3FA66B\]  
003FA724     83F8 01             cmp eax,1  
003FA727     74 06               je short CKI32h.003FA72F  
003FA729     FF25 14A03F00       jmp dword ptr ds:\[3FA014\]; CKI32h.003B1000  
//这里下断   飞向光明之巅！ ^O^  
  
—————————————————————————————————  
二、代码解压  
  
  
当我们到达 003FA71A 时，可以下断：BP UnmapViewOfFile  
中断后取消断点，Alt+F9 返回程序。  
  
003FADB4     7E B4               jle short CKI32h.003FAD6A  
003FADB6     E9 96FDFFFF         jmp CKI32h.003FAB51  
//循环解压代码  
003FADBB     8B55 D4             mov edx,dword ptr ss:\[ebp\-2C\]  
003FADBE     8B0D 0CA03F00       mov ecx,dword ptr ds:\[3FA00C\]; CKI32h.003D4C00  
003FADC4     2B4A 04             sub ecx,dword ptr ds:\[edx+4\]  
003FADC7     83E9 04             sub ecx,4  
003FADCA     8B45 D4             mov eax,dword ptr ss:\[ebp\-2C\]  
003FADCD     8948 04             mov dword ptr ds:\[eax+4\],ecx  
003FADD0     FF75 FC             push dword ptr ss:\[ebp\-4\]  
003FADD3     E8 66100000         call <jmp.&KERNEL32.UnmapViewOfFile>  
//返回这里，此时程序解压完毕。还没有重定位  
//可以在这里 Dump 进程，这样脱壳后就不必修改基址了 ★  
003FADD8     FF75 F8             push dword ptr ss:\[ebp\-8\]  
003FADDB     E8 1C100000         call <jmp.&KERNEL32.CloseHandle>  
003FADE0     FF75 F4             push dword ptr ss:\[ebp\-C\]  
003FADE3     E8 14100000         call <jmp.&KERNEL32.CloseHandle>  
003FADE8     E8 43030000         call CKI32h.003FB130  
//重定位处理  ★  
003FADED     E8 4A020000         call CKI32h.003FB03C  
  
—————————————————————————————————  
三、重定位表  
  
  
进入：003FADE8   call CKI32h.003FB130  
  
003FB130     55                  push ebp  
003FB131     8BEC                mov ebp,esp  
003FB133     83C4 F4             add esp,-0C  
003FB136     833D 20A03F00 00    cmp dword ptr ds:\[3FA020\],0  
003FB13D     74 48               je short CKI32h.003FB187  
003FB13F     A1 6FA63F00         mov eax,dword ptr ds:\[3FA66F\]  
003FB144     2B05 10A03F00       sub eax,dword ptr ds:\[3FA010\]; LOADDLL.00400000  
//减去基址  
003FB14A     8945 FC             mov dword ptr ss:\[ebp\-4\],eax  
003FB14D     837D FC 00          cmp dword ptr ss:\[ebp\-4\],0  
003FB151     74 34               je short CKI32h.003FB187  
//不跳则重定位  
003FB153     8B15 20A03F00       mov edx,dword ptr ds:\[3FA020\]  
//\[003FA020\]=00047000            重定位表的 RVA! ★  
003FB159     0315 6FA63F00       add edx,dword ptr ds:\[3FA66F\]; CKI32h.003B0000  
003FB15F     8955 F8             mov dword ptr ss:\[ebp\-8\],edx  
003FB162     33C9                xor ecx,ecx  
003FB164     894D F4             mov dword ptr ss:\[ebp\-C\],ecx  
003FB167     FF75 FC             push dword ptr ss:\[ebp\-4\]  
003FB16A     8B45 F8             mov eax,dword ptr ss:\[ebp\-8\]  
003FB16D     0345 F4             add eax,dword ptr ss:\[ebp\-C\]  
003FB170     50                  push eax  
003FB171     E8 3EFFFFFF         call CKI32h.003FB0B4  
003FB176     83C4 08             add esp,8  
003FB179     0145 F4             add dword ptr ss:\[ebp\-C\],eax  
003FB17C     8B55 F4             mov edx,dword ptr ss:\[ebp\-C\]  
003FB17F     3B15 24A03F00       cmp edx,dword ptr ds:\[3FA024\]  
003FB185     72 E0               jb short CKI32h.003FB167  
//循环处理  
003FB187     8BE5                mov esp,ebp  
//此处下断，中断后重定位处理完毕。  
//ECX=003F915C                   重定位表结束地址 ★  
003FB189     5D                  pop ebp  
003FB18A     C3                  retn  
  
可以看出壳没有加密重定位表。  
重定位表大小＝003F915C－003B0000－00047000 ＝215C  
  
—————————————————————————————————  
四、OEP  
  
  
因为这个东东没有加密 IAT，所以 F9 运行，直接从 003FA729 处跳至 OEP  
  
003B1000     EB 10               jmp short CKI32h.003B1012  
//也可以在这里用 LordPE 完全 DUMP 这个 DLL 进程  
003B1002     66:623A             bound di,dword ptr ds:\[edx\]  
003B1005     43                  inc ebx  
003B1006     2B2B                sub ebp,dword ptr ds:\[ebx\]  
003B1008     48                  dec eax  
003B1009     4F                  dec edi  
003B100A     4F                  dec edi  
003B100B     4B                  dec ebx  
003B100C     90                  nop  
003B100D     E9 CC513D00         jmp 007861DE  
003B1012     A1 53513D00         mov eax,dword ptr ds:\[3D5153\]  
003B1017     C1E0 02             shl eax,2  
003B101A     A3 57513D00         mov dword ptr ds:\[3D5157\],eax  
003B101F     8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
003B1023     A3 C5513D00         mov dword ptr ds:\[3D51C5\],eax  
003B1028     FF1485 B5513D00     call dword ptr ds:\[eax\*4+3D51B5\]; CKI32h.003B1192  
  
—————————————————————————————————  
五、输入表  
  
  
随便从程序找一个 API 调用，如：  
003D44D8     FF25 C4A03E00       jmp dword ptr ds:\[3EA0C4\]; ADVAPI32.RegCreateKeyA  
  
在转存中跟随 3EA0C4，上下看到许多函数地址，很明显的可以找到 IAT 开始和结束的地址：  
003EA0AC   7A A8 03 00 8A A8 03 00 9E A8 03 00 00 00 00 00  z?.姩.灗.....  
003EA0BC   AB AC DA 77 9A 18 DA 77 F0 72 DA 77 EA 22 DA 77  趙?趙餽趙?趙  
  
003EA7E0   27 7D D1 77 D3 3D D1 77 1E CD D1 77 12 C5 D3 77   '}褀?褀脱 w 庞 w  
003EA7F0   6A C9 D1 77 00 00 00 00 41 44 56 41 50 49 33 32   j 裳 w....ADVAPI32  
  
开始地址＝003EA0BC  
结束地址＝003EA7F5  
  
RVA＝003EA0BC-003B0000=0003A0BC  
大小＝003EA7F5-003EA0BC=00000739  
  
运行 ImportREC，去掉“使用来自磁盘的 PE 部首”的选项。               
选中 Ollydbg 的 loaddll.exe 的进程，然后选择 CKI32h.DLL，填入 RVA＝0003A0BC、大小＝00000739 ，点“Get Import”，CUT 掉无效的垃圾指针。改 OEP=00001000，去掉“创建新的 IAT”选项，FixDump！  
  
—————————————————————————————————     
六、PE 修正  
  
  
如果你没有避开重定位，则需要修改基址＝003B0000。  
修正重定位表 RVA=00047000，Size=215C  
OK，脱壳成功！                 
               
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
  
      
     UnPacked By 巢水工作坊——fly \[FCG\]\[NUKE\]\[DCM\]  
                2004-08-16  00:00  
 4dd466d0e1e0cc123afbe504da2462ca