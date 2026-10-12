
# 股霸 5.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue138.htm`](https://www.pediy.com/kssd/pediy04/kanxue138.htm)

知道这个论坛也将近一年了，经常来这里听各位的高论，受益匪浅，看了论坛精华 2 和 3 之后，禁不住手痒，也学着破软件，可功力浅，只好专捏“软柿子”了。（这是我第一次写破解，有言不达意之处，请多多包涵。＾O＾）

**软件**：股霸 5.0

**功能**：通视卡用户接收东方上证信息工具，有通视卡的兄弟可以试试，可能我不会用，算出注册码注册成功后仍就接收不到股市信息（不过拿来练练手倒不错）。

**下载地址**：`http://stockhot.my.west163.com/down/Gb50SHM.exe`

**软件保护**：加壳，代码采用花指令法加密，使用了 `cih` 的技术（使 `TRW2000` 在某些地址不能中断），注册码在 DLL 文件中计算对比（注册码算法中使用了硬件信息）。

**工具**：`TRW2000`, `WinHex9.5`, `caspr1.011`(一个非常好的脱壳软件，我喜欢，感谢作者的不懈努力）, `fs2k55`(也是一个很好的侦测加壳类型的软件），`w32dasm8.93`。

**过程**：

运行主程序 `sdlzc.exe` 后，输入注册码“`787878787878787878`”一共十六位，按现在注册按钮后，出现“对不起，注册失败！！！”注册错误消息窗口，至此，通常破解方法为设断 `bpx MessageBoxA`,然后采用逆推法，寻找能跳过 `CALL MessageBoxA` 的地方，这样可以尽快找到核心比较代码（大部分程序是这样的吧）。

用 `fs2k55` 侦测文件，发现有加壳，用 `caspr` 让它一丝不挂吧。

用 `w32dasm` 反汇编，寻找“对不起，注册失败！！！”的错误信息，Ok!, `w32dasm` 总是不会让人失望，在 `0040129A` 地址找到了这个错误信息，好！，往上找，原来是从 `00401266` 跳来的，看来 `0040125F` 的 `call 00401822` 必有问题，追进去看看。

```asm
:00401249 817D0C11010000          cmp dword ptr [ebp+0C], 00000111
:00401250 0F8584000000            jne 004012DA
:00401256 817D10E8030000          cmp dword ptr [ebp+10], 000003E8
:0040125D 7550                    jne 004012AF
:0040125F E8BE050000              call 00401822
:00401264 0BC0                    or eax, eax   *eax 如果为 0 则死,不为 0 则注册成功。
:00401266 752A                    jne 00401292
:00401268 6A00                    push 00000000
:0040126A FF35FF684000            push dword ptr [004068FF]
* Possible StringData Ref from Data Obj ->"恭喜您，注册成功！"
                  |
:00401270 68CD624000              push 004062CD
:00401275 FF358A644000            push dword ptr [0040648A]
* Reference To: USER32.MessageBoxA, Ord:01BBh
                  |
:0040127B E8560D0000              Call 00401FD6
:00401280 6A00                    push 00000000
:00401282 FF358A644000            push dword ptr [0040648A]
* Possible StringData Ref from Data Obj ->"对不起，注册失败！！！"
                  |
:0040129A 68E0624000              push 004062E0
:0040129F FF358A644000            push dword ptr [0040648A]
* Reference To: USER32.MessageBoxA, Ord:01BBh
                  |
:004012A5 E82C0D0000              Call 00401FD6
:004012AA E9B9000000              jmp 00401368
```

`0040125F` 的 `call 00401822` 果然有问题，以下见代码中注解。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040101E(U)
|

```asm
:00401822 55                      push ebp
:00401823 8BEC                    mov ebp, esp
:00401825 83C4F8                  add esp, FFFFFFF8
:00401828 60                      pushad
:00401829 B805000000              mov eax, 00000005
:0040182E 8945FC                  mov dword ptr [ebp\-04], eax
:00401831 6A1E                    push 0000001E
:00401833 68A5624000              push 004062A5
:00401838 6A64                    push 00000064
:0040183A FF358A644000            push dword ptr [0040648A]
* Reference To: KERNEL32.lstrlenA, Ord:02E2h
                  |
:00401865 E890070000              Call 00401FFA
:0040186A 83F810                  cmp eax, 00000010   *这里比较注册码长度，不等于 16 位则死
:0040186D 7516                    jne 00401885
:0040186F 6857634000              push 00406357
:00401874 6853634000              push 00406353
:00401879 68A5624000              push 004062A5
* Reference To: sdldll.GetDateInKJMReg, Ord:000Ah
                  |
:0040187E E8430B0000              Call 004023C6    *按 f10 到这里后，再按 f10 则直接出现注册
:00401883 EB0C                    jmp 00401891    *失败的错误信息，这里也有问题，追吧。
```

`sdldll.dll` 也有加壳，直接用 `caspr` 脱。

Exported fn(): `GetDateInKJMReg` - Ord:000Bh

```asm
:10007FFA 55                      push ebp
:10007FFB 8BEC                    mov ebp, esp
:10007FFD 83C4F8                  add esp, FFFFFFF8
:10008000 60                      pushad
:10008001 8B7D08                  mov edi, dword ptr [ebp+08]
:10008004 8D35BEF80010            lea esi, dword ptr [1000F8BE]
:1000800A 56                      push esi
:1000800B 57                      push edi
:1000800C 6A09                    push 00000009
:1000800E 57                      push edi
:1000800F 68AAF80010              push 1000F8AA
* Reference To: KERNEL32.lstrcpynA, Ord:02DFh
                  |
:10008037 E806070000              Call 10008742
:1000803C 68AAF80010              push 1000F8AA
:10008041 E86E050000              call 100085B4
:10008046 5F                      pop edi
:10008047 5E                      pop esi
:10008048 8906                    mov dword ptr [esi], eax
:1000804A 8D35BEF80010            lea esi, dword ptr [1000F8BE]
:10008050 8B06                    mov eax, dword ptr [esi]
:10008052 8945FC                  mov dword ptr [ebp\-04], eax
* Reference To: sdldll.GetJQZW
                  |
:10008055 E866FEFFFF              call 10007EC0    *此 call 获得硬件信息
:1000805A 50                      push eax
:1000805B FF1590F80010            call dword ptr [1000F890]   *此 F(1)对 EAX 中硬件信息作复杂转换
:10008061 83C404                  add esp, 00000004
:10008064 A38FF90010              mov dword ptr [1000F98F], eax
:10008069 3345FC                  xor eax, dword ptr [ebp\-04]  **[ebp\-4]中为注册码前 8 位
:1000806C 50                      push eax
:1000806D FF158CF80010            call dword ptr [1000F88C]   *此 F(2)对 EAX 作复杂转换（看的头都
:10008073 83C404                  add esp, 00000004   *大了还是不明白算法）
:10008076 6652                    push dx
:10008078 6650                    push ax
:1000807A 58                      pop eax       *EAX 为上 call 结果
:1000807B EB02                    jmp 1000807F
:1000807D EA3360EB026967          jmp 6769:02EB6033   ***这里以后采用了“花指令”法。***
:10008084 EB02                    jmp 10008088
```

我倒，用 `trw2000` 追到 `1000807B` 后，按一下 `F8`，代码变一变，再按几下 `F8`，前后代码都不对了（挂在半空的滋味真不错，你也来尝尝。），原来作者用了“花指令”，经仔细分析后，祭出了 `winHex`,修改后代码如下。

```asm
:1000807D 90                      nop
:1000807E 90                      nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000807B(U)
|
:1000807F 60                      pushad
:10008080 EB02                    jmp 10008084
:10008082 90                      nop
:10008083 90                      nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10008080(U)
|
:10008084 EB02                    jmp 10008088
```

这样看起来顺眼多了。

噫！追着追着怎么出现了......

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10008089(U)
|

```asm
:1000808D B903000000              mov ecx, 00000003
:10008092 8D05BB550010            lea eax, dword ptr [100055BB]   **这里为修改 int 3 入口作准备
:10008098 8B1D02A00010            mov ebx, dword ptr [1000A002]
:1000809E EB02                    jmp 100080A2
:100080A0 90                      nop
:100080A1 90                      nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000809E(U)
|
:100080A2 8D1CCB                  lea ebx, dword ptr [ebx+8*ecx]
:100080A5 668903                  mov word ptr [ebx], ax    **这里修改 int 3 入口
:100080A8 EB02                    jmp 100080AC
:100080AA 90                      nop
:100080AB 90                      nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100080A8(U)
|
:100080AC C1E810                  shr eax, 10
:100080AF EB02                    jmp 100080B3
:100080B1 90                      nop
:100080B2 90                      nop
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100080AF(U)
```



:100080B3 66894306 `mov word ptr [ebx+06], ax`

:100080B7 EB02 `jmp 100080BB`

:100080B9 90 `nop`

:100080BA 90 `nop`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:100080C0(U)`

:100080C4 CC `int 03` **这里是 cih 病毒的技术，获得了 0 级特权，它想做什么呢？？自己 `U 100055BB` 看看吧。**

**这里 `F8`、`F10` 都死，只好在 `100055BB` 中作作手脚了。**

继续追吧......

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10008111(U)`

:10008115 BB34122989 `mov ebx, 89291234`

:1000811A 81E3FFFF0000 `and ebx, 0000FFFF`

:10008120 EB02 `jmp 10008124`

:10008122 90 `nop`

:10008123 90 `nop`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:10008120(U)`

\* Reference To: `sdldll.NewKjmSuanFa`

:10008124 E824D0FFFF `call 1000514D` **关键，开机码（注册码前 8 位）算法，算法很简单**

:10008129 EB02 `jmp 1000812D` **（大概为 `eax xor 1234h`，`eax xor 52071197`，`eax xor 12345678`）但不容易找到计算子程序，此子程序代替了 `int 1` 处理程序，`trw2000` 没办法追。**

来到了......

:100081C3 80FC09 `cmp ah, 09` **`eax` 为注册码前 8 位经计算后内容，看来此程序只允许**

:100081C6 7704 `ja 100081CC` **用到 2009 年 12 月，程序中另有一处比较当前日期。**

:100081C8 3C12 `cmp al, 12`

:100081CA 7606 `jbe 100081D2`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:100081C6(C)`

:100081CC EB02 `jmp 100081D0`

:100081CE 90 `nop`

:100081CF 90 `nop`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:100081CC(U)`

:100081D0 33C0 `xor eax, eax` **`EAX` 为 0 则注册错误。**

以下比较计算注册码后 8 位。

:10008271 8D35BEF80010 `lea esi, dword ptr [1000F8BE]` \*注册码前 8 位地址

:10008277 FC `cld`

:10008278 AD `lodsd`

:10008279 33058FF90010 `xor eax, dword ptr [1000F98F]` \*经转换后硬件信息地址

:1000827F 8945F8 `mov dword ptr [ebp-08], eax`

:10008282 AD `lodsd`

:10008283 33058FF90010 `xor eax, dword ptr [1000F98F]` \*注册码后 8 位地址

:10008289 50 `push eax`

:1000828A FF158CF80010 `call dword ptr [1000F88C]` \*同前面 `:1000806D` 之 `F(2)`

:10008290 83C404 `add esp, 00000004`

:10008293 6652 `push dx`

:10008295 6650 `push ax`

:10008297 58 `pop eax` \*注册码后 8 位经转换后内容

:10008298 3345F8 `xor eax, dword ptr [ebp-08]`

:1000829B 66B90800 `mov cx, 0008`

:1000829F 8BD8 `mov ebx, eax`

:100082A1 53 `push ebx`

:100082A2 83E30F `and ebx, 0000000F`

:100082A5 83FB09 `cmp ebx, 00000009`

:100082A8 7602 `jbe 100082AC`

:100082AA 33C0 `xor eax, eax` \*`EAX` 为 0 注册失败，必须跳过。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:100082A8(C)`

:100082AC 5B `pop ebx`

:100082AD C1EB04 `shr ebx, 04`

:100082B0 E2EF `loop 100082A1`

:100082B2 8B7510 `mov esi, dword ptr [ebp+10]`

:100082B5 8906 `mov dword ptr [esi], eax`

:100082B7 61 `popad`

:100082B8 C9 `leave`

:100082B9 C20C00 `ret 000C`

至此程序注册算法基本上已弄明白，有兴趣的朋友可写注册机了（函数 `F(1)` 和 `F(2)` 互逆）。
`4dd466d0e1e0cc123afbe504da2462ca`
