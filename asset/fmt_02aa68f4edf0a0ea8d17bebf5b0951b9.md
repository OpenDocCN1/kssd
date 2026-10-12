
:0040817C 778D  ja 0040810B //循环   
:0040817E 8BE5  mov esp, ebp   
:00408180 5D  pop ebp   
:00408181 C3  ret   

7、比较真假 RegCode：   

:004227E4 33C0  xor eax, eax   
:004227E6 8945F0  mov dword ptr [ebp-10], eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0042280F(C)   
|   
:004227E9 8B45F0  mov eax, dword ptr [ebp-10]   
:004227EC 8A4405C9  mov al, byte ptr [ebp+eax-37]   
:004227F0 8B55F0  mov edx, dword ptr [ebp-10]   
:004227F3 8B4DFC  mov ecx, dword ptr [ebp-04]   
:004227F6 3A441168  cmp al, byte ptr [ecx+edx+68] //比较一个字节   
:004227FA 7506  jne 00422802   
:004227FC C645DF01  mov [ebp-21], 01 //good guy    
:00422800 EB06  jmp 00422808   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004227FA(C)   
|   
:00422802 C645DF00  mov [ebp-21], 00 //bad guy    
:00422806 EB09  jmp 00422811   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00422800(U)   
|   
:00422808 FF45F0  inc [ebp-10]   
:0042280B 837DF00A  cmp dword ptr [ebp-10], 0000000A //循环比较 10 个字节   
:0042280F 75D8  jne 004227E9   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00422806(U)   
|   
:00422811 8B45FC  mov eax, dword ptr [ebp-04]   
:00422814 C6407300  mov [eax+73], 00   
:00422818 8B45FC  mov eax, dword ptr [ebp-04]   
:0042281B C6407201  mov [eax+72], 01   
:0042281F 807DDF00  cmp byte ptr [ebp-21], 00 //bag guy?   
:00422823 740A  je 0042282F   

* Possible StringData Ref from Code Obj ->"sdf"   
                  |   
:00422825 686C284200  push 0042286C //注册与否的标志！   

* Reference To: kernel32.AddAtomA, Ord:0000h   
                  |   
:0042282A E8191EFEFF  Call 00404648   

可见关键的断点是 `GetAtomNameA()`。   

`BF 4dd466d0e1e0cc123afbe504da2462ca`
