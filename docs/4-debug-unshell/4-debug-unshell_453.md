

# ASProtect 1.23 SDK

> 原文：[`www.pediy.com/kssd/pediy06/pediy6486.htm`](https://www.pediy.com/kssd/pediy06/pediy6486.htm)

【脱文标题】 ASProtect 1.23 SDK 之 Aspack2.12r 主程序脱壳去暗桩   
  
【脱文作者】 weiyi75\[Dfcg\]   
  
【作者邮箱】 weiyi75@sohu.com   
  
【作者主页】 Dfcg 官方大本营   
  
【使用工具】 Ollydbg1.10b,Imprec1.42,LoadPe   
  
【破解平台】 Win2000/XP   
  
【软件名称】 Aspack2.12r 主程序   
  
【下载地址】 http://www.pediy.com/tools/PACK/Packers/Aspack/aspack212r.zip   
  
【软件简介】    
  
  ASPack is an advanced Win32 executable file compressor, capable of reducing the file size of 32-bit Windows programs by as much as 70%. (ASPack's compression ratio improves upon the industry-standard zip file format by as much as 10-20%.) ASPack makes Windows 95/98/NT programs and libraries smaller, and decrease load times across networks, and download times from the internet; it also protects programs against reverse engineering by non-professional hackers. Programs compressed with ASPack are self-contained and run exactly as before, with no runtime performance penalties.    
  
【软件大小】 189k   
  
【加壳方式】 ASProtect 1.22 - 1.23 Beta 21 -> Alexey Solodovnikov   
  
【保护方式】 RSAKEY 保护+30 天日期限制。   
  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）   
  
\--------------------------------------------------------------------------------   
  
【破解内容】   
  
前言，关于这个软件用 SDK 紧密结合壳，使脱壳文件不能直接运行。上次发过一篇 Aspack 没有 Key 的 Demo 版脱壳文章。   
  
相关页面   
  
http://www.chinadfcg.com/viewthread.php?tid=1247   
  
今天看见 Jeffzhang 用自己的名字可以注册成功,发现了这个壳使用 rsakey 保护时，会使 demo 版无法被 crack 成完整版本,如果你有 Key 脱壳时可以脱壳成完全整本,关键是自己的注册名字，不过有 Key 的时候不会太多。为了获取完整脱壳版本，我将 Key 导入了注册表。   
  
好，脱壳开始。   
  
这个版本的 Asprotect 对 Nt 调试器不检测，Softice 严格校验。OD 异常设置不忽略内存异常，其余全部忽略,载入程序。   
  
00401000 >  68 01704600     push ASPACK.00467001  //停在这里,F9 一直运行。    
00401005    E8 01000000     call ASPACK.0040100B   
0040100A    C3              retn   
0040100B    C3              retn   
0040100C    9F              lahf   
0040100D    24 9F           and al, 9F   
0040100F    24 9C           and al, 9C   
00401011    84E0            test al, ah   
00401013    10FB            adc bl, bh   
00401015    C022 90         shl byte ptr ds:\[edx\], 90   
00401018    1212            adc dl, byte ptr ds:\[edx\]   
0040101A    0921            or dword ptr ds:\[ecx\], esp   
0040101C    0148 70         add dword ptr ds:\[eax+70\], ecx   
0040101F    8BC2            mov eax, edx   
00401021    1242 02         adc al, byte ptr ds:\[edx+2\]   
......................................................................   
  
内存异常。   
  
00D30739    3100            xor dword ptr ds:\[eax\], eax   
00D3073B    EB 01           jmp short 00D3073E   
00D3073D    68 648F0500     push 58F64   
00D30742    0000            add byte ptr ds:\[eax\], al   
00D30744    00EB            add bl, ch   
00D30746    02E8            add ch, al   
00D30748    0158 68         add dword ptr ds:\[eax+68\], ebx   
00D3074B    F8              clc   
00D3074C  ^ E2 D2           loopd short 00D30720   
00D3074E    0068 74         add byte ptr ds:\[eax+74\], ch   
00D30751    08D3            or bl, dl   
00D30753    0068 A4         add byte ptr ds:\[eax\-5C\], ch   
00D30756    FD              std   
00D30757    D200            rol byte ptr ds:\[eax\], cl   
00D30759    68 54FAD200     push 0D2FA54   
00D3075E    68 2CF4D200     push 0D2F42C   
00D30763    68 BCEED200     push 0D2EEBC   
00D30768    68 4001D300     push 0D30140   
00D3076D    C3              retn   
......................................................................   
  
Shift+F9 22 次左右一直运行到硬盘指纹出现   
  
00D3099F    3100            xor dword ptr ds:\[eax\], eax   
00D309A1    EB 01           jmp short 00D309A4   
00D309A3    68 648F0500     push 58F64   
00D309A8    0000            add byte ptr ds:\[eax\], al   
00D309AA    00EB            add bl, ch   
00D309AC    02E8            add ch, al   
00D309AE    0158 B2         add dword ptr ds:\[eax\-4E\], ebx   
00D309B1    01B8 20CCD200   add dword ptr ds:\[eax+D2CC20\], edi   
00D309B7    E8 74C2FFFF     call 00D2CC30   
00D309BC    8BF0            mov esi, eax   
00D309BE    A1 A439D300     mov eax, dword ptr ds:\[D339A4\]   
00D309C3    8946 04         mov dword ptr ds:\[esi+4\], eax   
00D309C6    66:8B15 8C22D30>mov dx, word ptr ds:\[D3228C\]   
00D309CD    8BC6            mov eax, esi   
00D309CF    E8 BCC2FFFF     call 00D2CC90   
00D309D4    8BD8            mov ebx, eax   
00D309D6    85DB            test ebx, ebx   
00D309D8    74 0B           je short 00D309E5   
00D309DA    8D55 C8         lea edx, dword ptr ss:\[ebp\-38\]   
00D309DD    8D43 06         lea eax, dword ptr ds:\[ebx+6\]   
00D309E0    E8 8F9AFFFF     call 00D2A474   
00D309E5    66:8B15 DC22D30>mov dx, word ptr ds:\[D322DC\]   
00D309EC    8BC6            mov eax, esi   
00D309EE    E8 9DC2FFFF     call 00D2CC90   
......................................................................   
  
堆栈内容   
  
0012D71C   0012D724  指针到下一个 SEH 记录   
0012D720   00D30956  SE 句柄   
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00000000   
0012D740   00D45480  ASCII "ARInMgCAUXY="   
......................................................................   
  
btw:  这里涉及到 Aspr 壳的 pre-dip 的知识，Volx 关于 Aspr 壳的 pre-dip 我到现在还没有体会,另外 Asprotect 作者自己的拳头产品比普通的 Asprotect 程序更难。   
  
这里必须不断跟踪亲身在壳和程序中体会。   
  
再按一次 Shift+F9,注意每个 Asprotect 的处理方法都不同，不然也不会写那么多文章。   
  
00D30DFC    3100            xor dword ptr ds:\[eax\], eax   
00D30DFE    EB 01           jmp short 00D30E01   
00D30E00    68 648F0500     push 58F64   
00D30E05    0000            add byte ptr ds:\[eax\], al   
00D30E07    00EB            add bl, ch   
00D30E09    02E8            add ch, al   
00D30E0B    0158 E8         add dword ptr ds:\[eax\-18\], ebx   
00D30E0E    25 0000008B     and eax, 8B000000   
00D30E13    44              inc esp   
00D30E14    24 0C           and al, 0C   
00D30E16    8380 B8000000 0>add dword ptr ds:\[eax+B8\], 2   
00D30E1D    51              push ecx   
00D30E1E    31C9            xor ecx, ecx   
00D30E20    8948 04         mov dword ptr ds:\[eax+4\], ecx   
00D30E23    8948 08         mov dword ptr ds:\[eax+8\], ecx   
00D30E26    8948 0C         mov dword ptr ds:\[eax+C\], ecx   
00D30E29    8948 10         mov dword ptr ds:\[eax+10\], ecx   
00D30E2C    C740 18 5501000>mov dword ptr ds:\[eax+18\], 155   
00D30E33    59              pop ecx   
00D30E34    31C0            xor eax, eax   
00D30E36    C3              retn   
......................................................................   
  
堆栈内容   
  
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00000000   
0012D740   00D40414  ASCII "SOFTWAREASPack"   
......................................................................   
  
继续 Shift+F9   
  
00D30E3F    3100            xor dword ptr ds:\[eax\], eax   
00D30E41    64:8F05 0000000>pop dword ptr fs:\[0\]   
00D30E48    58              pop eax   
00D30E49    833F 00         cmp dword ptr ds:\[edi\], 0   
00D30E4C    74 0C           je short 00D30E5A   
00D30E4E    8B07            mov eax, dword ptr ds:\[edi\]   
00D30E50    E8 B724FFFF     call 00D2330C   
00D30E55    83F8 0A         cmp eax, 0A   
00D30E58    7D 0A           jge short 00D30E64   
00D30E5A    E8 15D1FFFF     call 00D2DF74   
00D30E5F    E9 71030000     jmp 00D311D5   
00D30E64    B8 6836D300     mov eax, 0D33668   
00D30E69    E8 CA25FFFF     call 00D23438   
00D30E6E    8945 E0         mov dword ptr ss:\[ebp\-20\], eax   
00D30E71    A1 6836D300     mov eax, dword ptr ds:\[D33668\]   
00D30E76    E8 9124FFFF     call 00D2330C   
00D30E7B    8945 E4         mov dword ptr ss:\[ebp\-1C\], eax   
00D30E7E    A1 D039D300     mov eax, dword ptr ds:\[D339D0\]   
00D30E83    8945 D8         mov dword ptr ss:\[ebp\-28\], eax   
00D30E86    A1 D439D300     mov eax, dword ptr ds:\[D339D4\]   
00D30E8B    8945 DC         mov dword ptr ss:\[ebp\-24\], eax   
00D30E8E    8D95 ACD7FFFF   lea edx, dword ptr ss:\[ebp\-2854\]   
00D30E94    8B07            mov eax, dword ptr ds:\[edi\]   
00D30E96    E8 D9F8FFFF     call 00D30774   
00D30E9B    8B85 ACD7FFFF   mov eax, dword ptr ss:\[ebp\-2854\]   
00D30EA1    8D95 B0D7FFFF   lea edx, dword ptr ss:\[ebp\-2850\]   
00D30EA7    E8 4093FFFF     call 00D2A1EC   
00D30EAC    8B95 B0D7FFFF   mov edx, dword ptr ss:\[ebp\-2850\]   
00D30EB2    8BC7            mov eax, edi   
00D30EB4    E8 5323FFFF     call 00D2320C   
00D30EB9    E8 3A000000     call 00D30EF8   
00D30EBE    68 C70ED300     push 0D30EC7   
00D30EC3    FF0424          inc dword ptr ss:\[esp\]   
00D30EC6    C3              retn   
......................................................................   
  
堆栈内容   
  
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00000000   
0012D740   00D40414  ASCII "SOFTWAREASPack"   
......................................................................   
  
00D30F07    3100            xor dword ptr ds:\[eax\], eax   
00D30F09    EB 01           jmp short 00D30F0C   
00D30F0B    68 648F0500     push 58F64   
00D30F10    0000            add byte ptr ds:\[eax\], al   
00D30F12    00EB            add bl, ch   
00D30F14    02E8            add ch, al   
00D30F16    0158 8D         add dword ptr ds:\[eax\-73\], ebx   
00D30F19    05 6C36D300     add eax, 0D3366C   
00D30F1E    50              push eax   
00D30F1F    E8 04F6FFFF     call 00D30528   
00D30F24    E8 3A000000     call 00D30F63   
00D30F29    68 320FD300     push 0D30F32   
00D30F2E    FF0424          inc dword ptr ss:\[esp\]   
00D30F31    C3              retn   
......................................................................   
  
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00D40430  ASCII    
  
"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt   
  
Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="   
0012D740   00D404EC  ASCII    
  
"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt   
  
Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="   
......................................................................   
  
00D305CC    8B43 02         mov eax, dword ptr ds:\[ebx+2\]   
00D305CF    50              push eax   
00D305D0    8D43 06         lea eax, dword ptr ds:\[ebx+6\]   
00D305D3    50              push eax   
00D305D4    E8 47BAFFFF     call 00D2C020   
00D305D9    85C0            test eax, eax   
00D305DB    74 15           je short 00D305F2   
00D305DD    33C9            xor ecx, ecx   
00D305DF    B2 01           mov dl, 1   
00D305E1    B8 F8ECD200     mov eax, 0D2ECF8                         ; ASCII 06,"EFound"   
00D305E6    E8 15ADFFFF     call 00D2B300   
00D305EB    E8 5C28FFFF     call 00D22E4C   
00D305F0    EB 13           jmp short 00D30605   
00D305F2    33C9            xor ecx, ecx   
00D305F4    B2 01           mov dl, 1   
00D305F6    B8 4C9CD200     mov eax, 0D29C4C   
00D305FB    E8 00ADFFFF     call 00D2B300   
00D30600    E8 4728FFFF     call 00D22E4C   
......................................................................   
  
堆栈内容   
  
0012D6D8   00D33670   
0012D6DC   00000010   
0012D6E0   0012D6EC  指针到下一个 SEH 记录   
0012D6E4   00D30612  SE 句柄   
0012D6E8   0012D718   
0012D6EC   0012D724  指针到下一个 SEH 记录   
0012D6F0   00D306C3  SE 句柄   
......................................................................   
  
00D30F72    3100            xor dword ptr ds:\[eax\], eax   
00D30F74    EB 01           jmp short 00D30F77   
00D30F76    68 648F0500     push 58F64   
00D30F7B    0000            add byte ptr ds:\[eax\], al   
00D30F7D    00EB            add bl, ch   
00D30F7F    02E8            add ch, al   
00D30F81    0158 8B         add dword ptr ds:\[eax\-75\], ebx   
00D30F84    07              pop es   
00D30F85    E8 8223FFFF     call 00D2330C   
00D30F8A    48              dec eax   
00D30F8B    50              push eax   
00D30F8C    8BC7            mov eax, edi   
00D30F8E    E8 A524FFFF     call 00D23438   
00D30F93    40              inc eax   
00D30F94    8D8D B0D7FFFF   lea ecx, dword ptr ss:\[ebp\-2850\]   
00D30F9A    5A              pop edx   
00D30F9B    E8 80D7FFFF     call 00D2E720   
00D30FA0    8B85 B0D7FFFF   mov eax, dword ptr ss:\[ebp\-2850\]   
00D30FA6    50              push eax   
00D30FA7    8D85 ACD7FFFF   lea eax, dword ptr ss:\[ebp\-2854\]   
00D30FAD    8B17            mov edx, dword ptr ds:\[edi\]   
00D30FAF    8A12            mov dl, byte ptr ds:\[edx\]   
00D30FB1    E8 1623FFFF     call 00D232CC   
......................................................................   
  
堆栈内容   
  
0012D71C   0012D724  指针到下一个 SEH 记录   
0012D720   00D30F29  SE 句柄   
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00D40430  ASCII    
  
"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt   
  
Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="   
0012D740   00D404EC  ASCII    
  
"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt   
  
Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="   
......................................................................   
  
00D31013    3100            xor dword ptr ds:\[eax\], eax   
00D31015    EB 01           jmp short 00D31018   
00D31017    68 648F0500     push 58F64   
00D3101C    0000            add byte ptr ds:\[eax\], al   
00D3101E    00EB            add bl, ch   
00D31020    02E8            add ch, al   
00D31022    0158 8B         add dword ptr ds:\[eax\-75\], ebx   
00D31025    45              inc ebp   
00D31026    C4E8            les ebp, eax                             ; 非法使用寄存器   
00D31028    E0 22           loopdne short 00D3104C   
00D3102A    FFFF            ???                                      ; 未知命令   
00D3102C    50              push eax   
00D3102D    8B45 C4         mov eax, dword ptr ss:\[ebp\-3C\]   
00D31030    E8 F723FFFF     call 00D2342C   
00D31035    8D4D D0         lea ecx, dword ptr ss:\[ebp\-30\]   
00D31038    5A              pop edx   
00D31039    E8 FED9FFFF     call 00D2EA3C   
00D3103E    8BD8            mov ebx, eax   
00D31040    E8 3A000000     call 00D3107F   
00D31045    68 4E10D300     push 0D3104E   
00D3104A    FF0424          inc dword ptr ss:\[esp\]   
00D3104D    C3              retn   
......................................................................   
  
堆栈内容   
  
0012D71C   0012D724  指针到下一个 SEH 记录   
0012D720   00D30FCA  SE 句柄   
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
......................................................................   
  
00D3108E    3100            xor dword ptr ds:\[eax\], eax   
00D31090    EB 01           jmp short 00D31093   
00D31092    68 648F0500     push 58F64   
00D31097    0000            add byte ptr ds:\[eax\], al   
00D31099    00EB            add bl, ch   
00D3109B    02E8            add ch, al   
00D3109D    0158 83         add dword ptr ds:\[eax\-7D\], ebx   
00D310A0    FB              sti   
00D310A1    0175 64         add dword ptr ss:\[ebp+64\], esi   
00D310A4    837D F8 00      cmp dword ptr ss:\[ebp\-8\], 0   
00D310A8    74 5E           je short 00D31108   
00D310AA    33C0            xor eax, eax   
00D310AC    55              push ebp   
00D310AD    68 F910D300     push 0D310F9   
00D310B2    64:FF30         push dword ptr fs:\[eax\]   
00D310B5    64:8920         mov dword ptr fs:\[eax\], esp   
00D310B8    8B45 F8         mov eax, dword ptr ss:\[ebp\-8\]   
00D310BB    E8 70F7FFFF     call 00D30830   
00D310C0    DD5D B4         fstp qword ptr ss:\[ebp\-4C\]   
00D310C3    9B              wait   
00D310C4    8B45 FC         mov eax, dword ptr ss:\[ebp\-4\]   
00D310C7    E8 64F7FFFF     call 00D30830   
00D310CC    DD5D AC         fstp qword ptr ss:\[ebp\-54\]   
00D310CF    9B              wait   
00D310D0    E8 D79BFFFF     call 00D2ACAC   
......................................................................   
  
堆栈内容   
  
0012D71C   0012D724  指针到下一个 SEH 记录   
0012D720   00D31045  SE 句柄   
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
......................................................................   
  
到这里，Alt+M 打开内存镜像。   
  
内存镜像，项目 12   
 地址=00401000     //对 401000 Code 段下内存访问断点，Shift+F9    
 大小=00042000 (270336.)   
 Owner=ASPACK   00400000   
 区段=   
 包含=code   
 类型=Imag 01001002   
 访问=R   
 初始访问=RWE   
......................................................................   
  
来到了 pre-dip 的总调用点。   
  
00442BA4    55              push ebp     //我们当前位置,Shift+F9 慢慢往下运行。    
00442BA5    8BEC            mov ebp, esp   
00442BA7    8B45 08         mov eax, dword ptr ss:\[ebp+8\]   
00442BAA    A3 08494400     mov dword ptr ds:\[444908\], eax   
00442BAF    5D              pop ebp   
00442BB0    C2 0400         retn 4   
00442BB3    90              nop   
00442BB4    55              push ebp   
00442BB5    8BEC            mov ebp, esp   
00442BB7    8B45 08         mov eax, dword ptr ss:\[ebp+8\]   
00442BBA    A3 0C494400     mov dword ptr ds:\[44490C\], eax   
00442BBF    5D              pop ebp   
00442BC0    C2 0400         retn 4   
00442BC3    90              nop   
00442BC4    55              push ebp   
00442BC5    8BEC            mov ebp, esp   
00442BC7    8B45 08         mov eax, dword ptr ss:\[ebp+8\]   
00442BCA    A3 BC564400     mov dword ptr ds:\[4456BC\], eax  //地址 4456bc 有秘密。    
00442BCF    5D              pop ebp   
00442BD0    C2 0400         retn 4   //当它运行到这里时。    
  
堆栈内容   
  
0012D71C   00D311EF  返回到 00D311EF 来自 ASPACK.00442BC4     
0012D720   00D49734  ASCII "SAC/UG2002!"   
0012D724   0012FFE0  指针到下一个 SEH 记录   
0012D728   00D312CF  SE 句柄   
0012D72C   0012FF90   
0012D730   00D20000   
0012D734   00D00000   
0012D738   00D306D4   
0012D73C   00D49670   
0012D740   00D49734  ASCII "SAC/UG2002!"  //这个是注册用户名。    
  
于是我们 dd 4456BC   
  
004456BC  00D49734  ASCII "SAC/UG2002!"  //这个地址就是我们存放自己名字的位置。    
  
00442BD3    90              nop   
00442BD4    55              push ebp   
00442BD5    8BEC            mov ebp, esp   
00442BD7    8B45 08         mov eax, dword ptr ss:\[ebp+8\]   
00442BDA    A3 00494400     mov dword ptr ds:\[444900\], eax   //剩余日期    
00442BDF    8B45 0C         mov eax, dword ptr ss:\[ebp+C\]   
00442BE2    A3 04494400     mov dword ptr ds:\[444904\], eax  //总共日期    
00442BE7    5D              pop ebp   
00442BE8    C2 0800         retn 8     
00442BEB    90              nop   
00442BEC    55              push ebp   
00442BED    8BEC            mov ebp, esp   
00442BEF    B9 B8564400     mov ecx, ASPACK.004456B8   
00442BF4    BA 20E94300     mov edx, ASPACK.0043E920   
00442BF9    A1 30564400     mov eax, dword ptr ds:\[445630\]   
00442BFE    E8 450EFEFF     call ASPACK.00423A48   
00442C03    5D              pop ebp   
00442C04    C3              retn   
00442C05    8D40 00         lea eax, dword ptr ds:\[eax\]   
00442C08    55              push ebp   
00442C09    8BEC            mov ebp, esp   
00442C0B    6A 24           push 24   
00442C0D    68 402C4400     push ASPACK.00442C40                     ; ASCII "Expired"   
00442C12    68 482C4400     push ASPACK.00442C48                     ; ASCII "Sorry, but the evaluation period of ASPack has    
  
expired!   
Would you like to register this program?"   //太君，你放心，我 101 年后注册这个程序。    
00442C17    6A 00           push 0   
00442C19    E8 0A25FCFF     call ASPACK.00405128   
00442C1E    83F8 06         cmp eax, 6   
00442C21    75 1B           jnz short ASPACK.00442C3E   
.............................................................................................   
  
我们继续 Shift+F9 执行到 Oep 入口。   
  
之前先检查一下刚才发现的几个关键点是否正确。   
  
dd  444900   
  
004448FC  0043DC8C  David.0043DC8C  //普通的 Asprotect 是这里存放用户地址。    
00444900  0000001E   //没注册的显示 30 天试用期   
00444904  0000001E   
  
004448FC  0043DC8C  ASPACK.0043DC8C   
00444900  00000001   //注册版这里直接是 1 天也就是无限制日期，其实脱壳后根本不会比较这里,日期限制就没有了。    
00444904  00000001    
  
dd 004456BC   
  
004456BC  00D49734  ASCII "SAC/UG2002!"   
  
00D49734  //这个地址是?壳中，名字也不是自己的。   
  
在脱壳前先写上名字，没用户名部分 Asprotect 程序脱壳后运行会出错，或显示没有注册，我们提前去掉这个暗桩。   
  
于是我们到程序尾部空地 445fe4  写上 Mr.David   
  
修正   
  
004456BC   
  
的   
  
00D49734   
  
为   
  
445fe4   
  
这个是 Od 的基本功，不会很难学下去啊。   
  
00442CE8    55              push ebp  //现在就可以 DUMP 了。    
00442CE9    8BEC            mov ebp, esp   
00442CEB    83C4 F4         add esp, -0C   
00442CEE    E8 D506FCFF     call ASPACK.004033C8   
00442CF3    E8 941CFCFF     call ASPACK.0040498C   
00442CF8    E8 A34FFCFF     call ASPACK.00407CA0   
00442CFD    E8 1AC2FCFF     call ASPACK.0040EF1C   
00442D02    E8 01C3FCFF     call ASPACK.0040F008   
00442D07    E8 58E2FCFF     call ASPACK.00410F64   
00442D0C    E8 2349FDFF     call ASPACK.00417634   
00442D11    E8 CA16FEFF     call ASPACK.004243E0   
00442D16    E8 6983FEFF     call ASPACK.0042B084   
00442D1B    E8 5896FEFF     call ASPACK.0042C378   
00442D20    E8 D796FEFF     call ASPACK.0042C3FC   
00442D25    E8 42A2FEFF     call ASPACK.0042CF6C   
00442D2A    E8 09A8FEFF     call ASPACK.0042D538   
00442D2F    E8 6CB7FEFF     call ASPACK.0042E4A0   
00442D34    E8 ABB7FEFF     call ASPACK.0042E4E4   
00442D39    E8 36B8FEFF     call ASPACK.0042E574   
00442D3E    E8 41DFFEFF     call ASPACK.00430C84   
00442D43    E8 4CAFFFFF     call ASPACK.0043DC94   
........................................................................   
  
接着用 Import REC 填入 oep 42CE8,iat 自动搜索-获得输入信息-显示无效的,右键先用追踪层次 1 修复大部分指针，剩下 8 个指针用 Asprotect1.2X 插件修复。运行修复后的程序出错，当然不可能这么简单了。所以接下来的工作就是寻找出错的位置然后修复。用 OD 载入修复后   
  
的程序，来到这里。   
  
00442D48  |.  A1 30564400   mov eax, dword ptr ds:\[445630\]   
00442D4D  |.  E8 E60CFEFF   call dumped\_.00423A38   
00442D52  |.  BA 842D4400   mov edx, dumped\_.00442D84                ;  ASCII "ASPack"   
00442D57  |.  A1 30564400   mov eax, dword ptr ds:\[445630\]   
00442D5C  |.  E8 F309FEFF   call dumped\_.00423754   
00442D61  |.  FF15 10494400 call dword ptr ds:\[444910\]  //这里指向壳中。    
  
跟踪原程序到达相应位置，F7 跟进。   
  
00D2C9A0    833D A835D300 0>cmp dword ptr ds:\[D335A8\], 0   
00D2C9A7    74 06           je short 00D2C9AF   
00D2C9A9    FF15 A835D300   call dword ptr ds:\[D335A8\]               ; ASPACK.00442BEC     
  
00442BEC    55              push ebp   
00442BED    8BEC            mov ebp, esp   
00442BEF    B9 B8564400     mov ecx, ASPACK.004456B8   
00442BF4    BA 20E94300     mov edx, ASPACK.0043E920   
00442BF9    A1 30564400     mov eax, dword ptr ds:\[445630\]  //原来是调用 pre-dip 的某处。    
00442BFE    E8 450EFEFF     call ASPACK.00423A48   
00442C03    5D              pop ebp   
00442C04    C3              retn   
.......................................................................   
  
好，明白原理，采取对策。   
  
dd 444910   
  
将   
  
原   
  
00D2C9A0   
  
修改为   
  
00442BEC    
  
复制修改到程序里，重启动 Od,F9 运行出错。   
  
堆栈友好提示。   
  
0012FDFC   0043F5E2  返回到 dumped\_.0043F5E2 来自 00D2C8F8   
0012FE00   0012FE54  指针到下一个 SEH 记录   
0012FE04   0043F8F4  SE 句柄   
0012FE08   0012FE4C   
0012FE0C   005516A8   
0012FE10   0043E920  dumped\_.0043E920   
..........................................................   
  
于是我们去   
  
0043F5DC   .  FF15 08494400 call dword ptr ds:\[444908\]   
0043F5E2   .  E9 83000000   jmp dumped\_.0043F66A   
0043F5E7      BE            db BE   
0043F5E8      46            db 46                                    ;  CHAR 'F'   
0043F5E9      23            db 23                                    ;  CHAR '#'   
0043F5EA      25            db 25                                    ;  CHAR '%'   
0043F5EB      9B            db 9B   
0043F5EC      3D            db 3D                                    ;  CHAR '='   
0043F5ED      40            db 40                                    ;  CHAR '@'   
0043F5EE      CB            db CB   
0043F5EF      FC            db FC   
0043F5F0      EF            db EF   
0043F5F1      93            db 93   
0043F5F2      C8            db C8   
0043F5F3      0F            db 0F   
0043F5F4      3C            db 3C                                    ;  CHAR '<'   
0043F5F5      F2            db F2   
0043F5F6      C5            db C5   
0043F5F7      0B            db 0B   
0043F5F8      24            db 24                                    ;  CHAR '$'   
0043F5F9      0C            db 0C   
...........................................................   
  
右键清除分析。   
  
0043F5DC    FF15 08494400   call dword ptr ds:\[444908\]  //访问壳中，解码。    
0043F5E2    E9 83000000     jmp dumped\_.0043F66A    //直接跳走，肯定有问题，我们脱过几个 Sdk 壳也有了一点经验了。    
0043F5E7    BE 4623259B     mov esi, 9B252346   
0043F5EC    3D 40CBFCEF     cmp eax, EFFCCB40   
0043F5F1    93              xchg eax, ebx   
0043F5F2    C8 0F3CF2       enter 3C0F, 0F2   
0043F5F6    C50B            lds ecx, fword ptr ds:\[ebx\]   
0043F5F8    24 0C           and al, 0C   
0043F5FA    A6              cmps byte ptr ds:\[esi\], byte ptr es:\[edi\>   
0043F5FB    A9 69B22AAF     test eax, AF2AB269   
0043F600    3AF8            cmp bh, al   
0043F602    6F              outs dx, dword ptr es:\[edi\]   
0043F603    52              push edx   
0043F604    3113            xor dword ptr ds:\[ebx\], edx   
0043F606    F4              hlt   
0043F607    47              inc edi   
0043F608    41              inc ecx   
0043F609    17              pop ss   
0043F60A    96              xchg eax, esi   
0043F60B    F6              ???                                      ; 未知命令   
0043F60C    8D8B F9436B80   lea ecx, dword ptr ds:\[ebx+806B43F9\]   
0043F612  ^ 7E E1           jle short dumped\_.0043F5F5   
0043F614    20F6            and dh, dh   
0043F616    04 14           add al, 14   
0043F618    9A 5D661EE7 A9D>call far D8A9:E71E665D   
0043F61F    79 24           jns short dumped\_.0043F645   
0043F621    294E 96         sub dword ptr ds:\[esi\-6A\], ecx   
0043F624    EF              out dx, eax   
0043F625    E8 5F8F6EBF     call BFB28589   
...........................................................   
  
  
0043F5DC    FF15 08494400   call dword ptr ds:\[444908\]  //访问壳中，解码。    
  
我们将 444908 指向壳中的地址，简单在程序中找一个 retn 地址给它，我找的是 401234   
  
于是    
  
dd    
  
444908   
  
00444908  00D2C8F8   //访问壳，改为 401234   
0044490C  00D2C924  //这个是解码时另一处访问壳,顺手改为 401234   
  
保存修改到文件，重启动 OD 来到 0043F5DC   
  
解码当然看原程序了，当经过   
  
0043F5DC   .  FF15 08494400 call dword ptr ds:\[444908\]   
  
后   
  
下面代码发生很大变化。   
  
先点分析代码过滤掉花指令，免的二进制复制有遗漏导致修复失败。   
  
0043F5E2   . /E9 01000000   jmp ASPACK.0043F5E8  //二进制复制下面到第一个 retn 的所有代码，虽然有多的，但绝对安全，不会漏掉一个代码。    
0043F5E7     |29            db 29                                    ;  CHAR ')'   
0043F5E8   > 8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F5EB   .  8B80 E0010000 mov eax, dword ptr ds:\[eax+1E0\]   
0043F5F1   .  8B40 44       mov eax, dword ptr ds:\[eax+44\]   
0043F5F4   .  33D2          xor edx, edx   
0043F5F6   .  E8 2993FDFF   call ASPACK.00418924   
0043F5FB   .  8D55 D4       lea edx, dword ptr ss:\[ebp\-2C\]   
0043F5FE   .  A1 BC564400   mov eax, dword ptr ds:\[4456BC\]   
0043F603   .  E8 586DFCFF   call ASPACK.00406360   
0043F608   .  8B55 D4       mov edx, dword ptr ss:\[ebp\-2C\]   
0043F60B   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F60E   .  8B80 E0010000 mov eax, dword ptr ds:\[eax+1E0\]   
0043F614   .  E8 073BFDFF   call ASPACK.00413120   
0043F619   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F61C   .  8B80 6C020000 mov eax, dword ptr ds:\[eax+26C\]   
0043F622   .  E8 E53EFDFF   call ASPACK.0041350C   
0043F627   .  B2 01         mov dl, 1   
0043F629   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F62C   .  8B80 7C020000 mov eax, dword ptr ds:\[eax+27C\]   
0043F632   .  E8 513AFDFF   call ASPACK.00413088   
0043F637   .  B2 01         mov dl, 1   
0043F639   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F63C   .  8B80 78020000 mov eax, dword ptr ds:\[eax+278\]   
0043F642   .  E8 413AFDFF   call ASPACK.00413088   
0043F647   .  B2 01         mov dl, 1   
0043F649   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F64C   .  8B80 EC020000 mov eax, dword ptr ds:\[eax+2EC\]   
0043F652   .  E8 313AFDFF   call ASPACK.00413088   
0043F657   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F65A   .  8B80 E4020000 mov eax, dword ptr ds:\[eax+2E4\]   
0043F660   .  C640 20 01    mov byte ptr ds:\[eax+20\], 1   
0043F664   .  EB 04         jmp short ASPACK.0043F66A   
0043F666   .  46            inc esi   
0043F667   .  8CDD          mov bp, ds   
0043F669   .  94            xchg eax, esp   
0043F66A   >  FF15 0C494400 call dword ptr ds:\[44490C\]   
0043F670   .  B2 01         mov dl, 1   
0043F672   .  B8 10B14200   mov eax, ASPACK.0042B110   
0043F677   .  E8 9CBBFEFF   call ASPACK.0042B218   
0043F67C   .  8945 E0       mov dword ptr ss:\[ebp\-20\], eax   
0043F67F   .  33C0          xor eax, eax   
0043F681   .  55            push ebp   
0043F682   .  68 D6F64300   push ASPACK.0043F6D6   
0043F687   .  64:FF30       push dword ptr fs:\[eax\]   
0043F68A   .  64:8920       mov dword ptr fs:\[eax\], esp   
0043F68D   .  B1 01         mov cl, 1   
0043F68F   .  BA 68F94300   mov edx, ASPACK.0043F968                 ;  ASCII "SoftwareASPackOptions"   
0043F694   .  8B45 E0       mov eax, dword ptr ss:\[ebp\-20\]   
0043F697   .  E8 54BDFEFF   call ASPACK.0042B3F0   
0043F69C   .  8D4D D4       lea ecx, dword ptr ss:\[ebp\-2C\]   
0043F69F   .  BA 88F94300   mov edx, ASPACK.0043F988                 ;  ASCII "Lang\_LanguageFile"   
0043F6A4   .  8B45 E0       mov eax, dword ptr ss:\[ebp\-20\]   
0043F6A7   .  E8 C0C0FEFF   call ASPACK.0042B76C   
0043F6AC   .  8B55 D4       mov edx, dword ptr ss:\[ebp\-2C\]   
0043F6AF   .  8B45 FC       mov eax, dword ptr ss:\[ebp\-4\]   
0043F6B2   .  8B80 E4010000 mov eax, dword ptr ds:\[eax+1E4\]   
0043F6B8   .  83C0 20       add eax, 20   
0043F6BB   .  E8 983EFCFF   call ASPACK.00403558   
0043F6C0   .  33C0          xor eax, eax   
0043F6C2   .  5A            pop edx   
0043F6C3   .  59            pop ecx   
0043F6C4   .  59            pop ecx   
0043F6C5   .  64:8910       mov dword ptr fs:\[eax\], edx   
0043F6C8   .  68 DDF64300   push ASPACK.0043F6DD   
0043F6CD   >  8B45 E0       mov eax, dword ptr ss:\[ebp\-20\]   
0043F6D0   .  E8 0735FCFF   call ASPACK.00402BDC   
0043F6D5   .  C3            retn       //到这里为止。    
  
........................................................................................   
  
Btw:  这里之所以解码是我们导入了正确的注册文件的结果，没导入注册文件时上面调用程序领空的 Call 根本不解码，只是一个简单的 Retn 返   
  
回，然后 jmp 直接跳走或运行失败。   
  
然后二进制粘贴到   
  
0043F5E2    
  
到   
  
0043F6D5   
  
之间，复制到程序里，保存重启动。   
  
呵呵，大功告成，自己的注册名，功能完全正常，无任何限制。   
  
请看  
  
【破解总结】   
  
我很忙，没时间总结了，感谢你耐心看完。   
  
  
\-------------------------------------------------------------------------------   
  
  
  
【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢! 

  
  
 4dd466d0e1e0cc123afbe504da2462ca