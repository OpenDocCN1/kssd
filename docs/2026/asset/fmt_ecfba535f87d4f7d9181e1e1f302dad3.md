
```
> :00453D9D 8D55E0
> lea edx, dword ptr [ebp-20]
> :00453DA0 8D45EC
> lea eax, dword ptr [ebp-14]
> :00453DA3 E8E000FBFF              call 00403E88
>
> :00453DA8 8B45EC
> mov eax, dword ptr [ebp-14]
> :00453DAB 8D55F0
> lea edx, dword ptr [ebp-10]
> :00453DAE E8D1FBFFFF              call 00453984
>
> :00453DB3 8D45EC
> lea eax, dword ptr [ebp-14]
> :00453DB6 50
>   push eax
> :00453DB7 B904000000              mov ecx,
> 00000004
> :00453DBC BA0B000000              mov edx,
> 0000000B
> :00453DC1 8B45FC
> mov eax, dword ptr [ebp-04]
> :00453DC4 E81F03FBFF              call 004040E8
>
> :00453DC9 8B45EC
> mov eax, dword ptr [ebp-14]
> :00453DCC 50
>   push eax
> :00453DCD 8D4DE8
> lea ecx, dword ptr [ebp-18]
> :00453DD0 8B45F0
> mov eax, dword ptr [ebp-10]
> :00453DD3 0FB600
> movzx eax, byte ptr [eax]
> :00453DD6 BA02000000              mov edx,
> 00000002
> :00453DDB E8184FFBFF              call 00408CF8
>
> :00453DE0 8D45E8
> lea eax, dword ptr [ebp-18]
> :00453DE3 50
>   push eax
> :00453DE4 8D4DD8
> lea ecx, dword ptr [ebp-28]
> :00453DE7 8B45F0
> mov eax, dword ptr [ebp-10]
> :00453DEA 0FB64001                movzx
> eax, byte ptr [eax+01]
> :00453DEE BA02000000              mov edx,
> 00000002
> :00453DF3 E8004FFBFF              call 00408CF8
>
> :00453DF8 8B55D8
> mov edx, dword ptr [ebp-28]
> :00453DFB 58
>   pop eax
> :00453DFC E8EB00FBFF              call 00403EEC
>           <-生成C31 C32 C33 C34
> :00453E01 8B55E8
> mov edx, dword ptr [ebp-18]    <-真的C31 C32 C33 C34 BPX6
> :00453E04 58
>   pop eax            <-假的C31
> C32 C33 C34
> :00453E05 E8EA01FBFF              call 00403FF4
>
> :00453E0A 0F94C3
> sete bl
>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00453D3C(C)
> |
> :00453E0D 84DB
>   test bl, bl
> :00453E0F 7460
>   je 00453E71            <-去死
>
> :00453E11 8D45EC
> lea eax, dword ptr [ebp-14]
> :00453E14 50
>   push eax
> :00453E15 B902000000              mov ecx,
> 00000002
> :00453E1A BA0F000000              mov edx,
> 0000000F
> :00453E1F 8B45FC
> mov eax, dword ptr [ebp-04]
> :00453E22 E8C102FBFF              call 004040E8
>
> :00453E27 8B45EC
> mov eax, dword ptr [ebp-14]
> :00453E2A 50
>   push eax
> :00453E2B 8B45F8
> mov eax, dword ptr [ebp-08]
> :00453E2E 8A00
>   mov al, byte ptr [eax]
> :00453E30 E82FECFAFF              call 00402A64
>
> :00453E35 8BD0
>   mov edx, eax
> :00453E37 8D45E4
> lea eax, dword ptr [ebp-1C]
> :00453E3A 885001
> mov byte ptr [eax+01], dl
> :00453E3D C60001
> mov byte ptr [eax], 01
> :00453E40 8D55E4
> lea edx, dword ptr [ebp-1C]
> :00453E43 8D45E0
> lea eax, dword ptr [ebp-20]
> :00453E46 E809EDFAFF              call 00402B54
>
>
> \* Possible StringData Ref from Data Obj ->"-"
>
>             |
> :00453E4B BA683F4500              mov edx,
> 00453F68
> :00453E50 8D45E0
> lea eax, dword ptr [ebp-20]
> :00453E53 B102
>   mov cl, 02
> :00453E55 E8CAECFAFF              call 00402B24
>
> :00453E5A 8D55E0
> lea edx, dword ptr [ebp-20]
> :00453E5D 8D45E8
> lea eax, dword ptr [ebp-18]
> :00453E60 E82300FBFF              call 00403E88
>           <-生成C35  NAME的第一个字母
>
> :00453E65 8B55E8
> mov edx, dword ptr [ebp-18]    <-真的C35  BPX7
> :00453E68 58
>   pop eax            <-假的C35
>
> :00453E69 E88601FBFF              call 00403FF4
>
> :00453E6E 0F94C3
> sete bl
>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00453E0F(C)
> |
> :00453E71 84DB
>   test bl, bl
> :00453E73 747B
>   je 00453EF0            <-去死
>
> :00453E75 8D45EC
> lea eax, dword ptr [ebp-14]
> :00453E78 50
>   push eax
> :00453E79 B902000000              mov ecx,
> 00000002
> :00453E7E BA01000000              mov edx,
> 00000001
> :00453E83 8B45F8
> mov eax, dword ptr [ebp-08]
> :00453E86 E85D02FBFF              call 004040E8
>
> :00453E8B 8B45EC
> mov eax, dword ptr [ebp-14]
> :00453E8E 8D55F0
> lea edx, dword ptr [ebp-10]
> :00453E91 E8EEFAFFFF              call 00453984
>
> :00453E96 8D45EC
> lea eax, dword ptr [ebp-14]
> :00453E99 50
>   push eax
> :00453E9A B909000000              mov ecx,
> 00000009
> :00453E9F BA11000000              mov edx,
> 00000011
> :00453EA4 8B45FC
> mov eax, dword ptr [ebp-04]
> :00453EA7 E83C02FBFF              call 004040E8
>
> :00453EAC 8B45EC
> mov eax, dword ptr [ebp-14]
> :00453EAF 50
>   push eax
> :00453EB0 8D4DE8
> lea ecx, dword ptr [ebp-18]
> :00453EB3 8B45F0
> mov eax, dword ptr [ebp-10]
> :00453EB6 0FB600
> movzx eax, byte ptr [eax]
> :00453EB9 BA02000000              mov edx,
> 00000002
> :00453EBE E8354EFBFF              call 00408CF8
>
> :00453EC3 8D45E8
> lea eax, dword ptr [ebp-18]
> :00453EC6 50
>   push eax
> :00453EC7 8D4DD8
> lea ecx, dword ptr [ebp-28]
> :00453ECA 8B45F0
> mov eax, dword ptr [ebp-10]
> :00453ECD 0FB64001                movzx
> eax, byte ptr [eax+01]
> :00453ED1 BA02000000              mov edx,
> 00000002
> :00453ED6 E81D4EFBFF              call 00408CF8
>
> :00453EDB 8B55D8
> mov edx, dword ptr [ebp-28]
> :00453EDE 58
>   pop eax
> :00453EDF E80800FBFF              call 00403EEC
>           <-生成C41 C42 C43 C44
> :00453EE4 8B55E8
> mov edx, dword ptr [ebp-18]    <-真的C41 C42 C43 C44 BPX8
> :00453EE7 58
>   pop eax            <-假的C41
> C42 C43 C44
> :00453EE8 E80701FBFF              call 00403FF4
>
> :00453EED 0F94C3
> sete bl
>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00453E73(C)
> |
> :00453EF0 8B45F4
> mov eax, dword ptr [ebp-0C]
> :00453EF3 8818
>   mov byte ptr [eax], bl
> :00453EF5 33C0
>   xor eax, eax
> :00453EF7 5A
>   pop edx
> :00453EF8 59
>   pop ecx
> :00453EF9 59
>   pop ecx
> :00453EFA 648910
> mov dword ptr fs:[eax], edx
> :00453EFD 682D3F4500              push 00453F2D
>
>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
> |:00453F2B(U)
> |
> :00453F02 8D45D8
> lea eax, dword ptr [ebp-28]
> :00453F05 E85EFDFAFF              call 00403C68
>
> :00453F0A 8D45E8
> lea eax, dword ptr [ebp-18]
> :00453F0D BA03000000              mov edx,
> 00000003
> :00453F12 E875FDFAFF              call 00403C8C
>
> :00453F17 8D45F8
> lea eax, dword ptr [ebp-08]
>
> \* Possible StringData Ref from Data Obj ->"TRGRegisterInfo"
>
>             |
> :00453F1A 8B1554394500            mov edx, dword
> ptr [00453954]
> :00453F20 E8A706FBFF              call 004045CC
>
> :00453F25 C3
>   ret
>
>
> :00453F26 E9FDF7FAFF              jmp 00403728
>
> :00453F2B EBD5
>   jmp 00453F02
> :00453F2D 5F
>   pop edi
> :00453F2E 5E
>   pop esi
> :00453F2F 5B
>   pop ebx
> :00453F30 8BE5
>   mov esp, ebp
> :00453F32 5D
>   pop ebp
> :00453F33 C3
>   ret
```

① 至此我们已经得到了除C23 C24的全部注册码,现在我们将得到C23 C24的注册码:

重新运行程序设断点 00453C9E

D EDX

就是正确的 C23 C24

C23 C24 由后面的注册码生成,因此要等到后面的注册码确定后才可以确定

ABCDEFGHIJKLMN          BM1-2486B-3157A-7878
            BM1-E086B-3157A-7878
  <-C21 C22
            BM1-E003B-3157A-7878
  <-C23 C24
            BM1-E003B-EDDAA-7878
  <-C31 C32
            BM1-E0C5B-EDDAA-7878
  <-C23 C24
            BM1-E0C5B-EDDAA-AF71
  <-C41 C42 C43 C44
            BM1-E0D2B-EDDAA-AF71
  <-C23 C24
至此注册成功

QWERTYUIOPASDFG        BM1-1234W-4567Q-7878
            BM1-E134W-4567Q-7878

            BM1-E1F7W-4567Q-7878

            BM1-E1F7W-EABDQ-7878

            BM1-E1A1W-EABDQ-7878
  <-出问题了 中断在C21 C22处不走了？只好BD * ？？？
            BM1-E1A1W-EABDQ-BF64

            BM1-E18DW-EABDQ-BF64
