
接上面分析 `004C355A` 处实现跳转后，直接按 F9 弹出错误提示，好，我们将注册码改为：`KFW9K-7J2G8-BQT4Y-V3GXV-QM4RF`，然后点击“激活”按钮：

```
004C3580 . 83C4 08 add esp, 8
004C3583 . 85C0 test eax, eax
004C3585 . 75 2E jnz short 004C35B5
004C3587 . 8D45 DC lea eax, dword ptr [ebp-24]
004C358A . 68 80D58E00 push 008ED580
004C358F . 8D4D B4 lea ecx, dword ptr [ebp-4C]
004C3592 . 50 push eax
004C3593 . 51 push ecx
004C3594 . E8 2D9B2000 call <jmp.&mfc42.#operator+\_924>
004C3599 . 8B00 mov eax, dword ptr [eax]
004C359B . 6A 00 push 0
004C359D . 6A 30 push 30
004C359F . 50 push eax
004C35A0 . C645 FC 05 mov byte ptr [ebp-4], 5
004C35A4 . E8 179B2000 call <jmp.&mfc42.#AfxMessageBox\_1200>
004C35A9 . C645 FC 04 mov byte ptr [ebp-4], 4
004C35AD . 8D4D B4 lea ecx, dword ptr [ebp-4C]
004C35B0 . E9 F1000000 jmp 004C36A6
```

`004C3585 . 75 2E jnz short 004C35B5` 处的跳转未实现，提示如下图：

![](https://bbs.kanxue.com/upload/attach/201201/65993_lkyoyjw43ra65e0.jpg)

也就是说此注册码只能激活 3.x 版本。

向下翻看代码，为了节约空间，仅仅截了部分图：

![](https://bbs.kanxue.com/upload/attach/201201/65993_58gkv9sywl21fph.gif)

看上图中的 ASCII `WB400-`，截图的下面还有雷同的的 `WB500-`、`WB600-`、`WB700-`，猜想，WB 不是软件名称 Web Bulder 的每个单词的首字母吗(缩写)，400、500、600、700，不就是版本号 4.x、5.x、6.x 以及 7.x 吗？也就是说，注册码的格式必须以软件名缩写 WB+软件版本开头，进一步大胆猜想，既然此次研究的是8.x版本，注册码应该以 `WB800-` 这样开始，OK,我们就随便构造一组吧，为了简化，我就在 3.x 注册码的基础上冠以 `WB800-`来构造，即等会输入：`WB800-KFW9K-7J2G8-BQT4Y-V3GXV-QM4RF`。直接按 F9 运行让它弹出错误信息，然后输入：`WB800-KFW9K-7J2G8-BQT4Y-V3GXV-QM4RF`(随意构造的一组，当然只能通过一部分验证，不可能完全正确，谁都没那手气吧，随便输入就注册成功！)

经验证 `WB400-`、`WB500-`、`WB600-`、`WB700-`，如果注册码是以这些开头的，则提示“提示此注册码只能激活 [4-7].x 版”的，因此，分析代码就不贴出了。直接进入 8.x 的验证流程如下：

```
004C36BC > \E8 8B952000 call <jmp.&mfc42.#AfxGetModuleState\_1168>
004C36C1 . 8B58 04 mov ebx, dword ptr [eax+4]
004C36C4 . 6A 00 push 0
004C36C6 . 68 44D58E00 push 008ED544 ; ASCII "NagCount"，经过分析，此注册表值项中的值表示注册时因各种原因未能注册成功的次数，最多给你3次机会
004C36CB . 68 88028E00 push 008E0288 ; ASCII "Settings"
004C36D0 . 8BCB mov ecx, ebx ; WebBuild.0092A0A0
004C36D2 . 895D C0 mov dword ptr [ebp-40], ebx ; WebBuild.0092A0A0
004C36D5 . E8 7C9A2000 call <jmp.&mfc42.#CWinApp::GetProfileIntA\_3521> ; 读取 NagCount 的值返给 eax，首次安装并未注册过是其值应该为 0
004C36DA . 8BF0 mov esi, eax ; 即 eax=0
004C36DC . 46 inc esi
004C36DD . 83FE 03 cmp esi, 3
004C36E0 . 7E 58 jle short 004C373A ; 如果未跳转，说明你尝试注册不成功超过3次了；正常时应该跳转
004C36E2 . 6A FF push -1
004C36E4 . 6A 34 push 34
004C36E6 . 68 CC820000 push 82CC
004C36EB . E8 F69A2000 call <jmp.&mfc42.#AfxMessageBox\_1199> ; 我们注意到你试图激活本软件遇到问题，你想访问我们的网站查阅解决方案吗？
```

通过 `004C36E0 . 7E 58 jle short 004C373A` 地址跳转到如下的代码：

```
004C373A > \8BCF mov ecx, edi
004C373C . E8 8F0E0000 call 004C45D0 ; 网络验证 call
004C3741 . 85C0 test eax, eax
004C3743 . 75 4A jnz short 004C378F ; 必须跳
004C3745 . 6A FF push -1
004C3747 . 6A 30 push 30
004C3749 . 68 EF730000 push 73EF
004C374E . E8 939A2000 call <jmp.&mfc42.#AfxMessageBox\_1199> ; 提示网络不通
```

好了，我们进入网络验证 `call` 中看看(为了进入网络验证模块，我们在它的上一行地址处下段(F2)),就像下图那样：

![](https://bbs.kanxue.com/upload/attach/201201/65993_gvxsjtota78vjrf.gif)

然后输入构造的注册码：`WB800-KFW9K-7J2G8-BQT4Y-V3GXV-QM4RF`，直到停在上图中的 `004C373A` 处，F8 和 F7 各一次就进入网络验证 `call` 中。我在分析此款软件时不知道坛中 playboysen 已经发布有关于“WYSIWYG Web Build V5.5.4 算法+网络验证分析”，之后我上传了本版的注册机后通过帖子中的“相似主题”才发现，如果早发现，对于此款软件的分析本人就不需要走很多弯路了，晕死！基于节约版面和尽可能缩短帖子的长度，仅将关键部分贴出(爱好者想进一步了解本款软件的网络验证机制请查看 playboysen 的帖子或自行研究)，关键部分如下：

```
004C4726 |. 8B45 E8 mov eax, [local.6] ; 通过网络验证，局部变量 local.6 中应该返回(ASCII "<WWB8>OK</WWB8>")
004C4729 |. 68 3CD68E00 push 008ED63C ; //s2 = "<WWB8>OK</WWB8>"，为软件值预置的值
004C472E |. 50 push eax ; |s1 = 网络验证的返回值，正确的话应为：<WWB8>OK</WWB8>
004C472F |. FF15 0017BB00 call dword ptr [<&msvcrt.\_mbscmp>] ; \\_mbscmp
004C4735 |. 83C4 08 add esp, 8
004C4738 |. C645 FC 04 mov byte ptr [ebp-4], 4
004C473C |. 85C0 test eax, eax ; 相等则 eax=0
004C473E |. 8D4D E8 lea ecx, [local.6]
004C4741 |. 75 4E jnz short 004C4791 ; 不能跳
004C4743 |. E8 D4842000 call <jmp.&mfc42.#CString::~CString\_800>
004C4748 |. 8D4D C0 lea ecx, [local.16]
004C474B |. C645 FC 03 mov byte ptr [ebp-4], 3
004C474F |. E8 328D2000 call <jmp.&mfc42.#CInternetSession::~CInternetS>
004C4754 |. 8D4D DC lea ecx, [local.9]
004C4757 |. C645 FC 01 mov byte ptr [ebp-4], 1
004C475B |. E8 BC842000 call <jmp.&mfc42.#CString::~CString\_800>
004C4760 |. 8D4D E4 lea ecx, [local.7]
004C4763 |. C645 FC 00 mov byte ptr [ebp-4], 0
004C4767 |. E8 B0842000 call <jmp.&mfc42.#CString::~CString\_800>
004C476C |. 8D4D E0 lea ecx, [local.8]
004C476F |. C745 FC FFFFF>mov [local.1], -1
004C4776 |. E8 A1842000 call <jmp.&mfc42.#CString::~CString\_800>
004C477B |. B8 01000000 mov eax, 1
004C4780 |. 8B4D F4 mov ecx, [local.3]
004C4783 |. 64:890D 00000>mov dword ptr fs:[0], ecx
004C478A |. 5F pop edi ; 0012F180
004C478B |. 5E pop esi ; 0012F180
004C478C |. 5B pop ebx ; 0012F180
004C478D |. 8BE5 mov esp, ebp
004C478F |. 5D pop ebp ; 0012F180
004C4790 |. C3 retn
```

并返回 `eax==1`，好，我们从网络验证模块返回后通过跳转来到下面的代码：

```
004C378F > \56 push esi
004C3790 . 68 44D58E00 push 008ED544 ; ASCII "NagCount"
004C3795 . 68 88028E00 push 008E0288 ; ASCII "Settings"
004C379A . 8BCB mov ecx, ebx ; WebBuild.0092A0A0
004C379C . E8 BB992000 call <jmp.&mfc42.#CWinApp::WriteProfileInt\_6402>
004C37A1 . 8B4D E8 mov ecx, dword ptr [ebp-18]
004C37A4 . 51 push ecx
004C37A5 . 8B4F 60 mov ecx, dword ptr [edi+60]
004C37A8 . E8 030C0300 call 004F43B0
004C37AD . 8B55 E4 mov edx, dword ptr [ebp-1C]
004C37B0 . 8BCB mov ecx, ebx ; WebBuild.0092A0A0
004C37B2 . 52 push edx
004C37B3 . 68 E83A8E00 push 008E3AE8 ; ASCII "Name"
004C37B8 . 68 88028E00 push 008E0288 ; ASCII "Settings"
004C37BD . E8 A8942000 call <jmp.&mfc42.#CWinApp::WriteProfileStringA\_>
004C37C2 . 8B45 E4 mov eax, dword ptr [ebp-1C] ; 输入的 Email
004C37C5 . 8B4F 60 mov ecx, dword ptr [edi+60]
004C37C8 . 50 push eax
004C37C9 . E8 A20F0300 call 004F4770 ; 将输入的 Email 复制到指针 ecx 指向的内存中
004C37CE . 8B4F 60 mov ecx, dword ptr [edi+60]
004C37D1 . E8 FA0B0300 call 004F43D0 ; 将注册信息的一部分(因完整的注册信息还有另一部分)写入到注册表中
```

我们进入 `call 004F43D0` 看看到底做了什么手脚。

```
004F4350 /$ 64:A1 0000000>mov eax, dword ptr fs:[0]
004F4356 |. 6A FF push -1
004F4358 |. 68 88EB7E00 push WebBuild.007EEB88
004F435D |. 50 push eax
004F435E |. 64:8925 00000>mov dword ptr fs:[0], esp
004F4365 |. 83EC 10 sub esp, 10
004F4368 |. 56 push esi
004F4369 |. 8BF1 mov esi, ecx
004F436B |. 57 push edi
004F436C |. 68 58F88E00 push WebBuild.008EF858 ; /String2 = "NotSet"
004F4371 |. 8D7E 08 lea edi, dword ptr [esi+8] ; |
004F4374 |. 57 push edi ; |String1
004F4375 |. FF15 D8A28200 call dword ptr [<&kernel32.lstrcpy>] ; \lstrcpyA
004F437B |. 8BCE mov ecx, esi
004F437D |. E8 8EFAFFFF call WebBuild.004F3E10 ; 注册码算法 call
004F4382 |. 85C0 test eax, eax ; 乱输入的注册码 eax=0
004F4384 |. 0F84 FF000000 je WebBuild.004F4489 ; 跳转不会向注册表写入注册信息
```

经过分析 `004F437D |. E8 8EFAFFFF call WebBuild.004F3E10` 处的 `call` 为注册码算法验证，因此，先进入此 `call` 看看是如何验证的：

仅贴出关键部分：

第一部分：

```
004F3E7E |. 8DB7 08040000 lea esi, dword ptr [edi+408] ; 输入的注册码
004F3E84 |. 6A 2D push 2D ; 很熟悉吧，2Dh="-"，注册码分隔符
004F3E86 |. 53 push ebx
004F3E87 |. 8D4424 2C lea eax, dword ptr [esp+2C]
004F3E8B |. 56 push esi
004F3E8C |. 50 push eax
004F3E8D |. C64424 68 05 mov byte ptr [esp+68], 5
004F3E92 |. E8 178F1D00 call <jmp.&mfc42.#AfxExtractSubString\_1140> ; 分割字符串函数，以 "-" 作为分割的依据将注册码分割
```

第二部分：

```
004F3EEF |. 56 push esi ; //String，输入的注册码
004F3EF0 |. FF15 CCA28200 call dword ptr [<&kernel32.lstrlen>] ; \lstrlenA
004F3EF6 |. 83F8 1E cmp eax, 1E ; 注册码长度23h(即10进制35)与 1Eh(即10进制的 30)
004F3EF9 |. 0F85 61030000 jnz WebBuild.004F4260 ; 不等 1Eh，则注册信息不正确
```

怎么不等于呢？纳闷，向下翻看发现：

![](https://bbs.kanxue.com/upload/attach/201201/65993_fv7bu2x1wn3555z.gif)

搞这些名堂，为了看清楚它的真面目，把我的脖子都搞歪了，阿拉伯语序式排列(从右到左，反序)，这个是不是注册码呢，刚好 30 的长度(包括分隔符)，那试试吧！

(Game Over)大侠你的游戏功底不足，请重新来过吧！晕！@！#

将所谓的注册码正序排列后为：`WB800-439F-V899-7R89-CDB0-4X47`(如果注册码长相确实如此，则应为 5-4-4-4-4-4，即第一组长度 5位，剩余的5组长度为 4 位，加上分隔符 "-"，共30位)，输入此注册码继续研究（仅贴出关键的代码）：

```
004F3EEF |. 56 push esi ; //String，输入的注册码
004F3EF0 |. FF15 CCA28200 call dword ptr [<&kernel32.lstrlen>] ; \lstrlenA
004F3EF6 |. 83F8 1E cmp eax, 1E ; 注册码长度23h(即10进制35)与 1Eh(即10进制的 30)
004F3EF9 |. 0F85 61030000 jnz WebBuild.004F4260 ; 不等 1Eh，则注册信息不正确
```
