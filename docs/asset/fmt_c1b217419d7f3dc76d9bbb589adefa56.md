
```
.text:00513700    var_48          =   word   ptr -48h
.text:00513700    var_46          =   byte   ptr -46h
.text:00513700    var_44          =   dword  ptr -44h
.text:00513700    var_10          =   dword  ptr -10h
.text:00513700    var_C           =   dword  ptr -0Ch
.text:00513700    var_8           =   dword  ptr -8
.text:00513700    var_4           =   dword  ptr -4
.text:00513700    arg_0           =   dword  ptr  4
.text:00513700    arg_4           =   dword  ptr  8

.text:00513700                          push     0FFFFFFFFh
.text:00513702                          push     offset loc_7DC9A5
.text:00513707                          mov      eax, large fs:0
.text:0051370D                          push     eax
.text:0051370E                          mov      large fs:0, esp
.text:00513715                          sub      esp, 90h
.text:0051371B                          mov      eax, dword_8F6114
.text:00513720                          push     ebx
.text:00513721                          push     ebp
.text:00513722                          push     esi
.text:00513723                          mov      [esp+0A8h+var_10], eax
.text:0051372A                          mov      eax, [esp+0A8h+arg_4] ; 注册码
.text:00513731                          push     edi               ; int
.text:00513732                          xor      ebx, ebx
.text:00513734                          mov      ebp, ecx
.text:00513736                          mov      [esp+0ACh+var_90], ebx
.text:0051373A                          mov      edi, 1
.text:0051373F                          push     eax
.text:00513740                          lea      ecx, [esp+0B0h+var_98_RegCode]
.text:00513744                          mov      [esp+0B0h+var_8C], ebx ; 初值 0
.text:00513748                          mov      [esp+0B0h+var_7C], edi ; 初值 1
.text:0051374C                          mov      [esp+0B0h+var_99_ver], 10 ; 初值 10
.text:00513751                          mov      [esp+0B0h+var_91_regf], bl ; 初值 0
.text:00513755                          call     ds:ATL::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>::CStringT<char,StrTraitMFC_DLL<char,ATL::ChTraitsCRT<char>>>(char const *)
.text:0051375B                          lea      ecx, [esp+0ACh+var_98_RegCode] ; 注册码
.text:0051375F                          mov      [esp+0ACh+var_4], ebx
.text:00513766                          call     ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:0051376C                          cmp      eax, 29             ; 比较长度是否等于 29
.text:0051376F                          jz       loc_51398A
.text:00513775                          lea      ecx, [esp+0ACh+var_98_RegCode]
.text:00513779                          call     ds:ATL::CSimpleStringT<char,1>::GetLength(void)
.text:0051377F                          cmp      eax, 25             ; 比较长度是否等于 25
.text:00513782                          jz       loc_51398A
                                          ;这里我删去了旧版本里留下来的代码
                                          ;从上面可以看出，注册码长度必须等于 25 或者 29
                                          ;删去的这些事实上永远不会执行

.text:0051398A  loc_51398A:
.text:0051398A                          lea      ecx, [esp+0ACh+var_98_RegCode] ; 注册码
.text:0051398E                          lea      esi, [ebp+0D4h] ; 这里要返回一个数组
.text:00513994                          call     ds:ATL::CSimpleStringT<char,1>::operator char const *(void)
.text:0051399A                          push     eax               ; 注册码
.text:0051399B                          mov      ecx, esi          ; 返回值
.text:0051399D                          call     v_6F38C0          ; 关键 CALL，要跟进******
.text:005139A2                          test     al, al
.text:005139A4                          jz       short loc_5139C6
.text:005139A6                          cmp      [ebp+0D8h], edi ; [1]必须为 1
.text:005139AC                          jnz      short loc_5139C6
.text:005139AE                          lea      ecx, [esp+0ACh+var_88]
.text:005139B2                          push     ecx
.text:005139B3                          mov      ecx, esi
.text:005139B5                          call     sub_513450
.text:005139BA                          cmp      dword ptr [eax], 0 ; 比较协议类型是否等于 0
.text:005139BD                          mov      ebx, edi
.text:005139BF                          mov      [esp+0ACh+var_9A_f1], 1
.text:005139C4                          jnz      short loc_5139CB

.text:005139C6  loc_5139C6:
.text:005139C6                          mov      [esp+0ACh+var_9A_f1], 0

.text:005139CB  loc_5139CB:
.text:005139CB                          test     bl, 1
.text:005139CE                          jz       short loc_5139DC
.text:005139D0                          lea      ecx, [esp+0ACh+var_88]
.text:005139D4                          and      ebx, 0FFFFFFFEh
.text:005139D7                          call     sub_4FCCA0      ; 最低位不为 1 则协议字符串释放

.text:005139DC  loc_5139DC:
.text:005139DC                          mov      al, [esp+0ACh+var_9A_f1]
.text:005139E0                          test     al, al
.text:005139E2                          jz       loc_513B89
.text:005139E8                          push     0
.text:005139EA                          push     0
.text:005139EC                          push     8
.text:005139EE                          lea      ecx, [esp+0B8h+var_78]
.text:005139F2                          call     formver_61E3F0
.text:005139F7                          mov      edi, eax
.text:005139F9                          lea      edx, [esp+0ACh+var_88]
.text:005139FD                          push     edx
.text:005139FE                          mov      ecx, esi
.text:00513A00                          call     getver_512E80
.text:00513A05                          push     edi
.text:00513A06                          mov      ecx, eax
.text:00513A08                          mov      byte ptr [esp+0B0h+var_4], 2
.text:00513A10                          call     compver_61E410  ; 检查版本信息
.text:00513A15                          lea      ecx, [esp+0ACh+var_88]
.text:00513A19                          mov      [esp+0ACh+var_9A_f1], al
.text:00513A1D                          mov      byte ptr [esp+0ACh+var_4], 1
.text:00513A25                          call     nullcall
.text:00513A2A                          lea      ecx, [esp+0ACh+var_78]
.text:00513A2E                          mov      byte ptr [esp+0ACh+var_4], 0
.text:00513A36                          call     nullcall
.text:00513A3B                          mov      al, [esp+0ACh+var_9A_f1]
.text:00513A3F                          test     al, al
.text:00513A41                          jz       short loc_513A80
.text:00513A43                          lea      eax, [esp+0ACh+var_88]
.text:00513A47                          push     eax
.text:00513A48                          mov      ecx, esi
.text:00513A4A                          mov      [esp+0B0h+var_91_regf], 1 ; 注册成功标志
.text:00513A4F                          call     sub_513450
.text:00513A54                          mov      eax, [eax]
.text:00513A56                          lea      ecx, [ebp+0CCh]
.text:00513A5C                          push     eax
.text:00513A5D                          mov      byte ptr [esp+0B0h+var_4], 3
.text:00513A65                          call     GetLicType_61E070
.text:00513A6A                          lea      ecx, [esp+0ACh+var_88]
.text:00513A6E                          mov      byte ptr [esp+0ACh+var_4], 0
.text:00513A76                          call     sub_4FCCA0
.text:00513A7B                          jmp      loc_513B89

.text:00513A80  loc_513A80:
                    ;删除一些检查是否是老版本的注册码的代码

.text:00513B7C  loc_513B7C:
.text:00513B7C                          mov      al, [esp+0ACh+var_9A_f1]
.text:00513B80                          test     al, al
.text:00513B82                          jz       short loc_513B89
.text:00513B84                          mov      [esp+0ACh+var_99_ver], 11 ; 旧版本标志

.text:00513B89  loc_513B89:
.text:00513B89                          lea      ecx, [esp+0ACh+var_98_RegCode]
```
