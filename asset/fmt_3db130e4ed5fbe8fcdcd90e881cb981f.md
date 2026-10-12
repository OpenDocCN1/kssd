
```
|:0040F46A(U)

:0040F471 A1B8644A00  mov eax, dword ptr [004A64B8]
:0040F476 8B00        mov eax, dword ptr [eax]
:0040F478 E837720700  call 004866B4
:0040F47D 66C746105C00 mov [esi+10], 005C
* Possible StringData Ref from Data Obj ->"UserName"

:0040F483 BA74E94900  mov edx, 0049E974
:0040F488 8D45E4      lea eax, dword ptr [ebp-1C]
:0040F48B E8C8720700  call 00486758
:0040F490 FF461C      inc [esi+1C]
:0040F493 8B10        mov edx, dword ptr [eax]
:0040F495 8B4DFC      mov ecx, dword ptr [ebp-04]
:0040F498 8BC7        mov eax, edi
:0040F49A E8754C0100  call 00424114
:0040F49F FF4E1C      dec [esi+1C]
:0040F4A2 8D45E4      lea eax, dword ptr [ebp-1C]
:0040F4A5 BA02000000  mov edx, 00000002
:0040F4AA E8AD730700  call 0048685C
:0040F4AF 66C746106800 mov [esi+10], 0068
* Possible StringData Ref from Data Obj ->"RegCode"

:0040F4B5 BA7DE94900  mov edx, 0049E97D
:0040F4BA 8D45E0      lea eax, dword ptr [ebp-20]
:0040F4BD E896720700  call 00486758
:0040F4C2 FF461C      inc [esi+1C]
:0040F4C5 8B10        mov edx, dword ptr [eax]
:0040F4C7 8B4DF8      mov ecx, dword ptr [ebp-08]
:0040F4CA 8BC7        mov eax, edi
:0040F4CC E8434C0100  call 00424114
:0040F4D1 FF4E1C      dec [esi+1C]
:0040F4D4 8D45E0      lea eax, dword ptr [ebp-20]
:0040F4D7 BA02000000  mov edx, 00000002
:0040F4DC E87B730700  call 0048685C
:0040F4E1 FF4E1C      dec [esi+1C]
:0040F4E4 8D45EC      lea eax, dword ptr [ebp-14]
:0040F4E7 BA02000000  mov edx, 00000002
:0040F4EC E86B730700  call 0048685C
:0040F4F1 66C746100800 mov [esi+10], 0008
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F413(C)

:0040F4F7 897DD8      mov dword ptr [ebp-28], edi
:0040F4FA 837DD800    cmp dword ptr [ebp-28], 00000000
:0040F4FE 7421        je 0040F521
:0040F500 8B4DD8      mov ecx, dword ptr [ebp-28]
:0040F503 8B01        mov eax, dword ptr [ecx]
:0040F505 8945DC      mov dword ptr [ebp-24], eax
:0040F508 66C746108000 mov [esi+10], 0080
:0040F50E BA03000000  mov edx, 00000003
:0040F513 8B45D8      mov eax, dword ptr [ebp-28]
:0040F516 8B08        mov ecx, dword ptr [eax]
:0040F518 FF51FC      call [ecx-04]
:0040F51B 66C746107400 mov [esi+10], 0074
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F4FE(C)

:0040F521 8BC3        mov eax, ebx
:0040F523 E8E4B80200  call 0043AE0C
:0040F528 EB7A        jmp 0040F5A4
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F3B7(C)

:0040F52A 66C746109800 mov [esi+10], 0098
:0040F530 8D45D0      lea eax, dword ptr [ebp-30]
* Possible StringData Ref from Data Obj ->"Kpxcnkf wugt pcog qt tgikuvtcvkqp "
                                        ->"eqfg."

:0040F533 8B15E4E74900 mov edx, dword ptr [0049E7E4]
:0040F539 E81A720700  call 00486758
:0040F53E FF461C      inc [esi+1C]
:0040F541 8D4DD4      lea ecx, dword ptr [ebp-2C]
:0040F544 8B10        mov edx, dword ptr [eax]
:0040F546 33C0        xor eax, eax
:0040F548 8945D4      mov dword ptr [ebp-2C], eax
:0040F54B 8BC3        mov eax, ebx
:0040F54D FF461C      inc [esi+1C]
:0040F550 E8430A0000  call 0040FF98
:0040F555 FF4E1C      dec [esi+1C]
:0040F558 8D45D0      lea eax, dword ptr [ebp-30]
:0040F55B BA02000000  mov edx, 00000002
:0040F560 E8F7720700  call 0048685C
:0040F565 66C746108C00 mov [esi+10], 008C
:0040F56B 6A10        push 00000010
* Possible StringData Ref from Data Obj ->"B-Jigsaw"

:0040F56D B985E94900  mov ecx, 0049E985
:0040F572 837DD400    cmp dword ptr [ebp-2C], 00000000
:0040F576 7405        je 0040F57D
:0040F578 8B55D4      mov edx, dword ptr [ebp-2C]
:0040F57B EB05        jmp 0040F582
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F576(C)

:0040F57D BA8EE94900  mov edx, 0049E98E
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F57B(U)

:0040F582 A1B8644A00  mov eax, dword ptr [004A64B8]
:0040F587 8B00        mov eax, dword ptr [eax]
:0040F589 E826710700  call 004866B4 <-------------------------------------------------------如果您走过这个 CALL 的话就玩完了
:0040F58E FF4E1C      dec [esi+1C]
:0040F591 8D45D4      lea eax, dword ptr [ebp-2C]
:0040F594 BA02000000  mov edx, 00000002
:0040F599 E8BE720700  call 0048685C
:0040F59E 66C746100800 mov [esi+10], 0008
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F528(U)

:0040F5A4 FF4E1C      dec [esi+1C]
:0040F5A7 8D45F8      lea eax, dword ptr [ebp-08]
:0040F5AA BA02000000  mov edx, 00000002
:0040F5AF E8A8720700  call 0048685C
:0040F5B4 FF4E1C      dec [esi+1C]
:0040F5B7 8D45FC      lea eax, dword ptr [ebp-04]
:0040F5BA BA02000000  mov edx, 00000002
:0040F5BF E898720700  call 0048685C
:0040F5C4 66C746100000 mov [esi+10], 0000
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F31E(C)

* Reference To: bjigsaw._FormRegistration

:0040F5CA 8B0DA0644A00 mov ecx, dword ptr [004A64A0]
:0040F5D0 8B19        mov ebx, dword ptr [ecx]
:0040F5D2 895DC8      mov dword ptr [ebp-38], ebx
:0040F5D5 85DB        test ebx, ebx
:0040F5D7 741E        je 0040F5F7
:0040F5D9 8B03        mov eax, dword ptr [ebx]
:0040F5DB 8945CC      mov dword ptr [ebp-34], eax
:0040F5DE 66C74610B000 mov [esi+10], 00B0
:0040F5E4 BA03000000  mov edx, 00000003
:0040F5E9 8B45C8      mov eax, dword ptr [ebp-38]
:0040F5EC 8B08        mov ecx, dword ptr [eax]
:0040F5EE FF51FC      call [ecx-04]
:0040F5F1 66C74610A400 mov [esi+10], 00A4
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F5D7(C)

:0040F5F7 8B16        mov edx, dword ptr [esi]
:0040F5F9 64891500000000 mov dword ptr fs:[00000000], edx
:0040F600 5F          pop edi
:0040F601 5E          pop esi
:0040F602 5B          pop ebx
:0040F603 8BE5        mov esp, ebp
:0040F605 5D          pop ebp
:0040F606 C3          ret
* Referenced by a CALL at Addresses:
```
