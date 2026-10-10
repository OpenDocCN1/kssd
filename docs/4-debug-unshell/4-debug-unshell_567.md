

# COOLcryptor V0.2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6999.htm`](https://www.pediy.com/kssd/pediy06/pediy6999.htm)

呵呵，刚才我也看了一下。升级的 COOLcryptor V0.2 比 0.1 版强了点，希望 kongfoo 兄弟能够继续前进！ 

实例本地下载  
  
  
一、Magic Jump ，避开 IAT 加密   
  
  
现在的 COOLcryptor 可以用 ImportREC 追踪层次 3 修复输入表，也能找到 Magic Jump   
  
  
004057C3     61                   popad   
004057C4     8902                 mov dword ptr ds:\[edx\],eax ; kernel32.CloseHandle//正确函数    
004057C6     EB 19                jmp short COOLcryp.004057E1   
  
  
00405858     60                   pushad   
00405859     8B85 D92A4000        mov eax,dword ptr ss:\[ebp+402AD9\]   
0040585F     8BD8                 mov ebx,eax   
00405861     8B40 3C              mov eax,dword ptr ds:\[eax+3C\]   
00405864     03D8                 add ebx,eax   
00405866     33C0                 xor eax,eax   
00405868     8B43 08              mov eax,dword ptr ds:\[ebx+8\]   
0040586B     8985 792C4000        mov dword ptr ss:\[ebp+402C79\],eax   
00405871     BA 0D000000          mov edx,0D   
00405876     F7E2                 mul edx   
00405878     50                   push eax   
00405879     6A 40                push 40   
0040587B     FF95 AD2C4000        call dword ptr ss:\[ebp+402CAD\]   
00405881     8985 F92A4000        mov dword ptr ss:\[ebp+402AF9\],eax   
00405887     33C9                 xor ecx,ecx   
00405889     33FF                 xor edi,edi   
0040588B     8B93 B0000000        mov edx,dword ptr ds:\[ebx+B0\]   
00405891     8B9D D92A4000        mov ebx,dword ptr ss:\[ebp+402AD9\]   
00405897     03D3                 add edx,ebx   
00405899     8B9D F92A4000        mov ebx,dword ptr ss:\[ebp+402AF9\]   
0040589F     8B0411               mov eax,dword ptr ds:\[ecx+edx\]   
004058A2     83F8 00              cmp eax,0   
004058A5     74 39                je short 004058E0   
004058A7     66:8138 E01E         cmp word ptr ds:\[eax\],1EE0   
004058AC     74 32                je short 004058E0   
004058AE     66:8138 8023         cmp word ptr ds:\[eax\],2380   
004058B3     74 2B                je short 004058E0   
004058B5     8338 00              cmp dword ptr ds:\[eax\],0   
004058B8     74 26                je short 004058E0   
004058BA     35 D5887F1E          xor eax,1E7F88D5   
004058BF     8985 30284000        mov dword ptr ss:\[ebp+402830\],eax   
004058C5     51                   push ecx   
004058C6     57                   push edi   
004058C7     8D3C1F               lea edi,dword ptr ds:\[edi+ebx\]   
004058CA     8DB5 2F284000        lea esi,dword ptr ss:\[ebp+40282F\]   
004058D0     FC                   cld   
004058D1     B9 0D000000          mov ecx,0D   
004058D6     F3:A4                rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]   
004058D8     5F                   pop edi   
004058D9     59                   pop ecx   
004058DA     8D041F               lea eax,dword ptr ds:\[edi+ebx\]   
004058DD     890411               mov dword ptr ds:\[ecx+edx\],eax//写入加密地址    
004058E0     83C1 04              add ecx,4   
004058E3     83C7 0D              add edi,0D   
004058E6     3B8D 792C4000        cmp ecx,dword ptr ss:\[ebp+402C79\]   
004058EC     75 B1                jnz short 0040589F   
004058EE     EB 0D                jmp short 004058FD   
004058F0     68 6379E577          push kernel32.CloseHandle   
004058F5     813424 D5887F1E      xor dword ptr ss:\[esp\],1E7F88D5   
004058FC     C3                   retn   
  
  
004058DD     890411               mov dword ptr ds:\[ecx+edx\],eax//把这里 NOP 掉就得到正确的函数了    
  
  
————————————————————————   
二、避开 OEP 处的 Stolen Code，走至 OEP    
  
  
  
00405A85     F3:A4                rep movs byte ptr es:\[edi\],byte ptr ds:\[esi\]//保存原 OEP 数据    
  
00405A90     59                   pop ecx   
00405A91     8BD1                 mov edx,ecx   
00405A93     B0 9A                mov al,9A   
00405A95     5F                   pop edi               ; 00402DF0   
00405A96     AA                   stos byte ptr es:\[edi\]//替换原 6A00 为 9A9A，NOP 掉就正常了    
00405A97     E2 FD                loopd short 00405A96   
00405A99     EB 08                jmp short 00405AA3   
  
  
00405AA7     8B4424 28            mov eax,dword ptr ss:\[esp+28\]   
00405AAB     C1C0 07              rol eax,7   
00405AAE     03C2                 add eax,edx   
00405AB0     EB 05                jmp short 00405AB7   
  
00405AF9     EB 06                jmp short 00405B01   
00405AFB     68 F22D4000          push 402DF2//返回程序的伪 OEP 值    
00405B00     C3                   retn   
00405B01     32C0                 xor al,al   
00405B03     8DBD 9F1F4000        lea edi,dword ptr ss:\[ebp+401F9F\]   
00405B09     B9 6F090000          mov ecx,96F   
00405B0E     AA                   stos byte ptr es:\[edi\]//打扫战场    
00405B0F     E2 FD                loopd short 00405B0E   
00405B11     8DBD 6B2A4000        lea edi,dword ptr ss:\[ebp+402A6B\]   
00405B17     B9 56020000          mov ecx,256   
00405B1C     AA                   stos byte ptr es:\[edi\]//打扫战场    
00405B1D     E2 FD                loopd short 00405B1C   
00405B1F     61                   popad   
00405B20     50                   push eax   
00405B21     33C0                 xor eax,eax   
00405B23     64:FF30              push dword ptr fs:\[eax\]   
00405B26     64:8920              mov dword ptr fs:\[eax\],esp   
00405B29     EB 01                jmp short 00405B2C   
  
异常   
  
004059F8     8F80 B8000000        pop dword ptr ds:\[eax+B8\] ; 00142998//去 00142998 处设断    
004059FE     B8 00000000          mov eax,0   
00405A03     5F                   pop edi   
00405A04     C9                   leave   
00405A05     C3                   retn   
  
00142998     6A 00                push 0//其实是把 00402DF0 处的代码挪到这里执行了      
0014299A     68 F22D4000          push 402DF2//返回程序的伪 OEP 值    
0014299F     C3                   retn //飞向光明之巅！![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)   
    
    
————————————————————————   
  
  
00402DF0     6A 00                push 0 //OEP      
00402DF2     E8 C3020000          call 004030BA   ; jmp to kernel32.GetModuleHandleA   
00402DF7     A3 00104000          mov dword ptr ds:\[401000\],eax   
00402DFC     6A 00                push 0   
00402DFE     68 142E4000          push 402E14   
00402E03     6A 00                push 0   
00402E05     6A 64                push 64   
00402E07     50                   push eax   
00402E08     E8 E3020000          call 004030F0  ; jmp to user32.DialogBoxParamA   
00402E0D     6A 00                push 0   
00402E0F     E8 9A020000          call 004030AE  ; jmp to kernel32.ExitProcess   
  
  
现在一切 OK 了，修复输入表时注意一下这个函数：   
00403132     FF25 FC134000      jmp dword ptr ds:\[4013FC\] ;imagehlp.ImageRvaToSection   
 4dd466d0e1e0cc123afbe504da2462ca