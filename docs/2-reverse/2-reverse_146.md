

# 给 OD 字符串溢出问题打补丁（打完可分析 mscorwks.dll 了）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-348.htm`](https://www.pediy.com/kssd/pediy08/pediy8-348.htm)

昨天用 ollydbg 调试一个加壳的.net 程序，结果发现 od 在分析 mscorwks.dll 时会崩溃，直接用 od 打开 mscorwks.dll 也这样，  
跟踪了一下，发现 od 的几个与字符串相关的 API 存在缓冲区溢出的 BUG，在 API 说明里这些 API 的字符串参数一般说至少要分  
配 TEXTLEN 长度的缓冲，似乎已经暗示着可能会溢出了，不过 od 自己在使用它们的时候好象也只分配了 TEXTLEN 长度的缓冲  
，也就是最低数量的缓冲，API 里没有检查，都是直接复制，这些缓冲区大多在栈上分配，因此一旦溢出程序就乱套了。  
我用 mscorwks.dll 试验发现了三个 API 存在这个问题，即 Findlabel,Findname,Findnextname，其它 API 也可能有这个问题，  
给这三个 API 打补丁后 mscorwks.dll 分析能通过了，所以偶就不深究了，呵呵。  
  
打补丁的方法参照了 kanxue 老大的“给 Ollydbg 的增加实用的快捷键操作功能”一文中的方法，在 OD 代码段后面的空地上  
加的补丁代码：）  
http://bbs1.pediy.com:8081/showthread.php?s=&threadid=24059&highlight=%B2%B9%B6%A1  
  
1、对原 API 代码的修改.  
\_Findlabel:  
00464F88     /E9 33A80400   jmp     004AF7C0  
00464F8D  |. |90            |nop  
00464F8E  |. |90            |nop  
00464F8F  |. |90            |nop  
00464F90  |. |90            |nop  
00464F91  |. |90            |nop  
00464F92  |. |90            |nop  
00464F93  |. |90            |nop  
00464F94  |. |90            |nop  
00464F95  |. |90            |nop  
00464F96  |. |90            |nop  
00464F97  |. |90            |nop  
00464F98  |. |90            |nop  
00464F99  |. |90            |nop  
00464F9A  |. |90            |nop  
00464F9B  |. |90            |nop  
00464F9C  |. |90            |nop  
00464F9D  |. |90            |nop  
00464F9E  |. |90            |nop  
00464F9F  |. |90            |nop  
00464FA0  |. |90            |nop  
00464FA1  |. |90            |nop  
00464FA2  |. |90            |nop  
  
\_Findname:  
00464A67     /E9 8CAD0400   jmp     004AF7F8  
00464A6C     |90            nop  
00464A6D     |90            |nop                                                                   ; |  
00464A6E     |90            nop  
00464A6F     |90            nop  
00464A70     |90            |nop                                                                   ; |  
00464A71     |90            nop  
00464A72     |90            nop  
00464A73     |90            nop  
00464A74     |90            nop  
00464A75     |90            nop  
00464A76     |90            |nop                                                                   ; |  
00464A77     |90            nop  
00464A78     |90            nop  
00464A79     |90            nop  
00464A7A     |90            |nop                                                                   ; |  
00464A7B     |90            nop  
00464A7C     |90            nop  
00464A7D     |90            nop  
00464A7E     |90            nop  
00464A7F     |90            nop  
00464A80     |90            |nop                                                                   ; |Arg2  
00464A81     |90            |nop                                                                   ; |Arg1  
00464A82     |90            |nop                                                                   ; \\OllyDBG\_.004A3530  
00464A83     |90            nop  
00464A84     |90            nop  
00464A85     |90            nop  
00464A86     |90            nop  
00464A87     |90            |nop  
00464A88     |90            nop  
00464A89     |90            nop  
  
\_Findnextname:  
00464EBA     /E9 77A90400   jmp     004AF836  
00464EBF     |90            |nop  
00464EC0     |90            nop  
00464EC1     |90            |nop  
00464EC2     |90            nop  
00464EC3     |90            |nop  
00464EC4     |90            nop  
00464EC5     |90            |nop  
00464EC6     |90            nop  
00464EC7     |90            |nop  
00464EC8     |90            nop  
00464EC9     |90            nop  
00464ECA     |90            |nop  
00464ECB     |90            nop  
00464ECC     |90            |nop  
00464ECD     |90            nop  
00464ECE     |90            |nop  
00464ECF     |90            nop  
00464ED0     |90            |nop  
00464ED1     |90            nop  
00464ED2     |90            nop  
00464ED3     |90            |nop  
00464ED4     |90            nop  
  
2、对应的 API 补丁代码，没干别的，就是判断一下字符串长度，如果长度大于 TEXTLEN-1 则截断到 TEXTLEN-1 个  
字符，再在后面补个 0.  
\_Findlabel:  
004AF7C0      83C9 FF       or      ecx, FFFFFFFF  
004AF7C3      F2:AE         repne   scas byte ptr es:\[edi\]  
004AF7C5      F7D1          not     ecx  
004AF7C7      2BF9          sub     edi, ecx  
004AF7C9      81F9 FF000000 cmp     ecx, 0FF  
004AF7CF      7E 0A         jle     short 004AF7DB  
004AF7D1      B9 FF000000   mov     ecx, 0FF  
004AF7D6      33C0          xor     eax, eax  
004AF7D8      880431        mov     \[ecx+esi\], al  
004AF7DB      8BD1          mov     edx, ecx  
004AF7DD      87F7          xchg    edi, esi  
004AF7DF      C1E9 02       shr     ecx, 2  
004AF7E2      8BC7          mov     eax, edi  
004AF7E4      F3:A5         rep     movs dword ptr es:\[edi\], dword ptr \[esi\]  
004AF7E6      8BCA          mov     ecx, edx  
004AF7E8      83E1 03       and     ecx, 3  
004AF7EB      F3:A4         rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
004AF7ED    ^ E9 AE57FBFF   jmp     00464FA0  
004AF7F2      90            nop  
004AF7F3      90            nop  
004AF7F4      90            nop  
004AF7F5      90            nop  
004AF7F6      90            nop  
004AF7F7      90            nop  
  
\_Findname:  
004AF7F8      81FE FF000000 cmp     esi, 0FF  
004AF7FE      7E 05         jle     short 004AF805  
004AF800      BE FF000000   mov     esi, 0FF  
004AF805      33C0          xor     eax, eax  
004AF807      88043E        mov     \[esi+edi\], al  
004AF80A      56            push    esi  
004AF80B      8B13          mov     edx, \[ebx\]  
004AF80D      8D1452        lea     edx, \[edx+edx\*2\]  
004AF810      8B0D 14AE4E00 mov     ecx, \[4EAE14\]  
004AF816      8B4491 04     mov     eax, \[ecx+edx\*4+4\]  
004AF81A      0305 30AE4E00 add     eax, \[4EAE30\]  
004AF820      50            push    eax  
004AF821      57            push    edi  
004AF822      E8 093DFFFF   call    004A3530  
004AF827      83C4 0C       add     esp, 0C  
004AF82A    ^ E9 5952FBFF   jmp     00464A88  
004AF82F      90            nop  
004AF830      90            nop  
004AF831      90            nop  
004AF832      90            nop  
004AF833      90            nop  
004AF834      90            nop  
004AF835      90            nop  
  
\_Findnextname:  
004AF836      83C9 FF       or      ecx, FFFFFFFF  
004AF839      F2:AE         repne   scas byte ptr es:\[edi\]  
004AF83B      F7D1          not     ecx  
004AF83D      2BF9          sub     edi, ecx  
004AF83F      81F9 FF000000 cmp     ecx, 0FF  
004AF845      7E 0A         jle     short 004AF851  
004AF847      B9 FF000000   mov     ecx, 0FF  
004AF84C      33C0          xor     eax, eax  
004AF84E      880431        mov     \[ecx+esi\], al  
004AF851      8BD1          mov     edx, ecx  
004AF853      87F7          xchg    edi, esi  
004AF855      C1E9 02       shr     ecx, 2  
004AF858      8BC7          mov     eax, edi  
004AF85A      F3:A5         rep     movs dword ptr es:\[edi\], dword ptr \[esi\]  
004AF85C      8BCA          mov     ecx, edx  
004AF85E      83E1 03       and     ecx, 3  
004AF861      F3:A4         rep     movs byte ptr es:\[edi\], byte ptr \[esi\]  
004AF863    ^ E9 6856FBFF   jmp     00464ED0  
  
偶是改的 OllyICE v1.10 修改版 \[2006.6.21\] 的英文版主程序  
（偶用 OD 的 dump 插件保存的，结果大了 300K，不知道如何减肥，谁帮偶弄一把吧）： 

 4dd466d0e1e0cc123afbe504da2462ca