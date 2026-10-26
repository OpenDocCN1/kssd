
``` 
:0041E4E1 C1E208                  shl edx, 08
:0041E4E4 0BD1                    or edx, ecx
:0041E4E6 4D                      dec ebp
:0041E4E7 8957FC                  mov dword ptr [edi-04], edx
:0041E4EA 75D9                    jne 0041E4C5
:0041E4EC 8D54240C                lea edx, dword ptr [esp+0C]  <--edx为存放MD5算法的初始值的首地址
:0041E4F0 8D7E08                  lea edi, dword ptr [esi+08]  <--edi为存放注册文件中非注册码的部分及填充部分的首地址
:0041E4F3 52                      push edx
:0041E4F4 57                      push edi
:0041E4F5 8BCE                    mov ecx, esi
:0041E4F7 E854010000              call 0041E650  <--进入MD5算法，按F8进入(不想看MD5算法的话可按F10带过此call)
:0041E4FC 8BC7                    mov eax, edi  <--edi为计算后的结果的首地址
:0041E4FE 8D4E59                  lea ecx, dword ptr [esi+59]
:0041E501 5F                      pop edi
\* Possible Reference to String Resource ID=00004: ".\*(&H)"
                                  |
:0041E502 BE04000000              mov esi, 00000004
:0041E507 5D                      pop ebp
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041E52E(C)
|
:0041E508 8A10                    mov dl, byte ptr [eax]
:0041E50A 83C004                  add eax, 00000004
:0041E50D 8851FF                  mov byte ptr [ecx-01], dl
:0041E510 8B50FC                  mov edx, dword ptr [eax-04]
:0041E513 C1EA08                  shr edx, 08
:0041E516 8811                    mov byte ptr [ecx], dl
:0041E518 8B50FC                  mov edx, dword ptr [eax-04]
:0041E51B C1EA10                  shr edx, 10
:0041E51E 885101                  mov byte ptr [ecx+01], dl
:0041E521 8B50FC                  mov edx, dword ptr [eax-04]
:0041E524 C1EA18                  shr edx, 18
:0041E527 885102                  mov byte ptr [ecx+02], dl
:0041E52A 83C104                  add ecx, 00000004
:0041E52D 4E                      dec esi
:0041E52E 75D8                    jne 0041E508
:0041E530 5E                      pop esi
:0041E531 83C440                  add esp, 00000040
:0041E534 C3                      ret
```

在上面的核心`call`按`F8`进入后会来到如下地方：

``` 
\* Referenced by a CALL at Addresses:
|:0041E45B   , :0041E4F7
|
:0041E650 83EC44                  sub esp, 00000044
:0041E653 8B44244C                mov eax, dword ptr [esp+4C]  <--eax为存放注册文件中非注册码的部分及填充部分的首地址
:0041E657 53                      push ebx
:0041E658 8B5C244C                mov ebx, dword ptr [esp+4C]  <--ebx为存放MD5算法的初始值的首地址
:0041E65C 55                      push ebp
:0041E65D 56                      push esi
:0041E65E 57                      push edi
:0041E65F 8B38                    mov edi, dword ptr [eax]
:0041E661 8B7304                  mov esi, dword ptr [ebx+04]
:0041E664 8B4B0C                  mov ecx, dword ptr [ebx+0C]
:0041E667 8B6B08                  mov ebp, dword ptr [ebx+08]
:0041E66A 897C245C                mov dword ptr [esp+5C], edi
:0041E66E 8BFE                    mov edi, esi
:0041E670 F7D7                    not edi
:0041E672 23F9                    and edi, ecx
:0041E674 8B13                    mov edx, dword ptr [ebx]
:0041E676 8BCD                    mov ecx, ebp
:0041E678 23CE                    and ecx, esi
:0041E67A 0BF9                    or edi, ecx
:0041E67C 8B4C245C                mov ecx, dword ptr [esp+5C]
:0041E680 03F9                    add edi, ecx
:0041E682 8D941778A46AD7          lea edx, dword ptr [edi+edx-28955B88]
:0041E689 8BFE                    mov edi, esi
:0041E68B 8BCA                    mov ecx, edx
:0041E68D C1E919                  shr ecx, 19
:0041E690 C1E207                  shl edx, 07
:0041E693 0BCA                    or ecx, edx
:0041E695 8B5004                  mov edx, dword ptr [eax+04]
:0041E698 03CE                    add ecx, esi
:0041E69A 89542428                mov dword ptr [esp+28], edx
:0041E69E 8BD1                    mov edx, ecx
:0041E6A0 23F9                    and edi, ecx
:0041E6A2 F7D2                    not edx
:0041E6A4 23D5                    and edx, ebp
:0041E6A6 0BD7                    or edx, edi
:0041E6A8 8B7C2428                mov edi, dword ptr [esp+28]
:0041E6AC 03D7                    add edx, edi
:0041E6AE 8B7B0C                  mov edi, dword ptr [ebx+0C]
:0041E6B1 8DBC1756B7C7E8          lea edi, dword ptr [edi+edx-173848AA]
:0041E6B8 8BD7                    mov edx, edi
:0041E6BA C1EA14                  shr edx, 14
:0041E6BD C1E70C                  shl edi, 0C
:0041E6C0 0BD7                    or edx, edi
:0041E6C2 8B7808                  mov edi, dword ptr [eax+08]
:0041E6C5 03D1                    add edx, ecx
:0041E6C7 897C2440                mov dword ptr [esp+40], edi
:0041E6CB 8BFA                    mov edi, edx
:0041E6CD F7D7                    not edi
:0041E6CF 23FE                    and edi, esi
:0041E6D1 8BF2                    mov esi, edx
:0041E6D3 23F1                    and esi, ecx
:0041E6D5 0BFE                    or edi, esi
:0041E6D7 8B742440                mov esi, dword ptr [esp+40]
:0041E6DB 03FE                    add edi, esi
:0041E6DD 8DBC2FDB702024          lea edi, dword ptr [edi+ebp+242070DB]
:0041E6E4 8BEA                    mov ebp, edx
:0041E6E6 8BF7                    mov esi, edi
:0041E6E8 C1EE0F                  shr esi, 0F
:0041E6EB C1E711                  shl edi, 11
:0041E6EE 0BF7                    or esi, edi
:0041E6F0 8B780C                  mov edi, dword ptr [eax+0C]
:0041E6F3 03F2                    add esi, edx
:0041E6F5 897C2420                mov dword ptr [esp+20], edi
:0041E6F9 8BFE                    mov edi, esi
:0041E6FB 23EE                    and ebp, esi
:0041E6FD F7D7                    not edi
:0041E6FF 23F9                    and edi, ecx
:0041E701 0BFD                    or edi, ebp
:0041E703 8B6C2420                mov ebp, dword ptr [esp+20]
:0041E707 03FD                    add edi, ebp
:0041E709 8B6B04                  mov ebp, dword ptr [ebx+04]
:0041E70C 8DAC2FEECEBDC1          lea ebp, dword ptr [edi+ebp-3E423112]
:0041E713 8BFD                    mov edi, ebp
:0041E715 C1E716                  shl edi, 16
:0041E718 C1ED0A                  shr ebp, 0A
:0041E71B 0BFD                    or edi, ebp
:0041E71D 8B6810                  mov ebp, dword ptr [eax+10]
:0041E720 03FE                    add edi, esi
:0041E722 896C2438                mov dword ptr [esp+38], ebp
:0041E726 897C2458                mov dword ptr [esp+58], edi
:0041E72A F7D7                    not edi
:0041E72C 23FA                    and edi, edx
:0041E72E 8BEE                    mov ebp, esi
:0041E730 236C2458                and ebp, dword ptr [esp+58]
:0041E734 0BFD                    or edi, ebp
:0041E736 8B6C2438                mov ebp, dword ptr [esp+38]
:0041E73A 03FD                    add edi, ebp
:0041E73C 8B6814                  mov ebp, dword ptr [eax+14]
:0041E73F 896C2418                mov dword ptr [esp+18], ebp
:0041E743 8D8C39AF0F7CF5          lea ecx, dword ptr [ecx+edi-0A83F051]
:0041E74A 8BF9                    mov edi, ecx
:0041E74C C1EF19                  shr edi, 19
:0041E74F C1E107                  shl ecx, 07
:0041E752 0BF9                    or edi, ecx
:0041E754 8B4C2458                mov ecx, dword ptr [esp+58]
:0041E758 03F9                    add edi, ecx
:0041E75A 8BEF                    mov ebp, edi
:0041E75C 23CF                    and ecx, edi
:0041E75E F7D5                    not ebp
:0041E760 23EE                    and ebp, esi
:0041E762 0BE9                    or ebp, ecx
:0041E764 8B4C2418                mov ecx, dword ptr [esp+18]
:0041E768 03E9                    add ebp, ecx
:0041E76A 8D942A2AC68747          lea edx, dword ptr [edx+ebp+4787C62A]
:0041E771 8B6C2458                mov ebp, dword ptr [esp+58]
:0041E775 8BCA                    mov ecx, edx
:0041E777 C1E914                  shr ecx, 14
:0041E77A C1E20C                  shl edx, 0C
:0041E77D 0BCA                    or ecx, edx
```
