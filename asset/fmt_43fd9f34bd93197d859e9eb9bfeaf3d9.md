
```
0040B2F9  50       push eax  

0040B2FA  E8 D1440200   call StickyNo.0042F7D0  

0040B2FF  50       push eax  

0040B300  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B304  C64424 20 03  mov byte ptr ss:\[esp+20\],3  

0040B309  E8 66430200   call StickyNo.0042F674  

0040B30E  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B312  C64424 1C 02  mov byte ptr ss:\[esp+1C\],2  

0040B317  E8 1F420200   call StickyNo.0042F53B  

0040B31C  885C24 1C    mov byte ptr ss:\[esp+1C\],bl  

0040B320  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

0040B324  EB 58      jmp short StickyNo.0040B37E  

0040B326  68 341B4500   push StickyNo.00451B34             ; ASCII "AA"  

0040B32B  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B32F  E8 E3450200   call StickyNo.0042F917  

0040B334  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B338  6A 02      push 2  

0040B33A  51       push ecx  

0040B33B  8D4C24 30    lea ecx,dword ptr ss:\[esp+30\]  

0040B33F  E8 B7DD0100   call StickyNo.004290FB  

0040B344  68 381B4500   push StickyNo.00451B38             ; ASCII "wefwfrw3rf32wasfaf"  

0040B349  8D5424 10    lea edx,dword ptr ss:\[esp+10\]  

0040B34D  50       push eax  

0040B34E  52       push edx  

0040B34F  C64424 28 04  mov byte ptr ss:\[esp+28\],4  

0040B354  E8 77440200   call StickyNo.0042F7D0  

0040B359  50       push eax  

0040B35A  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B35E  C64424 20 05  mov byte ptr ss:\[esp+20\],5  

0040B363  E8 0C430200   call StickyNo.0042F674  

0040B368  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

0040B36C  C64424 1C 04  mov byte ptr ss:\[esp+1C\],4  

0040B371  E8 C5410200   call StickyNo.0042F53B  

0040B376  885C24 1C    mov byte ptr ss:\[esp+1C\],bl  

0040B37A  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B37E  E8 B8410200   call StickyNo.0042F53B  

0040B383  8B7424 24    mov esi,dword ptr ss:\[esp+24\]  

0040B387  8D4424 28    lea eax,dword ptr ss:\[esp+28\]  

0040B38B  50       push eax  

0040B38C  8BCE      mov ecx,esi  

0040B38E  E8 1D3F0200   call StickyNo.0042F2B0  

0040B393  895C24 10    mov dword ptr ss:\[esp+10\],ebx  

0040B397  8D4C24 28    lea ecx,dword ptr ss:\[esp+28\]  

0040B39B  C64424 1C 00  mov byte ptr ss:\[esp+1C\],0  

0040B3A0  E8 96410200   call StickyNo.0042F53B  

0040B3A5  8B4C24 14    mov ecx,dword ptr ss:\[esp+14\]  

0040B3A9  8BC6      mov eax,esi  

0040B3AB  5E       pop esi  

0040B3AC  5B       pop ebx  

0040B3AD  64:890D 0000000>mov dword ptr fs:\[0\],ecx  

0040B3B4  83C4 18     add esp,18  

0040B3B7  C2 0800     retn 8  
```

### 跟进 2

```
00406080  6A FF      push -1                 ; 跟进来到这里  

00406082  68 98E54300   push StickyNo.0043E598  

00406087  64:A1 00000000 mov eax,dword ptr fs:\[0\]  

0040608D  50       push eax  

0040608E  64:8925 0000000>mov dword ptr fs:\[0\],esp  

00406095  83EC 60     sub esp,60  

00406098  56       push esi  

00406099  8B7424 7C    mov esi,dword ptr ss:\[esp+7C\]  

0040609D  57       push edi  

0040609E  8B7C24 7C    mov edi,dword ptr ss:\[esp+7C\]  

004060A2  6A 00      push 0  

004060A4  56       push esi  

004060A5  57       push edi  

004060A6  C74424 14 00000>mov dword ptr ss:\[esp+14\],0  

004060AE  E8 B6360200   call StickyNo.00429769  

004060B3  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

004060B7  E8 A40A0000   call StickyNo.00406B60             ; 跟进 3  

004060BC  56       push esi  

004060BD  57       push edi  

004060BE  8D4C24 14    lea ecx,dword ptr ss:\[esp+14\]  

004060C2  C74424 78 00000>mov dword ptr ss:\[esp+78\],0  

004060CA  E8 810C0000   call StickyNo.00406D50  

004060CF  8B7424 78    mov esi,dword ptr ss:\[esp+78\]  

004060D3  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

004060D7  56       push esi  

004060D8  E8 130B0000   call StickyNo.00406BF0             ; 跟进可以看到 MD5 计算,有兴趣的不妨进去看看!!  

004060DD  8B4C24 68    mov ecx,dword ptr ss:\[esp+68\]  

004060E1  8BC6      mov eax,esi  

004060E3  5F       pop edi  

004060E4  5E       pop esi  

004060E5  64:890D 0000000>mov dword ptr fs:\[0\],ecx  

004060EC  83C4 6C     add esp,6C  

004060EF  C3       retn  
```

### 跟进 3

```
00406B60  8BD1      mov edx,ecx              ; 跟进来到这里!看到下面的常数,我就不用多说了!MD5 算法  

00406B62  57       push edi  

00406B63  B9 10000000   mov ecx,10  

00406B68  33C0      xor eax,eax  

00406B6A  8D7A 04     lea edi,dword ptr ds:\[edx+4\]  

00406B6D  C702 FC1D4400  mov dword ptr ds:\[edx\],StickyNo.00441DFC  

00406B73  F3:AB     rep stos dword ptr es:\[edi\]  

00406B75  8942 48     mov dword ptr ds:\[edx+48\],eax  

00406B78  8942 44     mov dword ptr ds:\[edx+44\],eax  

00406B7B  C742 4C 0123456>mov dword ptr ds:\[edx+4C\],67452301  

00406B82  C742 50 89ABCDE>mov dword ptr ds:\[edx+50\],EFCDAB89  

00406B89  C742 54 FEDCBA9>mov dword ptr ds:\[edx+54\],98BADCFE  

00406B90  C742 58 7654321>mov dword ptr ds:\[edx+58\],10325476  

00406B97  8BC2      mov eax,edx  

00406B99  5F       pop edi  

00406B9A  C3       retn  
```

### 跟进 4

```
0041B528  55       push ebp                 ; 跟进来到这里  

0041B529  8BEC      mov ebp,esp  

0041B52B  833D 8CA04500 0>cmp dword ptr ds:\[45A08C\],0  

0041B532  53       push ebx  

0041B533  56       push esi  

0041B534  57       push edi  

0041B535  75 12     jnz short StickyNo.0041B549  

0041B537  FF75 0C     push dword ptr ss:\[ebp+C\]  

0041B53A  FF75 08     push dword ptr ss:\[ebp+8\]  

0041B53D  E8 CE350000   call StickyNo.0041EB10  

0041B542  59       pop ecx  

0041B543  59       pop ecx  

0041B544  E9 89000000   jmp StickyNo.0041B5D2  

0041B549  6A 19      push 19  

0041B54B  E8 AA320000   call StickyNo.0041E7FA  

0041B550  8B75 0C     mov esi,dword ptr ss:\[ebp+C\]         ; 假码送 ESI  

0041B553  8B7D 08     mov edi,dword ptr ss:\[ebp+8\]  

0041B556  59       pop ecx  

0041B557  66:0FB60F    movzx cx,byte ptr ds:\[edi\]          ; 依次取"4fee7c9b10388c33"ACII 值并送 CX  

0041B55B  0FB6C1     movzx eax,cl              ; CL 送 EAX,EAX 初始为 0  

0041B55E  47       inc edi               ; EDI+1  

0041B55F  894D 0C     mov dword ptr ss:\[ebp+C\],ecx  

0041B562  F680 A1A14500 0>test byte ptr ds:\[eax+45A1A1\],4  

0041B569  74 16     je short StickyNo.0041B581  

0041B56B  8A07      mov al,byte ptr ds:\[edi\]  

0041B56D  84C0      test al,al  

0041B56F  75 06     jnz short StickyNo.0041B577  

0041B571  8365 0C 00   and dword ptr ss:\[ebp+C\],0  

0041B575  EB 0A     jmp short StickyNo.0041B581  

0041B577  33D2      xor edx,edx  

0041B579  47       inc edi  

0041B57A  8AF1      mov dh,cl  

0041B57C  8AD0      mov dl,al  

0041B57E  8955 0C     mov dword ptr ss:\[ebp+C\],edx  

0041B581  66:0FB61E    movzx bx,byte ptr ds:\[esi\]          ; 取假码第一位 ASCII 值并送 BX  

0041B585  0FB6C3     movzx eax,bl              ; BL 送 EAX  

0041B588  46       inc esi               ; ESI+1  

0041B589  F680 A1A14500 0>test byte ptr ds:\[eax+45A1A1\],4  

0041B590  74 13     je short StickyNo.0041B5A5  

0041B592  8A06      mov al,byte ptr ds:\[esi\]  

0041B594  84C0      test al,al  

0041B596  75 04     jnz short StickyNo.0041B59C  

0041B598  33DB      xor ebx,ebx  

0041B59A  EB 09     jmp short StickyNo.0041B5A5  

0041B59C  33C9      xor ecx,ecx  

0041B59E  46       inc esi  

0041B59F  8AEB      mov ch,bl  

0041B5A1  8AC8      mov cl,al  

0041B5A3  8BD9      mov ebx,ecx  

0041B5A5  66:395D 0C   cmp word ptr ss:\[ebp+C\],bx          ; 假码第一位与 MD5 后结果第一位比较  

0041B5A9  75 09     jnz short StickyNo.0041B5B4         ; 不相等就 OVER  

0041B5AB  66:837D 0C 00  cmp word ptr ss:\[ebp+C\],0          ; 再与 0 比较  
```
