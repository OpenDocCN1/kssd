
```asm
.text:004107F9                   jmp     short loc_410811  

.text:004107FB ; ---------------------------------------------------------------------------  

.text:004107FB loc_4107FB:                               ; CODE XREF: NCrc32+DFj  

.text:004107FB                   mov     ecx, ebx        ; 处理剩余部分  

.text:004107FD                   xor     cl, [eax+edx]  

.text:00410800                   and     ecx, 0FFh  

.text:00410806                   shr     ebx, 8  

.text:00410809                   mov     ecx, [edi+ecx*4]  

.text:0041080C                   xor     ecx, ebx  

.text:0041080E                   mov     ebx, ecx  

.text:00410810                   inc     edx  

.text:00410811  

.text:00410811 loc_410811:                               ; CODE XREF: NCrc32+C5j  

.text:00410811                   cmp     esi, edx  

.text:00410813                   ja      short loc_4107FB ; 处理剩余部分  

.text:00410815                   mov     eax, ebx  

.text:00410817                   pop     ebp  

.text:00410818                   pop     edi  

.text:00410819                   pop     esi  

.text:0041081A                   pop     ebx  

.text:0041081B                   retn  

.text:0041081B  

.text:0041081B NCrc32            endp  
```

用 C 简单实现了一下:

```c
unsigned int  

NCrc32 (unsigned int crc32, unsigned char *buff, unsigned int len)  

{  

        if (crc32Table[1] == 0)  

                makeCrc32Table();  

        for ( ; (len>0) && (((unsigned int)buff & 7)!=0); --len) // 8 字节对齐  

        {  

                crc32 = crc32Table[(crc32 ^ *buff++) & 0xFF] ^ (crc32>>8);  

        }  

        // 每次计算 8 个字节  

        while (len >8)  

        {  

                crc32 ^= *(unsigned int*)buff;  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 ^= *(unsigned int*)(buff+4);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                crc32 = crc32Table[crc32 & 0xFF] ^(crc32>>8);  

                len -= 8;  

                buff += 8;  

        }  

        // 计算剩余部分  

        for (unsigned int i=0; len>i; i++)  

        {  

                crc32 = crc32Table[(crc32 ^ buff[i]) & 0xFF] ^ (crc32>>8);  

        }  

        return crc32;  

}  
```

下面的代码用到一个结构 `REG_DATA`:

```c
typedef struct _reg_data  

{  

        char usrname[256];  

        char licType[256];  

        char snCode1[256];  

        char snCode4[256];  

        char snCode3[256];  

        char snCode2[256];  

        u_int32_t crc32;  

}REG_DATA;  
```

`Proxies::CreateSubClass` 不知道做什么的, 大概是构造子串的, 既然是库函数直接看结果了,  

结果在 `edx` 指向的内存产生一个字符串, 每次都是 `rarreg.key` 文件的一行(在这之前文件的内  

容已经读到内存了, 读文件部分被略了:)).

```asm
.text:0040E6F8 HCODE0_CRC_OK:  

.text:0040E6F8                   push    604h            ; len  

.text:0040E6FD                   push    0               ; c  

.text:0040E6FF                   push    regData         ; REG_DATA  

.text:0040E700                   call    _memset                 ; 结构清零  

.text:0040E700  

.text:0040E705                   add     esp, 0Ch  

.text:0040E708                   jmp     short loc_40E715  

.text:0040E70A ; ---------------------------------------------------------------------------  

.text:0040E70A loc_40E70A:                               ; CODE XREF: check_regdata+63j  

.text:0040E70A                   xor     edx, edx  

.text:0040E70C                   mov     dl, byte ptr [esp+0E88h+strTemp]  

.text:0040E710                   cmp     edx, 23h  

.text:0040E713                   jnz     short loc_40E729  

.text:0040E715 loc_40E715:                               ; CODE XREF: check_regdata+44j  

.text:0040E715                   lea     edx, [esp+0E88h+strTemp]  

.text:0040E719                   mov     ecx, 400h  

.text:0040E71E                   mov     eax, edi      ;  

.text:0040E720                   call    Proxies::CreateSubClass(System::TObject *,System::AnsiString,System::TMetaClass *)  

.text:0040E725                   test    al, al  

.text:0040E727                   jnz     short loc_40E70A  

.text:0040E729  

.text:0040E729 loc_40E729:                               ; CODE XREF: check_regdata+4Fj  

.text:0040E729                   mov     edx, regData  

.text:0040E72B                   mov     ecx, 100h  

.text:0040E730                   mov     eax, edi  

.text:0040E732                   call    Proxies::CreateSubClass(System::TObject *,System::AnsiString,System::TMetaClass *)  

.text:0040E737                   lea     edx, [regData+REG_DATA.licType]  

.text:0040E73D                   mov     ecx, 100h  

.text:0040E742                   mov     eax, edi  

.text:0040E744                   call    Proxies::CreateSubClass(System::TObject *,System::AnsiString,System::TMetaClass *)  

.text:0040E749                   lea     edx, [esp+0E88h+regUid]  

.text:0040E750                   mov     ecx, 80h  

.text:0040E755                   mov     eax, edi  

.text:0040E757                   call    Proxies::CreateSubClass(System::TObject *,System::AnsiString,System::TMetaClass *)  

.text:0040E75C                   xor     ebx, ebx  

.text:0040E75E  

.text:0040E75E loc_40E75E:                               ; CODE XREF: check_regdata+B7j  

.text:0040E75E                   imul    eax, ebx, 36h  

.text:0040E761                   mov     ecx, 400h  

.text:0040E766                   mov     edx, eax  

.text:0040E768                   sub     ecx, eax  

.text:0040E76A                   lea     eax, [esp+0E88h+strTemp]  

.text:0040E76E                   add     edx, eax  

.text:0040E770                   mov     eax, edi  

.text:0040E772                   call    Proxies::CreateSubClass(System::TObject *,System::AnsiString,System::TMetaClass *)  

.text:0040E777                   inc     ebx  

.text:0040E778                   cmp     ebx, 7  

.text:0040E77B                   jl      short loc_40E75E    ;把 UID 行以后的, 当一串字符串(去掉 CR/LF)  

.text:0040E77D                   lea     eax, [esp+0E88h+strTemp]  

.text:0040E781                   mov     edx, 2  

.text:0040E786                   call    rar_atoi      ;rar 实现的把指定长度的字符串转换位整型  

.text:0040E78B                   mov     [esp+0E88h+snCode1_len], eax  

.text:0040E78F                   lea     eax, [esp+0E88h+strTemp+2]  

.text:0040E793                   mov     edx, 3  

.text:0040E798                   call    rar_atoi  

.text:0040E79D                   mov     edi, eax  

.text:0040E79F                   lea     eax, [esp+0E88h+strTemp+5]  

.text:0040E7A3                   mov     edx, 3  

.text:0040E7A8                   call    rar_atoi  

.text:0040E7AD                   mov     ebp, eax  

.text:0040E7AF                   lea     eax, [esp+0E88h+strTemp+8]  

.text:0040E7B3                   mov     edx, 2  

.text:0040E7B8                   call    rar_atoi  

.text:0040E7BD                   mov     [esp+0E88h+snCode4_len], eax  

.text:0040E7C1                   cmp     edi, 100h        ; [Check2] 比较[注册字符串 2]的长度  

.text:0040E7C7                   jg      short loc_40E7D1 ; [Check2]  

.text:0040E7C7  

.text:0040E7C9                   cmp     ebp, 100h        ; [Check2] 比较[注册字符串 3]的长度  

.text:0040E7CF                   jle     short loc_40E7D8  

.text:0040E7D1  

.text:0040E7D1 loc_40E7D1:                               ; CODE XREF: check_regdata+103j  

.text:0040E7D1                   xor     eax, eax         ; [Check2]  

.text:0040E7D3                   jmp     REG_FAIL_EXIT   ; 如果 [注册字符串 2], [注册字符串 3] 有一个长度大于 256,  

.text:0040E7D3                                     ; 注册失败!  
```

按字符串的前 10 位(`snHead="6412212250"`)分割字符串
