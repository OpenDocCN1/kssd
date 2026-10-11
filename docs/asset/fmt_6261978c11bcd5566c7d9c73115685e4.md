
# Password Keeper v6.3

> 原文：[`www.pediy.com/kssd/pediy05/pediy50073.htm`](https://www.pediy.com/kssd/pediy05/pediy50073.htm)

`Password Keeper` v6.3 破解过程 (难度：较易)

你是否有一大堆的密码，email，5460，rar 文件.....。让人头大。也许 `Password Keeper` 可以帮你一个忙。它是一个管理密码的优秀软件，只要记住一个密码，你就可以管理你所有的帐号和密码了。以下是其破解过程。

下断点 `bpx GetDlgItemTexta`,填入姓名 `gearboy`，组织 `energy`，假序列号 `654321`，点 OK，立即被 `TRW2000` 拦下。键入命令 `pmodule`，若干 F10 返回到调用的 `Call` 处，即 `00414962`。

```
017F:0041495F 6A65            PUSH    BYTE +65
017F:00414961 55              PUSH    EBP
017F:00414962 E88978FFFF      CALL    0040C1F0    //由此 Call 返回，此 Call 取组织名
017F:00414967 8D7E32          LEA     EDI,[ESI+32]  //停在此处，EDI 指向组织名 energy
017F:0041496A 6A32            PUSH    BYTE +32
017F:0041496C 57              PUSH    EDI
017F:0041496D 6A66            PUSH    BYTE +66
017F:0041496F 55              PUSH    EBP
017F:00414970 E87B78FFFF      CALL    0040C1F0
017F:00414975 8D442430        LEA     EAX,[ESP+30]
017F:00414979 6800010000      PUSH    DWORD 0100
017F:0041497E 50              PUSH    EAX
017F:0041497F 6A67            PUSH    BYTE +67
017F:00414981 55              PUSH    EBP
017F:00414982 E86978FFFF      CALL    0040C1F0        //此 Call 取假序列号
017F:00414987 8D4C2440        LEA     ECX,[ESP+40]    //ECX 指向假序列号 654321
017F:0041498B 51              PUSH    ECX             //ECX 传递参数
017F:0041498C E8BE7A0000      CALL    0041C44F        //将假序列号转化为十六进制数放入 EAX
017F:00414991 56              PUSH    ESI
017F:00414992 8BD8            MOV     EBX,EAX         //将假序列号的十六进制形式放入 EBX
017F:00414994 E8574A0000      CALL    004193F0        //将姓名长度放入 EDX,并计算根据姓名计算出的部分和，具体在后面分析
017F:00414999 83C438          ADD     ESP,BYTE +38
017F:0041499C 3D92A71901      CMP     EAX,0119A792    //检验部分和是否为 0119A792，可能是个超级用户名产生的部分和，只要是满足这个条件，任何密码都能注册，具体没有深入研究，一般的用户名都不会满足这个条件，所以都直接跳到 004149BB
017F:004149A1 7518            JNZ     004149BB        //不等则跳
017F:004149A3 8B1D68414200    MOV     EBX,[00424168]
017F:004149A9 68588B4200      PUSH    DWORD 00428B58
017F:004149AE 56              PUSH    ESI
017F:004149AF FFD3            CALL    EBX             //call lstrcpya,使 EAX 指向 Gregory Braun
017F:004149B1 68488B4200      PUSH    DWORD 00428B48
017F:004149B6 57              PUSH    EDI
017F:004149B7 FFD3            CALL    EBX             //call lstrcpya，使 EAX 指向 Software Design
017F:004149B9 EB07            JMP     SHORT 004149C2
017F:004149BB 3D3CCE5F0D      CMP     EAX,0D5FCE3C    //又是一个超级用户名比较
017F:004149C0 750C            JNZ     004149CE        //不等则跳
017F:004149C2 57              PUSH    EDI
017F:004149C3 56              PUSH    ESI
017F:004149C4 E8575E0000      CALL    0041A820
017F:004149C9 83C408          ADD     ESP,BYTE +08
017F:004149CC 8BD8            MOV     EBX,EAX
017F:004149CE 57              PUSH    EDI
017F:004149CF 56              PUSH    ESI
017F:004149D0 E84B5E0000      CALL    0041A820        //关键的 Call，计算密码，用 F8 跟入
017F:004149D5 83C408          ADD     ESP,BYTE +08
017F:004149D8 3BD8            CMP     EBX,EAX         //EBX 中为假序列号的十进制形式，EAX 中为真序列号的十进制形式，D EBX 和 D EAX 可分别看到
017F:004149DA 5F              POP     EDI
017F:004149DB 741D            JZ      004149FA        //不跳则出错
017F:004149DD 68CFEA0000      PUSH    DWORD EACF
017F:004149E2 6888130000      PUSH    DWORD 1388
017F:004149E7 55              PUSH    EBP
017F:004149E8 E87375FFFF      CALL    0040BF60        //出错对话框
```

-----------------------------------------------------------------------------------------

`004149D0` 处用 F8 跟入后，代码如下

```
017F:0041A820 8B442404        MOV     EAX,[ESP+04]    //使 EAX 指向姓名 gearboy
017F:0041A824 56              PUSH    ESI
017F:0041A825 8B3584D14300    MOV     ESI,[0043D184]
017F:0041A82B 50              PUSH    EAX             //参数传递给子程序
017F:0041A82C 81CE78030000    OR      ESI,0378
017F:0041A832 E8B9EBFFFF      CALL    004193F0        //计算部分和的子程序，结果入 EAX，关键，用 F8 跟入
017F:0041A837 8B4C2410        MOV     ECX,[ESP+10]    //使 ECX 指向组织名 energy
017F:0041A83B 03F0            ADD     ESI,EAX         //ESI 和 EAX 相加，结果入 ESI，注册码中间结果
017F:0041A83D 51              PUSH    ECX             //参数传递给子程序
017F:0041A83E E8ADEBFFFF      CALL    004193F0        //调用同一个计算部分和的子程序，所使用的是组织名
017F:0041A843 83C408          ADD     ESP,BYTE +08
017F:0041A846 03C6            ADD     EAX,ESI         //ESI 和 EAX 相加，结果入 EAX，此时 EAX 中即为正确注册码的十六进制形式
017F:0041A848 5E              POP     ESI
017F:0041A849 C3              RET
```

-----------------------------------------------------------------------------------------

`0041A832` 处用 F8 跟入后，代码如下

```
017F:004193F0 51              PUSH    ECX
017F:004193F1 53              PUSH    EBX
017F:004193F2 8B5C240C        MOV     EBX,[ESP+0C]        //EBX 指向姓名 gearboy
017F:004193F6 56              PUSH    ESI
017F:004193F7 33F6            XOR     ESI,ESI
017F:004193F9 53              PUSH    EBX
017F:004193FA 8974240C        MOV     [ESP+0C],ESI
017F:004193FE FF1560414200    CALL    `KERNEL32!lstrlenA`    //得到姓名长度，结果入 EAX
017F:00419404 85DB            TEST    EBX,EBX             //测试姓名是否为空
017F:00419406 744F            JZ      00419457            //不言而喻
017F:00419408 85C0            TEST    EAX,EAX             //测试姓名长度是否为零
017F:0041940A 744B            JZ      00419457            //懒得说了
017F:0041940C 33D2            XOR     EDX,EDX
017F:0041940E 85C0            TEST    EAX,EAX
017F:00419410 7E45            JNG     00419457
017F:00419412 55              PUSH    EBP
017F:00419413 57              PUSH    EDI
017F:00419414 BE00974200      MOV     ESI,00429700        //ESI 指向密钥, D 00429700 可看到
017F:00419419 BF01000000      MOV     EDI,01              //EDX 作计数器，相当于 i，i 从 1 到姓名长度 n
017F:0041941E 2BF3            SUB     ESI,EBX
017F:00419420 8BCB            MOV     ECX,EBX
017F:00419422 2BFB            SUB     EDI,EBX
017F:00419424 0FBE1C0E        MOVSX   EBX,BYTE [ESI+ECX]  //密钥 1 的第 i 字符 ASC 码符号扩展入 EBX
017F:00419428 0FBEAC10C8964200 MOVSX  EBP,BYTE [EAX+EDX+004296C8] //004296C8 指向第二个密钥，D 004296C8 可看到
017F:00419430 0FAFDD          IMUL    EBX,EBP             //两者带符号相乘
017F:00419433 8D2C0F          LEA     EBP,[EDI+ECX]       //计数值 i 入 EBP
017F:00419436 0FAFDD          IMUL    EBX,EBP             //和上面的计算结果带符号相乘
017F:00419439 0FBE29          MOVSX   EBP,BYTE [ECX]      //姓名的第 i 个字符的 ASC 码符号扩展入 EBP
017F:0041943C 0FAFDD          IMUL    EBX,EBP             //再和以上结果相乘入 EBX
017F:0041943F 8B6C2410        MOV     EBP,[ESP+10]        //将上次的结果入 EBP
017F:00419443 03EB            ADD     EBP,EBX             //上次结果和这次相加
017F:00419445 42              INC     EDX                 //计数器加 1
017F:00419446 41              INC     ECX                 //姓名指针向后移动 1 位
017F:00419447 3BD0            CMP     EDX,EAX             //检验计数值是否小于姓名长度
017F:00419449 896C2410        MOV     [ESP+10],EBP        //这次计算的中间结果入[ESP+10]
017F:0041944D 7CD5            JL      00419424            //小于姓名长度，循环
017F:0041944F 8BC5            MOV     EAX,EBP             //计算结果入 EAX
017F:00419451 5F              POP     EDI
017F:00419452 5D              POP     EBP
017F:00419453 5E              POP     ESI
017F:00419454 5B              POP     EBX
017F:00419455 59              POP     ECX
017F:00419456 C3              RET
```

密钥 1 为 `|b!pz*ls;rn|lf$vi^Axpe)rx5aic&9/2m5lsi4@0dmZw94cmqpfhw`，密钥 2 为 `#serB&nz|mfM1/5(!sd$Mq.{s]+sFjtKpzSdtzoXqmb^Al@dv:s?x/`。

所以这个子程序是由密钥 1，密钥 2 和姓名（或组织名）完成的求和运算过程。最后将这些结果相加，就是正确的注册码了。

欢迎转载，谢谢。

By 季风(`gearboy`)

2002.4.12 `4dd466d0e1e0cc123afbe504da2462ca`
