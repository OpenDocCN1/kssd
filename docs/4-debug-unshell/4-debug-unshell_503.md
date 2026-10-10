

# ASPR 学习笔记上篇：修复 IAT

> 原文：[`www.pediy.com/kssd/pediy10/80422.html`](https://www.pediy.com/kssd/pediy10/80422.html)

这次的目标是 AllNetic Working Time Tracker - Version 2.2，好像是一个工作时间安排的工具吧？  
下载地址是 http://www.allnetic.com/  
先查看一下壳的版本：  
PEiD 扫描：ASProtect 2.1x SKE -> Alexey Solodovnikov  
使用 verA0.13 插件扫描：Version: ASProtect 1.35 build 04.25 Release \[Extract\]  
  
找 OEP 就略过了，难度不大，太占篇幅了  
  
只总结一下 ASPR 程序找 OEP 的思路：  
1.刚开始 F7 慢慢跟，等跳到新的区段中后就可以观察后面的代码直接 F4 或 CTRL+F9 大段略过。  
2.当再次跳到新的区段时直接用 CTRL+F9 执行到返回。注意观察一下返回前的代码，有一个地方会连续把 8 个地址压入堆栈，这就说明很快要到达 OEP 了，经过 9 次 CTRL+F9 再 F7 慢慢跟，直到跳到一个新区段的开始位置时，跳过循环就到达 OEP 附近了。  
3.最简单快速的办法是下断点 bp GetSystemTime，然后经过十多次 RET，再 F7 到新区段的开始位置，再 F4 跳过循环，就到达 OEP 了。  
  
  
找到 OEP 后的程序的输入表很多都被加密了，需要修复 IAT。  
既然壳会解密出一部分 IAT，那么看看是在哪解密出来的？随便找一处有调用系统 API 的代码，发现形式是 call \[69CCXX\]，用 dd 69CC00 查看一下，然后向上翻找到最上面的一行，我这里是 69CB98，下硬件断点 hw 69CB98，重新载入后执行，经过几次数据初始化和代码解密的中断，来到这里：  

代码:

```
01263DC4    53               push    ebx
01263DC5    56               push    esi
01263DC6    57               push    edi
01263DC7    55               push    ebp
01263DC8    83C4 F4          add     esp, -0C
01263DCB    8BF0             mov     esi, eax
01263DCD    C60424 01        mov     byte ptr \[esp\], 1
01263DD1    EB 01            jmp     short 01263DD4
01263DD3    90               nop
01263DD4    837E 04 00       cmp     dword ptr \[esi+4\], 0        ; 导入表数据组为空？
01263DD8    0F84 24010000    je      01263F02
01263DDE    8B7E 04          mov     edi, dword ptr \[esi+4\]      ; 取导入表数据组基址
01263DE1    8B46 50          mov     eax, dword ptr \[esi+50\]     ; 取程序基址
01263DE4    894424 04        mov     dword ptr \[esp+4\], eax      ; 保存程序基址
01263DE8    EB 01            jmp     short 01263DEB
01263DEA    90               nop
01263DEB    8D46 34          lea     eax, dword ptr \[esi+34\]
01263DEE    B9 09000000      mov     ecx, 9
01263DF3    8BD7             mov     edx, edi
01263DF5    E8 9A1BFEFF      call    01245994                    ; 把导入表数据组前 9 字节复制出来
01263DFA    83C7 09          add     edi, 9
01263DFD    8B15 9C662601    mov     edx, dword ptr \[126669C\]
01263E03    8B12             mov     edx, dword ptr \[edx\]
01263E05    8BC6             mov     eax, esi
01263E07    E8 5CF1FFFF      call    01262F68
01263E0C    84C0             test    al, al
01263E0E    0F84 E2000000    je      01263EF6
01263E14    6A 14            push    14
01263E16    E8 89F1FEFF      call    01252FA4
01263E1B    01C7             add     edi, eax
01263E1D    E9 D4000000      jmp     01263EF6
01263E22    894424 08        mov     dword ptr \[esp+8\], eax      ; 保存 IAT 地址偏移
01263E26    6A FF            push    -1
01263E28    E8 77F1FEFF      call    01252FA4                    ; rnd
01263E2D    40               inc     eax
01263E2E    8903             mov     dword ptr \[ebx\], eax        ; 随机值覆盖掉导入表数据组 IAT 偏移
01263E30    83C7 04          add     edi, 4
01263E33    83C7 06          add     edi, 6
01263E36    8BEF             mov     ebp, edi                    ; DLL 名称
01263E38    8BC7             mov     eax, edi
01263E3A    E8 355EFEFF      call    01249C74                    ; 获取 DLL 名称长度
01263E3F    40               inc     eax
01263E40    0FB7C0           movzx   eax, ax
01263E43    03F8             add     edi, eax
01263E45    8B46 48          mov     eax, dword ptr \[esi+48\]
01263E48    8B00             mov     eax, dword ptr \[eax\]
01263E4A    3346 48          xor     eax, dword ptr \[esi+48\]
01263E4D    2B46 4C          sub     eax, dword ptr \[esi+4C\]
01263E50    E8 5FFFFFFF      call    01263DB4                    ; 校验 GetProcAddress 前 4 字节
01263E55    25 FF000000      and     eax, 0FF                    ; eax 必须为 0
01263E5A    03F8             add     edi, eax
01263E5C    8A1F             mov     bl, byte ptr \[edi\]          ; 取 API 名称字符串加密方式标志位 key
01263E5E    47               inc     edi
01263E5F    8B46 40          mov     eax, dword ptr \[esi+40\]
01263E62    8B00             mov     eax, dword ptr \[eax\]
01263E64    3346 40          xor     eax, dword ptr \[esi+40\]
01263E67    2B46 44          sub     eax, dword ptr \[esi+44\]
01263E6A    E8 45FFFFFF      call    01263DB4                    ; 校验 LoadLibraryA 前 4 字节
01263E6F    25 FF000000      and     eax, 0FF                    ; eax 必须为 0
01263E74    03F8             add     edi, eax
01263E76    8B4424 04        mov     eax, dword ptr \[esp+4\]      ; 取程序基址
01263E7A    014424 08        add     dword ptr \[esp+8\], eax      ; 程序基址+IAT 地址偏移：计算出 IAT 实际地址
01263E7E    EB 71            jmp     short 01263EF1
01263E80    3A5E 37          cmp     bl, byte ptr \[esi+37\]
01263E83    75 17            jnz     short 01263E9C
01263E85    83C7 02          add     edi, 2                      ; key=\[esi+37\]:
01263E88    8BC7             mov     eax, edi
01263E8A    50               push    eax
01263E8B    55               push    ebp
01263E8C    8D4424 10        lea     eax, dword ptr \[esp+10\]
01263E90    50               push    eax
01263E91    56               push    esi
01263E92    E8 EDFCFFFF      call    01263B84                    ; 保存 API 地址
01263E97    83C7 04          add     edi, 4                      ; 数据组指针+4
01263E9A    EB 52            jmp     short 01263EEE
01263E9C    3A5E 39          cmp     bl, byte ptr \[esi+39\]
01263E9F    75 17            jnz     short 01263EB8
01263EA1    83C7 02          add     edi, 2                      ; key=\[esi+39\]:
01263EA4    8BC7             mov     eax, edi
01263EA6    50               push    eax
01263EA7    55               push    ebp
01263EA8    8D4424 10        lea     eax, dword ptr \[esp+10\]
01263EAC    50               push    eax
01263EAD    56               push    esi
01263EAE    E8 D1FCFFFF      call    01263B84                    ; 保存 API 地址
01263EB3    83C7 02          add     edi, 2                      ; 数据组指针+2
01263EB6    EB 36            jmp     short 01263EEE
01263EB8    3A5E 38          cmp     bl, byte ptr \[esi+38\]
01263EBB    75 17            jnz     short 01263ED4
01263EBD    83C7 02          add     edi, 2                      ; key=\[esi+38\]:
01263EC0    8BC7             mov     eax, edi
01263EC2    50               push    eax
01263EC3    55               push    ebp
01263EC4    8D4424 10        lea     eax, dword ptr \[esp+10\]
01263EC8    50               push    eax
01263EC9    56               push    esi
01263ECA    E8 B5FCFFFF      call    01263B84                    ; 保存 API 地址
01263ECF    83C7 02          add     edi, 2                      ; 数据组指针+2
01263ED2    EB 1A            jmp     short 01263EEE
01263ED4    83C7 02          add     edi, 2                      ; 其他：
01263ED7    8BC7             mov     eax, edi
01263ED9    50               push    eax
01263EDA    55               push    ebp
01263EDB    8D4424 10        lea     eax, dword ptr \[esp+10\]
01263EDF    50               push    eax
01263EE0    56               push    esi
01263EE1    E8 9EFCFFFF      call    01263B84                    ; 保存 API 地址
01263EE6    0FB707           movzx   eax, word ptr \[edi\]         ; 取 API 字符串长度
01263EE9    83C0 02          add     eax, 2                      ; 数据组指针+2
01263EEC    03F8             add     edi, eax                    ; 数据组指针+字符串长
01263EEE    8A1F             mov     bl, byte ptr \[edi\]          ; 取下一个字符串解密标志位 key
01263EF0    47               inc     edi
01263EF1    3A5E 34          cmp     bl, byte ptr \[esi+34\]       ; key=\[ebx+34\]:循环结束
01263EF4  ^ 75 8A            jnz     short 01263E80
01263EF6    8BDF             mov     ebx, edi
01263EF8    8B03             mov     eax, dword ptr \[ebx\]        ; 取 IAT 地址偏移
01263EFA    85C0             test    eax, eax
01263EFC  ^ 0F85 20FFFFFF    jnz     01263E22                    ; 偏移为 0 时循环结束
01263F02    8A0424           mov     al, byte ptr \[esp\]
01263F05    83C4 0C          add     esp, 0C
01263F08    5D               pop     ebp
01263F09    5F               pop     edi
01263F0A    5E               pop     esi
01263F0B    5B               pop     ebx
01263F0C    C3               retn

```

其中保存 API 地址的 call 01263B84 为：  
代码:

```
01263B8D    53              push    ebx
01263B8E    56              push    esi
01263B8F    57              push    edi
01263B90    8B55 14         mov     edx, dword ptr \[ebp+14\]
01263B93    8B5D 08         mov     ebx, dword ptr \[ebp+8\]
01263B96    8DBD FAFEFFFF   lea     edi, dword ptr \[ebp-106\]
01263B9C    8BC2            mov     eax, edx
01263B9E    48              dec     eax
01263B9F    83E8 02         sub     eax, 2
01263BA2    0FB630          movzx   esi, byte ptr \[eax\]                 ; 解密方式标志位 key
01263BA5    8B45 10         mov     eax, dword ptr \[ebp+10\]
01263BA8    83E8 02         sub     eax, 2
01263BAB    0FB600          movzx   eax, byte ptr \[eax\]                 ; dll hash
01263BAE    3B43 2C         cmp     eax, dword ptr \[ebx+2C\]             ; is dll already loaded?
01263BB1    76 06           jbe     short 01263BB9
01263BB3    8943 2C         mov     dword ptr \[ebx+2C\], eax             ; save dll hash
01263BB6    EB 01           jmp     short 01263BB9
01263BB8    90              nop
01263BB9    33C0            xor     eax, eax
01263BBB    8A43 3B         mov     al, byte ptr \[ebx+3B\]               ; 第一种加密方式：key=ebx+3B
01263BBE    3BF0            cmp     esi, eax
01263BC0    75 5E           jnz     short 01263C20
01263BC2    EB 01           jmp     short 01263BC5
01263BC4    90              nop
01263BC5    66:8B02         mov     ax, word ptr \[edx\]                  ; 待解密 API 名称字符串的长度
01263BC8    66:8945 FA      mov     word ptr \[ebp-6\], ax
01263BCC    83C2 02         add     edx, 2
01263BCF    8955 FC         mov     dword ptr \[ebp-4\], edx              ; 待解密 API 名称字符串地址
01263BD2    66:817D FA FF00 cmp     word ptr \[ebp-6\], 0FF
01263BD8    76 0B           jbe     short 01263BE5                      ; 超长？
01263BDA    0FB745 FA       movzx   eax, word ptr \[ebp-6\]
01263BDE    E8 61E9FDFF     call    01242544
01263BE3    8BF8            mov     edi, eax
01263BE5    0FB775 FA       movzx   esi, word ptr \[ebp-6\]
01263BE9    8BCE            mov     ecx, esi
01263BEB    8B55 FC         mov     edx, dword ptr \[ebp-4\]
01263BEE    8BC7            mov     eax, edi
01263BF0    E8 9F1DFEFF     call    <CopyStringByByte>                  ; 加密的字符串复制出来
01263BF5    6A 0A           push    0A
01263BF7    8D4B 12         lea     ecx, dword ptr \[ebx+12\]             ; 解密：使用 ebx+12 开始的种子进行 XOR 解密
01263BFA    8BD6            mov     edx, esi
01263BFC    8BC7            mov     eax, edi
01263BFE    E8 9DDCFEFF     call    <DecyptString>                      ; 字符串解密
01263C03    897D FC         mov     dword ptr \[ebp-4\], edi
01263C06    8B45 FC         mov     eax, dword ptr \[ebp-4\]
01263C09    50              push    eax
01263C0A    8B45 10         mov     eax, dword ptr \[ebp+10\]
01263C0D    50              push    eax
01263C0E    53              push    ebx
01263C0F    E8 7CFDFFFF     call    <GetApiAddress>                     ; 根据 DLL 名和 API 名获取 API 地址
01263C14    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263C17    8B12            mov     edx, dword ptr \[edx\]
01263C19    8902            mov     dword ptr \[edx\], eax                ; 保存 IAT 地址
01263C1B    E9 74010000     jmp     01263D94
01263C20    33C0            xor     eax, eax
01263C22    8A43 35         mov     al, byte ptr \[ebx+35\]               ; 第二种加密方式：key=ebx+35,ebx+36,ebx+37,ebx+3A 及其他
01263C25    3BF0            cmp     esi, eax
01263C27    74 22           je      short 01263C4B
01263C29    33C0            xor     eax, eax
01263C2B    8A43 36         mov     al, byte ptr \[ebx+36\]
01263C2E    3BF0            cmp     esi, eax
01263C30    74 19           je      short 01263C4B
01263C32    33C0            xor     eax, eax
01263C34    8A43 37         mov     al, byte ptr \[ebx+37\]
01263C37    3BF0            cmp     esi, eax
01263C39    74 10           je      short 01263C4B
01263C3B    33C0            xor     eax, eax
01263C3D    8A43 3A         mov     al, byte ptr \[ebx+3A\]
01263C40    3BF0            cmp     esi, eax
01263C42    0F85 B7000000   jnz     01263CFF
01263C48    EB 01           jmp     short 01263C4B
01263C4A    90              nop
01263C4B    33C0            xor     eax, eax
01263C4D    8A43 36         mov     al, byte ptr \[ebx+36\]               ; key=ebx+36,ebx+3A 时才解密 IAT
01263C50    3BF0            cmp     esi, eax
01263C52    74 09           je      short 01263C5D                      ; \*是否解密 IAT
01263C54    33C0            xor     eax, eax
01263C56    8A43 3A         mov     al, byte ptr \[ebx+3A\]
01263C59    3BF0            cmp     esi, eax
01263C5B    75 43           jnz     short 01263CA0
01263C5D    66:8B02         mov     ax, word ptr \[edx\]                  ; 待解密 API 名称字符串的长度
01263C60    66:8945 FA      mov     word ptr \[ebp-6\], ax
01263C64    83C2 02         add     edx, 2
01263C67    8955 FC         mov     dword ptr \[ebp-4\], edx              ; 待解密 API 名称字符串地址
01263C6A    66:817D FA FF00 cmp     word ptr \[ebp-6\], 0FF
01263C70    76 0B           jbe     short 01263C7D                      ; 超长？
01263C72    0FB745 FA       movzx   eax, word ptr \[ebp-6\]
01263C76    E8 C9E8FDFF     call    01242544
01263C7B    8BF8            mov     edi, eax
01263C7D    0FB74D FA       movzx   ecx, word ptr \[ebp-6\]
01263C81    8B55 FC         mov     edx, dword ptr \[ebp-4\]
01263C84    8BC7            mov     eax, edi
01263C86    E8 091DFEFF     call    <CopyStringByByte>                  ; 字符串复制出来
01263C8B    6A 0A           push    0A
01263C8D    8D4B 08         lea     ecx, dword ptr \[ebx+8\]              ; 解密：使用 ebx+8 开始的种子进行 XOR 解密
01263C90    0FB755 FA       movzx   edx, word ptr \[ebp-6\]
01263C94    8BC7            mov     eax, edi
01263C96    E8 05DCFEFF     call    <DecyptString>                      ; 字符串解密
01263C9B    897D FC         mov     dword ptr \[ebp-4\], edi
01263C9E    EB 08           jmp     short 01263CA8
01263CA0    8B02            mov     eax, dword ptr \[edx\]
01263CA2    8945 FC         mov     dword ptr \[ebp-4\], eax
01263CA5    EB 01           jmp     short 01263CA8
01263CA7    90              nop
01263CA8    33C0            xor     eax, eax
01263CAA    8A43 3A         mov     al, byte ptr \[ebx+3A\]
01263CAD    3BF0            cmp     esi, eax
01263CAF    75 27           jnz     short 01263CD8                      ; key=ebx+3A 时获取 IAT 地址并修改成壳里调用
01263CB1    8B45 FC         mov     eax, dword ptr \[ebp-4\]
01263CB4    50              push    eax
01263CB5    8B45 10         mov     eax, dword ptr \[ebp+10\]
01263CB8    50              push    eax
01263CB9    53              push    ebx
01263CBA    E8 D1FCFFFF     call    <GetApiAddress>
01263CBF    8B15 AC672601   mov     edx, dword ptr \[12667AC\]
01263CC5    8902            mov     dword ptr \[edx\], eax
01263CC7    B8 9C752501     mov     eax, 125759C                        ; \*修改 IAT 地址为壳内容
01263CCC    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263CCF    8B12            mov     edx, dword ptr \[edx\]
01263CD1    8902            mov     dword ptr \[edx\], eax
01263CD3    E9 BC000000     jmp     01263D94
01263CD8    33C0            xor     eax, eax
01263CDA    8A43 35         mov     al, byte ptr \[ebx+35\]               ; key=ebx+35 时不保存 IAT
01263CDD    3BF0            cmp     esi, eax
01263CDF    0F84 AF000000   je      01263D94                            ; \*是否保存 IAT 地址
01263CE5    8B45 FC         mov     eax, dword ptr \[ebp-4\]
01263CE8    50              push    eax
01263CE9    8B45 10         mov     eax, dword ptr \[ebp+10\]
01263CEC    50              push    eax
01263CED    53              push    ebx
01263CEE    E8 9DFCFFFF     call    <GetApiAddress>
01263CF3    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263CF6    8B12            mov     edx, dword ptr \[edx\]
01263CF8    8902            mov     dword ptr \[edx\], eax                ; 保存 IAT 地址
01263CFA    E9 95000000     jmp     01263D94
01263CFF    33C0            xor     eax, eax
01263D01    8A43 38         mov     al, byte ptr \[ebx+38\]               ; 第三种加密方式：key=ebx+38,ebx+39
01263D04    3BF0            cmp     esi, eax                            ; 替换掉 IAT 中的 2 个地址
01263D06    75 41           jnz     short 01263D49
01263D08    EB 01           jmp     short 01263D0B
01263D0A    90              nop
01263D0B    B8 2C2E2601     mov     eax, 1262E2C                        ; key=ebx+38 时把地址替换为 1262E2C
01263D10    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263D13    8B12            mov     edx, dword ptr \[edx\]
01263D15    8902            mov     dword ptr \[edx\], eax
01263D17    B8 103F2601     mov     eax, 1263F10
01263D1C    BE 342E2601     mov     esi, 1262E34
01263D21    2BC6            sub     eax, esi
01263D23    50              push    eax
01263D24    56              push    esi
01263D25    E8 06D2FEFF     call    01250F30                            ; HASH(代码)
01263D2A    3B43 28         cmp     eax, dword ptr \[ebx+28\]             ; 校验 HASH
01263D2D    74 65           je      short 01263D94                      ; \*被修改时：更换 XOR 种子，改为 jmp
01263D2F    6A FF           push    -1
01263D31    E8 6EF2FEFF     call    01252FA4
01263D36    8D53 08         lea     edx, dword ptr \[ebx+8\]
01263D39    8902            mov     dword ptr \[edx\], eax
01263D3B    6A 1E           push    1E
01263D3D    E8 62F2FEFF     call    01252FA4
01263D42    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263D45    0102            add     dword ptr \[edx\], eax
01263D47    EB 4B           jmp     short 01263D94
01263D49    33C0            xor     eax, eax
01263D4B    8A43 39         mov     al, byte ptr \[ebx+39\]
01263D4E    3BF0            cmp     esi, eax
01263D50    75 42           jnz     short 01263D94
01263D52    EB 01           jmp     short 01263D55
01263D54    90              nop
01263D55    B8 74752501     mov     eax, 1257574                        ; key=ebx+39 时把地址替换为 1257574
01263D5A    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263D5D    8B12            mov     edx, dword ptr \[edx\]
01263D5F    8902            mov     dword ptr \[edx\], eax
01263D61    B8 103F2601     mov     eax, 1263F10
01263D66    BE 342E2601     mov     esi, 1262E34
01263D6B    2BC6            sub     eax, esi
01263D6D    50              push    eax
01263D6E    56              push    esi
01263D6F    E8 BCD1FEFF     call    01250F30                            ; HASH(代码)
01263D74    3B43 28         cmp     eax, dword ptr \[ebx+28\]             ; 校验 HASH
01263D77    74 1B           je      short 01263D94                      ; \*被修改时：更换 XOR 种子，改为 jmp
01263D79    6A FF           push    -1
01263D7B    E8 24F2FEFF     call    01252FA4
01263D80    8D53 08         lea     edx, dword ptr \[ebx+8\]
01263D83    8902            mov     dword ptr \[edx\], eax
01263D85    6A 32           push    32
01263D87    E8 18F2FEFF     call    01252FA4
01263D8C    8B55 0C         mov     edx, dword ptr \[ebp+C\]
01263D8F    0102            add     dword ptr \[edx\], eax
01263D91    EB 01           jmp     short 01263D94
01263D93    90              nop
01263D94    8B45 0C         mov     eax, dword ptr \[ebp+C\]
01263D97    8300 04         add     dword ptr \[eax\], 4
01263D9A    8D85 FAFEFFFF   lea     eax, dword ptr \[ebp-106\]
01263DA0    3BF8            cmp     edi, eax
01263DA2    74 07           je      short 01263DAB
01263DA4    8BC7            mov     eax, edi
01263DA6    E8 B1E7FDFF     call    0124255C
01263DAB    5F              pop     edi
01263DAC    5E              pop     esi
01263DAD    5B              pop     ebx
01263DAE    8BE5            mov     esp, ebp
01263DB0    5D              pop     ebp
01263DB1    C2 1000         retn    10

```

这一段就是 IAT 加密过程了，重点分析一下。  
流程分析：  
  见代码中的注释。  
使用的数据组分析：  
API 字符串基本数据组：  
代码:

```
EBX=012B04D0
012B04D0  B8 2D 26 01 9C 8C 2B 01 87 1F 11 BD 3B AE 95 94
012B04E0  4C BA 7B 5E 8C 79 70 8D 9A E1 A9 25 D9 B1 AF BF
012B04F0  37 1E B6 59 2D FB 00 00 B9 BE A4 40 2A 00 00 00
012B0500  30 05 2B 01 F9 85 BA A1 BD 30 96 F7 C7 00 00 00
   EBX +              34 35 36 37 38 39 3A 3B
012B0510  77 1D 80 7C FC E2 D5 F7 A0 AD 80 7C 2B 52 D5 F7
012B0520  00 00 40 00
含义：
+0:  dword     not used
+4:  dword     导入表数据组
+8:  byte\*A\*3  字符串加解密时所用的 XOR 种子，第 3 组似乎没用到
+26: byte\*2    not used
+28: dword     从 2E34 到 3F10 这一段数据和代码的 HASH
+2C: dword     上一次所用 DLL 名的 HASH
+30: dword     not used
+34: byte\*9    字符串加解密方式的标志位
+3D: byte\*3    not used
+40: dword     LoadLibraryA
+44: dword     LoadLibraryA 函数的 HASH
+48: dowrd     GetProcAddress
+4C: dword     GetProcAddress 函数的 HASH
+50: dword     程序基址
导入表字符串数据组分析：
+0:  byte\*9   字符串解密标志位
+9:  dowrd    从 DLL 获取 API 地址后存放位置的起始偏移地址
{
  +0:  dowrd    当前 DLL 字符串数据的总长度
  +4:  word     当前 DLL 的 HASH
  +6:  byte\*len(DLL name),\\0    DLL 名称，明文
  {
    +0:  byte     API 名称的加密方式标志位
    +1:  word     API 名称 HASH
    +3:  word     API 名称长度
    +5:  byte\*len(API name)    加密后的 API 名称字符串
  }\*n
}\*n

```

基本数据组的标志位从导入表数据组前 9 字节复制而来，含义为：  
34：循环结束  
35：不保存 IAT，使用 0125759C 替换  
36：加密方式二：使用+8 开始的 XOR 种子解密，直接保存  
37：不需解密直接保存字符串数据，API 硬编码  
38,39：加密方式三：替换为壳调用，并校验代码 HASH。只能对应一个 API  
3A：加密方式二：使用+8 开始的 XOR 种子解密，替换成壳里调用  
3B：加密方式一：使用+12 开始的 XOR 种子解密，直接保存  
3C：not used  
修复 IAT 需要更改保存 API 地址的 call 01263B84 中的代码流程，先归纳一下相关数据：  
1.使用到了 key=\[ebx+35\],\[ebx+36\],\[ebx+37\],\[ebx+38\],\[ebx+39\],\[ebx+3A\],\[ebx+3B\]，仅对这几个标志位的 IAT 进行计算。  
2.key=\[ebx+3B\]时使用+12 开始的 XOR 种子解密。  
3.key=\[ebx+37\]时没有 API 名称，直接用字符串数据填入到 IAT 中，相当于 API 硬编码。  
4.key=\[ebx+38\]时使用 01262E2C 替换，在数据窗口中分析 01262E2C 处的代码可以得出实际调用的是\[012679D0\]=kernel32.RaiseException。  
5.key=\[ebx+39\]时使用 01257574 替换，在数据窗口中分析 01257574 处的代码可以得出实际调用的是\[01269274\]=kernel32.GetProcAddress。  
6.key=\[ebx+35\],\[ebx+36\],\[ebx+3A\]时都可以解密出 API 地址，可以直接填入 IAT 中。  
所以需要调整代码流程为：  
1.key=\[ebx+3B\]流程不变。  
2.key=\[ebx+35\],\[ebx+36\],\[ebx+3A\]全部指向\[ebx+36\]流程。  
3.key=\[ebx+38\]修改保存地址为\[012679D0\]，并跳过代码 HASH 自校验过程。  
4.key=\[ebx+39\]修改保存地址为\[01269274\]，并跳过代码 HASH 自校验过程。  
需要修改的代码为：  
代码:

```
01263C27    74 34              je      short 01263C5D               ; \*\*\*\*\*\*je 01263C5D
01263C30    74 2B              je      short 01263C5D               ; \*\*\*\*\*\*je 01263C5D
01263C39    74 65              je      short 01263CA0               ; \*\*\*\*\*\*je 01263CA0
01263C48    EB 13              jmp     short 01263C5D               ; \*\*\*\*\*\*jmp 01263C5D
01263C9E    EB 45              jmp     short 01263CE5               ; \*\*\*\*\*\*jmp 01263CE5
01263CA5    EB 3E              jmp     short 01263CE5               ; \*\*\*\*\*\*jmp 01263CE5
01263D0B    A1 D0792601        mov     eax, dword ptr \[12679D0\]     ; \*\*\*\*\*\*mov eax,\[12679D0\]
01263D17    EB 7B              jmp     short 01263D94               ; \*\*\*\*\*\*jmp 01263D94
01263D55    A1 74922601        mov     eax, dword ptr \[1269274\]     ; \*\*\*\*\*\*mov eax, \[1269274\]
01263D61    EB 31              jmp     short 01263D94               ; \*\*\*\*\*\*jmp 01263D94

```

当外层循环执行完毕后需要把修改过的代码还原，因为在后面其他地方还有自校验。  
再次到达 OEP 后用 ImportREC 打开程序进程，填入  
OEP:289BE0  
RVA:29CB98  
SIZE:9B8  
搜索导入表，一个无效链接都没有了，心里感觉那叫一个爽快！  
敬请期待下一篇：修复跳转表