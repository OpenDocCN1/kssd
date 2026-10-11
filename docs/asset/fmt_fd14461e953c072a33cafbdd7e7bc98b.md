
```
.text:00404EA2           mov     eax, [ebp+var_28]  
.text:00404EA5           cmp     [eax-8], edi  
.text:00404EA8           jnz     short loc_404EB1  
.text:00404EAA           mov     edi, esi  
.text:00404EAC           jmp     loc_40552E      ; 跳向获取环境变量错误  
.text:00404EB1 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:00404EB1  
.text:00404EB1 loc_404EB1:                          ; CODE XREF: sub_404D84+124j  
.text:00404EB1           push    ebx  
.text:00404EB2           lea     eax, [ebp+var_38] ; 解密结果为 neoqaz2.dll  
.text:00404EB5           push    offset aSn        ; "推桃沦憤窍?  
.text:00404EBA           push    eax  
.text:00404EBB           call    sub_432191        ; 字符串解密，参数：地址，字符串，屏蔽字  
.text:00404EC0           push    dword ptr [eax]  
.text:00404EC2           lea     eax, [ebp+var_28]  
.text:00404EC5           push    eax  
.text:00404EC6           lea     eax, [ebp+Type]  
.text:00404EC9           push    eax  
.text:00404ECA           call    sub_42FDC6  
.text:00404ECF           add     esp, 18h  
.text:00404ED2           lea     ecx, [ebp+lpFileName]  
.text:00404ED5           push    eax  
.text:00404ED6           call    sub_40169C  
.text:00404EDB           lea     ecx, [ebp+Type]  
.text:00404EDE           call    sub_401541        ; delete[]和 new 对应  
.text:00404EE3           lea     ecx, [ebp+var_38]  
.text:00404EE6           call    sub_401541  
.text:00404EEB           push    0                 ; hTemplateFile  
.text:00404EED           mov     esi, ds:CreateFileW  
.text:00404EF3           push    80h               ; dwFlagsAndAttributes  
.text:00404EF8           push    3                 ; dwCreationDisposition  
.text:00404EFA           push    0                 ; lpSecurityAttributes  
.text:00404EFC           mov     edi, 80000000h  
.text:00404F01           push    3                 ; dwShareMode  
.text:00404F03           push    edi               ; dwDesiredAccess  
.text:00404F04           push    [ebp+lpFileName]  ; lpFileName  
.text:00404F07           call    esi ; CreateFileW  
.text:00404F09           cmp     eax, 0FFFFFFFFh  
.text:00404F0C           mov     [ebp+hFile], eax  
.text:00404F0F           jz      short loc_404F77  
.text:00404F11           lea     eax, [ebp+NumberOfBytesRead]  
.text:00404F14           push    0                 ; lpOverlapped  
.text:00404F16           push    eax               ; lpNumberOfBytesRead  
.text:00404F17           lea     eax, [ebp+Buffer]  
.text:00404F1D           push    0FFFh             ; nNumberOfBytesToRead  
.text:00404F22           push    eax               ; lpBuffer  
.text:00404F23           push    [ebp+hFile]       ; hFile  
.text:00404F26           mov     [ebp+var_4], 9  
.text:00404F2D           call    ds:ReadFile  
.text:00404F33           test    eax, eax  
.text:00404F35           jz      short loc_404F67  
.text:00404F37           mov     ecx, [ebp+NumberOfBytesRead]  
.text:00404F3A           cmp     ecx, 8  
.text:00404F3D           jb      short loc_404F67  
.text:00404F3F           lea     ecx, [ebp+ecx+Buffer]  
.text:00404F46           lea     edx, [ebp+Buffer]  
.text:00404F4C           cmp     edx, ecx  
.text:00404F4E           lea     eax, [ebp+Buffer]  
.text:00404F54           jnb     short loc_404F5E  
.text:00404F56  
.text:00404F56 loc_404F56:                          ; CODE XREF: sub_404D84+1D8j  
.text:00404F56           xor     byte ptr [eax], 5Dh  
.text:00404F59           inc     eax  
.text:00404F5A           cmp     eax, ecx  
.text:00404F5C           jb      short loc_404F56  
.text:00404F5E  
.text:00404F5E loc_404F5E:                          ; CODE XREF: sub_404D84+1D0j  
.text:00404F5E           mov     [ebp+var_40], 1  
.text:00404F65           jmp     short loc_404F6E  
.text:00404F67 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:00404F67  
.text:00404F67 loc_404F67:                          ; CODE XREF: sub_404D84+1B1j  
.text:00404F67                                   ; sub_404D84+1B9j  
.text:00404F67           mov     [ebp+var_4], 19h  
.text:00404F6E  
.text:00404F6E loc_404F6E:                          ; CODE XREF: sub_404D84+1E1j  
.text:00404F6E           push    [ebp+hFile]       ; hObject  
.text:00404F71           call    ds:CloseHandle  
.text:00404F77  
.text:00404F77 loc_404F77:                          ; CODE XREF: sub_404D84+18Bj  
.text:00404F77           push    1  
.text:00404F79           push    0  
.text:00404F7B           lea     ecx, [ebp+var_68]  
.text:00404F7E           call    Sub_ReadReg     ; 读取注册信息  
.text:00404F83           test    eax, eax  
.text:00404F85           mov     [ebp+hKey], eax  
.text:00404F88           jnz     short loc_404F9B  
.text:00404F8A           push    eax  
.text:00404F8B           push    eax  
.text:00404F8C           lea     ecx, [ebp+var_68]  
.text:00404F8F           call    Sub_ReadReg     ; 读取注册信息  
.text:00404F94           test    eax, eax  
.text:00404F96           mov     [ebp+hKey], eax  
.text:00404F99           jz      short loc_405002  
.text:00404F9B  
.text:00404F9B loc_404F9B:                          ; CODE XREF: sub_404D84+204j  
.text:00404F9B           lea     eax, [ebp+cbData]  
.text:00404F9E           and     [ebp+Type], 0  
.text:00404FA2           push    eax               ; lpcbData  
.text:00404FA3           lea     eax, [ebp+Data]  
.text:00404FA9           push    eax               ; lpData  
.text:00404FAA           lea     eax, [ebp+Type]  
.text:00404FAD           push    eax               ; lpType  
.text:00404FAE           push    0                 ; lpReserved  
.text:00404FB0           push    [ebp+lpValueName] ; lpValueName  
.text:00404FB3           mov     [ebp+cbData], 0FFFh  
.text:00404FBA           push    [ebp+hKey]        ; hKey  
.text:00404FBD           call    ds:RegQueryValueExW  
.text:00404FC3           test    eax, eax  
.text:00404FC5           jnz     short loc_405006  
.text:00404FC7           or      [ebp+var_4], 21h  
.text:00404FCB           cmp     [ebp+cbData], 8  
.text:00404FCF           jb      short loc_404FFC  
.text:00404FD1           mov     ecx, [ebp+cbData]  
.text:00404FD4           lea     edx, [ebp+Data]  
.text:00404FDA           lea     eax, [ebp+Data]  
.text:00404FE0           lea     ecx, [ebp+ecx+Data]  
.text:00404FE7           cmp     edx, ecx  
.text:00404FE9           jnb     short loc_404FF3  
.text:00404FEB  
.text:00404FEB loc_404FEB:                          ; CODE XREF: sub_404D84+26Dj  
.text:00404FEB           xor     byte ptr [eax], 12h  
.text:00404FEE           inc     eax  
.text:00404FEF           cmp     eax, ecx  
.text:00404FF1           jb      short loc_404FEB  
.text:00404FF3  
.text:00404FF3 loc_404FF3:                          ; CODE XREF: sub_404D84+265j  
.text:00404FF3           mov     [ebp+var_44], 1  
.text:00404FFA           jmp     short loc_405006  
.text:00404FFC ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?  
.text:00404FFC  
.text:00404FFC loc_404FFC:                          ; CODE XREF: sub_404D84+24Bj  
```
