
# 联众斗地主记牌器 V2.36 注册算法分析

日期：2005年8月25日　　破解人:Baby2008

---

- **『软件名称』**：联众斗地主记牌器 V2.36
- **『软件大小』**：642 KB
- **『下载地址』**：10bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4C8P5h3y4F1i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0p5@1x3K6k6Q4x3X3g2Z5N6r3#2D9
- **『软件介绍』**：可用于联众一副牌和两副牌的斗地主游戏，具有自动记录已出牌、剩余牌和剩余张数的功能，用户注册后不是地主也能看底牌（二副牌时），软件界面美观，使用方便。
- **『保护方式』**：注册码保护
- **『破解声明』**：初学Crack，只是感兴趣，失误之处敬请诸位大侠赐教!
- **『破解工具』**：flyODBG.V1.10 聆风听雨汉化第二版、PeID 0.93,ASPackDie v1.41.HH

『破解过程』：

## 一、查壳、脱壳 + 去除反调试

PeID 0.93,查壳,ASPack 2.12 -> Alexey Solodovnikov,老壳了,工具手脱都很方便,我是懒人,用ASPackDie v1.41.HH搞定,默认另存为`Unpacked.eXe`,OD载入,F9运行,晕死,flyODBG被自动关闭了,有Anti-Bebug,搞定它,重新运行OD,命令行下断点 `bp TerminateProcess`,F9运行,OD中断在:

```asm
7C801E16 k>  8BFF                 mov edi,edi                                 ; Unpacked.0045A7E0
7C801E18     55                   push ebp
7C801E19     8BEC                 mov ebp,esp
7C801E1B     837D 08 00           cmp dword ptr ss:[ebp+8],0
7C801E1F     75 09                jnz short kernel32.7C801E2A
```

堆栈提示:

```asm
0012FDE4     0045ABA0  /CALL 到 TerminateProcess 来自 Unpacked.0045AB9B //来源
0012FDE8     000000A4  |hProcess = 000000A4 (window)
0012FDEC     00000000  \ExitCode = 0
```

`Ctrl+G:0045AB9B`

```asm
0045AB7D    .  BA D8AB4500        mov edx,Unpacked.0045ABD8                   ;  ASCII "EXPLORER.EXE"
0045AB82    .  E8 219CFAFF        call Unpacked.004047A8
0045AB87    .  74 1D              je short Unpacked.0045ABA6                  ;  修改为JMP,解除Anti-Debug
0045AB89    .  56                 push esi                                    ; /ProcessId
0045AB8A    .  6A 00              push 0                                      ; |Inheritable = FALSE
0045AB8C    .  68 FF0F1F00        push 1F0FFF                                 ; |Access = PROCESS\_ALL\_ACCESS
0045AB91    .  E8 86BEFAFF        call <jmp.&kernel32.OpenProcess>            ; \OpenProcess
0045AB96    .  8BD8               mov ebx,eax
0045AB98    .  6A 00              push 0                                      ; /ExitCode = 0
0045AB9A    .  53                 push ebx                                    ; |hProcess
0045AB9B    .  E8 CCBEFAFF        call <jmp.&kernel32.TerminateProcess>       ; \TerminateProcess    这里
0045ABA0    .  53                 push ebx                                    ; /hObject
0045ABA1    .  E8 16BDFAFF        call <jmp.&kernel32.CloseHandle>            ; \CloseHandle
```

原来是查找父进程啊,修改为`JMP`或将`flyODBG.exe`文件名改为`EXPLORER.EXE`,即可解除Anti-Debug;

## 二、注册验证算法分析

脱壳后查得是Borland Delphi 6.0 - 7.0，好办，Dede出马找到注册按钮事件地址`0045A570`下断，F9运行,输入试炼码`1234567890`，点击注册，OD中断在:

```asm
0045A570 <>/.  55                 push ebp                                    ;  <-TForm2@BRegClick
0045A571   |.  8BEC               mov ebp,esp
0045A573   |.  6A 00              push 0
0045A575   |.  53                 push ebx
0045A576   |.  8BD8               mov ebx,eax
0045A578   |.  33C0               xor eax,eax
0045A57A   |.  55                 push ebp
0045A57B   |.  68 06A64500        push <Unpacked.->System.@HandleFinally;>
0045A580   |.  64:FF30            push dword ptr fs:[eax]
0045A583   |.  64:8920            mov dword ptr fs:[eax],esp
0045A586 <>|.  8B83 04030000      mov eax,dword ptr ds:[ebx+304]              ;  \*RCode:N.A.
0045A58C   |.  8B10               mov edx,dword ptr ds:[eax]
0045A58E   |.  FF52 50            call dword ptr ds:[edx+50]
0045A591   |.  84C0               test al,al
0045A593   |.  74 5B              je short Unpacked.0045A5F0
0045A595   |.  8D55 FC            lea edx,dword ptr ss:[ebp-4]
0045A598 <>|.  8B83 04030000      mov eax,dword ptr ds:[ebx+304]              ;  \*RCode:N.A.
0045A59E <>|.  E8 658CFDFF        call Unpacked.00433208                      ;  ->Controls.TControl.GetText(TControl):TCaption;
0045A5A3   |.  8B55 FC            mov edx,dword ptr ss:[ebp-4]
0045A5A6   |.  B8 1CA64500        mov eax,Unpacked.0045A61C                   ;  ASCII "RegCode"
0045A5AB   |.  E8 7CB6FFFF        call Unpacked.00455C2C                      ;  保存注册信息
0045A5B0   |.  84C0               test al,al
0045A5B2   |.  74 3C              je short Unpacked.0045A5F0
0045A5B4   |.  6A 00              push 0                                      ; /Title = NULL
0045A5B6   |.  68 24A64500        push Unpacked.0045A624                      ; |Class = "LZDDZHookWnd2003"
0045A5BB <>|.  E8 14C8FAFF        call <jmp.&user32.FindWindowA>              ; \->user32.FindWindowA()
0045A5C0   |.  85C0               test eax,eax
0045A5C2   |.  74 11              je short Unpacked.0045A5D5
0045A5C4   |.  6A 00              push 0                                      ; /lParam = 0
0045A5C6   |.  6A 00              push 0                                      ; |wParam = 0
0045A5C8   |.  68 02800000        push 8002                                   ; |Message = MSG(8002)
0045A5CD   |.  50                 push eax                                    ; |hWnd
0045A5CE <>|.  E8 49CAFAFF        call <jmp.&user32.PostMessageA>             ; \->user32.PostMessageA()
0045A5D3   |.  EB 1B              jmp short Unpacked.0045A5F0
0045A5D5   |>  6A 01              push 1
0045A5D7   |.  6A 03              push 3
0045A5D9   |.  68 02800000        push 8002
0045A5DE   |.  A1 48D04500        mov eax,dword ptr ds:[45D048]
0045A5E3   |.  8B00               mov eax,dword ptr ds:[eax]
0045A5E5 <>|.  E8 92F2FDFF        call Unpacked.0043987C                      ;  ->Controls.TWinControl.GetHandle(TWinControl):HWND;<+>
0045A5EA   |.  50                 push eax                                    ; |hWnd
0045A5EB <>|.  E8 2CCAFAFF        call <jmp.&user32.PostMessageA>             ; \->user32.PostMessageA()
0045A5F0   |>  33C0               xor eax,eax
0045A5F2   |.  5A                 pop edx
0045A5F3   |.  59                 pop ecx
0045A5F4   |.  59                 pop ecx
0045A5F5   |.  64:8910            mov dword ptr fs:[eax],edx
0045A5F8   |.  68 0DA64500        push Unpacked.0045A60D
0045A5FD   |>  8D45 FC            lea eax,dword ptr ss:[ebp-4]
0045A600 <>|.  E8 A79DFAFF        call Unpacked.004043AC                      ;  ->System.@LStrClr(void;void);
0045A605   \.  C3                 retn
0045A606 <> .^ E9 C997FAFF        jmp Unpacked.00403DD4                       ;  ->System.@HandleFinally;
0045A60B    .^ EB F0              jmp short Unpacked.0045A5FD
0045A60D    .  5B                 pop ebx
0045A60E    .  59                 pop ecx
0045A60F    .  5D                 pop ebp
0045A610    .  C3                 retn
```

晕，注册按钮事件并没有以往的注册验证过程，`PostMessageA`发送消息走人了。Delphi写的程序，如果用消息来传递数据，会给Cracker带来很大的麻烦。因为Delphi的消息处理机制嵌套太多了，另外想想办法吧；

突然发现软件有“软件已经注册”字样，哈哈不错，就是它了，插件查找字符串位于`0045A6C4`，向上查找来到；

```asm
0045A65D <>|.  E8 BAB4FFFF        call Unpacked.00455B1C                      ;  ->:TPassword.\_PROC\_00455B1C()
0045A662   |.  8B55 FC            mov edx,dword ptr ss:[ebp-4]
0045A665 <>|.  8B83 00030000      mov eax,dword ptr ds:[ebx+300]              ;  \*MCode:N.A.
0045A66B <>|.  E8 C88BFDFF        call Unpacked.00433238                      ;  ->Controls.TControl.SetText(TControl;TCaption);
0045A670   |.  8D55 F8            lea edx,dword ptr ss:[ebp-8]
0045A673   |.  B8 08A74500        mov eax,Unpacked.0045A708                   ;  ASCII "RegCode"
0045A678 <>|.  E8 DFB4FFFF        call Unpacked.00455B5C                      ;  ->:TPassword.\_PROC\_00455B5C()
0045A67D   |.  8B55 F8            mov edx,dword ptr ss:[ebp-8]
0045A680 <>|.  8B83 04030000      mov eax,dword ptr ds:[ebx+304]              ;  \*RCode:N.A.
0045A686 <>|.  E8 AD8BFDFF        call Unpacked.00433238                      ;  ->Controls.TControl.SetText(TControl;TCaption);
0045A68B   |.  8D55 F4            lea edx,dword ptr ss:[ebp-C]
0045A68E <>|.  8B83 04030000      mov eax,dword ptr ds:[ebx+304]              ;  \*RCode:N.A.
```
