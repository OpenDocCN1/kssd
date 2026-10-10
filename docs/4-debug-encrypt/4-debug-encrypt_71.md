

# 三江门诊收费系统注册算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-159.htm`](https://www.pediy.com/kssd/pediy08/pediy8-159.htm)

【破文标题】三江门诊收费系统注册算法分析（MD5 算法）  
【破文作者】紫色缘\[TFW\]\[PCG\]  
【作者邮箱】Cn\_Fish@126.com  
【作者主页】www.cniso.org  
【破解工具】Peid\\OD  
【破解平台】Win9x/NT/2000/XP  
【软件名称】\*\*\*\*门诊收费系统  
【软件大小】2MB  
【原版下载】因为是国产共享软件，所以不提供软件下载！望各位见谅..  
【保护方式】注册号及试用天数  
【软件简介】门诊收费系统  
\------------------------------------------------------------------------  
【破解过程】第一先启动主程序，在输入假码后出现错误提示框。  
再使用 Peid 查壳后，发现是 Borland Delphi 6.0 - 7.0 语言编写，于是用 OD 载入查找到字符：  
超级字串参考＋        , 条目 3467  
 地址=00578439  
 反汇编=mov     eax, 00578530  
 文本字串=未能注册成功，请检查注册码是否正确  
双击该行后来到地址 578439，因为要找出软件的算法流程，于是向上翻去，然后来到以下的代码处：  
  
00578295  |.  55            push    ebp                                          ;  来到此处，下断后运行软件，断下后 F8 继续跟  
00578296  |.  68 A6845700   push    005784A6  
0057829B  |.  64:FF30       push    dword ptr fs:\[eax\]  
0057829E  |.  64:8920       mov     fs:\[eax\], esp  
005782A1  |.  8D55 F8       lea     edx, \[ebp-8\]  
005782A4  |.  8B83 04030000 mov     eax, \[ebx+304\]  
005782AA  |.  E8 CD00F0FF   call    0047837C                                     ;  读取假码位数  
005782AF  |.  8B45 F8       mov     eax, \[ebp-8\]                                 ;  假码位数送入 EAX  
005782B2  |.  8D55 FC       lea     edx, \[ebp-4\]  
005782B5  |.  E8 0E0AE9FF   call    00408CC8                                     ;  读假码位数  
005782BA  |.  837D FC 00    cmp     dword ptr \[ebp-4\], 0                         ;  比较输入的注册码是否等于 0  
005782BE  |.  75 0F         jnz     short 005782CF                               ;  不是则跳转  
005782C0  |.  B8 BC845700   mov     eax, 005784BC                                ;  注册码不能为空  
005782C5  |.  E8 0237ECFF   call    0043B9CC  
005782CA  |.  E9 74010000   jmp     00578443  
005782CF  |>  8D55 F0       lea     edx, \[ebp-10\]  
005782D2  |.  8B83 04030000 mov     eax, \[ebx+304\]  
005782D8  |.  E8 9F00F0FF   call    0047837C  
005782DD  |.  8B45 F0       mov     eax, \[ebp-10\]                                ;  假码位数送入 EAX  
005782E0  |.  8D55 F4       lea     edx, \[ebp-C\]  
005782E3  |.  E8 E009E9FF   call    00408CC8  
005782E8  |.  8B45 F4       mov     eax, \[ebp-C\]                                 ;  假码位数送入 EAX  
005782EB  |.  50            push    eax                                          ;  假码入堆栈  
005782EC  |.  8D45 D8       lea     eax, \[ebp-28\]  
005782EF  |.  50            push    eax  
005782F0  |.  8D55 D4       lea     edx, \[ebp-2C\]  
005782F3  |.  8B83 FC020000 mov     eax, \[ebx+2FC\]  
005782F9  |.  E8 7E00F0FF   call    0047837C                                     ;  读取机器码  
005782FE  |.  8B45 D4       mov     eax, \[ebp-2C\]                                ;  机器码位数送入 EAX  
00578301  |.  B9 1A000000   mov     ecx, 1A                                      ;  ECX=1A  
00578306  |.  BA 05000000   mov     edx, 5                                       ;  EDX=5  
0057830B  |.  E8 20C6E8FF   call    00404930                                     ;  去除机器码前 4 位，后 2 位，最后取得的机器码  
  
为 26 位  
00578310  |.  8B45 D8       mov     eax, \[ebp-28\]                                ;  26 位值送入 EAX  
00578313  |.  8D55 DC       lea     edx, \[ebp-24\]  
00578316  |.  E8 79FBFFFF   call    00577E94                                     ;  MD5 出现 call，跟进  
0057831B  |.  8D45 DC       lea     eax, \[ebp-24\]  
0057831E  |.  8D55 EC       lea     edx, \[ebp-14\]  
00578321  |.  E8 E2FBFFFF   call    00577F08                                     ;  算法 CALL，跟进  
00578326  |.  8B55 EC       mov     edx, \[ebp-14\]                                ;  真码送入 EDX  
00578329  |.  58            pop     eax                                          ;  假码出堆栈  
0057832A  |.  E8 EDC4E8FF   call    0040481C                                     ;  经典比较  
0057832F  |.  0F85 04010000 jnz     00578439                                     ;  爆破 NOP  
00578335  |.  8B83 10030000 mov     eax, \[ebx+310\]  
0057833B  |.  E8 8CB2F3FF   call    004B35CC  
00578340  |.  8B83 10030000 mov     eax, \[ebx+310\]  
00578346  |.  E8 E56CF6FF   call    004DF030  
0057834B  |.  8B10          mov     edx, \[eax\]  
0057834D  |.  FF52 44       call    \[edx+44\]  
00578350  |.  8B83 10030000 mov     eax, \[ebx+310\]  
00578356  |.  E8 D56CF6FF   call    004DF030  
0057835B  |.  BA D4845700   mov     edx, 005784D4                                ;  select \* from tb\_zc  
00578360  |.  8B08          mov     ecx, \[eax\]  
00578362  |.  FF51 38       call    \[ecx+38\]  
00578365  |.  8B83 10030000 mov     eax, \[ebx+310\]  
0057836B  |.  E8 50B2F3FF   call    004B35C0  
00578370  |.  8B83 10030000 mov     eax, \[ebx+310\]  
00578376  |.  8B10          mov     edx, \[eax\]  
00578378  |.  FF92 4C010000 call    \[edx+14C\]  
0057837E  |.  85C0          test    eax, eax  
00578380  |.  0F85 9B000000 jnz     00578421  
00578386  |.  8B83 14030000 mov     eax, \[ebx+314\]  
0057838C  |.  E8 2FB2F3FF   call    004B35C0  
00578391  |.  8B83 14030000 mov     eax, \[ebx+314\]  
00578397  |.  E8 38DBF3FF   call    004B5ED4  
0057839C  |.  8D55 CC       lea     edx, \[ebp-34\]  
0057839F  |.  8B83 FC020000 mov     eax, \[ebx+2FC\]  
005783A5  |.  E8 D2FFEFFF   call    0047837C  
005783AA  |.  8B45 CC       mov     eax, \[ebp-34\]  
005783AD  |.  8D55 D0       lea     edx, \[ebp-30\]  
005783B0  |.  E8 1309E9FF   call    00408CC8  
005783B5  |.  8B45 D0       mov     eax, \[ebp-30\]  
005783B8  |.  50            push    eax  
005783B9  |.  BA F0845700   mov     edx, 005784F0                                ;  机器码  
005783BE  |.  8B83 14030000 mov     eax, \[ebx+314\]  
005783C4  |.  E8 07C2F3FF   call    004B45D0  
005783C9  |.  5A            pop     edx  
005783CA  |.  8B08          mov     ecx, \[eax\]  
005783CC  |.  FF91 B0000000 call    \[ecx+B0\]  
005783D2  |.  8D55 C4       lea     edx, \[ebp-3C\]  
005783D5  |.  8B83 04030000 mov     eax, \[ebx+304\]  
005783DB  |.  E8 9CFFEFFF   call    0047837C  
005783E0  |.  8B45 C4       mov     eax, \[ebp-3C\]  
005783E3  |.  8D55 C8       lea     edx, \[ebp-38\]  
005783E6  |.  E8 DD08E9FF   call    00408CC8  
005783EB  |.  8B45 C8       mov     eax, \[ebp-38\]  
005783EE  |.  50            push    eax  
005783EF  |.  BA 00855700   mov     edx, 00578500                                ;  注册码  
005783F4  |.  8B83 14030000 mov     eax, \[ebx+314\]  
005783FA  |.  E8 D1C1F3FF   call    004B45D0  
005783FF  |.  5A            pop     edx  
00578400  |.  8B08          mov     ecx, \[eax\]  
00578402  |.  FF91 B0000000 call    \[ecx+B0\]  
00578408  |.  8B83 14030000 mov     eax, \[ebx+314\]  
0057840E  |.  8B10          mov     edx, \[eax\]  
00578410  |.  FF92 4C020000 call    \[edx+24C\]  
00578416  |.  8B83 14030000 mov     eax, \[ebx+314\]  
0057841C  |.  E8 ABB1F3FF   call    004B35CC  
00578421  |>  B8 10855700   mov     eax, 00578510                                ;  注册成功,请重新运行软件  
00578426  |.  E8 A135ECFF   call    0043B9CC  
0057842B  |.  A1 F4285900   mov     eax, \[5928F4\]  
00578430  |.  8B00          mov     eax, \[eax\]  
00578432  |.  E8 7D15F2FF   call    004999B4  
00578437  |.  EB 0A         jmp     short 00578443  
00578439  |>  B8 30855700   mov     eax, 00578530                                ;  未能注册成功，请检查注册码是否正确  
0057843E  |.  E8 8935ECFF   call    0043B9CC  
00578443  |>  33C0          xor     eax, eax  
00578445  |.  5A            pop     edx  
00578446  |.  59            pop     ecx  
00578447  |.  59            pop     ecx  
00578448  |.  64:8910       mov     fs:\[eax\], edx  
0057844B  |.  68 AD845700   push    005784AD  
00578450  |>  8D45 C4       lea     eax, \[ebp-3C\]  
00578453  |.  E8 B8BFE8FF   call    00404410  
00578458  |.  8D45 C8       lea     eax, \[ebp-38\]  
0057845B  |.  E8 B0BFE8FF   call    00404410  
00578460  |.  8D45 CC       lea     eax, \[ebp-34\]  
00578463  |.  E8 A8BFE8FF   call    00404410  
00578468  |.  8D45 D0       lea     eax, \[ebp-30\]  
0057846B  |.  E8 A0BFE8FF   call    00404410  
00578470  |.  8D45 D4       lea     eax, \[ebp-2C\]  
00578473  |.  BA 02000000   mov     edx, 2  
00578478  |.  E8 B7BFE8FF   call    00404434  
0057847D  |.  8D45 EC       lea     eax, \[ebp-14\]  
00578480  |.  E8 8BBFE8FF   call    00404410  
00578485  |.  8D45 F0       lea     eax, \[ebp-10\]  
00578488  |.  E8 83BFE8FF   call    00404410  
0057848D  |.  8D45 F4       lea     eax, \[ebp-C\]  
00578490  |.  E8 7BBFE8FF   call    00404410  
00578495  |.  8D45 F8       lea     eax, \[ebp-8\]  
00578498  |.  E8 73BFE8FF   call    00404410  
0057849D  |.  8D45 FC       lea     eax, \[ebp-4\]  
005784A0  |.  E8 6BBFE8FF   call    00404410  
005784A5  \\.  C3            retn  
  
\--------------------跟进来到 578316 处后，代码如下----------------------------------------  
  
00577E94  /$  55            push    ebp  
00577E95  |.  8BEC          mov     ebp, esp  
00577E97  |.  83C4 A4       add     esp, -5C  
00577E9A  |.  53            push    ebx  
00577E9B  |.  8BDA          mov     ebx, edx  
00577E9D  |.  8945 FC       mov     \[ebp-4\], eax  
00577EA0  |.  8B45 FC       mov     eax, \[ebp-4\]  
00577EA3  |.  E8 18CAE8FF   call    004048C0  
00577EA8  |.  33C0          xor     eax, eax  
00577EAA  |.  55            push    ebp  
00577EAB  |.  68 FA7E5700   push    00577EFA  
00577EB0  |.  64:FF30       push    dword ptr fs:\[eax\]  
00577EB3  |.  64:8920       mov     fs:\[eax\], esp  
00577EB6  |.  8D45 A4       lea     eax, \[ebp-5C\]  
00577EB9  |.  E8 AEFEFFFF   call    00577D6C                         ;  F7 跟进  
00577EBE  |.  8B45 FC       mov     eax, \[ebp-4\]                     ;  \[EBP-4\]送入 EAX  
00577EC1  |.  E8 0AC8E8FF   call    004046D0  
00577EC6  |.  50            push    eax                              ;  EAX 压入堆栈  
00577EC7  |.  8B45 FC       mov     eax, \[ebp-4\]                     ;  \[EBP-4\]送入 EAX  
00577ECA  |.  E8 01CAE8FF   call    004048D0  
00577ECF  |.  8BD0          mov     edx, eax                         ;  EAX 送入 EDX  
00577ED1  |.  8D45 A4       lea     eax, \[ebp-5C\]  
00577ED4  |.  59            pop     ecx  
00577ED5  |.  E8 C6FEFFFF   call    00577DA0  
00577EDA  |.  8BD3          mov     edx, ebx  
00577EDC  |.  8D45 A4       lea     eax, \[ebp-5C\]  
00577EDF  |.  E8 3CFFFFFF   call    00577E20  
00577EE4  |.  33C0          xor     eax, eax  
00577EE6  |.  5A            pop     edx  
00577EE7  |.  59            pop     ecx  
00577EE8  |.  59            pop     ecx  
00577EE9  |.  64:8910       mov     fs:\[eax\], edx  
00577EEC  |.  68 017F5700   push    00577F01  
00577EF1  |>  8D45 FC       lea     eax, \[ebp-4\]  
00577EF4  |.  E8 17C5E8FF   call    00404410  
00577EF9  \\.  C3            retn  
  
\--------------------跟进来到 577EB9 处后，代码如下----------------------------------------  
  
00577D6C  /$  C700 01234567 mov     dword ptr \[eax\], 67452301        ;  出现 MD5  
00577D72  |.  C740 04 89ABC>mov     dword ptr \[eax+4\], EFCDAB89  
00577D79  |.  C740 08 FEDCB>mov     dword ptr \[eax+8\], 98BADCFE  
00577D80  |.  C740 0C 76543>mov     dword ptr \[eax+C\], 10325476  
00577D87  |.  33D2          xor     edx, edx  
00577D89  |.  8950 10       mov     \[eax+10\], edx  
00577D8C  |.  33D2          xor     edx, edx                         ;  EDX 清 0  
00577D8E  |.  8950 14       mov     \[eax+14\], edx  
00577D91  |.  83C0 18       add     eax, 18                          ;  EAX=EAX+18  
00577D94  |.  BA 40000000   mov     edx, 40                          ;  40 送入 EDX  
00577D99  |.  E8 76F9E8FF   call    00407714  
00577D9E  \\.  C3            retn  
  
\--------------------跟进来到 578321 处后，代码如下----------------------------------------  
  
00577F08  /$  55            push    ebp                                          ;  来到此，继续 F8  
00577F09  |.  8BEC          mov     ebp, esp  
00577F0B  |.  83C4 E8       add     esp, -18  
00577F0E  |.  53            push    ebx  
00577F0F  |.  56            push    esi  
00577F10  |.  57            push    edi  
00577F11  |.  33C9          xor     ecx, ecx  
00577F13  |.  894D EC       mov     \[ebp-14\], ecx  
00577F16  |.  894D E8       mov     \[ebp-18\], ecx  
00577F19  |.  8BF0          mov     esi, eax  
00577F1B  |.  8D7D F0       lea     edi, \[ebp-10\]  
00577F1E  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]            ;  这里系列接下来出现了 4 组数据,是 MD5  
本人技术有限，仅怀疑这可能是 MD5，于是用了 peid 的插件 KANAL 查询一下算法，证实此加密是 MD5  
00577F1F  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]            ;  MD5 加密  
00577F20  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]            ;  这也是  
00577F21  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]            ;  这里也是  
00577F22  |.  8BFA          mov     edi, edx  
00577F24  |.  33C0          xor     eax, eax  
00577F26  |.  55            push    ebp  
00577F27  |.  68 A37F5700   push    00577FA3  
00577F2C  |.  64:FF30       push    dword ptr fs:\[eax\]  
00577F2F  |.  64:8920       mov     fs:\[eax\], esp  
00577F32  |.  8BC7          mov     eax, edi  
00577F34  |.  E8 D7C4E8FF   call    00404410  
00577F39  |.  B3 10         mov     bl, 10                                  ;  BL=10  
00577F3B  |.  8D75 F0       lea     esi, \[ebp-10\]  
00577F3E  |> /FF37          /push    dword ptr \[edi\]                        ;  EDI 压入堆栈  
00577F40  |. |8D45 EC       |lea     eax, \[ebp-14\]  
00577F43  |. |33D2          |xor     edx, edx                               ;  EDX 清 0  
00577F45  |. |8A16          |mov     dl, \[esi\]                              ;  \[ESI\]送入 DL  
00577F47  |. |C1EA 04       |shr     edx, 4                                 ;  /EDX 的 2 的 4 次方 ，即/16  
00577F4A  |. |83E2 0F       |and     edx, 0F                                ;  然后再与 0F 作与运算  
00577F4D  |. |8A92 A4235900 |mov     dl, \[edx+5923A4\]                       ;  值送入 DL 后转换为小写  
00577F53  |. |E8 A0C6E8FF   |call    004045F8  
00577F58  |. |FF75 EC       |push    dword ptr \[ebp-14\]  
00577F5B  |. |8D45 E8       |lea     eax, \[ebp-18\]  
00577F5E  |. |8A16          |mov     dl, \[esi\]                              ;  \[ESI\]送入 DL  
00577F60  |. |80E2 0F       |and     dl, 0F                                 ;  与 0F 作与运算  
00577F63  |. |81E2 FF000000 |and     edx, 0FF                               ;  值再与 0FF 作与运算  
00577F69  |. |8A92 A4235900 |mov     dl, \[edx+5923A4\]                       ;  值送入 DL 后转换为小写  
00577F6F  |. |E8 84C6E8FF   |call    004045F8  
00577F74  |. |FF75 E8       |push    dword ptr \[ebp-18\]  
00577F77  |. |8BC7          |mov     eax, edi                               ;  EDI 送入 EAX  
00577F79  |. |BA 03000000   |mov     edx, 3                                 ;  EDX=EDX+3  
00577F7E  |. |E8 0DC8E8FF   |call    00404790  
00577F83  |. |46            |inc     esi                                    ;  ESI 加 1  
00577F84  |. |FECB          |dec     bl                                     ;  减 1  
00577F86  |.^\\75 B6         \\jnz     short 00577F3E                         ;  循环取  
00577F88  |.  33C0          xor     eax, eax  
00577F8A  |.  5A            pop     edx  
00577F8B  |.  59            pop     ecx  
00577F8C  |.  59            pop     ecx  
00577F8D  |.  64:8910       mov     fs:\[eax\], edx  
00577F90  |.  68 AA7F5700   push    00577FAA  
00577F95  |>  8D45 E8       lea     eax, \[ebp-18\]  
00577F98  |.  BA 02000000   mov     edx, 2  
00577F9D  |.  E8 92C4E8FF   call    00404434  
00577FA2  \\.  C3            retn  
  
  
\------------------------------------------------------------------------  
【破解总结】  
  
1、注册码及机器码为固定的 32 位，机器码记为 A  
2、去除 A 前 4 位及尾部 2 位后得到 26 位的机器码，可记为 B  
3、把 B 进行 MD5 运算转换，结果记为 C  
4、先取反 C 的第一部分的值后/16 AND 0F 送入 DL 低位寄存器转换为小写，记为 ZCM1。  
5、然后与上一步骤一样，循环取反 C 的 ASCII 值后（AND 0F）AND 0FF 送入 DL 低位寄存器转换为小写，记为 ZCM2。  
6、最后一系列的值连接起来即是注册码。  
  
附注：本人的技术有限，表达的不够清晰。  
        
  
\--------------------------以下是 MD5 的值------------------------------------------------------  
  
1、00577F1E  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]                
ds:\[esi\]=stack \[0012F288\]=A322009A  
es:\[edi\]=stack \[0012F244\]=00000000  
  
2、00577F1F  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]              
ds:\[esi\]=stack \[0012F28C\]=3800B6F8  
es:\[edi\]=stack \[0012F248\]=00000000  
  
3、00577F20  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]              
ds:\[esi\]=stack \[0012F290\]=DC973D09  
es:\[edi\]=stack \[0012F24C\]=00000000  
  
4、00577F21  |.  A5            movs    dword ptr es:\[edi\], dword ptr \[esi\]              
ds:\[esi\]=stack \[0012F294\]=3EB52A2D  
es:\[edi\]=stack \[0012F250\]=00000000  
  
取 B 后与 MD5 运算转换的结果：   A3 22 00 9A  38 00 B6 F8    DC 97 3D 09   3E B5 2A 2D  
  
一系列的运算后得到真注册码： 9a 00 22 a3  f8 b6 00 38    09 3d 97 dc   2d 2a b5 3e  
  
  
标准 MD5 初始化的四个常数应该为  
  state\[0\] = 0x67452301;  
  state\[1\] = 0xefcdab89;  
  state\[2\] = 0x98badcfe;  
  state\[3\] = 0x10325476;  
  
  
\------------------------------------------------------------------------  
【版权声明】本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!   

 4dd466d0e1e0cc123afbe504da2462ca