
# 排课高手 9.10 算法详细分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-104.htm`](https://www.pediy.com/kssd/pediy07/pediy7-104.htm)

- **破解作者**：`jsliyangsj`
- **作者邮箱**：`sjcrack@yahoo.com.cn`
- **使用工具**：`peid`、`OllyDbg1.10`
- **破解平台**：`Winxp`
- **软件名称**：`排课高手 9.10`
- **软件地址**：`http://www2.skycn.com/soft/5049.html`
- **软件语言**：`VB`

分析 VB 的东东只有一个体会：烦！

此软件尤其的烦，冗长，算法并不是很难，却用那么多代码！！！！！

越来越觉得，分析软件的第一要素：耐心。

输入注册码后来到下面的这一块，发现并没有计算注册码过程，只是一个冗长的比较过程，

靠！！！！（忍不住了）一个比较用这么多代码，

还发现，其实注册码已经在很早很早以前就算好了，（在弹出窗口前）估计是不是全部变量？

```
004A0660    > \55     push ebp
004A0661    .  8BEC   mov ebp,esp
004A0663    .  83EC 0>sub esp,0C
004A0666    .  68 A67>push <jmp.&MSVBVM60.\_\_vbaExce>;  SE handler installation
004A066B    .  64:A1 >mov eax,dword ptr fs:\[0\]
004A0671    .  50     push eax
004A0672    .  64:892>mov dword ptr fs:\[0\],esp
004A0679    .  81EC 8>sub esp,184
004A067F    .  53     push ebx
004A0680    .  56     push esi
004A0681    .  57     push edi
004A0682    .  8965 F>mov dword ptr ss:\[ebp-C\],esp
004A0685    .  C745 F>mov dword ptr ss:\[ebp-8\],pkgs>
004A068C    .  33FF   xor edi,edi
004A068E    .  897D F>mov dword ptr ss:\[ebp-4\],edi
004A0691    .  8B75 0>mov esi,dword ptr ss:\[ebp+8\]
004A0694    .  56     push esi
004A0695    .  8B06   mov eax,dword ptr ds:\[esi\]
004A0697    .  FF50 0>call dword ptr ds:\[eax+4\]
004A069A    .  897D E>mov dword ptr ss:\[ebp-1C\],edi
004A069D    .  897D D>mov dword ptr ss:\[ebp-24\],edi
004A06A0    .  897D D>mov dword ptr ss:\[ebp-28\],edi
004A06A3    .  897D D>mov dword ptr ss:\[ebp-2C\],edi
004A06A6    .  897D D>mov dword ptr ss:\[ebp-30\],edi
004A06A9    .  897D C>mov dword ptr ss:\[ebp-40\],edi
004A06AC    .  897D B>mov dword ptr ss:\[ebp-50\],edi
004A06AF    .  897D A>mov dword ptr ss:\[ebp-60\],edi
004A06B2    .  897D 9>mov dword ptr ss:\[ebp-70\],edi
004A06B5    .  897D 8>mov dword ptr ss:\[ebp-80\],edi
004A06B8    .  89BD 7>mov dword ptr ss:\[ebp-90\],edi
004A06BE    .  89BD 6>mov dword ptr ss:\[ebp-A0\],edi
004A06C4    .  89BD 5>mov dword ptr ss:\[ebp-B0\],edi
004A06CA    .  89BD 4>mov dword ptr ss:\[ebp-C0\],edi
004A06D0    .  89BD 3>mov dword ptr ss:\[ebp-D0\],edi
004A06D6    .  89BD 2>mov dword ptr ss:\[ebp-E0\],edi
004A06DC    .  89BD 1>mov dword ptr ss:\[ebp-F0\],edi
004A06E2    .  89BD 0>mov dword ptr ss:\[ebp-100\],ed>
004A06E8    .  89BD F>mov dword ptr ss:\[ebp-110\],ed>
004A06EE    .  89BD E>mov dword ptr ss:\[ebp-120\],ed>
004A06F4    .  89BD D>mov dword ptr ss:\[ebp-130\],ed>
004A06FA    .  89BD C>mov dword ptr ss:\[ebp-140\],ed>
004A0700    .  89BD B>mov dword ptr ss:\[ebp-150\],ed>
004A0706    .  89BD 9>mov dword ptr ss:\[ebp-170\],ed>
004A070C    .  E8 2F3>call pkgs910.00673D40
004A0711    .  8B0E   mov ecx,dword ptr ds:\[esi\]
004A0713    .  56     push esi
004A0714    .  FF91 0>call dword ptr ds:\[ecx+304\]
004A071A    .  8D55 D>lea edx,dword ptr ss:\[ebp-2C\]
004A071D    .  50     push eax
004A071E    .  52     push edx
004A071F    .  FF15 A>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaObjSet
004A0725    .  8BF0   mov esi,eax
004A0727    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]
004A072A    .  51     push ecx
004A072B    .  56     push esi
004A072C    .  8B06   mov eax,dword ptr ds:\[esi\]
004A072E    .  FF90 A>call dword ptr ds:\[eax+A0\]
004A0734    .  3BC7   cmp eax,edi
004A0736    .  DBE2   fclex
004A0738    .  7D 16  jge short pkgs910.004A0750
004A073A    .  8B1D 6>mov ebx,dword ptr ds:\[<&MSVBV>;  MSVBVM60.\_\_vbaHresultCheckObj
004A0740    .  68 A00>push 0A0
004A0745    .  68 C4B>push pkgs910.0042BEC4
004A074A    .  56     push esi
004A074B    .  50     push eax
004A074C    .  FFD3   call ebx                       ;  <&MSVBVM60.\_\_vbaHresultCheckObj>
004A074E    .  EB 06  jmp short pkgs910.004A0756
004A0750    >  8B1D 6>mov ebx,dword ptr ds:\[<&MSVBV>;  MSVBVM60.\_\_vbaHresultCheckObj
004A0756    >  BE 080>mov esi,8
004A075B    .  8D95 1>lea edx,dword ptr ss:\[ebp-F0\]
004A0761    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\]
004A0764    .  C785 1>mov dword ptr ss:\[ebp-E8\],pkg>
004A076E    .  89B5 1>mov dword ptr ss:\[ebp-F0\],esi
004A0774    .  FF15 1>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaVarDup
004A077A    .  8B45 D>mov eax,dword ptr ss:\[ebp-24\]
004A077D    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]
004A0780    .  8945 C>mov dword ptr ss:\[ebp-38\],eax
004A0783    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\]
004A0786    .  52     push edx
004A0787    .  8D4D A>lea ecx,dword ptr ss:\[ebp-60\]
004A078A    .  50     push eax
004A078B    .  51     push ecx
004A078C    .  897D D>mov dword ptr ss:\[ebp-24\],edi
004A078F    .  8975 C>mov dword ptr ss:\[ebp-40\],esi
004A0792    .  E8 595>call pkgs910.00675DF0
004A0797    .  8D55 A>lea edx,dword ptr ss:\[ebp-60\]
004A079A    .  52     push edx
004A079B    .  FF15 3>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarMove
004A07A1    .  8BD0   mov edx,eax
004A07A3    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]
004A07A6    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrMove
004A07AC    .  8D4D D>lea ecx,dword ptr ss:\[ebp-2C\]
004A07AF    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeObj
004A07B5    .  8D45 A>lea eax,dword ptr ss:\[ebp-60\]
004A07B8    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\]
004A07BB    .  50     push eax
004A07BC    .  8D55 C>lea edx,dword ptr ss:\[ebp-40\]
004A07BF    .  51     push ecx
004A07C0    .  52     push edx
004A07C1    .  6A 03  push 3
004A07C3    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList
004A07C9    .  8B45 E>mov eax,dword ptr ss:\[ebp-1C\]
004A07CC    .  83C4 1>add esp,10
004A07CF    .  50     push eax
004A07D0    .  FF15 3>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaLenBstr
004A07D6    .  83F8 1>cmp eax,14
004A07D9    .  0F8C 3>jl pkgs910.004A110F
004A07DF    .  8B3D F>mov edi,dword ptr ds:\[<&MSVBV>;  MSVBVM60.rtcMidCharVar
004A07E5    .  C785 7>mov dword ptr ss:\[ebp-188\],14
004A07EF    .  BE 010>mov esi,1
004A07F4    .  BB 084>mov ebx,4008
004A07F9    >  3BB5 7>cmp esi,dword ptr ss:\[ebp-188>
004A07FF    .  0F8F 3>jg pkgs910.004A093C
004A0805    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]
004A0808    .  8D55 C>lea edx,dword ptr ss:\[ebp-40\]
004A080B    .  898D 1>mov dword ptr ss:\[ebp-E8\],ecx
004A0811    .  52     push edx
004A0812    .  8D85 1>lea eax,dword ptr ss:\[ebp-F0\]
004A0818    .  56     push esi
004A0819    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\]
004A081C    .  50     push eax
004A081D    .  51     push ecx
004A081E    .  C745 C>mov dword ptr ss:\[ebp-38\],1
004A0825    .  C745 C>mov dword ptr ss:\[ebp-40\],2
004A082C    .  899D 1>mov dword ptr ss:\[ebp-F0\],ebx
004A0832    .  FFD7   call edi
004A0834    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]
004A0837    .  8D45 D>lea eax,dword ptr ss:\[ebp-24\]
004A083A    .  52     push edx
004A083B    .  50     push eax
004A083C    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarVal
004A0842    .  50     push eax                  ; /Arg1
004A0843    .  FF15 5>call dword ptr ds:\[<&MSVBVM60>;  \rtcAnsiValueBstr
004A0849    .  0FBFC8 movsx ecx,ax
004A084C    .  894D E>mov dword ptr ss:\[ebp-20\],ecx
004A084F    .  8D4D D>lea ecx,dword ptr ss:\[ebp-24\]
004A0852    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeStr
004A0858    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]
004A085B    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\]
004A085E    .  52     push edx
004A085F    .  50     push eax
004A0860    .  6A 02  push 2
004A0862    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList
```



```
004A0868    .  8B45 E>mov eax,dword ptr ss:\[ebp-20\]  

004A086B    .  83C4 0>add esp,0C  

004A086E    .  83F8 4>cmp eax,41                    ;  第一个字母  

004A0871    .  0F8C B>jl pkgs910.004A0928  

004A0877    .  83F8 5>cmp eax,5A  

004A087A    .  0F8F A>jg pkgs910.004A0928  

004A0880    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]  

004A0883    .  8D55 C>lea edx,dword ptr ss:\[ebp-40\]  

004A0886    .  898D 1>mov dword ptr ss:\[ebp-E8\],ecx  

004A088C    .  52     push edx  

004A088D    .  8D85 1>lea eax,dword ptr ss:\[ebp-F0\]  

004A0893    .  56     push esi  

004A0894    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\]  

004A0897    .  50     push eax  

004A0898    .  51     push ecx  

004A0899    .  C745 C>mov dword ptr ss:\[ebp-38\],1  

004A08A0    .  C745 C>mov dword ptr ss:\[ebp-40\],2  

004A08A7    .  899D 1>mov dword ptr ss:\[ebp-F0\],ebx  

004A08AD    .  FFD7   call edi  

004A08AF    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]  

004A08B2    .  8D45 D>lea eax,dword ptr ss:\[ebp-24\]  

004A08B5    .  52     push edx  

004A08B6    .  50     push eax  

004A08B7    .  FF15 9>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarVal  

004A08BD    .  50     push eax                     ;  /Arg1  

004A08BE    .  FF15 5>call dword ptr ds:\[<&MSVBVM60>; \\\rtcAnsiValueBstr  

004A08C4    .  66:05  >add ax,20                  ;  地一个字母  

004A08C8    .  8D55 A>lea edx,dword ptr ss:\[ebp-60\]  

004A08CB    .  0F80 A>jo pkgs910.004A1474  

004A08D1    .  0FBFC8 movsx ecx,ax  

004A08D4    .  51     push ecx                     ;  /Arg2  

004A08D5    .  52     push edx                     ;  |Arg1  

004A08D6    .  FF15 8>call dword ptr ds:\[<&MSVBVM60>; \\\rtcVarBstrFromAnsi  

004A08DC    .  8D45 E>lea eax,dword ptr ss:\[ebp-1C\]  

004A08DF    .  8D4D A>lea ecx,dword ptr ss:\[ebp-60\]  

004A08E2    .  50     push eax  

004A08E3    .  56     push esi  

004A08E4    .  6A 01  push 1  

004A08E6    .  51     push ecx  

004A08E7    .  FF15 3>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrVarMove  

004A08ED    .  8BD0   mov edx,eax  

004A08EF    .  8D4D D>lea ecx,dword ptr ss:\[ebp-28\]  

004A08F2    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrMove  

004A08F8    .  50     push eax                     ;  |Arg2  

004A08F9    .  6A 00  push 0                      ;  |Arg1 = 00000000  

004A08FB    .  FF15 8>call dword ptr ds:\[<&MSVBVM60>; \\\_vbaMidStmtBstr  

004A0901    .  8D55 D>lea edx,dword ptr ss:\[ebp-28\]  

004A0904    .  8D45 D>lea eax,dword ptr ss:\[ebp-24\]  

004A0907    .  52     push edx  

004A0908    .  50     push eax  

004A0909    .  6A 02  push 2  

004A090B    .  FF15 D>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeStrList  

004A0911    .  8D4D A>lea ecx,dword ptr ss:\[ebp-60\]  

004A0914    .  8D55 B>lea edx,dword ptr ss:\[ebp-50\]  

004A0917    .  51     push ecx  

004A0918    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\]  

004A091B    .  52     push edx  

004A091C    .  50     push eax  

004A091D    .  6A 03  push 3  

004A091F    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList  

004A0925    .  83C4 1>add esp,1C  

004A0928    >  B8 010>mov eax,1  

004A092D    .  03C6   add eax,esi  

004A092F    .  0F80 3>jo pkgs910.004A1474  

004A0935    .  8BF0   mov esi,eax  

004A0937    .^ E9 BDF>jmp pkgs910.004A07F9         ;  上面一段是判断你输入的是否为小写字母，如果不是转换成小写放于 001546F4  

004A093C    >  8D95 1>lea edx,dword ptr ss:\[ebp-F0\]  

004A0942    .  6A 06  push 6                     ;  /Arg3 = 00000006  

004A0944    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\] ;  |  

004A0947    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\] ;  |  

004A094A    .  52     push edx                   ;  |Arg2  

004A094B    .  50     push eax                   ;  |Arg1  

004A094C    .  898D 1>mov dword ptr ss:\[ebp-E8\],ecx ;  |  

004A0952    .  899D 1>mov dword ptr ss:\[ebp-F0\],ebx ;  |  

004A0958    .  FF15 3>call dword ptr ds:\[<&MSVBVM60>; \\\ 取前六位放于 0016E9BC  

004A095E    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]  

004A0961    .  8D55 A>lea edx,dword ptr ss:\[ebp-60\]  

004A0964    .  BE 080>mov esi,8  

004A0969    .  898D F>mov dword ptr ss:\[ebp-108\],ec>  

004A096F    .  52     push edx  

004A0970    .  8D85 F>lea eax,dword ptr ss:\[ebp-110>  

004A0976    .  56     push esi  

004A0977    .  8D4D 9>lea ecx,dword ptr ss:\[ebp-70\]  

004A097A    .  50     push eax  

004A097B    .  51     push ecx  

004A097C    .  C785 0>mov dword ptr ss:\[ebp-F8\],pkg>  

004A0986    .  89B5 0>mov dword ptr ss:\[ebp-100\],es>  

004A098C    .  C745 A>mov dword ptr ss:\[ebp-58\],6   ;  要 6 位  

004A0993    .  C745 A>mov dword ptr ss:\[ebp-60\],2   ;  两位两位的取  

004A099A    .  899D F>mov dword ptr ss:\[ebp-110\],eb>  

004A09A0    .  FFD7   call edi                   ;  取中间六位  

004A09A2    .  8D85 C>lea eax,dword ptr ss:\[ebp-140>  

004A09A8    .  6A 06  push 6                     ;  /Arg3 = 00000006  

004A09AA    .  8D8D 6>lea ecx,dword ptr ss:\[ebp-A0\] ;  |  

004A09B0    .  8D55 E>lea edx,dword ptr ss:\[ebp-1C\] ;  |  

004A09B3    .  50     push eax                   ;  |Arg2  

004A09B4    .  51     push ecx                   ;  |Arg1  

004A09B5    .  C785 D>mov dword ptr ss:\[ebp-128\],pk>;  |  

004A09BF    .  89B5 D>mov dword ptr ss:\[ebp-130\],es>;  |  

004A09C5    .  8995 C>mov dword ptr ss:\[ebp-138\],ed>;  |  

004A09CB    .  899D C>mov dword ptr ss:\[ebp-140\],eb>;  |  

004A09D1    .  FF15 5>call dword ptr ds:\[<&MSVBVM60>; \\\ 取后六位防于 00154A4C  

004A09D7    .  8B35 9>mov esi,dword ptr ds:\[<&MSVBV>;  MSVBVM60.\_\_vbaVarCat  

004A09DD    .  8D55 C>lea edx,dword ptr ss:\[ebp-40\]  

004A09E0    .  8D85 0>lea eax,dword ptr ss:\[ebp-100>  

004A09E6    .  52     push edx  

004A09E7    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\]  

004A09EA    .  50     push eax  

004A09EB    .  51     push ecx  

004A09EC    .  FFD6   call esi                   ;  在前 6 位后面加一条-;  <&MSVBVM60.\_\_vbaVarCat>  

004A09EE    .  50     push eax  

004A09EF    .  8D55 9>lea edx,dword ptr ss:\[ebp-70\]  

004A09F2    .  8D45 8>lea eax,dword ptr ss:\[ebp-80\]  

004A09F5    .  52     push edx  

004A09F6    .  50     push eax  

004A09F7    .  FFD6   call esi                   ;  把中间 6 位又加入其中防于 0016420C  

004A09F9    .  8D8D D>lea ecx,dword ptr ss:\[ebp-130>  

004A09FF    .  50     push eax  

004A0A00    .  8D95 7>lea edx,dword ptr ss:\[ebp-90\]  

004A0A06    .  51     push ecx  

004A0A07    .  52     push edx  

004A0A08    .  FFD6   call esi                   ;  再加一条“-”  

004A0A0A    .  50     push eax  

004A0A0B    .  8D85 6>lea eax,dword ptr ss:\[ebp-A0\]  

004A0A11    .  8D8D 5>lea ecx,dword ptr ss:\[ebp-B0\]  

004A0A17    .  50     push eax  

004A0A18    .  51     push ecx  

004A0A19    .  FFD6   call esi                   ;  把后 6 位又加入其中防于 00154A04  

004A0A1B    .  50     push eax  

004A0A1C    .  FF15 3>call dword ptr ds:\[<&MSVBVM60>;  拷贝到 EAX 中  

004A0A22    .  8BD0   mov edx,eax  

004A0A24    .  B9 549>mov ecx,pkgs910.00739354  

004A0A29    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaStrMove  

004A0A2F    .  8D95 5>lea edx,dword ptr ss:\[ebp-B0\]  

004A0A35    .  8D85 6>lea eax,dword ptr ss:\[ebp-A0\]  

004A0A3B    .  52     push edx  

004A0A3C    .  8D8D 7>lea ecx,dword ptr ss:\[ebp-90\]  

004A0A42    .  50     push eax  

004A0A43    .  8D55 8>lea edx,dword ptr ss:\[ebp-80\]  

004A0A46    .  51     push ecx  

004A0A47    .  8D45 9>lea eax,dword ptr ss:\[ebp-70\]  

004A0A4A    .  52     push edx  

004A0A4B    .  8D4D B>lea ecx,dword ptr ss:\[ebp-50\] ;  上面都是 8  

004A0A4E    .  50     push eax  

004A0A4F    .  8D55 A>lea edx,dword ptr ss:\[ebp-60\]  

004A0A52    .  51     push ecx  

004A0A53    .  52     push edx  

004A0A54    .  8D45 C>lea eax,dword ptr ss:\[ebp-40\]  

004A0A57    .  50     push eax  

004A0A58    .  6A 08  push 8  

004A0A5A    .  FF15 4>call dword ptr ds:\[<&MSVBVM60>;  MSVBVM60.\_\_vbaFreeVarList  
```



```
004A0A60    .   83C4 2>add esp,24

004A0A63    .   8D4D E>lea ecx,dword ptr ss:[ebp-1C]

004A0A66    .   8D55 C>lea edx,dword ptr ss:[ebp-40]

004A0A69    .   898D 1>mov dword ptr ss:[ebp-E8],ecx

004A0A6F    .   52      push edx

004A0A70    .   8D85 1>lea eax,dword ptr ss:[ebp-F0]

004A0A76    .   6A 02   push 2

004A0A78    .   8D4D B>lea ecx,dword ptr ss:[ebp-50]

004A0A7B    .   50      push eax

004A0A7C    .   51      push ecx

004A0A7D    .   C745 C>mov dword ptr ss:[ebp-38],5

004A0A84    .   C745 C>mov dword ptr ss:[ebp-40],2

004A0A8B    .   899D 1>mov dword ptr ss:[ebp-F0],ebx

004A0A91    .   FFD7    call edi                      ;  从第一组 6 个数据中，取后 5 位数放与 EBP-48

004A0A93    .   8D55 E>lea edx,dword ptr ss:[ebp-1C]

004A0A96    .   8D45 9>lea eax,dword ptr ss:[ebp-70]

004A0A99    .   8995 E>mov dword ptr ss:[ebp-118],ed>

004A0A9F    .   50      push eax

004A0AA0    .   8D8D E>lea ecx,dword ptr ss:[ebp-120>

004A0AA6    .   6A 09   push 9

004A0AA8    .   8D55 8>lea edx,dword ptr ss:[ebp-80]

004A0AAB    .   51      push ecx

004A0AAC    .   52      push edx

004A0AAD    .   C785 F>mov dword ptr ss:[ebp-108],pk>

004A0AB7    .   C785 F>mov dword ptr ss:[ebp-110],8

004A0AC1    .   C745 9>mov dword ptr ss:[ebp-68],5

004A0AC8    .   C745 9>mov dword ptr ss:[ebp-70],2

004A0ACF    .   899D E>mov dword ptr ss:[ebp-120],eb>

004A0AD5    .   FFD7    call edi                      ;  从第 2 组 6 个数据中，取后 5 位数放与 EBP-64

004A0AD7    .   8D8D B>lea ecx,dword ptr ss:[ebp-150>

004A0ADD    .   6A 04   push 4                        ; /Arg3 = 00000004

004A0ADF    .   8D95 5>lea edx,dword ptr ss:[ebp-B0]  ; |

004A0AE5    .   8D45 E>lea eax,dword ptr ss:[ebp-1C]  ; |

004A0AE8    .   51      push ecx                      ; |Arg2

004A0AE9    .   52      push edx                      ; |Arg1

004A0AEA    .   C785 C>mov dword ptr ss:[ebp-138],pk>; |

004A0AF4    .   C785 C>mov dword ptr ss:[ebp-140],8  ; |

004A0AFE    .   8985 B>mov dword ptr ss:[ebp-148],ea>; |

004A0B04    .   899D B>mov dword ptr ss:[ebp-150],eb>; |

004A0B0A    .   FF15 5>call dword ptr ds:[<&MSVBVM60>; \\\从第 3 组 6 个数据中，取后 4 位数放与 EBP-A8

004A0B10    .   8D45 B>lea eax,dword ptr ss:[ebp-50]

004A0B13    .   8D8D F>lea ecx,dword ptr ss:[ebp-110>

004A0B19    .   50      push eax

004A0B1A    .   8D55 A>lea edx,dword ptr ss:[ebp-60]

004A0B1D    .   51      push ecx

004A0B1E    .   52      push edx

004A0B1F    .   FFD6    call esi                      ;  第一组 5 位与“-”组合

004A0B21    .   50      push eax

004A0B22    .   8D45 8>lea eax,dword ptr ss:[ebp-80]

004A0B25    .   8D8D 7>lea ecx,dword ptr ss:[ebp-90]

004A0B2B    .   50      push eax

004A0B2C    .   51      push ecx

004A0B2D    .   FFD6    call esi                      ;  把第二组 5 位加入 EBP-88

004A0B2F    .   50      push eax

004A0B30    .   8D95 C>lea edx,dword ptr ss:[ebp-140>

004A0B36    .   8D85 6>lea eax,dword ptr ss:[ebp-A0]

004A0B3C    .   52      push edx

004A0B3D    .   50      push eax

004A0B3E    .   FFD6    call esi                      ;  加一个“-”

004A0B40    .   8D8D 5>lea ecx,dword ptr ss:[ebp-B0]

004A0B46    .   50      push eax

004A0B47    .   8D95 4>lea edx,dword ptr ss:[ebp-C0]

004A0B4D    .   51      push ecx

004A0B4E    .   52      push edx

004A0B4F    .   FFD6    call esi                      ;  与后 4 位组合 EBP-B8

004A0B51    .   50      push eax

004A0B52    .   FF15 3>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaStrVarMove

004A0B58    .   8BD0    mov edx,eax                   ;  拷贝到 EAX

004A0B5A    .   B9 989>mov ecx,pkgs910.00739498

004A0B5F    .   FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaStrMove

004A0B65    .   8D85 4>lea eax,dword ptr ss:[ebp-C0]

004A0B6B    .   8D8D 5>lea ecx,dword ptr ss:[ebp-B0]

004A0B71    .   50      push eax

004A0B72    .   8D95 6>lea edx,dword ptr ss:[ebp-A0]

004A0B78    .   51      push ecx

004A0B79    .   52      push edx

004A0B7A    .   8D85 7>lea eax,dword ptr ss:[ebp-90]

004A0B80    .   8D4D 8>lea ecx,dword ptr ss:[ebp-80]

004A0B83    .   50      push eax

004A0B84    .   8D55 A>lea edx,dword ptr ss:[ebp-60]

004A0B87    .   51      push ecx

004A0B88    .   8D45 9>lea eax,dword ptr ss:[ebp-70]

004A0B8B    .   52      push edx

004A0B8C    .   8D4D B>lea ecx,dword ptr ss:[ebp-50]

004A0B8F    .   50      push eax

004A0B90    .   8D55 C>lea edx,dword ptr ss:[ebp-40]

004A0B93    .   51      push ecx

004A0B94    .   52      push edx

004A0B95    .   6A 09   push 9

004A0B97    .   FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaFreeVarList

004A0B9D    .   A1 989>mov eax,dword ptr ds:[739498] ;  取第二次组合的值

004A0BA2    .   8B0D B>mov ecx,dword ptr ds:[7394B0]

004A0BA8    .   83C4 2>add esp,28

004A0BAB    .   8D55 C>lea edx,dword ptr ss:[ebp-40]

004A0BAE    .   8985 9>mov dword ptr ss:[ebp-168],ea>;  储存到 EBP-168

004A0BB4    .   898D 1>mov dword ptr ss:[ebp-E8],ecx

004A0BBA    .   52      push edx

004A0BBB    .   8D85 1>lea eax,dword ptr ss:[ebp-F0]

004A0BC1    .   6A 02   push 2

004A0BC3    .   8D4D B>lea ecx,dword ptr ss:[ebp-50]

004A0BC6    .   50      push eax

004A0BC7    .   51      push ecx

004A0BC8    .   C785 9>mov dword ptr ss:[ebp-170],80>

004A0BD2    .   C745 C>mov dword ptr ss:[ebp-38],5

004A0BD9    .   C745 C>mov dword ptr ss:[ebp-40],2

004A0BE0    .   899D 1>mov dword ptr ss:[ebp-F0],ebx

004A0BE6    .   FFD7    call edi                      ;  在 EBP-A8 中出现 hxhgg

004A0BE8    .   8B15 B>mov edx,dword ptr ds:[7394B0]

004A0BEE    .   8D45 9>lea eax,dword ptr ss:[ebp-70]

004A0BF1    .   8995 E>mov dword ptr ss:[ebp-118],ed>

004A0BF7    .   50      push eax

004A0BF8    .   8D8D E>lea ecx,dword ptr ss:[ebp-120>

004A0BFE    .   6A 09   push 9

004A0C00    .   8D55 8>lea edx,dword ptr ss:[ebp-80]

004A0C03    .   51      push ecx

004A0C04    .   52      push edx

004A0C05    .   C785 F>mov dword ptr ss:[ebp-108],pk>

004A0C0F    .   C785 F>mov dword ptr ss:[ebp-110],8

004A0C19    .   C745 9>mov dword ptr ss:[ebp-68],5

004A0C20    .   C745 9>mov dword ptr ss:[ebp-70],2

004A0C27    .   899D E>mov dword ptr ss:[ebp-120],eb>

004A0C2D    .   FFD7    call edi                      ;  在 EBP-78 中出现 2x3d9

004A0C2F    .   A1 B09>mov eax,dword ptr ds:[7394B0]

004A0C34    .   8D8D 5>lea ecx,dword ptr ss:[ebp-B0]

004A0C3A    .   8985 B>mov dword ptr ss:[ebp-148],ea>

004A0C40    .   51      push ecx

004A0C41    .   8D95 B>lea edx,dword ptr ss:[ebp-150>

004A0C47    .   6A 11   push 11

004A0C49    .   8D85 4>lea eax,dword ptr ss:[ebp-C0]

004A0C4F    .   52      push edx

004A0C50    .   50      push eax

004A0C51    .   C785 C>mov dword ptr ss:[ebp-138],pk>

004A0C5B    .   C785 C>mov dword ptr ss:[ebp-140],8

004A0C65    .   C785 5>mov dword ptr ss:[ebp-A8],800>

004A0C6F    .   C785 5>mov dword ptr ss:[ebp-B0],0A

004A0C79    .   899D B>mov dword ptr ss:[ebp-150],eb>

004A0C7F    .   FFD7    call edi                      ;  有出现 4 位，EBP-58

004A0C81    .   8D8D 9>lea ecx,dword ptr ss:[ebp-170>

004A0C87    .   8D55 B>lea edx,dword ptr ss:[ebp-50]

004A0C8A    .   51      push ecx

004A0C8B    .   8D85 F>lea eax,dword ptr ss:[ebp-110>

004A0C91    .   52      push edx

004A0C92    .   8D4D A>lea ecx,dword ptr ss:[ebp-60]

004A0C95    .   50      push eax

004A0C96    .   51      push ecx

004A0C97    .   FFD6    call esi                      ;  组合，hxhgg-

004A0C99    .   50      push eax

004A0C9A    .   8D55 8>lea edx,dword ptr ss:[ebp-80]

004A0C9D    .   8D85 7>lea eax,dword ptr ss:[ebp-90]

004A0CA3    .   52      push edx

004A0CA4    .   50      push eax

004A0CA5    .   FFD6    call esi                      ;  组合 hxhgg-2x3d9

004A0CA7    .   8D8D C>lea ecx,dword ptr ss:[ebp-140>

004A0CAD    .   50      push eax
```



```
004A0CAE    .  8D95 6>lea edx,dword ptr ss:[ebp-A0]  

004A0CB4    .  51     push ecx  

004A0CB5    .  52     push edx  

004A0CB6    .  FFD6   call esi                  ;  组合成 hxhgg-2x3d9-   EBP-98  

004A0CB8    .  50     push eax  

004A0CB9    .  8D85 4>lea eax,dword ptr ss:[ebp-C0]  

004A0CBF    .  8D8D 3>lea ecx,dword ptr ss:[ebp-D0]  

004A0CC5    .  50     push eax  

004A0CC6    .  51     push ecx  

004A0CC7    .  FFD6   call esi                  ;  hxhgg-2x3d9-2gdx  

004A0CC9    .  50     push eax  

004A0CCA    .  FF15 1>call dword ptr ds:[<&MSVBVM60>;  比较了，我输入的，与她计算出来的  

004A0CD0    .  66:898>mov word ptr ss:[ebp-180],ax  

004A0CD7    .  8D95 3>lea edx,dword ptr ss:[ebp-D0]  

004A0CDD    .  8D85 4>lea eax,dword ptr ss:[ebp-C0]  

004A0CE3    .  52     push edx  

004A0CE4    .  8D8D 6>lea ecx,dword ptr ss:[ebp-A0]  

004A0CEA    .  50     push eax  

004A0CEB    .  8D95 5>lea edx,dword ptr ss:[ebp-B0]  

004A0CF1    .  51     push ecx  

004A0CF2    .  8D85 7>lea eax,dword ptr ss:[ebp-90]  

004A0CF8    .  52     push edx  

004A0CF9    .  8D4D 8>lea ecx,dword ptr ss:[ebp-80]  

004A0CFC    .  50     push eax  

004A0CFD    .  8D55 A>lea edx,dword ptr ss:[ebp-60]  

004A0D00    .  51     push ecx  

004A0D01    .  8D45 9>lea eax,dword ptr ss:[ebp-70]  

004A0D04    .  52     push edx  

004A0D05    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]  

004A0D08    .  50     push eax  

004A0D09    .  8D55 C>lea edx,dword ptr ss:[ebp-40]  

004A0D0C    .  51     push ecx  

004A0D0D    .  52     push edx  

004A0D0E    .  6A 0A  push 0A  

004A0D10    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaFreeVarList  

004A0D16    .  83C4 2>add esp,2C  

004A0D19    .  66:83B>cmp word ptr ss:[ebp-180],0  

004A0D21    .  74 0A  je short pkgs910.004A0D2D  

004A0D23    .  B8 010>mov eax,1  

004A0D28    .  E9 9C0>jmp pkgs910.004A0EC9  

004A0D2D    >  8B0D B>mov ecx,dword ptr ds:[7394B0]  

004A0D33    .  A1 989>mov eax,dword ptr ds:[739498]  

004A0D38    .  83C1 0>add ecx,4  

004A0D3B    .  8D55 C>lea edx,dword ptr ss:[ebp-40]  

004A0D3E    .  8985 9>mov dword ptr ss:[ebp-168],ea>  

004A0D44    .  898D 1>mov dword ptr ss:[ebp-E8],ecx  

004A0D4A    .  52     push edx  

004A0D4B    .  8D85 1>lea eax,dword ptr ss:[ebp-F0]  

004A0D51    .  6A 02  push 2  

004A0D53    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]  

004A0D56    .  50     push eax  

004A0D57    .  51     push ecx  

004A0D58    .  C785 9>mov dword ptr ss:[ebp-170],80>  

004A0D62    .  C745 C>mov dword ptr ss:[ebp-38],5  

004A0D69    .  C745 C>mov dword ptr ss:[ebp-40],2  

004A0D70    .  899D 1>mov dword ptr ss:[ebp-F0],ebx  

004A0D76    .  FFD7   call edi                  ;  在 EBP-48 中 9fg88  

004A0D78    .  8B15 B>mov edx,dword ptr ds:[7394B0]  

004A0D7E    .  8D45 9>lea eax,dword ptr ss:[ebp-70]  

004A0D81    .  83C2 0>add edx,4  

004A0D84    .  50     push eax  

004A0D85    .  8995 E>mov dword ptr ss:[ebp-118],ed>  

004A0D8B    .  8D8D E>lea ecx,dword ptr ss:[ebp-120>  

004A0D91    .  6A 09  push 9  

004A0D93    .  8D55 8>lea edx,dword ptr ss:[ebp-80]  

004A0D96    .  51     push ecx  

004A0D97    .  52     push edx  

004A0D98    .  C785 F>mov dword ptr ss:[ebp-108],pk>  

004A0DA2    .  C785 F>mov dword ptr ss:[ebp-110],8  

004A0DAC    .  C745 9>mov dword ptr ss:[ebp-68],5  

004A0DB3    .  C745 9>mov dword ptr ss:[ebp-70],2  

004A0DBA    .  899D E>mov dword ptr ss:[ebp-120],eb>  

004A0DC0    .  FFD7   call edi                  ; 在 EBP-58 中 28dgf  

004A0DC2    .  A1 B09>mov eax,dword ptr ds:[7394B0]  

004A0DC7    .  8D8D 5>lea ecx,dword ptr ss:[ebp-B0]  

004A0DCD    .  83C0 0>add eax,4  

004A0DD0    .  51     push ecx  

004A0DD1    .  8985 B>mov dword ptr ss:[ebp-148],ea>  

004A0DD7    .  8D95 B>lea edx,dword ptr ss:[ebp-150>  

004A0DDD    .  6A 11  push 11  

004A0DDF    .  8D85 4>lea eax,dword ptr ss:[ebp-C0]  

004A0DE5    .  52     push edx  

004A0DE6    .  50     push eax  

004A0DE7    .  C785 C>mov dword ptr ss:[ebp-138],pk>  

004A0DF1    .  C785 C>mov dword ptr ss:[ebp-140],8  

004A0DFB    .  C785 5>mov dword ptr ss:[ebp-A8],800>  

004A0E05    .  C785 5>mov dword ptr ss:[ebp-B0],0A  

004A0E0F    .  899D B>mov dword ptr ss:[ebp-150],eb>  

004A0E15    .  FFD7   call edi                  ;在 EBP-b8 中 9hp9  

004A0E17    .  8D8D 9>lea ecx,dword ptr ss:[ebp-170>  

004A0E1D    .  8D55 B>lea edx,dword ptr ss:[ebp-50]  

004A0E20    .  51     push ecx  

004A0E21    .  8D85 F>lea eax,dword ptr ss:[ebp-110>  

004A0E27    .  52     push edx  

004A0E28    .  8D4D A>lea ecx,dword ptr ss:[ebp-60]  

004A0E2B    .  50     push eax  

004A0E2C    .  51     push ecx  

004A0E2D    .  FFD6   call esi                  ;  组合  

004A0E2F    .  50     push eax  

004A0E30    .  8D55 8>lea edx,dword ptr ss:[ebp-80]  

004A0E33    .  8D85 7>lea eax,dword ptr ss:[ebp-90]  

004A0E39    .  52     push edx  

004A0E3A    .  50     push eax  

004A0E3B    .  FFD6   call esi                  ;  组合  

004A0E3D    .  8D8D C>lea ecx,dword ptr ss:[ebp-140>  

004A0E43    .  50     push eax  

004A0E44    .  8D95 6>lea edx,dword ptr ss:[ebp-A0]  

004A0E4A    .  51     push ecx  

004A0E4B    .  52     push edx  

004A0E4C    .  FFD6   call esi                  ;  组合  

004A0E4E    .  50     push eax  

004A0E4F    .  8D85 4>lea eax,dword ptr ss:[ebp-C0]  

004A0E55    .  8D8D 3>lea ecx,dword ptr ss:[ebp-D0]  

004A0E5B    .  50     push eax  

004A0E5C    .  51     push ecx  

004A0E5D    .  FFD6   call esi                  ;  组合  

004A0E5F    .  50     push eax  

004A0E60    .  FF15 1>call dword ptr ds:[<&MSVBVM60>;  与我输入的 bcdef-hijkl-opqr 比较 9fg88-28dgf-ghp9  

004A0E66    .  8D95 3>lea edx,dword ptr ss:[ebp-D0]  

004A0E6C    .  66:8BF>mov si,ax  

004A0E6F    .  52     push edx  

004A0E70    .  8D85 4>lea eax,dword ptr ss:[ebp-C0]  

004A0E76    .  8D8D 6>lea ecx,dword ptr ss:[ebp-A0]  

004A0E7C    .  50     push eax  

004A0E7D    .  8D95 5>lea edx,dword ptr ss:[ebp-B0]  

004A0E83    .  51     push ecx  

004A0E84    .  8D85 7>lea eax,dword ptr ss:[ebp-90]  

004A0E8A    .  52     push edx  

004A0E8B    .  8D4D 8>lea ecx,dword ptr ss:[ebp-80]  

004A0E8E    .  50     push eax  

004A0E8F    .  8D55 A>lea edx,dword ptr ss:[ebp-60]  

004A0E92    .  51     push ecx  

004A0E93    .  8D45 9>lea eax,dword ptr ss:[ebp-70]  

004A0E96    .  52     push edx  

004A0E97    .  8D4D B>lea ecx,dword ptr ss:[ebp-50]  

004A0E9A    .  50     push eax  

004A0E9B    .  8D55 C>lea edx,dword ptr ss:[ebp-40]  

004A0E9E    .  51     push ecx  

004A0E9F    .  52     push edx  

004A0EA0    .  6A 0A  push 0A  

004A0EA2    .  FF15 4>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaFreeVarList  

004A0EA8    .  83C4 2>add esp,2C  

004A0EAB    .  66:85F>test si,si  

004A0EAE    .  74 07  je short pkgs910.004A0EB7     ;  爆破点啊！  

004A0EB0    .  B8 020>mov eax,2  

004A0EB5    .  EB 12  jmp short pkgs910.004A0EC9  

004A0EB7    >  BA 54C>mov edx,pkgs910.0042C754  

004A0EBC    .  B9 549>mov ecx,pkgs910.00739354  

004A0EC1    .  FF15 C>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaStrCopy  

004A0EC7    .  33C0   xor eax,eax  

004A0EC9    >  85C0   test eax,eax  

004A0ECB    .  A3 9C9>mov dword ptr ds:[73949C],eax  

004A0ED0    .  0F8E D>jle pkgs910.004A0FB4          ;  一处  

004A0ED6    .  A1 549>mov eax,dword ptr ds:[739354]  

004A0EDB    .  8B35 0>mov esi,dword ptr ds:[<&MSVBV>;  MSVBVM60.rtcSaveSetting  

004A0EE1    .  50     push eax  

004A0EE2    .  68 70D>push pkgs910.0042D170  

004A0EE7    .  68 64D>push pkgs910.0042D164         ;  UNICODE "ini"  

004A0EEC    .  68 5CD>push pkgs910.0042D15C  

004A0EF1    .  FFD6   call esi                  ;  <&MSVBVM60.#690>  

004A0EF3    .  68 54C>push pkgs910.0042C754  

004A0EF8    .  68 70D>push pkgs910.0042D170  
```



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



## 注册码验证逻辑分析

### 1. 初始比较与验证片段

以下汇编代码展示了注册码验证的第一部分逻辑：

```
004A1129    .  FF15 9>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaFreeStr
004A112F    >  393D 9>cmp dword ptr ds:[73949C],edi
004A1135    .  0F8E 9>jle pkgs910.004A13D9
004A113B    .  A1 FC9>mov eax,dword ptr ds:[7394FC]
004A1140    .  3BC7   cmp eax,edi
004A1142    .  75 15  jnz short pkgs910.004A1159
004A1144    .  68 FC9>push pkgs910.007394FC          ; /Arg2 = 007394FC
004A1149    .  68 4CB>push pkgs910.0040B04C          ; |Arg1 = 0040B04C
004A114E    .  FF15 B>call dword ptr ds:[<&MSVBVM60>; \\____vbaNew2
004A1154    .  A1 FC9>mov eax,dword ptr ds:[7394FC]
004A1159    >  8B10   mov edx,dword ptr ds:[eax]
004A115B    .  50     push eax
004A115C    .  FF92 1>call dword ptr ds:[edx+31C]
004A1162    .  50     push eax
004A1163    .  8D45 D>lea eax,dword ptr ss:[ebp-2C]
004A1166    .  50     push eax
004A1167    .  FF15 A>call dword ptr ds:[<&MSVBVM60>;  MSVBVM60.__vbaObjSet
004A116D    .  8BF0   mov esi,eax
004A116F    .  8D55 D>lea edx,dword ptr ss:[ebp-30]
004A1172    .  52     push edx
004A1173    .  6A 03  push 3
004A1175    .  8B0E   mov ecx,dword ptr ds:[esi]
004A1177    .  56     push esi
004A1178    .  FF51 4>call dword ptr ds:[ecx+40]
004A117B    .  3BC7   cmp eax,edi
004A117D    .  DBE2   fclex
004A117F    .  7D 0B  jge short pkgs910.004A118C
004A1181    .  6A 40  push 40
004A1183    .  68 0CC>push pkgs910.0042C00C
004A1188    .  56     push esi
004A1189    .  50     push eax
004A118A    .  FFD3   call ebx
004A118C    >  8B45 D>mov eax,dword ptr ss:[ebp-30]
004A118F    .  68 008>push 44F78000
004A1194    .  50     push eax
004A1195    .  8BF0   mov esi,eax
004A1197    .  8B08   mov ecx,dword ptr ds:[eax]
004A1199    .  FF51 7>call dword ptr ds:[ecx+74]
004A119C    .  3BC7   cmp eax,edi
004A119E    .  DBE2   fclex
004A11A0    .  7D 0B  jge short pkgs910.004A11AD
004A11A2    .  6A 74  push 74
004A11A4    .  68 B4B>push pkgs910.0042BEB4
004A11A9    .  56     push esi
```

### 2. 验证规则说明

上述比较逻辑要求注册码必须为 20 位，其中每 6 位为一组，用“－”隔开。在比较时，程序只校验第一组、第二组的后 5 位以及第三组的后 4 位，并不对全部 20 位进行完整比较。这意味着存在大量的有效注册码组合。

### 3. 注册码生成算法追踪

由于上述代码段中未直接包含注册码计算逻辑，推测注册码在软件运行初期即已完成计算。鉴于相关窗口程序代码较短，堆栈中信息有限，采用动态调试法：发现运行后注册码固定存储于地址 `00154a24`。

使用命令 `d 00154a24` 查看该地址，同时使用 `f8` 设置硬件断点观察变化。每当 `00154a24` 内容发生变化时，使用 `f7` 移除断点并记录，直到定位到关键调用点：

```
005D9FA9    .  FFD3   call ebx                      ;  得到机器码
005D9FAB    .  8BD0   mov edx,eax
005D9FAD    .  B9 489>mov ecx,pkgs910.00739448
005D9FB2    .  FFD6   call esi
005D9FB4    .  E9 E50>jmp pkgs910.005DA09E
005D9FB9    >  8B0D 5>mov ecx,dword ptr ds:[739354]
005D9FBF    .  51     push ecx
005D9FC0    .  68 54C>push pkgs910.0042C754
005D9FC5    .  FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaStrCmp>>;  MSVBVM60.__vbaStrCmp
005D9FCB    .  F7D8   neg eax
005D9FCD    .  1BC0   sbb eax,eax
005D9FCF    .  8D8D 6>lea ecx,dword ptr ss:[ebp-94]
005D9FD5    .  40     inc eax
005D9FD6    .  51     push ecx
005D9FD7    .  F7D8   neg eax
005D9FD9    .  66:898>mov word ptr ss:[ebp-12C],ax
005D9FE0    .  8B45 0>mov eax,dword ptr ss:[ebp+8]
005D9FE3    .  50     push eax
005D9FE4    .  C785 C>mov dword ptr ss:[ebp-134],0B
005D9FEE    .  8B10   mov edx,dword ptr ds:[eax]
005D9FF0    .  FF92 0>call dword ptr ds:[edx+700]
005D9FF6    .  85C0   test eax,eax
005D9FF8    .  7D 15  jge short pkgs910.005DA00F
005D9FFA    .  8B55 0>mov edx,dword ptr ss:[ebp+8]
005D9FFD    .  68 000>push 700
005DA002    .  68 FCD>push pkgs910.0042D5FC
005DA007    .  52     push edx
```

后续代码片段：

```
005DAC7F    .  50     push eax
005DAC80    .  6A 0B  push 0B
005DAC82    .  FF15 D>call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>;  MSVBVM60.__vbaFreeStrList
005DAC88    .  83C4 3>add esp,30
005DAC8B    >  8B7D 0>mov edi,dword ptr ss:[ebp+8]
005DAC8E    .  57     push edi
005DAC8F    .  8B0F   mov ecx,dword ptr ds:[edi]
005DAC91    .  FF91 F>call dword ptr ds:[ecx+6F8]                ;  一直走到它进入
```

### 4. 核心计算子程序入口

进入地址 `005DAC91` 对应的子程序，该处为注册码计算的核心逻辑：

```
005DC180    > \55     push ebp
005DC181    .  8BEC   mov ebp,esp
005DC183    .  83EC 0>sub esp,0C
005DC186    .  68 A67>push <jmp.&MSVBVM60.__vbaExceptHandler>    ;  SE handler installation
005DC18B    .  64:A1 >mov eax,dword ptr fs:[0]
005DC191    .  50     push eax
005DC192    .  64:892>mov dword ptr fs:[0],esp
005DC199    .  81EC 5>sub esp,15C
005DC19F    .  53     push ebx
005DC1A0    .  56     push esi
005DC1A1    .  57     push edi
005DC1A2    .  8965 F>mov dword ptr ss:[ebp-C],esp
005DC1A5    .  C745 F>mov dword ptr ss:[ebp-8],pkgs910.004043D8
005DC1AC    .  33FF   xor edi,edi
005DC1AE    .  897D F>mov dword ptr ss:[ebp-4],edi
005DC1B1    .  8B45 0>mov eax,dword ptr ss:[ebp+8]
005DC1B4    .  50     push eax
005DC1B5    .  8B08   mov ecx,dword ptr ds:[eax]
005DC1B7    .  FF51 0>call dword ptr ds:[ecx+4]
005DC1BA    .  8B15 5>mov edx,dword ptr ds:[739354]
005DC1C0    .  8B1D C>mov ebx,dword ptr ds:[<&MSVBVM60.__vbaStrC>;  MSVBVM60.__vbaStrCopy
005DC1C6    .  8D4D E>lea ecx,dword ptr ss:[ebp-18]
005DC1C9    .  897D E>mov dword ptr ss:[ebp-18],edi
005DC1CC    .  897D D>mov dword ptr ss:[ebp-28],edi
005DC1CF    .  897D C>mov dword ptr ss:[ebp-38],edi
005DC1D2    .  897D B>mov dword ptr ss:[ebp-48],edi
005DC1D5    .  897D A>mov dword ptr ss:[ebp-58],edi
005DC1D8    .  897D 9>mov dword ptr ss:[ebp-68],edi
005DC1DB    .  897D 8>mov dword ptr ss:[ebp-78],edi
005DC1DE    .  89BD 7>mov dword ptr ss:[ebp-88],edi
005DC1E4    .  89BD 6>mov dword ptr ss:[ebp-98],edi
005DC1EA    .  89BD 5>mov dword ptr ss:[ebp-A8],edi
005DC1F0    .  89BD 4>mov dword ptr ss:[ebp-B8],edi
005DC1F6    .  89BD 3>mov dword ptr ss:[ebp-C8],edi
005DC1FC    .  89BD 2>mov dword ptr ss:[ebp-D8],edi
005DC202    .  89BD 0>mov dword ptr ss:[ebp-F8],edi
005DC208    .  89BD F>mov dword ptr ss:[ebp-108],edi
005DC20E    .  89BD D>mov dword ptr ss:[ebp-128],edi
005DC214    .  89BD C>mov dword ptr ss:[ebp-138],edi
005DC21A    .  89BD A>mov dword ptr ss:[ebp-158],edi
005DC220    .  FFD3   call ebx                      ;  <&MSVBVM60.__vbaStrCopy>
005DC222    .  E8 197>call pkgs910.00673D40          ;  如果走过，注册码就计算过了进入
005DC227    .  8B55 E>mov edx,dword ptr ss:[ebp-18]
005DC22A    .  BE 549>mov esi,pkgs910.00739354
005DC22F    .  8BCE   mov ecx,esi
005DC231    .  FFD3   call ebx
005DC233    .  8B1D F>mov ebx,dword ptr ds:[<&MSVBVM60.#632>]    ;  MSVBVM60.rtcMidCharVar
005DC239    .  8D55 D>lea edx,dword ptr ss:[ebp-28]
005DC23C    .  52     push edx
005DC23D    .  8D85 2>lea eax,dword ptr ss:[ebp-D8]
005DC243    .  6A 02  push 2
005DC245    .  8D4D C>lea ecx,dword ptr ss:[ebp-38]
005DC248    .  50     push eax
005DC249    .  51     push ecx
005DC24A    .  C745 E>mov dword ptr ss:[ebp-20],5
005DC251    .  C745 D>mov dword ptr ss:[ebp-28],2
005DC258    .  89B5 3>mov dword ptr ss:[ebp-D0],esi
005DC25E    .  C785 2>mov dword ptr ss:[ebp-D8],4008
005DC268    .  FFD3   call ebx                      ;  <&MSVBVM60.#632>
005DC26A    .  8D55 A>lea edx,dword ptr ss:[ebp-58]
005DC26D    .  8D85 F>lea eax,dword ptr ss:[ebp-108]
005DC273    .  52     push edx
005DC274    .  6A 09  push 9
005DC276    .  8D4D 9>lea ecx,dword ptr ss:[ebp-68]
005DC279    .  50     push eax
005DC27A    .  51     push ecx
005DC27B    .  C785 1>mov dword ptr ss:[ebp-F0],pkgs910.0042D154
```



```
005DC285    .  C785 0>mov dword ptr ss:\[ebp-F8\],8

005DC28F    .  C745 B>mov dword ptr ss:\[ebp-50\],5

005DC296    .  C745 A>mov dword ptr ss:\[ebp-58\],2

005DC29D    .  89B5 0>mov dword ptr ss:\[ebp-100\],esi

005DC2A3    .  C785 F>mov dword ptr ss:\[ebp-108\],4008

005DC2AD    .  FFD3   call ebx

005DC2AF    .  8D95 6>lea edx,dword ptr ss:\[ebp-98\]

005DC2B5    .  8D85 C>lea eax,dword ptr ss:\[ebp-138\]

005DC2BB    .  52     push edx

005DC2BC    .  6A 11  push 11

005DC2BE    .  8D8D 5>lea ecx,dword ptr ss:\[ebp-A8\]

005DC2C4    .  50     push eax

005DC2C5    .  51     push ecx

005DC2C6    .  C785 E>mov dword ptr ss:\[ebp-120\],pkgs910.0042D15>

005DC2D0    .  C785 D>mov dword ptr ss:\[ebp-128\],8

005DC2DA    .  C785 7>mov dword ptr ss:\[ebp-90\],80020004

005DC2E4    .  C785 6>mov dword ptr ss:\[ebp-98\],0A

005DC2EE    .  89B5 D>mov dword ptr ss:\[ebp-130\],esi

005DC2F4    .  C785 C>mov dword ptr ss:\[ebp-138\],4008

005DC2FE    .  FFD3   call ebx

005DC300    .  8D55 C>lea edx,dword ptr ss:\[ebp-38\]

005DC303    .  52     push edx

005DC304    .  8B35 9>mov esi,dword ptr ds:\[<&MSVBVM60.\_\_vbaVarC>;  MSVBVM60.\_\_vbaVarCat

005DC30A    .  8D85 0>lea eax,dword ptr ss:\[ebp-F8\]

005DC310    .  8D4D B>lea ecx,dword ptr ss:\[ebp-48\]

005DC313    .  50     push eax

005DC314    .  51     push ecx

005DC315    .  FFD6   call esi                                           ;  <&MSVBVM60.\_\_vbaVarCat>

005DC317    .  50     push eax

005DC318    .  8D55 9>lea edx,dword ptr ss:\[ebp-68\]

005DC31B    .  8D45 8>lea eax,dword ptr ss:\[ebp-78\]

005DC31E    .  52     push edx

005DC31F    .  50     push eax

005DC320    .  FFD6   call esi

005DC322    .  8D8D D>lea ecx,dword ptr ss:\[ebp-128\]

005DC328    .  50     push eax

005DC329    .  8D95 7>lea edx,dword ptr ss:\[ebp-88\]

005DC32F    .  51     push ecx

005DC330    .  52     push edx

005DC331    .  FFD6   call esi

005DC333    .  50     push eax

005DC334    .  8D85 5>lea eax,dword ptr ss:\[ebp-A8\]

005DC33A    .  8D8D 4>lea ecx,dword ptr ss:\[ebp-B8\]

005DC340    .  50     push eax

005DC341    .  51     push ecx

005DC342    .  FFD6   call esi

005DC344    .  50     push eax

005DC345    .  FF15 3>call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrVarM>;  MSVBVM60.\_\_vbaStrVarMove

005DC34B    .  8BD0   mov edx,eax

005DC34D    .  B9 989>mov ecx,pkgs910.00739498

005DC352    .  FF15 4>call dword ptr ds:\[<&MSVBVM60.\_\_vbaStrMove>;  MSVBVM60.\_\_vbaStrMove

005DC358    .  8D95 4>lea edx,dword ptr ss:\[ebp-B8\]

005DC35E    .  8D85 5>lea eax,dword ptr ss:\[ebp-A8\]

005DC364    .  52     push edx

005DC365    .  8D8D 7>lea ecx,dword ptr ss:\[ebp-88\]

005DC36B    .  50     push eax

005DC36C    .  8D95 6>lea edx,dword ptr ss:\[ebp-98\]

005DC372    .  51     push ecx

005DC373    .  8D45 8>lea eax,dword ptr ss:\[ebp-78\]

005DC376    .  52     push edx

005DC377    .  8D4D 9>lea ecx,dword ptr ss:\[ebp-68\]

005DC37A    .  50     push eax

005DC37B    .  8D55 B>lea edx,dword ptr ss:\[ebp-48\]

005DC37E    .  51     push ecx

005DC37F    .  8D45 A>lea eax,dword ptr ss:\[ebp-58\]

005DC382    .  52     push edx

005DC383    .  8D4D C>lea ecx,dword ptr ss:\[ebp-38\]

005DC386    .  50     push eax

005DC387    .  8D55 D>lea edx,dword ptr ss:\[ebp-28\]

005DC38A    .  51     push ecx

005DC38B    .  52     push edx

005DC38C    .  6A 0A  push 0A

005DC38E    .  FF15 4>call dword ptr ds:\[<&MSVBVM60.\_\_vbaFreeVar>;  MSVBVM60.\_\_vbaFreeVarList

005DC394    .  A1 989>mov eax,dword ptr ds:\[739498\]

005DC399    .  8B0D B>mov ecx,dword ptr ds:\[7394B0\]

005DC39F    .  83C4 2>add esp,2C

005DC3A2    .  8D55 D>lea edx,dword ptr ss:\[ebp-28\]

005DC3A5    .  8985 B>mov dword ptr ss:\[ebp-150\],eax

005DC3AB    .  898D 3>mov dword ptr ss:\[ebp-D0\],ecx

005DC3B1    .  52     push edx

005DC3B2    .  8D85 2>lea eax,dword ptr ss:\[ebp-D8\]

005DC3B8    .  6A 02  push 2

005DC3BA    .  8D4D C>lea ecx,dword ptr ss:\[ebp-38\]

005DC3BD    .  50     push eax

005DC3BE    .  51     push ecx

005DC3BF    .  C785 A>mov dword ptr ss:\[ebp-158\],8008

005DC3C9    .  C745 E>mov dword ptr ss:\[ebp-20\],5

005DC3D0    .  C745 D>mov dword ptr ss:\[ebp-28\],2

005DC3D7    .  C785 2>mov dword ptr ss:\[ebp-D8\],4008

005DC3E1    .  FFD3   call ebx

005DC3E3    .  8B15 B>mov edx,dword ptr ds:\[7394B0\]

005DC3E9    .  8D45 A>lea eax,dword ptr ss:\[ebp-58\]

005DC3EC    .  50     push eax

005DC3ED    .  8D8D F>lea ecx,dword ptr ss:\[ebp-108\]

005DC3F3    .  8995 0>mov dword ptr ss:\[ebp-100\],edx

005DC3F9    .  6A 09  push 9

005DC3FB    .  C785 1>mov dword ptr ss:\[ebp-F0\],pkgs910.0042D154

005DC405    .  C785 0>mov dword ptr ss:\[ebp-F8\],8

005DC40F    .  C745 B>mov dword ptr ss:\[ebp-50\],5

005DC416    .  C745 A>mov dword ptr ss:\[ebp-58\],2

005DC41D    .  C785 F>mov dword ptr ss:\[ebp-108\],4008

005DC427    .  51     push ecx

005DC428    .  8D55 9>lea edx,dword ptr ss:\[ebp-68\]

005DC42B    .  52     push edx

005DC42C    .  FFD3   call ebx

005DC42E    .  A1 B09>mov eax,dword ptr ds:\[7394B0\]

005DC433    .  8D8D 6>lea ecx,dword ptr ss:\[ebp-98\]

005DC439    .  8985 D>mov dword ptr ss:\[ebp-130\],eax

005DC43F    .  51     push ecx

005DC440    .  8D95 C>lea edx,dword ptr ss:\[ebp-138\]

005DC446    .  6A 11  push 11

005DC448    .  8D85 5>lea eax,dword ptr ss:\[ebp-A8\]

005DC44E    .  52     push edx

005DC44F    .  50     push eax

005DC450    .  C785 E>mov dword ptr ss:\[ebp-120\],pkgs910.0042D15>

005DC45A    .  C785 D>mov dword ptr ss:\[ebp-128\],8

005DC464    .  C785 7>mov dword ptr ss:\[ebp-90\],80020004

005DC46E    .  C785 6>mov dword ptr ss:\[ebp-98\],0A

005DC478    .  C785 C>mov dword ptr ss:\[ebp-138\],4008

005DC482    .  FFD3   call ebx

005DC484    .  8D8D A>lea ecx,dword ptr ss:\[ebp-158\]

005DC48A    .  8D55 C>lea edx,dword ptr ss:\[ebp-38\]

005DC48D    .  51     push ecx

005DC48E    .  8D85 0>lea eax,dword ptr ss:\[ebp-F8\]

005DC494    .  52     push edx

005DC495    .  8D4D B>lea ecx,dword ptr ss:\[ebp-48\]

005DC498    .  50     push eax

005DC499    .  51     push ecx

005DC49A    .  FFD6   call esi

005DC49C    .  50     push eax

005DC49D    .  8D55 9>lea edx,dword ptr ss:\[ebp-68\]

005DC4A0    .  8D45 8>lea eax,dword ptr ss:\[ebp-78\]

005DC4A3    .  52     push edx

005DC4A4    .  50     push eax

005DC4A5    .  FFD6   call esi

005DC4A7    .  8D8D D>lea ecx,dword ptr ss:\[ebp-128\]

005DC4AD    .  50     push eax

005DC4AE    .  8D95 7>lea edx,dword ptr ss:\[ebp-88\]

005DC4B4    .  51     push ecx

005DC4B5    .  52     push edx

005DC4B6    .  FFD6   call esi

005DC4B8    .  50     push eax

005DC4B9    .  8D85 5>lea eax,dword ptr ss:\[ebp-A8\]

005DC4BF    .  8D8D 4>lea ecx,dword ptr ss:\[ebp-B8\]

005DC4C5    .  50     push eax

005DC4C6    .  51     push ecx

005DC4C7    .  FFD6   call esi

005DC4C9    .  50     push eax

005DC4CA    .  FF15 1>call dword ptr ds:\[<&MSVBVM60.\_\_vbaVarTstE>;  输入码与第一次计算出来的值进行比较

005DC4D0    .  8985 9>mov dword ptr ss:\[ebp-168\],eax

005DC4D6    .  8D95 4>lea edx,dword ptr ss:\[ebp-B8\]

005DC4DC    .  8D85 5>lea eax,dword ptr ss:\[ebp-A8\]

005DC4E2    .  52     push edx

005DC4E3    .  8D8D 7>lea ecx,dword ptr ss:\[ebp-88\]

005DC4E9    .  50     push eax

005DC4EA    .  8D95 6>lea edx,dword ptr ss:\[ebp-98\]

005DC4F0    .  51     push ecx

005DC4F1    .  8D45 8>lea eax,dword ptr ss:\[ebp-78\]

005DC4F4    .  52     push edx

005DC4F5    .  8D4D 9>lea ecx,dword ptr ss:\[ebp-68\]

005DC4F8    .  50     push eax

005DC4F9    .  8D55 B>lea edx,dword ptr ss:\[ebp-48\]

005DC4FC    .  51     push ecx

005DC4FD    .  8D45 A>lea eax,dword ptr ss:\[ebp-58\]

005DC500    .  52     push edx
```



```
005DC501     .  8D4D C>lea ecx,dword ptr ss:[ebp-38]  

005DC504     .  50     push eax  

005DC505     .  8D55 D>lea edx,dword ptr ss:[ebp-28]  

005DC508     .  51     push ecx  

005DC509     .  52     push edx  

005DC50A     .  6A 0A  push 0A  

005DC50C     .  FF15 4>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>];  MSVBVM60.__vbaFreeVarList  

005DC512     .  83C4 2>add esp,2C  

005DC515     .  66:39B>cmp word ptr ss:[ebp-168],di  

005DC51C     .  74 0F  je short pkgs910.005DC52D                       ;  如果等于将是 A 类注册  

005DC51E     .  C705 9>mov dword ptr ds:[73949C],1  

005DC528     .  E9 8A0>jmp pkgs910.005DC6B7  

005DC52D     >  8B0D B>mov ecx,dword ptr ds:[7394B0]  

005DC533     .  A1 989>mov eax,dword ptr ds:[739498]  

005DC538     .  83C1 0>add ecx,4  

005DC53B     .  8D55 D>lea edx,dword ptr ss:[ebp-28]  

005DC53E     .  8985 B>mov dword ptr ss:[ebp-150],eax  

005DC544     .  898D 3>mov dword ptr ss:[ebp-D0],ecx  

005DC54A     .  52     push edx  

005DC54B     .  8D85 2>lea eax,dword ptr ss:[ebp-D8]  

005DC551     .  6A 02  push 2  

005DC553     .  8D4D C>lea ecx,dword ptr ss:[ebp-38]  

005DC556     .  BF 084>mov edi,4008  

005DC55B     .  50     push eax  

005DC55C     .  51     push ecx  

005DC55D     .  C785 A>mov dword ptr ss:[ebp-158],8008  

005DC567     .  C745 E>mov dword ptr ss:[ebp-20],5  

005DC56E     .  C745 D>mov dword ptr ss:[ebp-28],2  

005DC575     .  89BD 2>mov dword ptr ss:[ebp-D8],edi  

005DC57B     .  FFD3   call ebx                                   ;  取第二次计算出来的前 5 位值  

005DC57D     .  8B15 B>mov edx,dword ptr ds:[7394B0]  

005DC583     .  8D45 A>lea eax,dword ptr ss:[ebp-58]  

005DC586     .  83C2 0>add edx,4  

005DC589     .  50     push eax  

005DC58A     .  8995 0>mov dword ptr ss:[ebp-100],edx  

005DC590     .  8D8D F>lea ecx,dword ptr ss:[ebp-108]  

005DC596     .  6A 09  push 9  

005DC598     .  8D55 9>lea edx,dword ptr ss:[ebp-68]  

005DC59B     .  51     push ecx  

005DC59C     .  52     push edx  

005DC59D     .  C785 1>mov dword ptr ss:[ebp-F0],pkgs910.0042D154  

005DC5A7     .  C785 0>mov dword ptr ss:[ebp-F8],8  

005DC5B1     .  C745 B>mov dword ptr ss:[ebp-50],5  

005DC5B8     .  C745 A>mov dword ptr ss:[ebp-58],2  

005DC5BF     .  89BD F>mov dword ptr ss:[ebp-108],edi  

005DC5C5     .  FFD3   call ebx                                   ;  取第二次计算出来的中间 5 位值  

005DC5C7     .  A1 B09>mov eax,dword ptr ds:[7394B0]  

005DC5CC     .  8D8D 6>lea ecx,dword ptr ss:[ebp-98]  

005DC5D2     .  83C0 0>add eax,4  

005DC5D5     .  51     push ecx  

005DC5D6     .  8985 D>mov dword ptr ss:[ebp-130],eax  

005DC5DC     .  8D95 C>lea edx,dword ptr ss:[ebp-138]  

005DC5E2     .  6A 11  push 11  

005DC5E4     .  8D85 5>lea eax,dword ptr ss:[ebp-A8]  

005DC5EA     .  52     push edx  

005DC5EB     .  50     push eax  

005DC5EC     .  C785 E>mov dword ptr ss:[ebp-120],pkgs910.0042D15>  

005DC5F6     .  C785 D>mov dword ptr ss:[ebp-128],8  

005DC600     .  C785 7>mov dword ptr ss:[ebp-90],80020004  

005DC60A     .  C785 6>mov dword ptr ss:[ebp-98],0A  

005DC614     .  89BD C>mov dword ptr ss:[ebp-138],edi  

005DC61A     .  FFD3   call ebx                                   ;  取第二次计算出来的后 45 位值  

005DC61C     .  8D8D A>lea ecx,dword ptr ss:[ebp-158]  

005DC622     .  8D55 C>lea edx,dword ptr ss:[ebp-38]  

005DC625     .  51     push ecx  

005DC626     .  8D85 0>lea eax,dword ptr ss:[ebp-F8]  

005DC62C     .  52     push edx  

005DC62D     .  8D4D B>lea ecx,dword ptr ss:[ebp-48]  

005DC630     .  50     push eax  

005DC631     .  51     push ecx  

005DC632     .  FFD6   call esi                                   ;  组合  

005DC634     .  50     push eax  

005DC635     .  8D55 9>lea edx,dword ptr ss:[ebp-68]  

005DC638     .  8D45 8>lea eax,dword ptr ss:[ebp-78]  

005DC63B     .  52     push edx  

005DC63C     .  50     push eax  

005DC63D     .  FFD6   call esi  

005DC63F     .  8D8D D>lea ecx,dword ptr ss:[ebp-128]             ;  组合  

005DC645     .  50     push eax  

005DC646     .  8D95 7>lea edx,dword ptr ss:[ebp-88]  

005DC64C     .  51     push ecx  

005DC64D     .  52     push edx  

005DC64E     .  FFD6   call esi                                   ;  组合  

005DC650     .  50     push eax  

005DC651     .  8D85 5>lea eax,dword ptr ss:[ebp-A8]  

005DC657     .  8D8D 4>lea ecx,dword ptr ss:[ebp-B8]  

005DC65D     .  50     push eax  

005DC65E     .  51     push ecx  

005DC65F     .  FFD6   call esi                                   ;  组合完毕，5 位-5 位-4 位  

005DC661     .  50     push eax  

005DC662     .  FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>;  进行比较了  

……………………………………………………………………………………………………………………  

005DC222     .  E8 197>call pkgs910.00673D40             进入关键点了！！！！     

……………………………………………………………………………………………………………………  

00673D40     $  55     push ebp  

00673D41     .  8BEC   mov ebp,esp  

00673D43     .  83EC 0>sub esp,8  

00673D46     .  68 A67>push <jmp.&MSVBVM60.__vbaExceptHandler>    ;  SE handler installation  

00673D4B     .  64:A1 >mov eax,dword ptr fs:[0]  

00673D51     .  50     push eax  

00673D52     .  64:892>mov dword ptr fs:[0],esp  

00673D59     .  81EC 8>sub esp,180  

00673D5F     .  53     push ebx  

00673D60     .  56     push esi  

00673D61     .  57     push edi  

00673D62     .  8965 F>mov dword ptr ss:[ebp-8],esp  

00673D65     .  C745 F>mov dword ptr ss:[ebp-4],pkgs910.00405DF0  

00673D6C     .  33C0   xor eax,eax  

00673D6E     .  8945 E>mov dword ptr ss:[ebp-1C],eax  

00673D71     .  8945 D>mov dword ptr ss:[ebp-2C],eax  

00673D74     .  8945 C>mov dword ptr ss:[ebp-3C],eax  

00673D77     .  8945 B>mov dword ptr ss:[ebp-4C],eax  

00673D7A     .  8945 A>mov dword ptr ss:[ebp-5C],eax  

00673D7D     .  8945 9>mov dword ptr ss:[ebp-6C],eax  

00673D80     .  8945 8>mov dword ptr ss:[ebp-7C],eax  

00673D83     .  8985 7>mov dword ptr ss:[ebp-8C],eax  

00673D89     .  8985 6>mov dword ptr ss:[ebp-9C],eax  

00673D8F     .  8985 5>mov dword ptr ss:[ebp-AC],eax  

00673D95     .  8985 4>mov dword ptr ss:[ebp-BC],eax  

00673D9B     .  8985 3>mov dword ptr ss:[ebp-CC],eax  

00673DA1     .  8985 2>mov dword ptr ss:[ebp-DC],eax  

00673DA7     .  8985 1>mov dword ptr ss:[ebp-EC],eax  

00673DAD     .  8985 0>mov dword ptr ss:[ebp-FC],eax  

00673DB3     .  8985 F>mov dword ptr ss:[ebp-10C],eax  

00673DB9     .  8985 E>mov dword ptr ss:[ebp-11C],eax  

00673DBF     .  8985 C>mov dword ptr ss:[ebp-13C],eax  

00673DC5     .  8985 B>mov dword ptr ss:[ebp-14C],eax  

00673DCB     .  8985 A>mov dword ptr ss:[ebp-15C],eax  

00673DD1     .  8985 8>mov dword ptr ss:[ebp-17C],eax  

00673DD7     .  A1 489>mov eax,dword ptr ds:[739448]  

00673DDC     .  50     push eax                                   ; /Arg1 => 001544FC  

00673DDD     .  FF15 9>call dword ptr ds:[<&MSVBVM60.#581>]      ; \rtcR8ValFromBstr  

00673DE3     .  DC0D 5>fmul qword ptr ds:[401658]                  ;  机器码与 3 相称  

00673DE9     .  8D4D D>lea ecx,dword ptr ss:[ebp-2C]  

00673DEC     .  C745 D>mov dword ptr ss:[ebp-2C],5  

00673DF3     .  51     push ecx  

00673DF4     .  DC05 E>fadd qword ptr ds:[405DE8]                  ;  结果与 74 想加  

00673DFA     .  DD5D D>fstp qword ptr ss:[ebp-24]                  ;  储存  

00673DFD     .  DFE0   fstsw ax  

00673DFF     .  A8 0D  test al,0D  

00673E01     .  0F85 2>jnz pkgs910.00674531  

00673E07     .  E8 B42>call pkgs910.006765C0              ;  取前 14 位防御 0012f434  

00673E0C     .  8B35 4>mov esi,dword ptr ds:[<&MSVBVM60.__vbaStrM>;  MSVBVM60.__vbaStrMove  

00673E12     .  8BD0   mov edx,eax  

00673E14     .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]  

00673E17     .  FFD6   call esi                                 ;  <&MSVBVM60.__vbaStrMove>  

00673E19     .  8B0D B>mov ecx,dword ptr ds:[7394B0]  

00673E1F     .  8B3D C>mov edi,dword ptr ds:[<&MSVBVM60.__vbaStrC>;  MSVBVM60.__vbaStrCopy  

00673E25     .  8BD0   mov edx,eax  
```



```assembly
00673E27    .   FFD7   call edi                       ; <&MSVBVM60.__vbaStrCopy>
00673E29    .   8B1D 9>mov ebx,dword ptr ds:[<&MSVBVM60.__vbaFree>; MSVBVM60.__vbaFreeStr
00673E2F    .   8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E32    .   FFD3   call ebx                       ; <&MSVBVM60.__vbaFreeStr>
00673E34    .   8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673E37    .   FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>; MSVBVM60.__vbaFreeVar
00673E3D    .   8B15 4>mov edx,dword ptr ds:[739448]        ; 又得到机器码
00673E43    .   52     push edx                       ; /Arg1 => 001544FC
00673E44    .   FF15 9>call dword ptr ds:[<&MSVBVM60.#581>]   ; \转化为实数
00673E4A    .   DC0D 0>fmul qword ptr ds:[401900]            ; 乘以 4
00673E50    .   C745 D>mov dword ptr ss:[ebp-2C],5
00673E57    .   DC25 E>fsub qword ptr ds:[405DE0]            ; 减 47
00673E5D    .   DD5D D>fstp qword ptr ss:[ebp-24]            ; 保存
00673E60    .   DFE0   fstsw ax
00673E62    .   A8 0D  test al,0D
00673E64    .   0F85 C>jnz pkgs910.00674531
00673E6A    .   8D45 D>lea eax,dword ptr ss:[ebp-2C]
00673E6D    .   50     push eax
00673E6E    .   E8 4D2>call pkgs910.006765C0              ; 取刚才算好的前 14 位
00673E73    .   8BD0   mov edx,eax
00673E75    .   8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E78    .   FFD6   call esi
00673E7A    .   8B0D B>mov ecx,dword ptr ds:[7394B0]
00673E80    .   8BD0   mov edx,eax
00673E82    .   83C1 0>add ecx,4
00673E85    .   FFD7   call edi                       ; 在内存中去掉 2 位了，前 2 位
00673E87    .   8D4D E>lea ecx,dword ptr ss:[ebp-1C]
00673E8A    .   FFD3   call ebx
00673E8C    .   8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673E8F    .   FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>; MSVBVM60.__vbaFreeVar
00673E95    .   BE 010>mov esi,1
00673E9A    .   8975 E>mov dword ptr ss:[ebp-14],esi
00673E9D    .   8B1D 8>mov ebx,dword ptr ds:[<&MSVBVM60.__vbaMidS>; MSVBVM60.__vbaMidStmtBstr
00673EA3    >   B8 020>mov eax,2
00673EA8    .   3BF0   cmp esi,eax
00673EAA    .   0F8F 9>jg pkgs910.00674144
00673EB0    .   4E     dec esi
00673EB1    .   3BF0   cmp esi,eax
00673EB3    .   72 06  jb short pkgs910.00673EBB
00673EB5    .   FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>; MSVBVM60.__vbaGenerateBoundsError
00673EBB    >   8B15 B>mov edx,dword ptr ds:[7394B0]
00673EC1    .   8B04B2 mov eax,dword ptr ds:[edx+esi*4]     ; 第一次计算的值
00673EC4    .   50     push eax
00673EC5    .   FF15 3>call dword ptr ds:[<&MSVBVM60.__vbaLenBstr>; 测长度
00673ECB    .   8985 7>mov dword ptr ss:[ebp-18C],eax
00673ED1    .   BF 010>mov edi,1
00673ED6    >   3BBD 7>cmp edi,dword ptr ss:[ebp-18C]
00673EDC    .   0F8F 4>jg pkgs910.0067412A
00673EE2    .   83FE 0>cmp esi,2
00673EE5    .   C745 D>mov dword ptr ss:[ebp-24],1
00673EEC    .   C745 D>mov dword ptr ss:[ebp-2C],2
00673EF3    .   72 06  jb short pkgs910.00673EFB
00673EF5    .   FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>; MSVBVM60.__vbaGenerateBoundsError
00673EFB    >   8B0D B>mov ecx,dword ptr ds:[7394B0]
00673F01    .   8D45 D>lea eax,dword ptr ss:[ebp-2C]
00673F04    .   50     push eax                       ; /Arg4
00673F05    .   57     push edi                       ; |Arg3
00673F06    .   8D14B1 lea edx,dword ptr ds:[ecx+esi*4]     ; |
00673F09    .   8D8D 2>lea ecx,dword ptr ss:[ebp-DC]       ; |
00673F0F    .   8995 2>mov dword ptr ss:[ebp-D4],edx       ; |
00673F15    .   8D55 C>lea edx,dword ptr ss:[ebp-3C]       ; |
00673F18    .   51     push ecx                       ; |Arg2
00673F19    .   52     push edx                       ; |Arg1
00673F1A    .   C785 2>mov dword ptr ss:[ebp-DC],4008       ; |
00673F24    .   FF15 F>call dword ptr ds:[<&MSVBVM60.#632>]   ; \从计算出来的码中一个一个取值
00673F2A    .   8D55 C>lea edx,dword ptr ss:[ebp-3C]
00673F2D    .   8D8D 8>lea ecx,dword ptr ss:[ebp-17C]
00673F33    .   FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarMove>; MSVBVM60.__vbaVarMove
00673F39    .   8D4D D>lea ecx,dword ptr ss:[ebp-2C]
00673F3C    .   FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>; MSVBVM60.__vbaFreeVar
00673F42    .   8D85 8>lea eax,dword ptr ss:[ebp-17C]
00673F48    .   8D8D 2>lea ecx,dword ptr ss:[ebp-DC]
00673F4E    .   50     push eax
00673F4F    .   51     push ecx
00673F50    .   C785 2>mov dword ptr ss:[ebp-D4],pkgs910.0042E128 ; 42E128 的值是 4
00673F5A    .   C785 2>mov dword ptr ss:[ebp-DC],8008
00673F64    .   FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>; 比较是否等于 4
00673F6A    .   66:85C>test ax,ax
00673F6D    .   74 22  je short pkgs910.00673F91
00673F6F    .   83FE 0>cmp esi,2
00673F72    .   72 06  jb short pkgs910.00673F7A
00673F74    .   FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>; MSVBVM60.__vbaGenerateBoundsError
00673F7A    >   8B15 B>mov edx,dword ptr ds:[7394B0]
00673F80    .   8D04B2 lea eax,dword ptr ds:[edx+esi*4]
00673F83    .   50     push eax
00673F84    .   57     push edi
00673F85    .   6A 01  push 1
00673F87    .   68 70D>push pkgs910.0042D170              ; 如果值为 4 将被替换为 d
00673F8C    .   E9 810>jmp pkgs910.00674112
00673F91    >   8D8D 8>lea ecx,dword ptr ss:[ebp-17C]
00673F97    .   8D95 2>lea edx,dword ptr ss:[ebp-DC]
00673F9D    .   51     push ecx
00673F9E    .   52     push edx
00673F9F    .   C785 2>mov dword ptr ss:[ebp-D4],pkgs910.0043ED9C ; 43ED9C 值为 5
00673FA9    .   C785 2>mov dword ptr ss:[ebp-DC],8008
00673FB3    .   FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>; 比较了
00673FB9    .   66:85C>test ax,ax
00673FBC    .   74 21  je short pkgs910.00673FDF
00673FBE    .   83FE 0>cmp esi,2
00673FC1    .   72 06  jb short pkgs910.00673FC9
00673FC3    .   FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>; MSVBVM60.__vbaGenerateBoundsError
00673FC9    >   A1 B09>mov eax,dword ptr ds:[7394B0]
00673FCE    .   8D0CB0 lea ecx,dword ptr ds:[eax+esi*4]
00673FD1    .   51     push ecx
00673FD2    .   57     push edi
00673FD3    .   6A 01  push 1
00673FD5    .   68 089>push pkgs910.00439808              ; 如果值为 5 将被替换为 f
00673FDA    .   E9 330>jmp pkgs910.00674112
00673FDF    >   8D95 8>lea edx,dword ptr ss:[ebp-17C]
00673FE5    .   8D85 2>lea eax,dword ptr ss:[ebp-DC]
00673FEB    .   52     push edx
00673FEC    .   50     push eax
00673FED    .   C785 2>mov dword ptr ss:[ebp-D4],pkgs910.0042F8F8 ; 42F8F8 的值 6 比较
00673FF7    .   C785 2>mov dword ptr ss:[ebp-DC],8008
00674001    .   FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>; 比较了
00674007    .   66:85C>test ax,ax
0067400A    .   74 22  je short pkgs910.0067402E
0067400C    .   83FE 0>cmp esi,2
0067400F    .   72 06  jb short pkgs910.00674017
00674011    .   FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>; MSVBVM60.__vbaGenerateBoundsError
00674017    >   8B0D B>mov ecx,dword ptr ds:[7394B0]
0067401D    .   8D14B1 lea edx,dword ptr ds:[ecx+esi*4]
00674020    .   52     push edx
00674021    .   57     push edi
00674022    .   6A 01  push 1
00674024    .   68 B4D>push pkgs910.0042DAB4              ; 如果值为 6 将被替换为 g
00674029    .   E9 E40>jmp pkgs910.00674112
0067402E    >   8D85 8>lea eax,dword ptr ss:[ebp-17C]
00674034    .   8D8D 2>lea ecx,dword ptr ss:[ebp-DC]
0067403A    .   50     push eax
0067403B    .   51     push ecx
0067403C    .   C785 2>mov dword ptr ss:[ebp-D4],pkgs910.00433FE8 ; 433FE8 的值 7
00674046    .   C785 2>mov dword ptr ss:[ebp-DC],8008
00674050    .   FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>; 与 7 比较
```



```
00674056    .  66:85C>test ax,ax  
00674059    .  74 22  je short pkgs910.0067407D  
0067405B    .  83FE 0>cmp esi,2  
0067405E    .  72 06  jb short pkgs910.00674066  
00674060    .  FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>;  MSVBVM60.__vbaGenerateBoundsError  
00674066    >  8B15 B>mov edx,dword ptr ds:[7394B0]  
0067406C    .  8D04B2 lea eax,dword ptr ds:[edx+esi*4]  
0067406F    .  50     push eax  
00674070    .  57     push edi  
00674071    .  6A 01  push 1  
00674073    .  68 BCD>push pkgs910.0042DABC                 ;  如果值为 7 将被替换为 h  
00674078    .  E9 950>jmp pkgs910.00674112  
0067407D    >  8D8D 8>lea ecx,dword ptr ss:[ebp-17C]  
00674083    .  8D95 2>lea edx,dword ptr ss:[ebp-DC]  
00674089    .  51     push ecx  
0067408A    .  52     push edx  
0067408B    .  C785 2>mov dword ptr ss:[ebp-D4],pkgs910.0042EB4C ;  42EB4C 的值为 0  
00674095    .  C785 2>mov dword ptr ss:[ebp-DC],8008  
0067409F    .  FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>;  与 0 比较  
006740A5    .  66:85C>test ax,ax  
006740A8    .  74 1E  je short pkgs910.006740C8  
006740AA    .  83FE 0>cmp esi,2  
006740AD    .  72 06  jb short pkgs910.006740B5  
006740AF    .  FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>;  MSVBVM60.__vbaGenerateBoundsError  
006740B5    >  A1 B09>mov eax,dword ptr ds:[7394B0]  
006740BA    .  8D0CB0 lea ecx,dword ptr ds:[eax+esi*4]  
006740BD    .  51     push ecx  
006740BE    .  57     push edi  
006740BF    .  6A 01  push 1  
006740C1    .  68 64C>push pkgs910.0042C664                 ;  如果值为 0 将被替换为 p  
006740C6    .  EB 4A  jmp short pkgs910.00674112  
006740C8    >  8D95 8>lea edx,dword ptr ss:[ebp-17C]  
006740CE    .  8D85 2>lea eax,dword ptr ss:[ebp-DC]  
006740D4    .  52     push edx  
006740D5    .  50     push eax  
006740D6    .  C785 2>mov dword ptr ss:[ebp-D4],pkgs910.00430580 ;  值为 1  
006740E0    .  C785 2>mov dword ptr ss:[ebp-DC],8008  
006740EA    .  FF15 1>call dword ptr ds:[<&MSVBVM60.__vbaVarTstE>;  与 1 比较  
006740F0    .  66:85C>test ax,ax  
006740F3    .  74 21  je short pkgs910.00674116  
006740F5    .  83FE 0>cmp esi,2  
006740F8    .  72 06  jb short pkgs910.00674100  
006740FA    .  FF15 0>call dword ptr ds:[<&MSVBVM60.__vbaGenerat>;  MSVBVM60.__vbaGenerateBoundsError  
00674100    >  8B0D B>mov ecx,dword ptr ds:[7394B0]  
00674106    .  8D14B1 lea edx,dword ptr ds:[ecx+esi*4]  
00674109    .  52     push edx  
0067410A    .  57     push edi  
0067410B    .  6A 01  push 1  
0067410D    .  68 A4E>push pkgs910.0043EDA4                 ;  如果值为 1 将被替换为 x  
00674112    >  6A 00  push 0  
00674114    .  FFD3   call ebx                               ;  如果有 4.5.6.7.0.1 中的一个到这里来替换  
00674116    >  B8 010>mov eax,1  
0067411B    .  03C7   add eax,edi  
0067411D    .  0F80 1>jo pkgs910.00674536  
00674123    .  8BF8   mov edi,eax  
00674125    .^ E9 ACF>jmp pkgs910.00673ED6  
0067412A    >  8B4D E>mov ecx,dword ptr ss:[ebp-14]  
0067412D    .  B8 010>mov eax,1  
00674132    .  03C1   add eax,ecx  
00674134    .  0F80 F>jo pkgs910.00674536  
0067413A    .  8945 E>mov dword ptr ss:[ebp-14],eax  
0067413D    .  8BF0   mov esi,eax  
0067413F    .^ E9 5FF>jmp pkgs910.00673EA3  
00674144    >  A1 B09>mov eax,dword ptr ds:[7394B0]  
00674149    .  8D8D 2>lea ecx,dword ptr ss:[ebp-DC]  
0067414F    .  6A 05  push 5                                 ; /Arg3 = 00000005  
00674151    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]          ; |  
00674154    .  BF 080>mov edi,8                              ; |  
00674159    .  BB 084>mov ebx,4008                           ; |  
0067415E    .  51     push ecx                               ; |Arg2  
0067415F    .  52     push edx                               ; |Arg1  
00674160    .  C785 1>mov dword ptr ss:[ebp-E4],pkgs910.00430580 ; |  
0067416A    .  89BD 1>mov dword ptr ss:[ebp-EC],edi          ; |  
00674170    .  8985 2>mov dword ptr ss:[ebp-D4],eax          ; |  
00674176    .  899D 2>mov dword ptr ss:[ebp-DC],ebx          ; |  
0067417C    .  FF15 3>call dword ptr ds:[<&MSVBVM60.#617>]    ; \\取前 5 位值  
00674182    .  A1 B09>mov eax,dword ptr ds:[7394B0]  
00674187    .  8D4D 9>lea ecx,dword ptr ss:[ebp-6C]  
0067418A    .  8985 E>mov dword ptr ss:[ebp-114],eax  
00674190    .  51     push ecx                               ; /Arg4  
00674191    .  8D95 E>lea edx,dword ptr ss:[ebp-11C]         ; |  
00674197    .  6A 06  push 6                                  ; |Arg3 = 00000006  
00674199    .  8D45 8>lea eax,dword ptr ss:[ebp-7C]          ; |  
0067419C    .  BE 54D>mov esi,pkgs910.0042D154               ; |0042d154 是“-”  
006741A1    .  52     push edx                               ; |Arg2  
006741A2    .  50     push eax                               ; |Arg1  
006741A3    .  89B5 0>mov dword ptr ss:[ebp-F4],esi          ; |  
006741A9    .  89BD 0>mov dword ptr ss:[ebp-FC],edi          ; |  
006741AF    .  C785 F>mov dword ptr ss:[ebp-104],pkgs910.0043EDA>; |0043edac 是字母 s  
006741B9    .  89BD F>mov dword ptr ss:[ebp-10C],edi         ; |  
006741BF    .  C745 9>mov dword ptr ss:[ebp-64],5            ; |  
006741C6    .  C745 9>mov dword ptr ss:[ebp-6C],2            ; |  
006741CD    .  899D E>mov dword ptr ss:[ebp-11C],ebx         ; |  
006741D3    .  FF15 F>call dword ptr ds:[<&MSVBVM60.#632>]    ; \\取中间 5 位  
006741D9    .  8B0D B>mov ecx,dword ptr ds:[7394B0]  
006741DF    .  8D95 A>lea edx,dword ptr ss:[ebp-15C]  
006741E5    .  6A 04  push 4                                  ; /Arg3 = 00000004  
006741E7    .  8D85 4>lea eax,dword ptr ss:[ebp-BC]          ; |  
006741ED    .  52     push edx                               ; |Arg2  
006741EE    .  50     push eax                               ; |Arg1  
006741EF    .  89B5 C>mov dword ptr ss:[ebp-134],esi         ; |  
006741F5    .  89BD C>mov dword ptr ss:[ebp-13C],edi         ; |  
006741FB    .  C785 B>mov dword ptr ss:[ebp-144],pkgs910.0043EDB>; |43EDB4 的值 35  
00674205    .  89BD B>mov dword ptr ss:[ebp-14C],edi         ; |  
0067420B    .  898D A>mov dword ptr ss:[ebp-154],ecx         ; |  
00674211    .  899D A>mov dword ptr ss:[ebp-15C],ebx         ; |  
00674217    .  FF15 5>call dword ptr ds:[<&MSVBVM60.#619>]    ; \\取最后 4 位都在堆栈中有  
0067421D    .  8B35 9>mov esi,dword ptr ds:[<&MSVBVM60.__vbaVarC>;  MSVBVM60.__vbaVarCat  
00674223    .  8D8D 1>lea ecx,dword ptr ss:[ebp-EC]  
00674229    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]  
0067422C    .  51     push ecx  
0067422D    .  8D45 C>lea eax,dword ptr ss:[ebp-3C]  
00674230    .  52     push edx  
00674231    .  50     push eax  
00674232    .  FFD6   call esi                               ;  组合 前 5 位前加了一个 1 变成 1hxhgg; <&MSVBVM60.__vbaVarCat>  
00674234    .  8D8D 0>lea ecx,dword ptr ss:[ebp-FC]  
0067423A    .  50     push eax  
0067423B    .  8D55 B>lea edx,dword ptr ss:[ebp-4C]  
0067423E    .  51     push ecx  
0067423F    .  52     push edx  
00674240    .  FFD6   call esi                               ;  组合  1hxhgg-  
00674242    .  50     push eax  
00674243    .  8D85 F>lea eax,dword ptr ss:[ebp-10C]  
00674249    .  8D4D A>lea ecx,dword ptr ss:[ebp-5C]  
0067424C    .  50     push eax  
0067424D    .  51     push ecx  
0067424E    .  FFD6   call esi                               ;  组合 1hxhgg-s 与 S 组合  
00674250    .  50     push eax  
00674251    .  8D55 8>lea edx,dword ptr ss:[ebp-7C]  
00674254    .  8D85 7>lea eax,dword ptr ss:[ebp-8C]  
0067425A    .  52     push edx  
0067425B    .  50     push eax  
0067425C    .  FFD6   call esi                               ;  与中间 5 位组合 1hxhgg-s2x3d9  
```



```
0067425E    .  8D8D C>lea ecx,dword ptr ss:\[ebp-13C\]  

00674264    .  50     push eax  

00674265    .  8D95 6>lea edx,dword ptr ss:\[ebp-9C\]  

0067426B    .  51     push ecx  

0067426C    .  52     push edx  

0067426D    .  FFD6   call esi                       ;  与“-”组合 1hxhgg-s2x3d9-  

0067426F    .  50     push eax  

00674270    .  8D85 B>lea eax,dword ptr ss:\[ebp-14C\]  

00674276    .  8D8D 5>lea ecx,dword ptr ss:\[ebp-AC\]  

0067427C    .  50     push eax  

0067427D    .  51     push ecx  

0067427E    .  FFD6   call esi                       ;  再与 35 组合 1hxhgg-s2x3d9-35  

00674280    .  50     push eax  

00674281    .  8D95 4>lea edx,dword ptr ss:\[ebp-BC\]  

00674287    .  8D85 3>lea eax,dword ptr ss:\[ebp-CC\]  

0067428D    .  52     push edx  

0067428E    .  50     push eax  

0067428F    .  FFD6   call esi                       ;  再组合后 4 位 1hxhgg-s2x3d9-352gdx  

00674291    .  50     push eax  

00674292    .  FF15 3>call dword ptr ds:\[<&MSVBVM60.__vbaStrVarM>;  MSVBVM60.__vbaStrVarMove  

00674298    .  8BD0   mov edx,eax  

0067429A    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]  

0067429D    .  FF15 4>call dword ptr ds:\[<&MSVBVM60.__vbaStrMove>;  MSVBVM60.__vbaStrMove  

006742A3    .  8B0D B>mov ecx,dword ptr ds:\[7394B0\]  

006742A9    .  8BD0   mov edx,eax  

006742AB    .  FF15 C>call dword ptr ds:\[<&MSVBVM60.__vbaStrCopy>;  MSVBVM60.__vbaStrCopy  

006742B1    .  8D4D E>lea ecx,dword ptr ss:\[ebp-1C\]  

006742B4    .  FF15 9>call dword ptr ds:\[<&MSVBVM60.__vbaFreeStr>;  MSVBVM60.__vbaFreeStr  

006742BA    .  8D8D 3>lea ecx,dword ptr ss:\[ebp-CC\]  

006742C0    .  8D95 4>lea edx,dword ptr ss:\[ebp-BC\]  

006742C6    .  51     push ecx  

006742C7    .  8D85 5>lea eax,dword ptr ss:\[ebp-AC\]  

006742CD    .  52     push edx  

006742CE    .  8D8D 6>lea ecx,dword ptr ss:\[ebp-9C\]  

006742D4    .  50     push eax  

006742D5    .  8D95 7>lea edx,dword ptr ss:\[ebp-8C\]  

006742DB    .  51     push ecx  

006742DC    .  8D45 8>lea eax,dword ptr ss:\[ebp-7C\]  

006742DF    .  52     push edx  

006742E0    .  8D4D A>lea ecx,dword ptr ss:\[ebp-5C\]  

006742E3    .  50     push eax  

006742E4    .  8D55 9>lea edx,dword ptr ss:\[ebp-6C\]  

006742E7    .  51     push ecx  

006742E8    .  8D45 B>lea eax,dword ptr ss:\[ebp-4C\]  

006742EB    .  52     push edx  

006742EC    .  8D4D C>lea ecx,dword ptr ss:\[ebp-3C\]  

006742EF    .  50     push eax  

006742F0    .  8D55 D>lea edx,dword ptr ss:\[ebp-2C\]  

006742F3    .  51     push ecx  

006742F4    .  52     push edx  

006742F5    .  6A 0B  push 0B  

006742F7    .  FF15 4>call dword ptr ds:\[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVarList  

006742FD    .  A1 B09>mov eax,dword ptr ds:\[7394B0\]  

00674302    .  83C4 3>add esp,30  

00674305    .  8D8D 2>lea ecx,dword ptr ss:\[ebp-DC\]  

0067430B    .  8D55 D>lea edx,dword ptr ss:\[ebp-2C\]  

0067430E    .  6A 05  push 5                     ; /Arg3 = 00000005  

00674310    .  83C0 0>add eax,4                  ; |  

00674313    .  51     push ecx                   ; |Arg2  

00674314    .  52     push edx                   ; |Arg1  

00674315    .  C785 1>mov dword ptr ss:\[ebp-E4\],pkgs910.00433218 ; |  

0067431F    .  89BD 1>mov dword ptr ss:\[ebp-EC\],edi            ; |  

00674325    .  8985 2>mov dword ptr ss:\[ebp-D4\],eax            ; |  

0067432B    .  899D 2>mov dword ptr ss:\[ebp-DC\],ebx            ; |  

00674331    .  FF15 3>call dword ptr ds:\[<&MSVBVM60.#617>\]      ; \\再取机器码第 2 次计算出来的值的前 5 位  

00674337    .  A1 B09>mov eax,dword ptr ds:\[7394B0\]  

0067433C    .  8D4D 9>lea ecx,dword ptr ss:\[ebp-6C\]  

0067433F    .  83C0 0>add eax,4  

00674342    .  51     push ecx                   ; /Arg4  

00674343    .  8985 E>mov dword ptr ss:\[ebp-114\],eax           ; |  

00674349    .  8D95 E>lea edx,dword ptr ss:\[ebp-11C\]            ; |  

0067434F    .  6A 06  push 6                     ; |Arg3 = 00000006  

00674351    .  8D45 8>lea eax,dword ptr ss:\[ebp-7C\]            ; |  

00674354    .  52     push edx                   ; |Arg2  

00674355    .  50     push eax                   ; |Arg1  

00674356    .  C785 0>mov dword ptr ss:\[ebp-F4\],pkgs910.0042D154 ; |0042D154 的值是“-“  

00674360    .  89BD 0>mov dword ptr ss:\[ebp-FC\],edi            ; |  

00674366    .  C785 F>mov dword ptr ss:\[ebp-104\],pkgs910.0042E86>; |0042e860 的值”y"下面要与他们组合  

00674370    .  89BD F>mov dword ptr ss:\[ebp-10C\],edi           ; |  

00674376    .  C745 9>mov dword ptr ss:\[ebp-64\],5              ; |  

0067437D    .  C745 9>mov dword ptr ss:\[ebp-6C\],2              ; |  

00674384    .  899D E>mov dword ptr ss:\[ebp-11C\],ebx           ; |  

0067438A    .  FF15 F>call dword ptr ds:\[<&MSVBVM60.#632>\]      ; \\取中间 5 位  

00674390    .  8B0D B>mov ecx,dword ptr ds:\[7394B0\]  

00674396    .  C785 C>mov dword ptr ss:\[ebp-134\],pkgs910.0042D15>;  是个“-”  

006743A0    .  83C1 0>add ecx,4  

006743A3    .  89BD C>mov dword ptr ss:\[ebp-13C\],edi  

006743A9    .  C785 B>mov dword ptr ss:\[ebp-144\],pkgs910.0043EDC>;  0043edc0 的值是个“45”  

006743B3    .  89BD B>mov dword ptr ss:\[ebp-14C\],edi  

006743B9    .  898D A>mov dword ptr ss:\[ebp-154\],ecx  

006743BF    .  899D A>mov dword ptr ss:\[ebp-15C\],ebx  

006743C5    .  6A 04  push 4                     ; /Arg3 = 00000004  

006743C7    .  8D95 A>lea edx,dword ptr ss:\[ebp-15C\]            ; |  

006743CD    .  8D85 4>lea eax,dword ptr ss:\[ebp-BC\]             ; |  

006743D3    .  52     push edx                   ; |Arg2  

006743D4    .  50     push eax                   ; |Arg1  

006743D5    .  FF15 5>call dword ptr ds:\[<&MSVBVM60.#619>\]      ; \\取后 4 位  

006743DB    .  8D8D 1>lea ecx,dword ptr ss:\[ebp-EC\]  

006743E1    .  8D55 D>lea edx,dword ptr ss:\[ebp-2C\]  

006743E4    .  51     push ecx  

006743E5    .  8D45 C>lea eax,dword ptr ss:\[ebp-3C\]  

006743E8    .  52     push edx  

006743E9    .  50     push eax  

006743EA    .  FFD6   call esi                   ;  前 5 位与“2”组合 29fg88  

006743EC    .  8D8D 0>lea ecx,dword ptr ss:\[ebp-FC\]  

006743F2    .  50     push eax  

006743F3    .  8D55 B>lea edx,dword ptr ss:\[ebp-4C\]  

006743F6    .  51     push ecx  

006743F7    .  52     push edx  

006743F8    .  FFD6   call esi                   ;  与“-”组合 29fg88-  

006743FA    .  50     push eax  

006743FB    .  8D85 F>lea eax,dword ptr ss:\[ebp-10C\]  

00674401    .  8D4D A>lea ecx,dword ptr ss:\[ebp-5C\]  

00674404    .  50     push eax  

00674405    .  51     push ecx  

00674406    .  FFD6   call esi                   ;  再与 y 组合 29fg88-y  

00674408    .  50     push eax  

00674409    .  8D55 8>lea edx,dword ptr ss:\[ebp-7C\]  

0067440C    .  8D85 7>lea eax,dword ptr ss:\[ebp-8C\]  

00674412    .  52     push edx  

00674413    .  50     push eax  

00674414    .  FFD6   call esi                   ;  再与中间 5 位组合 29fg88-y28dgf  

00674416    .  8D8D C>lea ecx,dword ptr ss:\[ebp-13C\]  

0067441C    .  50     push eax  

0067441D    .  8D95 6>lea edx,dword ptr ss:\[ebp-9C\]  

00674423    .  51     push ecx  

00674424    .  52     push edx  

00674425    .  FFD6   call esi                   ;  与“-”组合 29fg88-y28dgf-  

00674427    .  50     push eax  

00674428    .  8D85 B>lea eax,dword ptr ss:\[ebp-14C\]  

0067442E    .  8D8D 5>lea ecx,dword ptr ss:\[ebp-AC\]  

00674434    .  50     push eax  

00674435    .  51     push ecx  

00674436    .  FFD6   call esi                   ;  与“45”组合 29fg88-y28dgf-45  

00674438    .  50     push eax  

00674439    .  8D95 4>lea edx,dword ptr ss:\[ebp-BC\]  

0067443F    .  8D85 3>lea eax,dword ptr ss:\[ebp-CC\]  
```



```asm
00674445    .  52     push edx  

00674446    .  50     push eax  

00674447    .  FFD6   call esi                              ;  与后 4 位组合 29fg88-y28dgf-45ghp9  

00674449    .  50     push eax  

0067444A    .  FF15 3>call dword ptr ds:[<&MSVBVM60.__vbaStrVarM>;  MSVBVM60.__vbaStrVarMove  

00674450    .  8BD0   mov edx,eax  

00674452    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]  

00674455    .  FF15 4>call dword ptr ds:[<&MSVBVM60.__vbaStrMove>;  MSVBVM60.__vbaStrMove  

0067445B    .  8B0D B>mov ecx,dword ptr ds:[7394B0]  

00674461    .  8BD0   mov edx,eax  

00674463    .  83C1 0>add ecx,4  

00674466    .  FF15 C>call dword ptr ds:[<&MSVBVM60.__vbaStrCopy>;  MSVBVM60.__vbaStrCopy  

0067446C    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]  

0067446F    .  FF15 9>call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>;  MSVBVM60.__vbaFreeStr  

00674475    .  8D95 3>lea edx,dword ptr ss:[ebp-CC]  

0067447B    .  8D85 4>lea eax,dword ptr ss:[ebp-BC]  

00674481    .  52     push edx  

00674482    .  8D8D 5>lea ecx,dword ptr ss:[ebp-AC]  

00674488    .  50     push eax  

00674489    .  8D95 6>lea edx,dword ptr ss:[ebp-9C]  

0067448F    .  51     push ecx  

00674490    .  8D85 7>lea eax,dword ptr ss:[ebp-8C]  

00674496    .  52     push edx  

00674497    .  8D4D 8>lea ecx,dword ptr ss:[ebp-7C]  

0067449A    .  50     push eax  

0067449B    .  8D55 A>lea edx,dword ptr ss:[ebp-5C]  

0067449E    .  51     push ecx  

0067449F    .  8D45 9>lea eax,dword ptr ss:[ebp-6C]  

006744A2    .  52     push edx  

006744A3    .  8D4D B>lea ecx,dword ptr ss:[ebp-4C]  

006744A6    .  50     push eax  

006744A7    .  8D55 C>lea edx,dword ptr ss:[ebp-3C]  

006744AA    .  51     push ecx  

006744AB    .  8D45 D>lea eax,dword ptr ss:[ebp-2C]  

006744AE    .  52     push edx  

006744AF    .  50     push eax  

006744B0    .  6A 0B  push 0B  

006744B2    .  FF15 4>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVarList  

006744B8    .  83C4 3>add esp,30  

006744BB    .  9B     wait  

006744BC    .  68 204>push pkgs910.00674520  

006744C1    .  EB 50  jmp short pkgs910.00674513  

006744C3    .  8D4D E>lea ecx,dword ptr ss:[ebp-1C]  

006744C6    .  FF15 9>call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>;  MSVBVM60.__vbaFreeStr  

006744CC    .  8D8D 3>lea ecx,dword ptr ss:[ebp-CC]  

006744D2    .  8D95 4>lea edx,dword ptr ss:[ebp-BC]  

006744D8    .  51     push ecx  

006744D9    .  8D85 5>lea eax,dword ptr ss:[ebp-AC]  

006744DF    .  52     push edx  

006744E0    .  8D8D 6>lea ecx,dword ptr ss:[ebp-9C]  

006744E6    .  50     push eax  

006744E7    .  8D95 7>lea edx,dword ptr ss:[ebp-8C]  

006744ED    .  51     push ecx  

006744EE    .  8D45 8>lea eax,dword ptr ss:[ebp-7C]  

006744F1    .  52     push edx  

006744F2    .  8D4D 9>lea ecx,dword ptr ss:[ebp-6C]  

006744F5    .  50     push eax  

006744F6    .  8D55 A>lea edx,dword ptr ss:[ebp-5C]  

006744F9    .  51     push ecx  

006744FA    .  8D45 B>lea eax,dword ptr ss:[ebp-4C]  

006744FD    .  52     push edx  

006744FE    .  8D4D C>lea ecx,dword ptr ss:[ebp-3C]  

00674501    .  50     push eax  

00674502    .  8D55 D>lea edx,dword ptr ss:[ebp-2C]  

00674505    .  51     push ecx  

00674506    .  52     push edx  

00674507    .  6A 0B  push 0B  

00674509    .  FF15 4>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVarList  

0067450F    .  83C4 3>add esp,30  

00674512    .  C3     retn  

00674513    >  8D8D 8>lea ecx,dword ptr ss:[ebp-17C]  

00674519    .  FF15 2>call dword ptr ds:[<&MSVBVM60.__vbaFreeVar>;  MSVBVM60.__vbaFreeVar  

0067451F    .  C3     retn  
```

计算过程是这样的：

A 类：用机器码与 3 相乘得到的结果与 74 相加，得到 14 位数字，这 14 位数字分别转换就得到 A 类注册码，

B 类：用机器码与 4 相乘得到的结果与 47 相减，得到 14 位数字，这 14 位数字分别转换就得到 B 类注册码

是这样转换的：分别一个一个检查是否符合 4.5.6.7.0.1 这几个数字，如果是 4 则换成 `d`，如果是 5 换成 `f`，如果是 6 替换成 `g`，如果是 7 替换成 `h`，如果是 0 替换成 `p`，如果是 1 替换成 `x`，然后把他们组合成 20 位的

A 类：前 5 位前加 1，加“-”中间 5 位加 s，在加“-”，在最后 4 位前 35，即可。

B 类：前 5 位前加 2，加“-”中间 5 位加 y，在加“-”，在最后 4 位前 45，即可。

但它只比较 5 位-5 位-4 位，所以有很多的注册码。

```text
4dd466d0e1e0cc123afbe504da2462ca
```
