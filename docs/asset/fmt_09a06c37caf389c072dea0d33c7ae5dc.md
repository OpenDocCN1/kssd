
```
0060160C    55               push    ebp  
0060160D    8BEC             mov     ebp, esp  
0060160F    83C4 F4          add     esp, -0C  
00601612    53               push    ebx  
00601613    56               push    esi  
00601614    57               push    edi  
00601615    884D F7          mov     byte ptr [ebp-9], cl          ; cl 值赋予 ebp-9  
00601618    8955 F8          mov     dword ptr [ebp-8], edx        ; 参与计算数保存于 ebp-8  
0060161B    8945 FC          mov     dword ptr [ebp-4], eax        ; 程序名字符串地址保存于 ebp-4  
0060161E    8B45 FC          mov     eax, dword ptr [ebp-4]  
00601621    E8 363BE0FF      call    0040515C  
00601626    33C0             xor     eax, eax  
00601628    55               push    ebp  
00601629    68 08176000      push    00601708  
0060162E    64:FF30          push    dword ptr fs:[eax]  
00601631    64:8920          mov     dword ptr fs:[eax], esp  
00601634    33C0             xor     eax, eax  
00601636    55               push    ebp  
00601637    68 D6166000      push    006016D6  
0060163C    64:FF30          push    dword ptr fs:[eax]  
0060163F    64:8920          mov     dword ptr fs:[eax], esp  
00601642    33DB             xor     ebx, ebx  
00601644    837D FC 00       cmp     dword ptr [ebp-4], 0  
00601648    74 70            je      short 006016BA  
0060164A    8B45 FC          mov     eax, dword ptr [ebp-4]  
0060164D    E8 1A39E0FF      call    00404F6C  
00601652    8BF0             mov     esi, eax                      ; 字符串长度  
00601654    85F6             test    esi, esi  
00601656    7E 3B            jle     short 00601693  
00601658    B9 01000000      mov     ecx, 1  
0060165D    807D F7 00       cmp     byte ptr [ebp-9], 0           ; 这里由[ebp-9]，即 cl 是否为 0，进入 2 种不同算法 saunfa1 和 suanfa2  
00601661    74 15            je      short 00601678                第一种算法 suanfa1 开始  
00601663    8BC1             mov     eax, ecx  
00601665    F7E8             imul    eax                           ; eax 自乘  
00601667    03C0             add     eax, eax                       ; eax 自加  
00601669    40               inc     eax                           ; eax 增 1  
0060166A    8B55 FC          mov     edx, dword ptr [ebp-4]        ; 字符串地址送入 edx  
0060166D    0FB6540A FF      movzx   edx, byte ptr [edx+ecx-1]     ; 字符串逐位扩展送入 edx  
00601672    F7EA             imul    edx                           ; eax=eax*edx  
00601674    03D8             add     ebx, eax                      ; ebx=ebx+eax,循环之和 sum 保存于 ebx  
00601676    EB 17            jmp     short 0060168F  
00601678    8B45 FC          mov     eax, dword ptr [ebp-4]        ; 第二种算法 saunfa2 开始  
0060167B    0FB64408 FF      movzx   eax, byte ptr [eax+ecx-1]     ; 字符串逐位扩展送入 eax  
00601680    F7E8             imul    eax                           ; eax 自乘  
00601682    0FBFC0           movsx   eax, ax  
00601685    BF A0860100      mov     edi, 186A0                    ; edi=186A0h  
0060168A    99               cdq  
0060168B    F7FF             idiv    edi                           ; eax/edi  
0060168D    03DA             add     ebx, edx                      循环之和 sum 保存于 ebx  
0060168F    41               inc     ecx                           ; ecx 增 1，控制取字符串字符的位置  
00601690    4E               dec     esi                           ; esi 减 1，控制循环次数  
00601691    75 CA            jnz     short 0060165D  
00601693    8BC3             mov     eax, ebx                      ; 字符串逐位计算后的值 sum,送入 eax  
00601695    B9 10270000      mov     ecx, 2710                     ; ecx=2710h  
0060169A    99               cdq                                   ; edx 清零  
0060169B    F7F9             idiv    ecx                           ; eax/ecx,商保存于 eax，余数保存在 edx  
0060169D    8BC2             mov     eax, edx                      ; 余数送入 eax  
0060169F    F7E8             imul    eax                           ; eax 自乘  
006016A1    0345 F8          add     eax, dword ptr [ebp-8]        ; eax=eax+[ebp-8]  
006016A4    8BD8             mov     ebx, eax  
006016A6    8BC3             mov     eax, ebx  
006016A8    B9 A0860100      mov     ecx, 186A0                    ; ecx=186A0h  
006016AD    99               cdq                                   ; edx 清零  
006016AE    F7F9             idiv    ecx                           ; eax/ecx  
006016B0    81C2 A0860100    add     edx, 186A0                    ; edx=余数+186A0h  
006016B6    8BDA             mov     ebx, edx                      ; 上面计算值 sn 送入 ebx  
006016B8    EB 12            jmp     short 006016CC  
006016BA    807D F7 00       cmp     byte ptr [ebp-9], 0  
006016BE    74 07            je      short 006016C7  
006016C0    BB 6E740300      mov     ebx, 3746E  
006016C5    EB 05            jmp     short 006016CC  
006016C7    BB 01EB0B00      mov     ebx, 0BEB01  
006016CC    33C0             xor     eax, eax  
006016CE    5A               pop     edx  
006016CF    59               pop     ecx  
006016D0    59               pop     ecx  
006016D1    64:8910          mov     dword ptr fs:[eax], edx  
006016D4    EB 1C            jmp     short 006016F2  
006016D6    E9 292CE0FF      jmp     00404304  
006016DB    807D F7 00       cmp     byte ptr [ebp-9], 0  
006016DF    74 07            je      short 006016E8  
006016E1    BB 6E740300      mov     ebx, 3746E  
006016E6    EB 05            jmp     short 006016ED  
006016E8    BB 01EB0B00      mov     ebx, 0BEB01  
006016ED    E8 7A2FE0FF      call    0040466C  
006016F2    33C0             xor     eax, eax  
006016F4    5A               pop     edx  
006016F5    59               pop     ecx  
006016F6    59               pop     ecx  
006016F7    64:8910          mov     dword ptr fs:[eax], edx  
006016FA    68 0F176000      push    0060170F  
006016FF    8D45 FC          lea     eax, dword ptr [ebp-4]  
00601702    E8 9535E0FF      call    00404C9C  
00601707    C3               retn  
00601708    E9 AB2EE0FF      jmp     004045B8  
0060170D    EB F0            jmp     short 006016FF  
0060170F    8BC3             mov     eax, ebx                      ; sn 又传入 eax  
00601711    5F               pop     edi  
00601712    5E               pop     esi  
00601713    5B               pop     ebx  
00601714    8BE5             mov     esp, ebp  
00601716    5D               pop     ebp  
00601717    C3               retn  
```

## 二、算法总结

注册码由 2 部分连接而成，每一部分经过 2 次运算而成。4 次运算调用同一个算法 CALL（`call 0060160C`），这个算法 CALL 内部由传入参数 `cl` 值的不同，又分为 2 种算法，`suanfa1` 和 `suanfa2`。

### 1、注册码第一部分

- `a`、固定字符 `"3"` 由 `suanfa2` 运算后的值设为 `sn2`（`cl=0`，参与计算数 `0xBEB01`）；
- `b`、用户名 `name`，由算法 2 `suanfa2` 计算得到 `sn4`（`cl=0`，参与计算数 `sn2`）；
- `c`、`sn4` 转化为字符串 `code`；
- `d`、取 `code` 后 5 位后得到字符串 `code1`，即注册码第二部分；

### 2、注册码第二部分

- `a`、程序名 `"2nd Speech Cente"` 由 `suanfa1` 计算得到 `sn1`（`cl=1`，参与计算数为 `0x3746E`）；
- `b`、用户名 `name`，由算法 1 `suanfa1` 计算得到 `sn3`（`cl=1`，参与计算数为 `sn1`）；
- `c`、`sn3` 转化为字符串 `reg`；
- `d`、取 `reg` 前 5 位后得到字符串 `reg1`，即注册码第二部分；

### 3、组合

第 1 和第二部分由 `"-"` 连接得到对应用户名的注册码。

为缩短篇幅 `suanfa1` 和 `suanfa2` 可以参照注释和下面注册机部分。

## 三、C 语言注册机源代码

```c
#include "stdio.h"
```
