
# 瑞星个人防火墙 1.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue260.htm`](https://www.pediy.com/kssd/pediy04/kanxue260.htm)

瑞星个人防火墙 1.1 版破解手记。

原版从瑞星网站下载，需要 `ID` 号。

目前为 1.1 版，安装时需瑞星密钥盘。

文件名称 `rfw.exe`。

破解工具: `w32dasm` 黄金版。

1. 用 `winzip` 或 `winrar` 等解压缩软件将 `rfw.exe` 展到一个目录下
2. 研究目录下的 `setup.exe`
3. 用 `w32dasm` 黄金版打开 `setup.exe`
   ==》串式参考 "Please insert the Rising 'A' floppy. Please contact

   ```
   String Resource ID=00109: "For ensuring the installation to succeed, please close the L"
   String Resource ID=00111: "Unable to find Notes.ini file!"
   String Resource ID=00112: "Setup program will create shortcut model on your desktop, wh"
   String Resource ID=00113: "You are  not "Administrator", install maybe fail.
   you'd bet"
   String Resource ID=00116: "You have installed Rising Personal Firewall\[version:%s\], now"
   String Resource ID=00117: "You are installing version lower than\[%s\] the one you used \["
   String Resource ID=00119: "
   \\setup path:
   
      %s
   program group name:
   
      %s
   Disk free sp"
   String Resource ID=00120: "For ensuring the installation to succeed, please close Lotus"
   String Resource ID=00121: "Would you like install Rising Anti-virus software?"
   String Resource ID=00122: "Please insert the Rising 'A' floppy. Please contact dealer i"    \*\*\*\*\*\*\*\*\*\*\*\*
   String Resource ID=00123: "After long years technology tracking on network attacking me"
   String Resource ID=61440: "S"
   String Resource ID=61446: ""
   String Resource ID=61472: "?0?醥"
   String Resource ID=61473: "政 gL
   
   /勍\\""
   String Resource ID=61474: "?
   String Resource ID=61475: "匵
   ?"
   ```

   只有一处

   ```
   :0040C6AC 50                    push eax
   :0040C6AD 8BCF                  mov ecx, edi
   :0040C6AF E81CEDFFFF           call 0040B3D0
   :0040C6B4 33DB                 xor ebx, ebx
   :0040C6B6 83CDFF               or ebp, FFFFFFFF
   :0040C6B9 85C0                 test eax, eax
   :0040C6BB EB7D                 jmp 0040C73A
   ```
   这是修改后的, 74 还是 75 => `EB`, 跳过检测钥匙盘 \*\*\*\*\*\*\*\*\*\*\*\*

   \* Referenced by a (U)nconditional or (C)onditional Jump at Address:
   | :0040C738(C)
   |

   ```
   :0040C6BD A1C0C24300           mov eax, dword ptr [0043C2C0]
   :0040C6C2 89442410            mov dword ptr [esp+10], eax
   :0040C6C6 899C24F0060000      mov dword ptr [esp+000006F0], ebx
   :0040C6CD 89442414            mov dword ptr [esp+14], eax
   ```
   \* Possible Reference to String Resource ID=00122: "Please insert the Rising 'A' floppy. Please contact dealer i"
   |

   ```
   :0040C6D1 6A7A                 push 0000007A
   :0040C6D3 8D4C2414             lea ecx, dword ptr [esp+14]
   :0040C6D7 C68424F406000001     mov byte ptr [esp+000006F4], 01
   :0040C6DF E8F4B00100           call 004277D8
   ```
   \* Possible Reference to String Resource ID=00008: "Setup tips"
   |

   ```
   :0040C6E4 6A08                 push 00000008
   :0040C6E6 8D4C2418             lea ecx, dword ptr [esp+18]
   :0040C6EA E8E9B00100           call 004277D8
   :0040C6EF 8B4C2414             mov ecx, dword ptr [esp+14]
   :0040C6F3 8B542410             mov edx, dword ptr [esp+10]
   ```

4. 运行目录下的安装主程序 `setup.exe`，哇塞，原来很温柔，小家伙蛮可爱的
5. 也可修改目录下的 `setup.dll` 进行破解，具体参看论坛精华中我的文章

作者 风飘雪

主页 `http://fpx.yeah.net`

2001.11.25 4dd466d0e1e0cc123afbe504da2462ca
