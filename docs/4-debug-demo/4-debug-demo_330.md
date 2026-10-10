

# Key File 保护

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-41.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-41.htm)

**Key File 保护 **　**【习题】 **

**1、习题一** chap6-1-4-01 Keyfile 和 Serial/Name 103K 易

 **2、习题二 **chap6-1-4-02 Keyfile 120K 易  

**3、习题三 **chap6-1-4-03 Keyfile 3K 易

**4、习题四 **chap6-1-4-04 Keyfile 22K 中

**5、习题五 **chap6-1-4-05 Keyfile 14K 难

 

 1、习题一 答案          

| 保护类型： Keyfile and Serial/Name    
工具：Softice 和 W32dasm   
Cracker: FireWorx                      
用 W32DASM 装载程序，先用串式数据参考(String data refs) 检查程序。   
你运行 crame 时，选择任一文件，则出现:"Invalid Keyfile"，因此在 W32DASM 查找这字符串，双击来到 W32DASM 中，向上会看到：   
  
\* Possible StringData Ref from Code Obj ->"Runtime Error: 12FF:024"   
                                  |   
:00427DDE BAD07E4200              mov edx, 00427ED0      //打开文件   
:00427DE3 E8A0B8FDFF              call 00403688          //检测文件   
:00427DE8 750C                    jne 00427DF6             //如果无效的 keyword 则跳走   
:00427DEA C705E8A6420002000000    mov dword ptr \[0042A6E8\], 00000002   
:00427DF4 EB67                    jmp 00427E5D       
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00427DE8(C)   
|   
:00427DF6 C705E8A6420001000000    mov dword ptr \[0042A6E8\], 00000001  //准备 nag 窗口!   
:00427E00 6A00                    push 00000000   
  
\* Possible StringData Ref from Code Obj ->"Error"   
                                  |   
:00427E02 68E87E4200              push 00427EE8     //写字符"Invalid Keyfile" 到信息窗口...   
  
\* Possible StringData Ref from Code Obj ->"Invalid Keyfile"   
  
不过你们看到上面的字符串: "Runtime Error: 12FF:024" ，真有点奇怪？难到是程序的 BUG？再激发点灵感，是不是 keyfile 文件中的 Keywords？   
试试吧，建一文本文件，把 Runtime Error: 12FF:024 这一行放进去。运行程序，选择你刚才建的文件，OK,这下没出现错误，成功！现在程序充许你键入 name 和 code。你任意输入点击 OK,出现: "Wrong Entry! Try again!"   
因此在 W32DASM 下查找出现的错误，具体如下：   
  
\* Possible StringData Ref from Code Obj ->"No way"   
                                  |   
:0042818A 683C824200              push 0042823C   
  
\* Possible StringData Ref from Code Obj ->"Wrong entry! Try again."   
  
向上你会看到：   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0042810C(C)   
|   
:0042811A 8D55F4                  lea edx, dword ptr \[ebp-0C\]   
:0042811D 0FB7C6                  movzx eax, si   
:00428120 E8E3E1FDFF              call 00406308   
:00428125 8D55F4                  lea edx, dword ptr \[ebp-0C\]   
:00428128 B903000000              mov ecx, 00000003   
:0042812D B804824200              mov eax, 00428204   
:00428132 E8CDB6FDFF              call 00403804   
:00428137 8D55F4                  lea edx, dword ptr \[ebp-0C\]   
:0042813A B905000000              mov ecx, 00000005   
:0042813F B804824200              mov eax, 00428204   
:00428144 E8BBB6FDFF              call 00403804   
:00428149 8D95D0FBFFFF            lea edx, dword ptr \[ebp+FFFFFBD0\]   
:0042814F 8B87EC010000            mov eax, dword ptr \[edi+000001EC\]   
:00428155 E8E6D5FEFF              call 00415740   
:0042815A 8B85D0FBFFFF            mov eax, dword ptr \[ebp+FFFFFBD0\]   
:00428160 8B55F4                  mov edx, dword ptr \[ebp-0C\]   
:00428163 E820B5FDFF              call 00403688         //计算序列号   
:00428168 751E                    jne 00428188         //如两者相等，刚成功。   
:0042816A 6A00                    push 00000000   
  
\* Possible StringData Ref from Code Obj ->"Gratulations"   
                                  |   
:0042816C 6808824200              push 00428208   
  
\* Possible StringData Ref from Code Obj ->"Well Done! Try the next CrackMe."   
  
下面的工作就简单了，用 SOFTICE 和 TRW2000 来到 00428163 一行，进去，会发现: "cmp eax,edx"，键入"d edx" 会看到正确的序列号  |
| --- |

 2、习题二 答案         

| 保护类型：Keyfile Prot   
这个 Crackme 相当容易，很适合初学者掌握 KEY FILE 的破解。首先第一步我们要建一个假的 KEY FILE 文件，   
你用记事本建一个，内容：FFFFFFFFFF.注意：程序处理这文件是按十六进制进行的，你用用十六进制工具（我用的是 Hex Workshop v3.02，这软件很好用）打开这文件,内容如下：   
  00000000:      46 46 46 46 46 46 46 46 46 46   
  
用函数 ReadFile 设置断点，该函数功能是从文件中读取数据。运行 cracme，打开一这文件 TC.KEY，你将中断，按 F12 或 F10 直到你来到：   
  
:00429DA3  E840A7FDFF          CALL    004044E8        ; 读文件   
:00429DA8  E86B88FDFF          CALL    00402618   
:00429DAD  33C0                XOR    EAX,EAX        ; eax 清零   
:00429DAF  8A45FF              MOV    AL,\[EBP-01\]    ; al = 从 TC.KEY 文件读取的数据   
:00429DB2  03D8                ADD    EBX,EAX        ; ebx = ebx + al   
:00429DB4  8D85B0FEFFFF        LEA    EAX,\[EBP-0150\]   
:00429DBA  E8D5A6FDFF          CALL    00404494   
:00429DBF  E85488FDFF          CALL    00402618   
:00429DC4  84C0                TEST    AL,AL   
:00429DC6  74D2                JZ      00429E9A        ; 检测文件 EOF（EOF 文件结束标志）结束否   
:00429DC8  8D85B0FEFFFF        LEA    EAX,\[EBP-0150\]   
:00429DCE  E885A6FDFF          CALL    00404458   
:00429DD3  E84088FDFF          CALL    00402618   
:00429DD8  81FBA9200000        CMP    EBX,000020A9    ; keyfile 中所有字符和= 20A9h   
:00429DDE  750E                JNZ    00429DEE        ; 如不相等=> invalid KEY FILE   
:00429DE0  BA4C9E4200          MOV    EDX,00429E4C    ; edx = 显示: Registered - 成功   
:00429DE5  8BC6                MOV    EAX,ESI   
:00429DE7  E85CEEFEFF          CALL    00418C48   
:00429DEC  EB0C                JMP    00429DFA   
:00429DEE  BA2C9E4200          MOV    EDX,00429E2C    ; edx = 显示: Unregistered   
  
因此，到这里答案己出来了，我们的 KEY-FILE 文件的内容要等于 20A9h，满足这样条件很多，如：   
68 \* z 加 1 \* A  即:44\*7A+40＝20A9（注意：这些都是十六进制运算）   
因此你用记事本建一文件，内容如下：   
  
zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzA   
  
用十六进制工具打开这文本文件，会看到：   
00000000:  7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A   
00000010:  7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A   
00000020:  7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A   
00000030:  7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A-7A 7A 7A 7A   
00000040:  7A 7A 7A 7A-41   
  
这时你运行 cracme，打开这 KEY-FILE,标题栏出现： Registered   
  
现在我们再用一法，打补丁。在 W32DASM 打开这文件，查找 Unregistered，双击来到：   
  
\* Possible StringData Ref from Code Obj ->"Status: Unregistered"   
                                  |   
:00429D44 BA2C9E4200              mov edx, 00429E2C  <-- 注意这 push adress   
:00429D49 8BC6                    mov eax, esi   
  
再寻找 'Status: Registered - 你来到：   
  
\* Possible StringData Ref from Code Obj ->"Status: Registered - Well done"   
                                  |   
:00429DE0 BA4C9E4200              mov edx, 00429E4C  <-- 注意这 push adress   
:00429DE5 8BC6                    mov eax, esi   
:00429DE7 E85CEEFEFF              call 00418C48   
:00429DEC EB0C                    jmp 00429DFA   
  
将 00429D44 一行改成：mov edx,00429E4C   
现在寻找：BA2C9E4200   
  改成 ：BA4C9E4200   
现在完全被 cracked.  |
| --- |

 3、习题三 答案         

| 这是一 KEYFILE 保护文件，它不象前两个中的 keyfile 文件名可自己定。第一步就是找到这个文件名。   
1) BPX CreateFileA 来到：   
0137:004010AA  PUSH    004020E5   
0137:004010AF  CALL    00401192                      ；下 d 4020e5 即可看到文件名： N0P3X.KEY   
0137:004010B4  MOV    \[00402000\],EAX   
当然你在 W32DASM 中也能方便找到 KEYFILE 文件名。   
  
2)建立这文件，内容随便输入点东西，运行 cracme，来到：   
                                                      ; 004020F3 内就是 N0P3X.KEY 读出数据   
  :00401113    XOR    BYTE PTR \[EAX+004020F3\],43    ; 43h 和 N0P3X.KEY 中读出的数据 XOR   
  :0040111A    INC    EAX                            ; EAX＋1   
  :0040111B    CMP    BYTE PTR \[EAX+004020F3\],00    ; 判断 N0P3X.KEY 中是否有内容   
  :00401122    JNZ    00401113                      ;   
  :00401124    PUSH    004020F3                      ; PUSH 刚才 XOR 的数据   
  :00401129    PUSH    004020FD                      ; PUSH 要用来比较的数据，实际全是 0   
  :0040112E    CALL    KERNEL32!lstrcmp              ; 比较它们   
  :00401133    CMP    EAX,00                        ; EAX=0 则比较相等，注册成功   
  :00401136    JZ      0040113E                      ;   
  :00401138    JMP    00401158                      ; 否则 Unregistered   
  
3) 我们分析上面代码后，得知 N0P3X.KEY 中的字符要和 43H XOR 结果为 0,即可注册成功。43H 的 ASCII 码是 C.所以 N0P3X.KEY 内容是任意个数的：CCCCCCCCCC  |
| --- |

 4、习题四 答案        

| Mikl0 的破解教程   
目标：Kwazy\_W's PacMe   
工具：SoftICE   
  
 

> 大家好！花了足足 3 个多钟头，终于搞定了这个程序。这个程序的确可以说是 crackme 中的经典，而且很有趣味，又佩服作者设计它时的用心良苦和精巧构思。相信很多人都可以轻易的 crack 掉这个东东，但是我是新手，虽然有心做一名 cracker，但是学艺不精，因而花了很长的时间才得以解决，实在是惭愧。在此，只是写出我的一点想法和感受，和大家探讨一下，希望各位大虾多多指教！   
>   
>   言归正传，开始看这个程序。程序的 check 按钮是检查该目录下一个 key 文件，并显示是否注册成功。按照我平时的习惯，先用 W32Dasm 查看一番（可惜我用的 W32Dasm 版本中文支持都不好，:(，不知道哪里有没有好用的版本），发现有如下的字符串：Congratulations!  Mail me (KwazyWebbit@hotmail"，呵呵，多少和我们的目标有些关系的，双击看到如下内容：   
>   
> \* Reference To: GDI32.MoveToEx, Ord:0147h   
>                                   |   
> :0040113D E812070000              Call 00401854   
> :00401142 FF7518                  push \[ebp+18\]   
> :00401145 FF7514                  push \[ebp+14\]   
> :00401148 FF7508                  push \[ebp+08\]   
>   
> \* Reference To: GDI32.LineTo, Ord:0144h   
>                                   |   
> :0040114B E8FE060000              Call 0040184E   
> :00401150 C9                      leave   
> :00401151 C21400                  ret 0014   
>   
>   
> :00401154 33D2                    xor edx, edx   
> :00401156 B82E522E55              mov eax, 552E522E   
> :0040115B B9454D414C              mov ecx, 4C414D45   
> :00401160 33C1                    xor eax, ecx   
> :00401162 0553494854              add eax, 54484953   
> :00401167 B941205349              mov ecx, 49532041   
> :0040116C 0BC1                    or eax, ecx   
> :0040116E 2D454B4146              sub eax, 46414B45   
> :00401173 23D0                    and edx, eax   
> :00401175 81FADEC0AD0B            cmp edx, 0BADC0DE   
> :0040117B 7513                    jne 00401190   
> :0040117D 6A00                    push 00000000   
>   
> \* Possible StringData Ref from Data Obj ->"Success.."   
>                                   |   
> :0040117F 6859334000              push 00403359   
>   
> \* Possible StringData Ref from Data Obj ->"Congratulations!  Mail me (KwazyWebbit@hotmail"   
>                                         ->".com) how you did it.  Dont forget "   
>                                         ->"to include your keyfile! =\]"   
>                                   |   
> :00401184 68EC324000              push 004032EC   
> :00401189 6A00                    push 00000000   
>   
> \* Reference To: USER32.MessageBoxA, Ord:01BBh                  ；显示一个对话框，告诉你已经成功了！！！！！！！！   
>                                   |   
> :0040118B E81C060000              Call 004017AC   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:0040117B(C)   
> |   
> :00401190 C3                      ret   
>   
>   
>     分析一下，该字符串是用作参数，由系统调用 MessageBoxA 向你报告你已经成功破解了！然后就是返回。向前看，觉得实在不着边际（我水平有限），于是放弃用 W32Dasm 静态分析，不得不请出元老级工具 SoftIce。在这之前先用 File Monitor 跟踪分析一下，   
> 2    20:11:42    Pacme    Open    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    NOTFOUND    OPENEXISTING READONLY DENYNONE   
>        
>     发现启动或者按 check 按钮时，要打开一个名为 kwazyweb.bit 的文件，于是生成一个空文件，命名为 kwazyweb.bit，继续分析   
> 380    20:12:32    Pacme    Open    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    OPENEXISTING READONLY DENYNONE        
> 381    20:12:32    Pacme    Read    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    Offset: 0 Length: 1       
> 382    20:12:32    Pacme    Close    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    CLOSE\_FINAL   
>        
>     程序从文件头读取一个字节，往文件中写入一个字节‘1’，continue、、、   
> 1183    20:13:00    Pacme    Open    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    OPENEXISTING READONLY DENYNONE        
> 1184    20:13:00    Pacme    Read    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    Offset: 0 Length: 1       
> 1185    20:13:00    Pacme    Read    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    Offset: 1 Length: 49       
> 1186    20:13:00    Pacme    Read    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    Offset: 1 Length: 18       
> 1187    20:13:00    Pacme    Close    C:\\WINDOWS\\DESKTOP\\KWAZYWEB.BIT    SUCCESS    CLOSE\_FINAL       
>   
>     发现程序读取文件的次数由原来的一次变到了三次，研究一下，发现第二次读取数据的长度 49 刚好是'1'的 ASCII 值，好，这只是猜测。到 W32Dasm 中证实一下，发现 ReadFile 调用的次数刚好是 3 次，而且，第二次读取的长度为 eax，就是第一次读取的数值，这不是巧合！！！第三次读的长度为 0x12，刚好是十进制 18，与 File Monitor 分析结果完全吻合。   
>   
> :004016EA 6848344000              push 00403448   
> :004016EF 6A01                    push 00000001   
> :004016F1 68FA344000              push 004034FA   
> :004016F6 FF3544344000            push dword ptr \[00403444\]   
>   
> \* Reference To: KERNEL32.ReadFile, Ord:01FDh   
>                                   |   
> :004016FC E811010000              Call 00401812                ；读取文件的第一个字节，保存在\[004034FA\]中   
> :00401701 0FB605FA344000          movzx eax, byte ptr \[004034FA\]   
> :00401708 85C0                    test eax, eax   
> :0040170A 743B                    je 00401747   
> :0040170C 6A00                    push 00000000   
> :0040170E 6848344000              push 00403448   
> :00401713 50                      push eax   
> :00401714 6888324000              push 00403288                ；i am here，hehe   
> :00401719 FF3544344000            push dword ptr \[00403444\]   
>   
> \* Reference To: KERNEL32.ReadFile, Ord:01FDh   
>                                   |   
> :0040171F E8EE000000              Call 00401812                ；从第二个字节开始读取数据，长度由 eax，即第一个字   
>                                     ；的大小决定   
> :00401724 E8D7F8FFFF              call 00401000                ；计算一个关键数据，一定要进去看看，:)   
> :00401729 6A00                    push 00000000   
> :0040172B 6848344000              push 00403448   
> :00401730 6A12                    push 00000012   
> :00401732 68E8344000              push 004034E8   
> :00401737 FF3544344000            push dword ptr \[00403444\]   
>   
> \* Reference To: KERNEL32.ReadFile, Ord:01FDh   
>                                   |   
> :0040173D E8D0000000              Call 00401812                ；第三次读取数据，长度为 0x12（18）   
> :00401742 E882F9FFFF              call 004010C9                ；关键所在！！！F8 进去   
>   
>   
>     在 SoftICe 中下断点 bpx readfile，可以来到上面的位置，除了文件读取 ReadFile 是否成功的必要检查之外，基本上没有什么跳转，而且 call 也不多，这使我感到非常幸运。首先 F8 到 00401724 处的 call 00401000，来到下面的地方：   
>   
> :00401000 33C0                    xor eax, eax   
> :00401002 33D2                    xor edx, edx   
> :00401004 33C9                    xor ecx, ecx   
> :00401006 8A0DFA344000            mov cl, byte ptr \[004034FA\]        ；还记得这个值吗，看看上面，就是文件第一个字节的值   
> :0040100C BE88324000              mov esi, 00403288            ；在上面看看，是第二次读取数据的缓冲区   
> :00401011 AC                      lodsb                    ；读取缓冲区内容到 AL 中   
> :00401012 03D0                    add edx, eax                ；将 AL 加到 EDX 中   
> :00401014 E2FB                    loop 00401011                ；不断重复，结果是将第二次所有数据累加起来   
> :00401016 8815FB344000            mov byte ptr \[004034FB\], dl        ；放到\[004034FB\]，注意，以后要用的   
> :0040101C C3                      ret   
>   
>   
>     继续跟踪第二个 call，即 00401742 处的 call 004010C9，如下：   
>   
> :004010C9 55                      push ebp   
> :004010CA 8BEC                    mov ebp, esp   
> :004010CC 83C4FC                  add esp, FFFFFFFC   
>   
> \* Possible StringData Ref from Data Obj ->"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*C\*......\*...\*\*\*\*.\*.\*\*\*\*...\*..."  ；这里的数据非常有趣，慢慢看吧   
>                                         ->".\*.\*..\*\*\*\*\*\*\*\*\*\*.\*..\*....\*...\*...\*\*.\*\*\*\*.\*.\*.."   
>                                         ->".\*\*\*\*.\*....\*.\*\*\*\*\*\*\*..\*.\*\*\*..\*.....\*.\*..\*\*\*.\*\*"   
>                                         ->".\*\*\*.\*...\*\*\*\*....\*X..\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"   
>                                   |   
> :004010CF 6865334000              push 00403365   
>   
> \* Possible StringData Ref from Data Obj ->"\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*C\*......\*...\*\*\*\*.\*.\*\*\*\*...\*..."   
>                                         ->".\*.\*..\*\*\*\*\*\*\*\*\*\*.\*..\*....\*...\*...\*\*.\*\*\*\*.\*.\*.."   
>                                         ->".\*\*\*\*.\*....\*.\*\*\*\*\*\*\*..\*.\*\*\*..\*.....\*.\*..\*\*\*.\*\*"   
>                                         ->".\*\*\*.\*...\*\*\*\*....\*X..\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*"   
>                                   |   
> :004010D4 68BC314000              push 004031BC   
>   
> \* Reference To: KERNEL32.lstrcpyA, Ord:02DCh   
>                                   |   
> :004010D9 E83A070000              Call 00401818                ；简单的拷贝，不必理它。   
> :004010DE C70584314000CC314000    mov dword ptr \[00403184\], 004031CC    ；注意一下这个变量，指向上面的图案中的第二行，为了                                    ；方便后面的描述，记为 offsetOfPrettyPicture   
> :004010E8 E830FFFFFF              call 0040101D                ；处理第三次读取的 18 个数据   
> :004010ED C645FE00                mov \[ebp-02\], 00   
> :004010F1 33C0                    xor eax, eax   
> :004010F3 33C9                    xor ecx, ecx   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00401128(C)   
> |   
> :004010F5 C645FF08                mov \[ebp-01\], 08            ；大循环开始   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:0040111F(C)   
> |   
> :004010F9 806DFF02                sub byte ptr \[ebp-01\], 02        ；小循环开始   
> :004010FD 0FB64DFE                movzx ecx, byte ptr \[ebp-02\]   
> :00401101 81C1E8344000            add ecx, 004034E8   
> :00401107 8A01                    mov al, byte ptr \[ecx\]   
> :00401109 8A4DFF                  mov cl, byte ptr \[ebp-01\]   
> :0040110C D2E8                    shr al, cl   
> :0040110E 2403                    and al, 03   
> :00401110 E81EFFFFFF              call 00401033                ；注册在该 call 中实现!!!!!!!!!!!!   
> :00401115 85C0                    test eax, eax   
> :00401117 7411                    je 0040112A                ；!!!!!!!!这里是关键所在，如果跳转则退出该函数，不                                    ；产生任何变化   
> :00401119 0FB655FF                movzx edx, byte ptr \[ebp-01\]   
> :0040111D 85D2                    test edx, edx   
> :0040111F 75D8                    jne 004010F9                ；小循环的循环次数为 4 次，\[ebp-01\]从 8 减为 0   
> :00401121 FE45FE                  inc \[ebp-02\]   
> :00401124 807DFE12                cmp byte ptr \[ebp-02\], 12   
> :00401128 75CB                    jne 004010F5                ；大循环的循环次数为 0x12（18），为数据缓冲区的大小   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00401117(C)   
> |   
> :0040112A C9                      leave   
> :0040112B C3                      ret   
>   
>     看一看，call 也不多，呵呵，真幸运！   
> 首先看看 004010E8 处的 call 0040101D：   
>   
> :0040101D 8A15FB344000            mov dl, byte ptr \[004034FB\]        ；还记得吧上面提醒你注意的数据，就是第二次数据的和   
> :00401023 B912000000              mov ecx, 00000012   
> :00401028 B8E8344000              mov eax, 004034E8   
> :0040102D 3010                    xor byte ptr \[eax\], dl   
> :0040102F 40                      inc eax   
> :00401030 E2FB                    loop 0040102D   
> :00401032 C3                      ret   
>   
>     该函数的功能就是把第三次读取的 18 个数据与前面所得的和进行异或。看到这里，为了避免必要的麻烦，我修改了 kwazyweb.bit 文件，使得第一个字节为 0x01，记住，要用 16 进制工具编辑，第二个字节为 0x00，这样，所得的和为 0，任何数和 0 异或时不变，利用这一点，省去了不少麻烦。后面接着随便输入 18 个字节。该文件大小一共是 20 个字节。   
>   
>     好，继续我们的分析。该函数里面有两个 byte 类型的局部变量，分别是\[ebp-01\]和\[ebp-02\] (还有一个 word 类型的变量\[ebp-04\]，用于保存一个数据，下面会提到的)。有两个循环，大循环是为了处理 18 个数据，小循环对每一个数据进行 4 次处理，处理过程是右移 6，4，2，0 位之后，取出低 2 位，即若数据为 0x12345678，则分别处理 0x12，0x34，0x56，0x78。作为参数进入 00401110 处的 call 00401033。返回值决定是否退出当前函数。（多次试探知道注册在 00401110 处的 call 00401033 里面实现）。让我们一起来看看这个关键的函数到底做了写什么？   
>   
> :00401033 55                      push ebp   
> :00401034 8BEC                    mov ebp, esp   
> :00401036 83C4F8                  add esp, FFFFFFF8   
> :00401039 8B1584314000            mov edx, dword ptr \[00403184\]   
> :0040103F 8955FC                  mov dword ptr \[ebp-04\], edx        ;将 offsetOfPrettyPicture 值保存在局部变量\[ebp-04\]中   
> :00401042 0AC0                    or al, al   
> :00401044 7509                    jne 0040104F                ；AL＝0？   
> :00401046 832D8431400010          sub dword ptr \[00403184\], 00000010    ；AL=0，offsetOfPrettyPicture 减 0x10（向上移）   
> :0040104D EB1F                    jmp 0040106E                   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00401044(C)   
> |   
> :0040104F 3C01                    cmp al, 01                ；AL＝1？   
> :00401051 7508                    jne 0040105B                   
> :00401053 FF0584314000            inc dword ptr \[00403184\]        ；AL=1，offsetOfPrettyPicture 加 0x01（向右移）   
> :00401059 EB13                    jmp 0040106E   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00401051(C)   
> |   
> :0040105B 3C02                    cmp al, 02                ；AL=2？   
> :0040105D 7509                    jne 00401068   
> :0040105F 83058431400010          add dword ptr \[00403184\], 00000010    ；AL=2，offsetOfPrettyPicture 加 0x10（向下移）   
> :00401066 EB06                    jmp 0040106E   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:0040105D(C)   
> |   
> :00401068 FF0D84314000            dec dword ptr \[00403184\]        ；AL=3，offsetOfPrettyPicture 减 0x01（向左移）   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
> |:0040104D(U), :00401059(U), :00401066(U)   
> |   
> :0040106E 8B1584314000            mov edx, dword ptr \[00403184\]   
> :00401074 8A02                    mov al, byte ptr \[edx\]        ；看 offsetOfPrettyPicture 处的值   
> :00401076 3C2A                    cmp al, 2A                ；为 0x2A？'\*'   
> :00401078 7506                    jne 00401080   
> :0040107A 33C0                    xor eax, eax                ；'\*',则返回 0，上一级的函数退出，永远没有机会注册   
> :0040107C C9                      leave   
> :0040107D C3                      ret   
>   
>   
> :0040107E EB33                    jmp 004010B3   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:00401078(C)   
> |   
> :00401080 3C58                    cmp al, 58                ；为 0x58？'X'   
> :00401082 752F                    jne 004010B3   
> :00401084 6A00                    push 00000000                ;'X'，注册成功。在上面的图案中只有一个值是 X（Exit）   
> :00401086 8D1559334000            lea edx, dword ptr \[00403359\]        ;"Sucess..."   
> :0040108C 52                      push edx   
> :0040108D 8D15EC324000            lea edx, dword ptr \[004032EC\]        ;"Congratulations"....   
> :00401093 52                      push edx   
> :00401094 6A00                    push 00000000   
> :00401096 8D15AC174000            lea edx, dword ptr \[004017AC\]        ;\[User32.MessageBoxA\];在 SoftIce 中可以看到这个提示   
> :0040109C FFD2                    call edx                ；恭喜你，到这里就会显示成功信息！   
> :0040109E 8D157B324000            lea edx, dword ptr \[0040327B\]   
> :004010A4 52                      push edx   
> :004010A5 FF3520344000            push dword ptr \[00403420\]   
> :004010AB 8D15DC174000            lea edx, dword ptr \[004017DC\]        ;\[User32.SetWindowTextA\];同上   
> :004010B1 FFD2                    call edx   
>   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
> |:0040107E(U), :00401082(C)   
> |   
> :004010B3 8B1584314000            mov edx, dword ptr \[00403184\]           
> :004010B9 C60243                  mov byte ptr \[edx\], 43        ；将 offsetOfPrettyPicture 处的值改为 0x43，即'C'                                        ；（Current），代表当前位置   
> :004010BC 8B55FC                  mov edx, dword ptr \[ebp-04\]        ;则将以前的 offsetOfPrettyPicture 值调出   
> :004010BF C60220                  mov byte ptr \[edx\], 20        ；将上一个经历过的 offsetOfPrettyPicture 处值设为   
>                                     ；0x20，即空格，表示已经走过的路   
> :004010C2 B801000000              mov eax, 00000001   
> :004010C7 C9                      leave   
> :004010C8 C3                      ret   
>   
>     经过这一番分析之后，才发现作者原来是在教我们玩一个小游戏。大致思路是这样，一共走 18 次，每次可以走 4 步（18 次大循环和 4 次小循环），碰到'\*'就 game over，其他的就可以 continue，直到遇见'X'，游戏就算过关了。在图案（事先我只是感到这些数据有些古怪，到了分析清楚算法之后，才发现这实际上是一个迷宫，我们的任务就是正确的从迷宫中闯出来！）里面，有一个唯一的'X'，我就想，必须得到达这里才行，但是又不能碰到'\*'，刚开始试了好久，没想到合适的方法，直到我将数据区全部显示出来（将 data 区弄大了些），才发现这是一幅多么美妙的图片，到这里我不得不佩服作者的艺术天分！！！我把图片 dump 了出来，如下：   
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
> C\*......\*...\*\*\*\*   
> .\*.\*\*\*\*...\*....\*   
> .\*..\*\*\*\*\*\*\*\*\*\*.\*   
> ..\*....\*...\*...\*   
> \*.\*\*\*\*.\*.\*...\*\*\*   
> \*.\*....\*.\*\*\*\*\*\*\*   
> ..\*.\*\*\*..\*.....\*   
> .\*..\*\*\*.\*\*.\*\*\*.\*   
> ...\*\*\*\*....\*X..\*   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
> 
> 看到了吗？不就是一个标准的迷宫？从 C 开始，到达 X 结束！而且路线已经非常清楚了，顺着'.'走就行了，连岔道都没有，呵呵，从来没有见到如此容易的迷宫，大概是作者对我们劳动的一种慰问吧。下面的就容易了，想大家都很清楚，按照上面的程序分析，'0'代表↑,'1'代表→,'2'代表↓,'3'代表←，看着图片一步步向前进，就可以得到一系列数据：   
>   
> ↓↓↓→  ↓↓↓←  ↓↓→→  ↑→↑↑  →→→↑  ↑←←←  ↑←↑↑  →→→→  →↓→→              
> 2 2 2 1      2 2 2 3    2 2 1 1    0 1 0 0    1 1 1 0    0 3 3 3    0 3 0 0    1 1 1 1    1 2 1 1   
> ↑→→↓  →→→↓  ↓←←↓  ←←↑←  ←↓↓↓  ←↓↓→  →→↑↑  →→→→  ↓↓←←                        
> 0 1 1 2    1 1 1 2    2 3 3 2    3 3 0 3    3 2 2 2    3 2 2 1    1 1 0 0    1 1 1 1    2 2 3 3   
>   
> 看看我走的对不对。上面是所谓的 4 进制数，转换成 16 进制为   
>   
> A9 AB A5 10 54 3F 30 55 65 16 56 BE F3 EA E9 50 55 AF            ;KWAZYWEB.BIT   
>   
> 成功之后的图案变成了下面的样子：   
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
> \*      \*  \*\*\*\*   
> \* \*\*\*\*  \*    \*   
> \*  \*\*\*\*\*\*\*\*\*\* \*   
>   \*    \*  \*  \*   
> \* \*\*\*\* \* \*  \*\*\*   
> \* \*    \* \*\*\*\*\*\*\*   
>   \* \*\*\*  \*    \*   
> \*  \*\*\* \*\* \*\*\* \*   
>   \*\*\*\*    \*C  \*   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
> 走过的地方都一贫如洗，呵呵   
>   
>     这就是 KWAZYWEB.BIT 的全部内容。到此为止，基本上已经结束了所有的工作。在原程序中点 check，就弹出对话框告诉你注册成功，感觉真是爽啊！学 crack 就是欣赏这种感觉。不过，还有不足的就是文本框中的还是“Cracked by : ”，没有把我自己的名字给弄进去，本来想在弄一下的，可是实在太累了，还是算了吧。留给哪位大虾来解决这个不是问题的问题，呵呵、、、   
>   
>     还有一点，我想全部用 W32Dasm 处理这个问题可能效率更高一些，可是我说过，我学艺不精的，呵呵，要是谁全部用 W32Dasm 破解，希望能指点一二，不胜感激。这是我的第一篇文章，写的不好，望各位海涵。我现于大学就读，明年毕业，不知要飘向何方、、、我对计算机有着浓厚的兴趣，特别是喜欢编程，只可惜起步晚，加上时间不足，至今也只能算是个初级水平，唉，太差了，不谈。一直以来，很羡慕那些有着高水平的 cracker 和 hacker，也很欣赏成功那一刻的感觉，所以想多学习和了解有关方面的知识，有志同道合者，还望多多交流。   
>   
>   
>     我的 email 是 littlejackyonline@263.net，如果对这篇文章有什么疑问或者赐教，请发 email。另外 qq:22892341，但是别攻击我啊，:),bye------   
>   
>   
>                         ----------------  jackyspy 于 2001.8.8 夜 
> 
> * * *
> 
>  

 |
| --- |

 5、习题五 答案         

| I.  介绍   
I.1 这篇教程所需的工具   
II. 破解   
  
  
I.欢迎看我的第 21 篇教程.这次我将写我破的第一个 KEYFILE :)  虽然它很简单,但是我还是很高兴写这篇文章.   
I.1   
  
      W32Dasm 8.9   
    Cruehead's CrackMe 3.0   
  
II.破解:   
  
当你反编译后,你会看到一个很像文件名的字符串:Crackme3.key .很幸运这个就是真正的 KEYFILE.   
  
那么,让我们开始－你应该看这：   
  
  
    :00401021 6A03                    push 00000003   
    :00401023 68000000C0              push C0000000   
  
    \* Possible StringData Ref from Data Obj ->"CRACKME3.KEY"   
                                      |   
    :00401028 68D7204000              push 004020D7   
  
    \* Reference To: KERNEL32.CreateFileA, Ord:0000h   
                                      |   
    :0040102D E876040000              Call 004014A8  ;;寻找这个文件的 CALL   
    :00401032 83F8FF                  cmp eax, FFFFFFFF ;; 如果文件存在   
    :00401035 750C                    jne 00401043      ;; 就跳   
  
    ---省略了一部分---  ;;显示信息 Uncracked   
  
    :00401052 6A00                    push 00000000   
    :00401054 68A0214000              push 004021A0   
    :00401059 50                      push eax   
    :0040105A 53                      push ebx   
    :0040105B FF35F5204000            push dword ptr \[004020F5\]   
  
    \* Reference To: KERNEL32.ReadFile, Ord:0000h   
                                      |   
    :00401061 E830040000              Call 00401496 ;;开始读这个文件   
    :00401066 833DA021400012          cmp dword ptr \[004021A0\], 00000012 ;;大小是否为 18 字节   
    :0040106D 75C8                    jne 00401037  ;; 如果不是,则显示 Uncracked   
    :0040106F 6808204000              push 00402008 ;; 保存文件内容   
    :00401074 E898020000              call 00401311 ;; 进行计算   
    :00401079 8135F920400078563412    xor dword ptr \[004020F9\], 12345678 ;;与 12345678 做异或   
    :00401083 83C404                  add esp, 00000004   
    :00401086 6808204000              push 00402008   
    :0040108B E8AC020000              call 0040133C   
    :00401090 83C404                  add esp, 00000004   
    :00401093 3B05F9204000            cmp eax, dword ptr \[004020F9\] ;; 比较两个的值   
    :00401099 0F94C0                  sete al  ;; 如果相同,则在 AL 中做标志   
    :0040109C 50                      push eax ;; 保存 eax   
    :0040109D 84C0                    test al, al  ;;测试标志   
    :0040109F 7496                    je 00401037 ;; 如果为零就跳   
  
那好,创建一个 CrackMe3.key 大小为 18 bytes.创建后让我们看看 call 00401311.   
  
  \* Referenced by a CALL at Address:   
    |:00401074    
    |   
    :00401311 33C9                    xor ecx, ecx  ;; 清零   
    :00401313 33C0                    xor eax, eax  ;; 清零   
    :00401315 8B742404                mov esi, dword ptr \[esp+04\]  ;; esi 是文件内容   
    :00401319 B341                    mov bl, 41 ;; bl =41h   
  
    \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
    |:00401333(C)   
    |   
    :0040131B 8A06                    mov al, byte ptr \[esi\]  ;; al = 文件的第一个字节   
    :0040131D 32C3                    xor al, bl              ;; 第一个字节与 41h 做异或   
    :0040131F 8806                    mov byte ptr \[esi\], al  ;; 然后再存入 ESI   
    :00401321 46                      inc esi                ;; 下一个字节   
    :00401322 FEC3                    inc bl                  ;; bl = bl + 1   
    :00401324 0105F9204000            add dword ptr \[004020F9\], eax ;; 加上被异或的字节   
    :0040132A 3C00                    cmp al, 00              ;; AL 是否为零   
    :0040132C 7407                    je 00401335            ;; 如果为零就跳   
    :0040132E FEC1                    inc cl                  ;; cl = cl + 1   
    :00401330 80FB4F                  cmp bl, 4F              ;; bl = 4F?   
    :00401333 75E6                    jne 0040131B            ;; 如果小于 4F 就循环   
  
    \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
    |:0040132C(C)   
    |   
    :00401335 890D49214000            mov dword ptr \[00402149\], ecx ;;保存 ECX   
    :0040133B C3                      ret  ;; 返回   
  
  用 41H 与前 14 个字节做异或,然后再把异或后的值加起来.   
  那么 666999666999666999 是这样运算的:   
    36 XOR 41 +   
    36 XOR 42 +   
    36 XOR 43 +   
    39 XOR 44 +   
    39 XOR 45 +   
    39 XOR 46 +   
    36 XOR 47 +   
    36 XOR 48 +   
    36 XOR 49 +   
    39 XOR 4A +   
    39 XOR 4B +   
    39 XOR 4C +   
    36 XOR 4D +   
    36 XOR 4E +   
    36 XOR 4F   
    = 693h   
当我们离开这个 CALL 时,在 00401079 这,693H 与 12345678H 做异或. 结果就是 EB503412.   
在 00401093 处与 KEYFILE 的后 4 个字节比较.如果相同的话就破解成功.   
当你运行 CRACKME 时它将会显示说你已经破了它.可惜的是你的名字不会显示出来.   
KEYFILE 前 14 个字节是为名字预留的.这 14 个字节在显示之前是需要解码的.   
所以,我们要推算出 KEYFILE 被加密的字节.我们用 LaZaRuS 作为名字:   
  
    L = 4Ch XOR 41h = 0D   
    a = 61h XOR 42h = 23   
    Z = 5Ah XOR 43h = 19   
    a = 61h XOR 44h = 25   
    R = 52h XOR 45h = 17   
    u = 75h XOR 46h = 33   
    S = 53h XOR 47h = 14   
把这些字节放入 KEYFILE 的开始处.我为了凑足 18 个字节加了 00000000000000h.再运行 CRACKME,我什么也没看见.现在必须算出后四个字节的值.所以,在 401093 处设断.F7523412 就是要找的值.当我再运行时,它显示   
"Cracked by LaZaRuSHIJKLMN!"  现在我们知道,为凑足 18 个字节所输入的 00 是个错误.正确的 KEYFILE 应该是这样:   
  
    0D23 1925 1733 1448 494A 4B4C 4D4E F752 3412   
  
  BTW:  00 xor 48 =48h   
        00 xor 49 =49h   
        00 xor 4a =4ah   
        00 xor 4b =4bh   
        00 xor 4c =4ch   
        00 xor 4d =4dh   
        00 xor 4e =4eh   
事实上,这个 KEYFILE 并不正确.我们必须再重新算后四个字节.不过这已不再是问题了.   
  
  0D23 1925 1733 1448 494A 4B4C 4D4E FA54 3412          
        
这才是显示 Cracked by LaZaRuS!正确的 KEYFILE.(注:这些是 16 进制形式,不是 ASCII;ASCII 应该是:   
  
#%3HIJKLMN 鶷 4).garfield cat   
          译于 2000 年 9 月  |
| --- |

    

 

 4dd466d0e1e0cc123afbe504da2462ca