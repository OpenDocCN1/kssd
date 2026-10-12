
```
00425F7D    not edx
00425F7F    or edx,edi
00425F81    xor edx,ebx
00425F83    add edx,dword ptr ss:[ebp-8]
00425F86    lea eax,dword ptr ds:[eax+edx+F4292244]
00425F8D    mov edx,eax
00425F8F    shr edx,1A
00425F92    shl eax,6
00425F95    or edx,eax
00425F97    mov eax,ebx
00425F99    add edx,edi
00425F9B    not eax
00425F9D    or eax,edx
00425F9F    xor eax,edi
00425FA1    add eax,dword ptr ss:[ebp-C]
00425FA4    lea esi,dword ptr ds:[esi+eax+432AFF97]
00425FAB    mov eax,esi
00425FAD    shr eax,16
00425FB0    shl esi,0A
00425FB3    or eax,esi
00425FB5    mov esi,edi
00425FB7    add eax,edx
00425FB9    not esi
00425FBB    or esi,eax
00425FBD    xor esi,edx
00425FBF    add esi,dword ptr ss:[ebp-10]
00425FC2    lea ebx,dword ptr ds:[ebx+esi+AB9423A7]
00425FC9    mov esi,ebx
00425FCB    shr esi,11
00425FCE    shl ebx,0F
00425FD1    or esi,ebx
00425FD3    mov ebx,edx
00425FD5    add esi,eax
00425FD7    not ebx
00425FD9    or ebx,esi
00425FDB    xor ebx,eax
00425FDD    add ebx,dword ptr ss:[ebp-2C]
00425FE0    lea edi,dword ptr ds:[edi+ebx+FC93A039]
00425FE7    mov ebx,edi
00425FE9    shl ebx,15
00425FEC    shr edi,0B
00425FEF    or ebx,edi
00425FF1    mov edi,eax
00425FF3    add ebx,esi
00425FF5    not edi
00425FF7    or edi,ebx
00425FF9    xor edi,esi
00425FFB    add edi,dword ptr ss:[ebp-30]
00425FFE    lea edi,dword ptr ds:[edx+edi+655B59C3]
00426005    mov edx,edi
00426007    shr edx,1A
0042600A    shl edi,6
0042600D    or edx,edi
0042600F    mov edi,esi
00426011    add edx,ebx
00426013    not edi
00426015    or edi,edx
00426017    xor edi,ebx
00426019    add edi,dword ptr ss:[ebp-14]
0042601C    lea edi,dword ptr ds:[eax+edi+8F0CCC92]
00426023    mov eax,edi
00426025    shr eax,16
00426028    shl edi,0A
0042602B    or eax,edi
0042602D    mov edi,ebx
0042602F    add eax,edx
00426031    not edi
00426033    or edi,eax
00426035    xor edi,edx
00426037    add edi,dword ptr ss:[ebp-18]
0042603A    lea edi,dword ptr ds:[esi+edi+FFEFF47D]
00426041    mov esi,edi
00426043    shr esi,11
00426046    shl edi,0F
00426049    or esi,edi
0042604B    mov edi,edx
0042604D    add esi,eax
0042604F    not edi
00426051    or edi,esi
00426053    xor edi,eax
00426055    add edi,dword ptr ss:[ebp-1C]
00426058    lea ebx,dword ptr ds:[ebx+edi+85845DD1]
0042605F    mov edi,ebx
00426061    shr ebx,0B
00426064    shl edi,15
00426067    or edi,ebx
00426069    mov ebx,eax
0042606B    not ebx
0042606D    add edi,esi
0042606F    or ebx,edi
00426071    xor ebx,esi
00426073    add ebx,dword ptr ss:[ebp-34]
00426076    lea ebx,dword ptr ds:[edx+ebx+6FA87E4F]
0042607D    mov edx,ebx
0042607F    shl ebx,6
00426082    shr edx,1A
00426085    or edx,ebx
00426087    mov ebx,esi
00426089    add edx,edi
0042608B    not ebx
0042608D    or ebx,edx
0042608F    xor ebx,edi
00426091    add ebx,dword ptr ss:[ebp-38]
00426094    lea ebx,dword ptr ds:[eax+ebx+FE2CE6E0]
0042609B    mov eax,ebx
0042609D    shr eax,16
004260A0    shl ebx,0A
004260A3    or eax,ebx
004260A5    mov ebx,edi
004260A7    add eax,edx
004260A9    not ebx
004260AB    or ebx,eax
004260AD    xor ebx,edx
004260AF    add ebx,dword ptr ss:[ebp-20]
004260B2    lea ebx,dword ptr ds:[esi+ebx+A3014314]
004260B9    mov esi,ebx
004260BB    shr esi,11
004260BE    shl ebx,0F
004260C1    or esi,ebx
004260C3    mov ebx,edx
004260C5    add esi,eax
004260C7    not ebx
004260C9    or ebx,esi
004260CB    xor ebx,eax
004260CD    add ebx,dword ptr ss:[ebp-3C]
004260D0    lea ebx,dword ptr ds:[edi+ebx+4E0811A1]
004260D7    mov edi,ebx
004260D9    shl edi,15
004260DC    shr ebx,0B
004260DF    or edi,ebx
004260E1    mov ebx,eax
004260E3    add edi,esi
004260E5    not ebx
004260E7    or ebx,edi
004260E9    xor ebx,esi
004260EB    add ebx,dword ptr ss:[ebp-40]
004260EE    lea ebx,dword ptr ds:[edx+ebx+F7537E82]
004260F5    mov edx,ebx
004260F7    shr edx,1A
004260FA    shl ebx,6
004260FD    or edx,ebx
004260FF    mov ebx,esi
00426101    add edx,edi
00426103    not ebx
00426105    or ebx,edx
00426107    xor ebx,edi
00426109    add ebx,dword ptr ss:[ebp-24]
0042610C    lea eax,dword ptr ds:[eax+ebx+BD3AF235]
00426113    mov ebx,eax
00426115    shr ebx,16
00426118    shl eax,0A
0042611B    or ebx,eax
0042611D    mov eax,edi
0042611F    add ebx,edx
00426121    not eax
00426123    or eax,ebx
00426125    xor eax,edx
00426127    add eax,dword ptr ss:[ebp-28]
0042612A    lea esi,dword ptr ds:[esi+eax+2AD7D2BB]
00426131    mov eax,esi
00426133    shr eax,11
00426136    shl esi,0F
00426139    or eax,esi
0042613B    mov esi,edx
0042613D    add eax,ebx
0042613F    not esi
00426141    or esi,eax
00426143    xor esi,ebx
00426145    add esi,dword ptr ss:[ebp-44]
00426148    lea edi,dword ptr ds:[edi+esi+EB86D391]
0042614F    mov esi,dword ptr ds:[ecx]
00426151    add esi,edx
00426153    mov edx,edi
00426155    shl edx,15
00426158    shr edi,0B
; 上面的一大串代码所作的处理,可以当作黑箱操作看待
0042615B    or edx,edi
; 下面是保存结果，写注册机时设个变量替换那些地址就可以了
0042615D    mov dword ptr ds:[ecx],esi
; 保存结果
0042615F    add edx,dword ptr ds:[ecx+4]
00426162    pop edi
00426163    pop esi
00426164    add edx,eax
00426166    mov dword ptr ds:[ecx+4],edx
; 保存结果
00426169    mov edx,dword ptr ds:[ecx+8]
0042616C    add edx,eax
0042616E    mov eax,dword ptr ds:[ecx+C]
00426171    add eax,ebx
00426173    mov dword ptr ds:[ecx+8],edx
; 保存结果
00426176    mov dword ptr ds:[ecx+C],eax
; 保存结果
00426179    pop ebx
0042617A    leave
0042617B    retn
-------------------处理用户名的关键代码--------end
}
```

```
0042B6AC    add esp,24
0042B6AF    xor edi,edi
0042B6B1    lea esi,dword ptr ss:[ebp-24]
; 使 esi 指向前面处理的结果
0042B6B4    /xor ecx,ecx
0042B6B6    |xor eax,eax
0042B6B8    |/movzx edx,byte ptr ds:[esi+eax]
; 下面这个循环处理前面保存的结果,
0042B6BC    ||shl ecx,8
0042B6BF    ||add ecx,edx
; 其实整个过程就把数据的高低位换了一下
0042B6C1    ||inc eax
0042B6C2    ||cmp eax,4
0042B6C5    |\jl short dialogbl.0042B6B8
0042B6C7    |mov eax,dword ptr ds:[908840]
0042B6CC    |cmp edi,2
0042B6CF    |jge short dialogbl.0042B704
0042B6D1    |mov dword ptr ss:[ebp+10],eax
0042B6D4    |push ecx
0042B6D5    |lea eax,dword ptr ss:[ebp+10]
0042B6D8    |push dialogbl.008D87E4
; ASCII "%08lX-"
0042B6DD    |push eax
0042B6DE    |mov byte ptr ss:[ebp-4],3
0042B6E2    |call dialogbl.005871A0
; 这个 call 设置输出格式，把结果保存到一个地方
0042B6E7    |mov ecx,dword ptr ss:[ebp+10]
; 加了"-"的结果，作为注册码第一段
0042B6EA    |add esp,0C
0042B6ED    |mov eax,dword ptr ds:[ecx-8]
; 结果的长度，包括"-"
0042B6F0    |push eax
0042B6F1    |push ecx
0042B6F2    |push eax
0042B6F3    |lea ecx,dword ptr ss:[ebp+C]
0042B6F6    |call dialogbl.00585A10
0042B6FB    |mov byte ptr ss:[ebp-4],2
```
