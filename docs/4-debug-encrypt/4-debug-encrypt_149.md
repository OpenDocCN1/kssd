

# RSA-开心小闹钟 2.7 Build 0107

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-102.htm`](https://www.pediy.com/kssd/pediy09/pediy09-102.htm)

> 软件名称：开心小闹钟 2.7 Build 0107  
> 主程序：Clock.exe  
> 下载地址：[`www.famousoft.net/downloads/s....php?product=1`](http://www.famousoft.net/downloads/sendfile.php?product=1)  
> 其实主页已经关闭，呵呵。  
> 加壳保护：ASPack 2.12 -> Alexey Solodovnikov  
> 这是压缩壳，很容易就脱掉了。  
> 编译语言：Microsoft Visual C++ 6.0 编译  
> 启动软件，看它的皮肤与罗云彬写的那个免费软件很相像。想必作者也还在学习过程中？（注册费 26 人民币/1 台机）。不管啦，帮他摧广一下。  
> 这个软件有一个语音报时的功能，很爽呀。而且还有个网络对时。  
> 当初就是为了这个网络对时才下载的。  
> 本机机器码：Y2VHK5TE  (硬盘序列号啦)  
> 输入假注册码：123456  
> 要求重启验证。  
> 这类软件一般是要读注册表，INI 文件。  
> 这个软件是读注册表。  
> OD 载入，下断：  
> bpx RegQueryValueExA  
>   
> 0040E245   |.  C74424 1C 00010000                mov dword ptr ss:\[esp+1C\],100  
> 0040E24D   |.  50                                push eax                           ; /pHandle  
> 0040E24E   |.  6A 01                             push 1                             ; |Access = KEY\_QUERY\_VALUE  
> 0040E250   |.  6A 00                             push 0                             ; |Reserved = 0  
> 0040E252   |.  68 98F84600                       push cc\_.0046F898                  ; |Subkey = "SOFTWARE\\HappyClock\\License"  
> 0040E257   |.  68 02000080                       push 80000002                      ; |hKey = HKEY\_LOCAL\_MACHINE  
> 0040E25C   |.  FF15 18504600                     call dword ptr ds:\[<&advapi32.RegO>; \\RegOpenKeyExA  
> 0040E262   |.  85C0                              test eax,eax  
> 0040E264   |.  0F85 59010000                     jnz cc\_.0040E3C3－－－－－－因为注册码是假的，并不会写入注册表，所以这里就跳走了，直接就到了要求注册的那个 Nag。  
> 从 OD 中可以看到下面的东东，知道，如果注册成功，会有如下注册表值：  
> ……  
> 0040E27F   |.  8B4424 24                         mov eax,dword ptr ss:\[esp+24\]      ; |  
> 0040E283   |.  68 90F84600                       push cc\_.0046F890                  ; |ValueName = "Name"  
> 0040E288   |.  50                                push eax                           ; |hKey  
> 0040E289   |.  FFD7                              call edi                           ; \\RegQueryValueExA  
> ……  
> 0040E29B   |.  6A 00                             push 0                             ; |Reserved = NULL  
> 0040E29D   |.  68 88F84600                       push cc\_.0046F888                  ; |ValueName = "Keygen"  
> 0040E2A2   |.  50                                push eax                           ; |hKey  
> 0040E2A3   |.  FFD7                              call edi                           ; \\RegQueryValueExA  
>   
> 于是，我先建一个假的注册表值，导入  
>   
> HKEY\_LOCAL\_MACHINE\\SOFTWARE\\HappyClock\\License  
> 其中有两个字符串值：name 和 Keygen  
> name=Y2VHK5TE  
> keygen=123456  
> 如下：  
> Windows Registry Editor Version 5.00  
>   
> \[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\HappyClock\]  
>   
> \[HKEY\_LOCAL\_MACHINE\\SOFTWARE\\HappyClock\\License\]  
> "name"="Y2VHK5TE"  
> "keygen"="123456"  
>   
> OD 载入，F9 运行，在先前的断点处断下：  
> 0040E28F   |.  8D4C24 1C                         lea ecx,dword ptr ss:\[esp+1C\]------------------这里就出现了注册名（机器码）:Y2VHK5TE  
> 0040E293   |.  8D5424 24                         lea edx,dword ptr ss:\[esp+24\]  
> 0040E297   |.  51                                push ecx                           ; /pBufSize  
> 0040E298   |.  52                                push edx                           ; |Buffer  
> 0040E299   |.  6A 00                             push 0                             ; |pValueType = NULL  
> 0040E29B   |.  6A 00                             push 0                             ; |Reserved = NULL  
> 0040E29D   |.  68 88F84600                       push cc\_.0046F888                  ; |ValueName = "Keygen"  
> 0040E2A2   |.  50                                push eax                           ; |hKey  
> 0040E2A3   |.  FFD7                              call edi                           ; \\RegQueryValueExA  
> 0040E2A5   |.  8B4C24 14                         mov ecx,dword ptr ss:\[esp+14\]－－－－－－－－－读出了注册码：123456  
> 0040E2A9   |.  51                                push ecx                           ; /hKey  
> 0040E2AA   |.  FF15 10504600                     call dword ptr ds:\[<&advapi32.RegC>; \\RegCloseKey  
> 0040E2B0   |.  8D7C24 24                         lea edi,dword ptr ss:\[esp+24\]  
> 0040E2B4   |.  83C9 FF                           or ecx,FFFFFFFF  
> 0040E2B7   |.  33C0                              xor eax,eax  
> 0040E2B9   |.  F2:AE                             repne scas byte ptr es:\[edi\]  
> 0040E2BB   |.  F7D1                              not ecx  
> 0040E2BD   |.  49                                dec ecx  
> 0040E2BE   |.  33FF                              xor edi,edi  
> 0040E2C0   |.  3BCF                              cmp ecx,edi－－－－－－－－－－－比较了注册码的长度后，不为空，就该算号了  
> 0040E2C2   |.  894C24 18                         mov dword ptr ss:\[esp+18\],ecx  
> 0040E2C6   |.  897C24 10                         mov dword ptr ss:\[esp+10\],edi  
> 0040E2CA   |.  7E 1F                             jle short cc\_.0040E2EB  
> 0040E2CC   |>  0FBE543C 24                       /movsx edx,byte ptr ss:\[esp+edi+24>  
> 0040E2D1   |.  52                                |push edx  
> 0040E2D2   |.  E8 1B340400                       |call cc\_.004516F2  
> 0040E2D7   |.  83C4 04                           |add esp,4  
> 0040E2DA   |.  85C0                              |test eax,eax  
> 0040E2DC   |.  0F84 85000000                     |je cc\_.0040E367  
> 0040E2E2   |.  8B4424 18                         |mov eax,dword ptr ss:\[esp+18\]  
> 0040E2E6   |.  47                                |inc edi  
> 0040E2E7   |.  3BF8                              |cmp edi,eax  
> 0040E2E9   |.^ 7C E1                             \\jl short cc\_.0040E2CC  
> 0040E2EB   |>  8B4424 18                         mov eax,dword ptr ss:\[esp+18\]  
> 0040E2EF   |.  85C0                              test eax,eax  
> 0040E2F1   |.  74 7C                             je short cc\_.0040E36F  
> 0040E2F3   |.  8D4424 24                         lea eax,dword ptr ss:\[esp+24\]  
> 0040E2F7   |.  50                                push eax  
> 0040E2F8   |.  56                                push esi  
> 0040E2F9   |.  E8 120B0500                       call cc\_.0045EE10  
> 0040E2FE   |.  68 BCF84600                       push cc\_.0046F8BC                  ;  ASCII "963251DC5A9C90D9F203A03C363BA411"－－RSA 的大数，即 Modulus(N)  
> 0040E303   |.  55                                push ebp  
> 0040E304   |.  E8 070B0500                       call cc\_.0045EE10  
> 0040E309   |.  8B7C24 30                         mov edi,dword ptr ss:\[esp+30\]  
> 0040E30D   |.  68 B4F84600                       push cc\_.0046F8B4                  ;  ASCII "10001"－－RSA 的公钥 Public Exponent(E)(Hex)  
> 0040E312   |.  57                                push edi  
> 0040E313   |.  E8 F80A0500                       call cc\_.0045EE10  
> 0040E318   |.  55                                push ebp  
> 0040E319   |.  56                                push esi  
> 0040E31A   |.  E8 81F80400                       call cc\_.0045DBA0  
> 0040E31F   |.  83C4 20                           add esp,20  
> 0040E322   |.  83F8 FF                           cmp eax,-1  
> 0040E325   |.  75 40                             jnz short cc\_.0040E367  
> 0040E327   |.  53                                push ebx  
> 0040E328   |.  55                                push ebp  
> 0040E329   |.  57                                push edi  
> 0040E32A   |.  56                                push esi  
> 0040E32B   |.  E8 60080500                       call cc\_.0045EB90  
> 0040E330   |.  8D8C24 34010000                   lea ecx,dword ptr ss:\[esp+134\]  
> 0040E337   |.  6A 00                             push 0  
> 0040E339   |.  51                                push ecx  
> 0040E33A   |.  53                                push ebx  
> 0040E33B   |.  68 00010000                       push 100  
> 0040E340   |.  E8 4B020500                       call cc\_.0045E590  
> 0040E345   |.  56                                push esi  
> 0040E346   |.  E8 B5F20400                       call cc\_.0045D600  
> 0040E34B   |.  53                                push ebx  
> 0040E34C   |.  E8 AFF20400                       call cc\_.0045D600  
> 0040E351   |.  55                                push ebp  
> 0040E352   |.  E8 A9F20400                       call cc\_.0045D600  
> 0040E357   |.  57                                push edi  
> 0040E358   |.  E8 A3F20400                       call cc\_.0045D600  
> 0040E35D   |.  83C4 30                           add esp,30  
> 0040E360   |.  E8 BBF20400                       call cc\_.0045D620  
> 0040E365   |.  EB 08                             jmp short cc\_.0040E36F  
> 0040E367   |>  C74424 10 01000000                mov dword ptr ss:\[esp+10\],1  
> 0040E36F   |>  8DB424 24010000                   lea esi,dword ptr ss:\[esp+124\]  ;对注册码进行 RSA 加密：0012FC60  4E A5 23 D8 81 5A 14 4A 0D A5 08 52 2F 47 BA DD   
> 0040E376   |.  8D8424 24020000                   lea eax,dword ptr ss:\[esp+224\]  ;机器码扩展：0012FD60  593256484B3554450000350054004500  Y2VHK5TE..5.T.E.  
> 下面就是典型的比较过程，不用再索了。  
> 0040E37D  |>  8A10          /mov     dl, byte ptr \[eax\]  
> 0040E37F  |.  8A1E          |mov     bl, byte ptr \[esi\]  
> 0040E381  |.  8ACA          |mov     cl, dl  
> 0040E383  |.  3AD3          |cmp     dl, bl  
> 0040E385  |.  75 1E         |jnz     short 0040E3A5  
> 0040E387  |.  84C9          |test    cl, cl  
> 0040E389  |.  74 16         |je      short 0040E3A1  
> 0040E38B  |.  8A50 01       |mov     dl, byte ptr \[eax+1\]  
> 0040E38E  |.  8A5E 01       |mov     bl, byte ptr \[esi+1\]  
> 0040E391  |.  8ACA          |mov     cl, dl  
> 0040E393  |.  3AD3          |cmp     dl, bl  
> 0040E395  |.  75 0E         |jnz     short 0040E3A5  
> 0040E397  |.  83C0 02       |add     eax, 2  
> 0040E39A  |.  83C6 02       |add     esi, 2  
> 0040E39D  |.  84C9          |test    cl, cl  
> 0040E39F  |.^ 75 DC         \\jnz     short 0040E37D  
> 0040E3A1  |>  33C0          xor     eax, eax                         ;  全部相等，来到这里，使 EAX=0  
> 0040E3A3  |.  EB 05         jmp     short 0040E3AA  
> 0040E3A5  |>  1BC0          sbb     eax, eax  
> 0040E3A7  |.  83D8 FF       sbb     eax, -1  
> 0040E3AA  |>  85C0          test    eax, eax                         ;  然后跳到这里，测试一下，如果 EAX=0 表明注册成功  
> 0040E3AC  |.  75 15         jnz     short 0040E3C3  
> 0040E3AE  |.  8B4C24 10     mov     ecx, dword ptr \[esp+10\]  
> 0040E3B2  |.  B8 01000000   mov     eax, 1                           ;  赋注册标志，使 EAX=1  
> 0040E3B7  |.  3BC8          cmp     ecx, eax  
> 0040E3B9  |.  74 08         je      short 0040E3C3  
> 0040E3BB  |.  8B4C24 18     mov     ecx, dword ptr \[esp+18\]  
> 0040E3BF  |.  85C9          test    ecx, ecx  
> 0040E3C1  |.  75 02         jnz     short 0040E3C5  
> 0040E3C3  |>  33C0          xor     eax, eax                         ;  否则，EAX=0，玩完  
> 0040E3C5  |>  5F            pop     edi  
> 0040E3C6  |.  5E            pop     esi  
> 0040E3C7  |.  5D            pop     ebp  
> 0040E3C8  |.  5B            pop     ebx  
> 0040E3C9  |.  81C4 14030000 add     esp, 314  
> 0040E3CF  \\.  C3            retn  
>   
>   
>   
> 假注册码：123456  
> RSA 的大数：963251DC5A9C90D9F203A03C363BA411  
> RSA 的公钥：10001  
> 对假注册码进行 RSA 加密得到：4EA523D8815A144A0DA508522F47BADD，对该数解密得到：12 34 56  
> 它将与扩展的机器码比较，必须相等才能注册成功。  
> 本机机器码：Y2VHK5TE  
> 扩展为：  
> 593256484B3554450000350054004500  即：Y2VHK5TE..5.T.E.  
>   
> 于是有：  
> RSA 的 Modulus(N)为：963251DC5A9C90D9F203A03C363BA411  
> RSA 的 Public Exponent(E)Hex 为：10001  
> 先用工具(注：我用 RSATool2V110)分解，求得 P=E34436F5F48A227B Q=A92FA24467C4E3E3  
> 求出 Private Exponent(D):=56157D29A89D77BF2F669A8F0B123CC9  
> 需要解密的数据 Ciper 是：593256484B3554450000350054004500  
>   
> 因为有：M=C^D mod N  
> 于是：M=593256484B3554450000350054004500⁵⁶¹⁵⁷D29A89D77BF2F669A8F0B123CC9 mod 963251DC5A9C90D9F203A03C363BA411  
> 于是：M 为：  
> 8262E1FFE46462EF42EAE4B6C8E23B71  
> 它就是本机注册码！！！  
>   
> 但是如果是通过注册表导入的数据，则注册不会成功。如果是注册表导入的数据，则只读取 8 位，如果整个数据长度大于 8 位，则什么也不会读。  
> 最后注册肯定不能通过。  
> 唉。  
> 好了，关闭 OD，重启程序，输入注册码 8262E1FFE46462EF42EAE4B6C8E23B71，当的一声，注册成功！  
> by wofan\[OCN\]  
> 在电脑里存了很久了，稍作修改，贴出来。  
> 22:48 2007-3-31 

 4dd466d0e1e0cc123afbe504da2462ca