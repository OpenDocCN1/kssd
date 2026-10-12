
#### 逆向分析汇编代码

```asm
:00477BF9 0AC2          or al, dl           -> al=al|dl
:00477BFB 243F          and al, 3F          -> al=al&0x3f
:00477BFD 8845F7        mov byte ptr [ebp-09], al -> 保存 al

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00477BE9(C)

:00477C00 8A45F9        mov al, byte ptr [ebp-07] -> 取 end 的第一位
:00477C03 8BD0          mov edx, eax        -> edx=eax
:00477C05 C1E204        shl edx, 04         -> edx=edx<<4
:00477C08 33C9          xor ecx, ecx
:00477C0A 8A4DFA        mov cl, byte ptr [ebp-06] -> 取 end 的第二位到 cl
:00477C0D C1E904        shr ecx, 04         -> ecx=ecx>>4
:00477C10 0AD1          or dl, cl           -> dl=dl|cl
:00477C12 80E23F        and dl, 3F          -> dl=dl&0x3f
:00477C15 8855F6        mov byte ptr [ebp-0A], dl -> 保存 dl
:00477C18 25FF000000    and eax, 000000FF   -> eax=eax&0xff
:00477C1D C1E802        shr eax, 02         -> eax=eax>>2
:00477C20 243F          and al, 3F          -> al=al&0x3f
:00477C22 8845F5        mov byte ptr [ebp-0B], al -> 保存 al
:00477C25 8D45FC        lea eax, dword ptr [ebp-04]
:00477C28 8BCB          mov ecx, ebx
:00477C2A BA01000000    mov edx, 00000001
:00477C2F E838C4F8FF    call 0040406C
:00477C34 BE04000000    mov esi, 00000004
:00477C39 8D5DF5        lea ebx, dword ptr [ebp-0B]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00477C5A(C)

:00477C3C 8D45F0        lea eax, dword ptr [ebp-10]
:00477C3F 33D2          xor edx, edx
:00477C41 8A13          mov dl, byte ptr [ebx] -> 依次取计算的结果
:00477C43 8A929DE44700  mov dl, byte ptr [edx+0047E49D] -> 用结果在基数相对位置上的字符
:00477C49 E8FEC0F8FF    call 00403D4C
:00477C4E 8B55F0        mov edx, dword ptr [ebp-10] -> 将找到的字符传到 edx
:00477C51 8BC7          mov eax, edi
:00477C53 E8D4C1F8FF    call 00403E2C
:00477C58 43            inc ebx
:00477C59 4E            dec esi
:00477C5A 75E0          jne 00477C3C -> 比较是否取完，没有继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00477B80(U)

:00477C5C 837DFC00      cmp dword ptr [ebp-04], 00000000 -> 比较 end 是否取完
:00477C60 0F851FFFFFFF  jne 00477B85        -> 没有则继续，后面位数的取法，见注册机源码
:00477C66 33C0          xor eax, eax
:00477C68 5A            pop edx
:00477C69 59            pop ecx
:00477C6A 59            pop ecx
:00477C6B 648910        mov dword ptr fs:[eax], edx
:00477C6E 688B7C4700    push 00477C8B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| :00477C89(U)

:00477C73 8D45F0        lea eax, dword ptr [ebp-10]
:00477C76 E829BFF8FF    call 00403BA4
:00477C7B 8D45FC        lea eax, dword ptr [ebp-04]
:00477C7E E821BFF8FF    call 00403BA4
:00477C83 C3            ret
```

#### 注册机源码

以下是在 `CB v6.0 win98 SE` 下调试通过的注册机源码：

```cpp
void __fastcall Tform1::OKBtnClick(TObject *Sender)
{
    int key_11[29] = {'~','T','u','r','n','I','d','e','@','$','I',
                      'n','t','o','P','r','o','f','i','t','$','w',
                      '/','e','B','o','o','k'}, *p = key_11;
    int key_22[15] = {'0','0','0','0','0','0','0','0','0','0','0',
                      '0','0','0','0'}, *p1 = key_22;
    
    String name, key_1, key_2, T, key_3, name_key, code_1, code_2, code_3,
           code_4, code_5, code_6, code_7, code_8, code_9,
           code_10, code_11, code_12, code_13, code_14,
           code_15, code_16;
           
    int a, b = 1, c = 1, d = 0, d_1 = 0, j = 1, k = 1;
    unsigned long e = 0, f = 0, g = 0, h = 0, i = 0, l = 0, m = 0, x = 0;

    if (UEdit->Text == "") { CEdit->Text = "未输入注册名！"; return; }
    if (UEdit->Text.Length() > 10) { CEdit->Text = "注册名不能大于 10 位！"; return; }

    if (UEdit->Text != "")
    {
        name = UEdit->Text;
        a = UEdit->Text.Length();
        
        while (c <= 12)
        {
            if (c > a) { name = name + name; d++; e = name[d]; }
            else e = name[c];
            f = e ^ a;
            g = f ^ b;
            c++;
            b++;
            key_1 = key_1 + char(g);
        }
        
        while (a >= 1)
        {
            h = name[a] ^ key_1[j];
            a--;
            j++;
            key_2 = key_2 + char(h);
        }
        
        if (j < 12)
        {
            while (j <= 12)
            {
                x = key_1[j];
                name_key = name_key + char(x);
                j++;
            }
            key_2 = key_2 + name_key;
            
            while (k <= 12)
            {
                i = key_2[k] ^ *p1;
                l = i ^ *p;
                m = l ^ k;
                k++;
                p++;
                p1++;
                key_3 = key_3 + char(m);
            }
            
            unsigned long a_1 = 0, b_1 = 0, c_1 = 0, a_11 = 0, b_11 = 0, c_11 = 0,
                           c_12 = 0, c_13 = 0, c_14 = 0, c_15 = 0;
            
            a_1 = key_3[3] & 0x3f;
            b_1 = key_3[2] << 2;
            b_11 = key_3[2] >> 6;
            c_15 = ((b_11 | b_1) ^ 0x100) & 0x3f;
            c_1 = ((b_11 | b_1) & 0x100) + key_3[1];
            c_11 = c_1 << 4;
            c_12 = key_3[2] >> 4;
            c_13 = (c_11 | c_12) & 0x3f;
            c_14 = (c_1 & 0xff) >> 2;
            
            char key_111[] = {'I','Y','A','G','P','X','D','J','Q','W',
                              'M','H','V','C','N','F','U','Z','R','B',
                              'K','E','S','O','L','T','t','f','k','y',
                              's','b','o','h','l','u','j','w','e','c',
                              'p','m','i','a','q','n','d','x','z','v',
                              'g','r','4','6','+','0','2','5','7','3',
                              '/','8','1','=','9'};
            
            code_1 = key_111[c_14];
            code_2 = key_111[c_13];
            code_3 = key_111[c_15];
            code_4 = key_111[a_1];
            
            unsigned long aa_1 = 0, ab_1 = 0, ac_1 = 0, aa_11 = 0, ab_11 = 0, ac_11 = 0,
                           ac_12 = 0, ac_13 = 0, ac_14 = 0, ac_15 = 0;
            aa_1 = key_3[6] & 0x3f;
            ab_1 = key_3[5] << 2;
            ab_11 = key_3[6] >> 6;
            ac_15 = ((ab_11 | ab_1) ^ 0x100) & 0x3f;
            ac_1 = ((ab_11 | ab_1) & 0x100) + key_3[4];
            ac_11 = ac_1 << 4;
            ac_12 = key_3[5] >> 4;
            ac_13 = (ac_11 | ac_12) & 0x3f;
            ac_14 = (ac_1 & 0xff) >> 2;
            
            code_5 = key_111[ac_14];
            code_6 = key_111[ac_13];
            code_7 = key_111[ac_15];
            code_8 = key_111[aa_1];
            
            unsigned long ba_1 = 0, bb_1 = 0, bc_1 = 0, ba_11 = 0, bb_11 = 0, bc_11 = 0,
                           bc_12 = 0, bc_13 = 0, bc_14 = 0, bc_15 = 0;
            ba_1 = key_3[9] & 0x3f;
            bb_1 = key_3[8] << 2;
            bb_11 = key_3[9] >> 6;
            bc_15 = ((bb_11 | bb_1) ^ 0x100) & 0x3f;
            bc_1 = ((bb_11 | bb_1) & 0x100) + key_3[7];
            bc_11 = bc_1 << 4;
            bc_12 = key_3[8] >> 4;
            bc_13 = (bc_11 | bc_12) & 0x3f;
            bc_14 = (bc_1 & 0xff) >> 2;
            
            code_9 = key_111[bc_14];
            code_10 = key_111[bc_13];
            code_11 = key_111[bc_15];
            code_12 = key_111[ba_1];
            
            unsigned long ca_1 = 0, cb_1 = 0, cc_1 = 0, ca_11 = 0, cb_11 = 0, cc_11 = 0,
                           cc_12 = 0, cc_13 = 0, cc_14 = 0, cc_15 = 0;
            ca_1 = key_3[12] & 0x3f;
            cb_1 = key_3[11] << 2;
            cb_11 = key_3[12] >> 6;
            cc_15 = ((cb_11 | cb_1) ^ 0x100) & 0x3f;
            cc_1 = ((cb_11 | cb_1) & 0x100) + key_3[10];
            cc_11 = cc_1 << 4;
            cc_12 = key_3[11] >> 4;
            cc_13 = (cc_11 | cc_12) & 0x3f;
            cc_14 = (cc_1 & 0xff) >> 2;
            
            code_13 = key_111[cc_14];
            code_14 = key_111[cc_13];
            code_15 = key_111[cc_15];
            code_16 = key_111[ca_1];
            
            CEdit->Text = CEdit->Text + code_1 + code_2 + code_3 + code_4
                         + code_5 + code_6 + code_7 + code_8 + code_9 + code_10 + code_11
                         + code_12 + code_13 + code_14 + code_15 + code_16;
        }
    }
}
```
