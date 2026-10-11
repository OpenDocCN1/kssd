
```
004A0EFD    .  68 64D>push pkgs910.0042D164         ;  UNICODE "ini"  

004A0F02    .  68 78D>push pkgs910.0042D178  

004A0F07    .  FFD6   call esi  

004A0F09    .  83EC 1>sub esp,10  

004A0F0C    .  B9 0A0>mov ecx,0A                   ; |  

004A0F11    .  8BD4   mov edx,esp                  ; |  

004A0F13    .  898D 1>mov dword ptr ss:\[ebp-F0\],ecx ; |  

004A0F19    .  B8 040>mov eax,80020004             ; |  

004A0F1E    .  68 80D>push pkgs910.0042D180        ; |Arg3 = 0042D180  

004A0F23    .  890A   mov dword ptr ds:\[edx\],ecx  ; |  

004A0F25    .  8B8D 1>mov ecx,dword ptr ss:\[ebp-EC\] ; |  

004A0F2B    .  8985 1>mov dword ptr ss:\[ebp-E8\],eax ; |  

004A0F31    .  68 64D>push pkgs910.0042D164        ; |Arg2 = 0042D164  

004A0F36    .  894A 0>mov dword ptr ds:\[edx+4\],ecx  ; |  

004A0F39    .  68 78D>push pkgs910.0042D178        ; |Arg1 = 0042D178  

004A0F3E    .  8942 0>mov dword ptr ds:\[edx+8\],eax  ; |  

004A0F41    .  8B85 1>mov eax,dword ptr ss:\[ebp-E4\] ; |  

004A0F47    .  8942 0>mov dword ptr ds:\[edx+C\],eax  ; |  

004A0F4A    .  FF15 0>call dword ptr ds:\[<&MSVBVM60>;  \rtcGetSetting  

004A0F50    .  8BD0   mov edx,eax  

004A0F52    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]  

004A0F55    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaStrMove  

004A0F5B    .  8B4D E>mov ecx,dword ptr ss:\[ebp-1C\]  

004A0F5E    .  8B35 1>mov esi,dword ptr ds:\[<&MSVBV>;  MSVBVM60.__vbaStrCmp  

004A0F64    .  51     push ecx  

004A0F65    .  68 24D>push pkgs910.0042D424        ;  UNICODE "X6261"  

004A0F6A    .  FFD6   call esi                   ;  <&MSVBVM60.__vbaStrCmp>  

004A0F6C    .  8BD0   mov edx,eax  

004A0F6E    .  8B45 E>mov eax,dword ptr ss:\[ebp-1C\]  

004A0F71    .  F7DA   neg edx  

004A0F73    .  1BD2   sbb edx,edx  

004A0F75    .  50     push eax  

004A0F76    .  F7DA   neg edx  

004A0F78    .  68 88D>push pkgs910.0042D188        ;  UNICODE "bg3721"  

004A0F7D    .  8995 6>mov dword ptr ss:\[ebp-198\],ed>  

004A0F83    .  FFD6   call esi  

004A0F85    .  8B8D 6>mov ecx,dword ptr ss:\[ebp-198>  

004A0F8B    .  F7D8   neg eax  

004A0F8D    .  1BC0   sbb eax,eax  

004A0F8F    .  F7D8   neg eax  

004A0F91    .  85C8   test eax,ecx  

004A0F93    .  75 1A  jnz short pkgs910.004A0FAF  

004A0F95    .  68 54C>push pkgs910.0042C754        ; /Arg4 = 0042C754  

004A0F9A    .  68 80D>push pkgs910.0042D180        ; |Arg3 = 0042D180  

004A0F9F    .  68 64D>push pkgs910.0042D164        ; |Arg2 = 0042D164  

004A0FA4    .  68 78D>push pkgs910.0042D178        ; |Arg1 = 0042D178  

004A0FA9    .  FF15 0>call dword ptr ds:\[<&MSVBVM60>;  \rtcSaveSetting  

004A0FAF    >  A1 9C9>mov eax,dword ptr ds:\[73949C\]  

004A0FB4    >  83F8 0>cmp eax,1  

004A0FB7    .  75 0A  jnz short pkgs910.004A0FC3  

004A0FB9    .  BA 34D>mov edx,pkgs910.0042D434  

004A0FBE    .  E9 F50>jmp pkgs910.004A10B8  

004A0FC3    >  83F8 0>cmp eax,2  

004A0FC6    .  0F85 3>jnz pkgs910.004A1107          ;  二处  

004A0FCC    .  8945 C>mov dword ptr ss:\[ebp-40\],eax  

004A0FCF    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\]  

004A0FD2    .  50     push eax  

004A0FD3    .  8D8D 1>lea ecx,dword ptr ss:\[ebp-F0\]  

004A0FD9    .  6A 08  push 8  

004A0FDB    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]  

004A0FDE    .  51     push ecx  

004A0FDF    .  52     push edx  

004A0FE0    .  C745 C>mov dword ptr ss:\[ebp-38\],1  

004A0FE7    .  C785 1>mov dword ptr ss:\[ebp-E8\],pkg>  

004A0FF1    .  899D 1>mov dword ptr ss:\[ebp-F0\],ebx  

004A0FF7    .  FFD7   call edi  

004A0FF9    .  8D45 9>lea eax,dword ptr ss:\[ebp-70\]  

004A0FFC    .  8D8D E>lea ecx,dword ptr ss:\[ebp-120>  

004A1002    .  50     push eax  

004A1003    .  6A 08  push 8  

004A1005    .  8D55 8>lea edx,dword ptr ss:\[ebp-80\]  

004A1008    .  BE 088>mov esi,8008  

004A100D    .  51     push ecx  

004A100E    .  52     push edx  

004A100F    .  C785 F>mov dword ptr ss:\[ebp-108\],pk>  

004A1019    .  89B5 F>mov dword ptr ss:\[ebp-110\],es>  

004A101F    .  C745 9>mov dword ptr ss:\[ebp-68\],1  

004A1026    .  C745 9>mov dword ptr ss:\[ebp-70\],2  

004A102D    .  C785 E>mov dword ptr ss:\[ebp-118\],pk>  

004A1037    .  899D E>mov dword ptr ss:\[ebp-120\],eb>  

004A103D    .  FFD7   call edi  

004A103F    .  8D45 B>lea eax,dword ptr ss:\[ebp-50\]  

004A1042    .  8D8D F>lea ecx,dword ptr ss:\[ebp-110>  

004A1048    .  50     push eax  

004A1049    .  8D55 A>lea edx,dword ptr ss:\[ebp-60\]  

004A104C    .  89B5 C>mov dword ptr ss:\[ebp-140\],es>  

004A1052    .  8B35 5>mov esi,dword ptr ds:\[<&MSVBV>;  MSVBVM60.__vbaVarCmpNe  

004A1058    .  51     push ecx  

004A1059    .  52     push edx  

004A105A    .  C785 C>mov dword ptr ss:\[ebp-138\],pk>  

004A1064    .  FFD6   call esi                   ;  <&MSVBVM60.__vbaVarCmpNe>  

004A1066    .  50     push eax  

004A1067    .  8D45 8>lea eax,dword ptr ss:\[ebp-80\]  

004A106A    .  8D8D C>lea ecx,dword ptr ss:\[ebp-140>  

004A1070    .  50     push eax  

004A1071    .  8D95 7>lea edx,dword ptr ss:\[ebp-90\]  

004A1077    .  51     push ecx  

004A1078    .  52     push edx  

004A1079    .  FFD6   call esi  

004A107B    .  50     push eax  

004A107C    .  8D85 6>lea eax,dword ptr ss:\[ebp-A0\]  

004A1082    .  50     push eax  

004A1083    .  FF15 5>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaVarAnd  

004A1089    .  50     push eax  

004A108A    .  FF15 E>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaBoolVarNull  

004A1090    .  8D4D 8>lea ecx,dword ptr ss:\[ebp-80\]  

004A1093    .  66:8BF>mov si,ax  

004A1096    .  8D55 9>lea edx,dword ptr ss:\[ebp-70\]  

004A1099    .  51     push ecx  

004A109A    .  8D45 B>lea eax,dword ptr ss:\[ebp-50\]  

004A109D    .  52     push edx  

004A109E    .  8D4D C>lea ecx,dword ptr ss:\[ebp-40\]  

004A10A1    .  50     push eax  

004A10A2    .  51     push ecx  

004A10A3    .  6A 04  push 4  

004A10A5    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaFreeVarList  

004A10AB    .  83C4 1>add esp,14  

004A10AE    .  66:85F>test si,si  

004A10B1    .  74 2A  je short pkgs910.004A10DD  

004A10B3    .  BA 60D>mov edx,pkgs910.0042D460  

004A10B8    >  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A10BB    .  FF15 C>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaStrCopy  

004A10C1    .  8D55 D>lea edx,dword ptr ss:\[ebp-24\]  

004A10C4    .  52     push edx  

004A10C5    .  E8 B63>call pkgs910.00674780  

004A10CA    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A10CD    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaFreeStr  

004A10D3    .  8B1D 6>mov ebx,dword ptr ds:\[<&MSVBV>;  MSVBVM60.__vbaHresultCheckObj  

004A10D9    .  33FF   xor edi,edi  

004A10DB    .  EB 52  jmp short pkgs910.004A112F  

004A10DD    >  BA BCD>mov edx,pkgs910.0042D4BC  

004A10E2    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A10E5    .  FF15 C>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaStrCopy  

004A10EB    .  8D45 D>lea eax,dword ptr ss:\[ebp-24\]  

004A10EE    .  50     push eax  

004A10EF    .  E8 8C3>call pkgs910.00674780  

004A10F4    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A10F7    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaFreeStr  

004A10FD    .  8B1D 6>mov ebx,dword ptr ds:\[<&MSVBV>;  MSVBVM60.__vbaHresultCheckObj  

004A1103    .  33FF   xor edi,edi  

004A1105    .  EB 28  jmp short pkgs910.004A112F  

004A1107    >  8B1D 6>mov ebx,dword ptr ds:\[<&MSVBV>;  MSVBVM60.__vbaHresultCheckObj  

004A110D    .  33FF   xor edi,edi  

004A110F    >  BA ECD>mov edx,pkgs910.0042D4EC  

004A1114    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A1117    .  FF15 C>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.__vbaStrCopy  

004A111D    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  

004A1120    .  51     push ecx  

004A1121    .  E8 2A3>call pkgs910.00674850  

004A1126    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]  
```
