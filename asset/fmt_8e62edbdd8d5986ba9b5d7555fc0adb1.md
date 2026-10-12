
# ASProtect V1.23RC4 加壳的 DLL

> 原文：[`www.pediy.com/kssd/pediy06/pediy6648.htm`](https://www.pediy.com/kssd/pediy06/pediy6648.htm)

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg1.10`、`WinHex`、`LordPE`、`DLL_Loader`、`ImportREC 1.42+`

【实例下载】：点击此处下载（或鼠标右键另存为）

—————————————————————————————————

## 脱壳过程

`DLL` 脱壳方面的资料坛子上不多，于是我抽空写上几篇，聊作游戏。

这篇东东其实是《解密与解密》第 2 版中关于 `ASProtect` 的 `DLL` 脱壳教程的翻版啦。

但是我补充了手动寻找 `ASProtect V1.23RC4` 加壳 `DLL` 的重定位表地址的过程。

—————————————————————————————————

### 一、ASProtect V1.23RC4 加壳 DLL 的 OEP 很容易找

设置 `Ollydbg` 忽略所有的异常选项。老规矩：用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
代码:
003B7001     60                 pushad//进入 OD 后停在这
003B7002     E8 03000000        call EdrLib.003B700A
003B700A     5D                 pop ebp
003B700B     45                 inc ebp
003B700C     55                 push ebp
003B700D     C3                 retn
```

使用《加密与解密》第 2 版提及的方法吧。其实用以前的不忽略内存异常的方法也方便找 `DLL` 的 `OEP`。

我们在外壳入口 `003B7001` 处下断，`F9` 运行，`DLL` 被成功加载。

现在我们关闭 `Ollydbg` 的 `loaddll.exe` 窗口，`DLL` 卸载时会再次中断在外壳入口处！可以跟下去走到 `OEP` 了。

```
代码:
003B7008     EB 04              jmp short EdrLib.003B700E
003B700E     E8 01000000        call EdrLib.003B7014
003B7014     5D                 pop ebp
003B7015     BB EDFFFFFF        mov ebx,-13
003B701A     03DD              add ebx,ebp
003B701C     81EB 00700000     sub ebx,7000
003B7022     807D 4D 01        cmp byte ptr ss:[ebp+4D],1
003B7026     75 0C             jnz short EdrLib.003B7034
003B7028     8B7424 28         mov esi,dword ptr ss:[esp+28]
003B702C     83FE 01           cmp esi,1
003B702F     895D 4E           mov dword ptr ss:[ebp+4E],ebx
003B7032     75 31             jnz short EdrLib.003B7065//DLL 卸载时这里会跳转！
003B7065     B8 C9110000       mov eax,11C9//OEP 偏移值
003B706A     50                 push eax
003B706B     50                 push eax
003B706C     0345 4E            add eax,dword ptr ss:[ebp+4E]
003B706F     5B                 pop ebx
003B7070     85C0              test eax,eax
003B7072     74 1C             je short EdrLib.003B7090
003B7074     EB 01            jmp short EdrLib.003B7077
003B7077     81FB F8C0A523      cmp ebx,23A5C0F8
003B707D     74 35             je short EdrLib.003B70B4
003B707F     33D2              xor edx,edx
003B7081     56                 push esi
003B7082     6A 00              push 0
003B7084     56                 push esi
003B7085     FF75 4E            push dword ptr ss:[ebp+4E]
003B7088     FFD0              call eax; EdrLib.003B11C9//飞向光明之巅！ ^O^
```

下面我们借搞定重定位地址的时机，按照 `EXE` 的脱壳方法再找一次 `OEP`。

—————————————————————————————————

### 二、寻找重定位表地址

`ASProtect` 壳没有加密重定位表，这样使得其脱壳在重定位表方面设置比 `UPX` 加壳的 `DLL` 要简单。

再次用 `Ollydbg` 载入 `EdrLib.dll`，下断：`BP GetModuleHandleA`

当我们在 `GetModuleHandleA` 处中断 2 次后，就可以 `Alt+F9` 返回了。

```
代码:
008AC4A6     FF95 EC314400     call dword ptr ss:[ebp+4431EC]
008AC4AC     85C0              test eax,eax//返回这里
008AC4AE     75 07             jnz short 008AC4B7
008AC4B0     53                 push ebx
008AC4B1     FF95 F0314400      call dword ptr ss:[ebp+4431F0]
008AC4B7     8985 4D294400     mov dword ptr ss:[ebp+44294D],eax
008AC4BD     C785 51294400 00000>mov dword ptr ss:[ebp+442951],0
008AC4C7     8B95 D8304400      mov edx,dword ptr ss:[ebp+4430D8]
008AC4CD     8B06               mov eax,dword ptr ds:[esi]
008AC4CF     85C0              test eax,eax
008AC4D1     75 03             jnz short 008AC4D6
008AC4D3     8B46 10            mov eax,dword ptr ds:[esi+10]
008AC4D6     03C2              add eax,edx
008AC4D8     0385 51294400     add eax,dword ptr ss:[ebp+442951]
008AC4DE     8B18               mov ebx,dword ptr ds:[eax]
008AC4E0     8B7E 10            mov edi,dword ptr ds:[esi+10]
008AC4E3     03FA              add edi,edx
008AC4E5     03BD 51294400     add edi,dword ptr ss:[ebp+442951]
008AC4EB     85DB              test ebx,ebx
008AC4ED     0F84 A2000000    je 008AC595
008AC4F3     F7C3 00000080    test ebx,80000000
008AC4F9     75 04             jnz short 008AC4FF
008AC4FB     03DA              add ebx,edx
008AC4FD     43                 inc ebx
008AC4FE     43                 inc ebx
008AC4FF     53                 push ebx
008AC500     81E3 FFFFFF7F    and ebx,7FFFFFFF
008AC506     53                 push ebx
008AC507     FFB5 4D294400    push dword ptr ss:[ebp+44294D]
008AC50D     FF95 E8314400    call dword ptr ss:[ebp+4431E8]
008AC513     85C0              test eax,eax
008AC515     5B                 pop ebx
008AC516     75 6F             jnz short 008AC587
008AC518     F7C3 00000080    test ebx,80000000
008AC51E     75 19             jnz short 008AC539
008AC520     57                 push edi
008AC521     8B46 0C            mov eax,dword ptr ds:[esi+C]
008AC524     0385 D8304400     add eax,dword ptr ss:[ebp+4430D8]
008AC52A     50                 push eax
008AC52B     53                 push ebx
008AC52C     8D85 53314400    lea eax,dword ptr ss:[ebp+443153]
008AC532     50                 push eax
008AC533     57                 push edi
008AC534     E9 99000000       jmp 008AC5D2
008AC539     81E3 FFFFFF7F    and ebx,7FFFFFFF
008AC53F     8B85 DC304400    mov eax,dword ptr ss:[ebp+4430DC]
008AC545     3985 4D294400    cmp dword ptr ss:[ebp+44294D],eax
008AC54B     75 24             jnz short 008AC571
008AC54D     57                 push edi
008AC54E     8BD3              mov edx,ebx
008AC550     4A                 dec edx
008AC551     C1E2 02            shl edx,2
008AC554     8B9D 4D294400    mov ebx,dword ptr ss:[ebp+44294D]
008AC55A     8B7B 3C            mov edi,dword ptr ds:[ebx+3C]
008AC55D     8B7C3B 78         mov edi,dword ptr ds:[ebx+edi+78]
008AC561     035C3B 1C         add ebx,dword ptr ds:[ebx+edi+1C]
008AC565     8B0413            mov eax,dword ptr ds:[ebx+edx]
008AC568     0385 4D294400    add eax,dword ptr ss:[ebp+44294D]
008AC56E     5F                 pop edi
008AC56F     EB 16             jmp short 008AC587
008AC571     57                 push edi
008AC572     8B46 0C            mov eax,dword ptr ds:[esi+C]
008AC575     0385 D8304400     add eax,dword ptr ss:[ebp+4430D8]
008AC57B     50                 push eax
008AC57C     53                 push ebx
008AC57D     8D85 A4314400     lea eax,dword ptr ss:[ebp+4431A4]
008AC583     50                 push eax
008AC584     57                 push edi
008AC585     EB 4B             jmp short 008AC5D2
008AC587     8907               mov dword ptr ds:[edi],eax
008AC589     8385 51294400 04  add dword ptr ss:[ebp+442951],4
008AC590     E9 32FFFFFF       jmp 008AC4C7
008AC595     8906               mov dword ptr ds:[esi],eax
008AC597     8946 0C            mov dword ptr ds:[esi+C],eax
008AC59A     8946 10            mov dword ptr ds:[esi+10],eax
008AC59D     83C6 14            add esi,14
008AC5A0     8B95 D8304400     mov edx,dword ptr ss:[ebp+4430D8]
008AC5A6     E9 EBFEFFFF       jmp 008AC496
008AC5AB     8B85 652A4400     mov eax,dword ptr ss:[ebp+442A65]
008AC5B1     50                 push eax
008AC5B2     0385 D8304400     add eax,dword ptr ss:[ebp+4430D8]
008AC5B8     5B                 pop ebx
008AC5B9     0BDB              or ebx,ebx
008AC5BB     8985 112F4400    mov dword ptr ss:[ebp+442F11],eax
008AC5C1     61                 popad
008AC5C2     75 08             jnz short 008AC5CC
008AC5C4     B8 01000000       mov eax,1
008AC5C9     C2 0C00           retn 0C
008AC5CC     68 805A8A00       push 8A5A80
008AC5D1     C3                 retn
```
