
# IQ 网际搜寻家 99

> 原文：[`www.pediy.com/kssd/pediy01/forum057.htm`](https://www.pediy.com/kssd/pediy01/forum057.htm)

## 如何计算 “IQ 网际搜寻家 99” 注册码

“IQ 网际搜寻家 99”是资讯人公司出品的一个功能十分强大的搜索引擎，获得 zdnet 五星奖。

下载地址 `http://www.inforian.com/`，约 3M 左右，可以支持中文 GB、BIG5、英文等语言。

开始我在网上找了一个注册码，但每次运行时程序都会叫你去资讯人公司主页注册。当然你是不能注册的，因为这个注册码早就被注册了。忍无可忍之下，今天我准备将其注册码计算方式找到，自己注册一个。

`IQ99.exe` 被 `shrink3.4` 压过，你可以用 `deshrink1.6` 解压，不过不这样作也没关系，因为其比较程序是在 `checknodll.dll` 中。

首先，你要随便输入一个注册码，`01234567895` （11 位）

在 `trw2000` 中下断点

`bpx hmemcpy`

`g`

回到 `IQ99.exe` 中，按下“注册”按钮，trw 跳出。

`bd` 中断断点

`F12` 14 下

`F10` 走到

* Possible StringData Ref from Code Obj ->"bCheckSerialNumber"

:004C2CD1 A168205000 `mov eax, dword ptr [00502068]`
:004C2CD6 E8B512F4FF `call 00403F90`
:004C2CDB 50 `push eax`
:004C2CDC 8B45F4 `mov eax, dword ptr [ebp-0C]`
:004C2CDF 50 `push eax`

* Reference To: `kernel32.GetProcAddress`, Ord:0000h

:004C2CE0 E8BB2EF4FF `Call 00405BA0`
:004C2CE5 8BF8 `mov edi, eax`
:004C2CE7 89FB `mov ebx, edi`
:004C2CE9 85FF `test edi, edi` <--没有输入，失败
:004C2CEB 7417 `je 004C2D04`
:004C2CED 8B45FC `mov eax, dword ptr [ebp-04]`
:004C2CF0 E89B12F4FF `call 00403F90`
:004C2CF5 50 `push eax`
:004C2CF6 56 `push esi`
:004C2CF7 FFD3 `call ebx` <--**
:004C2CF9 83C408 `add esp, 00000008`
:004C2CFC 3C01 `cmp al, 01` <--以 `al=1` 作为注册标志
:004C2CFE 7504 `jne 004C2D04` <--跳则失败
:004C2D00 C645FB00 `mov [ebp-05], 00`

现在你要进入 :004C2CF7 FFD3 `call ebx` 中看看 `al` 是如何被赋值的，`F8` 进入

这就进入了 `checknodll.dll` 领域了。

:100010EB 83F90B `cmp ecx, 0000000B` <--注册码是否 11 位？
:100010EE 0F85AA000000 `jne 1000119E`
:100010F4 33FF `xor edi, edi`
......
:10001111 `mov [esp+0C], 00000007` <-- 1 -
:10001119 `mov [esp+10], 00000011` <-- 2 |
:10001121 `mov [esp+14], 00000017` <-- 3 |
:10001129 `mov [esp+18], 00000025` <-- 4 |
:10001131 `mov [esp+1C], 0000002F` <-- 5 |--依次对应注册码
:10001139 `mov [esp+20], 00000035` <-- 6 | 1-9 位
:10001141 `mov [esp+24], 00000043` <-- 7 |
:10001149 `mov [esp+28], 00000049` <-- 8 |
:10001151 `mov [esp+2C], 00000053` <-- 9 -
:10001159 `mov eax, ebx`
:1000115B `xor ecx, ecx`
:1000115D `lea edx, dword ptr [esp+0C]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001174(C)

:10001161 0FBE3C31 `movsx edi, byte ptr [ecx+esi]`
:10001165 83EF30 `sub edi, 00000030`
<--依次取 1-9 位注册码的数字
:10001168 0FAF3A `imul edi, dword ptr [edx]`
<--各位数与上述数字对应相乘
:1000116B 03C7 `add eax, edi` <--总和给 `eax`
:1000116D 41 `inc ecx`
:1000116E 83C204 `add edx, 00000004`
:10001171 83F909 `cmp ecx, 00000009`
:10001174 7CEB `jl 10001161`
:10001176 0FBE4E09 `movsx ecx, byte ptr [esi+09]`
<-- 第十位注册码的 `ascii` 码给 `ecx`
:1000117A 0FBE560A `movsx edx, byte ptr [esi+0A]`
<-- 第十一位注册码的 `ascii` 码给 `edx`
:1000117E 8D0C89 `lea ecx, dword ptr [ecx+4*ecx]`
<-- `ecx=5*` 第十位注册码的 `ascii` 码
:10001181 BE64000000 `mov esi, 00000064`
:10001186 5F `pop edi`
:10001187 8D8C4AF0FDFFFF `lea ecx, dword ptr [edx+2*ecx-00000210]`
<-- `ecx=` 第十一位注册码的 `ascii` 码
+ (`A*` 第十位注册码的 `ascii` 码)
- 210
:1000118E 99 `cdq`
:1000118F F7FE `idiv esi` <-- `edx` 为 `eax/64` 的余数
:10001191 33C0 `xor eax, eax`
:10001193 5E `pop esi`
:10001194 5B `pop ebx`
:10001195 3BCA `cmp ecx, edx` <-- 比较
:10001197 0F94C0 `sete al` <-- `al` 赋值

所以我计算出一个：`01234567804`

不要用这个去网站注册，因为我已经用了。要注册，自己算。

不知谁将这个计算式作一注册机？请在其中写上我的名字：小楼

谢谢。

## 如何除去“IQ 网际搜寻家 99”中“网上注册”窗口

今天我算出了“iq 网际搜索 99”的注册码计算方法，当然要去其网站登记为“合法用户”--主要是为了除去每次运行时出现的那个“网上注册”破窗户。谁知道在线注册后，破窗仍旧出现。所以，我就写了封 `e-mail` 给他们，要求服务——谁叫我是“正式用户”呢，当然要享受这一“权利”。

老美动作就是快，上午同时寄出的两封信，一封给总部，一封给它国内子公司。现在我收到了总部的答复，给了我一个补丁程序，说 `run it , and ok`. 当然我执行了，问题就解决了。

但是它作了些什么？脑子一转，猜它在注册表中作了记号。联想到微软是在注册表中加入 `"register"=1` 作为注册标志。就在 `[HKEY_USERS\.DEFAULT\Software\Inforia\Quest\SYSTEM]` 中找，果然，其注册标志为

```
[HKEY_USERS\.DEFAULT\Software\Inforia\Quest\SYSTEM]
"RegistrationDone"="1"
```

将之改为 `"0"`，破窗再次出现。有意思的是，若去掉注册表中的注册码，但保留 `"RegistrationDone"="1"`，会叫你填入注册码，但不会出现”网上注册“的窗口。
