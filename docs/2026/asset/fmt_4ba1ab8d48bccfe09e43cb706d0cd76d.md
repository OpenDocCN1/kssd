
0047E17C  77E560E1  `kernel32.lstrlenA`
0047E180  77E5274E  `kernel32.lstrcpynA`
0047E184  00B5B400
0047E188  77E5DA41  `kernel32.GetThreadLocale`
0047E18C  77E4177E  `kernel32.GetStartupInfoA`
0047E190  00B5A4EB
0047E194  00B5E2DA
0047E198  77E5ADA9  `kernel32.GetModuleFileNameA`
0047E19C  77E57362  `kernel32.GetLocaleInfoA`
0047E1A0  00B5ED84
0047E1A4  00B5B82A
0047E1A8  00B5DCF6
0047E1AC  77E5EE9D  `kernel32.FindClose`
0047E1B0  00B5BDB8
0047E1B4  00B5CA49
0047E1B8  77E730C0  `kernel32.UnhandledExceptionFilter`
0047E1BC  77F60C44  `ntdll.RtlUnwind`
0047E1C0  77E53837  `kernel32.RaiseException`
0047E1C4  77E5E494  `kernel32.GetStdHandle`
0047E1C8  00B5AF8E
0047E1CC  77D38A76  `USER32.GetKeyboardType`
0047E1D0  77D16E00  `USER32.LoadStringA`
0047E1D4  00B5E353
0047E1D8  77D1A310  `USER32.CharNextA`
0047E1DC  00B5AF22
0047E1E0  77DA2410  `ADVAPI32.RegQueryValueExA`
0047E1E4  77DA229A  `ADVAPI32.RegOpenKeyExA`
0047E1E8  00B5E403
0047E1EC  00B5AF22
0047E1F0  770F174B
0047E1F4  770FF42E
0047E1F8  770F1674
0047E1FC  00B5AEE5
0047E200  77E5AA4F  `kernel32.TlsSetValue`
0047E204  77E5ABF1  `kernel32.TlsGetValue`
0047E208  77E5A682  `kernel32.LocalAlloc`
0047E20C  00B5E2DA

在`0047E13C`处已无数据，则`RAV=47e13c-40000=7e13c`，关闭`OD`，接着进行另外一步了

## 重新附加进程

重新把原程序载入`OD`，下断`bp DebugActiveProcess`，`Shift+F9`运行，注意堆栈处

```
0012BCB0   004A6E60  /CALL 到 DebugActiveProcess 来自 loginTJ2.004A6E5A
0012BCB4   00000B14  \ProcessId = B14         ；这个是子进程ID，你的不一定相同
```

另开一个`OD`，在附加处选择`B14`这个进程，`Alt+F9`返回

```
004B6E10 >- EB FE           jmp short loginTJ2.<ModuleEntryPoint>  ；停在这里，修改为55 8b
004B6E12    EC              in al,dx
```

下断`BP OpenMutexA`运行(先不要取消断点)，断下后前往`00401000`处

## 填充欺骗代码

```
00401000     60                pushad
00401001     9C                pushfd
00401002    68 E0DD1200       push 12DDE0                             ; 堆栈处看到的ASCII "B14::DA7D1B79B5"
00401007     33C0              xor eax,eax
00401009     50                push eax
0040100A     50                push eax
0040100B     E8 2FDB407C       call kernel32.CreateMutexA
00401010     9D                popfd
00401011     61                popad
00401012   - E9 04DC407C       jmp kernel32.OpenMutexA
60 9C 68 E0 DD 12 00 33 C0 50 50 E8 B5 A6 A5 77 9D 61 E9 7A 13 A6 77
```

新建`EIP`，`Shift+F9`运行，取消`BP OpenMutexA`断点，再次来到`00401000`处，取消取消上面修改

下断`BP GetModuleHandleA+5`，`Shift+F9`，注意观察堆栈！

```
0012CECC   77C059FC  返回到 msvcrt.77C059FC 来自 kernel32.GetModuleHandleA
0012CED0   77BE31AC  ASCII "kernel32.dll"
0012CED4   77C2CA20  msvcrt.77C2CA20
0012CF90   772A8663  返回到 SHLWAPI.772A8663 来自 kernel32.GetModuleHandleA
0012CF94   772AF8FC  ASCII "KERNEL32.DLL"
0012CF98   00000000
0012CEB8   770FB124  返回到 OLEAUT32.770FB124 来自 kernel32.GetModuleHandleA
0012CEBC   771722E4  ASCII "KERNEL32.DLL"
0012CEB4   770FB124  返回到 OLEAUT32.770FB124 来自 kernel32.GetModuleHandleA
0012CEB8   771722E4  ASCII "KERNEL32.DLL"
00127B98   00B71B31  返回到 00B71B31 来自 kernel32.GetModuleHandleA
00127B9C   00B86364  ASCII "kernel32.dll"
00127BA0   00B87588  ASCII "VirtualAlloc"
00127B98   00B71B4E  返回到 00B71B4E 来自 kernel32.GetModuleHandleA
00127B9C   00B86364  ASCII "kernel32.dll"
00127BA0   00B8757C  ASCII "VirtualFree"
00127908   00B59CD7  返回到 00B59CD7 来自 kernel32.GetModuleHandleA
0012790C   00127A4C  ASCII "kernel32.dll"
```

不断`Shft+F9`直到经过`VirtualAlloc`、`VirtualFree`再次在堆栈中看见`kernel32.dll`，是返回的时机了，取消断点，`ALT+F9`返回

```
00B59CD7    8B0D 74B7B800   mov ecx,dword ptr ds:[B8B774]
00B59CDD    89040E          mov dword ptr ds:[esi+ecx],eax
00B59CE0    A1 74B7B800     mov eax,dword ptr ds:[B8B774]
00B59CE5    391C06          cmp dword ptr ds:[esi+eax],ebx
00B59CE8    75 16           jnz short 00B59D00
00B59CEA    8D85 B4FEFFFF   lea eax,dword ptr ss:[ebp-14C]
00B59CF0    50              push eax
00B59CF1    FF15 DC00B800   call dword ptr ds:[B800DC]              ; kernel32.LoadLibraryA
00B59CF7    8B0D 74B7B800   mov ecx,dword ptr ds:[B8B774]
00B59CFD    89040E          mov dword ptr ds:[esi+ecx],eax
00B59D00    A1 74B7B800     mov eax,dword ptr ds:[B8B774]
00B59D05    391C06          cmp dword ptr ds:[esi+eax],ebx
00B59D08    0F84 3B010000   je 00B59E49         ；LoadLibraryA下来的第一个跳转就是传说中的Magic Jump！修改成jmp！
```

`F9`直接运行了

## ImportREC Dump

打开`ImportREC`选择`B14`这个进程

`OEP=6e70c`

`RAV=7e13c`

`SIZE=1000`

直接点击获取输入表，`CUT`掉无效的指针修复`dumped`即可以了，这里注意一个问题，看看你的`ImportREC`设置，我这里选项中如果选了新建输入表中的三项任一项，都修复不成功，不知什么原因，我把这三项去掉修复即成功了

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
