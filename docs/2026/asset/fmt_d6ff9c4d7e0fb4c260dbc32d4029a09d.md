
```asm
004011AF  |.  FF05 94344000 |INC DWORD PTR DS:[403494]

004011B5  |.  A1 94344000   |MOV EAX,DWORD PTR DS:[403494]

004011BA  |.  3B05 8C344000 |CMP EAX,DWORD PTR DS:[40348C]           ;  DS：[0040348C]=09

004011C0  |.  73 02         |JNB SHORT 扫雷外挂.004011C4

004011C2  |.^ EB 81         \JMP SHORT 扫雷外挂.00401145

004011C4  |>  68 60334000   PUSH 扫雷外挂.00403360                       ; /Format = ".....程序结束，回车键退出"

004011C9  |.  68 80334000   PUSH 扫雷外挂.00403380                       ; |s = 扫雷外挂.00403380

004011CE  |.  E8 23000000   CALL <JMP.&user32.wsprintfA>             ; \wsprintfA

004011D3  |.  83C4 08       ADD ESP,8

004011D6  |.  68 80334000   PUSH 扫雷外挂.00403380                       ; /Arg1 = 00403380

004011DB  |.  E8 40000000   CALL 扫雷外挂.00401220                       ; \扫雷外挂.00401220

004011E0  |.  68 04010000   PUSH 104                                 ; /Arg2 = 00000104

004011E5  |.  68 80334000   PUSH 扫雷外挂.00403380                       ; |Arg1 = 00403380

004011EA  |.  E8 69000000   CALL 扫雷外挂.00401258                       ; \扫雷外挂.00401258

004011EF  |.  6A 00         PUSH 0                                   ; /ExitCode = 0

004011F1  \.  E8 12000000   CALL <JMP.&kernel32.ExitProcess>         ; \ExitProcess
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
