
```
 00412927: 03FD                      add edi,ebp
 00412929: 668B8567B44000            mov ax,word ptr [ebp+0040B467]
 00412930: 8A26                      mov ah,byte ptr [esi]
 00412932: 3A27                      cmp ah,byte ptr [edi]
 00412934: 7509                      jnz 0041293F
 00412936: 80FC00                    cmp ah,00
 00412939: 7409                      jz 00412944
 0041293B: 46                        inc esi
 0041293C: 47                        inc edi
 0041293D: EBF1                      jmp 00412930
 0041293F: 66B80000                  mov ax,0000
 00412943: C3                        ret
 00412944: 66C7857DB440000100        mov word ptr [ebp+0040B47D],0001 <--40B47D卷信息标志
 0041294D: 66B80100                  mov ax,0001
 00412951: C3                        ret
```

### 非代码区

```
 00412952: 0000                      add byte ptr [eax],al
 00412954: 0000                      add byte ptr [eax],al
 00412956: 0000                      add byte ptr [eax],al
 00412958: 0000                      add byte ptr [eax],al
 0041295A: 0000                      add byte ptr [eax],al
 0041295C: 0000                      add byte ptr [eax],al
 0041295E: 0000                      add byte ptr [eax],al
 00412960: 0000                      add byte ptr [eax],al
 00412962: 0000                      add byte ptr [eax],al
 00412964: 0000                      add byte ptr [eax],al
 00412966: 0000                      add byte ptr [eax],al
 00412968: 0000                      add byte ptr [eax],al
 0041296A: 0000                      add byte ptr [eax],al
 0041296C: 0000                      add byte ptr [eax],al
 0041296E: 0000                      add byte ptr [eax],al
```

### 根据卷信息保存到注册表

```
 00412970: B810000000                mov eax,00000010
 00412975: 50                        push eax            <--nFileSystemNameSize
 00412976: B8B8AD4000                mov eax,0040ADB8
 0041297B: 03C5                      add eax,ebp
 0041297D: 50                        push eax            <--lpFileSystemNameBuffer,
 0041297E: B8C8AD4000                mov eax,0040ADC8
 00412983: 03C5                      add eax,ebp
 00412985: 50                        push eax            <--lpFileSystemFlags,
 00412986: B8CCAD4000                mov eax,0040ADCC
 0041298B: 03C5                      add eax,ebp
 0041298D: 50                        push eax            <--lpMaximumComponentLength,
 0041298E: B8D0AD4000                mov eax,0040ADD0
 00412993: 03C5                      add eax,ebp
 00412995: 50                        push eax            <--lpVolumeSerialNumber,
 00412996: B810000000                mov eax,00000010
 0041299B: 50                        push eax            <--nVolumeNameSize,
 0041299C: B8D4AD4000                mov eax,0040ADD4
 004129A1: 03C5                      add eax,ebp
 004129A3: 50                        push eax            <--lpVolumeNameBuffer,
 004129A4: B8B4AD4000                mov eax,0040ADB4
 004129A9: 03C5                      add eax,ebp          <--412A67
 004129AB: 50                        push eax            <--lpRootPathName, "C:\"
 004129AC: FF95BFBB4000              call dword ptr [ebp+0040BBBF]  <--GetVolumeInformationA
 004129B2: B8A4AD4000                mov eax,0040ADA4
 004129B7: 03C5                      add eax,ebp
 004129B9: BBA8AD4000                mov ebx,0040ADA8
 004129BE: 03DD                      add ebx,ebp          lpTotalNumberOfBytes,
 004129C0: B9ACAD4000                mov ecx,0040ADAC
 004129C5: 03CD                      add ecx,ebp
 004129C7: C701633A5C00              mov dword ptr [ecx],005C3A63
 004129CD: 50                        push eax            lpTotalNumberOfFreeBytes
 004129CE: 53                        push ebx            lpTotalNumberOfBytes,
 004129CF: 6A00                      push 00000000          lpFreeBytesAvailableToCaller,
 004129D1: 51                        push ecx            lpDirectoryName,"C:\"
 004129D2: FF95B7BB4000              call dword ptr [ebp+0040BBB7] <--GetDiskFreeSpaceExA
 004129D8: 8B85A8AD4000              mov eax,dword ptr [ebp+0040ADA8] lpTotalNumberOfBytes,
 004129DE: F7D0                      not eax
 004129E0: 3385D0AD4000              xor eax,dword ptr [ebp+0040ADD0] lpVolumeSerialNumber,
 004129E6: 8985A8AD4000              mov dword ptr [ebp+0040ADA8],eax lpTotalNumberOfBytes,
 004129EC: E844020000                call 00412C35          <--查找注册表，提取数据
 004129F1: B801000000                mov eax,00000001
 004129F6: 33DB                      xor ebx,ebx
 004129F8: BE65B14000                mov esi,0040B165
 004129FD: 03F5                      add esi,ebp          <--412E18,存放Your Name
 004129FF: 8A1E                      mov bl,byte ptr [esi]
 00412A01: 80FB00                    cmp bl,00
 00412A04: 7405                      jz 00412A0B
 00412A06: F7E3                      mul ebx
 00412A08: 46                        inc esi
 00412A09: EBF4                      jmp 004129FF
 00412A0B: 33855DB74000              xor eax,dword ptr [ebp+0040DB75] <--413410,Register Key
 00412A11: 3385A8AD4000              xor eax,dword ptr [ebp+0040ADA8] lpTotalNumberOfBytes,
 00412A17: 86E0                      xchg al,ah
 00412A19: 8BD8                      mov ebx,eax
 00412A1B: BE79B14000                mov esi,0040B179        <--412E2C,存放Serial No
 00412A20: 03F5                      add esi,ebp
 00412A22: E80E000000                call 00412A35        <--数据转换
 00412A27: 3BD8                      cmp ebx,eax        <--比较是否正版
 00412A29: 7405                      jz 00412A30
 00412A2B: 66B80000                  mov ax,0000
 00412A2F: C3                        ret
 00412A30: 66B80100                  mov ax,0001
 00412A34: C3                        ret
```

### 字符转换成数值

```
 00412A35: 53                        push ebx
 00412A36: 56                        push esi
 00412A37: 33DB                      xor ebx,ebx
 00412A39: 8A06                      mov al,byte ptr [esi]
 00412A3B: 3C39                      cmp al,39
 00412A3D: 7606                      jbe 00412A45
 00412A3F: 24CF                      and al,CF
 00412A41: 2C37                      sub al,37
 00412A43: EB02                      jmp 00412A47
 00412A45: 240F                      and al,0F
 00412A47: C1E304                    shl ebx,04
 00412A4A: 0AD8                      or bl,al
 00412A4C: 46                        inc esi
 00412A4D: 803E00                    cmp byte ptr [esi],00
 00412A50: 75E7                      jnz 00412A39
 00412A52: 8BC3                      mov eax,ebx
 00412A54: 5E                        pop esi
 00412A55: 5B                        pop ebx
 00412A56: C3                        ret
```

### 非代码区

```
 00412A57: 0000                      add byte ptr [eax],al
 00412A59: 0000                      add byte ptr [eax],al
 00412A5B: 0000                      add byte ptr [eax],al
 00412A5D: 0000                      add byte ptr [eax],al
 00412A5F: 633A                      arpl word ptr [edx],di
 00412A61: 5C                        pop esp
 00412A62: 00256C580043              add byte ptr [4300586C],ah
 00412A68: 3A5C0000                  cmp bl,byte ptr [eax+eax]
 00412A6C: 0000                      add byte ptr [eax],al
 00412A6E: 0000                      add byte ptr [eax],al
 00412A70: 0000                      add byte ptr [eax],al
 00412A72: 0000                      add byte ptr [eax],al
```
