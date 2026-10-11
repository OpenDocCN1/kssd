
# Addiction Pinball

> 原文：[`www.pediy.com/kssd/pediy02/forum275.htm`](https://www.pediy.com/kssd/pediy02/forum275.htm)

Static Vengeance CD 保护破解教程---Addiction Pinball

工具:

1. 游戏:Addiction Pinball
2. `W32Dasm`
3. 十六进制编辑器

Addiction Pinball 是一个基于 win95 平台的游戏,所以破解它并不难。首先要作的当然是安装游戏,然后拿掉 CD 运行一下,看看它说了什么。是不是跳出破框要你放入光盘呢？没问题，让我们解决它。用 `W32Dasm` 反汇编 `pinball.exe` 文件，完成后查找一下“字串数据参考”，你要找 "Insert CD..."，什么？没找到。那只好用“查找”功能找函数"GetDriveTypeA"，这样我们会来到这段代码：

\* Referenced by a CALL at Addresses:

|:00401A8E , :004033BC , :00403BD5 ;从不同的地方 Call 以下代码

|

:0043E8F0 83EC44 sub esp, 00000044
:0043E8F3 53 push ebx
:0043E8F4 56 push esi
:0043E8F5 57 push edi
:0043E8F6 55 push ebp
:0043E8F7 BB02000000 mov ebx, 00000002 ; 我们后面要用到这条指令

\* Reference To: KERNEL32.GetLogicalDrives, Ord:00F9h ; 检测 CD 时常用到的例程

:0043E8FC FF156C934700 Call dword ptr [0047936C]
:0043E902 8BF8 mov edi, eax
:0043E904 8B74245C mov esi, dword ptr [esp+5C]
:0043E908 C64424113A mov [esp+11], 3A
:0043E90D C64424125C mov [esp+12], 5C
:0043E912 C644241300 mov [esp+13], 00

\* Reference To: KERNEL32.GetVolumeInformationA, Ord:014Eh ; 检测 CD 的卷标名

:0043E917 8B2D64934700 mov ebp, dword ptr [00479364]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043E96F(C)

|

:0043E91D B801000000 mov eax, 00000001
:0043E922 8ACB mov cl, bl
:0043E924 D3E0 shl eax, cl
:0043E926 85C7 test edi, eax
:0043E928 7441 je 0043E96B
:0043E92A 8D4361 lea eax, dword ptr [ebx+61]
:0043E92D 88442410 mov byte ptr [esp+10], al
:0043E931 8D442410 lea eax, dword ptr [esp+10]
:0043E935 50 push eax

\* Reference To: KERNEL32.GetDriveTypeA, Ord:00DEh ; 我们被带到这里

:0043E936 FF1568934700 Call dword ptr [00479368]
:0043E93C 83F805 cmp eax, 00000005 ; 05 是光驱的代码值
:0043E93F 752A jne 0043E96B ; 若不等就跳到 CD 检测失败的代码处
:0043E941 8D442414 lea eax, dword ptr [esp+14]
:0043E945 6A00 push 00000000
:0043E947 8D4C2414 lea ecx, dword ptr [esp+14]
:0043E94B 6A00 push 00000000
:0043E94D 6A00 push 00000000
:0043E94F 6A00 push 00000000
:0043E951 6A00 push 00000000
:0043E953 6A40 push 00000040
:0043E955 50 push eax
:0043E956 51 push ecx
:0043E957 FFD5 call ebp
:0043E959 8D4C2414 lea ecx, dword ptr [esp+14]
:0043E95D 56 push esi
:0043E95E 51 push ecx
:0043E95F E8FCDD0100 call 0045C760
:0043E964 83C408 add esp, 00000008
:0043E967 85C0 test eax, eax
:0043E969 7412 je 0043E97D ; 执行这个跳转继续 CD 的检测

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0043E928(C), :0043E93F(C)

|

:0043E96B 43 inc ebx ; CD 循环检测的计数器
:0043E96C 83FB20 cmp ebx, 00000020 ; 最高测试次数为 32 次
:0043E96F 7CAC jl 0043E91D
:0043E971 33C0 xor eax, eax ; EAX 置 0 表示 CD 检测失败
:0043E973 5D pop ebp
:0043E974 5F pop edi
:0043E975 5E pop esi
:0043E976 5B pop ebx
:0043E977 83C444 add esp, 00000044
:0043E97A C20800 ret 0008 ; 返回

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043E969(C)

|

:0043E97D 8B542458 mov edx, dword ptr [esp+58]
:0043E981 85D2 test edx, edx
:0043E983 7423 je 0043E9A8
:0043E985 8D7C2410 lea edi, dword ptr [esp+10]
:0043E989 B9FFFFFFFF mov ecx, FFFFFFFF
:0043E98E 2BC0 sub eax, eax
:0043E990 F2 repnz
:0043E991 AE scasb
:0043E992 F7D1 not ecx
:0043E994 2BF9 sub edi, ecx
:0043E996 8BC1 mov eax, ecx
:0043E998 C1E902 shr ecx, 02
:0043E99B 8BF7 mov esi, edi
:0043E99D 8BFA mov edi, edx
:0043E99F F3 repz
:0043E9A0 A5 movsd
:0043E9A1 8BC8 mov ecx, eax
:0043E9A3 83E103 and ecx, 00000003
:0043E9A6 F3 repz
:0043E9A7 A4 movsb

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043E983(C)

| ; 我们最终想要到达的地方!

:0043E9A8 B801000000 mov eax, 00000001 ; 设置 CD 检测通过的标记值
:0043E9AD 5D pop ebp
:0043E9AE 5F pop edi
:0043E9AF 5E pop esi
:0043E9B0 5B pop ebx
:0043E9B1 83C444 add esp, 00000044
:0043E9B4 C20800 ret 0008 ; 返回

好了，从以上 CD 检测代码的分析，你也能看出我们需要到达 `43E9A8` 才能在 `EAX` 置 1 以通过 CD 检测。那我们该怎么办好呢？这一次我决定找一处代码置入一个 `jump` 指令以直接到达 `43E9A8`。这样程序就会直接跳到"found the CD" 处的代码，并且不会执行 CD 的检测。向上看看，你会发现 `43E8F7` 的指令"mov ebx, 00000002"这是一个很好的插入点，因为它的字节数刚好和一个长跳转相等。那就来吧，将"mov ebx, 00000002"换成"jmp 43E9A8",用十六进制编辑器作如下修改：

Edit `pinball.exe` at offset `253,175`

```
\==================================
Search for: BB 02 00 00 00
Change to : E9 AC 00 00 00
```

ok,任务完成！

Static Vengeance `4dd466d0e1e0cc123afbe504da2462ca`
