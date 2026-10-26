
# [原创]菜鸟也来内存注册机

-------------------------------------------------【文章简介】-----------------------------------

【文章标题】 菜鸟使用内存注册机

【文章作者】 激动的马

【软件名称】 某屏幕录像器ZD Soft Screen Recorder

【软件大小】 2M

【下载地址】 0bbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4J5x3e0M7K6i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0j5$3x3o6p5&6i4K6u0W2K9s2c8E0L8l9``.

【加壳方式】 PECompact 2.x -> Jeremy Collake

【保护方式】 加壳

【编写语言】 VC++6.0

【使用工具】 Od

【操作平台】 windows

【软件介绍】 自己搜索

【作者声明】 大牛请飘过，仅供小菜玩乐,有不对或不足的地方还请多多指教，不胜感激!

-------------------------------------------------【文章正文】-------------------------------------

## 一.脱壳。

软件安装以后先用`peid`查壳是`PECompact 2.x -> Jeremy Collake`，所以直接用`UnPECompact2 v0.1`脱了（手工脱对我来说难度太大）。运行了一下发现原来华丽的界面没了不知道怎么回事不过用这算出的注册码是通用的。

![](https://bbs.kanxue.com/upload/attach/201411/583257_20ljviy9g6pit2s.jpg) ![](https://bbs.kanxue.com/upload/attach/201411/583257_h0o3npwfrf02vp6.jpg)

没脱壳 脱壳后

## 二.脱完壳下面开始进入正题了。

用`od`加载软件，然后在选项-》关于可以看到免费试用的时间是30天。需要注册才能继续使用。先随便输入用户、邮箱、密码，设置`bp` `GetWindowTextW`（先用`ctrl+N`查下）。

![](https://bbs.kanxue.com/upload/attach/201411/583257_1phnj82tghar1nt.jpg)

由于这个`api`会断在很多地方，所以要设置条件，点击`w`按钮可以看到很多信息（没有内容就刷新下）。

![](https://bbs.kanxue.com/upload/attach/201411/583257_8h1mtayomnksswi.jpg)

这里我对`2002f0`句柄下条件断点。

![](https://bbs.kanxue.com/upload/attach/201411/583257_6oo31f9hfxsrtgs.jpg)

确定后到注册的界面点击确定。会断下来再`alt+f9`回到应用程序的领空。可以在消息窗口中看到获取了用户名。

```
004184F2   .  E8 01BC0800   call <jmp.&MFC42u.#DDX_Text_2362>
004184F7   .  8D4E 64       lea ecx, dword ptr ds:[esi+0x64]；alt+F9回到这里
004184FA   .  51            push ecx             ；获取用户名
004184FB   .  68 4E4F0000   push 0x4F4E
00418500   .  57            push edi
00418501   .  E8 F2BB0800   call <jmp.&MFC42u.#DDX_Text_2362>；获取邮箱内容
00418506   .  83C6 68       add esi, 0x68
00418509   .  56            push esi
0041850A   .  68 4F4F0000   push 0x4F4F
0041850F   .  57            push edi
00418510   .  E8 E3BB0800   call <jmp.&MFC42u.#DDX_Text_2362> ；获取密码
00418515   .  5F            pop edi
00418516   .  5E            pop esi
00418517   .  C2 0400       retn 0x4
```

一直`f8`到`retn 0x4`，会到`mfc`中直接`alt+f9`回到这里来：

```
00418538   .  E8 03BC08>call <jmp.&MFC42u.#CWnd::UpdateData_6330>
0041853D   .  8D6E 60   lea ebp, dword ptr ds:[esi+0x60]  ；  来到这里
00418540   .  8BCD      mov ecx, ebp
00418542   .  E8 E9BC08>call <jmp.&MFC42u.#CString::TrimLeft_6278>；从函数名可以看出对用户名进行去左右空格的处理，下面也是一样。
00418547   .  8BCD      mov ecx, ebp
00418549   .  E8 DCBC08>call <jmp.&MFC42u.#CString::TrimRight_6279>
0041854E   .  8D7E 64   lea edi, dword ptr ds:[esi+0x64]
00418551   .  8BCF      mov ecx, edi
00418553   .  E8 D8BC08>call <jmp.&MFC42u.#CString::TrimLeft_6278> ;邮箱
00418558   .  8BCF      mov ecx, edi
0041855A   .  E8 CBBC08>call <jmp.&MFC42u.#CString::TrimRight_6279>
0041855F   .  8BCF      mov ecx, edi
00418561   .  E8 A2BD08>call <jmp.&MFC42u.#CString::MakeLower_4197>；转化成小写
00418566   .  8D5E 68   lea ebx, dword ptr ds:[esi+0x68]
00418569   .  8BCB      mov ecx, ebx
0041856B   .  E8 C0BC08>call <jmp.&MFC42u.#CString::TrimLeft_6278>；  密码
00418570   .  8BCB      mov ecx, ebx
00418572   .  E8 B3BC08>call <jmp.&MFC42u.#CString::TrimRight_6279>
00418577   .  8BCB      mov ecx, ebx
00418579   .  E8 84BD08>call <jmp.&MFC42u.#CString::MakeUpper_4199>   转化成大写
0041857E   .  8B6D 00   mov ebp, dword ptr ss:[ebp] ；          保存用户名
00418581   .  8B45 F8   mov eax, dword ptr ss:[ebp-0x8]；             保存密码
00418584   .  85C0      test eax, eax
00418586   .  0F84 C500>je ScnRec\_.00418651 ；            密码为空出错
0041858C   .  8B07      mov eax, dword ptr ds:[edi]
0041858E   .  8B48 F8   mov ecx, dword ptr ds:[eax-0x8]
00418591   .  85C9      test ecx, ecx
00418593   .  0F84 B800>je ScnRec\_.00418651 ；
00418599   .  8B0B      mov ecx, dword ptr ds:[ebx]；          邮箱
0041859B   .  8B41 F8   mov eax, dword ptr ds:[ecx-0x8]
0041859E   .  85C0      test eax, eax
004185A0   .  0F84 AB00>je ScnRec\_.00418651
004185A6   .  8B4E 6C   mov ecx, dword ptr ds:[esi+0x6C]
004185A9   .  55        push ebp
004185AA   .  83C1 04   add ecx, 0x4
004185AD   .  E8 22BB08>call <jmp.&MFC42u.#CString::operator=_861>
004185B2   .  8B17      mov edx, dword ptr ds:[edi]
004185B4   .  8B4E 6C   mov ecx, dword ptr ds:[esi+0x6C]
004185B7   .  52        push edx
004185B8   .  83C1 08   add ecx, 0x8
004185BB   .  E8 14BB08>call <jmp.&MFC42u.#CString::operator=_861>
004185C0   .  8B03      mov eax, dword ptr ds:[ebx]
004185C2   .  8B4E 6C   mov ecx, dword ptr ds:[esi+0x6C]
004185C5   .  50        push eax          ；密码压入
004185C6   .  E8 65F600>call ScnRec\_.00427C30                     ；先跟进看看
004185CB   .  8B4E 6C   mov ecx, dword ptr ds:[esi+0x6C]
004185CE   .  E8 5DFC00>call ScnRec\_.00428230                     ;关键call
004185D3   .  85C0      test eax, eax
004185D5   .  75 3A     jnz XScnRec\_.00418611
004185D7   .  E8 22BB08>call <jmp.&MFC42u.#AfxGetThread_1172>
004185DC   .  85C0      test eax, eax
004185DE   .  74 09     je XScnRec\_.004185E9
004185E0   .  8B10      mov edx, dword ptr ds:[eax]
004185E2   .  8BC8      mov ecx, eax
004185E4   .  FF52 7C   call Xdword ptr ds:[edx+0x7C]
004185E7   .  EB 02     jmp XScnRec\_.004185EB
004185E9   >  33C0      xor eax, eax
004185EB   >  6A 00     push 0x0
004185ED   .  6A 00     push 0x0
004185EF   .  6A 00     push 0x0
004185F1   .  6A 00     push 0x0
004185F3   .  6A 00     push 0x0
004185F5   .  6A 00     push 0x0
004185F7   .  6A 00     push 0x0
004185F9   .  6A 01     push 0x1
004185FB   .  6A 10     push 0x10
004185FD   .  6A 00     push 0x0
004185FF   .  68 AE4F00>push 0x4FAE
00418604   .  8BC8      mov ecx, eax
00418606   .  56        push esi
00418607   .  E8 745C00>call ScnRec\_.0041E280  ；执行这句就会出现出错窗口了
0041860C   .  5F        pop edi
0041860D   .  5E        pop esi
0041860E   .  5D        pop ebp
0041860F   .  5B        pop ebx
00418610   .  C3        retn
```

这里是`427c30`的`call`：

```
00427C30  /$  51        push ecx
00427C31  |.  53        push ebx
00427C32  |.  55        push ebp
00427C33  |.  56        push esi
00427C34  |.  83C1 0C   add ecx, 0xC
00427C37  |.  57        push edi
00427C38  |.  894C24 10 mov dword ptr ss:[esp+0x10], ecx
00427C3C  |.  E8 4DC507>call <jmp.&MFC42u.#CString::Empty_2606>
00427C41  |.  8B7C24 18 mov edi, dword ptr ss:[esp+0x18]
00427C45  |.  85FF      test edi, edi
00427C47  |.  74 61     je XScnRec\_.00427CAA
00427C49  |.  33ED      xor ebp, ebp
00427C4B  |>  33DB      /xor ebx, ebx                             ；大循环5次
00427C4D  |>  66:8B37   |/mov si, word ptr ds:[edi]      ；小循环5次
00427C50  |.  83C7 02   ||add edi, 0x2
00427C53  |.  66:85F6   ||test si, si
00427C56  |.  74 52     ||je XScnRec\_.00427CAA
00427C58  |.  56        ||push esi                                  ; /w
00427C59  |.  FF15 CC36>||call Xdword ptr ds:[<&MSVCRT.iswdigit>]   ; \iswdigit
00427C5F  |.  83C4 04   ||add esp, 0x4                          ；判断是否是数字
00427C62  |.  85C0      ||test eax, eax
00427C64  |.  75 11     ||jnz XScnRec\_.00427C77
00427C66  |.  56        ||push esi                                  ; /w
00427C67  |.  FF15 D036>||call Xdword ptr ds:[<&MSVCRT.iswalpha>]   ; \iswalpha
00427C6D  |.  83C4 04   ||add esp, 0x4                           ；判断是否是字母
00427C70  |.  85C0      ||test eax, eax
00427C72  |.  75 03     ||jnz XScnRec\_.00427C77
00427C74  |.  4B        ||dec ebx
00427C75  |.  EB 0A     ||jmp XScnRec\_.00427C81
00427C77  |>  8B4C24 10 ||mov ecx, dword ptr ss:[esp+0x10]
00427C7B  |.  56        ||push esi
00427C7C  |.  E8 B9C707>||call <jmp.&MFC42u.#CString::operator+=_94>；字符串连接操作
00427C81  |>  43        ||inc ebx
00427C82  |.  83FB 05   ||cmp ebx, 0x5
00427C85  |.^ 7C C6     |\jl XScnRec\_.00427C4D
00427C87  |.  8D75 01   |lea esi, dword ptr ss:[ebp+0x1]
00427C8A  |.  83FE 05   |cmp esi, 0x5
00427C8D  |.  7D 0B     |jge XScnRec\_.00427C9A
00427C8F  |.  8B4C24 10 |mov ecx, dword ptr ss:[esp+0x10]
```
