
# Armadillo 3.78 -> Silicon Realms Toolworks

> 原文：[www.pediy.com/kssd/pediy07/pediy7-588.htm](https://www.pediy.com/kssd/pediy07/pediy7-588.htm)

原创：Armadillo 3.78 -> Silicon Realms Toolworks

应聘成功之后的第一个杰作。

软件名称: 天骄外挂

下载地址：`http://down1.tj2wg.com/feifei/0703C.exe`

加壳方式：Armadillo 3.78 -> Silicon Realms Toolworks

破解工具：`OllyDbg v1.10` 原版，`ImportREC 1.6 Final`，`LordPE`

脱壳作者：夜凉如水

老规矩忽略所有异常，在添加以下几个：

*   `C0000005` (`ACCESS VIOLATION`)
*   `C000001D` (`ILLEGAL INSTRUCTION`)
*   `C000001E` (`INVALID LOCK SEQUENCE`)
*   `C0000096` (`PRIVILEGED INSTRUCTION`)

用 `OD` 载入，先下 `OpenMutexA` 断点 `shift+F9` 运行

```
047CE000 T> 60 pushad
047CE001 E8 00000000 call TJMan.047CE006
047CE006 5D pop ebp
047CE007 50 push eax
047CE008 51 push ecx
047CE009 0FCA bswap edx
047CE00B F7D2 not edx
047CE00D 9C pushfd
047CE00E F7D2 not edx
047CE010 0FCA bswap edx
047CE012 EB 0F jmp short TJMan.047CE023
```

看到堆栈

```
0012D784 047A5398 /CALL 到 OpenMutexA 来自 TJMan.047A5392
0012D788 001F0001 |Access = 1F0001
0012D78C 00000000 |Inheritable = FALSE
0012D790 0012DDC4 \MutexName = "E24:A920F9B22"
```

`7C80EC1B` `k>` `8BFF` `mov edi,edi` 断在这里

```
7C80EC1D 55 push ebp
7C80EC1E 8BEC mov ebp,esp
7C80EC20 51 push ecx
7C80EC21 51 push ecx
7C80EC22 837D 10 00 cmp dword ptr ss:[ebp+10],0
7C80EC26 56 push esi
7C80EC27 0F84 7A500300 je kernel32.7C843CA7
```

`Ctrl+G` `401000` 键入以下代码

```
00401000 60 pushad
00401001 9C pushfd
00401002 68 B4FB1200 push 0012DDC4★ 堆栈里看到的值
00401007 33C0 xor eax,eax
00401009 50 push eax
0040100A 50 push eax
0040100B E8 B4B2A577 call kernel32.CreateMutexA
00401010 9D popfd
00401011 61 popad
00401012 E9 33F7A577 jmp kernel32.OpenMutexA
```

在 `401000` 处新建起源，右键－》此处新建 `EIP`

`F9` 运行，再次中断在 `OpenMutexA` 处，取消断点。

再次 `Ctrl+G` `401000`

撤消刚才做的选择，右键-》撤消选择

2、下 `GetModuleHandleA` 断点 `F9` 运行

`7C80B529` `k>` `8BFF` `mov edi,edi` 断在这里，取消断点

```
7C80B52B 55 push ebp
7C80B52C 8BEC mov ebp,esp
7C80B52E 837D 08 00 cmp dword ptr ss:[ebp+8],0
7C80B532 74 18 je short kernel32.7C80B54C 再次下断
```

按 `F9`，注意堆栈

```
0012CE4C /0012CE84
0012CE50 |5D175394 返回到 5D175394 来自 kernel32.GetModuleHandleA
0012CE54 |5D1753E0 ASCII "kernel32.dll"

0012CF0C /0012CF28
0012CF10 |77F45BD8 返回到 SHLWAPI.77F45BD8 来自 kernel32.GetModuleHandleA
0012CF14 |77F4501C ASCII "KERNEL32.DLL"

0012D724 /0012D78C
0012D728 |047A44D3 返回到 TJMan.047A44D3 来自 kernel32.GetModuleHandleA

00127AB0 /0012CDDC
00127AB4 |04E94510 返回到 04E94510 来自 kernel32.GetModuleHandleA
00127AB8 |04EA7B20 ASCII "kernel32.dll"
00127ABC |04EA8BF8 ASCII "VirtualAlloc"

00127AB0 /0012CDDC
00127AB4 |04E9452D 返回到 04E9452D 来自 kernel32.GetModuleHandleA
00127AB8 |04EA7B20 ASCII "kernel32.dll"
00127ABC |04EA8BEC ASCII "VirtualFree"

00127814 /00127AB4
00127818 |04E847CB 返回到 04E847CB 来自 kernel32.GetModuleHandleA
0012781C |00127968 ASCII "kernel32.dll"//看到这里就是返回时机了
```

此时，取消断点，`ALT+F9` 返回

```
04E847CB 8B0D 7CF6EA04 mov ecx,dword ptr ds:[4EAF67C]
04E847D1 89040E mov dword ptr ds:[esi+ecx],eax
04E847D4 A1 7CF6EA04 mov eax,dword ptr ds:[4EAF67C]
04E847D9 391C06 cmp dword ptr ds:[esi+eax],ebx
04E847DC 75 16 jnz short 04E847F4
04E847DE 8D85 B4FEFFFF lea eax,dword ptr ss:[ebp-14C]
04E847E4 50 push eax
04E847E5 FF15 0C21EA04 call dword ptr ds:[4EA210C] ; kernel32.LoadLibraryA
04E847EB 8B0D 7CF6EA04 mov ecx,dword ptr ds:[4EAF67C]
04E847F1 89040E mov dword ptr ds:[esi+ecx],eax
04E847F4 A1 7CF6EA04 mov eax,dword ptr ds:[4EAF67C]
04E847F9 391C06 cmp dword ptr ds:[esi+eax],ebx
04E847FC 0F84 2F010000 je 04E84931//修改为 JMP
04E84802 33C9 xor ecx,ecx
04E84804 8B07 mov eax,dword ptr ds:[edi]
04E84806 3918 cmp dword ptr ds:[eax],ebx
```

第三、下 `GetCurrentThreadId` 断点 按 `F9` 运行

注意看堆栈

```
001270D8 66001E3A /CALL 到 GetCurrentThreadId 来自 66001E34
001270DC 00000001
001270E0 66001C1E 返回到 66001C1E 来自 66001DE6
001270E4 66001B64 返回到 66001B64 来自 66001B90
001270B8 66003505 /CALL 到 GetCurrentThreadId 来自 660034FF
00127328 73DC9AD7 /CALL 到 GetCurrentThreadId 来自 MFC42.73DC9AD1
00127320 73DC9AD7 /CALL 到 GetCurrentThreadId 来自 MFC42.73DC9AD1
0012D76C 04E84B97 /CALL 到 GetCurrentThreadId 来自 04E84B91 //返回时机
```

取消断点 `ALT+F9` 返回这里：

```
04E84B97 50 push eax //f8 f7 跟踪
04E84B98 FF75 FC push dword ptr ss:[ebp-4]
04E84B9B E8 05000000 call 04E84BA5
04E84BA0 83C4 0C add esp,0C
04E84BA3 C9 leave
04E84BA4 C3 retn

04E9CA4B 8B50 6C mov edx,dword ptr ds:[eax+6C]
04E9CA4E 3350 50 xor edx,dword ptr ds:[eax+50]
04E9CA51 3350 18 xor edx,dword ptr ds:[eax+18]
04E9CA54 2BCA sub ecx,edx
04E9CA56 FFD1 call ecx F7 进入飞向光明之巅 ; TJMan.0042C4D2

0042C4D2 55 push ebp //oep=0042C4D2-400000=2C4D2 用 LordPE Dump！
0042C4D3 8BEC mov ebp,esp
0042C4D5 6A FF push -1
0042C4D7 68 F8244300 push TJMan.004324F8
0042C4DC 68 36C64200 push TJMan.0042C636 ; jmp to msvcrt._except_handler3
0042C4E1 64:A1 00000000 mov eax,dword ptr fs:[0]
0042C4E7 50 push eax
```

注意不要关 `OD`，直接打开 `ImprotREC` 否则不能修复成功，选择我们要脱的进程。然后输入 `OEP` `2C4D2`。自动搜索 `IAT` -> 获取输入表，有一个假指针 -> 点显示无效函数 -> 右键 -> 剪切指针 -> 修复抓取文件

呵呵~~成功

`4dd466d0e1e0cc123afbe504da2462ca`
