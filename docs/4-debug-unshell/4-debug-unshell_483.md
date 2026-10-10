

# Magic NetTrace V2.5.4 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6716.htm`](https://www.pediy.com/kssd/pediy06/pediy6716.htm)

原文链接： [`bbs.pediy.com/showthread.php?s=&threadid=5453&perpage=15&pagenumber=1`](http://bbs.pediy.com/showthread.php?s=&threadid=5453&perpage=15&pagenumber=1)  
下载页面：  [`www.skycn.com/soft/17155.html`](http://www.skycn.com/soft/17155.html)   
软件大小：  1544 KB  
软件语言：  英文  
软件类别：  国外软件 / 共享版 / 网络辅助  
应用平台：  Win9x/NT/2000/XP  
加入时间：  2004-06-30 11:11:45  
下载次数：  601  
推荐等级：  \*\*\*\*  
软件介绍：  这个程序结合 ping，WHOIS 和 traceroute 功能。  
  
【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  
  
【调试环境】：WinXP、Ollydbg、PEiD、LordPE、ImportREC  
  
—————————————————————————————————   
【脱壳过程】：  
            
           
新版 ASProtect 的脱壳教程国内还不多，脱壳后索性再整理一篇吧。  
Thanks: VolX、stephenteh、jingulong  
—————————————————————————————————  
一、壳代码解压  
               
               
设置 Ollydbg 忽略所有的异常选项。老规矩：用 IsDebug V1.4 插件去掉 Ollydbg 的调试器标志。  
  
00401000     68 01F05A00         push mTrace.005AF001  
//进入 OD 后停在这  
00401005     E8 01000000         call mTrace.0040100B  
0040100A     C3                  retn  
  
下断：BP GetModuleHandleA  
Shift+F9，中断 2 次后，就可以取消断点，Alt+F9 返回。  
直接 F4 至下面的 popad 处  
  
00B994A6     FF95 EC314400       call dword ptr ss:\[ebp+4431EC\]  
00B994AC     85C0                test eax,eax  
00B994AE     75 07               jnz short 00B994B7  
00B994B0     53                  push ebx  
00B994B1     FF95 F0314400       call dword ptr ss:\[ebp+4431F0\]  
00B994B7     8985 4D294400       mov dword ptr ss:\[ebp+44294D\],eax  
00B994BD     C785 51294400 00000>mov dword ptr ss:\[ebp+442951\],0  
00B994C7     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
00B994CD     8B06                mov eax,dword ptr ds:\[esi\]  
00B994CF     85C0                test eax,eax  
00B994D1     75 03               jnz short 00B994D6  
00B994D3     8B46 10             mov eax,dword ptr ds:\[esi+10\]  
00B994D6     03C2                add eax,edx  
00B994D8     0385 51294400       add eax,dword ptr ss:\[ebp+442951\]  
00B994DE     8B18                mov ebx,dword ptr ds:\[eax\]  
00B994E0     8B7E 10             mov edi,dword ptr ds:\[esi+10\]  
00B994E3     03FA                add edi,edx  
00B994E5     03BD 51294400       add edi,dword ptr ss:\[ebp+442951\]  
00B994EB     85DB                test ebx,ebx  
00B994ED     0F84 A2000000       je 00B99595  
00B994F3     F7C3 00000080       test ebx,80000000  
00B994F9     75 04               jnz short 00B994FF  
00B994FB     03DA                add ebx,edx  
00B994FD     43                  inc ebx  
00B994FE     43                  inc ebx  
00B994FF     53                  push ebx  
00B99500     81E3 FFFFFF7F       and ebx,7FFFFFFF  
00B99506     53                  push ebx  
00B99507     FFB5 4D294400       push dword ptr ss:\[ebp+44294D\]  
00B9950D     FF95 E8314400       call dword ptr ss:\[ebp+4431E8\]  
00B99513     85C0                test eax,eax  
00B99515     5B                  pop ebx  
00B99516     75 6F               jnz short 00B99587  
00B99518     F7C3 00000080       test ebx,80000000  
00B9951E     75 19               jnz short 00B99539  
00B99520     57                  push edi  
00B99521     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
00B99524     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B9952A     50                  push eax  
00B9952B     53                  push ebx  
00B9952C     8D85 53314400       lea eax,dword ptr ss:\[ebp+443153\]  
00B99532     50                  push eax  
00B99533     57                  push edi  
00B99534     E9 99000000         jmp 00B995D2  
00B99539     81E3 FFFFFF7F       and ebx,7FFFFFFF  
00B9953F     8B85 DC304400       mov eax,dword ptr ss:\[ebp+4430DC\]  
00B99545     3985 4D294400       cmp dword ptr ss:\[ebp+44294D\],eax  
00B9954B     75 24               jnz short 00B99571  
00B9954D     57                  push edi  
00B9954E     8BD3                mov edx,ebx  
00B99550     4A                  dec edx  
00B99551     C1E2 02             shl edx,2  
00B99554     8B9D 4D294400       mov ebx,dword ptr ss:\[ebp+44294D\]  
00B9955A     8B7B 3C             mov edi,dword ptr ds:\[ebx+3C\]  
00B9955D     8B7C3B 78           mov edi,dword ptr ds:\[ebx+edi+78\]  
00B99561     035C3B 1C           add ebx,dword ptr ds:\[ebx+edi+1C\]  
00B99565     8B0413              mov eax,dword ptr ds:\[ebx+edx\]  
00B99568     0385 4D294400       add eax,dword ptr ss:\[ebp+44294D\]  
00B9956E     5F                  pop edi  
00B9956F     EB 16               jmp short 00B99587  
00B99571     57                  push edi  
00B99572     8B46 0C             mov eax,dword ptr ds:\[esi+C\]  
00B99575     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B9957B     50                  push eax  
00B9957C     53                  push ebx  
00B9957D     8D85 A4314400       lea eax,dword ptr ss:\[ebp+4431A4\]  
00B99583     50                  push eax  
00B99584     57                  push edi  
00B99585     EB 4B               jmp short 00B995D2  
00B99587     8907                mov dword ptr ds:\[edi\],eax  
00B99589     8385 51294400 04    add dword ptr ss:\[ebp+442951\],4  
00B99590     E9 32FFFFFF         jmp 00B994C7  
00B99595     8906                mov dword ptr ds:\[esi\],eax  
00B99597     8946 0C             mov dword ptr ds:\[esi+C\],eax  
00B9959A     8946 10             mov dword ptr ds:\[esi+10\],eax  
00B9959D     83C6 14             add esi,14  
00B995A0     8B95 D8304400       mov edx,dword ptr ss:\[ebp+4430D8\]  
00B995A6     E9 EBFEFFFF         jmp 00B99496  
00B995AB     8B85 652A4400       mov eax,dword ptr ss:\[ebp+442A65\]  
00B995B1     50                  push eax  
00B995B2     0385 D8304400       add eax,dword ptr ss:\[ebp+4430D8\]  
00B995B8     5B                  pop ebx  
00B995B9     0BDB                or ebx,ebx  
00B995BB     8985 112F4400       mov dword ptr ss:\[ebp+442F11\],eax  
00B995C1     61                  popad  
//直接 F4 到这里  
00B995C2     75 08               jnz short 00B995CC  
00B995C4     B8 01000000         mov eax,1  
00B995C9     C2 0C00             retn 0C  
00B995CC     68 4C1DB900         push 0B91D4C  
00B995D1     C3                  retn  
//壳代码解压完毕  
  
  
—————————————————————————————————  
二、处理输入表，避开加密  
             
             
壳代码已经解压完毕，可以开始处理输入表了，手动 Patch，避开输入表的加密。  
             
Ctrl+S 在“整个段块”搜索命令序列：  
add esp,38  
pop ebp  
pop edi  
pop esi  
pop ebx  
retn  
找到在 00B85F20 处，下个断点吧。  
下面是分析过程。可以在搜索到地址后直接修改了，免得检验出错。  
  
00B85B0F     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85B12     8B30                mov esi,dword ptr ds:\[eax\]  
00B85B14     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85B18     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85B1B     8A00                mov al,byte ptr ds:\[eax\]  
00B85B1D     884424 07           mov byte ptr ss:\[esp+7\],al  
00B85B21     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85B24     85F6                test esi,esi  
00B85B26     75 1A               jnz short 00B85B42  
00B85B28     EB 01               jmp short 00B85B2B  
//IAT 处理完毕则自这里跳转  
00B85B42     337424 28           xor esi,dword ptr ss:\[esp+28\]  
00B85B46     0373 40             add esi,dword ptr ds:\[ebx+40\]  
00B85B49     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85B4C     8A00                mov al,byte ptr ds:\[eax\]  
00B85B4E     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85B51     33D2                xor edx,edx  
00B85B53     8AD0                mov dl,al  
00B85B55     8BC7                mov eax,edi  
00B85B57     E8 30F4FFFF         call 00B84F8C  
00B85B5C     894424 14           mov dword ptr ss:\[esp+14\],eax  
00B85B60     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85B63     8A00                mov al,byte ptr ds:\[eax\]  
00B85B65     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85B68     84C0                test al,al  
00B85B6A     75 20               jnz short 00B85B8C  
//根据 AL 值分别进行处理  
  
————————————————————————  
1、AL=1 时的处理  
  
  
00B85CA8     3C 01               cmp al,1  
//AL=1   ?  
00B85CAA     0F85 B2000000       jnz 00B85D62  
00B85CB0     EB 01               jmp short 00B85CB3  
00B85CB3     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85CB6     8B00                mov eax,dword ptr ds:\[eax\]  
00B85CB8     890424              mov dword ptr ss:\[esp\],eax  
00B85CBB     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85CBF     837B 44 00          cmp dword ptr ds:\[ebx+44\],0  
00B85CC3     74 09               je short 00B85CCE  
00B85CC5     8B0424              mov eax,dword ptr ss:\[esp\]  
00B85CC8     FF53 44             call dword ptr ds:\[ebx+44\]  
00B85CCB     890424              mov dword ptr ss:\[esp\],eax  
00B85CCE     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85CD1     66:8B00             mov ax,word ptr ds:\[eax\]  
00B85CD4     66:894424 04        mov word ptr ss:\[esp+4\],ax  
00B85CD9     8343 08 02          add dword ptr ds:\[ebx+8\],2  
00B85CDD     807C24 1C 00        cmp byte ptr ss:\[esp+1C\],0  
00B85CE2     74 0B               je short 00B85CEF  
00B85CE4     8B4424 28           mov eax,dword ptr ss:\[esp+28\]  
00B85CE8     C64424 1C 00        mov byte ptr ss:\[esp+1C\],0  
00B85CED     EB 06               jmp short 00B85CF5  
00B85CEF     8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
00B85CF3     8B00                mov eax,dword ptr ds:\[eax\]  
00B85CF5     8BC8                mov ecx,eax  
00B85CF7     66:8B5424 04        mov dx,word ptr ss:\[esp+4\]  
00B85CFC     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85CFF     E8 9CE4FFFF         call 00B841A0  
00B85D04     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B85D08     E8 37C8FEFF         call 00B72544  
00B85D0D     894424 0C           mov dword ptr ss:\[esp+C\],eax  
00B85D11     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85D14     50                  push eax  
00B85D15     8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
00B85D19     50                  push eax  
00B85D1A     A1 C026B900         mov eax,dword ptr ds:\[B926C0\]  
00B85D1F     8B00                mov eax,dword ptr ds:\[eax\]  
00B85D21     FFD0                call eax  
//GetProcAddress        ★  
00B85D23     8BE8                mov ebp,eax  
00B85D25     85ED                test ebp,ebp  
00B85D27     75 0A               jnz short 00B85D33  
00B85D29     68 405FB800         push 0B85F40    ; ASCII "11"  
00B85D2E     E8 61E1FFFF         call 00B83E94  
00B85D33     8B0424              mov eax,dword ptr ss:\[esp\]  
00B85D36     50                  push eax  
00B85D37     68 204AB800         push 0B84A20  
00B85D3C     8D4C24 20           lea ecx,dword ptr ss:\[esp+20\]  
00B85D40     8BD5                mov edx,ebp  
00B85D42     8BC3                mov eax,ebx  
00B85D44     E8 83F4FFFF         call 00B851CC  
00B85D49     8B5424 0C           mov edx,dword ptr ss:\[esp+C\]  
00B85D4D     8902                mov dword ptr ds:\[edx\],eax  
//修改①：jmp 00B99950    ★   跳到 Patch 部分  
00B85D4F     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85D53     8906                mov dword ptr ds:\[esi\],eax  
00B85D55     0FB74424 04         movzx eax,word ptr ss:\[esp+4\]  
00B85D5A     0143 08             add dword ptr ds:\[ebx+8\],eax  
00B85D5D     E9 ADFDFFFF         jmp 00B85B0F  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理① ★ ：  
            
            
在程序下面找快空地，写入 Patch 代码。00B99950 下面都是空地，就这里吧  
提前在\[B9ACC0\]处写入现在 DLL 的基址，在\[B9ACC8\]处写入准备放输入表的地址：0056A000  
  
00B99950     51                  push ecx  
00B99951     52                  push edx  
00B99952     3E:8B5424 1C        mov edx,dword ptr ds:\[esp+1C\]  
//\[esp+1C\]＝0012FF10 此处保存的是处理的 DLL 基址 ★  
00B99957     3B15 C0ACB900       cmp edx,dword ptr ds:\[B9ACC0\]  
//在\[0B1BFD0\]处提前写入现在处理的 DLL 的基址：77E40000  
00B9995D     74 0D               je short 00B9996C  
//判断与上次 DLL 是否相同 ★  
00B9995F     8915 C0ACB900       mov dword ptr ds:\[B9ACC0\],edx  
//保存 DLL 基址  
00B99965     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
//不同则地址+4 ★   
00B9996C     8B0D C8ACB900       mov ecx,dword ptr ds:\[B9ACC8\]  
//提前在\[B9ACC8\]处写入准备放输入表的地址：0056A000  
00B99972     8929                mov dword ptr ds:\[ecx\],ebp  
//EBP 保存的是正确函数的地址 ★ 正确的函数写入  
00B99974     890E                mov dword ptr ds:\[esi\],ecx  
//取代原先入壳的跳转地址  
00B99976     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
//地址 +4  
00B9997D     5A                  pop edx  
00B9997E     59                  pop ecx  
00B9997F     E9 D1C3FEFF         jmp 00B85D55  
//跳回去继续流程  
  
————————————————————————  
2、AL=2 时的处理  
  
  
00B85B8C     3C 02               cmp al,2  
//AL=2   ?  
00B85B8E     0F85 14010000       jnz 00B85CA8  
00B85B94     33C0                xor eax,eax  
00B85B96     894424 20           mov dword ptr ss:\[esp+20\],eax  
00B85B9A     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85B9D     8A00                mov al,byte ptr ds:\[eax\]  
00B85B9F     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85BA2     EB 01               jmp short 00B85BA5  
00B85BA5     2C 01               sub al,1  
00B85BA7     73 34               jnb short 00B85BDD  
00B85BA9     8BC3                mov eax,ebx  
00B85BAB     E8 84F0FFFF         call 00B84C34  
00B85BB0     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85BB3     0FB600              movzx eax,byte ptr ds:\[eax\]  
00B85BB6     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85BB9     8B53 08             mov edx,dword ptr ds:\[ebx+8\]  
00B85BBC     8B12                mov edx,dword ptr ds:\[edx\]  
00B85BBE     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85BC2     8D4C24 24           lea ecx,dword ptr ss:\[esp+24\]  
00B85BC6     51                  push ecx  
00B85BC7     8A4C24 0B           mov cl,byte ptr ss:\[esp+B\]  
00B85BCB     51                  push ecx  
00B85BCC     8BCA                mov ecx,edx  
00B85BCE     8BD3                mov edx,ebx  
00B85BD0     92                  xchg eax,edx  
00B85BD1     E8 EAF7FFFF         call 00B853C0  
00B85BD6     894424 20           mov dword ptr ss:\[esp+20\],eax  
00B85BDA     EB 01               jmp short 00B85BDD  
00B85BDD     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85BE0     8B28                mov ebp,dword ptr ds:\[eax\]  
00B85BE2     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85BE6     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B85BEA     E8 55C9FEFF         call 00B72544  
00B85BEF     894424 0C           mov dword ptr ss:\[esp+C\],eax  
00B85BF3     8BD5                mov edx,ebp  
00B85BF5     8B4424 14           mov eax,dword ptr ss:\[esp+14\]  
00B85BF9     E8 5AECFFFF         call 00B84858  
//GetProcAddress  
00B85BFE     8BE8                mov ebp,eax  
00B85C00     85ED                test ebp,ebp  
00B85C02     75 0A               jnz short 00B85C0E  
00B85C04     68 305FB800         push 0B85F30       ; ASCII "10"  
00B85C09     E8 86E2FFFF         call 00B83E94  
00B85C0E     837C24 20 00        cmp dword ptr ss:\[esp+20\],0  
00B85C13     74 44               je short 00B85C59  
00B85C15     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85C19     8B5424 20           mov edx,dword ptr ss:\[esp+20\]  
00B85C1D     8910                mov dword ptr ds:\[eax\],edx  
00B85C1F     8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00B85C23     034424 24           add eax,dword ptr ss:\[esp+24\]  
00B85C27     C600 68             mov byte ptr ds:\[eax\],68  
00B85C2A     6A 00               push 0  
00B85C2C     68 204AB800         push 0B84A20  
00B85C31     8D4C24 20           lea ecx,dword ptr ss:\[esp+20\]  
00B85C35     8BD5                mov edx,ebp  
00B85C37     8BC3                mov eax,ebx  
00B85C39     E8 8EF5FFFF         call 00B851CC  
00B85C3E     8B5424 20           mov edx,dword ptr ss:\[esp+20\]  
00B85C42     035424 24           add edx,dword ptr ss:\[esp+24\]  
00B85C46     42                  inc edx  
00B85C47     8902                mov dword ptr ds:\[edx\],eax  
00B85C49     8B4424 20           mov eax,dword ptr ss:\[esp+20\]  
00B85C4D     034424 24           add eax,dword ptr ss:\[esp+24\]  
00B85C51     83C0 05             add eax,5  
00B85C54     C600 C3             mov byte ptr ds:\[eax\],0C3  
00B85C57     EB 29               jmp short 00B85C82  
00B85C59     6A 00               push 0  
00B85C5B     68 204AB800         push 0B84A20  
00B85C60     8D4C24 20           lea ecx,dword ptr ss:\[esp+20\]  
00B85C64     8BD5                mov edx,ebp  
00B85C66     8BC3                mov eax,ebx  
00B85C68     E8 5FF5FFFF         call 00B851CC  
00B85C6D     8B5424 0C           mov edx,dword ptr ss:\[esp+C\]  
00B85C71     8902                mov dword ptr ds:\[edx\],eax  
00B85C73     8D4C24 0C           lea ecx,dword ptr ss:\[esp+C\]  
00B85C77     8A5424 07           mov dl,byte ptr ss:\[esp+7\]  
00B85C7B     8BC3                mov eax,ebx  
00B85C7D     E8 BEF7FFFF         call 00B85440  
00B85C82     8BC6                mov eax,esi  
00B85C84     83E8 02             sub eax,2  
00B85C87     66:8338 00          cmp word ptr ds:\[eax\],0  
00B85C8B     75 10               jnz short 00B85C9D  
//下面的 00B85C9D 分支没有用  
00B85C8D     8B5424 0C           mov edx,dword ptr ss:\[esp+C\]  
00B85C91     8B12                mov edx,dword ptr ds:\[edx\]  
00B85C93     E8 4CDEFFFF         call 00B83AE4  
//加密 CALL   进入修改 ★  
00B85C98     E9 72FEFFFF         jmp 00B85B0  
//循环  
00B85C9D     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85CA1     8906                mov dword ptr ds:\[esi\],eax  
00B85CA3     E9 67FEFFFF         jmp 00B85B0F  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理② ★ ：  
  
  
00B83AE4     53                  push ebx  
00B83AE5     8BD8                mov ebx,eax  
00B83AE7     8BC3                mov eax,ebx  
00B83AE9     E8 56FFFFFF         call 00B83A44  
00B83AEE     C603 E8             mov byte ptr ds:\[ebx\],0E8  
//修改②：jmp 00B99988  ★   跳到 Patch 部分  
00B83AF1     43                  inc ebx  
00B83AF2     8903                mov dword ptr ds:\[ebx\],eax  
00B83AF4     5B                  pop ebx  
00B83AF5     C3                  retn  
  
00B99988     51                  push ecx  
00B99989     52                  push edx  
00B9998A     3E:8B5424 24        mov edx,dword ptr ds:\[esp+24\]  
00B9998F     3B15 C0ACB900       cmp edx,dword ptr ds:\[B9ACC0\]  
00B99995     74 0D               je short 00B999A4  
00B99997     8915 C0ACB900       mov dword ptr ds:\[B9ACC0\],edx  
00B9999D     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B999A4     8B0D C8ACB900       mov ecx,dword ptr ds:\[B9ACC8\]  
00B999AA     8929                mov dword ptr ds:\[ecx\],ebp  
00B999AC     C703 FF250000       mov dword ptr ds:\[ebx\],25FF  
00B999B2     890E                mov dword ptr ds:\[esi\],ecx  
00B999B4     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B999BB     5A                  pop edx  
00B999BC     59                  pop ecx  
00B999BD     E9 32A1FEFF         jmp 00B83AF4  
  
  
————————————————————————  
3、AL=3 时的处理  
            
            
这部分对于 ASProtect.exe 只处理 1 个特殊函数：GetProcAddress  
有 3 个分支，第一个是 GetProcAddress，第三个是错误提示。你可以到 00B04308 看看就明白了  
  
00B85E5E     3C 03               cmp al,3  
//AL=3   ?  
00B85E60     0F85 A5000000       jnz 00B85F0B  
00B85E66     EB 01               jmp short 00B85E69  
  
00B85E69     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85E6C     8A00                mov al,byte ptr ds:\[eax\]  
00B85E6E     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85E71     FEC8                dec al  
00B85E73     74 09               je short 00B85E7E  
00B85E75     FEC8                dec al  
00B85E77     74 41               je short 00B85EBA  
//下面的 00B85EBA 分支没有用  
00B85E79     E9 91FCFFFF         jmp 00B85B0F  
00B85E7E     8BC3                mov eax,ebx  
00B85E80     E8 AFEDFFFF         call 00B84C34  
00B85E85     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85E88     8A00                mov al,byte ptr ds:\[eax\]  
00B85E8A     884424 06           mov byte ptr ss:\[esp+6\],al  
00B85E8E     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85E91     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B85E95     E8 AAC6FEFF         call 00B72544  
00B85E9A     894424 0C           mov dword ptr ss:\[esp+C\],eax  
//修改③：jmp 00B999C8   ★   跳到 Patch 部分  
00B85E9E     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85EA2     8906                mov dword ptr ds:\[esi\],eax  
00B85EA4     8A4C24 06           mov cl,byte ptr ss:\[esp+6\]  
00B85EA8     8B5424 14           mov edx,dword ptr ss:\[esp+14\]  
00B85EAC     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85EB0     E8 FFF0FFFF         call 00B84FB4  
00B85EB5     E9 55FCFFFF         jmp 00B85B0F  
//循环  
00B85EBA     8BC3                mov eax,ebx  
//一直没碰到这个分支的使用，不知放了什么狠招  
00B85EBC     E8 73EDFFFF         call 00B84C34  
00B85EC1     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85EC4     8B28                mov ebp,dword ptr ds:\[eax\]  
00B85EC6     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85ECA     8D4424 08           lea eax,dword ptr ss:\[esp+8\]  
00B85ECE     8BD5                mov edx,ebp  
00B85ED0     E8 4FE3FFFF         call 00B84224  
00B85ED5     8BCD                mov ecx,ebp  
00B85ED7     8B53 08             mov edx,dword ptr ds:\[ebx+8\]  
00B85EDA     8B4424 08           mov eax,dword ptr ss:\[esp+8\]  
00B85EDE     E8 4DF9FEFF         call 00B75830  
00B85EE3     016B 08             add dword ptr ds:\[ebx+8\],ebp  
00B85EE6     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B85EEA     E8 55C6FEFF         call 00B72544  
00B85EEF     894424 0C           mov dword ptr ss:\[esp+C\],eax  
00B85EF3     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85EF7     8B5424 08           mov edx,dword ptr ss:\[esp+8\]  
00B85EFB     8910                mov dword ptr ds:\[eax\],edx  
00B85EFD     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85F01     8906                mov dword ptr ds:\[esi\],eax  
00B85F03     E9 07FCFFFF         jmp 00B85B0F  
00B85F0B     68 705FB800         push 0B85F70     ; ASCII "89"  
00B85F10     E8 7FDFFFFF         call 00B83E94  
//Protection Error  
00B85F15     E9 F5FBFFFF         jmp 00B85B0F  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理③ ★ ：  
  
  
00B999C8     3E:8B4424 14        mov eax,dword ptr ds:\[esp+14\]  
00B999CD     3B05 C0ACB900       cmp eax,dword ptr ds:\[B9ACC0\]  
00B999D3     74 0D               je short 00B999E2  
00B999D5     A3 C0ACB900         mov dword ptr ds:\[B9ACC0\],eax  
00B999DA     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B999E1     A1 C8ACB900         mov eax,dword ptr ds:\[B9ACC8\]  
//放入我们程序中的地址  
00B999E6     894424 0C           mov dword ptr ss:\[esp+C\],eax  
//00B85E9A 处代码挪到这里执行  
00B999EA     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B999EE     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B999F5     E9 A8C4FEFF         jmp 00B85EA2  
  
  
————————————————————————  
4、AL=4 时的处理  
  
  
00B85D62     3C 04               cmp al,4  
//AL=4   ?  
00B85D64     0F85 F4000000       jnz 00B85E5E  
00B85D6A     EB 01               jmp short 00B85D6D  
00B85D6D     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85D70     8A00                mov al,byte ptr ds:\[eax\]  
00B85D72     FF43 08             inc dword ptr ds:\[ebx+8\]  
00B85D75     84C0                test al,al  
00B85D77     75 5B               jnz short 00B85DD4  
00B85D79     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85D7C     8B30                mov esi,dword ptr ds:\[eax\]  
00B85D7E     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85D82     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85D85     8B28                mov ebp,dword ptr ds:\[eax\]  
00B85D87     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85D8B     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85D8E     8B00                mov eax,dword ptr ds:\[eax\]  
00B85D90     894424 2C           mov dword ptr ss:\[esp+2C\],eax  
00B85D94     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85D98     837B 30 00          cmp dword ptr ds:\[ebx+30\],0  
00B85D9C     75 0A               jnz short 00B85DA8  
00B85D9E     68 505FB800         push 0B85F50     ; ASCII "81"  
00B85DA3     E8 ECE0FFFF         call 00B83E94  
00B85DA8     8D5424 30           lea edx,dword ptr ss:\[esp+30\]  
00B85DAC     8BC3                mov eax,ebx  
00B85DAE     E8 51F8FFFF         call 00B85604  
00B85DB3     8B4424 2C           mov eax,dword ptr ss:\[esp+2C\]  
00B85DB7     50                  push eax  
00B85DB8     8D4424 34           lea eax,dword ptr ss:\[esp+34\]  
00B85DBC     50                  push eax  
00B85DBD     55                  push ebp  
00B85DBE     A1 7C25B900         mov eax,dword ptr ds:\[B9257C\]  
00B85DC3     8B00                mov eax,dword ptr ds:\[eax\]  
00B85DC5     50                  push eax  
00B85DC6     56                  push esi  
00B85DC7     8B4424 28           mov eax,dword ptr ss:\[esp+28\]  
00B85DCB     50                  push eax  
00B85DCC     FF53 30             call dword ptr ds:\[ebx+30\]  
00B85DCF     E9 3BFDFFFF         jmp 00B85B0F  
//修改④：jmp 00B99A36   ★   跳到 Patch 部分  
00B85DD4     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85DD7     8B00                mov eax,dword ptr ds:\[eax\]  
00B85DD9     890424              mov dword ptr ss:\[esp\],eax  
00B85DDC     8343 08 04          add dword ptr ds:\[ebx+8\],4  
00B85DE0     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85DE3     66:8B00             mov ax,word ptr ds:\[eax\]  
00B85DE6     66:894424 04        mov word ptr ss:\[esp+4\],ax  
00B85DEB     8343 08 02          add dword ptr ds:\[ebx+8\],2  
00B85DEF     8B0C24              mov ecx,dword ptr ss:\[esp\]  
00B85DF2     66:8B5424 04        mov dx,word ptr ss:\[esp+4\]  
00B85DF7     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85DFA     E8 A1E3FFFF         call 00B841A0  
00B85DFF     8B4424 10           mov eax,dword ptr ss:\[esp+10\]  
00B85E03     E8 3CC7FEFF         call 00B72544  
00B85E08     894424 0C           mov dword ptr ss:\[esp+C\],eax  
00B85E0C     8B43 08             mov eax,dword ptr ds:\[ebx+8\]  
00B85E0F     50                  push eax  
00B85E10     8B4424 18           mov eax,dword ptr ss:\[esp+18\]  
00B85E14     50                  push eax  
00B85E15     A1 C026B900         mov eax,dword ptr ds:\[B926C0\]  
00B85E1A     8B00                mov eax,dword ptr ds:\[eax\]  
00B85E1C     FFD0                call eax  
00B85E1E     8BE8                mov ebp,eax  
00B85E20     85ED                test ebp,ebp  
00B85E22     75 0C               jnz short 00B85E30  
00B85E24     68 605FB800         push 0B85F60    ; ASCII "250"  
00B85E29     E8 66E0FFFF         call 00B83E94  
00B85E2E     EB 15               jmp short 00B85E45  
00B85E30     A1 F425B900         mov eax,dword ptr ds:\[B925F4\]  
00B85E35     3B28                cmp ebp,dword ptr ds:\[eax\]  
00B85E37     75 0C               jnz short 00B85E45  
00B85E39     837B 34 00          cmp dword ptr ds:\[ebx+34\],0  
00B85E3D     74 06               je short 00B85E45  
00B85E3F     8B6B 34             mov ebp,dword ptr ds:\[ebx+34\]  
00B85E42     036B 40             add ebp,dword ptr ds:\[ebx+40\]  
00B85E45     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
00B85E49     8928                mov dword ptr ds:\[eax\],ebp  
00B85E4B     8B4424 0C           mov eax,dword ptr ss:\[esp+C\]  
//修改⑤：jmp 00B999FE   ★   跳到 Patch 部分  
00B85E4F     8906                mov dword ptr ds:\[esi\],eax  
00B85E51     0FB74424 04         movzx eax,word ptr ss:\[esp+4\]  
00B85E56     0143 08             add dword ptr ds:\[ebx+8\],eax  
00B85E59     E9 B1FCFFFF         jmp 00B85B0F  
  
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理④ ★ ：  
  
  
00B99A36     51                  push ecx  
00B99A37     52                  push edx  
00B99A38     3E:8B5424 1C        mov edx,dword ptr ds:\[esp+1C\]  
00B99A3D     3B15 C0ACB900       cmp edx,dword ptr ds:\[B9ACC0\]  
00B99A43     74 0D               je short 00B99A52  
00B99A45     8915 C0ACB900       mov dword ptr ds:\[B9ACC0\],edx  
00B99A4B     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B99A52     8B0D C8ACB900       mov ecx,dword ptr ds:\[B9ACC8\]  
00B99A58     3E:8B5424 E4        mov edx,dword ptr ds:\[esp\-1C\]  
00B99A5D     8911                mov dword ptr ds:\[ecx\],edx  
00B99A5F     3E:8B5424 C0        mov edx,dword ptr ds:\[esp\-40\]  
//用 VolX 兄的方法，呵呵  
00B99A64     3E:035424 C4        add edx,dword ptr ds:\[esp\-3C\]  
//\[esp-40\]和\[esp-3C\]要看你本机的代码来确定  ★  
00B99A69     81C2 00004000       add edx,400000  
00B99A6F     890A                mov dword ptr ds:\[edx\],ecx  
00B99A71     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B99A78     5A                  pop edx  
00B99A79     59                  pop ecx  
00B99A7A     E9 90C0FEFF         jmp 00B85B0F  
  
            
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  
★ Patch 处理⑤ ★ ：  
            
          
00B999FE     51                  push ecx  
00B999FF     52                  push edx  
00B99A00     3E:8B5424 1C        mov edx,dword ptr ds:\[esp+1C\]  
00B99A05     3B15 C0ACB900       cmp edx,dword ptr ds:\[B9ACC0\]  
00B99A0B     74 0D               je short 00B99A1A  
00B99A0D     8915 C0ACB900       mov dword ptr ds:\[B9ACC0\],edx  
00B99A13     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B99A1A     8B0D C8ACB900       mov ecx,dword ptr ds:\[B9ACC8\]  
00B99A20     8929                mov dword ptr ds:\[ecx\],ebp  
00B99A22     890E                mov dword ptr ds:\[esi\],ecx  
00B99A24     8305 C8ACB900 04    add dword ptr ds:\[B9ACC8\],4  
00B99A2B     5A                  pop edx  
00B99A2C     59                  pop ecx  
00B99A2D     E9 1FC4FEFF         jmp 00B85E51  
  
  
————————————————————————  
修改完以上部分后在 00B85F27 处下断，Shift+F9 中断  
  
  
00B85F20     83C4 38             add esp,38  
00B85F23     5D                  pop ebp  
00B85F24     5F                  pop edi  
00B85F25     5E                  pop esi  
00B85F26     5B                  pop ebx  
00B85F27     C3                  retn  
//中断在这里后 IAT 处理完毕！  
  
  
—————————————————————————————————  
三、附注：看看 AL=3 的 00B85E7E 处三个分支处理  
  
  
1、0B846B8 是 GetProcAddress  
  
00B84FB4     53                  push ebx  
00B84FB5     56                  push esi  
00B84FB6     8BF2                mov esi,edx  
00B84FB8     8BD8                mov ebx,eax  
00B84FBA     FEC9                dec cl  
00B84FBC     74 06               je short 00B84FC4  
00B84FBE     FEC9                dec cl  
00B84FC0     74 0C               je short 00B84FCE  
00B84FC2     EB 2B               jmp short 00B84FEF  
00B84FC4     B8 B846B800         mov eax,0B846B8  
00B84FC9     8903                mov dword ptr ds:\[ebx\],eax  
00B84FCB     5E                  pop esi  
00B84FCC     5B                  pop ebx  
00B84FCD     C3                  retn  
  
00B846B8     55                  push ebp  
00B846B9     8BEC                mov ebp,esp  
00B846BB     8B55 0C             mov edx,dword ptr ss:\[ebp+C\]  
00B846BE     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
00B846C1     3B05 F823B900       cmp eax,dword ptr ds:\[B923F8\]  
00B846C7     75 09               jnz short 00B846D2  
00B846C9     8B0495 F823B900     mov eax,dword ptr ds:\[edx\*4+B923F8>  
00B846D0     EB 07               jmp short 00B846D9  
00B846D2     52                  push edx  
00B846D3     50                  push eax  
00B846D4     E8 2710FFFF         call 00B75700 ; jmp to kernel32.GetProcAddress  
00B846D9     5D                  pop ebp  
00B846DA     C2 0800             retn 8  
  
————————————————————————  
2、这个小分支似乎是为 VB 程序准备的  
  
00B84FCE     68 FC4FB800         push 0B84FFC  ; ASCII "DllFunctionCall"  
00B84FD3     56                  push esi  
00B84FD4     A1 C026B900         mov eax,dword ptr ds:\[B926C0\]  
00B84FD9     8B00                mov eax,dword ptr ds:\[eax\]  
00B84FDB     FFD0                call eax  
00B84FDD     8B15 9426B900       mov edx,dword ptr ds:\[B92694\]  
00B84FE3     8902                mov dword ptr ds:\[edx\],eax  
00B84FE5     B8 E046B800         mov eax,0B846E0  
00B84FEA     8903                mov dword ptr ds:\[ebx\],eax  
00B84FEC     5E                  pop esi  
00B84FED     5B                  pop ebx  
00B84FEE     C3                  retn  
  
————————————————————————  
3、0B84F98 就 OVER 了：Protection Error  
  
00B84FEF     B8 984FB800         mov eax,0B84F98   
00B84FF4     8903                mov dword ptr ds:\[ebx\],eax  
00B84FF6     5E                  pop esi  
00B84FF7     5B                  pop ebx  
00B84FF8     C3                  retn  
  
00B84F98     68 AC4FB800         push 0B84FAC  ; ASCII "87"  
00B84F9D     E8 F2EEFFFF         call 00B83E94  
00B84FA2     C3                  retn  
  
00B83E94     55                  push ebp  
00B83E95     8BEC                mov ebp,esp  
00B83E97     6A 00               push 0  
00B83E99     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
00B83E9C     E8 07FBFEFF         call 00B739A8  
00B83EA1     33C0                xor eax,eax  
00B83EA3     55                  push ebp  
00B83EA4     68 F03EB800         push 0B83EF0  
00B83EA9     64:FF30             push dword ptr fs:\[eax\]  
00B83EAC     64:8920             mov dword ptr fs:\[eax\],esp  
00B83EAF     8D45 FC             lea eax,dword ptr ss:\[ebp\-4\]  
00B83EB2     8B4D 08             mov ecx,dword ptr ss:\[ebp+8\]  
00B83EB5     BA 043FB800         mov edx,0B83F04  ; ASCII "Error: "  
00B83EBA     E8 81F9FEFF         call 00B73840  
00B83EBF     8B45 FC             mov eax,dword ptr ss:\[ebp\-4\]  
00B83EC2     E8 F1FAFEFF         call 00B739B8  
00B83EC7     50                  push eax  
00B83EC8     68 0C3FB800         push 0B83F0C     ; ASCII "Protection Error"  
00B83ECD     E8 4E000000         call 00B83F20  
//Game Over  
00B83ED2     33C0                xor eax,eax  
00B83ED4     5A                  pop edx  
00B83ED5     59                  pop ecx  
00B83ED6     59                  pop ecx  
00B83ED7     64:8910             mov dword ptr fs:\[eax\],edx  
00B83EDA     68 F73EB800         push 0B83EF7  
00B83EDF     8D45 FC             lea eax,dword ptr ss:\[ebp\-4\]  
00B83EE2     E8 B1F6FEFF         call 00B73598  
00B83EE7     8D45 08             lea eax,dword ptr ss:\[ebp+8\]  
00B83EEA     E8 A9F6FEFF         call 00B73598  
00B83EEF     C3                  retn  
  
  
—————————————————————————————————  
四、Pre-Dip：用自己的名字注册  
  
  
取消以前的所有断点，现在设置 Ollydbg 忽略除了“内存访问异常”之外的所有其他异常选项。  
Shift+F9 运行，注意看堆栈，当第 2 次在堆栈中看见“8wh3JAAQjOI=”硬盘指纹时，可以处理了。  
  
00B90B36     893B                mov dword ptr ds:\[ebx\],edi  
//异常，第 2 次看见硬盘指纹  
  
0012FF20   0012FF2C   指针到下一个 SEH 记录  
0012FF24   00B90CEF   SE 句柄  
0012FF28   40E2AEE6  
0012FF2C   0012FF80   指针到下一个 SEH 记录  
0012FF30   00B9114C   SE 句柄  
0012FF34   0012FF78  
0012FF38   00B70000  
0012FF3C   00B40000   ASCII "MZ?  
0012FF40   00B9051C   ASCII "U 嬱 SVW′&?  
0012FF44   00000000  
0012FF48   00BAE59C   ASCII "8wh3JAAQjOI="//硬盘指纹  
0012FF4C   00B8F0F6   返回到 00B8F0F6 来自 00B81868  
  
Alt+M 打开内存查看窗口，在 00401000 段下 内存访问断点，Shift+F9 通过异常，断下  
  
00535838     55                  push ebp  
//中断在这里  
00535839     8BEC                mov ebp,esp  
0053583B     8B45 08             mov eax,dword ptr ss:\[ebp+8\]  
0053583E     A3 0CCE5400         mov dword ptr ds:\[54CE0C\],eax  
//保存注册名地址  
00535843     5D                  pop ebp  
00535844     C2 0400             retn 4  
  
过了 0053583E 之后，\[54CE0C\]处应该是保存的注册用户名，找一段空地，在 0056AB00 处写入：fly  \[CUG\]，然后把\[54CE0C\]处的值改为 0056AB00。OK，这个 Pre-Dip 处理完毕。  
  
  
—————————————————————————————————  
五、以壳解壳——ASProtect V1.3 壳的 Stolen Code 简便解决方案  
  
  
取消以前的所有断点。继续 Shift+F9 运行，直至中断在最后一次典型异常。  
  
00B8F3F3     893B                mov dword ptr ds:\[ebx\],edi  
//ASProtect V1.31 壳的最后一次异常  
00B8F3F5     3085 5798C864       xor byte ptr ss:\[ebp+64C89857\],al  
00B8F3FB     1B3F                sbb edi,dword ptr ds:\[edi\]  
00B8F3FD     05 7B462835         add eax,3528467B  
00B8F402     67:64:8F06 0000     pop dword ptr fs:\[0\]  
00B8F408     26:EB 02            jmp short 00B8F40D  
  
提供一个简便的方法快速达到使用“以壳解壳”的时机！  
在最后一次异常时，Ctrl+F 在“当前位置下”搜索命令：  
mov esp,dword ptr ss:\[ebp\-8\]  
OK，找到在 00B8F65E 处！下断，Shift+F9 运行，中断  
  
00B8F64A     E8 8B45FC8B         call 8CB53BDA  
00B8F64F     0085 C0752B8B       add byte ptr ss:\[ebp+8B2B75C0\],al  
00B8F655     0D 904BB900         or eax,0B94B90  
00B8F65A     85C9                test ecx,ecx  
00B8F65C     74 0B               je short 00B8F669  
00B8F65E     8B65 F8             mov esp,dword ptr ss:\[ebp\-8\]  
//中断后取消所有断点  
00B8F661     FF35 904BB900       push dword ptr ds:\[B94B90\]  
00B8F667     C3                  retn  
//返回 00DE0303  
  
  
00DE0303     55                  push ebp  
//这里开始处理 Stolen Code ★  
//在这儿用 PETools 完全 DUMP 这个进程  
00DE0304     EB 02               jmp short 00DE0308  
00DE0308     8D6C5E 4A           lea ebp,dword ptr ds:\[esi+ebx\*2+4A\]  
00DE0310     8D6C05 B6           lea ebp,dword ptr ss:\[ebp+eax\-4A\]  
00DE0318     2BE8                sub ebp,eax  
00DE031A     2BEB                sub ebp,ebx  
  
  
—————————————————————————————————  
六、Dump、修复输入表  
  
  
如果仅仅脱壳出上面的壳代码部分是不行的，如下面的访问地址就丢失了：  
02720167     FFD7                call edi  ;00B8BC98  
00409C6C     68 10D6BA00         push 0BAD610  
00409C71     C3                  retn  
00DE1C99     E8 62E39301         call 02720000  
00B8BC53     FF60 24             jmp dword ptr ds:\[eax+24\]; 02730000  
  
所以区域脱壳出以下部分壳代码：  
00B70000-00B9B000.dmp  
00BA8000-00BB0000.dmp  
00DE0000-00DE4000.dmp  
02720000-02721000.dmp  
02730000-02731000.dmp  
  
“组装”一下 dumped.exe，从磁盘载入刚才部分脱壳的五部分。  
“组装”一下 dumped.exe，从磁盘载入刚才部分脱壳的 3 部分。  
减去基址后分别修改其虚拟地址为 00770000、007A8000、009E0000、02320000、02330000  
只保留 PETools 的“验证 PE 文件”选项，重建 PE。  
  
运行 ImportREC，填入 RVA=0016A000、Size=00000850  
获取输入表，会发现有 2 个无效指针  
0016A078  ?  0000  00B846B8  
0016A1E0  ?  0000  00B846B8  
通过 AL=3 的分析，我们知道应该把这 2 个函数修复为：GetProcAddress  
修改 OEP=009E0303，FixDump！  
  
  
—————————————————————————————————  
七、暗桩修复  
  
  
运行后会发现有异常，其实还是壳造成的问题。  
————————————————————————  
1、004021FA 处异常  
  
00402184     55                  push ebp  
00402185     8BEC                mov ebp,esp  
00402187     83C4 F8             add esp,-8  
0040218A     53                  push ebx  
0040218B     56                  push esi  
0040218C     57                  push edi  
0040218D     8BD8                mov ebx,eax  
0040218F     803D 30E45400 00    cmp byte ptr ds:\[54E430\],0  
//把 54E430 处的 01 修改为 00，让其重新申请内存  
00402196     75 13               jnz short dumped\_.004021AB  
00402198     E8 13F9FFFF         call dumped\_.00401AB0  
0040219D     84C0                test al,al  
0040219F     75 0A               jnz short dumped\_.004021AB  
004021A1     33C0                xor eax,eax  
004021A3     8945 FC             mov dword ptr ss:\[ebp\-4\],eax  
004021A6     E9 54010000         jmp dumped\_.004022FF  
004021AB     33C9                xor ecx,ecx  
004021AD     55                  push ebp  
004021AE     68 F8224000         push dumped\_.004022F8  
004021B3     64:FF31             push dword ptr fs:\[ecx\]  
004021B6     64:8921             mov dword ptr fs:\[ecx\],esp  
004021B9     803D 49E05400 00    cmp byte ptr ds:\[54E049\],0  
004021C0     74 0A               je short dumped\_.004021CC  
004021C2     68 38E45400         push dumped\_.0054E438  
004021C7     E8 40F2FFFF         call <jmp.&kernel32.EnterCriticalSection>  
004021CC     83C3 07             add ebx,7  
004021CF     83E3 FC             and ebx,FFFFFFFC  
004021D2     83FB 0C             cmp ebx,0C  
004021D5     7D 05               jge short dumped\_.004021DC  
004021D7     BB 0C000000         mov ebx,0C  
004021DC     81FB 00100000       cmp ebx,1000  
004021E2     0F8F 93000000       jg dumped\_.0040227B  
004021E8     8BC3                mov eax,ebx  
004021EA     85C0                test eax,eax  
004021EC     79 03               jns short dumped\_.004021F1  
004021EE     83C0 03             add eax,3  
004021F1     C1F8 02             sar eax,2  
004021F4     8B15 90E45400       mov edx,dword ptr ds:\[54E490\]  
004021FA     8B5482 F4           mov edx,dword ptr ds:\[edx+eax\*4-C\]  
//异常  
004021FE     85D2                test edx,edx  
00402200     74 79               je short dumped\_.0040227B  
  
————————————————————————  
2、0040146C 处异常  
  
00401426     BE 54E45400         mov esi,dumped\_.0054E454  
0040142B     833E 00             cmp dword ptr ds:\[esi\],0  
//把\[0054E454\]处的 00153FF4 修改为 00  ★  
0040142E     75 3A               jnz short dumped\_.0040146A  
00401430     68 44060000         push 644  
00401435     6A 00               push 0  
00401437     E8 A8FFFFFF         call <jmp.&kernel32.LocalAlloc>  
0040143C     8BC8                mov ecx,eax  
0040143E     85C9                test ecx,ecx  
00401440     75 05               jnz short dumped\_.00401447  
00401442     33C0                xor eax,eax  
00401444     5E                  pop esi  
00401445     5B                  pop ebx  
00401446     C3                  retn  
00401447     A1 50E45400         mov eax,dword ptr ds:\[54E450\]  
0040144C     8901                mov dword ptr ds:\[ecx\],eax  
0040144E     890D 50E45400       mov dword ptr ds:\[54E450\],ecx  
00401454     33D2                xor edx,edx  
00401456     8BC2                mov eax,edx  
00401458     03C0                add eax,eax  
0040145A     8D44C1 04           lea eax,dword ptr ds:\[ecx+eax\*8+4\]  
0040145E     8B1E                mov ebx,dword ptr ds:\[esi\]  
00401460     8918                mov dword ptr ds:\[eax\],ebx  
00401462     8906                mov dword ptr ds:\[esi\],eax  
00401464     42                  inc edx  
00401465     83FA 64             cmp edx,64  
00401468     75 EC               jnz short dumped\_.00401456  
0040146A     8B06                mov eax,dword ptr ds:\[esi\]  
0040146C     8B10                mov edx,dword ptr ds:\[eax\]  
//异常  
0040146E     8916                mov dword ptr ds:\[esi\],edx  
00401470     5E                  pop esi  
00401471     5B                  pop ebx  
00401472     C3                  retn  
  
————————————————————————  
3、退出时在 00401C08 处异常  
  
00401BFF     A1 50E45400         mov eax,dword ptr ds:\[54E450\]  
//把\[54E450\]处的数据修改为 00000000  
00401C04     85C0                test eax,eax  
00401C06     74 17               je short dumped\_.00401C1F  
00401C08     8B10                mov edx,dword ptr ds:\[eax\]  
//异常  
00401C0A     8915 50E45400       mov dword ptr ds:\[54E450\],edx  
00401C10     50                  push eax  
00401C11     E8 D6F7FFFF         call <jmp.&kernel32.LocalFree>  
  
OK，正常运行了，脱壳成功！  
  
  
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
               2004-10-03 19:00

 4dd466d0e1e0cc123afbe504da2462ca