
```
0042D7D5    2BC1            sub     eax, ecx                  ;这个很熟悉了吧..
0042D7D7    60              pushad
0042D7D8    E8 06000000     call    0042D7E3
0042D7DD    8B6424 08       mov     esp, dword ptr [esp+8]
0042D7E1    EB 1A           jmp     short 0042D7FD
0042D7E3    64:67:FF36 0000 push    dword ptr fs:[0]
0042D7E9    64:67:8926 0000 mov     dword ptr fs:[0], esp
0042D7EF    9C              pushfd
0042D7F0    810C24 00010000 or      dword ptr [esp], 100
0042D7F7    9D              popfd
0042D7F8    F8              clc
0042D7F9  ^ 73 DC           jnb     short 0042D7D7
0042D8B3    F8              clc
0042D8B4    72 7D           jb      short 0042D933
0042D8B6    51              push    ecx
0042D8B7    8BCB            mov     ecx, ebx
0042D8B9    E3 03           jecxz   short 0042D8BE
0042D8BB    59              pop     ecx
0042D8BC  ^ EB AD           jmp     short 0042D86B
0042D8BE    59              pop     ecx                   ;解码,没什么意义直接 F4
0042D791    F3:AA           rep     stos byte ptr es:[edi]
0042D793    66:AB           stos    word ptr es:[edi]
0042D795    EB 02           jmp     short 0042D799      这里,再 F7,几步直接到达,oep
0042D799    61              popad
0042D79A  - FF6424 D0       jmp     dword ptr [esp-30]      ; wuliao11.004017B0..到 oep 了
```

总结，只要过前面一段 `seh`，也就是那个脚本。

然后在 `GetProcAddress` 下断点，然后在 `code` 下断点，就行了。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `wuliao1111.rar` |
| :--- | :--- |
