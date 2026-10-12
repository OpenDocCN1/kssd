
# 汇编代码分析

```
:00412210 55                   push ebp
:00412211 89E5                 mov ebp, esp
:00412213 53                   push ebx
:00412214 83EC58               sub esp, 00000058
:00412217 803D7967420000       cmp byte ptr [00426779], 00
:0041221E 750E                 jne 0041222E
:00412220 C6057867420000       mov byte ptr [00426778], 00
:00412227 C6057967420001       mov byte ptr [00426779], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041221E(C)
|

:0041222E 803D64D1530000       cmp byte ptr [0053D164], 00
:00412235 7555                 jne 0041228C
:00412237 803D7867420000       cmp byte ptr [00426778], 00
:0041223E 754C                 jne 0041228C
:00412240 C6057867420001       mov byte ptr [00426778], 01
:00412247 8D55EC               lea edx, dword ptr [ebp-14]
:0041224A 8D45AC               lea eax, dword ptr [ebp-54]
:0041224D 52                   push edx
:0041224E 50                   push eax
:0041224F E8ACCBFFFF           call 0040EE00  <-出来

:00412254 8B45A8               mov eax, dword ptr [ebp-58]
:00412257 59                   pop ecx
:00412258 3345A8               xor eax, dword ptr [ebp-58]
:0041225B 59                   pop ecx
:0041225C 8D4DEC               lea ecx, dword ptr [ebp-14]
:0041225F 8945A8               mov dword ptr [ebp-58], eax
:00412262 8D45AC               lea eax, dword ptr [ebp-54]
:00412265 51                   push ecx
:00412266 50                   push eax
:00412267 31DB                 xor ebx, ebx
:00412269 E8D2080000           call 00412B40  关键的比较函数，进入其中

:0041226E 85C0                 test eax, eax
:00412270 59                   pop ecx
:00412271 59                   pop ecx
:00412272 7507                 jne 0041227B
:00412274 C745A801000000       mov [ebp-58], 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412272(C)
|

:0041227B 807DA800             cmp byte ptr [ebp-58], 00
:0041227F 7505                 jne 00412286
:00412281 BB01000000           mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041227F(C)
|

:00412286 881D64D15300         mov byte ptr [0053D164], bl

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00412235(C), :0041223E(C)
|

:0041228C A064D15300           mov al, byte ptr [0053D164]
:00412291 8D65FC               lea esp, dword ptr [ebp-04]
:00412294 5B                   pop ebx
:00412295 5D                   pop ebp
:00412296 C3                   ret
```

`call 0040EE00` 是调用读取注册表的函数，所以从上面的地方出来，到达下面的比较的地方，在这里你将得到关键的比较地方，进入其中

## 关键比较函数 (`00412B40`)

```
:00412B40 55                   push ebp
:00412B41 31C0                 xor eax, eax
:00412B43 89E5                 mov ebp, esp
:00412B45 53                   push ebx
:00412B46 56                   push esi
:00412B47 57                   push edi
:00412B48 83EC10               sub esp, 00000010
:00412B4B 8B7D0C               mov edi, dword ptr [ebp+0C]
:00412B4E B9FFFFFFFF           mov ecx, FFFFFFFF
:00412B53 8B7508               mov esi, dword ptr [ebp+08]
:00412B56 C745F000000000       mov [ebp-10], 00000000
:00412B5D F2                   repnz
:00412B5E AE                   scasb
:00412B5F B8FEFFFFFF           mov eax, FFFFFFFE
:00412B64 29C8                 sub eax, ecx
:00412B66 3D0C000000           cmp eax, 0000000C
:00412B6B 0F850E010000         jne 00412C7F
:00412B71 31DB                 xor ebx, ebx
:00412B73 31D2                 xor edx, edx
:00412B75 EB7B                 jmp 00412BF2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412C09(C)
|

:00412B77 0FBFCA               movsx ecx, dx
:00412B7A 0FBE040E             movsx eax, byte ptr [esi+ecx]
:00412B7E 2D41000000           sub eax, 00000041
:00412B83 3D35000000           cmp eax, 00000035
:00412B88 7767                 ja 00412BF1
:00412B8A FF248568D25300       jmp dword ptr [4*eax+0053D268]

:00412B91 81C320A10700         add ebx, 0007A120
:00412B97 EB58                 jmp 00412BF1

:00412B99 8D842000000000       lea eax, dword ptr [eax]
:00412BA0 81C3A0860100         add ebx, 000186A0
:00412BA6 EB49                 jmp 00412BF1

:00412BA8 81C350C30000         add ebx, 0000C350
:00412BAE EB41                 jmp 00412BF1

:00412BB0 81C310270000         add ebx, 00002710
:00412BB6 EB39                 jmp 00412BF1

:00412BB8 81C388130000         add ebx, 00001388
:00412BBE EB31                 jmp 00412BF1

:00412BC0 81C3E8030000         add ebx, 000003E8
:00412BC6 EB29                 jmp 00412BF1

:00412BC8 81C3F4010000         add ebx, 000001F4
:00412BCE EB21                 jmp 00412BF1

:00412BD0 83C364               add ebx, 00000064
:00412BD3 EB1C                 jmp 00412BF1

:00412BD5 83C332               add ebx, 00000032
:00412BD8 EB17                 jmp 00412BF1

:00412BDA 8D8000000000         lea eax, dword ptr [eax+00000000]
:00412BE0 83C30A               add ebx, 0000000A
:00412BE3 EB0C                 jmp 00412BF1

:00412BE5 83C305               add ebx, 00000005
:00412BE8 EB07                 jmp 00412BF1

:00412BEA 8D8000000000         lea eax, dword ptr [eax+00000000]
:00412BF0 43                   inc ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00412B88(C), :00412B97(U), :00412BA6(U), :00412BAE(U), :00412BB6(U)
|:00412BBE(U), :00412BC6(U), :00412BCE(U), :00412BD3(U), :00412BD8(U)
|:00412BE3(U), :00412BE8(U)
|

:00412BF1 42                   inc edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412B75(U)
|

:00412BF2 B9FFFFFFFF           mov ecx, FFFFFFFF
:00412BF7 89F7                 mov edi, esi
:00412BF9 31C0                 xor eax, eax
:00412BFB F2                   repnz
:00412BFC AE                   scasb
:00412BFD 0FBFC2               movsx eax, dx
:00412C00 BFFEFFFFFF           mov edi, FFFFFFFE
:00412C05 29CF                 sub edi, ecx
:00412C07 39F8                 cmp eax, edi
:00412C09 0F8268FFFFFF         jb 00412B77
:00412C0F 8B4D0C               mov ecx, dword ptr [ebp+0C]
:00412C12 8A4109               mov al, byte ptr [ecx+09]
:00412C15 8945EC               mov dword ptr [ebp-14], eax
:00412C18 89C8                 mov eax, ecx
:00412C1A C6400900             mov [eax+09], 00
:00412C1E FF750C               push [ebp+0C]
:00412C21 E84AC30000           call 0041EF70
:00412C26 89C7                 mov edi, eax
:00412C28 59                   pop ecx
:00412C29 8B450C               mov eax, dword ptr [ebp+0C]
:00412C2C 8B4DEC               mov ecx, dword ptr [ebp-14]
:00412C2F 884809               mov byte ptr [eax+09], cl
:00412C32 29DF                 sub edi, ebx
:00412C34 8B450C               mov eax, dword ptr [ebp+0C]
:00412C37 0509000000           add eax, 00000009
:00412C3C 50                   push eax
:00412C3D E82EC30000           call 0041EF70
:00412C42 81FFFFFFFF0E         cmp edi, 0EFFFFFF
:00412C48 59                   pop ecx
:00412C49 89C3                 mov ebx, eax
:00412C4B 7432                 je 00412C7F
```
