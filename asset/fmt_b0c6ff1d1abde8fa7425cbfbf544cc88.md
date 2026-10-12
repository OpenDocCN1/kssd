
```
105C8E0B     8B55 08                   mov edx,dword ptr ss:[ebp+8]  
105C8E0E     8B82 F8030000            mov eax,dword ptr ds:[edx+3F8]  
105C8E14     8985 74FEFFFF            mov dword ptr ss:[ebp-18C],eax  
105C8E1A     C785 80FEFFFF 00000000   mov dword ptr ss:[ebp-180],0  
105C8E24     C785 88FEFFFF 00000000   mov dword ptr ss:[ebp-178],0  
105C8E2E     C785 90FEFFFF 00000000   mov dword ptr ss:[ebp-170],0  
105C8E38     C745 FC 00000000         mov dword ptr ss:[ebp-4],0  
105C8E3F     6A 0E                    push 0E  
105C8E41     8B4D 08                 mov ecx,dword ptr ss:[ebp+8]  
105C8E44     51                       push ecx  
105C8E45     E8 A63C0200             call CVI_1.105ECAF0  
105C8E4A     83C4 08                 add esp,8  
105C8E4D     3D ADDE0000             cmp eax,0DEAD  
105C8E52     74 15                   je short CVI_1.105C8E69  
105C8E54     6A 14                   push 14  
105C8E56     8B55 08                 mov edx,dword ptr ss:[ebp+8]  
105C8E59     52                       push edx  
105C8E5A     E8 913C0200             call CVI_1.105ECAF0  
105C8E5F     83C4 08                 add esp,8  
105C8E62     3D 55260000             cmp eax,2655  
105C8E67     75 3E                   jnz short CVI_1.105C8EA7  
105C8E69     8B45 08                 mov eax,dword ptr ss:[ebp+8]  
105C8E6C     0FBE88 0C030000         movsx ecx,byte ptr ds:[eax+30C]  
105C8E73     85C9                    test ecx,ecx  
105C8E75     74 3A                   je short CVI_1.105C8EB1  
105C8E77     68 B8287410             push CVI_1.107428B8                  ; ASCII "demo"  

熟悉的面孔  
......  
105C9156     /74 0C                  je short CVI_1.105C9164  
105C9158     |C785 88FEFFFF 00000000  mov dword ptr ss:[ebp-178],0  
105C9162     |EB 0A                  jmp short CVI_1.105C916E  
105C9164     \C785 88FEFFFF C8287410  mov dword ptr ss:[ebp-178],CVI_1.10742>; ASCII "ok"  
105C916E     E9 7F0D0000             jmp CVI_1.105C9EF2  

......  
105C9D2A     A2 9E358A10             mov byte ptr ds:[108A359E],al  
105C9D2F     33C9                    xor ecx,ecx  
105C9D31     8A0D 9F358A10          mov cl,byte ptr ds:[108A359F]  
105C9D37     33D2                    xor edx,edx  
105C9D39     8A15 98358A10          mov dl,byte ptr ds:[108A3598]  
105C9D3F     03CA                    add ecx,edx  
105C9D41     81E1 FF000000          and ecx,0FF  
105C9D47     81E1 FF000000          and ecx,0FF  
105C9D4D     880D 98358A10          mov byte ptr ds:[108A3598],cl  
105C9D53     33C0                    xor eax,eax  
105C9D55     A0 9E358A10             mov al,byte ptr ds:[108A359E]  
105C9D5A     33C9                    xor ecx,ecx  
105C9D5C     8A0D 99358A10          mov cl,byte ptr ds:[108A3599]  
105C9D62     03C1                    add eax,ecx  
105C9D64     25 FF000000            and eax,0FF  
105C9D69     25 FF000000            and eax,0FF  
105C9D6E     A2 99358A10             mov byte ptr ds:[108A3599],al  
105C9D73     C605 9F358A10 00        mov byte ptr ds:[108A359F],0  
105C9D7A     8A15 9F358A10          mov dl,byte ptr ds:[108A359F]  
105C9D80     8815 9E358A10          mov byte ptr ds:[108A359E],dl  
105C9D86     C785 78FEFFFF 08000000  mov dword ptr ss:[ebp-188],8  
105C9D90     817D 18 37B3D866       cmp dword ptr ss:[ebp+18],66D8B337  
105C9D97     /75 0F                  jnz short CVI_1.105C9DA8  
105C9D99     |8B85 78FEFFFF          mov eax,dword ptr ss:[ebp-188]  
105C9D9F     |83E8 02                sub eax,2  
105C9DA2     |8985 78FEFFFF          mov dword ptr ss:[ebp-188],eax  
105C9DA8     \C785 7CFEFFFF 00000000 mov dword ptr ss:[ebp-184],0  
105C9DB2     EB 0F                   jmp short CVI_1.105C9DC3  
105C9DB4     8B8D 7CFEFFFF           mov ecx,dword ptr ss:[ebp-184]  
105C9DBA     83C1 01                 add ecx,1  
105C9DBD     898D 7CFEFFFF           mov dword ptr ss:[ebp-184],ecx  
105C9DC3     8B95 7CFEFFFF           mov edx,dword ptr ss:[ebp-184]  
105C9DC9     3B95 78FEFFFF           cmp edx,dword ptr ss:[ebp-188]  
105C9DCF     0F8D 02010000          jge CVI_1.105C9ED7  
105C9DD5     8B85 7CFEFFFF           mov eax,dword ptr ss:[ebp-184]  
105C9DDB     8A8C45 9CFEFFFF         mov cl,byte ptr ss:[ebp+eax*2-164]  
105C9DE2     888D 34FEFFFF           mov byte ptr ss:[ebp-1CC],cl  
105C9DE8     0FBE95 34FEFFFF         movsx edx,byte ptr ss:[ebp-1CC]  
105C9DEF     52                       push edx  
105C9DF0     E8 5B300300             call CVI_1.105FCE50  

在 `call CVI_1.105FCE50` 之后, `d 108a359e`  

108A359E  25C00000  ..?  
108A35A2  0000105C  \..  
没看见，滚动数据窗口看到了吧：  

108A3596  62B30000  ..x(  
108A359A  B84926A9  ?I  
看到没有 `SIGN=B362A92649B8`  

呵呵，我估计这就是 `laoqian` 的明码比较办法在 `FLEXnet Licensing v10.1.0` 中的小小改通吧.  

......  
105C9EB0     8B95 30FEFFFF           mov edx,dword ptr ss:[ebp-1D0]  
105C9EB6     81E2 FF000000          and edx,0FF  
105C9EBC     8B85 7CFEFFFF           mov eax,dword ptr ss:[ebp-184]  
105C9EC2     33C9                    xor ecx,ecx  
105C9EC4     8A88 98358A10          mov cl,byte ptr ds:[eax+108A3598]  
105C9ECA     3BD1                    cmp edx,ecx                          ; 看到没有，明码比较  
105C9ECC     74 04                   je short CVI_1.105C9ED2  
105C9ECE     33C0                    xor eax,eax  
105C9ED0     EB 26                   jmp short CVI_1.105C9EF8  
105C9ED2    ^ E9 DDFEFFFF            jmp CVI_1.105C9DB4  
105C9ED7     8B55 18                 mov edx,dword ptr ss:[ebp+18]  
105C9EDA     52                       push edx  
105C9EDB     68 98358A10             push CVI_1.108A3598  
105C9EE0     8B45 08                 mov eax,dword ptr ss:[ebp+8]  
105C9EE3     50                       push eax  
105C9EE4     E8 E9000000             call CVI_1.105C9FD2  
105C9EE9     83C4 0C                 add esp,0C                         ; 返回真正的 SIGN  
105C9EEC     8985 88FEFFFF           mov dword ptr ss:[ebp-178],eax  
105C9EF2     8B85 88FEFFFF           mov eax,dword ptr ss:[ebp-178]  
105C9EF8     5B                       pop ebx  
105C9EF9     8BE5                    mov esp,ebp  
105C9EFB     5D                       pop ebp  
105C9EFC     C3                       retn 
```

`4dd466d0e1e0cc123afbe504da2462ca`
