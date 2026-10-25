
```
:00403E1E 741A                  je 00403E3A
:00403E20 C70300000000          mov dword ptr [ebx], 00000000
:00403E26 8B4AF8                mov ecx, dword ptr [edx-08]
:00403E29 49                    dec ecx
:00403E2A 7C0E                  jl 00403E3A
:00403E2C F0                    lock
:00403E2D FF4AF8                dec [edx-08]
:00403E30 7508                  jne 00403E3A
:00403E32 8D42F8                lea eax, dword ptr [edx-08]
:00403E35 E8DEE9FFFF            call 00402818
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00403E1E(C), :00403E2A(C), :00403E30(C)
|
:00403E3A 83C304                add ebx, 00000004
:00403E3D 4E                    dec esi
:00403E3E 75DA                  jne 00403E1A
:00403E40 5E                    pop esi
:00403E41 5B                    pop ebx
:00403E42 C3                    ret
```

conanxu[BCG]

Email:conanxu@eastday.com
