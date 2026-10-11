
```
.text:004926CF       mov eax, [ebp+var_4]  
.text:004926D2       lea edx, [ebx+ebx]  
.text:004926D5       mov [ebp+nNumberOfBytesToRead], edx  
.text:004926D8       lea ecx, [eax+eax]  
.text:004926DB  
.text:004926DB loc_4926DB:                 ; CODE XREF: sub_492570+F9j  
.text:004926DB       mov eax, [ebp+NumberOfBytesRead]  
.text:004926DE       dec eax  
.text:004926DF       mov [ebp+NumberOfBytesRead], eax  
.text:004926E2       jnz loc_492635  
//循环计算检验值  
.text:004926E8       mov eax, [ebp+var_14]  
.text:004926EB       xor ecx, ecx  
.text:004926ED       mov esi, [ebp+lpMem]  
.text:004926F0       mov edi, [ebp+var_18]  
.text:004926F3       mov ch, bl  
.text:004926F5       mov cl, byte ptr [ebp+var_4]  
.text:004926F8       add eax, ecx  
.text:004926FA       mov [ebp+var_14], eax  
.text:004926FD       mov ebx, eax 

使用 UPX-Ripper 自动脱壳后调试会有一个内存写入异常  

.text:00604EC7 loc_604EC7:         ; CODE XREF: sub_604540+A11j  
.text:00604EC7       xor edx, edx  
.text:00604EC9       mov eax, 80808081h  
.text:00604ECE       mov dl, [ebx]  
.text:00604ED0       imul edx, [esp+228h+nHeight]  
.text:00604ED5       mov [esp+228h+var_20C], edx  
.text:00604ED9       imul edx  
.text:00604EDB       add edx, [esp+228h+var_20C]  
.text:00604EDF       sar edx, 7  
.text:00604EE2       mov eax, edx  
.text:00604EE4       shr eax, 1Fh  
.text:00604EE7       add edx, eax  
.text:00604EE9       mov al, [edi]  
.text:00604EEB       add dl, al  
.text:00604EED       mov eax, 80808081h  
.text:00604EF2       mov [ecx], dl  
//ECX=00740060       内存写入异常   
.text:00604EF4       inc ecx  
.text:00604EF5       inc edi  
.text:00604EF6       inc ebx  
.text:00604EF7       xor edx, edx  
.text:00604EF9       mov dl, [ebx]  
.text:00604EFB       imul edx, [esp+228h+var_208]  
.text:00604F00       mov [esp+228h+var_20C], edx  
.text:00604F04       imul edx  
.text:00604F06       add edx, [esp+228h+var_20C]  
.text:00604F0A       sar edx, 7  
.text:00604F0D       mov eax, edx  
.text:00604F0F       shr eax, 1Fh  
.text:00604F12       add edx, eax  
.text:00604F14       mov al, [edi]  
.text:00604F16       add dl, al  
.text:00604F18       mov eax, 80808081h  
.text:00604F1D       mov [ecx], dl  
.text:00604F1F       inc ecx  
.text:00604F20       inc edi  
.text:00604F21       inc ebx  
.text:00604F22       xor edx, edx  
.text:00604F24       mov dl, [ebx]  
.text:00604F26       imul edx, [esp+228h+lpString]  
.text:00604F2B       mov [esp+228h+var_20C], edx  
.text:00604F2F       imul edx  
.text:00604F31       add edx, [esp+228h+var_20C]  
.text:00604F35       sar edx, 7  
.text:00604F38       mov eax, edx  
.text:00604F3A       shr eax, 1Fh  
.text:00604F3D       add edx, eax  
.text:00604F3F       mov al, [edi]  
.text:00604F41       add dl, al  
.text:00604F43       mov eax, [esp+228h+var_200]  
.text:00604F47       mov [ecx], dl  
.text:00604F49       inc ecx  
.text:00604F4A       inc edi  
.text:00604F4B       inc ebx  
.text:00604F4C       dec eax  
.text:00604F4D       mov [esp+228h+var_200], eax  
.text:00604F51       jnz loc_604EC7  
```

原来`UPX-Ripper`把脱壳文件的`.rsrc`区段属性设置为`40000040`，修改为`C0000040`（可读可写）就行了。 

`4dd466d0e1e0cc123afbe504da2462ca`
