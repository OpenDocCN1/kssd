
# 汇编代码分析

## 代码段 1
```
:10001558 894C2420             mov dword ptr [esp+20], ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:100014FE(U), :10001515(U)

:1000155C 43                 inc ebx
:1000155D 3BDF               cmp ebx, edi
:1000155F 0F8271FFFFFF       jb 100014D6
```

用户名会去掉前后空格

注册码`+'SP6'`分割成前后两部分:

`NameXor`的串就是奇数串

`Name`的累加和

`NameXOR`的异或

`NameXOR mod $7FFFFFFFF===>`转为字符串

`xor $67C2D76C=`

`Code2==>`数字，去只取一定范围的

`Code1:=Code1+Name2`

## 代码段 2
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100014D0(C)

:10001565 8D7C2448           lea edi, dword ptr [esp+48]
:10001569 83C9FF             or ecx, FFFFFFFF
:1000156C 33C0               xor eax, eax
:1000156E C6442C4800         mov [esp+ebp+48], 00
:10001573 F2                 repnz
:10001574 AE                 scasb
:10001575 8B542424           mov edx, dword ptr [esp+24]
:10001579 F7D1               not ecx
:1000157B 49                 dec ecx
:1000157C C644142800         mov [esp+edx+28], 00
:10001581 83E1FE             and ecx, FFFFFFFE
:10001584 83F90A             cmp ecx, 0000000A
:10001587 0F828C020000       jb 10001819============>错误 6
//====================>转换后的串 2 长度
:1000158D 8D442448           lea eax, dword ptr [esp+48]
:10001591 50                 push eax
:10001592 E8A49A0000         call 1000B03B==>串 2 转为数字 32bits 有效
:10001597 8B7C2418           mov edi, dword ptr [esp+18]
:1000159B 8BD8               mov ebx, eax
:1000159D A1388F0110         mov eax, dword ptr [10018F38]
:100015A2 83C9FF             or ecx, FFFFFFFF
:100015A5 33D8               xor ebx, eax
:100015A7 33C0               xor eax, eax
:100015A9 83C404             add esp, 00000004
:100015AC 8D542428           lea edx, dword ptr [esp+28]
:100015B0 F2                 repnz
:100015B1 AE                 scasb
:100015B2 F7D1               not ecx
:100015B4 2BF9               sub edi, ecx
:100015B6 8BF7               mov esi, edi
:100015B8 8BE9               mov ebp, ecx
:100015BA 8BFA               mov edi, edx
:100015BC 83C9FF             or ecx, FFFFFFFF
:100015BF F2                 repnz
:100015C0 AE                 scasb
:100015C1 8BCD               mov ecx, ebp
:100015C3 4F                 dec edi
:100015C4 C1E902              shr ecx, 02
:100015C7 F3                 repz
:100015C8 A5                 movsd
:100015C9 8BCD               mov ecx, ebp
:100015CB 83E103             and ecx, 00000003
:100015CE F3                 repz
:100015CF A4                 movsb
:100015D0 8DBC24A8000000     lea edi, dword ptr [esp+000000A8]
\========================>串 1+DLL 名
:100015D7 83C9FF             or ecx, FFFFFFFF
:100015DA F2                 repnz
:100015DB AE                 scasb
:100015DC F7D1               not ecx
:100015DE 49                 dec ecx
:100015DF 8D7C2428           lea edi, dword ptr [esp+28]
:100015E3 8BD1               mov edx, ecx
:100015E5 83C9FF             or ecx, FFFFFFFF
:100015E8 F2                 repnz
:100015E9 AE                 scasb
:100015EA F7D1               not ecx
:100015EC 49                 dec ecx
:100015ED 83FA20             cmp edx, 00000020
:100015F0 894C2418           mov dword ptr [esp+18], ecx
:100015F4 0F821F020000       jb 10001819============>错误 7
\========================>用户名长度>=$20 的时候,Name+余串
\========================>否则 Name+余串+''
\========================>DLL 为 12
\========================>串 1 的长必须>=20
\========================>重新输入假注册码 40 字节长
:100015FA 85D2               test edx, edx
:100015FC 7617               jbe 10001615
```

## 代码段 3
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001613(C)

:100015FE 0FBE8C04A8000000   movsx ecx, byte ptr [esp+eax+000000A8]

DIKEN5678901234567890SP6SRCH32_D.DLL

:10001606 8B7C241C           mov edi, dword ptr [esp+1C]
:1000160A 03F9               add edi, ecx
:1000160C 40                 inc eax
:1000160D 3BC2               cmp eax, edx
:1000160F 897C241C           mov dword ptr [esp+1C], edi\====>累加结果
:10001613 72E9               jb 100015FE
```

## 代码段 4
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100015FC(C)

:10001615 8BFA               mov edi, edx
:10001617 83E703             and edi, 00000003  是否为 4 的倍数
:1000161A 7420               je 1000163C=======>是则转
:1000161C 33F6               xor esi, esi
:1000161E 33C9               xor ecx, ecx
:10001620 33C0               xor eax, eax
:10001622 85FF               test edi, edi
:10001624 7622               jbe 10001648
```

## 代码段 5
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001638(C)

:10001626 0FBEAC04A8000000   movsx ebp, byte ptr [esp+eax+000000A8]====>Name 串
:1000162E D3E5               shl ebp, cl
:10001630 83C108             add ecx, 00000008
:10001633 0BF5               or esi, ebp
:10001635 40                 inc eax
:10001636 3BC7               cmp eax, edi
:10001638 72EC               jb 10001626
:1000163A EB0C               jmp 10001648
```

## 代码段 6
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000161A(C)

:1000163C 8BB424A8000000     mov esi, dword ptr [esp+000000A8]
:10001643 B804000000         mov eax, 00000004
```

## 代码段 7
```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:10001624(C), :1000163A(U)

:10001648 3BC2               cmp eax, edx
:1000164A 7310               jnb 1000165C
```

## 代码段 8
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000165A(C)

:1000164C 8BAC04A8000000     mov ebp, dword ptr [esp+eax+000000A8]
:10001653 83C004             add eax, 00000004
:10001656 33F5               xor esi, ebp\=========>ESI 的值 261B0109(我的)
:10001658 3BC2               cmp eax, edx
:1000165A 72F0               jb 1000164C
```

## 代码段 9
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000164A(C)

:1000165C 8B542418           mov edx, dword ptr [esp+18]
:10001660 83E203             and edx, 00000003
:10001663 741D               je 10001682
:10001665 33ED               xor ebp, ebp
:10001667 33C9               xor ecx, ecx
:10001669 33C0               xor eax, eax
:1000166B 85D2               test edx, edx
:1000166D 761C               jbe 1000168B
```

## 代码段 10
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000167E(C)

:1000166F 0FBE7C0428         movsx edi, byte ptr [esp+eax+28]===>Code 串 1+用户名余串
:10001674 D3E7               shl edi, cl
:10001676 83C108             add ecx, 00000008
:10001679 0BEF               or ebp, edi
:1000167B 40                 inc eax
:1000167C 3BC2               cmp eax, edx
:1000167E 72EF               jb 1000166F
:10001680 EB09               jmp 1000168B
```

## 代码段 11
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001663(C)

:10001682 8B6C2428           mov ebp, dword ptr [esp+28]=======>串 1
:10001686 B804000000         mov eax, 00000004
```
