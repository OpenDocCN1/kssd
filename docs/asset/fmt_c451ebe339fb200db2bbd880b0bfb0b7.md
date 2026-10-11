
## 汇编代码分析

00489FCE |. 896C24 14 mov dword ptr ss:[esp+14], ebp ; 第二段计算结果放入 [esp+14] 指向地址内
00489FD2 |. 897C24 18 mov dword ptr ss:[esp+18], edi
00489FD6 |. DF6C24 14 fild qword ptr ss:[esp+14]
00489FDA |. 50 push eax
00489FDB |. DEC9 fmulp st(1),st ; st(0)=假码第一段后五位字符计算结果 16 进制转换成 1 进制\*st(1)假码第二段计算结果 16 进制转换成 10 进制，结果放入 st(1)
00489FDD |. DD05 50894B00 fld qword ptr ds:[4B8950] ; 将地址[4B8950]内的固定值：44099087 装入 st(1)
00489FE3 |. E8 62B50100 call MathType.004A554A ; 计算，跟进⑥
00489FE8 |. DC05 48894B00 fadd qword ptr ds:[4B8948] ; st(0)+[004B848]地址内固定的值：1336337
00489FEE |. E8 799A0100 call MathType.004A3A6C
00489FF3 |. 50 push eax
00489FF4 |. E8 089A0100 call MathType.004A3A01 ; 这里计算出序列号最后一段，跟进⑦
00489FF9 |. 83C4 0C add esp,0C
00489FFC |. 8BCE mov ecx,esi
00489FFE |. E8 1D7D0000 call MathType.00491D20
0048A003 |. 397E 04 cmp dword ptr ds:[esi+4],edi
0048A006 |. 7E 3D jle short MathType.0048A045
0048A008 |> 8B4E 08 /mov ecx,dword ptr ds:[esi+8] ; 这里的循环将计算出的序列号最后一段小写字符转换成大写
0048A00B |. 8A0439 |mov al,byte ptr ds:[ecx+edi]
0048A00E |. 3C 49 |cmp al,49
0048A010 |. 74 21 |je short MathType.0048A033
0048A012 |. 3C 69 |cmp al,69
0048A014 |. 74 1D |je short MathType.0048A033
0048A016 |. 3C 4F |cmp al,4F
0048A018 |. 74 14 |je short MathType.0048A02E
0048A01A |. 3C 6F |cmp al,6F
0048A01C |. 74 10 |je short MathType.0048A02E
0048A01E |. 0FBED0 |movsx edx,al
0048A021 |. 57 |push edi
0048A022 |. 52 |push edx
0048A023 |. E8 46B00100 |call MathType.004A506E
0048A028 |. 83C4 04 |add esp,4
0048A02B |. 50 |push eax
0048A02C |. EB 08 |jmp short MathType.0048A036
0048A02E |> 57 |push edi
0048A02F |. 6A 5A |push 5A
0048A031 |. EB 03 |jmp short MathType.0048A036
0048A033 |> 57 |push edi
0048A034 |. 6A 59 |push 59
0048A036 |> 8BCE |mov ecx,esi
0048A038 |. E8 C3800000 |call MathType.00492100
0048A03D |. 8B46 04 |mov eax,dword ptr ds:[esi+4]
0048A040 |. 47 |inc edi
0048A041 |. 3BF8 |cmp edi,eax
0048A043 |.^ 7C C3 \jl short MathType.0048A008
0048A045 |> 5F pop edi
0048A046 |. 5E pop esi
0048A047 |. 5D pop ebp
0048A048 |. 5B pop ebx
0048A049 |. 83C4 08 add esp,8
0048A04C \. C3 retn

```
\===========================================================================================================
```

### ④call 004A3664

004A3664 /$ 6A 01 push 1
004A3666 |. FF7424 10 push dword ptr ss:[esp+10]
004A366A |. FF7424 10 push dword ptr ss:[esp+10]
004A366E |. FF7424 10 push dword ptr ss:[esp+10]
004A3672 |. E8 E5FDFFFF call MathType.004A345C ; 跟！！！⑤
004A3677 |. 83C4 10 add esp,10
004A367A \. C3 retn

```
\============================================================================================================================
```

### ⑤call 004A345C：

004A345C /$ 55 push ebp ; ebp=00000014 入栈

............代码略...........

004A3521 |> 83C8 FF or eax,FFFFFFFF
004A3524 |. 33D2 xor edx,edx
004A3526 |. F775 10 div dword ptr ss:[ebp+10]
004A3529 |. BF 03010000 mov edi,103
004A352E |. 8945 F4 mov dword ptr ss:[ebp-C],eax
004A3531 |> 833D 5C294E00 01 /cmp dword ptr ds:[4E295C],1
004A3538 |. 0FB6F3 |movzx esi,bl ; 依次取假码第一段后五位放入 esi
004A353B |. 7E 0C |jle short MathType.004A3549 ; 跳
004A353D |. 6A 04 |push 4
004A353F |. 56 |push esi
004A3540 |. E8 22420000 |call MathType.004A7767
004A3545 |. 59 |pop ecx
004A3546 |. 59 |pop ecx
004A3547 |. EB 0B |jmp short MathType.004A3554
004A3549 |> A1 50274E00 |mov eax,dword ptr ds:[4E2750]
004A354E |. 8A0470 |mov al,byte ptr ds:[eax+esi\*2]
004A3551 |. 83E0 04 |and eax,4 ; 与运算
004A3554 |> 85C0 |test eax,eax ; 与运算结果为 0 就跳，不等于 0 不跳，如果假码是字符就跳去和 37 进行减法运算，如果是数字不跳，就与 30 进行减法运算
004A3556 |. 74 08 |je short MathType.004A3560
004A3558 |. 0FBECB |movsx ecx,bl
004A355B |. 83E9 30 |sub ecx,30
004A355E |. EB 32 |jmp short MathType.004A3592
004A3560 |> 833D 5C294E00 01 |cmp dword ptr ds:[4E295C],1
004A3567 |. 7E 0B |jle short MathType.004A3574
004A3569 |. 57 |push edi
004A356A |. 56 |push esi
004A356B |. E8 F7410000 |call MathType.004A7767
004A3570 |. 59 |pop ecx
004A3571 |. 59 |pop ecx
004A3572 |. EB 0B |jmp short MathType.004A357F
004A3574 |> A1 50274E00 |mov eax,dword ptr ds:[4E2750]
004A3579 |. 66:8B0470 |mov ax,word ptr ds:[eax+esi\*2]
004A357D |. 23C7 |and eax,edi
004A357F |> 85C0 |test eax,eax
004A3581 |. 74 4A |je short MathType.004A35CD
004A3583 |. 0FBEC3 |movsx eax,bl ; 依次取假码第一段后五位字符的 Hex 值放入 eax
004A3586 |. 50 |push eax
004A3587 |. E8 E21A0000 |call MathType.004A506E
004A358C |. 59 |pop ecx
004A358D |. 8BC8 |mov ecx,eax ; eax 放入 ecx
004A358F |. 83E9 37 |sub ecx,37 ; ecx-37，减法运算，依次取假码第一段后五位字符的 Hex 值减 37，结果放入 ecx 参与下面的运算
004A3592 |> 3B4D 10 |cmp ecx,dword ptr ss:[ebp+10]
004A3595 |. 73 36 |jnb short MathType.004A35CD
004A3597 |. 8B75 F8 |mov esi,dword ptr ss:[ebp-8] ; [ebp-8]指向地址[0012EEA4]的值放入 esi
004A359A |. 834D 14 08 |or dword ptr ss:[ebp+14],8
004A359E |. 3B75 F4 |cmp esi,dword ptr ss:[ebp-C]
004A35A1 |. 72 14 |jb short MathType.004A35B7
004A35A3 |. 75 0C |jnz short MathType.004A35B1
004A35A5 |. 83C8 FF |or eax,FFFFFFFF
004A35A8 |. 33D2 |xor edx,edx
004A35AA |. F775 10 |div dword ptr ss:[ebp+10]
004A35AD |. 3BCA |cmp ecx,edx
004A35AF |. 76 06 |jbe short MathType.004A35B7
004A35B1 |> 834D 14 04 |or dword ptr ss:[ebp+14],4
004A35B5 |. EB 09 |jmp short MathType.004A35C0
004A35B7 |> 0FAF75 10 |imul esi,dword ptr ss:[ebp+10] ; esi 乘以固定值 24，结果放入 esi
