
`00408525` | `E8 06E6FFFF` | `call avprp.00406B30` ; `\\avprp.00406B30`

`0040852A` | `59` | `pop ecx`

`0040852B` | `EB 0D` | `jmp short avprp.0040853A`

### 【破解总结】

虽然不知道这样操作的原理，但是至少分析了 `AVPR` 的操作原理，高兴呀～

分析结果能干什么我就不说了。可以来个 `FILE PATCH`，也可以做自己的 `VBA BACKDOOR`。但是关于 `dll` 的读操作我还是有个疑问，我只知道通过一系列 API 找到窗口，得到进程，打开，写。`FindWindow` -> `GetWindowThreadProcessId` -> `OpenProcess` -> `WriteProcessMemory` 那样的话内存地址不固定，请各位指点。

在刚开始的对话框之前有如下代码，请问是否有关系呢？

```
00408113 |. E8 083D0D00 call <jmp.&user32.GetFocus> ; |\[GetFocus
00408118 |. 50 push eax ; |hOwner
00408119 |. E8 8E3E0D00 call <jmp.&user32.MessageBoxA> ; \\MessageBoxA
0040811E |. E8 219FFFFF call avprp.00402044
00408123 |> 6A FF /push -1 ; /Timeout = INFINITE
00408125 |. 8D85 0CFDFFFF |lea eax,dword ptr ss:\[ebp\-2F4\] ; |
0040812B |. 50 |push eax ; |pDebugEvent
0040812C |. E8 B7370D00 |call <jmp.&kernel32.WaitForDebugEv>; \\WaitForDebugEvent
00408131 |. 85C0 |test eax,eax
00408133 |. 75 12 |jnz short avprp.00408147
```

`4dd466d0e1e0cc123afbe504da2462ca`
