

# DebugView.exe 美化全过程

> 原文：[`www.pediy.com/kssd/pediy10/95454.html`](https://www.pediy.com/kssd/pediy10/95454.html)

相信写过驱动的朋友应该用过 debugview.exe 这个工具吧  
你会不会觉得它的界面有点“丑”呢，还有就是观察调试信息的时候窗口要不停的 ALT+TAB 切换来切换去  
今天心血来潮，决定将它改造一下  
1，让它有个窗口置顶功能  
    这个很容易实现：  
    SetWindowPos(wnd,HWND\_TOPMOST,0,0,0,0,SWP\_NOMOVE|SWP\_NOSIZE);  
      就 ok 了，我们的关键点就是要获取它的窗口句柄  
2，给它加个“换肤”的功能  
    这里我用到一个别人开发好的 dll.. ....SkinMagic.dll  
      SkinMagic Toolkit 是 vc 专用的一个皮肤开发包，有兴趣的朋友可以下下来玩玩。  
    下面介绍一下它在 vc 里面针对 SDI 程序的用法  
     (1)包含头文件 SkinMagic.h,引入库文件 SkinMagic.lib  
        (2)在 CWinAPP::Instance()里初始化库，这里用到的函数是  
             InitSkinMagicLib(hInstance,0,0,0);和  
             SetWindowSkin(wnd,"MainFrame");和  
             SetDialogSkin("Dialog");  
        (3)换肤时调用 LoadSkinFile(SkinFileName);进行换肤  
好了，流程都知道了，下面就来动手把  
我准备在主菜单里面在添加一个子菜单“附加功能”,再在这个子菜单里面添加换肤和窗口置顶几个子菜单  
首先我们来修改 DebugView.exe，给它的引入库添加我们要用的几个函数  
这里我用到的是 Stud\_PE，它里面有个添加引入函数的功能，这里我就不细述了，接着我们把 skin.dll 和 DebugView.exe 放在同一个目录下(我把 SkinMagic.dll 改名为了 skin.dll),再把几个皮肤文件也复制过来  
下面我们下修改让它初始化库文件  
用 od 载入 DebugView,在他的地址空间里我们要找一个全 0 的空间,用来添加 dll 的初始化代码,我找到的是 0x419531 这个地址,然后我们填入下面这段代码  

代码:

```
00419531 > $  6A 00         push 0                                   ; /pModule = NULL
00419533   .  E8 09223F7C   call kernel32.GetModuleHandleA           ; \\获取 Instance
00419538   .  6A 00         push 0
0041953A   .  E8 15CBBE0F   call skin.InitSkinMagicLib               ;  调用初始化函数
0041953F   .  68 20CF4100   push Dbgview.0041CF20                    ;  ASCII "1.skin"
00419544   .  E8 A1CFBE0F   call skin.LoadSkinFile                   ;  加载一个默认的皮肤
00419549   .  68 30CF4100   push Dbgview.0041CF30                    ;  ASCII "Dialog"
0041954E   .  E8 81D1BE0F   call skin.SetDialogSkin                  ;  调用 SetDialogSkin
00419553   .^ E9 F48BFFFF   jmp Dbgview.0041214C                     ;  跳到原 oep

```

上面的"1.skin"和"Dialog"两个字符串是我直接在数据段添加进去的  
我们用 od 保存 exe 文件,然后用 LOADPE 将 DebugView 的 oep 该我我们的新 oep:0x19531  
运行,发现皮肤没有改变..  
呵呵.因为我们该有一个初始化函数没有调用:SetWindowSkin(wnd,"MainFrame");  
因为这个函数需要窗口句柄作为参数,所以我们要在主程序窗口建立后在跳过来调用这个函数,再跳回去  
好的,F2 重新加载程序,eip 停在了我们新的 oep 上  
bp CreateWindowExA......F9.断下两次后,停在这里  
代码:

```
0012FCB4   00408621  /CALL 到 CreateWindowExA 来自 Dbgview.0040861B
0012FCB8   00000000  |ExtStyle = 0
0012FCBC   0041E9F8  |Class = "dbgviewClass"
0012FCC0   00423948  |WindowName = ""
0012FCC4   00CF0000  |Style = WS\_OVERLAPPED|WS\_MINIMIZEBOX|WS\_MAXIMIZEBOX|WS\_SYSMENU|WS\_THICKFRAME|WS\_CAPTION
0012FCC8   000000A6  |X = A6 (166.)
0012FCCC   00000136  |Y = 136 (310.)
0012FCD0   00000379  |Width = 379 (889.)
0012FCD4   00000174  |Height = 174 (372.)
0012FCD8   00000000  |hParent = NULL
0012FCDC   00000000  |hMenu = NULL
0012FCE0   00400000  |hInst = 00400000
0012FCE4   00000000  \\lParam = NULL

```

我们在代码段跟随\[esp\],来到:  
00408614   .  68 F8E94100   push Dbgview.0041E9F8                    ; |Class = "dbgviewClass"  
00408619   .  6A 00         push 0                                   ; |ExtStyle = 0  
0040861B   .  FF15 A4A34100 call dword ptr ds:\[<&USER32.CreateWindow>; \\CreateWindowExA  
00408621   .  85C0          test eax,eax  
00408621 出的 eax 就是我们需要的主窗口句柄了,我们修改它的下一条指令为 jmp 00419559  
00419559 同样是一个 0 区间,接下来我们在 00419559 处写入如下代码:  
代码:

```
00419559   > \\A3 04DE4200   mov dword ptr ds:\[42DE04\],eax            ;  这个是被我们覆盖掉的指令,这里要补上,还有记下这个 0x42de04,后面会用到
0041955E   .  83EC 0A       sub esp,0A
00419561   .  C60424 4D     mov byte ptr ss:\[esp\],4D                 ;  M
00419565   .  C64424 01 61  mov byte ptr ss:\[esp+1\],61               ;  a
0041956A   .  C64424 02 69  mov byte ptr ss:\[esp+2\],69               ;  i
0041956F   .  C64424 03 6E  mov byte ptr ss:\[esp+3\],6E               ;  n
00419574   .  C64424 04 46  mov byte ptr ss:\[esp+4\],46               ;  F
00419579   .  C64424 05 72  mov byte ptr ss:\[esp+5\],72               ;  r
0041957E   .  C64424 06 61  mov byte ptr ss:\[esp+6\],61               ;  a
00419583   .  C64424 07 6D  mov byte ptr ss:\[esp+7\],6D               ;  m
00419588   .  C64424 08 65  mov byte ptr ss:\[esp+8\],65               ;  e
0041958D   .  C64424 09 00  mov byte ptr ss:\[esp+9\],0                ;  0
00419592   .  54            push esp                                 ;  MainFrame
00419593   .  50            push eax                                 ;  wnd
00419594   .  E8 79D0BE0F   call skin.SetWindowSkin                  ;  调用初始化
00419599   .  83C4 0A       add esp,0A                               ;  堆栈平衡
0041959C   .^ E9 8AF0FEFF   jmp Dbgview.0040862B                     ;  跳回去

```

好了,现在我们保存为 exe 文件就可以看到皮肤改变了,效果图如下:  
![](img/d2f26a798ad4d056055dd4453ef4dbe8.jpg)  
接下来,我们添加菜单如下:  
![](img/fca5816e3ffb4338120d2b1633aadd46.jpg)  
下面我们的任务就是添加事件的响应了  
把断点都清掉,然后 bp DialogBoxParamA,f9 运行,接着点击 debugview 的 help->about->成功断下  
我们在代码段跟随\[esp\],来到这里:  
00406278   . /0F8F 410C0000 jg Dbgview.00406EBF  
0040627E   . |0F84 DF0B0000 je Dbgview.00406E63  
00406284   . |3D 589C0000   cmp eax,9C58  
00406289   . |0F8F 21050000 jg Dbgview.004067B0  
0040628F   . |0F84 D8040000 je Dbgview.0040676D  
00406295   . |3D 519C0000   cmp eax,9C51  
0040629A   . |0F8F B6010000 jg Dbgview.00406456  
004062A0   . |0F84 F4000000 je Dbgview.0040639A  
004062A6   . |3D 479C0000   cmp eax,9C47  
004062AB   . |0F8F A1000000 jg Dbgview.00406352  
004062B1   . |74 63         je short Dbgview.00406316  
004062B3   . |83F8 68       cmp eax,68  
004062B6   . |74 3B         je short Dbgview.004062F3  
004062B8   . |3D 2C010000   cmp eax,12C  
004062BD   . |0F85 DE320100 jnz Dbgview.004195A1  
004062C3   . |8B8C24 740400>mov ecx,dword ptr ss:\[esp+474\]           ;  Case 12C of switch 00406273  
004062CA   . |8B15 70134300 mov edx,dword ptr ds:\[431370\]            ;  Dbgview.00400000  
004062D0   . |6A 00         push 0                                   ; /lParam = NULL  
004062D2   . |68 50154000   push Dbgview.00401550                    ; |DlgProc = Dbgview.00401550  
004062D7   . |51            push ecx                                 ; |hOwner  
004062D8   . |68 DCE34100   push Dbgview.0041E3DC                    ; |pTemplate = "AboutBox"  
004062DD   . |52            push edx                                 ; |hInst => 00400000  
004062DE   . |FF15 00A44100 call dword ptr ds:\[<&USER32.DialogBoxPar>; \\DialogBoxParamA  
004062E4   . |5F            pop edi  
004062E5   . |5E            pop esi  
004062E6   . |5D            pop ebp  
004062E7   . |33C0          xor eax,eax  
上面的几个 je 就是对菜单项的 id 的判断,很明显 id 存放在 eax 中  
我们修改 0x4062bd 处的跳转地址,跳到一个全 0 的区间,这里我选得地址是 0x004195A1  
接下来我们再 0x004195A1 出写入如下代码:  
代码:

```
004195A1   > \\60            pushad                                   ;  保存寄存器
004195A2   .  2D 80380000   sub eax,3880                             ;  我定义的几个菜单的 id 是 3880,3881,3882,3883,,,,,,
004195A7   .  74 2B         je short Dbgview.004195D4                ;  判断
004195A9   .  48            dec eax
004195AA   .  74 3F         je short Dbgview.004195EB
004195AC   .  48            dec eax
004195AD   .  74 50         je short Dbgview.004195FF
004195AF   .  48            dec eax
004195B0   .  74 61         je short Dbgview.00419613
004195B2   .  48            dec eax
004195B3   .  74 72         je short Dbgview.00419627
004195B5   .  48            dec eax
004195B6   .  0F84 7F000000 je Dbgview.0041963B
004195BC   .  48            dec eax
004195BD   .  0F84 95000000 je Dbgview.00419658
004195C3   .  61            popad
004195C4   .^ E9 26E0FEFF   jmp Dbgview.004075EF                     ;  跳到原处继续处理
004195C9      90            nop
004195CA      90            nop
004195CB      90            nop
004195CC      90            nop                                      ;  这些 nop 是我修改代码时留下的,,没用
004195CD      90            nop
004195CE      90            nop
004195CF      90            nop
004195D0      90            nop
004195D1      90            nop
004195D2      90            nop
004195D3      90            nop
004195D4   >  68 696E0000   push 6E69                                ;  1; Case 3880 of switch 00406273
004195D9   .  68 312E736B   push 6B732E31                            ;  压入 1.skin
004195DE   .  54            push esp
004195DF   .  E8 06CFBE0F   call skin.LoadSkinFile
004195E4   .  58            pop eax
004195E5   .  58            pop eax
004195E6   .  E9 81000000   jmp Dbgview.0041966C
004195EB   >  68 696E0000   push 6E69                                ;  2; Case 3881 of switch 00406273
004195F0   .  68 322E736B   push 6B732E32                            ;  压入 2.skin
004195F5   .  54            push esp
004195F6   .  E8 EFCEBE0F   call skin.LoadSkinFile
004195FB   .  58            pop eax
004195FC   .  58            pop eax
004195FD   .  EB 6D         jmp short Dbgview.0041966C
004195FF   >  68 696E0000   push 6E69                                ;  3; Case 3882 of switch 00406273
00419604   .  68 332E736B   push 6B732E33                            ;  压入 3.skin
00419609   .  54            push esp
0041960A   .  E8 DBCEBE0F   call skin.LoadSkinFile
0041960F   .  58            pop eax
00419610   .  58            pop eax
00419611   .  EB 59         jmp short Dbgview.0041966C
00419613   >  68 696E0000   push 6E69                                ;  4; Case 3883 of switch 00406273
00419618   .  68 342E736B   push 6B732E34                            ;  压入 4.skin
0041961D   .  54            push esp
0041961E   .  E8 C7CEBE0F   call skin.LoadSkinFile
00419623   .  58            pop eax
00419624   .  58            pop eax
00419625   .  EB 45         jmp short Dbgview.0041966C
00419627   >  68 696E0000   push 6E69                                ;  5; Case 3884 of switch 00406273
0041962C   .  68 352E736B   push 6B732E35                            ;  压入 5.skin
00419631   .  54            push esp
00419632   .  E8 B3CEBE0F   call skin.LoadSkinFile
00419637   .  58            pop eax
00419638   .  58            pop eax
00419639   .  EB 31         jmp short Dbgview.0041966C
0041963B   >  6A 03         push 3                                   ; /6; Case 3885 of switch 00406273
0041963D   .  6A 00         push 0                                   ; |Height = 0
0041963F   .  6A 00         push 0                                   ; |Width = 0
00419641   .  6A 00         push 0                                   ; |Y = 0
00419643   .  6A 00         push 0                                   ; |X = 0
00419645   .  6A FF         push -1                                  ; |InsertAfter = HWND\_TOPMOST
00419647   .  90            nop                                      ; |
00419648   .  90            nop                                      ; |
00419649   .  90            nop                                      ; |
0041964A   .  A1 04DE4200   mov eax,dword ptr ds:\[42DE04\]            ; |
0041964F   .  90            nop                                      ; |
00419650   .  50            push eax                                 ; |hWnd => 00270756 ('DebugView - Not Connected',class='dbgviewClass')
00419651   .  E8 9D039177   call user32.SetWindowPos                 ; \\SetWindowPos
00419656   .  EB 14         jmp short Dbgview.0041966C
00419658   >  6A 05         push 5                                   ; /7; Case 3886 of switch 00406273
0041965A   .  6A 00         push 0                                   ; |DefDir = NULL
0041965C   .  6A 00         push 0                                   ; |Parameters = NULL
0041965E   .  68 40CF4100   push Dbgview.0041CF40                    ; |FileName = "http://315712565.qzone.qq.co5.qzone.qq.com"
00419663   .  6A 00         push 0                                   ; |Operation = NULL
00419665   .  6A 00         push 0                                   ; |hWnd = NULL
00419667   .  E8 7C7B1F7D   call shell32.ShellExecuteA               ; \\ShellExecuteA
0041966C   >  61            popad
0041966D   .  5F            pop edi
0041966E   .  5E            pop esi
0041966F   .  5D            pop ebp
00419670   .  33C0          xor eax,eax
00419672   .  5B            pop ebx
00419673   .  81C4 60040000 add esp,460
00419679   .  C2 1000       retn 10                                  ;  返回

```

最后用 od 保存为 exe 文件就 ok 了