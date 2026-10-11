
# Windows 优化大师 V4.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue198.htm`](https://www.pediy.com/kssd/pediy04/kanxue198.htm)

## 初步分析

`Windows 优化大师 V4.3` 破解

经 `FI V2.45` 侦查得知安装得到的所有可执行文件都是用 `Aapack V2.11` 加的壳

分别用 `unaspack V2.11c` 给 `OctoDll.dll` 和 `Windows 优化大师.exe` 脱壳

再用 `w32dasm V8.93` 反汇编脱壳后的可执行文件

## 漏洞定位

在 `Windows 优化大师.exe` 中：

```
:004E4D79 E8B6FCFFFF             Call 004E4A34       //进 OctoDll.dll 检查注册码

:004E4D7E 83F814                 cmp eax, 00000014    //返回值为 eax=14H 则正确

:004E4D81 7417                   je 004E4D9A          //标题栏变为"已注册"
```

因为跟踪发现退出后重新运行 `Windows 优化大师.exe` 又会执行到上面的代码，故考虑修改 `OctoDll.dll` 中得出返回值的代码：

```
:004043DC 85C0                   test eax, eax        //若 eax=0H             //若 eax<>0H

:004043DE 7504                   jne 004043E4         //这里不跳             //这里跳

:004043E0 33DB                   xor ebx, ebx         //置 ebx=0H

:004043E2 EB04                   jmp 004043E8         //ebx 赋给 eax 作为返回值

:004043E4 0FB75DF6               movzx ebx, word ptr [ebp-0A]             //把检查结果通过 ebx  赋给 eax 作为返回值

:004043E8 33C0                   xor eax, eax
```

在注册码错误的情况下，上面的执行过程都是错误的，干脆直接置 `ebx=14H` 再赋给 `eax` 作为返回值：

```
:004043DC 90                     nop

:004043DD BB14000000             mov ebx, 00000014

:004043E2 EB04                   jmp 004043E8
```

## 补丁实现

`OctoDll.dll` 中的代码应作如下修改：

```
85C0750433DBEB04 ==> 90BB14000000EB04
```

在 `Windows 优化大师.exe` 执行时代码修改地址为 `005F43DC`

由于 `Windows 优化大师.exe` 主程序对加壳的 `OctoDll.dll` 进行 `CRC` 校验，不能直接修改 `OctoDll.dll` 文件

故考虑通过 `Windows 优化大师.exe` 中在加壳代码还原后用 `SMC` 技术完成这项操作

用 `topo V1.2` 在加壳的 `Windows 优化大师.exe` 中拓展 `30` 个字节的空间，位于 `Offset=7CE19H` 处

注意选上重定向入口点（即将入口点改为 `Offset=7CE19H` 对应的绝对地址 `00577019`）及使代码可写

然后修改加壳的 `Windows 优化大师.exe` 中自 `Offset=7CE19H` 始的代码：

```
:00577019 66C705DC435F0090BB     mov word ptr [005F43DC], BB90

:00577022 C705DE435F0014000000   mov dword ptr [005F43DE], 00000014

:0057702C E913E40600             jmp 005E5444

:00577031 90
```

由此得到的 `Windows 优化大师.exe` 之彻底破解版可接受任意注册码，并可网上升级

另外未注册或注册失败时，注册表存在如下字符串键值：

```
HKEY_LOCAL_MACHINE\Software\Wom\Masters = "0F0F0F0F"
```

注册成功则此键值不存在

`Sysdoctor.exe` 会根据该键值存在与否决定 `Windows 优化大师` 是否正常注册过

-----------------------------------------------------------------------

## 深入内层的破解方法（OctoDll.dll）

在 `Windows 优化大师 V4.3` 的 `OctoDll.dll` 中：

```
:00404301 E8B2FDFFFF             call 004040B8

:00404306 83E802                 sub eax, 00000002

:00404309 8945E0                 mov dword ptr [ebp-20], eax

:0040430C C165DC02               shl dword ptr [ebp-24], 02
```

设法在 `00404310` 之前提取注册用户的正确校验码 `[ebp-1C]`，作校验过程的逆运算，用所得结果覆盖通过伪注册码计算得到的中间校验码 `[ebp-24]` 及 `[ebp-20]`

用 `SMC` 技术可这样实现：

先用 `topo V1.2` 在加壳的 `Windows 优化大师.exe` 中拓展 `61` 个字节的空间，位于 `Offset=7CE19H` 处

```
/005F4306 6810435F00             push 005F4310              //置返回地址

/005F430B E92B2DF8FF             jmp 0057703B
```

:00404310 8D4DDC                 lea ecx, dword ptr [ebp-24]

:00404313 8B01                   mov eax, dword ptr [ecx]

:00404315 8B5104                 mov edx, dword ptr [ecx+04]

:00404318 0FACD002               shrd eax, edx, 02          //双精度右移

:0040431C C1EA02                 shr edx, 02

:0040431F 8901                   mov dword ptr [ecx], eax

:00404321 895104                 mov dword ptr [ecx+04], edx

:00404324 8B45DC                 mov eax, dword ptr [ebp-24]

:00404327 3B45E4                 cmp eax, dword ptr [ebp-1C]

:0040432A 7404                   je 00404330

修改代码：

```
:00577019 66C70506435F006810     mov word ptr [005F4306], 1068      //修改代码,改变走向

:00577022 C70508435F00435F00E9   mov dword ptr [005F4308], E9005F43

:0057702C C7050C435F002B2DF8FF   mov dword ptr [005F430C], FFF82D2B

:00577036 E909E40600             jmp 005E5444      //回到原加壳的 Windows 优化大师.exe 的入口点

:0057703B 8B55E4                 mov edx, dword ptr [ebp-1C]        //注册用户的正确校验码

:0057703E C1EA1E                 shr edx, 1E

:00577041 83C250                 add edx, 00000050          //保存进位以备双精度右移

:00577044 8955E0                 mov dword ptr [ebp-20], edx

:00577047 8B55E4                 mov edx, dword ptr [ebp-1C]

:0057704A C1E202                 shl edx, 02                //双精度右移的反向操作

:0057704D 8955DC                 mov dword ptr [ebp-24], edx

:00577050 C3                     ret
```

* * *

**相关信息:** 麻烦，省点力改两字节：`83F8147417` 改作 `83F8147517`；`83F8147407` 改作 `83F8147507` (空)(**cray** 今天 19:58 阅读 0)

-----------------------------------------------------------------------

# Windows 优化大师 V4.31 破解

## 初步分析

经 `FI V2.45` 侦查得知安装得到的所有可执行文件都是用 `Aapack V2.11` 加的壳

可以分别用 `unaspack V2.11c` 给 `OctoDll.dll` 和 `Windows 优化大师.exe` 脱壳

在 `Windows 优化大师.exe` 中修改判断返回值的跳转使之成为彻底破解版（这里从略）

-----------------------------------------------------------------------

## 深入内层的破解方法（OctoDll.dll）

在 `Windows 优化大师 V4.31` 的 `OctoDll.dll` 中：

```
:00404301 E8B2FDFFFF             call 004040B8

:00404306 83E802                 sub eax, 00000002

:00404309 8945E0                 mov dword ptr [ebp-20], eax

:0040430C C165DC02               shl dword ptr [ebp-24], 02
```

设法在 `00404310` 之前提取注册用户的正确校验码 `[ebp-1C]`，作校验过程的逆运算，用所得结果覆盖通过伪注册码计算得到的中间校验码 `[ebp-24]` 及 `[ebp-20]`

用 `SMC` 技术可这样实现：

先用 `topo V1.2` 在加壳的 `Windows 优化大师.exe` 中开辟 `56` 个字节的空间，位于 `Offset=7D019H` 处

```
/005F4306 6810435F00             push 005F4310              //置返回地址

/005F430B E92B3DF8FF             jmp 0057803B
```

:00404310 8D4DDC                 lea ecx, dword ptr [ebp-24]

:00404313 8B01                   mov eax, dword ptr [ecx]

:00404315 8B5104                 mov edx, dword ptr [ecx+04]

:00404318 0FACD002               shrd eax, edx, 02          //双精度右移

:0040431C C1EA02                 shr edx, 02

:0040431F 8901                   mov dword ptr [ecx], eax

:00404321 895104                 mov dword ptr [ecx+04], edx

:00404324 8B45DC                 mov eax, dword ptr [ebp-24]

:00404327 3B45E4                 cmp eax, dword ptr [ebp-1C]

:0040432A 7404                   je 00404330

修改代码：

```
:00578019 66C70506435F006810     mov word ptr [005F4306], 1068      //修改代码,改变走向

:00578022 C70508435F00435F00E9   mov dword ptr [005F4308], E9005F43

:0057802C C7050C435F002B3DF8FF   mov dword ptr [005F430C], FFF83D2B

:00578036 E9C6DF0600             jmp 005E6001      //回到原加壳的 Windows 优化大师.exe 的入口点

:0057803B 8B55E4                 mov edx, dword ptr [ebp-1C]        //注册用户的正确校验码

:0057803E C1EA1E                 shr edx, 1E

:00578041 83C250                 add edx, 00000050          //保存进位以备双精度右移

:00578044 8955E0                 mov dword ptr [ebp-20], edx

:00578047 8B55E4                 mov edx, dword ptr [ebp-1C]

:0057804A C1E202                 shl edx, 02                //双精度右移的反向操作

:0057804D 8955DC                 mov dword ptr [ebp-24], edx

:00578050 C3                     ret
```

由此得到的 `Windows 优化大师.exe` 之彻底破解版可接受任意注册码，并可网上升级

另外未注册或注册失败时，注册表存在如下字符串键值：

```
HKEY_LOCAL_MACHINE\Software\Wom\Masters = "0F0F0F0F"
```

注册成功则此键值不存在

`Sysdoctor.exe` 会根据该键值存在与否决定 `Windows 优化大师` 是否正常注册过

## 附注

经由上面这种改动，`Windows 优化大师.exe` 相对于原来的文件的变化可以总结如下：

1) 修改原 `Windows 优化大师.exe` 文件的文件头 `Header`

即将 `128H~12BH` 这个 `dword` 单元的 `01601E00`（就是说原 `Header` 位于 `00400000+001E6001=005E6001H`）

改为 `19801700`（故新的 `Header` 的 `RVA`: `00578019-00400000=00178019H`, 所以把 `01601E` 改为 `198017`）

2) 开辟的 `56` 个字节的空间，位于 `Offset=7D019H` 处，所以把 `7D019H` 起的 `56` 个字节修改为：

```
:00578019 66C70506435F006810     mov word ptr [005F4306], 1068

:00578022 C70508435F00435F00E9   mov dword ptr [005F4308], E9005F43

:0057802C C7050C435F002B3DF8FF   mov dword ptr [005F430C], FFF83D2B

:00578036 E9C6DF0600             jmp 005E6001

:0057803B 8B55E4                 mov edx, dword ptr [ebp-1C]

:0057803E C1EA1E                 shr edx, 1E

:00578041 83C250                 add edx, 00000050

:00578044 8955E0                 mov dword ptr [ebp-20], edx

:00578047 8B55E4                 mov edx, dword ptr [ebp-1C]
```



`:0057804A` `C1E202` `shl edx, 02`

`:0057804D` `8955DC` `mov dword ptr [ebp-24], edx`

`:00578050` `C3` `ret` `4dd466d0e1e0cc123afbe504da2462ca`
