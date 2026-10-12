
```assembly
:00470D06 8A520F             mov dl, byte ptr [edx+0F]   <===取注册码的第 16 位 C
:00470D09 885008             mov byte ptr [eax+08], dl
:00470D0C 8D45FC             lea eax, dword ptr [ebp-04]
:00470D0F E8FC37F9FF         call 00404510
:00470D14 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D17 8A5213             mov dl, byte ptr [edx+13]   <===取注册码的第 20 位 R
:00470D1A 885009             mov byte ptr [eax+09], dl
:00470D1D 8D45FC             lea eax, dword ptr [ebp-04]
:00470D20 E8EB37F9FF         call 00404510
:00470D25 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D28 8A5214             mov dl, byte ptr [edx+14]   <===取注册码的第 21 位 K
:00470D2B 88500A             mov byte ptr [eax+0A], dl
:00470D2E 8D45FC             lea eax, dword ptr [ebp-04]
:00470D31 E8DA37F9FF         call 00404510
:00470D36 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D39 8A5216             mov dl, byte ptr [edx+16]   <===取注册码的第 23 位 R
:00470D3C 88500B             mov byte ptr [eax+0B], dl
:00470D3F 8D45FC             lea eax, dword ptr [ebp-04]
:00470D42 E8C937F9FF         call 00404510
:00470D47 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D4A 8A5219             mov dl, byte ptr [edx+19]   <===取注册码的第 26 位 H
:00470D4D 88500C             mov byte ptr [eax+0C], dl
:00470D50 8D45FC             lea eax, dword ptr [ebp-04]
:00470D53 E8B837F9FF         call 00404510
:00470D58 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D5B 8A521A             mov dl, byte ptr [edx+1A]   <===取注册码的第 27 位 C
:00470D5E 88500D             mov byte ptr [eax+0D], dl
:00470D61 8D45FC             lea eax, dword ptr [ebp-04]
:00470D64 E8A737F9FF         call 00404510
:00470D69 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D6C 8A521B             mov dl, byte ptr [edx+1B]   <===取注册码的第 28 位 C
:00470D6F 88500E             mov byte ptr [eax+0E], dl
:00470D72 8D45FC             lea eax, dword ptr [ebp-04]
:00470D75 E89637F9FF         call 00404510
:00470D7A 8B55E4             mov edx, dword ptr [ebp-1C]
:00470D7D 8A521C             mov dl, byte ptr [edx+1C]   <===取注册码的第 29 位 i
:00470D80 88500F             mov byte ptr [eax+0F], dl
:00470D83 8B45FC             mov eax, dword ptr [ebp-04]  <===EAX=KREKAHUTCRKRHCCi
:00470D86 E861FDFFFF         call 00470AEC          <===这里的 CALL 和上面是一样的
:00470D8B 8945F0             mov dword ptr [ebp-10], eax  <===eax=0x2535022E
:00470D8E 8955F4             mov dword ptr [ebp-0C], edx  <===edx=0x354310AC
:00470D91 837DF400           cmp dword ptr [ebp-0C], 00000000
:00470D95 7508               jne 00470D9F           <===这里跳走
:00470D97 837DF000           cmp dword ptr [ebp-10], 00000000
:00470D9B 7317               jnb 00470DB4
:00470D9D EB02               jmp 00470DA1
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470D95(C)

|

```assembly
:00470D9F 7D13               jge 00470DB4           <===到这里再跳
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470D9D(U)

|

```assembly
:00470DA1 8B45F0             mov eax, dword ptr [ebp-10]
:00470DA4 8B55F4             mov edx, dword ptr [ebp-0C]
:00470DA7 F7D8               neg eax
:00470DA9 83D200             adc edx, 00000000
:00470DAC F7DA               neg edx
:00470DAE 8945F0             mov dword ptr [ebp-10], eax
:00470DB1 8955F4             mov dword ptr [ebp-0C], edx
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470D9B(C), :00470D9F(C)

|

```assembly
:00470DB4 C745E802000000     mov [ebp-18], 00000002   <===跳到这里
:00470DBB C745EC00000000     mov [ebp-14], 00000000
:00470DC2 B301               mov bl, 01             <===一个关键的赋值
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470E15(C), :00470E1A(C)

|

```assembly
:00470DC4 FF75EC             push [ebp-14]    <===0
:00470DC7 FF75E8             push [ebp-18]    <===2
:00470DCA FF75EC             push [ebp-14]    <===0
:00470DCD FF75E8             push [ebp-18]    <===2
:00470DD0 FF75EC             push [ebp-14]    <===0
:00470DD3 FF75E8             push [ebp-18]    <===2
:00470DD6 8B45E8             mov eax, dword ptr [ebp-18]  <===EAX=2
:00470DD9 8B55EC             mov edx, dword ptr [ebp-14]  <===EDX=0
:00470DDC E8EB3FF9FF         call 00404DCC
:00470DE1 E8E63FF9FF         call 00404DCC
:00470DE6 E8E13FF9FF         call 00404DCC  <===进行 3 次同样的计算，等效于 EAX 的值*2*2*2=0x10
:00470DEB 3B55F4             cmp edx, dword ptr [ebp-0C]  <===[ebp-0C]=0x354310AC (要通过第 2 个检测点，则这里只可能是这个值，不可能有其它值)，EDX=0
:00470DEE 7507               jne 00470DF7     <===这里肯定要跳走了
:00470DF0 3B45F0             cmp eax, dword ptr [ebp-10]  <===eax=0x2535022E(这里可以有一些变化，但又影响不到大局)
:00470DF3 7604               jbe 00470DF9
:00470DF5 EB27               jmp 00470E1E     <===如果能这里跳走自然就成功了
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470DEE(C)

|

```assembly
:00470DF7 7F25               jg 00470E1E      <===什么时候从这里跳走呢？(我试了一下如果将 0x354310AC 改为 0xF54310AC 的话，就可以通过了，但要通过第 2 个检测点的话，就不可能有其它值)
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470DF3(C)

|

```assembly
:00470DF9 8345E801           add dword ptr [ebp-18], 00000001
:00470DFD 8355EC00           adc dword ptr [ebp-14], 00000000
:00470E01 FF75EC             push [ebp-14]       <===0
:00470E04 FF75E8             push [ebp-18]       <===3
:00470E07 8B45F0             mov eax, dword ptr [ebp-10]  <===eax=0x2535022E
:00470E0A 8B55F4             mov edx, dword ptr [ebp-0C]  <===edx=0x354310AC
:00470E0D E85A40F9FF         call 00404E6C
:00470E12 83FA00             cmp edx, 00000000
:00470E15 75AD               jne 00470DC4      <===向上跳，构成一个循环结构
:00470E17 83F800             cmp eax, 00000000
:00470E1A 75A8               jne 00470DC4      <===向上跳，构成一个循环结构
:00470E1C 33DB               xor ebx, ebx    <===世上最痛苦的事莫于经过此处(也就是说当 EAX 和 EDX 都为 0 时就 OVER 了)
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00470DF5(U), :00470DF7(C)

|

```assembly
:00470E1E 33C0               xor eax, eax
:00470E20 5A                 pop edx
:00470E21 59                 pop ecx
:00470E22 59                 pop ecx
:00470E23 648910             mov dword ptr fs:[eax], edx
:00470E26 68430E4700         push 00470E43
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00470E41(U)

|

```assembly
:00470E2B 8D45E4             lea eax, dword ptr [ebp-1C]
:00470E2E E8C531F9FF         call 00403FF8
:00470E33 8D45FC             lea eax, dword ptr [ebp-04]
:00470E36 E8BD31F9FF         call 00403FF8
:00470E3B C3                 ret
:00470E3C E9BB2BF9FF         jmp 004039FC
:00470E41 EBE8               jmp 00470E2B
:00470E43 8BC3               mov eax, ebx     <===EAX 必须为 1
:00470E45 5B                 pop ebx
:00470E46 8BE5               mov esp, ebp
:00470E48 5D                 pop ebp
:00470E49 C3                 ret
```

.......
.......

已经到了最后一关，却出现这个无奈的结局! 不知是我的水平有限，某个地方没有分析正确，还是作者就是故意留下的陷井，自相矛盾无法注册(当你汇去款后，再给你正式版。)

不能得到完全正确的注册算法，写注册机也就没有意思了。干脆来个完美爆破版

```assembly
00470C21 8A45FF    mov al, byte ptr [ebp-01] ------>改为 B00190    mov al,01    nop
```
