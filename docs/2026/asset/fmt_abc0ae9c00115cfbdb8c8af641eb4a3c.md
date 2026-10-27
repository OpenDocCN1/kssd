
```text
005DC630    .  50     push eax
005DC631    .  51     push ecx
005DC632    .  FFD6   call esi                                   ;  组合
005DC634    .  50     push eax
005DC635    .  8D55 9>lea edx,dword ptr ss:[ebp-68]
005DC638    .  8D45 8>lea eax,dword ptr ss:[ebp-78]
005DC63B    .  52     push edx
005DC63C    .  50     push eax
005DC63D    .  FFD6   call esi
005DC63F    .  8D8D D>lea ecx,dword ptr ss:[ebp-128]             ;  组合
005DC645    .  50     push eax
005DC646    .  8D95 7>lea edx,dword ptr ss:[ebp-88]
005DC64C    .  51     push ecx
005DC64D    .  52     push edx
005DC64E    .  FFD6   call esi                                   ;  组合
005DC650    .  50     push eax
005DC651    .  8D85 5>lea eax,dword ptr ss:[ebp-A8]
005DC657    .  8D8D 4>lea ecx,dword ptr ss:[ebp-B8]
005DC65D    .  50     push eax
005DC65E    .  51     push ecx
005DC65F    .  FFD6   call esi                                   ;  组合完毕，5位-5位-4位
005DC661    .  50     push eax
005DC662    .  FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>;  进行比较了

……………………………………………………………………………………………………………………

005DC222    .  E8 197>call pkgs910.00673D40                  进入关键点了！！！！

……………………………………………………………………………………………………………………

00673D40    $  55     push ebp
00673D41    .  8BEC   mov ebp,esp
00673D43    .  83EC 0>sub esp,8
00673D46    .  68 A67>push <jmp.&MSVBVM60.__vbaExceptHandler>    ;  SE handler installation
00673D4B    .  64:A1 >mov eax,dword ptr fs:[0]
00673D51    .  50     push eax
00673D52    .  64:892>mov dword ptr fs:[0],esp
00673D59    .  81EC 8>sub esp,180
00673D5F    .  53     push ebx
00673D60    .  56     push esi
00673D61    .  57     push edi
00673D62    .  8965 F>mov dword ptr ss:[ebp-8],esp
00673D65    .  C745 F>mov dword ptr ss:[ebp-4],pkgs910.00405DF0
00673D6C    .  33C0   xor eax,eax
00673D6E    .  8945 E>mov dword ptr ss:[ebp-1C],eax
00673D71    .  8945 D>mov dword ptr ss:[ebp-2C],eax
00673D74    .  8945 C>mov dword ptr ss:[ebp-3C],eax
00673D77    .  8945 B>mov dword ptr ss:[ebp-4C],eax
00673D7A    .  8945 A>mov dword ptr ss:[ebp-5C],eax
00673D7D    .  8945 9>mov dword ptr ss:[ebp-6C],eax
00673D80    .  8945 8>mov dword ptr ss:[ebp-7C],eax
00673D83    .  8985 7>mov dword ptr ss:[ebp-8C],eax
00673D89    .  8985 6>mov dword ptr ss:[ebp-9C],eax
00673D8F    .  8985 5>mov dword ptr ss:[ebp-AC],eax
00673D95    .  8985 4>mov dword ptr ss:[ebp-BC],eax
00673D9B    .  8985 3>mov dword ptr ss:[ebp-CC],eax
00673DA1    .  8985 2>mov dword ptr ss:[ebp-DC],eax
00673DA7    .  8985 1>mov dword ptr ss:[ebp-EC],eax
00673DAD    .  8985 0>mov dword ptr ss:[ebp-FC],eax
00673DB3    .  8985 F>mov dword ptr ss:[ebp-10C],eax
00673DB9    .  8985 E>mov dword ptr ss:[ebp-11C],eax
00673DBF    .  8985 C>mov dword ptr ss:[ebp-13C],eax
00673DC5    .  8985 B>mov dword ptr ss:[ebp-14C],eax
00673DCB    .  8985 A>mov dword ptr ss:[ebp-15C],eax
00673DD1    .  8985 8>mov dword ptr ss:[ebp-17C],eax
00673DD7    .  A1 489>mov eax,dword ptr ds:[739448]
00673DDC    .  50     push eax                                   ; /Arg1 => 001544FC
00673DDD    .  FF15 9>call dword ptr ds:[<&MSVBVM60.#581>]       ; \rtcR8ValFromBstr
00673DE3    .  DC0D 5>fmul qword ptr ds:[401658]                 ;  机器码与3相称
00673DE9    .  8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673DEC    .  C745 D>mov dword ptr ss:[ebp-2C],5
00673DF3    .  51     push ecx
00673DF4    .  DC05 E>fadd qword ptr ds:[405DE8]                 ;  结果与74想加
00673DFA    .  DD5D D>fstp qword ptr ss:[ebp-24]                 ;  储存
00673DFD    .  DFE0   fstsw ax
00673DFF    .  A8 0D  test al,0D
00673E01    .  0F85 2>jnz pkgs910.00674531
00673E07    .  E8 B42>call pkgs910.006765C0                      ;  取前14位防御0012f434
00673E0C    .  8B35 4>mov esi,dword ptr ds:[<&MSVBVM60.__vbaStrM>;  MSVBVM60.__vbaStrMove
00673E12    .  8BD0   mov edx,eax
00673E14    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E17    .  FFD6   call esi                                   ;  <&MSVBVM60.__vbaStrMove>
00673E19    .  8B0D B>mov ecx,dword ptr ds:[7394B0]
00673E1F    .  8B3D C>mov edi,dword ptr ds:[<&MSVBVM60.__vbaStrC>;  MSVBVM60.__vbaStrCopy
00673E25    .  8BD0   mov edx,eax
00673E27    .  FFD7   call edi                                   ;  <&MSVBVM60.__vbaStrCopy>
00673E29    .  8B1D 9>mov ebx,dword ptr ds:[<&MSVBVM60.__vbaFree>;  MSVBVM60.__vbaFreeStr
00673E2F    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E32    .  FFD3   call ebx                                   ;  <&MSVBVM60.__vbaFreeStr>
00673E34    .  8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673E37    .  FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVar
00673E3D    .  8B15 4>mov edx,dword ptr ds:[739448]              ;  又得到机器码
00673E43    .  52     push edx                                   ; /Arg1 => 001544FC
00673E44    .  FF15 9>call dword ptr ds:[<&MSVBVM60.#581>]       ; \转化为实数
00673E4A    .  DC0D 0>fmul qword ptr ds:[401900]                 ;  乘以4
00673E50    .  C745 D>mov dword ptr ss:[ebp-2C],5
00673E57    .  DC25 E>fsub qword ptr ds:[405DE0]                 ;  减47
00673E5D    .  DD5D D>fstp qword ptr ss:[ebp-24]                 ;  保存
00673E60    .  DFE0   fstsw ax
00673E62    .  A8 0D  test al,0D
00673E64    .  0F85 C>jnz pkgs910.00674531
00673E6A    .  8D45 D>lea eax,dword ptr ss:[ebp-2C]
00673E6D    .  50     push eax
00673E6E    .  E8 4D2>call pkgs910.006765C0                      ;  取刚才算好的前14位
00673E73    .  8BD0   mov edx,eax
00673E75    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E78    .  FFD6   call esi
00673E7A    .  8B0D B>mov ecx,dword ptr ds:[7394B0]
00673E80    .  8BD0   mov edx,eax
00673E82    .  83C1 0>add ecx,4
00673E85    .  FFD7   call edi                                   ;  在内存中去掉2位了，前2位
00673E87    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E8A    .  FFD3   call ebx
00673E8C    .  8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673E8F    .  FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVar
00673E95    .  BE 010>mov esi,1
00673E9A    .  8975 E>mov dword ptr ss:[ebp-14],esi
00673E9D    .  8B1D 8>mov ebx,dword ptr ds:[<&MSVBVM60.__vbaMidS>;  MSVBVM60.__vbaMidStmtBstr
00673EA3    >  B8 020>mov eax,2
00673EA8    .  3BF0   cmp esi,eax
00673EAA    .  0F8F 9>jg pkgs910.00674144
00673EB0    .  4E     dec esi
00673EB1    .  3BF0   cmp esi,eax
00673EB3    .  72 06  jb short pkgs910.00673EBB
00673EB5    .  FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>;  MSVBVM60.__vbaGenerateBoundsError
00673EBB    >  8B15 B>mov edx,dword ptr ds:[7394B0]
00673EC1    .  8B04B2 mov eax,dword ptr ds:[edx+esi*4]           ;  第一次计算的值
00673EC4    .  50     push eax
00673EC5    .  FF15 3>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>;  测长度
00673ECB    .  8985 7>mov dword ptr ss:[ebp-18C],eax
00673ED1    .  BF 010>mov edi,1
00673ED6    >  3BBD 7>cmp edi,dword ptr ss:[ebp-18C]
00673EDC    .  0F8F 4>jg pkgs910.0067412A
00673EE2    .  83FE 0>cmp esi,2
00673EE5    .  C745 D>mov dword ptr ss:[ebp-24],1
00673EEC    .  C745 D>mov dword ptr ss:[ebp-2C],2
00673EF3    .  72 06  jb short pkgs910.00673EFB
00673EF5    .  FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>;  MSVBVM60.__vbaGenerateBoundsError
00673EFB    >  8B0D B>mov ecx,dword ptr ds:[7394B0]
00673F01    .  8D45 D>lea eax,dword ptr ss:[ebp-2C]
00673F04    .  50     push eax                                   ; /Arg4
00673F05    .  57     push edi                                   ; |Arg3
00673F06    .  8D14B1 lea edx,dword ptr ds:[ecx+esi*4]           ; |
00673F09    .  8D8D 2>lea ecx,dword ptr ss:[ebp-DC]              ; |
00673F0F    .  8995 2>mov dword ptr ss:[ebp-D4],edx              ; |
00673F15    .  8D55 C>lea edx,dword ptr ss:[ebp-3C]              ; |
00673F18    .  51     push ecx                                   ; |Arg2
00673F19    .  52     push edx                                   ; |Arg1
00673F1A    .  C785 2>mov dword ptr ss:[ebp-DC],4008             ; |
```
