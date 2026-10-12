
# CuteFTP V4.2.4

> 原文：[`www.pediy.com/kssd/pediy04/kanxue066.htm`](https://www.pediy.com/kssd/pediy04/kanxue066.htm)

CuteFTP 最新版 V4.2.4 在线注册的破解

破解者：`moonlite[BCG][FCG]`

目标：`CuteFTP` 最新版 V4.2.4

应用平台：`Win9X/ME/WinNT/2K`

下载：`http://www.globalscape.com/`

大小：`1694k`

软件用途：当然是最 cool 的 FTP 客户端软件了，不用再多说了吧。

工具：`TRW1.22`, `W32dasm`, `filemon`, `regmon`, `Winhex`

保护：每次启动都弹出注册窗，提示上网注册; 30 天试用期；动态 CRC 校验。

### 前言

`xy2000[BCG]` 老兄推荐的软件，就拿它练练手吧。我很喜欢这个软件的原因有三：

1. 没加壳；
2. 没有反调试；
3. `CRC` 的出错信息中体现了对 crack 们的尊重：请看

```
CuteFTP consistency check failed. This means that you are probably using a corrupted version. This
may caused by a virus. Please, do a virus scan, reinstall CuteFTP and try to start it again.
```

它不象有些软件，你一调试，它就说 `"Hmm...Debug yourself"`。

好，开始工作吧!

### 第一部分：找注册码

1. 启动 `cutftp32.exe`, 提示在线注册 nag 窗口弹出。分析它的注册信息一定存放在注册表中，或有 `keyfile` 保护。

2. 分别启动 `filemon` 和 `regmon` 分析：

发现以下可疑点→

`AUTONAME.DAT`, `COMMANDS.DAT` -> 调用到的文件

```
QueryValueEx    HKLM\Software\GlobalSCAPE Inc.\CuteFTP\Key2                    NOTFOUND
QueryValueEx    HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\ProductId         SUCCESS    "80123-026-6304672-53376"
CloseKey        HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion                   SUCCESS
OpenKey         HKCR\Rl                                                          NOTFOUND   ※※※※
OpenKey         HKLM\Software\GlobalSCAPE Inc.\CuteFTP                           SUCCESS    hKey: 0xC2A0E050
QueryValueEx    HKLM\Software\GlobalSCAPE Inc.\CuteFTP\Key1                    NOTFOUND
```

3. 我尝试了建 `Key1` 和 `Key2` 两个键值，没有发现效果。就在 `HKCR\` 下建了个 `Rl\1`, 随便输入字符串 `"23232323232323"`。

`[HKEY_LOCAL_MACHINE\Software\GlobalSCAPE Inc.\CuteFTP]` 下建 `"RegUserName"`=`"moonLite[BCG]"`

4. 再次运行 `cutftp32.exe`，在线注册窗口弹出。唤出 `TRW`，点击按钮 `"Contiue Trial"` 并 `Ctrl+D` 激活 `TRW`。程序来到-->

```
* Reference To: USER32.GetMessageA, Ord:012Ah
              |
:004DD7E4 FF1594D75100      Call dword ptr [0051D794]
:004DD7EA 85C0             test eax, eax<-------------------------光标在这！
:004DD7EC 7426             je 004DD814
:004DD7EE 817E346A030000   cmp dword ptr [esi+34], 0000036A
:004DD7F5 741A             je 004DD811
:004DD7F7 8B06             mov eax, dword ptr [esi]
:004DD7F9 57               push edi
:004DD7FA 8BCE             mov ecx, esi
:004DD7FC FF5058           call [eax+58]
:004DD7FF 85C0             test eax, ea
:004DD801 750E             jne 004DD811
:004DD803 57               push edi
```

开始按 `F12+F10`，记录下来可疑的跳转：`4D8249`, `43B873`.

```
:0043B849 68F4235500       push 005523F4
:0043B84E 8BCB             mov ecx, ebx
:0043B850 E82CB70B00       call 004F6F81
:0043B855 8983DC000000     mov dword ptr [ebx+000000DC], eax
:0043B85B B801000000       mov eax, 00000001
:0043B860 898344060000     mov dword ptr [ebx+00000644], eax
:0043B866 898380060000     mov dword ptr [ebx+00000680], eax
:0043B86C E8FF4F0500       call 00490870------------------------->进入
:0043B871 85C0             test eax, eax\-------------------------|这里，让 eax=1 可以跳过 nag！
:0043B873 753D             jne 0043B8B2
:0043B875 33F6             xor esi, esi
:0043B877 8BCB             mov ecx, ebx
:0043B879 56               push esi
```

```
* Possible StringData Ref from Data Obj ->"TSUninstaller"
              |
:0043B87A 68DC465500       push 005546DC
* Possible StringData Ref from Data Obj ->"CtFPRgsraeoe"
              |
:0043B87F 68F4235500       push 005523F4
:0043B884 E85B890A00       call 004E41E4
:0043B889 89B380060000     mov dword ptr [ebx+00000680], esi
:0043B88F 6A01             push 00000001
:0043B891 8BCB             mov ecx, ebx
:0043B893 89B388060000     mov dword ptr [ebx+00000688], esi
:0043B899 E812130000       call 0043CBB0
:0043B89E 8BCB             mov ecx, ebx
:0043B8A0 E87B0A0000       call 0043C320-------------------------|在线注册窗口
:0043B8A5 85C0             test eax, eax
:0043B8A7 751E             jne 0043B8C7
:0043B8A9 56               push esi
```

可见，`0043B86C` 的 `CALL` 有问题，得进去看看！

5.

```
* Referenced by a CALL at Addresses:
|:004013FA  , :004300A8  , :004346DB  , :0043B86C  , :0044045B
|:004459D9  , :004476A3  , :00457F8F  , :0047D15E  , :0047D8FE
|:0048B82F  , :0048C470  , :00491F79  , :004ACB68
|

:00490870 64A100000000     mov eax, dword ptr fs:[00000000]
* Possible Reference to String Resource ID=00255: "No entry for the current site found. Do you wish to create o"
              |
:00490876 6AFF             push FFFFFFFF
:00490878 68D34F5100       push 00514FD3
:0049087D 50               push eax
:0049087E B81C180000       mov eax, 0000181C
:00490883 64892500000000   mov dword ptr fs:[00000000], esp
:0049088A E801130300       call 004C1B90
:0049088F 53               push ebx
:00490890 8D8424680C0000   lea eax, dword ptr [esp+00000C68]
:00490897 56               push esi
:00490898 50               push eax
:00490899 E882F9FFFF       call 00490220
:0049089E 83C404           add esp, 00000004
:004908A1 85C0             test eax, eax
:004908A3 7517             jne 004908BC
:004908A5 5E               pop esi
:004908A6 5B               pop ebx
:004908A7 8B8C241C180000  mov ecx, dword ptr [esp+0000181C]
:004908AE 64890D00000000   mov dword ptr fs:[00000000], ecx
:004908B5 81C428180000     add esp, 00001828
:004908BB C3               ret
```

-->不断按 `F10`，会来到：

```
:004908E7 83C40C           add esp, 0000000C
:004908EA 85C0             test eax, eax
:004908EC 5F               pop edi
:004908ED 0F857A020000     jne 00490B6D
:004908F3 8A84249C040000   mov al, byte ptr [esp+0000049C]---------------|从“23232323232323”取一个字符
:004908FA 84C0             test al, al
:004908FC 0F84C1020000     je 00490BC3
:00490902 8D8C249C040000   lea ecx, dword ptr [esp+0000049C]---------------|ecx 指向“23232323232323”字符串
:00490909 8D542418         lea edx, dword ptr [esp+18]
:0049090D 51               push ecx
:0049090E 52               push edx
:0049090F C7442420FFFFFF7F mov [esp+20], 7FFFFFFF
:00490917 E824690200       call 004B7240--------------->注意到紧跟的判断，得追进去
:0049091C 83C408           add esp, 00000008
:0049091F 6685C0           test ax, ax\---------------|ax 不为 0，就能成功了！
:00490922 7519             jne 0049093D---------------|不跳转则失败！
:00490924 5E               pop esi
:00490925 33C0             xor eax, eax\---------------|eax 为注册标志
:00490927 5B               pop ebx
:00490928 8B8C241C180000  mov ecx, dword ptr [esp+0000181C]
:0049092F 64890D00000000   mov dword ptr fs:[00000000], ecx
:00490936 81C428180000     add esp, 00001828
:0049093C C3               ret
```

```
* Referenced by a CALL at Addresses:
|:00490917  , :00490BA2  , :004915A6
|

:004B7240 83EC20           sub esp, 00000020--------------------------------------------|
```
