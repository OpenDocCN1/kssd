
```
00452048    .  E8 D3B1FEFF     call unpacked.0043D220

0045204D    .  50              push eax                                  ; |hWnd

0045204E    .  68 14834000     push <jmp.&user32.DefMDIChildProcA>       ; |PrevProc = unpacked.00408314

00452053    .  E8 4462FBFF     call <jmp.&user32.CallWindowProcA>        ; \CallWindowProcA

00452058    .  8B45 FC         mov eax,dword ptr ss:[ebp-4]

0045205B    .  E8 1052FEFF     call unpacked.00437270

00452060    >  6A 00           push 0                                    ; /lParam = 0

00452062    .  6A 00           push 0                                    ; |wParam = 0

00452064    .  68 34020000     push 234                                  ; |Message = WM\_MDIREFRESHMENU

00452069    .  A1 D4375900     mov eax,dword ptr ds:[5937D4]             ; |

0045206E    .  8B40 38         mov eax,dword ptr ds:[eax+38]             ; |

00452071    .  8B80 3C020000   mov eax,dword ptr ds:[eax+23C]            ; |

00452077    .  50              push eax                                  ; |hWnd

00452078    .  E8 8766FBFF     call <jmp.&user32.SendMessageA>           ; \SendMessageA

0045207D    .  E9 2A010000     jmp unpacked.004521AC

00452082    >  8B45 FC         mov eax,dword ptr ss:[ebp-4]　　　　　　　从00451FC7跳来

00452085    .  0FB680 13020000 movzx eax,byte ptr ds:[eax+213]

0045208C    .  8B0485 9C5A5800 mov eax,dword ptr ds:[eax\*4+585A9C]

00452093    .  50              push eax

00452094    .  8B45 FC         mov eax,dword ptr ss:[ebp-4]

00452097    .  E8 84B1FEFF     call unpacked.0043D220　　　　　　　　　　进去看看　　　　

0045209C    .  50              push eax                                  eax为窗口句柄

0045209D    .  E8 4267FBFF     call <jmp.&user32.ShowWindow>             经过这个，Nag调出

004520A2    .  E9 05010000     jmp unpacked.004521AC　　　　　　　　　　　　 然后跳到下面

.................................................

004521AC    > \33C0            xor eax,eax　　　　　　　　　　　　　　　　分析上面特点知，

004521AE    .  5A              pop edx　　　　　　　　　　　　　　　　　　调用showwindow后

004521AF    .  59              pop ecx　　　　　　　　　　　　　　　　　　都来到这里

004521B0    .  59              pop ecx

004521B1    .  64:8910         mov dword ptr fs:[eax],edx

004521B4    .  68 CB214500     push unpacked.004521CB

004521B9    >  8B45 FC         mov eax,dword ptr ss:[ebp-4]

004521BC    .  80A0 CC020000 F>and byte ptr ds:[eax+2CC],0FB

004521C3    .  C3              retn                                      ;  RET used as a jump to 004521CB
```

知道从哪调出Nag的，`Nop`掉行不行？如果你不想要这个软件的其它功能的话，好像也可以。经过试验，这个软件的其它功能也用这个`Call`调用的。（怎么试验？运行后，点其它的窗口，发现没有反应。）`eax`的句柄是怎样产生的？进入`call unpacked.0043D220`看看。进入后，没看到什么有用的。看看这个`eax`的值是什么？记下来后，再次重运行，再来到这里。`eax`的值改变了，但仍指向Nag框。总有个不变的吧！要不怎样固定句柄？所以，再次进入`call unpacked.0043D220`：

```
0043D220   /$  53              push ebx

0043D221   |.  8BD8            mov ebx,eax

0043D223   |.  8BC3            mov eax,ebx

0043D225   |.  E8 D2FFFFFF     call unpacked.0043D1FC

0043D22A       8B83 4C010000   mov eax,dword ptr ds:[ebx+14C]

0043D230       5B              pop ebx

0043D231       C3              retn
```

从上面知由`mov eax,dword ptr ds:[ebx+14C]`把值给`eax`。记下`ebx`的值。再次运行，对于弹出Nag时`ebx`的值不变。而在`call unpacked.0043D220`前，是从`00451FC7`跳过来的，到那里去看看。

```
00451FB3    > \8B45 FC         mov eax,dword ptr ss:[ebp-4]              ;  here

00451FB6    .  C680 18020000 0>mov byte ptr ds:[eax+218],0

00451FBD    .  8B45 FC         mov eax,dword ptr ss:[ebp-4]

00451FC0       80B8 17020000 0>cmp byte ptr ds:[eax+217],1　　　　　　　　

00451FC7       0F85 B5000000   jnz unpacked.00452082　　　　　　　　　　　跳后出Nag

00451FCD       8B45 FC         mov eax,dword ptr ss:[ebp-4]

00451FD0    .  80B8 13020000 0>cmp byte ptr ds:[eax+213],2

00451FD7    .  75 36           jnz short unpacked.0045200F
```

多次试验，对于调出不同窗口时，`00451FC7`句也跳，也就是说跳时不一定就为Nag口。但`eax`的值是不同的，但这个`eax`与`0043D22A`处的`ebx`值相同。猜测这就是放句柄的地方。就这样再试试。找了一个一堆0的地方，自己写了些代码来判断。我找的是`00581815`处。（自己笨，没找到更好的办法。时间有限，也没有继续深入。）代码自己会改吧？在代码区双击或按空格键就会弹出汇编框，打入代码就可以。

```
00451FB3    > \8B45 FC         mov eax,dword ptr ss:[ebp-4]              ;  here

00451FB6    .  C680 18020000 0>mov byte ptr ds:[eax+218],0

00451FBD    .  8B45 FC         mov eax,dword ptr ss:[ebp-4]

00451FC0       3D 0080B700     cmp eax,0B78000　　　　　　　　　　　　　　将原先的改为这个

00451FC5       0F84 E1010000   je ccc.004521AC　　　　　　　　　　　　　　相同就跳到结束处

00451FCB       E9 45F81200     jmp ccc.00581815　　　　　　　　　　　　　 不同，把原来的代　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　码补上。　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　0B78000是Nag句柄　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　的地址吧

00581815       80B8 27010000 0>cmp byte ptr ds:[eax+127],1                补上原来的代码

0058181C     ^ 0F85 6008EDFF   jnz ccc.00452082

00581822       8B45 FC         mov eax,dword ptr ss:[ebp-4]

00581825     ^ E9 A607EDFF     jmp ccc.00451FD0　　　　　　　　　　　　　 跳回继续执行
```

这样改完，在代码区选中几行，点右键－>复制到可执行文件－>全部修正，然后在弹出的另一个框中点右键－>保存文件。保存好后，运行，成功了，没有Nag了。可知猜的也许是对的。这样就算把它给爆破了。到现在我还没发现有什么问题。有可能还有问题，但现在还没发现，发现时再说吧。费话说了不少，不知你看懂了吗。第一次写，水平有限，见谅。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
