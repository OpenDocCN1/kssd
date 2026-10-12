
```
004039A6   . |898D 54FEFFFF mov dword ptr ss:[ebp-1AC], ecx
004039AC   . |8D95 ECFDFFFF lea edx, dword ptr ss:[ebp-214]
004039B2   . |8D8D 5CFEFFFF lea ecx, dword ptr ss:[ebp-1A4]
004039B8   . |8985 2CFEFFFF mov dword ptr ss:[ebp-1D4], eax
004039BE   . |8985 3CFEFFFF mov dword ptr ss:[ebp-1C4], eax
004039C4   . |8985 4CFEFFFF mov dword ptr ss:[ebp-1B4], eax
004039CA   . |C785 F4FDFFFF>mov dword ptr ss:[ebp-20C], crac.004026C4
004039D4   . |C785 ECFDFFFF>mov dword ptr ss:[ebp-214], 8
004039DE   . |FF15 DC104000 call dword ptr ds:[<&MSVBVM60.__vbaVarDup>]        004039E4   . |8D8D 2CFEFFFF lea ecx, dword ptr ss:[ebp-1D4]
004039EA   . |8D95 3CFEFFFF lea edx, dword ptr ss:[ebp-1C4]
004039F0   . |51            push ecx
004039F1   . |8D85 4CFEFFFF lea eax, dword ptr ss:[ebp-1B4]
004039F7   . |52            push edx
004039F8   . |50            push eax
004039F9   . |8D8D 5CFEFFFF lea ecx, dword ptr ss:[ebp-1A4]
004039FF   . |6A 00         push 0
00403A01   . |51            push ecx
00403A02   . |FF15 40104000 call dword ptr ds:[<&MSVBVM60.#595>]               ;  rtcMsgBox,正确弹出对话框
00403A08   . |8D95 2CFEFFFF lea edx, dword ptr ss:[ebp-1D4]
00403A0E   . |8D85 3CFEFFFF lea eax, dword ptr ss:[ebp-1C4]
00403A14   . |52            push edx
00403A15   . |8D8D 4CFEFFFF lea ecx, dword ptr ss:[ebp-1B4]
00403A1B   . |50            push eax
00403A1C   . |8D95 5CFEFFFF lea edx, dword ptr ss:[ebp-1A4]
00403A22   . |51            push ecx
00403A23   . |52            push edx
00403A24   . |6A 04         push 4
00403A26   . |FF15 14104000 call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]    00403A2C   . |83C4 14        add esp, 14
00403A2F   > \33FF         xor edi, edi
00403A31   >  897D FC       mov dword ptr ss:[ebp-4], edi
```

到了这里，这篇文章也就到了结尾，希望能带给大家更多的启发，我们已经通过了 `WindowsXP` 万能断点的亲身体验，其实方法与工具的结合要灵活运用，由于每款软件的编制手法不同，所以调试方法有很多种，但终究有一种应该适合您的。

**[参考]** `Point-h` in `WinXP` tutorial

**[附件]** `crac.exe`

**[作者]** `Moodsky`

**[日期]** 2005-11-6

本来是没有打算这篇文章的，看到在上篇文章中 `yumikeyabc` 兄弟提供了一个直接找 `Point-h` 的工具（16 楼），下载后试用了一下，发现得到的地址是不对的，以前也用过另一个，同样得到的 `Point-h` 地址不对，所以就自己动手写了一个垃圾品，还是老话，大家不要扔臭鸡蛋、烂柿子就好了，抛砖引玉嘛。

原理和上篇里面讲解的一样，从 `user32.dll` 文件里面搜索 `F3 A5 8B C8 83 E1 03 F3 A4 E8` 数值，用得到的值 `+` 基址 `+` 这个值所在区段的虚拟偏移 `–` 原始偏移量就可以了，代码很简单。
