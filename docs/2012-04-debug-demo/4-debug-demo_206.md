
# 菜鸟爆破破文（第二个破是“破烂”的破！）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-136.htm`](https://www.pediy.com/kssd/pediy07/pediy7-136.htm)

爆破多功能时钟日历 V2.5(菜鸟第一篇，高手可跳过)

【使用工具】 `ollydbg`，`DeDe3.5`

【破解平台】 `WinXP`

【软件名称】 多功能时钟日历 V2.5

【软件主页】 `http://www.magic2008.com/`

【破解原因】 1.快到我表妹的生日了，想确认一下时间，于是下了一个日历。说是破解版的。下了下来后，却有 Nag。感觉功能很好，比较喜欢它。但它是以你起动的次数做为让你等的时间（好像是 0.1 秒当一次吧），每次起动都要确定，而且等的时间越来起长。所以。。。嘿，谁让咱爱干这一行的呢，拿起家伙，爆了它。关于注册的钮是灰的，不知它有没有这个功能，只好先爆了。等以后再看看，如果有的话，再看看它的算法。2.在坛子里泡了也有大半年了，不能光吸收精华，也要做些什么吧。这个还有一定的难度（对菜鸟来说），就写成破文吧，算自己发的第一篇吧，做为对看雪的支持。废话多说，因为我目标是：没有蛀牙！（嘿，说错了，应该是：让最菜的菜鸟都能看懂！所以高手可以跳过。希望对菜菜们有帮助，否则我可是白浪费了不少时间。）难免有不之处，敬请指出！

【破解过程】

1. 先查壳。用 `PEid` 测出主程序用 `ASPack 2.12 -> Alexey Solodovnikov` 加了壳。可以手动（很简单），也可以用 `AspackDie` 来脱。脱完后，再测一下，得知是 `Borland Delphi 4.0 - 5.0`，这就应该用 `DeDe` 了。但这个程序有 `AntiDeDe`，没加载完就让退出。有时被破软件没退出，则 `stream read error`。改用 `exescope` 也看不到对话框。只有用 `OD` 载入，也看到有用的字符串，只有没具体目标的调来调去，乱下断点。因此花了半天，也没弄出什么。回宿舍，看了看第二版，才知改 `DeDe` 的名就可以躲过 `Anti`，所以改 `DeDe` 为 `CC`（任意）。这才能载入。

2. 用 `DeDe` 载入。载入后，我没选 `extended analasis`。然后选 窗体 ，从下面框中双击 `TFormR`，弹出 Nag 框和 `DFM Inspector` 窗口。从 `DFM Inspector` 中点击那些类，就会从 Nag 框中得到提示是选中的哪个。以闪烁几次表示。从而得知有用的 `Label` 和 `Button`。也可以用鼠标指向 Nag 框中，`DFM Inspector` 中也会改变。从而得到知：立即注册－－`Button1`，输入注册码－－`Button2`，运行的第几次－－`Label9`，同意－－`Button4`，退出－－`Button5`。

3. 再选 过程 ，选中 `TFormR` 那个类。右边的事件口就会显示对应的事件。可以双击事件，得到汇编代。记下有用的地址。如：`Timer1Timer` 的是 `00561C84`，`Timer2Timer` 的是 `00561CBC`，同意 的是 `00561C6C`。其它的都可以知道。记下地址后，再来到下一步。

4. 用 `OD` 载入。在你得到的地址处下断点。例如对于 `Timer1Timer` 的地址，用 `Ctrl+G`，输入 `00561C84`，点确定，来到 `00561C84` 处，按 `F2` 下断。

下面是在 `OD` 中找到对应处：

```
00561C56       8BC0            mov eax,eax
00561C58    .  53              push ebx
00561C59    .  8BDA            mov ebx,edx
00561C5B    .  8BD3            mov edx,ebx
00561C5D    .  E8 1AE9EEFF     call unpacked.0045057C
00561C62    .  E8 ED67EAFF     call <jmp.&user32.GetDesktopWindow>       ; \[GetDesktopWindow
00561C67    .  8943 1C         mov dword ptr ds:\[ebx+1C\],eax
00561C6A    .  5B              pop ebx
00561C6B    .  C3              retn
00561C6C    .  A1 70245900     mov eax,dword ptr ds:\[592470\]          ;  退出
00561C71    .  8B00            mov eax,dword ptr ds:\[eax\]
00561C73    .  E8 6406EFFF     call unpacked.004522DC
00561C78    .  C3              retn
00561C79       8D40 00         lea eax,dword ptr ds:\[eax\]             ;  同意
00561C7C    .  E8 5B06EFFF     call unpacked.004522DC
00561C81    .  C3              retn
00561C82       8BC0            mov eax,eax
00561C84    .  53              push ebx                                 ;  timer1
00561C85    .  8BD8            mov ebx,eax
00561C87    .  B2 01           mov dl,1
00561C89    .  8B83 18030000   mov eax,dword ptr ds:\[ebx+318\]
00561C8F    .  8B08            mov ecx,dword ptr ds:\[eax\]
00561C91    .  FF51 5C         call dword ptr ds:\[ecx+5C\]
00561C94    .  33D2            xor edx,edx
00561C96    .  8B83 20030000   mov eax,dword ptr ds:\[ebx+320\]
00561C9C    .  E8 7B78EFFF     call unpacked.0045951C
00561CA1    .  5B              pop ebx
00561CA2    .  C3              retn
00561CA3       90              nop
00561CA4    .  33C0            xor eax,eax
00561CA6    .  A3 684E5900     mov dword ptr ds:\[594E68\],eax
00561CAB    .  C3              retn
00561CAC    .  A1 AC235900     mov eax,dword ptr ds:\[5923AC\]          ;  输入注册码
00561CB1    .  8B00            mov eax,dword ptr ds:\[eax\]
00561CB3    .  8B10            mov edx,dword ptr ds:\[eax\]
00561CB5    .  FF92 D8000000   call dword ptr ds:\[edx+D8\]
00561CBB    .  C3              retn
00561CBC       55              push ebp                                 ;  timer2
00561CBD       8BEC            mov ebp,esp
00561CBF   |.  6A 00           push 0
00561CC1   |.  53              push ebx
00561CC2   |.  8BD8            mov ebx,eax
00561CC4   |.  33C0            xor eax,eax
00561CC6   |.  55              push ebp
00561CC7   |.  68 7F1D5600     push unpacked.00561D7F
00561CCC   |.  64:FF30         push dword ptr fs:\[eax\]
00561CCF   |.  64:8920         mov dword ptr fs:\[eax\],esp
00561CD2   |.  813D 684E5900 C>cmp dword ptr ds:\[594E68\],0C8
00561CDC   |.  7D 17           jge short unpacked.00561CF5
00561CDE   |.  A1 684E5900     mov eax,dword ptr ds:\[594E68\]
00561CE3   |.  B9 98080000     mov ecx,898
00561CE8   |.  99              cdq
00561CE9   |.  F7F9            idiv ecx
00561CEB   |.  85C0            test eax,eax
00561CED   |.  75 06           jnz short unpacked.00561CF5
```

然后 `F9` 运行。来到 `timer2` 断点处。Nag 框还没有弹出。再 `F9` 运行，出 Nag 框。说明在 Nag 这次调出。然后 `Ctrl+F2` 重来。运行一次。然后再慢慢用 `F7` 或 `F8` 来找。如果 `F8` 过不了，就用 `F7` 进入。对于这一步要有耐心，我都不知按了多少次 `Ctrl+F2`。如果你对 `API` 函数很了解，知道程序用 `ShowWindow` 来显示，你可以直接下断在那。用 `bp ShowWindow` 来下断。不过我用的是 `bp CreateWindowExA`。断是断下了，但是不断地 `retn`，一直没有找到关键的地方。这是我花了很长时间的一个原因。最后你会来到以下代码处。

```
00451C74       55              push ebp
00451C75    .  8BEC            mov ebp,esp
00451C77    .  83C4 F8         add esp,-8
00451C7A    .  53              push ebx
00451C7B    .  56              push esi
00451C7C    .  57              push edi
00451C7D    .  33C9            xor ecx,ecx
00451C7F    .  894D F8         mov dword ptr ss:\[ebp-8\],ecx
00451C82    .  8945 FC         mov dword ptr ss:\[ebp-4\],eax
00451C85    .  33C0            xor eax,eax
........................................................
00451FB3    > \\8B45 FC         mov eax,dword ptr ss:\[ebp-4\]           ;  here
00451FB6    .  C680 18020000 0>mov byte ptr ds:\[eax+218\],0
00451FBD    .  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00451FC0       80B8 17020000 0>cmp byte ptr ds:\[eax+217\],1          

00451FC7       0F85 B5000000   jnz unpacked.00452082           跳后就会出 Nag
00451FCD       8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00451FD0    .  80B8 13020000 0>cmp byte ptr ds:\[eax+213\],2
00451FD7    .  75 36           jnz short unpacked.0045200F
00451FD9    .  6A 00           push 0
00451FDB    .  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00451FDE    .  E8 3DB2FEFF     call unpacked.0043D220
00451FE3    .  50              push eax                             ; |wParam
00451FE4    .  68 23020000     push 223                              ; |Message = WM_MDIRESTORE
00451FE9    .  A1 D4375900     mov eax,dword ptr ds:\[5937D4\]       ; |
00451FEE    .  8B40 38         mov eax,dword ptr ds:\[eax+38\]       ; |
00451FF1    .  8B80 3C020000   mov eax,dword ptr ds:\[eax+23C\]      ; |
00451FF7    .  50              push eax                             ; |hWnd
00451FF8    .  E8 0767FBFF     call <jmp.&user32.SendMessageA>      ; \\SendMessageA
00451FFD    .  6A 03           push 3
00451FFF    .  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00452002    .  E8 19B2FEFF     call unpacked.0043D220
00452007    .  50              push eax                             ; |hWnd
00452008    .  E8 D767FBFF     call <jmp.&user32.ShowWindow>        ; \\ShowWindow
0045200D    .  EB 51           jmp short unpacked.00452060
0045200F    >  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00452012    .  0FB680 13020000 movzx eax,byte ptr ds:\[eax+213\]
00452019    .  8B0485 9C5A5800 mov eax,dword ptr ds:\[eax\*4+585A9C\]
00452020    .  50              push eax
00452021    .  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00452024    .  E8 F7B1FEFF     call unpacked.0043D220
00452029    .  50              push eax                             ; |hWnd
0045202A    .  E8 B567FBFF     call <jmp.&user32.ShowWindow>        ; \\ShowWindow
0045202F    .  8B45 FC         mov eax,dword ptr ss:\[ebp-4\]
00452032    .  8B40 38         mov eax,dword ptr ds:\[eax+38\]
00452035    .  8B55 FC         mov edx,dword ptr ss:\[ebp-4\]
00452038    .  8B52 3C         mov edx,dword ptr ds:\[edx+3C\]
0045203B    .  C1E2 10         shl edx,10
```



`0045203E`    .  `0BC2`            `or eax,edx`  

`00452040`    .  `50`              `push eax`  

`00452041`    .  `6A 00`           `push 0`  

`00452043`    .  `6A 05`           `push 5`  

`00452045`    .  `8B45 FC`         `mov eax,dword ptr ss:\[ebp-4\]`  

`00452048`    .  `E8 D3B1FEFF`     `call unpacked.0043D220`  

`0045204D`    .  `50`              `push eax`                             `; |hWnd`  

`0045204E`    .  `68 14834000`     `push <jmp.&user32.DefMDIChildProcA>`    `; |PrevProc = unpacked.00408314`  

`00452053`    .  `E8 4462FBFF`     `call <jmp.&user32.CallWindowProcA>`     `; \\CallWindowProcA`  

`00452058`    .  `8B45 FC`         `mov eax,dword ptr ss:\[ebp-4\]`  

`0045205B`    .  `E8 1052FEFF`     `call unpacked.00437270`  

`00452060`    >  `6A 00`           `push 0`                               `; /lParam = 0`  

`00452062`    .  `6A 00`           `push 0`                               `; |wParam = 0`  

`00452064`    .  `68 34020000`     `push 234`                             `; |Message = WM_MDIREFRESHMENU`  

`00452069`    .  `A1 D4375900`     `mov eax,dword ptr ds:\[5937D4\]`       `; |`  

`0045206E`    .  `8B40 38`        `mov eax,dword ptr ds:\[eax+38\]`       `; |`  

`00452071`    .  `8B80 3C020000`   `mov eax,dword ptr ds:\[eax+23C\]`      `; |`  

`00452077`    .  `50`              `push eax`                             `; |hWnd`  

`00452078`    .  `E8 8766FBFF`     `call <jmp.&user32.SendMessageA>`      `; \\SendMessageA`  

`0045207D`    .  `E9 2A010000`     `jmp unpacked.004521AC`  

`00452082`    >  `8B45 FC`         `mov eax,dword ptr ss:\[ebp-4\]`       从 `00451FC7` 跳来  

`00452085`    .  `0FB680 13020000` `movzx eax,byte ptr ds:\[eax+213\]`  

`0045208C`    .  `8B0485 9C5A5800` `mov eax,dword ptr ds:\[eax\*4+585A9C\]`  

`00452093`    .  `50`              `push eax`  

`00452094`    .  `8B45 FC`         `mov eax,dword ptr ss:\[ebp-4\]`  

`00452097`    .  `E8 84B1FEFF`     `call unpacked.0043D220`               进去看看           

`0045209C`    .  `50`              `push eax`                             `eax` 为窗口句柄  

`0045209D`    .  `E8 4267FBFF`     `call <jmp.&user32.ShowWindow>`        经过这个，`Nag` 调出  

`004520A2`    .  `E9 05010000`     `jmp unpacked.004521AC`                然后跳到下面  

.................................................  

`004521AC`    >  `\\33C0`          `xor eax,eax`                          分析上面特点知，  

`004521AE`    .  `5A`              `pop edx`                              调用 `showwindow` 后  

`004521AF`    .  `59`              `pop ecx`                              都来到这里  

`004521B0`    .  `59`              `pop ecx`  

`004521B1`    .  `64:8910`         `mov dword ptr fs:\[eax\],edx`  

`004521B4`    .  `68 CB214500`     `push unpacked.004521CB`  

`004521B9`    >  `8B45 FC`         `mov eax,dword ptr ss:\[ebp-4\]`  

`004521BC`    .  `80A0 CC020000 F>and byte ptr ds:\[eax+2CC\],0FB`  

`004521C3`    .  `C3`              `retn`                                 `; RET used as a jump to 004521CB`  

知道从哪调出 `Nag` 的，`Nop` 掉行不行？如果你不想要这个软件的其它功能的话，好像也可以。经过试验，这个软件的其它功能也用这个 `Call` 调用的。（怎么试验？运行后，点其它的窗口，发现没有反应。）`eax` 的句柄是怎样产生的？进入 `call unpacked.0043D220` 看看。进入后，没看到什么有用的。看看这个 `eax` 的值是什么？记下来后，再次重运行，再来到这里。`eax` 的值改变了，但仍指向 `Nag` 框。总有个不变的吧！要不怎样固定句柄？所以，再次进入 `call unpacked.0043D220`：  

`0043D220`   `/$`  `53`           `push ebx`  

`0043D221`   `|.`  `8BD8`         `mov ebx,eax`  

`0043D223`   `|.`  `8BC3`         `mov eax,ebx`  

`0043D225`   `|.`  `E8 D2FFFFFF`  `call unpacked.0043D1FC`  

`0043D22A`      `8B83 4C010000`  `mov eax,dword ptr ds:\[ebx+14C\]`  

`0043D230`      `5B`             `pop ebx`  

`0043D231`      `C3`             `retn`  

从上面知由 `mov eax,dword ptr ds:\[ebx+14C\]` 把值给 `eax`。记下 `ebx` 的值。再次运行，对于弹出 `Nag` 时 `ebx` 的值不变。而在 `call unpacked.0043D220` 前，是从 `00451FC7` 跳过来的，到那里去看看。  

`00451FB3`    >  `\\8B45 FC`     `mov eax,dword ptr ss:\[ebp-4\]`        `; here`  

`00451FB6`    .  `C680 18020000 0>mov byte ptr ds:\[eax+218\],0`  

`00451FBD`    .  `8B45 FC`       `mov eax,dword ptr ss:\[ebp-4\]`  

`00451FC0`       `80B8 17020000 0>cmp byte ptr ds:\[eax+217\],1`        

`00451FC7`       `0F85 B5000000`  `jnz unpacked.00452082`                跳后出 `Nag`  

`00451FCD`       `8B45 FC`       `mov eax,dword ptr ss:\[ebp-4\]`  

`00451FD0`    .  `80B8 13020000 0>cmp byte ptr ds:\[eax+213\],2`  

`00451FD7`    .  `75 36`         `jnz short unpacked.0045200F`  

多次试验，对于调出不同窗口时，`00451FC7` 句也跳，也就是说跳时不一定就为 `Nag` 口。但 `eax` 的值是不同的，但这个 `eax` 与 `0043D22A` 处的 `ebx` 值相同。猜测这就是放句柄的地方。就这样再试试。找了一个一堆 `0` 的地方，自己写了些代码来判断。我找的是 `00581815` 处。（自己笨，没找到更好的办法。时间有限，也没有继续深入。）代码自己会改吧？在代码区双击或按空格键就会弹出汇编框，打入代码就可以。  

`00451FB3`    >  `\\8B45 FC`     `mov eax,dword ptr ss:\[ebp-4\]`        `; here`  

`00451FB6`    .  `C680 18020000 0>mov byte ptr ds:\[eax+218\],0`  

`00451FBD`    .  `8B45 FC`       `mov eax,dword ptr ss:\[ebp-4\]`  

`00451FC0`       `3D 0080B700`   `cmp eax,0B78000`                       将原先的改为这个  

`00451FC5`       `0F84 E1010000`  `je ccc.004521AC`                       相同就跳到结束处  

`00451FCB`       `E9 45F81200`    `jmp ccc.00581815`                     不同，把原来的代码补上。                                      `0B78000` 是 `Nag` 句柄的地址吧  

`00581815`       `80B8 27010000 0>cmp byte ptr ds:\[eax+127\],1`       补上原来的代码  

`0058181C`     `^` `0F85 6008EDFF`  `jnz ccc.00452082`  

`00581822`       `8B45 FC`       `mov eax,dword ptr ss:\[ebp-4\]`  

`00581825`     `^` `E9 A607EDFF`    `jmp ccc.00451FD0`                    跳回继续执行  

这样改完，在代码区选中几行，点右键－>复制到可执行文件－>全部修正，然后在弹出的另一个框中点右键－>保存文件。保存好后，运行，成功了，没有 `Nag` 了。可知猜的也许是对的。这样就算把它给爆破了。到现在我还没发现什么问题。有可能还有问题，但现在还没发现，发现时再说吧。费话说了不少，不知你看懂了吗。第一次写，水平有限，见谅。 

`4dd466d0e1e0cc123afbe504da2462ca`
