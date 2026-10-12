
### 汇编指令分析与注释

#### 代码片段一

```
:0041BDFD 8BCA                   mov ecx, edx
:0041BDFF 83E103                 and ecx, 00000003
:0041BE02 F3                     repz
:0041BE03 A4                     movsb
:0041BE04 8A08                   mov cl, byte ptr [eax]
:0041BE06 84C9                   test cl, cl
:0041BE08 740D                   je 0041BE17

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BE15(C)
|

:0041BE0A 80E97F                 sub cl, 7F   <--加密字符串的解密算法
:0041BE0D 8808                   mov byte ptr [eax], cl
:0041BE0F 8A4801                 mov cl, byte ptr [eax+01]
:0041BE12 40                     inc eax
:0041BE13 84C9                   test cl, cl
:0041BE15 75F3                   jne 0041BE0A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BE08(C)
|

:0041BE17 8D83D4060000           lea eax, dword ptr [ebx+000006D4]
:0041BE1D 50                     push eax
:0041BE1E 8D442418               lea eax, dword ptr [esp+18]
:0041BE22 50                     push eax
:0041BE23 E8A8590000             call 004217D0   <--检查注册文件中是否有黑名单中的"PC'"
:0041BE28 83C408                 add esp, 00000008
:0041BE2B 85C0                   test eax, eax
:0041BE2D 7414                   je 0041BE43   <--应该跳转
:0041BE2F C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920   <--送注册失败处理子程序的地址

* Possible Reference to String Resource ID=00001: "萣
*

                           |

:0041BE39 C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001   <--把注册失败值送全局变量 reg

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BE2D(C)
|

:0041BE43 8BFD                   mov edi, ebp
:0041BE45 83C9FF                 or ecx, FFFFFFFF
:0041BE48 33C0                   xor eax, eax
:0041BE4A 8D542414               lea edx, dword ptr [esp+14]
:0041BE4E F2                     repnz
:0041BE4F AE                     scasb
:0041BE50 F7D1                   not ecx
:0041BE52 2BF9                   sub edi, ecx
:0041BE54 8BC1                   mov eax, ecx
:0041BE56 8BF7                   mov esi, edi
:0041BE58 8BFA                   mov edi, edx
:0041BE5A C1E902                 shr ecx, 02
:0041BE5D F3                     repz
:0041BE5E A5                     movsd
:0041BE5F 8BC8                   mov ecx, eax
:0041BE61 83E103                 and ecx, 00000003
:0041BE64 F3                     repz
:0041BE65 A4                     movsb
:0041BE66 8D4C2414               lea ecx, dword ptr [esp+14]
:0041BE6A 51                     push ecx
:0041BE6B E881420100             call 004300F1

* Possible StringData Ref from Data Obj ->"子难*"   <--该字符串解密后为"XTERM"

                           |

:0041BE70 BF08474500             mov edi, 00454708
:0041BE75 83C9FF                 or ecx, FFFFFFFF
:0041BE78 33C0                   xor eax, eax
:0041BE7A 83C404                 add esp, 00000004
:0041BE7D F2                     repnz
:0041BE7E AE                     scasb
:0041BE7F F7D1                   not ecx
:0041BE81 2BF9                   sub edi, ecx
:0041BE83 5D                     pop ebp
:0041BE84 8BF7                   mov esi, edi
:0041BE86 8BD1                   mov edx, ecx
:0041BE88 8DBBD4060000           lea edi, dword ptr [ebx+000006D4]
:0041BE8E 8BC7                   mov eax, edi
:0041BE90 C1E902                 shr ecx, 02
:0041BE93 F3                     repz
:0041BE94 A5                     movsd
:0041BE95 8BCA                   mov ecx, edx
:0041BE97 83E103                 and ecx, 00000003
:0041BE9A F3                     repz
:0041BE9B A4                     movsb
:0041BE9C 8A08                   mov cl, byte ptr [eax]
:0041BE9E 84C9                   test cl, cl
:0041BEA0 740D                   je 0041BEAF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BEAD(C)
|

:0041BEA2 80E97F                 sub cl, 7F   <--加密字符串的解密算法
:0041BEA5 8808                   mov byte ptr [eax], cl
:0041BEA7 8A4801                 mov cl, byte ptr [eax+01]
:0041BEAA 40                     inc eax
:0041BEAB 84C9                   test cl, cl
:0041BEAD 75F3                   jne 0041BEA2

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BEA0(C)
|

:0041BEAF 8D83D4060000           lea eax, dword ptr [ebx+000006D4]
:0041BEB5 50                     push eax
:0041BEB6 8D442414               lea eax, dword ptr [esp+14]
:0041BEBA 50                     push eax
:0041BEBB E810590000             call 004217D0   <--检查注册文件中是否有黑名单中的"XTERM"
:0041BEC0 83C408                 add esp, 00000008
:0041BEC3 85C0                   test eax, eax
:0041BEC5 7414                   je 0041BEDB   <--应该跳转
:0041BEC7 C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920   <--送注册失败处理子程序的地址

* Possible Reference to String Resource ID=00001: "萣
*

                           |

:0041BED1 C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001   <--把注册失败值送全局变量 reg

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041BEC5(C)
|

:0041BEDB 5F                     pop edi
:0041BEDC 5E                     pop esi
:0041BEDD 5B                     pop ebx
:0041BEDE 81C404040000           add esp, 00000404
:0041BEE4 C3                     ret
```

//****...****

#### 第七次调用 `call eax` 会来到如下地方：

```
:0041B560 81EC04040000           sub esp, 00000404
:0041B566 53                     push ebx
:0041B567 8BD9                   mov ebx, ecx
:0041B569 56                     push esi
:0041B56A C783B405000040B74100    mov dword ptr [ebx+000005B4], 0041B740   <--送下一个检查子程序的地址
:0041B574 8B8344050000           mov eax, dword ptr [ebx+00000544]
:0041B57A 8DB348050000           lea esi, dword ptr [ebx+00000548]
:0041B580 6A00                   push 00000000
:0041B582 8A08                   mov cl, byte ptr [eax]
:0041B584 40                     inc eax
:0041B585 51                     push ecx
:0041B586 8BCE                   mov ecx, esi
:0041B588 898344050000           mov dword ptr [ebx+00000544], eax
:0041B58E E8FD2F0000             call 0041E590   <--这个 call 比对第 9 个字符
:0041B593 85C0                   test eax, eax
:0041B595 751D                   jne 0041B5B4   <--不应该跳转
:0041B597 8B8344050000           mov eax, dword ptr [ebx+00000544]
:0041B59D 6A00                   push 00000000
:0041B59F 8A08                   mov cl, byte ptr [eax]
:0041B5A1 40                     inc eax
:0041B5A2 51                     push ecx
:0041B5A3 8BCE                   mov ecx, esi
:0041B5A5 898344050000           mov dword ptr [ebx+00000544], eax
:0041B5AB E8E02F0000             call 0041E590   <--这个 call 比对第 10 个字符
:0041B5B0 85C0                   test eax, eax
:0041B5B2 7414                   je 0041B5C8   <--应该跳转

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B595(C)
|

:0041B5B4 C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920   <--送注册失败处理子程序的地址

* Possible Reference to String Resource ID=00001: "萣
*

                           |

:0041B5BE C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001   <--把注册失败值送全局变量 reg

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041B5B2(C)
|

:0041B5C8 55                     push ebp
```
