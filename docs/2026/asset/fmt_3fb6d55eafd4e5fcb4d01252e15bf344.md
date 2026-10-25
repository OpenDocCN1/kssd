
## 反汇编代码

```asm
:004C580C E83FD7F3FF              call 00402F50
:004C5811 E802D1F3FF              call 00402918
:004C5816 4F                      dec edi
:004C5817 740F                    je 004C5828
:004C5819 8D5601                  lea edx, dword ptr [esi+01]
:004C581C 8B45F8                  mov eax, dword ptr [ebp-08]
:004C581F E8683DF4FF              call 0040958C
:004C5824 EB02                    jmp 004C5828
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C570B(C)
|
:004C5826 33DB                    xor ebx, ebx
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004C56F5(U), :004C5790(C), :004C5817(C), :004C5824(U)
|
:004C5828 33C0                    xor eax, eax
:004C582A 5A                      pop edx
:004C582B 59                      pop ecx
:004C582C 59                      pop ecx
:004C582D 648910                  mov dword ptr fs:[eax], edx
:004C5830 6858584C00              push 004C5858
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C5856(U)
|
:004C5835 8D8560FEFFFF            lea eax, dword ptr [ebp+FFFFFE60]
:004C583B E8FCF0F3FF              call 0040493C
:004C5840 8D45F8                  lea eax, dword ptr [ebp-08]
:004C5843 E8F4F0F3FF              call 0040493C
:004C5848 8D4508                  lea eax, dword ptr [ebp+08]
:004C584B E8ECF0F3FF              call 0040493C
:004C5850 C3                      ret
:004C5851 E90EEBF3FF              jmp 00404364
:004C5856 EBDD                    jmp 004C5835
:004C5858 8BC3                    mov eax, ebx
:004C585A 5F                      pop edi
:004C585B 5E                      pop esi
:004C585C 5B                      pop ebx
:004C585D 8BE5                    mov esp, ebp
:004C585F 5D                      pop ebp
:004C5860 C20400                  ret 0004
```

以下是 V1.32 版本的：

```asm
\* Referenced by a CALL at Address:
|:004C50ED
|
:004C4B70 55                      push ebp
:004C4B71 8BEC                    mov ebp, esp
:004C4B73 83C4F4                  add esp, FFFFFFF4
:004C4B76 53                      push ebx
:004C4B77 56                      push esi
:004C4B78 57                      push edi
:004C4B79 894DF8                  mov dword ptr [ebp-08], ecx
:004C4B7C 8955FC                  mov dword ptr [ebp-04], edx
:004C4B7F 8B45F8                  mov eax, dword ptr [ebp-08]
:004C4B82 E87502F4FF              call 00404DFC
:004C4B87 33C0                    xor eax, eax
:004C4B89 55                      push ebp
:004C4B8A 68314C4C00              push 004C4C31
:004C4B8F 64FF30                  push dword ptr fs:[eax]
:004C4B92 648920                  mov dword ptr fs:[eax], esp
:004C4B95 8B45F8                  mov eax, dword ptr [ebp-08]
:004C4B98 E87700F4FF              call 00404C14
:004C4B9D 85C0                    test eax, eax
:004C4B9F 7504                    jne 004C4BA5
:004C4BA1 33DB                    xor ebx, ebx
:004C4BA3 EB76                    jmp 004C4C1B
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C4B9F(C)
|
:004C4BA5 33F6                    xor esi, esi
:004C4BA7 85C0                    test eax, eax
:004C4BA9 7E13                    jle 004C4BBE
:004C4BAB BA01000000              mov edx, 00000001
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C4BBC(C)
|
:004C4BB0 8B4DF8                  mov ecx, dword ptr [ebp-08]
:004C4BB3 0FB64C11FF              movzx ecx, byte ptr [ecx+edx-01]
:004C4BB8 03F1                    add esi, ecx
:004C4BBA 42                      inc edx
:004C4BBB 48                      dec eax
:004C4BBC 75F2                    jne 004C4BB0
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C4BA9(C)
|
:004C4BBE 8BC6                    mov eax, esi
:004C4BC0 B9DF000000              mov ecx, 000000DF
:004C4BC5 99                      cdq
:004C4BC6 F7F9                    idiv ecx
:004C4BC8 8BDA                    mov ebx, edx
:004C4BCA 8BC6                    mov eax, esi
:004C4BCC B985000000              mov ecx, 00000085
:004C4BD1 99                      cdq
:004C4BD2 F7F9                    idiv ecx
:004C4BD4 8855F7                  mov byte ptr [ebp-09], dl
:004C4BD7 8BC6                    mov eax, esi
:004C4BD9 B9AF000000              mov ecx, 000000AF
:004C4BDE 99                      cdq
:004C4BDF F7F9                    idiv ecx
:004C4BE1 8BC2                    mov eax, edx
:004C4BE3 33D2                    xor edx, edx
:004C4BE5 8AD3                    mov dl, bl
:004C4BE7 8BF2                    mov esi, edx
:004C4BE9 C1E60C                  shl esi, 0C
:004C4BEC 2BF2                    sub esi, edx
:004C4BEE 33C9                    xor ecx, ecx
:004C4BF0 8A4DF7                  mov cl, byte ptr [ebp-09]
:004C4BF3 8BF9                    mov edi, ecx
:004C4BF5 C1E708                  shl edi, 08
:004C4BF8 2BF9                    sub edi, ecx
:004C4BFA 03F7                    add esi, edi
:004C4BFC 8BF8                    mov edi, eax
:004C4BFE 81E7FF000000            and edi, 000000FF
:004C4C04 6BFF0F                  imul edi, 0000000F
:004C4C07 03F7                    add esi, edi
:004C4C09 03F2                    add esi, edx
:004C4C0B 03F1                    add esi, ecx
:004C4C0D 25FF000000              and eax, 000000FF
:004C4C12 03F0                    add esi, eax
:004C4C14 8BC6                    mov eax, esi
:004C4C16 8B5DFC                  mov ebx, dword ptr [ebp-04]
:004C4C19 33D8                    xor ebx, eax
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C4BA3(U)
|
:004C4C1B 33C0                    xor eax, eax
:004C4C1D 5A                      pop edx
:004C4C1E 59                      pop ecx
:004C4C1F 59                      pop ecx
:004C4C20 648910                  mov dword ptr fs:[eax], edx
:004C4C23 68384C4C00              push 004C4C38
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C4C36(U)
|
:004C4C28 8D45F8                  lea eax, dword ptr [ebp-08]
:004C4C2B E82CFDF3FF              call 0040495C
:004C4C30 C3                      ret
```

---

因为我的机器上这个软件已经注册了，所以我还要首先改一下，如果有和我相同情况的朋友请如下更改：

```asm
004C5767  |.  8A93 30030000 MOV DL,BYTE PTR DS:[EBX+330]------>这个是注册与否得关键标志（DL为1表示已注册）
004C576D  |.  8BC3          MOV EAX,EBX
004C576F  |.  E8 D0FEFFFF   CALL QM.004C5644------------------------->这个调用显示关于对话框
```

## 分析过程

下面的是我的分析过程：

以下是 V1.32 版本的注册码生成过程。

最后返回生成的注册码在 `EAX` 中（跟踪可以发现，在本段中是在 `EBX` 中）。

```asm
\* Referenced by a CALL at Address:
|:004C50ED
|
:004C4B70 55                      push ebp
:004C4B71 8BEC                    mov ebp, esp
:004C4B73 83C4F4                  add esp, FFFFFFF4
:004C4B76 53                      push ebx
:004C4B77 56                      push esi
:004C4B78 57                      push edi
:004C4B79 894DF8                  mov dword ptr [ebp-08], ecx--------->“起名正宗dnpf”
:004C4B7C 8955FC                  mov dword ptr [ebp-04], edx--------->机器码
:004C4B7F 8B45F8                  mov eax, dword ptr [ebp-08]--------->
:004C4B82 E87502F4FF              call 00404DFC
:004C4B87 33C0                    xor eax, eax
:004C4B89 55                      push ebp
:004C4B8A 68314C4C00              push 004C4C31
:004C4B8F 64FF30                  push dword ptr fs:[eax]
:004C4B92 648920                  mov dword ptr fs:[eax], esp
:004C4B95 8B45F8                  mov eax, dword ptr [ebp-08]--------->"起名正宗dnpf"
```
