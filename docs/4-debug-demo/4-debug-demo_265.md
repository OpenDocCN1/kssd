
# PicaView 1.32

> 原文：[`www.pediy.com/kssd/pediy01/forum014.htm`](https://www.pediy.com/kssd/pediy01/forum014.htm)

用过 `ACDSee` 的朋友，可能也知道同一公司出品的 `Picaview`。今天从 `ACD` 公司的主页下载了 `Picaview` `1.32`，是个 60 天试用版本。下面是破解过程，用时大约 30 分钟。

1. `Picaview.dll` 是用 `aspack1.084` 压过的，所以先用 `unaspack` 脱壳（我用版本的是 `1.0.5.0`），OK！感谢毕卫国先生。:)

2. 用 `W32dasm` 反汇编 `picaview.dll`。

3. 在 `Picaview` 菜单上大家可以见到"Purchase `PicaView`..."的字样，寻找它。

4. 见到下面的代码

```
* Reference To: USER32.InsertMenuA, Ord:0174h

:10001160 8B3DD8C40910     mov edi, dword ptr [1009C4D8]
:10001166 55               push ebp
:10001167 8BAC2418020000   mov ebp, dword ptr [esp+00000218]
:1000116E 8D4E01           lea ecx, dword ptr [esi+01]
:10001171 6A00             push 00000000
:10001173 8BC3             mov eax, ebx
:10001175 51               push ecx
:10001176 68000C0000       push 00000C00
:1000117B 50               push eax
:1000117C 55               push ebp
:1000117D 43               inc ebx
:1000117E FFD7             call edi
:10001180 B908A10B10       mov ecx, 100BA108
:10001185 E8B6870100       call 10019940 <- ???
:1000118A 85C0             test eax, eax
:1000118C 752B             jne 100011B9 <- 如果 eax 不等于 0, hehe
:1000118E 8B0D5CA20B10     mov ecx, dword ptr [100BA25C]
:10001194 8D442454         lea eax, dword ptr [esp+54]
:10001198 6A64             push 00000064
:1000119A 50               push eax

* Possible Reference to String Resource ID=00040: "Purchase PicaView..."

:1000119B 6A28             push 00000028 <- 我们在这
:1000119D 51               push ecx
```

5. 仔细看看那个可疑的 `CALL`，调用的地方有多处

```
* Referenced by a CALL at Addresses:
:10001108 , :10001185 , :10002135 , :100022A8 , :100076FB
:100385EE

:10019940 33C0             xor eax, eax
:10019942 C3               ret
```

6. 挨个看一看，都是哪些地方调用这个 `CALL`，又会发现下面这段代码

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:10002291(C)

:100022A3 B908A10B10       mov ecx, 100BA108
:100022A8 E893760100       call 10019940 <-这里
:100022AD 85C0             test eax, eax <-若 eax 不为 0
:100022AF 0F85BD000000     jne 10002372 <-跳过过期检查
:100022B5 55               push ebp
:100022B6 B908A10B10       mov ecx, 100BA108
:100022BB E8D02C0800       call 10084F90

* Reference To: USER32.GetDlgItemTextA, Ord:0104h

:100022C0 8B2D70C40910     mov ebp, dword ptr [1009C470]
:100022C6 8BF8             mov edi, eax
:100022C8 85FF             test edi, edi
:100022CA 7F21             jg 100022ED <-未过期,跳走
:100022CC 8B155CA20B10     mov edx, dword ptr [100BA25C] <-否则
:100022D2 8D8C2498000000  lea ecx, dword ptr [esp+00000098]
:100022D9 6896000000       push 00000096
:100022DE 51               push ecx

* Possible Reference to String Resource ID=00209: "Your evaluation period is over.
Please register this softwar"

:100022DF 68D1000000       push 000000D1
:100022E4 52               push edx
```

7. 打补丁，将 `xor eax, eax` 改成 `mov al,01`。

8. 再试试看，高兴了吧。:)

9. 程序的时间标志在注册表 `HKLM\software\acd systems\picaview` 项。你要程序显示你的大名，也请自己动手吧。

一时粗心，忘记试一下时间限制是否去除，结果... :)

`BPX dialogboxparama`，点右键选图片，`softice` 拦住，`F12`，`F10`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:100010AE(C)

:100010E0 83F8FB           cmp eax, FFFFFFFB
:100010E3 7516             jne 100010FB <-改成 EB16
:100010E5 E846150000       call 10002630
:100010EA 5F               pop edi <-我们在这里
:100010EB 5E               pop esi
:100010EC B811000000       mov eax, 00000011
:100010F1 5B               pop ebx
:100010F2 81C400020000     add esp, 00000200
:100010F8 C21800           ret 0018

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:100010E3(C)

:100010FB 85C0             test eax, eax
:100010FD 0F8530070000     jne 10001833 <-改成 6 个 90
```
