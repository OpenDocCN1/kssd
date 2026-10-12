
# 魔镜系列之心灵感应

> 原文：[`www.pediy.com/kssd/pediy06/pediy6088.htm`](https://www.pediy.com/kssd/pediy06/pediy6088.htm)

`<<魔镜系列之心灵感应>>`注册算法分析

作者：`lordor[BCG]`

Mail:`lordor@sina.com`

目的：属技术交流，无其它目的，请不要任意散布或用用商业用途。初学破解,如有不对的地方欢迎批评指出。

工具：`softice`,`w32Dasm`,`ollydbg`

试炼码：

机器码：`558109930`

注册码：`654321`

一看是 VB 程序，心里就有点怕，而且还用到浮点运算，心里更怕，但看了一下，注册算法还算简单，于是就凑合着写篇教程。以下为从 `od` 中摘取的代码，“；”后是我加的注解。

```
00999837   .  50                 PUSH EAX
00999838   .  57                 PUSH EDI
00999839   .  8B17               MOV EDX,DWORD PTR DS:[EDI]
0099983B   .  FF92 A0000000 CALL DWORD PTR DS:[EDX+A0]               ;  取输入的注册码 654321
00999841   .  3BC6               CMP EAX,ESI
00999843   .  DBE2               FCLEX                                 ;  清除所有异常标志
00999845   .  7D 12              JGE SHORT mj.00999859
00999847   .  68 A0000000        PUSH 0A0
0099984C   .  68 0CE84500        PUSH mj.0045E80C
00999851   .  57                 PUSH EDI
00999852   .  50                 PUSH EAX
00999853   .  FF15 58104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaHresu>;  MSVBVM60.__vbaHresultCheckObj
00999859   >  8B4D E8            MOV ECX,DWORD PTR SS:[EBP-18]         ;  输入的注册码入 ecx
0099985C   .  51                 PUSH ECX
0099985D   .  FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.#581>]     ;  MSVBVM60.rtcR8ValFromBstr
00999863   .  FF15 8C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpR8>>;  MSVBVM60.__vbaFpR8
00999869   .  DD9D 20FFFFFF FSTP QWORD PTR SS:[EBP-E0]                ;  上面注册码转换为浮点数。此句为把转换注册码存入[ebp-e0]
0099986F   .  DB85 4CFFFFFF FILD DWORD PTR SS:[EBP-B4]                ;  取机器码，此为 558109930
00999875   .  DD9D 18FFFFFF FSTP QWORD PTR SS:[EBP-E8]                ;  保存
0099987B   .  DD85 18FFFFFF FLD QWORD PTR SS:[EBP-E8]                 ;  取机器码，入 st0
00999881   .  833D 00209A00>CMP DWORD PTR DS:[9A2000],0               ;  判断机器码是否为 0
00999888   .  75 08              JNZ SHORT mj.00999892
0099988A   .  DC35 A0114000 FDIV QWORD PTR DS:[4011A0]               ;  机器码除以 44221(d)
00999890   .  EB 11              JMP SHORT mj.009998A3               ;  得数 12620.925126071323579294905135569
00999892   >  FF35 A4114000 PUSH DWORD PTR DS:[4011A4]
00999898   .  FF35 A0114000 PUSH DWORD PTR DS:[4011A0]
0099989E   .  E8 817EA6FF       CALL <JMP.&MSVBVM60._adj_fdiv_m64>
009998A3   >  DC05 98114000 FADD QWORD PTR DS:[401198]               ;  然后加上 427616（d）
009998A9   .  DFE0               FSTSW AX                              ;  保存浮点状态字
009998AB   .  A8 0D              TEST AL,0D
009998AD   .  0F85 31040000 JNZ mj.00999CE4
009998B3   .  FF15 78114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFPInt>;  MSVBVM60.__vbaFPInt
009998B9   .  FF15 8C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpR8>>;  MSVBVM60.__vbaFpR8
009998BF   .  DC9D 20FFFFFF FCOMP QWORD PTR SS:[EBP-E0]              ;  上面为将浮点数转换为整数，这句是比较与输入的注册码是否相等
009998C5   .  DFE0               FSTSW AX                              ;  保存浮点状态字
009998C7   .  F6C4 40            TEST AH,40
009998CA   .  74 07              JE SHORT mj.009998D3
009998CC   .  B8 01000000        MOV EAX,1
009998D1   .  EB 02              JMP SHORT mj.009998D5
009998D3   >  33C0               XOR EAX,EAX
009998D5   >  F7D8               NEG EAX
009998D7   .  66:8BF8            MOV DI,AX
009998DA   .  8D55 E4            LEA EDX,DWORD PTR SS:[EBP-1C]
009998DD   .  8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]         ;  注册码入 eax
009998E0   .  52                 PUSH EDX
009998E1   .  50                 PUSH EAX
009998E2   .  6A 02              PUSH 2
009998E4   .  FF15 30114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStrList
009998EA   .  83C4 0C            ADD ESP,0C
009998ED   .  8D4D D4            LEA ECX,DWORD PTR SS:[EBP-2C]
009998F0   .  FF15 88114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeO>;  MSVBVM60.__vbaFreeObj
009998F6   .  66:3BFE            CMP DI,SI
009998F9     0F84 8E020000 JE mj.00999B8D                              ;  出错
009998FF   .  8B0B               MOV ECX,DWORD PTR DS:[EBX]
00999901   .  53                 PUSH EBX
00999902   .  FF91 FC020000 CALL DWORD PTR DS:[ECX+2FC]
```

---

### 总结

算法过程：机器码除以 `44221`，得数 `A`，然后 `A` 加上 `427616`，取整数部分即为注册码。

机器码：`558109930`

注册码：`440236`

注册码保存在：

`[HKEY_LOCAL_MACHINE\SOFTWARE\select\wally]`

`"mpsd"="440236"`

暗桩：注册成功后，要按作者的操作说明作相应的解码，可能是要按什么键的。如不，则会提示“请按注册后收到的说明操作，非法解码无效。”。可以用 `rtcMsg` 下断，定位相关核心部分。

下面举例说明“心灵感应/扑克心灵感应术”模块去掉解码失败提示及解除次数限制。

让我们比较一下在没注册时正常运行及提示解码失败的程序运行的情况：

正常试用时为，会记录次数：

```
00993F34   .  FF51 04            CALL DWORD PTR DS:[ECX+4]           ＝＝＞在这下断
00993F37   .  0FBF86 940000>MOVSX EAX,WORD PTR DS:[ESI+94]           ;  提示失败：eax=1,bx=1。试用为：ea=1,ebx=1。(eax 中，1 表示第一模块，2 为第二模块)
00993F3E   .  33FF               XOR EDI,EDI                          ;  Switch (cases 1..3)
00993F40   .  48                 DEC EAX
00993F41   .  897D E8            MOV DWORD PTR SS:[EBP-18],EDI
00993F44   .  897D E4            MOV DWORD PTR SS:[EBP-1C],EDI
00993F47   .  897D E0            MOV DWORD PTR SS:[EBP-20],EDI
00993F4A   .  897D DC            MOV DWORD PTR SS:[EBP-24],EDI
00993F4D   .  897D D8            MOV DWORD PTR SS:[EBP-28],EDI
00993F50   .  897D D4            MOV DWORD PTR SS:[EBP-2C],EDI
00993F53   .  897D D0            MOV DWORD PTR SS:[EBP-30],EDI
00993F56   .  897D CC            MOV DWORD PTR SS:[EBP-34],EDI
00993F59   .  897D C8            MOV DWORD PTR SS:[EBP-38],EDI
00993F5C   .  897D B8            MOV DWORD PTR SS:[EBP-48],EDI
00993F5F   .  897D A8            MOV DWORD PTR SS:[EBP-58],EDI
00993F62   .  897D 98            MOV DWORD PTR SS:[EBP-68],EDI
00993F65   .  897D 88            MOV DWORD PTR SS:[EBP-78],EDI
00993F68   .  89BD 78FFFFFF MOV DWORD PTR SS:[EBP-88],EDI
00993F6E   .  89BD 68FFFFFF MOV DWORD PTR SS:[EBP-98],EDI
00993F74   .  89BD 44FFFFFF MOV DWORD PTR SS:[EBP-BC],EDI
00993F7A   .  89BD 40FFFFFF MOV DWORD PTR SS:[EBP-C0],EDI
00993F80   .  89BD 3CFFFFFF MOV DWORD PTR SS:[EBP-C4],EDI
00993F86   .  0F84 43060000 JE mj.009945CF                             ;  提示失败：此为跳。试用也会跳。（第一模块）
00993F8C   .  48                 DEC EAX
00993F8D   . /0F84 71010000 JE mj.00994104　　　　　　　　　　　　　；（执行第二模块）
00993F93   . |48                DEC EAX
00993F94   . |0F85 94100000 JNZ mj.0099502E
00993F9A   . |66:391D 2C209>CMP WORD PTR DS:[9A202C],BX               ;  Case 3 of switch 00993F3E
00993FA1   . |0F85 8C000000 JNZ mj.00994033
00993FA7   . |8B3D 50114000 MOV EDI,DWORD PTR DS:[<&MSVBVM60.__vbaVa>;  MSVBVM60.__vbaVarDup

........(省略)........

009945C5   .  68 F4E14500        PUSH mj.0045E1F4
009945CA   .  E9 570A0000        JMP mj.00995026                       ;  下句中：试用时为 DS:[9A202C]＝2（试用次数），提示注册时 DS:[9A202C]＝1
009945CF   >  66:391D 2C209>CMP WORD PTR DS:[9A202C],BX               ;  由上面跳到这。
009945D6   .^ 0F85 35FBFFFF JNZ mj.00994111                            ;  提示失败：此跳走,如果在这 nop 掉，就会提示要注册。试用：还有两次则不跳走。所以要成功一定得 nop 掉
009945DC   .  8B35 2C114000 MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaSt>;  MSVBVM60.__vbaStrCopy
009945E2   .  BA 44DD4500        MOV EDX,mj.0045DD44                  ;  UNICODE "timest"
009945E7   .  8D4D E4            LEA ECX,DWORD PTR SS:[EBP-1C]
009945EA   .  FFD6               CALL ESI                              ;  <&MSVBVM60.__vbaStrCopy>
009945EC   .  BA 10DD4500        MOV EDX,mj.0045DD10                  ;  UNICODE "system\\software\\timest1"
009945F1   .  8D4D E8            LEA ECX,DWORD PTR SS:[EBP-18]
009945F4   .  FFD6               CALL ESI
009945F6   .  8D55 E4            LEA EDX,DWORD PTR SS:[EBP-1C]
009945F9   .  8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]
009945FC   .  52                 PUSH EDX
009945FD   .  8D8D 44FFFFFF LEA ECX,DWORD PTR SS:[EBP-BC]
00994603   .  50                 PUSH EAX
00994604   .  51                 PUSH ECX
00994605   .  C785 44FFFFFF>MOV DWORD PTR SS:[EBP-BC],80000002
```



```
0099460F   .  E8 EC680000    CALL mj.0099AF00                       ;  次数写入注册表中  

00994614   .  8D55 B8        LEA EDX,DWORD PTR SS:[EBP-48]  

00994617   .  B9 40209A00    MOV ECX,mj.009A2040  

0099461C   .  8945 C0        MOV DWORD PTR SS:[EBP-40],EAX  

0099461F   .  C745 B8 08000>MOV DWORD PTR SS:[EBP-48],8  

00994626   .  FF15 18104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarMo>;  MSVBVM60.__vbaVarMove  

0099462C   .  8B3D 30114000 MOV EDI,DWORD PTR DS:[<&MSVBVM60.__vbaFr>;  MSVBVM60.__vbaFreeStrList  

00994632   .  8D55 E4        LEA EDX,DWORD PTR SS:[EBP-1C]  

00994635   .  8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]  

00994638   .  52             PUSH EDX  

00994639   .  50             PUSH EAX  

0099463A   .  6A 02          PUSH 2  

0099463C   .  FFD7          CALL EDI                               ;  <&MSVBVM60.__vbaFreeStrList>  

0099463E   .  83C4 0C        ADD ESP,0C  

00994641   .  8D4D E8        LEA ECX,DWORD PTR SS:[EBP-18]  

00994644   .  68 40209A00    PUSH mj.009A2040                       ;  这里是关键  

00994649   .  51             PUSH ECX  

0099464A   .  FF15 00114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarVal  

00994650   .  50             PUSH EAX  

00994651   .  FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.#581>]    ;  MSVBVM60.rtcR8ValFromBstr  

00994657   .  FF15 8C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpR8>>;  MSVBVM60.__vbaFpR8  

0099465D   .  DC1D 98124000 FCOMP QWORD PTR DS:[401298]  

00994663   .  DFE0          FSTSW AX  

00994665   .  F6C4 40       TEST AH,40  

00994668   .  74 07         JE SHORT mj.00994671                   ;  试用：跳走。失败：跳  

0099466A   .  BB 01000000    MOV EBX,1  

0099466F   .  EB 02         JMP SHORT mj.00994673  

00994671   >  33DB          XOR EBX,EBX  

00994673   >  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  

00994676   .  FF15 84114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr  

0099467C   .  F7DB          NEG EBX  

0099467E   .  66:85DB      TEST BX,BX  

00994681   .  0F84 FD030000 JE mj.00994A84                         ;  试用：跳走  

00994687   .  BA 00DD4500    MOV EDX,mj.0045DD00  

0099468C   .  8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20] 

........(省略)........ 

00994A7F   . /E9 3A040000    JMP mj.00994EBE  

00994A84   > |8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]　　　＝＝＝＞由上面 `00994681` 处跳来  

00994A87   . |68 40209A00    PUSH mj.009A2040  

00994A8C   . |50             PUSH EAX  

00994A8D   . |FF15 00114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarVal  

00994A93   . |50             PUSH EAX  

00994A94   . |FF15 8C114000 CALL DWORD PTR DS:[<&MSVBVM60.#581>]    ;  MSVBVM60.rtcR8ValFromBstr  

00994A9A   . |FF15 8C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFpR8>>;  MSVBVM60.__vbaFpR8  

00994AA0   . |DC1D 90124000 FCOMP QWORD PTR DS:[401290]  

00994AA6   . |DFE0          FSTSW AX  

00994AA8   . |F6C4 40       TEST AH,40  

00994AAB   . |75 07         JNZ SHORT mj.00994AB4                  ;  试用：不跳。失败：跳。所以这里也得 nop 掉  

00994AAD   . |BB 01000000    MOV EBX,1  

00994AB2   . |EB 02         JMP SHORT mj.00994AB6  

00994AB4   > |33DB          XOR EBX,EBX  

00994AB6   > |8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]  

00994AB9   . |FF15 84114000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr  

00994ABF   . |F7DB          NEG EBX  

00994AC1   . |66:85DB      TEST BX,BX  

00994AC4   . |0F84 88040000 JE mj.00994F52                         ;  试用：不跳 00994  

00994ACA   . |BA 28E14500    MOV EDX,mj.0045E128  

00994ACF   . |8D4D E0        LEA ECX,DWORD PTR SS:[EBP-20]  

00994AD2   . |C785 3CFFFFFF>MOV DWORD PTR SS:[EBP-C4],2  

00994ADC   . |FFD6          CALL ESI  

00994ADE   . |BA 44DD4500    MOV EDX,mj.0045DD44                   ;  UNICODE "timest"  

00994AE3   . |8D4D E4        LEA ECX,DWORD PTR SS:[EBP-1C]  

00994AE6   . |C785 40FFFFFF>MOV DWORD PTR SS:[EBP-C0],1 
```

`-------------------------------------------------------`

破掉提示解码失败及次数限制，只在把下面两句 nop 掉即可

```
009945D6   .^ 0F85 35FBFFFF JNZ mj.00994111     

00994AAB   . |75 07         JNZ SHORT mj.00994AB4  
```

其它几个模块都可以用 `rtcMsg` 下断，作相应的破解，在此不作详述。 

如果大家有更好的方法去掉那些非法解码无效的提示，希望告知我。 

cracked by `lordor[BCG]`

03.6.1

`4dd466d0e1e0cc123afbe504da2462ca`
