
```
.text:00401540 s_cpy           proc near               ; CODE XREF: s_mul_imd_1+6Cp

.text:00401540                                         ; s_mul_imd_1+BAp ...

.text:00401540

.text:00401540 arg_Len         = dword ptr  4

.text:00401540

.text:00401540                 push    esi

.text:00401541                 mov     esi, [esp+4+arg_Len]

.text:00401545                 xor     edx, edx

.text:00401547                 mov     eax, [esi+4]

.text:0040154A                 test    eax, eax

.text:0040154C                 mov     [ecx+4], eax

.text:0040154F                 jle     short loc_401579

.text:00401551                 push    ebx

.text:00401552                 push    ebp

.text:00401553                 push    edi

.text:00401554                 mov     edi, esi

.text:00401556                 lea     eax, [ecx+1008h]

.text:0040155C                 sub     edi, ecx

.text:0040155E

.text:0040155E loc_40155E:                             ; CODE XREF: s_cpy+34j

.text:0040155E                 mov     ebx, [edi+eax]

.text:00401561                 mov     ebp, [eax]

.text:00401563                 inc     edx

.text:00401564                 add     eax, 4

.text:00401567                 mov     ebx, [esi+ebx*4+8]

.text:0040156B                 mov     [ecx+ebp*4+8], ebx

.text:0040156F                 mov     ebx, [ecx+4]

.text:00401572                 cmp     edx, ebx

.text:00401574                 jl      short loc_40155E

.text:00401576                 pop     edi

.text:00401577                 pop     ebp

.text:00401578                 pop     ebx

.text:00401579

.text:00401579 loc_401579:                             ; CODE XREF: s_cpy+Fj

.text:00401579                 pop     esi

.text:0040157A                 retn    4

.text:0040157A s_cpy           endp
```

## 复制或相加
```
.text:004015E0 ; copy or add

.text:004015E0

.text:004015E0 sub_4015E0      proc near               ; CODE XREF: s_mul_imd_1+91p

.text:004015E0                                         ; s_mul_v1_v2+C4p

.text:004015E0

.text:004015E0 var_4           = dword ptr -4

.text:004015E0 arg_0           = dword ptr  4

.text:004015E0

.text:004015E0                 push    ecx

.text:004015E1                 mov     edx, [ecx+4]

.text:004015E4                 push    ebx

.text:004015E5                 push    esi

.text:004015E6                 mov     esi, [esp+0Ch+arg_0]

.text:004015EA                 mov     [esp+0Ch+var_4], edx

.text:004015EE                 mov     eax, [esi+4]

.text:004015F1                 cmp     edx, eax
```

## 用立即数乘，注意乘法的顺序是反的，即如果你输入的是123，去乘以4，实际上是321*4＝1284，结果再扭一下变成4821
```
.text:004016E0 ; mul imd?

.text:004016E0

.text:004016E0 s_mul_imd       proc near               ; CODE XREF: s_mul_imd_1+80p

.text:004016E0

.text:004016E0 arg_0           = dword ptr  4

.text:004016E0

.text:004016E0                 push    edi

.text:004016E1                 mov     edi, [esp+4+arg_0]

.text:004016E5                 test    edi, edi

.text:004016E7                 jl      short loc_401726

.text:004016E9                 cmp     edi, 0Ah

.text:004016EC                 jge     short loc_401726

.text:004016EE                 mov     eax, [ecx+4]

.text:004016F1                 xor     edx, edx

.text:004016F3                 test    eax, eax

.text:004016F5                 jle     short loc_40171B

.text:004016F7                 push    ebx

.text:004016F8                 push    esi

.text:004016F9                 lea     esi, [ecx+1008h]

.text:004016FF

.text:004016FF loc_4016FF:                             ; CODE XREF: s_mul_imd+37j

.text:004016FF                 mov     eax, [esi]

.text:00401701                 add     esi, 4

.text:00401704                 mov     ebx, [ecx+eax*4+8]

.text:00401708                 lea     eax, [ecx+eax*4+8]

.text:0040170C                 imul    ebx, edi

.text:0040170F                 mov     [eax], ebx

.text:00401711                 mov     eax, [ecx+4]

.text:00401714                 inc     edx

.text:00401715                 cmp     edx, eax

.text:00401717                 jl      short loc_4016FF

.text:00401719                 pop     esi

.text:0040171A                 pop     ebx

.text:0040171B

.text:0040171B loc_40171B:                             ; CODE XREF: s_mul_imd+15j

.text:0040171B                 call    s_adjust        ; adjust data, make sure each digit occupy its position

.text:00401720                 xor     eax, eax

.text:00401722                 pop     edi

.text:00401723                 retn    4
```

## 作了一些封装的乘法
```
.text:00401730 s_mul_imd_1     proc near               ; CODE XREF: _main+E9p

.text:00401730                                         ; _main+132p ...

.text:00401730

.text:00401730 vAAA            = byte ptr -402Ch

.text:00401730 vBBB            = byte ptr -201Ch

.text:00401730 var_C           = dword ptr -0Ch

.text:00401730 var_4           = dword ptr -4

.text:00401730 arg_0           = dword ptr  4

.text:00401730

.text:00401730                 push    0FFFFFFFFh

.text:00401732                 mov     eax, large fs:0

.text:00401738                 push    offset loc_4075D6

.text:0040173D                 push    eax

.text:0040173E                 mov     eax, 4020h

.text:00401743                 mov     large fs:0, esp

.text:0040174A                 call    __alloca_probe

.text:0040174F                 push    ebx

.text:00401750                 push    ebp

.text:00401751                 push    esi

.text:00401752                 mov     ebp, ecx

.text:00401754                 push    edi

.text:00401755                 lea     ecx, [esp+403Ch+vAAA]

.text:00401759                 call    s_init1?

.text:0040175E                 xor     edi, edi
```

## 两个大数相乘，结果在`Ecx`指向的大数中
```
.text:00401840 s_mul_v1_v2     proc near               ; CODE XREF: _main+122p

.text:00401840

.text:00401840 var_4030        = dword ptr -4030h

.text:00401840 vCCC            = byte ptr -402Ch

.text:00401840 vDDD            = byte ptr -201Ch

.text:00401840 var_C           = dword ptr -0Ch

.text:00401840 var_4           = dword ptr -4

.text:00401840 arg_0           = dword ptr  4

.text:00401840

.text:00401840                 push    0FFFFFFFFh

.text:00401842                 mov     eax, large fs:0

.text:00401848                 push    offset loc_4075F6

.text:0040184D                 push    eax

.text:0040184E                 mov     eax, 4024h

.text:00401853                 mov     large fs:0, esp

.text:0040185A                 call    __alloca_probe

.text:0040185F                 push    ebx

.text:00401860                 push    ebp

.text:00401861                 push    esi

.text:00401862                 mov     esi, ecx

.text:00401864                 push    edi

.text:00401865                 lea     ecx, [esp+4040h+vCCC]

.text:00401869                 call    s_init1?

.text:0040186E                 xor     edi, edi

.text:00401870                 lea     ecx, [esp+4040h+vDDD]

.text:00401877                 push    edi
```

## 比较大数，根据参数可以选择比较的方向
