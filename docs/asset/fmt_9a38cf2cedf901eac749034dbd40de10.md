
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
