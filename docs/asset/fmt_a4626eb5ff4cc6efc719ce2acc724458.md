
# QQ 聊天记录查看器 5.3 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-414.htm`](https://www.pediy.com/kssd/pediy09/pediy09-414.htm)

> 【文章标题】: QQ 聊天记录查看器 5.3 算法分析  
> 【文章作者】: 斜阳残雪  
> 【作者邮箱】: zlm324@126.com  
> 【编写语言】: Delphi  
> 【软件介绍】: 可以绕过密码直接读取 QQ 的本地数据文件查看聊天记录  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> ---  
> 【详细过程】  
> 
> 上网闲逛的时候发现的，觉得应该有点用处，就载下来玩儿。  
> 
> 使用发现要注册，未注册版只给 2 次试用机会(太抠门了)，注册码输入错误有提示。  
> 
> `PEiD` 查壳，没有发现，编程语言为 `Delphi`。  
> 
> 用 `OD` 载入，下 `bp` `MessageBoxA`，注册失败后断下，向上看到这里：  
> 
> ```
> 0046EEF0  /.  55             push    ebp
> 0046EEF1  |.  8BEC           mov     ebp, esp
> 0046EEF3  |.  83C4 F4        add     esp, -0C
> 0046EEF6  |.  33C9           xor     ecx, ecx
> 0046EEF8  |.  894D F4        mov     dword ptr [ebp-C], ecx
> 0046EEFB  |.  8955 F8        mov     dword ptr [ebp-8], edx
> 0046EEFE  |.  8945 FC        mov     dword ptr [ebp-4], eax
> 0046EF01  |.  33C0           xor     eax, eax
> 0046EF03  |.  55             push    ebp
> 0046EF04  |.  68 AFEF4600    push    0046EFAF
> 0046EF09  |.  64:FF30        push    dword ptr fs:[eax]
> 0046EF0C  |.  64:8920        mov     dword ptr fs:[eax], esp
> 0046EF0F  |.  8D55 F4        lea     edx, dword ptr [ebp-C]
> 0046EF12  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EF15  |.  8B80 F0020000  mov     eax, dword ptr [eax+2F0]
> 0046EF1B  |.  E8 3496FCFF    call    00438554
> 0046EF20  |.  8B55 F4        mov     edx, dword ptr [ebp-C]
> 0046EF23  |.  B8 E89C4800    mov     eax, 00489CE8
> 0046EF28  |.  E8 A353F9FF    call    004042D0
> 0046EF2D  |.  A1 E89C4800    mov     eax, dword ptr [489CE8]
> 0046EF32  |.  E8 2DFDFFFF    call    0046EC64                     ;  关键 Call
> 0046EF37  |.  A3 EC9C4800    mov     dword ptr [489CEC], eax
> 0046EF3C      833D EC9C4800  >cmp     dword ptr [489CEC], 0              ;比较标志位
> 0046EF43      74 34           je      short 0046EF79                   ;跳走则失败
> 0046EF45      6A 00           push    0
> 0046EF47  |.  A1 E89C4800    mov     eax, dword ptr [489CE8]
> 0046EF4C  |.  E8 DB57F9FF    call    0040472C
> 0046EF51  |.  50             push    eax
> 0046EF52  |.  68 BCEF4600    push    0046EFBC
> 0046EF57  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EF5A  |.  E8 75FCFCFF    call    0043EBD4
> 0046EF5F  |.  50             push    eax                      ; |hOwner
> 0046EF60  |.  E8 4380F9FF    call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA
> 0046EF65  |.  A1 E89C4800    mov     eax, dword ptr [489CE8]
> 0046EF6A  |.  E8 A1FEFFFF    call    0046EE10
> 0046EF6F  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EF72  |.  E8 755DFEFF    call    00454CEC
> 0046EF77  |.  EB 20          jmp     short 0046EF99
> 0046EF79  |>  6A 00          push    0
> 0046EF7B  |.  A1 E89C4800    mov     eax, dword ptr [489CE8]
> 0046EF80  |.  E8 A757F9FF    call    0040472C
> 0046EF85  |.  50             push    eax
> 0046EF86  |.  68 C8EF4600    push    0046EFC8
> 0046EF8B  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EF8E  |.  E8 41FCFCFF    call    0043EBD4
> 0046EF93  |.  50             push    eax                      ; |hOwner
> 0046EF94  |.  E8 0F80F9FF    call    <jmp.&user32.MessageBoxA>        ; \MessageBoxA(出错信息)
> 0046EF99  |>  33C0           xor     eax, eax
> 0046EF9B  |.  5A             pop     edx
> 0046EF9C  |.  59             pop     ecx
> 0046EF9D  |.  59             pop     ecx
> 0046EF9E  |.  64:8910        mov     dword ptr fs:[eax], edx
> 0046EFA1  |.  68 B6EF4600    push    0046EFB6
> 0046EFA6  |>  8D45 F4        lea     eax, dword ptr [ebp-C]
> 0046EFA9  |.  E8 CE52F9FF    call    0040427C
> 0046EFAE  \\.  C3             retn
> ```
> 
> 跟进关键 Call：
> 
> ```
> 0046EC64  /$  55             push    ebp
> 0046EC65  |.  8BEC           mov     ebp, esp
> 0046EC67  |.  83C4 C8        add     esp, -38
> 0046EC6A  |.  8945 FC        mov     dword ptr [ebp-4], eax
> 0046EC6D  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EC70  |.  E8 A75AF9FF    call    0040471C
> 0046EC75  |.  33C0           xor     eax, eax
> 0046EC77  |.  55             push    ebp
> 0046EC78  |.  68 92ED4600    push    0046ED92
> 0046EC7D  |.  64:FF30        push    dword ptr fs:[eax]
> 0046EC80  |.  64:8920        mov     dword ptr fs:[eax], esp
> 0046EC83  |.  33C0           xor     eax, eax
> 0046EC85  |.  8945 F8        mov     dword ptr [ebp-8], eax
> 0046EC88  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EC8B  |.  E8 A458F9FF    call    00404534                   ;  取注册码长度
> 0046EC90  |.  83F8 10        cmp     eax, 10                    ;  判断是否为 16 位
> 0046EC93  |.  0F8C E3000000  jl      0046ED7C                       ;不是就完蛋
> 0046EC99  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046EC9C  |.  BA A8ED4600    mov     edx, 0046EDA8                ;  ASCII "1163659294813585"
> 0046ECA1      E8 D259F9FF    call    00404678
> 0046ECA6      0F84 D0000000  je      0046ED7C
> 0046ECAC  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046ECAF  |.  BA C4ED4600    mov     edx, 0046EDC4                ;  ASCII "0386848021608060"
> 0046ECB4  |.  E8 BF59F9FF    call    00404678
> 0046ECB9      0F84 BD000000  je      0046ED7C
> 0046ECBF  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046ECC2  |.  BA E0ED4600    mov     edx, 0046EDE0                ;  ASCII "8319E4005F00PYG0"
> 0046ECC7  |.  E8 AC59F9FF    call    00404678
> 0046ECCC      0F84 AA000000  je      0046ED7C
> 0046ECD2  |.  8B45 FC        mov     eax, dword ptr [ebp-4]
> 0046ECD5  |.  BA FCED4600    mov     edx, 0046EDFC                ;  ASCII "0566838690673180"
> 0046ECDA  |.  E8 9959F9FF    call    00404678
> 0046ECDF      0F84 97000000  je      0046ED7C
> 0046ECE5  |.  33C0           xor     eax, eax
> 0046ECE7  |.  8945 DC        mov     dword ptr [ebp-24], eax       ;  第一数据段清空
> 0046ECEA  |>  8B45 FC        /mov     eax, dword ptr [ebp-4]                ;eax 赋注册码的起始位置
> 0046ECED  |.  8B55 DC        |mov     edx, dword ptr [ebp-24]                ;edx 取计数器
> 0046ECF0  |.  8A0410         |mov     al, byte ptr [eax+edx]                ;送入一位注册码
> 0046ECF3  |.  E8 24FFFFFF    |call    0046EC1C                      ;  Ascii 码的处理(详见后面分析)
> 0046ECF8  |.  8B55 DC        |mov     edx, dword ptr [ebp-24]
> 0046ECFB  |.  884415 CB      |mov     byte ptr [ebp+edx-35], al                ;处理过的 Asc 码送入内存数据块
> 0046ECFF  |.  FF45 DC        |inc     dword ptr [ebp-24]                      ;计数器增 1
> 0046ED02  |.  837D DC 10     |cmp     dword ptr [ebp-24], 10                ;是否到 16 了？
> 0046ED06  |.^ 75 E2         \jnz      short 0046ECEA
> ```
> 
> 第一节处理后的数据不妨命名为串 1
> 
> ```
> 0046ED08  |.  33C0           xor     eax, eax
> 0046ED0A  |.  8945 E0        mov     dword ptr [ebp-20], eax       ;  第二数据段清空
> 0046ED0D  |>  8B45 E0        /mov     eax, dword ptr [ebp-20]                ;eax 为第一个计数器
> 0046ED10  |.  03C0           |add     eax, eax                              ;乘 2
> 0046ED12  |.  8A4405 CC      |mov     al, byte ptr [ebp+eax-34]                ;取上一节处理后得到的字符串的一位
> ;最后一次使用的是上一节的注册码长度"16"！
> 0046ED16  |.  C1E0 04        |shl     eax, 4                         ;  相当于 eax*2⁴
> 0046ED19  |.  8B55 E0        |mov     edx, dword ptr [ebp-20]                ;edx 是第二个计数器
> ```
