
# Obsidium_1.0.0.69 口令加密功能的完全分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-614.htm`](https://www.pediy.com/kssd/pediy08/pediy8-614.htm)

**【文章标题】**: Obsidium_1.0.0.69 口令加密功能的完全分析

**【文章作者】**: SigCell

**【测试软件】**: 对 `Notepad` 进行 `Obsidium_1.0.0.69` 口令加密

**【作者声明】**: 本文旨在探讨 `Obsidium_1.0.0.69` 口令加密功能，然而对于“口令加密”问题无力提出解决方案。本文的研究对象不是 `Obsidium_1.0.0.69` 的脱壳，这点在 kanxue 老大的文章中已经写的很清楚了。

---

**【技巧说明】**: 可以用“花指令去除器”去除 `Obsidium` 的花指令，此时可以把干净的代码拷贝出来，去掉无效的 `nop` 指令，首先静态的观察和分析，找出关键点以及猜测代码的大致功能。在调试之前，需要把花指令还原回来，避免自校验带来的不必要麻烦。（另外下面的代码绝大部分是去除花指令后整理所得。）

**【需要基础】**: `SEH` + `Stolen Code` + `WndProc`

**【详细过程】**: (原文有色彩、下划线等标记,附件中提供原文和测试软件)

### （1）确定密码框位置

确定密码框是在第一个 `div 0` 之后和第二 `div 0` 之前出现。来到第一个 `div 0` 异常处。

### （2）设置断点

`Ctrl + G` ==> `009304F5`（`[ESP+4]`） ==> `F2` ==> `SHIFT+F9` ==> 撤消断点:

```
1th_Handler ()

{

009304F5    90                 nop                       ;

009304FA    C8 000000          enter   0, 0
009304FE    E8 00000000       call    00930503
00930507    5A                 pop     edx
0093050C    8B4D 10           mov     ecx, [ebp+10]     ; ecx 指向 CONTEXT
0093050F    33C0              xor     eax, eax
00930516    8D92 D4000000     lea     edx, [edx+D4]     ; edx = 9305D7
00930522    8991 B8000000     mov     [ecx+B8], edx     ; eip = edx = 9305D7
00930530    8941 04           mov     [ecx+4], eax      ; dr0 = 0
00930536    4A                 dec     edx
0093053D    8951 0C           mov     [ecx+C], edx      ; dr2 = edx - 1
00930544    42                 inc     edx
0093054B    8941 08           mov     [ecx+8], eax      ; dr1 = 0
00930553    8951 10           mov     [ecx+10], edx     ; dr3 = edx
0093055A    C741 18 5500333>  mov     dword ptr [ecx+18], 33330055
00930567    C9                 leave
00930568    C3                 retn

}
```

### （3）设置断点

`Ctrl + G` ==> `9305D7` ==> `F2` ==> `SHIFT+F9` ==> 撤消断点:

```
009305D7    90                 nop                                 ; 返回到这里
009305DA    8B4D 10           mov     ecx, [ebp+10]
009305DD    8D93 62FFFFFF     lea     edx, [ebx-9E]
009305E9    81B9 AC000000 C>  cmp     dword ptr [ecx+AC], 0CC     ; 检测 int 3
009305F7  ^ 74 C3             je      short 009305BC
009305FD    8991 B8000000     mov     [ecx+B8], edx               ; eip = edx =009304E0
0093060D    8941 04           mov     [ecx+4], eax
00930614    4A                 dec     edx
0093061A    8941 08           mov     [ecx+8], eax
00930620    8951 0C           mov     [ecx+C], edx
00930626    8951 10           mov     [ecx+10], edx
0093062E    C741 18 5500333>  mov     dword ptr [ecx+18], 33330055
0093063E    64:8F00           pop     dword ptr fs:[eax]
00930644    83C4 04           add     esp, 4
0093064C    5B                 pop     ebx
0093064D    C9                 leave
0093064E    C3                 retn
```

### （4）设置断点

`Ctrl + G` ==> `009304E0` ==> `F2` ==> `SHIFT+F9` ==> 撤消断点:

```
009304E0    64:8F00           pop     dword ptr fs:[eax]      ; 删除异常帧
009304E9    83C4 04           add     esp, 4
009304F0    5B                 pop     ebx
009304F1    C3                 retn
```

### （5）返回到这里

```
0101DEDF    FF55FC            call  [ebp-4]
0101DEE6  F7470C040000>      test  dword ptr [edi+C], 4 ; 返回到这里
0101DEF3    74 1E             je      short 0101DF13
0101DEF9    85C0              test    eax, eax
0101DEFE    74 13             je      short 0101DF13
0101DF09    50                 push    eax
0101DF0A    FF53 2C           call    [ebx+2C]
0101DF19    8B07              mov     eax, [edi]
0101DF1E    8D7C07 14         lea     edi, [edi+eax+14]
0101DF28  ^ E9 64FCFFFF      jmp     0101DB91                ; 大循环
0101DF35    8D47 14           lea     eax, [edi+14]
0101DF3C    64:67:8F06 0000   pop     dword ptr fs:[0]
0101DF46    83C4 04           add     esp, 4
0101DF4C    5F                 pop     edi
0101DF4D    5B                 pop     ebx
0101DF4E    5E                 pop     esi
0101DF4F    C9                 leave
0101DF50    C2 0C00            retn    0C                      ; 直接在这里 F2，F9，跳出大循环
```

### （6）返回到这里

```
0101D61E    90                 nop                     ; 返回到这里
0101D623    85C0              test    eax, eax
0101D628    0F84 C3000000     je      0101D6F1
0101D634    8BF8              mov     edi, eax
0101D642    83BD B4B5B200 0>  cmp     dword ptr [ebp+B2B5B4], 0
0101D64C    74 25             je      short 0101D673
0101D658    55                 push    ebp
0101D659    56                 push    esi
0101D65A    E8 FA010000       call    0101D859          ; 关键函数，进 ==(7)
0101D663    85C0              test    eax, eax          ; 如果在上一行 F8 的话，会显示密码输入框
0101D668  ^ 0F84 B6FBFFFF     je      0101D224          ; 伪爆破点 1(暴力分子最喜欢的地方）
0101D67C    8B46 04           mov     eax, [esi+4]      ; eax = 981750
0101D684    8D50 10           lea     edx, [eax+10]     ; edx = 981760
0101D68A    8D85 B7BAB200     lea     eax, [ebp+B2BAB7] ; eax = 101E01B
0101D699    68 80000000       push    80                ; 参数 3：80
0101D69E    50                 push    eax               ; 参数 2：eax = 101E01B
0101D69F    52                 push    edx               ; 参数 1：edx = 981760
0101D6A0  FF56 18          call   [esi+18]           ; 根据输入的密码产生子密钥
0101D6AB    8B46 04           mov     eax, [esi+4]
0101D6B1    8D50 10           lea     edx, [eax+10]
0101D6B7    8D8D DBBAB200     lea     ecx, [ebp+B2BADB]
0101D6C6    68 5D0C0000       push    0C5D            ; 参数 4：长度 = C50
0101D6CB    51                 push    ecx             ; 参数 3：存放解密代码的地址，ecx = 101E03F
0101D6CC    6A 0A             push    0A              ; 参数 2：
0101D6CE    52                 push    edx             ; 参数 1：edx = 931760
0101D6CF  FF56 1C          call   [esi+1C]           ; 利用子密钥解密代码
0101D6DC    E9 5E090000       jmp     0101E03F        ; 关键的 JMP，去执行解密后的代码（如果正确解密的话，可以正常执行，否则就是执行垃圾指令，无法正常执行）
```

**伪爆破点 1**：如果修改为 `jne` 后，最终却发现不能正常执行，原因是 `0101E03F` 处的代码是动态解密的，如果没有输入正确的密码解密出来的就是错误的垃圾代码。大致跟了下 `0101D6A0` 和 `0101D6CF`，发现是密码学算法，`N` 复杂，所以估计前者是计算子密钥，后着是解密代码。

起初到了这里看到爆破点，眼神一亮，调了两次，绝望了。动态解密，而使用的加密算法肯定是不可逆的，也就是说在没有正确密码的情况下，绝对无法正确解密代码，此路不通。那么唯一的一条路：跟踪用于密码输入的对话框。解剖它的消息处理函数。绝望了，又出现希望，害怕再一次绝望。`continue_debug`，路漫漫，看不到终点。

### （7）进入（6）中的关键函数

```
0101D859    C8 040000         enter   4, 0              ; 进到这里
0101D85D    53                 push    ebx
0101D85E    56                 push    esi
0101D85F    57                 push    edi
0101D860    8B5D 08           mov     ebx, [ebp+8]      ; 第 2 参数
0101D863    8B45 0C           mov     eax, [ebp+C]      ; 第 1 参数
0101D866    8B7B 04           mov     edi, [ebx+4]
0101D869    83C7 03           add     edi, 3
0101D86C    8DB0 0DB2B200     lea     esi, [eax+B2B20D]
0101D872    83E7 FC           and     edi, FFFFFFFFC
0101D875    B9 3A000000       mov     ecx, 3A
0101D87A    897D FC           mov     [ebp-4], edi
0101D87D    F3:A5             rep     movs dword ptr es:[edi], dword ptr [esi]
0101D87F    8143 04 E800000>  add     dword ptr [ebx+4], 0E8
0101D886    6A 00             push    0
0101D888    68 706586B1       push    B1866570
0101D88D    6A 00             push    0
0101D88F    FF53 20           call    [ebx+20]          ; [ebx+20] = 010264CA
0101D892    8B55 0C           mov     edx, [ebp+C]      ; 此时 eax = 1000000
0101D895    899A B0B5B200     mov     [edx+B2B5B0], ebx
0101D89B    8D92 6CB3B200     lea     edx, [edx+B2B36C]
0101D8A1    6A 00             push    0
0101D8A3    52                 push    edx
0101D8A4    6A 00             push    0
0101D8A6    FF75 FC           push    dword ptr [ebp-4]
0101D8A9    50                 push    eax
0101D8AA    68 5C96F40A       push    0AF4965C
0101D8AF    6A 01             push    1
0101D8B1   FF53 20     call   [ebx+20]   ; 关键函数，实现 Stolen Code,并执行（显示对话框）==>(8)
```



### 代码段 1：对话框返回处理

```
0101D8B4    816B 04 E800000> sub     dword ptr [ebx+4], 0E8 ；密码输入的对话框结束后返回到这一行（不能在这里 F2 设断，有自校验的，可以通过在上行直接 F8），密码正确时 eax = 1，点击对话中取消按钮时 eax = 0
```

```
0101D8BB    83F8 FF         cmp     eax, -1                ；这两行为无效指令(伪爆破点 2）
0101D8BE    74 07           je      short 0101D8C7         ；总是不跳
0101D8C0    5F              pop     edi
0101D8C1    5E              pop     esi
0101D8C2    5B              pop     ebx
0101D8C3    C9              leave
0101D8C4    C2 0800         retn    8
```

```
0101D8C7    33C0            xor     eax, eax
0101D8C9    5F              pop     edi
0101D8CA    5E              pop     esi
0101D8CB    5B              pop     ebx
0101D8CC    C9              leave
0101D8CD    C2 0800         retn    8
```

伪爆破点 2：看到这里，有了上次的经验，知道怎么爆也是没有用的。

### （8）进入 `101D8B1` 的 `CALL`，分析下 `Stolen Code` 的细节（抽的是 `DialogBoxIndirectParamA`）：

```
010264CA    90              nop
010264D0    60              pushad
010264D5    E8 00000000     call    010264DA
010264DE    5B              pop     ebx
010264E5    81EB C85AB300   sub     ebx, 0B35AC8
010264EF    8B9B B45AB300   mov     ebx, [ebx+B35AB4]
010264FD    8B4424 24       mov     eax, [esp+24]
0102650B    33C9            xor     ecx, ecx
01026513    8B4483 48       mov     eax, [ebx+eax*4+48]
0102651C    8B5424 28       mov     edx, [esp+28]
01026525    51              push    ecx
01026526    51              push    ecx
01026527    51              push    ecx
01026528    52              push    edx
01026529    50              push    eax
0102652A    FF53 40         call    [ebx+40]               ; 取 USER32.DialogBoxIndirectParamA 地址
01026532    85C0            test    eax, eax               ；检测是否成功
01026538    0F84 DE010000   je      0102671C
01026544    8BF0            mov     esi, eax               ；esi = USER32.DialogBoxIndirectParamA
0102654D    83EC 28         sub     esp, 28
01026553    8BFC            mov     edi, esp               ；edi = 0006FDEC,stolen code 存放点(动态变化）
01026559    C74424 4C 02000> mov     dword ptr [esp+4C], 2
01026567    897424 50       mov     [esp+50], esi
01026571    90              nop                  ；循环起点
0102657B    803E C2         cmp     byte ptr [esi], 0C2   ；如果当前指令的第一字节是 C2
01026584    0F84 B1000000   je      0102663B               ；就跳出循环
0102658F    803E C3         cmp     byte ptr [esi], 0C3   ；如果当前指令的第一字节是 C3
01026598    0F84 9D000000   je      0102663B               ；就跳出循环
010265A1    56              push    esi                  ；参数：指令地址
010265A2    FF53 24         call    [ebx+24]               ；取当前指令的长度到 eax
010265A8    83F8 FF         cmp     eax, -1
010265AB    0F84 21010000   je      010266D2
010265B7    F7C2 00060000   test    edx, 600
010265C3    75 76           jnz     short 0102663B
010265D1    56              push    esi
010265D8    57              push    edi
010265DE    8BC8            mov     ecx, eax               ; ecx = eax = 2
010265E3    F3:A4           rep     movs byte ptr es:[edi]>; 拷贝指令
010265E9    5F              pop     edi
010265EF    5E              pop     esi
010265FB    803F CC         cmp     byte ptr [edi], 0CC    ；检测 int 3
01026603    0F84 DF000000   je      010266E8               ；也就是说，stolen code 中不能下断
0102661A    03F0            add     esi, eax
01026622    03F8            add     edi, eax
0102662A    FF4C24 4C       dec     dword ptr [esp+4C]     ；指令数减一
01026631    0F85 3AFFFFFF   jnz     01026571              ; 循环尾部
01026642    C607 E9         mov     byte ptr [edi], 0E9    ; JMP
01026649    8BC6            mov     eax, esi
0102664E    2BC7            sub     eax, edi
01026653    83E8 05         sub     eax, 5                 ；计算偏移
01026659    8947 01         mov     [edi+1], eax           ；在 edi 位置形成指令 JMP ********
01026666    8BC4            mov     eax, esp
01026675    8B4C24 48       mov     ecx, [esp+48]
01026686    8039 CC         cmp     byte ptr [ecx], 0CC    ；检测 int 3
0102668F    74 57           je      short 010266E8
010266A0    894C24 50       mov     [esp+50], ecx
010266AA    894424 4C       mov     [esp+4C], eax
010266B6    83C4 28         add     esp, 28
010266BE    61              popad
010266C4    83C4 04         add     esp, 4
010266CD    C3              retn                           ；[esp] = 0006FDEC，stolen code 存放点
```

### （9）从 `10266CD` 处转到 `stolen code` 处执行：

```
0006FDEC    8BFF            mov     edi, edi               ；此时在堆栈观察 DialogBoxIndirectParamA 的 5 个参数
0006FDEE    55              push    ebp
0006FDEF    - E9 3E0A7A7C   jmp     kernel32.7C810832      ；这 3 句即是 stolen code
```

```
0006FF6C    01000000        ASCII "MZP"          ；第 1 参数
0006FF70    00931750                  ；第 2 参数
0006FF74    00000000                  ；第 3 参数
0006FF78    0101D8D0      NotepadX.0101D8D0    ；第 4 参数：lpDialogProc
0006FF7C    00000000                  ；第 5 参数
```

`Ctrl + G` ==>`0101D8D0` ==> 显示如下：

```c
DialogProc ( HWND hWndDlg, UINT uMsg, WPARAM wParam, LPARAM lParam )
{
    0101D8D0    C8 000000       enter   0, 0
    0101D8D4    53              push    ebx
    0101D8D5    8B45 0C         mov     eax, [ebp+C]    ；eax = uMsg
    0101D8D8    3D 11010000     cmp     eax, 111        ；判断 WM_COMMAND
    0101D8DD    74 34           je      short 0101D913  ；==> （10）
    0101D8DF    3D 10010000     cmp     eax, 110        ；判断是否 WM_INITDIALOG
    0101D8E4    74 07           je      short 0101D8ED  ；
    0101D8E6    33C0            xor     eax, eax        ；EAX = 0
    0101D8E8    5B              pop     ebx
    0101D8E9    C9              leave
    0101D8EA    C2 1000         retn    10              ；return FALSE
}
```

### （10）处理 `WM_COMMAND` 消息

```
0101D913    8B45 10           mov     eax, [ebp+10]           ；eax = uParam，控件 ID,低字为控件 ID
0101D916    66:3D 6700        cmp     ax, 67                  ；ID == 67
0101D91A    74 34             je      short 0101D950          ；
0101D91C    66:3D 0100        cmp     ax, 1                   ；ID == 1
0101D920    74 2E             je      short 0101D950
0101D922    66:3D 0200        cmp     ax, 2                   ；ID == 2
0101D926    74 06             je      short 0101D92E
0101D928    66:3D 6600        cmp     ax, 66                  ；ID == 66
0101D92C    ^ 75 B8           jnz     short 0101D8E6          ；其他控件消息就 return false.
0101D92E    E8 00000000       call    0101D933                ；ID = 2 时，执行这里
0101D933    58              pop     eax
0101D934    8B80 E1010000   mov     eax, [eax+1E1]
0101D93A    6A 00           push    0
0101D93C    FF75 08         push    dword ptr [ebp+8]
0101D93F    68 6EF49B3B     push    3B9BF46E
0101D944    6A 01           push    1
0101D946    FF50 20         call    [eax+20]                  ；调用默认的消息处理体
0101D949    33C0            xor     eax, eax                  ；eax = 0
0101D94B    5B              pop     ebx
0101D94C    C9              leave
0101D94D    C2 1000         retn    10                        ；return FALSE
0101D950    FF75 08         push    dword ptr [ebp+8]         ；(参数：hWndDlg)ID = 1 或 ID = 67 时，跳到这里执行
0101D953    E8 D8000000     call    0101DA30                  ；检测密码是否正确 ==> ( 11 )
0101D958    83F8 FF         cmp     eax, -1
0101D95B    ^ 74 D1         je      short 0101D92E
0101D95D    85C0            test    eax, eax                  ；输入的密码是否正确
0101D95F    ^ 74 85         je      short 0101D8E6            ；如果输入的密码错误，return false
0101D961    E8 00000000     call    0101D966
0101D966    58              pop     eax
0101D967    8B80 AE010000   mov     eax, [eax+1AE]
0101D96D    6A 01           push    1
0101D96F    FF75 08         push    dword ptr [ebp+8]
0101D972    68 6EF49B3B     push    3B9BF46E
0101D977    6A 01           push    1
```



```
0101D979    FF50 20         call    [eax+20]                ；调用默认的消息处理体
0101D97C    33C0          xor     eax, eax                ；eax = 0
0101D97E    5B            pop     ebx
0101D97F    C9            leave
0101D980    C2 1000       retn    10                      ；return FALSE
```

### （11）检测输入的密码是否正确

```
0101DA30    C8 340000     enter   34, 0
0101DA34    56            push    esi
0101DA35    53            push    ebx
0101DA36    57            push    edi
0101DA37    E8 00000000   call    0101DA3C
0101DA3C    5B            pop     ebx
0101DA3D    8BF3          mov     esi, ebx
0101DA3F    8B9B D8000000 mov     ebx, [ebx+D8]
0101DA45    81EE D8B4B200 sub     esi, 0B2B4D8
0101DA4B    68 80000000   push    80
0101DA50    FF73 04       push    dword ptr [ebx+4]       ; 字符串缓冲区
0101DA53    6A 65         push    65
0101DA55    FF75 08       push    dword ptr [ebp+8]
0101DA58    68 79D84448   push    4844D879
0101DA5D    6A 01         push    1
0101DA5F    FF53 20       call    [ebx+20]                  ; 取得输入的字符串，返回长度
0101DA62    85C0          test    eax, eax
0101DA64    0F84 97000000 je      0101DB01
0101DA6A    8BF8          mov     edi, eax
0101DA6C    8D55 F0       lea     edx, [ebp-10]
0101DA6F    52            push    edx                       ; 参数 3：
0101DA70    50            push    eax                       ; 参数 2：输入的长度，7
0101DA71    FF73 04       push    dword ptr [ebx+4]         ; 参数 1：输入的字符串
0101DA74    FF53 0C       call    [ebx+C]                   ；单向的加密函数
0101DA77    8D8E B8B5B200 lea     ecx, [esi+B2B5B8]         ; ecx 为标准的密文
0101DA7D    C745 CC 04000000 mov   dword ptr [ebp-34], 4   ; 存放比较次数
0101DA84    8D55 F0       lea     edx, [ebp-10]             ; edx 为 unsigned 数组，长度 4
0101DA87    8B02          mov     eax, [edx]                ；这里开始的 3 句 nop 掉
0101DA89    3901          cmp     [ecx], eax                ；修改为 mov eax, [ecx]；mov [edx], eax
0101DA8B    75 3E         jnz     short 0101DACB            ；这样虽然可以把密文盗窃过来，但还是不能解决问题
0101DA8D    83C2 04       add     edx, 4
0101DA90    83C1 04       add     ecx, 4
0101DA93    FF4D CC       dec     dword ptr [ebp-34]        ; 比较的次数减 1
0101DA96    75 EF         jnz     short 0101DA87            ; 继续比较（这 7 个指令的功能是比较密文）
0101DA98    8D45 D0       lea     eax, [ebp-30]
0101DA9B    50            push    eax
0101DA9C    57            push    edi
0101DA9D    FF73 04       push    dword ptr [ebx+4]
0101DAA0    FF53 58       call    [ebx+58]
0101DAA3    8DBE B7BAB200 lea     edi, [esi+B2BAB7]
0101DAA9    B9 04000000   mov     ecx, 4
0101DAAE    8D55 D0       lea     edx, [ebp-30]
0101DAB1    8B02          mov     eax, [edx]
0101DAB3    3342 04       xor     eax, [edx+4]
0101DAB6    3107          xor     [edi], eax
0101DAB8    83C7 04       add     edi, 4
0101DABB    83C2 08       add     edx, 8
0101DABE    49            dec     ecx
0101DABF    75 F0         jnz     short 0101DAB1
0101DAC1    33C0          xor     eax, eax
0101DAC3    40            inc     eax
0101DAC4    5F            pop     edi
0101DAC5    5B            pop     ebx
0101DAC6    5E            pop     esi
0101DAC7    C9            leave
0101DAC8    C2 0400       retn    4
```

### (12)再次绝望了，彻底的绝望

软件需要输入正确的密码才能正确执行，否则立即结束。

验证密码正确性的算法为单向的（`MD5`、`RSA`）之类算法，所以在软件中只出现明文。

代码的加密/解密算法是分组密码算法，需要正确的密码才能解密。

这个问题，从理论上都是无法解决的。

带着希望来，带着失望去。

有了希望，可以风雨无阻；

没了希望，人生就走到了终点~~

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
