

# CoolCryptor V0.9

> 原文：[`www.pediy.com/kssd/pediy06/pediy6462.htm`](https://www.pediy.com/kssd/pediy06/pediy6462.htm)

kongfoo 兄弟的 CoolCryptor V0.9 脱壳   
    
    
    
直接下载：点击下载（或鼠标右键另存为）  
软件大小：  38 KB   
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！   
  
【调试环境】：WinXP、flyODBG 1.10B 修改版、PEiD、LordPE、ImportREC   
  
—————————————————————————————————    
【脱壳过程】：   
             
            
           
“几多天真的思想，几多找到是颓丧，沉默去迎失望，几多心中创伤”   
——呵呵，经过 kongfoo 兄弟几多辛苦的努力，COOLcryptor 升级至 V0.9 了。花指令繁多；其反跟踪方面已经比一般的保护壳更强了；其双进程方式则是除 Armadillo 之外所仅见的，虽然 COOLcryptor 的双进程还不具备更多的功能，但是 kongfoo 擅于溶和、吸收其它壳优点的能力已经让人佩服了！   
  
再提几点建议吧：   
  
1、COOLcryptor 自身的兼容性。如今 COOLcryptor 只是 For XP，以发展的眼光来看，一款优秀的壳最好能够顺利的跨平台运行，至少能够在目前流行的各 WinDows 平台上都能够运行。现在的 COOLcryptor V0.9 在 Win XP 的中文目录下也无法运行起来。这是 COOLcryptor 所必须解决的问题，否则随着 COOLcryptor 的不断升级，想纠正就会花费更多的精力了。   
  
2、COOLcryptor 加壳后被保护程序的兼容性。COOLcryptor 加壳后的一些程序在 XP 上也无法运行了，如果这样下去，COOLcryptor 只是成为一个自身的 UnPackMe，而无法成为一款优秀的壳。   
  
3、IAT 处理，代码变形等方面还要继续变态，呵呵，对于壳来说，越变态越好啦。 8)   
  
  
原来这个笔记应由 kitty 兄弟来写的，既然兄弟客气，我就来简单写一下吧，希望大家多多写作脱壳笔记，呵呵   
OK，说了一堆废话，下面就以今天刚发布的 COOLcryptor0.9.exe 来脱壳演示一下。   
  
—————————————————————————————————   
设置 Ollydbg 忽略所有其它选项。用 IsDebug 1.4 插件去掉调试器标志。   
  
  
一、反 反跟踪   
  
  
1、去掉 lock cmpxchg    
  
  
00408060     E8 03000000         call COOLcryp.00408068//进入 OD 后停在这    
00408065     E9 EB07FF04         jmp 053F8855   
  
Ctrl+B 搜索 2 进制字符：F00FC7C8    在 00408181 处，下断   
  
00408181     F0:0FC7C8           lock cmpxchg8b eax; 非法使用寄存器   
00408185     83AD D5214000 04    sub dword ptr ss:\[ebp+4021D5\],4//\[ebp+4021D5\]=324C    
0040818C     79 F3               jns short COOLcryp.00408181   
0040818E     64:8F05 00000000    pop dword ptr fs:\[0\]   
00408195     83C4 04             add esp,4//此处下断，F9 断在这！    
00408198     EB 51               jmp short COOLcryp.004081EB   
  
00408181 处利用 EXECryptor 的 lock cmpxchg8b 异常解码。用修改的 Ollydbg 1.10B 版调试，增加忽略 C000001E 异常，在 00408195 处下断，F9 运行，异常解码完毕后就停在 00408195 了。   
  
当然，可以不管 lock cmpxchg8b 这个异常解码，直接开始第 2 步。   
  
  
—————————————————————————————————   
2、进程比较  BP GetModuleFileNameA+3     
  
  
00409524     FF95 3D5B4000       call dword ptr ss:\[ebp+405B3D\]; kernel32.GetModuleFileNameA   
0040952A     8BC7                mov eax,edi//中断后 Alt+F9 返回这里    
0040952C     8038 60             cmp byte ptr ds:\[eax\],60   
0040952F     72 03               jb short COOLcryp.00409534   
00409531     8028 20             sub byte ptr ds:\[eax\],20   
00409534     40                  inc eax   
00409535     8038 00             cmp byte ptr ds:\[eax\],0   
00409538     75 F2               jnz short COOLcryp.0040952C   
0040953A     8D85 9B574000       lea eax,dword ptr ss:\[ebp+40579B\]   
00409540     8D9D B1594000       lea ebx,dword ptr ss:\[ebp+4059B1\]   
00409546     33C9                xor ecx,ecx    
00409548     8A1401              mov dl,byte ptr ds:\[ecx+eax\]   
                               //修改：mov dl,byte ptr ds:\[ecx+ebx\] ★    
0040954B     381419              cmp byte ptr ds:\[ecx+ebx\],dl   
0040954E     75 0E               jnz short COOLcryp.0040955E//跳就 OVER 了    
00409550     41                  inc ecx    
00409551     81F9 03010000       cmp ecx,103   
00409557     75 EF               jnz short COOLcryp.00409548   
00409559     E9 73070000         jmp COOLcryp.00409CD1   
  
  
————————————————————————   
3、大小比较  BP GetFileSize   
  
  
00409F81     FF95 415B4000       call dword ptr ss:\[ebp+405B41\]; kernel32.CreateFileA   
00409F87     8BD8                mov ebx,eax   
00409F89     6A 00               push 0   
00409F8B     50                  push eax   
00409F8C     FF95 515B4000       call dword ptr ss:\[ebp+405B51\]; kernel32.GetFileSize   
00409F92     8985 2F564000       mov dword ptr ss:\[ebp+40562F\],eax//中断后 Alt+F9 返回这里    
00409F98     53                  push ebx   
00409F99     FF95 555B4000       call dword ptr ss:\[ebp+405B55\]; kernel32.CloseHandle   
00409F9F     8D85 B1594000       lea eax,dword ptr ss:\[ebp+4059B1\]   
00409FA5     6A 00               push 0   
00409FA7     68 80000000         push 80   
00409FAC     6A 03               push 3   
00409FAE     6A 00               push 0   
00409FB0     6A 01               push 1   
00409FB2     68 00000080         push 80000000   
00409FB7     50                  push eax   
00409FB8     FF95 415B4000       call dword ptr ss:\[ebp+405B41\]; kernel32.CreateFileA   
00409FBE     8BD8                mov ebx,eax   
00409FC0     6A 00               push 0   
00409FC2     50                  push eax   
00409FC3     FF95 515B4000       call dword ptr ss:\[ebp+405B51\]; kernel32.GetFileSize   
00409FC9     8BF8                mov edi,eax   
00409FCB     53                  push ebx   
00409FCC     FF95 555B4000       call dword ptr ss:\[ebp+405B55\]; kernel32.CloseHandle   
00409FD2     33BD 2F564000       xor edi,dword ptr ss:\[ebp+40562F\]//修改：xor edi,edi ★    
00409FD8     75 03               jnz short COOLcryp.00409FDD//跳就 ExitProcess 了     
00409FDA     61                  popad   
00409FDB     EB 20               jmp short COOLcryp.00409FFD   
  
  
————————————————————————   
4、还有一处 进程比较   
  
  
Ctr+F 在当前位置搜索命令：cmp byte ptr ds:\[ecx+ebx\],dl 在 0040A102 处   
  
0040A0F1     8D85 9B574000       lea eax,dword ptr ss:\[ebp+40579B\]   
0040A0F7     8D9D B1594000       lea ebx,dword ptr ss:\[ebp+4059B1\]   
0040A0FD     33C9                xor ecx,ecx   
0040A0FF     8A1401              mov dl,byte ptr ds:\[ecx+eax\]   
                               //修改：mov dl,byte ptr ds:\[ecx+ebx\] ★    
0040A102     381419              cmp byte ptr ds:\[ecx+ebx\],dl   
0040A105     75 0B               jnz short COOLcryp.0040A112//跳就 OVER 了    
0040A107     41                  inc ecx   
0040A108     81F9 04010000       cmp ecx,104   
0040A10E     75 EF               jnz short COOLcryp.0040A0FF   
0040A110     EB 20               jmp short COOLcryp.0040A132   
  
  
修改之后 COOLcryptor 的双进程就 OVER 啦  :-)   
  
    
———————————————————————   
5、MeltICE 表 BP CreateFileA   
  
  
0040B59E     FF95 415B4000       call dword ptr ss:\[ebp+405B41\]; kernel32.CreateFileA   
0040B5A4     40                  inc eax//中断后 Alt+F9 返回这里    
0040B5A5     75 0D               jnz short COOLcryp.0040B5B4   
0040B5A7     803E 00             cmp byte ptr ds:\[esi\],0   
0040B5AA     74 DE               je short COOLcryp.0040B58A   
0040B5AC     46                  inc esi   
0040B5AD     803E FF             cmp byte ptr ds:\[esi\],0FF   
0040B5B0     75 F5               jnz short COOLcryp.0040B5A7   
0040B5B2     EB 02               jmp short COOLcryp.0040B5B6   
0040B5B4     61                  popad   
0040B5B5     C3                  retn   
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
MeltICE 表   
  
0040B466  5C 5C 2E 5C 53 49 43 45 00 5C 5C 2E 5C 4E 54 49   \\.SICE.\\.NTI   
0040B476  43 45 00 5C 5C 2E 5C 4E 54 49 43 45 37 38 37 31   CE.\\.NTICE7871   
0040B486  00 5C 5C 2E 5C 4E 54 49 43 45 44 30 35 32 00 5C   .\\.NTICED052.   
0040B496  5C 2E 5C 54 52 57 44 45 42 55 47 00 5C 5C 2E 5C   .TRWDEBUG.\\.   
0040B4A6  54 52 57 00 5C 5C 2E 5C 54 52 57 32 30 30 30 00   TRW.\\.TRW2000.   
0040B4B6  5C 5C 2E 5C 53 55 50 45 52 42 50 4D 00 5C 5C 2E   \\.SUPERBPM.\\.   
0040B4C6  5C 49 43 45 44 55 4D 50 00 5C 5C 2E 5C 52 45 47   ICEDUMP.\\.REG   
0040B4D6  4D 4F 4E 00 5C 5C 2E 5C 46 49 4C 45 4D 4F 4E 00   MON.\\.FILEMON.   
0040B4E6  5C 5C 2E 5C 52 45 47 56 58 44 00 5C 5C 2E 5C 46   \\.REGVXD.\\.F   
0040B4F6  49 4C 45 56 58 44 00 5C 5C 2E 5C 56 4B 45 59 50   ILEVXD.\\.VKEYP   
0040B506  52 4F 44 00 5C 5C 2E 5C 42 57 32 4B 00 5C 5C 2E   ROD.\\.BW2K.\\.   
0040B516  5C 53 49 57 44 45 42 55 47 00 5C 5C 2E 5C 53 49   SIWDEBUG.\\.SI   
0040B526  57 56 49 44 00 5C 5C 2E 5C 47 6C 6F 62 61 6C 5C   WVID.\\.Global   
0040B536  46 49 4C 45 4D 4F 4E 00 5C 5C 2E 5C 47 6C 6F 62   FILEMON.\\.Glob   
0040B546  61 6C 5C 52 45 47 4D 4F 4E 00 5C 5C 2E 5C 46 49   alREGMON.\\.FI   
0040B556  4C 45 56 58 44 2E 56 58 44 00 5C 5C 2E 5C 52 45   LEVXD.VXD.\\.RE   
0040B566  47 56 58 44 2E 56 58 44 00 5C 5C 2E 5C 46 52 4F   GVXD.VXD.\\.FRO   
0040B576  47 53 49 43 45 00 5C 5C 2E 5C 49 63 65 45 78 74   GSICE.\\.IceExt   
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆   
  
我用 Ollydbg 修改版调试，这里就没有威胁啦。 :-)   
  
  
————————————————————————   
6、时间监测  BP GetTickCount+5   
  
  
COOLcryptor V0.9 加了时间监测，要下 BP GetTickCount+5，呵呵。Shift+F9 走   
  
0040A35F     FF95 F15A4000       call dword ptr ss:\[ebp+405AF1\]; kernel32.77E5751F   
0040A365     8985 1F564000       mov dword ptr ss:\[ebp+40561F\],eax//第 1 次返回    
  
0040A5B7     FF95 F15A4000       call dword ptr ss:\[ebp+405AF1\]; kernel32.77E5751F   
0040A5BD     33D2                xor edx,edx//第 2 次返回    
0040A5BF     52                  push edx   
0040A5C0     50                  push eax   
0040A5C1     8B85 1F564000       mov eax,dword ptr ss:\[ebp+40561F\]   
0040A5C7     99                  cdq   
0040A5C8     290424              sub dword ptr ss:\[esp\],eax//修改\[esp\]值＝EAX    
0040A5CB     195424 04           sbb dword ptr ss:\[esp+4\],edx   
0040A5CF     58                  pop eax ; COOLcryp.00409E78   
0040A5D0     5A                  pop edx ; COOLcryp.00409E78   
0040A5D1     83FA 00             cmp edx,0   
0040A5D4     75 05               jnz short COOLcryp.0040A5DB   
0040A5D6     83F8 04             cmp eax,4   
0040A5D9     76 02               jbe short COOLcryp.0040A5DD//不跳就 OVER 了    
0040A5DB     61                  popad   
0040A5DC     C3                  retn   
  
或者直接改 0040A5D9 处为 JMP，跳过来就 OK 了。   
至此，解压主程序代码前的反跟踪已经全部干掉了。8)   
  
  
—————————————————————————————————   
二、寻找 OEP   
  
  
由于 COOLcryptor 是在 yoda's cryptor 的基础上修改的，所以有些地方是相同的。   
  
0040A5DD     8D85 80494000       lea eax,dword ptr ss:\[ebp+404980\]   
0040A5E3     8D9D D74D4000       lea ebx,dword ptr ss:\[ebp+404DD7\]   
0040A5E9     BA E112D536         mov edx,36D512E1   
0040A5EE     E8 500E0000         call COOLcryp.0040B443//解码 0040A992 处    
0040A5F3     F785 0F564000 10000>test dword ptr ss:\[ebp+40560F\],10   
0040A5FD     74 37               je short COOLcryp.0040A636   
0040A5FF     64:FF35 30000000    push dword ptr fs:\[30\]   
0040A606     58                  pop eax     ; COOLcryp.00409E78   
0040A607     85C0                test eax,eax; COOLcryp.0040A992   
0040A609     78 0F               js short COOLcryp.0040A61A   
0040A60B     8B40 0C             mov eax,dword ptr ds:\[eax+C\]   
0040A60E     8B40 0C             mov eax,dword ptr ds:\[eax+C\]   
0040A611     C740 20 00020000    mov dword ptr ds:\[eax+20\],200   
0040A618     EB 1C               jmp short COOLcryp.0040A636   
0040A61A     6A 00               push 0   
0040A61C     FF95 D15A4000       call dword ptr ss:\[ebp+405AD1\]; kernel32.GetModuleHandleA   
0040A622     85D2                test edx,edx   
0040A624     79 10               jns short COOLcryp.0040A636   
0040A626     837A 08 FF          cmp dword ptr ds:\[edx+8\],-1   
0040A62A     75 0A               jnz short COOLcryp.0040A636   
0040A62C     8B52 04             mov edx,dword ptr ds:\[edx+4\]   
0040A62F     C742 50 00020000    mov dword ptr ds:\[edx+50\],200   
0040A636     8D85 BF3C4000       lea eax,dword ptr ss:\[ebp+403CBF\]   
0040A63C     8D9D 24464000       lea ebx,dword ptr ss:\[ebp+404624\]   
0040A642     E8 0B0E0000         call COOLcryp.0040B452   
0040A647     8BBD 07564000       mov edi,dword ptr ss:\[ebp+405607\]  ; COOLcryp.00400000   
0040A64D     037F 3C             add edi,dword ptr ds:\[edi+3C\]   
0040A650     8BB5 07564000       mov esi,dword ptr ss:\[ebp+405607\]  ; COOLcryp.00400000   
0040A656     8B4F 54             mov ecx,dword ptr ds:\[edi+54\]   
0040A659     8D85 755B4000       lea eax,dword ptr ss:\[ebp+405B75\]   
0040A65F     50                  push eax                           ; COOLcryp.0040A992   
0040A660     6A 04               push 4   
0040A662     51                  push ecx   
0040A663     FFB5 07564000       push dword ptr ss:\[ebp+405607\]     ; COOLcryp.00400000   
0040A669     FF95 D55A4000       call dword ptr ss:\[ebp+405AD5\]     ; kernel32.VirtualProtect   
0040A66F     EB 25               jmp short COOLcryp.0040A696   
  
可以直接 Ctrl+F9 执行到返回，也可以 Ctr+F 在当前位置搜索命令：ror ebx,7   
呵呵，这可是 yoda's cryptor 壳计算出 OEP 值代码的特征地方呀   :-)   
  
0040A784     8B9D 07564000       mov ebx,dword ptr ss:\[ebp+405607\]  ; COOLcryp.00400000   
0040A78A     039D 0B564000       add ebx,dword ptr ss:\[ebp+40560B\]   
                               //00400000+00005CC6=00405CC6  这就是 OEP 值了  :-)    
0040A790     C1CB 07             ror ebx,7//yoda's cryptor 典型特征码    
  
  
—————————————————————————————————   
三、Magic Jump ，避开 IAT 加密   
  
  
这段 IAT 处理代码是借用 新仙剑 的。Ctrl+F 搜索命令：mov dword ptr ds:\[edx\],eax   
  
0040A8FD     8902                mov dword ptr ds:\[edx\],eax; kernel32.CloseHandle   
                                 //正确函数进入正确地址。 要的就是这个效果 :-)    
0040A8FF     EB 19               jmp short COOLcryp.0040A91A   
  
0040A91A     F785 0F564000 20000>test dword ptr ss:\[ebp+40560F\],20   
0040A924     74 45               je short COOLcryp.0040A96B//改为 JMP ★    
  
0040A96B     83C1 04             add ecx,4   
0040A96E     83C2 04             add edx,4   
0040A971     8339 00             cmp dword ptr ds:\[ecx\],0   
0040A974     0F85 34FFFFFF       jnz COOLcryp.0040A8AE   
0040A97A     83C6 0C             add esi,0C   
0040A97D     837E 04 00          cmp dword ptr ds:\[esi+4\],0   
0040A981     0F85 B4FEFFFF       jnz COOLcryp.0040A83B   
0040A987     33C0                xor eax,eax   
0040A989     40                  inc eax   
0040A98A     83F8 01             cmp eax,1   
0040A98D     74 02               je short COOLcryp.0040A991//这里下断    
0040A98F     61                  popad   
0040A990     C3                  retn   
  
注意：在 0040A98D 处断下后 IAT 已经处理完毕，此时脱壳的时机最好，程序代码已经解开，输入表完全恢复，再继续运行 kongfoo 就要来破坏啦。   :-)   
  
  
继续看看避开 IAT 重定位：   
  
0040AA85     60                  pushad   
0040AA86     8B85 07564000       mov eax,dword ptr ss:\[ebp+405607\]     ; COOLcryp.00400000   
0040AA8C     8BD8                mov ebx,eax                           ; COOLcryp.0040AA3F   
0040AA8E     8B40 3C             mov eax,dword ptr ds:\[eax+3C\]   
0040AA91     03D8                add ebx,eax                           ; COOLcryp.0040AA3F   
0040AA93     33C0                xor eax,eax                           ; COOLcryp.0040AA3F   
0040AA95     8B43 08             mov eax,dword ptr ds:\[ebx+8\]   
0040AA98     8985 C95A4000       mov dword ptr ss:\[ebp+405AC9\],eax     ; COOLcryp.0040AA3F   
0040AA9E     BA 3E000000         mov edx,3E   
0040AAA3     F7E2                mul edx                               ; COOLcryp.00401400   
0040AAA5     50                  push eax                              ; COOLcryp.0040AA3F   
0040AAA6     6A 40               push 40   
0040AAA8     FF95 455B4000       call dword ptr ss:\[ebp+405B45\]        ; kernel32.GlobalAlloc   
0040AAAE     8985 27564000       mov dword ptr ss:\[ebp+405627\],eax     ; COOLcryp.0040AA3F   
0040AAB4     33C9                xor ecx,ecx                           ; COOLcryp.00401384   
0040AAB6     33FF                xor edi,edi                           ; COOLcryp.004015C0   
0040AAB8     8B93 B0000000       mov edx,dword ptr ds:\[ebx+B0\]   
0040AABE     8B9D 07564000       mov ebx,dword ptr ss:\[ebp+405607\]     ; COOLcryp.00400000   
0040AAC4     03D3                add edx,ebx   
0040AAC6     8B9D 27564000       mov ebx,dword ptr ss:\[ebp+405627\]   
0040AACC     8B0411              mov eax,dword ptr ds:\[ecx+edx\]   
0040AACF     83F8 00             cmp eax,0   
0040AAD2     74 32               je short COOLcryp.0040AB06//改为 JMP ★    
  
0040AB06     83C1 04             add ecx,4   
0040AB09     83C7 3E             add edi,3E   
0040AB0C     3B8D C95A4000       cmp ecx,dword ptr ss:\[ebp+405AC9\]   
0040AB12     75 B8               jnz short COOLcryp.0040AACC//循环破坏 :-)    
0040AB14     EB 3E               jmp short COOLcryp.0040AB54   
     
     
—————————————————————————————————   
四、避开 OEP 处的 Stolen Code   
  
  
如果没有在 0040A98D 脱壳的话，那就继续看下去吧。可以 BP GlobalAlloc，也可以在 OEP 处下内存断点   
  
0040B012     803B E8             cmp byte ptr ds:\[ebx\],0E8   
0040B015     74 04               je short COOLcryp.0040B01B   
0040B017     41                  inc ecx   
0040B018     43                  inc ebx                         ; COOLcryp.00405CC6   
0040B019     EB F7               jmp short COOLcryp.0040B012   
0040B01B     66:817B 01 FF15     cmp word ptr ds:\[ebx+1\],15FF   
0040B021     75 01               jnz short COOLcryp.0040B024   
0040B023     41                  inc ecx   
0040B024     E8 03000000         call COOLcryp.0040B02C   
  
0040B06A     5B                  pop ebx                         ; COOLcryp.00405CC6   
0040B06B     83C1 06             add ecx,6   
0040B06E     51                  push ecx   
0040B06F     51                  push ecx   
0040B070     6A 40               push 40   
0040B072     FF95 455B4000       call dword ptr ss:\[ebp+405B45\] ; kernel32.GlobalAlloc   
0040B078     8985 23564000       mov dword ptr ss:\[ebp+405623\],eax//EAX=001434B0    
0040B07E     59                  pop ecx                         ; COOLcryp.00405CC6   
0040B07F     83E9 06             sub ecx,6   
0040B082     51                  push ecx   
0040B083     8BF3                mov esi,ebx                     ; COOLcryp.00405CC8   
0040B085     8BF8                mov edi,eax                     ; COOLcryp.0040AFBB   
0040B087     FC                  cld   
0040B088     F3:A4               rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]   
                               //保存原 OEP 处的 2 个字节 6A00 至\[00142A28\]处    
0040B08A     E8 03000000         call COOLcryp.0040B092//Stolen Code 进去看看    
  
0040B0D0     59                  pop ecx   
0040B0D1     8BD1                mov edx,ecx   
0040B0D3     B0 9A               mov al,9A   
0040B0D5     5F                  pop edi   
0040B0D6     AA                  stos byte ptr es:\[edi\]//NOP 掉就 OK 了  :-)    
0040B0D7     E2 FD               loopd short COOLcryp.0040B0D6   
0040B0D9     E8 03000000         call COOLcryp.0040B0E1   
  
  
—————————————————————————————————   
五、DUMP 进程、修复输入表、优化   
  
  
在 00142A28 处下断，F9 就断下了。   
  
00142A28     6A 00               push 0//Stolen Code    
00142A2A     68 C85C4000         push 405CC8   
00142A2F     C3                  retn //飞向光明之巅！:-)    
  
00405CC6     6A 00               push 0//用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程    
00405CC8     E8 C1020000         call COOLcryp.00405F8E; jmp to kernel32.GetModuleHandleA   
00405CCD     A3 00104000         mov dword ptr ds:\[401000\],eax   
00405CD2     6A 00               push 0   
00405CD4     68 EA5C4000         push COOLcryp.00405CEA   
00405CD9     6A 00               push 0   
00405CDB     6A 64               push 64   
00405CDD     50                  push eax   
00405CDE     E8 E1020000         call COOLcryp.00405FC4; jmp to user32.DialogBoxParamA   
00405CE3     6A 00               push 0   
00405CE5     E8 98020000         call COOLcryp.00405F82; jmp to kernel32.ExitProcess   
  
  
用 LordPE 删除 dumped.exe 最后的 COOL 区段。运行 ImportREC，选择这个进程。把 OEP 改为 00005CC6，点 IT AutoSearch，点“Get Import”，所有函数都是有效的。去掉“增加新的区段”选项，在原程序找一段空地，我选择处，填入“新的输入信息”的 RVA=00005610,FixDump，再用 LordPE 重建 PE！38.4K->22.6K   
  
  
—————————————————————————————————   
六、修复程序中被抽取的代码   
  
  
运行脱壳后的程序加壳时出错，呵呵，如果在 040A98D 处脱壳的话就没有这个问题了。   
  
00401ABA     E8 B2020000         call COOLcryp.00401D71//出错    
  
00401D71     68 98291400         push 142998   
00401D76     C3                  retn   
  
看到 00401D76 下面很长一段 0000，处理也很简单，用原程序加壳，跟踪入 00142998 就看到被抽取的代码了。   
  
00142998     55                  push ebp   
00142999     8BEC                mov ebp,esp   
0014299B     83C4 FC             add esp,-4   
0014299E     57                  push edi   
0014299F     56                  push esi                         ; COOLcryp.004016F3   
001429A0     53                  push ebx   
001429A1     51                  push ecx   
001429A2     52                  push edx   
001429A3     8B7D 08             mov edi,dword ptr ss:\[ebp+8\]     ; COOLcryp.00401054   
001429A6     037F 3C             add edi,dword ptr ds:\[edi+3C\]   
001429A9     33C0                xor eax,eax   
001429AB     66:8B47 06          mov ax,word ptr ds:\[edi+6\]   
001429AF     8945 FC             mov dword ptr ss:\[ebp\-4\],eax   
001429B2     B9 28000000         mov ecx,28   
001429B7     0FAFC1              imul eax,ecx   
001429BA     83C0 28             add eax,28   
001429BD     8BCF                mov ecx,edi   
001429BF     2B4D 08             sub ecx,dword ptr ss:\[ebp+8\]     ; COOLcryp.00401054   
001429C2     03C8                add ecx,eax   
001429C4     81C1 F8000000       add ecx,0F8   
001429CA     3B4F 54             cmp ecx,dword ptr ds:\[edi+54\]   
001429CD     76 07               jbe short 001429D6   
001429CF     33C0                xor eax,eax   
001429D1     E9 80000000         jmp 00142A56   
001429D6     8BF7                mov esi,edi   
001429D8     81C6 F8000000       add esi,0F8   
001429DE     8B55 FC             mov edx,dword ptr ss:\[ebp\-4\]   
001429E1     83EA 01             sub edx,1   
001429E4     8B46 24             mov eax,dword ptr ds:\[esi+24\]   
001429E7     0D 00000080         or eax,80000000   
001429EC     8946 24             mov dword ptr ds:\[esi+24\],eax   
001429EF     83C6 28             add esi,28   
001429F2     4A                  dec edx   
001429F3     0BD2                or edx,edx   
001429F5     75 ED               jnz short 001429E4   
001429F7     8BD6                mov edx,esi                      ; COOLcryp.004016F3   
001429F9     83C2 28             add edx,28   
001429FC     8B46 0C             mov eax,dword ptr ds:\[esi+C\]   
001429FF     0346 08             add eax,dword ptr ds:\[esi+8\]   
00142A02     68 00100000         push 1000   
00142A07     50                  push eax//把上面这些代码复制、粘贴到 00401D71 处就 OK 了  :-)    
00142A08     68 E11D4000         push 401DE1   
00142A0D     C3                  retn   
  
  
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
  
                2004-03-24  14:00  
  
 4dd466d0e1e0cc123afbe504da2462ca