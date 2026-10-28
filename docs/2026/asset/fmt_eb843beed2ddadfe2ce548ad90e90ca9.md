
在函数`00418C4C`中层层调用，最终是调用`WriteProcessMemory`函数恢复指令码的：

```
00461814  |> \6A 00         push    0                             ; /pBytesWritten = NULL
00461816  |.  A1 685A4D00   mov     eax, dword ptr [4D5A68]       ; |
0046181B  |.  8B55 10       mov     edx, dword ptr [ebp+10]       ; |
0046181E  |.  52            push    edx                           ; |BytesToWrite
0046181F  |.  8B4D 08       mov     ecx, dword ptr [ebp+8]        ; |
00461822  |.  51            push    ecx                           ; |Buffer
00461823  |.  56            push    esi                           ; |Address
00461824  |.  50            push    eax                           ; |hProcess => 0000026C (window)
00461825  |.  E8 F8D90400   call    <jmp.&KERNEL32.WriteProcessMe>; \WriteProcessMemory
```

在运行完该指令之后，OD又把该指令码设为`Int3`断点，即`0xCC`。

关键`Call`是函数`41B5A4`，在这个函数里，调用`WriteMemory`设置`0xCC`断点，而`WriteMemory`最终调用的还是上面所说的`WriteProcessMemory`。

```
0041B6A4  |.  C60424 CC     |mov     byte ptr [esp], 0CC                        ;直接设置0xCC断点
0041B6A8  |.  6A 02         |push    2                              ; /Arg4 = 00000002
0041B6AA  |.  6A 01         |push    1                              ; |Arg3 = 00000001
0041B6AC  |.  55            |push    ebp                            ; BreakPointAddress
0041B6AD  |.  8D5424 0C     |lea     edx, dword ptr [esp+C]             ; [esp+C] = 0xCC
0041B6B1  |.  52            |push    edx                            ; [edx] = 0xCC
0041B6B2  |.  E8 71600400   |call    \_Writememory                    ; \\_Writememory
```

通过以上方法，OD实现了在运行时动态处理`Int3`断点的功能。

武汉科锐学员: `driverox`

2008-5-19

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
