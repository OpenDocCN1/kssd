

# 请求加精！绕过.Net 2.0 强名称验证，解决混合代码无法反编译的问题。

> 原文：[`www.pediy.com/kssd/pediy10/66392.html`](https://www.pediy.com/kssd/pediy10/66392.html)

目标：破解.NET2.0 的强名称验证。对于经过强名称签名的混合代码(含有原生代码或者非托管代码)程序，无法通过反编译修改的方法进行 Crack， 需要破解强名称验证才能进行修改。  
工具：OD,PEBrowsePro，WinHex  
关键：mscorwks.dll  
  
笔记一  破解强签名验证  
  
1、根据前人破.Net1.1 的经验，找到 Net 运行库目录，定位到 mscorsn.dll，OD 查找 String，得到结果  
  
  文本字串参考位于 mscorsn:.text, 条目 7  
   地址=79E611C1  
   反汇编=push    79E610C0  
   文本字串=ASCII "mscorsn.dll has been obsoleted. Please use mscoree.dll for Strong Name APIs.",LF  
2、找到 mscoree.dll，果然函数 StrongNameSignatureVerification 存在，下断，运行程序，没有断下来。  
  原因出在哪里呢？难道是 2.0 另有模块验证强名称？  
3、  在 OD 中设置选项-调试设置-中断于新模块，特别关注 system32 和.Net 运行目录下以 MS 开始命名的模块，  
  Executable modules, 条目 11  
   基址=79E70000  
   大小=00561000 (5640192.)  
   入口=79EA0B2C mscorwks.<模块入口点>  
   名称=mscorwks  
   文件版本=2.0.50727.42 (RTM.050727-4200)  
   路径=C:\\WINDOWS\\Microsoft.NET\\Framework\\v2.0.50727\\mscorwks.dll  
  果然在 mscorwks.dll 中发现 StrongNameSignatureVerification 函数，定位函数发现使用了重定位技术，看来有点不容易破  
  查找参考，发现运行时先调用 StrongNameSignatureVerification，成功后即加载，失败即报错，也就是说，只要修改判断即可  
  测试果然成功。具体如下：  
79F06A0A   . /0F85 CF981E00 jnz     7A0F02DF  
79F06A10   > |897D FC       mov     dword ptr \[ebp-4\], edi  
79F06A13   . |8B45 EC       mov     eax, dword ptr \[ebp-14\]  
79F06A16   . |85C0          test    eax, eax  
79F06A18   . |0F85 CD981E00 jnz     7A0F02EB  
79F06A1E   . |8D4E 20       lea     ecx, dword ptr \[esi+20\]  
79F06A21   . |E8 6316F8FF   call    79E88089  
79F06A26   . |FF75 08       push    dword ptr \[ebp+8\]  
79F06A29   . |68 12000080   push    80000012  
79F06A2E   . |50            push    eax  
79F06A2F   . |E8 A1000000   call    StrongNameSignatureVerification  ；调用函数  
79F06A34   > |0FB6C0        movzx   eax, al  
79F06A37   . |85C0          test    eax, eax            ；检查结果  
79F06A39     |E9 7C8A0800   jmp     79F8F4BA                                ;返回 0 报错，返回 1 通过验证，修改为直接跳到正常程序即可。  
79F06A3E     |90            nop  
79F06A3F   . |E8 41000000   call    StrongNameErrorInfo  
79F06A44   . |8BF8          mov     edi, eax  
79F06A46   . |85FF          test    edi, edi  
79F06A48   . |7D 0A         jge     short 79F06A54  
79F06A4A   . |57            push    edi  
79F06A4B   . |E8 24ABFBFF   call    79EC1574  
79F06A50   . |85C0          test    eax, eax  
79F06A52   . |75 1B         jnz     short 79F06A6F  
79F06A54   > |8B45 08       mov     eax, dword ptr \[ebp+8\]  
79F06A57   . |89BE 90000000 mov     dword ptr \[esi+90\], edi  
79F06A5D   . |8B00          mov     eax, dword ptr \[eax\]  
79F06A5F   . |8986 94000000 mov     dword ptr \[esi+94\], eax  
79F06A65   . |C786 8C000000>mov     dword ptr \[esi+8C\], 1  
79F06A6F   > |834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
79F06A73   . |8D4D EC       lea     ecx, dword ptr \[ebp-14\]  
79F06A76   . |E8 F779F9FF   call    79E9E472  
79F06A7B   . |8BC7          mov     eax, edi  
79F06A7D   > |E8 65ADF6FF   call    79E717E7  
79F06A82   . |C2 0400       retn    4  
79F06A85 > $ |6A 0C         push    0C  
79F06A87   . |B8 E427327A   mov     eax, 7A3227E4  
79F06A8C   . |E8 23ADF6FF   call    79E717B4  
79F06A91   . |6A 00         push    0  
79F06A93   . |8D4D E8       lea     ecx, dword ptr \[ebp-18\]  
79F06A96   . |E8 74ADF6FF   call    79E7180F  
79F06A9B   . |8365 FC 00    and     dword ptr \[ebp-4\], 0  
79F06A9F   . |A1 5C1C387A   mov     eax, dword ptr \[7A381C5C\]  
79F06AA4   . |85C0          test    eax, eax  
79F06AA6   . |0F85 43F21D00 jnz     7A0E5CEF  
79F06AAC   > |E8 828DFBFF   call    79EBF833  
79F06AB1   . |85C0          test    eax, eax  
79F06AB3   . |0F84 50F21D00 je      7A0E5D09  
79F06AB9   . |8B00          mov     eax, dword ptr \[eax\]  
79F06ABB   > |8BF0          mov     esi, eax  
79F06ABD   > |8365 EC 00    and     dword ptr \[ebp-14\], 0  
79F06AC1   . |834D FC FF    or      dword ptr \[ebp-4\], FFFFFFFF  
79F06AC5   . |8D4D E8       lea     ecx, dword ptr \[ebp-18\]  
79F06AC8   . |E8 6BADF6FF   call    79E71838  
79F06ACD   . |8BC6          mov     eax, esi  
79F06ACF   . |E8 13ADF6FF   call    79E717E7  
79F06AD4   . |C3            retn  
79F06AD5 > $ |6A 34         push    34  
79F06AD7   . |B8 A856327A   mov     eax, 7A3256A8  
79F06ADC   . |E8 D3ACF6FF   call    79E717B4  
79F06AE1   . |33DB          xor     ebx, ebx  
79F06AE3   . |53            push    ebx  
79F06AE4   . |8D4D E4       lea     ecx, dword ptr \[ebp-1C\]  
79F06AE7   . |885D F2       mov     byte ptr \[ebp-E\], bl  
79F06AEA   . |E8 20ADF6FF   call    79E7180F  
79F06AEF   . |895D FC       mov     dword ptr \[ebp-4\], ebx  
79F06AF2   . |A1 5C1C387A   mov     eax, dword ptr \[7A381C5C\]  
79F06AF7   . |3BC3          cmp     eax, ebx  
79F06AF9   . |0F85 910A1E00 jnz     7A0E7590  
79F06AFF   > |6A 24         push    24                               ; /n = 24 (36.)  
79F06B01   . |8D45 C0       lea     eax, dword ptr \[ebp-40\]          ; |  
79F06B04   . |53            push    ebx                              ; |c  
79F06B05   . |50            push    eax                              ; |s  
79F06B06   . |E8 4ECDF6FF   call    <jmp.&MSVCR80.memset>            ; \\memset  
79F06B0B   . |83C4 0C       add     esp, 0C  
79F06B0E   . |6A 01         push    1  
79F06B10   . |FF75 08       push    dword ptr \[ebp+8\]  
79F06B13   . |8D45 C0       lea     eax, dword ptr \[ebp-40\]  
79F06B16   . |50            push    eax  
79F06B17   . |885D F3       mov     byte ptr \[ebp-D\], bl  
79F06B1A   . |C645 E0 01    mov     byte ptr \[ebp-20\], 1  
79F06B1E   . |E8 02FDFFFF   call    79F06825  
  
4、绕过系统的强名称验证后，就可以为所欲为的进行 DIY 了。是不是很方便呢。