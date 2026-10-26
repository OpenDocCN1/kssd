
素描软件 `Photo to ColorSketch v3.11` 是一款可以把自己的照片或是其它图片制作成彩色或者黑白素描画的软件,分析了下,算法居然和`Cartoon_Maker v3.15` 的一模一样，看主要代码：

```
0048D46C   .  837D E0 00    cmp     dword ptr [ebp-20], 0            ;  注册名不为空
0048D470   .  75 15         jnz     short colorske.0048D487
0048D472   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D478   .  BA 64D84800   mov     edx, colorske.0048D864           ;  Please fill in your UserCode
0048D47D   .  E8 6E27FBFF   call    colorske.0043FBF0
0048D482   .  E9 6B030000   jmp     colorske.0048D7F2
0048D487   >  8D55 DC       lea     edx, dword ptr [ebp-24]
0048D48A   .  8B83 04030000 mov     eax, dword ptr [ebx+304]
0048D490   .  E8 2B27FBFF   call    colorske.0043FBC0
0048D495   .  837D DC 00    cmp     dword ptr [ebp-24], 0
0048D499   .  75 15         jnz     short colorske.0048D4B0          ;  注册码不为空
0048D49B   .  8B83 04030000 mov     eax, dword ptr [ebx+304]
0048D4A1   .  BA 8CD84800   mov     edx, colorske.0048D88C           ;  Please fill in your Serial Number
0048D4A6   .  E8 4527FBFF   call    colorske.0043FBF0
0048D4AB   .  E9 42030000   jmp     colorske.0048D7F2
0048D4B0   >  8D55 D8       lea     edx, dword ptr [ebp-28]
0048D4B3   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D4B9   .  E8 0227FBFF   call    colorske.0043FBC0
0048D4BE   .  8B45 D8       mov     eax, dword ptr [ebp-28]
0048D4C1   .  8A00          mov     al, byte ptr [eax]               ;  注册码第1位
0048D4C3   .  8845 FF       mov     byte ptr [ebp-1], al
0048D4C6   .  8D55 D4       lea     edx, dword ptr [ebp-2C]
0048D4C9   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D4CF   .  E8 EC26FBFF   call    colorske.0043FBC0
0048D4D4   .  8B45 D4       mov     eax, dword ptr [ebp-2C]
0048D4D7   .  8A40 01       mov     al, byte ptr [eax+1]             ;  注册码第2位
0048D4DA   .  8845 FE       mov     byte ptr [ebp-2], al
0048D4DD   .  8D55 D0       lea     edx, dword ptr [ebp-30]
0048D4E0   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D4E6   .  E8 D526FBFF   call    colorske.0043FBC0
0048D4EB   .  8B45 D0       mov     eax, dword ptr [ebp-30]
0048D4EE   .  8A40 02       mov     al, byte ptr [eax+2]
0048D4F1   .  8845 FD       mov     byte ptr [ebp-3], al             ;  注册码第3位
0048D4F4   .  8D55 CC       lea     edx, dword ptr [ebp-34]
0048D4F7   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D4FD   .  E8 BE26FBFF   call    colorske.0043FBC0
0048D502   .  8B45 CC       mov     eax, dword ptr [ebp-34]
0048D505   .  8A40 03       mov     al, byte ptr [eax+3]
0048D508   .  8845 FC       mov     byte ptr [ebp-4], al             ;  注册码第4位
0048D50B   .  8D55 C8       lea     edx, dword ptr [ebp-38]
0048D50E   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D514   .  E8 A726FBFF   call    colorske.0043FBC0
0048D519   .  8B45 C8       mov     eax, dword ptr [ebp-38]
0048D51C   .  8A40 04       mov     al, byte ptr [eax+4]             ;  注册码第5位
0048D51F   .  8845 FB       mov     byte ptr [ebp-5], al
0048D522   .  8D55 C4       lea     edx, dword ptr [ebp-3C]
0048D525   .  8B83 00030000 mov     eax, dword ptr [ebx+300]
0048D52B   .  E8 9026FBFF   call    colorske.0043FBC0
0048D530   .  8B45 C4       mov     eax, dword ptr [ebp-3C]
0048D533   .  8A40 05       mov     al, byte ptr [eax+5]             ;  注册码第6位
0048D536   .  33D2          xor     edx, edx
0048D538   .  8A55 FF       mov     dl, byte ptr [ebp-1]             ;  注册码第1位
0048D53B   .  83C2 02       add     edx, 2                           ;  edx := edx + $2;
0048D53E   .  0FB675 FE     movzx   esi, byte ptr [ebp-2]            ;  注册码第2位
0048D542   .  83C6 09       add     esi, 9                           ;  esi := esi + $9;
0048D545   .  33C9          xor     ecx, ecx
0048D547   .  8A4D FD       mov     cl, byte ptr [ebp-3]             ;  注册码第3位
0048D54A   .  41            inc     ecx                              ;  ecx := ecx + $1;
0048D54B   .  894D F4       mov     dword ptr [ebp-C], ecx
0048D54E   .  0FB67D FC     movzx   edi, byte ptr [ebp-4]            ;  注册码第4位
0048D552   .  83EF 04       sub     edi, 4                           ;  edi := edi - $4;
0048D555   .  33C9          xor     ecx, ecx
0048D557   .  8A4D FB       mov     cl, byte ptr [ebp-5]             ;  注册码第5位
0048D55A   .  49            dec     ecx                              ;  ecx := ecx - $1;
0048D55B   .  894D F0       mov     dword ptr [ebp-10], ecx
0048D55E   .  25 FF000000   and     eax, 0FF                         ;  注册码第6位
0048D563   .  83C0 05       add     eax, 5                           ;  eax := eax + $5;
0048D566   .  8855 EF       mov     byte ptr [ebp-11], dl
0048D569   .  8BD6          mov     edx, esi
0048D56B   .  8855 EE       mov     byte ptr [ebp-12], dl
0048D56E   .  8A55 F4       mov     dl, byte ptr [ebp-C]
0048D571   .  8855 ED       mov     byte ptr [ebp-13], dl
0048D574   .  8BD7          mov     edx, edi
0048D576   .  8855 EC       mov     byte ptr [ebp-14], dl
0048D579   .  8A55 F0       mov     dl, byte ptr [ebp-10]
0048D57C   .  8855 EB       mov     byte ptr [ebp-15], dl
0048D57F   .  8845 EA       mov     byte ptr [ebp-16], al
0048D582   .  8D45 B8       lea     eax, dword ptr [ebp-48]
0048D585   .  8A55 ED       mov     dl, byte ptr [ebp-13]
0048D588   .  8850 01       mov     byte ptr [eax+1], dl
0048D58B   .  C600 01       mov     byte ptr [eax], 1
0048D58E   .  8D55 B8       lea     edx, dword ptr [ebp-48]
0048D591   .  8D45 B4       lea     eax, dword ptr [ebp-4C]
0048D594   .  E8 C358F7FF   call    colorske.00402E5C
0048D599   .  8D45 B0       lea     eax, dword ptr [ebp-50]
0048D59C   .  8A55 EA       mov     dl, byte ptr [ebp-16]
0048D59F   .  8850 01       mov     byte ptr [eax+1], dl
0048D5A2   .  C600 01       mov     byte ptr [eax], 1
0048D5A5   .  8D55 B0       lea     edx, dword ptr [ebp-50]
0048D5A8   .  8D45 B4       lea     eax, dword ptr [ebp-4C]
0048D5AB   .  B1 02         mov     cl, 2
0048D5AD   .  E8 7A58F7FF   call    colorske.00402E2C
0048D5B2   .  8D55 B4       lea     edx, dword ptr [ebp-4C]
0048D5B5   .  8D45 BC       lea     eax, dword ptr [ebp-44]
0048D5B8   .  E8 8B70F7FF   call    colorske.00404648
0048D5BD   .  FF75 BC       push    dword ptr [ebp-44]
0048D5C0   .  8D55 AC       lea     edx, dword ptr [ebp-54]
0048D5C3   .  8BC6          mov     eax, esi
0048D5C5   .  E8 4AB3F7FF   call    colorske.00408914
0048D5CA   .  FF75 AC       push    dword ptr [ebp-54]
0048D5CD   .  8D45 A8       lea     eax, dword ptr [ebp-58]
0048D5D0   .  8A55 EB       mov     dl, byte ptr [ebp-15]
0048D5D3   .  E8 F46FF7FF   call    colorske.004045CC
0048D5D8   .  FF75 A8       push    dword ptr [ebp-58]
0048D5DB   .  8D45 A4       lea     eax, dword ptr [ebp-5C]
0048D5DE   .  8A55 EE       mov     dl, byte ptr [ebp-12]
0048D5E1   .  E8 E66FF7FF   call    colorske.004045CC
0048D5E6   .  FF75 A4       push    dword ptr [ebp-5C]
0048D5E9   .  8D55 A0       lea     edx, dword ptr [ebp-60]
0048D5EC   .  8BC7          mov     eax, edi
0048D5EE   .  E8 21B3F7FF   call    colorske.00408914
0048D5F3   .  FF75 A0       push    dword ptr [ebp-60]
0048D5F6   .  8D45 9C       lea     eax, dword ptr [ebp-64]
0048D5F9   .  8A55 EF       mov     dl, byte ptr [ebp-11]
0048D5FC   .  E8 CB6FF7FF   call    colorske.004045CC
0048D601   .  FF75 9C       push    dword ptr [ebp-64]
0048D604   .  8D45 98       lea     eax, dword ptr [ebp-68]
0048D607   .  8A55 EC       mov     dl, byte ptr [ebp-14]
0048D60A   .  E8 BD6FF7FF   call    colorske.004045CC
0048D60F   .  FF75 98       push    dword ptr [ebp-68]
0048D612   .  8D45 C0       lea     eax, dword ptr [ebp-40]
0048D615   .  BA 07000000   mov     edx, 7
0048D61A   .  E8 4571F7FF   call    colorske.00404764                ;  重新排顺序,排成:新字符串的 char(4)+char(1)+ ord(5) +char(2)+char(5)+ ord(3) +char(6)+char(3) ;
0048D61F   .  8B55 C0       mov     edx, dword ptr [ebp-40]          ;  结果送到edx,即真的注册码!
```
