
# 算法分析

## 一、准备工作

有一个存储空间（或者叫数组）`M[3Dh]`，`ecx=0`

1. `M[ecx]=ecx*3-3`
2. `ecx=ecx+1`

重复执行 1、2 共 `3Dh` 次，再将 `name` 复制到 `M` 中，也就是上面的 `[ebp+ffffff70]`。

## 二、对 name 的处理

设 `name` 为 `X`，`name` 的长度为 `Z`，`n∈[1,3ch]`，`C=2h`，`S=0`

1. `T=[X(n)*X(n+2)-X(n+1)-C]`
2. `C=C+2`
3. `n=n+1`
4. `S=S+T+3`

重复执行 1、2、3、4 共 `Z-1` 次，最后结果放在 `S` 中也就是 `[ebp-10]` 中，只用到 `S` 的低 8 位。

## 三、计算注册码

**初始化:**

- `[ebp-4]=0`
- `ecx=0`
- `[5933f4]=0`

1. 计算 `ecx mod 4h`，将算出的值 `*4+00586C38`，从目录(见附录)中取出相应的地址 `p`，再计算 `(ecx mod 20h)/2` 算出 `q`，然后以 `p+q` 为地址从码表取出相应的值，假设为 `Q`。
   计算 `(ecx+S xor Q) mod 3bh`，以算出的值为索引值，从 `M` 中取出相应的值，假设该值为 `t`，计算 `[(t+ecx) mod 12h] mod 4h` 将算出的值 `*4+00586C38`，从目录(见附录)中取出相应的地址 `p`，`([(t+ecx) mod 12h] mod 20h)/2` 算出 `q`，然后以 `p+q` 为地址从码表取出相应的值 `AL`，计算 `[(AL xor t)+t] xor M[esi+1]*2`，将结果保存在 `M[esi]` 中。

2. 计算 `ecx mod 5h`，将算出的值保存到 `[ebp+10]` 中，如果等于 0 或等于 2 或等于 4，则计算 `(M[esi] mod 1Ah)+41h`。如果等于 1 或 3，则计算 `(M[esi] mod 1Ah)+30h`，结果放在 `[EBP+EDI+FFFFFF30]` 中，这就是注册码，当然这不是最后的注册码。

3. `ecx=ecx+1`

重复 1,2,3 直到 `ecx>3ch` 为止。在计算注册码的过程中，每隔 5 位插入一个 `"-"`。

## 汇编代码片段

```
016F:004187CE  JNL       004187E5   

016F:004187D0  MOV       EAX,[EBP+08]   

016F:004187D3  MOVSX     EDX,BYTE [ECX+EAX-01]   

016F:004187D8  MOVSX     EAX,BYTE [EDI+EAX]   

016F:004187DC  IMUL      EDX,EAX   

016F:004187DF  ADD       [005933F4],EDX     ;这个值好像没用到呀?   

016F:004187E5  INC       ECX   

016F:004187E6  CMP       ECX,BYTE +3C   

016F:004187E9  JL        NEAR 004186B7   
```

## 详细汇编代码

```
.text:004187F1 loc_4187F1:                     ; CODE XREF: sub_4185B8+295j   

.text:004187F1             mov    cl, [ebp+edx+var_90]   

.text:004187F8             lea    eax, [ebp+edx+var_90]   

.text:004187FF             cmp    cl, 80h   

.text:00418802             jb     short loc_418809   

.text:00418804             add    cl, 80h   

.text:00418807             mov    [eax], cl   

.text:00418809   

.text:00418809 loc_418809:                     ; CODE XREF: sub_4185B8+24Aj   

.text:00418809             mov    cl, [eax]   

.text:0041880B             cmp    cl, 7Fh   

.text:0041880E             jl     short loc_418815   

.text:00418810             sub    cl, 7Fh   

.text:00418813             mov    [eax], cl   

.text:00418815   

.text:00418815 loc_418815:                     ; CODE XREF: sub_4185B8+256j   

.text:00418815             mov    cl, [eax]   

.text:00418817             cmp    cl, 21h   

.text:0041881A             jg     short loc_418821   

.text:0041881C             add    cl, 21h   

.text:0041881F             mov    [eax], cl   

.text:00418821   

.text:00418821 loc_418821:                     ; CODE XREF: sub_4185B8+262j   

.text:00418821             cmp    byte ptr [eax], 22h   

.text:00418824             jnz    short loc_418829   

.text:00418826             mov    byte ptr [eax], 23h   

.text:00418829   

.text:00418829 loc_418829:                     ; CODE XREF: sub_4185B8+26Cj   

.text:00418829             cmp    byte ptr [eax], 27h   

.text:0041882C             jnz    short loc_418831   

.text:0041882E             mov    byte ptr [eax], 28h   

.text:00418831   

.text:00418831 loc_418831:                     ; CODE XREF: sub_4185B8+274j   

.text:00418831             cmp    byte ptr [eax], 2Eh   

.text:00418834             jnz    short loc_418839   

.text:00418836             mov    byte ptr [eax], 2Fh   

.text:00418839   

.text:00418839 loc_418839:                     ; CODE XREF: sub_4185B8+27Cj   

.text:00418839             cmp    byte ptr [eax], 60h   

.text:0041883C             jnz    short loc_418841   

.text:0041883E             mov    byte ptr [eax], 61h   

.text:00418841   

.text:00418841 loc_418841:                     ; CODE XREF: sub_4185B8+284j   

.text:00418841             cmp    byte ptr [eax], 7Ch   

.text:00418844             jnz    short loc_418849   

.text:00418846             mov    byte ptr [eax], 6Ch   

.text:00418849   

.text:00418849 loc_418849:                     ; CODE XREF: sub_4185B8+28Cj   

.text:00418849             inc    edx   

.text:0041884A             cmp    edx, 3Ch   

.text:0041884D             jl     short loc_4187F1   

.text:0041884F             mov    esi, [ebp+arg_C]   

.text:00418852             push   6   

.text:00418854             and    [ebp+var_81], 0   

.text:0041885B             and    [ebp+var_B9], 0   

.text:00418862             pop    ecx   

.text:00418863             lea    eax, [ebp+var_50]   

.text:00418866             push   eax            ; char *   

.text:00418867             lea    edi, [ebp+var_50]   

.text:0041886A             push   [ebp+arg_4]    ; char *   

.text:0041886D             mov    [ebp+var_CF], 30h   

.text:00418874             repe movsd   

.text:00418876             and    [ebp+var_39], 0   

.text:0041887A             mov    [ebp+var_4F], 30h   

.text:0041887E             mov    [ebp+var_46], 30h   

.text:00418882             mov    [ebp+var_44], 30h   

.text:00418886             mov    [ebp+var_42], 30h   

.text:0041888A             mov    [ebp+var_3A], 30h   

.text:0041888E             mov    [ebp+var_C6], 30h   

.text:00418895             mov    [ebp+var_C4], 30h   

.text:0041889C             mov    [ebp+var_C2], 30h   

.text:004188A3             mov    [ebp+var_BA], 30h   

.text:004188AA             call   _strcmp   

.text:004188AF             pop    ecx   

.text:004188B0             pop    ecx   

.text:004188B1             pop    esi   

.text:004188B2             test   eax, eax   

.text:004188B4             jnz    short loc_4188D6   

.text:004188B6             lea    eax, [ebp+var_50]   

.text:004188B9             push   eax            ; char *   

.text:004188BA             push   [ebp+arg_4]    ; char *   

.text:004188BD             call   _strcmp   

.text:004188C2             pop    ecx   

.text:004188C3             test   eax, eax   

.text:004188C5             pop    ecx   

.text:004188C6             jnz    short loc_418919   

.text:004188C8             push   [ebp+arg_4]    ; char *   

.text:004188CB             call   _strlen   

.text:004188D0             cmp    eax, 0Ch   

.text:004188D3             pop    ecx   

.text:004188D4             jz     short loc_418919   

.text:004188D6   

.text:004188D6 loc_4188D6:                     ; CODE XREF: sub_4185B8+2FCj   

.text:004188D6             lea    eax, [ebp+var_90]   

.text:004188DC             push   eax            ; char *   

.text:004188DD             lea    eax, [ebp+var_50]   

.text:004188E0             push   eax            ; char *   

.text:004188E1             call   _strcmp   

.text:004188E6             pop    ecx   

.text:004188E7             test   eax, eax   

.text:004188E9             pop    ecx   

.text:004188EA             jz     short loc_418912   

.text:004188EC             lea    eax, [ebp+var_D0] ;这是最后算出的注册码   

.text:004188F2             push   eax            ; char *   

.text:004188F3             lea    eax, [ebp+var_50] ;这是我们输入的注册码   

.text:004188F6             push   eax            ; char *   

.text:004188F7             call   _strcmp        ;比较注册码   

.text:004188FC             pop    ecx   

.text:004188FD             test   eax, eax   

.text:004188FF             pop    ecx   

.text:00418900             jz     short loc_418912   

.text:00418902             mov    dword_593518, 1   

.text:0041890C   

.text:0041890C loc_41890C:                     ; CODE XREF: sub_4185B8+29j   
```
