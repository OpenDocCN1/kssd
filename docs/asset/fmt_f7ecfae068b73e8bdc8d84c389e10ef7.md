
```asm
:004162B4 03FD                      add edi, ebp
:004162B6 8B6C2434                  mov ebp, dword ptr [esp+34]
:004162BA 8DBC39A111084E            lea edi, dword ptr [ecx+edi+4E0811A1]
:004162C1 8BCF                      mov ecx, edi
:004162C3 C1EF0B                    shr edi, 0B
:004162C6 C1E115                    shl ecx, 15
:004162C9 0BCF                      or ecx, edi
:004162CB 03CE                      add ecx, esi
:004162CD 8BFA                      mov edi, edx
:004162CF F7D7                      not edi
:004162D1 0BF9                      or edi, ecx
:004162D3 33FE                      xor edi, esi
:004162D5 03FD                      add edi, ebp
:004162D7 8B6C2438                  mov ebp, dword ptr [esp+38]
:004162DB 8DBC38827E53F7            lea edi, dword ptr [eax+edi-08AC817E]
:004162E2 8BC7                      mov eax, edi
:004162E4 C1E706                    shl edi, 06
:004162E7 C1E81A                    shr eax, 1A
:004162EA 0BC7                      or eax, edi
:004162EC 8BFE                      mov edi, esi
:004162EE 03C1                      add eax, ecx
:004162F0 F7D7                      not edi
:004162F2 0BF8                      or edi, eax
:004162F4 33F9                      xor edi, ecx
:004162F6 03FD                      add edi, ebp
:004162F8 8DBC3A35F23ABD            lea edi, dword ptr [edx+edi-42C50DCB]
:004162FF 8BD7                      mov edx, edi
:00416301 C1E70A                    shl edi, 0A
:00416304 C1EA16                    shr edx, 16
:00416307 0BD7                      or edx, edi
:00416309 03D0                      add edx, eax
:0041630B 8BF9                      mov edi, ecx
:0041630D F7D7                      not edi
:0041630F 0BFA                      or edi, edx
:00416311 33F8                      xor edi, eax
:00416313 03FB                      add edi, ebx
:00416315 8B5C243C                  mov ebx, dword ptr [esp+3C]
:00416319 8DBC3EBBD2D72A            lea edi, dword ptr [esi+edi+2AD7D2BB]
:00416320 8BF7                      mov esi, edi
:00416322 C1E70F                    shl edi, 0F
:00416325 C1EE11                    shr esi, 11
:00416328 0BF7                      or esi, edi
:0041632A 8BF8                      mov edi, eax
:0041632C F7D7                      not edi
:0041632E 03F2                      add esi, edx
:00416330 0BFE                      or edi, esi
:00416332 33FA                      xor edi, edx
:00416334 03FB                      add edi, ebx
:00416336 8DBC3991D386EB            lea edi, dword ptr [ecx+edi-14792C6F]
:0041633D 8B4C2450                  mov ecx, dword ptr [esp+50]
:00416341 8B19                      mov ebx, dword ptr [ecx]
:00416343 03D8                      add ebx, eax
:00416345 8BC7                      mov eax, edi
:00416347 C1E015                    shl eax, 15
:0041634A C1EF0B                    shr edi, 0B
:0041634D 0BC7                      or eax, edi
:0041634F 8919                      mov dword ptr [ecx], ebx
:00416351 034104                    add eax, dword ptr [ecx+04]
:00416354 03C6                      add eax, esi
:00416356 894104                    mov dword ptr [ecx+04], eax
:00416359 8B4108                    mov eax, dword ptr [ecx+08]
:0041635C 03C6                      add eax, esi
:0041635E 5F                        pop edi
:0041635F 894108                    mov dword ptr [ecx+08], eax
:00416362 8B410C                    mov eax, dword ptr [ecx+0C]
:00416365 5E                        pop esi
:00416366 03C2                      add eax, edx
:00416368 5D                        pop ebp
:00416369 89410C                    mov dword ptr [ecx+0C], eax
:0041636C 5B                        pop ebx
:0041636D 83C444                    add esp, 00000044
:00416370 C3                        ret
```

---

### 【注册信息保存】

```reg
REGEDIT4

[HKEY_CURRENT_USER\Software\UDP Soft\The All-Seeing Eye\Settings]

"RegCodeHi"=dword:01234567
"RegCodeLo"=dword:789abcde
```

---

### 【整理】

一个可用的注册码：`abcdefghhijklmnookbg`

---

Cracked By 巢水工作坊——fly【OCN】

9:16 03-4-3 4dd466d0e1e0cc123afbe504da2462ca
