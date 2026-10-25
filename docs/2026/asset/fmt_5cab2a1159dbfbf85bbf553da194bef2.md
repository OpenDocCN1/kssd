
```assembly
mov    dl, [ebp-5]
004B0BC4            call    sub_403EE4
004B0BC9              mov    edx, [ebp-10h]
004B0BCC                lea    eax, [esi+5ECh]
004B0BD2    call    sub_403FC4
004B0BD7 loc_4B0BD7:
mov    eax, [esi+5ECh]
004B0BDD      call    sub_403FBC
004B0BE2          cmp    eax, 0Ah
004B0BE5              jge    short loc_4B0BF5
004B0BE7                mov    eax, [esi+5ECh]
004B0BED    call    sub_403FBC
004B0BF2        dec    eax
004B0BF3          jg      short loc_4B0B97
004B0BF5 loc_4B0BF5:    lea    eax, [esi+5ECh]
004B0BFB              mov    edx, 0Ah
004B0C00                call    sub_4042F0
004B0C05    lea    edx, [ebp-14h]
004B0C08      mov    eax, [esi+5ECh]
004B0C0E            call    sub_4097F0
004B0C13              mov    edx, [ebp-14h]
004B0C16                lea    eax, [esi+5ECh]
004B0C1C    call    sub_403D90
004B0C21        lea    eax, [ebp-4]
004B0C24              mov    edx, [esi+5E0h]
004B0C2A                call    sub_403DD4              ;上面一大段算法把strname1转为10位的strname2
004B0C2F                mov    byte ptr [esi+60Ch], 1
004B0C36      mov    edi, 1
004B0C3B loc_4B0C3B:
cmp    byte ptr [esi+60Ch], 0
004B0C42        jz      short loc_4B0C60
004B0C44              mov    eax, [esi+5ECh]        ;strname2 ，len=10
004B0C4A          mov    al, [eax+edg-1]      ;从前往后正向取
004B0C4E      mov    edx, 0Bh
004B0C53          sub    edx, edi
004B0C55              mov    ecx, [ebp-4]          ;strcode2 ，len=10
004B0C58            mov    dl, [ecx+edw-1]        ;从后往前反向取
004B0C5C        xor    al,dl            ;比较＝？
004B0C5E          jz      short loc_4B0C64
004B0C60 loc_4B0C60:    xor    eax, eax
004B0C62              jmp    short loc_4B0C66
004B0C64 loc_4B0C64:    mov    al, 1
004B0C66 loc_4B0C66:
  mov    [esi+60Ch], al
004B0C6C        inc    edi
004B0C6D          cmp    edi, 0Bh
004B0C70              jnz    short loc_4B0C3B
004B0C72                jmp    short loc_4B0C9E      ;比较结束，注册成功标志是byte ptr [esi+60Ch]＝1
```

====================================================================================

文章写的不太好，请见谅，谢谢您有兴趣看完

最后给出一组注册码，以便于大家跟踪分析

用户名：heXer

注册码：KSHPNBY7S7

====================================================================================

heXer/iPB

2002.06.15
