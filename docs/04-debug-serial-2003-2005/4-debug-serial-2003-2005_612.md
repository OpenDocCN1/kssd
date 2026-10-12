
# CDRWIN 5.05.001

> 原文：[`www.pediy.com/kssd/pediy06/pediy6108.htm`](https://www.pediy.com/kssd/pediy06/pediy6108.htm)

软件: `CDRWIN 5.05.001`, 不是 `GoldenHawk` 的，是 `Padus` 和 `Englemann Media GmBH` 联合开发的。

功能：很酷的 CD 刻录软件

工具：`OllyDbg1.09b`. (已载入 `MFC42.Lib`, 这样看 MFC 的程序容易一些）

这是从天天精品下的，本来想用它烧几个音乐 CD, 不料随附的注册机不工作（算出的注册码通不过），便随手看看，发现很适合新手学习，所以就上一贴。菜鸟乐园嘛，总得有点菜鸟东东才好。

拿到一个软件，首先总要试着随便注册一下，当然不会成功，但可以看看其反映，看能否得到一些有用的信息。刚安装完，第一次运行的时候，软件说经有 xx 天试用期，然后正式运行。照例，注册选项在 `help` 菜单中。输入

名字：`lianzi2000`

公司：`Home`

随附的注册机给出如下号码：`5CC8R-D7TZA-50432-5T3F4-757H4`, 也输入。“OK”! 哈哈，跳出 `messagebox` 说 "wrong serial number"! 作者真帮忙。 有了这样的提示，我们可以在 `messageboxa` 下断点，然后从调用这个函数的地方向前找，如果真是菜鸟级的加密，一般来说很快就会找到判断注册码是否正确的地方，也就是爆破点。反之，如果要重新启动程序才能知道注册码是否正确的话，我们的工作就会艰苦很多。

现在，正式开始破解了。首先要看看程序是否被加了壳。如果有壳，先要想法把壳脱去。脱壳又是一个很大的话题（看这里有专门的脱壳版），需要专门研究。在命令窗口中执行：`fi cdrwin5.exe`, 显示没有壳，是一个标准的 PE 文件。 好，`OD` 载入程序，直接运行。因为输入了一次错误注册号，这下没有使用期了，一开始就跳出注册对话框。注册机虽然不对，但我们还是可以借鉴其基本形式，把试炼码定为 `13579-ABCDE-24680-EDCBA-98765`. 当然这不是必需的，反正我们要看个仔细。将来真正要破解的时候，学会利用一切有用的信息可以减少很多工作量。采用这种有规律的试炼码是为了在内存中容易辨认，什么时候程序在操作我们的试炼码，一眼就可以看出来。

输入这些东西以后、按“OK”以前，到 `OD` 中，打开“可执行模块”窗，找到并选择 `user32.dll`, 按 `ctrl-N` 打开函数名字窗口，找到并选择`getwindowtexta`,按 `F2` 下断。然后回到 `CDRWIN` 去按 `OK` 键。立刻被 `OD` 截住，停在 `getwindowtexta` 开始的地方。这时候，看屏幕右下方的堆栈显示，如果你对这个 API 熟悉的话，你就可以知道，现在 `esp` 指向的栈顶是返回地址，`[esp+4]`是子窗口句柄，他指定读哪一个编辑框里的字符串，而`[esp+8]`则是一个存放读出来的字符串的缓冲区地址，这样我们知道我们的输入被存放在内存的何处。我们不需要跟踪系统函数，所以直接按 `Alt-F9` 执行完这个函数并回到程序领空，发现是在 `MFC42` 中。我们也不需要跟踪这个模块，所以一路 `F8` 直到返回真正的程序模块 `CDRWIN5`.这时，我们刚刚从一个 `CALL` 返回，如果往上看，就会发现上一个 call 指令：

```
00423317  |. E8 70D30300    CALL <JMP.&MFC42.#2370_?DDX_Text@@YGXPAVCDataExchange@@HAAVC>
```

显然我们知道这个过程 `MFC42.#2370_?DDX_Text@@YGXPAVCDataExchange@@HAAVC` 调用了 `getwindowtexta` 系统函数来取得我们输入。下面还有两个类似的对这个函数的调用，多半是在获取另外两项输入。同时我们也注意两个事实：（1）call 指令之前有 3 个 push,说明这个函数可能需要 3 个参数。（2）函数返回后，我们没有看见任何返回值被保存利用。从函数的功能来看，它应当取得字符串，既然没有返回值，那么参数值一极可能是一个指针，它将得到字符缓冲区的地址（如果你熟悉 MFC 的函数，自然一切都很清楚）。第二个参数是一个立即数，显然是对话框子窗口的 ID，第一个参数（最后 push 的）对这三次 call 都是一样的，都来自`[ebp+8]`, 所以不像。所以只有第三个参数有可能。我们继续 `F8`，发现 `arg3=0x12FB8C,arg2=0x413,arg3=0x12F394`, 从堆栈上看到它的值为 1，显然是一个什么标志，先不用管。`F8` 执行函数，然后到对展上找 `12FB8C(arg3 的值)看到了什么？我们的试炼码！如果用右键点击堆栈的这一行，并选择 "following DUmp",就会在左边的窗口中看到，我们的试炼码被存在那里。现在我们已经知道这一段程序的功能，就是读取我们的输入，并放在数据段中。第三个 call 不用仔细看了，都一样，只不过是读取名字罢了。直接运行到 `ret` 指令上。一般的做法是记下我们的输入在内存中的地址，这里是：名字@`3A80F8`, 公司@`3A8058`, 试炼码@`3A80A8`， 但先不用忙着下断。现在我们可以执行 `ret` 指令返回，又到了 `MFC` 中一个函数的尾部，这里没有什么可疑的操作，一路 `F8` 直到函数返回 `CDRWIN5` 领空。在我们下面有几个对 `MDC.CString` 对象成员函数的调用。因为程序已经取得我们的输入，从这里开始已经有可能对我们的数据进行操作了，但不用着急，破解和编程一样，一开始总是粗线条大手笔，把握总体概念和思路，然后逐步求精解决问题。下面的 call 指令之前有 1 各 push,好像只有一个参数，但这里要注意，MFC 的对象成员函数常常利用 `ECX` 传递第一个参数。这里，在调用函数前 `ECX` 被装入了一个地址 `0x4815E4`。那个压入栈中的参数是 `0x12FB8C`. 如果你对数字敏感的话，你会记得它正是指向我们的试炼码的指针！不记得的话，就到堆栈上查一下看这个变量指向什么。我们现在粗跟踪，即先不跟进函数，用 `F8` 越过，直接看运行结果。结果函数返回了 `0x4815E4(记得返回值在 eax 中吗？)即那个通过 ECX 传递的变量。我们右击 eax 跟进显示(follow in Dump)，发现他指向我们的试炼码。所以这个函数可能是一个 CString 对象的赋值函数。下面两个同样的调用把名字和公司赋给其他两个 CString 对象。下面的一个函数有一个堆栈中的参数，就是刚才那个指向试炼码的指针。ECX 则被赋予当前堆栈顶(esp\=12f3c8)的值。函数返回后，eax=12f3c8. 通常这样的情况说明该参数确实是一个指针，而函数已经向这个指针赋了值，并返回该指针。查堆栈，发现它已经指向试炼码。后面一句保存了这个指针。接下来的 call 好像没有参数。对这种在用户模块里的调用要特别注意，因为它是作者自己写的，很可能是关键。在看下面，返回值被保存，并与一个立即数进行比较，十分可疑。但我们仍然先不跟进，直接 F8 越过，看到返回值为 3。下面的语句表明如果返回值大于或等于 0x41A 则跳走。再看下面，俨然两个 messagebox 函数！如果跳走，则执行第二个消息框。根据我们的返回值是显然不跳的，我们知道我们不会有这样的好运，即随便用一个试炼码就会正确，所以跳走多半是好事。所以我们修改 eax 为 0x41A. 一路 F8 就去执行第二个消息框。“注册完成！”。于是我们知道我们找到了判断处和一个爆破点。显然，这个返回 eax=3 的函数 CDRWIN5.457535 是注册码运算的关键。重新开始，我们就可以在 457535 直接下断，然后跟进。

```
00457535  /$ 55               PUSH EBP                                                                      
00457536  |. 8BEC             MOV EBP,ESP                                                                    
00457538  |. 6A FF            PUSH -1                                                                        
0045753A  |. 68 7D6E4600      PUSH CDRWIN5.00466E7D                                                          ; SE handler installation
0045753F  |. 64:A1 00000000   MOV EAX,DWORD PTR FS:[0]                                                      
00457545  |. 50               PUSH EAX                                                                        
00457546  |. 64:8925 000000>  MOV DWORD PTR FS:[0],ESP                                                     
0045754D  |. 81EC F0000000    SUB ESP,0F0                                                                    
00457553  |. 56               PUSH ESI                                                                        
00457554  |. C745 FC 000000>  MOV DWORD PTR SS:[EBP-4],0                                                   
0045755B  |. 68 9A044600      PUSH <JMP.&MFC42.#800_??1CString@@QAE@XZ>                                     ; Entry address
00457560  |. 68 A6044600      PUSH <JMP.&MFC42.#540_??0CString@@QAE@XZ>                                     ; Entry address
00457565  |. 6A 05            PUSH 5                                                                          
00457567  |. 6A 04            PUSH 4                                                                          
00457569  |. 8D45 DC          LEA EAX,DWORD PTR SS:[EBP-24]                                                  
0045756C  |. 50               PUSH EAX                                                                        
0045756D  |. E8 C49C0000      CALL CDRWIN5.00461236                                                         ; install SEH
00457572  |. C645 FC 01       MOV BYTE PTR SS:[EBP-4],1                                                    
00457576  |. 68 CCDF4700      PUSH CDRWIN5.0047DFCC                                                          ; ASCII "5BS8X-CCZDR-59B88-CBCK5-DGTSQ"
0045757B  |. 8D4D DC          LEA ECX,DWORD PTR SS:[EBP-24]                                                  
0045757E  |. E8 1D8F0000      CALL <JMP.&MFC42.#860_??4CString@@QAEABV0@PB                                   
00457583  |. 68 ECDF4700      PUSH CDRWIN5.0047DFEC                                                          ; ASCII "59S0@-5CRCD-57647-UPMY3-CEVRF"
00457588  |. 8D4D E0          LEA ECX,DWORD PTR SS:[EBP-20]                                                  
0045758B  |. E8 108F0000      CALL <JMP.&MFC42.#860_??4CString@@QAEABV0@PB                                   
00457590  |. 68 0CE04700      PUSH CDRWIN5.0047E00C                                                          ; ASCII "5ES8D-D5CRT-55606-D1CL0-BNAZL"
00457595  |. 8D4D E4          LEA ECX,DWORD PTR SS:[EBP-1C]                                                  
00457598  |. E8 038F0000      CALL <JMP.&MFC42.#860_??4CString@@QAEABV0@PB                                   
0045759D  |. 68 2CE04700      PUSH CDRWIN5.0047E02C                                                          ; ASCII "59S0C-RDQMN-57B22-6U6Y6-7TQDU"
004575A2  |. 8D4D E8          LEA ECX,DWORD PTR SS:[EBP-18]                                                  
004575A5  |. E8 F68E0000      CALL <JMP.&MFC42.#860_??4CString@@QAEABV0@PB                                   
004575AA  |. 68 4CE04700      PUSH CDRWIN5.0047E04C                                                          ; ASCII "5DC8R-LASH8-57228-LASH8-DLASH"
004575AF  |. 8D4D EC          LEA ECX,DWORD PTR SS:[EBP-14]                                                  
004575B2  |. E8 E98E0000      CALL <JMP.&MFC42.#860_??4CString@@QAEABV0@PB                                   
004575B7  |. C745 B4 000000>  MOV DWORD PTR SS:[EBP-4C],0                                                    
```



```
004575BE  |. EB 09             JMP SHORT CDRWIN5.004575C9

004575C0  |> 8B4D B4          /MOV ECX,DWORD PTR SS:[EBP-4C]            ; 上面的代码从数据段中取出 5 个内部的，很像注册码得东东。

004575C3  |. 83C1 01          |ADD ECX,1                                ; 极可能是黑名单

004575C6  |. 894D B4          |MOV DWORD PTR SS:[EBP-4C],ECX           ;

004575C9  |> 837D B4 04       CMP DWORD PTR SS:[EBP-4C],4

004575CD  |. 7F 4F            |JG SHORT CDRWIN5.0045761E

004575CF  |. 8B55 B4          |MOV EDX,DWORD PTR SS:[EBP-4C]

004575D2  |. 8D4C95 DC        |LEA ECX,DWORD PTR SS:[EBP+EDX*4-24]     ; 取一个黑码

004575D6  |. E8 95BBFAFF      |CALL CDRWIN5.00403170                  ; 从指针得到真正的字符串

004575DB  |. 50               |PUSH EAX                                ; /返回值为字符串首地址，用作下一个函数的参数

004575DC  |. 8D4D 08          |LEA ECX,DWORD PTR SS:[EBP+8]            ; |ECX 放上试炼码

004575DF  |. E8 BCEEFAFF      |CALL CDRWIN5.004064A0                  ; \比较字符串

004575E4  |. 85C0             |TEST EAX,EAX                            ；若不等，比较下一轮。

004575E6  |. 75 34            |JNZ SHORT CDRWIN5.0045761C

004575E8  |. C745 90 19FCFF   >MOV DWORD PTR SS:[EBP-70],-3E7         <===|   ；否则返回-3E7,失败

004575EF  |. C645 FC 00       |MOV BYTE PTR SS:[EBP-4],0              |

004575F3  |. 68 9A044600      |PUSH <JMP.&MFC42.#800_??1CString@@QAE@XZ> |   ; 所以这个循环检查是否是黑码之一

004575F8  |. 6A 05            |PUSH 5                                  |

004575FA  |. 6A 04            |PUSH 4                                  |

004575FC  |. 8D45 DC          |LEA EAX,DWORD PTR SS:[EBP-24]          |

004575FF  |. 50               |PUSH EAX                                |

00457600  |. E8 1B9B0000      |CALL CDRWIN5.00461120                 |

00457605  |. C745 FC FFFFFF   >MOV DWORD PTR SS:[EBP-4],-1            |

0045760C  |. 8D4D 08          |LEA ECX,DWORD PTR SS:[EBP+8]           |

0045760F  |. E8 868E0000      |CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ> |

00457614  |. 8B45 90          |MOV EAX,DWORD PTR SS:[EBP-70]          <===|

00457617  |. E9 CB0F0000      |JMP CDRWIN5.004585E7                  ;4585E7 是本过程的尾部。

0045761C  |>^EB A2            \\JMP SHORT CDRWIN5.004575C0

0045761E  |> 6A 00            PUSH 0                                  ; 字符索引

00457620  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]             ; 试炼码

00457623  |. E8 4889FCFF      CALL CDRWIN5.41FF70                    ; cdrwin5.41ff70 从字符串中返回索引指定的字符

00457628  |. 0FBEC8          MOVSX ECX,AL                            ；比较返回字符的是否为'5'

0045762B  |. 83F9 35         CMP ECX,35                              ;  code[0]=?'5'

0045762E  |. 75 7C           JNZ SHORT CDRWIN5.004576AC              ;若不是，跳走

00457630  |. 6A 01           PUSH 1                                  ；若是，取下一个字符

00457632  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]

00457635  |. E8 3689FCFF      CALL CDRWIN5.41FF70                    ;

0045763A  |. 0FBED0          MOVSX EDX,AL

0045763D  |. 83FA 43         CMP EDX,43                              ;是'C' 吗？

00457640  |. 75 6A           JNZ SHORT CDRWIN5.004576AC              ；若不是，跳走

00457642  |. 6A 02           PUSH 2

00457644  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]

00457647  |. E8 2489FCFF      CALL CDRWIN5.41FF70                    ;      .....

0045764C  |. 0FBEC0          MOVSX EAX,AL

0045764F  |. 83F8 43         CMP EAX,43                              ;  'C'

00457652  |. 75 58           JNZ SHORT CDRWIN5.004576AC

00457654  |. 6A 03           PUSH 3

00457656  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]

00457659  |. E8 1289FCFF      CALL CDRWIN5.41FF70

0045765E  |. 0FBEC8          MOVSX ECX,AL

00457661  |. 83F9 38         CMP ECX,38                              ;  '8'

00457664  |. 75 46           JNZ SHORT CDRWIN5.004576AC

00457666  |. 6A 04           PUSH 4

00457668  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]

0045766B  |. E8 0089FCFF      CALL CDRWIN5.41FF70                    ;

00457670  |. 0FBED0          MOVSX EDX,AL

00457673  |. 83FA 52         CMP EDX,52                              ;  'R'

00457676  |. 75 34           JNZ SHORT CDRWIN5.004576AC              ;  显然，上面的代码比较试炼码的前 5 个字符是不是'5CC8R'.

00457678  |. C745 8C 66FDFF  >MOV DWORD PTR SS:[EBP-74],-29A         <==========|       ;  如果是，返回-29A --死

0045767F  |. C645 FC 00       MOV BYTE PTR SS:[EBP-4],0             |       ；怪不得注册机不工作，原来也上了黑名单。

00457683  |. 68 9A044600      PUSH <JMP.&MFC42.#800_??1CString@@QAE@XZ>   |       ;  Entry address

00457688  |. 6A 05            PUSH 5                                  |

0045768A  |. 6A 04            PUSH 4                                  |
```



```
0045768C  |. 8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]    |
0045768F  |. 50           PUSH EAX                          |
00457690  |. E8 8B9A0000  CALL CDRWIN5.00461120             |
00457695  |. C745 FC FFFFFF>MOV DWORD PTR SS:[EBP-4],-1     |
0045769C  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]       |
0045769F  |. E8 F68D0000  CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ> |
004576A4  |. 8B45 8C      MOV EAX,DWORD PTR SS:[EBP-74]   <===========|
004576A7  |. E9 3B0f0000  JMP CDRWIN5.004585E7

004576AC  |> 6A 00        PUSH 0                              ;下面的代码与上差不多，看是否'5CD8R'.
004576AE  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]         ; 大概也有注册机用这个。
004576B1  |. E8 BA88FCFF  CALL CDRWIN5.41FF70                  ; \\
004576B6  |. 0FBEC8       MOVSX ECX,AL
004576B9  |. 83F9 35      CMP ECX,35                            ;  '5'
004576BC  |. 75 7C        JNZ SHORT CDRWIN5.0045773A

004576BE  |. 6A 01        PUSH 1                              ; /Arg1 = 00000001
004576C0  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]         ; |
004576C3  |. E8 A888FCFF  CALL CDRWIN5.41FF70                  ; \\CDRWIN5.41FF70
004576C8  |. 0FBED0       MOVSX EDX,AL
004576CB  |. 83FA 43      CMP EDX,43                            ;  'C'
004576CE  |. 75 6A        JNZ SHORT CDRWIN5.0045773A

004576D0  |. 6A 02        PUSH 2                              ; /Arg1 = 00000002
004576D2  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]         ; |
004576D5  |. E8 9688FCFF  CALL CDRWIN5.41FF70                  ; \\CDRWIN5.41FF70
004576DA  |. 0FBEC0       MOVSX EAX,AL
004576DD  |. 83F8 44      CMP EAX,44                            ;  'D'
004576E0  |. 75 58        JNZ SHORT CDRWIN5.0045773A

004576E2  |. 6A 03        PUSH 3                              ; /Arg1 = 00000003
004576E4  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]         ; |
004576E7  |. E8 8488FCFF  CALL CDRWIN5.41FF70                  ; \\CDRWIN5.41FF70
004576EC  |. 0FBEC8       MOVSX ECX,AL
004576EF  |. 83F9 38      CMP ECX,38                            ;  '8'
004576F2  |. 75 46        JNZ SHORT CDRWIN5.0045773A

004576F4  |. 6A 04        PUSH 4                              ; /Arg1 = 00000004
004576F6  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]         ; |
004576F9  |. E8 7288FCFF  CALL CDRWIN5.41FF70                  ; \\CDRWIN5.41FF70
004576FE  |. 0FBED0       MOVSX EDX,AL
00457701  |. 83FA 52      CMP EDX,52                            ;  'R'
00457704  |. 75 34        JNZ SHORT CDRWIN5.0045773A

00457706  |. C745 88 66FDFF>MOV DWORD PTR SS:[EBP-78],-29A      ;若一'5CD8R'开始，则返回-29A 去死

....................

00457735  |. E9 AD0E0000  JMP CDRWIN5.004585E7

0045773A  |> C745 9C 030000>MOV DWORD PTR SS:[EBP-64],3
00457741  |. 8B4D 9C      MOV ECX,DWORD PTR SS:[EBP-64]
00457744  |. 6BC9 03      IMUL ECX,ECX,3
00457747  |. 894D 9C      MOV DWORD PTR SS:[EBP-64],ECX         ；[ebp-64]=9
0045774A  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]           ;  试炼码
0045774D  |. E8 EE3EFBFF  CALL CDRWIN5.0040B640                  ;  cdrwin5.40B640 返回字符串长度
00457752  |. 8B55 9C      MOV EDX,DWORD PTR SS:[EBP-64]
00457755  |. 83C2 14      ADD EDX,14                              ;  不知道为什么这么麻烦
00457758  |. 3BC2         CMP EAX,EDX                               ；总之它比较试炼码长度是否为 1D(29)
0045775A  |. 74 34        JE SHORT CDRWIN5.00457790                ;  若是则跳走
0045775C  |. C745 84 010000>MOV DWORD PTR SS:[EBP-7C],1            ;  否则返回 1。失败

.............

00457790  |> 68 6CE04700  PUSH CDRWIN5.0047E06C                   ;  DS:[47e06c]='-'
00457795  |. 8D4D 98      LEA ECX,DWORD PTR SS:[EBP-68]
00457798  |. E8 218D0000  CALL <JMP.&MFC42.#537_??0CString@@QAE@PBD@Z> ;
0045779D  |. C645 FC 02   MOV BYTE PTR SS:[EBP-4],2
004577A1  |. 6A 01        PUSH 1
004577A3  |. 6A 05        PUSH 5                                    ; 字符索引
004577A5  |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]              ; 试炼码
004577A8  |. E8 C387FCFF  CALL CDRWIN5.41FF70                      ; 试炼码[5]='-'
```



004577AD  |. 50                 PUSH EAX
004577AE  |. 8D8D 7CFFFFFF      LEA ECX,DWORD PTR SS:[EBP-84]
004577B4  |. E8 05930000        CALL <JMP.&MFC42.#536_??0CString@@QAE@DH@Z>
004577B9  |. C645 FC 03         MOV BYTE PTR SS:[EBP-4],3
004577BD  |. 8D4D 98            LEA ECX,DWORD PTR SS:[EBP-68]
004577C0  |. 51                 PUSH ECX
004577C1  |. 8D95 7CFFFFFF      LEA EDX,DWORD PTR SS:[EBP-84]
004577C7  |. 52                 PUSH EDX
004577C8  |. E8 239DFBFF        CALL CDRWIN5.004114F0 ; cdrwin5.4114f0: 看试炼码第 5 位是不是‘-’
004577CD  |. 25 FF000000        AND EAX,0FF
004577D2  |. 85C0               TEST EAX,EAX
004577D4  |. 0F85 03010000      JNZ CDRWIN5.004578DD ; 不是则失败
004577DA  |. 6A 01              PUSH 1
004577DC  |. 6A 0B              PUSH 0B ; 第 B 位
004577DE  |. 8D4D 08            LEA ECX,DWORD PTR SS:[EBP+8]
004577E1  |. E8 8A87FCFF        CALL CDRWIN5.41FF70
004577E6  |. 50                 PUSH EAX
004577E7  |. 8D8D 74FFFFFF      LEA ECX,DWORD PTR SS:[EBP-8C]
004577ED  |. E8 CC920000        CALL <JMP.&MFC42.#536_??0CString@@QAE@DH@Z>
004577F2  |. C645 FC 04         MOV BYTE PTR SS:[EBP-4],4
004577F6  |. 8D45 98            LEA EAX,DWORD PTR SS:[EBP-68]
004577F9  |. 50                 PUSH EAX
004577FA  |. 8D8D 74FFFFFF      LEA ECX,DWORD PTR SS:[EBP-8C]
00457800  |. 51                 PUSH ECX
00457801  |. E8 EA9CFBFF        CALL CDRWIN5.004114F0
00457806  |. 8885 78FFFFFF      MOV BYTE PTR SS:[EBP-88],AL
0045780C  |. C645 FC 03         MOV BYTE PTR SS:[EBP-4],3
00457810  |. 8D8D 74FFFFFF      LEA ECX,DWORD PTR SS:[EBP-8C] ; v_12f4b0 => '-'
00457816  |. E8 7F8C0000        CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
0045781B  |. 8B95 78FFFFFF      MOV EDX,DWORD PTR SS:[EBP-88]
00457821  |. 81E2 FF000000      AND EDX,0FF
00457827  |. 85D2               TEST EDX,EDX
00457829  |. 0F85 AE000000      JNZ CDRWIN5.004578DD
0045782F  |. 6A 01              PUSH 1
00457831  |. 6A 11              PUSH 11
00457833  |. 8D4D 08            LEA ECX,DWORD PTR SS:[EBP+8]
00457836  |. E8 3587FCFF        CALL CDRWIN5.41FF70 ; 第 0x11 位
0045783B  |. 50                 PUSH EAX
0045783C  |. 8D8D 6CFFFFFF      LEA ECX,DWORD PTR SS:[EBP-94]
00457842  |. E8 77920000        CALL <JMP.&MFC42.#536_??0CString@@QAE@DH@Z>
00457847  |. C645 FC 05         MOV BYTE PTR SS:[EBP-4],5
0045784B  |. 8D45 98            LEA EAX,DWORD PTR SS:[EBP-68]
0045784E  |. 50                 PUSH EAX
0045784F  |. 8D8D 6CFFFFFF      LEA ECX,DWORD PTR SS:[EBP-94]
00457855  |. 51                 PUSH ECX
00457856  |. E8 959CFBFF        CALL CDRWIN5.004114F0
0045785B  |. 8885 70FFFFFF      MOV BYTE PTR SS:[EBP-90],AL
00457861  |. C645 FC 03         MOV BYTE PTR SS:[EBP-4],3
00457865  |. 8D8D 6CFFFFFF      LEA ECX,DWORD PTR SS:[EBP-94]
0045786B  |. E8 2A8C0000        CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
00457870  |. 8B95 70FFFFFF      MOV EDX,DWORD PTR SS:[EBP-90]
00457876  |. 81E2 FF000000      AND EDX,0FF
0045787C  |. 85D2               TEST EDX,EDX
0045787E  |. 75 5D              JNZ SHORT CDRWIN5.004578DD
00457880  |. 6A 01              PUSH 1
00457882  |. 6A 17              PUSH 17
00457884  |. 8D4D 08            LEA ECX,DWORD PTR SS:[EBP+8]
00457887  |. E8 E486FCFF        CALL CDRWIN5.41FF70 ; 第 0x17 位
0045788C  |. 50                 PUSH EAX
0045788D  |. 8D8D 64FFFFFF      LEA ECX,DWORD PTR SS:[EBP-9C]
00457893  |. E8 26920000        CALL <JMP.&MFC42.#536_??0CString@@QAE@DH@Z>
```
```



```asm
00457898  |. C645 FC 06     MOV BYTE PTR SS:[EBP-4],6
0045789C  |. 8D45 98        LEA EAX,DWORD PTR SS:[EBP-68]
0045789F  |. 50             PUSH EAX
004578A0  |. 8D8D 64FFFFFF  LEA ECX,DWORD PTR SS:[EBP-9C]
004578A6  |. 51             PUSH ECX
004578A7  |. E8 449CFBFF    CALL CDRWIN5.004114F0
004578AC  |. 8885 68FFFFFF  MOV BYTE PTR SS:[EBP-98],AL
004578B2  |. C645 FC 03     MOV BYTE PTR SS:[EBP-4],3
004578B6  |. 8D8D 64FFFFFF  LEA ECX,DWORD PTR SS:[EBP-9C]
004578BC  |. E8 D98B0000    CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
004578C1  |. 8B95 68FFFFFF  MOV EDX,DWORD PTR SS:[EBP-98]
004578C7  |. 81E2 FF000000  AND EDX,0FF
004578CD  |. 85D2           TEST EDX,EDX
004578CF  |. 75 0C          JNZ SHORT CDRWIN5.004578DD
                                ; 可见，上述代码比较注册码的第 5,b,11,17 位是否‘-’。
004578D1  |. C785 14FFFFFF  MOV DWORD PTR SS:[EBP-EC],0
                                ; 若不对，返回 0 失败
004578DB  |. EB 0A          JMP SHORT CDRWIN5.004578E7
004578DD  |> C785 14FFFFFF  MOV DWORD PTR SS:[EBP-EC],1
                                ; 这里，[ebp-EC]是一个标志，置 1 说明注册码形式正确
004578E7  |> 8A85 14FFFFFF  MOV AL,BYTE PTR SS:[EBP-EC]
004578ED  |. 8845 80        MOV BYTE PTR SS:[EBP-80],AL
                                ; 标志复制到[ebp-80]
004578F0  |. C645 FC 02     MOV BYTE PTR SS:[EBP-4],2
004578F4  |. 8D8D 7CFFFFFF  LEA ECX,DWORD PTR SS:[EBP-84]
004578FA  |. E8 9B8B0000    CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
004578FF  |. 8B4D 80        MOV ECX,DWORD PTR SS:[EBP-80]
00457902  |. 81E1 FF000000  AND ECX,0FF
00457908  |. 85C9           TEST ECX,ECX
0045790A  |. 74 46          JE SHORT CDRWIN5.00457952
                                ; 若形式正确，跳走
0045790C  |. C785 60FFFFFF  MOV DWORD PTR SS:[EBP-A0],2
                                ; 否则返回 2 失败
...........................................
0045794D  |. E9 950C0000    JMP CDRWIN5.004585E7
00457952  |> C745 CC 000000 MOV DWORD PTR SS:[EBP-34],0
                                ; 注册码形式正确就会跳到这里
00457959  |. 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]
0045795C  |. E8 458B0000    CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>
00457961  |. C645 FC 07     MOV BYTE PTR SS:[EBP-4],7
00457965  |. 6A 0B          PUSH 0B
00457967  |. 8D85 5CFFFFFF  LEA EAX,DWORD PTR SS:[EBP-A4]
                                ; v_12f498
0045796D  |. 50             PUSH EAX
0045796E  |. 8D4D 08        LEA ECX,DWORD PTR SS:[EBP+8]
                                ; 试炼码
00457971  |. E8 268F0000    CALL <JMP.&MFC42.#4129_?Left@CString@@QBE?AV
                                ; 返回式炼码开始的 0B 个字符
00457976  |. 8985 10FFFFFF  MOV DWORD PTR SS:[EBP-F0],EAX
                                ; 保存
0045797C  |. 8B8D 10FFFFFF  MOV ECX,DWORD PTR SS:[EBP-F0]
00457982  |. 898D 0CFFFFFF  MOV DWORD PTR SS:[EBP-F4],ECX
00457988  |. C645 FC 08     MOV BYTE PTR SS:[EBP-4],8
0045798C  |. 8B95 0CFFFFFF  MOV EDX,DWORD PTR SS:[EBP-F4]
00457992  |. 52             PUSH EDX
00457993  |. 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]
00457996  |. E8 118B0000    CALL <JMP.&MFC42.#858_??4CString@@QAEABV0@AB
0045799B  |. C645 FC 07     MOV BYTE PTR SS:[EBP-4],7
0045799F  |. 8D8D 5CFFFFFF  LEA ECX,DWORD PTR SS:[EBP-A4]
004579A5  |. E8 F08A0000    CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
004579AA  |. 68 70E04700    PUSH CDRWIN5.0047E070
                                ; 内部数据=>'C'
004579AF  |. 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]
                                ; 试炼码前 0B 个字符
004579B2  |. E8 978E0000    CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPB
                                ; 看是否包含'C'
004579B7  |. 83F8 FF        CMP EAX,-1
004579BA  |. 75 09          JNZ SHORT CDRWIN5.004579C5
                                ; 必须包含 'C', 否则 [ebp-34]=0 die
004579BC  |. C745 CC 000000 MOV DWORD PTR SS:[EBP-34],0
004579C3  |. EB 07          JMP SHORT CDRWIN5.004579CC
004579C5  |> C745 CC 010000 MOV DWORD PTR SS:[EBP-34],1
                                ; 又是一个标志[ebp-34],置 1 good
004579CC  |> 837D CC 01     CMP DWORD PTR SS:[EBP-34],1
004579D0  |. 75 1B          JNZ SHORT CDRWIN5.004579ED
                                ; 'D'
004579D2  |. 68 74E04700    PUSH CDRWIN5.0047E074
004579D7  |. 8D4D AC        LEA ECX,DWORD PTR SS:[EBP-54]
                                ; code[0-A]
004579DA  |. E8 6F8E0000    CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>
004579DF  |. 83F8 FF        CMP EAX,-1
                                ; 必须包含 'D', 否则失败
004579E2  |. 74 09          JE SHORT CDRWIN5.004579ED
004579E4  |. C745 CC 010000 MOV DWORD PTR SS:[EBP-34],1
004579EB  |. EB 07          JMP SHORT CDRWIN5.004579F4
004579ED  |> C745 CC 000000 MOV DWORD PTR SS:[EBP-34],0
```



```
004579F4 |> 837D CC 01    CMP DWORD PTR SS:[EBP-34],1                          ;  
004579F8 |. 75 1B         JNZ SHORT CDRWIN5.00457A15  
004579FA |. 68 78E04700   PUSH CDRWIN5.0047E078                                ;  'R'  
004579FF |. 8D4D AC       LEA ECX,DWORD PTR SS:[EBP-54]  
00457A02 |. E8 478E0000   CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A07 |. 83F8 FF      CMP EAX,-1                                             ;  也必须有'R'  
00457A0A |. 74 09        JE SHORT CDRWIN5.00457A15  
00457A0C |. C745 CC 010000>MOV DWORD PTR SS:[EBP-34],1  
00457A13 |. EB 07        JMP SHORT CDRWIN5.00457A1C  
00457A15 |> C745 CC 000000>MOV DWORD PTR SS:[EBP-34],0  
00457A1C |> 68 7CE04700  PUSH CDRWIN5.0047E07C                                  ;  'P'  
00457A21 |. 8D4D AC      LEA ECX,DWORD PTR SS:[EBP-54]  
00457A24 |. E8 258E0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A29 |. 83F8 FF     CMP EAX,-1                                             ;  却不能有 'P'  
00457A2C |. 74 52       JE SHORT CDRWIN5.00457A80                              ；否则返回 0D 失败  
00457A2E |. C785 58FFFFFF >MOV DWORD PTR SS:[EBP-A8],0D  
.............................  
00457A7B |. E9 670B0000 JMP CDRWIN5.004585E7  
00457A80 |> 68 80E04700  PUSH CDRWIN5.0047E080                                  ;  'O'  
00457A85 |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;  试炼码  
00457A88 |. E8 C18D0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A8D |. 83F8 FF     CMP EAX,-1                                             ；整个注册码不能有'O'  
00457A90 |. 74 52       JE SHORT CDRWIN5.00457AE4                              ；否则返回 0E 失败  
00457A92 |. C785 54FFFFFF >MOV DWORD PTR SS:[EBP-AC],0E  
................................  
00457ADF |. E9 030B0000  JMP CDRWIN5.004585E7  
00457AE4 |> 68 84E04700  PUSH CDRWIN5.0047E084                                  ;  'I'  
00457AE9 |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;  试炼码  
00457AEC |. E8 5D8D0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>       ;  
00457AF1 |. 83F8 FF      CMP EAX,-1  
00457AF4 |. 74 52        JE SHORT CDRWIN5.00457B48                              ;  也不能有 'I'  
00457AF6 |. C785 50FFFFFF >MOV DWORD PTR SS:[EBP-B0],0F                         ；否则返回 0F 失败  
..............................  
00457B43 |. E9 9F0A0000  JMP CDRWIN5.004585E7  
00457B48 |> C745 B0 000000>MOV DWORD PTR SS:[EBP-50],0  
00457B4F |. C745 A8 060000>MOV DWORD PTR SS:[EBP-58],6                          ;  字符索引，从试炼码的第二部分开始。  
00457B56 |. EB 09        JMP SHORT CDRWIN5.00457B61                              ;   
00457B58 |> 8B45 A8      /MOV EAX,DWORD PTR SS:[EBP-58]  
00457B5B |. 83C0 01      |ADD EAX,1  
00457B5E |. 8945 A8      |MOV DWORD PTR SS:[EBP-58],EAX  
00457B61 |> 837D A8 0B   CMP DWORD PTR SS:[EBP-58],0B                           ;  终止位置  
00457B65 |. 7D 33        |JGE SHORT CDRWIN5.00457B9A  
00457B67 |. 8B4D A8      |MOV ECX,DWORD PTR SS:[EBP-58]  
00457B6A |. 51           |PUSH ECX                                               ; 6  
00457B6B |. 8D4D 08      |LEA ECX,DWORD PTR SS:[EBP+8]                          ; 试炼码  
00457B6E |. E8 FD83FCFF  |CALL CDRWIN5.0041FF70                                 ; 返回第六个字符  
00457B73 |. 0FBED0       |MOVSX EDX,AL  
00457B76 |. 8B45 B0      |MOV EAX,DWORD PTR SS:[EBP-50]                         ;  [ebp-50] 存放加和结果  
00457B79 |. 03C2         |ADD EAX,EDX  
00457B7B |. 8945 B0      |MOV DWORD PTR SS:[EBP-50],EAX  
00457B7E |. 8B4D A8      |MOV ECX,DWORD PTR SS:[EBP-58]  
00457B81 |. 83C1 06      |ADD ECX,6                                              ;  字符索引+6  
00457B84 |. 51           |PUSH ECX                                               ;  即取试炼码第三部分的对应字符  
00457B85 |. 8D4D 08      |LEA ECX,DWORD PTR SS:[EBP+8]                          ;   
00457B88 |. E8 E383FCFF  |CALL CDRWIN5.0041FF70                                 ;   
00457B8D |. 0FBED0       |MOVSX EDX,AL  
00457B90 |. 8B45 B0      |MOV EAX,DWORD PTR SS:[EBP-50]  
00457B93 |. 03C2         |ADD EAX,EDX  
00457B95 |. 8945 B0      |MOV DWORD PTR SS:[EBP-50],EAX                         ；加上  
00457B98 |.^EB BE        \\JMP SHORT CDRWIN5.00457B58                           ;  以上循环计算第二、第三部分共十个字符的 ascii 码的加和。  
00457B9A |> 8B4D B0      MOV ECX,DWORD PTR SS:[EBP-50]                          ;   
00457B9D |. 81E1 01000080 AND ECX,80000001                                      ；检查是否为偶数  
00457BA3 |. 79 05        JNS SHORT CDRWIN5.00457BAA                             ; 没有符号是肯定的。   
00457BA5 |. 49           DEC ECX                                                ;   
00457BA6 |. 83C9 FE      OR ECX,FFFFFFFE  
00457BA9 |. 41           INC ECX  
00457BAA |> 85C9         TEST ECX,ECX  
00457BAC |. 74 52       JE SHORT CDRWIN5.00457C00                              ；必须是偶数  
00457BAE |. C785 4CFFFFFF >MOV DWORD PTR SS:[EBP-B4],3                          ；否则返回 3 失败  
.............................  
00457BFB |. E9 E7090000 JMP CDRWIN5.004585E7  
00457C00 |> 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]                          ;   
00457C03 |. E8 9E880000  CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>  
00457C08 |. C645 FC 09   MOV BYTE PTR SS:[EBP-4],9  
00457C0C |. 6A 10        PUSH 10                                                ; 取试炼码第 0x10 个字符  
00457C0E |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;   
00457C11 |. E8 5A83FCFF  CALL CDRWIN5.0041FF70                                 ;   
00457C16 |. 50           PUSH EAX  
00457C17 |. 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]  
00457C1A |. E8 A9930000  CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z>        ;  存入数据段 [3A8360]处  
00457C1F |. 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]  
00457C22 |. E8 49B5FAFF  CALL CDRWIN5.00403170  
00457C27 |. 50           PUSH EAX                                              ;   
00457C28 |. FF15 948A4600 CALL DWORD PTR DS:[<&MSVCRT.atoi>]                  ; 转换成对应的值  
00457C2E |. 83C4 04      ADD ESP,4  
00457C31 |. 8945 F0      MOV DWORD PTR SS:[EBP-10],EAX                         ;  必须是一个阿拉伯数字  
00457C34 |. 837D F0 00   CMP DWORD PTR SS:[EBP-10],0                           ；返回零通常表示不是数字  
00457C38 |. 75 70        JNZ SHORT CDRWIN5.00457CAA  
00457C3A |. 6A 10        PUSH 10                                              ;   
00457C3C |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                          ; 但必须检查他是不是就是'0'  
00457C3F |. E8 2C83FCFF  CALL CDRWIN5.0041FF70                                 ;   
00457C44 |. 0FBEC0       MOVSX EAX,AL  
00457C47 |. 83F8 30      CMP EAX,30                                            ;   
00457C4A |. 74 5E       JE SHORT CDRWIN5.00457CAA  
00457C4C |. C785 48FFFFFF >MOV DWORD PTR SS:[EBP-B8],4                         ;若不是阿拉伯数字，则返回 4 失败  
.............................  
00457CA5 |. E9 3D090000  JMP CDRWIN5.004585E7  
00457CAA |> 8B55 F0      MOV EDX,DWORD PTR SS:[EBP-10]                        ;  code[0x10]对应的值加上 5  
00457CAD |. 83C2 05      ADD EDX,5  
00457CB0 |. 8955 F0      MOV DWORD PTR SS:[EBP-10],EDX  
00457CB3 |. 837D F0 09   CMP DWORD PTR SS:[EBP-10],9                          ;  超过 9?  
00457CB7 |. 0F8E 84000000 JLE CDRWIN5.00457D41  
00457CBD |. 8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]  
00457CC0 |. 83E8 0A      SUB EAX,0A                                            ;  若超过 9 则对 10 取余  
00457CC3 |. 8945 F0      MOV DWORD PTR SS:[EBP-10],EAX  
00457CC6 |. 8B4D F0      MOV ECX,DWORD PTR SS:[EBP-10]                        ；于数加上 0x41 变成大写字母  
00457CC9 |. 83C1 41      ADD ECX,41                                            ;   
```



```
00457CCC  .| 894D F0        MOV DWORD PTR SS:[EBP-10],ECX

00457CCF  .| 6A 18          PUSH 18                             ; 取 code[0x18]

00457CD1  .| 8D4D 08       LEA ECX,DWORD PTR SS:[EBP+8]         ;

00457CD4  .| E8 9782FCFF   CALL CDRWIN5.0041FF70               ;

00457CD9  .| 0FBED0        MOVSX EDX,AL                         ;

00457CDC  .| 3B55 F0       CMP EDX,DWORD PTR SS:[EBP-10]        ;code[0x18]必须等于刚才计算的大写字母

00457CDF  .| 74 5E         JE SHORT CDRWIN5.00457D3F            ;

00457CE1  .| C785 44FFFFFF >MOV DWORD PTR SS:[EBP-BC],5         ；否则返回 5 失败

.....................

00457D3A  .| E9 A8080000   JMP CDRWIN5.004585E7

00457D3F  |> EB 79         JMP SHORT CDRWIN5.00457DBA

.............

00457DBA  |> C745 BC 000000>MOV DWORD PTR SS:[EBP-44],0

00457DC1  .| C745 A0 000000>MOV DWORD PTR SS:[EBP-60],0         ；字符索引，从试炼码的开头开始

00457DC8  .| EB 09         JMP SHORT CDRWIN5.00457DD3

00457DCA  |> 8B4D A0       /MOV ECX,DWORD PTR SS:[EBP-60]

00457DCD  .| 83C1 01       |ADD ECX,1

00457DD0  .| 894D A0       |MOV DWORD PTR SS:[EBP-60],ECX

00457DD3  |> 837D A0 05    CMP DWORD PTR SS:[EBP-60],5          ;  0x5: 计数器

00457DD7  .| 7D 67         |JGE SHORT CDRWIN5.00457E40

00457DD9  .| 8B55 A0       |MOV EDX,DWORD PTR SS:[EBP-60]

00457DDC  .| 52            |PUSH EDX                             ;  从第一部分取一个字符

00457DDD  .| 8D4D 08       |LEA ECX,DWORD PTR SS:[EBP+8]         ;

00457DE0  .| E8 8B81FCFF   |CALL CDRWIN5.0041FF70               ;

00457DE5  .| 0FBEC0        |MOVSX EAX,AL                        ；

00457DE8  .| 8B4D BC      |MOV ECX,DWORD PTR SS:[EBP-44]        ;  [ebp-44] 存放加和结果

00457DEB  .| 03C8          |ADD ECX,EAX                         ；加和

00457DED  .| 894D BC      |MOV DWORD PTR SS:[EBP-44],ECX        ;

00457DF0  .| 8B55 A0       |MOV EDX,DWORD PTR SS:[EBP-60]

00457DF3  .| 83C2 06       |ADD EDX,6                           ;  到第二部分区对应的字符

00457DF6  .| 52            |PUSH EDX                            ;

00457DF7  .| 8D4D 08       |LEA ECX,DWORD PTR SS:[EBP+8]        ;

00457DFA  .| E8 7181FCFF   |CALL CDRWIN5.0041FF70               ;

00457DFF  .| 0FBEC0        |MOVSX EAX,AL

00457E02  .| 8B4D BC       |MOV ECX,DWORD PTR SS:[EBP-44]

00457E05  .| 03C8          |ADD ECX,EAX                         ;  也加上

00457E07  .| 894D BC       |MOV DWORD PTR SS:[EBP-44],ECX       ;  存储结果

00457E0A  .| 8B55 A0       |MOV EDX,DWORD PTR SS:[EBP-60]

00457E0D  .| 83C2 12       |ADD EDX,12                          ;  到第四部分取相应的字符

00457E10  .| 52            |PUSH EDX                            ;

00457E11  .| 8D4D 08       |LEA ECX,DWORD PTR SS:[EBP+8]        ;

00457E14  .| E8 5781FCFF   |CALL CDRWIN5.0041FF70               ;

00457E19  .| 0FBEC0        |MOVSX EAX,AL

00457E1C  .| 8B4D BC       |MOV ECX,DWORD PTR SS:[EBP-44]

00457E1F  .| 03C8          |ADD ECX,EAX                         ;  加

00457E21  .| 894D BC       |MOV DWORD PTR SS:[EBP-44],ECX       ;

00457E24  .| 8B55 A0       |MOV EDX,DWORD PTR SS:[EBP-60]

00457E27  .| 83C2 18       |ADD EDX,18                          ;  到第五部分取相应的字符

00457E2A  .| 52            |PUSH EDX                            ;

00457E2B  .| 8D4D 08       |LEA ECX,DWORD PTR SS:[EBP+8]        ;

00457E2E  .| E8 3D81FCFF   |CALL CDRWIN5.0041FF70               ;

00457E33  .| 0FBEC0        |MOVSX EAX,AL

00457E36  .| 8B4D BC       |MOV ECX,DWORD PTR SS:[EBP-44]

00457E39  .| 03C8          |ADD ECX,EAX

00457E3B  .| 894D BC       |MOV DWORD PTR SS:[EBP-44],ECX        ;  加上

00457E3E  .|^EB 8A          \JMP SHORT CDRWIN5.00457DCA           ;  这个循环计算注册码的第 1,2,4,5 部分共 20 个字符的 ascii 码加和

00457E40  |> 8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]        ；

00457E43  .| E8 5E860000   CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>

00457E48  .| C645 FC 0A    MOV BYTE PTR SS:[EBP-4],0A

00457E4C  .| 8B55 BC       MOV EDX,DWORD PTR SS:[EBP-44]        ;  加和结果

00457E4F  .| 52            PUSH EDX                             ;  作为参数

00457E50  .| 68 88E04700   PUSH CDRWIN5.0047E088                ;  数据段=〉"%X" 格式化字符串

00457E55  .| 8D45 A4       LEA EAX,DWORD PTR SS:[EBP-5C]

00457E58  .| 50            PUSH EAX

00457E59  .| E8 9C860000   CALL <JMP.&MFC42.#2818_?format@CString@@QAAXPBDZZ>  ;  加和结果转换成字符串

00457E5E  .| 83C4 0C       ADD ESP,0C                           ;  在[ebp-5c]中

00457E61  .| 8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]       ；

00457E64  .| E8 3D860000   CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>         ；

00457E69  .| C645 FC 0B    MOV BYTE PTR SS:[EBP-4],0B

00457E6D  .| 6A 03         PUSH 3                               ；子串长度

00457E6F  .| 6A 0C         PUSH 0C                              ；子串位置

00457E71  .| 8D8D 3CFFFFFF LEA ECX,DWORD PTR SS:[EBP-C4]       ;  v_12f2fC

00457E77  .| 51            PUSH ECX

00457E78  .| 8D4D 08       LEA ECX,DWORD PTR SS:[EBP+8]        ;  试炼码

00457E7B  .| E8 42910000   CALL <JMP.&MFC42.#4278_?Mid@CString@@QBE?AV1@HH@Z>  ;  取子串，从 0xC 开始，3 个字符

00457E80  .| 8985 08FFFFFF MOV DWORD PTR SS:[EBP-F8],EAX       ;  保存结果

00457E86  .| 8B95 08FFFFFF MOV EDX,DWORD PTR SS:[EBP-F8]

00457E8C  .| 8995 04FFFFFF MOV DWORD PTR SS:[EBP-FC],EDX

00457E92  .| C645 FC 0C    MOV BYTE PTR SS:[EBP-4],0C

00457E96  .| 8B85 04FFFFFF MOV EAX,DWORD PTR SS:[EBP-FC]

00457E9C  .| 50            PUSH EAX

00457E9D  .| 8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

00457EA0  .| E8 07860000   CALL <JMP.&MFC42.#858_??4CString@@QAEABV0@ABV0@@Z>

00457EA5  .| C645 FC 0B    MOV BYTE PTR SS:[EBP-4],0B

00457EA9  .| 8D8D 3CFFFFFF LEA ECX,DWORD PTR SS:[EBP-C4]

00457EAF  .| E8 E6850000   CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>

00457EB4  .| 8D4D B8       LEA ECX,DWORD PTR SS:[EBP-48]

00457EB7  .| 51            PUSH ECX                             ; code[C,3]子串

00457EB8  .| 8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]        ;

00457EBB  .| 52            PUSH EDX                             ; 加和结果转换成的字符串

00457EBC  .| E8 2F96FBFF   CALL CDRWIN5.004114F0               ; 比较

00457EC1  .| 25 FF000000   AND EAX,0FF

00457EC6  .| 85C0          TEST EAX,EAX                         ; 两者必须相等

00457EC8  .| 74 76         JE SHORT CDRWIN5.00457F40

00457ECA  .| C785 38FFFFFF >MOV DWORD PTR SS:[EBP-C8],7         ；否则返回 7 失败

.................................

00457F3B  .| E9 A7060000   JMP CDRWIN5.004585E7

00457F40  |> 8D4D D0       LEA ECX,DWORD PTR SS:[EBP-30]        ；

00457F43  .| E8 5E850000   CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>

00457F48  .| C645 FC 0D    MOV BYTE PTR SS:[EBP-4],0D

00457F4C  .| 6A 03         PUSH 3                               ;

00457F4E  .| 8D4D 08       LEA ECX,DWORD PTR SS:[EBP+8]         ;

00457F51  .| E8 1A80FCFF   CALL CDRWIN5.0041FF70               ; 取 code[3]

00457F56  .| 50            PUSH EAX
```



```
00457F57  |. 8D4D D0         LEA ECX,DWORD PTR SS:[EBP-30]  

00457F5A  |. E8 69900000     CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z>  

00457F5F  |. 8D4D D0         LEA ECX,DWORD PTR SS:[EBP-30]                   ;   

00457F62  |. E8 09B2FAFF     CALL CDRWIN5.00403170  

00457F67  |. 50              PUSH EAX                                       ;   

00457F68  |. FF15 948A4600   CALL DWORD PTR DS:[<&MSVCRT.atoi>]            ; 转成数值  ->必须是阿拉伯数字  

00457F6E  |. 83C4 04         ADD ESP,4                                      ;   

00457F71  |. 8945 94         MOV DWORD PTR SS:[EBP-6C],EAX  

00457F74  |. 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00457F78  |. 0F85 C0000000   JNZ CDRWIN5.0045803E  

00457F7E  |. 6A 03           PUSH 3                                         ;   

00457F80  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

00457F83  |. E8 E87FFCFF     CALL CDRWIN5.0041FF70                          ;   

00457F88  |. 0FBEC8          MOVSX ECX,AL  

00457F8B  |. 83F9 30         CMP ECX,30  

00457F8E  |. 0F84 AA000000   JE CDRWIN5.0045803E  

.........................  

0045803E  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00458042  |. 75 1B           JNZ SHORT CDRWIN5.0045805F  

.........................  

0045805F  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00458063  |. 75 13          JNZ SHORT CDRWIN5.00458078  

.........................  

00458078  |> 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

0045807B  |. E8 26840000     CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>  

00458080  |. C645 FC 0E      MOV BYTE PTR SS:[EBP-4],0E  

00458084  |. 6A 01           PUSH 1                                         ;   

00458086  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

00458089  |. E8 E27EFCFF     CALL CDRWIN5.0041FF70                          ; 取 code[1]  

0045808E  |. 50              PUSH EAX  

0045808F  |. 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

00458092  |. E8 318F0000     CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z> ;   

00458097  |. 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

0045809A  |. E8 D1B0FAFF     CALL CDRWIN5.00403170  

0045809F  |. 50              PUSH EAX                                       ;   

004580A0  |. FF15 948A4600   CALL DWORD PTR DS:[<&MSVCRT.atoi>]            ; 也必须是阿拉伯数字  

004580A6  |. 83C4 04         ADD ESP,4                                      ;   

004580A9  |. 8945 C4         MOV DWORD PTR SS:[EBP-3C],EAX  

004580AC  |. 837D C4 00      CMP DWORD PTR SS:[EBP-3C],0  

004580B0  |. 0F85 CC000000   JNZ CDRWIN5.00458182  

004580B6  |. 6A 01           PUSH 1                                         ; /Arg1 = 00000001  

004580B8  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ; |  

004580BB  |. E8 B07EFCFF     CALL CDRWIN5.0041FF70                          ; \CDRWIN5.0041FF70  

004580C0  |. 0FBEC8          MOVSX ECX,AL  

004580C3  |. 83F9 30         CMP ECX,30  

004580C6  |. 0F84 B6000000   JE CDRWIN5.00458182  

.........................  

00458182  |> 837D C4 00      CMP DWORD PTR SS:[EBP-3C],0  

00458186  |. 75 1B           JNZ SHORT CDRWIN5.004581A3  

........................  

004581A3  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

004581A7  |. 75 13          JNZ SHORT CDRWIN5.004581BC  

004581A9  |. 6A 01           PUSH 1                                         ; /Arg1 = 00000001  

004581AB  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ; |  

004581AE  |. E8 BD7DFCFF     CALL CDRWIN5.0041FF70                          ; \CDRWIN5.0041FF70  

004581B3  |. 0FBEC0          MOVSX EAX,AL  

004581B6  |. 83E8 37         SUB EAX,37  

004581B9  |. 8945 C4         MOV DWORD PTR SS:[EBP-3C],EAX                 ;   

004581BC  |> 8B4D C4         MOV ECX,DWORD PTR SS:[EBP-3C]                 ；code [1]  

004581BF  |. C1E1 04         SHL ECX,4                                      ; 乘以 16  

004581C2  |. 894D C4         MOV DWORD PTR SS:[EBP-3C],ECX  

004581C5  |. 8B55 C4         MOV EDX,DWORD PTR SS:[EBP-3C]                 ；code[3]  

004581C8  |. 0355 94         ADD EDX,DWORD PTR SS:[EBP-6C]                 ;加和  

004581CB  |. 8955 C4         MOV DWORD PTR SS:[EBP-3C],EDX  

004581CE  |. 8B45 C4         MOV EAX,DWORD PTR SS:[EBP-3C]  

004581D1  |. 25 07000080     AND EAX,80000007                              ;  测试结果的后 3 位是否都为 0  

004581D6  |. 79 05           JNS SHORT CDRWIN5.004581DD                   ；由于 code[1]对应的数字已经乘以 16，它不影响这个测试，  

004581D8  |. 48              DEC EAX                                      ；主要是看 code[3]，因此实际上 code[3]只能为 0 或 8   

004581D9  |. 83C8 F8         OR EAX,FFFFFFF8  

004581DC  |. 40              INC EAX  

004581DD  |> 85C0            TEST EAX,EAX                                  ;   

004581DF  |. 0F84 8E000000   JE CDRWIN5.00458273                          ；若后 3 位不全为 0，  

004581E5  |. C785 2CFFFFFF   >MOV DWORD PTR SS:[EBP-D4],0A                ；返回 0A 失败  

............................  

00458268  |. 8B85 2CFFFFFF   MOV EAX,DWORD PTR SS:[EBP-D4]  

0045826E  |. E9 74030000     JMP CDRWIN5.004585E7  

00458273  |> C745 D8 000000>MOV DWORD PTR SS:[EBP-28],0  

0045827A  |. 6A 12           PUSH 12                                       ; 取 code[0x12]  

0045827C  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045827F  |. E8 EC7CFCFF     CALL CDRWIN5.0041FF70                         ;   

00458284  |. 0FBEF0          MOVSX ESI,AL  

00458287  |. 6A 13           PUSH 13                                       ; code[0x13]  

00458289  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045828C  |. E8 DF7CFCFF     CALL CDRWIN5.0041FF70                         ;   

00458291  |. 0FBED0          MOVSX EDX,AL  

00458294  |. 03F2            ADD ESI,EDX                                    ;  加和  

00458296  |. 6A 14           PUSH 14                                       ; code[0x14]  

00458298  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045829B  |. E8 D07CFCFF     CALL CDRWIN5.0041FF70                         ;   

004582A0  |. 0FBEC0          MOVSX EAX,AL  

004582A3  |. 03F0            ADD ESI,EAX                                    ;  加和  

004582A5  |. 6A 15           PUSH 15                                       ; code[0x15]  

004582A7  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

004582AA  |. E8 C17CFCFF     CALL CDRWIN5.0041FF70                         ;   

004582AF  |. 0FBEC8          MOVSX ECX,AL  

004582B2  |. 034D D8         ADD ECX,DWORD PTR SS:[EBP-28]                 ;  [ebp-28] 存放结果  

004582B5  |. 03CE            ADD ECX,ESI                                    ；全部加和  

004582B7  |. 894D D8         MOV DWORD PTR SS:[EBP-28],ECX                 ；  

004582BA  |. 8B55 D8         MOV EDX,DWORD PTR SS:[EBP-28]  

004582BD  |. 6BD2 0D         IMUL EDX,EDX,0D                               ；结果乘以 0D  

004582C0  |. 8955 D8         MOV DWORD PTR SS:[EBP-28],EDX  

004582C3  |. 8B45 D8         MOV EAX,DWORD PTR SS:[EBP-28]  

004582C6  |. 99              CDQ                                           ;符号扩展为 4 字在 edx:eax 中  

004582C7  |. B9 0A000000     MOV ECX,0A  

004582CC  |. F7F9            IDIV ECX  

004582CE  |. 8955 C8         MOV DWORD PTR SS:[EBP-38],EDX                ;  除以 0A  

004582D1  |. 8B55 C8         MOV EDX,DWORD PTR SS:[EBP-38]                ;   

004582D4  |. 83C2 30         ADD EDX,30                                    ;  余数加 0x30 变回字符  

004582D7  |. 8955 C8         MOV DWORD PTR SS:[EBP-38],EDX  
```



# 汇编代码片段

```
004582DA  |. 6A 16                 PUSH 16                                            ;
004582DC  |. 8D4D 08               LEA ECX,DWORD PTR SS:[EBP+8]                        ;
004582DF  |. E8 8C7CFCFF           CALL CDRWIN5.0041FF70                               ; 取 code[0x16]
004582E4  |. 0FBEC0                MOVSX EAX,AL
004582E7  |. 3B45 C8               CMP EAX,DWORD PTR SS:[EBP-38]
004582EA  |. 0F84 8E000000         JE CDRWIN5.0045837E                                  ; code[0x16]必须等于上述算出的字符
004582F0  |. C785 28FFFFFF        >MOV DWORD PTR SS:[EBP-D8],0B                         ; 否则返回 0B 失败

..........................
00458373  |. 8B85 28FFFFFF        MOV EAX,DWORD PTR SS:[EBP-D8]
00458379  |. E9 69020000          JMP CDRWIN5.004585E7
0045837E  |> 6A 02                PUSH 2                                                 ; code[2]
00458380  |. 8D4D 08              LEA ECX,DWORD PTR SS:[EBP+8]                          ;
00458383  |. E8 E87BFCFF          CALL CDRWIN5.0041FF70                                 ;
00458388  |. 0FBED0               MOVSX EDX,AL
0045838B  |. 83FA 4D              CMP EDX,4D                                             ;  'M'
0045838E  |. 0F85 8E000000        JNZ CDRWIN5.00458422                                   ;  code[2] 不能是 'M'
00458394  |. C785 24FFFFFF       >MOV DWORD PTR SS:[EBP-DC],10                           ; 否则返回 0x10 失败

.........................
00458417  |. 8B85 24FFFFFF        MOV EAX,DWORD PTR SS:[EBP-DC]
0045841D  |. E9 C5010000         JMP CDRWIN5.004585E7
00458422  |> 6A 00                PUSH 0                                                 ; code[0]
00458424  |. 8D4D 08              LEA ECX,DWORD PTR SS:[EBP+8]                          ;
00458427  |. E8 447BFCFF          CALL CDRWIN5.0041FF70                                 ;
0045842C  |. 0FBEC8               MOVSX ECX,AL
0045842F  |. 83F9 35              CMP ECX,35                                             ;  code[0] 必须是 '5'
00458432  |. 0F84 8E000000        JE CDRWIN5.004584C6
00458438  |. C785 20FFFFFF       >MOV DWORD PTR SS:[EBP-E0],11                           ; 否则返回 0x11 失败

...........................
004584BB  |. 8B85 20FFFFFF        MOV EAX,DWORD PTR SS:[EBP-E0]
004584C1  |. E9 21010000         JMP CDRWIN5.004585E7
004584C6  |> 837D CC 01          CMP DWORD PTR SS:[EBP-34],1                              ;  [ebp-34]是前面的标志，表示包含正确的字符
004584CA  |. 0F85 8E000000        JNZ CDRWIN5.0045855E                                   
004584D0  |. C785 1CFFFFFF       >MOV DWORD PTR SS:[EBP-E4],41A  <=================|      ;  返回 41a... 终于找到我们要的东东了！
004584DA  |. C645 FC 0D          MOV BYTE PTR SS:[EBP-4],0D                             |
004584DE  |. 8D4D C0             LEA ECX,DWORD PTR SS:[EBP-40]                          |
004584E1  |. E8 B47F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
004584E6  |. C645 FC 0B          MOV BYTE PTR SS:[EBP-4],0B                             |
004584EA  |. 8D4D D0             LEA ECX,DWORD PTR SS:[EBP-30]                          |
004584ED  |. E8 A87F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |  
004584F2  |. C645 FC 0A          MOV BYTE PTR SS:[EBP-4],0A                             |
004584F6  |. 8D4D B8             LEA ECX,DWORD PTR SS:[EBP-48]                          |
004584F9  |. E8 9C7F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
004584FE  |. C645 FC 09          MOV BYTE PTR SS:[EBP-4],9                              |
00458502  |. 8D4D A4             LEA ECX,DWORD PTR SS:[EBP-5C]                          |
00458505  |. E8 907F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
0045850A  |. C645 FC 07          MOV BYTE PTR SS:[EBP-4],7                              |
0045850E  |. 8D4D D4             LEA ECX,DWORD PTR SS:[EBP-2C]                          |
00458511  |. E8 847F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
00458516  |. C645 FC 02          MOV BYTE PTR SS:[EBP-4],2                              |
0045851A  |. 8D4D AC             LEA ECX,DWORD PTR SS:[EBP-54]                          |
0045851D  |. E8 787F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
00458522  |. C645 FC 01          MOV BYTE PTR SS:[EBP-4],1                              |
00458526  |. 8D4D 98             LEA ECX,DWORD PTR SS:[EBP-68]                          |
00458529  |. E8 6C7F0000         CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |
0045852E  |. C645 FC 00          MOV BYTE PTR SS:[EBP-4],0                              |
00458532  |. 68 9A044600        PUSH <JMP.&MFC42.#800_??1CString@@QAE@XZ>              |    ;  Entry address
00458537  |. 6A 05                PUSH 5                                                  |
00458539  |. 6A 04                PUSH 4                                                  |
0045853B  |. 8D45 DC             LEA EAX,DWORD PTR SS:[EBP-24]                           |
0045853E  |. 50                  PUSH EAX                                               |
0045853F  |. E8 DC8B0000         CALL CDRWIN5.00461120                                   |
00458544  |. C745 FC FFFFFF     >MOV DWORD PTR SS:[EBP-4],-1                             |
0045854B  |. 8D4D 08             LEA ECX,DWORD PTR SS:[EBP+8]                             |
0045854E  |. E8 477F0000        CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>                |
00458553  |. 8B85 1CFFFFFF        MOV EAX,DWORD PTR SS:[EBP-E4] <================|
00458559  |. E9 89000000         JMP CDRWIN5.004585E7

........................
004585E7  |> 8B4D F4             MOV ECX,DWORD PTR SS:[EBP-C]
004585EA  |. 64:890D 000000     >MOV DWORD PTR FS:[0],ECX
004585F1  |. 5E                  POP ESI
004585F2  |. 8BE5               MOV ESP,EBP
004585F4  |. 5D                  POP EBP
004585F5  \. C3                  RETN
```

## 总结

### 注册码黑名单

`5BS8X-CCZDR-59B88-CBCK5-DGTSQ`

`59S0@-5CRCD-57647-UPMY3-CEVRF`

`5ES8D-D5CRT-55606-D1CL0-BNAZL`

`59S0C-RDQMN-57B22-6U6Y6-7TQDU`

`5DC8R-LASH8-57228-LASH8-DLASH`

### 注册机黑名单

以 `5CC8R` 或 `5CD8R` 开头

===============================

注册码必须有 29 个字符长，（从 0 开始编号）第 5，11，17 和 23 个字符必须是 `-`

由此把注册码分为 5 个部分

注册码前 11 个字符（第一和第二部分）必须含有 `C`, `D`, `R`, 但不能有 `P`

整个注册码不能有 `O`, `I`

第二部分和第三部分共 10 个字符的 ascii 码加和必须得到一个偶数

第 16 个字符（第三部分的最后一个字符）必须是一个阿拉伯数字，其值加上 5 再对 10 取余，再加上 `0x41`, 必须得到第 24 个字符

第一，二，四，五部分共 20 个字符的 ascii 码加和得到的 16 进制结果必须对应第三部分开始的三个字符

第 1 个字符必须为数字，第 3 个字符必须为 `0` 或 `8`

第 12，13，14 和 15 个字符的 ascii 相加结果乘以 13 再除以 10，余数必须等于第 22 个字符表示的数

第 0 个字符必须为 `5`

第 2 个字符不能为 `M`

一个可用的注册码： `57R8D-7C345-4B187-67896-CDCBA`

`4dd466d0e1e0cc123afbe504da2462ca`
