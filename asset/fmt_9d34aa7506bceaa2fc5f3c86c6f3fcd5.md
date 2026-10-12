
```asm
and edx, 0000001Fh  
mov dwSum , edx  
pop esi  
pop edx  
pop ecx  
pop eax  
}  

szSNPro[0x3] = (char)dwSum;  

_asm  
{  
push eax  
push ecx  
push edx  
push esi  
mov ecx, 00000038Ch  
xor eax, eax  

L_Label7:  
movsx edx, byte ptr [eax+szUserPro]  
cmp eax, 00000009h  
jge L_Label8  
lea edx, dword ptr [edx+2*edx]  
lea ecx, dword ptr [ecx+2*edx]  
jmp L_Label9  

L_Label8:  
lea esi, dword ptr [4*edx+00000000h]  
sub esi, edx  
neg esi  
add ecx, esi  

L_Label9:  
inc eax  
cmp eax, 00000010h  
jl L_Label7  
and ecx, 0000001Fh  
mov dwSum , ecx  
pop esi  
pop edx  
pop ecx  
pop eax  
}  
```

`szSNPro[0x3] = (char)dwSum;`

```c
dwSum = 0x49;  
for (i = 0 ; i < 0x10 ; i ++)  
    dwSum += szUserPro[i];  

for (i = 0 ; i < 0x11 ; i ++)  
{  
    dwSum += szProTable[szSNPro[i]];  
}  

szSNPro[0x11] = (char)(dwSum & 0x1f);  

dwSum = 0x32;  
for (i = 0 ; i < 0x10; i++)  
    dwSum -= szUserPro[i];  

for (i = 0 ; i < 0x12; i ++)  
    dwSum += szProTable[szSNPro[i]];  

szSNPro[0x12] = (char)(dwSum & 0x1f);  

dwSum = 0x79;  
for (i = 0 ; i < 0x13; i ++)  
    dwSum -= szProTable[szSNPro[i]];  

szSNPro[0x13] = (char)(dwSum & 0x1f);  

for (int k = 0x0 ; k < 0x14 ; k++)  
    lpSN[k] = szProTable[szSNPro[k]];  

lpSN[0x14] = '\0';  

return TRUE;  
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
