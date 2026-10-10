

# 英雄无敌四之疾风战场汉化过程详解

> 原文：[`www.pediy.com/kssd/pediy12/131902.html`](https://www.pediy.com/kssd/pediy12/131902.html)

首先通过 New Game 字符串跟踪到以下的这个重要函数：  
0071BD50  /$  51                    push ecx  
0071BD51  |.  53                    push ebx  
0071BD52  |.  55                    push ebp  
0071BD53  |.  8B6C24 1C             mov ebp,dword ptr ss:\[esp+1C\]  
0071BD57  |.  894C24 08             mov dword ptr ss:\[esp+8\],ecx  
0071BD5B  |.  B3 01                 mov bl,1  
0071BD5D  |.  8A45 00               mov al,byte ptr ss:\[ebp\]  
0071BD60  |.  84C0                  test al,al  
0071BD62  |.  74 69                 je short heroes4.0071BDCD  
0071BD64  |.  56                    push esi  
0071BD65  |.  57                    push edi  
0071BD66  |.  8B7C24 1C             mov edi,dword ptr ss:\[esp+1C\]  
0071BD6A  |.  EB 04                 jmp short heroes4.0071BD70  
0071BD6C  |>  8B4C24 10             /mov ecx,dword ptr ss:\[esp+10\]  
0071BD70  |>  8B51 08                mov edx,dword ptr ds:\[ecx+8\]  
0071BD73  |.  25 FF000000           |and eax,0FF  
0071BD78  |.  2BC2                  |sub eax,edx  
0071BD7A  |.  78 05                 |js short heroes4.0071BD81  
0071BD7C  |.  3B41 14               |cmp eax,dword ptr ds:\[ecx+14\]  
0071BD7F  |.  7C 02                 |jl short heroes4.0071BD83  
0071BD81  |>  33C0                  |xor eax,eax  
0071BD83  |>  8B49 18               |mov ecx,dword ptr ds:\[ecx+18\]  
0071BD86  |.  8D1480                |lea edx,dword ptr ds:\[eax+eax\*4\]          
0071BD89  |.  84DB                  |test bl,bl  
0071BD8B  |.  8D0450                |lea eax,dword ptr ds:\[eax+edx\*2\]  
0071BD8E  |.  8D3481                |lea esi,dword ptr ds:\[ecx+eax\*4\]  ;  字符编码基地址存到 esi  
0071BD91  |.  75 03                 |jnz short heroes4.0071BD96  
0071BD93  |.  037E 14               |add edi,dword ptr ds:\[esi+14\]  
0071BD96  |>  8B5424 30             |mov edx,dword ptr ss:\[esp+30\]  
0071BD9A  |.  8B4424 2C             |mov eax,dword ptr ss:\[esp+2C\]  
0071BD9E  |.  8B4C24 28             |mov ecx,dword ptr ss:\[esp+28\]  
0071BDA2  |.  52                    |push edx                                ; /Arg6  
0071BDA3  |.  8B5424 24             |mov edx,dword ptr ss:\[esp+24\]           ; |  
0071BDA7  |.  50                    |push eax                                ; |Arg5  
0071BDA8  |.  8B4424 20             |mov eax,dword ptr ss:\[esp+20\]           ; |  
0071BDAC  |.  51                    |push ecx                                ; |Arg4  
0071BDAD  |.  52                    |push edx                                ; |Arg3  
0071BDAE  |.  57                    |push edi                                ; |Arg2  
0071BDAF  |.  50                    |push eax                                ; |Arg1  
0071BDB0  |.  8BCE                  |mov ecx,esi                             ; |  
0071BDB2  |.  32DB                  |xor bl,bl                               ; |  
0071BDB4  |.  E8 67FAFFFF           |call heroes4.0071B820                   ; \\字库读取函数  
0071BDB9  |.  8B4E 18               |mov ecx,dword ptr ds:\[esi+18\]  
0071BDBC  |.  8B46 04               |mov eax,dword ptr ds:\[esi+4\]  
0071BDBF  |.  03C8                  |add ecx,eax  
0071BDC1  |.  8A45 01               |mov al,byte ptr ss:\[ebp+1\]  
0071BDC4  |.  03F9                  |add edi,ecx  
0071BDC6  |.  45                    |inc ebp  
0071BDC7  |.  84C0                  |test al,al  
0071BDC9  |.^ 75 A1                 \\jnz short heroes4.0071BD6C  
0071BDCB  |.  5F                    pop edi  
0071BDCC  |.  5E                    pop esi  
0071BDCD  |>  5D                    pop ebp  
0071BDCE  |.  5B                    pop ebx  
0071BDCF  |.  59                    pop ecx  
0071BDD0  \\.  C2 1C00               retn 1C  
  
edx 为字符编码基地址，每个字符占 2C 个字节，  
edx+0 标志位 0x988798  
edx+4 表示字宽  
edx+8 表示字高  
edx+C 字宽  
edx+10 表示该字符点阵在内存的开始地址，  
edx+14 表示该字符前面的间距  
edx+18 表示该字符后面的间距  
edx+1c 一直为 2  
edx+20 表示该字符点阵在内存的开始地址  
edx+24 表示该字符点阵在内存的结束地址  
edx+28 表示该字符点阵在内存的结束地址  
  
  
以下是英雄无敌四风之战争的汉化分析：  
  
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
00 00 40 00 00 00 00 00 00 00 00 00 00 30 00  
90 F0 F0 54 00 00 00 00 00 00 00 00 F0 F0 03   
B0 F0 F0 F0 45 00 00 00 00 00 00 30 F0 DF 0F   
90 F0 F0 F0 EF 24 00 00 00 00 00 30 F0 BF 0D  
70 F0 EF F0 F0 EE 22 00 00 00 00 30 F0 BF 0B   
70 F0 4F 9E F0 F0 EE 22 00 00 00 30 F0 AF 0B   
70 F0 3F 04 99 F0 F0 EE 22 00 00 30 F0 7F 0A   
70 F0 3F 03 00 99 F0 F0 EE 22 00 30 F0 7F 07   
70 F0 3F 03 00 00 99 F0 F0 EE 22 30 F0 7F 07   
70 F0 3F 03 00 00 00 99 F0 F0 EE 62 F0 7F 07   
70 F0 3F 03 00 00 00 00 99 F0 F0 F0 F0 7F 07   
90 F0 3F 03 00 00 00 00 00 99 F0 F0 F0 7F 07   
B0 F0 3F 03 00 00 00 00 00 00 99 F0 F0 7F 07    
D0 9B 1F 03 00 00 00 00 00 00 00 89 AF 1F 07   
10 0D 09 01 00 00 00 00 00 00 00 00 08 0A 01   
00 01 00 00 00 00 00 00 00 00 00 00 00 00 00   
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   
这是本人从内存中导出字符 N 的点阵字库，一个字节表示一个点，数值越大，颜色越深  
  
  
  
  
  
参数表  
arg1:\[arg1+10\]+\[arg1+C\]\*arg3+arg2\*2 存放缓冲区(应该就是屏幕显示区在内存的映像)基地址  
arg2:字符串宽度 x  
arg3:y  
arg4:颜色  
arg5:与颜色深度有关  
arg6:与颜色深度有关  
ecx:字符的编码基地址  
  
0071B820  /$  83EC 54               sub esp,54  
0071B823  |.  53                    push ebx  
0071B824  |.  8B5C24 5C             mov ebx,dword ptr ss:\[esp+5C\]  
0071B828  |.  55                    push ebp  
0071B829  |.  56                    push esi  
0071B82A  |.  8B43 08               mov eax,dword ptr ds:\[ebx+8\]  
0071B82D  |.  8BF1                  mov esi,ecx                       ;This 指针  
0071B82F  |.  8B4B 04               mov ecx,dword ptr ds:\[ebx+4\]  
0071B832  |.  57                    push edi  
0071B833  |.  50                    push eax  
0071B834  |.  51                    push ecx  
0071B835  |.  6A 00                 push 0  
0071B837  |.  6A 00                 push 0  
0071B839  |.  8D4C24 54             lea ecx,dword ptr ss:\[esp+54\]  
0071B83D  |.  897424 20             mov dword ptr ss:\[esp+20\],esi  
0071B841  |.  E8 6ABCCEFF           call heroes4.004074B0  
0071B846  |.  8B56 08               mov edx,dword ptr ds:\[esi+8\]  
0071B849  |.  8B46 04               mov eax,dword ptr ds:\[esi+4\]  
0071B84C  |.  52                    push edx  
0071B84D  |.  50                    push eax  
0071B84E  |.  6A 00                 push 0  
0071B850  |.  6A 00                 push 0  
0071B852  |.  8D4C24 44             lea ecx,dword ptr ss:\[esp+44\]  
0071B856  |.  E8 55BCCEFF           call heroes4.004074B0  
0071B85B  |.  8B4C24 34             mov ecx,dword ptr ss:\[esp+34\]  
0071B85F  |.  8B5424 38             mov edx,dword ptr ss:\[esp+38\]  
0071B863  |.  8B4424 3C             mov eax,dword ptr ss:\[esp+3C\]  
0071B867  |.  894C24 24             mov dword ptr ss:\[esp+24\],ecx  
0071B86B  |.  8B4C24 40             mov ecx,dword ptr ss:\[esp+40\]  
0071B86F  |.  895424 28             mov dword ptr ss:\[esp+28\],edx  
0071B873  |.  8D5424 6C             lea edx,dword ptr ss:\[esp+6C\]  
0071B877  |.  894C24 30             mov dword ptr ss:\[esp+30\],ecx  
0071B87B  |.  52                    push edx  
0071B87C  |.  8D4C24 28             lea ecx,dword ptr ss:\[esp+28\]  
0071B880  |.  894424 30             mov dword ptr ss:\[esp+30\],eax  
0071B884  |.  E8 47BCCEFF           call heroes4.004074D0  
0071B889  |.  8B4424 30             mov eax,dword ptr ss:\[esp+30\]  
0071B88D  |.  8B4C24 50             mov ecx,dword ptr ss:\[esp+50\]  
0071B891  |.  3BC1                  cmp eax,ecx  
0071B893  |.  8BF8                  mov edi,eax  
0071B895  |.  7C 02                 jl short heroes4.0071B899  
0071B897  |.  8BF9                  mov edi,ecx  
0071B899  |>  8B4424 2C             mov eax,dword ptr ss:\[esp+2C\]  
0071B89D  |.  8B5424 4C             mov edx,dword ptr ss:\[esp+4C\]  
0071B8A1  |.  3BC2                  cmp eax,edx  
0071B8A3  |.  7D 02                 jge short heroes4.0071B8A7  
0071B8A5  |.  8BD0                  mov edx,eax  
0071B8A7  |>  8B4424 28             mov eax,dword ptr ss:\[esp+28\]  
0071B8AB  |.  8B4C24 48             mov ecx,dword ptr ss:\[esp+48\]  
0071B8AF  |.  3BC1                  cmp eax,ecx  
0071B8B1  |.  7E 02                 jle short heroes4.0071B8B5  
0071B8B3  |.  8BC8                  mov ecx,eax  
0071B8B5  |>  8B4424 24             mov eax,dword ptr ss:\[esp+24\]  
0071B8B9  |.  8B7424 44             mov esi,dword ptr ss:\[esp+44\]  
0071B8BD  |.  3BC6                  cmp eax,esi  
0071B8BF  |.  7F 02                 jg short heroes4.0071B8C3  
0071B8C1  |.  8BC6                  mov eax,esi  
0071B8C3  |>  57                    push edi  
0071B8C4  |.  52                    push edx  
0071B8C5  |.  51                    push ecx  
0071B8C6  |.  50                    push eax  
0071B8C7  |.  8D4C24 64             lea ecx,dword ptr ss:\[esp+64\]  
0071B8CB  |.  E8 E0BBCEFF           call heroes4.004074B0  
0071B8D0  |.  8B4424 60             mov eax,dword ptr ss:\[esp+60\]  
0071B8D4  |.  8B7424 58             mov esi,dword ptr ss:\[esp+58\]  
0071B8D8  |.  2BC6                  sub eax,esi  
0071B8DA  |.  85C0                  test eax,eax  
0071B8DC  |.  894424 18             mov dword ptr ss:\[esp+18\],eax  
0071B8E0  |.  0F8E B0010000         jle heroes4.0071BA96  
0071B8E6  |.  8B6C24 5C             mov ebp,dword ptr ss:\[esp+5C\]  
0071B8EA  |.  8B4C24 54             mov ecx,dword ptr ss:\[esp+54\]     ;  数值上等于 arg2  
0071B8EE  |.  2BE9                  sub ebp,ecx  
0071B8F0  |.  85ED                  test ebp,ebp  
0071B8F2  |.  896C24 1C             mov dword ptr ss:\[esp+1C\],ebp  
0071B8F6  |.  0F8E 9A010000         jle heroes4.0071BA96  
0071B8FC  |.  8B4424 6C             mov eax,dword ptr ss:\[esp+6C\]     ;  arg2  
0071B900  |.  8BF9                  mov edi,ecx  
0071B902  |.  2BF8                  sub edi,eax  
0071B904  |.  8B4424 70             mov eax,dword ptr ss:\[esp+70\]     ;  arg3  
0071B908  |.  8BD6                  mov edx,esi  
0071B90A  |.  894C24 24             mov dword ptr ss:\[esp+24\],ecx  
0071B90E  |.  2BD0                  sub edx,eax  
0071B910  |.  8B43 0C               mov eax,dword ptr ds:\[ebx+C\]  
0071B913  |.  8BD8                  mov ebx,eax  
0071B915  |.  0FAFDE                imul ebx,esi  
0071B918  |.  8D344B                lea esi,dword ptr ds:\[ebx+ecx\*2\]  
0071B91B  |.  8B4C24 68             mov ecx,dword ptr ss:\[esp+68\]  
0071B91F  |.  8B59 10               mov ebx,dword ptr ds:\[ecx+10\]  
0071B922  |.  8B4C24 10             mov ecx,dword ptr ss:\[esp+10\]  
0071B926  |.  03F3                  add esi,ebx  
0071B928  |.  8B49 0C               mov ecx,dword ptr ds:\[ecx+C\]  
0071B92B  |.  897424 68             mov dword ptr ss:\[esp+68\],esi;  重点：将缓冲区地址写入栈内（原 arg1 处）  
0071B92F  |.  8BD9                  mov ebx,ecx  
0071B931  |.  2BCD                  sub ecx,ebp  
0071B933  |.  0FAFDA                imul ebx,edx  
0071B936  |.  8B5424 10             mov edx,dword ptr ss:\[esp+10\]  
0071B93A  |.  894C24 20             mov dword ptr ss:\[esp+20\],ecx  
0071B93E  |.  8D4C2D 00             lea ecx,dword ptr ss:\[ebp+ebp\]  
0071B942  |.  035A 10               add ebx,dword ptr ds:\[edx+10\]  
0071B945  |.  8B5424 18             mov edx,dword ptr ss:\[esp+18\]  
0071B949  |.  2BC1                  sub eax,ecx  
0071B94B  |.  894424 24             mov dword ptr ss:\[esp+24\],eax  
0071B94F  |.  03DF                  add ebx,edi  
0071B951  |.  8BC2                  mov eax,edx  
0071B953  |.  4A                    dec edx  
0071B954  |.  85C0                  test eax,eax  
0071B956  |.  0F84 3A010000         je heroes4.0071BA96  
0071B95C  |.  42                    inc edx  
0071B95D  |.  895424 18             mov dword ptr ss:\[esp+18\],edx  
0071B961  |>  85ED                  /test ebp,ebp  
0071B963  |.  0F84 0E010000         |je heroes4.0071BA77  
0071B969  |.  8B4424 74             |mov eax,dword ptr ss:\[esp+74\]  
0071B96D  |.  896C24 14             |mov dword ptr ss:\[esp+14\],ebp  
0071B971  |.  25 FFFF0000           |and eax,0FFFF  
0071B976  |.  894424 10             |mov dword ptr ss:\[esp+10\],eax  
0071B97A  |>  8A4424 78             |/mov al,byte ptr ss:\[esp+78\]  
0071B97E  |.  8B2D DCF0AA00         ||mov ebp,dword ptr ds:\[AAF0DC\]  
0071B984  |.  84C0                  ||test al,al  
0071B986  |.  74 6A                 ||je short heroes4.0071B9F2  
0071B988  |.  8A0B                  ||mov cl,byte ptr ds:\[ebx\]        ;  取字模的像素点  
0071B98A  |.  8B4424 7C             ||mov eax,dword ptr ss:\[esp+7C\]  
0071B98E  |.  83E1 0F               ||and ecx,0F                      ;  取低四位  
0071B991  |.  25 FFFF0000           ||and eax,0FFFF  
0071B996  |.  8BF9                  ||mov edi,ecx  
0071B998  |.  33C9                  ||xor ecx,ecx  
0071B99A  |.  66:8B0E               ||mov cx,word ptr ds:\[esi\]        ;  取缓冲区的原值  
0071B99D  |.  85FF                  ||test edi,edi                    ;  检查字模的像素点低四位是否为 0  
0071B99F  |.  74 54                 ||je short heroes4.0071B9F5       ;  为 0 则不作处理  
0071B9A1  |.  8BD7                  ||mov edx,edi  
0071B9A3  |.  8BF1                  ||mov esi,ecx  
0071B9A5  |.  C1FA 03               ||sar edx,3  
0071B9A8  |.  83E2 01               ||and edx,1  
0071B9AB  |.  03FA                  ||add edi,edx  
0071B9AD  |.  8BD1                  ||mov edx,ecx  
0071B9AF  |.  8B0D 04F1AA00         ||mov ecx,dword ptr ds:\[AAF104\]  
0071B9B5  |.  23D5                  ||and edx,ebp  
0071B9B7  |.  23F1                  ||and esi,ecx  
0071B9B9  |.  8BC8                  ||mov ecx,eax  
0071B9BB  |.  230D 04F1AA00         ||and ecx,dword ptr ds:\[AAF104\]  
0071B9C1  |.  23C5                  ||and eax,ebp  
0071B9C3  |.  2BC2                  ||sub eax,edx  
0071B9C5  |.  2BCE                  ||sub ecx,esi  
0071B9C7  |.  0FAFC7                ||imul eax,edi  
0071B9CA  |.  0FAFCF                ||imul ecx,edi  
0071B9CD  |.  C1E6 04               ||shl esi,4  
0071B9D0  |.  03CE                  ||add ecx,esi  
0071B9D2  |.  8B35 F4F0AA00         ||mov esi,dword ptr ds:\[AAF0F4\]  
0071B9D8  |.  C1E2 04               ||shl edx,4  
0071B9DB  |.  23CE                  ||and ecx,esi  
0071B9DD  |.  8B35 D8F0AA00         ||mov esi,dword ptr ds:\[AAF0D8\]  
0071B9E3  |.  03C2                  ||add eax,edx  
0071B9E5  |.  23C6                  ||and eax,esi  
0071B9E7  |.  8B7424 68             ||mov esi,dword ptr ss:\[esp+68\]  
0071B9EB  |.  03C8                  ||add ecx,eax  
0071B9ED  |.  C1F9 04               ||sar ecx,4  
0071B9F0  |.  EB 03                 ||jmp short heroes4.0071B9F5  
0071B9F2  |>  66:8B0E               ||mov cx,word ptr ds:\[esi\]  
0071B9F5  |>  33D2                  ||xor edx,edx  
0071B9F7  |.  81E1 FFFF0000         ||and ecx,0FFFF  
0071B9FD  |.  8A13                  ||mov dl,byte ptr ds:\[ebx\]  
0071B9FF  |.  8BC1                  ||mov eax,ecx  
0071BA01  |.  C1EA 04               ||shr edx,4                       ;  取高四位  
0071BA04  |.  85D2                  ||test edx,edx                    ;  检查字模的像素点高四位是否为 0  
0071BA06  |.  74 51                 ||je short heroes4.0071BA59       ;  为 0 则不作处理  
0071BA08  |.  8BCA                  ||mov ecx,edx  
0071BA0A  |.  8BF0                  ||mov esi,eax  
0071BA0C  |.  C1F9 03               ||sar ecx,3  
0071BA0F  |.  83E1 01               ||and ecx,1  
0071BA12  |.  23F5                  ||and esi,ebp  
0071BA14  |.  03D1                  ||add edx,ecx  
0071BA16  |.  8BC8                  ||mov ecx,eax  
0071BA18  |.  8B4424 10             ||mov eax,dword ptr ss:\[esp+10\]  
0071BA1C  |.  8B3D 04F1AA00         ||mov edi,dword ptr ds:\[AAF104\]  
0071BA22  |.  23C5                  ||and eax,ebp  
0071BA24  |.  23CF                  ||and ecx,edi  
0071BA26  |.  2BC6                  ||sub eax,esi  
0071BA28  |.  0FAFC2                ||imul eax,edx  
0071BA2B  |.  C1E6 04               ||shl esi,4  
0071BA2E  |.  03C6                  ||add eax,esi  
0071BA30  |.  8B35 D8F0AA00         ||mov esi,dword ptr ds:\[AAF0D8\]  
0071BA36  |.  23C6                  ||and eax,esi  
0071BA38  |.  8B7424 10             ||mov esi,dword ptr ss:\[esp+10\]  
0071BA3C  |.  23F7                  ||and esi,edi  
0071BA3E  |.  8B3D F4F0AA00         ||mov edi,dword ptr ds:\[AAF0F4\]  
0071BA44  |.  2BF1                  ||sub esi,ecx  
0071BA46  |.  0FAFF2                ||imul esi,edx  
0071BA49  |.  C1E1 04               ||shl ecx,4  
0071BA4C  |.  03F1                  ||add esi,ecx  
0071BA4E  |.  23F7                  ||and esi,edi  
0071BA50  |.  03C6                  ||add eax,esi  
0071BA52  |.  8B7424 68             ||mov esi,dword ptr ss:\[esp+68\]  
0071BA56  |.  C1F8 04               ||sar eax,4  
0071BA59  |>  66:8906               ||mov word ptr ds:\[esi\],ax        ;  将经过处理后的缓冲区值放回原处  
0071BA5C  |.  8B4424 14             ||mov eax,dword ptr ss:\[esp+14\]  
0071BA60  |.  43                    ||inc ebx                         ;  指向字模的下个像素点  
0071BA61  |.  83C6 02               ||add esi,2                       ;  指向缓冲区的下个点  
0071BA64  |.  48                    ||dec eax                         ;  计数值减 1  
0071BA65  |.  897424 68             ||mov dword ptr ss:\[esp+68\],esi  
0071BA69  |.  894424 14             ||mov dword ptr ss:\[esp+14\],eax  
0071BA6D  |.^ 0F85 07FFFFFF         |\\jnz heroes4.0071B97A  
0071BA73  |.  8B6C24 1C             |mov ebp,dword ptr ss:\[esp+1C\]  
0071BA77  |>  8B5424 20             |mov edx,dword ptr ss:\[esp+20\]  
0071BA7B  |.  8B4C24 24             |mov ecx,dword ptr ss:\[esp+24\]  
0071BA7F  |.  8B4424 18             |mov eax,dword ptr ss:\[esp+18\]  
0071BA83  |.  03DA                  |add ebx,edx                      ;  指向字模的下行像素点  
0071BA85  |.  03F1                  |add esi,ecx                      ;  指向缓冲区的下行点  
0071BA87  |.  48                    |dec eax                          ;  计数值减 1  
0071BA88  |.  897424 68             |mov dword ptr ss:\[esp+68\],esi  
0071BA8C  |.  894424 18             |mov dword ptr ss:\[esp+18\],eax  
0071BA90  |.^ 0F85 CBFEFFFF         \\jnz heroes4.0071B961  
0071BA96  |>  5F                    pop edi  
0071BA97  |.  5E                    pop esi  
0071BA98  |.  5D                    pop ebp  
0071BA99  |.  5B                    pop ebx  
0071BA9A  |.  83C4 54               add esp,54  
0071BA9D  \\.  C2 1800               retn 18  
本函数的核心是两层循环，从 71B9A1 开始就是循环体，整个换算过程是一个比较复杂的算法，可以简单表示为：  
  
buf\[x\]\[y\]=func(buf\[x\]\[y\],zm\[x/2\]\[y/2\])  
  
buf：缓冲区二维数组，两个字节表示一个点  
zm:字模二维数组，一个字节表示一个点  
  
分析至此为止，下面是汉化的大致流程：  
1.先从网上下载字库工具制作字库文件，格式为一个字节表示一个点，字体大小可以自己选择，  
本人选的是 16X16  
2.修改原文件 heroes4.exe 的入口地址，开辟两段内存放字库和字符编码，并用全局变量保存  
这两个指针  
3.修改字库读取函数，把原来的字库和字符编码改成我们自己的。  
（此游戏现已汉化成功）下面的附件是本人修改过的主文件和增加的 dll 文件、字库文件：  
附件：由于文件过大，有兴趣可以在以下网址下载附件。文件名称为 H4 汉化版  
http://jinxin8866.ys168.com/ 

有了下面的东西，汉化就没那么复杂了。。。。。至少可以参考![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")  
  
How to build the Heroes 4 executible  
  
You will need Microsoft Visual C++ 6.0 installed on your machine.  If you have not upgraded it with service pack 4 you will need to do so, this has been provided for you, see folder VC60\_sp4.  
  
There are several other folders included here that will need to be copied to your local machine.    
  
The folder ‘mssdk’ - must be copied to your c:\\ and it must be at the root level.  
  Final path  “C:\\mssdk”  
  
The folder ‘Program files’ contains a folder called ‘Stingray’  it must be in a folder called ‘Program files’ that is at the root level on your c:\\.    
Final path  “C:\\Program files\\Stingray”  
  
The folder ‘Work’ contains all the source code etc that will be needed to build the game, campaign editor and the resource editor.  It should be copied along with its sub-directories to your c:\\.  
  Final path  “C:\\Work”  
  
Once these directories have been established start Visual C++ 6.0.  From the file menu select ‘Open Workspace.’  Navigate to c:\\work\\game it should show the ‘heroes4.dsw’ workspace file.  Select it and open it.