
```
.text:006F3C82 loc_6F3C82:                                         
.text:006F3C82                  inc     eax  
.text:006F3C83                  cmp     eax, 26  
.text:006F3C86                  jle     short loc_6F3C60 ; 21-26,必须为 0  
.text:006F3C88                  cmp     edx, edi  
.text:006F3C8A                  mov     eax, [esp+5Ch+var_40]  
.text:006F3C8E                  mov     [eax], edx     ; 保存结果  
.text:006F3C90                  jnz     loc_6F3E40  
.text:006F3C96                  xor     eax, eax  
.text:006F3C98                  xor     ecx, ecx  
.text:006F3C9A                  lea     ebx, [ebx+0]  
.text:006F3CA0  

.text:006F3CA0 loc_6F3CA0:                                        
.text:006F3CA0                  cmp     ecx, edi  
.text:006F3CA2                  jl      failed_6F3EA9  
.text:006F3CA8                  cmp     ecx, ebx  
.text:006F3CAA                  jge     failed_6F3EA9  
.text:006F3CB0                  cmp     byte ptr [esi+ecx], 0  
.text:006F3CB4                  jz      short loc_6F3CBF  
.text:006F3CB6                  mov     edx, 1  
.text:006F3CBB                  shl     edx, cl  
.text:006F3CBD                  or      eax, edx  
.text:006F3CBF  

.text:006F3CBF loc_6F3CBF:                                        
.text:006F3CBF                  inc     ecx  
.text:006F3CC0                  cmp     ecx, 4         ; 0-4  
.text:006F3CC3                  jle     short loc_6F3CA0  
.text:006F3CC5                  mov     ebp, [esp+5Ch+var_40]  
.text:006F3CC9                  mov     [ebp+1Ch], eax  ; 保存结果  
.text:006F3CCC                  xor     edi, edi  
.text:006F3CCE                  mov     eax, 5  
.text:006F3CD3  

.text:006F3CD3 loc_6F3CD3:                                         
.text:006F3CD3                  test    eax, eax  
.text:006F3CD5                  jl      failed_6F3EA9  
.text:006F3CDB                  cmp     eax, ebx  
.text:006F3CDD                  jge     failed_6F3EA9  
.text:006F3CE3                  cmp     byte ptr [esi+eax], 0  
.text:006F3CE7                  jz      short loc_6F3CF5  
.text:006F3CE9                  lea     ecx, [eax-5]  
.text:006F3CEC                  mov     edx, 1  
.text:006F3CF1                  shl     edx, cl  
.text:006F3CF3                  or      edi, edx  
.text:006F3CF5  

.text:006F3CF5 loc_6F3CF5:                                        
.text:006F3CF5                  inc     eax  
.text:006F3CF6                  cmp     eax, 8         ; 5-8，主版本号  
.text:006F3CF9                  jle     short loc_6F3CD3  
.text:006F3CFB                  xor     esi, esi  
.text:006F3CFD                  mov     eax, 9  
.text:006F3D02  

.text:006F3D02 loc_6F3D02:                                        
.text:006F3D02                  test    eax, eax  
.text:006F3D04                  jl      failed_6F3EA9  
.text:006F3D0A                  cmp     eax, ebx  
.text:006F3D0C                  jge     failed_6F3EA9  
.text:006F3D12                  mov     ecx, [esp+5Ch+var_30_pStr]  
.text:006F3D16                  cmp     byte ptr [ecx+eax], 0  
.text:006F3D1A                  jz      short loc_6F3D28  
.text:006F3D1C                  lea     ecx, [eax-9]  
.text:006F3D1F                  mov     edx, 1  
.text:006F3D24                  shl     edx, cl  
.text:006F3D26                  or      esi, edx  
.text:006F3D28  

.text:006F3D28 loc_6F3D28:                                        
.text:006F3D28                  inc     eax  
.text:006F3D29                  cmp     eax, 12        ; 9-12，从版本号  
.text:006F3D2C                  jle     short loc_6F3D02  
.text:006F3D2E                  xor     edx, edx  
.text:006F3D30                  mov     eax, 13  
.text:006F3D35  

.text:006F3D35 loc_6F3D35:                                        
.text:006F3D35                  test    eax, eax  
.text:006F3D37                  jl      failed_6F3EA9  
.text:006F3D3D                  cmp     eax, ebx  
.text:006F3D3F                  jge     failed_6F3EA9  
.text:006F3D45                  mov     ecx, [esp+5Ch+var_30_pStr]  
.text:006F3D49                  cmp     byte ptr [ecx+eax], 0  
.text:006F3D4D                  jz      short loc_6F3D5F  
.text:006F3D4F                  lea     ecx, [eax-13]  
.text:006F3D52                  mov     ebx, 1  
.text:006F3D57                  shl     ebx, cl  
.text:006F3D59                  or      edx, ebx  
.text:006F3D5B                  mov     ebx, [esp+5Ch+var_2C_Len]  
.text:006F3D5F  

.text:006F3D5F loc_6F3D5F:                                        
.text:006F3D5F                  inc     eax  
.text:006F3D60                  cmp     eax, 16        ; 13-16，从版本号  
.text:006F3D63                  jle     short loc_6F3D35  
.text:006F3D65                  push    edx  
.text:006F3D66                  push    esi  
.text:006F3D67                  push    edi  
.text:006F3D68                  lea     ecx, [ebp+8]  
.text:006F3D6B                  call    sub_61E140  
.text:006F3D70                  mov     esi, [esp+5Ch+var_30_pStr]  
.text:006F3D74                  xor     edx, edx  
.text:006F3D76                  mov     eax, 17  
.text:006F3D7B                  jmp     short loc_6F3D80  
.text:006F3D7D                  lea     ecx, [ecx+0]  
.text:006F3D80  

.text:006F3D80 loc_6F3D80:                                        
.text:006F3D80                  test    eax, eax  
.text:006F3D82                  jl      failed_6F3EA9  
.text:006F3D88                  cmp     eax, ebx  
.text:006F3D8A                  jge     failed_6F3EA9  
.text:006F3D90                  cmp     byte ptr [esi+eax], 0  
.text:006F3D94                  jz      short loc_6F3DA2  
.text:006F3D96                  lea     ecx, [eax-11h]  
.text:006F3D99                  mov     edi, 1  
.text:006F3D9E                  shl     edi, cl  
.text:006F3DA0                  or      edx, edi  
.text:006F3DA2  

.text:006F3DA2 loc_6F3DA2:                                        
.text:006F3DA2                  inc     eax  
.text:006F3DA3                  cmp     eax, 20        ; 17-20 协议类型  
.text:006F3DA6                  jle     short loc_6F3D80  
.text:006F3DA8                  push    edx  
.text:006F3DA9                  lea     ecx, [ebp+14h]  
.text:006F3DAC                  call    GetLicType_61E070  
.text:006F3DB1                  xor     edx, edx  
.text:006F3DB3                  mov     eax, 27  
.text:006F3DB8  

.text:006F3DB8 loc_6F3DB8:                                        
.text:006F3DB8                  test    eax, eax  
.text:006F3DBA                  jl      failed_6F3EA9  
.text:006F3DC0                  cmp     eax, ebx  
.text:006F3DC2                  jge     failed_6F3EA9  
.text:006F3DC8                  cmp     byte ptr [esi+eax], 0  
.text:006F3DCC                  jz      short loc_6F3DDA  
.text:006F3DCE                  lea     ecx, [eax-1Bh]  
.text:006F3DD1                  mov     edi, 1  
.text:006F3DD6                  shl     edi, cl  
.text:006F3DD8                  or      edx, edi  
.text:006F3DDA  

.text:006F3DDA loc_6F3DDA:                                        
.text:006F3DDA                  inc     eax  
.text:006F3DDB                  cmp     eax, 31        ; 27-31，必须为 1  
.text:006F3DDE                  jle     short loc_6F3DB8  
.text:006F3DE0                  mov     [ebp+4], edx  
.text:006F3DE3                  xor     eax, eax  
.text:006F3DE5                  mov     edx, 32  
.text:006F3DEA                  lea     ebx, [ebx+0]  
.text:006F3DF0  

.text:006F3DF0 loc_6F3DF0:                                        
.text:006F3DF0                  test    edx, edx  
.text:006F3DF2                  jl      failed_6F3EA9  
.text:006F3DF8                  cmp     edx, ebx  
.text:006F3DFA                  jge     failed_6F3EA9  
.text:006F3E00                  cmp     byte ptr [esi+edx], 0  
.text:006F3E04                  jz      short loc_6F3E12  
.text:006F3E06                  lea     ecx, [edx-32]  
```
