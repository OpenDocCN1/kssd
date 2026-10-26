
```asm
.text:004160B9                 movsx   eax, word ptr [eax+59D6D2h]
.text:004160C0                 mov     [ecx], eax
.text:004160C2                 mov     al, 1
.text:004160C4                 pop     ebx
.text:004160C5                 add     esp, 180h
.text:004160CB                 retn
.text:004160CC ; ----------------------------------------------------------------------------
.text:004160CC
.text:004160CC loc\_4160CC:                             ; CODE XREF: sub\_415EC0+5Dj
.text:004160CC                 lea     eax, ds:0[ecx\*8]
.text:004160D3                 mov     edx, [esp+190h+arg\_8]
.text:004160DA                 sub     eax, ecx
.text:004160DC                 pop     edi
.text:004160DD                 shl     eax, 1
.text:004160DF                 pop     esi
.text:004160E0                 pop     ebp
.text:004160E1                 movsx   ecx, word ptr [eax+59D6CEh]
.text:004160E8                 mov     [edx], ecx
.text:004160EA                 mov     edx, [esp+184h+arg\_C]
.text:004160F1                 movsx   ecx, word ptr [eax+59D6D0h]
.text:004160F8                 mov     [edx], ecx
.text:004160FA                 mov     ecx, [esp+184h+arg\_10]
.text:00416101                 movsx   eax, word ptr [eax+59D6D2h]
.text:00416108                 mov     [ecx], eax
.text:0041610A                 mov     al, 1
.text:0041610C                 pop     ebx
.text:0041610D                 add     esp, 180h
.text:00416113                 retn
.text:00416114 ; ----------------------------------------------------------------------------
.text:00416114
.text:00416114 loc\_416114:                             ; CODE XREF: sub\_415EC0+1Cj
.text:00416114                                         ; sub\_415EC0+34j ...
.text:00416114                 pop     edi
.text:00416115                 pop     esi
.text:00416116                 pop     ebp
.text:00416117                 xor     al, al
.text:00416119                 pop     ebx
.text:0041611A                 add     esp, 180h
.text:00416120                 retn
.text:00416120 sub\_415EC0      endp
```

### 利用漏洞写外挂的认证服务器

```asm
.text:004428A7                 shr     eax, 10h
.text:004428AA   25FF7F0000    and     eax, 7FFFh
                 B811110000    MOV     EAX,1111
.text:004428AF                 retn
.text:004428AF \_rand           endp
```

这样做是为了使分析更容易一点!

```asm
00435481    F3:A5           rep movs dword ptr es:[edi],dword ptr ds:[esi]
```

这里是把解密之后的数据挪动到使用的地方!`EDI`中最终保存者我们要分析的数据

### 返回的数据包进行分离把有关挂机的有用数据分离出来进行保存(`59D6C8`)

```asm
.text:00415E4B                 mov     cx, word ptr [esp+18Ch+arg\_4]   ;CX=0000
.text:00415E53                 shl     eax, 1
.text:00415E55                 pop     esi
.text:00415E56                 mov     [eax+59D6C8h], cx               ;这里是数据的开始地址
.text:00415E5D                 mov     cx, [esp+188h+var\_136]          ;CX=0024  0021  过门的x坐标
.text:00415E62                 mov     [eax+59D6CAh], bp               ;BP=0004  000C
.text:00415E69                 mov     [eax+59D6CCh], dx               ;DX=0004  000C
.text:00415E70                 mov     dx, [esp+188h+var\_138]          ;DX=0146  0093  过门的y坐标
.text:00415E75                 pop     ebp
.text:00415E76                 mov     [eax+59D6CEh], dx               ;
.text:00415E7D                 mov     dx, [esp+184h+var\_12A]          ;DX=0001  0001
.text:00415E82                 mov     [eax+59D6D0h], cx               ;
.text:00415E89                 mov     ecx, [esp+184h+arg\_C]
.text:00415E90                 mov     [eax+59D6D2h], dx
.text:00415E97                 mov     word ptr [eax+59D6D4h], 1
.text:00415EA0                 movsx   eax, word ptr [eax+59D6CCh]
.text:00415EA7                 mov     [ecx], eax
.text:00415EA9                 mov     al, 1
.text:00415EAB                 pop     ebx
.text:00415EAC                 add     esp, 180h
.text:00415EB2                 retn
```

### 坐标数据进一步参与挂机的运算!

```asm
004160E1    0FBF88 CED65900 movsx ecx,word ptr ds:[eax+59D6CE]
004160E8    890A            mov dword ptr ds:[edx],ecx
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

这里需要对`EDX`的地址设置内存访问断点进行数据处理追踪!

```asm
0040828A    8B7424 24                 mov esi,dword ptr ss:[esp+24]
0040828E    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
00408292    83FE FF                   cmp esi,-1
00408295    75 09                     jnz short pj.004082A0
00408648    8B4C24 20                 mov ecx,dword ptr ss:[esp+20]
0040864C    8B5424 24                 mov edx,dword ptr ss:[esp+24]
00408650    6A 00                     push 0
00408652    6A 00                     push 0
00408654    6A 00                     push 0
00408656    51                        push ecx
00408657    52                        push edx
00408658    53                        push ebx
00408659    56                        push esi
0040865A    55                        push ebp
0040865B    E8 10040000               call pj.00408A70
00408660    83C4 20                   add esp,20
00408663    56                        push esi
00408664    55                        push ebp
00408665    E8 66510000               call pj.0040D7D0
0040866A    83C4 08                   add esp,8
0040866D    85C0                      test eax,eax
0040866F    0F85 AA010000             jnz pj.0040881F
```

### 登陆部分的明文修改...

#### 发送明文

```asm
00435913    8B9C24 94100000           mov ebx,dword ptr ss:[esp+1094]
0043591A    8D4424 10                 lea eax,dword ptr ss:[esp+10]
0043591E    53                        push ebx                       ;这里的数据还没有加密
0043591F    50                        push eax
00435920    E8 DBFCFFFF               call pj.00435600               ;加密函数
00435925    8B15 BC845900             mov edx,dword ptr ds:[5984BC]
0043592B    6A00                      push 0
0043592D    8D4C241C                  lea ecx,dword ptr ss:[esp+1C]
00435931    6A50                      push 50
00435933    51                        push ecx
00435934    52                        push edx
00435935    E8 06F2FFFF               call pj.00434B40               ;发送函数
0043593A    83C4 18                   add esp,18
0043593D    83F8 50                   cmp eax,50
00435940    74 1C                     je short pj.0043595E

6A008D4C241C6A505152  //3.60

6A008D4C241C6A505352  //pj
```

```asm
00435913    8B9C24 94100000           mov ebx,dword ptr ss:[esp+1094]
0043591A    8D4424 10                 lea eax,dword ptr ss:[esp+10]
0043591E    53                        push ebx
0043591F    50                        push eax
00435920    E8 DBFCFFFF               call mw.00435600
00435925    8B15 BC845900             mov edx,dword ptr ds:[5984BC]
0043592B    6A00                      push 0
0043592D    8D4C241C                  lea ecx,dword ptr ss:[esp+1C]
00435931    6A50                      push 50
00435933    53                        push ebx                       ;这里的数据是没有加密之前的!
```
