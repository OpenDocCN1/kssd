
#### 机器码分析

`:0040D764 8B55A0 mov edx, dword ptr [ebp-60]`

续

我们假设运算函数为`f1(x)`，机器码为`N0`,`N1`,`N2`,`N3`,`N4`,`N5`,`N6`,`N7`，相应的机器码位置为：`LNx`（`x=0 to 7`），实际用到运算的机器码只有`N0`,`N1`,`N3`,`N4`,`N5`：

\-----------------------------------------------

```
:0040B785 8BF1                mov esi, ecx
:0040B787 881424              mov byte ptr [esp], dl
:0040B78A 33DB                xor ebx, ebx
:0040B78C 56                  push esi
:0040B78D E8A6720800          call 00492A38  <=====取机器码长度 L(8)
:0040B792 59                  pop ecx
:0040B793 8BF8                mov edi, eax
:0040B795 D1EF                shr edi, 1    <=====右移一位
:0040B797 56                  push esi
:0040B7A3 8A141E              mov dl, byte ptr [esi+ebx]  <====取机器码 N0
:0040B7A6 59                  pop ecx
:0040B7A7 02C2                add al, dl
:0040B7A9 8BD7                mov edx, edi
:0040B7AB 0FAFD7              imul edx, edi <====(L shr 1)*(L shr 1)
:0040B7AE 8A0C24              mov cl, byte ptr [esp]  <====取函数参数 x
:0040B7B1 43                  inc ebx       <====EBX=EBX=1
:0040B7B2 32C1                xor al, cl    <====N0 xor X
:0040B7B4 8A0C3E              mov cl, byte ptr [esi+edi]  <=====取 N4
:0040B7B7 02D1                add dl, cl    <=====N4+(L shr 1)*(L shr 1)
:0040B7B9 8A0C2E              mov cl, byte ptr [esi+ebp]  <=====取 N1
:0040B7BC 32C2                xor al, dl    <=====(N0 xor X) xor (N4+(L shr 1)*(L shr 1))
:0040B7BE 8BD5                mov edx, ebp
:0040B7C0 0FAFD5              imul edx, ebp <====L*L
:0040B7C3 02D1                add dl, cl
:0040B7C5 4D                  dec ebp
:0040B7C6 4F                  dec edi
:0040B7C7 32C2                xor al, dl    <====从这里整理出公式：R1=(((N0 xor X)
                                xor ((L shr 1)*(L shr 1)+N4)) xor L*L
:0040B7C9 4F                  dec edi
:0040B7CA 4D                  dec ebp
:0040B7CB 4D                  dec ebp
:0040B7CC 4F                  dec edi
:0040B7CD 880424              mov byte ptr [esp], al  <===保存 R1，作为下一个数的计
                                算的参数
```

\-------------------------------------------------

```
:0040B7FB 8BC3                mov eax, ebx    <====EAX=LN3
:0040B7FD 8A0C24              mov cl, byte ptr [esp]  <====取 R1
:0040B800 F7EB                imul ebx        <=====LN3*LN3
:0040B802 8A141E              mov dl, byte ptr [esi+ebx]  <=====取 N3
:0040B805 43                  inc ebx
:0040B806 02C2                add al, dl      <=====N3+LN3*LN3
:0040B808 8BD7                mov edx, edi    <=====LN1
:0040B80A 0FAFD7              imul edx, edi   <=====LN1*LN1
:0040B80D 32C1                xor al, cl      <=====R1 xor (N3+LN3*LN3)
:0040B80F 8A0C3E              mov cl, byte ptr [esi+edi]  <=====取 N1
:0040B812 02D1                add dl, cl      <=====N1+LN1*LN1
:0040B814 8A0C2E              mov cl, byte ptr [esi+ebp]  <=====取 N5
:0040B817 32C2                xor al, dl      <====(R1 xor (N3+LN3*LN3)) xor
                                (N1+LN1*LN1)
:0040B819 8BD5                mov edx, ebp    <====LN5
:0040B81B 0FAFD5              imul edx, ebp   <====LN5*LN5
:0040B81E 02D1                add dl, cl      <====LN5*LN5+N5
:0040B820 4D                  dec ebp
:0040B821 4F                  dec edi
:0040B822 32C2                xor al, dl      <====整理出公式:R2=((R1 xor
                                (N3+LN3*LN3)) xor (N1+LN1*LN1)) xor (LN5*LN5+N5)
:0040B824 4F                  dec edi
:0040B825 4D                  dec ebp
:0040B826 4D                  dec ebp
:0040B827 4F                  dec edi
:0040B828 880424              mov byte ptr [esp], al  <====保存 R2
```

\----------------------------------------------

```
:0040B84C 33C0                xor eax, eax
:0040B84E 8A1424              mov dl, byte ptr [esp] <====取 R2
:0040B851 8A0424              mov al, byte ptr [esp] <====取 R2
:0040B854 8BFE                mov edi, esi
:0040B856 C1E202              shl edx, 02     <===
:0040B859 C1F806              sar eax, 06     <===
:0040B85C 0AC2                or al, dl       <===整理出公式:R3=(R2 shl 2) or (R2 sar 6)
:0040B85E 33DB                xor ebx, ebx
:0040B860 880424              mov byte ptr [esp], al  <===保存 R3
:0040B863 EB0B                jmp 0040B870
```

\----------------------------------------------

下面是前四位机器码运算：

```
:0040B865 8A17                mov dl, byte ptr [edi]  <====取 N0
:0040B867 8BC3                mov eax, ebx    <====LN0
:0040B869 02C2                add al, dl      <====N0+LN0
:0040B86B 43                  inc ebx
:0040B86C 300424              xor byte ptr [esp], al  <====循环 xor,结果为 R4
:0040B86F 47                  inc edi
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:0040B863(U)`

```
:0040B870 56                  push esi
:0040B871 E8C2710800          call 00492A38
:0040B876 D1E8                shr eax, 1
:0040B878 59                  pop ecx
:0040B879 3BD8                cmp ebx, eax
:0040B87B 72E8                jb 0040B865
:0040B87D 8A0424              mov al, byte ptr [esp]  <====取 R4
:0040B880 C1E006              shl eax, 06     <====
:0040B883 33D2                xor edx, edx
:0040B885 8A1424              mov dl, byte ptr [esp]
:0040B888 C1FA02              sar edx, 02
:0040B88B 0AC2                or al, dl       <==== 整理出公式:R5=(R4 shl 6) or (R4 sar 2)
:0040B88D 880424              mov byte ptr [esp], al  <====保存 R5
:0040B890 56                  push esi
:0040B891 E8A2710800          call 00492A38
:0040B896 59                  pop ecx
:0040B897 8BD8                mov ebx, eax
:0040B899 D1EB                shr ebx, 1
:0040B89B 8D3C1E              lea edi, dword ptr [esi+ebx]
:0040B89E EB0D                jmp 0040B8AD
```

\-------------------------------------------------

下面是后四位运算：

```
:0040B8A0 8BC3                mov eax, ebx    <====LN5
:0040B8A2 F7EB                imul ebx        <====LN5*LN5
:0040B8A4 8A17                mov dl, byte ptr [edi]  <====取 R5
:0040B8A6 43                  inc ebx
:0040B8A7 02C2                add al, dl
:0040B8A9 300424              xor byte ptr [esp], al  <===循环 xor,结果为 R6
:0040B8AC 47                  inc edi
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:0040B89E(U)`

```
:0040B8AD 56                  push esi
:0040B8AE E885710800          call 00492A38
:0040B8B3 59                  pop ecx
:0040B8B4 3BD8                cmp ebx, eax
:0040B8B6 72E8                jb 0040B8A0
:0040B8B8 33C0                xor eax, eax
:0040B8BA 8A0424              mov al, byte ptr [esp]  <====取 R6
:0040B8BD C1F806              sar eax, 06     <=====
:0040B8C0 8A1424              mov dl, byte ptr [esp]
:0040B8C3 C1E202              shl edx, 02     <=====
:0040B8C6 0AC2                or al, dl       <=====整理出公式:R7=(R6 sar 6) or (R6 shl 2)
:0040B8C8 880424              mov byte ptr [esp], al  <====保存 R7,同时作为函数的返
                                回值
:0040B8CB 8A0424              mov al, byte ptr [esp]  <====取结果 R7
:0040B8CE 5A                  pop edx
:0040B8CF 5D                  pop ebp
:0040B8D0 5F                  pop edi
:0040B8D1 5E                  pop esi
:0040B8D2 5B                  pop ebx
:0040B8D3 C3                  ret
```

\------------------------------------------

下面是函数返回后的代码：
