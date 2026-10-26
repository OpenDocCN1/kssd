

```text
  014010E2  |.  8B3D 0CB04001   MOV EDI,DWORD PTR DS:[<&KERNEL32.VirtualAlloc>] ;  断在这
  014010E8  |.  6A 40           PUSH 40                                         ; /Protect = PAGE\_EXECUTE\_READWRITE
  014010EA  |.  68 00100000     PUSH 1000                                       ; |AllocationType = MEM\_COMMIT
  014010EF  |.  8BF1            MOV ESI,ECX                                     ; |
  014010F1  |.  68 00100000     PUSH 1000                                       ; |Size = 1000 (4096.)
  014010F6  |.  6A 00           PUSH 0                                          ; |Address = NULL
  014010F8  |.  C746 04 0000000>MOV DWORD PTR DS:[ESI+4],0                      ; |
  014010FF  |.  C746 0C 0000000>MOV DWORD PTR DS:[ESI+C],0                      ; |
  01401106  |.  FFD7            CALL EDI                                        ; \VirtualAlloc
  01401108  |.  85C0            TEST EAX,EAX                                    ;  地址00A60000
```

管它，让它去解密代码去，继续F9，来到这：

```text
  01401150  /$  56              PUSH ESI
  01401151  |.  8BF1            MOV ESI,ECX
  01401153  |.  8B06            MOV EAX,DWORD PTR DS:[ESI]
  01401155  |.  85C0            TEST EAX,EAX
  01401157  |.  57              PUSH EDI
  01401158  |.  8B3D 08B04001   MOV EDI,DWORD PTR DS:[<&KERNEL32.VirtualFree>]  ;  kernel32.VirtualFree
  0140115E  |.  74 0A           JE SHORT Rcomclt.0140116A
  01401160  |.  68 00800000     PUSH 8000                                       ; /FreeType = MEM\_RELEASE
  01401165  |.  6A 00           PUSH 0                                          ; |Size = 0
  01401167  |.  50              PUSH EAX                                        ; |起始地址A60000处
  01401168  |.  FFD7            CALL EDI                                        ; \VirtualFree
```

原来释放了一块内存，后面我们没断点了，不要瞎来，F8慢慢跟：

```text
  01401177  |.  68 00800000       PUSH 8000
  0140117C  |.  6A 00             PUSH 0
  0140117E  |.  50                PUSH EAX                                        ;  内存地址00A70000
  0140117F  |.  FFD7              CALL EDI                                        ;  调用VirtualFree函数
  01401181  |>  5F                POP EDI
  01401182  |.  C746 08 00000000  MOV DWORD PTR DS:[ESI+8],0
  01401189  |.  5E                POP ESI                                         ;  我们来到的地方
  0140118A  \.  C3                RETN
```

看了`00A70000`处的数据，原来是 `KERNEL32.DLL` 什么的，呵呵，毁尸灭迹啊！经过上面那个 `RETN`，我们来到这：

```text
  01401909   .  C745 E0 00000000  MOV DWORD PTR SS:[EBP-20],0                     ;  来到这里
  01401910   .  EB 09             JMP SHORT Rcomclt.0140191B
  01401912   >  8B4D E0           MOV ECX,DWORD PTR SS:[EBP-20]
  01401915   .  83C1 01           ADD ECX,1
  01401918   .  894D E0           MOV DWORD PTR SS:[EBP-20],ECX
  0140191B   >  8B55 E0           MOV EDX,DWORD PTR SS:[EBP-20]
  0140191E   .  3B55 AC           CMP EDX,DWORD PTR SS:[EBP-54]                   ;  .text段
  01401921   .  0F83 C4000000     JNB Rcomclt.014019EB
  01401927   .  BA 38004101       MOV EDX,Rcomclt.01410038                        ;  mycol
  0140192C   .  8B45 E0           MOV EAX,DWORD PTR SS:[EBP-20]
  0140192F   .  6BC0 18           IMUL EAX,EAX,18
  01401932   .  8B4D A8           MOV ECX,DWORD PTR SS:[EBP-58]                   ;  段名称
  01401935   .  03C8              ADD ECX,EAX
  01401937   .  E8 14010000       CALL Rcomclt.01401A50                           ;  这里把段名称与上面的mycol比较
```

再来到下面：

```text
  0140198B   > /8B55 F4           MOV EDX,DWORD PTR SS:[EBP-C]
  0140198E   . |83C2 01           ADD EDX,1
  01401991   . |8955 F4           MOV DWORD PTR SS:[EBP-C],EDX
  01401994   > |8B45 F4           MOV EAX,DWORD PTR SS:[EBP-C]
  01401997 > . |3B85 A8FAFFFF     CMP EAX,DWORD PTR SS:[EBP-558]                  ;  代码段大小，24CC
  0140199D   . |73 45             JNB SHORT Rcomclt.014019E4
  0140199F   . |8B4D F4           MOV ECX,DWORD PTR SS:[EBP-C]
  014019A2   . |8B15 14284101     MOV EDX,DWORD PTR DS:[1412814]                  ;  内存地址00A56000
  014019A8   . |8B048A            MOV EAX,DWORD PTR DS:[EDX+ECX\*4]
  014019AB   . |0305 10284101     ADD EAX,DWORD PTR DS:[1412810]                  ;  内存地址009A0000
  014019B1   . |8B4D F4           MOV ECX,DWORD PTR SS:[EBP-C]
  014019B4   . |8B15 14284101     MOV EDX,DWORD PTR DS:[1412814]
  014019BA   . |89048A            MOV DWORD PTR DS:[EDX+ECX\*4],EAX
  014019BD   . |8B45 F4           MOV EAX,DWORD PTR SS:[EBP-C]
  014019C0   . |8B0D 14284101     MOV ECX,DWORD PTR DS:[1412814]
  014019C6   . |8B1481            MOV EDX,DWORD PTR DS:[ECX+EAX\*4]
  014019C9   . |8995 A4FAFFFF     MOV DWORD PTR SS:[EBP-55C],EDX
  014019CF   . |8B85 A4FAFFFF     MOV EAX,DWORD PTR SS:[EBP-55C]
  014019D5   . |8B08              MOV ECX,DWORD PTR DS:[EAX]
  014019D7   . |034D D8           ADD ECX,DWORD PTR SS:[EBP-28]
  014019DA   . |8B95 A4FAFFFF     MOV EDX,DWORD PTR SS:[EBP-55C]
  014019E0   . |890A              MOV DWORD PTR DS:[EDX],ECX
  014019E2   .^\EB A7             JMP SHORT Rcomclt.0140198B
```

不管它，我知道它是往内存地址`00A56000`开始处的空间写东西就行了，要用到这里我再研究。在`01401997`地址那条指令上设个条件断点，`EAX==24CC`，F9继续，条件断点断下后，我们F8，到这：

```text
  01401A0E   > \68 00800000       PUSH 8000                                       ; /FreeType = MEM\_RELEASE
  01401A13   .  6A 00             PUSH 0                                          ; |Size = 0
  01401A15   .  8B4D E8           MOV ECX,DWORD PTR SS:[EBP-18]                   ; |
  01401A18   .  51                PUSH ECX                                        ; |Address = 008D0000
  01401A19   .  FF15 08B04001     CALL DWORD PTR DS:[<&KERNEL32.VirtualFree>]     ; \又要毁尸灭迹了
  01401A1F   .  8B15 10284101     MOV EDX,DWORD PTR DS:[1412810]                  ;  9A0000，解密后代码基址，以后要用
  01401A25   .  0395 D8FEFFFF     ADD EDX,DWORD PTR SS:[EBP-128]
  01401A2B   .  8955 B8           MOV DWORD PTR SS:[EBP-48],EDX
  01401A2E   .  8B45 B8           MOV EAX,DWORD PTR SS:[EBP-48]
  01401A31   .- FFE0              JMP EAX                                         ;  我这EAX是00A1D855，要跳到解完密的代码那执行了
```

继续F8，一看全是PE文件初始化的一些代码。向下翻翻代码，F4到这：

```text
  00A1D92F    50                  PUSH EAX                                        ; F4运行到这，看看还要干什么
  00A1D930    E8 3B18F9FF         CALL 009AF170                                   ; 跟进去
```

F4到 `00A1D92F` 的时候发现 `EAX` 就是原程序的镜像基址`01400000`，跟进`00A1D930`地址处的那个`CALL`，看看干了些什么，来到下面：

```text
  009AF1EB    68 F4A5A400         PUSH 0A4A5F4                                    ; UNICODE "\*.lng\_rcc"
  009AF1F0    68 8EA00000         PUSH 0A08E                                      ; 要开始判断是否有语言文件了
```

继续，到这：

```text
  009AF263    E8 A8F60000         CALL 009BE910
  009AF268    85C0                TEST EAX,EAX
  009AF26A    0F84 58040000       JE 009AF6C8                                     ; 这里一跳就完蛋了，EAX不能为0
```

在`009AF268`地址处这条指令检测`EAX`时，我先修改一下寄存器，让`EAX`等于1，下面那条指令它肯定不会跳的。继续，来到这：

```text
  009AF27F    E8 BCFA0000         CALL 009BED40
  009AF284    85C0                TEST EAX,EAX
  009AF286    0F84 3C040000       JE 009AF6C8                                     ; 这里一跳也完蛋，EAX不能为0
```

同样在执行到`009AF284`地址处的指令时把`EAX`改为1，顺便看一下地址 `009AF6C8` 处都有什么东西：

```text
  009AF6C8    68 30200000         PUSH 2030
  009AF6CD    68 44A8A400         PUSH 0A4A844                                    ; ASCII "Error"
  009AF6D2    68 24A8A400         PUSH 0A4A824                                    ; ASCII "Executable file is corrupted"
  009AF6D7    53                  PUSH EBX
  009AF6D8    FF15 FC23A300       CALL DWORD PTR DS:[A323FC]                      ; USER32.MessageBoxA
```

呵呵，到这就歇菜了。当场给你个出错对话框。我们上面都改了标志位，现在肯定不会跳到这了。分析过原程序，这里的`EAX`都是1。F9运行一下，呵呵，现在正常了。到此就要考虑了，不可能我每次都挂个调试器来修改标志位让程序运行啊。做个 `Loader`？人家运行你的汉化版之前还要先运行 `Loader` 来修改内存，多郁闷啊！还有种办法就是把解密算法全部搞清楚，写个程序，把解密后的代码修改了再加密回去，替换原来的资源。这。。。这也太让人抓狂了吧？这活我不干。还是找个简单的方法吧，我在程序跳到内存中已解密后的代码执行之前，让程序先把解密后的代码按我的要求修改一下，然后再让程序去执行，这样行吧？我只要把原来那两条 `TEST EAX,EAX` 指令改为 `INC EAX` 就行了，当然后面还要补个 `NOP` 来补足代码长度。`INC EAX` 再加上 `NOP` 机器码是 `4090`。现在剩下的问题就是在哪里接管程序，在什么地方写我们的补丁代码了。我现在先考虑这一段代码：

```text
  01401A1F   .  8B15 10284101     MOV EDX,DWORD PTR DS:[1412810]                  ;  9A0000，解密后代码镜像基址，以后要用
```

