
# 用 Ollydbg 破解 SWFBrowser 2.93 (7千字)

- 作者：CoolWolF[SCD]
- 破解时间：2001-1-10
- 破解工具：WIN2K环境下 `Ollydbg1.05a`、`W32dasm9.0汉化版`
- 作者主页：http://swifftools.com/stools
- 难度：中低
- 说明：一个极好的Flash工具，可以把SWF文件中的各种元素提取出来，闪客必备。

---

以下文字纯粹是供各位爱好逆向工程同好参考交流，请尊重软件作者的权益。

---

这个我记得在精华区好像已经有了，但还是想再贴一次用 `Ollydbg` 破解的过程，因为越来越发现 `Ollydbg` 的方便之处，有什么不对地方请各位老大指正。

执行程序，弹出注册菜单，按 Register 按钮，输入用户名：`CoolWolF[SCD]`，注册码：`650033`，程序提示 `The serial number is invalid` 之后进入主画面。

用 `W32DSM` 打开 `SWFBrowser.exe`（没有加壳），找到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A99C8(C)
|
:004A9A60 6A00 push 00000000
:004A9A62 668B0DA49A4A00          mov cx, word ptr [004A9AA4]
:004A9A69 B201 mov dl, 01
* Possible StringData Ref from Code Obj ->"The serial number is invalid."
            |
:004A9A6B B8409B4A00              mov eax, 004A9B40
:004A9A70 E823B6FAFF              call 00455098
:004A9A75 8BC3 mov eax, ebx
:004A9A77 E8E4FCF9FF              call 00449760

很明显是从004A99C8跳过来的,按Shift+F12 转到004A99C8:
:004A99BB 8B55F0 mov edx, dword ptr [ebp-10]
:004A99BE 8BC3 mov eax, ebx
:004A99C0 59 pop ecx
:004A99C1 E806FEFFFF              call 004A97CC
//问题之所在
:004A99C6 84C0 test al, al
:004A99C8 0F8492000000            je 004A9A60
//跳走就完蛋
:004A99CE 6A00 push 00000000
:004A99D0 668B0DA49A4A00          mov cx, word ptr [004A9AA4]
:004A99D7 B202 mov dl, 02
* Possible StringData Ref from Code Obj ->"Thank you for registering SWF Browser!"
            |
:004A99D9 B8B09A4A00              mov eax, 004A9AB0
:004A99DE E8B5B6FAFF              call 00455098
:004A99E3 B201 mov dl, 01
:004A99E5 A1F4E84400              mov eax, dword ptr [0044E8F4]
:004A99EA E87150FAFF              call 0044EA60
:004A99EF 8BF0 mov esi, eax
:004A99F1 BA01000080              mov edx, 80000001
:004A99F6 8BC6 mov eax, esi
:004A99F8 E83F51FAFF              call 0044EB3C
:004A99FD C6460C01                mov [esi+0C], 01
:004A9A01 B101 mov cl, 01
* Possible StringData Ref from Code Obj ->"Software\Grooveware Multimedia\SWF Browser\Registration"
            |
:004A9A03 BAE09A4A00              mov edx, 004A9AE0
:004A9A08 8BC6 mov eax, esi
:004A9A0A E87152FAFF              call 0044EC80
```

这样就看得比较清楚了，程序先比对用户名和注册码是否匹配，如果是则放入注册表的 `HKEY_CURRENT_USER\Software\Grooveware Multimedia\SWF Browser\Registration` 键，以后每次启动的时候进行检查。

上面的代码很好理解，`004A97CC` 肯定是一个关键 Call，那么我们现在可以打开 `Ollydbg`，加载 `SWFBrowser.exe` 运行。

在 `004A99C1` 处按 F2 下断，运行程序输入任意的用户名和注册码，按确定后程序被中断：

```
004A99C0  |. 59            POP ECX
004A99C1  |. E8 06FEFFFF    CALL SWFBrows.004A97CC //断在这里
004A99C6  |. 84C0          TEST AL,AL
```

按 F7 跟进，然后 F8 慢慢往下走：

```
004A97CC  $ 55            PUSH EBP
004A97CD  . 8BEC          MOV EBP,ESP
004A97CF  . 6A 00          PUSH 0
004A97D1  . 6A 00          PUSH 0
004A97D3  . 6A 00          PUSH 0
004A97D5  . 6A 00          PUSH 0
004A97D7  . 6A 00          PUSH 0
004A97D9  . 6A 00          PUSH 0
004A97DB  . 6A 00          PUSH 0
004A97DD  . 53            PUSH EBX
004A97DE  . 56            PUSH ESI
004A97DF  . 57            PUSH EDI
004A97E0  . 894D F8        MOV DWORD PTR SS:[EBP-8],ECX
004A97E3  . 8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
004A97E6  . 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
//用户名到EAX
004A97E9  . E8 1AA8F5FF    CALL SWFBrows.00404008 //检查用户名长度的合法性
004A97EE  . 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
//假注册码到EAX
004A97F1  . E8 12A8F5FF    CALL SWFBrows.00404008 //检查注册码长度的合法性
004A97F6  . 33C0          XOR EAX,EAX
004A97F8  . 55            PUSH EBP
004A97F9  . 68 F5984A00    PUSH SWFBrows.004A98F5
004A97FE  . 64:FF30        PUSH DWORD PTR FS:[EAX]
004A9801  . 64:8920        MOV DWORD PTR FS:[EAX],ESP
004A9804  . 33C0          XOR EAX,EAX
004A9806  . 55            PUSH EBP
004A9807  . 68 C6984A00    PUSH SWFBrows.004A98C6
004A980C  . 64:FF30        PUSH DWORD PTR FS:[EAX]
004A980F  . 64:8920        MOV DWORD PTR FS:[EAX],ESP
004A9812  . 33C9          XOR ECX,ECX
004A9814  . B2 01          MOV DL,1
004A9816  . A1 20874A00    MOV EAX,DWORD PTR DS:[4A8720]
004A981B  . E8 4CFCFFFF    CALL SWFBrows.004A946C
004A9820  . 8BD8          MOV EBX,EAX
004A9822  . 33D2          XOR EDX,EDX
004A9824  . 8BC3          MOV EAX,EBX
004A9826  . E8 79F4FFFF    CALL SWFBrows.004A8CA4
004A982B  . 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
004A982E  . BA 10994A00    MOV EDX,SWFBrows.004A9910
            ;  ASCII "1232hfbsdjdh2834121"
//程序算法的密匙
004A9833  . E8 34A4F5FF    CALL SWFBrows.00403C6C
004A9838  . 8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]
004A983B  . 8BC3          MOV EAX,EBX
004A983D  . E8 B6F1FFFF    CALL SWFBrows.004A89F8
004A9842  . 8D4D F0        LEA ECX,DWORD PTR SS:[EBP-10]
004A9845  . 8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004A9848  . 8BC3          MOV EAX,EBX
004A984A  . E8 F5F2FFFF    CALL SWFBrows.004A8B44
004A984F  . BA 2C994A00    MOV EDX,SWFBrows.004A992C
            ;  ASCII "ewrwk214134g7df2" //同上
004A9854  . 8BC3          MOV EAX,EBX
004A9856  . E8 9DF1FFFF    CALL SWFBrows.004A89F8
004A985B  . 8D4D EC        LEA ECX,DWORD PTR SS:[EBP-14]
004A985E  . 8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]
004A9861  . 8BC3          MOV EAX,EBX
004A9863  . E8 DCF2FFFF    CALL SWFBrows.004A8B44
004A9868  . C745 E8 EFFFFF>MOV DWORD PTR SS:[EBP-18],-11
004A986F  . 8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
004A9872  . E8 DDA5F5FF    CALL SWFBrows.00403E54
004A9877  . 85C0          TEST EAX,EAX
004A9879  . 7E 1A          JLE SHORT SWFBrows.004A9895
004A987B  . 8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
004A987E  . E8 D1A5F5FF    CALL SWFBrows.00403E54
004A9883  . 50            PUSH EAX
004A9884  . 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
004A9887  . E8 98A7F5FF    CALL SWFBrows.00404024
004A988C  . 8D4D E8        LEA ECX,DWORD PTR SS:[EBP-18]
004A988F  . 5A            POP EDX
004A9890  . E8 83FCFFFF    CALL SWFBrows.004A9518
004A9895  > 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]
004A9898  . 33D2          XOR EDX,EDX
004A989A  . 52            PUSH EDX
004A989B  . 50            PUSH EAX
004A989C  . 8D55 E4        LEA EDX,DWORD PTR SS:[EBP-1C]
004A989F  . B8 20000000    MOV EAX,20
004A98A4  . E8 9FF8F5FF    CALL SWFBrows.00409148
004A98A9  . 8B55 E4        MOV EDX,DWORD PTR SS:[EBP-1C]
//真注册码入EDX 这里你可以看见正确的注册码
004A98AC  . 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
//假注册码入EAX 这里可以看见你输入的注册码
004A98AF  . E8 B0A6F5FF    CALL SWFBrows.00403F64 // 真假注册码比对
(这种比较方法算是比较常见的那种了)
004A98B4  . 75 04          JNZ SHORT SWFBrows.004A98BA
004A98B6  . B3 01          MOV BL,1
004A98B8  . EB 02          JMP SHORT SWFBrows.004A98BC
```

整理：用户名 `CoolWolF[SCD]`，注册码 `DD5C4800`。

如果想再来一次，就删除注册表的 `HKEY_CURRENT_USER\Software\Grooveware Multimedia\SWF Browser\Registration` 键。

完

---

哪位老大能贴一下它的注册机？我对 BlowFish 算法实在头疼。
