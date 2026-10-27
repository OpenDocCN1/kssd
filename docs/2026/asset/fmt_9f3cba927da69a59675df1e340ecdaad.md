
# [原创]一个RC4加密程序（Secure PE V1.6）的分析

一个RC4加密程序的分析

快雪时晴

2009-2-25

目标程序Secure PE V1.6

=============================================================================================

> [7e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5K9h3&6C8j5$3!0J5k6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@4b6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5K9h3&6C8j5$3!0J5k6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.)

> ThinkCore Research

> Menu

> Home | Articles | Utilities | Laboratory | Blog | About | Donate | Contact

> Secure PE

> SecurePE is a user friendly utility that protects Microsoft Windows? PE files with a password, strong RC4 encryption and checksum while leaving them totally functional.

> Read more ?

> Description

> SecurePE is a user friendly freeware utility coded in 32bit assembly language that protects Microsoft Windows? PE files with a password, strong RC4 encryption and checksum while leaving them totally functional.

> SecurePE user-friendly interface supports both drag & drop and shell operation of Windows Explorer. So you can protect files easily with a mouse click.
> When a protected file is executed it displays a dialogbox asking the user for the correct password. The password is not stored in the file but a CRC32 hash algorithm. You must remember the password or you will not be able to execute the protected program again until you restore it from a backup. With the correct password you can run the program and even deprotect the file and give back it to its original state.
> Unlike other protection utilities you may copy SecurePE protected files to another computer and it stays protected and functional without the need of re-protect the file again no matter what operating system this computer runs.
> At last, but not at least, SecurePE has a checksum routine that detects any manipulation in the PE Header of the protected files.

> System Requirements

> Personal Computer (PC) with at least 386 CPU.
> Microsoft Windows 2008; Windows Vista; Windows 2003, Windows XP; Windows 2000; Windows NT4 (SP6a); Windows ME; Windows 9x.
> 300kb of free disk space.

> Download

> The latest version of SecurePE can always be downloaded from this site.
> The current version is: Version 1.6
> Download: SecurePE v1.6
> MD5 : 01b7015f580a523e90b34613adf3493d
> SHA1: 357ddebb745a7d1434919bb8a266016967a0cb33

> SecurePE do not include ads, adware, spyware or any other malware.

> Changelog

> Version 1.6
> -Added: Support for x64 based systems.
> -Added: Windows Vista UAC compliant.
> -Added: Show/Hide password while typing.
> -Improved: Some minor tweaks.
> Notice: SecurePE v1.6 is no longer compatible with old versions so you need to deprotect your files, desinstall old version, install 1.6 and protect them again.

> Version 1.5
> -First stable version.

> Versions 1.0b to 1.4b
> -Initial beta releases.

> License

> This software is freeware for non-commercial use, you may copy and distribute it in its entirety as long as no fee is charged. However, if you want to support the development of this software, you are welcome to make a donation.
> If you plan to use this software commercially, you must request permission via e-mail.

> Disclaimer

> This software is provided "AS IS" and any express or implied warranties, including, but not limited to, the implied warranties of merchantability and fitness for a particular purpose are disclaimed. The author will not be held responsible for any losses or damages incurred, either directly or indirectly, by the use or misuse of this program. Use this program at your own risk.

> Feedback

> If you have any questions, suggestions or bug reports concerning SecurePE, please feel free to contact me.
> Copyright ? 2008 ThinkCore Research · All rights reserved · Contact

========================================================================================================

自己简单用VC6自动创建一个helloworld的WIN32程序来实验。

初始状态：

> `Microsoft Visual C++ 6.0`
> `oep`：00001350 `.text`

加SECUREPE16(仅加PASSWORD,暂不加CRC32)：

> 会把最后一个段.rsrc更名为SEPE，
> 并会修改前面各段数据（其所称的RC4加密？）而且会设置那些原来只有只读和执行的区段属性新增一个可写。
> Unidentified Packer. \*
> `oep`：0000A000 `.SEPE`

拿出OD1.1+STRONGOD调试：

一开始是这样的：

```
0040A000 > /EB 02                  JMP SHORT 0.0040A004

0040A002   |C606 E8                MOV BYTE PTR DS:[ESI],0E8

0040A005    0000                   ADD BYTE PTR DS:[EAX],AL

0040A007    0000                   ADD BYTE PTR DS:[EAX],AL

0040A009    5D                     POP EBP                                  ; kernel32.7C816FD7

0040A00A    81ED D3314000          SUB EBP,0.004031D3

0040A010    8D9D E2314000          LEA EBX,DWORD PTR SS:[EBP+4031E2]

0040A016    53                     PUSH EBX

0040A017    C3                     RETN

0040A018    8DB5 37324000          LEA ESI,DWORD PTR SS:[EBP+403237]

0040A01E    8BFE                   MOV EDI,ESI                              ; ntdll.7C910738

0040A020    BB E2314000            MOV EBX,0.004031E2

0040A025    B9 C3010000            MOV ECX,1C3

0040A02A    AD                     LODS DWORD PTR DS:[ESI]

0040A02B    2BC3                   SUB EAX,EBX

0040A02D    C1C0 03                ROL EAX,3

0040A030    33C3                   XOR EAX,EBX

0040A032    AB                     STOS DWORD PTR ES:[EDI]

0040A033    43                     INC EBX

0040A034    81FB 0B324000          CMP EBX,0.0040320B

0040A03A    75 05                  JNZ SHORT 0.0040A041

0040A03C    BB E2314000            MOV EBX,0.004031E2

0040A041  ^ E2 E7                  LOOPD SHORT 0.0040A02A

0040A043    89AD 92334000          MOV DWORD PTR SS:[EBP+403392],EBP

0040A049    89AD C6364000          MOV DWORD PTR SS:[EBP+4036C6],EBP

0040A04F    89AD D9364000          MOV DWORD PTR SS:[EBP+4036D9],EBP

0040A055    8D85 37324000          LEA EAX,DWORD PTR SS:[EBP+403237]

0040A05B    50                     PUSH EAX

0040A05C    C3                     RETN
//EAX中就是壳自身解码完后要跳转的地址，这里用RETN到达。
```

F7慢慢走，程序会自己sfx，然后来到

```
0040A118    55              PUSH EBP

0040A119    68 00040000     PUSH 400

0040A11E    52              PUSH EDX                                 ; 1msgbox.0040A1C1
```

参照

`HWND CreateDialogIndirectParam(`

`    HINSTANCE  hInstance, // handle of application instance`

`    LPCDLGTEMPLATE  lpTemplate, // address of dialog box template`

`    HWND  hWndParent, // handle of owner window`

`    DLGPROC  lpDialogFunc, // address of dialog box procedure  `

`    LPARAM  lParamInit  // initialization value`

`   );`

这里是PASSWORD验证对话框的`lpDialogFunc`

```
0040A11F    6A 00           PUSH 0

0040A121    51              PUSH ECX                                 ; 1msgbox.0040A238

0040A122    50              PUSH EAX                                 ; 1msgbox.00400000

0040A123    FF95 52364000   CALL DWORD PTR SS:[EBP+403652]           ; user32.CreateDialogIndirectParamA
```

PASSWORD验证对话框的`lpDialogFunc`过程：
