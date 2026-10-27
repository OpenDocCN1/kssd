
```asm
004D8AA3    EB 1E           jmp short SoftDefe.004D8AC3

004D8AA5    8B5424 10       mov edx, dword ptr ss:[esp+10]

004D8AA9    57              push edi

004D8AAA    52              push edx

004D8AAB    E8 35EAFFFF     call SoftDefe.004D74E5

004D8AB0    50              push eax

004D8AB1    E8 B3F8FFFF     call SoftDefe.004D8369

004D8AB6    EB 0B           jmp short SoftDefe.004D8AC3

004D8AB8    8B4424 10       mov eax, dword ptr ss:[esp+10]

004D8ABC    57              push edi

004D8ABD    50              push eax

004D8ABE    E8 22EAFFFF     call SoftDefe.004D74E5

004D8AC3    33C9            xor ecx, ecx

004D8AC5    8906            mov dword ptr ds:[esi], eax    ;这里就写入了
```

所以要想办法让它在那能断下来，在

`004D81E2`    `8B4C24 0C`       `mov ecx, dword ptr ss:[esp+C]`   ;Stop here

`004D81E6`    `8B5424 08`       `mov edx, dword ptr ss:[esp+8]`

`004D81EA`    `2BCA`            `sub ecx, edx`

`004D81EC`    `81F9 401F0000`   `cmp ecx, 1F40`

`004D81F2`    `72 66`           `jb short SoftDefe.004D825A`   ;一定要跳！

跳完以后，如果在`004D88EF`处直接下硬执行断点有问题，估计和OD本身有关，可能是个Bug，要先在`0044E000`处下内存断点，`F9`，会断下，之后按4次`F9`后再在`004D88EF`处下硬执行断点，再`F9`，等断在`004D88EF`时，修改`[esp+440]`的值为0，取消硬断点以及内存断点，再`g 42966d`，如果程序停住了就暂停一下再继续执行，就能到OEP了，这个时候就可以用ImportREC来修复了，一个坏的都没有，是不是很爽啊 :lol:

OK,Fix Dump吧，然后优化一下
