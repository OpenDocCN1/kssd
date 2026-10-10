

# WinRAR 3.62

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-510.htm`](https://www.pediy.com/kssd/pediy08/pediy8-510.htm)

WinRAR 3.62 注册算法 - CRC 校验  
  
\[声明\] 本文为逆向学习参考文章, 若使用 winrar 请向作者注册.  
  
用 WinRAR3.51 的 key 文件 rarreg.key 发现被加入了黑名单, 所以分析了一下.  
  
3.51 注册 key 内容:  
RAR registration data  
Carol Thompson  
Single PC usage license  
UID=b8bc6fb0a8094b9eeb29  
6412212250eb294bd5b605e535f7334b6e2e56a9e405a044f60225  
c843a161a156aa01684c6035c6ab9048e2c5c62f0238f183d28519  
aa87488bf38f5b634cf28190bdf438ac593b1857cdb55a7fcb0eb0  
c3e4c2736090b3dfa45384e08e9de05c5860ae8049eaa9443b44f9  
faac06b7ced5f95ab06b40a99e850616dc92fc5301fe63c674ea55  
3971fefd9e10f300d2a515c74b02f673b7fe5a89fa92f51260a5af  
78a306093f5763d6acc779488f5d42e9b044836a837c0424153795  
  
1. CRC 校验  
  
使用 CreateFileA 函数断点, 跟踪对文件 rarreg.key 文件的读取, 到以下代码:  
  
  
.text:0040E6C4 check\_regdata   proc near   
.text:0040E6C4                 push    ebx  
.text:0040E6C5                 push    esi  
.text:0040E6C6                 push    edi  
.text:0040E6C7                 push    ebp  
.text:0040E6C8                 add     esp, 0FFFFF178h ; data  
.text:0040E6CE                 mov     edi, eax  
.text:0040E6D0                 or      eax, 0FFFFFFFFh ; crc32  
.text:0040E6D3                 mov     esi, offset regData  
.text:0040E6D8                 mov     \[esp+0E88h+var\_E88\], dl  
.text:0040E6DB                 mov     edx, offset g\_HCode0 ; "70c2441db366d92ea7be1342b3bf629026ba92b"...  
.text:0040E6E0                 mov     ecx, 40h        ; len  
.text:0040E6E5                 call    NCrc32  
.text:0040E6E5  
.text:0040E6EA                 cmp     eax, 26E831B8h  
.text:0040E6EF                 jz      short HCODE0\_CRC\_OK ; \[Check1\] 如果硬编码串的 CRC32 出错, 注册失败!  
.text:0040E6EF  
.text:0040E6F1                 xor     eax, eax  
.text:0040E6F3                 jmp     REG\_FAIL\_EXIT  
  
首先出现了一个硬编码的字符串  
g\_HCode0 = "70c2441db366d92ea7be1342b3bf629026ba92bb675f06e684bdd34511097434"  
  
  
然后调用 NCrc32 计算 g\_HCode0 的 ~CRC32 值, 比较此值是否等于 0x26E831B8, 这里注册成功  
的关键是 WinRAR 未修改 g\_HCode0 的值.  
  
  
也许你已经发现为什么函数的名字前面加了一个 N, 其实查看此函数就知道, 这里计算出的结  
果并非 CRC32 值, 而是 CRC32 的非, 代码如下  
  
.text:00410734 ; DWORD \_\_fastcall NCrc32(DWORD crc32,unsigned \_\_int8 \*buffer,DWORD len)  
.text:00410734 NCrc32          proc near    
.text:00410734                 push    ebx  
.text:00410735                 push    esi  
.text:00410736                 push    edi  
.text:00410737                 mov     edi, offset crc32\_table  
.text:0041073C                 push    ebp  
.text:0041073D                 mov     ebp, edx  
.text:0041073F                 cmp     dword ptr \[edi+4\], 0  
.text:00410743                 mov     esi, ecx  
.text:00410745                 mov     ebx, eax  
.text:00410747                 jnz     short CRC32TABLE\_OK ; 如果 CRC32 表的第 2 个 dword 值为 0,  
.text:00410747                                         ; 则重新构建 CRC32 表 crc32\_table  
.text:00410747  
.text:00410749                 call    makeCrc32Table  ; 构建 CRC32 表  
.text:0041074E  
.text:0041074E CRC32TABLE\_OK:                          ; CODE XREF: NCrc32+13j  
.text:0041074E                 mov     eax, ebp  
.text:00410750                 jmp     short FOR1\_condition ; FOR1 的条件, buffer 未处理部分长度大于 0,  
.text:00410750                                         ; 且 buffer 地址 8 字节未对齐, 则继续循环  
.text:00410752 ; ---------------------------------------------------------------------------  
.text:00410752 FOR1:                                   ; CODE XREF: NCrc32+38j  
.text:00410752                 mov     edx, ebx        ; 第一个 for 循环处理 buffer 地址未对齐部分  
.text:00410754                 xor     ecx, ecx  
.text:00410756                 xor     dl, \[eax\]  
.text:00410758                 dec     esi  
.text:00410759                 mov     cl, dl  
.text:0041075B                 shr     ebx, 8  
.text:0041075E                 mov     edx, \[edi+ecx\*4\]  
.text:00410761                 xor     edx, ebx  
.text:00410763                 inc     eax  
.text:00410764                 mov     ebx, edx  
.text:00410766  
.text:00410766 FOR1\_condition:                         ; CODE XREF: NCrc32+1Cj  
.text:00410766                 test    esi, esi        ; FOR1 的条件, buffer 未处理部分长度大于 0,  
.text:00410766                                         ; 且 buffer 地址 8 字节未对齐, 则继续循环  
.text:00410768                 jbe     short loc\_41076E  
.text:00410768  
.text:0041076A                 test    al, 7  
.text:0041076C                 jnz     short FOR1      ; 第一个 for 循环处理 buffer 地址未对齐部分  
.text:0041076E  
.text:0041076E loc\_41076E:                             ; CODE XREF: NCrc32+34j  
.text:0041076E                 cmp     esi, 8  
.text:00410771                 jb      loc\_4107F7      ; 判断 buffer 未处理部分的长度是否大于 8  
.text:00410771  
.text:00410777 loc\_410777:                             ; CODE XREF: NCrc32+C1j  
.text:00410777                 xor     ebx, \[eax\]      ; 每次循环计算 8 个字节  
.text:00410779                 xor     ecx, ecx  
.text:0041077B                 mov     cl, bl  
.text:0041077D                 sub     esi, 8  
.text:00410780                 shr     ebx, 8  
.text:00410783                 mov     edx, \[edi+ecx\*4\]  
.text:00410786                 xor     edx, ebx  
.text:00410788                 xor     ecx, ecx  
.text:0041078A                 mov     ebx, edx  
.text:0041078C                 mov     cl, bl  
.text:0041078E                 shr     ebx, 8  
.text:00410791                 mov     edx, \[edi+ecx\*4\]  
.text:00410794                 xor     edx, ebx  
.text:00410796                 xor     ecx, ecx  
.text:00410798                 mov     ebx, edx  
.text:0041079A                 mov     cl, bl  
.text:0041079C                 shr     ebx, 8  
.text:0041079F                 mov     edx, \[edi+ecx\*4\]  
.text:004107A2                 xor     edx, ebx  
.text:004107A4                 xor     ecx, ecx  
.text:004107A6                 mov     ebx, edx  
.text:004107A8                 mov     cl, bl  
.text:004107AA                 shr     ebx, 8  
.text:004107AD                 mov     edx, \[edi+ecx\*4\]  
.text:004107B0                 xor     edx, ebx  
.text:004107B2                 xor     ecx, ecx  
.text:004107B4                 mov     ebx, edx  
.text:004107B6                 xor     ebx, \[eax+4\]  
.text:004107B9                 add     eax, 8  
.text:004107BC                 mov     cl, bl  
.text:004107BE                 shr     ebx, 8  
.text:004107C1                 mov     edx, \[edi+ecx\*4\]  
.text:004107C4                 xor     edx, ebx  
.text:004107C6                 xor     ecx, ecx  
.text:004107C8                 mov     ebx, edx  
.text:004107CA                 mov     cl, bl  
.text:004107CC                 shr     ebx, 8  
.text:004107CF                 mov     edx, \[edi+ecx\*4\]  
.text:004107D2                 xor     edx, ebx  
.text:004107D4                 xor     ecx, ecx  
.text:004107D6                 mov     ebx, edx  
.text:004107D8                 mov     cl, bl  
.text:004107DA                 shr     ebx, 8  
.text:004107DD                 mov     edx, \[edi+ecx\*4\]  
.text:004107E0                 xor     edx, ebx  
.text:004107E2                 xor     ecx, ecx  
.text:004107E4                 mov     ebx, edx  
.text:004107E6                 mov     cl, bl  
.text:004107E8                 shr     ebx, 8  
.text:004107EB                 mov     edx, \[edi+ecx\*4\]  
.text:004107EE                 xor     edx, ebx  
.text:004107F0                 cmp     esi, 8  
.text:004107F3                 mov     ebx, edx  
.text:004107F5                 jnb     short loc\_410777 ; 每次循环计算 8 个字节  
.text:004107F5  
.text:004107F7 loc\_4107F7:                             ; CODE XREF: NCrc32+3Dj  
.text:004107F7                 xor     edx, edx  
.text:004107F9                 jmp     short loc\_410811  
.text:004107FB ; ---------------------------------------------------------------------------  
.text:004107FB loc\_4107FB:                             ; CODE XREF: NCrc32+DFj  
.text:004107FB                 mov     ecx, ebx        ; 处理剩余部分  
.text:004107FD                 xor     cl, \[eax+edx\]  
.text:00410800                 and     ecx, 0FFh  
.text:00410806                 shr     ebx, 8  
.text:00410809                 mov     ecx, \[edi+ecx\*4\]  
.text:0041080C                 xor     ecx, ebx  
.text:0041080E                 mov     ebx, ecx  
.text:00410810                 inc     edx  
.text:00410811  
.text:00410811 loc\_410811:                             ; CODE XREF: NCrc32+C5j  
.text:00410811                 cmp     esi, edx  
.text:00410813                 ja      short loc\_4107FB ; 处理剩余部分  
.text:00410815                 mov     eax, ebx  
.text:00410817                 pop     ebp  
.text:00410818                 pop     edi  
.text:00410819                 pop     esi  
.text:0041081A                 pop     ebx  
.text:0041081B                 retn  
.text:0041081B  
.text:0041081B NCrc32          endp  
  
用 C 简单实现了一下:  
  
unsigned int   
NCrc32 (unsigned int crc32, unsigned char \*buff, unsigned int len)  
{  
        if (crc32Table\[1\] == 0)  
                makeCrc32Table();  
  
        for ( ; (len>0) && (((unsigned int)buff & 7)!=0); --len) // 8 字节对齐  
        {  
                crc32 = crc32Table\[(crc32 ^ \*buff++) & 0xFF\] ^ (crc32>>8);  
        }  
        // 每次计算 8 个字节  
        while (len >8)  
        {  
                crc32 ^= \*(unsigned int\*)buff;  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
  
                crc32 ^= \*(unsigned int\*)(buff+4);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
                crc32 = crc32Table\[crc32 & 0xFF\] ^(crc32>>8);  
          
                len -= 8;  
                buff += 8;  
        }  
        // 计算剩余部分  
        for (unsigned int i=0; len>i; i++)  
        {  
                crc32 = crc32Table\[(crc32 ^ buff\[i\]) & 0xFF\] ^ (crc32>>8);  
        }  
        return crc32;  
}  
  
下面的代码用到一个结构 REG\_DATA:  
  
typedef struct \_reg\_data  
{  
        char usrname\[256\];  
        char licType\[256\];  
        char snCode1\[256\];  
        char snCode4\[256\];  
        char snCode3\[256\];  
        char snCode2\[256\];  
        u\_int32\_t crc32;  
}REG\_DATA;  
  
  
Proxies::CreateSubClass 不知道做什么的, 大概是构造子串的, 既然是库函数直接看结果了,  
结果在 edx 指向的内存产生一个字符串, 每次都是 rarreg.key 文件的一行(在这之前文件的内  
容已经读到内存了, 读文件部分被略了:)).  
  
  
.text:0040E6F8 HCODE0\_CRC\_OK:   
.text:0040E6F8                 push    604h            ; len  
.text:0040E6FD                 push    0               ; c  
.text:0040E6FF                 push    regData         ; REG\_DATA  
.text:0040E700                 call    \_memset               ; 结构清零  
.text:0040E700  
.text:0040E705                 add     esp, 0Ch  
.text:0040E708                 jmp     short loc\_40E715  
.text:0040E70A ; ---------------------------------------------------------------------------  
.text:0040E70A loc\_40E70A:                             ; CODE XREF: check\_regdata+63j  
.text:0040E70A                 xor     edx, edx  
.text:0040E70C                 mov     dl, byte ptr \[esp+0E88h+strTemp\]  
.text:0040E710                 cmp     edx, 23h  
.text:0040E713                 jnz     short loc\_40E729  
.text:0040E715 loc\_40E715:                             ; CODE XREF: check\_regdata+44j  
.text:0040E715                 lea     edx, \[esp+0E88h+strTemp\]  
.text:0040E719                 mov     ecx, 400h  
.text:0040E71E                 mov     eax, edi    ;   
.text:0040E720                 call    Proxies::CreateSubClass(System::TObject \*,System::AnsiString,System::TMetaClass \*)  
.text:0040E725                 test    al, al  
.text:0040E727                 jnz     short loc\_40E70A  
.text:0040E729  
.text:0040E729 loc\_40E729:                             ; CODE XREF: check\_regdata+4Fj  
.text:0040E729                 mov     edx, regData  
.text:0040E72B                 mov     ecx, 100h  
.text:0040E730                 mov     eax, edi  
.text:0040E732                 call    Proxies::CreateSubClass(System::TObject \*,System::AnsiString,System::TMetaClass \*)  
.text:0040E737                 lea     edx, \[regData+REG\_DATA.licType\]  
.text:0040E73D                 mov     ecx, 100h  
.text:0040E742                 mov     eax, edi  
.text:0040E744                 call    Proxies::CreateSubClass(System::TObject \*,System::AnsiString,System::TMetaClass \*)  
.text:0040E749                 lea     edx, \[esp+0E88h+regUid\]  
.text:0040E750                 mov     ecx, 80h  
.text:0040E755                 mov     eax, edi  
.text:0040E757                 call    Proxies::CreateSubClass(System::TObject \*,System::AnsiString,System::TMetaClass \*)  
.text:0040E75C                 xor     ebx, ebx  
.text:0040E75E  
.text:0040E75E loc\_40E75E:                             ; CODE XREF: check\_regdata+B7j  
.text:0040E75E                 imul    eax, ebx, 36h  
.text:0040E761                 mov     ecx, 400h  
.text:0040E766                 mov     edx, eax  
.text:0040E768                 sub     ecx, eax  
.text:0040E76A                 lea     eax, \[esp+0E88h+strTemp\]  
.text:0040E76E                 add     edx, eax  
.text:0040E770                 mov     eax, edi  
.text:0040E772                 call    Proxies::CreateSubClass(System::TObject \*,System::AnsiString,System::TMetaClass \*)  
.text:0040E777                 inc     ebx  
.text:0040E778                 cmp     ebx, 7  
.text:0040E77B                 jl      short loc\_40E75E    ;把 UID 行以后的, 当一串字符串(去掉 CR/LF)  
  
  
按字符串的前 10 位(snHead="6412212250" )分割字符串  
.text:0040E77D                 lea     eax, \[esp+0E88h+strTemp\]  
.text:0040E781                 mov     edx, 2  
.text:0040E786                 call    rar\_atoi      ;rar 实现的把指定长度的字符串转换位整型  
.text:0040E78B                 mov     \[esp+0E88h+snCode1\_len\], eax  
.text:0040E78F                 lea     eax, \[esp+0E88h+strTemp+2\]  
.text:0040E793                 mov     edx, 3  
.text:0040E798                 call    rar\_atoi  
.text:0040E79D                 mov     edi, eax  
.text:0040E79F                 lea     eax, \[esp+0E88h+strTemp+5\]  
.text:0040E7A3                 mov     edx, 3  
.text:0040E7A8                 call    rar\_atoi  
.text:0040E7AD                 mov     ebp, eax  
.text:0040E7AF                 lea     eax, \[esp+0E88h+strTemp+8\]  
.text:0040E7B3                 mov     edx, 2  
.text:0040E7B8                 call    rar\_atoi  
.text:0040E7BD                 mov     \[esp+0E88h+snCode4\_len\], eax  
.text:0040E7C1                 cmp     edi, 100h       ; \[Check2\] 比较\[注册字符串 2\]的长度  
.text:0040E7C7                 jg      short loc\_40E7D1 ; \[Check2\]  
.text:0040E7C7  
.text:0040E7C9                 cmp     ebp, 100h       ; \[Check2\] 比较\[注册字符串 3\]的长度  
.text:0040E7CF                 jle     short loc\_40E7D8  
.text:0040E7D1  
.text:0040E7D1 loc\_40E7D1:                             ; CODE XREF: check\_regdata+103j  
.text:0040E7D1                 xor     eax, eax        ; \[Check2\]  
.text:0040E7D3                 jmp     REG\_FAIL\_EXIT   ; 如果 \[注册字符串 2\], \[注册字符串 3\] 有一个长度大于 256,  
.text:0040E7D3                                         ; 注册失败!  
.text:0040E7D8 ; ---------------------------------------------------------------------------  
  
\[跳过一段代码, 这段代码是检查黑名单, 稍后讲\]  
  
  
下面这部分是按照 snHead="6412212250", 把字符串分割后拷贝到 regData 结构中, 分割方法  
就是按上面的 2, 3, 3, 2, 即 64, 122, 122, 50(十进制)长分割.  
  
  
.text:0040E882 loc\_40E882:                             ; CODE XREF: check\_regdata+1A5j  
.text:0040E882                                         ; check\_regdata+1B5j  
.text:0040E882                 inc     ebx  
.text:0040E883                 cmp     ebx, 6  
.text:0040E886                 jl      short NEXT\_UBLACK  
.text:0040E888                 lea     ebx, \[esp+0E88h+strTemp+0Ah\]  
.text:0040E88C                 push    \[esp+0E88h+snCode1\_len\] ; snCode1\_len  
.text:0040E890                 push    ebx             ; src  
.text:0040E891                 lea     eax, \[regData+REG\_DATA.snCode1\]  
.text:0040E897                 push    eax             ; dest  
.text:0040E898                 call    \_strncpy  
.text:0040E89D                 add     esp, 0Ch  
.text:0040E8A0                 add     ebx, \[esp+0E88h+snCode1\_len\]  
.text:0040E8A4                 push    edi             ; snCode2\_len  
.text:0040E8A5                 push    ebx             ; src  
.text:0040E8A6                 lea     edx, \[regData+REG\_DATA.snCode2\]  
.text:0040E8AC                 push    edx             ; dest  
.text:0040E8AD                 call    \_strncpy  
.text:0040E8B2                 add     esp, 0Ch  
.text:0040E8B5                 add     ebx, edi  
.text:0040E8B7                 push    ebp             ; snCode3\_len  
.text:0040E8B8                 push    ebx             ; src  
.text:0040E8B9                 lea     ecx, \[regData+REG\_DATA.snCode3\]  
.text:0040E8BF                 push    ecx             ; dest  
.text:0040E8C0                 call    \_strncpy  
.text:0040E8C5                 add     esp, 0Ch  
.text:0040E8C8                 add     ebx, ebp  
.text:0040E8CA                 push    \[esp+0E88h+snCode4\_len\] ; maxlen  
.text:0040E8CE                 push    ebx             ; src  
.text:0040E8CF                 lea     eax, \[regData+REG\_DATA.snCode4\]  
.text:0040E8D5                 push    eax             ; dest  
.text:0040E8D6                 call    \_strncpy  
.text:0040E8DB                 add     esp, 0Ch  
.text:0040E8DE                 add     ebx, \[esp+0E88h+snCode4\_len\]  
.text:0040E8E2                 push    0Ah             ; radix  
.text:0040E8E4                 push    0               ; endptr  
.text:0040E8E6                 push    ebx             ; s  
.text:0040E8E7                 call    \_strtoul        ; 把最后 10 位转化作为 CRC32  
.text:0040E8EC                 add     esp, 0Ch  
.text:0040E8EF                 mov     \[regData+REG\_DATA.crc32\], eax  
  
分割后各部分数据如下:  
  
regHead          = "RAR registration data"  
regData.username = "Carol Thompson"    //   
regData.licType  = "Single PC usage license"  // regData.licType 注册类型  
regUid           = "UID=b8bc6fb0a8094b9eeb29"  //   
snHead           = "6412212250"      // \[注册字符串头\] len = 10  
regData.snCode1   = "eb294bd5b605e535f7334b6e2e56a9e405a044f60225c843a161a156aa01684c" // \[注册字符串 1\] len = 64  
regData.snCode2  = "6035c6ab9048e2c5c62f0238f183d28519aa87488bf38f5b634cf28190bdf438ac593b1857cdb55a7fcb0eb0c3e4c2736090b3dfa45384e08e9de05c58" // \[注册字符串 2\] len = 122   
regData.snCode3  = "60ae8049eaa9443b44f9faac06b7ced5f95ab06b40a99e850616dc92fc5301fe63c674ea553971fefd9e10f300d2a515c74b02f673b7fe5a89fa92f512" // \[注册字符串 3\] len = 122   
regData.snCode4  = "60a5af78a306093f5763d6acc779488f5d42e9b044836a837c", // \[注册字符串 4\] len = 50  
regData.crc32    = "0424153795"; // \[头和 1,2,3 部分的 NCRC32 值(10 进制)\] len = 10  
  
  
下面代码是计算 regData.licType + username + regData.snCode1 + .snCode2 + .snCode3 + .snCode4  
的~CRC32 值, 判断是否等于 regData.crc32, 不等注册失败!  
  
.text:0040E8F5                 lea     edx, \[regData+REG\_DATA.licType\]  
.text:0040E8FB                 push    edx             ; s  
.text:0040E8FC                 call    \_strlen  
.text:0040E901                 pop     ecx  
.text:0040E902                 mov     ecx, eax        ; len  
.text:0040E904                 lea     edx, \[regData+REG\_DATA.licType\] ; buffer  
.text:0040E90A                 or      eax, 0FFFFFFFFh ; crc32  
.text:0040E90D                 call    NCrc32  
.text:0040E912                 mov     ebx, eax  
.text:0040E914                 push    regData         ; s  
.text:0040E915                 call    \_strlen  
.text:0040E91A                 pop     ecx  
.text:0040E91B                 mov     ecx, eax        ; len  
.text:0040E91D                 mov     edx, regData    ; buffer  
.text:0040E91F                 mov     eax, ebx        ; crc32  
.text:0040E921                 call    NCrc32  
.text:0040E926                 mov     ebx, eax  
.text:0040E928                 lea     eax, \[regData+REG\_DATA.snCode1\]  
.text:0040E92E                 push    eax             ; s  
.text:0040E92F                 call    \_strlen  
.text:0040E934                 pop     ecx  
.text:0040E935                 mov     ecx, eax        ; len  
.text:0040E937                 lea     edx, \[regData+REG\_DATA.snCode1\] ; buffer  
.text:0040E93D                 mov     eax, ebx        ; crc32  
.text:0040E93F                 call    NCrc32  
.text:0040E944                 mov     ebx, eax  
.text:0040E946                 lea     edx, \[regData+REG\_DATA.snCode2\]  
.text:0040E94C                 push    edx             ; s  
.text:0040E94D                 call    \_strlen  
.text:0040E952                 pop     ecx  
.text:0040E953                 mov     ecx, eax        ; len  
.text:0040E955                 lea     edx, \[regData+REG\_DATA.snCode2\] ; buffer  
.text:0040E95B                 mov     eax, ebx        ; crc32  
.text:0040E95D                 call    NCrc32  
.text:0040E962                 mov     ebx, eax  
.text:0040E964                 lea     ecx, \[regData+REG\_DATA.snCode3\]  
.text:0040E96A                 push    ecx             ; s  
.text:0040E96B                 call    \_strlen  
.text:0040E970                 pop     ecx  
.text:0040E971                 mov     ecx, eax        ; len  
.text:0040E973                 lea     edx, \[regData+REG\_DATA.snCode3\] ; buffer  
.text:0040E979                 mov     eax, ebx        ; crc32  
.text:0040E97B                 call    NCrc32  
.text:0040E980                 mov     ebx, eax  
.text:0040E982                 lea     eax, \[regData+REG\_DATA.snCode4\]  
.text:0040E988                 push    eax             ; s  
.text:0040E989                 call    \_strlen  
.text:0040E98E                 pop     ecx  
.text:0040E98F                 mov     ecx, eax        ; len  
.text:0040E991                 lea     edx, \[regData+REG\_DATA.snCode4\] ; buffer  
.text:0040E997                 mov     eax, ebx        ; crc32  
.text:0040E999                 call    NCrc32  
.text:0040E99E                 mov     ebx, eax  
.text:0040E9A0                 cmp     ebx, \[regData+REG\_DATA.crc32\] ; \[Check5\] licType + username + snCode1/2/3/4 的~CRC32 值,  
                                                                     ; 和 regData.crc32 比较, 不相等注册失败!  
.text:0040E9A6                 jz      short loc\_40E9AF  
.text:0040E9A8                 xor     eax, eax  
.text:0040E9AA                 jmp     REG\_FAIL\_EXIT  
.text:0040E9AF ; ---------------------------------------------------------------------------  
  
到此基本了解了文件的 rarreg.key 的结构, 下一篇介绍黑名单部分.  

WinRAR 3.62 注册算法 - 黑名单  
  
  
\[声明\] 本文为逆向学习参考文章, 若使用 winrar 请向作者注册.  
  
2. 注册码黑名单  
  
回到上回跳过的黑名单部分:  
  
.text:0040E7D8 loc\_40E7D8:                             ; CODE XREF: check\_regdata+10Bj  
.text:0040E7D8                 xor     ebx, ebx  
.text:0040E7DA NEXT\_BLACK:                             ; CODE XREF: check\_regdata+159j  
.text:0040E7DA                 mov     eax, ebx  
.text:0040E7DC                 lea     edx, \[esp+0E88h+black\_string\]  
.text:0040E7E3                 shl     eax, 3  
.text:0040E7E6                 mov     ecx, 8          ; 单个 black 的长度  
.text:0040E7EB                 add     eax, offset g\_blacklist ; 一个全局的黑名单列表  
.text:0040E7F1                 call    getOneBlack     ; 从黑名单中取 8 个字节, 按 16 进制转化为字符串 black\_string  
.text:0040E7F1  
.text:0040E7F6                 lea     edx, \[esp+0E88h+black\_string\] ; lpString2  
.text:0040E7FD                 lea     eax, \[esp+0E88h+strTemp+0Ah\] ; lpString1  
.text:0040E801                 mov     ecx, 8          ; cchCount2  
.text:0040E806                 call    rar\_strncmp  
.text:0040E80B                 test    eax, eax        ; \[Check3\] 检查 snCode1\[注册字符串 1\] 前 8 位是否在黑名  
                                                       ; 单中,在黑名单注册失败!  
.text:0040E80D                 jnz     short NO\_BLACK  
.text:0040E80F                 xor     eax, eax  
.text:0040E811                 jmp     REG\_FAIL\_EXIT  
.text:0040E816 NO\_BLACK:                               ; CODE XREF: check\_regdata+149j  
.text:0040E816                 inc     ebx  
.text:0040E817                 cmp     ebx, 9Dh        ; 157(0x9D)个黑名单, 每个黑名单占 8 个字节,那么  
                                                       ; g\_blacklist 总共占 157\*8=1256 个字节  
.text:0040E81D                 jl      short NEXT\_BLACK  
  
  
void \_\_fastcall getOneBlack(BYTE \*black,char \*black\_string,int len=8)  
{  
        for (; len>0; len--)  
        {  
                sprintf(black\_string, "%02x", \*black++);  
                black\_string += 2;          
        }  
}  
  
int \_\_fastcall rar\_strncmp(LPCSTR lpString1,LPCSTR lpString2,int cCount)  
{  
        return CompareString(LOCALE\_USER\_DEFAULT,  
                             NORM\_IGNORECASE|SORT\_STRINGSORT,   
                             lpString1,   
                             cCount,  
                             lpString2,  
                             cCount) - CSTR\_EQUAL;  
}  
  
getOneBlack 函数很简单在这里就是每次取 8 个字节, 按 16 进制转化为字符串 black\_string,  
然后又调用 rar\_strncmp 对字符串进行比较, 这里用到了 strTemp+10, 看上一篇可知  
strTemp 存储的是:  
  
6412212250eb294bd5b605e535f7334b6e2e56a9e405a044f60225  
c843a161a156aa01684c6035c6ab9048e2c5c62f0238f183d28519  
aa87488bf38f5b634cf28190bdf438ac593b1857cdb55a7fcb0eb0  
c3e4c2736090b3dfa45384e08e9de05c5860ae8049eaa9443b44f9  
faac06b7ced5f95ab06b40a99e850616dc92fc5301fe63c674ea55  
3971fefd9e10f300d2a515c74b02f673b7fe5a89fa92f51260a5af  
78a306093f5763d6acc779488f5d42e9b044836a837c0424153795  
  
那么跳过 10 个字节后就是 snCode1, 然后调用 rar\_strncmp 比较 snCode1 的前 8 位和黑名单返回  
16 位字符串的前 8 位, 返回 0 表示相等也就注册失败了.  
  
看一下黑名单都有那些:  
  
.data:004A012C g\_blacklist     db  6Ch,   4, 0Ch, 9Dh, 3Bh, 41h, 9Ah, 18h; 0  
.data:004A012C                 db  1Dh, 29h, 3Eh,0D1h, 42h,0AFh, 12h, 2Bh; 8  
.data:004A012C                 db    9,0B6h,0C1h,0A2h,0F4h,   7,0C7h, 75h; 16  
.data:004A012C                 db 0C4h, 7Fh,0A7h,0D1h, 85h,0A5h,0DAh,0FCh; 24  
.data:004A012C                 db 0A2h,0BAh, 6Ch,0FDh, 60h,0D8h, 69h, 0Eh; 32  
.data:004A012C                 db  89h, 14h,0D2h,0EAh, 34h, 56h, 8Bh, 68h; 40  
.data:004A012C                 db  30h,0FEh,0C8h, 17h,   6,0E2h, 5Ch,0B1h; 48  
.data:004A012C                 db  39h, 74h, 75h, 40h, 4Bh, 56h, 6Bh,0C2h; 56  
.data:004A012C                 db  71h, 26h,0B3h, 24h, 0Fh,0D7h,   5,0ACh; 64  
.data:004A012C                 db 0F8h,0EFh, 97h, 1Ch,0D3h,0A6h, 8Eh,0BDh; 72  
.data:004A012C                 db  10h, 56h,0F5h, 60h,   6, 53h, 4Bh,0C8h; 80  
.data:004A012C                 db  68h, 26h, 54h, 92h, 31h, 68h, 18h, 85h; 88  
.data:004A012C                 db  6Dh,   1, 5Dh, 59h, 82h,0D4h,0E1h,0B6h; 96  
.data:004A012C                 db 0F9h,0C2h,0AAh, 70h, 17h, 24h, 82h, 9Ah; 104  
.data:004A012C                 db  48h, 6Ch, 45h,0ABh, 74h, 3Dh, 84h,0E0h; 112  
.data:004A012C                 db    7, 47h, 65h,0F3h,0F7h, 37h,0AFh,0B1h; 120  
.data:004A012C                 db 0ABh,0FDh, 70h, 24h, 64h,0FEh, 55h, 22h; 128  
.data:004A012C                 db  33h,0D0h,0FAh, 0Eh, 3Fh, 3Dh,0A1h, 74h; 136  
.data:004A012C                 db  25h, 96h, 8Fh, 67h,0A2h, 6Dh, 9Dh, 4Ah; 144  
.data:004A012C                 db  62h, 2Ah, 23h, 47h, 78h, 5Bh, 1Ch, 71h; 152  
.data:004A012C                 db  51h, 0Ah,0FCh, 94h,0E2h, 6Dh, 74h,0EAh; 160  
.data:004A012C                 db 0C6h,   4, 8Ah, 7Ah, 32h, 8Eh, 3Eh, 67h; 168  
.data:004A012C                 db  0Fh, 3Eh, 35h,0CAh,0DEh,0B1h, 4Eh,0D6h; 176  
.data:004A012C                 db 0E1h, 76h,0D4h,0B2h,0BBh, 99h, 69h,0AAh; 184  
.data:004A012C                 db  99h, 8Eh, 5Fh,0B5h,   5, 2Eh,0A4h, 55h; 192  
.data:004A012C                 db  6Fh,0C6h, 58h, 39h, 34h,0A1h,0FDh,0EAh; 200  
.data:004A012C                 db  34h, 99h, 0Bh,0F0h, 3Eh, 2Ch, 1Eh,   7; 208  
.data:004A012C                 db  1Dh, 89h, 82h,0BAh,0F0h,0DDh, 2Bh,0BBh; 216  
.data:004A012C                 db 0ADh, 0Dh, 2Bh, 61h, 9Eh, 40h, 28h, 95h; 224  
.data:004A012C                 db  95h, 51h, 20h,0F3h,0F5h, 39h, 40h,0D8h; 232  
.data:004A012C                 db 0D1h,0A1h, 18h,0E3h, 84h,0AEh, 19h, 96h; 240  
.data:004A012C                 db 0F1h, 17h, 3Eh,0EAh, 71h, 5Ah,   4,0D1h; 248  
.data:004A012C                 db  7Ah,   2, 7Dh,0AAh, 8Eh,0EAh, 3Dh,0EEh; 256  
.data:004A012C                 db 0A8h,   0, 3Ah, 7Dh,0F5h, 30h, 99h,0B9h; 264  
.data:004A012C                 db 0AFh, 67h, 46h,0CEh, 78h, 0Eh, 38h, 1Eh; 272  
.data:004A012C                 db 0C5h,0CDh, 3Ch, 5Fh, 65h, 96h, 96h,0F8h; 280  
.data:004A012C                 db  46h, 30h, 23h, 79h,0AFh,   0, 29h, 4Ch; 288  
.data:004A012C                 db  9Dh,0D5h,0A1h, 81h, 26h,0FAh, 74h, 27h; 296  
.data:004A012C                 db  71h,0FAh,0A9h,0A9h,0A3h, 64h,0A1h, 66h; 304  
.data:004A012C                 db  62h,0D4h, 96h, 95h, 13h, 36h,0D3h, 37h; 312  
.data:004A012C                 db  2Ah, 8Bh, 50h, 7Ah, 41h,0B9h,0A8h,0A5h; 320  
.data:004A012C                 db 0E3h, 40h,0AAh,0EDh,0FBh,0B2h,0ABh, 68h; 328  
.data:004A012C                 db 0BEh, 80h, 9Fh, 38h,   4, 9Eh,0D3h,0BFh; 336  
.data:004A012C                 db  86h, 78h, 9Ch, 3Ah, 9Eh,0E7h,0C7h,0F2h; 344  
.data:004A012C                 db  6Ch, 99h,0F6h, 90h,0A7h,0F4h,   0,0B2h; 352  
.data:004A012C                 db  57h, 70h,0AEh, 56h, 8Ch,0B9h,0C6h, 5Eh; 360  
.data:004A012C                 db 0D4h, 29h, 63h, 39h,   2, 0Fh, 5Ch,0DAh; 368  
.data:004A012C                 db 0AEh, 3Dh, 2Ah, 72h,0F9h,0EDh,0E8h,0B1h; 376  
.data:004A012C                 db 0B8h,0C4h,0CCh,0FAh, 89h,0EEh, 80h,0D7h; 384  
.data:004A012C                 db 0BEh, 2Dh,0E6h, 78h, 3Ch,0CDh, 64h, 28h; 392  
.data:004A012C                 db  9Fh, 1Ch, 69h,0E8h, 62h, 6Dh, 32h, 2Bh; 400  
.data:004A012C                 db 0B0h, 20h, 80h, 6Fh,0D3h, 16h, 51h, 87h; 408  
.data:004A012C                 db  12h, 8Ah,0EBh, 39h,0FFh,0C5h, 30h, 9Bh; 416  
.data:004A012C                 db  0Eh, 50h, 40h,0C2h,0CFh,0EDh,0DDh,0F4h; 424  
.data:004A012C                 db 0D9h, 83h, 32h, 45h, 0Ch,0CAh,0D8h,0F7h; 432  
.data:004A012C                 db 0CEh,0D0h,   9, 79h, 33h,0B6h, 26h, 7Ch; 440  
.data:004A012C                 db 0DCh,0C0h, 1Eh, 50h, 67h, 83h, 33h,0FDh; 448  
.data:004A012C                 db 0B8h,   2, 84h, 8Bh, 5Fh,   9,0B0h, 33h; 456  
.data:004A012C                 db 0D2h,   8,0D0h,   5,   6, 90h,   3, 5Fh; 464  
.data:004A012C                 db  48h, 60h,0A4h, 8Dh, 73h,0F3h,0A8h, 8Fh; 472  
.data:004A012C                 db 0ACh, 4Ch,0FEh, 7Eh, 29h,0A9h, 44h,0E0h; 480  
.data:004A012C                 db 0B4h, 9Dh, 3Eh, 65h,0DBh,0B7h,0F4h,0C9h; 488  
.data:004A012C                 db  26h, 1Ah, 96h,0A0h,0C0h, 0Eh,0D3h,0B7h; 496  
.data:004A012C                 db  57h, 6Dh,0B0h, 3Ch, 64h, 9Ah,0C9h,0E6h; 504  
.data:004A012C                 db 0EDh, 90h, 97h, 8Eh,0DEh, 78h, 43h,0E9h; 512  
.data:004A012C                 db 0ABh,0E5h,0D1h,0BDh,0D1h, 5Ch, 68h,0A4h; 520  
.data:004A012C                 db 0B5h,0F9h, 65h, 4Eh,0DDh,0A8h, 70h,0CDh; 528  
.data:004A012C                 db 0CFh, 10h, 2Bh,0E6h,0F4h, 3Dh, 0Dh, 3Ah; 536  
.data:004A012C                 db  3Fh,0C8h, 3Dh, 7Ah, 5Dh,0FBh,0EDh, 3Ah; 544  
.data:004A012C                 db 0D3h,0BAh,0DDh,   9, 1Ah,0F6h, 1Ah, 8Bh; 552  
.data:004A012C                 db  6Eh,0DBh,0D7h,   9,0EFh, 34h,0EBh, 50h; 560  
.data:004A012C                 db 0E0h,0EEh, 2Eh,0A8h,0A2h, 28h, 10h,0ABh; 568  
.data:004A012C                 db  2Fh, 6Bh,0D1h,0EBh, 53h,   5, 87h,0DEh; 576  
.data:004A012C                 db  41h,0FCh, 1Eh, 11h, 0Fh, 42h,   6, 26h; 584  
.data:004A012C                 db  89h, 0Ah,0FDh,0BBh, 5Ah, 52h, 41h,0F7h; 592  
.data:004A012C                 db  3Eh, 10h, 7Dh, 70h, 5Ah, 40h, 87h,0C2h; 600  
.data:004A012C                 db 0BFh, 7Ch, 7Eh,0A6h, 7Ah, 1Eh,0B1h, 3Dh; 608  
.data:004A012C                 db 0FBh,0C5h, 2Dh,0F7h, 5Fh, 86h,0C8h, 0Ah; 616  
.data:004A012C                 db 0FBh, 2Ah,   1,0C2h,0E5h, 10h,0F7h,0F2h; 624  
.data:004A012C                 db 0B4h, 62h,0F9h,0FCh,0A3h,0FDh,0BBh, 0Eh; 632  
.data:004A012C                 db  9Fh,0C9h,0BFh,0E9h,0C5h,0AAh, 3Dh,0EDh; 640  
.data:004A012C                 db  2Ah,0C8h,0C2h, 62h,0F0h, 3Dh, 73h,0A0h; 648  
.data:004A012C                 db 0D3h, 23h, 1Bh,0BDh, 75h,0E7h,   9,0E1h; 656  
.data:004A012C                 db 0EBh, 29h, 4Bh,0D5h,0B6h,   5,0E5h, 35h; 664  
.data:004A012C                 db  19h, 29h, 28h,0EBh,0A6h,0DCh,0ADh, 50h; 672  
.data:004A012C                 db  2Fh, 1Dh,   6, 84h, 42h, 52h,0D9h, 9Eh; 680  
.data:004A012C                 db  93h,0DBh, 47h, 15h, 41h,0A6h,   0, 83h; 688  
.data:004A012C                 db  23h, 8Ah,   8, 3Ch,0CEh,   1, 81h, 85h; 696  
.data:004A012C                 db 0C3h,0B2h, 63h,0A4h, 7Dh,0ABh,0CDh, 7Ah; 704  
.data:004A012C                 db  4Fh, 16h, 9Ah, 6Eh, 4Dh,0F0h,0CAh,0FEh; 712  
.data:004A012C                 db  85h, 48h, 4Bh, 1Dh,0BFh, 76h, 9Ch, 5Ch; 720  
.data:004A012C                 db  9Bh, 54h,0A6h,0B0h,0B7h,0F9h, 64h, 89h; 728  
.data:004A012C                 db    1,0C2h, 21h,0A2h,0DDh, 6Ah,0F1h, 61h; 736  
.data:004A012C                 db 0A2h,   5, 95h,0FCh,0B5h,   5,0A9h, 10h; 744  
.data:004A012C                 db  66h, 62h,0ADh,0EEh, 1Eh,0A0h, 2Dh, 7Eh; 752  
.data:004A012C                 db 0ECh,0D4h, 5Dh, 27h,0F6h, 88h,0A2h, 79h; 760  
.data:004A012C                 db 0F2h,0FCh, 56h, 79h, 89h, 4Fh,0C4h,0E9h; 768  
.data:004A012C                 db  28h, 32h,0F1h, 4Ch,   8,0E5h,0D3h, 8Ah; 776  
.data:004A012C                 db 0C3h, 0Fh,   7,0FBh, 7Bh, 2Ch,0E5h,   9; 784  
.data:004A012C                 db  59h, 69h,0EBh,0F2h, 23h, 4Bh,0F5h,0FEh; 792  
.data:004A012C                 db  17h,0F7h, 47h,0C0h, 2Fh,0A1h, 56h, 74h; 800  
.data:004A012C                 db  88h, 76h, 90h, 58h,0BBh, 5Ah,0D1h,0F0h; 808  
.data:004A012C                 db  1Bh, 65h, 80h,0A4h,0E4h, 12h,0EBh, 32h; 816  
.data:004A012C                 db  3Ah,0ECh, 99h,0E6h, 3Ah, 8Eh, 8Ch,0E0h; 824  
.data:004A012C                 db 0C0h, 2Bh,0C6h, 6Bh, 62h, 52h, 0Ch,0AAh; 832  
.data:004A012C                 db 0ECh,0CDh, 6Bh,0E3h,0B2h,0CFh, 57h, 26h; 840  
.data:004A012C                 db  8Fh, 77h,0E4h,0C7h, 29h, 35h, 55h,0ECh; 848  
.data:004A012C                 db  68h, 26h,0AEh, 8Fh,0D2h, 67h, 3Ah,0F1h; 856  
.data:004A012C                 db  8Fh, 3Dh, 0Ah,0E0h, 9Ch, 83h, 41h, 2Dh; 864  
.data:004A012C                 db 0E9h,0A8h, 60h, 67h, 5Fh,0CFh, 80h, 51h; 872  
.data:004A012C                 db  33h,0B0h, 57h, 11h,0CEh, 1Fh,0EFh, 19h; 880  
.data:004A012C                 db    0, 79h,0A8h, 72h, 57h, 5Ch, 90h,   6; 888  
.data:004A012C                 db 0BDh,   6,0E6h,0F4h,   7, 6Ah,0DCh, 1Dh; 896  
.data:004A012C                 db  69h, 95h, 0Eh, 52h,0E1h,   4,0D7h, 1Bh; 904  
.data:004A012C                 db 0D4h, 38h,0CDh,0DFh,   1,0CFh,0F3h, 68h; 912  
.data:004A012C                 db 0C9h, 13h, 40h, 8Ch, 8Eh,   6,0FEh,0D3h; 920  
.data:004A012C                 db  7Bh,0ADh, 2Ah, 97h, 56h,0ADh,   8, 53h; 928  
.data:004A012C                 db 0C2h,0B7h, 81h, 7Bh, 26h, 23h, 9Dh, 54h; 936  
.data:004A012C                 db 0A1h,0A3h,0E5h,0C6h, 8Eh, 67h,0F1h, 96h; 944  
.data:004A012C                 db  81h, 0Bh, 68h,0E8h, 54h, 66h, 0Bh, 7Ah; 952  
.data:004A012C                 db  32h, 57h, 8Eh, 7Eh,0E8h, 0Eh, 8Ah, 7Ch; 960  
.data:004A012C                 db    3, 53h,0D5h,0CEh,0F4h,   3, 8Bh, 91h; 968  
.data:004A012C                 db 0FEh,0DFh, 1Ah, 8Eh, 7Fh, 1Eh,0F0h, 75h; 976  
.data:004A012C                 db  11h,   8,0E2h, 23h, 74h, 97h,   3, 9Ch; 984  
.data:004A012C                 db  80h,0DEh, 79h,0D5h,0B0h, 2Ch, 5Ch, 0Eh; 992  
.data:004A012C                 db  15h, 5Bh, 33h,0D5h,0F4h,0ADh, 70h,0C2h; 1000  
.data:004A012C                 db  9Bh,0C1h, 86h,0F1h, 4Eh,0EFh, 3Dh, 65h; 1008  
.data:004A012C                 db    2, 81h,0ACh,0B4h,0E9h, 25h,0B6h, 35h; 1016  
.data:004A012C                 db  1Dh, 7Eh, 16h,0C4h,   5,0F0h, 0Bh,0D2h; 1024  
.data:004A012C                 db 0F8h, 3Eh, 9Fh, 2Dh, 43h, 9Ch, 66h,0C5h; 1032  
.data:004A012C                 db  2Bh,0FDh, 23h,0FEh,0C7h, 72h,0F7h,0E0h; 1040  
.data:004A012C                 db  64h, 10h, 34h, 8Ch,0B3h,0FFh, 4Dh, 3Dh; 1048  
.data:004A012C                 db  52h,0C8h, 4Ah, 82h,0E5h,0B0h, 7Fh, 93h; 1056  
.data:004A012C                 db 0F2h,0EEh,0CEh,0B0h, 67h,0CBh, 6Fh,0D1h; 1064  
.data:004A012C                 db  0Eh,0AAh, 43h,0C4h, 19h, 56h, 9Dh, 15h; 1072  
.data:004A012C                 db 0B8h,0BFh,0ACh,   1, 80h, 8Ch, 30h, 2Bh; 1080  
.data:004A012C                 db  95h,0D8h, 6Ah,0A5h, 5Dh, 9Ah,0BEh, 48h; 1088  
.data:004A012C                 db  6Bh,0CDh,0C6h, 6Bh,0D0h,0C3h,0DDh,0FAh; 1096  
.data:004A012C                 db  6Fh, 47h,0AFh,0A0h,0F5h, 25h,0DFh, 98h; 1104  
.data:004A012C                 db 0D3h, 17h,0D5h,0EAh, 5Ah, 69h,0EAh, 59h; 1112  
.data:004A012C                 db 0CCh,0FAh,0D0h,0FBh,0F1h, 3Eh,0ACh, 1Eh; 1120  
.data:004A012C                 db  3Ah, 53h, 59h, 18h,0E0h,0A7h, 5Fh,0ACh; 1128  
.data:004A012C                 db 0FEh, 26h,0DEh,0B2h, 42h,0B1h,0E6h, 33h; 1136  
.data:004A012C                 db  31h,0DEh, 21h, 69h, 6Fh,0F9h, 34h, 5Dh; 1144  
.data:004A012C                 db  33h,0CEh, 79h,0E8h, 0Dh, 32h,   4, 24h; 1152  
.data:004A012C                 db  98h,0D5h,0B8h, 91h, 5Bh,0F6h, 20h,0B2h; 1160  
.data:004A012C                 db 0C7h,0AFh,0CFh,0E4h,0DAh, 9Ah, 3Ah, 6Dh; 1168  
.data:004A012C                 db 0DBh, 1Fh, 16h,0C8h, 8Ah, 5Ah, 28h,0C6h; 1176  
.data:004A012C                 db 0FFh,   5, 38h, 0Eh, 7Bh, 91h, 1Ch, 82h; 1184  
.data:004A012C                 db  69h, 77h, 27h,0F1h, 68h, 83h, 9Eh, 9Eh; 1192  
.data:004A012C                 db 0A3h,0E5h, 40h, 3Dh, 63h, 9Bh, 47h, 82h; 1200  
.data:004A012C                 db  83h,0F6h,0D3h, 1Dh, 50h,0B6h, 97h,0DDh; 1208  
.data:004A012C                 db  8Ah,0E6h, 6Ah, 16h, 0Dh, 0Bh,0DFh, 6Fh; 1216  
.data:004A012C                 db  48h, 10h,0ABh, 62h,0F6h,0B1h,0E8h,0C9h; 1224  
.data:004A012C                 db  8Ah, 6Eh, 49h, 9Fh,0E1h,0CCh, 1Eh,0C1h; 1232  
.data:004A012C                 db  24h, 20h,0A9h, 0Ch, 8Ah,0D4h, 31h,0EAh; 1240  
.data:004A012C                 db 0D8h,0C2h, 91h, 8Ch, 90h, 91h, 0Fh, 67h; 1248  
  
  
这里写一个 idc 脚本把 g\_blacklist 输出到文件 blacklist.txt  
  
#include <idc.idc>  
  
static print\_blacklist()  
{  
        auto blacklist\_addr;  
        auto i, p;  
        auto fl;  
  
        blacklist\_addr = 0x4a012c;  
        p = blacklist\_addr;  
  
        fl = fopen("blacklist.txt", "w");  
        if ( fl == 0)  
        {  
                Message("打开文件失败！\\n");  
                return;  
        }  
        for (i=0; i<1256; i++)  
        {  
                if ( (i!=0) && (i%8 == 0))  
                {  
                        Message("\\n");  
                        fprintf(fl,"\\n");  
                }  
                Message("%02x", Byte(p) );  
                fprintf(fl, "%02x", Byte(p) );  
                p++;  
        }  
        fclose(fl);  
}  
  
  
3. 用户名黑名单  
  
  
紧跟着下面有一个用户名黑名单 user\_blacklist, 存储的是用户名的 sha1 值的前 64bit, 不过  
这里的 sha1 好像作了一些手脚, 最后的 HASH 摘要并没有处理字节序问题(按 DWORD 存储了, 不  
过下面比较的时候也用的是 DWORD 比较所以比较结果也是正确的), 发现 sha1\_update,  
sha1\_final 都有一个参数 1, 下一篇跟踪一下 sha1 算法.  
  
有人可能要问我怎么知道是 sha1 的, 其实俺也是老菜的方法, 在跟进 sha1\_init 函数是看到了  
只是简单的赋值 0x67452301, 0xefcdab89, 0x98badcfe, 0x10325476, 0x0c3d2e1f0, 俺猜  
想可能是遇到 hash 算法了(如:md5, sha), 就计算 username 的各种 hash 值和内存对比就判断它  
是 sha1 了.  
  
  
.text:0040E81F                 xor     ebx, ebx  
.text:0040E821 NEXT\_UBLACK:                            ; CODE XREF: check\_regdata+1C2j  
.text:0040E821                 lea     eax, \[esp+0E88h+sha1\_ctx\] ; ctx  
.text:0040E828                 call    sha1\_init  
.text:0040E82D                 push    1                 
.text:0040E82F                 push    regData         ; username  
.text:0040E830                 call    \_strlen  
.text:0040E835                 pop     ecx  
.text:0040E836                 mov     ecx, eax        ; len  
.text:0040E838                 mov     edx, regData    ; data  
.text:0040E83A                 lea     eax, \[esp+0E8Ch+sha1\_ctx\] ; ctx  
.text:0040E841                 call    sha1\_update  
.text:0040E846                 lea     edx, \[esp+0E88h+sha1\_digest\] ; digest  
.text:0040E84D                 lea     eax, \[esp+0E88h+sha1\_ctx\] ; ctx  
.text:0040E854                 mov     cl, 1           ; endian  
.text:0040E856                 call    sha1\_final  
.text:0040E85B                 mov     edx, user\_blacklist.sha\_0\[ebx\*8\] ;   
.text:0040E862                 cmp     edx, dword ptr \[esp+0E88h+sha1\_digest\] ; \[Check4\]  
.text:0040E869                 jnz     short loc\_40E882  
.text:0040E86B                 mov     eax, user\_blacklist.sha\_1\[ebx\*8\] ;   
.text:0040E872                 cmp     eax, dword ptr \[esp+0E88h+sha1\_digest+4\] ; \[Check4\]  
.text:0040E879                 jnz     short loc\_40E882  
.text:0040E87B                 xor     eax, eax        ; \[Check4\] 检查注册用户名的 SHA1 值的前 8 字节是否在  
                                                       ; user\_blacklist 里, 在则注册失败!  
.text:0040E87D                 jmp     REG\_FAIL\_EXIT  
.text:0040E882 loc\_40E882:                             ; CODE XREF: check\_regdata+1A5j  
.text:0040E882                                         ; check\_regdata+1B5j  
.text:0040E882                 inc     ebx  
.text:0040E883                 cmp     ebx, 6  
.text:0040E886                 jl      short NEXT\_UBLACK  
  
有 6 个用户黑名单, 也 idc 脚本打印出来  
  
.data:004A0614 user\_blacklist  \_SHA\_ <539D5CC7h, 1C48938Ch>; 0  
.data:004A0614                 \_SHA\_ <0AEE5DA2Fh, 83182A3Ch>; 1   
.data:004A0614                 \_SHA\_ <0EF15C666h, 0BB1FDF30h>; 2  
.data:004A0614                 \_SHA\_ <2B2CB836h, 1589154Ah>; 3  
.data:004A0614                 \_SHA\_ <0C16C2007h, 4EC9F4CEh>; 4  
.data:004A0614                 \_SHA\_ <37A1C984h, 20A6A0D5h>; 5  
  
#include <idc.idc>  
  
static print\_user\_blacklist()  
{  
        auto user\_blacklist\_addr;  
        auto i, j, p;  
        auto fl;  
  
        user\_blacklist\_addr = 0x4a0614;  
        p = user\_blacklist\_addr;  
          
        fl = fopen("user\_blacklist.txt", "w");  
        if ( fl == 0 )  
        {  
                Message("打开文件失败！\\n");  
                return;  
        }  
        for (i=0; i<6\*2; i++)  
        {  
                if ( (i!=0) && (i%2 == 0))  
                {  
                        Message("\\n");  
                        fprintf(fl,"\\n");  
                }  
                Message("%08x", Dword(p) );  
                fprintf(fl, "%08x", Dword(p) );  
                p = p+4;  
        }  
        fclose(fl);  
}  
  
到此已经有 5 处检查, 1). g\_HCode0 的 CRC32 校验; 2). 从 snHead 中取出的 snCode2, snCode3  
的长度不大于 256; 3). snCode1 的前 8 位是否在黑名单; 4). username 的 sha1 前 8 字节是否在  
用户黑名单; 5).regData.licType + username + regData.snCode1/2/3/4 的~CRC32 值是否等  
于 regData.crc32  

WinRAR 3.62 注册算法 - SHA1 hash 算法  
  
  
\[声明\] 本文为逆向学习参考文章, 若使用 winrar 请向作者注册.  
  
4. SHA1 算法  
  
上回讲用户黑名单是遇到了 SHA1 算法, 这回就看看这个算法.  
首先用到一个结构:  
  
typedef unsigned int u\_int32\_t;  
typedef unsigned char u\_int8\_t;  
  
typedef struct \_sha1\_ctx  
{  
    u\_int32\_t state\[5\];  
    u\_int32\_t count\[2\];  
    u\_int8\_t buffer\[64\];  
} SHA1\_CTX;  
  
  
在这里我都是先把结构给出来了, 然后再分析相关函数的, 其实逆向分析正好是相反的过程,  
个人认为数据结构的确定是逆向过程的一个很重要的部分, 而数据结构的确定可能需要查看  
基于某地址不同偏移的内存访问的所有函数.  
  
首先是初始化函数, 非常简单:  
  
.text:0042FFB0 ; void \_\_fastcall sha1\_init(SHA1\_CTX \*ctx)  
.text:0042FFB0 sha1\_init       proc near  
  
.text:0042FFB0 \_ctx = eax   
.text:0042FFB0                 mov     \[\_ctx+SHA1\_CTX.sha\_0\], 67452301h   
.text:0042FFB6                 xor     edx, edx  
.text:0042FFB8                 mov     \[\_ctx+SHA1\_CTX.sha\_1\], 0EFCDAB89h  
.text:0042FFBF                 mov     \[\_ctx+SHA1\_CTX.sha\_2\], 98BADCFEh  
.text:0042FFC6                 mov     \[\_ctx+SHA1\_CTX.sha\_3\], 10325476h   
.text:0042FFCD                 mov     \[\_ctx+SHA1\_CTX.sha\_4\], 0C3D2E1F0h  
.text:0042FFD4                 mov     \[\_ctx+SHA1\_CTX.count1\], edx  
.text:0042FFD7                 mov     \[\_ctx+SHA1\_CTX.count0\], edx  
.text:0042FFDA                 retn  
.text:0042FFDA sha1\_init       endp  
  
  
sha1\_init C 实现:  
  
void   
sha1\_init(SHA1\_CTX \*ctx)  
{  
        ctx->state\[0\] = 0x67452301;  
        ctx->state\[1\] = 0xEFCDAB89;  
        ctx->state\[2\] = 0x98BADCFE;  
        ctx->state\[3\] = 0x10325476;  
        ctx->state\[4\] = 0xC3D2E1F0;  
        ctx->count\[0\] = 0;  
        ctx->count\[1\] = 0;  
}  
  
看下一个函数:  
.text:0040E82D                 push    1  
.text:0040E82F                 push    regData         ; username  
.text:0040E830                 call    \_strlen  
.text:0040E830  
.text:0040E835                 pop     ecx  
.text:0040E836                 mov     ecx, eax        ; len  
.text:0040E838                 mov     edx, regData    ; data  
.text:0040E83A                 lea     eax, \[esp+0E8Ch+sha1\_ctx\] ; ctx  
.text:0040E841                 call    sha1\_update  
  
函数有 3 个参数, eax 指向 sha1\_ctx 结构, edx 指向一个 buffer, 这里指向 regData.username  
ecx 是 buffer 的长度, 还有一个参数 1(push 1), 跟进去往后看这个参数干什么用的.  
  
.text:0042FFDC ; void \_\_fastcall sha1\_update(SHA1\_CTX \*ctx,BYTE \*data,DWORD len,BYTE trans\_copies)  
.text:0042FFDC sha1\_update     proc near               ; CODE XREF: sub\_407D7C+88p  
.text:0042FFDC  
.text:0042FFDC var\_10          = dword ptr -10h  
.text:0042FFDC \_data           = dword ptr -4  
.text:0042FFDC trans\_copies    = byte ptr  8  
.text:0042FFDC  
.text:0042FFDC \_len = edi  
.text:0042FFDC \_ctx = esi  
.text:0042FFDC                 push    ebp  
.text:0042FFDD                 mov     ebp, esp  
.text:0042FFDF                 push    ecx  
.text:0042FFE0                 push    ebx  
.text:0042FFE1                 push    \_ctx  
.text:0042FFE2                 push    \_len  
.text:0042FFE3                 mov     \_len, ecx  
.text:0042FFE5                 mov     \[ebp+\_data\], edx  
.text:0042FFE8                 mov     \_ctx, eax  
.text:0042FFEA                 mov     edx, \_len  
.text:0042FFEC                 shl     edx, 3          ; 把长度\*8, 把字节数转换成 bit 位  
.text:0042FFEF                 mov     eax, \[\_ctx+SHA1\_CTX.count0\]  
.text:0042FFF2                 shr     eax, 3          ; 把 bit 位数转换成字节数  
.text:0042FFF5                 and     eax, 3Fh        ; 是否是 64 字节的倍数, 因为低于 64 字节部分还没有处理  
.text:0042FFF8                 add     \[\_ctx+SHA1\_CTX.count0\], edx ; 计数相加, 这里应该知道 count0 存储的是 buffer 的 bit 数  
.text:0042FFFB                 mov     ecx, \[\_ctx+SHA1\_CTX.count0\]  
.text:0042FFFE                 cmp     edx, ecx  
.text:00430000                 jbe     short loc\_430005  
.text:00430002                 inc     \[\_ctx+SHA1\_CTX.count1\] ; 处理加法运算发生的进位, 看来 count0, connt1  
.text:00430002                                         ; 应该是一个 64bit 整数.  
.text:00430005  
.text:00430005 loc\_430005:                             ; CODE XREF: sha1\_update+24j  
.text:00430005                 mov     edx, \_len  
.text:00430007                 shr     edx, 1Dh  
.text:0043000A                 add     \[\_ctx+SHA1\_CTX.count1\], edx ; 处理左移发生的进位  
.text:0043000D                 lea     ecx, \[\_len+eax\]  
.text:00430010                 cmp     ecx, 3Fh  
.text:00430013                 jbe     short LESS\_64  
.text:00430015                 mov     ebx, 40h  
.text:0043001A                 sub     ebx, eax  
.text:0043001C                 add     eax, \_ctx  
.text:0043001E                 push    ebx             ; n  
.text:0043001F                 add     eax, 1Ch  
.text:00430022                 push    \[ebp+\_data\]     ; src  
.text:00430025                 push    eax             ; dest  
.text:00430026                 call    \_memcpy         ; 拷贝 64 个字节到 sha1\_ctx.buffer 中  
.text:0043002B                 add     esp, 0Ch  
.text:0043002E                 lea     edx, \[\_ctx+SHA1\_CTX.buffer\] ; data  
.text:00430031                 mov     cl, \[ebp+trans\_copies\] ; trans\_copies  
.text:00430034                 mov     eax, \_ctx       ; ctx\_state  
.text:00430036                 call    sha1\_transform  ; 处理 sha1\_ctx.buffer, sha1\_transform 是 sha 的核心运算.  
.text:00430036  
.text:0043003B                 jmp     short loc\_43004F  
.text:0043003D ; ---------------------------------------------------------------------------  
.text:0043003D NEXT\_64B:                               ; CODE XREF: sha1\_update+78j  
.text:0043003D                 mov     edx, \[ebp+\_data\]  
.text:00430040                 add     edx, ebx        ; data  
.text:00430042                 mov     cl, \[ebp+trans\_copies\] ; trans\_copies  
.text:00430045                 mov     eax, \_ctx       ; ctx\_state  
.text:00430047                 call    sha1\_transform  
.text:0043004C loc\_43004C:                             ; DATA XREF: .data:004A5BC0o  
.text:0043004C                 add     ebx, 40h  
.text:0043004F loc\_43004F:                             ; CODE XREF: sha1\_update+5Fj  
.text:0043004F                 lea     eax, \[ebx+3Fh\]  
.text:00430052                 cmp     \_len, eax  
.text:00430054                 ja      short NEXT\_64B  
.text:00430056                 xor     eax, eax  
.text:00430058                 jmp     short loc\_43005C  
.text:0043005A ; ---------------------------------------------------------------------------  
.text:0043005A LESS\_64:                                ; CODE XREF: sha1\_update+37j  
.text:0043005A                 xor     ebx, ebx  
.text:0043005C loc\_43005C:                             ; CODE XREF: sha1\_update+7Cj  
.text:0043005C                 cmp     ebx, \_len  
.text:0043005E                 jnb     short EXIT  
.text:00430060                 sub     \_len, ebx  
.text:00430062                 add     \_ctx, eax  
.text:00430064                 push    \_len            ; n  
.text:00430065                 mov     edx, \[ebp+\_data\]  
.text:00430068                 add     edx, ebx  
.text:0043006A                 add     \_ctx, 1Ch  
.text:0043006D                 push    edx             ; src  
.text:0043006E                 push    \_ctx            ; dest  
.text:0043006F                 call    \_memcpy         ; 不到 64 字节的部分拷贝到 sha1\_ctx.buffer 中下次处理.  
.text:00430074                 add     esp, 0Ch  
.text:00430077 EXIT:                                   ; CODE XREF: sha1\_update+82j  
.text:00430077                 pop     \_len  
.text:00430078                 pop     \_ctx  
.text:00430079                 pop     ebx  
.text:0043007A                 pop     ecx  
.text:0043007B                 pop     ebp  
.text:0043007C                 retn    4  
.text:0043007C sha1\_update     endp  
  
  
sha1\_update C 实现:  
void \_\_fastcall   
sha1\_update(SHA1\_CTX \*ctx, u\_int8\_t \*data, u\_int32\_t len, u\_int8\_t trans\_copies)   
{  
        u\_int32\_t remain;  
        u\_int32\_t count;  
  
        remain = ctx->count\[0\] >> 3 & 0x3F; /\* 未处理的剩余字节数 \*/  
        count = len << 3;  
        ctx->count\[0\] += count;  
  
        if (count > ctx->count\[0\]) /\* 进位处理 \*/  
                ctx->count\[1\]++;  
        ctx->count\[1\] += len >> 29;  
  
        if (len + remain > 63)  
        {  
                count = 64 - remain;  
                memcpy(ctx->buffer+remain, data, count);  
                sha1\_transform(ctx->state, ctx->buffer, trans\_copies);  
                for (; len > count+63 ; count += 64) /\* 循环每次计算 64 字节 \*/  
                {  
                        sha1\_transform(ctx->state, data+count, trans\_copies);  
                }  
        }  
        else  
                count = 0;  
  
        if (count < len)        /\* 把不到 64 字节的部分拷贝到 ctx->buffer \*/  
                memcpy(ctx->buffer+remain, data+count, len-count);  
}  
  
  
核心运算 sha1\_transform:  
  
.text:0042E374 ; void \_\_fastcall sha1\_transform(DWORD \*ctx\_state,BYTE \*data,BYTE trans\_copies)  
.text:0042E374 sha1\_transform  proc near               ; CODE XREF: sha1\_update+5Ap  
.text:0042E374                                         ; sha1\_update+6Bp  
.text:0042E374                                         ; sha1\_final+119p  
.text:0042E374  
.text:0042E374 \_state          = dword ptr -14h  
.text:0042E374 \_buffer         = dword ptr -10h  
.text:0042E374 \_trans\_copies   = byte ptr -0Ch  
.text:0042E374 A               = dword ptr -8  
.text:0042E374 E               = dword ptr -4  
.text:0042E374  
.text:0042E374 eax\_B = eax  
.text:0042E374 edx\_C = edx  
.text:0042E374 ecx\_D = ecx  
.text:0042E374                 push    ebx  
.text:0042E375                 push    esi  
.text:0042E376                 push    edi  
.text:0042E377                 push    ebp  
.text:0042E378                 add     esp, 0FFFFFFECh  
.text:0042E37B                 mov     \[esp+14h+\_trans\_copies\], cl  
.text:0042E37F                 mov     \[esp+14h+\_buffer\], edx\_C  
.text:0042E383                 mov     \[esp+14h+\_state\], eax\_B  
.text:0042E386                 cmp     \[esp+14h+\_trans\_copies\], 0  
.text:0042E38B                 jz      short IF1\_ELSE  
.text:0042E38B  
.text:0042E38D                 mov     ebx, offset g\_Buffer64  
.text:0042E392                 push    40h             ; n  
.text:0042E394                 push    \[esp+18h+\_buffer\] ; src  
.text:0042E398                 push    ebx             ; dest  
.text:0042E399                 call    \_memcpy  
.text:0042E399  
.text:0042E39E                 add     esp, 0Ch  
.text:0042E3A1                 jmp     short loc\_42E3A7  
.text:0042E3A1  
.text:0042E3A3 ; ---------------------------------------------------------------------------  
.text:0042E3A3  
.text:0042E3A3 IF1\_ELSE:                               ; CODE XREF: sha1\_transform+17j  
.text:0042E3A3                 mov     ebx, \[esp+14h+\_buffer\]  
.text:0042E3A3  
.text:0042E3A7  
.text:0042E3A7 loc\_42E3A7:                             ; CODE XREF: sha1\_transform+2Dj  
.text:0042E3A7                 mov     eax\_B, \[esp+14h+\_state\]  
.text:0042E3AA                 mov     edx\_C, \[eax\_B\]  
.text:0042E3AC                 mov     \[esp+14h+A\], edx\_C  
.text:0042E3B0                 mov     ecx\_D, \[esp+14h+\_state\]  
.text:0042E3B3                 mov     edx\_C, \[esp+14h+\_state\]  
.text:0042E3B6                 mov     esi, \[esp+14h+\_state\]  
.text:0042E3B9                 mov     eax\_B, \[ecx\_D+4\]  
.text:0042E3BC                 mov     ecx\_D, \[esp+14h+\_state\]  
.text:0042E3BF                 mov     edx\_C, \[edx\_C+8\]  
.text:0042E3C2                 mov     esi, \[esi+10h\]  
.text:0042E3C5                 mov     ecx\_D, \[ecx\_D+0Ch\]  
.text:0042E3C8                 mov     \[esp+14h+E\], esi ; 这里我们把 ctx->state\[5\]数组中的值分别赋值给 A, eax, edx, ecx,  
.text:0042E3C8                                         ; E, 变量 A,E 是内存分配的局域变量, 其它几个值用的寄存器,  
.text:0042E3C8                                         ; 我在这里分别把 eax, edx, ecx 重新命名位, eax\_B, edx\_C,  
.text:0042E3C8                                         ; ecx\_D(以下分别称其 B, C, D), ebx 指向输入 64 字节的数据  
.text:0042E3C8                                         ; (以下都称其 buf); 因为 buf 是按 DWORD 取值的, 故我把 ebx 定  
.text:0042E3C8                                         ; 义为一个指向 UI32\[16\]的数组:  
.text:0042E3C8                                         ; 00000000 UI32            struc ; (sizeof=0x40)  
.text:0042E3C8                                         ; 00000000 \_0              dd ?  
.text:0042E3C8                                         ; 00000004 \_1              dd ?  
.text:0042E3C8                                         ; 00000008 \_2              dd ?  
.text:0042E3C8                                         ; 0000000C \_3              dd ?  
.text:0042E3C8                                         ; 00000010 \_4              dd ?  
.text:0042E3C8                                         ; 00000014 \_5              dd ?  
.text:0042E3C8                                         ; 00000018 \_6              dd ?  
.text:0042E3C8                                         ; 0000001C \_7              dd ?  
.text:0042E3C8                                         ; 00000020 \_8              dd ?  
.text:0042E3C8                                         ; 00000024 \_9              dd ?  
.text:0042E3C8                                         ; 00000028 \_10             dd ?  
.text:0042E3C8                                         ; 0000002C \_11             dd ?  
.text:0042E3C8                                         ; 00000030 \_12             dd ?  
.text:0042E3C8                                         ; 00000034 \_13             dd ?  
.text:0042E3C8                                         ; 00000038 \_14             dd ?  
.text:0042E3C8                                         ; 0000003C \_15             dd ?  
.text:0042E3C8                                         ; 00000040 UI32            ends  
.text:0042E3C8                                         ;  
.text:0042E3C8                                         ;  
.text:0042E3C8                                         ;  
.text:0042E3CC                 mov     esi, \[ebx+UI32.\_0\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 1 次运算开始\*\*\*\*  
.text:0042E3CE                 mov     edi, \[ebx\]      ; 取 buf 中第一个 DWORD, 称其 buf\[0\], 本函数此行以下,  
.text:0042E3CE                                         ; 若未另声明则 buf 为一 DWORD 数组, 数组大小 64 字节.  
.text:0042E3CE                                         ;  
.text:0042E3CE                                         ;  
.text:0042E3D0                 shr     edi, 8          ; buf\[0\]右移 8bit  
.text:0042E3D3                 mov     ebp, \[ebx+UI32.\_0\]  
.text:0042E3D5                 shl     esi, 18h        ; buf\[0\]左移 24bit(32-8)  
.text:0042E3D8                 or      esi, edi        ; 左移和右移的结果按位或, (buf\[0\]>>8) | (buf\[0\]<<24)  
.text:0042E3DA                 mov     edi, \[ebx+UI32.\_0\]  
.text:0042E3DC                 shl     edi, 8          ; buf\[0\]左移 8bit  
.text:0042E3DF                 and     esi, 0FF00FF00h ; (buf\[0\]>>8) | (buf\[0\]<<24) & 0xFF00FF00  
.text:0042E3E5                 shr     ebp, 18h        ; buf\[0\]右移 24bit  
.text:0042E3E8                 or      edi, ebp        ; (buf\[0\]<<8) | (buf\[0\]>>24)  
.text:0042E3EA                 and     edi, 0FF00FFh   ; (buf\[0\]<<8) | (buf\[0\]>>24) & 0x0FF00FF  
.text:0042E3F0                 or      esi, edi        ; ((buf\[0\]>>8) | (buf\[0\]<<24) & 0xFF00FF00) |  
.text:0042E3F0                                         ; ((buf\[0\]<<8) | (buf\[0\]>>24) & 0x0FF00FF)  
.text:0042E3F0                                         ;  
.text:0042E3F0                                         ; buf\[0\]>>8 求或 buf\[0\]<<24 求与 0xFF00FF00, 也就是只留  
.text:0042E3F0                                         ; 最高字节和第 2 个字节, 而计算后最高字节是 buf\[0\]的最低字  
.text:0042E3F0                                         ; 节, 第 2 个字节是 buf\[0\]的第 3 个字节.  
.text:0042E3F0                                         ;  
.text:0042E3F0                                         ; buf\[0\]<<8 求或 buf\[0\]>>24 求与 0x0FF00FF, 就是只留最  
.text:0042E3F0                                         ; 低字节和第 3 个字节, 而计算后最低字节就是 buf\[0\]的最高字  
.text:0042E3F0                                         ; 节, 第 3 个字节就是 buf\[0\]的第 2 个字节  
.text:0042E3F0                                         ;  
.text:0042E3F0                                         ; 两者再按位或以后, 就是把原来的字节序反了一下,  
.text:0042E3F0                                         ; 以后称这种反序运算为 RE(v), 如: RE(buf\[0\])  
.text:0042E3F0                                         ;  
.text:0042E3F2                 mov     edi, ecx\_D  
.text:0042E3F4                 xor     edi, edx\_C      ; D 异或 C, D^C  
.text:0042E3F6                 mov     \[ebx+UI32.\_0\], esi ; 把 buf\[0\]反序后的的值重新复制给 buf\[0\]  
.text:0042E3F6                                         ; buf\[0\] = RE(buf\[0\])  
.text:0042E3F8                 and     edi, eax\_B      ; (D^C)&B  
.text:0042E3FA                 mov     ebp, \[esp+14h+A\]  
.text:0042E3FE                 xor     edi, ecx\_D      ; ((D^C)&B)^D  
.text:0042E400                 add     esi, edi        ; RE(buf\[0\]) & (((D^C)&B)^D)  
.text:0042E402                 mov     edi, \[esp+14h+A\]  
.text:0042E406                 shl     edi, 5          ; A<<5  
.text:0042E409                 shr     ebp, 1Bh        ; A>>27; // 27=32-5  
.text:0042E40C                 or      edi, ebp        ; (A>>27) | (A<<5)  
.text:0042E40C                                         ; 这种运算下面统一叫做 RL(value, n) = (value<<n) | (value>>(32-n))  
.text:0042E40C                                         ; 如: 此处 RL(A,5)  
.text:0042E40C                                         ; 同时: RE(v) =(RL(v,24)&0xFF00FF00) | (RL(v,8)&0x0FF00FF)  
.text:0042E40C                                         ;  
.text:0042E40E                 add     esi, edi        ; RE(buf\[0\]) + (((D^C)&B)^D) + RL(A,5)  
.text:0042E40E                                         ; 从下面得知 RX 运算以后, 整个表达式变成了:  
.text:0042E40E                                         ; RE(buf\[0\]) + RX(B,C,D) + RL(A,5)  
.text:0042E40E                                         ;  
.text:0042E40E                                         ;  
.text:0042E410                 add     esi, 5A827999h ; <suspicious> ; (RE(buf\[0\]) + RX(B,C,D) + RL(A,5)) + 0x5A827999  
.text:0042E416                 add     \[esp+14h+E\], esi ; E += (RE(buf\[0\]) + RX(B,C,D) + RL(A,5)) + 0x5A827999  
.text:0042E41A                 mov     esi, eax\_B  
.text:0042E41C                 shr     eax\_B, 2        ; B>>2  
.text:0042E41F                 mov     edi, \[ebx+UI32.\_1\] ; edi = buf\[1\]  
.text:0042E422                 shl     esi, 1Eh        ; B<<(32-2)  
.text:0042E425                 mov     ebp, \[ebx+UI32.\_1\] ; ebp = buf\[1\]  
.text:0042E428                 or      esi, eax\_B      ; (B>>2) | (B<<(32-2))  
.text:0042E428                                         ; 即 RL(B,30)  
.text:0042E42A                 mov     eax\_B, esi      ; B = RL(B,30)  
.text:0042E42A                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 1 次运算结束\*\*\*\*  
.text:0042E42A                                         ;  
.text:0042E42A                                         ;  
.text:0042E42C                 mov     esi, \[ebx+UI32.\_1\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2 次运算开始\*\*\*\*  
.text:0042E42C                                         ; buf\[1\]  
.text:0042E42F                 shl     esi, 18h        ; buf\[1\]<<24  
.text:0042E432                 shr     edi, 8          ; buf\[1\]>>8  
.text:0042E435                 or      esi, edi        ; RL(buf\[1\],24) = (buff\[1\]>>8) | (buff\[1\]<<24)  
.text:0042E437                 mov     edi, \[ebx+UI32.\_1\]  
.text:0042E43A                 shl     edi, 8          ; buf\[1\]<<8  
.text:0042E43D                 and     esi, 0FF00FF00h ; RL(buf\[1\],24)&0xFF00FF00  
.text:0042E443                 shr     ebp, 18h        ; buf\[1\]>>24  
.text:0042E446                 or      edi, ebp        ; RL(buf\[1\],8) = (buf\[1\]>>24) | (buf\[1\]<<8)  
.text:0042E448                 and     edi, 0FF00FFh   ; RL(buf\[1\],8)&0xFF00FF  
.text:0042E44E                 or      esi, edi        ; RE(buf\[1\]) = (RL(buf\[1\],24) & 0xFF00FF00)| (RL(buf\[1\],8)&0x0FF00FF)  
.text:0042E44E                                         ; 反序 buf\[1\]  
.text:0042E450                 mov     edi, edx\_C  
.text:0042E452                 mov     \[ebx+UI32.\_1\], esi ; buf\[1\] = RE(buf\[1\])  
.text:0042E455                 xor     edi, eax\_B      ; C^B  
.text:0042E457                 and     edi, \[esp+14h+A\] ; (C^B)&A  
.text:0042E45B                 mov     ebp, \[esp+14h+E\]  
.text:0042E45F                 xor     edi, edx\_C      ; ((C^B)&A)^C  
.text:0042E45F                                         ; 这个运算是不是很面熟? 对前面出现过((D^C)&B)^D  
.text:0042E45F                                         ; 定义一个运算: RX(w,x,y) = ((x^y)&w)^y  
.text:0042E45F                                         ; 这里 RX(A,B,C) = ((B^C)&A)^C  
.text:0042E45F                                         ;  
.text:0042E461                 add     esi, edi        ; RE(buf\[1\])+RX(A,B,C)  
.text:0042E463                 mov     edi, \[esp+14h+E\]  
.text:0042E467                 shl     edi, 5          ; E<<5  
.text:0042E46A                 shr     ebp, 1Bh        ; E>>27  
.text:0042E46D                 or      edi, ebp        ; RL(E,5) = (E<<5) | (E>>(32-5))  
.text:0042E46F                 add     esi, edi        ; RE(buf\[1\]) + RX(A,B,C) + RL(E,5)  
.text:0042E471                 mov     edi, \[esp+14h+A\]  
.text:0042E475                 add     esi, 5A827999h ; <suspicious> ; RE(buf\[1\]) + RX(A,B,C) + RL(E,5) + 0x5a827999  
.text:0042E47B                 add     ecx\_D, esi      ; D += RE(buf\[1\]) + RX(A,B,C) + RL(E,5) + 0x5a827999  
.text:0042E47D                 mov     esi, \[esp+14h+A\]  
.text:0042E481                 shl     esi, 1Eh        ; A<<30  
.text:0042E484                 shr     edi, 2          ; A>>2  
.text:0042E487                 or      esi, edi        ; RL(A,30) = (A<<30) | (A>>(32-30))  
.text:0042E489                 mov     \[esp+14h+A\], esi ; A = RL(A,30)  
.text:0042E489                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2 次运算结束\*\*\*\*  
.text:0042E489                                         ;  
.text:0042E489                                         ;  
.text:0042E48D                 mov     esi, \[ebx+UI32.\_2\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3 次运算开始\*\*\*\*  
.text:0042E490                 mov     edi, \[ebx+UI32.\_2\] ; buf\[2\]  
.text:0042E493                 shr     edi, 8          ; buf\[2\]>>8  
.text:0042E496                 mov     ebp, \[ebx+UI32.\_2\]  
.text:0042E499                 shl     esi, 18h        ; buf\[2\]<<24  
.text:0042E49C                 or      esi, edi        ; RL(buf\[2\],24)  
.text:0042E49E                 mov     edi, \[ebx+UI32.\_2\]  
.text:0042E4A1                 shl     edi, 8          ; buf\[2\]<<8  
.text:0042E4A4                 and     esi, 0FF00FF00h ; RL(buf\[2\],24)&0xff00ff00  
.text:0042E4AA                 shr     ebp, 18h        ; buf\[2\]>>24  
.text:0042E4AD                 or      edi, ebp        ; RL(buf\[2\],8)  
.text:0042E4AF                 mov     ebp, ecx\_D  
.text:0042E4B1                 and     edi, 0FF00FFh   ; RL(buf\[2\],8)&0x0ff00ff  
.text:0042E4B7                 or      esi, edi        ; RE(buf\[2\]) = (RL(buf\[2\],24)&0xff00ff00) | (RL(buf\[2\],8)&0x0ff00ff)  
.text:0042E4B9                 mov     \[ebx+UI32.\_2\], esi ; buf\[2\] = RE(buf\[2\])  
.text:0042E4BC                 mov     edi, \[esp+14h+A\]  
.text:0042E4C0                 xor     edi, eax\_B      ; A^B  
.text:0042E4C2                 and     edi, \[esp+14h+E\] ; (B^A)&E  
.text:0042E4C6                 xor     edi, eax\_B      ; RX(E,A,B) = ((B^A)&E)^B  
.text:0042E4C8                 add     esi, edi        ; RE(buf\[2\]) + RX(E,A,B)  
.text:0042E4CA                 mov     edi, ecx\_D  
.text:0042E4CC                 shl     edi, 5          ; D<<5  
.text:0042E4CF                 shr     ebp, 1Bh        ; D>>27  
.text:0042E4D2                 or      edi, ebp        ; RL(D,5) = (D<<5) | (D>>(32-5))  
.text:0042E4D4                 add     esi, edi        ; RE(buf\[2\]) + RX(E,A,B) + RL(D,5)  
.text:0042E4D6                 mov     edi, \[esp+14h+E\]  
.text:0042E4DA                 add     esi, 5A827999h ; <suspicious> ; RE(buf\[2\]) + RX(E,A,B) + RL(D,5) + 0x5a827999  
.text:0042E4E0                 add     edx\_C, esi      ; C += RE(buf\[2\]) + RX(E,A,B) + RL(D,5) + 0x5a827999  
.text:0042E4E2                 mov     esi, \[esp+14h+E\]  
.text:0042E4E6                 shl     esi, 1Eh        ; E<<30  
.text:0042E4E9                 shr     edi, 2          ; E>>2  
.text:0042E4EC                 or      esi, edi        ; RL(E,30) = (E<<30) | (E>>(32-30))  
.text:0042E4EE                 mov     \[esp+14h+E\], esi ; E = RL(E,30)  
.text:0042E4EE                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3 次运算结束\*\*\*\*  
.text:0042E4EE                                         ; 观察前 3 次运算总结:  
.text:0042E4EE                                         ; 1. 从 buf 中取一个 DWORD, 反序后保存.  
.text:0042E4EE                                         ; 2. 每次运算都用到了 A,B,C,D,E, 只是用的顺序不同.  
.text:0042E4EE                                         ; 3. 每次运算除了修改了 buf\[i\]的内容, 还修改了 A-E 中  
.text:0042E4EE                                         ; 两个'变量'  
.text:0042E4EE                                         ;  
.text:0042E4EE                                         ; 定义一个函数:  
.text:0042E4EE                                         ; void inline R0(u\_int32\_t a,  
.text:0042E4EE                                         ;                u\_int32\_t b,  
.text:0042E4EE                                         ;                u\_int32\_t c,  
.text:0042E4EE                                         ;                u\_int32\_t d,  
.text:0042E4EE                                         ;                u\_int32\_t e,  
.text:0042E4EE                                         ;                u\_int32\_t &n)  
.text:0042E4EE                                         ; {  
.text:0042E4EE                                         ;   \*n = RE(\*n);  
.text:0042E4EE                                         ;   e += \*n + RX(b,c,d) + 0x5a827999;  
.text:0042E4EE                                         ;   b = RL(b,30);  
.text:0042E4EE                                         ; }  
.text:0042E4EE                                         ;  
.text:0042E4EE                                         ; 那么前 3 次调用此函数分别为:  
.text:0042E4EE                                         ; R\_0(A,B,C,D,E,&buf\[0\]);  
.text:0042E4EE                                         ; R\_0(E,A,B,C,D,&buf\[1\]);  
.text:0042E4EE                                         ; R\_0(D,E,A,B,C,&buf\[2\]);  
.text:0042E4EE                                         ;  
.text:0042E4EE                                         ;  
.text:0042E4EE                                         ;  
.text:0042E4F2                 mov     esi, \[ebx+UI32.\_3\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4 次运算开始\*\*\*\*  
.text:0042E4F5                 mov     edi, \[ebx+UI32.\_3\] ; buf\[3\]  
.text:0042E4F8                 shr     edi, 8          ; buf\[3\]>>8  
.text:0042E4FB                 mov     ebp, \[ebx+UI32.\_3\]  
.text:0042E4FE                 shl     esi, 18h        ; buf\[3\]<<24  
.text:0042E501                 or      esi, edi        ; RL(buf\[3\],24) = (buf\[3\]<<24) | (buf\[3\]>>(32-24))  
.text:0042E503                 mov     edi, \[ebx+UI32.\_3\]  
.text:0042E506                 shl     edi, 8          ; buf\[3\]<<8  
.text:0042E509                 and     esi, 0FF00FF00h ; RL(buf\[3\],24) & 0xff00ff00  
.text:0042E50F                 shr     ebp, 18h        ; buf\[3\]>>24  
.text:0042E512                 or      edi, ebp        ; RL(buf\[3\],8) = (buf\[3\]<<8) | (buf\[3\]>>(32-8))  
.text:0042E514                 mov     ebp, edx\_C  
.text:0042E516                 and     edi, 0FF00FFh   ; RL(buf\[3\],8) &0x0ff00ff  
.text:0042E51C                 or      esi, edi        ; RE(buf\[3\]) = (RL(buf\[3\],24)&0xff00ff00) | (RL(buf\[3\],8)&0xff00ff)  
.text:0042E51E                 mov     \[ebx+UI32.\_3\], esi ; buf\[3\] = RE(buf\[3\])  
.text:0042E521                 mov     edi, \[esp+14h+E\]  
.text:0042E525                 xor     edi, \[esp+14h+A\] ; (E^A)  
.text:0042E529                 and     edi, ecx\_D      ; (E^A)&D  
.text:0042E52B                 xor     edi, \[esp+14h+A\] ; RX(D,E,A) = (E^A)&D^A  
.text:0042E52F                 add     esi, edi        ; RE(buf\[3\]) + RX(D,E,A)  
.text:0042E531                 mov     edi, edx\_C  
.text:0042E533                 shl     edi, 5          ; C<<5  
.text:0042E536                 shr     ebp, 1Bh        ; C>>27  
.text:0042E539                 or      edi, ebp        ; RL(C,5) = (C<<5) | (C>>(32-5))  
.text:0042E53B                 mov     ebp, \[ebx+UI32.\_4\] ; buf\[4\]  
.text:0042E53E                 add     esi, edi        ; RE(buf\[3\]) + RX(D,E,A) + RL(C,5)  
.text:0042E540                 mov     edi, \[ebx+UI32.\_4\]  
.text:0042E543                 add     esi, 5A827999h ; <suspicious>  
.text:0042E549                 add     eax\_B, esi      ; B += RE(buf\[3\]) + RX(D,E,A) + RL(C,5) + 0x5a827999  
.text:0042E54B                 mov     esi, ecx\_D  
.text:0042E54D                 shr     ecx\_D, 2        ; D>>2  
.text:0042E550                 shl     esi, 1Eh        ; D<<30  
.text:0042E553                 or      esi, ecx\_D      ; RL(D,30) = (D<<30) | (D>>(32-30))  
.text:0042E555                 mov     ecx\_D, esi      ; D = RL(D,30)  
.text:0042E555                                         ;  
.text:0042E555                                         ; 整个第 4 次运算用函数 R0 表示: R0(C,D,E,A,B, &buf\[3\])  
.text:0042E555                                         ;  
.text:0042E555                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4 次运算结束\*\*\*\*  
.text:0042E555                                         ;  
.text:0042E555                                         ;  
.text:0042E557                 mov     esi, \[ebx+UI32.\_4\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 5 次运算开始\*\*\*\*  
.text:0042E557                                         ; buf\[4\]  
.text:0042E55A                 shl     esi, 18h        ; buf\[4\]<<24  
.text:0042E55D                 shr     edi, 8          ; buf\[4\]>>8  
.text:0042E560                 or      esi, edi        ; RL(buf\[4\],24)  
.text:0042E562                 mov     edi, \[ebx+UI32.\_4\]  
.text:0042E565                 shl     edi, 8          ; buf\[4\]<<8  
.text:0042E568                 and     esi, 0FF00FF00h ; RL(buf\[4\],24) &0xff00ff00  
.text:0042E56E                 shr     ebp, 18h        ; buf\[4\]>>24  
.text:0042E571                 or      edi, ebp        ; RL(buf\[4\],8)  
.text:0042E573                 mov     ebp, eax\_B  
.text:0042E575                 and     edi, 0FF00FFh   ; RL(buf\[4\],8)&0x0ff00ff  
.text:0042E57B                 or      esi, edi        ; RE(buf\[4\])  
.text:0042E57D                 mov     \[ebx+UI32.\_4\], esi ; buf\[4\] = RE(buf\[4\])  
.text:0042E580                 mov     edi, \[esp+14h+E\]  
.text:0042E584                 xor     edi, ecx\_D      ; (E^D)  
.text:0042E586                 and     edi, edx\_C      ; (E^D)&C  
.text:0042E588                 xor     edi, \[esp+14h+E\] ; RX(C,D,E) = ((D^E)&C)^E  
.text:0042E58C                 add     esi, edi        ; RE(buf\[4\]) + RX(C,D,E)  
.text:0042E58E                 mov     edi, eax\_B  
.text:0042E590                 shl     edi, 5          ; B<<5  
.text:0042E593                 shr     ebp, 1Bh        ; B>>27  
.text:0042E596                 or      edi, ebp        ; RL(B,5)  
.text:0042E598                 add     esi, edi        ; RE(buf\[4\]) + RX(C,D,E) + RL(B,5)  
.text:0042E59A                 add     esi, 5A827999h ; <suspicious>  
.text:0042E5A0                 add     \[esp+14h+A\], esi ; A += RE(buf\[4\]) + RX(C,D,E) + RL(B,5) + 0x5a827999;  
.text:0042E5A4                 mov     esi, edx\_C  
.text:0042E5A6                 shr     edx\_C, 2        ; C>>2  
.text:0042E5A9                 mov     edi, \[ebx+UI32.\_5\] ; buf\[5\]  
.text:0042E5AC                 shl     esi, 1Eh        ; C<<30  
.text:0042E5AF                 mov     ebp, \[ebx+UI32.\_5\] ; buf\[5\]  
.text:0042E5B2                 or      esi, edx\_C      ; RL(C,30)  
.text:0042E5B4                 mov     edx\_C, esi      ; C = RL(C,30);  
.text:0042E5B4                                         ;  
.text:0042E5B4                                         ; (5) R0(B,C,D,E,A, &buf\[4\]);  
.text:0042E5B4                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 5 次运算结束\*\*\*\*  
.text:0042E5B4                                         ;  
.text:0042E5B4                                         ;  
.text:0042E5B6                 mov     esi, \[ebx+UI32.\_5\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 6 次运算开始\*\*\*\*  
.text:0042E5B9                 shl     esi, 18h        ; buf\[5\]<<24  
.text:0042E5BC                 shr     edi, 8          ; buf\[5\]>>8  
.text:0042E5BF                 or      esi, edi        ; RL(buf\[5\],24)  
.text:0042E5C1                 mov     edi, \[ebx+UI32.\_5\] ; buf\[5\]  
.text:0042E5C4                 shl     edi, 8          ; buf\[5\]<<8  
.text:0042E5C7                 and     esi, 0FF00FF00h ; RL(buf\[5\],24)&0xff00ff00  
.text:0042E5CD                 shr     ebp, 18h        ; buf\[5\]>>24  
.text:0042E5D0                 or      edi, ebp        ; RL(buf\[5\],8)  
.text:0042E5D2                 and     edi, 0FF00FFh   ; RL(buf\[5\],8) & 0x0ff00ff  
.text:0042E5D8                 or      esi, edi        ; RE(buf\[5\])  
.text:0042E5DA                 mov     edi, ecx\_D  
.text:0042E5DC                 xor     edi, edx\_C      ; (C^D)  
.text:0042E5DE                 mov     \[ebx+UI32.\_5\], esi ; buf\[5\] = RE(buf\[5\])  
.text:0042E5E1                 and     edi, eax\_B      ; (C^D)&B  
.text:0042E5E3                 mov     ebp, \[esp+14h+A\]  
.text:0042E5E7                 xor     edi, ecx\_D      ; RX(B,C,D) = ((C^D)&B)^D  
.text:0042E5E9                 add     esi, edi        ; RE(buf\[5\]) + RX(B,C,D)  
.text:0042E5EB                 mov     edi, \[esp+14h+A\]  
.text:0042E5EF                 shl     edi, 5          ; A<<5  
.text:0042E5F2                 shr     ebp, 1Bh        ; A>>27  
.text:0042E5F5                 or      edi, ebp        ; RL(A,5)  
.text:0042E5F7                 add     esi, edi        ; RE(buf\[5\]) + RX(B,C,D) + RL(A,5)  
.text:0042E5F9                 add     esi, 5A827999h ; <suspicious>  
.text:0042E5FF                 add     \[esp+14h+E\], esi ; E += RE(buf\[5\]) + RX(B,C,D) + RL(A,5) + 0x5a827999;  
.text:0042E603                 mov     esi, eax\_B  
.text:0042E605                 shr     eax\_B, 2        ; B>>2  
.text:0042E608                 mov     edi, \[ebx+UI32.\_6\]  
.text:0042E60B                 shl     esi, 1Eh        ; B<<30  
.text:0042E60E                 mov     ebp, \[ebx+UI32.\_6\]  
.text:0042E611                 or      esi, eax\_B      ; RL(B,30)  
.text:0042E613                 mov     eax\_B, esi      ; B = RL(B,30);  
.text:0042E613                                         ; (6). R0(A,B,C,D,E, &buf\[5\])  
.text:0042E613                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 6 次运算结束\*\*\*\*  
.text:0042E613                                         ;  
.text:0042E613                                         ;  
.text:0042E615                 mov     esi, \[ebx+UI32.\_6\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 7 次运算开始\*\*\*\*  
.text:0042E618                 shl     esi, 18h  
.text:0042E61B                 shr     edi, 8  
.text:0042E61E                 or      esi, edi  
.text:0042E620                 mov     edi, \[ebx+UI32.\_6\]  
.text:0042E623                 shl     edi, 8  
.text:0042E626                 and     esi, 0FF00FF00h  
.text:0042E62C                 shr     ebp, 18h  
.text:0042E62F                 or      edi, ebp  
.text:0042E631                 and     edi, 0FF00FFh  
.text:0042E637                 or      esi, edi  
.text:0042E639                 mov     edi, edx\_C  
.text:0042E63B                 mov     \[ebx+UI32.\_6\], esi  
.text:0042E63E                 xor     edi, eax\_B  
.text:0042E640                 and     edi, \[esp+14h+A\]  
.text:0042E644                 mov     ebp, \[esp+14h+E\]  
.text:0042E648                 xor     edi, edx\_C  
.text:0042E64A                 add     esi, edi  
.text:0042E64C                 mov     edi, \[esp+14h+E\]  
.text:0042E650                 shl     edi, 5  
.text:0042E653                 shr     ebp, 1Bh  
.text:0042E656                 or      edi, ebp  
.text:0042E658                 add     esi, edi  
.text:0042E65A                 mov     edi, \[esp+14h+A\]  
.text:0042E65E                 add     esi, 5A827999h ; <suspicious>  
.text:0042E664                 add     ecx\_D, esi  
.text:0042E666                 mov     esi, \[esp+14h+A\]  
.text:0042E66A                 shl     esi, 1Eh  
.text:0042E66D                 shr     edi, 2  
.text:0042E670                 or      esi, edi  
.text:0042E672                 mov     \[esp+14h+A\], esi ; (7). R0(E,A,B,C,D, &buf\[6\])  
.text:0042E672                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 7 次运算结束\*\*\*\*  
.text:0042E672                                         ;  
.text:0042E672                                         ;  
.text:0042E676                 mov     esi, \[ebx+UI32.\_7\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 8 次运算开始\*\*\*\*  
.text:0042E679                 mov     edi, \[ebx+UI32.\_7\]  
.text:0042E67C                 shr     edi, 8  
.text:0042E67F                 mov     ebp, \[ebx+UI32.\_7\]  
.text:0042E682                 shl     esi, 18h  
.text:0042E685                 or      esi, edi  
.text:0042E687                 mov     edi, \[ebx+UI32.\_7\]  
.text:0042E68A                 shr     ebp, 18h  
.text:0042E68D                 and     esi, 0FF00FF00h  
.text:0042E693                 shl     edi, 8  
.text:0042E696                 or      edi, ebp  
.text:0042E698                 and     edi, 0FF00FFh  
.text:0042E69E                 or      esi, edi  
.text:0042E6A0                 mov     \[ebx+UI32.\_7\], esi  
.text:0042E6A3                 mov     edi, \[esp+14h+A\]  
.text:0042E6A7                 xor     edi, eax\_B  
.text:0042E6A9                 and     edi, \[esp+14h+E\]  
.text:0042E6AD                 mov     ebp, ecx\_D  
.text:0042E6AF                 xor     edi, eax\_B  
.text:0042E6B1                 add     esi, edi  
.text:0042E6B3                 mov     edi, ecx\_D  
.text:0042E6B5                 shl     edi, 5  
.text:0042E6B8                 shr     ebp, 1Bh  
.text:0042E6BB                 or      edi, ebp  
.text:0042E6BD                 add     esi, edi  
.text:0042E6BF                 mov     edi, \[esp+14h+E\]  
.text:0042E6C3                 add     esi, 5A827999h ; <suspicious>  
.text:0042E6C9                 add     edx\_C, esi  
.text:0042E6CB                 mov     esi, \[esp+14h+E\]  
.text:0042E6CF                 shl     esi, 1Eh  
.text:0042E6D2                 shr     edi, 2  
.text:0042E6D5                 or      esi, edi  
.text:0042E6D7                 mov     \[esp+14h+E\], esi ; (8). R0(D,E,A,B,C, &buf\[7\])  
.text:0042E6D7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 8 次运算结束\*\*\*\*  
.text:0042E6D7                                         ;  
.text:0042E6D7                                         ;  
.text:0042E6DB                 mov     esi, \[ebx+UI32.\_8\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 9 次运算开始\*\*\*\*  
.text:0042E6DE                 mov     edi, \[ebx+UI32.\_8\]  
.text:0042E6E1                 shr     edi, 8  
.text:0042E6E4                 mov     ebp, \[ebx+UI32.\_8\]  
.text:0042E6E7                 shl     esi, 18h  
.text:0042E6EA                 or      esi, edi  
.text:0042E6EC                 mov     edi, \[ebx+UI32.\_8\]  
.text:0042E6EF                 shl     edi, 8  
.text:0042E6F2                 and     esi, 0FF00FF00h  
.text:0042E6F8                 shr     ebp, 18h  
.text:0042E6FB                 or      edi, ebp  
.text:0042E6FD                 mov     ebp, edx\_C  
.text:0042E6FF                 and     edi, 0FF00FFh  
.text:0042E705                 or      esi, edi  
.text:0042E707                 mov     \[ebx+UI32.\_8\], esi  
.text:0042E70A                 mov     edi, \[esp+14h+E\]  
.text:0042E70E                 xor     edi, \[esp+14h+A\]  
.text:0042E712                 and     edi, ecx\_D  
.text:0042E714                 xor     edi, \[esp+14h+A\]  
.text:0042E718                 add     esi, edi  
.text:0042E71A                 mov     edi, edx\_C  
.text:0042E71C                 shl     edi, 5  
.text:0042E71F                 shr     ebp, 1Bh  
.text:0042E722                 or      edi, ebp  
.text:0042E724                 mov     ebp, \[ebx+UI32.\_9\]  
.text:0042E727                 add     esi, edi  
.text:0042E729                 mov     edi, \[ebx+UI32.\_9\]  
.text:0042E72C                 add     esi, 5A827999h ; <suspicious>  
.text:0042E732                 add     eax\_B, esi  
.text:0042E734                 mov     esi, ecx\_D  
.text:0042E736                 shr     ecx\_D, 2  
.text:0042E739                 shl     esi, 1Eh  
.text:0042E73C                 or      esi, ecx\_D  
.text:0042E73E                 mov     ecx\_D, esi      ; (9). R0(C,D,E,A,B, &buf\[8\])  
.text:0042E73E                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 9 次运算结束\*\*\*\*  
.text:0042E73E                                         ;  
.text:0042E73E                                         ;  
.text:0042E740                 mov     esi, \[ebx+UI32.\_9\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 10 次运算开始\*\*\*\*  
.text:0042E743                 shl     esi, 18h  
.text:0042E746                 shr     edi, 8  
.text:0042E749                 or      esi, edi  
.text:0042E74B                 mov     edi, \[ebx+UI32.\_9\]  
.text:0042E74E                 shl     edi, 8  
.text:0042E751                 and     esi, 0FF00FF00h  
.text:0042E757                 shr     ebp, 18h  
.text:0042E75A                 or      edi, ebp  
.text:0042E75C                 mov     ebp, eax\_B  
.text:0042E75E                 and     edi, 0FF00FFh  
.text:0042E764                 or      esi, edi  
.text:0042E766                 mov     \[ebx+UI32.\_9\], esi  
.text:0042E769                 mov     edi, \[esp+14h+E\]  
.text:0042E76D                 xor     edi, ecx\_D  
.text:0042E76F                 and     edi, edx\_C  
.text:0042E771                 xor     edi, \[esp+14h+E\]  
.text:0042E775                 add     esi, edi  
.text:0042E777                 mov     edi, eax\_B  
.text:0042E779                 shl     edi, 5  
.text:0042E77C                 shr     ebp, 1Bh  
.text:0042E77F                 or      edi, ebp  
.text:0042E781                 add     esi, edi  
.text:0042E783                 add     esi, 5A827999h ; <suspicious>  
.text:0042E789                 add     \[esp+14h+A\], esi  
.text:0042E78D                 mov     esi, edx\_C  
.text:0042E78F                 shr     edx\_C, 2  
.text:0042E792                 mov     edi, \[ebx+UI32.\_10\]  
.text:0042E795                 shl     esi, 1Eh  
.text:0042E798                 mov     ebp, \[ebx+UI32.\_10\]  
.text:0042E79B                 or      esi, edx\_C  
.text:0042E79D                 mov     edx\_C, esi      ; (10). R0(B,C,D,E,A, &buf\[9\])  
.text:0042E79D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 10 次运算 end\*\*\*\*  
.text:0042E79D                                         ;  
.text:0042E79D                                         ;  
.text:0042E79F                 mov     esi, \[ebx+UI32.\_10\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 11 次运算开始\*\*\*\*  
.text:0042E7A2                 shl     esi, 18h  
.text:0042E7A5                 shr     edi, 8  
.text:0042E7A8                 or      esi, edi  
.text:0042E7AA                 mov     edi, \[ebx+UI32.\_10\]  
.text:0042E7AD                 shl     edi, 8  
.text:0042E7B0                 and     esi, 0FF00FF00h  
.text:0042E7B6                 shr     ebp, 18h  
.text:0042E7B9                 or      edi, ebp  
.text:0042E7BB                 and     edi, 0FF00FFh  
.text:0042E7C1                 or      esi, edi  
.text:0042E7C3                 mov     edi, ecx\_D  
.text:0042E7C5                 xor     edi, edx\_C  
.text:0042E7C7                 mov     \[ebx+UI32.\_10\], esi  
.text:0042E7CA                 and     edi, eax\_B  
.text:0042E7CC                 mov     ebp, \[esp+14h+A\]  
.text:0042E7D0                 xor     edi, ecx\_D  
.text:0042E7D2                 add     esi, edi  
.text:0042E7D4                 mov     edi, \[esp+14h+A\]  
.text:0042E7D8                 shl     edi, 5  
.text:0042E7DB                 shr     ebp, 1Bh  
.text:0042E7DE                 or      edi, ebp  
.text:0042E7E0                 add     esi, edi  
.text:0042E7E2                 add     esi, 5A827999h ; <suspicious>  
.text:0042E7E8                 add     \[esp+14h+E\], esi  
.text:0042E7EC                 mov     esi, eax\_B  
.text:0042E7EE                 shr     eax\_B, 2  
.text:0042E7F1                 mov     edi, \[ebx+UI32.\_11\]  
.text:0042E7F4                 shl     esi, 1Eh  
.text:0042E7F7                 mov     ebp, \[ebx+UI32.\_11\]  
.text:0042E7FA                 or      esi, eax\_B  
.text:0042E7FC                 mov     eax\_B, esi      ; (11). R0(A,B,C,D,E, &buf\[10\])  
.text:0042E7FC                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 11 次运算 end\*\*\*\*  
.text:0042E7FC                                         ;  
.text:0042E7FC                                         ;  
.text:0042E7FE                 mov     esi, \[ebx+UI32.\_11\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 12 次运算开始\*\*\*\*  
.text:0042E801                 shl     esi, 18h  
.text:0042E804                 shr     edi, 8  
.text:0042E807                 or      esi, edi  
.text:0042E809                 mov     edi, \[ebx+UI32.\_11\]  
.text:0042E80C                 shl     edi, 8  
.text:0042E80F                 and     esi, 0FF00FF00h  
.text:0042E815                 shr     ebp, 18h  
.text:0042E818                 or      edi, ebp  
.text:0042E81A                 and     edi, 0FF00FFh  
.text:0042E820                 or      esi, edi  
.text:0042E822                 mov     edi, edx\_C  
.text:0042E824                 mov     \[ebx+UI32.\_11\], esi  
.text:0042E827                 xor     edi, eax\_B  
.text:0042E829                 and     edi, \[esp+14h+A\]  
.text:0042E82D                 mov     ebp, \[esp+14h+E\]  
.text:0042E831                 xor     edi, edx\_C  
.text:0042E833                 add     esi, edi  
.text:0042E835                 mov     edi, \[esp+14h+E\]  
.text:0042E839                 shl     edi, 5  
.text:0042E83C                 shr     ebp, 1Bh  
.text:0042E83F                 or      edi, ebp  
.text:0042E841                 add     esi, edi  
.text:0042E843                 mov     edi, \[esp+14h+A\]  
.text:0042E847                 add     esi, 5A827999h ; <suspicious>  
.text:0042E84D                 add     ecx\_D, esi  
.text:0042E84F                 mov     esi, \[esp+14h+A\]  
.text:0042E853                 shl     esi, 1Eh  
.text:0042E856                 shr     edi, 2  
.text:0042E859                 or      esi, edi  
.text:0042E85B                 mov     \[esp+14h+A\], esi ; (12). R0(E,A,B,C,D, &buf\[11\])  
.text:0042E85B                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 12 次运算 end\*\*\*\*  
.text:0042E85B                                         ;  
.text:0042E85B                                         ;  
.text:0042E85F                 mov     esi, \[ebx+UI32.\_12\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 13 次运算开始\*\*\*\*  
.text:0042E862                 mov     edi, \[ebx+UI32.\_12\]  
.text:0042E865                 shr     edi, 8  
.text:0042E868                 mov     ebp, \[ebx+UI32.\_12\]  
.text:0042E86B                 shl     esi, 18h  
.text:0042E86E                 or      esi, edi  
.text:0042E870                 mov     edi, \[ebx+UI32.\_12\]  
.text:0042E873                 shl     edi, 8  
.text:0042E876                 and     esi, 0FF00FF00h  
.text:0042E87C                 shr     ebp, 18h  
.text:0042E87F                 or      edi, ebp  
.text:0042E881                 mov     ebp, ecx\_D  
.text:0042E883                 and     edi, 0FF00FFh  
.text:0042E889                 or      esi, edi  
.text:0042E88B                 mov     \[ebx+UI32.\_12\], esi  
.text:0042E88E                 mov     edi, \[esp+14h+A\]  
.text:0042E892                 xor     edi, eax\_B  
.text:0042E894                 and     edi, \[esp+14h+E\]  
.text:0042E898                 xor     edi, eax\_B  
.text:0042E89A                 add     esi, edi  
.text:0042E89C                 mov     edi, ecx\_D  
.text:0042E89E                 shl     edi, 5  
.text:0042E8A1                 shr     ebp, 1Bh  
.text:0042E8A4                 or      edi, ebp  
.text:0042E8A6                 add     esi, edi  
.text:0042E8A8                 mov     edi, \[esp+14h+E\]  
.text:0042E8AC                 add     esi, 5A827999h ; <suspicious>  
.text:0042E8B2                 add     edx\_C, esi  
.text:0042E8B4                 mov     esi, \[esp+14h+E\]  
.text:0042E8B8                 shl     esi, 1Eh  
.text:0042E8BB                 shr     edi, 2  
.text:0042E8BE                 or      esi, edi  
.text:0042E8C0                 mov     \[esp+14h+E\], esi ; (13). R0(D,E,A,B,C, &buf\[12\])  
.text:0042E8C0                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 13 次运算 end\*\*\*\*  
.text:0042E8C0                                         ;  
.text:0042E8C0                                         ;  
.text:0042E8C4                 mov     esi, \[ebx+UI32.\_13\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 14 次运算开始\*\*\*\*  
.text:0042E8C7                 mov     edi, \[ebx+UI32.\_13\]  
.text:0042E8CA                 shr     edi, 8  
.text:0042E8CD                 mov     ebp, \[ebx+UI32.\_13\]  
.text:0042E8D0                 shl     esi, 18h  
.text:0042E8D3                 or      esi, edi  
.text:0042E8D5                 mov     edi, \[ebx+UI32.\_13\]  
.text:0042E8D8                 shl     edi, 8  
.text:0042E8DB                 and     esi, 0FF00FF00h  
.text:0042E8E1                 shr     ebp, 18h  
.text:0042E8E4                 or      edi, ebp  
.text:0042E8E6                 mov     ebp, edx\_C  
.text:0042E8E8                 and     edi, 0FF00FFh  
.text:0042E8EE                 or      esi, edi  
.text:0042E8F0                 mov     \[ebx+UI32.\_13\], esi  
.text:0042E8F3                 mov     edi, \[esp+14h+E\]  
.text:0042E8F7                 xor     edi, \[esp+14h+A\]  
.text:0042E8FB                 and     edi, ecx\_D  
.text:0042E8FD                 xor     edi, \[esp+14h+A\]  
.text:0042E901                 add     esi, edi  
.text:0042E903                 mov     edi, edx\_C  
.text:0042E905                 shl     edi, 5  
.text:0042E908                 shr     ebp, 1Bh  
.text:0042E90B                 or      edi, ebp  
.text:0042E90D                 mov     ebp, \[ebx+UI32.\_14\]  
.text:0042E910                 add     esi, edi  
.text:0042E912                 mov     edi, \[ebx+UI32.\_14\]  
.text:0042E915                 add     esi, 5A827999h ; <suspicious>  
.text:0042E91B                 add     eax\_B, esi  
.text:0042E91D                 mov     esi, ecx\_D  
.text:0042E91F                 shr     ecx\_D, 2  
.text:0042E922                 shl     esi, 1Eh  
.text:0042E925                 or      esi, ecx\_D  
.text:0042E927                 mov     ecx\_D, esi      ; (14). R0(C,D,E,A,B, &buf\[13\])  
.text:0042E927                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 14 次运算 end\*\*\*\*  
.text:0042E927                                         ;  
.text:0042E927                                         ;  
.text:0042E929                 mov     esi, \[ebx+UI32.\_14\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 15 次运算开始\*\*\*\*  
.text:0042E92C                 shl     esi, 18h  
.text:0042E92F                 shr     edi, 8  
.text:0042E932                 or      esi, edi  
.text:0042E934                 mov     edi, \[ebx+UI32.\_14\]  
.text:0042E937                 shl     edi, 8  
.text:0042E93A                 and     esi, 0FF00FF00h  
.text:0042E940                 shr     ebp, 18h  
.text:0042E943                 or      edi, ebp  
.text:0042E945                 mov     ebp, eax\_B  
.text:0042E947                 and     edi, 0FF00FFh  
.text:0042E94D                 or      esi, edi  
.text:0042E94F                 mov     \[ebx+UI32.\_14\], esi  
.text:0042E952                 mov     edi, \[esp+14h+E\]  
.text:0042E956                 xor     edi, ecx\_D  
.text:0042E958                 and     edi, edx\_C  
.text:0042E95A                 xor     edi, \[esp+14h+E\]  
.text:0042E95E                 add     esi, edi  
.text:0042E960                 mov     edi, eax\_B  
.text:0042E962                 shl     edi, 5  
.text:0042E965                 shr     ebp, 1Bh  
.text:0042E968                 or      edi, ebp  
.text:0042E96A                 add     esi, edi  
.text:0042E96C                 add     esi, 5A827999h ; <suspicious>  
.text:0042E972                 add     \[esp+14h+A\], esi  
.text:0042E976                 mov     esi, edx\_C  
.text:0042E978                 shr     edx\_C, 2  
.text:0042E97B                 mov     edi, \[ebx+UI32.\_15\]  
.text:0042E97E                 shl     esi, 1Eh  
.text:0042E981                 mov     ebp, \[ebx+UI32.\_15\]  
.text:0042E984                 or      esi, edx\_C  
.text:0042E986                 mov     edx\_C, esi      ; (15). R0(B,C,D,E,A, &buf\[14\])  
.text:0042E986                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 15 次运算 end\*\*\*\*  
.text:0042E986                                         ;  
.text:0042E986                                         ;  
.text:0042E988                 mov     esi, \[ebx+UI32.\_15\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 16 次运算开始\*\*\*\*  
.text:0042E98B                 shl     esi, 18h        ; buf\[15\]<<24  
.text:0042E98E                 shr     edi, 8          ; buf\[15\]>>8  
.text:0042E991                 or      esi, edi        ; RL(buf\[15\],24)  
.text:0042E993                 mov     edi, \[ebx+UI32.\_15\]  
.text:0042E996                 shr     ebp, 18h        ; buf\[15\]>>24  
.text:0042E999                 and     esi, 0FF00FF00h ; RL(buf\[15\],24)&0xff00ff00  
.text:0042E99F                 shl     edi, 8          ; buf\[15\]<<8  
.text:0042E9A2                 or      edi, ebp        ; RL(buf\[15\],8)  
.text:0042E9A4                 and     edi, 0FF00FFh   ; RL(buf\[15\],8)&0xff00ff  
.text:0042E9AA                 or      esi, edi        ; RE(buf\[15\])  
.text:0042E9AC                 mov     edi, ecx\_D  
.text:0042E9AE                 xor     edi, edx\_C      ; (C^D)  
.text:0042E9B0                 mov     \[ebx+UI32.\_15\], esi ; buf\[15\] = RE(buf\[15\])  
.text:0042E9B3                 and     edi, eax\_B      ; (C^D)&B  
.text:0042E9B5                 mov     ebp, \[esp+14h+A\]  
.text:0042E9B9                 xor     edi, ecx\_D      ; RX(B,C,D) = ((C^D)&B)^D  
.text:0042E9BB                 add     esi, edi        ; RE(buf\[15\]) + RX(B,C,D)  
.text:0042E9BD                 mov     edi, \[esp+14h+A\]  
.text:0042E9C1                 shl     edi, 5          ; A<<5  
.text:0042E9C4                 shr     ebp, 1Bh        ; A>>30  
.text:0042E9C7                 or      edi, ebp        ; RL(A,5)  
.text:0042E9C9                 add     esi, edi        ; RE(buf\[15\]) + RX(B,C,D) + RL(A,5)  
.text:0042E9CB                 add     esi, 5A827999h ; <suspicious>  
.text:0042E9D1                 add     \[esp+14h+E\], esi ; E += RE(buf\[15\]) + RX(B,C,D) + RL(A,5) + 0x5a827999;  
.text:0042E9D5                 mov     esi, eax\_B  
.text:0042E9D7                 shr     eax\_B, 2        ; B>>2  
.text:0042E9DA                 mov     edi, \[ebx+UI32.\_13\]  
.text:0042E9DD                 shl     esi, 1Eh        ; B<<30  
.text:0042E9E0                 xor     edi, \[ebx+UI32.\_8\] ; buf\[13\] ^ buf\[8\]  
.text:0042E9E3                 or      esi, eax\_B      ; RL(B,30)  
.text:0042E9E5                 xor     edi, \[ebx+UI32.\_2\] ; buf\[13\] ^ buf\[8\] ^ buf\[2\]  
.text:0042E9E8                 mov     eax\_B, esi      ; B = RL(B,30)  
.text:0042E9E8                                         ; (16). R0(A,B,C,D,E, &buf\[15\])  
.text:0042E9E8                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 16 次运算 end\*\*\*\*  
.text:0042E9E8                                         ;  
.text:0042E9E8                                         ;  
.text:0042E9EA                 mov     esi, \[ebx+UI32.\_13\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 17 次运算开始\*\*\*\*  
.text:0042E9ED                 xor     esi, \[ebx+UI32.\_8\] ; buf\[13\]^buf\[8\]  
.text:0042E9F0                 xor     edi, \[ebx+UI32.\_0\] ; buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]  
.text:0042E9F2                 xor     esi, \[ebx+UI32.\_2\] ; buf\[13\] ^ buf\[8\] ^ buf\[2\]  
.text:0042E9F5                 xor     esi, \[ebx+UI32.\_0\] ; buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]  
.text:0042E9F7                 shr     edi, 1Fh        ; (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]) >> 31  
.text:0042E9FA                 add     esi, esi        ; (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\])<<1  
.text:0042E9FA                                         ; 这里把++转换成左移 1bit 是为了下面用 RL 运算  
.text:0042E9FC                 or      esi, edi        ; RL((buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]),1) =  
.text:0042E9FC                                         ; ((buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\])<<1) |  
.text:0042E9FC                                         ; ((buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\])>>(32-1))  
.text:0042E9FE                 mov     edi, edx\_C  
.text:0042EA00                 mov     \[ebx+UI32.\_0\], esi ; buf\[0\] = RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]), 1)  
.text:0042EA00                                         ; buf 又从\[0\]开始参与运算  
.text:0042EA02                 xor     edi, eax\_B      ; (B^C)  
.text:0042EA04                 and     edi, \[esp+14h+A\] ; (B^C)&A  
.text:0042EA08                 mov     ebp, \[esp+14h+E\]  
.text:0042EA0C                 xor     edi, edx\_C      ; RX(A,B,C) = ((B^C)&A)^C  
.text:0042EA0E                 add     esi, edi        ; RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]),1) + RX(A,B,C)  
.text:0042EA10                 mov     edi, \[esp+14h+E\]  
.text:0042EA14                 shl     edi, 5          ; E<<5  
.text:0042EA17                 shr     ebp, 1Bh        ; E>>27  
.text:0042EA1A                 or      edi, ebp        ; RL(E,5)  
.text:0042EA1C                 add     esi, edi        ; RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]),1) + RX(A,B,C) + RL(E,5)  
.text:0042EA1E                 mov     edi, \[esp+14h+A\]  
.text:0042EA22                 add     esi, 5A827999h ; <suspicious>  
.text:0042EA28                 add     ecx\_D, esi      ; D += RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]),1) + RX(A,B,C) + RL(E,5) + 0x5a827999  
.text:0042EA2A                 mov     esi, \[esp+14h+A\]  
.text:0042EA2E                 shl     esi, 1Eh        ; A<<30  
.text:0042EA31                 mov     ebp, ecx\_D  
.text:0042EA33                 shr     edi, 2          ; A>>2  
.text:0042EA36                 or      esi, edi        ; RL(A,30)  
.text:0042EA38                 mov     \[esp+14h+A\], esi ; A = RL(A,30)  
.text:0042EA38                                         ; 观察第 17-20 次运算不难得到如下数据:  
.text:0042EA38                                         ; 17  
.text:0042EA38                                         ; buf\[0\] = RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]), 1)  
.text:0042EA38                                         ; D += RL( (buf\[13\] ^ buf\[8\] ^ buf\[2\] ^ buf\[0\]),1) + RX(A,B,C) + RL(E,5) + 0x5a827999  
.text:0042EA38                                         ; A = RL(A,30)  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; 18  
.text:0042EA38                                         ; buf\[1\] = RL( (buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1)  
.text:0042EA38                                         ; C += RL((buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1) + RX(E,A,B) + RL(D,5) + 0x5a827999;  
.text:0042EA38                                         ; E = RL(E,30)  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; 19  
.text:0042EA38                                         ; buf\[2\] = RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1)  
.text:0042EA38                                         ; B += RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1) + RX(D,E,A) + 0x5a827999  
.text:0042EA38                                         ; D = RL(D,30)  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; 20  
.text:0042EA38                                         ; buf\[3\] = RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1)  
.text:0042EA38                                         ; A += RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1) + RX(C,D,E) + RL(B,5) + 0x5a827999  
.text:0042EA38                                         ; C = RL(C,30)  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; (宏定义参考了 Steve Reid 的 SHA1 代码)  
.text:0042EA38                                         ; 总结定义宏 RXL:  
.text:0042EA38                                         ; #define RXL(i)  
.text:0042EA38                                         ; ( buf\[i&15\] = RL( (buf\[(i+13)&15\] ^ buf\[(i+8)&15\] ^ buf\[(i+2)&15\] ^ buf\[i&15\]), 1) )  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; 定义宏 R1:  
.text:0042EA38                                         ; #define R1(a,b,c,d,e, i)  
.text:0042EA38                                         ; ( e+=RXL(i)+RX(b,c,d)+RL(a,5)+0x5a827999;b=RL(b,30); )  
.text:0042EA38                                         ;  
.text:0042EA38                                         ; 那么第 17 次运算表示为:  
.text:0042EA38                                         ; R1(E,A,B,C,D, 16)  
.text:0042EA38                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 17 次运算 end\*\*\*\*  
.text:0042EA38                                         ;  
.text:0042EA38                                         ;  
.text:0042EA3C                 mov     edi, \[ebx+UI32.\_14\] ; 根据 R1,RXL 宏, 修改 RE,R0:  
.text:0042EA3C                                         ;  
.text:0042EA3C                                         ; #define RE(i) ( buf\[i\] = (RL(buf\[i\],24)&0xff00ff00) | (RL(buf\[i\],8)&0x0ff00ff) )  
.text:0042EA3C                                         ;  
.text:0042EA3C                                         ; #define R0(a,b,c,d,e,i) ( e+=RE(i)+RX(b,c,d)+RL(a,5)+0x5a827999; b=RL(b,30); )  
.text:0042EA3C                                         ;  
.text:0042EA3C                                         ; 那前 16 次就要改为:  
.text:0042EA3C                                         ; R0(A,B,C,D,E,0);  
.text:0042EA3C                                         ; R0(E,A,B,C,D,1);  
.text:0042EA3C                                         ; R0(D,E,A,B,C,2);  
.text:0042EA3C                                         ; R0(C,D,E,A,B,3);  
.text:0042EA3C                                         ; R0(B,C,D,E,A,4);  
.text:0042EA3C                                         ; R0(A,B,C,D,E,5);  
.text:0042EA3C                                         ; ....  
.text:0042EA3C                                         ; R0(A,B,C,D,E,15)  
.text:0042EA3C                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 18 次运算开始\*\*\*\*  
.text:0042EA3F                 mov     esi, \[ebx+UI32.\_14\]  
.text:0042EA42                 xor     edi, \[ebx+UI32.\_9\] ; buf\[14\] ^ buf\[9\]  
.text:0042EA45                 xor     esi, \[ebx+UI32.\_9\] ; buf\[14\] ^ buf\[9\]  
.text:0042EA48                 xor     edi, \[ebx+UI32.\_3\] ; buf\[14\] ^ buf\[9\] ^ buf\[3\]  
.text:0042EA4B                 xor     esi, \[ebx+UI32.\_3\] ; buf\[14\] ^ buf\[9\] ^ buf\[3\]  
.text:0042EA4E                 xor     edi, \[ebx+UI32.\_1\] ; buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]  
.text:0042EA51                 xor     esi, \[ebx+UI32.\_1\] ; buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]  
.text:0042EA54                 shr     edi, 1Fh        ; (buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\])>>31  
.text:0042EA57                 add     esi, esi        ; (buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\])<<1  
.text:0042EA59                 or      esi, edi        ; RL((buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1)  
.text:0042EA5B                 mov     \[ebx+UI32.\_1\], esi ; buf\[1\] = RL( (buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1)  
.text:0042EA5E                 mov     edi, \[esp+14h+A\]  
.text:0042EA62                 xor     edi, eax\_B      ; A^B  
.text:0042EA64                 and     edi, \[esp+14h+E\] ; (A^B)&E  
.text:0042EA68                 xor     edi, eax\_B      ; RX(E,A,B) = ((A^B)&E)^B  
.text:0042EA6A                 add     esi, edi        ; RL((buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1) + RX(E,A,B)  
.text:0042EA6C                 mov     edi, ecx\_D  
.text:0042EA6E                 shl     edi, 5          ; D<<5  
.text:0042EA71                 shr     ebp, 1Bh        ; D>>27  
.text:0042EA74                 or      edi, ebp        ; RL(D,5)  
.text:0042EA76                 add     esi, edi        ; RL((buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1) + RX(E,A,B) + RL(D,5)  
.text:0042EA78                 mov     edi, \[esp+14h+E\]  
.text:0042EA7C                 add     esi, 5A827999h ; <suspicious>  
.text:0042EA82                 add     edx\_C, esi      ; C += RL((buf\[14\] ^ buf\[9\] ^ buf\[3\] ^ buf\[1\]),1) + RX(E,A,B) + RL(D,5)+ 0x5a827999;  
.text:0042EA84                 mov     esi, \[esp+14h+E\]  
.text:0042EA88                 shl     esi, 1Eh        ; E<<30  
.text:0042EA8B                 mov     ebp, edx\_C  
.text:0042EA8D                 shr     edi, 2          ; E>>2  
.text:0042EA90                 or      esi, edi        ; RL(E,30)  
.text:0042EA92                 mov     \[esp+14h+E\], esi ; E = RL(E,30);  
.text:0042EA92                                         ;  
.text:0042EA92                                         ; R1(D,E,A,B,C,17)  
.text:0042EA92                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 18 次运算 end\*\*\*\*  
.text:0042EA92                                         ;  
.text:0042EA92                                         ;  
.text:0042EA96                 mov     edi, \[ebx+UI32.\_15\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 19 次运算开始\*\*\*\*  
.text:0042EA99                 mov     esi, \[ebx+UI32.\_15\]  
.text:0042EA9C                 xor     edi, \[ebx+UI32.\_10\] ; buf\[15\] ^ buf\[10\]  
.text:0042EA9F                 xor     esi, \[ebx+UI32.\_10\]  
.text:0042EAA2                 xor     edi, \[ebx+UI32.\_4\] ; buf\[15\] ^ buf\[10\] ^ buf\[4\]  
.text:0042EAA5                 xor     esi, \[ebx+UI32.\_4\]  
.text:0042EAA8                 xor     edi, \[ebx+UI32.\_2\] ; buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]  
.text:0042EAAB                 xor     esi, \[ebx+UI32.\_2\]  
.text:0042EAAE                 shr     edi, 1Fh        ; (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\])>>31  
.text:0042EAB1                 add     esi, esi        ; (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\])<<1  
.text:0042EAB3                 or      esi, edi        ; RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1)  
.text:0042EAB5                 mov     \[ebx+UI32.\_2\], esi ; buf\[2\] = RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1)  
.text:0042EAB8                 mov     edi, \[esp+14h+E\]  
.text:0042EABC                 xor     edi, \[esp+14h+A\] ; E^A  
.text:0042EAC0                 and     edi, ecx\_D      ; (E^A)&D  
.text:0042EAC2                 xor     edi, \[esp+14h+A\] ; RX(D,E,A) = ((E^A)&D)^A  
.text:0042EAC6                 add     esi, edi        ; RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1) + RX(D,E,A)  
.text:0042EAC8                 mov     edi, edx\_C  
.text:0042EACA                 shl     edi, 5          ; C<<5  
.text:0042EACD                 shr     ebp, 1Bh        ; C>>27  
.text:0042EAD0                 or      edi, ebp        ; RL(C,5)  
.text:0042EAD2                 add     esi, edi        ; RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1) + RX(D,E,A) + RL(C,5)  
.text:0042EAD4                 mov     edi, \[ebx+UI32.\_0\]  
.text:0042EAD6                 add     esi, 5A827999h ; <suspicious>  
.text:0042EADC                 xor     edi, \[ebx+UI32.\_11\]  
.text:0042EADF                 add     eax\_B, esi      ; B += RL( (buf\[15\] ^ buf\[10\] ^ buf\[4\] ^ buf\[2\]),1) + RX(D,E,A) + 0x5a827999  
.text:0042EAE1                 mov     esi, ecx\_D  
.text:0042EAE3                 shr     ecx\_D, 2        ; D>>2  
.text:0042EAE6                 mov     ebp, eax\_B  
.text:0042EAE8                 shl     esi, 1Eh        ; D<<30  
.text:0042EAEB                 xor     edi, \[ebx+UI32.\_5\]  
.text:0042EAEE                 or      esi, ecx\_D      ; RL(D,30)  
.text:0042EAF0                 xor     edi, \[ebx+UI32.\_3\] ; buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]  
.text:0042EAF3                 mov     ecx\_D, esi      ; D = RL(D,30)  
.text:0042EAF3                                         ;  
.text:0042EAF3                                         ; 19). R1(C,D,E,A,B,18)  
.text:0042EAF3                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 19 次运算 end\*\*\*\*  
.text:0042EAF3                                         ;  
.text:0042EAF3                                         ;  
.text:0042EAF5                 mov     esi, \[ebx+UI32.\_0\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 20 次运算开始\*\*\*\*  
.text:0042EAF7                 xor     esi, \[ebx+UI32.\_11\] ; buf\[0\] ^ buf\[11\]  
.text:0042EAFA                 xor     esi, \[ebx+UI32.\_5\] ; buf\[0\] ^ buf\[11\] ^ buf\[5\]  
.text:0042EAFD                 xor     esi, \[ebx+UI32.\_3\] ; buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]  
.text:0042EB00                 shr     edi, 1Fh        ; (buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]) >>31  
.text:0042EB03                 add     esi, esi        ; (buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\])<<1  
.text:0042EB05                 or      esi, edi        ; RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1)  
.text:0042EB07                 mov     \[ebx+UI32.\_3\], esi ; buf\[3\] = RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1)  
.text:0042EB0A                 mov     edi, \[esp+14h+E\]  
.text:0042EB0E                 xor     edi, ecx\_D      ; D^E  
.text:0042EB10                 and     edi, edx\_C      ; (D^E)&C  
.text:0042EB12                 xor     edi, \[esp+14h+E\] ; RX(C,D,E) = ((D^E)&C)^E  
.text:0042EB16                 add     esi, edi        ; RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1) + RX(C,D,E)  
.text:0042EB18                 mov     edi, eax\_B  
.text:0042EB1A                 shl     edi, 5          ; B<<5  
.text:0042EB1D                 shr     ebp, 1Bh        ; B>>27  
.text:0042EB20                 or      edi, ebp        ; RL(B,5)  
.text:0042EB22                 add     esi, edi        ; RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1) + RX(C,D,E) + RL(B,5)  
.text:0042EB24                 add     esi, 5A827999h ; <suspicious>  
.text:0042EB2A                 add     \[esp+14h+A\], esi ; A += RL((buf\[0\] ^ buf\[11\] ^ buf\[5\] ^ buf\[3\]),1) + RX(C,D,E) + RL(B,5) + 0x5a827999  
.text:0042EB2E                 mov     esi, edx\_C  
.text:0042EB30                 shr     edx\_C, 2        ; C>>2  
.text:0042EB33                 mov     edi, \[ebx+UI32.\_1\]  
.text:0042EB36                 shl     esi, 1Eh        ; C<<30  
.text:0042EB39                 xor     edi, \[ebx+UI32.\_12\]  
.text:0042EB3C                 or      esi, edx\_C      ; RL(C,30)  
.text:0042EB3E                 xor     edi, \[ebx+UI32.\_6\]  
.text:0042EB41                 mov     edx\_C, esi      ; C = RL(C,30)  
.text:0042EB41                                         ;  
.text:0042EB41                                         ; 20). R1(B,C,D,E,A,19)  
.text:0042EB41                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 20 次运算 end\*\*\*\*  
.text:0042EB41                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 1 轮计算结束\*\*\*\*  

  
.text:0042EB43                 mov     esi, \[ebx+UI32.\_1\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2 轮计算开始\*\*\*\*  
.text:0042EB43                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.0 次运算开始\*\*\*\*  
.text:0042EB46                 xor     esi, \[ebx+UI32.\_12\] ; buf\[1\] ^ buf\[12\]  
.text:0042EB49                 xor     edi, \[ebx+UI32.\_4\] ; buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\]  
.text:0042EB4C                 xor     esi, \[ebx+UI32.\_6\] ; buf\[1\] ^ buf\[12\] ^ buf\[6\]  
.text:0042EB4F                 xor     esi, \[ebx+UI32.\_4\] ; buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\]  
.text:0042EB52                 shr     edi, 1Fh        ; (buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\])>>31  
.text:0042EB55                 add     esi, esi        ; (buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\])<<1  
.text:0042EB57                 or      esi, edi        ; RL((buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\]),1)  
.text:0042EB59                 mov     edi, edx\_C  
.text:0042EB5B                 xor     edi, eax\_B      ; (B^C)  
.text:0042EB5D                 mov     \[ebx+UI32.\_4\], esi ; buf\[4\] = RL((buf\[1\] ^ buf\[12\] ^ buf\[6\] ^ buf\[4\]),1)  
.text:0042EB5D                                         ; RXL(20)  
.text:0042EB60                 xor     edi, ecx\_D      ; (B^C^D)  
.text:0042EB60                                         ;  
.text:0042EB60                                         ; 定义一个宏 RX1:  
.text:0042EB60                                         ; #define RX1(w,x,y) ( w^x^y )  
.text:0042EB60                                         ; 则: RX1(B,C,D) = (B^C^D)  
.text:0042EB62                 mov     ebp, \[esp+14h+A\]  
.text:0042EB66                 add     esi, edi        ; RXL(20) + RX1(B,C,D)  
.text:0042EB68                 mov     edi, \[esp+14h+A\]  
.text:0042EB6C                 shl     edi, 5          ; A<<5  
.text:0042EB6F                 shr     ebp, 1Bh        ; A>>27  
.text:0042EB72                 or      edi, ebp        ; RL(A,5)  
.text:0042EB74                 add     esi, edi        ; RXL(20) + RX1(B,C,D) + RL(A,5)  
.text:0042EB76                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EB7C                 add     \[esp+14h+E\], esi ; E += RXL(20) + RX1(B,C,D) + RL(A,5) + 0x6ed9eba1;  
.text:0042EB80                 mov     esi, eax\_B  
.text:0042EB82                 shr     eax\_B, 2        ; B>>2  
.text:0042EB85                 mov     edi, \[ebx+UI32.\_2\]  
.text:0042EB88                 shl     esi, 1Eh        ; B<<30  
.text:0042EB8B                 xor     edi, \[ebx+UI32.\_13\]  
.text:0042EB8E                 or      esi, eax\_B      ; RL(B,30)  
.text:0042EB90                 xor     edi, \[ebx+UI32.\_7\]  
.text:0042EB93                 mov     eax\_B, esi      ; B = RL(B,30)  
.text:0042EB93                                         ;  
.text:0042EB93                                         ; 定义一个宏 R2:  
.text:0042EB93                                         ; #define R2(a,b,c,d,e,i) ( e+=RXL(i)+RX1(b,c,d)+RL(a,5)+0x6ed9eba1; b=RL(b,30); )  
.text:0042EB93                                         ;  
.text:0042EB93                                         ; R2(A,B,C,D,E,20)  
.text:0042EB93                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.0 次运算 end\*\*\*\*  
.text:0042EB93                                         ;  
.text:0042EB93                                         ;  
.text:0042EB95                 mov     esi, \[ebx+UI32.\_2\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.1 次运算开始\*\*\*\*  
.text:0042EB98                 xor     esi, \[ebx+UI32.\_13\] ; buf\[2\] ^ buf\[13\]  
.text:0042EB9B                 xor     edi, \[ebx+UI32.\_5\]  
.text:0042EB9E                 xor     esi, \[ebx+UI32.\_7\] ; buf\[2\] ^ buf\[13\] ^ buf\[7\]  
.text:0042EBA1                 xor     esi, \[ebx+UI32.\_5\] ; buf\[2\] ^ buf\[13\] ^ buf\[7\] ^ buf\[5\]  
.text:0042EBA4                 shr     edi, 1Fh        ; (buf\[2\] ^ buf\[13\] ^ buf\[7\] ^buf\[5\])>>31  
.text:0042EBA7                 add     esi, esi        ; (buf\[2\] ^ buf\[13\] ^ buf\[7\] ^ buf\[5\])<<1  
.text:0042EBA9                 or      esi, edi  
.text:0042EBAB                 mov     \[ebx+UI32.\_5\], esi ; RXL(21)  
.text:0042EBAB                                         ; buf\[5\] = RL((buf\[2\] ^ buf\[13\] ^ buf\[7\] ^buf\[5\]),1)  
.text:0042EBAE                 mov     edi, \[esp+14h+A\]  
.text:0042EBB2                 mov     ebp, \[esp+14h+E\]  
.text:0042EBB6                 xor     edi, eax\_B      ; A^B  
.text:0042EBB8                 xor     edi, edx\_C      ; RX1(A,B,C) = A^B^C  
.text:0042EBBA                 add     esi, edi        ; RXL(21) + RX1(A,B,C)  
.text:0042EBBC                 mov     edi, \[esp+14h+E\]  
.text:0042EBC0                 shl     edi, 5          ; E<<5  
.text:0042EBC3                 shr     ebp, 1Bh        ; E>>27  
.text:0042EBC6                 or      edi, ebp        ; RL(E,30)  
.text:0042EBC8                 add     esi, edi        ; RXL(21) + RX1(A,B,C) + RL(E,30)  
.text:0042EBCA                 mov     edi, \[esp+14h+A\]  
.text:0042EBCE                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EBD4                 add     ecx\_D, esi      ; D += RXL(21) + RX1(A,B,C) + RL(E,30) + 0x6ed9eba1  
.text:0042EBD6                 mov     esi, \[esp+14h+A\]  
.text:0042EBDA                 shl     esi, 1Eh        ; A<<30  
.text:0042EBDD                 mov     ebp, ecx\_D  
.text:0042EBDF                 shr     edi, 2          ; A>>2  
.text:0042EBE2                 or      esi, edi        ; RL(A,30)  
.text:0042EBE4                 mov     \[esp+14h+A\], esi ; A = RL(A,30)  
.text:0042EBE4                                         ; R2(E,A,B,C,D,21)  
.text:0042EBE4                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.1 次运算 end\*\*\*\*  
.text:0042EBE4                                         ;  
.text:0042EBE4                                         ;  
.text:0042EBE8                 mov     edi, \[ebx+UI32.\_3\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.2 次运算开始\*\*\*\*  
.text:0042EBEB                 mov     esi, \[ebx+UI32.\_3\]  
.text:0042EBEE                 xor     edi, \[ebx+UI32.\_14\]  
.text:0042EBF1                 xor     esi, \[ebx+UI32.\_14\]  
.text:0042EBF4                 xor     edi, \[ebx+UI32.\_8\]  
.text:0042EBF7                 xor     esi, \[ebx+UI32.\_8\]  
.text:0042EBFA                 xor     edi, \[ebx+UI32.\_6\]  
.text:0042EBFD                 xor     esi, \[ebx+UI32.\_6\]  
.text:0042EC00                 shr     edi, 1Fh  
.text:0042EC03                 add     esi, esi  
.text:0042EC05                 or      esi, edi  
.text:0042EC07                 mov     \[ebx+UI32.\_6\], esi  
.text:0042EC0A                 mov     edi, \[esp+14h+E\]  
.text:0042EC0E                 xor     edi, \[esp+14h+A\]  
.text:0042EC12                 xor     edi, eax\_B  
.text:0042EC14                 add     esi, edi  
.text:0042EC16                 mov     edi, ecx\_D  
.text:0042EC18                 shl     edi, 5  
.text:0042EC1B                 shr     ebp, 1Bh  
.text:0042EC1E                 or      edi, ebp  
.text:0042EC20                 add     esi, edi  
.text:0042EC22                 mov     edi, \[esp+14h+E\]  
.text:0042EC26                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EC2C                 add     edx\_C, esi  
.text:0042EC2E                 mov     esi, \[esp+14h+E\]  
.text:0042EC32                 shl     esi, 1Eh  
.text:0042EC35                 mov     ebp, edx\_C  
.text:0042EC37                 shr     edi, 2  
.text:0042EC3A                 or      esi, edi  
.text:0042EC3C                 mov     \[esp+14h+E\], esi ; R2(D,E,A,B,C,22)  
.text:0042EC3C                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.2 次运算 end\*\*\*\*  
.text:0042EC3C                                         ;  
.text:0042EC3C                                         ;  
.text:0042EC40                 mov     edi, \[ebx+UI32.\_4\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.3 次运算开始\*\*\*\*  
.text:0042EC43                 mov     esi, \[ebx+UI32.\_4\]  
.text:0042EC46                 xor     edi, \[ebx+UI32.\_15\]  
.text:0042EC49                 xor     esi, \[ebx+3Ch\]  
.text:0042EC4C                 xor     edi, \[ebx+24h\]  
.text:0042EC4F                 xor     esi, \[ebx+24h\]  
.text:0042EC52                 xor     edi, \[ebx+1Ch\]  
.text:0042EC55                 xor     esi, \[ebx+1Ch\]  
.text:0042EC58                 shr     edi, 1Fh  
.text:0042EC5B                 add     esi, esi  
.text:0042EC5D                 or      esi, edi  
.text:0042EC5F                 mov     \[ebx+1Ch\], esi  
.text:0042EC62                 mov     edi, \[esp+14h+E\]  
.text:0042EC66                 xor     edi, ecx\_D  
.text:0042EC68                 xor     edi, \[esp+14h+A\]  
.text:0042EC6C                 add     esi, edi  
.text:0042EC6E                 mov     edi, edx\_C  
.text:0042EC70                 shr     ebp, 1Bh  
.text:0042EC73                 shl     edi, 5  
.text:0042EC76                 or      edi, ebp  
.text:0042EC78                 add     esi, edi  
.text:0042EC7A                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EC80                 add     eax\_B, esi  
.text:0042EC82                 mov     esi, ecx\_D  
.text:0042EC84                 mov     edi, \[ebx+14h\]  
.text:0042EC87                 shr     ecx\_D, 2  
.text:0042EC8A                 mov     ebp, eax\_B  
.text:0042EC8C                 shl     esi, 1Eh  
.text:0042EC8F                 xor     edi, \[ebx\]  
.text:0042EC91                 or      esi, ecx\_D  
.text:0042EC93                 xor     edi, \[ebx+28h\]  
.text:0042EC96                 mov     ecx\_D, esi      ; R2(C,D,E,A,B,23)  
.text:0042EC96                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.3 次运算 end\*\*\*\*  
.text:0042EC98                 mov     esi, \[ebx+14h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.4 次运算开始\*\*\*\*  
.text:0042EC9B                 xor     esi, \[ebx\]  
.text:0042EC9D                 xor     edi, \[ebx+20h\]  
.text:0042ECA0                 xor     esi, \[ebx+28h\]  
.text:0042ECA3                 xor     esi, \[ebx+20h\]  
.text:0042ECA6                 shr     edi, 1Fh  
.text:0042ECA9                 add     esi, esi  
.text:0042ECAB                 or      esi, edi  
.text:0042ECAD                 mov     edi, ecx\_D  
.text:0042ECAF                 mov     \[ebx+20h\], esi  
.text:0042ECB2                 xor     edi, edx\_C  
.text:0042ECB4                 xor     edi, \[esp+14h+E\]  
.text:0042ECB8                 add     esi, edi  
.text:0042ECBA                 mov     edi, eax\_B  
.text:0042ECBC                 shl     edi, 5  
.text:0042ECBF                 shr     ebp, 1Bh  
.text:0042ECC2                 or      edi, ebp  
.text:0042ECC4                 add     esi, edi  
.text:0042ECC6                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042ECCC                 add     \[esp+14h+A\], esi  
.text:0042ECD0                 mov     esi, edx\_C  
.text:0042ECD2                 shr     edx\_C, 2  
.text:0042ECD5                 mov     edi, \[ebx+18h\]  
.text:0042ECD8                 shl     esi, 1Eh  
.text:0042ECDB                 xor     edi, \[ebx+4\]  
.text:0042ECDE                 or      esi, edx\_C  
.text:0042ECE0                 xor     edi, \[ebx+2Ch\]  
.text:0042ECE3                 mov     edx\_C, esi      ; R2(B,C,D,E,A,24)  
.text:0042ECE3                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.4 次运算 end\*\*\*\*  
.text:0042ECE5                 mov     esi, \[ebx+18h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.5 次运算开始\*\*\*\*  
.text:0042ECE8                 xor     esi, \[ebx+4\]  
.text:0042ECEB                 xor     edi, \[ebx+24h\]  
.text:0042ECEE                 xor     esi, \[ebx+2Ch\]  
.text:0042ECF1                 xor     esi, \[ebx+24h\]  
.text:0042ECF4                 shr     edi, 1Fh  
.text:0042ECF7                 add     esi, esi  
.text:0042ECF9                 or      esi, edi  
.text:0042ECFB                 mov     edi, edx\_C  
.text:0042ECFD                 xor     edi, eax\_B  
.text:0042ECFF                 mov     \[ebx+24h\], esi  
.text:0042ED02                 xor     edi, ecx\_D  
.text:0042ED04                 mov     ebp, \[esp+14h+A\]  
.text:0042ED08                 add     esi, edi  
.text:0042ED0A                 mov     edi, \[esp+14h+A\]  
.text:0042ED0E                 shl     edi, 5  
.text:0042ED11                 shr     ebp, 1Bh  
.text:0042ED14                 or      edi, ebp  
.text:0042ED16                 add     esi, edi  
.text:0042ED18                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042ED1E                 add     \[esp+14h+E\], esi  
.text:0042ED22                 mov     esi, eax\_B  
.text:0042ED24                 shr     eax\_B, 2  
.text:0042ED27                 mov     edi, \[ebx+1Ch\]  
.text:0042ED2A                 shl     esi, 1Eh  
.text:0042ED2D                 xor     edi, \[ebx+8\]  
.text:0042ED30                 or      esi, eax\_B  
.text:0042ED32                 xor     edi, \[ebx+30h\]  
.text:0042ED35                 mov     eax\_B, esi      ; R2(A,B,C,D,E,25)  
.text:0042ED35                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.5 次运算 end\*\*\*\*  
.text:0042ED37                 mov     esi, \[ebx+1Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.6 次运算开始\*\*\*\*  
.text:0042ED3A                 xor     esi, \[ebx+8\]  
.text:0042ED3D                 xor     edi, \[ebx+28h\]  
.text:0042ED40                 xor     esi, \[ebx+30h\]  
.text:0042ED43                 xor     esi, \[ebx+28h\]  
.text:0042ED46                 shr     edi, 1Fh  
.text:0042ED49                 add     esi, esi  
.text:0042ED4B                 or      esi, edi  
.text:0042ED4D                 mov     \[ebx+28h\], esi  
.text:0042ED50                 mov     edi, \[esp+14h+A\]  
.text:0042ED54                 mov     ebp, \[esp+14h+E\]  
.text:0042ED58                 xor     edi, eax\_B  
.text:0042ED5A                 xor     edi, edx\_C  
.text:0042ED5C                 add     esi, edi  
.text:0042ED5E                 mov     edi, \[esp+14h+E\]  
.text:0042ED62                 shl     edi, 5  
.text:0042ED65                 shr     ebp, 1Bh  
.text:0042ED68                 or      edi, ebp  
.text:0042ED6A                 add     esi, edi  
.text:0042ED6C                 mov     edi, \[esp+14h+A\]  
.text:0042ED70                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042ED76                 add     ecx\_D, esi  
.text:0042ED78                 mov     esi, \[esp+14h+A\]  
.text:0042ED7C                 shl     esi, 1Eh  
.text:0042ED7F                 mov     ebp, ecx\_D  
.text:0042ED81                 shr     edi, 2  
.text:0042ED84                 or      esi, edi  
.text:0042ED86                 mov     \[esp+14h+A\], esi ; R2(E,A,B,C,D,26)  
.text:0042ED86                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.6 次运算 end\*\*\*\*  
.text:0042ED8A                 mov     edi, \[ebx+20h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.7 次运算开始\*\*\*\*  
.text:0042ED8D                 mov     esi, \[ebx+20h\]  
.text:0042ED90                 xor     edi, \[ebx+0Ch\]  
.text:0042ED93                 xor     esi, \[ebx+0Ch\]  
.text:0042ED96                 xor     edi, \[ebx+34h\]  
.text:0042ED99                 xor     esi, \[ebx+34h\]  
.text:0042ED9C                 xor     edi, \[ebx+2Ch\]  
.text:0042ED9F                 xor     esi, \[ebx+2Ch\]  
.text:0042EDA2                 shr     edi, 1Fh  
.text:0042EDA5                 add     esi, esi  
.text:0042EDA7                 or      esi, edi  
.text:0042EDA9                 mov     \[ebx+2Ch\], esi  
.text:0042EDAC                 mov     edi, \[esp+14h+E\]  
.text:0042EDB0                 xor     edi, \[esp+14h+A\]  
.text:0042EDB4                 xor     edi, eax\_B  
.text:0042EDB6                 add     esi, edi  
.text:0042EDB8                 mov     edi, ecx\_D  
.text:0042EDBA                 shl     edi, 5  
.text:0042EDBD                 shr     ebp, 1Bh  
.text:0042EDC0                 or      edi, ebp  
.text:0042EDC2                 add     esi, edi  
.text:0042EDC4                 mov     edi, \[esp+14h+E\]  
.text:0042EDC8                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EDCE                 add     edx\_C, esi  
.text:0042EDD0                 mov     esi, \[esp+14h+E\]  
.text:0042EDD4                 shl     esi, 1Eh  
.text:0042EDD7                 mov     ebp, edx\_C  
.text:0042EDD9                 shr     edi, 2  
.text:0042EDDC                 or      esi, edi  
.text:0042EDDE                 mov     \[esp+14h+E\], esi ; R2(D,E,A,B,C,27)  
.text:0042EDDE                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.7 次运算 END\*\*\*\*  
.text:0042EDE2                 mov     edi, \[ebx+24h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.8 次运算开始\*\*\*\*  
.text:0042EDE5                 mov     esi, \[ebx+24h\]  
.text:0042EDE8                 xor     edi, \[ebx+10h\]  
.text:0042EDEB                 xor     esi, \[ebx+10h\]  
.text:0042EDEE                 xor     edi, \[ebx+38h\]  
.text:0042EDF1                 xor     esi, \[ebx+38h\]  
.text:0042EDF4                 xor     edi, \[ebx+30h\]  
.text:0042EDF7                 xor     esi, \[ebx+30h\]  
.text:0042EDFA                 shr     edi, 1Fh  
.text:0042EDFD                 add     esi, esi  
.text:0042EDFF                 or      esi, edi  
.text:0042EE01                 mov     \[ebx+30h\], esi  
.text:0042EE04                 mov     edi, \[esp+14h+E\]  
.text:0042EE08                 xor     edi, ecx\_D  
.text:0042EE0A                 xor     edi, \[esp+14h+A\]  
.text:0042EE0E                 add     esi, edi  
.text:0042EE10                 mov     edi, edx\_C  
.text:0042EE12                 shl     edi, 5  
.text:0042EE15                 shr     ebp, 1Bh  
.text:0042EE18                 or      edi, ebp  
.text:0042EE1A                 add     esi, edi  
.text:0042EE1C                 mov     edi, \[ebx+28h\]  
.text:0042EE1F                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EE25                 xor     edi, \[ebx+14h\]  
.text:0042EE28                 add     eax\_B, esi  
.text:0042EE2A                 mov     esi, ecx\_D  
.text:0042EE2C                 shr     ecx\_D, 2  
.text:0042EE2F                 mov     ebp, eax\_B  
.text:0042EE31                 shl     esi, 1Eh  
.text:0042EE34                 xor     edi, \[ebx+3Ch\]  
.text:0042EE37                 or      esi, ecx\_D  
.text:0042EE39                 xor     edi, \[ebx+34h\]  
.text:0042EE3C                 mov     ecx\_D, esi      ; R2(C,D,E,A,B,28)  
.text:0042EE3C                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.8 次运算 END\*\*\*\*  
.text:0042EE3E                 mov     esi, \[ebx+28h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.9 次运算开始\*\*\*\*  
.text:0042EE41                 xor     esi, \[ebx+14h\]  
.text:0042EE44                 xor     esi, \[ebx+3Ch\]  
.text:0042EE47                 xor     esi, \[ebx+34h\]  
.text:0042EE4A                 shr     edi, 1Fh  
.text:0042EE4D                 add     esi, esi  
.text:0042EE4F                 or      esi, edi  
.text:0042EE51                 mov     edi, ecx\_D  
.text:0042EE53                 mov     \[ebx+34h\], esi  
.text:0042EE56                 xor     edi, edx\_C  
.text:0042EE58                 xor     edi, \[esp+14h+E\]  
.text:0042EE5C                 add     esi, edi  
.text:0042EE5E                 mov     edi, eax\_B  
.text:0042EE60                 shl     edi, 5  
.text:0042EE63                 shr     ebp, 1Bh  
.text:0042EE66                 or      edi, ebp  
.text:0042EE68                 add     esi, edi  
.text:0042EE6A                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EE70                 add     \[esp+14h+A\], esi  
.text:0042EE74                 mov     esi, edx\_C  
.text:0042EE76                 shr     edx\_C, 2  
.text:0042EE79                 mov     edi, \[ebx+2Ch\]  
.text:0042EE7C                 shl     esi, 1Eh  
.text:0042EE7F                 xor     edi, \[ebx+18h\]  
.text:0042EE82                 or      esi, edx\_C  
.text:0042EE84                 xor     edi, \[ebx\]  
.text:0042EE86                 mov     edx\_C, esi      ; R2(B,C,D,E,A,29)  
.text:0042EE86                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.9 次运算 END\*\*\*\*  
.text:0042EE88                 mov     esi, \[ebx+2Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.10 次运算开始\*\*\*\*  
.text:0042EE8B                 xor     esi, \[ebx+18h\]  
.text:0042EE8E                 xor     edi, \[ebx+38h\]  
.text:0042EE91                 xor     esi, \[ebx\]  
.text:0042EE93                 xor     esi, \[ebx+38h\]  
.text:0042EE96                 shr     edi, 1Fh  
.text:0042EE99                 add     esi, esi  
.text:0042EE9B                 or      esi, edi  
.text:0042EE9D                 mov     edi, edx\_C  
.text:0042EE9F                 xor     edi, eax\_B  
.text:0042EEA1                 mov     \[ebx+38h\], esi  
.text:0042EEA4                 xor     edi, ecx\_D  
.text:0042EEA6                 mov     ebp, \[esp+14h+A\]  
.text:0042EEAA                 add     esi, edi  
.text:0042EEAC                 mov     edi, \[esp+14h+A\]  
.text:0042EEB0                 shl     edi, 5  
.text:0042EEB3                 shr     ebp, 1Bh  
.text:0042EEB6                 or      edi, ebp  
.text:0042EEB8                 add     esi, edi  
.text:0042EEBA                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EEC0                 add     \[esp+14h+E\], esi  
.text:0042EEC4                 mov     esi, eax\_B  
.text:0042EEC6                 shr     eax\_B, 2  
.text:0042EEC9                 mov     edi, \[ebx+30h\]  
.text:0042EECC                 shl     esi, 1Eh  
.text:0042EECF                 xor     edi, \[ebx+1Ch\]  
.text:0042EED2                 or      esi, eax\_B  
.text:0042EED4                 xor     edi, \[ebx+4\]  
.text:0042EED7                 mov     eax\_B, esi      ; R2(A,B,C,D,E,30)  
.text:0042EED7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.10 次运算 END\*\*\*\*  
.text:0042EED9                 mov     esi, \[ebx+30h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.11 次运算开始\*\*\*\*  
.text:0042EEDC                 xor     esi, \[ebx+1Ch\]  
.text:0042EEDF                 xor     edi, \[ebx+3Ch\]  
.text:0042EEE2                 xor     esi, \[ebx+4\]  
.text:0042EEE5                 xor     esi, \[ebx+3Ch\]  
.text:0042EEE8                 shr     edi, 1Fh  
.text:0042EEEB                 add     esi, esi  
.text:0042EEED                 or      esi, edi  
.text:0042EEEF                 mov     \[ebx+3Ch\], esi  
.text:0042EEF2                 mov     edi, \[esp+14h+A\]  
.text:0042EEF6                 mov     ebp, \[esp+14h+E\]  
.text:0042EEFA                 xor     edi, eax\_B  
.text:0042EEFC                 xor     edi, edx\_C  
.text:0042EEFE                 add     esi, edi  
.text:0042EF00                 mov     edi, \[esp+14h+E\]  
.text:0042EF04                 shr     ebp, 1Bh  
.text:0042EF07                 shl     edi, 5  
.text:0042EF0A                 or      edi, ebp  
.text:0042EF0C                 add     esi, edi  
.text:0042EF0E                 mov     edi, \[esp+14h+A\]  
.text:0042EF12                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EF18                 add     ecx\_D, esi  
.text:0042EF1A                 mov     esi, \[esp+14h+A\]  
.text:0042EF1E                 shl     esi, 1Eh  
.text:0042EF21                 shr     edi, 2  
.text:0042EF24                 or      esi, edi  
.text:0042EF26                 mov     \[esp+14h+A\], esi ; R2(E,A,B,C,D,31)  
.text:0042EF26                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.11 次运算 end\*\*\*\*  
.text:0042EF2A                 mov     edi, \[ebx+34h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.12 次运算开始\*\*\*\*  
.text:0042EF2D                 mov     esi, \[ebx+34h\]  
.text:0042EF30                 xor     edi, \[ebx+20h\]  
.text:0042EF33                 xor     esi, \[ebx+20h\]  
.text:0042EF36                 xor     edi, \[ebx+8\]  
.text:0042EF39                 xor     esi, \[ebx+8\]  
.text:0042EF3C                 xor     edi, \[ebx\]  
.text:0042EF3E                 xor     esi, \[ebx\]  
.text:0042EF40                 shr     edi, 1Fh  
.text:0042EF43                 add     esi, esi  
.text:0042EF45                 or      esi, edi  
.text:0042EF47                 mov     \[ebx\], esi  
.text:0042EF49                 mov     edi, \[esp+14h+E\]  
.text:0042EF4D                 xor     edi, \[esp+14h+A\]  
.text:0042EF51                 xor     edi, eax\_B  
.text:0042EF53                 add     esi, edi  
.text:0042EF55                 mov     edi, ecx\_D  
.text:0042EF57                 mov     ebp, ecx\_D  
.text:0042EF59                 shr     ebp, 1Bh  
.text:0042EF5C                 shl     edi, 5  
.text:0042EF5F                 or      edi, ebp  
.text:0042EF61                 add     esi, edi  
.text:0042EF63                 mov     edi, \[esp+14h+E\]  
.text:0042EF67                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EF6D                 add     edx\_C, esi  
.text:0042EF6F                 mov     esi, \[esp+14h+E\]  
.text:0042EF73                 shl     esi, 1Eh  
.text:0042EF76                 mov     ebp, edx\_C  
.text:0042EF78                 shr     edi, 2  
.text:0042EF7B                 or      esi, edi  
.text:0042EF7D                 mov     \[esp+14h+E\], esi ; R2(D,E,A,B,C,32)  
.text:0042EF7D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.12 次运算 END\*\*\*\*  
.text:0042EF81                 mov     edi, \[ebx+38h\]  ; ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.13 次运算开始\*\*\*\*  
.text:0042EF84                 mov     esi, \[ebx+38h\]  
.text:0042EF87                 xor     edi, \[ebx+24h\]  
.text:0042EF8A                 xor     esi, \[ebx+24h\]  
.text:0042EF8D                 xor     edi, \[ebx+0Ch\]  
.text:0042EF90                 xor     esi, \[ebx+0Ch\]  
.text:0042EF93                 xor     edi, \[ebx+4\]  
.text:0042EF96                 xor     esi, \[ebx+4\]  
.text:0042EF99                 shr     edi, 1Fh  
.text:0042EF9C                 add     esi, esi  
.text:0042EF9E                 or      esi, edi  
.text:0042EFA0                 mov     \[ebx+4\], esi  
.text:0042EFA3                 mov     edi, \[esp+14h+E\]  
.text:0042EFA7                 xor     edi, ecx\_D  
.text:0042EFA9                 xor     edi, \[esp+14h+A\]  
.text:0042EFAD                 add     esi, edi  
.text:0042EFAF                 mov     edi, edx\_C  
.text:0042EFB1                 shl     edi, 5  
.text:0042EFB4                 shr     ebp, 1Bh  
.text:0042EFB7                 or      edi, ebp  
.text:0042EFB9                 add     esi, edi  
.text:0042EFBB                 mov     edi, \[ebx+3Ch\]  
.text:0042EFBE                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042EFC4                 xor     edi, \[ebx+28h\]  
.text:0042EFC7                 add     eax\_B, esi  
.text:0042EFC9                 mov     esi, ecx\_D  
.text:0042EFCB                 shr     ecx\_D, 2  
.text:0042EFCE                 mov     ebp, eax\_B  
.text:0042EFD0                 shl     esi, 1Eh  
.text:0042EFD3                 xor     edi, \[ebx+10h\]  
.text:0042EFD6                 or      esi, ecx\_D  
.text:0042EFD8                 xor     edi, \[ebx+8\]  
.text:0042EFDB                 mov     ecx\_D, esi      ; R2(C,D,E,A,B,33)  
.text:0042EFDB                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.13 次运算 END\*\*\*\*  
.text:0042EFDD                 mov     esi, \[ebx+3Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.14 次运算开始\*\*\*\*  
.text:0042EFE0                 xor     esi, \[ebx+28h\]  
.text:0042EFE3                 xor     esi, \[ebx+10h\]  
.text:0042EFE6                 xor     esi, \[ebx+8\]  
.text:0042EFE9                 shr     edi, 1Fh  
.text:0042EFEC                 add     esi, esi  
.text:0042EFEE                 or      esi, edi  
.text:0042EFF0                 mov     edi, ecx\_D  
.text:0042EFF2                 mov     \[ebx+8\], esi  
.text:0042EFF5                 xor     edi, edx\_C  
.text:0042EFF7                 xor     edi, \[esp+14h+E\]  
.text:0042EFFB                 add     esi, edi  
.text:0042EFFD                 mov     edi, eax\_B  
.text:0042EFFF                 shl     edi, 5  
.text:0042F002                 shr     ebp, 1Bh  
.text:0042F005                 or      edi, ebp  
.text:0042F007                 add     esi, edi  
.text:0042F009                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F00F                 add     \[esp+14h+A\], esi  
.text:0042F013                 mov     esi, edx\_C  
.text:0042F015                 shr     edx\_C, 2  
.text:0042F018                 mov     edi, \[ebx\]  
.text:0042F01A                 shl     esi, 1Eh  
.text:0042F01D                 xor     edi, \[ebx+2Ch\]  
.text:0042F020                 or      esi, edx\_C  
.text:0042F022                 xor     edi, \[ebx+14h\]  
.text:0042F025                 mov     edx\_C, esi      ; R2(B,C,D,E,A,34)  
.text:0042F025                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.14 次运算 END\*\*\*\*  
.text:0042F027                 mov     esi, \[ebx\]      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.15 次运算开始\*\*\*\*  
.text:0042F029                 xor     esi, \[ebx+2Ch\]  
.text:0042F02C                 xor     edi, \[ebx+0Ch\]  
.text:0042F02F                 xor     esi, \[ebx+14h\]  
.text:0042F032                 xor     esi, \[ebx+0Ch\]  
.text:0042F035                 shr     edi, 1Fh  
.text:0042F038                 add     esi, esi  
.text:0042F03A                 or      esi, edi  
.text:0042F03C                 mov     edi, edx\_C  
.text:0042F03E                 xor     edi, eax\_B  
.text:0042F040                 mov     \[ebx+0Ch\], esi  
.text:0042F043                 xor     edi, ecx\_D  
.text:0042F045                 mov     ebp, \[esp+14h+A\]  
.text:0042F049                 add     esi, edi  
.text:0042F04B                 mov     edi, \[esp+14h+A\]  
.text:0042F04F                 shl     edi, 5  
.text:0042F052                 shr     ebp, 1Bh  
.text:0042F055                 or      edi, ebp  
.text:0042F057                 add     esi, edi  
.text:0042F059                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F05F                 add     \[esp+14h+E\], esi  
.text:0042F063                 mov     esi, eax\_B  
.text:0042F065                 shr     eax\_B, 2  
.text:0042F068                 mov     edi, \[ebx+4\]  
.text:0042F06B                 shl     esi, 1Eh  
.text:0042F06E                 xor     edi, \[ebx+30h\]  
.text:0042F071                 or      esi, eax\_B  
.text:0042F073                 xor     edi, \[ebx+18h\]  
.text:0042F076                 mov     eax\_B, esi      ; R2(A,B,C,D,E,35)  
.text:0042F076                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.15 次运算 END\*\*\*\*  
.text:0042F078                 mov     esi, \[ebx+4\]    ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.16 次运算开始\*\*\*\*  
.text:0042F07B                 xor     esi, \[ebx+30h\]  
.text:0042F07E                 xor     edi, \[ebx+10h\]  
.text:0042F081                 xor     esi, \[ebx+18h\]  
.text:0042F084                 xor     esi, \[ebx+10h\]  
.text:0042F087                 shr     edi, 1Fh  
.text:0042F08A                 add     esi, esi  
.text:0042F08C                 or      esi, edi  
.text:0042F08E                 mov     \[ebx+10h\], esi  
.text:0042F091                 mov     edi, \[esp+14h+A\]  
.text:0042F095                 mov     ebp, \[esp+14h+E\]  
.text:0042F099                 xor     edi, eax\_B  
.text:0042F09B                 xor     edi, edx\_C  
.text:0042F09D                 add     esi, edi  
.text:0042F09F                 mov     edi, \[esp+14h+E\]  
.text:0042F0A3                 shl     edi, 5  
.text:0042F0A6                 shr     ebp, 1Bh  
.text:0042F0A9                 or      edi, ebp  
.text:0042F0AB                 add     esi, edi  
.text:0042F0AD                 mov     edi, \[esp+14h+A\]  
.text:0042F0B1                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F0B7                 add     ecx\_D, esi  
.text:0042F0B9                 mov     esi, \[esp+14h+A\]  
.text:0042F0BD                 shl     esi, 1Eh  
.text:0042F0C0                 mov     ebp, ecx\_D  
.text:0042F0C2                 shr     edi, 2  
.text:0042F0C5                 or      esi, edi  
.text:0042F0C7                 mov     \[esp+14h+A\], esi ; R2(E,A,B,C,D,36)  
.text:0042F0C7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.16 次运算 END\*\*\*\*  
.text:0042F0CB                 mov     edi, \[ebx+8\]    ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.17 次运算开始\*\*\*\*  
.text:0042F0CE                 mov     esi, \[ebx+8\]  
.text:0042F0D1                 xor     edi, \[ebx+34h\]  
.text:0042F0D4                 xor     esi, \[ebx+34h\]  
.text:0042F0D7                 xor     edi, \[ebx+1Ch\]  
.text:0042F0DA                 xor     esi, \[ebx+1Ch\]  
.text:0042F0DD                 xor     edi, \[ebx+14h\]  
.text:0042F0E0                 xor     esi, \[ebx+14h\]  
.text:0042F0E3                 shr     edi, 1Fh  
.text:0042F0E6                 add     esi, esi  
.text:0042F0E8                 or      esi, edi  
.text:0042F0EA                 mov     \[ebx+14h\], esi  
.text:0042F0ED                 mov     edi, \[esp+14h+E\]  
.text:0042F0F1                 xor     edi, \[esp+14h+A\]  
.text:0042F0F5                 xor     edi, eax\_B  
.text:0042F0F7                 add     esi, edi  
.text:0042F0F9                 mov     edi, ecx\_D  
.text:0042F0FB                 shl     edi, 5  
.text:0042F0FE                 shr     ebp, 1Bh  
.text:0042F101                 or      edi, ebp  
.text:0042F103                 add     esi, edi  
.text:0042F105                 mov     edi, \[esp+14h+E\]  
.text:0042F109                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F10F                 add     edx\_C, esi  
.text:0042F111                 mov     esi, \[esp+14h+E\]  
.text:0042F115                 shl     esi, 1Eh  
.text:0042F118                 mov     ebp, edx\_C  
.text:0042F11A                 shr     edi, 2  
.text:0042F11D                 or      esi, edi  
.text:0042F11F                 mov     \[esp+14h+E\], esi ; R2(D,E,A,B,C,37)  
.text:0042F11F                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.17 次运算 END\*\*\*\*  
.text:0042F123                 mov     edi, \[ebx+0Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.18 次运算开始\*\*\*\*  
.text:0042F126                 mov     esi, \[ebx+0Ch\]  
.text:0042F129                 xor     edi, \[ebx+38h\]  
.text:0042F12C                 xor     esi, \[ebx+38h\]  
.text:0042F12F                 xor     edi, \[ebx+20h\]  
.text:0042F132                 xor     esi, \[ebx+20h\]  
.text:0042F135                 xor     edi, \[ebx+18h\]  
.text:0042F138                 xor     esi, \[ebx+18h\]  
.text:0042F13B                 shr     edi, 1Fh  
.text:0042F13E                 add     esi, esi  
.text:0042F140                 or      esi, edi  
.text:0042F142                 mov     \[ebx+18h\], esi  
.text:0042F145                 mov     edi, \[esp+14h+E\]  
.text:0042F149                 xor     edi, ecx\_D  
.text:0042F14B                 xor     edi, \[esp+14h+A\]  
.text:0042F14F                 add     esi, edi  
.text:0042F151                 mov     edi, edx\_C  
.text:0042F153                 shl     edi, 5  
.text:0042F156                 shr     ebp, 1Bh  
.text:0042F159                 or      edi, ebp  
.text:0042F15B                 add     esi, edi  
.text:0042F15D                 mov     edi, \[ebx+10h\]  
.text:0042F160                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F166                 xor     edi, \[ebx+3Ch\]  
.text:0042F169                 add     eax\_B, esi  
.text:0042F16B                 mov     esi, ecx\_D  
.text:0042F16D                 shr     ecx\_D, 2  
.text:0042F170                 mov     ebp, eax\_B  
.text:0042F172                 shl     esi, 1Eh  
.text:0042F175                 xor     edi, \[ebx+24h\]  
.text:0042F178                 or      esi, ecx\_D  
.text:0042F17A                 xor     edi, \[ebx+1Ch\]  
.text:0042F17D                 mov     ecx\_D, esi      ; R2(C,D,E,A,B,38)  
.text:0042F17D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.18 次运算 END\*\*\*\*  
.text:0042F17F                 mov     esi, \[ebx+10h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.19 次运算开始\*\*\*\*  
.text:0042F182                 xor     esi, \[ebx+3Ch\]  
.text:0042F185                 xor     esi, \[ebx+24h\]  
.text:0042F188                 xor     esi, \[ebx+1Ch\]  
.text:0042F18B                 shr     edi, 1Fh  
.text:0042F18E                 add     esi, esi  
.text:0042F190                 or      esi, edi  
.text:0042F192                 mov     edi, ecx\_D  
.text:0042F194                 mov     \[ebx+1Ch\], esi  
.text:0042F197                 xor     edi, edx\_C  
.text:0042F199                 xor     edi, \[esp+14h+E\]  
.text:0042F19D                 add     esi, edi  
.text:0042F19F                 mov     edi, eax\_B  
.text:0042F1A1                 shl     edi, 5  
.text:0042F1A4                 shr     ebp, 1Bh  
.text:0042F1A7                 or      edi, ebp  
.text:0042F1A9                 add     esi, edi  
.text:0042F1AB                 add     esi, 6ED9EBA1h ; <suspicious>  
.text:0042F1B1                 add     \[esp+14h+A\], esi  
.text:0042F1B5                 mov     esi, edx\_C  
.text:0042F1B7                 shr     edx\_C, 2  
.text:0042F1BA                 mov     ebp, \[ebx+UI32.\_5\]  
.text:0042F1BD                 shl     esi, 1Eh  
.text:0042F1C0                 xor     ebp, \[ebx+UI32.\_0\]  
.text:0042F1C2                 or      esi, edx\_C  
.text:0042F1C4                 xor     ebp, \[ebx+UI32.\_10\]  
.text:0042F1C7                 mov     edx\_C, esi      ; R2(B,C,D,E,A,39)  
.text:0042F1C7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2.19 次运算 END\*\*\*\*  
.text:0042F1C7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 2 轮计算结束\*\*\*\*  
.text:0042F1C7                                         ;  
.text:0042F1C7                                         ;  
.text:0042F1C9                 xor     ebp, \[ebx+UI32.\_8\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3 轮计算开始\*\*\*\*  
.text:0042F1C9                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.0 次运算开始\*\*\*\*  
.text:0042F1CC                 mov     esi, edx\_C  
.text:0042F1CE                 mov     edi, edx\_C  
.text:0042F1D0                 or      esi, eax\_B      ; C|B  
.text:0042F1D2                 and     edi, eax\_B      ; C&B  
.text:0042F1D4                 and     esi, ecx\_D      ; (C|B)&D  
.text:0042F1D6                 or      esi, edi        ; ((C|B)&D) | (C&B)  
.text:0042F1D6                                         ;  
.text:0042F1D6                                         ; 定义宏:  
.text:0042F1D6                                         ; RX2(w,x,y) ( ((w|x)&y) | (w&x))  
.text:0042F1D6                                         ;  
.text:0042F1D6                                         ; RX2(B,C,D) = ((C|B)&D) | (C&B)  
.text:0042F1D8                 mov     edi, \[ebx+UI32.\_5\]  
.text:0042F1DB                 xor     edi, \[ebx+UI32.\_0\] ; buf\[5\] ^ buf\[0\]  
.text:0042F1DD                 xor     edi, \[ebx+UI32.\_10\] ; buf\[5\] ^ buf\[0\] ^ buf\[10\]  
.text:0042F1E0                 xor     edi, \[ebx+UI32.\_8\] ; buf\[5\] ^ buf\[0\] ^ buf\[10\] ^ buf\[8\]  
.text:0042F1E3                 shr     ebp, 1Fh        ; (buf\[5\] ^ buf\[0\] ^ buf\[10\] ^ buf\[8\])>>31  
.text:0042F1E6                 add     edi, edi        ; (buf\[5\] ^ buf\[0\] ^ buf\[10\] ^ buf\[8\])<<1  
.text:0042F1E8                 or      edi, ebp  
.text:0042F1EA                 mov     \[ebx+UI32.\_8\], edi ; buf\[8\] =RL((buf\[5\] ^ buf\[0\] ^ buf\[10\] ^ buf\[8\]),1)  
.text:0042F1EA                                         ; RXL(40)  
.text:0042F1ED                 add     esi, edi        ; RXL(40) + RX2(B,C,D)  
.text:0042F1EF                 mov     edi, \[esp+14h+A\]  
.text:0042F1F3                 mov     ebp, \[esp+14h+A\]  
.text:0042F1F7                 shr     ebp, 1Bh        ; A>>27  
.text:0042F1FA                 shl     edi, 5          ; A<<5  
.text:0042F1FD                 or      edi, ebp        ; RL(A,5)  
.text:0042F1FF                 add     esi, edi        ; RXL(40) + RX2(B,C,D) + RL(A,5)  
.text:0042F201                 add     esi, 8F1BBCDCh  
.text:0042F207                 add     \[esp+14h+E\], esi ; E += RXL(40) + RX2(B,C,D) + RL(A,5) + 0x8f1bbcdc  
.text:0042F20B                 mov     esi, eax\_B  
.text:0042F20D                 shr     eax\_B, 2        ; B>>2  
.text:0042F210                 shl     esi, 1Eh        ; B<<30  
.text:0042F213                 or      esi, eax\_B      ; RL(B,30)  
.text:0042F215                 mov     eax\_B, esi      ; B = RL(B,30)  
.text:0042F215                                         ;  
.text:0042F215                                         ; 定义宏 R3:  
.text:0042F215                                         ; #define R3(a,b,c,d,e,i) ( e+=RXL(i)+RX2(b,c,d)+RL(a,5)+0x8f1bbcdc; b=RL(b,30); )  
.text:0042F215                                         ;  
.text:0042F215                                         ; 此次运算为: R3(A,B,C,D,E,40)  
.text:0042F215                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.0 次运算 end\*\*\*\*  
.text:0042F215                                         ;  
.text:0042F217                 mov     esi, \[esp+14h+A\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.1 次运算开始\*\*\*\*  
.text:0042F21B                 or      esi, eax\_B  
.text:0042F21D                 and     esi, edx\_C  
.text:0042F21F                 mov     edi, \[esp+14h+A\]  
.text:0042F223                 mov     ebp, \[ebx+18h\]  
.text:0042F226                 and     edi, eax\_B  
.text:0042F228                 xor     ebp, \[ebx+4\]  
.text:0042F22B                 or      esi, edi  
.text:0042F22D                 mov     edi, \[ebx+18h\]  
.text:0042F230                 xor     edi, \[ebx+4\]  
.text:0042F233                 xor     ebp, \[ebx+2Ch\]  
.text:0042F236                 xor     edi, \[ebx+2Ch\]  
.text:0042F239                 xor     ebp, \[ebx+24h\]  
.text:0042F23C                 xor     edi, \[ebx+24h\]  
.text:0042F23F                 shr     ebp, 1Fh  
.text:0042F242                 add     edi, edi  
.text:0042F244                 or      edi, ebp  
.text:0042F246                 mov     \[ebx+24h\], edi  
.text:0042F249                 add     esi, edi  
.text:0042F24B                 mov     edi, \[esp+14h+E\]  
.text:0042F24F                 mov     ebp, \[esp+14h+E\]  
.text:0042F253                 shr     ebp, 1Bh  
.text:0042F256                 shl     edi, 5  
.text:0042F259                 or      edi, ebp  
.text:0042F25B                 add     esi, edi  
.text:0042F25D                 mov     edi, \[esp+14h+A\]  
.text:0042F261                 add     esi, 8F1BBCDCh  
.text:0042F267                 add     ecx\_D, esi  
.text:0042F269                 mov     esi, \[esp+14h+A\]  
.text:0042F26D                 shl     esi, 1Eh        ; A<<30  
.text:0042F270                 shr     edi, 2          ; A>>2  
.text:0042F273                 or      esi, edi        ; RL(A,30)  
.text:0042F275                 mov     \[esp+14h+A\], esi ; A = RL(A,30)  
.text:0042F275                                         ; R3(E,A,B,C,D,41)  
.text:0042F275                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.1 次运算 END\*\*\*\*  
.text:0042F279                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.2 次运算开始\*\*\*\*  
.text:0042F27D                 mov     edi, \[esp+14h+E\]  
.text:0042F281                 or      esi, \[esp+14h+A\]  
.text:0042F285                 and     edi, \[esp+14h+A\]  
.text:0042F289                 and     esi, eax\_B  
.text:0042F28B                 mov     ebp, \[ebx+1Ch\]  
.text:0042F28E                 or      esi, edi  
.text:0042F290                 mov     edi, \[ebx+1Ch\]  
.text:0042F293                 xor     ebp, \[ebx+8\]  
.text:0042F296                 xor     edi, \[ebx+8\]  
.text:0042F299                 xor     ebp, \[ebx+30h\]  
.text:0042F29C                 xor     edi, \[ebx+30h\]  
.text:0042F29F                 xor     ebp, \[ebx+28h\]  
.text:0042F2A2                 xor     edi, \[ebx+28h\]  
.text:0042F2A5                 shr     ebp, 1Fh  
.text:0042F2A8                 add     edi, edi  
.text:0042F2AA                 or      edi, ebp  
.text:0042F2AC                 mov     ebp, ecx\_D  
.text:0042F2AE                 mov     \[ebx+28h\], edi  
.text:0042F2B1                 add     esi, edi  
.text:0042F2B3                 mov     edi, ecx\_D  
.text:0042F2B5                 shl     edi, 5  
.text:0042F2B8                 shr     ebp, 1Bh  
.text:0042F2BB                 or      edi, ebp  
.text:0042F2BD                 add     esi, edi  
.text:0042F2BF                 mov     edi, \[esp+14h+E\]  
.text:0042F2C3                 add     esi, 8F1BBCDCh  
.text:0042F2C9                 add     edx\_C, esi  
.text:0042F2CB                 mov     esi, \[esp+14h+E\]  
.text:0042F2CF                 shl     esi, 1Eh  
.text:0042F2D2                 shr     edi, 2  
.text:0042F2D5                 or      esi, edi  
.text:0042F2D7                 mov     \[esp+14h+E\], esi ; R3(D,E,A,B,C,42)  
.text:0042F2D7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.2 次运算 end\*\*\*\*  
.text:0042F2DB                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.3 次运算开始\*\*\*\*  
.text:0042F2DF                 mov     edi, \[esp+14h+E\]  
.text:0042F2E3                 or      esi, ecx\_D  
.text:0042F2E5                 and     edi, ecx\_D  
.text:0042F2E7                 and     esi, \[esp+14h+A\]  
.text:0042F2EB                 mov     ebp, \[ebx+20h\]  
.text:0042F2EE                 or      esi, edi  
.text:0042F2F0                 mov     edi, \[ebx+20h\]  
.text:0042F2F3                 xor     ebp, \[ebx+0Ch\]  
.text:0042F2F6                 xor     edi, \[ebx+0Ch\]  
.text:0042F2F9                 xor     ebp, \[ebx+34h\]  
.text:0042F2FC                 xor     edi, \[ebx+34h\]  
.text:0042F2FF                 xor     ebp, \[ebx+2Ch\]  
.text:0042F302                 xor     edi, \[ebx+2Ch\]  
.text:0042F305                 shr     ebp, 1Fh  
.text:0042F308                 add     edi, edi  
.text:0042F30A                 or      edi, ebp  
.text:0042F30C                 mov     ebp, edx\_C  
.text:0042F30E                 mov     \[ebx+2Ch\], edi  
.text:0042F311                 add     esi, edi  
.text:0042F313                 mov     edi, edx\_C  
.text:0042F315                 shl     edi, 5  
.text:0042F318                 shr     ebp, 1Bh  
.text:0042F31B                 or      edi, ebp  
.text:0042F31D                 mov     ebp, \[ebx+24h\]  
.text:0042F320                 add     esi, edi  
.text:0042F322                 xor     ebp, \[ebx+10h\]  
.text:0042F325                 add     esi, 8F1BBCDCh  
.text:0042F32B                 xor     ebp, \[ebx+38h\]  
.text:0042F32E                 add     eax\_B, esi  
.text:0042F330                 mov     esi, ecx\_D  
.text:0042F332                 shr     ecx\_D, 2  
.text:0042F335                 xor     ebp, \[ebx+30h\]  
.text:0042F338                 shl     esi, 1Eh  
.text:0042F33B                 or      esi, ecx\_D  
.text:0042F33D                 mov     ecx\_D, esi      ; R3(C,D,E,A,B,43)  
.text:0042F33D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.3 次运算 END\*\*\*\*  
.text:0042F33F                 mov     esi, ecx\_D      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.4 次运算开始\*\*\*\*  
.text:0042F341                 mov     edi, ecx\_D  
.text:0042F343                 or      esi, edx\_C  
.text:0042F345                 and     edi, edx\_C  
.text:0042F347                 and     esi, \[esp+14h+E\]  
.text:0042F34B                 or      esi, edi  
.text:0042F34D                 mov     edi, \[ebx+24h\]  
.text:0042F350                 xor     edi, \[ebx+10h\]  
.text:0042F353                 xor     edi, \[ebx+38h\]  
.text:0042F356                 xor     edi, \[ebx+30h\]  
.text:0042F359                 shr     ebp, 1Fh  
.text:0042F35C                 add     edi, edi  
.text:0042F35E                 or      edi, ebp  
.text:0042F360                 mov     ebp, eax\_B  
.text:0042F362                 mov     \[ebx+30h\], edi  
.text:0042F365                 add     esi, edi  
.text:0042F367                 mov     edi, eax\_B  
.text:0042F369                 shl     edi, 5  
.text:0042F36C                 shr     ebp, 1Bh  
.text:0042F36F                 or      edi, ebp  
.text:0042F371                 add     esi, edi  
.text:0042F373                 add     esi, 8F1BBCDCh  
.text:0042F379                 add     \[esp+14h+A\], esi  
.text:0042F37D                 mov     esi, edx\_C  
.text:0042F37F                 shr     edx\_C, 2  
.text:0042F382                 mov     ebp, \[ebx+28h\]  
.text:0042F385                 shl     esi, 1Eh  
.text:0042F388                 xor     ebp, \[ebx+14h\]  
.text:0042F38B                 or      esi, edx\_C  
.text:0042F38D                 xor     ebp, \[ebx+3Ch\]  
.text:0042F390                 mov     edx\_C, esi      ; R3(B,C,D,E,A,44)  
.text:0042F390                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.4 次运算 END\*\*\*\*  
.text:0042F392                 xor     ebp, \[ebx+34h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.5 次运算开始\*\*\*\*  
.text:0042F395                 mov     esi, edx\_C  
.text:0042F397                 mov     edi, edx\_C  
.text:0042F399                 or      esi, eax\_B  
.text:0042F39B                 and     edi, eax\_B  
.text:0042F39D                 and     esi, ecx\_D  
.text:0042F39F                 or      esi, edi  
.text:0042F3A1                 mov     edi, \[ebx+28h\]  
.text:0042F3A4                 xor     edi, \[ebx+14h\]  
.text:0042F3A7                 xor     edi, \[ebx+3Ch\]  
.text:0042F3AA                 xor     edi, \[ebx+34h\]  
.text:0042F3AD                 shr     ebp, 1Fh  
.text:0042F3B0                 add     edi, edi  
.text:0042F3B2                 or      edi, ebp  
.text:0042F3B4                 mov     \[ebx+34h\], edi  
.text:0042F3B7                 add     esi, edi  
.text:0042F3B9                 mov     edi, \[esp+14h+A\]  
.text:0042F3BD                 mov     ebp, \[esp+14h+A\]  
.text:0042F3C1                 shr     ebp, 1Bh  
.text:0042F3C4                 shl     edi, 5  
.text:0042F3C7                 or      edi, ebp  
.text:0042F3C9                 add     esi, edi  
.text:0042F3CB                 add     esi, 8F1BBCDCh  
.text:0042F3D1                 add     \[esp+14h+E\], esi  
.text:0042F3D5                 mov     esi, eax\_B  
.text:0042F3D7                 shr     eax\_B, 2  
.text:0042F3DA                 mov     edi, \[esp+14h+A\]  
.text:0042F3DE                 shl     esi, 1Eh  
.text:0042F3E1                 mov     ebp, \[ebx+2Ch\]  
.text:0042F3E4                 or      esi, eax\_B  
.text:0042F3E6                 xor     ebp, \[ebx+18h\]  
.text:0042F3E9                 mov     eax\_B, esi      ; R3(A,B,C,D,E,45)  
.text:0042F3E9                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.5 次运算 END\*\*\*\*  
.text:0042F3EB                 mov     esi, \[esp+14h+A\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.6 次运算开始\*\*\*\*  
.text:0042F3EF                 or      esi, eax\_B  
.text:0042F3F1                 and     edi, eax\_B  
.text:0042F3F3                 and     esi, edx\_C  
.text:0042F3F5                 xor     ebp, \[ebx\]  
.text:0042F3F7                 or      esi, edi  
.text:0042F3F9                 mov     edi, \[ebx+2Ch\]  
.text:0042F3FC                 xor     edi, \[ebx+18h\]  
.text:0042F3FF                 xor     ebp, \[ebx+38h\]  
.text:0042F402                 xor     edi, \[ebx\]  
.text:0042F404                 xor     edi, \[ebx+38h\]  
.text:0042F407                 shr     ebp, 1Fh  
.text:0042F40A                 add     edi, edi  
.text:0042F40C                 or      edi, ebp  
.text:0042F40E                 mov     \[ebx+38h\], edi  
.text:0042F411                 add     esi, edi  
.text:0042F413                 mov     edi, \[esp+14h+E\]  
.text:0042F417                 mov     ebp, \[esp+14h+E\]  
.text:0042F41B                 shr     ebp, 1Bh  
.text:0042F41E                 shl     edi, 5  
.text:0042F421                 or      edi, ebp  
.text:0042F423                 add     esi, edi  
.text:0042F425                 mov     edi, \[esp+14h+A\]  
.text:0042F429                 add     esi, 8F1BBCDCh  
.text:0042F42F                 add     ecx\_D, esi  
.text:0042F431                 mov     esi, \[esp+14h+A\]  
.text:0042F435                 shl     esi, 1Eh  
.text:0042F438                 shr     edi, 2  
.text:0042F43B                 or      esi, edi  
.text:0042F43D                 mov     \[esp+14h+A\], esi ; R3(E,A,B,C,D,46)  
.text:0042F43D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.6 次运算 END\*\*\*\*  
.text:0042F441                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.7 次运算开始\*\*\*\*  
.text:0042F445                 mov     edi, \[esp+14h+E\]  
.text:0042F449                 or      esi, \[esp+14h+A\]  
.text:0042F44D                 and     edi, \[esp+14h+A\]  
.text:0042F451                 and     esi, eax\_B  
.text:0042F453                 mov     ebp, \[ebx+30h\]  
.text:0042F456                 or      esi, edi  
.text:0042F458                 mov     edi, \[ebx+30h\]  
.text:0042F45B                 xor     ebp, \[ebx+1Ch\]  
.text:0042F45E                 xor     edi, \[ebx+1Ch\]  
.text:0042F461                 xor     ebp, \[ebx+4\]  
.text:0042F464                 xor     edi, \[ebx+4\]  
.text:0042F467                 xor     ebp, \[ebx+3Ch\]  
.text:0042F46A                 xor     edi, \[ebx+3Ch\]  
.text:0042F46D                 shr     ebp, 1Fh  
.text:0042F470                 add     edi, edi  
.text:0042F472                 or      edi, ebp  
.text:0042F474                 mov     ebp, ecx\_D  
.text:0042F476                 mov     \[ebx+3Ch\], edi  
.text:0042F479                 add     esi, edi  
.text:0042F47B                 mov     edi, ecx\_D  
.text:0042F47D                 shl     edi, 5  
.text:0042F480                 shr     ebp, 1Bh  
.text:0042F483                 or      edi, ebp  
.text:0042F485                 add     esi, edi  
.text:0042F487                 mov     edi, \[esp+14h+E\]  
.text:0042F48B                 add     esi, 8F1BBCDCh  
.text:0042F491                 add     edx\_C, esi  
.text:0042F493                 mov     esi, \[esp+14h+E\]  
.text:0042F497                 shl     esi, 1Eh  
.text:0042F49A                 shr     edi, 2  
.text:0042F49D                 or      esi, edi  
.text:0042F49F                 mov     \[esp+14h+E\], esi ; R3(D,E,A,B,C,47)  
.text:0042F49F                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.7 次运算 END\*\*\*\*  
.text:0042F4A3                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.8 次运算开始\*\*\*\*  
.text:0042F4A7                 mov     edi, \[esp+14h+E\]  
.text:0042F4AB                 or      esi, ecx\_D  
.text:0042F4AD                 and     edi, ecx\_D  
.text:0042F4AF                 and     esi, \[esp+14h+A\]  
.text:0042F4B3                 mov     ebp, \[ebx+34h\]  
.text:0042F4B6                 or      esi, edi  
.text:0042F4B8                 mov     edi, \[ebx+34h\]  
.text:0042F4BB                 xor     ebp, \[ebx+20h\]  
.text:0042F4BE                 xor     edi, \[ebx+20h\]  
.text:0042F4C1                 xor     ebp, \[ebx+8\]  
.text:0042F4C4                 xor     edi, \[ebx+8\]  
.text:0042F4C7                 xor     ebp, \[ebx\]  
.text:0042F4C9                 xor     edi, \[ebx\]  
.text:0042F4CB                 shr     ebp, 1Fh  
.text:0042F4CE                 add     edi, edi  
.text:0042F4D0                 or      edi, ebp  
.text:0042F4D2                 mov     ebp, edx\_C  
.text:0042F4D4                 add     esi, edi  
.text:0042F4D6                 mov     \[ebx\], edi  
.text:0042F4D8                 mov     edi, edx\_C  
.text:0042F4DA                 shr     ebp, 1Bh  
.text:0042F4DD                 shl     edi, 5  
.text:0042F4E0                 or      edi, ebp  
.text:0042F4E2                 add     esi, edi  
.text:0042F4E4                 add     esi, 8F1BBCDCh  
.text:0042F4EA                 add     eax\_B, esi  
.text:0042F4EC                 mov     esi, ecx\_D  
.text:0042F4EE                 shl     esi, 1Eh  
.text:0042F4F1                 shr     ecx\_D, 2  
.text:0042F4F4                 or      esi, ecx\_D  
.text:0042F4F6                 mov     ebp, \[ebx+38h\]  
.text:0042F4F9                 mov     ecx\_D, esi      ; R3(C,D,E,A,B,48)  
.text:0042F4F9                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.8 次运算 END\*\*\*\*  
.text:0042F4FB                 xor     ebp, \[ebx+24h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.9 次运算开始\*\*\*\*  
.text:0042F4FE                 mov     esi, ecx\_D  
.text:0042F500                 mov     edi, ecx\_D  
.text:0042F502                 or      esi, edx\_C  
.text:0042F504                 and     edi, edx\_C  
.text:0042F506                 and     esi, \[esp+14h+E\]  
.text:0042F50A                 xor     ebp, \[ebx+0Ch\]  
.text:0042F50D                 or      esi, edi  
.text:0042F50F                 mov     edi, \[ebx+38h\]  
.text:0042F512                 xor     edi, \[ebx+24h\]  
.text:0042F515                 xor     ebp, \[ebx+4\]  
.text:0042F518                 xor     edi, \[ebx+0Ch\]  
.text:0042F51B                 xor     edi, \[ebx+4\]  
.text:0042F51E                 shr     ebp, 1Fh  
.text:0042F521                 add     edi, edi  
.text:0042F523                 or      edi, ebp  
.text:0042F525                 mov     ebp, eax\_B  
.text:0042F527                 mov     \[ebx+4\], edi  
.text:0042F52A                 add     esi, edi  
.text:0042F52C                 mov     edi, eax\_B  
.text:0042F52E                 shl     edi, 5  
.text:0042F531                 shr     ebp, 1Bh  
.text:0042F534                 or      edi, ebp  
.text:0042F536                 add     esi, edi  
.text:0042F538                 add     esi, 8F1BBCDCh  
.text:0042F53E                 add     \[esp+14h+A\], esi  
.text:0042F542                 mov     esi, edx\_C  
.text:0042F544                 shr     edx\_C, 2  
.text:0042F547                 mov     ebp, \[ebx+3Ch\]  
.text:0042F54A                 shl     esi, 1Eh  
.text:0042F54D                 xor     ebp, \[ebx+28h\]  
.text:0042F550                 or      esi, edx\_C  
.text:0042F552                 xor     ebp, \[ebx+10h\]  
.text:0042F555                 mov     edx\_C, esi      ; R3(B,C,D,E,A,49)  
.text:0042F555                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.9 次运算 END\*\*\*\*  
.text:0042F557                 xor     ebp, \[ebx+8\]    ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.10 次运算开始\*\*\*\*  
.text:0042F55A                 mov     esi, edx\_C  
.text:0042F55C                 mov     edi, edx\_C  
.text:0042F55E                 or      esi, eax\_B  
.text:0042F560                 and     edi, eax\_B  
.text:0042F562                 and     esi, ecx\_D  
.text:0042F564                 or      esi, edi  
.text:0042F566                 mov     edi, \[ebx+3Ch\]  
.text:0042F569                 xor     edi, \[ebx+28h\]  
.text:0042F56C                 xor     edi, \[ebx+10h\]  
.text:0042F56F                 xor     edi, \[ebx+8\]  
.text:0042F572                 shr     ebp, 1Fh  
.text:0042F575                 add     edi, edi  
.text:0042F577                 or      edi, ebp  
.text:0042F579                 mov     \[ebx+8\], edi  
.text:0042F57C                 add     esi, edi  
.text:0042F57E                 mov     edi, \[esp+14h+A\]  
.text:0042F582                 mov     ebp, \[esp+14h+A\]  
.text:0042F586                 shr     ebp, 1Bh  
.text:0042F589                 shl     edi, 5  
.text:0042F58C                 or      edi, ebp  
.text:0042F58E                 add     esi, edi  
.text:0042F590                 add     esi, 8F1BBCDCh  
.text:0042F596                 add     \[esp+14h+E\], esi  
.text:0042F59A                 mov     esi, eax\_B  
.text:0042F59C                 shr     eax\_B, 2  
.text:0042F59F                 mov     edi, \[esp+14h+A\]  
.text:0042F5A3                 shl     esi, 1Eh  
.text:0042F5A6                 mov     ebp, \[ebx\]  
.text:0042F5A8                 or      esi, eax\_B  
.text:0042F5AA                 xor     ebp, \[ebx+2Ch\]  
.text:0042F5AD                 mov     eax\_B, esi      ; R3(A,B,C,D,E,50)  
.text:0042F5AD                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.10 次运算 END\*\*\*\*  
.text:0042F5AF                 mov     esi, \[esp+14h+A\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.11 次运算开始\*\*\*\*  
.text:0042F5B3                 or      esi, eax\_B  
.text:0042F5B5                 and     edi, eax\_B  
.text:0042F5B7                 and     esi, edx\_C  
.text:0042F5B9                 xor     ebp, \[ebx+14h\]  
.text:0042F5BC                 or      esi, edi  
.text:0042F5BE                 mov     edi, \[ebx\]  
.text:0042F5C0                 xor     edi, \[ebx+2Ch\]  
.text:0042F5C3                 xor     ebp, \[ebx+0Ch\]  
.text:0042F5C6                 xor     edi, \[ebx+14h\]  
.text:0042F5C9                 xor     edi, \[ebx+0Ch\]  
.text:0042F5CC                 shr     ebp, 1Fh  
.text:0042F5CF                 add     edi, edi  
.text:0042F5D1                 or      edi, ebp  
.text:0042F5D3                 mov     \[ebx+0Ch\], edi  
.text:0042F5D6                 add     esi, edi  
.text:0042F5D8                 mov     edi, \[esp+14h+E\]  
.text:0042F5DC                 mov     ebp, \[esp+14h+E\]  
.text:0042F5E0                 shr     ebp, 1Bh  
.text:0042F5E3                 shl     edi, 5  
.text:0042F5E6                 or      edi, ebp  
.text:0042F5E8                 add     esi, edi  
.text:0042F5EA                 mov     edi, \[esp+14h+A\]  
.text:0042F5EE                 add     esi, 8F1BBCDCh  
.text:0042F5F4                 add     ecx\_D, esi  
.text:0042F5F6                 mov     esi, \[esp+14h+A\]  
.text:0042F5FA                 shl     esi, 1Eh  
.text:0042F5FD                 shr     edi, 2  
.text:0042F600                 or      esi, edi  
.text:0042F602                 mov     \[esp+14h+A\], esi ; R3(E,A,B,C,D,51)  
.text:0042F602                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.11 次运算 END\*\*\*\*  
.text:0042F606                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.12 次运算开始\*\*\*\*  
.text:0042F60A                 mov     edi, \[esp+14h+E\]  
.text:0042F60E                 or      esi, \[esp+14h+A\]  
.text:0042F612                 and     edi, \[esp+14h+A\]  
.text:0042F616                 and     esi, eax\_B  
.text:0042F618                 mov     ebp, \[ebx+4\]  
.text:0042F61B                 or      esi, edi  
.text:0042F61D                 mov     edi, \[ebx+4\]  
.text:0042F620                 xor     ebp, \[ebx+30h\]  
.text:0042F623                 xor     edi, \[ebx+30h\]  
.text:0042F626                 xor     ebp, \[ebx+18h\]  
.text:0042F629                 xor     edi, \[ebx+18h\]  
.text:0042F62C                 xor     ebp, \[ebx+10h\]  
.text:0042F62F                 xor     edi, \[ebx+10h\]  
.text:0042F632                 shr     ebp, 1Fh  
.text:0042F635                 add     edi, edi  
.text:0042F637                 or      edi, ebp  
.text:0042F639                 mov     ebp, ecx\_D  
.text:0042F63B                 mov     \[ebx+10h\], edi  
.text:0042F63E                 add     esi, edi  
.text:0042F640                 mov     edi, ecx\_D  
.text:0042F642                 shl     edi, 5  
.text:0042F645                 shr     ebp, 1Bh  
.text:0042F648                 or      edi, ebp  
.text:0042F64A                 add     esi, edi  
.text:0042F64C                 mov     edi, \[esp+14h+E\]  
.text:0042F650                 add     esi, 8F1BBCDCh  
.text:0042F656                 add     edx\_C, esi  
.text:0042F658                 mov     esi, \[esp+14h+E\]  
.text:0042F65C                 shl     esi, 1Eh  
.text:0042F65F                 shr     edi, 2  
.text:0042F662                 or      esi, edi  
.text:0042F664                 mov     \[esp+14h+E\], esi ; R3(D,E,A,B,C,D,52)  
.text:0042F664                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.12 次运算 END\*\*\*\*  
.text:0042F668                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.13 次运算开始\*\*\*\*  
.text:0042F66C                 mov     edi, \[esp+14h+E\]  
.text:0042F670                 or      esi, ecx\_D  
.text:0042F672                 and     edi, ecx\_D  
.text:0042F674                 and     esi, \[esp+14h+A\]  
.text:0042F678                 mov     ebp, \[ebx+8\]  
.text:0042F67B                 or      esi, edi  
.text:0042F67D                 mov     edi, \[ebx+8\]  
.text:0042F680                 xor     ebp, \[ebx+34h\]  
.text:0042F683                 xor     edi, \[ebx+34h\]  
.text:0042F686                 xor     ebp, \[ebx+1Ch\]  
.text:0042F689                 xor     edi, \[ebx+1Ch\]  
.text:0042F68C                 xor     ebp, \[ebx+14h\]  
.text:0042F68F                 xor     edi, \[ebx+14h\]  
.text:0042F692                 shr     ebp, 1Fh  
.text:0042F695                 add     edi, edi  
.text:0042F697                 or      edi, ebp  
.text:0042F699                 mov     ebp, edx\_C  
.text:0042F69B                 mov     \[ebx+14h\], edi  
.text:0042F69E                 add     esi, edi  
.text:0042F6A0                 mov     edi, edx\_C  
.text:0042F6A2                 shl     edi, 5  
.text:0042F6A5                 shr     ebp, 1Bh  
.text:0042F6A8                 or      edi, ebp  
.text:0042F6AA                 mov     ebp, \[ebx+0Ch\]  
.text:0042F6AD                 add     esi, edi  
.text:0042F6AF                 xor     ebp, \[ebx+38h\]  
.text:0042F6B2                 add     esi, 8F1BBCDCh  
.text:0042F6B8                 xor     ebp, \[ebx+20h\]  
.text:0042F6BB                 add     eax\_B, esi  
.text:0042F6BD                 mov     esi, ecx\_D  
.text:0042F6BF                 shr     ecx\_D, 2  
.text:0042F6C2                 xor     ebp, \[ebx+18h\]  
.text:0042F6C5                 shl     esi, 1Eh  
.text:0042F6C8                 or      esi, ecx\_D  
.text:0042F6CA                 mov     ecx\_D, esi      ; R3(C,D,E,A,B,53)  
.text:0042F6CA                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.13 次运算 END\*\*\*\*  
.text:0042F6CC                 mov     esi, ecx\_D      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.14 次运算开始\*\*\*\*  
.text:0042F6CE                 mov     edi, ecx\_D  
.text:0042F6D0                 or      esi, edx\_C  
.text:0042F6D2                 and     edi, edx\_C  
.text:0042F6D4                 and     esi, \[esp+14h+E\]  
.text:0042F6D8                 or      esi, edi  
.text:0042F6DA                 mov     edi, \[ebx+0Ch\]  
.text:0042F6DD                 xor     edi, \[ebx+38h\]  
.text:0042F6E0                 xor     edi, \[ebx+20h\]  
.text:0042F6E3                 xor     edi, \[ebx+18h\]  
.text:0042F6E6                 shr     ebp, 1Fh  
.text:0042F6E9                 add     edi, edi  
.text:0042F6EB                 or      edi, ebp  
.text:0042F6ED                 mov     ebp, eax\_B  
.text:0042F6EF                 mov     \[ebx+18h\], edi  
.text:0042F6F2                 add     esi, edi  
.text:0042F6F4                 mov     edi, eax\_B  
.text:0042F6F6                 shl     edi, 5  
.text:0042F6F9                 shr     ebp, 1Bh  
.text:0042F6FC                 or      edi, ebp  
.text:0042F6FE                 add     esi, edi  
.text:0042F700                 add     esi, 8F1BBCDCh  
.text:0042F706                 add     \[esp+14h+A\], esi  
.text:0042F70A                 mov     esi, edx\_C  
.text:0042F70C                 shr     edx\_C, 2  
.text:0042F70F                 mov     ebp, \[ebx+10h\]  
.text:0042F712                 shl     esi, 1Eh  
.text:0042F715                 xor     ebp, \[ebx+3Ch\]  
.text:0042F718                 or      esi, edx\_C  
.text:0042F71A                 xor     ebp, \[ebx+24h\]  
.text:0042F71D                 mov     edx\_C, esi      ; R3(B,C,D,E,A,54)  
.text:0042F71D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.14 次运算 END\*\*\*\*  
.text:0042F71F                 xor     ebp, \[ebx+1Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.15 次运算开始\*\*\*\*  
.text:0042F722                 mov     esi, edx\_C  
.text:0042F724                 mov     edi, edx\_C  
.text:0042F726                 or      esi, eax\_B  
.text:0042F728                 and     edi, eax\_B  
.text:0042F72A                 and     esi, ecx\_D  
.text:0042F72C                 or      esi, edi  
.text:0042F72E                 mov     edi, \[ebx+10h\]  
.text:0042F731                 xor     edi, \[ebx+3Ch\]  
.text:0042F734                 xor     edi, \[ebx+24h\]  
.text:0042F737                 xor     edi, \[ebx+1Ch\]  
.text:0042F73A                 shr     ebp, 1Fh  
.text:0042F73D                 add     edi, edi  
.text:0042F73F                 or      edi, ebp  
.text:0042F741                 mov     \[ebx+1Ch\], edi  
.text:0042F744                 add     esi, edi  
.text:0042F746                 mov     edi, \[esp+14h+A\]  
.text:0042F74A                 mov     ebp, \[esp+14h+A\]  
.text:0042F74E                 shr     ebp, 1Bh  
.text:0042F751                 shl     edi, 5  
.text:0042F754                 or      edi, ebp  
.text:0042F756                 add     esi, edi  
.text:0042F758                 add     esi, 8F1BBCDCh  
.text:0042F75E                 add     \[esp+14h+E\], esi  
.text:0042F762                 mov     esi, eax\_B  
.text:0042F764                 shr     eax\_B, 2  
.text:0042F767                 mov     edi, \[esp+14h+A\]  
.text:0042F76B                 shl     esi, 1Eh  
.text:0042F76E                 mov     ebp, \[ebx+14h\]  
.text:0042F771                 or      esi, eax\_B  
.text:0042F773                 xor     ebp, \[ebx\]  
.text:0042F775                 mov     eax\_B, esi      ; R3(A,B,C,D,E,55)  
.text:0042F775                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.15 次运算 END\*\*\*\*  
.text:0042F777                 mov     esi, \[esp+14h+A\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.16 次运算开始\*\*\*\*  
.text:0042F77B                 or      esi, eax\_B  
.text:0042F77D                 and     edi, eax\_B  
.text:0042F77F                 and     esi, edx\_C  
.text:0042F781                 xor     ebp, \[ebx+28h\]  
.text:0042F784                 or      esi, edi  
.text:0042F786                 mov     edi, \[ebx+14h\]  
.text:0042F789                 xor     edi, \[ebx\]  
.text:0042F78B                 xor     ebp, \[ebx+20h\]  
.text:0042F78E                 xor     edi, \[ebx+28h\]  
.text:0042F791                 xor     edi, \[ebx+20h\]  
.text:0042F794                 shr     ebp, 1Fh  
.text:0042F797                 add     edi, edi  
.text:0042F799                 or      edi, ebp  
.text:0042F79B                 mov     \[ebx+20h\], edi  
.text:0042F79E                 add     esi, edi  
.text:0042F7A0                 mov     edi, \[esp+14h+E\]  
.text:0042F7A4                 mov     ebp, \[esp+14h+E\]  
.text:0042F7A8                 shr     ebp, 1Bh  
.text:0042F7AB                 shl     edi, 5  
.text:0042F7AE                 or      edi, ebp  
.text:0042F7B0                 add     esi, edi  
.text:0042F7B2                 add     esi, 8F1BBCDCh  
.text:0042F7B8                 add     ecx\_D, esi  
.text:0042F7BA                 mov     esi, \[esp+14h+A\]  
.text:0042F7BE                 mov     edi, \[esp+14h+A\]  
.text:0042F7C2                 shr     edi, 2  
.text:0042F7C5                 shl     esi, 1Eh  
.text:0042F7C8                 or      esi, edi  
.text:0042F7CA                 mov     \[esp+14h+A\], esi ; R3(E,A,B,C,D,56)  
.text:0042F7CA                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.16 次运算 END\*\*\*\*  
.text:0042F7CE                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.17 次运算开始\*\*\*\*  
.text:0042F7D2                 mov     edi, \[esp+14h+E\]  
.text:0042F7D6                 or      esi, \[esp+14h+A\]  
.text:0042F7DA                 and     edi, \[esp+14h+A\]  
.text:0042F7DE                 and     esi, eax\_B  
.text:0042F7E0                 mov     ebp, \[ebx+18h\]  
.text:0042F7E3                 or      esi, edi  
.text:0042F7E5                 mov     edi, \[ebx+18h\]  
.text:0042F7E8                 xor     ebp, \[ebx+4\]  
.text:0042F7EB                 xor     edi, \[ebx+4\]  
.text:0042F7EE                 xor     ebp, \[ebx+2Ch\]  
.text:0042F7F1                 xor     edi, \[ebx+2Ch\]  
.text:0042F7F4                 xor     ebp, \[ebx+24h\]  
.text:0042F7F7                 xor     edi, \[ebx+24h\]  
.text:0042F7FA                 shr     ebp, 1Fh  
.text:0042F7FD                 add     edi, edi  
.text:0042F7FF                 or      edi, ebp  
.text:0042F801                 mov     ebp, ecx\_D  
.text:0042F803                 mov     \[ebx+24h\], edi  
.text:0042F806                 add     esi, edi  
.text:0042F808                 mov     edi, ecx\_D  
.text:0042F80A                 shl     edi, 5  
.text:0042F80D                 shr     ebp, 1Bh  
.text:0042F810                 or      edi, ebp  
.text:0042F812                 add     esi, edi  
.text:0042F814                 mov     edi, \[esp+14h+E\]  
.text:0042F818                 add     esi, 8F1BBCDCh  
.text:0042F81E                 add     edx\_C, esi  
.text:0042F820                 mov     esi, \[esp+14h+E\]  
.text:0042F824                 shl     esi, 1Eh  
.text:0042F827                 shr     edi, 2  
.text:0042F82A                 or      esi, edi  
.text:0042F82C                 mov     \[esp+14h+E\], esi ; R3(D,E,A,B,C,57)  
.text:0042F82C                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.17 次运算 END\*\*\*\*  
.text:0042F830                 mov     esi, \[esp+14h+E\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.18 次运算开始\*\*\*\*  
.text:0042F834                 mov     edi, \[esp+14h+E\]  
.text:0042F838                 or      esi, ecx\_D  
.text:0042F83A                 and     edi, ecx\_D  
.text:0042F83C                 and     esi, \[esp+14h+A\]  
.text:0042F840                 mov     ebp, \[ebx+1Ch\]  
.text:0042F843                 or      esi, edi  
.text:0042F845                 mov     edi, \[ebx+1Ch\]  
.text:0042F848                 xor     ebp, \[ebx+8\]  
.text:0042F84B                 xor     edi, \[ebx+8\]  
.text:0042F84E                 xor     ebp, \[ebx+30h\]  
.text:0042F851                 xor     edi, \[ebx+30h\]  
.text:0042F854                 xor     ebp, \[ebx+28h\]  
.text:0042F857                 xor     edi, \[ebx+28h\]  
.text:0042F85A                 shr     ebp, 1Fh  
.text:0042F85D                 add     edi, edi  
.text:0042F85F                 or      edi, ebp  
.text:0042F861                 mov     ebp, edx\_C  
.text:0042F863                 mov     \[ebx+28h\], edi  
.text:0042F866                 add     esi, edi  
.text:0042F868                 mov     edi, edx\_C  
.text:0042F86A                 shl     edi, 5  
.text:0042F86D                 shr     ebp, 1Bh  
.text:0042F870                 or      edi, ebp  
.text:0042F872                 mov     ebp, \[ebx+20h\]  
.text:0042F875                 add     esi, edi  
.text:0042F877                 xor     ebp, \[ebx+0Ch\]  
.text:0042F87A                 add     esi, 8F1BBCDCh  
.text:0042F880                 xor     ebp, \[ebx+34h\]  
.text:0042F883                 add     eax\_B, esi  
.text:0042F885                 mov     esi, ecx\_D  
.text:0042F887                 shr     ecx\_D, 2  
.text:0042F88A                 xor     ebp, \[ebx+2Ch\]  
.text:0042F88D                 shl     esi, 1Eh  
.text:0042F890                 or      esi, ecx\_D  
.text:0042F892                 mov     ecx\_D, esi      ; R3(C,D,E,A,B,58)  
.text:0042F892                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.18 次运算 END\*\*\*\*  
.text:0042F894                 mov     esi, ecx\_D      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.19 次运算开始\*\*\*\*  
.text:0042F896                 mov     edi, ecx\_D  
.text:0042F898                 or      esi, edx\_C  
.text:0042F89A                 and     edi, edx\_C  
.text:0042F89C                 and     esi, \[esp+14h+E\]  
.text:0042F8A0                 or      esi, edi  
.text:0042F8A2                 mov     edi, \[ebx+20h\]  
.text:0042F8A5                 xor     edi, \[ebx+0Ch\]  
.text:0042F8A8                 xor     edi, \[ebx+34h\]  
.text:0042F8AB                 xor     edi, \[ebx+2Ch\]  
.text:0042F8AE                 shr     ebp, 1Fh  
.text:0042F8B1                 add     edi, edi  
.text:0042F8B3                 or      edi, ebp  
.text:0042F8B5                 mov     ebp, eax\_B  
.text:0042F8B7                 mov     \[ebx+2Ch\], edi  
.text:0042F8BA                 add     esi, edi  
.text:0042F8BC                 mov     edi, eax\_B  
.text:0042F8BE                 shl     edi, 5  
.text:0042F8C1                 shr     ebp, 1Bh  
.text:0042F8C4                 or      edi, ebp  
.text:0042F8C6                 add     esi, edi  
.text:0042F8C8                 add     esi, 8F1BBCDCh  
.text:0042F8CE                 add     \[esp+14h+A\], esi  
.text:0042F8D2                 mov     esi, edx\_C  
.text:0042F8D4                 shr     edx\_C, 2  
.text:0042F8D7                 mov     edi, \[ebx+UI32.\_9\]  
.text:0042F8DA                 shl     esi, 1Eh  
.text:0042F8DD                 xor     edi, \[ebx+UI32.\_4\]  
.text:0042F8E0                 or      esi, edx\_C  
.text:0042F8E2                 xor     edi, \[ebx+UI32.\_14\]  
.text:0042F8E5                 mov     edx\_C, esi      ; R3(B,C,D,E,A,59)  
.text:0042F8E5                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3.19 次运算 END\*\*\*\*  
.text:0042F8E5                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 3 轮计算 END\*\*\*\*  
.text:0042F8E7                 mov     esi, \[ebx+UI32.\_9\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4 轮计算开始\*\*\*\*  
.text:0042F8E7                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.0 次运算开始\*\*\*\*  
.text:0042F8EA                 xor     esi, \[ebx+UI32.\_4\] ; buf\[9\] ^ buf\[4\]  
.text:0042F8ED                 xor     edi, \[ebx+UI32.\_12\]  
.text:0042F8F0                 xor     esi, \[ebx+UI32.\_14\] ; buf\[9\] ^ buf\[4\] ^ buf\[14\]  
.text:0042F8F3                 xor     esi, \[ebx+UI32.\_12\] ; buf\[9\] ^ buf\[4\] ^ buf\[14\] ^ buf\[12\]  
.text:0042F8F6                 shr     edi, 1Fh        ; (buf\[9\] ^ buf\[4\] ^ buf\[14\] ^ buf\[12\])>>31  
.text:0042F8F9                 add     esi, esi        ; (buf\[9\] ^ buf\[4\] ^ buf\[14\] ^ buf\[12\])<<1  
.text:0042F8FB                 or      esi, edi        ; RL((buf\[9\] ^ buf\[4\] ^ buf\[14\] ^ buf\[12\]),1)  
.text:0042F8FD                 mov     edi, edx\_C  
.text:0042F8FF                 xor     edi, eax\_B      ; B^C  
.text:0042F901                 mov     \[ebx+UI32.\_12\], esi ; RXL\[60\]  
.text:0042F904                 xor     edi, ecx\_D      ; RX1(B,C,D) = B^C^D  
.text:0042F906                 mov     ebp, \[esp+14h+A\]  
.text:0042F90A                 add     esi, edi        ; RXL\[60\] + RX1(B,C,D)  
.text:0042F90C                 mov     edi, \[esp+14h+A\]  
.text:0042F910                 shl     edi, 5          ; A<<5  
.text:0042F913                 shr     ebp, 1Bh        ; A>>27  
.text:0042F916                 or      edi, ebp        ; RL(A,5)  
.text:0042F918                 add     esi, edi        ; RXL\[60\] + RX1(B,C,D) + RL(A,5)  
.text:0042F91A                 add     esi, 0CA62C1D6h  
.text:0042F920                 add     \[esp+14h+E\], esi ; E += RXL\[60\] + RX1(B,C,D) + RL(A,5) + 0xca62c1d6  
.text:0042F924                 mov     esi, eax\_B  
.text:0042F926                 shr     eax\_B, 2        ; B>>2  
.text:0042F929                 mov     edi, \[ebx+UI32.\_10\]  
.text:0042F92C                 shl     esi, 1Eh        ; B<<30  
.text:0042F92F                 xor     edi, \[ebx+UI32.\_5\]  
.text:0042F932                 or      esi, eax\_B      ; RL(B,30)  
.text:0042F934                 xor     edi, \[ebx+UI32.\_15\]  
.text:0042F937                 mov     eax\_B, esi      ; B = RL(B,30)  
.text:0042F937                                         ;  
.text:0042F937                                         ; 定义宏 R4:  
.text:0042F937                                         ; #define R4(a,b,c,d,e,i) ( e+=RXL(i)+RX1(b,c,d)+RL(a,5)+0xca62c1d6; b=RL(b,30); )  
.text:0042F937                                         ;  
.text:0042F937                                         ; 那么这里用 R4 表示为: R4(A,B,C,D,E,60)  
.text:0042F937                                         ;  
.text:0042F937                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.0 次运算 END\*\*\*\*  
.text:0042F939                 mov     esi, \[ebx+28h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.1 次运算开始\*\*\*\*  
.text:0042F93C                 xor     esi, \[ebx+14h\]  
.text:0042F93F                 xor     edi, \[ebx+34h\]  
.text:0042F942                 xor     esi, \[ebx+3Ch\]  
.text:0042F945                 xor     esi, \[ebx+34h\]  
.text:0042F948                 shr     edi, 1Fh  
.text:0042F94B                 add     esi, esi  
.text:0042F94D                 or      esi, edi  
.text:0042F94F                 mov     \[ebx+34h\], esi  
.text:0042F952                 mov     edi, \[esp+14h+A\]  
.text:0042F956                 mov     ebp, \[esp+14h+E\]  
.text:0042F95A                 xor     edi, eax\_B  
.text:0042F95C                 xor     edi, edx\_C  
.text:0042F95E                 add     esi, edi  
.text:0042F960                 mov     edi, \[esp+14h+E\]  
.text:0042F964                 shl     edi, 5  
.text:0042F967                 shr     ebp, 1Bh  
.text:0042F96A                 or      edi, ebp  
.text:0042F96C                 add     esi, edi  
.text:0042F96E                 mov     edi, \[esp+14h+A\]  
.text:0042F972                 add     esi, 0CA62C1D6h  
.text:0042F978                 add     ecx\_D, esi  
.text:0042F97A                 mov     esi, \[esp+14h+A\]  
.text:0042F97E                 shl     esi, 1Eh  
.text:0042F981                 mov     ebp, ecx\_D  
.text:0042F983                 shr     edi, 2  
.text:0042F986                 or      esi, edi  
.text:0042F988                 mov     \[esp+14h+A\], esi ; R4(E,A,B,C,D,61)  
.text:0042F988                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.1 次运算 END\*\*\*\*  
.text:0042F98C                 mov     edi, \[ebx+2Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.2 次运算开始\*\*\*\*  
.text:0042F98F                 mov     esi, \[ebx+2Ch\]  
.text:0042F992                 xor     edi, \[ebx+18h\]  
.text:0042F995                 xor     esi, \[ebx+18h\]  
.text:0042F998                 xor     edi, \[ebx\]  
.text:0042F99A                 xor     esi, \[ebx\]  
.text:0042F99C                 xor     edi, \[ebx+38h\]  
.text:0042F99F                 xor     esi, \[ebx+38h\]  
.text:0042F9A2                 shr     edi, 1Fh  
.text:0042F9A5                 add     esi, esi  
.text:0042F9A7                 or      esi, edi  
.text:0042F9A9                 mov     \[ebx+38h\], esi  
.text:0042F9AC                 mov     edi, \[esp+14h+E\]  
.text:0042F9B0                 xor     edi, \[esp+14h+A\]  
.text:0042F9B4                 xor     edi, eax\_B  
.text:0042F9B6                 add     esi, edi  
.text:0042F9B8                 mov     edi, ecx\_D  
.text:0042F9BA                 shl     edi, 5  
.text:0042F9BD                 shr     ebp, 1Bh  
.text:0042F9C0                 or      edi, ebp  
.text:0042F9C2                 add     esi, edi  
.text:0042F9C4                 mov     edi, \[esp+14h+E\]  
.text:0042F9C8                 add     esi, 0CA62C1D6h  
.text:0042F9CE                 add     edx\_C, esi  
.text:0042F9D0                 mov     esi, \[esp+14h+E\]  
.text:0042F9D4                 shl     esi, 1Eh  
.text:0042F9D7                 mov     ebp, edx\_C  
.text:0042F9D9                 shr     edi, 2  
.text:0042F9DC                 or      esi, edi  
.text:0042F9DE                 mov     \[esp+14h+E\], esi ; R4(D,E,A,B,C,62)  
.text:0042F9DE                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.2 次运算 END\*\*\*\*  
.text:0042F9E2                 mov     edi, \[ebx+30h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.3 次运算开始\*\*\*\*  
.text:0042F9E5                 mov     esi, \[ebx+30h\]  
.text:0042F9E8                 xor     edi, \[ebx+1Ch\]  
.text:0042F9EB                 xor     esi, \[ebx+1Ch\]  
.text:0042F9EE                 xor     edi, \[ebx+4\]  
.text:0042F9F1                 xor     esi, \[ebx+4\]  
.text:0042F9F4                 xor     edi, \[ebx+3Ch\]  
.text:0042F9F7                 xor     esi, \[ebx+3Ch\]  
.text:0042F9FA                 shr     edi, 1Fh  
.text:0042F9FD                 add     esi, esi  
.text:0042F9FF                 or      esi, edi  
.text:0042FA01                 mov     \[ebx+3Ch\], esi  
.text:0042FA04                 mov     edi, \[esp+14h+E\]  
.text:0042FA08                 xor     edi, ecx\_D  
.text:0042FA0A                 xor     edi, \[esp+14h+A\]  
.text:0042FA0E                 add     esi, edi  
.text:0042FA10                 mov     edi, edx\_C  
.text:0042FA12                 shl     edi, 5  
.text:0042FA15                 shr     ebp, 1Bh  
.text:0042FA18                 or      edi, ebp  
.text:0042FA1A                 add     esi, edi  
.text:0042FA1C                 mov     edi, \[ebx+34h\]  
.text:0042FA1F                 add     esi, 0CA62C1D6h  
.text:0042FA25                 xor     edi, \[ebx+20h\]  
.text:0042FA28                 add     eax\_B, esi  
.text:0042FA2A                 mov     esi, ecx\_D  
.text:0042FA2C                 shr     ecx\_D, 2  
.text:0042FA2F                 mov     ebp, eax\_B  
.text:0042FA31                 shl     esi, 1Eh  
.text:0042FA34                 xor     edi, \[ebx+8\]  
.text:0042FA37                 or      esi, ecx\_D  
.text:0042FA39                 xor     edi, \[ebx\]  
.text:0042FA3B                 mov     ecx\_D, esi      ; R4(C,D,E,A,B,63)  
.text:0042FA3B                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.3 次运算 END\*\*\*\*  
.text:0042FA3D                 mov     esi, \[ebx+34h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.4 次运算开始\*\*\*\*  
.text:0042FA40                 xor     esi, \[ebx+20h\]  
.text:0042FA43                 xor     esi, \[ebx+8\]  
.text:0042FA46                 xor     esi, \[ebx\]  
.text:0042FA48                 shr     edi, 1Fh  
.text:0042FA4B                 add     esi, esi  
.text:0042FA4D                 or      esi, edi  
.text:0042FA4F                 mov     edi, ecx\_D  
.text:0042FA51                 mov     \[ebx\], esi  
.text:0042FA53                 xor     edi, edx\_C  
.text:0042FA55                 xor     edi, \[esp+14h+E\]  
.text:0042FA59                 add     esi, edi  
.text:0042FA5B                 mov     edi, eax\_B  
.text:0042FA5D                 shr     ebp, 1Bh  
.text:0042FA60                 shl     edi, 5  
.text:0042FA63                 or      edi, ebp  
.text:0042FA65                 add     esi, edi  
.text:0042FA67                 add     esi, 0CA62C1D6h  
.text:0042FA6D                 add     \[esp+14h+A\], esi  
.text:0042FA71                 mov     esi, edx\_C  
.text:0042FA73                 shr     edx\_C, 2  
.text:0042FA76                 shl     esi, 1Eh  
.text:0042FA79                 or      esi, edx\_C  
.text:0042FA7B                 mov     edx\_C, esi      ; R4(B,C,D,E,A,64)  
.text:0042FA7B                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.4 次运算 END\*\*\*\*  
.text:0042FA7D                 mov     esi, \[ebx+38h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.5 次运算开始\*\*\*\*  
.text:0042FA80                 xor     esi, \[ebx+24h\]  
.text:0042FA83                 xor     esi, \[ebx+0Ch\]  
.text:0042FA86                 xor     esi, \[ebx+4\]  
.text:0042FA89                 add     esi, esi  
.text:0042FA8B                 mov     edi, \[ebx+38h\]  
.text:0042FA8E                 xor     edi, \[ebx+24h\]  
.text:0042FA91                 xor     edi, \[ebx+0Ch\]  
.text:0042FA94                 xor     edi, \[ebx+4\]  
.text:0042FA97                 shr     edi, 1Fh  
.text:0042FA9A                 or      esi, edi  
.text:0042FA9C                 mov     edi, edx\_C  
.text:0042FA9E                 xor     edi, eax\_B  
.text:0042FAA0                 mov     \[ebx+4\], esi  
.text:0042FAA3                 xor     edi, ecx\_D  
.text:0042FAA5                 mov     ebp, \[esp+14h+A\]  
.text:0042FAA9                 add     esi, edi  
.text:0042FAAB                 mov     edi, \[esp+14h+A\]  
.text:0042FAAF                 shl     edi, 5  
.text:0042FAB2                 shr     ebp, 1Bh  
.text:0042FAB5                 or      edi, ebp  
.text:0042FAB7                 add     esi, edi  
.text:0042FAB9                 add     esi, 0CA62C1D6h  
.text:0042FABF                 add     \[esp+14h+E\], esi  
.text:0042FAC3                 mov     esi, eax\_B  
.text:0042FAC5                 shr     eax\_B, 2  
.text:0042FAC8                 mov     edi, \[ebx+3Ch\]  
.text:0042FACB                 shl     esi, 1Eh  
.text:0042FACE                 xor     edi, \[ebx+28h\]  
.text:0042FAD1                 or      esi, eax\_B  
.text:0042FAD3                 xor     edi, \[ebx+10h\]  
.text:0042FAD6                 mov     eax\_B, esi      ; R4(A,B,C,D,E,65)  
.text:0042FAD6                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.5 次运算 END\*\*\*\*  
.text:0042FAD8                 mov     esi, \[ebx+3Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.6 次运算开始\*\*\*\*  
.text:0042FADB                 xor     esi, \[ebx+28h\]  
.text:0042FADE                 xor     edi, \[ebx+8\]  
.text:0042FAE1                 xor     esi, \[ebx+10h\]  
.text:0042FAE4                 xor     esi, \[ebx+8\]  
.text:0042FAE7                 shr     edi, 1Fh  
.text:0042FAEA                 add     esi, esi  
.text:0042FAEC                 or      esi, edi  
.text:0042FAEE                 mov     \[ebx+8\], esi  
.text:0042FAF1                 mov     edi, \[esp+14h+A\]  
.text:0042FAF5                 mov     ebp, \[esp+14h+E\]  
.text:0042FAF9                 xor     edi, eax\_B  
.text:0042FAFB                 xor     edi, edx\_C  
.text:0042FAFD                 add     esi, edi  
.text:0042FAFF                 mov     edi, \[esp+14h+E\]  
.text:0042FB03                 shl     edi, 5  
.text:0042FB06                 shr     ebp, 1Bh  
.text:0042FB09                 or      edi, ebp  
.text:0042FB0B                 add     esi, edi  
.text:0042FB0D                 mov     edi, \[esp+14h+A\]  
.text:0042FB11                 add     esi, 0CA62C1D6h  
.text:0042FB17                 add     ecx\_D, esi  
.text:0042FB19                 mov     esi, \[esp+14h+A\]  
.text:0042FB1D                 shl     esi, 1Eh  
.text:0042FB20                 mov     ebp, ecx\_D  
.text:0042FB22                 shr     edi, 2  
.text:0042FB25                 or      esi, edi  
.text:0042FB27                 mov     \[esp+14h+A\], esi ; R4(E,A,B,C,D,66)  
.text:0042FB27                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.6 次运算 END\*\*\*\*  
.text:0042FB2B                 mov     edi, \[ebx\]      ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.7 次运算开始\*\*\*\*  
.text:0042FB2D                 mov     esi, \[ebx\]  
.text:0042FB2F                 xor     edi, \[ebx+2Ch\]  
.text:0042FB32                 xor     esi, \[ebx+2Ch\]  
.text:0042FB35                 xor     edi, \[ebx+14h\]  
.text:0042FB38                 xor     esi, \[ebx+14h\]  
.text:0042FB3B                 xor     edi, \[ebx+0Ch\]  
.text:0042FB3E                 xor     esi, \[ebx+0Ch\]  
.text:0042FB41                 shr     edi, 1Fh  
.text:0042FB44                 add     esi, esi  
.text:0042FB46                 or      esi, edi  
.text:0042FB48                 mov     \[ebx+0Ch\], esi  
.text:0042FB4B                 mov     edi, \[esp+14h+E\]  
.text:0042FB4F                 xor     edi, \[esp+14h+A\]  
.text:0042FB53                 xor     edi, eax\_B  
.text:0042FB55                 add     esi, edi  
.text:0042FB57                 mov     edi, ecx\_D  
.text:0042FB59                 shl     edi, 5  
.text:0042FB5C                 shr     ebp, 1Bh  
.text:0042FB5F                 or      edi, ebp  
.text:0042FB61                 add     esi, edi  
.text:0042FB63                 mov     edi, \[esp+14h+E\]  
.text:0042FB67                 add     esi, 0CA62C1D6h  
.text:0042FB6D                 add     edx\_C, esi  
.text:0042FB6F                 mov     esi, \[esp+14h+E\]  
.text:0042FB73                 shl     esi, 1Eh  
.text:0042FB76                 mov     ebp, edx\_C  
.text:0042FB78                 shr     edi, 2  
.text:0042FB7B                 or      esi, edi  
.text:0042FB7D                 mov     \[esp+14h+E\], esi ; R4(D,E,A,B,C,67)  
.text:0042FB7D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.7 次运算 END\*\*\*\*  
.text:0042FB81                 mov     edi, \[ebx+4\]    ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.8 次运算开始\*\*\*\*  
.text:0042FB84                 mov     esi, \[ebx+4\]  
.text:0042FB87                 xor     edi, \[ebx+30h\]  
.text:0042FB8A                 xor     esi, \[ebx+30h\]  
.text:0042FB8D                 xor     edi, \[ebx+18h\]  
.text:0042FB90                 xor     esi, \[ebx+18h\]  
.text:0042FB93                 xor     edi, \[ebx+10h\]  
.text:0042FB96                 xor     esi, \[ebx+10h\]  
.text:0042FB99                 shr     edi, 1Fh  
.text:0042FB9C                 add     esi, esi  
.text:0042FB9E                 or      esi, edi  
.text:0042FBA0                 mov     \[ebx+10h\], esi  
.text:0042FBA3                 mov     edi, \[esp+14h+E\]  
.text:0042FBA7                 xor     edi, ecx\_D  
.text:0042FBA9                 xor     edi, \[esp+14h+A\]  
.text:0042FBAD                 add     esi, edi  
.text:0042FBAF                 mov     edi, edx\_C  
.text:0042FBB1                 shl     edi, 5  
.text:0042FBB4                 shr     ebp, 1Bh  
.text:0042FBB7                 or      edi, ebp  
.text:0042FBB9                 add     esi, edi  
.text:0042FBBB                 mov     edi, \[ebx+8\]  
.text:0042FBBE                 add     esi, 0CA62C1D6h  
.text:0042FBC4                 xor     edi, \[ebx+34h\]  
.text:0042FBC7                 add     eax\_B, esi  
.text:0042FBC9                 mov     esi, ecx\_D  
.text:0042FBCB                 shr     ecx\_D, 2  
.text:0042FBCE                 mov     ebp, eax\_B  
.text:0042FBD0                 shl     esi, 1Eh  
.text:0042FBD3                 xor     edi, \[ebx+1Ch\]  
.text:0042FBD6                 or      esi, ecx\_D  
.text:0042FBD8                 xor     edi, \[ebx+14h\]  
.text:0042FBDB                 mov     ecx\_D, esi      ; R4(C,D,E,A,B,68)  
.text:0042FBDB                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.8 次运算 END\*\*\*\*  
.text:0042FBDD                 mov     esi, \[ebx+8\]    ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.9 次运算开始\*\*\*\*  
.text:0042FBE0                 xor     esi, \[ebx+34h\]  
.text:0042FBE3                 xor     esi, \[ebx+1Ch\]  
.text:0042FBE6                 xor     esi, \[ebx+14h\]  
.text:0042FBE9                 shr     edi, 1Fh  
.text:0042FBEC                 add     esi, esi  
.text:0042FBEE                 or      esi, edi  
.text:0042FBF0                 mov     edi, ecx\_D  
.text:0042FBF2                 mov     \[ebx+14h\], esi  
.text:0042FBF5                 xor     edi, edx\_C  
.text:0042FBF7                 xor     edi, \[esp+14h+E\]  
.text:0042FBFB                 add     esi, edi  
.text:0042FBFD                 mov     edi, eax\_B  
.text:0042FBFF                 shl     edi, 5  
.text:0042FC02                 shr     ebp, 1Bh  
.text:0042FC05                 or      edi, ebp  
.text:0042FC07                 add     esi, edi  
.text:0042FC09                 add     esi, 0CA62C1D6h  
.text:0042FC0F                 add     \[esp+14h+A\], esi  
.text:0042FC13                 mov     esi, edx\_C  
.text:0042FC15                 shr     edx\_C, 2  
.text:0042FC18                 mov     edi, \[ebx+0Ch\]  
.text:0042FC1B                 shl     esi, 1Eh  
.text:0042FC1E                 xor     edi, \[ebx+38h\]  
.text:0042FC21                 or      esi, edx\_C  
.text:0042FC23                 xor     edi, \[ebx+20h\]  
.text:0042FC26                 mov     edx\_C, esi      ; R4(B,C,D,E,A,69)  
.text:0042FC26                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.9 次运算 END\*\*\*\*  
.text:0042FC28                 mov     esi, \[ebx+0Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.10 次运算开始\*\*\*\*  
.text:0042FC2B                 xor     esi, \[ebx+38h\]  
.text:0042FC2E                 xor     edi, \[ebx+18h\]  
.text:0042FC31                 xor     esi, \[ebx+20h\]  
.text:0042FC34                 xor     esi, \[ebx+18h\]  
.text:0042FC37                 shr     edi, 1Fh  
.text:0042FC3A                 add     esi, esi  
.text:0042FC3C                 or      esi, edi  
.text:0042FC3E                 mov     edi, edx\_C  
.text:0042FC40                 xor     edi, eax\_B  
.text:0042FC42                 mov     \[ebx+18h\], esi  
.text:0042FC45                 xor     edi, ecx\_D  
.text:0042FC47                 mov     ebp, \[esp+14h+A\]  
.text:0042FC4B                 add     esi, edi  
.text:0042FC4D                 mov     edi, \[esp+14h+A\]  
.text:0042FC51                 shl     edi, 5  
.text:0042FC54                 shr     ebp, 1Bh  
.text:0042FC57                 or      edi, ebp  
.text:0042FC59                 add     esi, edi  
.text:0042FC5B                 add     esi, 0CA62C1D6h  
.text:0042FC61                 add     \[esp+14h+E\], esi  
.text:0042FC65                 mov     esi, eax\_B  
.text:0042FC67                 shr     eax\_B, 2  
.text:0042FC6A                 mov     edi, \[ebx+10h\]  
.text:0042FC6D                 shl     esi, 1Eh  
.text:0042FC70                 xor     edi, \[ebx+3Ch\]  
.text:0042FC73                 or      esi, eax\_B  
.text:0042FC75                 xor     edi, \[ebx+24h\]  
.text:0042FC78                 mov     eax\_B, esi      ; R4(A,B,C,D,E,70)  
.text:0042FC78                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.10 次运算 END\*\*\*\*  
.text:0042FC7A                 mov     esi, \[ebx+10h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.11 次运算开始\*\*\*\*  
.text:0042FC7D                 xor     esi, \[ebx+3Ch\]  
.text:0042FC80                 xor     edi, \[ebx+1Ch\]  
.text:0042FC83                 xor     esi, \[ebx+24h\]  
.text:0042FC86                 xor     esi, \[ebx+1Ch\]  
.text:0042FC89                 shr     edi, 1Fh  
.text:0042FC8C                 add     esi, esi  
.text:0042FC8E                 or      esi, edi  
.text:0042FC90                 mov     \[ebx+1Ch\], esi  
.text:0042FC93                 mov     edi, \[esp+14h+A\]  
.text:0042FC97                 mov     ebp, \[esp+14h+E\]  
.text:0042FC9B                 xor     edi, eax\_B  
.text:0042FC9D                 xor     edi, edx\_C  
.text:0042FC9F                 add     esi, edi  
.text:0042FCA1                 mov     edi, \[esp+14h+E\]  
.text:0042FCA5                 shl     edi, 5  
.text:0042FCA8                 shr     ebp, 1Bh  
.text:0042FCAB                 or      edi, ebp  
.text:0042FCAD                 add     esi, edi  
.text:0042FCAF                 mov     edi, \[esp+14h+A\]  
.text:0042FCB3                 add     esi, 0CA62C1D6h  
.text:0042FCB9                 add     ecx\_D, esi  
.text:0042FCBB                 mov     esi, \[esp+14h+A\]  
.text:0042FCBF                 shl     esi, 1Eh  
.text:0042FCC2                 mov     ebp, ecx\_D  
.text:0042FCC4                 shr     edi, 2  
.text:0042FCC7                 or      esi, edi  
.text:0042FCC9                 mov     \[esp+14h+A\], esi ; R4(E,A,B,C,D,71)  
.text:0042FCC9                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.11 次运算 END\*\*\*\*  
.text:0042FCCD                 mov     edi, \[ebx+14h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.12 次运算开始\*\*\*\*  
.text:0042FCD0                 mov     esi, \[ebx+14h\]  
.text:0042FCD3                 xor     edi, \[ebx\]  
.text:0042FCD5                 xor     esi, \[ebx\]  
.text:0042FCD7                 xor     edi, \[ebx+28h\]  
.text:0042FCDA                 xor     esi, \[ebx+28h\]  
.text:0042FCDD                 xor     edi, \[ebx+20h\]  
.text:0042FCE0                 xor     esi, \[ebx+20h\]  
.text:0042FCE3                 shr     edi, 1Fh  
.text:0042FCE6                 add     esi, esi  
.text:0042FCE8                 or      esi, edi  
.text:0042FCEA                 mov     \[ebx+20h\], esi  
.text:0042FCED                 mov     edi, \[esp+14h+E\]  
.text:0042FCF1                 xor     edi, \[esp+14h+A\]  
.text:0042FCF5                 xor     edi, eax\_B  
.text:0042FCF7                 add     esi, edi  
.text:0042FCF9                 mov     edi, ecx\_D  
.text:0042FCFB                 shl     edi, 5  
.text:0042FCFE                 shr     ebp, 1Bh  
.text:0042FD01                 or      edi, ebp  
.text:0042FD03                 add     esi, edi  
.text:0042FD05                 mov     edi, \[esp+14h+E\]  
.text:0042FD09                 add     esi, 0CA62C1D6h  
.text:0042FD0F                 add     edx\_C, esi  
.text:0042FD11                 mov     esi, \[esp+14h+E\]  
.text:0042FD15                 shl     esi, 1Eh  
.text:0042FD18                 mov     ebp, edx\_C  
.text:0042FD1A                 shr     edi, 2  
.text:0042FD1D                 or      esi, edi  
.text:0042FD1F                 mov     \[esp+14h+E\], esi ; R4(D,E,A,B,C,72)  
.text:0042FD1F                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.12 次运算 END\*\*\*\*  
.text:0042FD23                 mov     edi, \[ebx+18h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.13 次运算开始\*\*\*\*  
.text:0042FD26                 mov     esi, \[ebx+18h\]  
.text:0042FD29                 xor     edi, \[ebx+4\]  
.text:0042FD2C                 xor     esi, \[ebx+4\]  
.text:0042FD2F                 xor     edi, \[ebx+2Ch\]  
.text:0042FD32                 xor     esi, \[ebx+2Ch\]  
.text:0042FD35                 xor     edi, \[ebx+24h\]  
.text:0042FD38                 xor     esi, \[ebx+24h\]  
.text:0042FD3B                 shr     edi, 1Fh  
.text:0042FD3E                 add     esi, esi  
.text:0042FD40                 or      esi, edi  
.text:0042FD42                 mov     \[ebx+24h\], esi  
.text:0042FD45                 mov     edi, \[esp+14h+E\]  
.text:0042FD49                 xor     edi, ecx\_D  
.text:0042FD4B                 xor     edi, \[esp+14h+A\]  
.text:0042FD4F                 add     esi, edi  
.text:0042FD51                 mov     edi, edx\_C  
.text:0042FD53                 shr     ebp, 1Bh  
.text:0042FD56                 shl     edi, 5  
.text:0042FD59                 or      edi, ebp  
.text:0042FD5B                 add     esi, edi  
.text:0042FD5D                 add     esi, 0CA62C1D6h  
.text:0042FD63                 add     eax\_B, esi  
.text:0042FD65                 mov     esi, ecx\_D  
.text:0042FD67                 mov     edi, \[ebx+1Ch\]  
.text:0042FD6A                 shr     ecx\_D, 2  
.text:0042FD6D                 mov     ebp, eax\_B  
.text:0042FD6F                 shl     esi, 1Eh  
.text:0042FD72                 xor     edi, \[ebx+8\]  
.text:0042FD75                 or      esi, ecx\_D  
.text:0042FD77                 xor     edi, \[ebx+30h\]  
.text:0042FD7A                 mov     ecx\_D, esi      ; R4(C,D,E,A,B,73)  
.text:0042FD7A                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.13 次运算 END\*\*\*\*  
.text:0042FD7C                 mov     esi, \[ebx+1Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.14 次运算开始\*\*\*\*  
.text:0042FD7F                 xor     esi, \[ebx+8\]  
.text:0042FD82                 xor     edi, \[ebx+28h\]  
.text:0042FD85                 xor     esi, \[ebx+30h\]  
.text:0042FD88                 xor     esi, \[ebx+28h\]  
.text:0042FD8B                 shr     edi, 1Fh  
.text:0042FD8E                 add     esi, esi  
.text:0042FD90                 or      esi, edi  
.text:0042FD92                 mov     edi, ecx\_D  
.text:0042FD94                 mov     \[ebx+28h\], esi  
.text:0042FD97                 xor     edi, edx\_C  
.text:0042FD99                 xor     edi, \[esp+14h+E\]  
.text:0042FD9D                 add     esi, edi  
.text:0042FD9F                 mov     edi, eax\_B  
.text:0042FDA1                 shl     edi, 5  
.text:0042FDA4                 shr     ebp, 1Bh  
.text:0042FDA7                 or      edi, ebp  
.text:0042FDA9                 add     esi, edi  
.text:0042FDAB                 add     esi, 0CA62C1D6h  
.text:0042FDB1                 add     \[esp+14h+A\], esi  
.text:0042FDB5                 mov     esi, edx\_C  
.text:0042FDB7                 shr     edx\_C, 2  
.text:0042FDBA                 mov     edi, \[ebx+20h\]  
.text:0042FDBD                 shl     esi, 1Eh  
.text:0042FDC0                 xor     edi, \[ebx+0Ch\]  
.text:0042FDC3                 or      esi, edx\_C  
.text:0042FDC5                 xor     edi, \[ebx+34h\]  
.text:0042FDC8                 mov     edx\_C, esi      ; R4(B,C,D,E,A,74)  
.text:0042FDC8                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.14 次运算 END\*\*\*\*  
.text:0042FDCA                 mov     esi, \[ebx+20h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.15 次运算开始\*\*\*\*  
.text:0042FDCD                 xor     esi, \[ebx+0Ch\]  
.text:0042FDD0                 xor     edi, \[ebx+2Ch\]  
.text:0042FDD3                 xor     esi, \[ebx+34h\]  
.text:0042FDD6                 xor     esi, \[ebx+2Ch\]  
.text:0042FDD9                 shr     edi, 1Fh  
.text:0042FDDC                 add     esi, esi  
.text:0042FDDE                 or      esi, edi  
.text:0042FDE0                 mov     edi, edx\_C  
.text:0042FDE2                 xor     edi, eax\_B  
.text:0042FDE4                 mov     \[ebx+2Ch\], esi  
.text:0042FDE7                 xor     edi, ecx\_D  
.text:0042FDE9                 mov     ebp, \[esp+14h+A\]  
.text:0042FDED                 add     esi, edi  
.text:0042FDEF                 mov     edi, \[esp+14h+A\]  
.text:0042FDF3                 shl     edi, 5  
.text:0042FDF6                 shr     ebp, 1Bh  
.text:0042FDF9                 or      edi, ebp  
.text:0042FDFB                 add     esi, edi  
.text:0042FDFD                 add     esi, 0CA62C1D6h  
.text:0042FE03                 add     \[esp+14h+E\], esi  
.text:0042FE07                 mov     esi, eax\_B  
.text:0042FE09                 shr     eax\_B, 2  
.text:0042FE0C                 mov     edi, \[ebx+24h\]  
.text:0042FE0F                 shl     esi, 1Eh  
.text:0042FE12                 xor     edi, \[ebx+10h\]  
.text:0042FE15                 or      esi, eax\_B  
.text:0042FE17                 xor     edi, \[ebx+38h\]  
.text:0042FE1A                 mov     eax\_B, esi      ; R4(A,B,C,D,E,75)  
.text:0042FE1A                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.15 次运算 END\*\*\*\*  
.text:0042FE1C                 mov     esi, \[ebx+24h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.16 次运算开始\*\*\*\*  
.text:0042FE1F                 xor     esi, \[ebx+10h\]  
.text:0042FE22                 xor     edi, \[ebx+30h\]  
.text:0042FE25                 xor     esi, \[ebx+38h\]  
.text:0042FE28                 xor     esi, \[ebx+30h\]  
.text:0042FE2B                 shr     edi, 1Fh  
.text:0042FE2E                 add     esi, esi  
.text:0042FE30                 or      esi, edi  
.text:0042FE32                 mov     \[ebx+30h\], esi  
.text:0042FE35                 mov     edi, \[esp+14h+A\]  
.text:0042FE39                 mov     ebp, \[esp+14h+E\]  
.text:0042FE3D                 xor     edi, eax\_B  
.text:0042FE3F                 xor     edi, edx\_C  
.text:0042FE41                 add     esi, edi  
.text:0042FE43                 mov     edi, \[esp+14h+E\]  
.text:0042FE47                 shl     edi, 5  
.text:0042FE4A                 shr     ebp, 1Bh  
.text:0042FE4D                 or      edi, ebp  
.text:0042FE4F                 add     esi, edi  
.text:0042FE51                 mov     edi, \[esp+14h+A\]  
.text:0042FE55                 add     esi, 0CA62C1D6h  
.text:0042FE5B                 add     ecx\_D, esi  
.text:0042FE5D                 mov     esi, \[esp+14h+A\]  
.text:0042FE61                 shl     esi, 1Eh  
.text:0042FE64                 mov     ebp, ecx\_D  
.text:0042FE66                 shr     edi, 2  
.text:0042FE69                 or      esi, edi  
.text:0042FE6B                 mov     \[esp+14h+A\], esi ; R4(E,A,B,C,D,76)  
.text:0042FE6B                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.16 次运算 END\*\*\*\*  
.text:0042FE6F                 mov     edi, \[ebx+28h\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.17 次运算开始\*\*\*\*  
.text:0042FE72                 mov     esi, \[ebx+28h\]  
.text:0042FE75                 xor     edi, \[ebx+14h\]  
.text:0042FE78                 xor     esi, \[ebx+14h\]  
.text:0042FE7B                 xor     edi, \[ebx+3Ch\]  
.text:0042FE7E                 xor     esi, \[ebx+3Ch\]  
.text:0042FE81                 xor     edi, \[ebx+34h\]  
.text:0042FE84                 xor     esi, \[ebx+34h\]  
.text:0042FE87                 shr     edi, 1Fh  
.text:0042FE8A                 add     esi, esi  
.text:0042FE8C                 or      esi, edi  
.text:0042FE8E                 mov     \[ebx+34h\], esi  
.text:0042FE91                 mov     edi, \[esp+14h+E\]  
.text:0042FE95                 xor     edi, \[esp+14h+A\]  
.text:0042FE99                 xor     edi, eax\_B  
.text:0042FE9B                 add     esi, edi  
.text:0042FE9D                 mov     edi, ecx\_D  
.text:0042FE9F                 shl     edi, 5  
.text:0042FEA2                 shr     ebp, 1Bh  
.text:0042FEA5                 or      edi, ebp  
.text:0042FEA7                 add     esi, edi  
.text:0042FEA9                 mov     edi, \[esp+14h+E\]  
.text:0042FEAD                 add     esi, 0CA62C1D6h  
.text:0042FEB3                 add     edx\_C, esi  
.text:0042FEB5                 mov     esi, \[esp+14h+E\]  
.text:0042FEB9                 shl     esi, 1Eh  
.text:0042FEBC                 mov     ebp, edx\_C  
.text:0042FEBE                 shr     edi, 2  
.text:0042FEC1                 or      esi, edi  
.text:0042FEC3                 mov     \[esp+14h+E\], esi ; R4(D,E,A,B,C,77)  
.text:0042FEC3                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.17 次运算 END\*\*\*\*  
.text:0042FEC7                 mov     edi, \[ebx+2Ch\]  ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.18 次运算开始\*\*\*\*  
.text:0042FECA                 mov     esi, \[ebx+2Ch\]  
.text:0042FECD                 xor     edi, \[ebx+18h\]  
.text:0042FED0                 xor     esi, \[ebx+18h\]  
.text:0042FED3                 xor     edi, \[ebx\]  
.text:0042FED5                 xor     esi, \[ebx\]  
.text:0042FED7                 xor     edi, \[ebx+38h\]  
.text:0042FEDA                 xor     esi, \[ebx+38h\]  
.text:0042FEDD                 shr     edi, 1Fh  
.text:0042FEE0                 add     esi, esi  
.text:0042FEE2                 or      esi, edi  
.text:0042FEE4                 mov     \[ebx+38h\], esi  
.text:0042FEE7                 mov     edi, \[esp+14h+E\]  
.text:0042FEEB                 xor     edi, ecx\_D  
.text:0042FEED                 xor     edi, \[esp+14h+A\]  
.text:0042FEF1                 add     esi, edi  
.text:0042FEF3                 mov     edi, edx\_C  
.text:0042FEF5                 shr     ebp, 1Bh  
.text:0042FEF8                 shl     edi, 5  
.text:0042FEFB                 or      edi, ebp  
.text:0042FEFD                 add     esi, edi  
.text:0042FEFF                 mov     edi, \[ebx+30h\]  
.text:0042FF02                 add     esi, 0CA62C1D6h  
.text:0042FF08                 xor     edi, \[ebx+1Ch\]  
.text:0042FF0B                 add     eax\_B, esi  
.text:0042FF0D                 mov     esi, ecx\_D  
.text:0042FF0F                 shr     ecx\_D, 2  
.text:0042FF12                 xor     edi, \[ebx+4\]  
.text:0042FF15                 shl     esi, 1Eh  
.text:0042FF18                 xor     edi, \[ebx+3Ch\]  
.text:0042FF1B                 or      esi, ecx\_D  
.text:0042FF1D                 mov     ecx\_D, esi      ; R4(C,D,E,A,B,78)  
.text:0042FF1D                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.18 次运算 END\*\*\*\*  
.text:0042FF1F                 mov     esi, \[ebx+UI32.\_12\] ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.19 次运算开始\*\*\*\*  
.text:0042FF22                 xor     esi, \[ebx+UI32.\_7\]  
.text:0042FF25                 xor     esi, \[ebx+UI32.\_1\]  
.text:0042FF28                 xor     esi, \[ebx+UI32.\_15\]  
.text:0042FF2B                 shr     edi, 1Fh  
.text:0042FF2E                 add     esi, esi  
.text:0042FF30                 or      esi, edi  
.text:0042FF32                 mov     edi, eax\_B  
.text:0042FF34                 mov     \[ebx+UI32.\_15\], esi  
.text:0042FF37                 mov     ebx, ecx\_D  
.text:0042FF39                 xor     ebx, edx\_C  
.text:0042FF3B                 xor     ebx, \[esp+14h+E\]  
.text:0042FF3F                 add     esi, ebx  
.text:0042FF41                 mov     ebx, eax\_B  
.text:0042FF43                 shr     edi, 1Bh  
.text:0042FF46                 shl     ebx, 5  
.text:0042FF49                 or      ebx, edi  
.text:0042FF4B                 add     esi, ebx  
.text:0042FF4D                 mov     ebx, edx\_C  
.text:0042FF4F                 shr     edx\_C, 2  
.text:0042FF52                 add     esi, 0CA62C1D6h  
.text:0042FF58                 shl     ebx, 1Eh  
.text:0042FF5B                 add     \[esp+14h+A\], esi  
.text:0042FF5F                 or      ebx, edx\_C  
.text:0042FF61                 mov     esi, \[esp+14h+A\]  
.text:0042FF65                 mov     edx\_C, ebx      ; R4(B,C,D,E,A,79)  
.text:0042FF65                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4.19 次运算 END\*\*\*\*  
.text:0042FF65                                         ; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*第 4 轮计算结束\*\*\*\*  
.text:0042FF67                 mov     ebx, \[esp+14h+\_state\]  
.text:0042FF6A                 add     \[ebx\], esi      ; state\[0\] += A;  
.text:0042FF6C                 mov     ebx, \[esp+14h+\_state\]  
.text:0042FF6F                 add     \[ebx+4\], eax\_B  ; state\[1\] += B;  
.text:0042FF72                 mov     eax\_B, \[esp+14h+\_state\]  
.text:0042FF75                 add     \[eax\_B+8\], edx\_C ; state\[2\] += C;  
.text:0042FF78                 mov     edx\_C, \[esp+14h+\_state\]  
.text:0042FF7B                 add     \[edx\_C+0Ch\], ecx\_D ; state\[3\] += D;  
.text:0042FF7E                 mov     ecx\_D, \[esp+14h+\_state\]  
.text:0042FF81                 mov     eax\_B, \[esp+14h+E\]  
.text:0042FF85                 add     \[ecx\_D+10h\], eax\_B ; state\[4\] += E;  
.text:0042FF88                 xor     edx\_C, edx\_C  
.text:0042FF8A                 mov     \[esp+14h+E\], edx\_C  
.text:0042FF8E                 mov     \[esp+14h+A\], edx\_C  
.text:0042FF92                 push    4               ; n  
.text:0042FF94                 push    0               ; c  
.text:0042FF96                 mov     eax\_B, edx\_C  
.text:0042FF98                 mov     ecx\_D, edx\_C  
.text:0042FF9A                 lea     eax\_B, \[esp+1Ch+A\]  
.text:0042FF9E                 push    eax\_B           ; s  
.text:0042FF9F                 call    \_memset  
.text:0042FF9F  
.text:0042FFA4                 add     esp, 0Ch  
.text:0042FFA7                 add     esp, 14h  
.text:0042FFAA                 pop     ebp  
.text:0042FFAB                 pop     edi  
.text:0042FFAC                 pop     esi  
.text:0042FFAD                 pop     ebx  
.text:0042FFAE                 retn  
.text:0042FFAE  
.text:0042FFAE sha1\_transform  endp  
  
  
sha1\_transform C 实现:  
  
u\_int8\_t g\_buffer64\[64\];  
  
void \_\_fastcall   
sha1\_transform(u\_int32\_t state\[5\], u\_int8\_t \*data, u\_int8\_t trans\_copies )  
{  
        u\_int32\_t \*buf;  
  
        u\_int32\_t A,B,C,D,E;  
  
        if (  trans\_copies != TRANSFORM\_RAW )                /\* 判断是否在一个拷贝上变换 TRANSFORM\_RAW=0 \*/  
        {  
                buf = (u\_int32\_t\*)g\_buffer64;  
                memcpy(buf, data, 64);  
        }  
        else  
                buf  = (u\_int32\_t\*)data;  
  
        A = state\[0\];  
        B = state\[1\];  
        C = state\[2\];  
        D = state\[3\];  
        E = state\[4\];  
  
        /\* 第 1 轮变换 \*/  
        R0 (A, B, C, D, E, 0);  R0 (E, A, B, C, D, 1);  R0 (D, E, A, B, C, 2);  R0 (C, D, E, A, B, 3);  
        R0 (B, C, D, E, A, 4);  R0 (A, B, C, D, E, 5);  R0 (E, A, B, C, D, 6);  R0 (D, E, A, B, C, 7);  
        R0 (C, D, E, A, B, 8);  R0 (B, C, D, E, A, 9);  R0 (A, B, C, D, E, 10); R0 (E, A, B, C, D, 11);  
        R0 (D, E, A, B, C, 12); R0 (C, D, E, A, B, 13); R0 (B, C, D, E, A, 14); R0 (A, B, C, D, E, 15);  
        R1 (E, A, B, C, D, 16); R1 (D, E, A, B, C, 17); R1 (C, D, E, A, B, 18); R1 (B, C, D, E, A, 19);  
  
        /\* 第 2 轮变换 \*/  
        R2 (A, B, C, D, E, 20); R2 (E, A, B, C, D, 21); R2 (D, E, A, B, C, 22); R2 (C, D, E, A, B, 23);  
        R2 (B, C, D, E, A, 24); R2 (A, B, C, D, E, 25); R2 (E, A, B, C, D, 26); R2 (D, E, A, B, C, 27);  
        R2 (C, D, E, A, B, 28); R2 (B, C, D, E, A, 29); R2 (A, B, C, D, E, 30); R2 (E, A, B, C, D, 31);  
        R2 (D, E, A, B, C, 32); R2 (C, D, E, A, B, 33); R2 (B, C, D, E, A, 34); R2 (A, B, C, D, E, 35);  
        R2 (E, A, B, C, D, 36); R2 (D, E, A, B, C, 37); R2 (C, D, E, A, B, 38); R2 (B, C, D, E, A, 39);  
  
        /\* 第 3 轮变换 \*/  
        R3 (A, B, C, D, E, 40); R3 (E, A, B, C, D, 41); R3 (D, E, A, B, C, 42); R3 (C, D, E, A, B, 43);  
        R3 (B, C, D, E, A, 44); R3 (A, B, C, D, E, 45); R3 (E, A, B, C, D, 46); R3 (D, E, A, B, C, 47);  
        R3 (C, D, E, A, B, 48); R3 (B, C, D, E, A, 49); R3 (A, B, C, D, E, 50); R3 (E, A, B, C, D, 51);  
        R3 (D, E, A, B, C, 52); R3 (C, D, E, A, B, 53); R3 (B, C, D, E, A, 54); R3 (A, B, C, D, E, 55);  
        R3 (E, A, B, C, D, 56); R3 (D, E, A, B, C, 57); R3 (C, D, E, A, B, 58); R3 (B, C, D, E, A, 59);  
  
        /\* 第 4 轮变换 \*/  
        R4 (A, B, C, D, E, 60); R4 (E, A, B, C, D, 61); R4 (D, E, A, B, C, 62); R4 (C, D, E, A, B, 63);  
        R4 (B, C, D, E, A, 64); R4 (A, B, C, D, E, 65); R4 (E, A, B, C, D, 66); R4 (D, E, A, B, C, 67);  
        R4 (C, D, E, A, B, 68); R4 (B, C, D, E, A, 69); R4 (A, B, C, D, E, 70); R4 (E, A, B, C, D, 71);  
        R4 (D, E, A, B, C, 72); R4 (C, D, E, A, B, 73); R4 (B, C, D, E, A, 74); R4 (A, B, C, D, E, 75);  
        R4 (E, A, B, C, D, 76); R4 (D, E, A, B, C, 77); R4 (C, D, E, A, B, 78); R4 (B, C, D, E, A, 79);  
  
        state\[0\] += A;  
        state\[1\] += B;  
        state\[2\] += C;  
        state\[3\] += D;  
        state\[4\] += E;  
        A=B=C=D=E=0;   
  
        memset((void\*)&A, 0, 4); /\* ???? \*/  
}  
  
最后一个函数 sha1\_final:  
  
.text:00430080 ; int \_\_fastcall sha1\_final(SHA1\_CTX \*ctx,BYTE \*digest,BYTE trans\_copies)  
.text:00430080 sha1\_final      proc near               ; CODE XREF: sub\_407E6C+2Dp  
.text:00430080  
.text:00430080 \_trans\_copies   = dword ptr -24h  
.text:00430080 i               = dword ptr -20h  
.text:00430080 var\_1C          = dword ptr -1Ch  
.text:00430080 bigcount        = byte ptr -18h  
.text:00430080 end             = dword ptr -10h  
.text:00430080 s               = dword ptr -0Ch  
.text:00430080 data            = byte ptr -4  
.text:00430080  
.text:00430080 \_ctx = ebx  
.text:00430080 \_digest = edi  
.text:00430080                 push    \_ctx  
.text:00430081                 push    esi             ; len  
.text:00430082                 push    \_digest         ; data  
.text:00430083                 add     esp, 0FFFFFFE8h ; len  
.text:00430086                 mov     \_ctx, eax  
.text:00430088                 xor     eax, eax  
.text:0043008A                 lea     esi, \[esp+24h+i\]  
.text:0043008E                 mov     byte ptr \[esp+24h+\_trans\_copies\], cl  
.text:00430091                 mov     \_digest, edx  
.text:00430093                 mov     \[esi\], eax  
.text:00430095  
.text:00430095 NEXT\_ENDIAN:                            ; CODE XREF: sha1\_final+40j  
.text:00430095                 mov     edx, \[esi\]      ; i 循环计数  
.text:00430097                 and     edx, 3          ; i&3  
.text:0043009A                 mov     ecx, 3  
.text:0043009F                 sub     ecx, edx        ; 3 - (i&3)  
.text:004300A1                 shl     ecx, 3          ; (3-(i&3))\*8 或者 (3-(i&3))<<3  
.text:004300A4                 xor     eax, eax  
.text:004300A6                 cmp     dword ptr \[esi\], 4 ; if (i<4) eax=1;  
.text:004300A6                                         ; else eax = 0;  
.text:004300A9                 jnb     short TO\_BIG\_ENDIAN  
.text:004300A9  
.text:004300AB                 inc     eax  
.text:004300AB  
.text:004300AC  
.text:004300AC TO\_BIG\_ENDIAN:                          ; CODE XREF: sha1\_final+29j  
.text:004300AC                 mov     edx, \[\_ctx+eax\*4+SHA1\_CTX.count0\]  
.text:004300B0                 shr     edx, cl         ; ctx->count\[eax\] >> ((3-(i&3))\*8)  
.text:004300B2                 and     dl, 0FFh        ; (ctx->count\[eax\] >> ((3-(i&3))\*8)) & 0x0ff;  
.text:004300B5                 mov     ecx, \[esi\]      ; i  
.text:004300B7                 mov     byte ptr \[esp+ecx+24h+bigcount\], dl  
.text:004300BB                 inc     dword ptr \[esi\] ; i++  
.text:004300BD                 cmp     dword ptr \[esi\], 8 ; i<8  
.text:004300C0                 jb      short NEXT\_ENDIAN ; 处理字节序, 把 ctx 中的 count\[0\],count\1\ 数据 bit 数  
.text:004300C0                                         ; 转化为 BIG\_ENDIAN 存储到 bigcount\[8\]  
.text:004300C0  
.text:004300C2                 mov     byte ptr \[esp+24h+end\], 80h  
.text:004300C7                 mov     al, byte ptr \[esp+24h+\_trans\_copies\]  
.text:004300CA                 mov     ecx, 1          ; len  
.text:004300CF                 push    eax             ; trans\_copies  
.text:004300D0                 mov     eax, \_ctx       ; ctx  
.text:004300D2                 lea     edx, \[esp+28h+end\] ; unsigned int  
.text:004300D6                 call    sha1\_update     ; sha1\_update 一个 bit 数据 0x80, 这也许是表示数据的结束?  
.text:004300D6  
.text:004300DB                 jmp     short FOR\_condition  
.text:004300DD ; ---------------------------------------------------------------------------  
.text:004300DD FOR:                                    ; CODE XREF: sha1\_final+83j  
.text:004300DD                 mov     byte ptr \[esp+14h\], 0 ; end  
.text:004300E2                 mov     dl, \[esp\]       ; \_trans\_copies  
.text:004300E5                 mov     ecx, 1          ; data  
.text:004300EA                 push    edx             ; trans\_copies  
.text:004300EB                 lea     edx, \[esp+18h\]  ; end  
.text:004300EF                 mov     eax, \_ctx       ; ctx  
.text:004300F1                 call    sha1\_update  
.text:004300F1  
.text:004300F6  
.text:004300F6 FOR\_condition:                          ; CODE XREF: sha1\_final+5Bj  
.text:004300F6                 mov     eax, \[\_ctx+SHA1\_CTX.count0\]  
.text:004300F9                 and     eax, 1F8h       ; 0x1F8 = 504 = 63\*8  
.text:004300FE                 cmp     eax, 1C0h       ; 0x1c0 = 488 = 56\*8  
.text:00430103                 jnz     short FOR       ; 判断剩余的 bit 数是不是等于 488(56\*8, 字节数是不是等  
.text:00430103                                         ; 于 56), 不等于添加'\\0', 直到等于 56, 剩余 8 个字节添加  
.text:00430103                                         ; bigcount\[8\], 凑够 64 字节, 计算.  
.text:00430103  
.text:00430105                 mov     dl, \[esp\]       ; \_trans\_copies  
.text:00430108                 mov     ecx, 8          ; data  
.text:0043010D                 push    edx             ; trans\_copies  
.text:0043010E                 lea     edx, \[esp+10h\]  ; bigcount  
.text:00430112                 mov     eax, \_ctx       ; ctx  
.text:00430114                 call    sha1\_update     ; 把 bigcount\[8\]添加到 ctx->buffer, 计算最后 64 字节,  
.text:00430114                                         ; 整个计算过程结束  
.text:00430114  
.text:00430119                 xor     eax, eax  
.text:0043011B                 mov     \[esi\], eax      ; i = 0  
.text:0043011B  
.text:0043011D  
.text:0043011D OUT\_DIGEST:                             ; CODE XREF: sha1\_final+AFj  
.text:0043011D                 mov     edx, \[esi\]  
.text:0043011F                 mov     ecx, \[esi\]  
.text:00430121                 mov     eax, \[\_ctx+ecx\*4+SHA1\_CTX.state\]  
.text:00430124                 and     eax, 0FFFFFFFFh  
.text:00430127                 mov     \[\_digest+edx\*4\], eax  
.text:0043012A                 inc     dword ptr \[esi\] ; i++  
.text:0043012C                 cmp     dword ptr \[esi\], 5  
.text:0043012F                 jb      short OUT\_DIGEST ; 输出 sha1 摘要的时候是按 DWORD 输出, LITTLE\_ENDIA 字节徐  
.text:0043012F  
.text:00430131                 push    4               ; n  
.text:00430133                 push    0               ; c  
.text:00430135                 push    esi  
.text:00430136                 call    \_memset         ; i 清零  
.text:00430136  
.text:0043013B                 add     esp, 0Ch  
.text:0043013E                 push    4               ; n  
.text:00430140                 push    0               ; c  
.text:00430142                 lea     edx, \[esp+10h\]  
.text:00430146                 push    edx             ; s  
.text:00430147                 call    \_memset         ; var\_1C 清零  
.text:00430147  
.text:0043014C                 add     esp, 0Ch  
.text:0043014F                 lea     esi, \[\_ctx+SHA1\_CTX.buffer\]  
.text:00430152                 push    40h             ; n  
.text:00430154                 push    0               ; c  
.text:00430156                 push    esi             ; s  
.text:00430157                 call    \_memset         ; ctx->buffer 清零  
.text:00430157  
.text:0043015C                 add     esp, 0Ch  
.text:0043015F                 push    14h             ; n  
.text:00430161                 push    0               ; c  
.text:00430163                 push    \_ctx            ; s  
.text:00430164                 call    \_memset         ; ctx->state 清零  
.text:00430164  
.text:00430169                 add     esp, 0Ch  
.text:0043016C                 lea     eax, \[\_ctx+SHA1\_CTX.count0\]  
.text:0043016F                 push    8               ; n  
.text:00430171                 push    0               ; c  
.text:00430173                 push    eax             ; s  
.text:00430174                 call    \_memset         ; ctx->count\[0\], ctx->count\[1\]清零  
.text:00430174  
.text:00430179                 add     esp, 0Ch  
.text:0043017C                 push    8               ; n  
.text:0043017E                 push    0               ; c  
.text:00430180                 lea     edx, \[esp+20h+s\]  
.text:00430184                 push    edx             ; s  
.text:00430185                 call    \_memset         ; bigcount 清零  
.text:00430185  
.text:0043018A                 add     esp, 0Ch  
.text:0043018D                 cmp     byte ptr \[esp\], 0 ; \_trans\_copies  
.text:00430191                 jz      short loc\_43019E  
.text:00430191  
.text:00430193                 mov     edx, esi        ; data  
.text:00430195                 mov     cl, 1           ; trans\_copies  
.text:00430197                 mov     eax, \_ctx       ; ctx\_state  
.text:00430199                 call    sha1\_transform  ;/\* ctx 中产生一个坏的 SHA1 摘要信息 \*/  
.text:00430199  
.text:0043019E loc\_43019E:                             ; CODE XREF: sha1\_final+111j  
.text:0043019E                 add     esp, 18h  
.text:004301A1                 pop     \_digest  
.text:004301A2                 pop     esi  
.text:004301A3                 pop     \_ctx  
.text:004301A4                 retn  
.text:004301A4 sha1\_final      endp ; sp =  0Ch  
  
  
C 实现:  
  
void \_\_fastcall  
sha1\_final(SHA1\_CTX \*ctx, u\_int32\_t \*digest, u\_int8\_t trans\_copies)   
{  
        int i;  
        int var\_1C;                /\* ???? 没用到的变量 \*/  
        u\_int8\_t bigcount\[8\];  
        u\_int8\_t end;  
  
        for (i = 0; i < 8; i++)        /\* 字节序转换为 BIG\_ENDIA \*/  
        {  
                bigcount\[i\] = (u\_int8\_t) ((ctx->count\[(i >= 4 ? 0 : 1)\] >> ((3 - (i & 3)) \* 8)) & 255);   
        }  
        end = 0x80;  
        sha1\_update(ctx,  &end, 1, trans\_copies);  
        for (; (ctx->count\[0\] & 504) != 448; )  
        {  
                end = 0;  
                sha1\_update(ctx, &end, 1, trans\_copies);  
        }  
        sha1\_update(ctx, bigcount, 8, trans\_copies);  
        for(i=0; i<5; i++)        /\* 计算结束,输出 SHA1 摘要 \*/  
                \*(digest+i) = ctx->state\[i\];   
  
  
        memset(&i, 0, sizeof(i));  
        memset(&var\_1C, 0, sizeof(i));  
        memset(ctx->buffer, 0, 64);  /\* 销毁 buffer 中存储的数据 \*/  
        memset(ctx->state, 0, 4\*5);  /\* 销毁 ctx 中出现的摘要 \*/  
        memset(ctx->count, 0, 4\*2);  
  
        if (trans\_copies != 0)      /\* ctx 中产生一个坏的 SHA1 摘要信息 \*/  
                sha1\_transform(ctx->state, ctx->buffer, TRANSFORM\_COPIES);  
}