
> 0052E1EB . 52 push edx ; |hInst => 10000000
> 0052E1EC . FF15 08376600 call dword ptr [&lt;USER32.LoadStringA&gt;>; \LoadStringA&lt;=========加载注册成功字符串
> 0052E1F2 . 6A 40 push 40 ; /Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
> 0052E1F4 . 8D4424 34 lea eax, dword ptr [esp+34] ; |
> 0052E1F8 . 68 9CAA6D00 push 006DAA9C ; |Title = ""
> 0052E1FD . 50 push eax ; |Text
> 0052E1FE . 56 push esi ; |hOwner
> 0052E1FF . FF15 90366600 call dword ptr [&lt;USER32.MessageBoxA&gt;>; \MessageBoxA&lt;=========显示注册成功字符串
> 0052E205 . 6A 00 push 0 ; /Result = 0
> 0052E207 . 56 push esi ; |hWnd
> 0052E208 . FF15 74366600 call dword ptr [&lt;USER32.EndDialog&gt;] ; \EndDialog&lt;===========结束注册对话框
> 0052E20E . 5F pop edi
> 0052E20F . B8 01000000 mov eax, 1
> 0052E214 . 5E pop esi
> 0052E215 . 81C4 68010000 add esp, 168
> 0052E21B . C3 retn

> 0052E21C &gt; 8B15 582A6E00 mov edx, dword ptr [6E2A58] ; xnviewzh.10000000
> 0052E222 . 8D4C24 30 lea ecx, dword ptr [esp+30]
> 0052E226 . 6A 40 push 40 ; /Count = 40 (64.)
> 0052E228 . 51 push ecx ; |Buffer
> 0052E229 . 68 93130000 push 1393 ; |RsrcID = STRING "Invalid registration"
> 0052E22E . 52 push edx ; |hInst => 10000000
> 0052E22F . FF15 08376600 call dword ptr [&lt;USER32.LoadStringA&gt;>; \LoadStringA
> 0052E235 . 6A 10 push 10 ; /Style = MB_OK|MB_ICONHAND|MB_APPLMODAL
> 0052E237 . 8D4424 34 lea eax, dword ptr [esp+34] ; |
> 0052E23B . 68 9CAA6D00 push 006DAA9C ; |Title = ""
> 0052E240 . 50 push eax ; |Text
> 0052E241 . 56 push esi ; |hOwner
> 0052E242 . FF15 90366600 call dword ptr [&lt;USER32.MessageBoxA&gt;>; \MessageBoxA

关键 call:

F7 跟进后到这里:

```
004D0DE0 /$ 8B5424 04 mov edx, dword ptr [esp+4]
004D0DE4 |. 53 push ebx
004D0DE5 |. 55 push ebp
004D0DE6 |. 56 push esi
004D0DE7 |. 57 push edi
004D0DE8 |. 8BFA mov edi, edx
004D0DEA |. 83C9 FF or ecx, FFFFFFFF
004D0DED |. 33C0 xor eax, eax
004D0DEF |. F2:AE repne scas byte ptr es:[edi]
004D0DF1 |. F7D1 not ecx
004D0DF3 |. 49 dec ecx
004D0DF4 |. BE C0656B00 mov esi, 006B65C0
004D0DF9 |. 8BE9 mov ebp, ecx
004D0DFB |. B9 05000000 mov ecx, 5
004D0E00 |. BF 30C76D00 mov edi, 006DC730
004D0E05 |. F3:A5 rep movs dword ptr es:[edi], dword p>
004D0E07 |. 8BF0 mov esi, eax
004D0E09 |. 74 21 je short 004D0E2C
004D0E0B |&gt; 8A0C16 /mov cl, byte ptr [esi+edx]&lt;============注册名字符传到 cl
004D0E0E |. 8AD9 |mov bl, cl&lt;============================再传到 bl
004D0E10 |. 3298 30C76D00 |xor bl, byte ptr [eax+6DC730]&lt;=========与一串十六进制数异或
004D0E16 |. 40 |inc eax&lt;===============================eax+1
004D0E17 |. 83F8 05 |cmp eax, 5&lt;============================eax 和 5 比较
004D0E1A |. 881C16 |mov byte ptr [esi+edx], bl&lt;============异或结果传到 12f608
004D0E1D |. 8888 2FC76D00 |mov byte ptr [eax+6DC72F], cl&lt;=========原字符传到 6dc730
004D0E23 |. 75 02 |jnz short 004D0E27&lt;====================eax 不等于 5 就跳
004D0E25 |. 33C0 |xor eax, eax&lt;==========================eax 等于 5,eax 就为 0
004D0E27 |&gt; 46 |inc esi&lt;===============================计数器加一
004D0E28 |. 3BF5 |cmp esi, ebp
004D0E2A |.^ 72 DF \\jb short 004D0E0B&lt;====================没完跳上去
```

根据 `eax`，可以知道十六进制数串只算其前五位

```
004D0E2C |&gt; 33FF xor edi, edi
004D0E2E |. 33C9 xor ecx, ecx
004D0E30 |. 85ED test ebp, ebp
004D0E32 |. 76 26 jbe short 004D0E5A
004D0E34 |&gt; 8A9F 35C76D00 /mov bl, byte ptr [edi+6DC735]&lt;=========十六进制数串从第六位开始循环依次传到 bl
004D0E3A |. 8BF5 |mov esi, ebp
004D0E3C |. 2BF1 |sub esi, ecx&lt;==========================注册名长度减去计数器值
004D0E3E |. 4E |dec esi&lt;===============================结果减一(指向第一区循环结果的尾部)
004D0E3F |. 8A0416 |mov al, byte ptr [esi+edx]&lt;============从上面循环结果后面开始取字符到 al
004D0E42 |. 32D8 |xor bl, al&lt;============================两者异或
004D0E44 |. 47 |inc edi&lt;===============================edi+1
004D0E45 |. 881C16 |mov byte ptr [esi+edx], bl
004D0E48 |. 8887 34C76D00 |mov byte ptr [edi+6DC734], al
004D0E4E |. 83FF 05 |cmp edi, 5&lt;============================edi 是否为 5
004D0E51 |. 75 02 |jnz short 004D0E55
004D0E53 |. 33FF |xor edi, edi&lt;==========================edi 为 5,edi 就清零
004D0E55 |&gt; 41 |inc ecx
004D0E56 |. 3BCD |cmp ecx, ebp&lt;==========================是否算完
004D0E58 |.^ 72 DA \\jb short 004D0E34&lt;====================没完跳上去
```

根据 `edi`,可以知道十六进制数串在这个循环只算其第六到第十位

```
004D0E5A |&gt; 33F6 xor esi, esi
004D0E5C |. 33FF xor edi, edi
004D0E5E |. 85ED test ebp, ebp
004D0E60 |. 76 21 jbe short 004D0E83
004D0E62 |&gt; 8A0417 /mov al, byte ptr [edi+edx]&lt;============从上面循环结果的前面开始取字符
004D0E65 |. 8A8E 3AC76D00 |mov cl, byte ptr [esi+6DC73A]&lt;=========十六进制数串的一个字节到 cl
004D0E6B |. 32C8 |xor cl, al&lt;============================两者异或
004D0E6D |. 46 |inc esi&lt;===============================esi+1
004D0E6E |. 880C17 |mov byte ptr [edi+edx], cl&lt;============传回去
004D0E71 |. 8886 39C76D00 |mov byte ptr [esi+6DC739], al
004D0E77 |. 83FE 05 |cmp esi, 5&lt;============================esi 是否为 5
004D0E7A |. 75 02 |jnz short 004D0E7E
004D0E7C |. 33F6 |xor esi, esi&lt;==========================esi 为 5 时，esi 就清零
004D0E7E |&gt; 47 |inc edi
004D0E7F |. 3BFD |cmp edi, ebp&lt;==========================是否已算完
004D0E81 |.^ 72 DF \\jb short 004D0E62&lt;====================没完就跳上去
```

根据 `esi`,可以知道十六进制数串在这个循环不只算其第十一位到第十五位

```
004D0E83 |&gt; 33FF xor edi, edi
004D0E85 |. 33C9 xor ecx, ecx
004D0E87 |. 85ED test ebp, ebp
004D0E89 |. 76 26 jbe short 004D0EB1
004D0E8B |&gt; 8A9F 3FC76D00 /mov bl, byte ptr [edi+6DC73F]&lt;========= 从十六进制的第十五位开始取字符
004D0E91 |. 8BF5 |mov esi, ebp
004D0E93 |. 2BF1 |sub esi, ecx
004D0E95 |. 4E |dec esi
004D0E96 |. 8A0416 |mov al, byte ptr [esi+edx]&lt;============从上面循环结果的尾部开始向前取字符
004D0E99 |. 32D8 |xor bl, al&lt;============================两者异或
004D0E9B |. 47 |inc edi&lt;===============================edi+1
004D0E9C |. 881C16 |mov byte ptr [esi+edx], bl
```
