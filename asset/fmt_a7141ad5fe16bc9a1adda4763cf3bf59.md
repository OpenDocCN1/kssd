
```asm
    mov ebx, ecx   
    shr edi, 05   
    shl ebx, 04   
    xor edi, ebx   
    add edi, ecx   
    mov ebx, eax   
    and ebx, 0x3   
    mov ebx, dword ptr [esp+4*ebx+0x0C]   
    add ebx, eax   
    xor edi, ebx   
    add edx, edi       <- 关键位置   
    sub eax, 0x61C88647   <- 关键位置   
    mov edi, edx   
    mov ebx, edx   
    shr edi, 05   
    shl ebx, 04   
    xor edi, ebx   
    add edi, edx   
    mov ebx, eax   
    shr ebx, 0x0B   
    and ebx, 0x3   
    mov ebx, dword ptr [esp+4*ebx+0x0C]   
    add ebx, eax   
    xor edi, ebx   
    add ecx, edi       <- 关键位置   
    dec esi   
    jnz loop1   

exit1:   
    pop edi   
    pop esi   
    pop ebx   
    add esp, 00000030   
```

OK，经过这个循环，`edx` = 705987235，`ecx` = 970065253。好了重新注册，输入名字：`43214321`，注册码：`705987235-970065253`，Coolll! "Thank you for...."。

其实被这个逆算法折磨很长时间，一开始想偷懒，只输入一个数字怎样都不能成功，原因在于 `edx` 的值可知，而 `ecx` 相当于是随机的（就是未知的一个值），后来偶然想到 `'-'` 的用处，才成功。

关于注册码：名字可以是为少于、等于 8 个的任意字符，若长度大于 8 则会失败；保存在同一目录下的 `NewsReactor.ini`,

`[Registration  4dd466d0e1e0cc123afbe504da2462ca`
