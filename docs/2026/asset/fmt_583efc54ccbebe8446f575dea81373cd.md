
# 媲美AcdSee软件－IrfanView32 v3.97注册码算法分析

与AcdSee媲美地看图软件 IrfanView32 v3.97 注册码算法分析

日期：2005年5月2日　　破解人：Baby2008

-------------------------------------------------------------------------------------------------------------------------

『软件名称』：IrfanView32 v3.97

『软件大小』：872B

『下载地址』：fc5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2J5k6X3q4F1N6X3W2W2N6#2)9J5k6h3y4G2L8g2)9J5c8R3`.`.

『保护方式』：注册码保护

『破解声明』：初学Crack，只是感兴趣,失误之处敬请诸位大侠赐教!

『破解工具』：OllyDbg.V1.10 聆风听雨汉化第二版、PeID 0.93、ASPackDie v1.41.HH

## 破解过程

PeID查壳,ASPack 2.12 -> Alexey Solodovnikov，用`ASPackDie v1.41.HH`搞定,OD载入,F9运行，输入注册信息,用户名:`Baby2008`,注册码:`1234567890`,下bp `GetDlgItemTextA`中断,点击确定OD中断在:

```
77D6AC06 >  8BFF            mov edi,edi                                    ; USER32.GetDlgItemTextA
77D6AC08    55              push ebp
77D6AC09    8BEC            mov ebp,esp
77D6AC0B    FF75 0C         push dword ptr ss:[ebp+C]
77D6AC0E    FF75 08         push dword ptr ss:[ebp+8]
77D6AC11    E8 8EA6FBFF     call USER32.GetDlgItem
77D6AC16    85C0            test eax,eax
77D6AC18    74 0E           je short USER32.77D6AC28
```

取消bc `GetDlgItemTextA`，Alt+F9

```
0043E52B    8B15 301A5000   mov edx,dword ptr ds:[501A30]                  ; 返回这里,用户名
0043E531    68 00010000     push 100
0043E536    81C2 04010000   add edx,104
0043E53C    52              push edx
0043E53D    68 2D070000     push 72D
0043E542    56              push esi
0043E543    FFD7            call edi
0043E545    6A 01           push 1
0043E547    56              push esi
0043E548    FF15 C8F34C00   call dword ptr ds:[<&USER32.EndDialog>]        ; USER32.EndDialog
0043E54E    5F              pop edi
0043E54F    5E              pop esi
0043E550    33C0            xor eax,eax
0043E552    C2 1000         retn 10
```

继续返回到主模块:

```
00464A72    FF15 28F54C00   call dword ptr ds:[<&USER32.DialogBoxParamA>]  ; USER32.DialogBoxParamA
00464A78    85C0            test eax,eax                                   ; 返回这里
00464A7A    0F84 AD000000   je Unpacked.00464B2D
00464A80    8DBC24 E4050000 lea edi,dword ptr ss:[esp+5E4]                 ; 用户名
00464A87    83C9 FF         or ecx,FFFFFFFF
00464A8A    33C0            xor eax,eax
00464A8C    F2:AE           repne scas byte ptr es:[edi]
00464A8E    F7D1            not ecx
00464A90    49              dec ecx
00464A91    83F9 02         cmp ecx,2
00464A94    0F82 5C010000   jb Unpacked.00464BF6                           ; 用户名长度要求>2
00464A9A    8DBC24 E4050000 lea edi,dword ptr ss:[esp+5E4]
00464AA1    83C9 FF         or ecx,FFFFFFFF
00464AA4    F2:AE           repne scas byte ptr es:[edi]
00464AA6    F7D1            not ecx
00464AA8    49              dec ecx
00464AA9    83F9 55         cmp ecx,55
00464AAC    0F87 44010000   ja Unpacked.00464BF6                           ; 用户名长度要求<$55
00464AB2    8DBC24 E8060000 lea edi,dword ptr ss:[esp+6E8]                 ; 注册码，记为SN
00464AB9    83C9 FF         or ecx,FFFFFFFF
00464ABC    33D2            xor edx,edx                                    ; i,i=0
00464ABE    F2:AE           repne scas byte ptr es:[edi]
00464AC0    F7D1            not ecx
00464AC2    49              dec ecx
00464AC3    85C9            test ecx,ecx
00464AC5    7E 75           jle short Unpacked.00464B3C                    ; 注册码长度不能为0
00464AC7    8A8414 E8060000 mov al,byte ptr ss:[esp+edx+6E8]               ; SN[i]
00464ACE    3C 30           cmp al,30
00464AD0    7C 04           jl short Unpacked.00464AD6                     ; SN[i]<'0'，置非数字字符标志EBX=1
00464AD2    3C 39           cmp al,39
00464AD4    7E 05           jle short Unpacked.00464ADB                    ; SN[1]<='9'
00464AD6    BB 01000000     mov ebx,1                                      ; 置非数字字符标志EBX=1
00464ADB    8DBC24 E8060000 lea edi,dword ptr ss:[esp+6E8]                 ; SN
00464AE2    83C9 FF         or ecx,FFFFFFFF
00464AE5    33C0            xor eax,eax
00464AE7    42              inc edx
00464AE8    F2:AE           repne scas byte ptr es:[edi]
00464AEA    F7D1            not ecx
00464AEC    49              dec ecx
00464AED    3BD1            cmp edx,ecx                                    ; ECX=Length(SN),EDX=i
00464AEF  ^ 7C D6           jl short Unpacked.00464AC7                     ; 循环要求注册码为数字字符
00464AF1    85DB            test ebx,ebx
00464AF3    74 47           je short Unpacked.00464B3C
00464AF5    8B0D 28275000   mov ecx,dword ptr ds:[502728]                  ; Unpacked.00400000
00464AFB    68 04010000     push 104
00464B00    68 40355000     push Unpacked.00503540                         ; ASCII "No file loaded (use File->Open menu)"
00464B05    68 D6040000     push 4D6
00464B0A    51              push ecx
00464B0B    FF15 7CF44C00   call dword ptr ds:[<&USER32.LoadStringA>]      ; USER32.LoadStringA
00464B11    8B15 64395000   mov edx,dword ptr ds:[503964]
00464B17    68 30200000     push 2030
00464B1C    68 E03C5000     push Unpacked.00503CE0                         ; ASCII "IrfanView"
00464B21    68 40355000     push Unpacked.00503540                         ; ASCII "No file loaded (use File->Open menu)"
00464B26    52              push edx
00464B27    FF15 E8F34C00   call dword ptr ds:[<&USER32.MessageBoxA>]      ; USER32.MessageBoxA
00464B2D    33C0            xor eax,eax
00464B2F    5F              pop edi
00464B30    5E              pop esi
00464B31    5D              pop ebp
00464B32    5B              pop ebx
00464B33    81C4 E4090000   add esp,9E4
00464B39    C2 1000         retn 10
00464B3C    8D8424 E8060000 lea eax,dword ptr ss:[esp+6E8]                 ; 注册码
00464B43    8D8C24 E4050000 lea ecx,dword ptr ss:[esp+5E4]                 ; 用户名
00464B4A    50              push eax
00464B4B    51              push ecx
00464B4C    E8 DF96FDFF     call Unpacked.0043E230                         ; 关键
00464B51    83C4 08         add esp,8
00464B54    85C0            test eax,eax
00464B56    75 33           jnz short Unpacked.00464B8B                    ; 爆破
00464B58    8B15 28275000   mov edx,dword ptr ds:[502728]                  ; Unpacked.00400000
00464B5E    68 04010000     push 104
00464B63    68 40355000     push Unpacked.00503540                         ; ASCII "No file loaded (use File->Open menu)"
00464B68    68 D6040000     push 4D6
00464B6D    52              push edx
00464B6E    FF15 7CF44C00   call dword ptr ds:[<&USER32.LoadStringA>]      ; USER32.LoadStringA
```

-------------------------------------------------------------------------------------------------------------------------

功能:先判断注册码的格式，再调用`00464B4C    E8 DF96FDFF     call Unpacked.0043E230`进行注册验证，跟进：

-------------------------------------------------------------------------------------------------------------------------

```
0043E230    8B4424 08       mov eax,dword ptr ss:[esp+8]                   ; 注册码
0043E234    83EC 14         sub esp,14
0043E237    53              push ebx
0043E238    55              push ebp
0043E239    56              push esi
0043E23A    57              push edi
0043E23B    50              push eax
0043E23C    33DB            xor ebx,ebx
0043E23E    E8 4A500800     call Unpacked.004C328D
0043E243    8B7424 2C       mov esi,dword ptr ss:[esp+2C]                  ; 用户名
0043E247    8BE8            mov ebp,eax
0043E249    8BFE            mov edi,esi
0043E24B    83C9 FF         or ecx,FFFFFFFF
0043E24E    33C0            xor eax,eax
0043E250    83C4 04         add esp,4
0043E253    33D2            xor edx,edx                                    ; i,i=0
0043E255    F2:AE           repne scas byte ptr es:[edi]
0043E257    F7D1            not ecx
0043E259    49              dec ecx
0043E25A    85C9            test ecx,ecx                                   ; Length(用户名)
0043E25C    7E 17           jle short Unpacked.0043E275
0043E25E    0FBE0C32        movsx ecx,byte ptr ds:[edx+esi]                ; Name[i]
0043E262    03D9            add ebx,ecx
0043E264    8BFE            mov edi,esi
```
