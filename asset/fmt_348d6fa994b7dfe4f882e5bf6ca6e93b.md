
```
01004F38  |.  8B45 6C       |||mov     eax, dword ptr [ebp+6C]
01004F3B  |.  3B45 60       |||cmp     eax, dword ptr [ebp+60]
01004F3E  |.  7E 10         |||jle     short 01004F50
01004F40  |.  85FF         |||test    edi, edi
01004F42  |.  7E 0C         |||jle     short 01004F50
01004F44  |.  8BC7         |||mov     eax, edi
01004F46  |.  F7D8         |||neg     eax
01004F48  |.  C1E0 03      |||shl     eax, 3
01004F4B  |.  99           |||cdq
01004F4C  |.  F7F9         |||idiv    ecx
01004F4E  |.  8BF8         |||mov     edi, eax
01004F50  |>  E8 EEFDFFFF  |||call    01004D43
01004F55  |.  85C0         |||test    eax, eax
01004F57  |.  75 2E        |||jnz     short 01004F87
01004F59  |.  8B45 68      |||mov     eax, dword ptr [ebp+68]
01004F5C  |>  8B0D 08730001|| mov     ecx, dword ptr [1007308]
01004F62  |.  F7D9         |||neg     ecx
01004F64  |.  3BC1         |||cmp     eax, ecx
01004F66  |.^ 7F AA       ||\jg      short 01004F12
01004F68  |>  FF45 70      ||inc     dword ptr [ebp+70]
01004F6B  |.  8345 64 04   ||add     dword ptr [ebp+64], 4
01004F6F  |.  837D 70 06   ||cmp     dword ptr [ebp+70], 6
01004F73  |.^ 0F8C 4FFFFFFF|\jl      01004EC8
01004F79  |.  836D 7C 0C   |sub      dword ptr [ebp+7C], 0C
01004F7D  |.  837D 7C 24   |cmp      dword ptr [ebp+7C], 24
01004F81  |.^ 0F8D 34FFFFFF\jge      01004EBB
01004F87  |>  E8 79C4FFFF  call    01001405
01004F8C  |>  68 FF1F0000  push    1FFF                             ; /Arg1 = 00001FFF
01004F91  |.  E8 F80E0000  call    01005E8E                         ; \sol.01005E8E
01004F96  |.  E8 0FD2FFFF  call    010021AA
01004F9B  |.  33C0         xor     eax, eax
01004F9D  |.  50           push    eax                             ; /Arg4 => 00000000
01004F9E  |.  50           push    eax                             ; |Arg3 => 00000000
01004F9F  |.  6A 0D        push    0D                              ; |Arg2 = 0000000D
01004FA1  |.  53           push    ebx                             ; |Arg1
01004FA2  |.  A3 CC710001  mov     dword ptr [10071CC], eax        ; |
01004FA7  |.  E8 4DE1FFFF  call    010030F9                         ; \sol.010030F9
01004FAC  |.  5F           pop     edi
01004FAD  |.  5E           pop     esi
01004FAE  |.  5B           pop     ebx
01004FAF  |.  83C5 74      add     ebp, 74
01004FB2  |.  C9           leave
01004FB3  \.  C2 0400      retn    4
```

关键代码就在这里了，我就不分析了（有点吃力哦），当然另外一个回调函数也不必看了。看下胜利截图：

![](img/5e35467dc12420e664476a884f4ba665.jpg)
