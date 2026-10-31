
# [原创]压缩壳之 Pohernah壳分析

## Pohernah壳 加密算法分析

关于Pohernah壳的脱壳已经完成了，见链接<http://bbs.pediy.com/showthread.php?t=75219>, 由于很多事情，脱壳机一直拖着没有写完，现在对于这个的加壳流程和加密算法一探究竟。

首先，是入口

```
00469000 >  60              pushad
00469001    E8 00000000     call    00469006 ；F7步入，在壳中常见
00469006    5D              pop     ebp
00469007    81ED F1264000   sub     ebp, 004026F1
0046900D    8BBD 18284000   mov     edi, dword ptr ss:[ebp+402818]
00469013    8B8D 20284000   mov     ecx, dword ptr ss:[ebp+402820]
0046901E    01E8            add     eax, ebp
00469020    8030 05         xor     byte ptr ds:[eax], 5
00469023    83F9 00         cmp     ecx, 0

; 这层有一个循环是用来判断是否读完区段的，循环是以区块数为循环层数，每次循环减一，也就是说这个壳是对每一个区块进行加密的
00469026    74 71           je      short 00469099   ; if ecx ==0 jump
00469031    8B57 0C         mov     edx, dword ptr ds:[edi+C]        ;
00469034    0395 1C284000   add     edx, dword ptr ss:[ebp+<dword_40>
; .text的地址，也就是第一个区块的地址
0046903A    31C0            xor     eax, eax                         ; eax = 0
0046903C    51              push    ecx
0046903D    31C9            xor     ecx, ecx                         ; ecx = 0
0046903F    66:B9 FA00      mov     cx, 0FA                          ; ecx = FA
00469043    66:83F9 00      cmp     cx, 0                            ; if ecx ! = 0
00469047    74 49           je      short 00469092
00469049    8B57 0C         mov     edx, dword ptr ds:[edi+C]        ; 将基地址赋给edx
0046904C    0395 1C284000   add     edx, dword ptr ss:[ebp+<dword_40>; edx = 401000
00469052    8B85 24284000   mov     eax, dword ptr ss:[ebp+<dword_40>;
00469058    83F8 02         cmp     eax, 2                           ; if eax = = 2
0046905B    75 06           jnz     short 00469063
0046905D    81C2 00020000   add     edx, 200
00469063    51              push    ecx
00469064    8B4F 10         mov     ecx, dword ptr ds:[edi+10]       ; ecx = FA --> ecx = 00049200
00469067    83F8 02         cmp     eax, 2                           ; if ecx == 2
0046906A    75 06           jnz     short 00469072
0046906C    81E9 00020000   sub     ecx, 200
00469072    57              push    edi                              ; .text
00469073    BF C8000000     mov     edi, 0C8                         ; edi = 0c8
00469078    89CE            mov     esi, ecx                      ; esi = ecx =49200
0046907A    E8 27000000     call    004690A6
0046907F    89C1            mov     ecx, eax          ; 49200  .text 的原始大小
00469081    5F              pop     edi                              ; .text
00469082    B8 38284000     mov     eax, <dword_402838>              ;
00469087    01E8            add     eax, ebp        ; eax = 402838 ebp = 66915!!!!!!!!
00469089    E8 24000000     call    004690B2
{
    004690B2    60              pushad
004690B3    83F0 05         xor     eax, 5
004690B6    40              inc     eax
004690B7    90              nop
004690B8    48              dec     eax
004690B9    83F0 05         xor     eax, 5                        ;eax = 46914D 就是加密用的key ，后发现这个每次没有变化？
004690BC    89C6            mov     esi, eax                     ; esi 是 key
004690BE    89D7            mov     edi, edx
004690C0    60              pushad
004690C1    E8 0B000000     call    004690D1                    ；这个就是解密函数了
{
    push    edi
    mov     ebx, [edi]      ; y = v[0] edi .text的基地址
    mov     ecx, [edi+4]    ; z = v [1]
    push    9E3779B9h
    pop     edx
    inc     edx             ; edx =0x9e3779ba
    mov     eax, edx
    dec     eax             ; eax = 0x9e3779b9   ----delta
    shl     eax, 5          ; sum = delta<<5
    mov     edi, 20h        ; while(n -- >0)  edi = 20h  32轮循环解密  n=32
    dec     edx             ; edx = 0x9e3779b9
    mov     ebp, ebx
    shl     ebp, 4          ; y<<4
    sub     ecx, ebp        ; v[1] - y
    mov     ebp, [esi+8]    ; ebp = k [2]
    xor     ebp, ebx        ; k [2] ^ v[0]
    sub     ecx, ebp        ; (v [1] - y-(k [2] ^ v[0]) )
    mov     ebp, ebx
    shr     ebp, 5          ; v [0] >>5
    xor     ebp, eax        ; v [0] >> 5 ^ sum
    sub     ecx, ebp        ; ((v [1] - y-(k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum)
    sub     ecx, [esi+0Ch]  ; ((v [1] - (k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum) - v [3]
    mov     ebp, ecx        ; ebp = (((v [1] - y - (k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum) - v [3])
    shl     ebp, 4          ; (((v [1] - y - (k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum) - v [3])<<4
    sub     ebx, ebp        ; v [0] -((((v [1] - y -(k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum) - v [3])<<4)
    mov     ebp, [esi]      ; v[0]
    xor     ebp, ecx        ; v [0] ^((v [1] - y) - ((v[0]>>5) ^ sum) - v [3])
    sub     ebx, ebp        ; v [0] -((((v [1] -y - (k [2] ^ v[0]) )) - ((v[0]>>5) ^ sum) - v [3])<<4) -v [0] ^((v [1] - y) - ((v[0]>>5) ^ sum) - v [3])
    mov     ebp, ecx        ; ebp = (v [1] - y) - ((v[0]>>5) ^ sum) - v [3]
    shr     ebp, 5          ; ((v [1] - y) - ((v[0]>>5) ^ sum) - v [3])>>5
    xor     ebp, eax        ; (((v [1] - y) - ((v[0]>>5) ^ sum) - v [3])>>5)  ^ sum
    sub     ebx, ebp        ;
    sub     ebx, [esi+4]    ;
    sub     eax, edx        ; sum - = telta
    dec     edi             ; n --
    jnz     short loc_4690EA
    pop     edi
    mov     [edi], ebx      ; v[0] = y
    mov     [edi+4], ecx    ; v[1]=z
    retn                    ; 一轮加密结束
}
004690C6    61              popad
004690C7    83C7 08         add     edi, 8                   ；读取后面8个字节，因为DWORD类型，一次读两个
004690CA    83E9 07         sub     ecx, 7
004690CD  ^ E2 F1           loopd   short 004690C0         ;循环，进行解密
004690CF    61              popad
004690D0    C3              retn
}
0046908E    59              pop     ecx
0046908F    49              dec     ecx
00469090  ^ EB B1           jmp     short 00469043
00469092    59              pop     ecx
00469093    83C7 28         add     edi, 28                          ; .text + 8
00469096    49              dec     ecx       ; ecx = 5 代表5个区段
00469097  ^ EB 8A           jmp     short 00469023
00469099    8B85 14284000   mov     eax, dword ptr ss:[ebp+<dword_40>
0046909F    894424 1C       mov     dword ptr ss:[esp+1C], eax       ; eax 为OEP
004690A3    61              popad
```

到这里，壳的整个流程就走了一遍，壳的流程就是

![](https://bbs.kanxue.com/upload/attach/200811/90551_e0u4s62uot4xeeq.jpg)

因此我们的脱壳机的思路，就是将文件映射到内存，然后 `for (DWORD i = 0; i < dwSectionNum; i++)` 进行循环解密。

然后在循环中调用 `DeCrypty` 函数，将数据解密出来，更改OEP 地址，重建文件… …

```
void DeCrypt(DWORD* v,DWORD* k)
{
    DWORD y=v[0],z=v[1],sum,delta=0x9e3779b9,n=32;
    sum=delta<<5;
    while (n-->0)
    {
        z-=((y<<4)+k[2])^(y+sum)^((y>>5)+k[3]);
        y-=((z<<4)+k[0])^(z+sum)^((z>>5)+k[1]);
        sum-=delta;
    }
    v[0]=y;
    v[1]=z;
}
```

脱壳机写完后传到附件

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [liucheng.JPG](javascript:void(0);)

  （13.70kb，192次下载）
