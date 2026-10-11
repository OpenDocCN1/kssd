
### 2. 汇编代码分析

```
.text:004081A5                   test    eax, eax        ; 判断计数是否为 0

.text:004081A7                   jz      short retnull   ; 计数为 0 则直接返回

.text:004081A9                   mov     edx, [esp+4+Buf] ; edx=buffer 地址

.text:004081AD                   xor     ebx, ebx        ; ebx=0

.text:004081AF                   mov     bl, byte ptr [esp+4+Val] ; bl = value 要查询的字符

.text:004081B3                   test    edx, 3          ; buffer 地址是否为 4 字节对齐

.text:004081B9                   jz      short main_loop_start

.text:004081BB

.text:004081BB str_misaligned:                      ; CODE XREF: _memchr+2Bj

.text:004081BB                   mov     cl, [edx]       ; 取 buff 中的字符

.text:004081BD                   inc     edx             ; 地址递增

.text:004081BE                   xor     cl, bl          ; cl==bl 则跳走

.text:004081C0                   jz      short found

.text:004081C2                   dec     eax             ; if (--count) 如果 count=1 则跳到返回位置

.text:004081C3                   jz      short retnull   ; 没有找到则直接返回

.text:004081C5                   test    edx, 3          ; 判断是否对齐，如果已 4 字节对齐则跳到主循环中，否则继续

.text:004081C5                                       ; 循环调整地址

.text:004081CB                   jnz     short str_misaligned

.text:004081CD

.text:004081CD main_loop_start:                       ; CODE XREF: _memchr+19j

.text:004081CD                   sub     eax, 4        ; eax -= 4; 计数减去 4

.text:004081D0                   jb      short tail_less_then_4 ; 小于 4 则跳到比对位置

.text:004081D2                   push    edi          ; 计数大于等于 4 的情况

.text:004081D3                   mov     edi, ebx        ; edi = value

.text:004081D5                   shl     ebx, 8

.text:004081D8                   add     ebx, edi

.text:004081DA                   mov     edi, ebx

.text:004081DC                   shl     ebx, 10h

.text:004081DF                   add     ebx, edi        ; 将 value 扩展成 4 字节

.text:004081E1                   jmp     short main_loop_entry

.text:004081E3 ; ------------------------------------------------------------

.text:004081E3

.text:004081E3 return_from_main:                      ; CODE XREF: _memchr+58j

.text:004081E3                   pop     edi

.text:004081E4

.text:004081E4 tail_less_then_4:                       ; CODE XREF: _memchr+30j

.text:004081E4                   add     eax, 4        ; count -= 4;

.text:004081E4                                       ; count += 4; 如果 zf=1 即表示 count=0 此时函数返回

.text:004081E7                   jz      short retnull

.text:004081E9

.text:004081E9 tail_loop:                             ; CODE XREF: _memchr+51j

.text:004081E9                   mov     cl, [edx]       ; 取字符到 cl

.text:004081EB                   inc     edx             ; 地址递增 CF 不变

.text:004081EC                   xor     cl, bl          ; 判断是否相等

.text:004081EE                   jz      short found

.text:004081F0                   dec     eax             ; 计数递减

.text:004081F1                   jnz     short tail_loop

.text:004081F3

.text:004081F3 retnull:                               ; CODE XREF: _memchr+7j

.text:004081F3                                       ; _memchr+23j ...

.text:004081F3                   pop     ebx           ; 直接返回

.text:004081F4                   retn

.text:004081F5 ; ------------------------------------------------------------

.text:004081F5

.text:004081F5 main_loop:                              ; CODE XREF: _memchr+73j

.text:004081F5                                       ; _memchr+8Bj

.text:004081F5                   sub     eax, 4        ; 判断计数

.text:004081F8                   jb      short return_from_main

.text:004081FA

.text:004081FA main_loop_entry:                        ; CODE XREF: _memchr+41j

.text:004081FA                   mov     ecx, [edx]     ; 取 4 字符到 ecx

.text:004081FC                   xor     ecx, ebx        ; 判断是否为要查询的字符

.text:004081FE                   mov     edi, 7EFEFEFFh  ; 快速定位 0 字符的代码 strlen()中就是这样的

.text:00408203                   add     edi, ecx

.text:00408205                   xor     ecx, 0FFFFFFFFh

.text:00408208                   xor     ecx, edi

.text:0040820A                   add     edx, 4

.text:0040820D                   and     ecx, 81010100h

.text:00408213                   jz      short main_loop ; 发现 0 字符跳走，调用计数小于 4 时的代码

.text:00408215

.text:00408215 char_is_found:                          ; 发现字符的处理

.text:00408215                   mov     ecx, [edx-4]

.text:00408218                   xor     cl, bl

.text:0040821A                   jz      short loc_40823F

.text:0040821C                   xor     ch, bl

.text:0040821E                   jz      short loc_408239

.text:00408220                   shr     ecx, 10h

.text:00408223                   xor     cl, bl

.text:00408225                   jz      short loc_408233

.text:00408227                   xor     ch, bl

.text:00408229                   jz      short loc_40822D

.text:0040822B                   jmp     short main_loop

.text:0040822D ; ------------------------------------------------------------

.text:0040822D

.text:0040822D loc_40822D:                             ; CODE XREF: _memchr+89j

.text:0040822D                   pop     edi

.text:0040822E

.text:0040822E found:                                  ; CODE XREF: _memchr+20j

.text:0040822E                                       ; _memchr+4Ej

.text:0040822E                   lea     eax, [edx-1]    ; eax = buffer - 1 即返回寻找到的地址

.text:00408231                   pop     ebx

.text:00408232                   retn

.text:00408233 ; ------------------------------------------------------------

.text:00408233

.text:00408233 loc_408233:                             ; CODE XREF: _memchr+85j

.text:00408233                   lea     eax, [edx-2]

.text:00408236                   pop     edi

.text:00408237                   pop     ebx

.text:00408238                   retn

.text:00408239 ; ------------------------------------------------------------

.text:00408239

.text:00408239 loc_408239:                             ; CODE XREF: _memchr+7Ej

.text:00408239                   lea     eax, [edx-3]

.text:0040823C                   pop     edi

.text:0040823D                   pop     ebx

.text:0040823E                   retn

.text:0040823F ; ------------------------------------------------------------

.text:0040823F

.text:0040823F loc_40823F:                             ; CODE XREF: _memchr+7Aj

.text:0040823F                   lea     eax, [edx-4]

.text:00408242                   pop     edi

.text:00408243                   pop     ebx

.text:00408244                   retn

.text:00408244 _memchr            endp
```

与前面的类似

### 2. 2. memcmp (memicmp 略)

```
.text:004081E0 arg_buf1         = dword ptr  4
.text:004081E0 arg_buf2         = dword ptr  8
.text:004081E0 arg_size         = dword ptr  0Ch

.text:004081E0

.text:004081E0                   mov     eax, [esp+arg_size] ; arg_ 开头 函数参数与栈帧中的位置对应是一个 90 度顺时针旋转

.text:004081E4                   test    eax, eax        ; 判断计数是否为 0

.text:004081E6                   jz      short retnull   ; 函数直接返回

.text:004081E8                   mov     edx, [esp+arg_buf1] ; edx = buf1

.text:004081EC                   push    esi

.text:004081ED                   push    edi

.text:004081EE                   mov     esi, edx        ; esi = buf1

.text:004081F0                   mov     edi, [esp+8+arg_buf2] ; edi = buf2

.text:004081F4                   or      edx, edi

.text:004081F6                   and     edx, 3          ; 判断 4 字节对齐

.text:004081F9                   jz      short dwords    ; 对齐跳到 dwords 处理
```
