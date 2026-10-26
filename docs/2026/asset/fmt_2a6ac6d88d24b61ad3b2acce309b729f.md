
# [原创]电影剪辑软件“VCDCutterV4.0”注册码分析 for 菜鸟

题目：电影剪辑软件“VCDCutterV4.0”注册码分析

## 软件功能

此软件为国人开发的小软件，才400KB，非常小巧玲珑。不必安装直接运行。系MPEG，VCD媒体播放器。本版本的播放器可以播放MPEG，VCD和其它媒体文件（诸如MPG，DAT，AVI，MOV，WAV）。可以从MPG或VCD碟片截取媒体画面和MPG片段(你可以选择截取MPG系统流，视频流或音频流).未注册版只可以剪辑7秒钟的电影，而且最大片段是2段（2个7秒！！）。太那个了点吧！！

## 工具

`SOFTICE`，`PEID`

## 引子

今天到一台隔壁机房电脑上查找Microsot Encarta资料，非常有名的大百科全书。结果发现几个小软件，没有注册。顺便拷贝到U盘，带回宿舍研究了一下。首先就是这个国人开发的小工具，可能是印度老师安装上去的，因为界面全部是英语的，所以外国人也能够使用。分析了1个小时就得出结论了。赶紧把破解过程撰写出来，以便分享俺的快乐！！首先拿`PEID`查看，没有加壳，便于倒出代码写文章了。是VC写的。启动程序，打开help菜单，单击Register，弹出注册窗口，输入用户名wanggang,注册码654321，注意窗口上显示出了一个根据CMOS信息计算得到的ID号。调出`SOFTICE`，设断点`bpx hmemcpy`，（用`getwindowtexta`无效），F5退出，单击OK，被拦住。按一阵F12，来到主程序空间。

## 代码分析

代码分析如下：

```
0042A244   > 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]
0042A248   . 6A 40              PUSH 40
0042A24A   . 50                 PUSH EAX
0042A24B   . 68 01040000        PUSH 401
0042A250   . 56                 PUSH ESI
0042A251   . FFD5               CALL EBP                                    //取用户名。
0042A253   . 8D4C24 50          LEA ECX,DWORD PTR SS:[ESP+50]               //我们返回到这里。
0042A257   . 6A 40              PUSH 40
0042A259   . 51                 PUSH ECX
0042A25A   . 68 02040000        PUSH 402
0042A25F   . 56                 PUSH ESI
0042A260   . FFD5               CALL EBP                                    //取注册码。
0042A262   . 8D5424 50          LEA EDX,DWORD PTR SS:[ESP+50]               //注册码地址送EDX。
0042A266   . 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]               //用户名地址送EDX。
0042A26A   . 52                 PUSH EDX
0042A26B   . 50                 PUSH EAX
0042A26C   . E8 0F3CFFFF        CALL VCDCUT.0041DE80                        //注册码产生与比较的函数。  （1）
0042A271   . 83C4 08            ADD ESP,8
0042A274   . 85C0               TEST EAX,EAX                                //EAX返回0则错，返回1则成功。
0042A276   . 0F84 83000000      JE VCDCUT.0042A2FF
0042A27C   . 8B0D 280D4900      MOV ECX,DWORD PTR DS:[490D28]
0042A282   . 51                 PUSH ECX
0042A283   . FF15 DC744900      CALL DWORD PTR DS:[<&GDI32.DeleteObject>]
0042A289   . 8B15 840C4900      MOV EDX,DWORD PTR DS:[490C84]
0042A28F   . 6A 7B              PUSH 7B
0042A291   . 52                 PUSH EDX
0042A292   . FF15 64764900      CALL DWORD PTR DS:[<&USER32.LoadBitmapA>]
0042A298   . A3 280D4900        MOV DWORD PTR DS:[490D28],EAX
0042A29D   . 8D4424 50          LEA EAX,DWORD PTR SS:[ESP+50]
0042A2A1   . 6A 01              PUSH 1
0042A2A3   . 8D4C24 14          LEA ECX,DWORD PTR SS:[ESP+14]
0042A2A7   . 50                 PUSH EAX
0042A2A8   . 51                 PUSH ECX
0042A2A9   . E8 4236FFFF        CALL VCDCUT.0041D8F0
0042A2AE   . 83C4 0C            ADD ESP,0C
0042A2B1   . 6A 01              PUSH 1
0042A2B3   . 56                 PUSH ESI
0042A2B4   . FF15 E8764900      CALL DWORD PTR DS:[<&USER32.EndDialog>]
0042A2BA   . 8B15 58CA4700      MOV EDX,DWORD PTR DS:[47CA58]
0042A2C0   . 52                 PUSH EDX
0042A2C1   . FF15 EC764900      CALL DWORD PTR DS:[<&USER32.SetActiveWindow>]
0042A2C7   . 8B0D 58CA4700      MOV ECX,DWORD PTR DS:[47CA58]
0042A2CD   . 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]
0042A2D1   . 6A 00              PUSH 0
0042A2D3   . 50                 PUSH EAX
0042A2D4   . 68 C4294400        PUSH VCDCUT.004429C4 ;  ASCII "Thanks for your support !!!Registeration is succeed !!!"
0042A2D9   . 51                 PUSH ECX
0042A2DA   . FFD3               CALL EBX                                       //显示成功对话框。
0042A2DC   . 8B15 58CA4700      MOV EDX,DWORD PTR DS:[47CA58]
0042A2E2   . 6A 00              PUSH 0
0042A2E4   . 6A 00              PUSH 0
0042A2E6   . 52                 PUSH EDX
0042A2E7   . FF15 68764900      CALL DWORD PTR DS:[<&USER32.InvalidateRect>]
0042A2ED   . B8 01000000        MOV EAX,1
0042A2F2   . 5F                 POP EDI
0042A2F3   . 5E                 POP ESI
0042A2F4   . 5D                 POP EBP
0042A2F5   . 5B                 POP EBX
0042A2F6   . 81C4 80030000      ADD ESP,380
0042A2FC   . C2 1000            RETN 10
```

## 注册码产生与比较函数分析

下面分析（1）处的函数代码：

```
0041DE80  /$ 81EC 08020000      SUB ESP,208
0041DE86  |. 83C9 FF            OR ECX,FFFFFFFF
0041DE89  |. 33C0               XOR EAX,EAX
0041DE8B  |. 8D5424 08          LEA EDX,DWORD PTR SS:[ESP+8]
0041DE8F  |. 53                 PUSH EBX
0041DE90  |. 56                 PUSH ESI
0041DE91  |. 57                 PUSH EDI
0041DE92  |. BF 10144400        MOV EDI,VCDCUT.00441410        ;  ASCII "abmz0129"
0041DE97  |. F2:AE              REPNE SCAS BYTE PTR ES:[EDI]                  //把 "abmz0129"串拷贝到另外地方。记为S1。
0041DE99  |. F7D1               NOT ECX
0041DE9B  |. 2BF9               SUB EDI,ECX
0041DE9D  |. 8BC1               MOV EAX,ECX
0041DE9F  |. 8BF7               MOV ESI,EDI
0041DEA1  |. 8BFA               MOV EDI,EDX
0041DEA3  |. 33D2               XOR EDX,EDX
0041DEA5  |. C1E9 02            SHR ECX,2
0041DEA8  |. F3:A5              REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]
0041DEAA  |. 8BC8               MOV ECX,EAX
0041DEAC  |. 83E1 03            AND ECX,3
0041DEAF  |. F3:A4              REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
0041DEB1  |. 8BB424 18020000    MOV ESI,DWORD PTR SS:[ESP+218]
0041DEB8  |. 8D4C24 14          LEA ECX,DWORD PTR SS:[ESP+14]
0041DEBC  |. 8BDE               MOV EBX,ESI
0041DEBE  |. 2BD9               SUB EBX,ECX
0041DEC0  |> 8BFE               /MOV EDI,ESI
0041DEC2  |. 83C9 FF            |OR ECX,FFFFFFFF
0041DEC5  |. 33C0               |XOR EAX,EAX
0041DEC7  |. F2:AE              |REPNE SCAS BYTE PTR ES:[EDI]                //扫描用户名，计算长度。
0041DEC9  |. F7D1               |NOT ECX
0041DECB  |. 49                 |DEC ECX
0041DECC  |. 83F9 08            |CMP ECX,8
0041DECF  |. 76 07              |JBE SHORT VCDCUT.0041DED8                   //小于等于8位跳走。
0041DED1  |. B9 08000000        |MOV ECX,8
0041DED6  |. EB 0C              |JMP SHORT VCDCUT.0041DEE4
0041DED8  |> 8BFE               |MOV EDI,ESI
0041DEDA  |. 83C9 FF            |OR ECX,FFFFFFFF
0041DEDD  |. 33C0               |XOR EAX,EAX
0041DEDF  |. F2:AE              |REPNE SCAS BYTE PTR ES:[EDI]
0041DEE1  |. F7D1               |NOT ECX
0041DEE3  |. 49                 |DEC ECX
0041DEE4  |> 3BD1               |CMP EDX,ECX
0041DEE6  |. 73 0E              |JNB SHORT VCDCUT.0041DEF6
0041DEE8  |. 8D4414 14          |LEA EAX,DWORD PTR SS:[ESP+EDX+14]           //S1串地址送EAX。
0041DEEC  |. 42                 |INC EDX
0041DEED  |. 8A0C18             |MOV CL,BYTE PTR DS:[EAX+EBX]                //用户名字符送CL。
0041DEF0  |. 884C14 13          |MOV BYTE PTR SS:[ESP+EDX+13],CL             //用户名覆盖掉串S1。
0041DEF4  |.^EB CA              \JMP SHORT VCDCUT.0041DEC0
0041DEF6  |> C64414 14 00       MOV BYTE PTR SS:[ESP+EDX+14],0
0041DEFB  |. 6A 01              PUSH 1
0041DEFD  |. 68 B8FC4400        PUSH VCDCUT.0044FCB8
0041DF02  |. 8D5424 1C          LEA EDX,DWORD PTR SS:[ESP+1C]                //用户名地址送EDX。
0041DF06  |. 6A 08              PUSH 8
0041DF08  |. 52                 PUSH EDX
0041DF09  |. 68 98134400        PUSH VCDCUT.00441398
0041DF0E  |. E8 AD8AFFFF        CALL VCDCUT.004169C0                         //注册码计算就在这个函数内了。（2）
0041DF13  |. 83C4 14            ADD ESP,14
0041DF16  |. 8D4424 14          LEA EAX,DWORD PTR SS:[ESP+14]
0041DF1A  |. 8D4C24 0C          LEA ECX,DWORD PTR SS:[ESP+C]
0041DF1E  |. 6A 04              PUSH 4
0041DF20  |. 50                 PUSH EAX
0041DF21  |. 51                 PUSH ECX
0041DF22  |. E8 49680100        CALL <JMP.&MSVCRT.memmove>
0041DF27  |. 83C4 0C            ADD ESP,0C
0041DF2A  |. 8D5424 18          LEA EDX,DWORD PTR SS:[ESP+18]
0041DF2E  |. 8D4424 10          LEA EAX,DWORD PTR SS:[ESP+10]
0041DF32  |. 6A 04              PUSH 4
0041DF34  |. 52                 PUSH EDX
0041DF35  |. 50                 PUSH EAX
0041DF36  |. E8 35680100        CALL <JMP.&MSVCRT.memmove>
0041DF3B  |. 83C4 0C            ADD ESP,0C
```
