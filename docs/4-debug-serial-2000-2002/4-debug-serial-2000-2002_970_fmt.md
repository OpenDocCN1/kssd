
# 小李注册表大师 1.2

> 原文：[`www.pediy.com/kssd/pediy04/kanxue151.htm`](https://www.pediy.com/kssd/pediy04/kanxue151.htm)

**作者**：`crackhl`

**软件**：`小李注册表大师 1.2`

**工具**：`fi`, `Caspr`, `Trw2000`

---

首先，用 `fi` 侦测，可以知道这个软件是用 `ASPack v1.06b` 加的壳！用 `Caspr` 脱掉，运行脱壳后的，填入注册

姓名：`crackhl`

注册码：`787878`

按下 `Ctrl+N` 呼叫出 `trw`, 下断点：`bpx hmemcpy`, 按 `F5` 回到界面，按下确定，立即被 `trw` 拦截，清除断点：`bc *`, 跳到程序：`pmodule`, 按 `F12`, 6 次（因为按 7 此就失败）

```
:0048507F BAF8514800                mov edx, 004851F8
:00485084 E833FDFCFF                call 00454DBC
:00485089 A1FCF24800                mov eax, dword ptr [0048F2FC]
:0048508E 8B00                      mov eax, dword ptr [eax]
:00485090 E887F6FCFF                call 0045471C
:00485095 8D55F0                    lea edx, dword ptr [ebp-10]
:00485098 8B83DC020000              mov eax, dword ptr [ebx+000002DC]
:0048509E E8D9AFFAFF                call 0043007C
:004850A3 8B55F0                    mov edx, dword ptr [ebp-10]
:004850A6 A1D0F04800                mov eax, dword ptr [0048F0D0]
:004850AB 8B00                      mov eax, dword ptr [eax]
:004850AD 8B8064030000              mov eax, dword ptr [eax+00000364]
:004850B3 E8908DFEFF                call 0046DE48
:004850B8 8D45FC                    lea eax, dword ptr [ebp-04]
:004850BB 8B15D0F04800              mov edx, dword ptr [0048F0D0]
:004850C1 8B12                      mov edx, dword ptr [edx]
:004850C3 8B9264030000              mov edx, dword ptr [edx+00000364]
:004850C9 8B524C                    mov edx, dword ptr [edx+4C]          <- 下 d edx, 得到正确的注册码 91589281
:004850CC E827E9F7FF                call 004039F8
:004850D1 8D55EC                    lea edx, dword ptr [ebp-14]
:004850D4 8B83E0020000              mov eax, dword ptr [ebx+000002E0]
:004850DA E89DAFFAFF                call 0043007C
:004850DF 8B45EC                    mov eax, dword ptr [ebp-14]
:004850E2 8B55FC                    mov edx, dword ptr [ebp-04]
:004850E5 E806ECF7FF                call 00403CF0
:004850EA 755F                      jne 0048514B                        <- 跳到失败的地方
```

**注册器**：

用 `Keymake`

中断地址：`4850CC`

中断次数：`1`

第一字节：`E8`

指令长度：`4`

内存方式：`EDX`

经过加壳

姓名：`crackhl`

注册码：`91589281`

........................................................

**作者**：`crackhl`

**软件**：`Windows 整容专家`

**工具**：`fi`, `odbg104`

---

首先，用 `fi` 侦测，可以知道这个软件是用 `VBasic` 编的，`VBasic` 编的最好破解了，用 `odbg` 还是用 `smartcheckn` 呢？`odbg` 比 `smartcheckn` 更简单，小巧，就用 `odbg`

用 `odbg` 载入 `IeIcon.exe`，按 `F9` 运行，运行后选注册

姓名：`crackhl`

注册码：`787878`

回到 `odbg`，按 `Alt+E`, 双击 `c:\windows\system\msvbvm60.dll`，然后按 `Ctrl+N`, 在 `Name` 里面选中 `__vbaStrComp`, 按 `F2`, 回到程序，点击注册，立刻被 `odbg` 拦下来，在 `odbg` 的右下角的方框中看到了自己输入的假的注册码 `787878`，在假的注册码的上面的那个 `42494242464546` 就是真的注册码，按 `Ctrl+N`，再按 `F2` 取消断点，按 `Shift+F9` 继续，把 `42494242464546` 填入注册码，注册成功！

姓名：`crackhl`

注册码：`42494242464546`

........................................................

`4dd466d0e1e0cc123afbe504da2462ca`
