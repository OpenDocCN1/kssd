

# 饶过瑞星 2006 网络升级验证

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-4.htm`](https://www.pediy.com/kssd/pediy09/pediy09-4.htm)

> 饶过瑞星网络升级验证  
> 其中考试结束了有点无聊，于是便研究下了瑞星 2006 的升级系统.结果研究出一些  
>   
> 心得和在这里和大家分享.  
> 抓取网络封包并分析  
> 前提是你有正版的 KEY，用做比较.打开防火墙把 SmartUp.exe 这个规则删除(后面  
>   
> 有用的).好现在使用的是正版 KEY，点击升级,防火墙提示访问网络，现在打开  
>   
> WINSockExpert 选择程 Smartup 监听数据.防火墙则选允许访问网络.  
> 看看我们截取到的数据.  
> GET /register/pcver/autoupgradepad/ver2006/NewVer.asp?tag=&exp=0   
>   
> HTTP/1.1                                     ;验证开始  
> Accept: \*/\*  
> User-Agent: Mozilla/4.0 (compatible; MSIE 5.00; Windows 98; Rising)  
> Host: update.rising.com.cn  
> Connection: Keep-Alive  
> GET /register/PcVer/AutoUpgradePad/ver2006/PcVerLayerRequest.asp?  
>   
> Product=278921232132&Ver=18.51.42 HTTP/1.1  
> Accept: \*/\*  
> User-Agent: Mozilla/4.0 (compatible; MSIE 5.00; Windows 98; Rising)  
> Host: update.rising.com.cn  
> Connection: Keep-Alive  
> Cookie: ASPSESSIONIDAQBARTQT=JOGJHFLDIKLFGBMNOOMCHFDA  
> <head><title>Object moved</title></head>  
> <body><h1>Object Moved</h1>This object may be found <a   
>   
> HREF="http://219.238.233.223/register/pcver/autoupgradePad/ver2006/PcVerR  
>   
> equestUpgrade.asp">here</a>.</body>  
>   
> GET /register/pcver/autoupgradePad/ver2006/PcVerRequestUpgrade.asp?  
>   
> Ver=18.51.42&Info=C8zxN3MDAF21321321321321321321GwgODAodaRUaGV  
>   
> IQfVZbUAUcfVNRT2FMIwgHCENIclJ32133123213  
> User-Agent: Mozilla/4.0 (compatible; MSIE 5.00; Windows 98; Rising)  
> Host: 219.238.233.223  
> Connection: Keep-Alive  
>   
> GET /register/pcver/autoupgradePad/ver2006/PcVerRequestUpgrade.asp?  
>   
> Ver=18.51.42                                ;到这里已经通过验证拉  
>   
> &Info=C8zx1321321321YaRI213213213MiPxpuHVcuIHkABVcxUGQeYlkvL32132  
>   
> 13Kj4sH1JfGwgODAodaRUaGVI213213MIwgHCENIclJSXg4asw== HTTP/1.1  
> Accept: \*/\*  
> User-Agent: Mozilla/4.0 (compatible; MSIE 5.00; Windows 98; Rising)  
> Host: 219.238.233.223  
> Connection: Keep-Alive  
> 一些不重要的信息省略.  
> 最后抓到的升级文件信息，到这里我们抓包已经结束拉.  
> http://download.rising.com.cn/re ... pad/pcver2006new/Co  
>   
> mpsVer18.53.42.inf  
> 调试分析升级程序  
> 我们知道瑞星的升级程序是 Smartup,用 OD 载入，右键分析找找关键信息.  
> 004115A3    E8 C4060100     call <jmp.&MFC42.#540>  
> 004115A8    8B55 00         mov edx,dword ptr ss:\[ebp\]  
> 004115AB    68 10334300     push SmartUp.00433310           ; ASCII                    
>   
> ;"CompsVer.inf"         取得本地路径  
> 004115B0    52              push edx  
> 004115B1    8D4424 18       lea eax,dword ptr ss:\[esp+18\]  
> 004115B5    68 70324300     push SmartUp.00433270           ; ASCII "%s\\%s"  
> 004115BA    50              push eax  
> 004115BB    C74424 44 00000>mov dword ptr ss:\[esp+44\],0  
> 004115C3    E8 1C070100     call <jmp.&MFC42.#2818>  
> 004115C8    8B4C24 20       mov ecx,dword ptr ss:\[esp+20\]  
> 004115CC    83C4 10         add esp,10  
> 004115CF    8DBE 84070000   lea edi,dword ptr ds:\[esi+784\]  
> 004115D5    51              push ecx  
> 004115D6    6A 20           push 20  
> 004115D8    6A 20           push 20  
> 004115DA    8BCF            mov ecx,edi  
> 004115DC    E8 3F070100     call <jmp.&MFC42.#2915>  
> 004115E1    50              push eax  
> 004115E2    68 F8324300     push SmartUp.004332F8           ; ASCII "18.00"  
> 004115E7    68 F0324300     push SmartUp.004332F0           ; ASCII "Version"  
> 004115EC    68 E0414300     push SmartUp.004341E0           ; ASCII "Update"  
> 004115F1    FF15 ECC04200   call dword ptr ds:\[<&KERNEL32.G>;   
>   
> kernel32.GetPrivateProfileStringA           ;取得本地升级版本号,下面验证是不是最  
>   
> 新版本  
> 004115F7    6A FF           push -1  
> 004115F9    8BCF            mov ecx,edi  
> 004115FB    E8 1A070100     call <jmp.&MFC42.#5572>  
> 00411600    8D4C24 18       lea ecx,dword ptr ss:\[esp+18\]  
> 00411604    E8 63060100     call <jmp.&MFC42.#540>  
> 00411609    8B13            mov edx,dword ptr ds:\[ebx\]                                    
> 0041160B    68 10334300     push SmartUp.00433310           ; ASCII   
>   
> "CompsVer.inf"  
> 00411610    52              push edx  
> 00411611    8D4424 18       lea eax,dword ptr ss:\[esp+18\]  
> 00411615    68 70324300     push SmartUp.00433270           ; ASCII "%s\\%s"  
> 0041161A    50              push eax  
> 0041161B    C64424 44 01    mov byte ptr ss:\[esp+44\],1  
> 00411620    E8 BF060100     call <jmp.&MFC42.#2818>  
> 00411625    8B4C24 20       mov ecx,dword ptr ss:\[esp+20\]  
> 00411629    83C4 10         add esp,10  
> 0041162C    51              push ecx  
> 0041162D    6A 20           push 20  
> 0041162F    6A 20           push 20  
> 00411631    8D4C24 24       lea ecx,dword ptr ss:\[esp+24\]  
> 00411635    E8 E6060100     call <jmp.&MFC42.#2915>  
> 0041163A    50              push eax  
> 0041163B    68 F8324300     push SmartUp.004332F8           ; ASCII "18.00"  
> 00411640    68 F0324300     push SmartUp.004332F0           ; ASCII "Version"  
> 00411645    68 E0414300     push SmartUp.004341E0           ; ASCII "Update"  
> 0041164A    FF15 ECC04200   call dword ptr ds:\[<&KERNEL32.G>;   
>   
> kernel32.GetPrivateProfileStringA  
> 00411650    6A FF           push -1  
> 00411652    8D4C24 1C       lea ecx,dword ptr ss:\[esp+1C\]  
> 00411656    E8 BF060100     call <jmp.&MFC42.#5572>  
> 0041165B    8B3F            mov edi,dword ptr ds:\[edi\]  
> 0041165D    8B5424 18       mov edx,dword ptr ss:\[esp+18\]  
> 00411661    57              push edi  
> 00411662    52              push edx  
> 00411663    FF15 54C44200   call dword ptr ds:\[<&MSVCRT.\_mb>;   
>   
> msvcrt.\_mbscmp  
> 00411669    83C4 08         add esp,8  
> 0041166C    85C0            test eax,eax  
> ...........................................  
> 00407601    BF 98364300     mov edi,SmartUp.00433698        ; ASCII "&sn="        
>   
>          ;这里 EBP=序列号,EBX=ID  
> 00407606    F2:AE           repne scas byte ptr es:\[edi\]  
> 00407608    F7D1            not ecx  
> 0040760A    2BF9            sub edi,ecx  
> 0040760C    8BF7            mov esi,edi  
> 0040760E    8BD1            mov edx,ecx  
> 00407610    83C9 FF         or ecx,FFFFFFFF  
> \----  
> 到这里要开始了，是关键的地方大家看好了。  
> 0040C4E4    50              push eax                                      
> EAX=11EFADC,http://download.rising.com.cn/register/pcver/autoupgradepad/pc  
>   
> ver2006new/?Info=MIGIAkIBOFxRs/mtaetkR/YB 后面省略(这个信息很重要!)  
> 0040C4E5    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> esp+14 处变为  
>   
> http://download.rising.com.cn/re ... epad/pcver2006new/?  
>   
> Info=MIGIAkIBOFxRs/mtaetkR/YB 后面省略  
> 0040C4E9    E8 78570100     call <jmp.&MFC42.#860>  
> 0040C4EE    8B86 74070000   mov eax,dword ptr ds:\[esi+774\]  
> 0040C4F4    85C0            test eax,eax  
> 0040C4F6    0F85 32080000   jnz SmartUp.0040CD2E                      不跳  
> 0040C4FC    8B56 20         mov edx,dword ptr ds:\[esi+20\]  
> 0040C4FF    6A 00           push 0  
> 。。  
> 0040C531    50              push eax  
> 0040C532    51              push ecx  
> 0040C533    FF15 98C04200   call dword ptr ds:\[<&KERNEL32.l>;   
>   
> kernel32.lstrcpyA  
> 0040C539    8D4C24 10       lea ecx,dword ptr ss:\[esp+10\]  
> 0040C53D    E8 56580100     call <jmp.&MFC42.#4202>  
> 0040C542    68 C83D4300     push SmartUp.00433DC8           ; ASCII   
>   
> "notuse.asp"  
> 0040C547    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C54B    E8 DC570100     call <jmp.&MFC42.#2764>  
> 0040C550    83CB FF         or ebx,FFFFFFFF  
> 0040C553    3BC3            cmp eax,ebx  
> 0040C555    74 2A           je short SmartUp.0040C581           跳  
> 0040C557    68 6FEA0000     push 0EA6F  
> 0040C55C    8BCE            mov ecx,esi  
> 0040C55E    E8 0D3F0000     call SmartUp.00410470  
> 0040C581    68 B83D4300     push SmartUp.00433DB8           ; ASCII   
>   
> "toomoreid.asp"    ；升级次数过多  
> 0040C586    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C58A    E8 9D570100     call <jmp.&MFC42.#2764>  
> 0040C58F    3BC3            cmp eax,ebx  
> 0040C591   /74 0A           je short SmartUp.0040C59D            跳  
> 0040C593   |68 70EA0000     push 0EA70  
> 0040C598   |E9 84010000     jmp SmartUp.0040C721  
> 0040C59D   \\68 A83D4300     push SmartUp.00433DA8           ; ASCII   
>   
> "notthisid.asp"     ；ID 错误  
>   
> 0040C5AD   /0F84 25010000   je SmartUp.0040C6D8                      ；跳  
>   
> 0040C6D8    68 8C3D4300     push SmartUp.00433D8C           ; ASCII   
>   
> "nomatch.asp"   ；还在验证  
> 0040C6DD    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C6E1    E8 46560100     call <jmp.&MFC42.#2764>  
> 0040C6E6    3BC3            cmp eax,ebx  
> 0040C6E8    74 07           je short SmartUp.0040C6F1       ;              还是要跳  
>   
> 0040C701   /74 07           je short SmartUp.0040C70A       ; 跳  
> 0040C703   |68 73EA0000     push 0EA73  
> 0040C708   |EB 17           jmp short SmartUp.0040C721  
> 0040C70A   \\68 6C3D4300     push SmartUp.00433D6C           ; ASCII   
>   
> "wrongtype.asp"  
> 0040C70F    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C713    E8 14560100     call <jmp.&MFC42.#2764>  
> 0040C718    3BC3            cmp eax,ebx  
> 0040C71A    74 3A           je short SmartUp.0040C756       ; 跳  
>   
> 0040C75E   /0F85 CA010000   jnz SmartUp.0040C92E            ; 不跳  
>   
> 0040C764    68 5C3D4300     push SmartUp.00433D5C           ; ASCII   
>   
> "notregister.asp"  
> 0040C769    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C76D    E8 BA550100     call <jmp.&MFC42.#2764>  
> 0040C772    3BC3            cmp eax,ebx  
> 0040C774    74 63           je short SmartUp.0040C7D9       ; 不跳则没有注册  
> 0040C776    81C6 18040000   add esi,418  
>   
> 0040C7E1   /0F85 47010000   jnz SmartUp.0040C92E                ；不跳  
> 0040C7E7   |68 3C3D4300     push SmartUp.00433D3C           ; ASCII   
>   
> "overtime.asp"  
> 0040C7EC   |8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0040C7F0   |E8 37550100     call <jmp.&MFC42.#2764>  
> 0040C7F5   |3BC3            cmp eax,ebx                     ; eax fff  
> 0040C7F7   |0F84 31010000   je SmartUp.0040C92E             ; 不跳则提示 ID 过  
>   
> 期  
> 到这里  
> 0040C92E    8D8424 A4000000 lea eax,dword ptr ss:\[esp+A4\]   ；\[ESP+A4\]  
>   
> \=11EFADC，压入 EAX 就是上面的地址  
> 0040C935    6A 3F           push 3F                         ; eax =wanzhi  
> 0040C937    50              push eax  
> 0040C938    FF15 24C44200   call dword ptr ds:\[<&MSVCRT.\_mb>;   
>   
> msvcrt.\_mbsrchr  
> 0040C93E    8BF8            mov edi,eax  
> 0040C940    83C4 08         add esp,8  
> 0040C943    33DB            xor ebx,ebx  
> 0040C945    85FF            test edi,edi  
> 0040C947    0F84 C6030000   je SmartUp.0040CD13             ; 跳则提示返回信  
>   
> 息错误，其实就是地址后面的 info=xxxx  
> 下面继续  
> 0040CA80   /0F85 A8020000   jnz SmartUp.0040CD2E            ; 不要跳  
> 0040CA86   |8B5424 10       mov edx,dword ptr ss:\[esp+10\]   ; \[ESP+10\]  
>   
> \=11EFADC 压入 EDX(就是地址，经过上面的处理已经变成  
>   
> http://download.rising.com.cn/re ... depad/pcver2006new/呵  
>   
> 呵和我门抓到的比较一下就知道拉~  
>   
> 0040CA8A   |B9 94714300     mov ecx,SmartUp.00437194  
> 0040CA8F   |52              push edx  
> 0040CA90   |E8 D1510100     call <jmp.&MFC42.#860>  
> 0040CA95   |8D4424 10       lea eax,dword ptr ss:\[esp+10\]  
> 0040CA99   |68 203D4300     push SmartUp.00433D20           ; ASCII   
>   
> "CompsVer"  
> 0040CA9E   |8D4C24 20       lea ecx,dword ptr ss:\[esp+20\]  
> 0040CAA2   |50              push eax  
> 0040CAA3   |51              push ecx  
> 0040CAA4   |E8 8B530100     call <jmp.&MFC42.#924>  
> 0040CAA9   |8D8E 88070000   lea ecx,dword ptr ds:\[esi+788\]  
> 0040CAAF   |8D5424 20       lea edx,dword ptr ss:\[esp+20\]  
> 0040CAB3   |51              push ecx  
> 0040CAB4   |50              push eax  
> 0040CAB5   |52              push edx  
> 0040CAB6   |C68424 B8040000>mov byte ptr ss:\[esp+4B8\],0A  
> 0040CABE   |E8 9B530100     call <jmp.&MFC42.#922>  
> 0040CAC3   |68 183D4300     push SmartUp.00433D18           ; ASCII ".inf"  
> 0040CAC8   |50              push eax  
> 0040CAC9   |8D4424 2C       lea eax,dword ptr ss:\[esp+2C\]  
> 0040CACD   |B3 0B           mov bl,0B  
> 0040CACF   |50              push eax  
> 0040CAD0   |889C24 B8040000 mov byte ptr ss:\[esp+4B8\],bl  
> 0040CAD7   |E8 58530100     call <jmp.&MFC42.#924>  
> 0040CADC   |50              push eax  
> 0040CADD   |8D4C24 18       lea ecx,dword ptr ss:\[esp+18\]  
> 0040CAE1   |C68424 B0040000>mov byte ptr ss:\[esp+4B0\],0C  
> 0040CAE9   |E8 FC510100     call <jmp.&MFC42.#858>  
> 0040CAEE   |8D4C24 24       lea ecx,dword ptr ss:\[esp+24\]  
> 0040CAF2   |889C24 AC040000 mov byte ptr ss:\[esp+4AC\],bl  
> 0040CAF9   |E8 62510100     call <jmp.&MFC42.#800>  
> 0040CAFE   |8D4C24 20       lea ecx,dword ptr ss:\[esp+20\]  
> 0040CB02   |C68424 AC040000>mov byte ptr ss:\[esp+4AC\],0A  
> 0040CB0A   |E8 51510100     call <jmp.&MFC42.#800>  
> 0040CB0F   |8D4C24 1C       lea ecx,dword ptr ss:\[esp+1C\]  
> 0040CB13   |C68424 AC040000>mov byte ptr ss:\[esp+4AC\],3  
> 0040CB1B   |E8 40510100     call <jmp.&MFC42.#800>  
> 0040CB20   |8D8E 18040000   lea ecx,dword ptr ds:\[esi+418\]  
> 0040CB26   |68 0C3D4300     push SmartUp.00433D0C           ; ASCII   
>   
> "\\Download\\"  
> 0040CB2B   |8D5424 4C       lea edx,dword ptr ss:\[esp+4C\]  
> 0040CB2F   |51              push ecx  
> 0040CB30   |52              push edx  
> 0040CB31   |E8 FE520100     call <jmp.&MFC42.#924>  
> 0040CB36   |68 10334300     push SmartUp.00433310           ; ASCII   
>   
> "CompsVer.inf"  
> 0040CB3B   |50              push eax  
> 0040CB3C   |8D4424 54       lea eax,dword ptr ss:\[esp+54\]  
> 0040CB40   |B3 0D           mov bl,0D  
> 0040CB42   |50              push eax  
> 0040CB43   |889C24 B8040000 mov byte ptr ss:\[esp+4B8\],bl  
> 0040CB4A   |E8 E5520100     call <jmp.&MFC42.#924>  
> 0040CB4F   |8DAE 7C070000   lea ebp,dword ptr ds:\[esi+77C\]  
> 0040CB55   |50              push eax  
> 0040CB56   |8BCD            mov ecx,ebp  
> 0040CB58   |C68424 B0040000>mov byte ptr ss:\[esp+4B0\],0E  
> 0040CB60   |E8 85510100     call <jmp.&MFC42.#858>  
> 0040CB65   |8D4C24 4C       lea ecx,dword ptr ss:\[esp+4C\]  
> 0040CB69   |889C24 AC040000 mov byte ptr ss:\[esp+4AC\],bl  
> 0040CB70   |E8 EB500100     call <jmp.&MFC42.#800>  
> 0040CB75   |8D4C24 48       lea ecx,dword ptr ss:\[esp+48\]  
> 0040CB79   |C68424 AC040000>mov byte ptr ss:\[esp+4AC\],3  
> 0040CB81   |E8 DA500100     call <jmp.&MFC42.#800>  
> 0040CB86   |33DB            xor ebx,ebx  
> 0040CB88   |43              inc ebx  
> 0040CB89   |83FB 03         cmp ebx,3  
> 0040CB8C   |7F 42           jg short SmartUp.0040CBD0       ; 不跳  
> 0040CB8E > |8B45 00         mov eax,dword ptr ss:\[ebp\]        取得保存升级文件的  
>   
> 路径 EAX=D:\\Program Files\\Rising\\Rav\\Download\\CompsVer.inf  
> 0040CB91   |8B4C24 14       mov ecx,dword ptr ss:\[esp+14\]   ; \[ESP+14\]压入  
>   
> ECX 就是  
>   
> http://download.rising.com.cn/re ... pad/pcver2006new/Co  
>   
> mpsVer18.53.42.inf，这个是经过上面的处理得到的  
> 0040CB95   |6A 00           push 0                          ;   
> 0040CB97   |50              push eax  
> 0040CB98   |51              push ecx  
> 0040CB99   |8D8E C0030000   lea ecx,dword ptr ds:\[esi+3C0\]  
> 0040CB9F   |E8 EC5AFFFF     call SmartUp.00402690           ;CALL 下载文件  
> 0040CBA4   |8BF8            mov edi,eax  
> 0040CBA6   |85FF            test edi,edi                     ；比较是否下载成功  
> 0040CBA8   |74 44           je short SmartUp.0040CBEE        ；下载成功就跳  
> 0040CBAA   |8B86 10040000   mov eax,dword ptr ds:\[esi+410\]  
> 0040CBB0   |50              push eax  
> 0040CBB1   |57              push edi  
> 0040CBB2   |68 10334300     push SmartUp.00433310           ; ASCII   
>   
> "CompsVer.inf"  
> 0040CBB7   |68 D83C4300     push SmartUp.00433CD8           ; ASCII   
>   
> "Download %s Error: ErrCode = 0x%x; LastError = %d"  
> 0040CBBC   |6A 04           push 4  
>   
> 哎。。。。。。。。。。。。。。。下面的文件我丢了，也不想写了，直接给出  
>   
> SmartUp.exe 的 补丁方法。。。。当然有很多你可以直接把  
>   
> http://download.rising.com.cn/re ... pad/pcver2006new/Co  
>   
> mpsVer18.53.42.inf 弄进去，这个以后太麻烦还有自己更新。我给大家的破解就  
>   
> 是这个方法  
>   
> 个人认为  
>   
> 记忆里弄下了。。。。 原来的代码就不给了  
>   
> 0040C4E4   /E9 15F80100     jmp SmartUp2.0042BCFE                             这里  
>   
> 开始补丁 跳往补丁  
> 0040C4E9   |E8 78570100     call <jmp.&MFC42.#860>  
>   
> 0042BCFE    B8 34BD4200     mov eax,SmartUp2.0042BD34                          ;   
>   
> ASCII   
>   
> "http://download.rising.com.cn/register/pcver/autoupgradepad/pcver2006new/"  
> 0042BD03    50              push eax  
> 0042BD04    8D4C24 14       lea ecx,dword ptr ss:\[esp+14\]  
> 0042BD08  ^ E9 DC07FEFF     jmp SmartUp2.0040C4E9                         返回，  
>   
> 继续执行  
>   
>   
> 0042BD34    68 7474703A     push 3A707474  
> 0042BD39    2F              das  
> 0042BD3A    2F              das  
> 0042BD3B    64:6F           outs dx,dword ptr es:\[edi\]  
> 0042BD3D    77 6E           ja short SmartUp2.0042BDAD  
> 0042BD3F    6C              ins byte ptr es:\[edi\],dx  
> 0042BD40    6F              outs dx,dword ptr es:\[edi\]  
> 0042BD41    61              popad  
> 0042BD42    64:             prefix fs:  
> 0042BD43    2E:72 69        jb short SmartUp2.0042BDAF  
> 0042BD46    73 69           jnb short SmartUp2.0042BDB1  
> 0042BD48    6E              outs dx,byte ptr es:\[edi\]  
> 0042BD49    67:2E:636F 6D   arpl word ptr cs:\[bx+6D\],bp  
> 0042BD4E    2E:636E 2F      arpl word ptr cs:\[esi+2F\],bp  
> 0042BD52    72 65           jb short SmartUp2.0042BDB9  
> 0042BD54    67:6973 74 6572>imul esi,dword ptr ss:\[bp+di+74\],702F7265  
> 0042BD5C    6376 65         arpl word ptr ds:\[esi+65\],si  
> 0042BD5F    72 2F           jb short SmartUp2.0042BD90  
> 0042BD61    61              popad  
> 0042BD62    75 74           jnz short SmartUp2.0042BDD8  
> 0042BD64    6F              outs dx,dword ptr es:\[edi\]  
> 0042BD65    75 70           jnz short SmartUp2.0042BDD7  
> 0042BD67    67:72 61        jb short SmartUp2.0042BDCB  
> 0042BD6A    64:             prefix fs:  
> 0042BD6B    65:70 61        jo short SmartUp2.0042BDCF  
> 0042BD6E    64:2F           das  
> 0042BD70    70 63           jo short SmartUp2.0042BDD5  
> 0042BD72    76 65           jbe short SmartUp2.0042BDD9  
> 0042BD74    72 32           jb short SmartUp2.0042BDA8  
> 0042BD76    3030            xor byte ptr ds:\[eax\],dh  
> 0042BD78    36:6E           outs dx,byte ptr es:\[edi\]  
> 0042BD7A    65:77 2F        ja short SmartUp2.0042BDAC  
> 0042BD7D    0000            add byte ptr ds:\[eax\],al  
> 0042BD7F    0000            add byte ptr ds:\[eax\],al  
> 这样可以跳过 SmartUP 的验证了。。。。 开始下载文件。  
> 启动 RAVCOPY 时还有一次，方法类似就不写下去了。。。  
>   
>   
> 升级了 2007 老的不能用了/////////  
> 一些关键的地方手记资料丢了,,,升级成 2007 的了,也不能再分析给大家了.... 大家有兴趣自己玩玩..请海涵.  
> By   FoBnN    2007.1.1   
> http://chinatrojan.com/0day/pkrav2006/SmartUp.rar 

 4dd466d0e1e0cc123afbe504da2462ca