
# 汇编代码片段

```
:0040E3AE 0F854B010000             jne 0040E4FF   

:0040E3B4 8A5114                   mov dl, byte ptr [ecx+14]    /输入 Product 的第 15 位到 DL 中   

:0040E3B7 56                       push esi                     /弹出的是？？？   

:0040E3B8 8B74240C                 mov esi, dword ptr [esp+0C]   

:0040E3BC 0FBEC2                   movsx eax, dl                /DL=>EAX   

:0040E3BF 81E6FF000000             and esi, 000000FF            /正确的 Product 到 ESI 中，   

:0040E3C5 3BC6                     cmp eax, esi                 /第 15 位正确与否！   

:0040E3C7 5E                       pop esi                      /压入的是？？？   

:0040E3C8 0F8531010000             jne 0040E4FF                 /不能跳走！   

    .   

    .   

    .
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0040E4C9(C)` |

```
:0040E4D2 0FBE511D                 movsx edx, byte ptr [ecx+1D]   

:0040E4D6 8B442404                 mov eax, dword ptr [esp+04]   

:0040E4DA 25FF000000             and eax, 000000FF   

:0040E4DF 3BD0                     cmp edx, eax   

:0040E4E1 751C                     jne 0040E4FF   

:0040E4E3 0FBE491E                 movsx ecx, byte ptr [ecx+1E]   

:0040E4E7 8B542408                 mov edx, dword ptr [esp+08]   

:0040E4EB 33C0                     xor eax, eax                 

:0040E4ED 81E2FF000000             and edx, 000000FF   

:0040E4F3 5B                       pop ebx   

:0040E4F4 3BCA                     cmp ecx, edx                 /至此可找出全部的 Product。输入后成功，也不知道还有没有校验。   

:0040E4F6 0F94C0                   sete al   

:0040E4F9 83C408                   add esp, 00000008   

:0040E4FC C20400                   ret 0004   

```

`8086[CCG]` 17:16 2001-7-10

***

`4dd466d0e1e0cc123afbe504da2462ca`
