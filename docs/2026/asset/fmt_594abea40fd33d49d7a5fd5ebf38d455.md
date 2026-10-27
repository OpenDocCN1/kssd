
# 原创深思3加密狗破解

日期：2004年3月23日　　　破解人：8FZ

---

【软件名称】：不公布名字　　　软件版本：

【软件大小】：

【下载地址】：

【软件简介】：软件是个商业软件。无狗运行无任何提示，并口狗。带狗破解。

【软件限制】：

【破解声明】：初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【操作系统】：

【破解工具】：

---

## 破解过程

OD载入EXE。稍等片刻，OK。

反汇编窗口往上翻到顶。。。看到如下的东西

```
00401000   /$  68 F0294400       push Phone.004429F0
00401005   |.  66:C705 F4294400 >mov word ptr ds:[4429F4], 0DFF6
0040100E   |.  66:C705 F6294400 >mov word ptr ds:[4429F6], 686B
00401017   |.  66:C705 F8294400 >mov word ptr ds:[4429F8], 399
00401020   |.  66:C705 F2294400 >mov word ptr ds:[4429F2], 0FFFF
00401029   |.  E8 92FF0200       call Phone.00430FC0
0040102E       66:A1 F0294400    mov ax, word ptr ds:[4429F0]
```

深思比较典型的东西

看到这个基本可以定下，`401029`处读狗，`mov ax, word ptr ds:[4429F0]`返回有狗无狗标志，顺便下个断，有狗0无狗非零。搜索全部`call Phone.00430FC0`全部下断，OK。运行

果然被拦截，手工返回O。避免EXIT啦。运行，报错，OK，再来

```
0041E800    .  F2:AE             repne scas byte ptr es:[edi]
0041E802    .  F7D1              not ecx
0041E804    .  49                dec ecx
0041E805    .  BF 82A58300       mov edi, 83A582
0041E80A    .  8D440A 01         lea eax, dword ptr ds:[edx+ecx+1]
0041E80E    .  8986 E8000000     mov dword ptr ds:[esi+E8], eax
0041E814    .  E8 671B0100       call Phone.00430380
0041E819    .  83C4 08           add esp, 8
0041E81C    .  85C0              test eax, eax
0041E81E       74 42             je short Phone.0041E862       跳一错误地方，直接NOP
0041E820    .  6A 6C             push 6C
0041E822    .  E8 ED400100       call <jmp.&MFC42.#823>
0041E827    .  83C4 04           add esp, 4
0041E82A    .  894424 14         mov dword ptr ss:[esp+14], eax
003452B8
0041E82E    .  3BC3              cmp eax, ebx
0041E830    .  C68424 50010000 0>mov byte ptr ss:[esp+150], 0B
0041E838    .  74 1D             je short Phone.0041E857
```

`NOP`后正常又被OD断下。。

```
0040104C   |.  66:C705 F4294400 >mov word ptr ds:[4429F4], 0DFF6
00401055   |.  66:C705 F6294400 >mov word ptr ds:[4429F6], 686B
0040105E   |.  66:C705 F8294400 >mov word ptr ds:[4429F8], 399
00401067   |.  66:A3 F2294400    mov word ptr ds:[4429F2], ax
0040106D   |.  E8 4EFF0200       call Phone.00430FC0
00401072       66:A1 F0294400    mov ax, word ptr ds:[4429F0] 手工返回
```

到

```
0042BE8A   |.  E8 B151FDFF       call Phone.00401040
0042BE8F   |.  8D4C24 18         lea ecx, dword ptr ss:[esp+18]
0042BE93   |.  8D443F 01         lea eax, dword ptr ds:[edi+edi+1]
0042BE97   |.  51                push ecx
0042BE98   |.  66:894424 1E      mov word ptr ss:[esp+1E], ax
0042BE9D   |.  66:897424 22      mov word ptr ss:[esp+22], si
0042BEA2       E8 E959FDFF       call Phone.00401890             到狗进行数据变换
0042BEA7   |.  8D4424 1C         lea eax, dword ptr ss:[esp+1C]
0042BEAB   |.  8D543F 02         lea edx, dword ptr ds:[edi+edi+2]
0042BEAF   |.  C1EE 10           shr esi, 10
0042BEB2   |.  50                push eax
0042BEB3   |.  66:895424 22      mov word ptr ss:[esp+22], dx
0042BEB8   |.  66:897424 26      mov word ptr ss:[esp+26], si
0042BEBD   |.  E8 DE57FDFF       call Phone.004016A0             到狗进行数据变换
0042BEC2   |.  83C4 10           add esp, 10
0042BEC5   |.  5F                pop edi
0042BEC6   |.  5E                pop esi
0042BEC7   |.  5B                pop ebx
0042BEC8   |.  83C4 4C           add esp, 4C
0042BECB   .  C3                retn
```

采用的是随鸡数变化，这个就不要跟进去了。。跟了也没用。说的这里，就要提下深思3的数据变换方式了，其实前面的牛人早就发表在网上了 ，也就是码表法，副本法，逆变换法

好继续GO，OD断下。返回调用

```
0041EF80   /$  83EC 4C           sub esp, 4C
0041EF83   |.  53                push ebx
0041EF84   |.  56                push esi
0041EF85   |.  57                push edi
0041EF86   |.  8BF9              mov edi, ecx
0041EF88   |.  51                push ecx
0041EF89   |.  8D87 D4000000     lea eax, dword ptr ds:[edi+D4]
0041EF8F   |.  8BCC              mov ecx, esp
0041EF91   |.  896424 10         mov dword ptr ss:[esp+10], esp
0041EF95   |.  50                push eax
0041EF96   |.  E8 FB380100       call <jmp.&MFC42.#535>
0041EF9B   |.  E8 10130100       call Phone.004302B0
0041EFA0   |.  6A 05             push 5
0041EFA2   |.  8BD8              mov ebx, eax
0041EFA4   |.  E8 9720FEFF       call Phone.00401040
0041EFA9   |.  8D5424 18         lea edx, dword ptr ss:[esp+18]
0041EFAD   |.  8D4C1B 01         lea ecx, dword ptr ds:[ebx+ebx+1]
0041EFB1   |.  52                push edx
0041EFB2   |.  66:894C24 1E      mov word ptr ss:[esp+1E], cx
0041EFB7   |.  E8 E424FEFF       call Phone.004014A0          到狗进行数据变换
0041EFBC   |.  8B7424 1C         mov esi, dword ptr ss:[esp+1C]
0041EFC0   |.  8D4C24 1C         lea ecx, dword ptr ss:[esp+1C]
0041EFC4   |.  8D441B 02         lea eax, dword ptr ds:[ebx+ebx+2]
0041EFC8   |.  51                push ecx
0041EFC9   |.  81E6 FFFF0000     and esi, 0FFFF
0041EFCF   |.  66:894424 22      mov word ptr ss:[esp+22], ax
0041EFD4   |.  E8 C720FEFF       call Phone.004010A0          到狗进行数据变换
0041EFD9   |.  8B4424 20         mov eax, dword ptr ss:[esp+20]
0041EFDD   |.  8B8F EC000000     mov ecx, dword ptr ds:[edi+EC]
0041EFE3   |.  25 FFFF0000       and eax, 0FFFF
0041EFE8   |.  83C4 10           add esp, 10
0041EFEB   |.  C1E0 10           shl eax, 10
0041EFEE   |.  0BC6              or eax, esi
0041EFF0   |.  5F                pop edi
0041EFF1   |.  5E                pop esi
0041EFF2   |.  33C1              xor eax, ecx
0041EFF4   |.  5B                pop ebx
0041EFF5   |.  83C4 4C           add esp, 4C
0041EFF8   .  C3                retn
```

OD随便做个记号。方便以后分析，返回调用

```
00412270   /$  56                push esi
00412271   |.  6A 00             push 0
00412273   |.  8BF1              mov esi, ecx
00412275   |.  68 9C000000       push 9C
0041227A   |.  E8 E1F6FFFF       call Phone.00411960
0041227F   |.  E8 50070200       call <jmp.&MFC42.#1168>
00412284   |.  8B48 04           mov ecx, dword ptr ds:[eax+4]
00412287   |.  E8 F4CC0000       call Phone.0041EF80
0041228C       05 90000000       add eax, 90            返回处。EAX重点。打个记号
00412291   |.  68 2D040000       push 42D
00412296   |.  8BCE              mov ecx, esi
00412298   |.  8986 2C010000     mov dword ptr ds:[esi+12C], eax
0041229E   |.  E8 7D090200       call <jmp.&MFC42.#3092>
004122A3   |.  85C0              test eax, eax
004122A5   |.  74 1A             je short Phone.004122C1
004122A7   |.  8B40 20           mov eax, dword ptr ds:[eax+20]
004122AA   |.  6A 00             push 0
004122AC   |.  6A 01             push 1
004122AE   |.  68 F1000000       push 0F1
004122B3   |.  50                push eax
004122B4   |.  FF15 94994300     call dword ptr ds:[<&USER32.SendMessageA>]
004122BA   |.  8BCE              mov ecx, esi
004122BC   |.  E8 1F040000       call Phone.004126E0
004122C1   |>  5E                pop esi
004122C2   .  C3                retn
```

OD继续，同样一处调用上面

```
00413E60   /$  6A FF             push -1
00413E62   |.  68 10664300       push Phone.00436610
00413E67   |.  64:A1 00000000    mov eax, dword ptr fs:[0]
00413E6D   |.  50                push eax
00413E6E   |.  64:8925 00000000  mov dword ptr fs:[0], esp
00413E75   |.  83EC 08           sub esp, 8
00413E78   |.  8D4424 00         lea eax, dword ptr ss:[esp]
00413E7C   |.  50                push eax
00413E7D   |.  E8 5EFEFFFF       call Phone.00413CE0
00413E82   |.  8B4C24 00         mov ecx, dword ptr ss:[esp]
00413E86   |.  C74424 10 0000000>mov dword ptr ss:[esp+10], 0
00413E8E   |.  8B41 F8           mov eax, dword ptr ds:[ecx-8]
00413E91       85C0              test eax, eax
00413E93   |.  74 27             je short Phone.00413EBC
00413E95   |.  51                push ecx
00413E96   |.  8D5424 04         lea edx, dword ptr ss:[esp+4]
00413E9A   |.  8BCC              mov ecx, esp
00413E9C   |.  896424 08         mov dword ptr ss:[esp+8], esp
00413EA0   |.  52                push edx
00413EA1   |.  E8 F0E90100       call <jmp.&MFC42.#535>
00413EA6   |.  B9 B0EE4400       mov ecx, Phone.0044EEB0
```
