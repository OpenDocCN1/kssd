
```
0081C0ED  ^ 75 F1           jnz short ZZZX添加.0081C0E0
0081C0EF    E8 00000000     call ZZZX添加.0081C0F4
0081C0F4    58              pop eax
0081C0F5    83C0 7A         add eax,0x7A
0081C0F8    8D8B 00FFFFFF   lea ecx,dword ptr ds:[ebx-0x100]
0081C0FE    8908            mov dword ptr ds:[eax],ecx
0081C100    83C0 06         add eax,0x6
0081C103    8D8B 00FFFFFF   lea ecx,dword ptr ds:[ebx-0x100]
0081C109    8908            mov dword ptr ds:[eax],ecx
0081C10B    83C0 05         add eax,0x5
0081C10E    8D8B 00FEFFFF   lea ecx,dword ptr ds:[ebx-0x200]
0081C114    8908            mov dword ptr ds:[eax],ecx
0081C116    83C0 0A         add eax,0xA
0081C119    8D8B 00FEFFFF   lea ecx,dword ptr ds:[ebx-0x200]
0081C11F    8908            mov dword ptr ds:[eax],ecx
0081C121    83C0 11         add eax,0x11
0081C124    8D8B 00FEFFFF   lea ecx,dword ptr ds:[ebx-0x200]
0081C12A    8908            mov dword ptr ds:[eax],ecx
0081C12C    83E8 05         sub eax,0x5
0081C12F    8D8B 00FDFFFF   lea ecx,dword ptr ds:[ebx-0x300]
0081C135    8908            mov dword ptr ds:[eax],ecx
0081C137    2D D2000000     sub eax,0xD2
0081C13C    8D8B 24F5FFFF   lea ecx,dword ptr ds:[ebx-0xADC]
0081C142    8908            mov dword ptr ds:[eax],ecx
0081C144    05 1F070000     add eax,0x71F
0081C149    C700 D6D0B9FA   mov dword ptr ds:[eax],0xFAB9D0D6                        //此处开始可以定义弹窗标题。
0081C14F    C740 04 C6AED4C>mov dword ptr ds:[eax+0x4],0xC6D4AEC6
0081C156    C740 08 B8F3A1B>mov dword ptr ds:[eax+0x8],0xBEA1F3B8
0081C15D    C740 0C 504447A>mov dword ptr ds:[eax+0xC],0xA1474450
0081C164    C740 10 BF00000>mov dword ptr ds:[eax+0x10],0xBF                            //弹窗标题结束
0081C16B    9D              popfd
0081C16C    61              popad
0081C16D    A3 CAC78100     mov dword ptr ds:[0x81C7CA],eax
0081C172    FF35 CAC78100   push dword ptr ds:[0x81C7CA]
0081C178    68 CAC68100     push ZZZX添加.0081C6CA
0081C17D    E8 25000000     call <jmp.&VMProtectSDK32.VMProtectGetCu>
0081C182    68 CAC68100     push ZZZX添加.0081C6CA
0081C187    E8 21FFFFFF     call ZZZX添加.0081C0AD
0081C18C    6A 00           push 0x0
0081C18E    68 CAC58100     push ZZZX添加.0081C5CA
0081C193    68 CAC68100     push ZZZX添加.0081C6CA
0081C198    6A 00           push 0x0
0081C19A    E8 1A000000     call <jmp.&user32.MessageBoxA>
0081C19F    6A 00           push 0x0
0081C1A1    E8 25000000     call <jmp.&kernel32.ExitProcess>
0081C1A6    CC              int3
0081C1A7  - FF25 48D08100   jmp dword ptr ds:[<&VMProtectSDK32.VMPro>; VMProtec.VMProtectGetCurrentHWID        //现在只需要手动修复以下函数地址就好了。
0081C1AD  - FF25 E8765F00   jmp dword ptr ds:[<&user32.CloseClipboar>; user32.CloseClipboard
0081C1B3  - FF25 8C765F00   jmp dword ptr ds:[<&user32.EmptyClipboar>; user32.EmptyClipboard
0081C1B9  - FF25 C4705F00   jmp dword ptr ds:[<&user32.MessageBoxA>] ; user32.MessageBoxA
0081C1BF  - FF25 1C755F00   jmp dword ptr ds:[<&user32.OpenClipboard>; user32.OpenClipboard
0081C1C5  - FF25 CC745F00   jmp dword ptr ds:[<&user32.SetClipboardD>; user32.SetClipboardData
0081C1CB  - FF25 7C705F00   jmp dword ptr ds:[<&kernel32.ExitProcess>; kernel32.ExitProcess
0081C1D1  - FF25 C8715F00   jmp dword ptr ds:[<&kernel32.GlobalAlloc>; kernel32.GlobalAlloc
0081C1D7  - FF25 B8715F00   jmp dword ptr ds:[<&kernel32.GlobalLock>>; kernel32.GlobalLock
0081C1DD  - FF25 A8715F00   jmp dword ptr ds:[<&kernel32.GlobalUnloc>; kernel32.GlobalUnlock
```

视频及附件下载：

视频解压密码:aa5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9h3&6S2M7s2W2Y4i4K6u0W2j5$3!0E0

抱歉，忘记换密码重新打包了。明天有时间再换附件。

链接:f73K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0U0y4b7d9e0q4*7y4q4N6v1b7i4q4i4c8V1S2C8f1K6u0Z5P5V1y4y4f1g2!0q4y4W2)9^5c8W2)9&6x3q4!0q4y4g2)9^5c8W2)9&6y4W2!0q4y4#2!0m8x3q4)9^5x3g2)9K6b7i4q4F1L8U0u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7

完整版附件及使用方法视频：

链接: 0caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4y4q4m8b7M7@1!0t1i4K6g2X3k6q4A6E0i4K6g2X3M7h3k6@1d9e0W2S2g2q4p5J5y4$3M7`. 提取码: 7q4v

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-3-1 00:36

被daohaodaye编辑，原因： 补充解决几个硬编码问题。可以少修复几处了。
