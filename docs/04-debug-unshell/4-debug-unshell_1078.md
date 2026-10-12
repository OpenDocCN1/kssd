
# 手脱 The Enigma Protector1.55 默认保护

> 原文：[`www.pediy.com/kssd/pediy10/77905.html`](https://www.pediy.com/kssd/pediy10/77905.html)

```
00401DBA >   55             push ebp                 //OD 载入程序
00401DBB 8BEC             mov ebp,esp              //我们下 bp VirtualProtectEx
00401DBD 83C4 F0          add esp,-10              //下好断点我们 shift+f9 运行程序
00401DC0 B8 00104000      mov eax,notepad\_.00401000
00401DC5 E8 01000000      call notepad\_.00401DCB
00401DCA 9A 83C4108B E55> call far 5DE5:8B10C483
00401DD1 - E9 66890400    jmp notepad\_.0044A73C
00401DD6 0000             add byte ptr ds:[eax],al
```

**数据窗口**

```
00402000 004C1F78 notepad\_.004C1F78       //此时可明显感觉出 IAT 已经被加密处理
00402004 00B8B540                         //不管他我们取消断点后 alt+f9 返回程序
00402008 00000000
0040200C 73D98D67 mfc42.#4486_CWinApp::OnDDECommand
00402010 73D35EF1 mfc42.#2554_CWinApp::DoWaitCursor
00402014 73D497A0 mfc42.#2512_CWinApp::DoMessageBox
00402018 73D4224E mfc42.#5731_CWinApp::SaveAllModified
```

```
00B7302C  5D             pop ebp                //我们开始 F8 单步跟上吧
00B7302D  C2 1000        retn 10
00B73030  57             push edi
00B73031  5F             pop edi
00B73032  55             push ebp
00B73033  8D6C24 00      lea ebp,dword ptr ss:[esp]
```

```
004C25BE  5A             pop edx
004C25BF  5E             pop esi
004C25C0  5B             pop ebx
004C25C1  C3             retn                   //这里 retn 返回到下一段代码
004C25C2  8BC0           mov eax,eax
004C25C4  55             push ebp
```

```
004C9FD3  C643 28 01     mov byte ptr ds:[ebx+28],1    //继续 F8 单步跟上
004C9FD7  E8 D484FFFF    call notepad\_.004C24B0        //记住打断向上跳转，执行向下跳转
004C9FDC  C643 29 01     mov byte ptr ds:[ebx+29],1
004C9FE0  C643 2A 01     mov byte ptr ds:[ebx+2A],1
004C9FE4  E8 CBF2FFFF    call notepad\_.004C92B4
004C9FE9  E8 CAB7FAFF    call notepad\_.004757B8
```

```
004CA1BB  50             push eax                      //单步跟到这里时需要小心一些啦
004CA1BC  8B45 18        mov eax,dword ptr ss:[ebp+18]
004CA1BF  50             push eax
004CA1C0  E8 FF1CFFFF    call notepad\_.004BBEC4       //这个 call 我们需要 F7 跟进，他是一个远 CALL
004CA1C5  E8 7E84FFFF    call notepad\_.004C2648
```

**call 内代码**

```
004BBEC4  55             push ebp                      //我们可以单步 F8 向下冲啦
004BBEC5  8BEC           mov ebp,esp                    //同样注意我们需要打断向上跳转执行向下跳转
004BBEC7  83C4 F4        add esp,-0C
004BBECA  53             push ebx
004BBECB  56             push esi
004BBECC  57             push edi
```

```
004BC5A4  2B16           sub edx,dword ptr ds:[esi]    //经过一路慢长的 F8 我们来到了这里
004BC5A6  8BC6           mov eax,esi                   //此时单步脚步放慢一些吧，因为跳过去后代码会比较花
004BC5A8  E8 F38EFAFF    call notepad\_.004654A0
004BC5AD  8B45 FC        mov eax,dword ptr ss:[ebp-4]
004BC5B0  FFE0           jmp eax                       //这里跳向下一断代码，我们单步走吧
004BC5B2  5F             pop edi
```

```
00B6E030  E8 67CC94FF    call notepad\_.004BAC9C       //代码开始混乱起来
00B6E035  E9 04000000    jmp 00B6E03E                  //并且大家也可发现我们的地址已经被重定位啦
00B6E03A  D366 C5        shl dword ptr ds:[esi-3B],cl  //即我们的地址被加密处理
00B6E03D  3E:8905 90B74E0> mov dword ptr ds:[4EB790],eax //F8 单步小心的走
00B6E044  E9 04000000    jmp 00B6E04D
```

```
00B6E128  /E9 04000000   jmp 00B6E131
00B6E12D  |D274B7 F4     sal byte ptr ds:[edi+esi*4-C],cl
00B6E131 ^\0F85 D8FFFFFF  jnz 00B6E10F                //注意这个 JNZ 是向上跳转的
00B6E137  E9 04000000    jmp 00B6E140                  //我们在这行打断向上跳转的 JNZ
00B6E13C  1F             pop ds                         //继续 F8 单步向前走
00B6E13D  14 6A         adc al,6A
```

```
00B6E17F  FFE0           jmp eax                       //这里跳向我们 Stolen code 的开始
00B6E181  E9 04000000    jmp 00B6E18A                  //单步 F8 一步步入
00B6E186  61             popad
00B6E187  E0 56          loopdne short 00B6E1DF
00B6E189  B8 E9040000    mov eax,4E9
00B6E18E  00EA           add dl,ch
```

```
00B6DBD4  55             push ebp                       //呵呵，被抽了不少代码，直接大末尾全部是
00B6DBD5  8BEC           mov ebp,esp                    //但是这个壳有一点好，那就是会把所有抽取的存在某个地方
00B6DBD7  6A FF          push -1                        //我们在补 OEP 时需要注意把他的跳转改回来，跳向我们对应的地址
00B6DBD9  68 00254000    push 402500
00B6DBDE  68 86184000    push 401886                    ; jmp to msvcrt._except_handler3
00B6DBE3  64:8B05 0000000> mov eax,dword ptr fs:[0]
00B6DBEA  50             push eax
00B6DBEB  64:8925 0000000> mov dword ptr fs:[0],esp
00B6DBF2  83EC 68        sub esp,68
00B6DBF5  53             push ebx
00B6DBF6  56             push esi
00B6DBF7  57             push edi
00B6DBF8  89A5 E8FFFFFF  mov dword ptr ss:[ebp-18],esp
00B6DBFE  33DB           xor ebx,ebx
00B6DC00  899D FCFFFFFF  mov dword ptr ss:[ebp-4],ebx
00B6DC06  6A 02          push 2
00B6DC08  FF15 90214000  call dword ptr ds:[402190]    ; msvcrt.__set_app_type
00B6DC0E  59             pop ecx
00B6DC0F  810D 2C314000 F> or dword ptr ds:[40312C],FFFFFFFF
00B6DC19  810D 30314000 F> or dword ptr ds:[403130],FFFFFFFF
00B6DC23  FF15 8C214000  call dword ptr ds:[40218C]    ; msvcrt._p__fmode
00B6DC29  8B0D 20314000  mov ecx,dword ptr ds:[403120]
00B6DC2F  8908           mov dword ptr ds:[eax],ecx
00B6DC31  FF15 88214000  call dword ptr ds:[402188]    ; msvcrt._p__commode
00B6DC37  8B0D 1C314000  mov ecx,dword ptr ds:[40311C]
00B6DC3D  8908           mov dword ptr ds:[eax],ecx
00B6DC3F  8B05 84214000  mov eax,dword ptr ds:[402184] ; msvcrt._adjust_fdiv
00B6DC45  8B40 00        mov eax,dword ptr ds:[eax]
00B6DC48  8905 28314000  mov dword ptr ds:[403128],eax
00B6DC4E  E8 323C89FF    call notepad\_.00401885
00B6DC53  391D 40304000  cmp dword ptr ds:[403040],ebx
00B6DC59  0F85 0C000000  jnz 00B6DC6B                 //这里跳转变形，我们做个记号
00B6DC5F  68 82184000    push 401882
00B6DC64  FF15 80214000  call dword ptr ds:[402180]   ; msvcrt.__setusermatherr
00B6DC6A  59             pop ecx
00B6DC6B  E8 003C89FF    call notepad\_.00401870       //上面的 JNZ 跳到这里
00B6DC70  68 14304000    push 403014                   //我们补好 OEP 后，也需要把他修改成跳转到对应的地址
00B6DC75  68 10304000    push 403010                   //列如在 OEP 跳转是跳 3 行，那么我们就象下数三行和这样的对应就可以啦
00B6DC7A  E8 EB3B89FF    call notepad\_.0040186A       ; jmp to msvcrt._initterm
00B6DC7F  8B05 18314000  mov eax,dword ptr ds:[403118]
00B6DC85  8985 94FFFFFF  mov dword ptr ss:[ebp-6C],eax
00B6DC8B  8D45 94        lea eax,dword ptr ss:[ebp-6C]
00B6DC8E  50             push eax
00B6DC8F  FF35 14314000  push dword ptr ds:[403114]
00B6DC95  8D45 9C        lea eax,dword ptr ss:[ebp-64]
00B6DC98  50             push eax
00B6DC99  8D45 90        lea eax,dword ptr ss:[ebp-70]
00B6DC9C  50             push eax
00B6DC9D  8D45 A0        lea eax,dword ptr ss:[ebp-60]
00B6DCA0  50             push eax
00B6DCA1  FF15 78214000  call dword ptr ds:[402178]    ; msvcrt.__getmainargs
00B6DCA7  68 0C304000    push 40300C
00B6DCAC  68 00304000    push 403000
```



```
00B6DCB1    E8 B43B89FF     call notepad_.0040186A                   ; jmp to msvcrt._initterm

00B6DCB6    83C4 24         add esp,24

00B6DCB9    8B05 74214000   mov eax,dword ptr ds:[402174]           ; msvcrt._acmdln

00B6DCBF    8B70 00         mov esi,dword ptr ds:[eax]

00B6DCC2    89B5 8CFFFFFF   mov dword ptr ss:[ebp-74],esi

00B6DCC8    80BE 00000000 2>cmp byte ptr ds:[esi],22

00B6DCCF    0F85 6A000000   jnz 00B6DD3F                             //这个跳转也变形啦

00B6DCD5    46              inc esi

00B6DCD6    89B5 8CFFFFFF   mov dword ptr ss:[ebp-74],esi

00B6DCDC    8A06            mov al,byte ptr ds:[esi]

00B6DCDE    3AC3            cmp al,bl

00B6DCE0    0F84 09000000   je 00B6DCEF                              //这个 JE 也变形啦

00B6DCE6    80F8 22         cmp al,22

00B6DCE9  ^ 0F85 E6FFFFFF   jnz 00B6DCD5                             //这个 JNZ 也变形啦

00B6DCEF    80BE 00000000 2>cmp byte ptr ds:[esi],22

00B6DCF6    0F85 07000000   jnz 00B6DD03                             //这个 JNZ 也变形啦

00B6DCFC    46              inc esi

00B6DCFD    89B5 8CFFFFFF   mov dword ptr ss:[ebp-74],esi

00B6DD03    8A06            mov al,byte ptr ds:[esi]

00B6DD05    3AC3            cmp al,bl

00B6DD07    0F84 09000000   je 00B6DD16                              //这个 JE 也变形啦

00B6DD0D    80F8 20         cmp al,20

00B6DD10  ^ 0F86 E6FFFFFF   jbe 00B6DCFC

00B6DD16    899D D0FFFFFF   mov dword ptr ss:[ebp-30],ebx

00B6DD1C    8D45 A4         lea eax,dword ptr ss:[ebp-5C]

00B6DD1F    50              push eax

00B6DD20    FF15 04204000   call dword ptr ds:[402004]

00B6DD26    F685 D0FFFFFF 0>test byte ptr ss:[ebp-30],1

00B6DD2D    0F84 25000000   je 00B6DD58                              //还有这个 JE 也变形啦

00B6DD33    0FB785 D4FFFFFF movzx eax,word ptr ss:[ebp-2C]

00B6DD3A    E9 1C000000     jmp 00B6DD5B                             //这个 JMP 也变形啦

00B6DD3F    80BE 00000000 2>cmp byte ptr ds:[esi],20                

00B6DD46  ^ 0F86 B7FFFFFF   jbe 00B6DD03                             //这个 JBE 也变形啦

00B6DD4C    46              inc esi

00B6DD4D    89B5 8CFFFFFF   mov dword ptr ss:[ebp-74],esi

00B6DD53  ^ E9 E7FFFFFF     jmp 00B6DD3F                             //这个 JMP 也变形啦

00B6DD58    6A 0A           push 0A

00B6DD5A    58              pop eax

00B6DD5B    50              push eax

00B6DD5C    56              push esi

00B6DD5D    53              push ebx

00B6DD5E    53              push ebx

00B6DD5F    FF15 00204000   call dword ptr ds:[402000]               ; notepad_.004C1F78

00B6DD65    50              push eax

00B6DD66    E8 273B89FF     call notepad_.00401892

00B6DD6B    8985 98FFFFFF   mov dword ptr ss:[ebp-68],eax

00B6DD71    50              push eax

00B6DD72    FF15 70214000   call dword ptr ds:[402170]               ; msvcrt.exit

00B6DD78    8B45 EC         mov eax,dword ptr ss:[ebp-14]

00B6DD7B    8B48 00         mov ecx,dword ptr ds:[eax]

00B6DD7E    8B49 00         mov ecx,dword ptr ds:[ecx]

00B6DD81    898D 88FFFFFF   mov dword ptr ss:[ebp-78],ecx

00B6DD87    50              push eax

00B6DD88    51              push ecx

00B6DD89    E8 D63A89FF     call notepad_.00401864                   ; jmp to msvcrt._XcptFilter

00B6DD8E    59              pop ecx

00B6DD8F    59              pop ecx

00B6DD90    C3              retn                                     //这里是 Stolen code 的结束

00B6DD91  - E9 BC3A89FF     jmp notepad_.00401852                   //这里跳向我们的伪 OEP

00B6DD96    A5              movs dword ptr es:[edi],dword ptr ds:[es>  //我们在上面的 JMP 右键新建 EIP，千万不能单步下来会出错的

00B6DD97    3307            xor eax,dword ptr ds:[edi]               //我们直接右键在上面一句 JMP 新建 EIP

00B6DD99    A0 4822CBD5     mov al,byte ptr ds:[D5CB2248]            //好啦，我们在 2 进制复制一份被抽取的 OEP，如果体力好的话也可逐行填加

00B6DD9E    07              pop es                                     //新建好 EIP 后我们单步一步 F8

00B6DD9F    B8 73A1357F     mov eax,7F35A173                         //为了不增加脱文字数，我这里就不复制 2 进制代码啦

━━━━━━━━━━━━━━━━━━━━━━━━━━

00401852    8B65 E8         mov esp,dword ptr ss:[ebp-18]            ; notepad_.004EFC38               //来到了这里，这里就是我们的伪 OEP

00401855    FF75 88         push dword ptr ss:[ebp-78]                                   //向上拖动滚动条找段首

00401858    FF15 AC214000   call dword ptr ds:[4021AC]               ; msvcrt._exit

0040185E  - FF25 A0214000   jmp dword ptr ds:[4021A0]                ; msvcrt.__dllonexit

00401864  - FF25 6C214000   jmp dword ptr ds:[40216C]                ; msvcrt._XcptFilter

0040186A  - FF25 7C214000   jmp dword ptr ds:[40217C]                ; msvcrt._initterm

━━━━━━━━━━━━━━━━━━━━━━━━━━

004016F5    C3              retn

004016F6  - FF25 A4214000   jmp dword ptr ds:[4021A4]                ; msvcrt.__CxxFrameHandler        //这里就是我们的段首

004016FC    CC              int3                                     //而这四句 int3 是不用 nop 掉的，因为他是 MFC 写的

004016FD    CC              int3                                     //要是不熟悉的话可以去找几个对比下

004016FE    CC              int3                                     //那我们的 OEP 就是 00401700

004016FF    CC              int3                                     //大家应该比对 98 记事本还熟悉，这个是我们 52 脱壳练习的程序

00401700    D887 B3BDE865   fadd dword ptr ds:[edi+65E8BDB3]         //从这里开始向下拉，拉到我们的伪 OEP 处的上一句，全部 NOP 掉

00401706    1213            adc dl,byte ptr ds:[ebx]

00401708    EC              in al,dx

00401709    A2 C9F50C07     mov byte ptr ds:[70CF5C9],al

━━━━━━━━━━━━━━━━━━━━━━━━━━

00401700    D887 B3BDE865   fadd dword ptr ds:[edi+65E8BDB3]         //由于代码过多，我就不全部复制啦，中间部分省略过

00401706    1213            adc dl,byte ptr ds:[ebx]                 //从 00401700-----0040184D 的代码全部 NOP 掉

00401708    EC              in al,dx                                 //切记不要 NOP 多啦，不然程序会出错的

00401709    A2 C9F50C07     mov byte ptr ds:[70CF5C9],al

0040170E    91              xchg eax,ecx

0040170F    8691 B220D6DF   xchg byte ptr ds:[ecx+DFD620B2],dl

00401715    9C              pushfd

................................................

00401846    4A              dec edx

00401847    DA12            ficom dword ptr ds:[edx]

00401849    06              push es

0040184A    6E              outs dx,byte ptr es:[edi]

0040184B    1C 94           sbb al,94

0040184D    15 8FD31B5A     adc eax,5A1BD38F                         //NOP 到这一句就可以啦

━━━━━━━━━━━━━━━━━━━━━━━━━━

00B6DBD4    55              push ebp                                   //这个补跳转时其实也很容易的，大家只要留心就好

00B6DBD5    8BEC            mov ebp,esp                                //这里补跳转我们可以把我们复制的代码进行比对跳了几行就把对应跳转跳向几行

00B6DBD7    6A FF           push -1                                    //补跳转时一定要注意，千万别补错，不然程序会报错或者直接运行不起来

00B6DBD9    68 00254000     push 402500

00B6DBDE    68 86184000     push 401886                                ; jmp to msvcrt._except_handler3

00B6DBE3    64:8B05 0000000>mov eax,dword ptr fs:[0]

00B6DBEA    50              push eax

00B6DBEB    64:8925 0000000>mov dword ptr fs:[0],esp

00B6DBF2    83EC 68         sub esp,68

00B6DBF5    53              push ebx

00B6DBF6    56              push esi

00B6DBF7    57              push edi

00B6DBF8    89A5 E8FFFFFF   mov dword ptr ss:[ebp-18],esp

00B6DBFE    33DB            xor ebx,ebx

00B6DC00    899D FCFFFFFF   mov dword ptr ss:[ebp-4],ebx

00B6DC06    6A 02           push 2

00B6DC08    FF15 90214000   call dword ptr ds:[402190]                ; msvcrt.__set_app_type

00B6DC0E    59              pop ecx

00B6DC0F    810D 2C314000 F>or dword ptr ds:[40312C],FFFFFFFF

00B6DC19    810D 30314000 F>or dword ptr ds:[403130],FFFFFFFF

00B6DC23    FF15 8C214000   call dword ptr ds:[40218C]                ; msvcrt.__p__fmode
```



```asm
00B6DC29    8B0D 20314000    mov ecx,dword ptr ds:[403120]  

00B6DC2F    8908             mov dword ptr ds:[eax],ecx  

00B6DC31    FF15 88214000    call dword ptr ds:[402188]             ; msvcrt.__p__commode  

00B6DC37    8B0D 1C314000    mov ecx,dword ptr ds:[40311C]  

00B6DC3D    8908             mov dword ptr ds:[eax],ecx  

00B6DC3F    8B05 84214000    mov eax,dword ptr ds:[402184]           ; msvcrt._adjust_fdiv  

00B6DC45    8B40 00          mov eax,dword ptr ds:[eax]  

00B6DC48    8905 28314000    mov dword ptr ds:[403128],eax  

00B6DC4E    E8 323C89FF      call notepad_.00401885  

00B6DC53    391D 40304000    cmp dword ptr ds:[403040],ebx  

00B6DC59    0F85 0C000000    jnz 00B6DC6B                            //这个 JNZ 对应 00401783  

00B6DC5F    68 82184000      push 401882  

00B6DC64    FF15 80214000    call dword ptr ds:[402180]             ; msvcrt.__setusermatherr  

00B6DC6A    59               pop ecx  

00B6DC6B    E8 003C89FF      call notepad_.00401870                 

00B6DC70    68 14304000      push 403014                              

00B6DC75    68 10304000      push 403010                              

00B6DC7A    E8 EB3B89FF      call notepad_.0040186A                 ; jmp to msvcrt._initterm  

00B6DC7F    8B05 18314000    mov eax,dword ptr ds:[403118]  

00B6DC85    8985 94FFFFFF    mov dword ptr ss:[ebp-6C],eax  

00B6DC8B    8D45 94          lea eax,dword ptr ss:[ebp-6C]  

00B6DC8E    50               push eax  

00B6DC8F    FF35 14314000    push dword ptr ds:[403114]  

00B6DC95    8D45 9C          lea eax,dword ptr ss:[ebp-64]  

00B6DC98    50               push eax  

00B6DC99    8D45 90          lea eax,dword ptr ss:[ebp-70]  

00B6DC9C    50               push eax  

00B6DC9D    8D45 A0          lea eax,dword ptr ss:[ebp-60]  

00B6DCA0    50               push eax  

00B6DCA1    FF15 78214000    call dword ptr ds:[402178]             ; msvcrt.__getmainargs  

00B6DCA7    68 0C304000      push 40300C  

00B6DCAC    68 00304000      push 403000  

00B6DCB1    E8 B43B89FF      call notepad_.0040186A                 ; jmp to msvcrt._initterm  

00B6DCB6    83C4 24          add esp,24  

00B6DCB9    8B05 74214000    mov eax,dword ptr ds:[402174]           ; msvcrt._acmdln  

00B6DCBF    8B70 00          mov esi,dword ptr ds:[eax]  

00B6DCC2    89B5 8CFFFFFF    mov dword ptr ss:[ebp-74],esi  

00B6DCC8    80BE 00000000 2  cmp byte ptr ds:[esi],22  

00B6DCCF    0F85 6A000000    jnz 00B6DD3F                            //这个对应 00401816  

00B6DCD5    46               inc esi  

00B6DCD6    89B5 8CFFFFFF    mov dword ptr ss:[ebp-74],esi  

00B6DCDC    8A06             mov al,byte ptr ds:[esi]  

00B6DCDE    3AC3             cmp al,bl  

00B6DCE0    0F84 09000000    je 00B6DCEF                             //这个 JE 对应 004017EA  

00B6DCE6    80F8 22          cmp al,22  

00B6DCE9    0F85 E6FFFFFF    jnz 00B6DCD5                            //这个 JNZ 对应 004017DC  

00B6DCEF    80BE 00000000 2  cmp byte ptr ds:[esi],22  

00B6DCF6    0F85 07000000    jnz 00B6DD03                            //这个 JNZ 对应 004017F3  

00B6DCFC    46               inc esi  

00B6DCFD    89B5 8CFFFFFF    mov dword ptr ss:[ebp-74],esi  

00B6DD03    8A06             mov al,byte ptr ds:[esi]  

00B6DD05    3AC3             cmp al,bl  

00B6DD07    0F84 09000000    je 00B6DD16                             //这个 JE 对应 004017FD  

00B6DD0D    80F8 20          cmp al,20  

00B6DD10    0F86 E6FFFFFF    jbe 00B6DCFC                            //这个 JBE 对应 004017EF  

00B6DD16    899D D0FFFFFF    mov dword ptr ss:[ebp-30],ebx  

00B6DD1C    8D45 A4          lea eax,dword ptr ss:[ebp-5C]  

00B6DD1F    50               push eax  

00B6DD20    FF15 04204000    call dword ptr ds:[402004]  

00B6DD26    F685 D0FFFFFF 0  test byte ptr ss:[ebp-30],1  

00B6DD2D    0F84 25000000    je 00B6DD58                             //这个 JE 对应 00401821  

00B6DD33    0FB785 D4FFFFFF  movzx eax,word ptr ss:[ebp-2C]  

00B6DD3A    E9 1C000000      jmp 00B6DD5B                            //这个 JMP 对应 00401824  

00B6DD3F    80BE 00000000 2  cmp byte ptr ds:[esi],20                

00B6DD46    0F86 B7FFFFFF    jbe 00B6DD03                            //这个 JBE 对应 004017F3  

00B6DD4C    46               inc esi  

00B6DD4D    89B5 8CFFFFFF    mov dword ptr ss:[ebp-74],esi  

00B6DD53    E9 E7FFFFFF      jmp 00B6DD3F                            //这个 JMP 对应 0040181F  

00B6DD58    6A 0A            push 0A  

00B6DD5A    58               pop eax  

00B6DD5B    50               push eax  

00B6DD5C    56               push esi  

00B6DD5D    53               push ebx  

00B6DD5E    53               push ebx  

00B6DD5F    FF15 00204000    call dword ptr ds:[402000]             ; notepad_.004C1F78  

00B6DD65    50               push eax  

00B6DD66    E8 273B89FF      call notepad_.00401892  

00B6DD6B    8985 98FFFFFF    mov dword ptr ss:[ebp-68],eax  

00B6DD71    50               push eax  

00B6DD72    FF15 70214000    call dword ptr ds:[402170]             ; msvcrt.exit  

00B6DD78    8B45 EC          mov eax,dword ptr ss:[ebp-14]  

00B6DD7B    8B48 00          mov ecx,dword ptr ds:[eax]  

00B6DD7E    8B49 00          mov ecx,dword ptr ds:[ecx]  

00B6DD81    898D 88FFFFFF    mov dword ptr ss:[ebp-78],ecx  

00B6DD87    50               push eax  

00B6DD88    51               push ecx  

00B6DD89    E8 D63A89FF      call notepad_.00401864                 ; jmp to msvcrt._XcptFilter  

00B6DD8E    59               pop ecx  

00B6DD8F    59               pop ecx  

00B6DD90    C3               retn              
```

---

```asm
00401700    55               push ebp                                //修补好后我们在 00401700 处右键新建 EIP  
00401701    8BEC             mov ebp,esp                              //然后运行我们的 LORDPE 把程序 DUMP  
00401703    6A FF            push -1                                  //运行我们的 importRCE 一看有很多无效，好接下来就是我们的 IAT 处理啦  
00401705    68 00254000      push notepad_.00402500                   //记录一下我们的第一个 IAT 无效指针是 00002000+我们的基址 00400000=00402000  
0040170A    68 86184000      push notepad_.00401886                   ; jmp to msvcrt._except_handler3  
0040170F    64:A1 00000000   mov eax,dword ptr fs:[0]                 //重载程序吧  
00401715    50               push eax  
00401716    64:8925 0000000  mov dword ptr fs:[0],esp  
0040171D    83EC 68          sub esp,68  
```

---

```asm
00401DBA >   55               push ebp                               //OD 重载程序  
00401DBB     8BEC             mov ebp,esp                              //这个壳对断点的检测还是满严格的，我们设置调试选项卡，忽略除内存访问外的所有异常  
00401DBD     83C4 F0          add esp,-10                              //然后在命令行输入 D 00402000(就是跟踪的意思)  
00401DC0     B8 00104000      mov eax,notepad_.00401000               
00401DC5     E8 01000000      call notepad_.00401DCB  
00401DCA     9A 83C4108B E55  call far 5DE5:8B10C483  
00401DD1  -  E9 66890400      jmp notepad_.0044A73C  
```

---

```
00402000   71B58CB4          //在此右键断点--内存访问  
00402004   30FAAA1A          //好我们 shift+f9 运行程序  
00402008   64CB9DB7  
0040200C   7FEB7CBC  
00402010   530B2B31  
```

---

```asm
004C04CC    8910             mov dword ptr ds:[eax],edx ; notepad_.004C1F78     //25 次之后我们来到这里  
004C04CE    46               inc esi                                            //大家可以根据数据窗口中的变化来进行判断  
004C04CF    4F               dec edi  
004C04D0   ^ 75 98           jnz short notepad_.004C046A  
004C04D2    43               inc ebx  
004C04D3    FF0C24           dec dword ptr ss:[esp]  
004C04D6   ^ 0F85 75FFFFFF   jnz notepad_.004C0451  
```

---

```
00402000   00000000          //25 次之时我们的 IAT 出现啦，但是我们的 00402000 却没有出现  
00402004   00000000          //其实这个壳也可以勉强算是 IAT 分块处理的  
```



### 1. 初始调用栈分析

```
00402008  00000000                                                    //好我们开始单步 F8 跟一下吧  

0040200C  73D98D67  mfc42.#4486_CWinApp::OnDDECommand  

00402010  73D35EF1  mfc42.#2554_CWinApp::DoWaitCursor  

00402014  73D497A0  mfc42.#2512_CWinApp::DoMessageBox  

00402018  73D4224E  mfc42.#5731_CWinApp::SaveAllModified  
```

### 2. 第一处加密逻辑分析

```
004C04CC    8910            mov dword ptr ds:[eax],edx               //开始单步 F8 走吧  

004C04CE    46              inc esi                                   //这里是加一指令  

004C04CF    4F              dec edi                                   //这里是减一指令  

004C04D0  ^ 75 98           jnz short notepad_.004C046A             //这里应该是判断是否需要继续处理  

004C04D2    43              inc ebx                                   //这里同样加一  

004C04D3    FF0C24          dec dword ptr ss:[esp]                   

004C04D6  ^ 0F85 75FFFFFF  jnz notepad_.004C0451                     //这个 JNZ 判断是否处理完  

004C04DC    5A              pop edx                                   //处理好则跳下来继续执行  

004C04DD    5D              pop ebp  
```

```
004C0451    8D045B          lea eax,dword ptr ds:[ebx+ebx*2]         //这里取当前程序 IAT 起始的地址  

004C0454    8B55 00         mov edx,dword ptr ss:[ebp]               //而这一句应该是取当前 IAT 的地址，但是已经被加密啦  

004C0457    8B4482 08       mov eax,dword ptr ds:[edx+eax*4+8]       //这里取 13B 进行下一句 IAT 地址的计算  

004C045B    E8 702AF9FF     call notepad_.00452ED0                  

004C0460    8BF8            mov edi,eax                              //把 EAX 里的值 0 传送 EDI  

004C0462    4F              dec edi                                  //为 EDI 减 1  

004C0463    85FF            test edi,edi                             //比较相等则下面的跳转不实现  

004C0465    72 6B           jb short notepad_.004C04D2               //这里不跳  

004C0467    47              inc edi                                  //在次为 EDI 加一处理  

004C0468    33F6            xor esi,esi                              //清空 ESI 里的数据  

004C046A    8D045B          lea eax,dword ptr ds:[ebx+ebx*2]         //将有效加密计算后的指令传送 EAX  

004C046D    8B55 00         mov edx,dword ptr ss:[ebp]               //将 EBP 里的地址传送 EDX  

004C0470    8B5482 08       mov edx,dword ptr ds:[edx+eax*4+8]       //以下略过........  

004C0474    F604F2 01       test byte ptr ds:[edx+esi*8],1           //我们直接去看赋值的代码  
```

#### 信息窗口状态

```
ss:[004EB8D4]=00B76C04                                               //这里都是几句被传送计算中的加密指令  

edx=004C1F78  (notepad_.004C1F78)  

ss:[004EB8D4]=00B76C04  

edx=00B76C04  

ds:[00B770F8]=00B749F0  

edx=00B76C04  
```

### 3. 深入加密指令分析

```
004C04B1    FF7482 04       push dword ptr ds:[edx+eax*4+4]         //这里是计算出的加密 IAT 地址压栈  

004C04B5    8D045B          lea eax,dword ptr ds:[ebx+ebx*2]         //有效指令 13B 计算出我们当前需要被加密的 IAT 地址  

004C04B8    8B55 00         mov edx,dword ptr ss:[ebp]               //将 EBP 里的指传送入 EDX 加密  

004C04BB    8B4482 08       mov eax,dword ptr ds:[edx+eax*4+8]       //辅助计算下面一句 IAT 地址的数值  

004C04BF    8B44F0 04       mov eax,dword ptr ds:[eax+esi*8+4]       //取下一句 IAT 地址的数值  

004C04C3    8B15 30224D00   mov edx,dword ptr ds:[4D2230]           //取当前程序的基址  

004C04C9    0302            add eax,dword ptr ds:[edx]               //把程序的基址和 IAT 地址的数值相加  

004C04CB    5A              pop edx                                   //弹出被加密的 IAT 指针  

004C04CC    8910            mov dword ptr ds:[eax],edx               //讲 EDX 里的垃圾 IAT 指针传送给 IAT 地址  
```

#### 寄存器窗口状态

```
EAX 00402004 notepad_.00402004                     //这里是停在赋值时的情况  

ECX 00B76C04                                      //EAX 里为需要被加密的 IAT 地址  

EDX 00B94000                                      //EDX 里为已经被加密了的 IAT 的垃圾指针  

EBX 00000069                                      //从而很明显判断出我们怎么改都是多余的，因为指针是早在我们计算 IAT 之间已经加密  

ESP 0012FE30                                      //接下来我们去看看 IAT 怎么加密的吧  

EBP 004EB8D4 notepad_.004EB8D4                    //重载程序  

ESI 00000000  

EDI 00000001  

EIP 004C04CC notepad_.004C04CC  
```

首先我们随意找一个 API 函数，刚我们跟踪 IAT 时发现 IAT 被变形处理的这里我也就大体说一下，列如我们准备好一个 API 函数如：`CloseHandle`。

好我们重载程序 `ctrl+G` 搜索 `CloseHandle`。

### 4. 重新载入并跟踪 API

```
00401DBA >  55              push ebp                                   //从新载入 OD 的程序，我们把异常选项卡设置忽略全部异常  

00401DBB    8BEC            mov ebp,esp                                //ctrl+G 搜索一下我们的 CloseHandle 这个 API 看一下 IAT 是如何加密的  

00401DBD    83C4 F0         add esp,-10                                //这里的 API 函数大家任选，只要可以停下就好  

00401DC0    B8 00104000     mov eax,notepad_.00401000                 

00401DC5    E8 01000000     call notepad_.00401DCB                    

00401DCA    9A 83C4108B E55>call far 5DE5:8B10C483                   

00401DD1  - E9 66890400     jmp notepad_.0044A73C                      
```

### 5. 系统层面断点

```
7C809BD7 >  8BFF            mov edi,edi                                //查找到了这里，我们右键断点---内存访问  

7C809BD9    55              push ebp                                   //我们 shift+f9 运行程序，但是会有一些卡，这壳会检测的  

7C809BDA    8BEC            mov ebp,esp                                

7C809BDC    64:A1 18000000  mov eax,dword ptr fs:[18]                 

7C809BE2    8B48 30         mov ecx,dword ptr ds:[eax+30]             

7C809BE5    8B45 08         mov eax,dword ptr ss:[ebp+8]               
```

### 6. 解密循环细节

```
00479984    8A06            mov al,byte ptr ds:[esi]                   //我们停下来啦，停下的地址和刚不一样啦  

00479986    3D FF000000     cmp eax,0FF                                //不管他，我们 F8 开始跟吧，在跟的时候注意其他窗口的变化  

0047998B    0F87 BC3E0000  ja notepad_.0047D84D                       //一路执行向下跳转，打断向上跳转  

00479991    8A80 9E994700   mov al,byte ptr ds:[eax+47999E]          

00479997    FF2485 9E9A4700 jmp dword ptr ds:[eax*4+479A9E]          

0047999E    0001            add byte ptr ds:[ecx],al                  

004799A0    0203            add al,byte ptr ds:[ebx]                  
```

```
0047D850    5E              pop esi                                 

0047D851    5B              pop ebx                                 

0047D852    5D              pop ebp                                  //F8 继续走，下面的 retn 返回到下一段代码  

0047D853    C2 0C00          retn 0C                                 //这里返回到下一段代码，我们走吧  

0047D856    8BC0            mov eax,eax                             

0047D858    55              push ebp                                 
```

```
004C0B2D    C645 F7 00      mov byte ptr ss:[ebp-9],0                //一路漫长我们来到了这里  

004C0B31    03F3            add esi,ebx                              

004C0B33  ^ E9 ADFBFFFF     jmp notepad_.004C06E5                    //这个跳转向上跳，我们向下打断他  

004C0B38    807D F6 00      cmp byte ptr ss:[ebp-A],0                //继续 F8 小心些走啦  

004C0B3C    75 55           jnz short notepad_.004C0B93              

004C0B3E    B8 02000000     mov eax,2                                

004C0B43    E8 5400F9FF     call notepad_.00450B9C                   
```

### 7. 最终加密指令确认

```
004C0B93    8D047F          lea eax,dword ptr ds:[edi+edi*2]         //走到这里时放慢我们的脚步  

004C0B96    8B55 FC         mov edx,dword ptr ss:[ebp-4]            

004C0B99    8B4D F0         mov ecx,dword ptr ss:[ebp-10]           

004C0B9C    894C82 04       mov dword ptr ds:[edx+eax*4+4],ecx       //当我们路过这一句时，发现我们的 IAT 指针出现，并未加密  

004C0BA0    47              inc edi                                  //如下  

004C0BA1    FF4D D0         dec dword ptr ss:[ebp-30]               
```

#### 信息窗口状态

```
ecx=00B781A2                                               //这里是我们停在 mov dword ptr ds:[edx+eax*4+4],ecx 的情况  

ds:[00B6F5F0]=7C809BD7  (kernel32.CloseHandle)             //IAT 保持完整并未加密  
```

```
004C0B99    8B4D F0         mov ecx,dword ptr ss:[ebp-10]           

004C0B9C    894C82 04       mov dword ptr ds:[edx+eax*4+4],ecx       //当我们从这里走下来的时候会发现寄存器里的值发生变化啦  

004C0BA0    47              inc edi                                  //IAT 指针已经不见啦，那么就证明上面一句就是我们的加密指令  

004C0BA1    FF4D D0         dec dword ptr ss:[ebp-30]              

004C0BA4  ^ 0F85 CAFAFFFF  jnz notepad_.004C0674                    //这里跳上去循环加密  
```

经过多次跟踪之后发现他会调用 `mov dword ptr ds:[edx+eax*4+4],ecx` 这句三次，从而判断出我们的 IAT 指针是由这一句执行了加密处理，我们从载程序。

```
00401DBA >  55              push ebp                             //我们先设置异常选项卡为忽略除内存访问外的所有异常  
```



# Notepad 壳分析记录

## 1. 初始断点设置

```
00401DBB    8BEC                mov ebp,esp                       // 然后输入我们的第一个加密指针地址 00402000

00401DBD    83C4 F0             add esp,-10                       // 命令提示行输入 d 00402000(即跟踪的意思)

00401DC0    B8 00104000         mov eax,notepad\_.00401000
```

## 2. 数据窗口初始状态

```
━━━━━━━━数据窗口━━━━━━━━━━━

00402000   71B58CB4                      // 我们在这句右键---断点---内存访问

00402004   30FAAA1A                      // 下好断点后我们 shift+f9 运行程序

00402008   64CB9DB7

0040200C   7FEB7CBC

00402010   530B2B31

00402014   69E061B0

━━━━━━━━━━━━━━━━━━━━━━━
```

## 3. IAT 初始化前的处理逻辑

```
004C9E2C    3100                xor dword ptr ds:\[eax\],eax       // 我们到这里时就最佳的时机，因为在走一步 IAT 就会开始初始

004C9E2E    ^ E9 1588FFFF       jmp notepad\_.004C2648             // 所以我们要在他初始化之前把他的加密地址全部解决掉

004C9E33    8B4424 0C           mov eax,dword ptr ss:\[esp+C\]     // 我们右键---搜索---全部命令 mov dword ptr ds:\[edx+eax*4+4\],ecx

004C9E37    8B4C24 04           mov ecx,dword ptr ss:\[esp+4\]

004C9E3B    C740 04 0000000>    mov dword ptr ds:\[eax+4\],0

004C9E42    C740 08 0000000>    mov dword ptr ds:\[eax+8\],0

004C9E49    C740 0C 0000000>    mov dword ptr ds:\[eax+C\],0

004C9E50    C740 10 0000000>    mov dword ptr ds:\[eax+10\],0

004C9E57    8160 14 F00FFFF>    and dword ptr ds:\[eax+14\],FFFF0FF0

004C9E5E    8160 18 00DC000>    and dword ptr ds:\[eax+18\],0DC00

004C9E65    C780 B8000000 7>    mov dword ptr ds:\[eax+B8\],notepad\_.004C9>

━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 4. 搜索并修补 IAT 指令

搜索所有 `mov dword ptr ds:[edx+eax*4+4], ecx` 指令：

```
004BFEC3    894C82 04           mov dword ptr ds:\[edx+eax*4+4\],ecx        // 这里是第一句

004BFFC1    894C82 04           mov dword ptr ds:\[edx+eax*4+4\],ecx        // 这里是第二句

004C0B9C    894C82 04           mov dword ptr ds:\[edx+eax*4+4\],ecx        // 这里是第三句
```

把他全部 `NOP` 掉，处理好之后我们继续 `shift+f9` 运行程序

## 5. 数据窗口状态变化

### 5.1 初始化前

```
━━━━━━━数据窗口━━━━━━━━━━━━━━

00402000   00000000                      // IAT 开始初始化

00402004   00000000                      // 即将填充我们的 IAT 指针

00402008   00000000                      // shift+f9 运行程序吧

0040200C   00000000

00402010   00000000

00402014   00000000

━━━━━━━数据窗口━━━━━━━━━━━━━━
```

### 5.2 初始化后

```
00402000   7C80B731   kernel32.GetModuleHandleA             // IAT 全部回来啦

00402004   7C801EF2   kernel32.GetStartupInfoA              // 反汇编窗口一片空白

00402008   00000000                                          // 不管他直接运行我们的 importRCE 修复吧

0040200C   73D98D67   mfc42.#4486_CWinApp::OnDDECommand     // 直接输入我们的 OEP 1700，然后点自动查找 IAT 指针

00402010   73D35EF1   mfc42.#2554_CWinApp::DoWaitCursor     // 修复后发现正常运行，解决啦

00402014   73D497A0   mfc42.#2512_CWinApp::DoMessageBox     // 到此我们脱壳结束

00402018   73D4224E   mfc42.#5731_CWinApp::SaveAllModified
```

写得不好请高手别笑话!
