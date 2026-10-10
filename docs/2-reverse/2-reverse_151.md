

# 打造 Ollydbg

> 原文：[`www.pediy.com/kssd/pediy06/pediy6470.htm`](https://www.pediy.com/kssd/pediy06/pediy6470.htm)

Written by askformore ![](img/620448a77d884bd6b2c4a7cfef437501.jpg)   
  
上次上传了个英文版 OD1.10 的 DIY，感觉很粗糙，当时由于打包压缩时误删了笔记，所以只能上传成品附件，也没空重写。近来，有些时间就重来一次，把功能稍作完善一下，本想使菜单能看见工具的图标的，美化一下的，可是脑子里没门，现在也够用，就算了。  
  
DIY 过程如下（下面的内容多数不是为新手准备的）：  
  
首先，我们用 LordPE 加载 OD1.10 中文版，翻到 区段 和目录 部分，将重定位表偏移地址记住，然后擦除重定位表的所有相关信息，用 16 进制工具裁掉那个 section 的内容，因为我们要加入资源，而资源表又不是在最后的 section，或者你可以独立将重定位表保存起来，等 DIY 成功了再补回来也是可以的。另外，对于截获 OD1.10 的 消息流 的位置的方法，可参阅 pll621 老大的文章  
  
在菜单资源上 添加需要的菜单“工具(&T)”，如下：  
……  
POPUP "帮助(&H)"  
{  
  MENUITEM "版本信息(&A)",  2501  
  MENUITEM "帮助内容(&C)",  2502  
  MENUITEM SEPARATOR  
  MENUITEM "选择 API 帮助文件(&P)",  2503  
  MENUITEM "打开 API 帮助文件(&H)",  2504  
}  
POPUP "工具(&T)"  
{  
  MENUITEM "自定义工具",  2509  
  MENUITEM SEPARATOR  
  MENUITEM "计算器",  2510  
}  
}  
  
  
下面是我自己取名的对话窗口资源脚本： DIA\_CFG\_TOOLMENU  
  
DIA\_CFG\_TOOLMENU DIALOG 80, 30, 271, 225  
STYLE DS\_MODALFRAME | WS\_POPUP | WS\_VISIBLE | WS\_CAPTION | WS\_SYSMENU  
CAPTION "配置工具菜单"  
LANGUAGE LANG\_NEUTRAL, SUBLANG\_NEUTRAL  
FONT 9, "宋体"  
{  
   CONTROL "选择工具", 9099, BUTTON, BS\_GROUPBOX | WS\_CHILD | WS\_VISIBLE, 8, 9, 258, 191   
   CONTROL "", 9041, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 20, 160, 12   
   CONTROL "", 9042, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 38, 160, 12   
   CONTROL "", 9043, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 56, 160, 12   
   CONTROL "", 9044, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 74, 160, 12   
   CONTROL "", 9045, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 92, 160, 12   
   CONTROL "", 9046, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 110, 160, 12   
   CONTROL "", 9047, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 128, 160, 12   
   CONTROL "", 9048, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 146, 160, 12   
   CONTROL "", 9049, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 164, 160, 12   
   CONTROL "", 9050, EDIT, ES\_LEFT | ES\_AUTOHSCROLL | WS\_CHILD | WS\_VISIBLE | WS\_BORDER | WS\_TABSTOP, 45, 182, 160, 12   
   CONTROL "Tool01:", 9021, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 21, 30, 9   
   CONTROL "Tool02:", 9022, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 39, 30, 9   
   CONTROL "Tool03:", 9023, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 57, 30, 9   
   CONTROL "Tool04:", 9024, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 75, 30, 9   
   CONTROL "Tool05:", 9025, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 94, 30, 9   
   CONTROL "Tool06:", 9026, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 14, 112, 30, 9   
   CONTROL "Tool07:", 9027, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 129, 30, 9   
   CONTROL "Tool08:", 9028, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 147, 30, 9   
   CONTROL "Tool09:", 9029, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 165, 30, 9   
   CONTROL "Tool10:", 9030, STATIC, SS\_LEFT | WS\_CHILD | WS\_VISIBLE | WS\_GROUP, 13, 183, 30, 9   
   CONTROL "更改", 9001, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 17, 50, 14   
   CONTROL "更改", 9002, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 36, 50, 14   
   CONTROL "更改", 9003, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 55, 50, 14   
   CONTROL "更改", 9004, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 72, 50, 14   
   CONTROL "更改", 9005, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 90, 50, 14   
   CONTROL "更改", 9006, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 108, 50, 14   
   CONTROL "更改", 9007, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 126, 50, 14   
   CONTROL "更改", 9008, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 144, 50, 14   
   CONTROL "更改", 9009, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 162, 50, 14   
   CONTROL "更改", 9010, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 209, 180, 50, 14   
   CONTROL "确定(&O)", 8888, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 74, 205, 50, 14   
   CONTROL "取消(&C)", 2, BUTTON, BS\_PUSHBUTTON | WS\_CHILD | WS\_VISIBLE | WS\_TABSTOP, 154, 205, 50, 14   
}  
  
上面是一个我盗取一个游戏模拟器配置菜单的对话窗口模块，各控件名称和 ID（最好不存在冲突） 全改了，位置也微调好了，反正调整到自己满意就可以!  
你自己可以在 ResHacker 上测试，可是用它进补成功的！  
  
浏览资源得知，“确定”和“取消”的 ID 分别是 2 和 1 ，我这里确定改为 8888，希望它发落去!  
  
我找了这对话窗口（消息流较少的进行“盗版”）--> “版本信息”菜单  

> ```
> 代码:
> 00440C3C   push ebp
> 00440C3D   mov ebp,esp
> 00440C3F   add esp,-480
> 00440C45   mov eax,dword ptr ss:\[ebp+10\]
> 00440C48   mov edx,dword ptr ss:\[ebp+C\]
> 00440C4B   sub edx,110                     ;  Switch (cases 110..112)
> 00440C51   je short 00440C62               ;  NEWHAND.00440C62
> 00440C53   dec edx
> 00440C54   je short 00440CB2               ;  NEWHAND.00440CB2
> 00440C56   dec edx
> 00440C57   je 00440CF7                     ;  NEWHAND.00440CF7
> 00440C5D   jmp 00440D0E                    ;  NEWHAND.00440D0E
> 00440C62   mov byte ptr ss:\[ebp\-480\],0     ;  Case 110 (WM\_INITDIALOG) of switch 00440C4B
> 00440C69   mov byte ptr ss:\[ebp\-440\],0
> 00440C70   lea eax,dword ptr ss:\[ebp\-440\]
> 00440C76   lea ecx,dword ptr ss:\[ebp\-480\]
> 00440C7C   push eax                        ; /Arg6
> 00440C7D   push ecx                        ; |Arg5
> 00440C7E   push 0A                         ; |Arg4 = 0000000A
> 00440C80   push 1                          ; |Arg3 = 00000001
> 00440C82   lea eax,dword ptr ss:\[ebp\-400\]  ; |
> 00440C88   push 4B9073                     ; |Arg2 = 004B9073 ASCII 0A,"NewHand v%"
> 00440C8D   push eax                        ; |Arg1
> 00440C8E   call 004A6C2C                   ; \\NEWHAND.004A6C2C
> 00440C93   add esp,18
> 00440C96   lea edx,dword ptr ss:\[ebp\-400\]
> 00440C9C   push edx                        ; /Text
> 00440C9D   push 0E75                       ; |ControlID = E75 (3701.)
> 00440CA2   mov ecx,dword ptr ss:\[ebp+8\]    ; |
> 00440CA5   push ecx                        ; |hWnd
> 00440CA6   call 004AF58E                   ; \\SetDlgItemTextA
> 00440CAB   call 00546360                   ;  // 原指令 mov eax,1
> 00440CB0   jmp short 00440D10              ;  NEWHAND.00440D10
> 00440CB2   mov edx,eax                     ;  Case 111 (WM\_COMMAND) of switch 00440C4B
> 00440CB4   and dx,0FFFF
> 00440CB9   cmp dx,1
> 00440CBD   je short 00440CC5               ;  NEWHAND.00440CC5
> 00440CBF   cmp dx,2
> 00440CC3   jnz short 00440CD2              ;  NEWHAND.00440CD2
> 00440CC5   push 0                          ; /Result = 0
> 00440CC7   mov ecx,dword ptr ss:\[ebp+8\]    ; |
> 00440CCA   push ecx                        ; |hWnd
> 00440CCB   call 004AF3EA                   ; \\EndDialog
> 00440CD0   jmp short 00440D0E              ;  NEWHAND.00440D0E
> 00440CD2   call 00546206                   ;  // 原指令 cmp dx,0e76
> 00440CD7   jnz short 00440D0E              ;  NEWHAND.00440D0E
> 00440CD9   push 0                          ; /IsShown = 0
> 00440CDB   push 4B91A4                     ; |DefDir = "."
> 00440CE0   push 0                          ; |Parameters = NULL
> 00440CE2   push 4B917F                     ; |FileName = "http://home.t-online.de/home/NewHand"
> 00440CE7   push 4B917A                     ; |Operation = "open"
> 00440CEC   mov eax,dword ptr ss:\[ebp+8\]    ; |
> 00440CEF   push eax                        ; |hWnd
> 00440CF0   call 004AF342                   ; \\ShellExecuteA
> 00440CF5   jmp short 00440D0E              ;  NEWHAND.00440D0E
> 00440CF7   and eax,0FFF0                   ;  Case 112 (WM\_SYSCOMMAND) of switch 00440C4B
> 00440CFC   cmp eax,0F060
> 00440D01   jnz short 00440D0E              ;  NEWHAND.00440D0E
> 00440D03   push 0                          ; /Result = 0
> 00440D05   mov edx,dword ptr ss:\[ebp+8\]    ; |
> 00440D08   push edx                        ; |hWnd
> 00440D09   call 004AF3EA                   ; \\EndDialog
> 00440D0E   xor eax,eax                     ;  Default case of switch 00440C4B
> 00440D10   mov esp,ebp
> 00440D12   pop ebp
> 00440D13   retn 10
> 00440D16   nop
> 00440D17   nop
> 00440D18   push 0                          ; /lParam = NULL
> 00440D1A   mov eax,dword ptr ds:\[4D3B80\]   ; |
> 00440D1F   push 440C3C                     ; |DlgProc = NEWHAND.00440C3C //借用一下这里
> 00440D24   push eax                        ; |hOwner => NULL
> 00440D25   mov edx,dword ptr ds:\[4D3B78\]   ; |
> 00440D2B   push 4B91A6                     ; |pTemplate = "DIA\_ABOUT"
> 00440D30   push edx                        ; |hInst => NULL
> 00440D31   call 004AF3C6                   ; \\DialogBoxParamA
> 00440D36   retn
> ```
> 
>   

  
接下来为刚才处理过的 OD1.10 中文版增加一个 1000h 长度的 section，作为编写运行代码（数据）的空间  
  
下面是 DIY 我们自己需要的代码，对于修改消息流的指令就不贴了，已有注释，代码很乱，边改边写的，没办法了，下面看到的 API 都是从 OD1.10 里面找到的。。。代码区定为 va: 546000 - 5463FF  

> ```
> 代码:
> 00546000   nop
> 00546001   nop
> 00546002   nop
> 00546003   nop
> 00546004   nop
> 00546005   nop
> 00546006   nop
> 00546007   nop
> 00546008   nop
> 00546009   nop
> 0054600A   nop
> 0054600B   push ebp                         ; // 激活启动 计算器 功能
> 0054600C   mov ebp,esp
> 0054600E   pushad
> 0054600F   add esp,-120
> 00546015   push 104
> 0054601A   lea eax,dword ptr ss:\[esp+4\]
> 0054601E   push eax
> 0054601F   call 004AF0EA                    ; <jmp.&KERNEL32.GetWindowsDirectoryA>
> 00546024   add eax,esp
> 00546026   inc eax
> 00546027   mov dword ptr ds:\[eax\],636C6143
> 0054602D   mov dword ptr ds:\[eax+4\],6578652>
> 00546034   mov byte ptr ds:\[eax+8\],0
> 00546038   mov ebx,esp
> 0054603A   push 0
> 0054603C   push ebx
> 0054603D   push 0
> 0054603F   push eax
> 00546040   push 4B917A                      ; ASCII "open"
> 00546045   mov eax,dword ptr ss:\[ebp+8\]
> 00546048   push eax
> 00546049   call 004AF342                    ; <jmp.&SHELL32.ShellExecuteA>
> 0054604E   add esp,120
> 00546054   popad
> 00546055   mov esp,ebp
> 00546057   pop ebp
> 00546058   jmp 00434212                     ; // 返回执行结束的应该到达的地方
> 0054605D   nop
> 0054605E   cmp dx,9CE                       ; // 来自： 433A1F 的 sub edx,9C7
> 00546063   je short 0054600B                ; // ID 相等 则跳往 执行 计算器
> 00546065   sub eax,9CD
> 0054606A   je 005461E0                      ; // 自定义工具菜单 ID
> 00546070   cmp eax,0B
> 00546073   ja short 0054607E                ; // 是否 新添加菜单的  ID
> 00546075   cmp eax,2
> 00546078   jnb 005461A8                     ; NEWHAND.005461A8
> 0054607E   sub edx,9C7
> 00546084   jmp 00433A25                     ; // 返回原来的 消息检测队列
> 00546089   nop
> 0054608A   nop
> 0054608B   nop
> 0054608C   nop
> 0054608D   nop
> 0054608E   nop
> 0054608F   nop
> 00546090   nop
> 00546091   nop
> 00546092   nop
> 00546093   nop
> 00546094   push ebx                         ; // 初始化“工具”菜单
> 00546095   push ebp
> 00546096   mov ebp,esp
> 00546098   xor ebx,ebx
> 0054609A   push 0                           ; // 为重建菜单准备
> 0054609C   lea eax,dword ptr ds:\[ebx+9CF\]
> 005460A2   push eax
> 005460A3   mov edx,dword ptr ss:\[ebp+4\]
> 005460A6   push edx
> 005460A7   call 004AF55E                    ; <jmp.&USER32.RemoveMenu>
> 005460AC   inc ebx
> 005460AD   cmp ebx,0A
> 005460B0   jl short 0054609A                ; NEWHAND.0054609A
> 005460B2   xor ebx,ebx
> 005460B4   xor edi,edi
> 005460B6   push 4D53A4                      ; // Ollydbg.ini 路径
> 005460BB   push 100                         ; // 定义缓冲区大小
> 005460C0   push dword ptr ds:\[546E14\]       ; // 缓冲区，用于读取所对应的数据
> 005460C6   push 4C1700
> 005460CB   push dword ptr ds:\[546E10\]       ; NEWHAND.00546E08
> 005460D1   push 546E00                      ; ASCII "Tools"
> 005460D6   call 004AF08A                    ; <jmp.&KERNEL32.GetPrivateProfileStringA>
> 005460DB   cmp eax,0
> 005460DE   je short 0054614C                ; NEWHAND.0054614C
> 005460E0   mov eax,dword ptr ds:\[546E14\]
> 005460E5   mov esi,eax
> 005460E7   push 0
> 005460E9   push 80
> 005460EE   push 3
> 005460F0   push 0
> 005460F2   push 3
> 005460F4   push 0
> 005460F6   push eax
> 005460F7   call 004AEFCA                    ; <jmp.&KERNEL32.CreateFileA>
> 005460FC   xor ebx,ebx
> 005460FE   cmp eax,-1
> 00546101   je short 0054610A                ; // 检测文件的存在有效性
> 00546103   inc ebx
> 00546104   push eax
> 00546105   call 004AEFB8                    ; <jmp.&KERNEL32.CloseHandle>
> 0054610A   cmp bl,1
> 0054610D   je short 00546117                ; NEWHAND.00546117
> 0054610F   mov eax,dword ptr ds:\[546E14\]
> 00546114   mov byte ptr ds:\[eax\],0
> 00546117   cmp byte ptr ds:\[esi\],0
> 0054611A   je short 0054614C                ; NEWHAND.0054614C
> 0054611C   inc esi
> 0054611D   cmp byte ptr ds:\[esi\],0
> 00546120   jnz short 0054611C               ; NEWHAND.0054611C
> 00546122   dec esi
> 00546123   cmp byte ptr ds:\[esi\],5C
> 00546126   je short 0054612A                ; NEWHAND.0054612A
> 00546128   jmp short 00546122               ; NEWHAND.00546122
> 0054612A   mov byte ptr ds:\[esi\],0
> 0054612D   inc esi
> 0054612E   nop
> 0054612F   nop
> 00546130   nop
> 00546131   nop
> 00546132   nop
> 00546133   nop
> 00546134   nop
> 00546135   nop
> 00546136   nop
> 00546137   nop
> 00546138   nop
> 00546139   push esi
> 0054613A   lea eax,dword ptr ds:\[edi+9CF\]   ; // 根据消息 ID 偏移量进行计算
> 00546140   push eax
> 00546141   push 0
> 00546143   mov ecx,dword ptr ss:\[ebp+4\]
> 00546146   push ecx
> 00546147   call 004AF34E                    ; <jmp.&USER32.AppendMenuA>
> 0054614C   inc edi
> 0054614D   add byte ptr ds:\[546E0D\],1       ; // 调整 Key
> 00546154   nop
> 00546155   nop
> 00546156   nop
> 00546157   nop
> 00546158   nop
> 00546159   nop
> 0054615A   add dword ptr ds:\[546E14\],100    ; // 调整对应的缓冲区指针
> 00546164   cmp edi,0A
> 00546167   jl 005460B6                      ; // 10 个为限
> 0054616D   mov esp,ebp
> 0054616F   pop ebp
> 00546170   pop ecx
> 00546171   call 00546370                    ; // 还原初始变量值（全局）
> 00546176   jmp 00434247                     ; // 返回循环点
> 0054617B   nop
> 0054617C   nop
> 0054617D   nop
> 0054617E   nop
> 0054617F   nop
> 00546180   nop
> 00546181   nop
> 00546182   nop
> 00546183   nop
> 00546184   nop
> 00546185   nop
> 00546186   cmp ax,9CD                       ; // 来自 43359C 的 jne 434247
> 0054618A   jnz 00434247                     ; NEWHAND.00434247
> 00546190   call 00546370                    ; // 还原初始变量值（全局）
> 00546195   mov edi,546400
> 0054619A   mov ecx,0A00
> 0054619F   xor eax,eax
> 005461A1   rep stos byte ptr es:\[edi\]
> 005461A3   jmp 00546094                     ; NEWHAND.00546094
> 005461A8   sub al,2                         ; // 执行 exe 工具
> 005461AA   imul ebx,eax,100
> 005461B0   add ebx,dword ptr ds:\[546E14\]    ; NEWHAND.00546400
> 005461B6   mov ecx,ebx
> 005461B8   inc ecx
> 005461B9   cmp byte ptr ds:\[ecx\],0
> 005461BC   jnz short 005461B8               ; NEWHAND.005461B8
> 005461BE   inc ecx
> 005461BF   nop
> 005461C0   push 0
> 005461C2   push ebx
> 005461C3   push 0
> 005461C5   push ecx
> 005461C6   push 4B917A                      ; ASCII "open"
> 005461CB   mov eax,dword ptr ss:\[ebp+8\]
> 005461CE   push eax
> 005461CF   call 004AF342                    ; <jmp.&SHELL32.ShellExecuteA>
> 005461D4   jmp 00434212                     ; NEWHAND.00434212
> 005461D9   nop
> 005461DA   nop
> 005461DB   nop
> 005461DC   nop
> 005461DD   nop
> 005461DE   nop
> 005461DF   nop
> 005461E0   push 434212                      ; // 压入返回地址
> 005461E5   push 0
> 005461E7   mov eax,dword ptr ds:\[4D3B80\]
> 005461EC   push 440C3C
> 005461F1   push eax
> 005461F2   mov edx,dword ptr ds:\[4D3B78\]
> 005461F8   push 4B8FDA                      ; ASCII "DIA\_CFG\_TOOLMENU"
> 005461FD   push edx
> 005461FE   call 004AF3C6                    ; <jmp.&USER32.DialogBoxParamA>
> 00546203   retn
> 00546204   nop
> 00546205   nop
> 00546206   cmp dx,0E76                      ; // 来自 440CD2 的 cmp dx,0e76
> 0054620B   jnz short 0054620E               ; NEWHAND.0054620E
> 0054620D   retn
> 0054620E   cmp dx,22B8
> 00546213   je 0054630E                      ; // 是否点击了 DIY 的 “确定”按钮
> 00546219   sub eax,2328
> 0054621E   cmp eax,0A
> 00546221   ja short 00546298                ; NEWHAND.00546298
> 00546223   cmp eax,1
> 00546226   jb short 00546298                ; NEWHAND.00546298
> 00546228   nop                              ; // 是否点击了 DIY 的 “更改”按钮
> 00546229   nop
> 0054622A   pushad
> 0054622B   push eax
> 0054622C   push 546E20                      ; // 结构
> 00546231   call 004AF268                    ; <jmp.&COMDLG32.GetOpenFileNameA>
> 00546236   test eax,eax
> 00546238   jnz short 0054623F               ; NEWHAND.0054623F
> 0054623A   pop eax
> 0054623B   jmp short 00546297               ; // 没有选择则不登记，直接回到 消息队列
> 0054623D   nop
> 0054623E   nop
> 0054623F   push 2030
> 00546244   push 546E9F                      ; // 标题
> 00546249   push 546EAA                      ; // 提示 更改信息
> 0054624E   mov eax,dword ptr ds:\[4D3B7C\]
> 00546253   push eax
> 00546254   call 004AF516                    ; <jmp.&USER32.MessageBoxA>
> 00546259   pop eax
> 0054625A   nop
> 0054625B   nop
> 0054625C   dec eax                          ; // \*\*\*这里不即时登记 “更新”也是可以的
> 0054625D   add al,30                        ; // 计算所对应的 工具序列
> 0054625F   mov byte ptr ds:\[546E0D\],al
> 00546264   push 4D53A4
> 00546269   push dword ptr ds:\[546E18\]       ; NEWHAND.00546F00
> 0054626F   push dword ptr ds:\[546E10\]       ; // \*\*\*登记所选择的
> 00546275   push 546E00                      ; ASCII "Tools"
> 0054627A   call 004AF21C                    ; <jmp.&KERNEL32.WritePrivateProfileStringA>
> 0054627F   popad
> 00546280   pushad
> 00546281   lea ecx,dword ptr ds:\[eax+2350\]  ; // 对应的 Edit 控件 id
> 00546287   mov eax,dword ptr ds:\[546E18\]    ; // 对应的工具路径
> 0054628C   mov edx,dword ptr ss:\[ebp+8\]     ; // 窗口句柄
> 0054628F   push eax
> 00546290   push ecx
> 00546291   push edx
> 00546292   call 004AF58E                    ; // 更新对应的 Edit 控件内容
> 00546297   popad
> 00546298   mov al,1
> 0054629A   test al,al
> 0054629C   retn
> 0054629D   nop
> 0054629E   nop
> 0054629F   nop
> 005462A0   nop
> 005462A1   nop
> 005462A2   nop
> 005462A3   pushad                           ; // 初始化对话框 Edit 控件 的内容
> 005462A4   xor edi,edi
> 005462A6   lea eax,dword ptr ds:\[edi+30\]
> 005462A9   mov byte ptr ds:\[546E0D\],al
> 005462AE   push 4D53A4
> 005462B3   push 100
> 005462B8   push dword ptr ds:\[546E14\]       ; // 缓冲区读取所对应的数据
> 005462BE   push 4C1700
> 005462C3   push dword ptr ds:\[546E10\]       ; NEWHAND.00546E08
> 005462C9   push 546E00                      ; ASCII "Tools"
> 005462CE   call 004AF08A                    ; <jmp.&KERNEL32.GetPrivateProfileStringA>
> 005462D3   cmp al,0
> 005462D5   je short 005462ED                ; // 为 0 不进行刷新数据
> 005462D7   lea ecx,dword ptr ds:\[edi+2351\]
> 005462DD   mov eax,dword ptr ds:\[546E14\]
> 005462E2   mov edx,dword ptr ss:\[ebp+8\]
> 005462E5   push eax                         ; // 下面是读取并设置各工具菜单路径
> 005462E6   push ecx
> 005462E7   push edx
> 005462E8   call 004AF58E                    ; <jmp.&USER32.SetDlgItemTextA>
> 005462ED   inc edi
> 005462EE   add dword ptr ds:\[546E14\],100    ; // 调整对应的缓冲区指针
> 005462F8   cmp edi,0A
> 005462FB   jl short 005462A6                ; // 10 个为限
> 005462FD   popad
> 005462FE   retn
> 005462FF   nop
> 00546300   nop
> 00546301   nop
> 00546302   nop
> 00546303   nop
> 00546304   nop
> 00546305   nop
> 00546306   nop
> 00546307   nop
> 00546308   nop
> 00546309   nop
> 0054630A   nop
> 0054630B   nop
> 0054630C   nop
> 0054630D   nop
> 0054630E   pushad                           ; // 点按了“确定”按钮，登记所有对应的内容
> 0054630F   xor edi,edi
> 00546311   lea eax,dword ptr ds:\[edi+30\]
> 00546314   mov byte ptr ds:\[546E0D\],al
> 00546319   lea ecx,dword ptr ds:\[edi+2351\]
> 0054631F   mov eax,dword ptr ds:\[546E18\]
> 00546324   mov edx,dword ptr ss:\[ebp+8\]
> 00546327   push 100
> 0054632C   push eax
> 0054632D   push ecx
> 0054632E   push edx
> 0054632F   call 004AF45C                    ; <jmp.&USER32.GetDlgItemTextA>
> 00546334   push 4D53A4
> 00546339   push dword ptr ds:\[546E18\]       ; NEWHAND.00546F00
> 0054633F   push dword ptr ds:\[546E10\]       ; NEWHAND.00546E08
> 00546345   push 546E00                      ; ASCII "Tools"
> 0054634A   call 004AF21C                    ; <jmp.&KERNEL32.WritePrivateProfileStringA>
> 0054634F   inc edi
> 00546350   cmp edi,0A
> 00546353   jl short 00546311                ; NEWHAND.00546311
> 00546355   popad
> 00546356   pop eax
> 00546357   jmp 00440CC5                     ; // 返回到 点击“确定”按钮应该返回的地方
> 0054635C   nop
> 0054635D   nop
> 0054635E   nop
> 0054635F   nop
> 00546360   cmp al,1
> 00546362   jnz 005462A0                     ; // 由于借用别的 Dia 的事件，当它初始失败，al 为 0，就是应该属于我们的初始化
> 00546368   retn
> 00546369   nop
> 0054636A   nop
> 0054636B   nop
> 0054636C   nop
> 0054636D   nop
> 0054636E   nop
> 0054636F   nop
> 00546370   mov byte ptr ds:\[546E0D\],30      ; // 还原变量初值
> 00546377   mov dword ptr ds:\[546E14\],546400
> 00546381   retn
> 00546382   nop
> 
> ```
> 
>   

  
  
Ok，数据区定为 va: 546400 - 546FFF，以下是部分初始的数据区：  

> ```
> 代码:
> Offset      0  1  2  3  4  5  6  7   8  9  A  B  C  D  E  F
> 
> 变量和指针：
> 00105400   54 6F 6F 6C  73 00 00 00  54 6F 6F 6C  5B 30 5D 00   Tools...Tool\[0\].
> 00105410   08 6E 54 00  00 64 54 00  00 6F 54 00  00 00 00 00   .nT..dT..oT.....
> 
> OpenFileNameA 结构：
> 00105420   4C 00 00 00  00 00 00 00  00 00 00 00  70 6E 54 00   L...........pnT.
> 00105430   00 00 00 00  00 00 00 00  00 00 00 00  00 6F 54 00   .............oT.
> 00105440   04 01 00 00  00 00 00 00  00 00 00 00  00 00 00 00   ................
> 00105450   90 6E 54 00  06 28 28 00  00 00 00 00  00 00 00 00   恘 T..((.........
> 00105460   00 00 00 00  00 00 00 00  00 00 00 00  00 00 00 00   ................
> 
> 字符串常量：
> 00105470   BF C9 D6 B4  D0 D0 CE C4  BC FE 20 28  2A 2E 65 78   可执行文件 (\*.ex
> 00105480   65 29 00 2A  2E 65 78 65  00 00 00 00  00 00 00 00   e).\*.exe........
> 00105490   D1 A1 D4 F1  65 78 65 B9  A4 BE DF 00  00 00 00 B8   选择 exe 工具....?
> 001054A0   FC B8 C4 CC  E1 CA BE A3  BA 00 C4 E3  B5 C4 B8 FC   奶崾荆?你的更
> 001054B0   B8 C4 BC B4  BD AB BC A4  BB EE A3 A1  00 00 00 00   改即将激活！....
> 
> ```
> 
>   

  
注意找出主窗口句柄（参看一下其它窗口使用的函数参数是如何实现叫出来的，模拟一下指令就行了），测试中注意堆栈平衡，这个出错会很麻烦的，修修补补到现在，保存所有，运行测试通过了！特写下这篇文章，以慰已劳，Enjoy! 4dd466d0e1e0cc123afbe504da2462ca