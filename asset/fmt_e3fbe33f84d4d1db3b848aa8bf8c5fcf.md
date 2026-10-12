
```asm
.text:006F3B1D lea ecx, [esp+5Ch+var_38_s]
.text:006F3B21 mov byte ptr [esp+5Ch+var_4], 6
.text:006F3B26 call ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::~CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(void)
.text:006F3B2C xor esi, esi
.text:006F3B2E mov edi, edi
.text:006F3B30
.text:006F3B30 loc_6F3B30:
.text:006F3B30 push esi
.text:006F3B31 lea ecx, [esp+60h+var_48_md5s]
.text:006F3B35 call ds:ATL::CSimpleStringT<char,1>::operator\
.text:006F3B3B cmp al, '0' ; 把'0'替换成'R'
.text:006F3B3D lea ecx, [esp+5Ch+var_48_md5s]
.text:006F3B41 jnz short loc_6F3B47
.text:006F3B43 push 'R'
.text:006F3B45 jmp short loc_6F3B58
.text:006F3B47
.text:006F3B47 loc_6F3B47:
.text:006F3B47 push esi
.text:006F3B48 call ds:ATL::CSimpleStringT<char,1>::operator\
.text:006F3B4E cmp al, '1' ; 把'1'替换成'M'
.text:006F3B50 jnz short loc_6F3B5F
.text:006F3B52 push 'M'
.text:006F3B54 lea ecx, [esp+60h+var_48_md5s]
.text:006F3B58
.text:006F3B58 loc_6F3B58:
.text:006F3B58 push esi
.text:006F3B59 call ds:ATL::CSimpleStringT<char,1>::SetAt(int,char)
.text:006F3B5F
.text:006F3B5F loc_6F3B5F:
.text:006F3B5F inc esi
.text:006F3B60 cmp esi, 4
.text:006F3B63 jl short loc_6F3B30
.text:006F3B65 lea ecx, [esp+5Ch+var_48_md5s] ; 转换成大写
.text:006F3B69 call ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::MakeUpper(void)
.text:006F3B6F lea ecx, [esp+5Ch+var_3C_r4]
.text:006F3B73 call ds:ATL::CSimpleStringT<char,1>::operator char const *(void)
.text:006F3B79 push eax
.text:006F3B7A lea ecx, [esp+60h+var_48_md5s]
.text:006F3B7E call ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::Compare(char const *)
.text:006F3B84 test eax, eax ; 和注册码的最后 4 个字符比较
.text:006F3B86 jnz loc_6F3E52 ; 不等则校验失败
.text:006F3B8C mov [esp+5Ch+var_34_sa2], ebp ; 构造一个动态数组
.text:006F3B90 mov [esp+5Ch+var_30_pStr], edi
.text:006F3B94 mov [esp+5Ch+var_24], edi
.text:006F3B98 mov [esp+5Ch+var_28], edi
.text:006F3B9C mov [esp+5Ch+var_2C_Len], edi
.text:006F3BA0 push 0FFFFFFFFh
.text:006F3BA2 push 60 ; 数组大小
.text:006F3BA4 lea ecx, [esp+64h+var_34_sa2]
.text:006F3BA8 mov byte ptr [esp+64h+var_4], 8
.text:006F3BAD call new_6F3410 ; 分配 60 个字节空间
.text:006F3BB2 lea ecx, [esp+5Ch+var_4C_r21] ; r21
.text:006F3BB6 xor esi, esi
.text:006F3BB8 xor ebx, ebx
.text:006F3BBA call ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:006F3BC0 test eax, eax
.text:006F3BC2 jle loc_6F3C4E
.text:006F3BC8 xor ebp, ebp
.text:006F3BCA lea ebx, [ebx+0]
.text:006F3BD0
.text:006F3BD0 loc_6F3BD0:
.text:006F3BD0 push ebx
.text:006F3BD1 lea ecx, [esp+60h+var_4C_r21] ; 取 r21 一个字符
.text:006F3BD5 call ds:ATL::CSimpleStringT<char,1>::operator\
.text:006F3BDB or edi, 0FFFFFFFFh ; 没找到则返回-1
.text:006F3BDE xor ecx, ecx
.text:006F3BE0
.text:006F3BE0 loc_6F3BE0:
.text:006F3BE0 cmp al, byte ptr ds:Table1_865058[ecx] ; "CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8"
.text:006F3BE6 jz short loc_6F3BF0
.text:006F3BE8 inc ecx
.text:006F3BE9 cmp ecx, 32
.text:006F3BEC jl short loc_6F3BE0
.text:006F3BEE jmp short loc_6F3BF2 ; 查表
.text:006F3BF0
.text:006F3BF0 loc_6F3BF0:
.text:006F3BF0 mov edi, ecx ; 取得在表中的索引
.text:006F3BF2
.text:006F3BF2 loc_6F3BF2:
.text:006F3BF2 xor ecx, ecx
.text:006F3BF4
.text:006F3BF4 loc_6F3BF4:
.text:006F3BF4 lea edx, [ecx+ebp]
.text:006F3BF7 mov eax, offset Table2_864FE0
.text:006F3BFC lea esp, [esp+0]
.text:006F3C00
.text:006F3C00 loc_6F3C00:
.text:006F3C00 cmp edx, [eax]
.text:006F3C02 jz short loc_6F3C34
.text:006F3C04 add eax, 4
.text:006F3C07 cmp eax, offset Table1_865058 ; "CA5BDWF4H9PJK3SUM2XEL7GRZNQTY6V8"
.text:006F3C0C jl short loc_6F3C00
.text:006F3C0E test esi, esi
.text:006F3C10 jl failed_6F3EA9
.text:006F3C16 cmp esi, [esp+5Ch+var_2C_Len]
.text:006F3C1A jge failed_6F3EA9
.text:006F3C20 mov edx, 1
.text:006F3C25 shl edx, cl
.text:006F3C27 test edx, edi
.text:006F3C29 mov edx, [esp+5Ch+var_30_pStr]
.text:006F3C2D setnz al
.text:006F3C30 mov [edx+esi], al
.text:006F3C33 inc esi
.text:006F3C34
.text:006F3C34 loc_6F3C34:
.text:006F3C34 inc ecx
.text:006F3C35 cmp ecx, 5
.text:006F3C38 jl short loc_6F3BF4
.text:006F3C3A lea ecx, [esp+5Ch+var_4C_r21]
.text:006F3C3E inc ebx
.text:006F3C3F add ebp, 5
.text:006F3C42 call ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:006F3C48 cmp ebx, eax
.text:006F3C4A jl short loc_6F3BD0
.text:006F3C4C xor edi, edi
.text:006F3C4E
.text:006F3C4E loc_6F3C4E:
.text:006F3C4E mov esi, [esp+5Ch+var_30_pStr]
.text:006F3C52 mov ebx, [esp+5Ch+var_2C_Len]
.text:006F3C56 xor edx, edx
.text:006F3C58 mov eax, 21
.text:006F3C5D lea ecx, [ecx+0]
.text:006F3C60
.text:006F3C60 loc_6F3C60:
.text:006F3C60 cmp eax, edi
.text:006F3C62 jl failed_6F3EA9
.text:006F3C68 cmp eax, ebx
.text:006F3C6A jge failed_6F3EA9
.text:006F3C70 cmp byte ptr [esi+eax], 0
.text:006F3C74 jz short loc_6F3C82
.text:006F3C76 lea ecx, [eax-21]
.text:006F3C79 mov ebp, 1
.text:006F3C7E shl ebp, cl
.text:006F3C80 or edx, ebp
.text:006F3C82
```
