
# Turbo Note+ V4.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue220.htm`](https://www.pediy.com/kssd/pediy04/kanxue220.htm)

## Turbo Note+ V4.4 注册机制分析

**作者**: 囚童\[FCG\]\[BCG\]   
**课题**: Turbo Note+ V4.4 注册机制分析   
**出品**: South Pacific InformationServices Ltd   
**背景**: PACKED WITH Thrink v3   
**平台**: win95/98, winNT/2000   
**下载**: [http://turbopress.com/tnplus44.exe](http://turbopress.com/tnplus44.exe)   
**尺寸**: 769KB   
**工具**: TRW 2000, 冲击波 2000, W32Dasm V8.93   
**关键字**: 软件破解 注册码

### 软件背景

`TurboNote+`是一个方便的“即时帖”软件，可以工作在 `Win95/98`、`WinNT` 和 `Win2000` 环境下。它可以即时在自己或通过 IP 地址向其他人的屏幕上贴上一个可定义尺寸和颜色的"留言条"，也可以按预定的时间启动备忘提示功能。

通过"留言条"，可以方便地启动可执行程序、互联网址、`E-mail` 地址，也可以把它们作为附件发送给指定的 IP 地址。它可以隐藏，也可以再现，甚至关机后信息也不会丢失。使用十分方便。

`TurboNote` V4.4 未注册版有 30 天的时间限制。过期后会随时出现延时的注册窗口，程序仍能使用。

`TurboNote+` 是一个被压缩了的可执行软件，用一般的方法无法修改。它的脱壳较简单，用 `冲击波 2000` 找到程序入口，在 `TRW2000` 下定位入口，再用 `MAKEPE` 命令很容易就脱壳成功。

`TurboNote+` 在注册时不出现明码，大多数情况下，输入了错误的注册码得不到提示，因此用动态跟踪寻找注册入口较困难。相反，用静态分析却相当简单。

### 注册机制静态分析

以下是对其注册机制的分析:

用 `W32Dasm` 打开 `TurboNote+` 主文件 `TBNOTE.EXE`，以 `REGISTER` 为关键字搜索，定位在：

```
\* Possible StringData Ref from Code Obj ->"There was a problem when saving "
                                      ->"the registry key Try again "
                                      ->"please."
                                      |
:00412BF4  push 00470968
:00412BF9  push eax
```

向上看，可以找到：

```
\* Reference To: KERNEL32.lstrlenA， Ord:0000h
                       |
:00412B4D　Call　dword ptr \[00467128\]
:00412B53　cmp 　eax， 00000014　　　　　　　　　　　　　　　<==串长≥Ox14
:00412B56　jl  　00412DD8
:00412B5C　push　00000064
:00412B5E　lea 　ecx， dword ptr \[ebp\-44\]
:00412B61　call　0042AA60
:00412B66　xor 　ebx， ebx
:00412B68　push　00000019
:00412B6A　lea 　ecx， dword ptr \[ebp\-50\]
:00412B6D　mov 　dword ptr \[ebp\-04\]， ebx
:00412B70　call　0042AA60
```

嗯，有点象，注册码长度可能大于 20 个字符。怪不得输错了注册码得不到提示，因为大多数人一开始是不会输入那么长的一串码的。

### 动态跟踪与调试

启动 `TRW2000`，点击 `OK` 使其最小化。

双击 `TurboNote+` 图标，`TurboNote+` 在屏幕右下脚生成一个 `TurboNote+` 图标。

右击这个图标，会弹出一个右键菜单，选 `GENERAL SETTINGS`，`GENERAL SETTINGS` 窗口弹出，点 `REGISTER TURBONOTE+` 选项卡，该卡被调到窗口最前面，在 `REGISTRATION CODE:` 框中随便输入 20 个字母数字，我们后面称它串。注意，`TurboNote+` 的注册码拒绝键盘上的某些字符，例如`[BCG]`的方括号。

按 `CTRL+N`，激活 `TRW`。

下指令: `BPX 412B53`，按 `F5`，`TurboNote+` 再次被 `TRW` 截获。

按 `F10`, 一路走一路看，来到：

```
\* Possible StringData Ref from Code Obj ->"SOFTWARE\\SPIS Ltd\\TurboNote\\"
                                       ->"Registration"
                                      |
:00412B86　push　004709B4
:00412B8B　push　80000002
:00412B90　lea 　ecx, dword ptr \[ebp\-6C\]
:00412B93　mov 　\[ebp\-04\], 02
:00412B97　call　0044CA50
:00412B9C　push　0000012C
:00412BA1　lea 　ecx, dword ptr \[ebp\-20\]
:00412BA4　mov 　\[ebp\-04\], 03
:00412BA8　call　0042AA60
:00412BAD　push　edi
:00412BAE　lea 　ecx, dword ptr \[ebp\-20\]
:00412BB1　mov 　\[ebp\-04\], 04
:00412BB5　call　0042AF00
:00412BBA　mov 　eax, dword ptr \[ebp\-4C\]
:00412BBD　mov 　ecx, dword ptr \[ebp\-40\]
:00412BC0　mov 　edx, dword ptr \[ebp\-1C\]
:00412BC3　mov 　eax, dword ptr \[eax\]
:00412BC5　mov 　ecx, dword ptr \[ecx\]
:00412BC7　mov 　edx, dword ptr \[edx\]
:00412BC9　push　eax
:00412BCA　push　ecx
:00412BCB　push　edx
:00412BCC　call　0044B470　　　　　　　　　　　　<==注册模块入口，按 F8 进入
:00412BD1　add 　esp, 0000000C
:00412BD4　test　eax, eax
:00412BD6　je 0　0412D2A
:00412BDC　mov 　ecx, esi
:00412BDE　call　00412DF0
:00412BE3　test　eax, eax
:00412BE5　jne 　00412C05
:00412BE7　mov 　eax, dword ptr \[esi+000001B0\]
:00412BED　push　00000010
```

`412BCC` 是注册模块入口，按 `F8` 进入，来到：

```
\* Referenced by a CALL at Addresses:
|:00412BCC, :0044AC60, :0044AEE5
|
:0044B470　sub 　esp, 00000304
:0044B476　lea 　eax, dword ptr \[esp+00000080\]
:0044B47D　push　ebx
:0044B47E　push　ebp
:0044B47F　push　esi
:0044B480　push　edi

\* Possible StringData Ref from Code Obj ->"ABCDEFGHIJKLMNOPQRSTUVWXYZabcde"
                                       ->"fghijklmnopqrst"
                                       ->"uvwxyz0123456789+/="
                                      |
:0044B481　push　00473000
:0044B486　push　eax

\* Reference To: KERNEL32.lstrcpyA, Ord:0000h
                       |
:0044B487　Call　dword ptr \[0046712C\]
:0044B48D　mov 　ecx, 00000040　　　　　　　　　　<==从这里开始建密匙对照表
:0044B492　or e　ax, FFFFFFFF
:0044B495　lea 　edi, dword ptr \[esp+000000D2\]
:0044B49C　repz
:0044B49D　stosd
:0044B49E　mov 　eax, 00000041

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B513(C)
|
:0044B4FF　mov 　byte ptr \[esp+eax+000001DC\], 00
:0044B507　mov 　byte ptr \[esp+eax+00000241\], 00
:0044B50F　inc 　eax
:0044B510　cmp 　eax, 00000064
:0044B513　jl  　0044B4FF
:0044B515　mov 　esi, dword ptr \[esp+00000318\]
:0044B51C　mov 　dword ptr \[esp+000002A8\], 3F7FDEFF
:0044B527　push　esi
:0044B528　mov 　byte ptr \[esp+000001DC\], FF　　<==第二遍运算时使用的密匙表
:0044B530　mov 　byte ptr \[esp+000001DD\], DE
:0044B538　mov 　byte ptr \[esp+000001DE\], 7F
:0044B540　mov 　byte ptr \[esp+000001DF\], cl
:0044B547　Call　 dword ptr \[00467128\]
:0044B54D　mov 　edx, eax
:0044B54F　cmp 　edx, 00000014
:0044B552　jl  　0044B6D5
:0044B558　mov 　dword ptr \[esp+18\], edi
:0044B55C　mov 　ecx, 00000019
:0044B561　xor 　eax, eax
:0044B563　lea 　edi, dword ptr \[esp+28\]
:0044B567　lea 　ebp, dword ptr \[edx+esi\-04\]
:0044B56B　lea 　ebx, dword ptr \[edx+esi\-03\]
:0044B56F　repz
:0044B570　stosd
:0044B571　mov 　al, byte ptr \[ebp+00\]
:0044B574　lea 　edi, dword ptr \[edx+esi\-02\]
:0044B578　mov 　byte ptr \[esp+10\], al
:0044B57C　mov 　cl, byte ptr \[ebx\]
:0044B57E　mov 　byte ptr \[esp+11\], cl
:0044B582　mov 　al, byte ptr \[edi\]
:0044B584　lea 　edx, dword ptr \[edx+esi\-01\]
:0044B588　mov 　byte ptr \[esp+12\], al
:0044B58C　mov 　dword ptr \[esp+24\], edx
:0044B590　push　0016EB35
:0044B595　mov 　cl, byte ptr \[edx\]
:0044B597　lea 　edx, dword ptr \[esp+1C\]
:0044B59B　mov 　byte ptr \[esp+17\], cl
:0044B59F　push　edx
:0044B5A0　lea 　eax, dword ptr \[esp+18\]
:0044B5A4　push　00000004　　　　　　　　　　　<==先处理末４位
:0044B5A6　lea 　ecx, dword ptr \[esp+34\]
:0044B5AA　push　eax
:0044B5AB　push　ecx
:0044B5AC　lea 　ecx, dword ptr \[esp+000000A4\]
:0044B5B3　mov 　dword ptr \[esp+34\], edi
:0044B5B7　mov 　\[esp+28\], 00
:0044B5BC　call　00406F20　　　　　　　　　　　 <==末４位处理模块，按 F8 进入
:0044B5C1　test　eax, eax
:0044B5C3　jne 　0044B5D8
:0044B5C5　mov 　edx, dword ptr \[esp+18\]
:0044B5C9　pop 　edi
:0044B5CA　pop 　esi
:0044B5CB　pop 　ebp
:0044B5CC　mov 　byte ptr \[esp+edx+1C\], al
:0044B5D0　pop 　ebx
:0044B5D1　add 　esp, 00000304
:0044B5D7　ret
```

通过后面对注册机制的分析，可以从上面建立的密匙对照表总结出，`TurboNote+` 的注册码只接受 `0-9`、`A-Z`、`a-z`、“＋”和“/”，其它为无效字符。

### 末四位处理模块分析

在 `44B5BC` 按 `F8` 进入，来到 `406F20`，这是对串尾末 4 位的处理。我们可以看两种串尾的结构，一种串尾为任意码，另一种串尾为"=="，两种算法的效果基本是一样的，这里我们来看第一种。

```
\* Referenced by a CALL at Addresses:
|:004074D6, :0044B5BC
|
:00406F20　sub 　esp, 000000D0
:00406F26　xor 　eax, eax
:00406F28　push　ebx
:00406F29　mov 　ebx, dword ptr \[esp+000000DC\]
:00406F30　push　ebp
:00406F31　push　esi
:00406F32　mov 　esi, dword ptr \[esp+000000E8\]
:00406F39　mov 　ebp, ecx
:00406F3B　push　edi
:00406F3C　mov 　ecx, 00000019
:00406F41　lea 　edi, dword ptr \[esp+78\]
:00406F45　repz
:00406F46　stosd
:00406F47　cmp 　byte ptr \[ebx+esi\-01\], 3D　　　　<==注意这里,串尾一个"="号
:00406F4C　je 0　0406FC9
:00406F4E　mov 　ecx, 00000019　　　　　　　　　　<==串尾为任意码的处理
:00406F53　xor 　eax, eax
:00406F55　lea 　edi, dword ptr \[esp+10\]
:00406F59　push　esi
:00406F5A　repz
:00406F5B　stosd
:00406F5C　lea 　eax, dword ptr \[esi+2\*esi\]　　　　　　　　　　　<==串长 x3
:00406F5F　mov 　edi, dword ptr \[esp+000000F4\]
:00406F66　shl 　eax, 1　　　　　　　　　　　　　　　　　　　　　<==结果 x2
:00406F68　cdq
:00406F69　and 　edx, 00000007
:00406F6C　push　ebx
:00406F6D　add 　eax, edx
```



# Turbo Note+ V4.4 窗口界面注册机

## 汇编代码分析

### 调用与结果处理

```
:00406F6F　mov 　ecx, ebp

:00406F71　sar 　eax, 03
             <==结果/8

:00406F74　mov 　dword ptr [edi], eax
               <==码长关键字,记下这个地址

:00406F76　lea 　eax, dword ptr [esp+18]

:00406F7A　push　eax

:00406F7B　call　00407220
             <==对串尾进行处理

:00406F80　test　eax, eax

:00406F82　jne 　00406F91
             <==成功则跳

:00406F84　pop 　edi

:00406F85　pop 　esi

:00406F86　pop 　ebp

:00406F87　pop 　ebx

:00406F88　add 　esp, 000000D0

:00406F8E　ret 　0014
```

\* Referenced by a CALL at Address:

```
:00406F7B
```

### 核心运算逻辑 (00407220)

```
:00407220　push ebx

:00407221　push ebp

:00407222　mov ebp, dword ptr [esp+14]

:00407226　push esi

:00407227　push edi

:00407228　xor edi, edi

:0040722A　xor edx, edx

:0040722C　test ebp, ebp

:0040722E　jle 004072D2
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004072CC(C)
```

```
:00407234　mov ebx, dword ptr [esp+18]

:00407238　lea esi, dword ptr [edx+01]

:0040723B　movsx eax, byte ptr [ebx+esi-01]
             <==四位数一组，取第１位数

:00407240　mov al, byte ptr [eax+ecx+42]
             <==查表取值

:00407244　test al, al

:00407246　jl 004072DE

:0040724C　and eax, 0000003F
             <==3f 内有效

:0040724F　shl eax, 06
             <==结果=值*Ox40

:00407252　cmp esi, ebp

:00407254　jge 0040726C

:00407256　movsx ebx, byte ptr [edx+ebx+01]
             <==取下一位数

:0040725B　mov bl, byte ptr [ebx+ecx+42]
             <==查表取值

:0040725F　test bl, bl

:00407261　jl 004072DE

:00407263　and ebx, 0000003F
             <==3f 内有效

:00407266　or eax, ebx
             <==加这个值到结果

:00407268　mov ebx, dword ptr [esp+18]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00407254(C)
```

```
:0040726C　lea ebp, dword ptr [esi+01]

:0040726F　shl eax, 06
             <==结果=值*Ox40

:00407272　cmp ebp, dword ptr [esp+1C]

:00407276　jge 0040728E

:00407278　movsx ebx, byte ptr [edx+ebx+02]
             <==取下一位数

:0040727D　mov bl, byte ptr [ebx+ecx+42]
             <==查表取值

:00407281　test bl, bl

:00407283　jl 004072DE

:00407285　and ebx, 0000003F
             <==3f 内有效

:00407288　or eax, ebx
             <==加这个值到结果

:0040728A　mov ebx, dword ptr [esp+18]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00407276(C)
```

```
:0040728E　mov ebp, dword ptr [esp+1C]

:00407292　add esi, 00000002

:00407295　shl eax, 06
             <==结果=值*Ox40

:00407298　cmp esi, ebp

:0040729A　jge 004072AE

:0040729C　movsx esi, byte ptr [edx+ebx+03]
             <==取最后一位数

:004072A1　mov bl, byte ptr [esi+ecx+42]
             <==查表取值

:004072A5　test bl, bl

:004072A7　jl 004072DE

:004072A9　and ebx, 0000003F
             <==3f 内有效

:004072AC　or eax, ebx
             <==加这个值到结果
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040729A(C)
```

```
:004072AE　mov esi, dword ptr [esp+14]

:004072B2　add edx, 00000004

:004072B5　mov byte ptr [edi+esi+02], al
             <==结果为三位，存为新串

:004072B9　add edi, 00000003

:004072BC　shr eax, 08
             <==结果/Ox100

:004072BF　mov byte ptr [edi+esi-02], al
             <==取余

:004072C3　shr eax, 08
             <==结果/Ox100

:004072C6　mov byte ptr [edi+esi-03], al
             <==取商

:004072CA　cmp edx, ebp

:004072CC　jl 00407234
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0040722E(C)
```

```
:004072D2　pop edi

:004072D3　pop esi

:004072D4　pop ebp

:004072D5　mov eax, 00000001

:004072DA　pop ebx

:004072DB　ret 000C
```

`406F7B` 的 `call` 若运算成功，退出后会跳到 `406F91`：

### 成功路径与校验赋值

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00406F82(C)
```

```
:00406F91　mov eax, dword ptr [edi]

:00406F93　xor ecx, ecx

:00406F95　test eax, eax

:00406F97　jle 004070F9

:00406F9D　mov eax, dword ptr [esp+000000E4]

:00406FA4　lea esi, dword ptr [esp+10]

:00406FA8　sub esi, eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00406FB5(C)
```

```
:00406FAA　mov dl, byte ptr [esi+eax]
             <==取第一遍的运算结果，存为新串

:00406FAD　inc ecx

:00406FAE　mov byte ptr [eax], dl
             <==校验关键字，记下这个地址

:00406FB0　mov edx, dword ptr [edi]

:00406FB2　inc eax

:00406FB3　cmp ecx, edx

:00406FB5　jl 00406FAA

:00406FB7　pop edi

:00406FB8　pop esi

:00406FB9　pop ebp

:00406FBA　mov eax, 00000001

:00406FBF　pop ebx

:00406FC0　add esp, 000000D0

:00406FC6　ret 0014
```

校验关键字是由串的末四位运算而来的，它实际上是注册用户名累加和的尾数。

对串的前半部分，要做两遍运算处理，其第一遍运算同样调用的是上面的模块，这里不再赘述。

### 第二遍运算 (00407553)

第二遍运算，是在 `407553` 的调用中进行的，我们来看一下：

\* Referenced by a CALL at Address:

```
:00407553
```

```
:00407580　mov 　eax, dword ptr [ecx+00000144]
             <==从串首取第一遍的运算结果

:00407586　mov 　dl, byte ptr [ecx+eax+00000148]

:0040758D　mov 　al, byte ptr [esp+04]
             <==取 4 个字的密匙表

:00407591　xor 　dl, al
             <==4 位一组，做 XOR 运算

:00407593　mov 　eax, dword ptr [esp+08]

:00407597　mov 　byte ptr [eax], dl

:00407599　mov 　edx, dword ptr [ecx+00000144]

:0040759F　inc 　edx

:004075A0　and 　edx, 80000003

:004075A6　jns 　004075AD

:004075A8　dec 　edx

:004075A9　or　　edx, FFFFFFFC

:004075AC　inc 　edx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004075A6(C)
```

```
:004075AD　mov　dword ptr [ecx+00000144], edx

:004075B3　mov　eax, 00000001

:004075B8　ret　0008
```

### 注册码效验 (0040751C)

下面进行注册码效验：

\* Referenced by a CALL at Address:

```
:0040751C
```

```
:004075C0　sub 　esp, 0000001C

:004075C3　mov 　eax, 00000009

:004075C8　push　ebp

:004075C9　push　esi

:004075CA　mov 　esi, ecx

:004075CC　xor 　ebp, ebp

:004075CE　mov 　ecx, dword ptr [esp+2C]
             <==取前面记下的码长关键字

:004075D2　push　edi

:004075D3　mov 　edi, dword ptr [esp+2C]
             <==取第二遍运算结果的串首地址

:004075D7　add 　ecx, FFFFFFFC
             <==减 4

:004075DA　cmp 　ecx, eax
             <==大于等于 9？

:004075DC　jl　　004075F0

:004075DE　lea 　ebp, dword ptr [ecx-08]
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004075EE(C)
```

```
:004075E1　mov　dl, byte ptr [eax+edi]
             <==从第 9 个数取值

:004075E4　mov　byte ptr [esi+eax+00000143], dl
             <==存入存储单元

:004075EB　inc　eax

:004075EC　cmp　eax, ecx
             <==取 N 个，N=码长关键字－4-9

:004075EE　jle　004075E1
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:004075DC(C)
```

```
:004075F0　mov 　byte ptr [esi+ebp+0000014C], 00

:004075F8　mov 　di, word ptr [edi+03]

:004075FC　and 　edi, 0000FFFF

:00407602　lea 　eax, dword ptr [esp+0C]

:00407606　push　edi
```

\* Possible StringData Ref from Code Obj -> `"N=%d"`

```
:00407607　push　 0046F31C

:0040760C　push　 eax
```

\* Reference To: `USER32.wsprintfA`, Ord:`0000h`

```
:0040760D　Call　dword ptr [004672B4]

:00407613　add 　esi, 000001B1

:00407619　push　0000000A

:0040761B　push　esi

:0040761C　push　edi

:0040761D　call　004628BC

:00407622　add 　esp, 00000018

:00407625　mov 　dword ptr [004771BC], 00262D75

:0040762F　mov 　eax, 00000001

:00407634　pop 　edi

:00407635　pop 　esi

:00407636　pop 　ebp

:00407637　add 　esp, 0000001C

:0040763A　ret 　0008
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:0044B633(C)
```

### 最终比较逻辑

```
:0044B655　mov 　ecx, dword ptr [esp+1C]

:0044B659　mov 　byte ptr [esp+ecx+000002AC], 00

:0044B661　lea 　ecx, dword ptr [esp+00000090]

:0044B668　call　00407640
             <==在 EAX 中返回 4075E4 处存储的值

:0044B66D　mov 　edx, dword ptr [esp+28]
             <==取 406FAE 处存储的校验关键字

:0044B671　and 　edx, 000000FF

:0044B677　cmp 　eax, edx
```

## 作者信息

- 作者: 囚童`[FCG][BCG]`
- 课题: `Turbo Note+ V4.4` 窗口界面注册机
- 出品: `South Pacific InformationServices Ltd`



# TurboNote+ 注册机说明

## 环境信息
- 平台：`win95/98`, `winNT/2000`
- 下载：`http://turbopress.com/tnplus44.exe`
- 尺寸：`769KB`
- 工具：`LCC-WIN32`
- 关键字：软件破解 注册机

## 注册码规则
`TurboNote+`的注册码限定在 `0-9`、`A-Z`、`a-z`、`+`和`/`的有效值范围内，由以下部分组成：

1. 第一部分，`4` 位，随机码。
2. 第二部分，`4` 位，由注册时申请连接的计算机台数经两遍运算得出。
3. 第三部分，`4` 位，随机码。
4. 第四部分，`4`－`36` 位，由用户给出的全注册名经两遍运算得出。
5. 第五部分，`4` 位，由用户给出的注册名的累加和经一遍运算得出，允许末两位为`==`。

> **注意**：累加和只是以指定的算法求出的有限长度的前 `N` 位注册名的累加和。

### 运算模式一
用于第二部分和第四部分的第一遍运算：

四位 `ASC` 码一组：
`A(i)B(i)C(i)D(i) xor 0xff3f7fde`

### 运算模式二
用于第二部分和第四部分的第二遍运算以及第五部分的运算：

三位 `ASC` 码一组，作为例外，第二部分注册时申请连接的计算机台数以及第五部分的累加和是事先转换为长整数再参与运算的：

`(A(i)<<8|B(i))<<8|C(i)`

得到的结果再以四位为一组：
- `O(i)P(i)Q(i)R(i)/0x40` 取余得 `R'(i)`
- `O(i)(P(i)Q(i)R(i)/0x40/0x40` 取余得 `Q'(i)`
- `O(i)(P(i)Q(i)R(i)/0x40/0x40/0x40` 取余得 `P'(i)`
- `O(i)(P(i)Q(i)R(i)/0x40/0x40` 取商得 `O'(i)`

重组为 `O'(i)P'(i)Q'(i)R'(i)`。

以上混合使用了算术表达和逻辑表达，只是为了较清楚地说明运算过程。这里：
- `>>6` 等效于 `/0x40`
- `<<8` 等效于 `*0x100`

然后，用运算结果去查表，以对应的下脚标为新值。

### 运算模式三
用于求出计算累加和的有限长度的位数 `N`：

`N = 用户给出的全注册名长度减 1 后的能被 3 整除的最大数。`

### 运算模式四
用于求出计算前 `N` 位注册名的累加和：

`累加和 = 用户给出的全注册名前 N 位的累加和`

## LCC-Win32 介绍
下面介绍用 `LCC-WIN32` (一个小巧的 C 集成编译环境）为 `TurboNote+`写的一个窗口界面的注册机。

`LCC-Win32` 是运行于 `Windows9x/Me/NT/2000` 下的免费的 32 位 C 语言编译系统。它的原始发布文件只有 `3M`，但却拥有一个功能强大的集成开发环境(`IDE`)、速度极快的编译器与连接器、方便顺手的调试器、资源编辑器和版本控制系统。另外还提供图像编辑器和 C 语言源文件格式化缩进器等附加工具。你可以用 `LCC-Win32` 开发 32 位的控制台程序、Windows 常规程序、动态连接库(`DLL`)以及静态连接库(`LIB`)。`LCC` 编译器支持标准的 `ANSI C`，同时支持 C 语言的扩展。

## 注册码生成的 C 语言实现
在下面的注册机源代码中已有清楚的注释，只对几处要点加以说明：

1. **随机数**：`LCC` 的库中没有 `RANDMON()`函数，可用下面方法实现：
   ```c
   srand(time(NULL));
   for(i=0;i<4;i++)Buff0[i]=Buff1[rand()%j];
   ```
   其中 `rand()%j` 能确保所引用的数据脚标不大于串的实际长度。

2. **数值转换**：
   申请连接的计算机台数输入值是 `ASC` 码，必需转换成为长整数才能参与运算：
   ```c
   q=atol(Buff2);
   ```

3. **下脚标**：
   我在 C 语言中没能发现有返回下脚标的函数，只好查表求出字符所在位置的指针，然后减去字符串首指针，实现返回下脚标：
   ```c
   Buff0[j] = (int)memchr(Key1,q%64,192)-(int)Key1;
   ```

由于实现了窗口界面，有条件使用户界面更加友好，采取了以下措施：

1. **OK 按钮初始化为灰色**：只有用户名长度大于 `3`，申请连接的计算机台数不为 `0` 时，才加亮。进行过一次运算后，编辑框必需清空，才能再次启用 `OK` 按钮。
   ```c
   DEFPUSHBUTTON "OK", IDOK, 20, 87, 40, 14, WS_DISABLED
   if (Len1 > 3 && Len2 > 0 && w==0)EnableWindow(IDOKCode,1);
   ```

2. **Key 输出框设定为只读**：控制状态改变时，加亮状态不可改变；并初始化为禁止访问。
   ```c
   EDITTEXT IDKey, 60, 58, 144, 12, ES_NOHIDESEL | ES_READONLY | WS_DISABLED
   ```

3. **限制输入串长和限制输入字符**：不采用提示，而是用程序实现限制。

   **注意**：有效字符区域除了可显示字符外，还应包括必要的编辑键，其中：
   - `08` : `←` 退格
   - `03` : `Ctrl + C` 复制
   - `0x16` : `Ctrl + V` 粘贴
   - `0x18` : `Ctrl + X` 剪切

   注意不要被屏蔽掉。

   其它编辑键象 `Home`、`End`、`Page Up`、`Page Down`、`Ins`、`Del`、`Tab` 及方向键受系统支持，不可能被屏蔽，可以不考虑。

4. **使初始状态控制光标停留在用户名输入框中**：
   这个功能不是用命令语句实现的，而是使用了一个小小的技巧。在处理初始化消息 `WM_INITDIALOG` 时，只要返回 `1`，控制光标就一定会停留在资源脚本文件中第一个出现的控制中。而大多数集成编译环境在自动生成资源脚本文件时，都会让 `OK` 键放在第一个控制的位置上。这就是为什么刚刚编出来的程序，控制总是停留在 `OK` 键上的原因。

   所以，除了返回 `1` 外，还应该用文本编辑器打开资源脚本文件 `tbnotekeygen.rc`，将你想要初始状态控制光标停留在那上面的编辑框放在紧靠 `BEGIN` 的第一行：
   ```rc
   BEGIN
       EDITTEXT IDYour_name, 60, 18, 130, 12, ES_AUTOHSCROLL | WS_GROUP
   ```

5. **加亮输出的注册码**：
   我始终没有搞清在非 MFC 环境中哪些 C 语言函数可以实现文本加亮，只好使用下面的笨办法：先用 `GetNextDlgTabItem(hwndDlg,0,0)`获得第一个控制（不知为什么，这个函数的名字和它的实际功能不符），增加一个偏移量，让它指向第二个控制，再用 `SetFocus()` 函数把焦点设定在第二个控制上，紧接着用 `keybd_event()`函数模拟一个 `Tab` 键，第三个控制就处于加亮状态了。

   ```c
   SetFocus((HWND)((long)GetNextDlgTabItem(hwndDlg,0,0)+4));//焦点定位
   keybd_event(9,0,0,0);              //模拟按下 TAB 键
   keybd_event(9,0,2,0);              //模拟松开 TAB 键
   ```

   由于已经设定了 `Key` 注册码输出框控制状态改变时，加亮状态不可改变（`ES_NOHIDESEL`），所以此时即使用 `Tab` 键改变控制，输出的注册码总是加亮的。

   有人会问，把焦点再定位在清空键上，不是更加友好吗？那是因为 `Lcc` 自动把回车键捆绑在 `OK` 键上，这不失是一个友好的安排，我就保留了它。

6. **将结果放入剪切板中**：
   剪切板使用的是系统内存，而不是分配给线程的内存，所以不能简单地直接使用 `SetClipboardData()`函数，而是要先申请系统内存，锁定后复制，复制后解锁；另外，打开、清空和关闭剪切板也是必不可少的：

   ```c
   OpenClipboard(hwndDlg);
   EmptyClipboard();
   hGlobal = GlobalAlloc(GMEM_ZEROINIT, lstrlen(Buff0)+1);
   if (hGlobal == NULL) return;
   strcpy(GlobalLock(hGlobal), Buff0);
   GlobalUnlock(hGlobal);
   SetClipboardData(CF_TEXT, hGlobal);
   CloseClipboard();
   ```

## 注册机源代码

```c
/////////////////////////Begin of tbnotekeygenres.c////////////////////////
/*@@ Wedit generated application. Written Mon Dec 31 19:31:37 2001
/*<---------------------------------------------------------------------->*/

#include <windows.h>
#include <windowsx.h>
#include <commctrl.h>
#include <string.h>
#include <time.h>
#include "tbnotekeygenres.h"

/*<---------------------------------------------------------------------->*/

static BOOL CALLBACK DialogFunc(HWND, UINT, WPARAM, LPARAM);
WNDPROC LimitCharYour_nameProc = 0;

int APIENTRY WinMain(HINSTANCE hinst, HINSTANCE hinstPrev, LPSTR lpCmdLine, int nCmdShow)
{
    WNDCLASS wc;
    memset(&wc,0,sizeof(wc));
    wc.lpfnWndProc = DefDlgProc;
    wc.cbWndExtra = DLGWINDOWEXTRA;
    wc.hInstance = hinst;
    wc.hCursor = LoadCursor(NULL, IDC_ARROW);
    wc.hbrBackground = (HBRUSH) (COLOR_WINDOW + 1);
    wc.lpszClassName = "Cubprisoner";
    RegisterClass(&wc);
    return DialogBox(hinst, MAKEINTRESOURCE(IDD_MAINDIALOG), NULL, (DLGPROC) DialogFunc);
}

int __stdcall LimitCharProc(HWND hwndDlg, UINT message, WPARAM wparam, LPARAM lparam) //限制输入字符
{
    TCHAR chCharCode;
    if(message == WM_CHAR)
    {
        chCharCode = (TCHAR) wparam;
        if(chCharCode == 0x03 || chCharCode == 0x08 || chCharCode == 0x16 || chCharCode == 0x18 || chCharCode == 0x2b || chCharCode >= 0x2f && chCharCode <= 0x39 || chCharCode >= 0x41 && chCharCode <= 0x5a || chCharCode >= 0x61 && chCharCode <= 0x7a) //有效字符区域
        {
            return(CallWindowProc(LimitCharYour_nameProc, hwndDlg, message, wparam, lparam));
        }
        else return(0);
    }
    else return(CallWindowProc(LimitCharYour_nameProc, hwndDlg, message, wparam, lparam));
}

/*<---------------------------------------------------------------------->*/

void RegProc(HWND hwndDlg, WPARAM wParam) //注册过程
{
    const unsigned char Key0[4] = {0xde,0x7f,0x3f,0xff}; //0xff3f7fde
    HGLOBAL hGlobal; // Global memory handle
    TCHAR Buff0[60];
    TCHAR Buff1[45];
    TCHAR Buff2[32];
    register int Slen,i=0,j,k,m=0;
    long q,sum=0;
    
    Slen = GetDlgItemTextA(hwndDlg,IDYour_name,Buff0,32)-1;
    for(i=0;i<(Slen-Slen%3);i++)sum+=Buff0[i]; //计算名字累加和
    Slen++;
    k=Slen%3;
    if(k!=0) //补 0 使串长能被 3 整除
    {
        k=3-k;
        m=k;
        while(k)
        {
            Buff0[Slen+k] = 0;
            k--;
        }
    }
    Slen+=m;
    for (i=0;i<Slen;i++)Buff1[i]= Buff0[i]^Key0[i%4];//对名字做第一遍处理
    Buff1[Slen] = 0;
    Buff1[Slen+1] = 0;
    CharOperation(Buff1); //对名字做第二遍处理
    j=strlen(Buff1);
    srand(time(NULL));
    for(i=0;i<4;i++)Buff0[i]=Buff1[rand()%j]; //生成第一节随机串
    Buff0[4] = 0;
    GetDlgItemTextA(hwndDlg,IDComputers,Buff2,5);//取计算机台数
    q=atol(Buff2); //对计算机台数做第一遍处理
    if(q>0xff)q=0xff; //计算机台数设限
    q^=0xff3f;
    strcat(Buff0,(TCHAR *)CharOperation(&q));//对计算机台数做第二遍处理，生成首段注册码
    j=strlen(Buff0);
    srand(time(NULL));
    for(i=0;i<4;i++)Buff0[i+j]=Buff0[rand()%(i+j)]; //生成第三节随机串
    Buff0[j+i] = 0;
    strcat(Buff0,Buff1);//生成前段注册码
    strcat(Buff0,(TCHAR *)CharOperation(&sum)); //生成全部注册码
    SetDlgItemTextA(hwndDlg, IDKey, Buff0);
    OpenClipboard(hwndDlg); //将结果放入剪切板中
    EmptyClipboard();
    hGlobal = GlobalAlloc(GMEM_ZEROINIT, lstrlen(Buff0)+1);
    if (hGlobal == NULL) return;
    strcpy(GlobalLock(hGlobal), Buff0);
    GlobalUnlock(hGlobal);
    SetClipboardData(CF_TEXT, hGlobal);
    CloseClipboard();
    return ;
}

TCHAR *CharOperation(TCHAR *Buff)
{
    const unsigned char Key1[192]=
    {0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,
```



```
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0xFF,0x3E,0xFF,
0xFF,0xFF,0x3F,0x34,0x35,0x36,0x37,0x38,0x39,0x3A,0x3B,0x3C,0x3D,0xFF,0xFF,
0xFF,0xFF,0xFF,0xFF,0xFF,0,1,2,3,4,5,6,7,8,9,0x0A,0x0B,0x0C,0x0D,0x0E,
0x0F,0x10,0x11,0x12,0x13,0x14,0x15,0x16,0x17,0x18,0x19,0xFF,0xFF,0xFF,0xFF,
0xFF,0xFF,0x1A,0x1B,0x1C,0x1D,0x1E,0x1F,0x20,0x21,0x22,0x23,0x24,0x25,0x26,
0x27,0x28,0x29,0x2A,0x2B,0x2C,0x2D,0x2E,0x2F,0x30,0x31,0x32,0x33,0xFF,0xFF,
0xFF};
```

```c
TCHAR Buff0[45];
register int Slen, i = 0, j;
register long q;

Slen = strlen((char *)Buff);
for (i = 0, j = 0; i < Slen; i += 3, j += 4) // 第二遍处理
{
    q = ((Buff[i] & 0xff) << 8 | (Buff[i + 1] & 0xff)) << 8 | Buff[i + 2] & 0xff;
    for (j += 3; j % 4 != 0; j--)
    {
        Buff0[j] = (int)memchr(Key1, q % 64, 192) - (int)Key1; // 用余数查表, 取其下标
        q ^= q % 64;
        q /= 64;
    }
    Buff0[j] = (int)memchr(Key1, q, 192) - (int)Key1; // 用商查表, 取其下标
}
Buff0[j] = 0;
strcpy(Buff, Buff0);
return Buff;
}
```

```c
static BOOL CALLBACK DialogFunc(HWND hwndDlg, UINT msg, WPARAM wParam, LPARAM lParam) // 主程序
{
    static HWND IDOKCode;
    static HWND IDKeyCode;
    static HWND InClipboardCode;
    static int w = 0;
    TCHAR Buff0[32];
    register int Len1, Len2; // 串长

    switch (msg)
    {
    case WM_INITDIALOG: // 初始化
        IDOKCode = GetDlgItem(hwndDlg, IDOK); // 取按钮句柄
        IDKeyCode = GetDlgItem(hwndDlg, IDKey); // 取对话框句柄
        InClipboardCode = GetDlgItem(hwndDlg, InClipboard); // 取提示句柄
        SendMessage(GetDlgItem(hwndDlg, IDYour_name), EM_LIMITTEXT, 32, 0L); // 限制输入串长
        SendMessage(GetDlgItem(hwndDlg, IDComputers), EM_LIMITTEXT, 5, 0L);
        LimitCharYour_nameProc = (WNDPROC)SetWindowLong(GetDlgItem(hwndDlg, IDYour_name), GWL_WNDPROC, (long)LimitCharProc); // 限制输入字符
        return 1;

    case WM_COMMAND: // 响应输入
        Len1 = GetDlgItemText(hwndDlg, IDYour_name, Buff0, 32); // 有有效输入时使能 OK 键
        Len2 = GetDlgItemText(hwndDlg, IDComputers, Buff0, 5);
        if (Len1 > 3 && Len2 > 0 && w == 0)
            EnableWindow(IDOKCode, 1);
        else
            EnableWindow(IDOKCode, 0);

        switch (LOWORD(wParam))
        {
        case IDOK:
            RegProc(hwndDlg, wParam); // 注册过程
            EnableWindow(IDKeyCode, 1);
            EnableWindow(InClipboardCode, 1);
            w = 1;
            SetFocus((HWND)((long)GetNextDlgTabItem(hwndDlg, 0, 0) + 4)); // 焦点定位
            keybd_event(9, 0, 0, 0); // 模拟按下 TAB 键
            keybd_event(9, 0, 2, 0); // 模拟松开 TAB 键
            // OpenClipboard(hwndDlg); EnableWindow(IDOKCode, 0);
            // SetClipboardData(CF_TEXT, 0); // Code);
            return 0;

        case IDErease: // 清除输入，重新禁止 OK 键
            SetFocus(GetNextDlgTabItem(hwndDlg, 0, 0)); // 焦点归位
            SetDlgItemTextA(hwndDlg, IDYour_name, 0);
            SetDlgItemTextA(hwndDlg, IDComputers, 0);
            SetDlgItemTextA(hwndDlg, IDKey, 0);
            EnableWindow(InClipboardCode, 0);
            EnableWindow(IDKeyCode, 0);
            EnableWindow(IDOKCode, 0);
            w = 0;
            return 1;

        case IDCANCEL: // 退出
            EndDialog(hwndDlg, 0);
            return 1;
        }
        break;

    case WM_CLOSE:
        EndDialog(hwndDlg, 0);
        return TRUE;
    }
    return FALSE;
}
```

```
/////////////////////End of tbnotekeygenres.c//////////////////////////////
/////////////////////Begin of tbnotekeygen.rc//////////////////////////////

#ifdef __LCC__
#include <windows.h>
#endif
#include "tbnotekeygenres.h"

IDD_MAINDIALOG DIALOG 121, 40, 210, 107
STYLE DS_MODALFRAME | WS_MINIMIZEBOX | WS_POPUP | WS_VISIBLE | WS_CAPTION | WS_SYSMENU
CAPTION "Key Generator for Turbo Note + V4.4  Program by Cubprisoner"
FONT 12, "Fixedsys"

BEGIN
    EDITTEXT        IDYour_name, 60, 18, 130, 12, ES_AUTOHSCROLL | WS_GROUP
    EDITTEXT        IDComputers, 60, 38, 24, 12, ES_AUTOHSCROLL | ES_NUMBER
    EDITTEXT        IDKey, 60, 58, 144, 12, ES_NOHIDESEL | ES_READONLY | WS_DISABLED
    DEFPUSHBUTTON   "OK", IDOK, 20, 87, 40, 14, WS_DISABLED
    PUSHBUTTON      "Erease", IDErease, 82, 87, 40, 14
    PUSHBUTTON      "Cancel", IDCANCEL, 140, 87, 40, 14
    RTEXT           "Your name:", Your_name, 9, 20, 41, 12
    RTEXT           "Computers:", Computers, 9, 40, 41, 12
    RTEXT           "Key:", ID_Key, 34, 59, 16, 12
    LTEXT           "The key has been in the System Clipboard.", InClipboard, 39, 74, 165, 8, WS_DISABLED | NOT WS_GROUP
END

/////////////////////End of tbnotekeygen.rc////////////////////////////////
/////////////////////Begin of tbnotekeygen.h///////////////////////////////

#define IDD_MAINDIALOG 100
#define IDErease       101
#define IDYour_name    102
#define IDComputers    103
#define Your_name      104
#define Computers      105
#define IDKey          106
#define ID_Key         107
#define InClipboard    108

/////////////////////End of tbnotekeygen.h/////////////////////////////////
```

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B633(C)
|
:0044B655    mov     ecx, dword ptr [esp+1C]
:0044B659    mov     byte ptr [esp+ecx+000002AC], 00
:0044B661    lea     ecx, dword ptr [esp+00000090]
:0044B668    call    00407640                <==在 EAX 中返回 4075E4 处存储的值
:0044B66D    mov     edx, dword ptr [esp+28]  <==取 406FAE 处存储的校验关键字
:0044B671    and     edx, 000000FF
:0044B677    cmp     eax, edx                <==关键校验
:0044B679    jne     0044B6C9                <==失败则跳
:0044B67B    mov     eax, dword ptr [esp+0000031C]
:0044B682    mov     dword ptr [004771C4], 002E8B00
:0044B68C    test    eax, eax
:0044B68E    je      0044B69F
:0044B690    lea     ecx, dword ptr [esp+000001DC]
:0044B697    push    ecx
:0044B698    push    eax

* Reference To: KERNEL32.lstrcpyA, Ord:0000h
                                      |
:0044B699    Call    dword ptr [0046712C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B68E(C)
|
:0044B69F    mov     eax, dword ptr [esp+00000320]
:0044B6A6    test    eax, eax
:0044B6A8    je      0044B6B9
:0044B6AA    lea     edx, dword ptr [esp+00000241]
:0044B6B1    push    edx
:0044B6B2    push    eax

* Reference To: KERNEL32.lstrcpyA, Ord:0000h
                                      |
:0044B6B3 FF152C714600                Call dword ptr [0046712C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0044B6A8(C)
|
:0044B6B9    pop     edi
:0044B6BA    pop     esi
:0044B6BB    pop     ebp
:0044B6BC    mov     eax, 00000001
:0044B6C1    pop     ebx
:0044B6C2    add     esp, 00000304
:0044B6C8    ret
```

第二遍运算结果的第 4、5 位包含了注册时申请连接的运算机台数，程序会将其由长整数转换为 `ASC` 码，如果校验成功，这个码会显示在 `ABOUT` 窗口中。

注册成功后程序会将注册结果另行加密后存入注册表的 `HKEY_LOCAL_MACHINE\Software\SPIS Ltd\TurboNote\Registration` 键中，并在 `REGISTER TURBONOTE+` 选项卡和 `ABOUT` 弹出窗口中显示一个握手的图标、注册用户名和允许连接的计算机台数。

上面的校验过程程序会在程序加载、注册和 `ABOUT` 中多次重复进行。

### 总结

`TurboNote+` 的注册码限定在 `0-9`、`A-Z`、`a-z`、`“＋”` 和 `“/”` 的有效值范围内，由以下部分组成：

1.  第一部分，4 位，随机码。
2.  第二部分，4 位，经两遍运算可得出注册时申请连接的计算机台数。
3.  第三部分，4 位，随机码。
4.  第四部分，4－36 位，经两遍运算可得出用户给出的全注册名。
5.  第五部分，4 位，经一遍运算可得出由用户给出的注册名的累加和，允许末两位为 `“＝＝”`。

**注意**：累加和只是以指定的算法求出的有限长度的前 `N` 位注册名的累加和。

### 运算模式一

用于第二部分和第四部分的第一遍运算以及第五部分的运算：

#### 查表

```
A(i)      B(i)      C(i)      D(i)
↓         ↓         ↓         ↓
A'(i)     B'(i)     C'(i)     D'(i)
```

四位 `ASC` 码一组：

```
((A'(i)<<6|B'(i))<<6|C'(i))<<6|D'(i)
```

得到的结果再以 3 位为一组：

```
P(i)Q(i)R(i)/0x100 取余得 R'(i)
```



`(P(i)Q(i)R(i)/0x100)/0x100` 取余得 `Q'(i)`

`(P(i)Q(i)R(i)/0x100)/0x100` 取商得 `P'(i)`

重组为 `P'(i)Q'(i)R'(i)`

### 运算模式二

用于第二部分和第四部分的第二遍运算：

四位 ASC 码一组：

`A(i)B(i)C(i)D(i) xor 0xff3f7fde`

以上混合使用了算术表达和逻辑表达，只是为了较清楚地说明运算过程。这里：

- `<<6` 等效于 `*0x40`
- `>>8` 等效于 `/0x100`

### 运算模式三

用于求出计算累加和的有限长度的位数 `N`

`N = 第四部分求出的用户给出的全注册名长度减 1 后的能被 3 整除的最大数`

### 运算模式四

用于求出计算前 `N` 位注册名的累加和：

`累加和 = 用户给出的全注册名前 N 位的累加和`

### 校验

经运算模式四计算出的累加和的末字节 = 对第五部分经运算模式二所得结果的末字节

由于 `N` 的算法的局限性，在 `REGISTER TURBONOTE+` 选项卡和 `ABOUT` 弹出窗口中显示的用户名无法保证是全名，又由于 `TURBONOTE+` 不支持注册名中使用空格，所以显示出的用户名有时会使你显得很尴尬。

### 用以上方法逆运算求出注册码

- 用户名：`Cubprisoner/FCG`
- 申请连接的计算机台数：`11`
- 注册码：`006jNP8AN66AnQpdj6wWTJCwGk3QmDx40N5/iwUA`
- 显示出的用户名：`Cubprisoner/FCG`

我用 `LCC-WIN32`（一个小巧的 C 集成编译环境）写了一个窗口界面的注册机，在用户界面友好方面做了一些改进，将另文发表。

`4dd466d0e1e0cc123afbe504da2462ca`
