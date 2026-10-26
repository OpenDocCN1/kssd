
# [原创]图像转换工具“魔法转换2.0”注册码分析 only for 菜鸟

题目：图像转换工具“魔法转换2.0”注册码分析

功能：功能强大的图像转换工具（吹牛？）。包括图像浏览、批量转换、图像增强及作品预览四大部分。图像增强包括调整图像大小、调整层次、旋转、镜象，还有锐化、模糊、马塞克、浮雕、底片、旋涡、喷雾等效果。你可以将这些效果自定义成方案，然后再进行批量转换。而且它还支持拖入文件到批量转换列表。同时它还能方便的将图像生成.EXE可执行文件。

工具：softice，PEID，Casper

引子：今天又实验了一下这个图片批量转换工具，实在不敢恭维，里面没有帮助，我转换图片不成功，只是对其注册码机制感兴趣，所以才有了这篇破文。拿PEID检查一下，还是用ASPack 2.000 -> Alexey Solodovnikov加壳的。用CASPER可以脱壳，方便抓取代码写破文。启动程序，打开“帮助”菜单，点击“注册”，弹出注册窗口，输入姓名wanggang，注册码654321。调出Softice，下断点`bpx hmemcpy`，F5退出，然后点击“确定”，被拦截。按7次F12即可来到主程序空间内：

```
00561263  |> 8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
00561266  |. 8B83 E8020000  MOV EAX,DWORD PTR DS:[EBX+2E8]
0056126C  |. E8 FB78EDFF    CALL MAGCT1.00438B6C
00561271  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]      //注册码地址送EAX。
00561274  |. 50             PUSH EAX
00561275  |. 8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]
00561278  |. 8B83 E4020000  MOV EAX,DWORD PTR DS:[EBX+2E4]
0056127E  |. E8 E978EDFF    CALL MAGCT1.00438B6C
00561283  |. 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]
00561286  |. 8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
00561289  |. E8 4E4CFFFF    CALL MAGCT1.00555EDC             //此函数产生注册码。分析在后面。 （1）
0056128E  |. 8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]
00561291  |. 58             POP EAX
00561292  |. E8 052FEAFF    CALL MAGCT1.0040419C             //这个函数比较真假码。分析在后面。（2）
00561297  |. 0F85 CE000000  JNZ MAGCT1.0056136B              //如果注册码正确，则这里就不跳。
0056129D  |. 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
005612A0  |. E8 EF48FFFF    CALL MAGCT1.00555B94
005612A5  |. 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
005612A8  |. BA D8135600    MOV EDX,MAGCT1.005613D8                 ;  ASCII "\win.ini"
005612AD  |. E8 E22DEAFF    CALL MAGCT1.00404094             //把注册码和用户名写入win.ini内。
005612B2  |. 8B4D EC        MOV ECX,DWORD PTR SS:[EBP-14]
005612B5  |. B2 01          MOV DL,1
005612B7  |. A1 9C344700    MOV EAX,DWORD PTR DS:[47349C]
005612BC  |. E8 2B23F1FF    CALL MAGCT1.004735EC
005612C1  |. A3 E4675700    MOV DWORD PTR DS:[5767E4],EAX
005612C6  |. 8D55 E8        LEA EDX,DWORD PTR SS:[EBP-18]
005612C9  |. 8B83 E4020000  MOV EAX,DWORD PTR DS:[EBX+2E4]
005612CF  |. E8 9878EDFF    CALL MAGCT1.00438B6C
005612D4  |. 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]
005612D7  |. 50             PUSH EAX
005612D8  |. B9 EC135600    MOV ECX,MAGCT1.005613EC                  ;  ASCII "name"
005612DD  |. BA FC135600    MOV EDX,MAGCT1.005613FC                  ;  ASCII "magct"
005612E2  |. A1 E4675700    MOV EAX,DWORD PTR DS:[5767E4]
005612E7  |. 8B30           MOV ESI,DWORD PTR DS:[EAX]
005612E9  |. FF56 04        CALL DWORD PTR DS:[ESI+4]
005612EC  |. 8D55 E4        LEA EDX,DWORD PTR SS:[EBP-1C]
005612EF  |. 8B83 E8020000  MOV EAX,DWORD PTR DS:[EBX+2E8]
005612F5  |. E8 7278EDFF    CALL MAGCT1.00438B6C
005612FA  |. 8B45 E4        MOV EAX,DWORD PTR SS:[EBP-1C]
005612FD  |. 50             PUSH EAX
005612FE  |. B9 0C145600    MOV ECX,MAGCT1.0056140C                  ;  ASCII "code"
00561303  |. BA FC135600    MOV EDX,MAGCT1.005613FC                  ;  ASCII "magct"
00561308  |. A1 E4675700    MOV EAX,DWORD PTR DS:[5767E4]
0056130D  |. 8B30           MOV ESI,DWORD PTR DS:[EAX]
0056130F  |. FF56 04        CALL DWORD PTR DS:[ESI+4]
00561312  |. A1 E4675700    MOV EAX,DWORD PTR DS:[5767E4]
00561317  |. E8 781DEAFF    CALL MAGCT1.00403094
0056131C  |. BA 1C145600    MOV EDX,MAGCT1.0056141C
00561321  |. 8B83 F4020000  MOV EAX,DWORD PTR DS:[EBX+2F4]
00561327  |. E8 7078EDFF    CALL MAGCT1.00438B9C
0056132C  |. 33D2           XOR EDX,EDX
0056132E  |. 8B83 F0020000  MOV EAX,DWORD PTR DS:[EBX+2F0]
00561334  |. 8B08           MOV ECX,DWORD PTR DS:[EAX]
00561336  |. FF51 5C        CALL DWORD PTR DS:[ECX+5C]
00561339  |. 6A 40          PUSH 40
0056133B  |. 68 24145600    PUSH MAGCT1.00561424
00561340  |. 8D55 E0        LEA EDX,DWORD PTR SS:[EBP-20]
00561343  |. 8B83 E4020000  MOV EAX,DWORD PTR DS:[EBX+2E4]
00561349  |. E8 1E78EDFF    CALL MAGCT1.00438B6C
0056134E  |. 8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
00561351  |. BA 38145600    MOV EDX,MAGCT1.00561438
00561356  |. E8 392DEAFF    CALL MAGCT1.00404094
0056135B  |. 8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]
0056135E  |. E8 ED2EEAFF    CALL MAGCT1.00404250
00561363  |. 50             PUSH EAX                                 ; |Text = "U??鹩3?E鸶?V"
00561364  |. 6A 00          PUSH 0                                   ; |hOwner = NULL
00561366  |. E8 F567EAFF    CALL <JMP.&user32.MessageBoxA>           ; \MessageBox     //显示注册成功。
```

=====================================================================================================

下面分析（1）处的函数：

```
00555EDC  /$ 55             PUSH EBP
00555EDD  |. 8BEC           MOV EBP,ESP
00555EDF  |. 83C4 F4        ADD ESP,-0C
00555EE2  |. 53             PUSH EBX
00555EE3  |. 56             PUSH ESI
00555EE4  |. 57             PUSH EDI
00555EE5  |. 33C9           XOR ECX,ECX
00555EE7  |. 894D F4        MOV DWORD PTR SS:[EBP-C],ECX
00555EEA  |. 8BFA           MOV EDI,EDX
00555EEC  |. 8BF0           MOV ESI,EAX
00555EEE  |. 33C0           XOR EAX,EAX
00555EF0  |. 55             PUSH EBP
00555EF1  |. 68 A95F5500    PUSH MAGCT1.00555FA9
00555EF6  |. 64:FF30        PUSH DWORD PTR FS:[EAX]
00555EF9  |. 64:8920        MOV DWORD PTR FS:[EAX],ESP
00555EFC  |. 85F6           TEST ESI,ESI
00555EFE  |. 0F84 8F000000  JE MAGCT1.00555F93
00555F04  |. 8BC6           MOV EAX,ESI
00555F06  |. E8 81E1EAFF    CALL MAGCT1.0040408C              //取用户名长度。
00555F0B  |. 85C0           TEST EAX,EAX
00555F0D  |. 7E 4D          JLE SHORT MAGCT1.00555F5C         //用户名长度为0则错。
00555F0F  |. 8945 F8        MOV DWORD PTR SS:[EBP-8],EAX      //用户名长度送[EBP-8]，作为循环次数。
00555F12  |. BB 01000000    MOV EBX,1                         //EBX初始化为1。
00555F17  |> 33C0           /XOR EAX,EAX
00555F19  |. 8A441E FF      |MOV AL,BYTE PTR DS:[ESI+EBX-1]   //用户名字符依次送AL。
00555F1D  |. 99             |CDQ                              //扩展到EDX。
00555F1E  |. F7FB           |IDIV EBX                         //除以EBX。EBX分别取1到用户名长度。
00555F20  |. 8945 FC        |MOV DWORD PTR SS:[EBP-4],EAX     //EAX为商，保存到[EBP-4]
00555F23  |. 8BC6           |MOV EAX,ESI
00555F25  |. E8 62E1EAFF    |CALL MAGCT1.0040408C             //取用户名长度。
00555F2A  |. 50             |PUSH EAX                         //长度压栈。
00555F2B  |. 8B45 FC        |MOV EAX,DWORD PTR SS:[EBP-4]     //把商送EAX。
00555F2E  |. 5A             |POP EDX                          //长度出栈，送EDX。
00555F2F  |. 8BCA           |MOV ECX,EDX                      //EDX送ECX。
00555F31  |. 99             |CDQ
00555F32  |. F7F9           |IDIV ECX                         //EAX再次除以长度。
00555F34  |. 33D2           |XOR EDX,EDX
00555F36  |. 8A541E FF      |MOV DL,BYTE PTR DS:[ESI+EBX-1]   //用户名字符取到DL。
00555F3A  |. 83C2 03        |ADD EDX,3                        //字符加3。
00555F3D  |. F7EA           |IMUL EDX                         //EAX乘以EDX。
00555F3F  |. 8D4D F4        |LEA ECX,DWORD PTR SS:[EBP-C]
00555F42  |. BA 03000000    |MOV EDX,3
00555F47  |. E8 BC34EBFF    |CALL MAGCT1.00409408             //此函数把得到的3字节数字变换为对应的3个ASSIC码。（3）
00555F4C  |. 8B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]     //新串地址送EDX。
00555F4F  |. 8BC7           |MOV EAX,EDI
00555F51  |. E8 3EE1EAFF    |CALL MAGCT1.00404094             //把每次得到的3位字符串接起来。
00555F56  |. 43             |INC EBX
00555F57  |. FF4D F8        |DEC DWORD PTR SS:[EBP-8]         //计数器减1。
00555F5A  |.^75 BB          \JNZ SHORT MAGCT1.00555F17        //未完则继续循环。
00555F5C  |> 8B07           MOV EAX,DWORD PTR DS:[EDI]
00555F5E  |. E8 29E1EAFF    CALL MAGCT1.0040408C              //取新串长度。
00555F63  |. 83F8 08        CMP EAX,8
00555F66  |. 7D 10          JGE SHORT MAGCT1.00555F78         //大于8则OK。
00555F68  |. 8B0F           MOV ECX,DWORD PTR DS:[EDI]
00555F6A  |. 8BC7           MOV EAX,EDI
00555F6C  |. BA C05F5500    MOV EDX,MAGCT1.00555FC0
00555F71  |. E8 62E1EAFF    CALL MAGCT1.004040D8
```
