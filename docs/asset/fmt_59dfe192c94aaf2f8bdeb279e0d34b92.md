
> :1000528D EB90                     `jmp` 1000521F

> \----------------------------------------------------------------------------

> 作者也太小心眼了，又要玩 `Game`!一不小心就 `game over`!^\_^

> \----------------------------------------------------------------------------

> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:

> |:10004CE0(C)---->返回看一下！★

> |

> :1000528F A1286A0810                `mov eax, dword ptr [10086A28]`
> :10005294 8D956CFFFFFF             `lea edx, dword ptr [ebp+FFFFFF6C]`
> :1000529A 68B0D10010               `push 1000D1B0`
> :1000529F 52                       `push edx`
> :100052A0 FF5008                   `call [eax+08]`
> :100052A3 8D8D6CFFFFFF             `lea ecx, dword ptr [ebp+FFFFFF6C]`
> :100052A9 68D8360710               `push 100736D8`
> :100052AE 8B55FC                   `mov edx, dword ptr [ebp-04]`
> :100052B1 52                       `push edx`
> :100052B2 68AE830310               `push 100383AE`

>
> \* Possible Reference to String Resource ID=00003: "Selected Pages"
>
>                               |               ^^^^^^^^^^^^^^^^------->搜索页面

> :100052B7 6A03                     `push 00000003` 到这里才是正常生成 `html`,所以向上看。
> :100052B9 51                       `push ecx`
> :100052BA E845390300               `call 10038C04`
> :100052BF 8B7D10                   `mov edi, dword ptr [ebp+10]`
> :100052C2 83C41C                   `add esp, 0000001C`
> :100052C5 85C0                     `test eax, eax`
> :100052C7 0F8584000000             `jne 10005351`
> :100052CD 8B750C                   `mov esi, dword ptr [ebp+0C]`
> :100052D0 83BE4411000004           `cmp dword ptr [esi+00001144], 00000004`
> :100052D7 742D                     `je 10005306`
> :100052D9 8B45EC                   `mov eax, dword ptr [ebp-14]`
> :100052DC 8B4D08                   `mov ecx, dword ptr [ebp+08]`
> :100052DF 56                       `push esi`
> :100052E0 50                       `push eax`
> :100052E1 51                       `push ecx`
> :100052E2 E8D9420000               `call 100095C0`
> :100052E7 8945DC                   `mov dword ptr [ebp-24], eax`
> :100052EA A1286A0810               `mov eax, dword ptr [10086A28]`
> :100052EF 83C40C                   `add esp, 0000000C`
> :100052F2 FF500C                   `call [eax+0C]`
> :100052F5 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :100052F8 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :100052FB 83C002                   `add eax, 00000002`
> :100052FE 8945CC                   `mov dword ptr [ebp-34], eax`
> :10005301 E919FFFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:100052D7(C)
>
> |

> :10005306 8D4DD4                   `lea ecx, dword ptr [ebp-2C]`
> :10005309 E8B2D5FFFF               `call 100028C0`
> :1000530E 8B4D08                   `mov ecx, dword ptr [ebp+08]`
> :10005311 50                       `push eax`
> :10005312 8B45EC                   `mov eax, dword ptr [ebp-14]`
> :10005315 8D9524F6FFFF             `lea edx, dword ptr [ebp+FFFFF624]`
> :1000531B 52                       `push edx`
> :1000531C 8B1518690810             `mov edx, dword ptr [10086918]`
> :10005322 50                       `push eax`
> :10005323 51                       `push ecx`
> :10005324 FF5210                   `call [edx+10]`
> :10005327 83C410                   `add esp, 00000010`
> :1000532A 8945DC                   `mov dword ptr [ebp-24], eax`
> :1000532D 85C0                     `test eax, eax`
> :1000532F 7407                     `je 10005338`

>
> \* Possible Reference to String Resource ID=00013: "BCL Magellan Online Guide"
>
>                               |

> :10005331 C745DC0D000000           `mov [ebp-24], 0000000D`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000532F(C)
>
> |

> :10005338 A1286A0810               `mov eax, dword ptr [10086A28]`
> :1000533D FF500C                   `call [eax+0C]`
> :10005340 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :10005343 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :10005346 83C002                   `add eax, 00000002`
> :10005349 8945CC                   `mov dword ptr [ebp-34], eax`
> :1000534C E9CEFEFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:100052C7(C)
>
> |

> :10005351 8B0D286A0810             `mov ecx, dword ptr [10086A28]`
> :10005357 FF510C                   `call [ecx+0C]`
> :1000535A 8B750C                   `mov esi, dword ptr [ebp+0C]`

>
> \* Possible Reference to String Resource ID=00013: "BCL Magellan Online Guide"
>
>                               |

> :1000535D C745DC0D000000           `mov [ebp-24], 0000000D`
> :10005364 85FF                     `test edi, edi`
> :10005366 8B965C110000             `mov edx, dword ptr [esi+0000115C]`
> :1000536C 8955EC                   `mov dword ptr [ebp-14], edx`
> :1000536F 7407                     `je 10005378`
> :10005371 8B07                     `mov eax, dword ptr [edi]`
> :10005373 8BCF                     `mov ecx, edi`
> :10005375 FF504C                   `call [eax+4C]`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000536F(C)
>
> |

> :10005378 8B45CC                   `mov eax, dword ptr [ebp-34]`
> :1000537B 8B5DC4                   `mov ebx, dword ptr [ebp-3C]`
> :1000537E 83C002                   `add eax, 00000002`
> :10005381 8945CC                   `mov dword ptr [ebp-34], eax`
> :10005384 E996FEFFFF               `jmp 1000521F`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:10005224(C)
>
> |

> :10005389 85FF                     `test edi, edi`
> :1000538B 7439                     `je 100053C6`
> :1000538D 8B07                     `mov eax, dword ptr [edi]`
> :1000538F 8BCF                     `mov ecx, edi`
> :10005391 8945E0                   `mov dword ptr [ebp-20], eax`
> :10005394 FF5044                   `call [eax+44]`
> :10005397 8B55E0                   `mov edx, dword ptr [ebp-20]`
> :1000539A 48                       `dec eax`
> :1000539B 50                       `push eax`
> :1000539C 8BCF                     `mov ecx, edi`
> :1000539E FF5240                   `call [edx+40]`
> :100053A1 51                       `push ecx`
> :100053A2 8B4DDC                   `mov ecx, dword ptr [ebp-24]`
> :100053A5 8BC4                     `mov eax, esp`
> :100053A7 8965B0                   `mov dword ptr [ebp-50], esp`
> :100053AA 51                       `push ecx`
> :100053AB 50                       `push eax`
> :100053AC E8FF0E0000               `call 100062B0`
> :100053B1 8B865C110000             `mov eax, dword ptr [esi+0000115C]`
> :100053B7 8B4DEC                   `mov ecx, dword ptr [ebp-14]`
> :100053BA 8B17                     `mov edx, dword ptr [edi]`
> :100053BC 83C408                   `add esp, 00000008`
> :100053BF 50                       `push eax`
> :100053C0 51                       `push ecx`
> :100053C1 8BCF                     `mov ecx, edi`
> :100053C3 FF5248                   `call [edx+48]`

>
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
>
> |:1000538B(C)
>
> |

> :100053C6 8B451C                   `mov eax, dword ptr [ebp+1C]`
> :100053C9 85C0                     `test eax, eax`
> :100053CB 743D                     `je 1000540A`
> :100053CD 8A45EB                   `mov al, byte ptr [ebp-15]`
> :100053D0 84C0                     `test al, al`
> :100053D2 751C                     `jne 100053F0`
