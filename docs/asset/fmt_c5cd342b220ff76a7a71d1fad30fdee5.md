
# AnimateIt Screen Saver Toolkit (Ver 2.02)

> 原文：[`www.pediy.com/kssd/pediy06/pediy6304.htm`](https://www.pediy.com/kssd/pediy06/pediy6304.htm)

`AnimateIt Screen Saver Toolkit (Ver 2.02)` 是一款制作屏保的工具，它也能够提供声音图像的功能。更有一点就是，它能够让你自由地发布屏保或者以收费的方式发送！前提是你要注册！你可以在 `http://www.allersoft.com` 得到该软件的更多信息。

需要注意的是这个软件有两个版本，就是 `Standard Edition` 和 `Power Edition`，可能跟输入的注册码有关！

好了，打开软件，在 `Ordering Info` 下输入：

`Registered: dengkeng`
`Registration: 123456`

下断点

`bpx hmemcpy`

点击 `Register`，拦截下来

`bc *`

`pmodule`

看看模块名字不是 `Launcher`，而是 `Animateit`，这一点需要注意。搜索 `Animateit` 是屏保的后缀名 `.scr`，想必就是它了。打开它！我们看到了我们需要的字符 `"Thank you for registering for the Standard Edition of %s %s"`。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: `|:00414739(C)`
|

```
:0041476C 56                 push esi
:0041476D 8D4C2410           lea ecx, dword ptr [esp+10]
:00414771 E8DCF90500         call 00474152
:00414776 8B8F90000000       mov ecx, dword ptr [edi+00000090]
:0041477C C744241800000000   mov [esp+18], 00000000
:00414784 51                 push ecx
:00414785 51                 push ecx
:00414786 8BCC               mov ecx, esp
:00414788 89642410           mov dword ptr [esp+10], esp
:0041478C 56                 push esi
:0041478D E8C0F90500         call 00474152
:00414792 E85997FFFF         call 0040DEF0 ;关键 Call,跟进
:00414797 83C408             add esp, 00000008
:0041479A 83F801             cmp eax, 00000001
:0041479D 7542               jne 004147E1
:0041479F 8D4C2408           lea ecx, dword ptr [esp+08]
:004147A3 E89AF90500         call 00474142
:004147A8 68B4714B00         push 004B71B4
:004147AD 68B46F4B00         push 004B6FB4
:004147B2 8D542410           lea edx, dword ptr [esp+10]
```

\* Possible StringData Ref from Data Obj -> `"Thank you for registering for "` -> `"the Standard Edition of %s %s."` |

```
:004147B6 68F4D54900         push 0049D5F4
:004147BB 52                 push edx
:004147BC C644242801         mov [esp+28], 01
:004147C1 E81D9C0500         call 0046E3E3;出错提示
```

F8 跟进关键 Call

\* Referenced by a CALL at Addresses: `|:0040DE9C` , `:00414792`
|

```
:0040DEF0 6AFF               push FFFFFFFF
:0040DEF2 6890334800         push 00483390
:0040DEF7 64A100000000       mov eax, dword ptr fs:[00000000]
:0040DEFD 50                 push eax
:0040DEFE 64892500000000     mov dword ptr fs:[00000000], esp
:0040DF05 83EC08             sub esp, 00000008
:0040DF08 53                 push ebx
:0040DF09 56                 push esi
:0040DF0A 68B46F4B00         push 004B6FB4
:0040DF0F 8D4C240C           lea ecx, dword ptr [esp+0C]
:0040DF13 C744241C00000000   mov [esp+1C], 00000000
:0040DF1B E8DB630600         call 004742FB
:0040DF20 8B742424           mov esi, dword ptr [esp+24]
:0040DF24 C644241801         mov [esp+18], 01
:0040DF29 85F6               test esi, esi
:0040DF2B 0F8408010000       je 0040E039
```

\* Possible Reference to String Resource ID=00001: `"AnimateIt"` |

```
:0040DF31 6A01               push 00000001
:0040DF33 8D44240C           lea eax, dword ptr [esp+0C]
```

\* Possible StringData Ref from Data Obj -> `"STANDARD"` |

```
:0040DF37 68F0D14900         push 0049D1F0
:0040DF3C 8D4C242C           lea ecx, dword ptr [esp+2C]
:0040DF40 50                 push eax
:0040DF41 51                 push ecx
:0040DF42 E89D650600         call 004744E4
:0040DF47 51                 push ecx
:0040DF48 8D4C2428           lea ecx, dword ptr [esp+28]
:0040DF4C 8BD4               mov edx, esp
:0040DF4E 89642414           mov dword ptr [esp+14], esp
:0040DF52 51                 push ecx
:0040DF53 50                 push eax
:0040DF54 52                 push edx
:0040DF55 C644242C02         mov [esp+2C], 02
:0040DF5A E81F650600         call 0047447E
:0040DF5F E81CD8FFFF         call 0040B780 ;关键 Call,GoOn.......
:0040DF64 83C408             add esp, 00000008
:0040DF67 33DB               xor ebx, ebx
:0040DF69 3BC6               cmp eax, esi;? esi 假注册码,? eax 真的
:0040DF6B 8D4C2424           lea ecx, dword ptr [esp+24]
:0040DF6F 0F94C3             sete bl
:0040DF72 C644241801         mov [esp+18], 01
:0040DF77 E811630600         call 0047428D
:0040DF7C 84DB               test bl, bl
:0040DF7E 7435               je 0040DFB5
:0040DF80 8D4C2408           lea ecx, dword ptr [esp+08]
:0040DF84 C644241800         mov [esp+18], 00
:0040DF89 E8FF620600         call 0047428D
:0040DF8E 8D4C2420           lea ecx, dword ptr [esp+20]
:0040DF92 C7442418FFFFFFFF   mov [esp+18], FFFFFFFF
:0040DF9A E8EE620600         call 0047428D
```

\* Possible Reference to String Resource ID=00001: `"AnimateIt"` |

```
:0040DF9F B801000000         mov eax, 00000001
:0040DFA4 8B4C2410           mov ecx, dword ptr [esp+10]
:0040DFA8 64890D00000000     mov dword ptr fs:[00000000], ecx
:0040DFAF 5E                 pop esi
:0040DFB0 5B                 pop ebx
:0040DFB1 83C414             add esp, 00000014
:0040DFB4 C3                 ret
```

要写注册机跟进 `0040DF5F` 处的 `Call`...

\* Referenced by a CALL at Addresses: `|:0040DB1F` , `:0040DF5F` , `:0040DFE3`
|

```
:0040B780 64A100000000       mov eax, dword ptr fs:[00000000]
:0040B786 6AFF               push FFFFFFFF
:0040B788 6858304800         push 00483058
:0040B78D 50                 push eax
:0040B78E 64892500000000     mov dword ptr fs:[00000000], esp
:0040B795 56                 push esi
:0040B796 33F6               xor esi, esi
:0040B798 8D4C2414           lea ecx, dword ptr [esp+14]
:0040B79C 8974240C           mov dword ptr [esp+0C], esi
:0040B7A0 E89A2C0600         call 0046E43F;三个字符串连起来"AnimateItSTANDARDdengkeng"
:0040B7A5 8D4C2414           lea ecx, dword ptr [esp+14]
:0040B7A9 E8482C0600         call 0046E3F6
:0040B7AE 8B442414           mov eax, dword ptr [esp+14]
:0040B7B2 3970F8             cmp dword ptr [eax-08], esi
:0040B7B5 7523               jne 0040B7DA;关键计算注册码,继续....
:0040B7B7 8D4C2414           lea ecx, dword ptr [esp+14]
:0040B7BB C744240CFFFFFFFF   mov [esp+0C], FFFFFFFF
:0040B7C3 E8C58A0600         call 0047428D
:0040B7C8 33C0               xor eax, eax
:0040B7CA 8B4C2404           mov ecx, dword ptr [esp+04]
:0040B7CE 64890D00000000     mov dword ptr fs:[00000000], ecx
:0040B7D5 5E                 pop esi
:0040B7D6 83C40C             add esp, 0000000C
:0040B7D9 C3                 ret
```

要得到注册码就要到 `0040B7B5` 完成 `jne` 指令，`GoOn`...

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: `|:0040B7B5(C)`
|

```
:0040B7DA 57                 push edi
:0040B7DB 8D4C2418           lea ecx, dword ptr [esp+18]
:0040B7DF E8538F0600         call 00474737;把连接起来的字符,全转换成大写
:0040B7E4 8B7C2418           mov edi, dword ptr [esp+18]
:0040B7E8 33C9               xor ecx, ecx
:0040B7EA 8B57F8             mov edx, dword ptr [edi-08]
```
