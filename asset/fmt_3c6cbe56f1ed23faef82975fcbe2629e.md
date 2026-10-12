
```
:00443EBF 3A4206                cmp al, byte ptr [edx+06] // 第七位真假码比较
:00443EC2 0F8597060000          jne 0044455F // 不能跳
:00443EC8 8BD6                  mov edx, esi
:00443ECA 8BC7                  mov eax, edi
:00443ECC E8EBFDFFFF            call 00443CBC
:00443ED1 B91A000000            mov ecx, 0000001A
:00443ED6 33D2                  xor edx, edx
:00443ED8 F7F1                  div ecx
:00443EDA 8BC2                  mov eax, edx
:00443EDC E8D3FDFFFF            call 00443CB4
:00443EE1 8B55FC                mov edx, dword ptr [ebp-04]
:00443EE4 3A4207                cmp al, byte ptr [edx+07] // 第八位真假码比较
:00443EE7 0F8572060000          jne 0044455F // 不能跳
:00443EED 8BD6                  mov edx, esi
:00443EEF 8BC7                  mov eax, edi
:00443EF1 E8C6FDFFFF            call 00443CBC
:00443EF6 B91A000000            mov ecx, 0000001A
:00443EFB 33D2                  xor edx, edx
:00443EFD F7F1                  div ecx
:00443EFF 8BC2                  mov eax, edx
:00443F01 E8AEFDFFFF            call 00443CB4
:00443F06 8B55FC                mov edx, dword ptr [ebp-04]
:00443F09 3A4208                cmp al, byte ptr [edx+08] // 第九位真假码比较
:00443F0C 0F854D060000          jne 0044455F // 不能跳
:00443F12 8BD6                  mov edx, esi
:00443F14 8BC7                  mov eax, edi
:00443F16 E8A1FDFFFF            call 00443CBC
:00443F1B B91A000000            mov ecx, 0000001A
:00443F20 33D2                  xor edx, edx
:00443F22 F7F1                  div ecx
:00443F24 8BC2                  mov eax, edx
:00443F26 E889FDFFFF            call 00443CB4
:00443F2B 8B55FC                mov edx, dword ptr [ebp-04]
:00443F2E 3A4209                cmp al, byte ptr [edx+09] // 第十位真假码比较
:00443F31 0F8528060000          jne 0044455F // 不能跳
:00443F37 8BD6                  mov edx, esi
:00443F39 8BC7                  mov eax, edi
:00443F3B E87CFDFFFF            call 00443CBC
:00443F40 B91A000000            mov ecx, 0000001A
:00443F45 33D2                  xor edx, edx
:00443F47 F7F1                  div ecx
:00443F49 8BC2                  mov eax, edx
:00443F4B E864FDFFFF            call 00443CB4
:00443F50 8B55FC                mov edx, dword ptr [ebp-04]
:00443F53 3A420A                cmp al, byte ptr [edx+0A] // 第十一位真假码比较
:00443F56 0F8503060000          jne 0044455F // 不能跳
:00443F5C 8BD6                  mov edx, esi
:00443F5E 8BC7                  mov eax, edi
:00443F60 E857FDFFFF            call 00443CBC
:00443F65 B91A000000            mov ecx, 0000001A
:00443F6A 33D2                  xor edx, edx
:00443F6C F7F1                  div ecx
:00443F6E 8BC2                  mov eax, edx
:00443F70 E83FFDFFFF            call 00443CB4
:00443F75 8B55FC                mov edx, dword ptr [ebp-04]
:00443F78 3A420B                cmp al, byte ptr [edx+0B] // 第十二位真假码比较
:00443F7B 0F85DE050000          jne 0044455F // 不能跳
:00443F81 8BD6                  mov edx, esi
:00443F83 8BC7                  mov eax, edi
:00443F85 E832FDFFFF            call 00443CBC
:00443F8A B91A000000            mov ecx, 0000001A
:00443F8F 33D2                  xor edx, edx
:00443F91 F7F1                  div ecx
:00443F93 8BC2                  mov eax, edx
:00443F95 E81AFDFFFF            call 00443CB4
:00443F9A 8B55FC                mov edx, dword ptr [ebp-04]
:00443F9D 3A420C                cmp al, byte ptr [edx+0C] // 第十三位真假码比较
:00443FA0 0F85B9050000          jne 0044455F // 不能跳
:00443FA6 8BD6                  mov edx, esi
:00443FA8 8BC7                  mov eax, edi
:00443FAA E80DFDFFFF            call 00443CBC
:00443FAF B91A000000            mov ecx, 0000001A
:00443FB4 33D2                  xor edx, edx
:00443FB6 F7F1                  div ecx
:00443FB8 8BC2                  mov eax, edx
:00443FBA E8F5FCFFFF            call 00443CB4
:00443FBF 8B55FC                mov edx, dword ptr [ebp-04]
:00443FC2 3A420D                cmp al, byte ptr [edx+0D] // 第十四位真假码比较
:00443FC5 0F8594050000          jne 0044455F // 不能跳
:00443FCB 8BD6                  mov edx, esi
:00443FCD 8BC7                  mov eax, edi
:00443FCF E8E8FCFFFF            call 00443CBC
:00443FD4 B91A000000            mov ecx, 0000001A
:00443FD9 33D2                  xor edx, edx
:00443FDB F7F1                  div ecx
:00443FDD 8BC2                  mov eax, edx
:00443FDF E8D0FCFFFF            call 00443CB4
:00443FE4 8B55FC                mov edx, dword ptr [ebp-04]
:00443FE7 3A420E                cmp al, byte ptr [edx+0E] // 第十五位真假码比较
:00443FEA 0F856F050000          jne 0044455F // 不能跳
:00443FF0 8BD6                  mov edx, esi
:00443FF2 8BC7                  mov eax, edi
:00443FF4 E8C3FCFFFF            call 00443CBC
:00443FF9 B91A000000            mov ecx, 0000001A
:00443FFE 33D2                  xor edx, edx
:00444000 F7F1                  div ecx
:00444002 8BC2                  mov eax, edx
:00444004 E8ABFCFFFF            call 00443CB4
:00444009 8B55FC                mov edx, dword ptr [ebp-04]
:0044400C 3A420F                cmp al, byte ptr [edx+0F] // 第十六位真假码比较
:0044400F 0F854A050000          jne 0044455F // 不能跳
```

### 引用说明

被无条件或条件跳转引用的地址：

- `:00443E53(C)`

```
:00444015 837DEC40              cmp dword ptr [ebp-14], 00000040 // 第四位是否大写字母
:00444019 0F85BC010000          jne 004441DB // 不是就跳去和 80(P) 进行比较产生另一种第五到十六位真假码比较比较方法同上(作者不会有病吧)
:0044401F 8BD6                  mov edx, esi
:00444021 8BC7                  mov eax, edi
:00444023 E8A0FCFFFF            call 00443CC8
:00444028 B91A000000            mov ecx, 0000001A
:0044402D 33D2                  xor edx, edx
:0044402F F7F1                  div ecx
:00444031 8BC2                  mov eax, edx
:00444033 E87CFCFFFF            call 00443CB4
:00444038 8B55FC                mov edx, dword ptr [ebp-04]
:0044403B 3A4204                cmp al, byte ptr [edx+04]
:0044403E 0F851B050000          jne 0044455F
:00444044 8BD6                  mov edx, esi
:00444046 8BC7                  mov eax, edi
:00444048 E87BFCFFFF            call 00443CC8
:0044404D B91A000000            mov ecx, 0000001A
:00444052 33D2                  xor edx, edx
:00444054 F7F1                  div ecx
:00444056 8BC2                  mov eax, edx
:00444058 E857FCFFFF            call 00443CB4
:0044405D 8B55FC                mov edx, dword ptr [ebp-04]
:00444060 3A4205                cmp al, byte ptr [edx+05]
:00444063 0F85F6040000          jne 0044455F
:00444069 8BD6                  mov edx, esi
:0044406B 8BC7                  mov eax, edi
:0044406D E856FCFFFF            call 00443CC8
:00444072 B91A000000            mov ecx, 0000001A
:00444077 33D2                  xor edx, edx
:00444079 F7F1                  div ecx
:0044407B 8BC2                  mov eax, edx
:0044407D E832FCFFFF            call 00443CB4
:00444082 8B55FC                mov edx, dword ptr [ebp-04]
:00444085 3A4206                cmp al, byte ptr [edx+06]
:00444088 0F85D1040000          jne 0044455F
:0044408E 8BD6                  mov edx, esi
:00444090 8BC7                  mov eax, edi
:00444092 E831FCFFFF            call 00443CC8
:00444097 B91A000000            mov ecx, 0000001A
:0044409C 33D2                  xor edx, edx
```
