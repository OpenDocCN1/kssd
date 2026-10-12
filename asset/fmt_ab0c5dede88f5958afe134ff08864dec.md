
# Windows 优化大师

> 原文：[`www.pediy.com/kssd/pediy04/kanxue500.htm`](https://www.pediy.com/kssd/pediy04/kanxue500.htm)

> 软件名称：Windows 优化大师

> 版本：4.20

> 作者：时空幻影

> 时间：2001 年 11 月 25 日

> 使用工具：`TRW2000` V1.22 已注册版、`CASPR`（脱壳工具）、`PW32DSM` 白金版汉化版

先用 `CASPR` 脱掉 `WINDOWS优化大师.EXE` 和 `OCTODLL.DLL` 的壳，然后进入 `TRW2000`，再进入 Windows 优化大师，点击“软件注册”，然后在注册者姓名栏中填入自己的名字，按回车键，点击“取消”，就会看见注册申请码，在注册认证码的两个栏中分别随便填入一个长度为 8 的十进制数。按 `CTRL+N` 激活 `TRW2000` 后，设“万能断点” `BPX HMEMCPY`，回车，然后按 `F5` 回到 WINDOWS 下，点击“注册认证”会被拦下，输入指令 `BD *`、`PMODULE`，回车，然后按 `F10` 过了几个 `RET` 后会来到如下所指的地方：

```
* Possible StringData Ref from Code Obj ->"
TSearchRecX"
                    |
:004D12C2 8B156C7E4000            mov edx, dword ptr [00407E6C]
:004D12C8 E87732F3FF              call 00404544
:004D12CD 33C0                    xor eax, eax
:004D12CF 55                      push ebp
:004D12D0 682C164D00              push 004D162C
:004D12D5 64FF30                  push dword ptr fs:[eax]
:004D12D8 648920                  mov dword ptr fs:[eax], esp
:004D12DB 8D95A0FDFFFF            lea edx, dword ptr [ebp+FFFFFDA0]
:004D12E1 8B861C030000            mov eax, dword ptr [esi+0000031C]
:004D12E7 E8E805F6FF              call 004318D4
:004D12EC 83BDA0FDFFFF00          cmp dword ptr [ebp+FFFFFDA0], 00000000  <--停在这里
:004D12F3 751D                    jne 004D1312
:004D12F5 6A10                    push 00000010

* Possible StringData Ref from Code Obj ->"Windows 优化大师"
                    |
:004D12F7 B93C164D00              mov ecx, 004D163C

* Possible StringData Ref from Code Obj ->"错误！没有输入注册者姓名。"  <--提示你没有输入注册者姓名
                    |
:004D12FC BA4C164D00              mov edx, 004D164C
:004D1301 A174C25600              mov eax, dword ptr [0056C274]
:004D1306 8B00                    mov eax, dword ptr [eax]
:004D1308 E89BEBF7FF              call 0044FEA8
:004D130D E9E3020000              jmp 004D15F5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D12F3(C)
|
:004D1312 8D959CFDFFFF            lea edx, dword ptr [ebp+FFFFFD9C]
:004D1318 8B86E4020000            mov eax, dword ptr [esi+000002E4]
:004D131E E8B105F6FF              call 004318D4
:004D1323 83BD9CFDFFFF00          cmp dword ptr [ebp+FFFFFD9C], 00000000
:004D132A 741A                    je 004D1346
:004D132C 8D9598FDFFFF            lea edx, dword ptr [ebp+FFFFFD98]
:004D1332 8B86E8020000            mov eax, dword ptr [esi+000002E8]
:004D1338 E89705F6FF              call 004318D4
:004D133D 83BD98FDFFFF00          cmp dword ptr [ebp+FFFFFD98], 00000000
:004D1344 751D                    jne 004D1363

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D132A(C)
|
:004D1346 6A10                    push 00000010

* Possible StringData Ref from Code Obj ->"Windows 优化大师"
                    |
:004D1348 B93C164D00              mov ecx, 004D163C

* Possible StringData Ref from Code Obj ->"错误！没有输入注册认证码。"  <--提示你没有输入注册认证码
                    |
:004D134D BA68164D00              mov edx, 004D1668
:004D1352 A174C25600              mov eax, dword ptr [0056C274]
:004D1357 8B00                    mov eax, dword ptr [eax]
:004D1359 E84AEBF7FF              call 0044FEA8
:004D135E E992020000              jmp 004D15F5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D1344(C)
|
:004D1363 8D8DA4FEFFFF            lea ecx, dword ptr [ebp+FFFFFEA4]
:004D1369 BA3F000000              mov edx, 0000003F

* Possible StringData Ref from Code Obj ->"*. *"  <--查找当前目录下的所有文件
                    |
:004D136E B88C164D00              mov eax, 004D168C
:004D1373 E8B87EF3FF              call 00409230
:004D1378 81BDA8FEFFFF00680200    cmp dword ptr [ebp+FFFFFEA8], 00026800
:004D1382 0F846D020000            je 004D15F5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D13A9(C)
|
:004D1388 8D85A4FEFFFF            lea eax, dword ptr [ebp+FFFFFEA4]
:004D138E E8ED7EF3FF              call 00409280
:004D1393 85C0                    test eax, eax
:004D1395 7510                    jne 004D13A7
:004D1397 81BDA8FEFFFF00680200    cmp dword ptr [ebp+FFFFFEA8], 00026800
:004D13A1 0F844E020000            je 004D15F5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D1395(C)
|
:004D13A7 85C0                    test eax, eax
:004D13A9 74DD                    je 004D1388
:004D13AB 8D85A4FEFFFF            lea eax, dword ptr [ebp+FFFFFEA4]
:004D13B1 E8EE7EF3FF              call 004092A4
:004D13B6 6A00                    push 00000000
:004D13B8 8BC6                    mov eax, esi
:004D13BA E87566F6FF              call 00437A34
:004D13BF 50                      push eax

* Reference To: user32.GetWindow, Ord:0000h
                    |
:004D13C0 E81762F3FF              Call 004075DC
:004D13C5 8BD8                    mov ebx, eax
:004D13C7 85DB                    test ebx, ebx
:004D13C9 0F848A000000            je 004D1459

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D1453(C)
|
:004D13CF 68FF000000              push 000000FF
:004D13D4 8D85A5FDFFFF            lea eax, dword ptr [ebp+FFFFFDA5]
:004D13DA 50                      push eax
:004D13DB 53                      push ebx

* Reference To: user32.GetWindowTextA, Ord:0000h  <--调用 WIN32 API 函数 GetWindowTextA 取得标题栏上的字符串
                    |
:004D13DC E82362F3FF              Call 00407604
:004D13E1 85C0                    test eax, eax
:004D13E3 7E62                    jle 004D1447
:004D13E5 8D55FC                  lea edx, dword ptr [ebp\-04]
:004D13E8 8D85A5FDFFFF            lea eax, dword ptr [ebp+FFFFFDA5]
:004D13EE E8B584F3FF              call 004098A8
:004D13F3 8B55FC                  mov edx, dword ptr [ebp\-04]

* Possible StringData Ref from Code Obj ->"注册机"
                    |
:004D13F6 B898164D00              mov eax, 004D1698
:004D13FB E8242EF3FF              call 00404224  <--检查字符串中是否含有"注册机"
:004D1400 85C0                    test eax, eax
:004D1402 0F85ED010000            jne 004D15F5  <--不应该跳转
:004D1408 8B55FC                  mov edx, dword ptr [ebp\-04]

* Possible StringData Ref from Code Obj ->"wom"
                    |
:004D140B B8A8164D00              mov eax, 004D16A8
:004D1410 E80F2EF3FF              call 00404224  <--检查字符串中是否含有"wom"
:004D1415 85C0                    test eax, eax
:004D1417 0F85D8010000            jne 004D15F5  <--不应该跳转
:004D141D 8B55FC                  mov edx, dword ptr [ebp\-04]

* Possible StringData Ref from Code Obj ->"cr-wom"
                    |
:004D1420 B8B4164D00              mov eax, 004D16B4
:004D1425 E8FA2DF3FF              call 00404224  <--检查字符串中是否含有"cr-wom"
:004D142A 85C0                    test eax, eax
:004D142C 0F85C3010000            jne 004D15F5  <--不应该跳转
:004D1432 8B55FC                  mov edx, dword ptr [ebp\-04]

* Possible StringData Ref from Code Obj ->"Windowsyhds"
                    |
:004D1435 B8C4164D00              mov eax, 004D16C4
:004D143A E8E52DF3FF              call 00404224  <--检查字符串中是否含有"Windowsyhds"
:004D143F 85C0                    test eax, eax
```
