

# 逆向 Keygen for Matlab R14

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-130.htm`](https://www.pediy.com/kssd/pediy09/pediy09-130.htm)

> 逆向 Keygen for Matlab R14  
>   
>   
> 很想知道 Matlab 的注册算法，不过自己去跟 Matlab 难度太大，还是分析现成的 keygen 来得快。  
> TEAM ROR 在很久以前曾经出过几个 Matlab 的 keygen，分别对应 Matlab v7.0,v7.01,v7.1。  
> 我选择了 Keygen for Matlab R14（for v7.1）动手。废话少说，开始工作。  
>   
> 1、查壳  
> UPX 0.71 - 0.72 -> Markus & Laszlo  
> 手动脱壳，轻松搞定。  
>   
> 2、脱壳后检查  
> Microsoft Visual C++ 6.0  
> 没有使用任何密码学算法。  
>   
> 3、找注册码生成函数  
> IDA 加载，分析完成后搜索字符串“14”，找到一处。该处的函数正是生成注册码的关键函数：  
>   
> \--------------------------------------------------------------------------------  
>   
> UPX0:00407890 GenerateKey     proc near               ; CODE XREF: sub\_407470+1E8p  
> UPX0:00407890                                         ; UPX0:00407880j  
> UPX0:00407890  
> UPX0:00407890 szTemp          = byte ptr -1020h  
> UPX0:00407890 szSerial        = dword ptr -1000h  
> UPX0:00407890  
> UPX0:00407890                 mov     eax, 1020h  
> UPX0:00407895                 call    \_\_alloca\_probe  
> UPX0:0040789A                 push    ebx  
> UPX0:0040789B                 push    ebp  
> UPX0:0040789C                 push    esi  
> UPX0:0040789D                 push    edi  
> UPX0:0040789E                 mov     ebp, ecx  
> UPX0:004078A0                 mov     szBuf, 62h  
> UPX0:004078A7                 mov     byte\_412215, 9  
> UPX0:004078AE                 call    GetTickCount  
> UPX0:004078B4                 mov     esi, eax  
> UPX0:004078B6                 call    rand  
> UPX0:004078BC                 imul    esi, eax  
> UPX0:004078BF                 mov     word\_412216, si  
> UPX0:004078C6                 mov     byte ptr dword\_412218, 33h  
> UPX0:004078CD                 mov     byte ptr dword\_412218+1, 0  
> UPX0:004078D4                 call    func1           ; 这是一个关键 call,比较简单  
> UPX0:004078D9                 call    func4           ; 这也是一个关键 call，稍微复杂一些  
> UPX0:004078DE                 lea     eax, \[esp+1030h+szSerial\]  
> UPX0:004078E2                 push    offset a14      ; "14"  
> UPX0:004078E7                 push    eax  
> UPX0:004078E8                 call    lstrcpy  
> UPX0:004078EE                 mov     ebx, wsprintfA  
> UPX0:004078F4                 mov     edi, lstrcat  
> UPX0:004078FA                 mov     esi, offset szBuf  
> UPX0:004078FF  
> UPX0:004078FF loc\_4078FF:                             ; CODE XREF: GenerateKey+AEj  
> UPX0:004078FF                 push    esi  
> UPX0:00407900                 call    func5           ; 这个函数比较简单  
> UPX0:00407905                 add     esp, 4  
> UPX0:00407908                 and     eax, 0FFFFh  
> UPX0:0040790D                 lea     ecx, \[esp+1030h+szTemp\]  
> UPX0:00407911                 push    eax  
> UPX0:00407912                 push    offset a05lu    ; "%05lu"  
> UPX0:00407917                 push    ecx             ; LPSTR  
> UPX0:00407918                 call    ebx ; wsprintfA  
> UPX0:0040791A                 add     esp, 0Ch  
> UPX0:0040791D                 lea     edx, \[esp+1030h+szSerial\]  
> UPX0:00407921                 push    offset asc\_412060 ; "-"  
> UPX0:00407926                 push    edx  
> UPX0:00407927                 call    edi ; lstrcat  
> UPX0:00407929                 lea     eax, \[esp+1030h+szTemp\]  
> UPX0:0040792D                 lea     ecx, \[esp+1030h+szSerial\]  
> UPX0:00407931                 push    eax  
> UPX0:00407932                 push    ecx  
> UPX0:00407933                 call    edi ; lstrcat  
> UPX0:00407935                 add     esi, 2  
> UPX0:00407938                 cmp     esi, offset word\_41226A  
> UPX0:0040793E                 jb      short loc\_4078FF  
> UPX0:00407940                 lea     edx, \[esp+1030h+szSerial\]  
> UPX0:00407944                 lea     ecx, \[ebp+9D8h\]  
> UPX0:0040794A                 push    edx  
> UPX0:0040794B                 call    CWnd::SetWindowTextA(char const \*)  
> UPX0:00407950                 pop     edi  
> UPX0:00407951                 pop     esi  
> UPX0:00407952                 pop     ebp  
> UPX0:00407953                 pop     ebx  
> UPX0:00407954                 add     esp, 1020h  
> UPX0:0040795A                 retn  
> UPX0:0040795A GenerateKey     endp  
>   
> 这个函数不太复杂，其中只有 3 个未知函数需要继续分析。逆向出来的 cpp 代码如下：  
>   
> \--------------------------------------------------------------------------------  
>   
> unsigned char szBuf\[87\] = {0};  
>   
> void CKeyGenDlg::OnGenerate()   
> {  
>   char szSerial\[1024\] = {0};  
>   char szTemp\[32\] = {0};  
>   unsigned long  dw = 0;  
>   
>   szBuf\[0\] = 0x62;  
>   szBuf\[1\] = 0x09;  
>   dw = GetTickCount() \* rand();  
>   szBuf\[2\] = (char)(dw & 0xFF);  
>   szBuf\[3\] = (char)((dw >> 8) & 0xFF);  
>   szBuf\[4\] = 0x33;  
>   szBuf\[5\] = 0x0;  
>   
>   func1();  
>   func4();  
>     
>   lstrcpy( szSerial, "14" );  
>     
>   int  i = 0;  
>   while( i < 85 )  
>   {  
>     wsprintf( szTemp, "%05lu", func5(szBuf+i)&0xFFFF );  
>     lstrcat( szSerial, "-" );  
>     lstrcat( szSerial, szTemp );  
>     i += 2;  
>   }  
>     
>   SetDlgItemText( IDC\_SERIAL, szSerial );  
> }  
>   
> \--------------------------------------------------------------------------------  
>   
> 下面是 func1()的汇编代码，由 3 个独立的循环组成，也不是很复杂。  
>   
> UPX0:00407170 func1           proc near               ; CODE XREF: GenerateKey+44p  
> UPX0:00407170  
> UPX0:00407170 var\_A80         = dword ptr -0A80h  
> UPX0:00407170 var\_A00         = dword ptr -0A00h  
> UPX0:00407170  
> UPX0:00407170                 sub     esp, 0A80h  
> UPX0:00407176                 mov     ecx, 4  
> UPX0:0040717B                 mov     \[esp+0A80h+var\_A80\], 1F1F1F1Fh  
> UPX0:00407183                 mov     edx, ecx  
> UPX0:00407185  
> UPX0:00407185 loc\_407185:                             ; CODE XREF: func1+26j  
> UPX0:00407185                 mov     eax, 1  
> UPX0:0040718A  
> UPX0:0040718A loc\_40718A:                             ; CODE XREF: func1+23j  
> UPX0:0040718A                 mov     byte ptr \[esp+ecx+0A80h+var\_A80\], al  
> UPX0:0040718E                 inc     ecx  
> UPX0:0040718F                 inc     eax  
> UPX0:00407190                 cmp     eax, 31  
> UPX0:00407193                 jle     short loc\_40718A  
> UPX0:00407195                 dec     edx  
> UPX0:00407196                 jnz     short loc\_407185  
> UPX0:00407198                 push    ebx  
> UPX0:00407199                 push    ebp  
> UPX0:0040719A                 push    esi  
> UPX0:0040719B                 push    edi  
> UPX0:0040719C                 xor     ebp, ebp  
> UPX0:0040719E                 lea     edi, \[esp+0A90h+var\_A00\]  
> UPX0:004071A5  
> UPX0:004071A5 loc\_4071A5:                             ; CODE XREF: func1+60j  
> UPX0:004071A5                 xor     eax, eax  
> UPX0:004071A7                 mov     edx, 4  
> UPX0:004071AC                 mov     al, byte ptr \[esp+ebp+0A90h+var\_A80\]  
> UPX0:004071B0                 mov     esi, eax  
> UPX0:004071B2                 mov     eax, edi  
> UPX0:004071B4                 add     edi, 20  
> UPX0:004071B7  
> UPX0:004071B7 loc\_4071B7:                             ; CODE XREF: func1+57j  
> UPX0:004071B7                 mov     ebx, esi  
> UPX0:004071B9                 mov     cl, dl  
> UPX0:004071BB                 shr     ebx, cl  
> UPX0:004071BD                 add     eax, 4  
> UPX0:004071C0                 and     ebx, 1  
> UPX0:004071C3                 dec     edx  
> UPX0:004071C4                 mov     \[eax-4\], ebx  
> UPX0:004071C7                 jns     short loc\_4071B7  
> UPX0:004071C9                 inc     ebp  
> UPX0:004071CA                 cmp     ebp, 128  
> UPX0:004071D0                 jl      short loc\_4071A5  
> UPX0:004071D2                 xor     edx, edx  
> UPX0:004071D4                 xor     eax, eax  
> UPX0:004071D6                 lea     ebp, \[esp+0A90h+var\_A00\]  
> UPX0:004071DD  
> UPX0:004071DD loc\_4071DD:                             ; CODE XREF: func1+9Bj  
> UPX0:004071DD                 mov     esi, ebp  
> UPX0:004071DF                 mov     \[eax+41221Ah\], dl  
> UPX0:004071E5                 mov     edi, 7  
> UPX0:004071EA                 add     ebp, 32  
> UPX0:004071ED  
> UPX0:004071ED loc\_4071ED:                             ; CODE XREF: func1+95j  
> UPX0:004071ED                 mov     bl, \[esi\]  
> UPX0:004071EF                 mov     ecx, edi  
> UPX0:004071F1                 shl     bl, cl  
> UPX0:004071F3                 mov     cl, \[eax+41221Ah\]  
> UPX0:004071F9                 add     esi, 4  
> UPX0:004071FC                 or      cl, bl  
> UPX0:004071FE                 dec     edi  
> UPX0:004071FF                 mov     \[eax+41221Ah\], cl  
> UPX0:00407205                 jns     short loc\_4071ED  
> UPX0:00407207                 inc     eax  
> UPX0:00407208                 cmp     eax, 80  
> UPX0:0040720B                 jl      short loc\_4071DD  
> UPX0:0040720D                 pop     edi  
> UPX0:0040720E                 pop     esi  
> UPX0:0040720F                 pop     ebp  
> UPX0:00407210                 pop     ebx  
> UPX0:00407211                 add     esp, 0A80h  
> UPX0:00407217                 retn  
> UPX0:00407217 func1           endp  
>   
> \--------------------------------------------------------------------------------  
>   
> void func1()  
> {  
>   int  i, j;  
>   char szA00\[2560\] = {0};  
>   char szA80\[128\] = {0};  
>   char \*p, \*p1, c, d;  
>   unsigned long  dw;  
>   
>   szA80\[0\] = 0x1F;  
>   szA80\[1\] = 0x1F;  
>   szA80\[2\] = 0x1F;  
>   szA80\[3\] = 0x1F;  
>   
>   for( i=0; i<4; i++ )  
>   {  
>     for( j=1; j<32; j++ )  
>     {  
>       szA80\[i\*31+j+3\] = j;  
>     }  
>   }  
>   
>   p = szA00;  
>   for( i=0; i<128; i++ )  
>   {  
>     c = szA80\[i\];  
>     p1 = p;  
>     p += 20;  
>     for( j=4; j>=0; j-- )  
>     {  
>       dw = c;  
>       dw = dw>>j;  
>       dw = dw & 1;  
>       memcpy( p1, (char\*)&dw, 4 );  
>       p1 += 4;  
>     }  
>   }  
>     
>   p = szA00;  
>   for( i=0; i<80; i++ )  
>   {  
>     p1 = p;  
>     p += 32;  
>     szBuf\[6+i\] = 0;  
>     for( j=7; j>=0; j-- )  
>     {  
>       c = \*p1;  
>       p1 += 4;  
>       c = c<<j;  
>       d = szBuf\[6+i\];  
>       szBuf\[6+i\] = d|c;  
>     }  
>   }  
> }  
>   
> \--------------------------------------------------------------------------------  
>   
> 这是 func2()的反汇编代码，作用是将连续 4 个字节转化为一个 32 位的无符号整数。  
>   
> UPX0:00407040 func2           proc near               ; CODE XREF: func4+1Bp  
> UPX0:00407040                                         ; func4+23p  
> UPX0:00407040  
> UPX0:00407040 arg\_0           = dword ptr  4  
> UPX0:00407040  
> UPX0:00407040                 mov     ecx, \[esp+arg\_0\]  
> UPX0:00407044                 xor     edx, edx  
> UPX0:00407046                 movsx   eax, byte ptr \[ecx\]  
> UPX0:00407049                 mov     dl, \[ecx+1\]  
> UPX0:0040704C                 shl     eax, 8  
> UPX0:0040704F                 or      eax, edx  
> UPX0:00407051                 xor     edx, edx  
> UPX0:00407053                 mov     dl, \[ecx+2\]  
> UPX0:00407056                 shl     eax, 8  
> UPX0:00407059                 or      eax, edx  
> UPX0:0040705B                 xor     edx, edx  
> UPX0:0040705D                 mov     dl, \[ecx+3\]  
> UPX0:00407060                 shl     eax, 8  
> UPX0:00407063                 or      eax, edx  
> UPX0:00407065                 retn  
> UPX0:00407065 func2           endp  
>   
> \--------------------------------------------------------------------------------  
>   
> unsigned long func2( unsigned char \*p )  
> {  
>   unsigned long  dw = 0;  
>   dw = \*p;  
>   dw = (dw<<8) + \*(p+1);  
>   dw = (dw<<8) + \*(p+2);  
>   dw = (dw<<8) + \*(p+3);  
>   return dw;  
> }  
>   
> \--------------------------------------------------------------------------------  
> 这是 func3()的反汇编代码，作用是将一个 32 位的无符号整数转化为连续 4 个字节。  
>   
> UPX0:00407070 func3           proc near               ; CODE XREF: func4+8Dp  
> UPX0:00407070                                         ; func4+9Ep  
> UPX0:00407070  
> UPX0:00407070 arg\_0           = dword ptr  4  
> UPX0:00407070 arg\_4           = dword ptr  8  
> UPX0:00407070  
> UPX0:00407070                 mov     eax, \[esp+arg\_0\]  
> UPX0:00407074                 mov     ecx, \[esp+arg\_4\]  
> UPX0:00407078                 mov     edx, eax  
> UPX0:0040707A                 shr     edx, 24  
> UPX0:0040707D                 mov     \[ecx\], dl  
> UPX0:0040707F                 mov     edx, eax  
> UPX0:00407081                 shr     edx, 16  
> UPX0:00407084                 mov     \[ecx+1\], dl  
> UPX0:00407087                 mov     edx, eax  
> UPX0:00407089                 shr     edx, 8  
> UPX0:0040708C                 mov     \[ecx+2\], dl  
> UPX0:0040708F                 mov     \[ecx+3\], al  
> UPX0:00407092                 retn  
> UPX0:00407092 func3           endp  
>   
> \--------------------------------------------------------------------------------  
>   
> void func3( unsigned char \*p, unsigned long dw )  
> {  
>   \*p     = (unsigned char)((dw>>24) & 0xFF);  
>   \*(p+1) = (unsigned char)((dw>>16) & 0xFF);  
>   \*(p+2) = (unsigned char)((dw>>8 ) & 0xFF);  
>   \*(p+3) = (unsigned char)( dw      & 0xFF);  
> }  
>   
> \--------------------------------------------------------------------------------  
>   
> 这个函数稍微有点麻烦，它还要用到一个常数表。  
>   
> UPX0:004070B0 func4           proc near               ; CODE XREF: GenerateKey+49p  
> UPX0:004070B0  
> UPX0:004070B0 var\_8           = dword ptr -8  
> UPX0:004070B0 var\_4           = dword ptr -4  
> UPX0:004070B0  
> UPX0:004070B0                 sub     esp, 8  
> UPX0:004070B3                 push    ebx  
> UPX0:004070B4                 push    ebp  
> UPX0:004070B5                 push    esi  
> UPX0:004070B6                 xor     esi, esi  
> UPX0:004070B8                 push    edi  
> UPX0:004070B9                 mov     \[esp+18h+var\_8\], esi  
> UPX0:004070BD  
> UPX0:004070BD loc\_4070BD:                             ; CODE XREF: func4+AEj  
> UPX0:004070BD                 lea     edi, dword\_412218\[esi\]  
> UPX0:004070C3                 lea     eax, \[edi-4\]  
> UPX0:004070C6                 push    eax  
> UPX0:004070C7                 mov     \[esp+1Ch+var\_4\], eax  
> UPX0:004070CB                 call    func2  
> UPX0:004070D0                 push    edi  
> UPX0:004070D1                 mov     esi, eax  
> UPX0:004070D3                 call    func2  
> UPX0:004070D8                 mov     ecx, dword\_40EF5C  
> UPX0:004070DE                 mov     edi, eax  
> UPX0:004070E0                 mov     eax, dword\_40EF58  
> UPX0:004070E5                 add     esp, 8  
> UPX0:004070E8                 add     edi, ecx  
> UPX0:004070EA                 mov     edx, offset dword\_40EF60  
> UPX0:004070EF                 add     esi, eax  
> UPX0:004070F1                 mov     ebx, 10  
> UPX0:004070F6  
> UPX0:004070F6 loc\_4070F6:                             ; CODE XREF: func4+85j  
> UPX0:004070F6                 mov     eax, edi  
> UPX0:004070F8                 xor     esi, edi  
> UPX0:004070FA                 and     eax, 31  
> UPX0:004070FD                 mov     ecx, 32  
> UPX0:00407102                 sub     ecx, eax  
> UPX0:00407104                 mov     ebp, esi  
> UPX0:00407106                 shr     ebp, cl  
> UPX0:00407108                 mov     ecx, eax  
> UPX0:0040710A                 mov     eax, \[edx\]  
> UPX0:0040710C                 shl     esi, cl  
> UPX0:0040710E                 mov     ecx, 32  
> UPX0:00407113                 add     edx, 8  
> UPX0:00407116                 or      ebp, esi  
> UPX0:00407118                 add     ebp, eax  
> UPX0:0040711A                 mov     esi, ebp  
> UPX0:0040711C                 mov     eax, esi  
> UPX0:0040711E                 xor     edi, esi  
> UPX0:00407120                 and     eax, 31  
> UPX0:00407123                 mov     ebp, edi  
> UPX0:00407125                 sub     ecx, eax  
> UPX0:00407127                 shr     ebp, cl  
> UPX0:00407129                 mov     ecx, eax  
> UPX0:0040712B                 shl     edi, cl  
> UPX0:0040712D                 or      ebp, edi  
> UPX0:0040712F                 mov     edi, \[edx-4\]  
> UPX0:00407132                 add     edi, ebp  
> UPX0:00407134                 dec     ebx  
> UPX0:00407135                 jnz     short loc\_4070F6  
> UPX0:00407137                 mov     eax, \[esp+18h+var\_4\]  
> UPX0:0040713B                 push    eax  
> UPX0:0040713C                 push    esi  
> UPX0:0040713D                 call    func3  
> UPX0:00407142                 mov     esi, \[esp+20h+var\_8\]  
> UPX0:00407146                 lea     eax, dword\_412218\[esi\]  
> UPX0:0040714C                 push    eax  
> UPX0:0040714D                 push    edi  
> UPX0:0040714E                 call    func3  
> UPX0:00407153                 add     esp, 10h  
> UPX0:00407156                 inc     esi  
> UPX0:00407157                 cmp     esi, 78  
> UPX0:0040715A                 mov     \[esp+18h+var\_8\], esi  
> UPX0:0040715E                 jbe     loc\_4070BD  
> UPX0:00407164                 pop     edi  
> UPX0:00407165                 pop     esi  
> UPX0:00407166                 pop     ebp  
> UPX0:00407167                 pop     ebx  
> UPX0:00407168                 add     esp, 8  
> UPX0:0040716B                 retn  
> UPX0:0040716B func4           endp  
>   
> \--------------------------------------------------------------------------------  
>   
> void func4()  
> {  
>   unsigned int  i, j, k;  
>   unsigned long  data\[22\] = { 0x5F9C979BUL, 0xE5DBF044UL,  
>     0x11E7ABDAUL, 0x6550DA97UL, 0xBD21EB0DUL, 0x90E30E59UL, 0x90C7D548UL,   
>     0x7EEBE540UL, 0xBE8C93F3UL, 0x4A01992FUL, 0xBAB151A7UL, 0x347ABF9BUL,   
>     0x66B4E1F0UL, 0x23A7B353UL, 0x03BC780CUL, 0xF89C1689UL, 0xBAC1F8A6UL,   
>     0xE16BBD31UL, 0xE862289FUL, 0x6F783C83UL, 0xBE85A948UL, 0x2303D5DDUL  
>   };  
>   unsigned char \*p;  
>   unsigned long  esi, edi, eax, ecx, ebp;  
>   
>   p = szBuf;  
>   for( i=0; i<=78; i++ )  
>   {  
>     esi = func2(p+i) + data\[0\];  
>     edi = func2(p+4+i) + data\[1\];  
>     for( j=10; j>0; j-- )  
>     {  
>       esi ^= edi;  
>       eax = edi & 31;  
>       ecx = 32 - eax;  
>       ebp = esi>>ecx;  
>       esi = esi<<eax;  
>       k = (10-j)\*2+2;  
>       esi = (ebp|esi) + data\[k\];  
>   
>       edi ^= esi;  
>       eax = esi & 31;  
>       ecx = 32 - eax;  
>       ebp = edi>>ecx;  
>       edi = edi<<eax;  
>       k = (10-j)\*2+3;  
>       edi = (ebp|edi) + data\[k\];  
>     }  
>     func3( p+i, esi );  
>     func3( p+4+i, edi );  
>   }  
> }  
>   
> \--------------------------------------------------------------------------------  
>   
> 这是 func5()的反汇编代码，作用是将连续 2 个字节转化为一个 32 位的无符号整数。  
>   
> UPX0:004070A0 func5           proc near               ; CODE XREF: GenerateKey+70p  
> UPX0:004070A0  
> UPX0:004070A0 arg\_0           = dword ptr  4  
> UPX0:004070A0  
> UPX0:004070A0                 mov     eax, \[esp+arg\_0\]  
> UPX0:004070A4                 xor     ecx, ecx  
> UPX0:004070A6                 mov     ch, \[eax\]  
> UPX0:004070A8                 mov     cl, \[eax+1\]  
> UPX0:004070AB                 mov     eax, ecx  
> UPX0:004070AD                 retn  
> UPX0:004070AD func5           endp  
>   
> \--------------------------------------------------------------------------------  
>   
> unsigned int func5( unsigned char \*p )  
> {  
>   unsigned int  t = 0;  
>   t = \*p;  
>   t = (t<<8) + \*(p+1);  
>   return t;  
> }  
>   
> \--------------------------------------------------------------------------------  
>   
> 分析完这个 keygen 以后，我顺便又看了看其它的两个 keygen。我发现 v7.01 和 v7.1 的 keygen  
> 其实是一样的，而 v7.0 的 keygen 算法也差不多，相比之下更简单，生成的 key 也要短一截。  
>   
> 至于更新一些的 Matlab，比如 R2006b，TEAM ACME 曾经出过一个 keygen，不过加了壳，暂时还  
> 没有搞定。依 key 的格式推断，注册算法应该差不多。  
>   
> 最新的 R2007a 也出来了，不过只有一个现成的 key，没见过 keygen，期待哪位高手试试，  
> 我等菜鸟只能在旁边看热闹了，呵呵~~  
>   
>   
> 逆向出来的注册机源码见附件,已调试通过。  
>   
> Just for fun ! 

 4dd466d0e1e0cc123afbe504da2462ca