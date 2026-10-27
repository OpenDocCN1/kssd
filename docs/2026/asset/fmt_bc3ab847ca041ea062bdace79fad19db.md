
```text
004A088D    .  8D85 1>lea eax,dword ptr ss:[ebp-F0]
004A0893    .  56     push esi
004A0894    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]
004A0897    .  50     push eax
004A0898    .  51     push ecx
004A0899    .  C745 C>mov dword ptr ss:[ebp-38],1
004A08A0    .  C745 C>mov dword ptr ss:[ebp-40],2
004A08A7    .  899D 1>mov dword ptr ss:[ebp-F0],ebx
004A08AD    .  FFD7   call edi
004A08AF    .  8D55 B>lea edx,dword ptr ss:[ebp-50]
004A08B2    .  8D45 D>lea eax,dword ptr ss:[ebp-24]
004A08B5    .  52     push edx
004A08B6    .  50     push eax
004A08B7    .  FF15 9>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarVal
004A08BD    .  50     push eax                      ; /Arg1
004A08BE    .  FF15 5>call dword ptr ds:[<&MSVBVM60>; \rtcAnsiValueBstr
004A08C4    .  66:05 >add ax,20                     ;  地一个字母
004A08C8    .  8D55 A>lea edx,dword ptr ss:[ebp-60]
004A08CB    .  0F80 A>jo pkgs910.004A1474
004A08D1    .  0FBFC8 movsx ecx,ax
004A08D4    .  51     push ecx                      ; /Arg2
004A08D5    .  52     push edx                      ; |Arg1
004A08D6    .  FF15 8>call dword ptr ds:[<&MSVBVM60>; \rtcVarBstrFromAnsi
004A08DC    .  8D45 E>lea eax,dword ptr ss:[ebp-1C]
004A08DF    .  8D4D A>lea ecx,dword ptr ss:[ebp-60]
004A08E2    .  50     push eax
004A08E3    .  56     push esi
004A08E4    .  6A 01  push 1
004A08E6    .  51     push ecx
004A08E7    .  FF15 3>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarMove
004A08ED    .  8BD0   mov edx,eax
004A08EF    .  8D4D D>lea ecx,dword ptr ss:[ebp-28]
004A08F2    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrMove
004A08F8    .  50     push eax                      ; |Arg2
004A08F9    .  6A 00  push 0                        ; |Arg1 = 00000000
004A08FB    .  FF15 8>call dword ptr ds:[<&MSVBVM60>; \\_\_vbaMidStmtBstr
004A0901    .  8D55 D>lea edx,dword ptr ss:[ebp-28]
004A0904    .  8D45 D>lea eax,dword ptr ss:[ebp-24]
004A0907    .  52     push edx
004A0908    .  50     push eax
004A0909    .  6A 02  push 2
004A090B    .  FF15 D>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeStrList
004A0911    .  8D4D A>lea ecx,dword ptr ss:[ebp-60]
004A0914    .  8D55 B>lea edx,dword ptr ss:[ebp-50]
004A0917    .  51     push ecx
004A0918    .  8D45 C>lea eax,dword ptr ss:[ebp-40]
004A091B    .  52     push edx
004A091C    .  50     push eax
004A091D    .  6A 03  push 3
004A091F    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList
004A0925    .  83C4 1>add esp,1C
004A0928    >  B8 010>mov eax,1
004A092D    .  03C6   add eax,esi
004A092F    .  0F80 3>jo pkgs910.004A1474
004A0935    .  8BF0   mov esi,eax
004A0937    .^ E9 BDF>jmp pkgs910.004A07F9          ;  上面一段是判断你输入的是否为小写字母，如果不是转换成小写放于001546F4
004A093C    >  8D95 1>lea edx,dword ptr ss:[ebp-F0]
004A0942    .  6A 06  push 6                        ; /Arg3 = 00000006
004A0944    .  8D45 C>lea eax,dword ptr ss:[ebp-40] ; |
004A0947    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C] ; |
004A094A    .  52     push edx                      ; |Arg2
004A094B    .  50     push eax                      ; |Arg1
004A094C    .  898D 1>mov dword ptr ss:[ebp-E8],ecx ; |
004A0952    .  899D 1>mov dword ptr ss:[ebp-F0],ebx ; |
004A0958    .  FF15 3>call dword ptr ds:[<&MSVBVM60>; \取前六位放于0016E9BC
004A095E    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
004A0961    .  8D55 A>lea edx,dword ptr ss:[ebp-60]
004A0964    .  BE 080>mov esi,8
004A0969    .  898D F>mov dword ptr ss:[ebp-108],ec>
004A096F    .  52     push edx
004A0970    .  8D85 F>lea eax,dword ptr ss:[ebp-110>
004A0976    .  56     push esi
004A0977    .  8D4D 9>lea ecx,dword ptr ss:[ebp-70]
004A097A    .  50     push eax
004A097B    .  51     push ecx
004A097C    .  C785 0>mov dword ptr ss:[ebp-F8],pkg>
004A0986    .  89B5 0>mov dword ptr ss:[ebp-100],es>
004A098C    .  C745 A>mov dword ptr ss:[ebp-58],6   ;  要6位
004A0993    .  C745 A>mov dword ptr ss:[ebp-60],2   ;  两位两位的取
004A099A    .  899D F>mov dword ptr ss:[ebp-110],eb>
004A09A0    .  FFD7   call edi                      ;  取中间六位
004A09A2    .  8D85 C>lea eax,dword ptr ss:[ebp-140>
004A09A8    .  6A 06  push 6                        ; /Arg3 = 00000006
004A09AA    .  8D8D 6>lea ecx,dword ptr ss:[ebp-A0] ; |
004A09B0    .  8D55 E>lea edx,dword ptr ss:[ebp-1C] ; |
004A09B3    .  50     push eax                      ; |Arg2
004A09B4    .  51     push ecx                      ; |Arg1
004A09B5    .  C785 D>mov dword ptr ss:[ebp-128],pk>; |
004A09BF    .  89B5 D>mov dword ptr ss:[ebp-130],es>; |
004A09C5    .  8995 C>mov dword ptr ss:[ebp-138],ed>; |
004A09CB    .  899D C>mov dword ptr ss:[ebp-140],eb>; |
004A09D1    .  FF15 5>call dword ptr ds:[<&MSVBVM60>; \取后六位防于00154A4C
004A09D7    .  8B35 9>mov esi,dword ptr ds:[<&MSVBV>;  MSVBVM60.\_\_vbaVarCat
004A09DD    .  8D55 C>lea edx,dword ptr ss:[ebp-40]
004A09E0    .  8D85 0>lea eax,dword ptr ss:[ebp-100>
004A09E6    .  52     push edx
004A09E7    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]
004A09EA    .  50     push eax
004A09EB    .  51     push ecx
004A09EC    .  FFD6   call esi                      ;  在前6位后面加一条-; <&MSVBVM60.\_\_vbaVarCat>
004A09EE    .  50     push eax
004A09EF    .  8D55 9>lea edx,dword ptr ss:[ebp-70]
004A09F2    .  8D45 8>lea eax,dword ptr ss:[ebp-80]
004A09F5    .  52     push edx
004A09F6    .  50     push eax
004A09F7    .  FFD6   call esi                      ;  把中间6位又加入其中防于0016420C
004A09F9    .  8D8D D>lea ecx,dword ptr ss:[ebp-130>
004A09FF    .  50     push eax
004A0A00    .  8D95 7>lea edx,dword ptr ss:[ebp-90]
004A0A06    .  51     push ecx
004A0A07    .  52     push edx
004A0A08    .  FFD6   call esi                      ;  再加一条“-”
004A0A0A    .  50     push eax
004A0A0B    .  8D85 6>lea eax,dword ptr ss:[ebp-A0]
004A0A11    .  8D8D 5>lea ecx,dword ptr ss:[ebp-B0]
004A0A17    .  50     push eax
004A0A18    .  51     push ecx
004A0A19    .  FFD6   call esi                      ;  把后6位又加入其中防于00154A04
004A0A1B    .  50     push eax
004A0A1C    .  FF15 3>call dword ptr ds:[<&MSVBVM60>;  拷贝到EAX中
004A0A22    .  8BD0   mov edx,eax
004A0A24    .  B9 549>mov ecx,pkgs910.00739354
004A0A29    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrMove
004A0A2F    .  8D95 5>lea edx,dword ptr ss:[ebp-B0]
004A0A35    .  8D85 6>lea eax,dword ptr ss:[ebp-A0]
004A0A3B    .  52     push edx
004A0A3C    .  8D8D 7>lea ecx,dword ptr ss:[ebp-90]
004A0A42    .  50     push eax
004A0A43    .  8D55 8>lea edx,dword ptr ss:[ebp-80]
004A0A46    .  51     push ecx
004A0A47    .  8D45 9>lea eax,dword ptr ss:[ebp-70]
004A0A4A    .  52     push edx
004A0A4B    .  8D4D B>lea ecx,dword ptr ss:[ebp-50] ;  上面都是8
004A0A4E    .  50     push eax
004A0A4F    .  8D55 A>lea edx,dword ptr ss:[ebp-60]
004A0A52    .  51     push ecx
004A0A53    .  52     push edx
004A0A54    .  8D45 C>lea eax,dword ptr ss:[ebp-40]
004A0A57    .  50     push eax
004A0A58    .  6A 08  push 8
004A0A5A    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList
004A0A60    .  83C4 2>add esp,24
004A0A63    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]
004A0A66    .  8D55 C>lea edx,dword ptr ss:[ebp-40]
004A0A69    .  898D 1>mov dword ptr ss:[ebp-E8],ecx
004A0A6F    .  52     push edx
004A0A70    .  8D85 1>lea eax,dword ptr ss:[ebp-F0]
004A0A76    .  6A 02  push 2
004A0A78    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]
004A0A7B    .  50     push eax
004A0A7C    .  51     push ecx
004A0A7D    .  C745 C>mov dword ptr ss:[ebp-38],5
004A0A84    .  C745 C>mov dword ptr ss:[ebp-40],2
004A0A8B    .  899D 1>mov dword ptr ss:[ebp-F0],ebx
004A0A91    .  FFD7   call edi                      ;  从第一组6个数据中，取后5位数放与EBP-48
004A0A93    .  8D55 E>lea edx,dword ptr ss:[ebp-1C]
004A0A96    .  8D45 9>lea eax,dword ptr ss:[ebp-70]
004A0A99    .  8995 E>mov dword ptr ss:[ebp-118],ed>
004A0A9F    .  50     push eax
004A0AA0    .  8D8D E>lea ecx,dword ptr ss:[ebp-120>
```
