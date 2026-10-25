
# CCproxy6.0的算法分析

【破文标题】 CCproxy6.0的算法分析

【破文作者】 daxia2002

【作者邮箱】 daxia2002@126.com

【使用工具】 Peid,Ollydbg

【破解平台】 Win2000/XP

【软件名称】 CCproxy6.0

【下载地址】 <http://www.youngzsoft.com/download/ccproxysetup.exe>

【软件大小】815KB

【保护方式】无

【破解目的】研究算法分析

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

用`Peid`查壳，发现无壳，为Microsoft Visual C++ 6.0 编写。

用`OllyDbg`加载、运行，点击“注册”按钮弹出注册窗口。在注册窗口里，序列号栏输入：`daxia2002`，注册码栏输入:`88888888`，点击“注册”按钮，当然弹出“对不起，注册失败!”对话框。

切换到`OllyDbg`，按`F12`暂停，按`ALT+K`打开调用堆栈窗口，显示的信息如下：

调用堆栈

地址       堆栈       例程 / 参数                                                       调用来自

0012BC98   77D13C6B   包含 7FFE0304                                                     USER32.77D13C69               0012BCCC

0012BC9C   77D1B406   USER32.WaitMessage                                                USER32.77D1B401               0012BCCC

0012BCD0   77D1D9AA   USER32.77D1B279                                                   USER32.77D1D9A5               0012BCCC

0012BCF8   77D362F4   USER32.77D1D8F6                                                   USER32.77D362EF               0012BCF4

0012BFB0   77D35D77   ? USER32.SoftModalMessageBox                                      USER32.77D35D72               0012BF38

0012C0F8   77D36441   ? USER32.77D35C3D                                                 USER32.77D3643C               0012C080

0012C14C   77D36529   USER32.MessageBoxTimeoutW                                         USER32.77D36524               0012C148

0012C180   77D364B6   ? USER32.MessageBoxTimeoutA                                       USER32.77D364B1               0012C17C

0012C1A0   77D3649A   ? USER32.MessageBoxExA                                            USER32.77D36495               0012C19C

0012C1A4   000703E8     hOwner = 000703E8 ('注册',class='#32770',parent=000D03BA)

0012C1A8   00C57758     Text = "对不起，注册失败!"

0012C1AC   00A906A0     Title = "CCProxy"

0012C1B0   00000010     Style = MB_OK|MB_ICONHAND|MB_APPLMODAL

0012C1B4   00000000     LanguageID = 0 (LANG_NEUTRAL)

0012C1B8   00454F7F   ? USER32.MessageBoxA                                              CCProxy.00454F79

0012C1BC   000703E8     hOwner = 000703E8 ('注册',class='#32770',parent=000D03BA)

0012C1C0   00C57758     Text = "对不起，注册失败!"

0012C1C4   00A906A0     Title = "CCProxy"

0012C1C8   00000010     Style = MB_OK|MB_ICONHAND|MB_APPLMODAL

堆栈友好显示调用来自`CCProxy.00454F79`显示了注册失败的窗口，好的，双击这一行，跳转到地址`00454f79`:

```
00454EC7   /$  55                push ebp
00454EC8   |.  8BEC              mov ebp,esp
00454ECA   |.  81EC 14010000     sub esp,114
00454ED0   |.  53                push ebx
00454ED1   |.  56                push esi
00454ED2   |.  57                push edi
00454ED3   |.  33DB              xor ebx,ebx
00454ED5   |.  8BF9              mov edi,ecx
00454ED7   |.  53                push ebx
00454ED8   |.  897D F0           mov dword ptr ss:[ebp-10],edi
00454EDB   |.  E8 B9FFFFFF       call CCProxy.00454E99
00454EE0   |.  8D45 FC           lea eax,dword ptr ss:[ebp-4]
00454EE3   |.  50                push eax
00454EE4   |.  53                push ebx
00454EE5   |.  E8 55010000       call CCProxy.0045503F
00454EEA   |.  33F6              xor esi,esi
00454EEC   |.  3BC3              cmp eax,ebx
00454EEE   |.  8945 F4           mov dword ptr ss:[ebp-C],eax
00454EF1   |.  74 18             je short CCProxy.00454F0B
00454EF3   |.  53                push ebx                                    ; /lParam
00454EF4   |.  53                push ebx                                    ; |wParam
00454EF5   |.  68 76030000       push 376                                    ; |Message = MSG(376)
00454EFA   |.  FF75 FC           push dword ptr ss:[ebp-4]                   ; |hWnd
00454EFD   |.  FF15 BCE54500     call dword ptr ds:[<&USER32.SendMessageA>]  ; \SendMessageA
00454F03   |.  3BC3              cmp eax,ebx
00454F05   |.  74 04             je short CCProxy.00454F0B
00454F07   |.  8BF0              mov esi,eax
00454F09   |.  EB 0A             jmp short CCProxy.00454F15
00454F0B   |>  3BFB              cmp edi,ebx
00454F0D   |.  74 06             je short CCProxy.00454F15
00454F0F   |.  8DB7 9C000000     lea esi,dword ptr ds:[edi+9C]
00454F15   |>  3BF3              cmp esi,ebx
00454F17   |.  895D F8           mov dword ptr ss:[ebp-8],ebx
00454F1A   |.  74 13             je short CCProxy.00454F2F
00454F1C   |.  8B06              mov eax,dword ptr ds:[esi]
00454F1E   |.  8945 F8           mov dword ptr ss:[ebp-8],eax
00454F21   |.  8B45 10           mov eax,dword ptr ss:[ebp+10]
00454F24   |.  3BC3              cmp eax,ebx
00454F26   |.  74 07             je short CCProxy.00454F2F
00454F28   |.  05 00000300       add eax,30000
00454F2D   |.  8906              mov dword ptr ds:[esi],eax
00454F2F   |>  8B5D 0C           mov ebx,dword ptr ss:[ebp+C]
00454F32   |.  F6C3 F0           test bl,0F0
00454F35   |.  75 17             jnz short CCProxy.00454F4E
00454F37   |.  8BC3              mov eax,ebx
00454F39   |.  83E0 0F           and eax,0F
00454F3C   |.  83F8 01           cmp eax,1
00454F3F   |.  76 0A             jbe short CCProxy.00454F4B
00454F41   |.  83F8 02           cmp eax,2
00454F44   |.  76 08             jbe short CCProxy.00454F4E
00454F46   |.  83F8 04           cmp eax,4
00454F49   |.  77 03             ja short CCProxy.00454F4E
00454F4B   |>  83CB 30           or ebx,30
00454F4E   |>  85FF              test edi,edi
00454F50   |.  74 05             je short CCProxy.00454F57
00454F52   |.  8B7F 78           mov edi,dword ptr ds:[edi+78]
00454F55   |.  EB 1A             jmp short CCProxy.00454F71
00454F57   |>  8D85 ECFEFFFF     lea eax,dword ptr ss:[ebp-114]
00454F5D   |.  68 04010000       push 104                                    ; /BufSize = 104 (260.)
00454F62   |.  50                push eax                                    ; |PathBuffer
00454F63   |.  6A 00             push 0                                      ; |hModule = NULL
00454F65   |.  8DBD ECFEFFFF     lea edi,dword ptr ss:[ebp-114]              ; |
00454F6B   |.  FF15 18E24500     call dword ptr ds:[<&KERNEL32.GetModuleFile>; \GetModuleFileNameA
00454F71   |>  53                push ebx                                    ; /Style
00454F72   |.  57                push edi                                    ; |Title
00454F73   |.  FF75 08           push dword ptr ss:[ebp+8]                   ; |Text
00454F76   |.  FF75 F4           push dword ptr ss:[ebp-C]                   ; |hOwner
00454F79   |.  FF15 30E44500     call dword ptr ds:[<&USER32.MessageBoxA>]   ; \MessageBoxA        //跳到这里
00454F7F   |.  85F6              test esi,esi                                ;  CCProxy.00473E94
```

看了一下，这里并没有关键判断的地方。下断点`bp GetWindowTextA`,点击“注册”按钮，断了下来，`ALT+F9`返回到程序领空，返回到`00454C8C`，这里取注册码。

```
00454C57   /$  55                push ebp
00454C58   |.  8BEC              mov ebp,esp
00454C5A   |.  56                push esi
00454C5B   |.  57                push edi
00454C5C   |.  8B7D 08           mov edi,dword ptr ss:[ebp+8]
00454C5F   |.  FF75 0C           push dword ptr ss:[ebp+C]
00454C62   |.  8BCF              mov ecx,edi
00454C64   |.  E8 2FFEFFFF       call CCProxy.00454A98
00454C69   |.  833F 00           cmp dword ptr ds:[edi],0
00454C6C   |.  8BF0              mov esi,eax
00454C6E   |.  74 28             je short CCProxy.00454C98
00454C70   |.  56                push esi                                    ; /hWnd
00454C71   |.  FF15 44E44500     call dword ptr ds:[<&USER32.GetWindowTextLe>; \GetWindowTextLengthA
00454C77   |.  8D48 01           lea ecx,dword ptr ds:[eax+1]
00454C7A   |.  51                push ecx
00454C7B   |.  8B4D 10           mov ecx,dword ptr ss:[ebp+10]
00454C7E   |.  50                push eax
00454C7F   |.  E8 4DC6FFFF       call CCProxy.004512D1
00454C84   |.  50                push eax                                    ; |Buffer
00454C85   |.  56                push esi                                    ; |hWnd
```
