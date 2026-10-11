
```
:004AD69A BA2CDE4A00    mov edx, 004ADE2C
:004AD69F E8E466F5FF    call 00403D88
:004AD6A4 8B8574FFFFFF  mov eax, dword ptr [ebp+FFFFFF74]
:004AD6AA E849F1FDFF    call 0048C7F8
:004AD6AF 8B45FC        mov eax, dword ptr [ebp-04]
:004AD6B2 E8754E0000    call 004B252C        \ <----- 来到这里
:004AD6B7 84C0          test al, al         - 看起来很眼熟呀!
:004AD6B9 754C          jne 004AD707        / 将这里改为 je 试试
:004AD6BB 8B45FC        mov eax, dword ptr [ebp-04]
:004AD6BE 0570B35101    add eax, 0151B370

\* Possible StringData Ref from Data Obj -> "Unregistered Shareware"
                              |
:004AD6C3 BA48DE4A00      mov edx, 004ADE48
:004AD6C8 E84764F5FF      call 00403B14
:004AD6CD 8B0DF06F4B00    mov ecx, dword ptr [004B6FF0]
:004AD6D3 A1B86F4B00      mov eax, dword ptr [004B6FB8]
:004AD6D8 8B00            mov eax, dword ptr [eax]

\* Possible StringData Ref from Data Obj -> "念@"
                              |
:004AD6DA 8B15548D4900    mov edx, dword ptr [00498D54]
:004AD6E0 E85F37F8FF      call 00430E44
:004AD6E5 A1F06F4B00      mov eax, dword ptr [004B6FF0]
:004AD6EA 8B00            mov eax, dword ptr [eax]
:004AD6EC E8DB18F8FF      call 0042EFCC
:004AD6F1 83F802          cmp eax, 00000002
:004AD6F4 7511            jne 004AD707
:004AD6F6 A1B86F4B00      mov eax, dword ptr [004B6FB8]
:004AD6FB 8B00            mov eax, dword ptr [eax]
:004AD6FD E88238F8FF      call 00430F84
:004AD702 E951060000      jmp 004ADD58

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
| :004AD6B9(C), :004AD6F4(C)
|
\* Possible StringData Ref from Data Obj -> "Load Database"
                              |
:004AD707 B868DE4A00      mov eax, 004ADE68
:004AD70C E8E7F0FDFF      call 0048C7F8
:004AD711 8B45FC          mov eax, dword ptr [ebp-04]
:004AD714 80B86053000000  cmp byte ptr [eax+00005360], 00
:004AD71B 7417            je 004AD734
:004AD71D A12C6F4B00      mov eax, dword ptr [004B6F2C]
:004AD722 8B00            mov eax, dword ptr [eax]
:004AD724 8B80E4010000    mov eax, dword ptr [eax+000001E4]

\* Possible StringData Ref from Data Obj -> "Loading database..."
                              |
:004AD72A BA80DE4A00      mov edx, 004ADE80
:004AD72F E88C57F9FF      call 00442EC0
```

将 `:004AD6B9` `754C` `jne` `004AD707`

改为: `744C` `je` `004ad707`

运行一下，果然注册成功，再也不会出现注册提示框了。 `4dd466d0e1e0cc123afbe504da2462ca`
