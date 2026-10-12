
# RemoteExecute 2.3 注册分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-424.htm`](https://www.pediy.com/kssd/pediy09/pediy09-424.htm)
>
> 【文章标题】: `RemoteExecute` 2.3 注册分析
>
> 【文章作者】: 南宫涤尘
>
> 【作者邮箱】: `zlm324@126.com`
>
> 【下载地址】: `http://www.newhua.com/soft/60828.htm`
>
> 【编写语言】: `VC++ 6.0`
>
> 【使用工具】: `OD` + `VB`
>
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!
>
> \--------------------------------------------------------------------------------
>
> 【详细过程】
>
> 运行，输入试炼码，有错误提示，下 `bp MessageBoxA`，断下，来到这里：
>
> ```
> 00402663   > /B8 54074A00   mov     eax, 004A0754
> 00402668   . |E8 7F240600   call    00464AEC
> 0040266D   . |83EC 18       sub     esp, 18
> 00402670   . |57            push    edi
> 00402671   . |8BF9          mov     edi, ecx
> 00402673   . |8B47 30       mov     eax, dword ptr [edi+30]
> 00402676   . |83F8 14       cmp     eax, 14                      ;  注册码应为 20 位
> 00402679   . |73 07         jnb     short 00402682
> 0040267B   . |33C0          xor     eax, eax
> 0040267D   . |E9 58010000   jmp     004027DA
> 00402682   > |8A45 F2       mov     al, byte ptr [ebp-E]
> 00402685   . |53            push    ebx
> 00402686   . |33DB          xor     ebx, ebx
> 00402688   . |56            push    esi
> 00402689   . |53            push    ebx
> 0040268A   . |8D4D DC       lea     ecx, dword ptr [ebp-24]
> 0040268D   . |8845 DC       mov     byte ptr [ebp-24], al
> 00402690   . |E8 44FBFFFF   call    004021D9
> 00402695   . |BE D8B64C00   mov     esi, 004CB6D8                 ;  固定字符串：ASCII "HJLMCAKN3P4OBUID05F6EVSZ2Q1TRG7YWX98: message too long for this public key"
> 0040269A   . |56            push    esi
> 0040269B   . |E8 B0260600   call    00464D50
> 004026A0   . |59            pop     ecx
> 004026A1   . |50            push    eax
> 004026A2   . |56            push    esi
> 004026A3   . |8D4D DC       lea     ecx, dword ptr [ebp-24]
> 004026A6   . |E8 D1F4FFFF   call    00401B7C
> 004026AB   . |6A 02         push    2
> 004026AD   . |8D77 28       lea     esi, dword ptr [edi+28]
> 004026B0   . |895D FC       mov     dword ptr [ebp-4], ebx
> 004026B3   . |5F            pop     edi
> 004026B4   . |EB 02         jmp     short 004026B8
> 004026B6   > |33DB          xor     ebx, ebx                      ;  循环头
> 004026B8   > |8D47 FE       lea     eax, dword ptr [edi-2]
> 004026BB   . |3946 08       cmp     dword ptr [esi+8], eax
> 004026BE   . |72 15         jb      short 004026D5
> 004026C0   . |395E 04       cmp     dword ptr [esi+4], ebx
> 004026C3   . |74 10         je      short 004026D5
> 004026C5   . |8BCE          mov     ecx, esi
> 004026C7   . |E8 EC030000   call    00402AB8
> 004026CC   . |8B46 04       mov     eax, dword ptr [esi+4]
> 004026CF   . |8D4438 FE     lea     eax, dword ptr [eax+edi-2]
> 004026D3   . |EB 05         jmp     short 004026DA
> 004026D5   > |B8 18994A00   mov     eax, 004A9918
> 004026DA   > |8A00          mov     al, byte ptr [eax]            ;  取第 i 位
> 004026DC   . |6A 01         push    1
> 004026DE   . |8845 F3       mov     byte ptr [ebp-D], al
> 004026E1   . |8D45 F3       lea     eax, dword ptr [ebp-D]
> 004026E4   . |53            push    ebx
> 004026E5   . |50            push    eax
> 004026E6   . |8D4D DC       lea     ecx, dword ptr [ebp-24]
> 004026E9   . |E8 48030000   call    00402A36                      ;  此 Call 用于找出第 i 位试炼码出现于固定字符串的位置
> 004026EE   . |8945 EC       mov     dword ptr [ebp-14], eax
> 004026F1   . |8D47 FF       lea     eax, dword ptr [edi-1]
> 004026F4   . |3946 08       cmp     dword ptr [esi+8], eax
> 004026F7   . |72 15         jb      short 0040270E
> 004026F9   . |395E 04       cmp     dword ptr [esi+4], ebx
> 004026FC   . |74 10         je      short 0040270E
> 004026FE   . |8BCE          mov     ecx, esi
> 00402700   . |E8 B3030000   call    00402AB8
> 00402705   . |8B46 04       mov     eax, dword ptr [esi+4]
> 00402708   . |8D4438 FF     lea     eax, dword ptr [eax+edi-1]
> 0040270C   . |EB 05         jmp     short 00402713
> 0040270E   > |B8 18994A00   mov     eax, 004A9918
> 00402713   > |8A00          mov     al, byte ptr [eax]            ;  取第 i+1 位
> 00402715   . |6A 01         push    1
> 00402717   . |8845 F3       mov     byte ptr [ebp-D], al
> 0040271A   . |8D45 F3       lea     eax, dword ptr [ebp-D]
> 0040271D   . |53            push    ebx
> 0040271E   . |50            push    eax
> 0040271F   . |8D4D DC       lea     ecx, dword ptr [ebp-24]
> 00402722   . |E8 0F030000   call    00402A36                      ;  还是找位置，记第 i 位的位置为 A，第 i+1 位的为 B
> 00402727   . |8BD8          mov     ebx, eax
> 00402729   . |8B45 EC       mov     eax, dword ptr [ebp-14]
> 0040272C   . |3BC3          cmp     eax, ebx
> 0040272E   . |0F84 B3000000  je      004027E7                      ;  两个试炼码在固定字符串中对应的位置不可以相同，即每一组的两个试炼码不可以相同
> 00402734   . |2BC3          sub     eax, ebx                      ;  A-B
> 00402736   . |6A 24         push    24
> 00402738   . |99            cdq
> 00402739   . |59            pop     ecx
> 0040273A   . |F7F9          idiv    ecx
> 0040273C   . |52            push    edx                           ;  (A-B) mod 24h -> EDX
> 0040273D   . |E8 C9230600   call    00464B0B                      ;  此 Call 用于实现：if a-b<0 then edx=b-a
> 00402742   . |8A80 D8B64C00 mov     al, byte ptr [eax+4CB6D8]     ;  固定字符串[EDX] -> AL
> 00402748   . |6A 24         push    24
> 0040274A   . |8845 F3       mov     byte ptr [ebp-D], al
> 0040274D   . |8B45 EC       mov     eax, dword ptr [ebp-14]
> 00402750   . |03C3          add     eax, ebx                      ;  A+B
> 00402752   . |59            pop     ecx
> 00402753   . |99            cdq
> 00402754   . |F7F9          idiv    ecx
> 00402756   . |52            push    edx                           ;  (A+B) mod 24h -> EDX
> 00402757   . |E8 AF230600   call    00464B0B
> 0040275C   . |8A80 D8B64C00 mov     al, byte ptr [eax+4CB6D8]     ;  固定字符串[EDX] -> AL
> 00402762   . |397E 08       cmp     dword ptr [esi+8], edi
> 00402765   . |59            pop     ecx
> 00402766   . |8845 F2       mov     byte ptr [ebp-E], al
> 00402769   . |59            pop     ecx
> 0040276A   . |72 14         jb      short 00402780
> 0040276C   . |837E 04 00    cmp     dword ptr [esi+4], 0
> 00402770   . |74 0E         je      short 00402780
> 00402772   . |8BCE          mov     ecx, esi
> 00402774   . |E8 3F030000   call    00402AB8
> 00402779   . |8B46 04       mov     eax, dword ptr [esi+4]
> 0040277C   . |03C7          add     eax, edi
> 0040277E   . |EB 05         jmp     short 00402785
> 00402780   > |B8 18994A00   mov     eax, 004A9918
> 00402785   > |0FBE00        movsx   eax, byte ptr [eax]           ;  试炼码的第 i+2 位
> 00402788   . |0FB64D F3     movzx   ecx, byte ptr [ebp-D]         ;  (A-B)那次运算的结果
> 0040278C   . |3BC1          cmp     eax, ecx                      ;  If sn[i+2]=固定字符串[(A-B) mod 24h]?
> 0040278E   . |75 57         jnz     short 004027E7
> 00402790   . |8D5F 01       lea     ebx, dword ptr [edi+1]
> 00402793   . |395E 08       cmp     dword ptr [esi+8], ebx
> 00402796   . |72 12         jb      short 004027AA
> 00402798   . |837E 04 00    cmp     dword ptr [esi+4], 0
> 0040279C   . |74 0C         je      short 004027AA
> 0040279E   . |8BCE          mov     ecx, esi
> 004027A0   . |E8 13030000   call    00402AB8
> ```



```asm
004027A5   . |035E 04       add     ebx, dword ptr [esi+4]  
004027A8   . |EB 05        jmp     short 004027AF  
004027AA   > |BB 18994A00  mov     ebx, 004A9918  
004027AF   > |0FBE03       movsx   eax, byte ptr [ebx]           ;  这是另一位的处理  
004027B2   . |0FB64D F2    movzx   ecx, byte ptr [ebp-E]  
004027B6   . |3BC1         cmp     eax, ecx  
004027B8   . |75 2D       jnz     short 004027E7  
004027BA   . |83C7 04     add     edi, 4  
004027BD   . |8D47 FE     lea     eax, dword ptr [edi-2]  
004027C0   . |83F8 14     cmp     eax, 14  
004027C3   .^|0F8C EDFEFFFF jl    004026B6                       ;  循环尾  
004027C9   . |6A 01       push    1  
004027CB   . |5E          pop     esi  
004027CC   > |6A 01       push    1  
004027CE   . |8D4D DC     lea     ecx, dword ptr [ebp-24]  
004027D1   . |E8 03FAFFFF call    004021D9  
004027D6   . |8BC6        mov     eax, esi  
004027D8   . |5E          pop     esi  
004027D9   . |5B          pop     ebx  
004027DA   > |8B4D F4     mov     ecx, dword ptr [ebp-C]  
004027DD   . |5F          pop     edi  
004027DE   . |64:890D 00000>mov   dword ptr fs:[0], ecx  
004027E5   . |C9          leave  
004027E6   . |C3          retn  
```

算法总结：

1. 把注册码分解成 4×5 的结构。
2. 在固定字符串中搜索每一个 4 字组的第 1、2 位字符的位置，记为`a`、`b`。
3. 令`c = (a - b) mod 24h` (if `a - b < 0` then `c = b - a`)、`d = (a + b) mod 24h`
4. 取固定字符串第`c`、`d`个字符，此两字符应与注册码 4 字组的第 3、4 位相同。

注册机代码如下：

```vba
Const str = "HJLMCAKN3P4OBUID05F6EVSZ2Q1TRG7YWX98"

Private Sub CmdGen_Click()

    TxtSN.Text = PartGen & PartGen & PartGen & PartGen & PartGen

End Sub

Function PartGen() As String

    Randomize Timer

    Dim a As Byte, b As Byte

    1 a = Int(36 * Rnd + 1): b = Int(36 * Rnd + 1): If a = b Then GoTo 1

    If a > b Then c = (a - b) Mod 36 Else c = (b - a) Mod 36
    d = (a + b) Mod 36

    PartGen = Mid(str, a + 1, 1) + Mid(str, b + 1, 1) + Mid(str, c + 1, 1) + Mid(str, d + 1, 1)

End Function
```

【经验总结】

算法比较简单……

2007 年 08 月 26 日 2:36:47

`4dd466d0e1e0cc123afbe504da2462ca`
