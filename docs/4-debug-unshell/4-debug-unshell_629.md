

# 特定码——用 OllyDBG 手脱 Enigma Protector V1.12 加壳的试炼品

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-648.htm`](https://www.pediy.com/kssd/pediy08/pediy8-648.htm)

特定码——用 OllyDBG 手脱 Enigma Protector V1.12 加壳的试炼品  
              
             
               
破解下载：  http://www.unpack.cn/viewthread.php?tid=3806  
软件大小：  1.21 M    
软件简介：  Enigma Protector serves for software protection of applications. It defending your work from program crackers with state-of-the-art encryption, data compression, and other security features. Enigma Protector provides creation trial and registered applications, with possibility of the checking mode parameters with help internal Enigma API functions. It allows you to design and add a complete software protection and registration-key system to your existing programs. It works with any language which produce Windows PE files. Enigma works with Win PE 32 executables (\*.exe), screen savers (\*.scr), dynamic libraryes (\*.dll, \*.ocx).    
               
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教  
               
【调试环境】：WinXP、OllyDBD、PEiD、LordPE、ImportREC  
               
—————————————————————————————————   
【脱壳过程】：  
            
           
ENIGMA Protector 是俄罗斯新出的一款保护壳，有发展为猛壳的潜质。  
使用 ENIGMA Protector V1.12.CracKed.eXe 对 Win98 的记事本加壳。  
选项如下：  
  
  
—————————————————————————————————  
一、特定码  
  
  
特定码 ？  
  
先简单说个例子吧，UPX 的手动脱壳大家想必都清楚了。  
一般我们会 Ctrl+F 搜索 popad，或者 Ctrl+B 搜索 61E9  
  
0040EA1E    61                 popad  
//会搜索到这里  
0040EA1F    E9 A826FFFF        jmp 004010CC  
//飞向光明之巅  
  
我们都知道，对于 UPX 来说，popad 就是其跳转 OEP 前的特定命令。  
如果写 OllyScript 脚本的话可以这样：  
findop eip,#61#  
  
推而广之：对于其它壳来说，我们在其处理输入表、跳 OEP 等过程中有的也可以找到这样特定的命令，而这些命令对于某壳某些版本来说是固定的，可以被反复验证使用，借助这些特定代码，可以大大简化对此壳的脱壳步骤、节省脱壳时间。  
此类代码姑且我名之曰：特定码。  
  
当然，你可以用其它的名称。下面就使用 ENIGMA Protector 的特定码来演示对其试炼品脱壳。  
shoooo 写过《Enigma protector V1.02.build.4.00 主程序脱壳》，建议先参看。  
  
—————————————————————————————————  
二、代码段解码+首次处理输入表加密  
               
               
设置 OllyDBD 忽略所有异常选项。  
清除以前的所有断点，用 IsDebug 插件去掉 OllyDBD 的调试器标志。  
  
00421B93    60                 pushad  
//进入 OllyDBG 后暂停在这  
00421B94    E8 00000000        call 00421B99  
00421B99    5D                 pop ebp  
00421B9A    83C5 FA            add ebp,-6  
  
Alt+M，在代码段 401000 段设置内存写入断点，Shift+F9  
Wait ……  
  
00D4267F    F3:A5              rep movs dword ptr es:\[edi\],dword ptr ds:\[esi\]  
//中断在这里  
00D42681    89C1               mov ecx,eax  
00D42683    83E1 03            and ecx,3  
00D42686    F3:A4              rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]  
00D42688    5F                 pop edi  
00D42689    5E                 pop esi  
00D4268A    C3                 retn  
  
Enigma Protector 对程序输入表加密的样式如下：  
0040343A    FF15 F0634000      call dword ptr ds:\[4063F0\] ; SHELL32.ShellExecuteA  
//原始的  
0040343A    FF15 396ED900      call dword ptr ds:\[D96E39\] ; 00D97318  
//加密后  
\[D96E39\]处原先保存的是 API 系统函数地址，Enigma Protector 在下面要改成加密地址。  
或许不容易理解，多做几遍认真分析就明白了。  
  
Ctrl+S 搜索命令序列：  
inc esi  
dec ebp  
  
00D509D9    8B07               mov eax,dword ptr ds:\[edi\]  
00D509DB    33D2               xor edx,edx  
00D509DD    E8 DA050000        call 00D50FBC  
00D509E2    8BD8               mov ebx,eax  
00D509E4    84DB               test bl,bl  
00D509E6    74 24              je short 00D50A0C  
00D509E8    33C9               xor ecx,ecx  
00D509EA    8ACB               mov cl,bl  
00D509EC    8BD7               mov edx,edi  
00D509EE    8B4424 08          mov eax,dword ptr ss:\[esp+8\]  
00D509F2    E8 C145FFFF        call 00D44FB8  
00D509F7    33C0               xor eax,eax  
00D509F9    8AC3               mov al,bl  
00D509FB    03F8               add edi,eax  
00D509FD    33C0               xor eax,eax  
00D509FF    8AC3               mov al,bl  
00D50A01    014424 08          add dword ptr ss:\[esp+8\],eax  
00D50A05    33DB               xor ebx,ebx  
00D50A07    E9 82F8FFFF        jmp 00D5028E  
00D50A0C    803C24 00          cmp byte ptr ss:\[esp\],0  
00D50A10    75 21              jnz short 00D50A33  
00D50A12    8B4424 08          mov eax,dword ptr ss:\[esp+8\]  
00D50A16    C600 68            mov byte ptr ds:\[eax\],68  
00D50A19    FF4424 08          inc dword ptr ss:\[esp+8\]  
00D50A1D    8B4424 08          mov eax,dword ptr ss:\[esp+8\]  
00D50A21    8938               mov dword ptr ds:\[eax\],edi  
00D50A23    834424 08 04       add dword ptr ss:\[esp+8\],4  
00D50A28    8B4424 08          mov eax,dword ptr ss:\[esp+8\]  
00D50A2C    C600 C3            mov byte ptr ds:\[eax\],0C3  
00D50A2F    FF4424 08          inc dword ptr ss:\[esp+8\]  
00D50A33    8D04F6             lea eax,dword ptr ds:\[esi+esi\*8\]  
00D50A36    8B15 1000D700      mov edx,dword ptr ds:\[D70010\]  
00D50A3C    8B4C24 04          mov ecx,dword ptr ss:\[esp+4\]  
00D50A40    894C02 01          mov dword ptr ds:\[edx+eax+1\],ecx  
//改写加密的 API 地址，需要修改。  
00D50A44    46                 inc esi  
//找到这里  
00D50A45    4D                 dec ebp  
00D50A46    0F85 D3F7FFFF      jnz 00D5021F  
00D50A4C    83C4 14            add esp,14  
00D50A4F    5D                 pop ebp  
00D50A50    5F                 pop edi  
00D50A51    5E                 pop esi  
00D50A52    5B                 pop ebx  
00D50A53    C3                 retn  
  
但是现在还不能修改 00D50A40，因为有检验。继续看  
  
  
—————————————————————————————————  
三、去除壳检验  
  
  
注意，此时我们不能使用硬件断点也不能使用普通断点以及修改壳代码。  
硬件断点的 Anti 没有分析，猜测是采用类似 tELock 清除 DRx 的方法，而壳代码内的普通断点则是由壳检验来“包含”检测。  
  
现在选择 00D50A40 处的几个字节，右键->断点->内存访问  
Shit+F9，中断在 00D525E8 处，这里就是计算检验值了。  
  
00D525E3    33F6               xor esi,esi  
00D525E5    8D143E             lea edx,dword ptr ds:\[esi+edi\]  
00D525E8    8A12               mov dl,byte ptr ds:\[edx\]  
00D525EA    92                 xchg eax,edx  
00D525EB    E8 BCFFFFFF        call 00D525AC  
00D525F0    46                 inc esi  
00D525F1    4B                 dec ebx  
00D525F2    75 F1              jnz short 00D525E5  
//循环计算壳代码检验值  
00D525F4    83F0 FF            xor eax,FFFFFFFF  
00D525F7    EB 02              jmp short 00D525FB  
00D525F9    33C0               xor eax,eax  
00D525FB    5F                 pop edi  
00D525FC    5E                 pop esi  
00D525FD    5B                 pop ebx  
00D525FE    5D                 pop ebp  
00D525FF    C2 0800            retn 8  
  
清除内存断点，F4 至 00D525F7 处，EAX=6386B0E0，这个就是此程序壳代码的检验值。  
因为我们避开代码段、输入表加密时要修改壳代码，所以这里需要把检验值固定下来，否则壳就挂了。  
  
0D525F4 处固定检验值修改如下：  
00D525F4    B8 E0B08663        mov eax,6386B0E0   ★  
00D525F9    90                 nop  
00D525FA    90                 nop  
00D525FB    5F                 pop edi  
00D525FC    5E                 pop esi  
00D525FD    5B                 pop ebx  
00D525FE    5D                 pop ebp  
00D525FF    C2 0800            retn 8  
  
  
现在可以修改 00D50A40 处避开加密了：  
00D50A40    894C02 01          mov dword ptr ds:\[edx+eax+1\],ecx  
//改写加密的 API 地址，修改为 NOP   ★  
00D50A44    46                 inc esi  
  
  
————————————————————————  
附：ENIGMA Protector 对 API 普通断点的检测  
  
00D50BAA    8B1E               mov ebx,dword ptr ds:\[esi\]  
00D50BAC    80FB CC            cmp bl,0CC  
00D50BAF    75 18              jnz short 00D50BC9  
00D50BB1    8D45 F8            lea eax,dword ptr ss:\[ebp-8\]  
00D50BB4    8B15 E0D4D500      mov edx,dword ptr ds:\[D5D4E0\]  
00D50BBA    8B12               mov edx,dword ptr ds:\[edx\]  
00D50BBC    E8 872DFFFF        call 00D43948  
00D50BC1    8B45 F8            mov eax,dword ptr ss:\[ebp-8\]  
00D50BC4    E8 4F000000        call 00D50C18  
00D50BC9    B2 01              mov dl,1  
00D50BCB    8BC3               mov eax,ebx  
00D50BCD    E8 EA030000        call 00D50FBC  
00D50BD2    25 FF000000        and eax,0FF  
00D50BD7    03F0               add esi,eax  
00D50BD9    80C3 3E            add bl,3E  
00D50BDC    80EB 02            sub bl,2  
00D50BDF    72 09              jb short 00D50BEA  
00D50BE1    80EB 25            sub bl,25  
00D50BE4    74 04              je short 00D50BEA  
00D50BE6    85C0               test eax,eax  
00D50BE8    75 C0              jnz short 00D50BAA  
00D50BEA    83C7 04            add edi,4  
00D50BED    FF4D FC            dec dword ptr ss:\[ebp-4\]  
00D50BF0    75 AB              jnz short 00D50B9D  
  
  
—————————————————————————————————  
四、搞定普通加密的输入表  
  
  
Ctrl+S 在“整个段块”搜索命令序列：  
inc ebx  
dec dword ptr ss:\[esp\]  
找到在 00D4FB00 处，在其上的 00D4FAFA 处下断，Shift+F9，确定弹出的演示提示，中断下来  
  
00D4FAA8    E8 3B47FFFF        call 00D441E8  
00D4FAAD    48                 dec eax  
00D4FAAE    85C0               test eax,eax  
00D4FAB0    72 54              jb short 00D4FB06  
00D4FAB2    40                 inc eax  
00D4FAB3    890424             mov dword ptr ss:\[esp\],eax  
00D4FAB6    33DB               xor ebx,ebx  
00D4FAB8    8D04DB             lea eax,dword ptr ds:\[ebx+ebx\*8\]  
00D4FABB    8B16               mov edx,dword ptr ds:\[esi\]  
00D4FABD    8B4402 05          mov eax,dword ptr ds:\[edx+eax+5\]  
00D4FAC1    E8 2247FFFF        call 00D441E8  
00D4FAC6    8BD0               mov edx,eax  
00D4FAC8    4A                 dec edx  
00D4FAC9    85D2               test edx,edx  
00D4FACB    72 33              jb short 00D4FB00  
00D4FACD    42                 inc edx  
00D4FACE    33C0               xor eax,eax  
00D4FAD0    8D0CDB             lea ecx,dword ptr ds:\[ebx+ebx\*8\]  
00D4FAD3    8B3E               mov edi,dword ptr ds:\[esi\]  
00D4FAD5    8B7C0F 05          mov edi,dword ptr ds:\[edi+ecx+5\]  
00D4FAD9    833C87 00          cmp dword ptr ds:\[edi+eax\*4\],0  
00D4FADD    74 1D              je short 00D4FAFC  
00D4FADF    8B3E               mov edi,dword ptr ds:\[esi\]  
00D4FAE1    8D4C0F 01          lea ecx,dword ptr ds:\[edi+ecx+1\]  
00D4FAE5    8D3CDB             lea edi,dword ptr ds:\[ebx+ebx\*8\]  
00D4FAE8    8B2E               mov ebp,dword ptr ds:\[esi\]  
00D4FAEA    8B7C3D 05          mov edi,dword ptr ss:\[ebp+edi+5\]  
00D4FAEE    8B3C87             mov edi,dword ptr ds:\[edi+eax\*4\]  
00D4FAF1    8B2D C4D5D500      mov ebp,dword ptr ds:\[D5D5C4\]  
00D4FAF7    037D 00            add edi,dword ptr ss:\[ebp\]  
00D4FAFA    890F               mov dword ptr ds:\[edi\],ecx  
//修改 IAT 地址为加密地址，这里下断。  
//修改为：JMP 00D4FB0C  ★  
00D4FAFC    40                 inc eax  
00D4FAFD    4A                 dec edx  
00D4FAFE    75 D0              jnz short 00D4FAD0  
00D4FB00    43                 inc ebx  
//找到这里  
00D4FB01    FF0C24             dec dword ptr ss:\[esp\]  
00D4FB04    75 B2              jnz short 00D4FAB8  
00D4FB06    5A                 pop edx  
00D4FB07    5D                 pop ebp  
00D4FB08    5F                 pop edi  
00D4FB09    5E                 pop esi  
00D4FB0A    5B                 pop ebx  
00D4FB0B    C3                 retn  
//再次下断  
  
Patch 代码如下：  
00D4FB0C    56                 push esi  
00D4FB0D    53                 push ebx  
00D4FB0E    8B31               mov esi,dword ptr ds:\[ecx\] ; shell32.ShellExecuteA  
//\[ecx\]是正确的 API 函数系统地址  
00D4FB10    8B1F               mov ebx,dword ptr ds:\[edi\]  
//\[edi\]是 IAT 地址  
00D4FB12    8933               mov dword ptr ds:\[ebx\],esi  
//放入正确的 API 函数系统地址  
00D4FB14    5B                 pop ebx  
00D4FB15    5E                 pop esi  
00D4FB16    EB E4              jmp short 00D4FAFC  
//循环  
  
从 OllyDBG 中二进制复制：  
56 53 8B 31 8B 1F 89 33 5B 5E EB E4  
  
写好 Patch 代码后在 00D4FB0B 处下断，Shift+F9，中断后输入表处理完毕  
可以运行 LordPE 完全 Dump 此进程了，否则下面要进行某些重定位处理了  ★  
  
在数据窗口察看输入表函数：  
004062DC  00000000  
004062E0  77DAEBE7  advapi32.RegSetValueExA  
//开始  
004062E4  77DA7883  advapi32.RegQueryValueExA  
…… ……  
00406514  763300CE  comdlg32.CommDlgExtendedError  
00406518  76322533  comdlg32.GetFileTitleA  
0040651C  00000000  
//结束  
  
运行 ImportREC，填入 IAT RVA=000062E0，Size=0000023C  
获取输入表，发现几个无效的特殊函数，等到 OEP 后再分析看看。  
  
  
—————————————————————————————————  
五、避开数据段地址重定位  
  
  
Enigma Protector 还会对数据段地址重定位，如：  
00401159    A1 B8574000        mov eax,dword ptr ds:\[4057B8\]  
//原始的  
00401159    A1 B8F7D800        mov eax,dword ptr ds:\[D8F7B8\]  
//重定位后  
  
Ctrl+F 搜索命令:  
and eax,7FFFFFFF  
找到在 00D5AC35 处，在其上的 00D5ABE0 处下断，Shift+F9，中断后取消断点  
  
00D5ABE0    A9 00000080        test eax,80000000  
//这里下断  
00D5ABE5    75 28              jnz short 00D5AC0F  
00D5ABE7    8B15 84D4D500      mov edx,dword ptr ds:\[D5D484\]  
00D5ABED    8B12               mov edx,dword ptr ds:\[edx\]  
//\[edx\]=\[00D5C304\]=00D8F000    ★  
00D5ABEF    8B0D C4D5D500      mov ecx,dword ptr ds:\[D5D5C4\]  
00D5ABF5    2B11               sub edx,dword ptr ds:\[ecx\]  
00D5ABF7    8B0D C8D5D500      mov ecx,dword ptr ds:\[D5D5C8\]  
00D5ABFD    2B91 ED000000      sub edx,dword ptr ds:\[ecx+ED\]  
00D5AC03    8B0D C4D5D500      mov ecx,dword ptr ds:\[D5D5C4\]  
00D5AC09    0301               add eax,dword ptr ds:\[ecx\]  
00D5AC0B    0110               add dword ptr ds:\[eax\],edx  
00D5AC0D    EB 36              jmp short 00D5AC45  
00D5AC0F    8B0D 84D4D500      mov ecx,dword ptr ds:\[D5D484\]  
00D5AC15    8B09               mov ecx,dword ptr ds:\[ecx\]  
00D5AC17    8B35 C4D5D500      mov esi,dword ptr ds:\[D5D5C4\]  
00D5AC1D    2B0E               sub ecx,dword ptr ds:\[esi\]  
00D5AC1F    8B15 C8D5D500      mov edx,dword ptr ds:\[D5D5C8\]  
00D5AC25    8B92 ED000000      mov edx,dword ptr ds:\[edx+ED\]  
00D5AC2B    2BCA               sub ecx,edx  
00D5AC2D    8B35 84D4D500      mov esi,dword ptr ds:\[D5D484\]  
00D5AC33    8B36               mov esi,dword ptr ds:\[esi\]  
00D5AC35    25 FFFFFF7F        and eax,7FFFFFFF  
//找到这里  
00D5AC3A    03F0               add esi,eax  
00D5AC3C    A1 C8D5D500        mov eax,dword ptr ds:\[D5D5C8\]  
00D5AC41    2BF2               sub esi,edx  
00D5AC43    010E               add dword ptr ds:\[esi\],ecx  
00D5AC45    8305 D002D700 04   add dword ptr ds:\[D702D0\],4  
00D5AC4C    A1 D002D700        mov eax,dword ptr ds:\[D702D0\]  
00D5AC51    8B00               mov eax,dword ptr ds:\[eax\]  
00D5AC53    85C0               test eax,eax  
00D5AC55    75 89              jnz short 00D5ABE0  
00D5AC57    A1 C8D5D500        mov eax,dword ptr ds:\[D5D5C8\]  
//数据段地址重定位处理完毕  
00D5AC5C    8378 34 00         cmp dword ptr ds:\[eax+34\],0  
00D5AC60    0F84 9E000000      je 00D5AD04  
  
当我们走至 00D5ABED 时，在数据窗口中跟随\[edx\]，00D8F000 处保存的就是还未重定位的数据段！  
看看 Test.ENIGMA.Protector.eXe 的数据段 VirtualSize=00001000，00D8F000+00001000＝00D90000  
在 OllyDBG 中把 00D8F000-00D90000 处数据二进制复制，然后用 WinHex 保存为 data.bin  
F4 至 00D5AC57 处，数据段地址重定位处理完毕  
    
  
—————————————————————————————————  
六、OEP + Stolen OEP Code  
  
  
准备找 OEP 吧，你也可以根据目标程序的语言特征或者其它方法来走至 OEP  
Ctrl+S 在“整个段块”搜索命令序列：  
popad  
retn  
找到在 00D4B4DB 处，选择 00D4B4DB 处一些字节，设置内存访问断点  
  
00D4B4C4    68 544F4550        push 50454F54  
00D4B4C9    60                 pushad  
00D4B4CA    31C0               xor eax,eax  
00D4B4CC    B9 5453495A        mov ecx,5A495354  
00D4B4D1    BF 54414452        mov edi,52444154  
00D4B4D6    F2:AA              repne stos byte ptr es:\[edi\]  
00D4B4D8    47                 inc edi  
00D4B4D9    AB                 stos dword ptr es:\[edi\]  
00D4B4DA    AB                 stos dword ptr es:\[edi\]  
00D4B4DB    61                 popad  
//找到这里，设置内存访问断点  
00D4B4DC    C3                 retn  
  
Shift+F9，中断后取消内存断点  
  
00D525E8    8A12               mov dl,byte ptr ds:\[edx\]  
//中断在这里  
00D525EA    92                 xchg eax,edx  
00D525EB    E8 BCFFFFFF        call 00D525AC  
00D525F0    46                 inc esi  
00D525F1    4B                 dec ebx  
00D525F2    75 F1              jnz short 00D525E5  
00D525F4    B8 E0B08663        mov eax,6386B0E0  
00D525F9    90                 nop  
00D525FA    90                 nop  
00D525FB    5F                 pop edi  
00D525FC    5E                 pop esi  
00D525FD    5B                 pop ebx  
00D525FE    5D                 pop ebp  
00D525FF    C2 0800            retn 8  
  
原来这里还要校验，取消内存断点，00D525FF 处下断，Shift+F9，中断后取消断点。  
再次在 00D4B4DB 处设置内存访问断点，Shift+F9，中断后取消断点。  
  
00D4B52C    8B16               mov edx,dword ptr ds:\[esi\]  
//中断在这里  
00D4B52E    81FA 54414452      cmp edx,52444154  
00D4B534    74 18              je short 00D4B54E  
00D4B536    81FA 544F4550      cmp edx,50454F54  
00D4B53C    74 10              je short 00D4B54E  
00D4B53E    81FA 5453495A      cmp edx,5A495354  
00D4B544    74 08              je short 00D4B54E  
00D4B546    81FA 43524354      cmp edx,54435243  
00D4B54C    75 40              jnz short 00D4B58E  
00D4B54E    8BDA               mov ebx,edx  
00D4B550    81EB 544F4550      sub ebx,50454F54  
00D4B556    74 1F              je short 00D4B577  
00D4B558    81EB 00F2FE01      sub ebx,1FEF200  
00D4B55E    74 12              je short 00D4B572  
00D4B560    81EB EF10FF01      sub ebx,1FF10EF  
00D4B566    74 19              je short 00D4B581  
00D4B568    81EB 11010606      sub ebx,6060111  
00D4B56E    74 0C              je short 00D4B57C  
00D4B570    EB 12              jmp short 00D4B584  
00D4B572    8B55 0C            mov edx,dword ptr ss:\[ebp+C\]  
00D4B575    EB 0D              jmp short 00D4B584  
00D4B577    8B55 FC            mov edx,dword ptr ss:\[ebp-4\]  
00D4B57A    EB 08              jmp short 00D4B584  
00D4B57C    8B55 10            mov edx,dword ptr ss:\[ebp+10\]  
00D4B57F    EB 03              jmp short 00D4B584  
00D4B581    8B55 08            mov edx,dword ptr ss:\[ebp+8\]  
00D4B584    8910               mov dword ptr ds:\[eax\],edx  
00D4B586    83C0 04            add eax,4  
00D4B589    83C6 04            add esi,4  
00D4B58C    EB 9E              jmp short 00D4B52C  
00D4B58E    81FA 54454E44      cmp edx,444E4554  
00D4B594    74 06              je short 00D4B59C  
00D4B596    8810               mov byte ptr ds:\[eax\],dl  
//注意这里，EAX=00D92CD8，壳把 00D4B4DB 处数据又挪移了   ★  
00D4B598    40                 inc eax  
00D4B599    46                 inc esi  
00D4B59A    EB 90              jmp short 00D4B52C  
00D4B59C    2BF1               sub esi,ecx  
00D4B59E    8BCE               mov ecx,esi  
00D4B5A0    8BC1               mov eax,ecx  
00D4B5A2    5E                 pop esi  
00D4B5A3    5B                 pop ebx  
00D4B5A4    59                 pop ecx  
00D4B5A5    5D                 pop ebp  
00D4B5A6    C2 0C00            retn 0C  
//这里下断，，Shift+F9，中断后取消断点。  
  
Enigma Protector 当完搬运工后我们去 00D92CD8 处看看  
在 00D92CDB 的 popad 处下断，Shift+F9，中断后取消断点。  
  
00D92CD7    AA                 stos byte ptr es:\[edi\]  
//清理战场  
00D92CD8    47                 inc edi  
00D92CD9    AB                 stos dword ptr es:\[edi\]  
00D92CDA    AB                 stos dword ptr es:\[edi\]  
00D92CDB    61                 popad  
//这里下断  
00D92CDC    C3                 retn  
//返回到 004010D3          --> 飞向光明之巅  ★  
  
  
004010D3    FF15 E0634000      call dword ptr ds:\[4063E0\]  
//Stolen OEP  
004010D9    8BF0               mov esi,eax  
004010DB    8A00               mov al,byte ptr ds:\[eax\]  
004010DD    3C 22              cmp al,22  
004010DF    75 13              jnz short 004010F4  
  
很明显，OEP 被 Stolen 了几行代码。  
由于 Enigma Protector 这部分处理比较长，没有跟踪了，直接看堆栈和寄存器情况恢复吧。  
  
EAX FFFFFF02  
ECX 00D92C9A  
EDX 29ACE3D8  
EBX 07A25951  
ESP 0012FF78  
EBP 0012FFC0  
ESI 6B7BFD9F  
EDI 29ACE452  
EIP 004010D3  
  
0012FF78    6B7BFD9F  
0012FF7C    00000000  
0012FF80    00000000  
0012FF84    00000000  
  
ESP=0012FF78  
EBP=0012FFC0  
ESI=6B7BFD9F  
  
Stolen OEP Code 修复代码为：  
  push ebp  
  mov ebp,esp  
  sub esp,44  
  push esi  
  
当然，Stolen OEP Code 这样修复需要你分析好调试器当时情况以及了解各编译语言的特征。  
  
  
—————————————————————————————————  
七、修复输入表的特殊函数+完成脱壳  
  
  
ImportREC 里面显示的无效函数如下：  
0  00006394  ?  0000  00D4E50C  
0  00006398  ?  0000  00D4E4FC  
0  000063E0  ?  0000  00D4E74C  
  
Ctrl+G:00D4E50C  
00D4E50C    55                 push ebp  
00D4E50D    8BEC               mov ebp,esp  
00D4E50F    8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
00D4E512    85C0               test eax,eax  
00D4E514    74 08              je short 00D4E51E  
00D4E516    50                 push eax  
00D4E517    E8 4C69FFFF        call 00D44E68  ; jmp to kernel32.GetModuleHandleA  
00D4E51C    EB 05              jmp short 00D4E523  
00D4E51E    A1 D0FFD600        mov eax,dword ptr ds:\[D6FFD0\]  
00D4E523    5D                 pop ebp  
00D4E524    C2 0400            retn 4  
可以判定 00006394 是 GetModuleHandleA  
  
Ctrl+G:00D4E4FC  
00D4E4FC    55                 push ebp  
00D4E4FD    8BEC               mov ebp,esp  
00D4E4FF    8B45 08            mov eax,dword ptr ss:\[ebp+8\]  
00D4E502    50                 push eax  
00D4E503    E8 0869FFFF        call 00D44E10 ; jmp to kernel32.ExitProcess  
00D4E508    5D                 pop ebp  
00D4E509    C2 0400            retn 4  
可以判定 00006398 是 ExitProcess  
  
000063E0 的需要跟踪看看了，正好在 Stolen OEP 处，跟踪看看  
004010D3    FF15 E0634000      call dword ptr ds:\[4063E0\]  
00D4E74C    E8 02000000        call 00D4E753  
00D4E753    83C4 04            add esp,4  
00D4E756    68 7EE7D400        push 0D4E77E  
00D4E75B    64:FF35 00000000   push dword ptr fs:\[0\]  
00D4E762    EB 05              jmp short 00D4E769  
00D4E769    64:8925 00000000   mov dword ptr fs:\[0\],esp  
00D4E770    31C0               xor eax,eax  
00D4E772    E8 02000000        call 00D4E779  
00D4E779    83C4 04            add esp,4  
00D4E77C    3100               xor dword ptr ds:\[eax\],eax  
//异常  
00D4E77E    E8 05000000        call 00D4E788  
//SE 句柄   下断，Shift+F9 后中断  
00D4E788    83C4 04            add esp,4  
00D4E78B    8B4424 0C          mov eax,dword ptr ss:\[esp+C\]  
00D4E78F    C780 B8000000 B5E7>mov dword ptr ds:\[eax+B8\],0D4E7B5  
//00D4E7B5 处下断  
00D4E799    8B0D E4FFD600      mov ecx,dword ptr ds:\[D6FFE4\]  
//注意\[D6FFE4\]处的值  
//"D:\\UnPacK\\ENIGMA Protector\\Enigma Protector V1.12\\Test\\Test.ENIGMA.Protector.eXe""  
00D4E79F    8988 B0000000      mov dword ptr ds:\[eax+B0\],ecx  
00D4E7A5    E8 05000000        call 00D4E7AF  
00D4E7AF    83C4 04            add esp,4  
00D4E7B2    31C0               xor eax,eax  
00D4E7B4    C3                 retn  
//Shift+F9  
00D4E7B5    EB 05              jmp short 00D4E7BC  
//00D4E7B5 处中断，EAX=\[D6FFE4\]处的值  
  
注意，\[D6FFE4\]处的值是壳执行 GetCommandLineA 后获得的结果  
Enigma Protector 准备了不少特殊函数，类似 ASProtect V1.2X 的特殊函数。  
  
在 ImportREC 里面修正这几个函数，填入 OEP RVA=10CC 后 FixDump  
修正 dumped\_.exe 的 OEP 处 Stolen OEP Code  
用 WinHex 把第五步获得的 data.bin 数据写入到 dumped\_.exe 的数据段 0X00005000 处，保存。  
终于完成这个试炼品的脱壳了。  
  
当然，Enigma Protector 主程序的脱壳比试炼品要复杂，有几个 SDK 需要修复。  
  
  
—————————————————————————————————  
八、特定码  
  
  
痛并快乐着 ……  
用心地写脱壳教程，越来越像是自虐  
  
如何知道哪些是可用的特定码？  
多次的调试和分析从而总结出来的。  
  
特定码：  
特定壳在运行的特定流程中可以被利用的特定代码。  
  
            
—————————————————————————————————                                   
         ,     \_/   
        /| \_.-~/            \\\_     ,        青春都一晌  
       ( /~   /              \\~-.\_ |\\  
       \`\\\\  \_/                \\   ~\\ )          忍把浮名   
   \_-~~~-.)  )\_\_/;;,.          \\\_  //'  
  /'\_,\\   --~   \\ ~~~-  ,;;\\\_\_\_(  (.-~~~-.        换了脱壳轻狂  
 \`~ \_( ,\_..--\\ (     ,;'' /    ~--   /.\_\`\\   
  /~~//'   /' \`~\\         ) /--..\_, )\_  \`~  
  "  \`~"  "      \`"      /~'\`\\    \`\\\\~~\\     
                         "     "   "~'  ""  
      
              UnPacKed By :  fly  
               2006-04-14 19:00 

 4dd466d0e1e0cc123afbe504da2462ca