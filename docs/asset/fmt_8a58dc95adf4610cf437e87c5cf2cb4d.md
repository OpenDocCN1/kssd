
```
004160EA    8B9424 94010000 mov edx,dword ptr ss:[esp+194]  

004160F1    0FBF88 D0D65900 movsx ecx,word ptr ds:[eax+59D6D0]  

004160F8    890A            mov dword ptr ds:[edx],ecx  

004160FA    8B8C24 98010000 mov ecx,dword ptr ss:[esp+198]  

00416101    0FBF80 D2D65900 movsx eax,word ptr ds:[eax+59D6D2]  

00416108    8901            mov dword ptr ds:[ecx],eax  

0041610A    B0 01           mov al,1  

0041610C    5B              pop ebx  

0041610D    81C4 80010000   add esp,180  

00416113    C3              retn  
```

这里需要对 `EDX` 的地址设置内存访问断点进行数据处理追踪！

```
0040828A    8B7424 24             mov esi,dword ptr ss:[esp+24]  

0040828E    8B4C24 20             mov ecx,dword ptr ss:[esp+20]  

00408292    83FE FF               cmp esi,-1  

00408295    75 09                 jnz short pj.004082A0  

00408648    8B4C24 20             mov ecx,dword ptr ss:[esp+20]  

0040864C    8B5424 24             mov edx,dword ptr ss:[esp+24]  

00408650    6A 00                 push 0  

00408652    6A 00                 push 0  

00408654    6A 00                 push 0  

00408656    51                    push ecx  

00408657    52                    push edx  

00408658    53                    push ebx  

00408659    56                    push esi  

0040865A    55                    push ebp  

0040865B    E8 10040000           call pj.00408A70  

00408660    83C4 20               add esp,20  

00408663    56                    push esi  

00408664    55                    push ebp  

00408665    E8 66510000           call pj.0040D7D0  

0040866A    83C4 08               add esp,8  

0040866D    85C0                  test eax,eax  

0040866F    0F85 AA010000         jnz pj.0040881F  
```

\===================================================================================================

登陆部分的明文修改...

\--------------------------------------发送明文-------------------------------------------

```
00435913    8B9C24 94100000       mov ebx,dword ptr ss:[esp+1094]  

0043591A    8D4424 10             lea eax,dword ptr ss:[esp+10]  

0043591E    53                    push ebx                           ;这里的数据还没有加密  

0043591F    50                    push eax  

00435920    E8 DBFCFFFF           call pj.00435600                   ;加密函数  

00435925    8B15 BC845900         mov edx,dword ptr ds:[5984BC]  

0043592B    6A00                  push 0  

0043592D    8D4C241C              lea ecx,dword ptr ss:[esp+1C]  

00435931    6A50                  push 50  

00435933    51                    push ecx  

00435934    52                    push edx  

00435935    E8 06F2FFFF           call pj.00434B40                   ;发送函数  

0043593A    83C4 18               add esp,18  

0043593D    83F8 50               cmp eax,50  

00435940    74 1C                 je short pj.0043595E  
```

```
6A008D4C241C6A505152  //3.60  

6A008D4C241C6A505352  //pj  
```

```
00435913    8B9C24 94100000       mov ebx,dword ptr ss:[esp+1094]  

0043591A    8D4424 10             lea eax,dword ptr ss:[esp+10]  

0043591E    53                    push ebx  

0043591F    50                    push eax  

00435920    E8 DBFCFFFF           call mw.00435600  

00435925    8B15 BC845900         mov edx,dword ptr ds:[5984BC]  

0043592B    6A00                  push 0  

0043592D    8D4C241C              lea ecx,dword ptr ss:[esp+1C]  

00435931    6A50                  push 50                            

00435933    51                    push ebx                             ;这里的数据是没有加密之前的!  

00435934    52                    push edx  

00435935    E8 06F2FFFF           call pj.00434B40                   ;发送函数  

0043593A    83C4 18               add esp,18  

0043593D    83F8 50               cmp eax,50  

00435940    74 1C                 je short pj.0043595E  
```

\----------------------------------------接收明文(登陆)-------------------------------------------

```
004359CB    8B5304                mov edx,dword ptr ds:[ebx+4]  

004359CE    8B03                  mov eax,dword ptr ds:[ebx]  

004359D0    8BB42498100000        mov esi,dword ptr ss:[esp+1098]                  ;解密之后的数据放置在 ESI 中  

004359D7    52                    push edx  

004359D8    8D8C2494000000        lea ecx,dword ptr ss:[esp+94]  

004359DF    50                    push eax  

004359E0    51                    push ecx  

004359E1    56                    push esi  

004359E2    E879F9FFFF            call pj.00435360                               ;解密函数  

004359E7    8B4610                mov eax,dword ptr ds:[esi+10]                     ;需要把解密之后的数据地址放在 ESI 中  

004359EA    83C410                add esp,10  

004359ED    85C0                  test eax,eax  

004359EF    76 5A                 jbe short pj.00435A4B  
```

```
8B53048B038BB42498100000528D8C2494000000505156E879F9FFFF8B461083C410   //3.60  

B9140000008BB4249810000083EF508BD78BFE8BF2F3A583EF508BF78B4610909090   //pj  
```

```
004359CB    B914000000            mov ecx,14  

004359D0    8BB42498100000        mov esi,dword ptr ss:[esp+1098]  

004359D7    83EF50                sub edi,50  

004359DA    8BD7                  mov edx,edi  

004359DC    8BFE                  mov edi,esi  

004359DE    8BF2                  mov esi,edx  

004359E0    F3:A5                 rep movs dword ptr es:[edi],dword ptr ds:[esi]  

004359E2    83EF50                sub edi,50  

004359E5    8BF7                  mov esi,edi  

004359E7    8B4610                mov eax,dword ptr ds:[esi+10]  

004359EA    90                    nop  

004359EB    90                    nop  

004359EC    90                    nop  

004359ED    85C0                  test eax,eax  

004359EF    76 5A                 jbe short mw.00435A4B  
```

\----------------------------------------接收明文(地图)-------------------------------------------

```
00415DB7    8B5304                mov edx,dword ptr ds:[ebx+4]  

00415DBA    8B03                  mov eax,dword ptr ds:[ebx]  

00415DBC    52                    push edx  

00415DBD    8D8C2494000000        lea ecx,dword ptr ss:[esp+94]   ;需要解密的数据 BUFF  

00415DC4    50                    push eax  

00415DC5    8D542458              lea edx,dword ptr ss:[esp+58]   ;解密之后数据的 BUFF  

00415DC9    51                    push ecx  

00415DCA    52                    push edx  

00415DCB    E890F50100            call mw.00435360                ;解密  

00415DD0    668B5424 74           mov dx,word ptr ss:[esp+74]     ;这里有修改  

00415DD5    83C410                add esp,10  

00415DD8    6683FA FF             cmp dx,0FFFF  

00415DDC    753A                  jnz short mw.00415E18  

                                   50FC                             //3.65  
```

```
8B53048B03528D8C2494000000508D5424585152E890F50100668B54247483C410   //3.60  

B9140000008DB424900000008D7C2450F3A583EF5090909090668B542464909090   //pj  
```

```
00415DB7    B914000000            mov ecx,14  

00415DBC    8DB42490000000        lea esi,dword ptr ss:[esp+90]  

00415DC3    8D7C2450              lea edi,dword ptr ss:[esp+50]  

00415DC7    F3A5                  rep movs dword ptr es:[edi],dword ptr ds:[esi]  

00415DC9    83EF50                sub edi,50  

00415DCC    90                    nop  

00415DCD    90                    nop  

00415DCE    90                    nop  

00415DCF    90                    nop  

00415DD0    668B542464            mov dx,word ptr ss:[esp+64]      ;平衡堆栈  

00415DD5    90                    nop  

00415DD6    90                    nop  

00415DD7    90                    nop  

00415DD8    66:83FA FF            cmp dx,0FFFF  
```
