
# 扫描仪阅卷系统 V5.24

> 原文：[`www.pediy.com/kssd/pediy06/pediy6097.htm`](https://www.pediy.com/kssd/pediy06/pediy6097.htm)

软件名称：`扫描仪阅卷系统 V5.24`

作者：`nahum`

Email：`nahum@163.com`

下载地址：`http://www.skycn.com/soft/9446.html`

破解工具：`ollydb`, `pescan`, `Regmon`

破解难度：一般

### 文章简介

本文只是对这种类型的加密方法进行讨论。并非用于商业用途。请软件作者原谅。

### 功能和保护方式简介

重起验证注册号（不知道应该叫那种，生成一个 `INI` 的文件包含注册信息）

### 破解分析

下载后安装，一切正常。`OK`。运行 `GradeSheet` 看看先。

机器码：`314408964`

注册码：随便填写

点注册，提示已经写入注册信息，请重新启动软件，如不出现此窗口则表明注册成功。

好了，进行下一步。用 `Regmon` 看看是否往注册表里写东西，没有，忽然发现访问 `WINDOWS` 目录下 `GRADESHEET.INI` 文件。

打开看看内容如下：

```
[MyChoice]
Serial=314408964
information=Luo JianDa at YunLong Senior High School of CiXi City ZheJiang Province China  LUOJZNB@ZJ165.COM
```

看来 `Serial` 是关键了。进行下一步。

看看是否有壳，`PESCAN` 很好用哟。`ASPACK2.12` 脱壳完毕。运行看看是否有问题。好，没有问题运行正常。

开试用 `OLLYDB` 加载运行。搜索提示字符串 `Serial` 的找到两处，分别下断点。

其中一处如下：

```
0040E77A   . 50                PUSH EAX                              》》断到这里的时候正确的注册码已经产生了。
0040E77B   . 51                PUSH ECX
0040E77C   . 52                PUSH EDX
0040E77D   . E8 7C060100       CALL GRADESHE.0041EDFE
0040E782   . 50                PUSH EAX
0040E783   . 8D4C24 1C         LEA ECX,DWORD PTR SS:[ESP+1C]
0040E787   . C68424 D000000>MOV BYTE PTR SS:[ESP+D0],4
0040E78F   . E8 74050100       CALL GRADESHE.0041ED08
0040E794   . 8D4C24 10         LEA ECX,DWORD PTR SS:[ESP+10]
0040E798   . C68424 CC00000>MOV BYTE PTR SS:[ESP+CC],3
0040E7A0   . E8 76040100       CALL GRADESHE.0041EC1B
0040E7A5   . E8 35D70100       CALL GRADESHE.0042BEDF
0040E7AA   . 8B40 04          MOV EAX,DWORD PTR DS:[EAX+4]
0040E7AD   . 68 F4924300       PUSH GRADESHE.004392F4                 ;  ASCII "unregisted"
0040E7B2   . 68 E8914300       PUSH GRADESHE.004391E8                 ;  ASCII "Serial"参考在这里
0040E7B7   . 8D4C24 2C         LEA ECX,DWORD PTR SS:[ESP+2C]
0040E7BB   . 68 DC914300       PUSH GRADESHE.004391DC                 ;  ASCII "MyChoice"
0040E7C0   . 51                PUSH ECX
0040E7C1   . 8BC8              MOV ECX,EAX
0040E7C3   . E8 33D90100       CALL GRADESHE.0042C0FB               》》读取 INI 文件中的注册码
0040E7C8   . 8B5424 24         MOV EDX,DWORD PTR SS:[ESP+24]        》》输入的注册码
0040E7CC   . 8B4424 18         MOV EAX,DWORD PTR SS:[ESP+18]        》》正确的注册码 我的是 9902911522
0040E7D0   . 52                PUSH EDX                               ; /Arg2
0040E7D1   . 50                PUSH EAX                               ; |Arg1
0040E7D2   . C68424 D400000>MOV BYTE PTR SS:[ESP+D4],5                ; |
0040E7DA   . E8 06530000       CALL GRADESHE.00413AE5                 比较注册码 跟进
0040E7DF   . 83C4 08          ADD ESP,8
0040E7E2   . 85C0             TEST EAX,EAX
0040E7E4   . 75 0B            JNZ SHORT GRADESHE.0040E7F1
0040E7E6   . 899E F42A0000    MOV DWORD PTR DS:[ESI+2AF4],EBX
0040E7EC   . E9 CB000000      JMP GRADESHE.0040E8BC
0040E7F1   > 53                PUSH EBX                               ; /Arg1
0040E7F2   . 8D4C24 44         LEA ECX,DWORD PTR SS:[ESP+44]           ; |
0040E7F6   . E8 75E9FFFF       CALL GRADESHE.0040D170                 ; \GRADESHE.0040D170
0040E7FB   . 8D4C24 40         LEA ECX,DWORD PTR SS:[ESP+40]
0040E7FF   . C68424 CC00000>MOV BYTE PTR SS:[ESP+CC],6
0040E807   . E8 A74B0100       CALL GRADESHE.004233B3
0040E80C   . 8D8C24 C000000>LEA ECX,DWORD PTR SS:[ESP+C0]
0040E813   . C68424 CC00000>MOV BYTE PTR SS:[ESP+CC],0B
0040E81B   . E8 FB030100       CALL GRADESHE.0041EC1B
0040E820   . 8D8C24 BC00000>LEA ECX,DWORD PTR SS:[ESP+BC]
0040E827   . C68424 CC00000>MOV BYTE PTR SS:[ESP+CC],0A
0040E82F   . E8 E7030100       CALL GRADESHE.0041EC1B
```

注册码比较部分：此处是一循环，逐个比较，如果第一个不正确就跳出循环。

```
00413B14  |> 66:0FB60F        /MOVZX CX,BYTE PTR DS:[EDI]
00413B18  |. 0FB6C1           |MOVZX EAX,CL
00413B1B  |. 47               |INC EDI
00413B1C  |. 894D 0C          |MOV DWORD PTR SS:[EBP+C],ECX
00413B1F  |. F680 81B54700    >|TEST BYTE PTR DS:[EAX+47B581],4
00413B26  |. 74 16            |JE SHORT GRADESHE.00413B3E
00413B28  |. 8A07             |MOV AL,BYTE PTR DS:[EDI]
00413B2A  |. 84C0             |TEST AL,AL
00413B2C  |. 75 06            |JNZ SHORT GRADESHE.00413B34
00413B2E  |. 8365 0C 00       |AND DWORD PTR SS:[EBP+C],0
00413B32  |. EB 0A            |JMP SHORT GRADESHE.00413B3E
00413B34  |> 33D2             |XOR EDX,EDX
00413B36  |. 47               |INC EDI
00413B37  |. 8AF1             |MOV DH,CL
00413B39  |. 8AD0             |MOV DL,AL
00413B3B  |. 8955 0C          |MOV DWORD PTR SS:[EBP+C],EDX
00413B3E  |> 66:0FB61E        |MOVZX BX,BYTE PTR DS:[ESI]
00413B42  |. 0FB6C3           |MOVZX EAX,BL
00413B45  |. 46               |INC ESI
00413B46  |. F680 81B54700    >|TEST BYTE PTR DS:[EAX+47B581],4
00413B4D  |. 74 13            |JE SHORT GRADESHE.00413B62
00413B4F  |. 8A06             |MOV AL,BYTE PTR DS:[ESI]
00413B51  |. 84C0             |TEST AL,AL
00413B53  |. 75 04            |JNZ SHORT GRADESHE.00413B59
00413B55  |. 33DB             |XOR EBX,EBX
00413B57  |. EB 09            |JMP SHORT GRADESHE.00413B62
00413B59  |> 33C9             |XOR ECX,ECX
00413B5B  |. 46               |INC ESI
00413B5C  |. 8AEB             |MOV CH,BL
00413B5E  |. 8AC8             |MOV CL,AL
00413B60  |. 8BD9             |MOV EBX,ECX
00413B62  |> 66:395D 0C       |CMP WORD PTR SS:[EBP+C],BX  比较注册码的单个字符是否相同
00413B66  |. 75 09            |JNZ SHORT GRADESHE.00413B71 跳就 OVER
00413B68  |. 66:837D 0C 00    |CMP WORD PTR SS:[EBP+C],0
00413B6D  |. 74 16            |JE SHORT GRADESHE.00413B85 比较完毕跳出循环。
00413B6F  |.^EB A3            \JMP SHORT GRADESHE.00413B14
```

以上部分意义不大，感兴趣的是如何计算的注册码，到断点的以上部分看看，部分看看。

```
0040E5A0   > 8B96 F4000000    MOV EDX,DWORD PTR DS:[ESI+F4]
0040E5A6   . 57                PUSH EDI
0040E5A7   . 53                PUSH EBX
0040E5A8   . 68 03100000       PUSH 1003
0040E5AD   . 52                PUSH EDX
0040E5AE   . FFD5              CALL EBP
0040E5B0   . 50                PUSH EAX
0040E5B1   . E8 95EE0000       CALL GRADESHE.0041D44B
0040E5B6   . 83CD FF          OR EBP,FFFFFFFF
0040E5B9   . 899E F01E0000    MOV DWORD PTR DS:[ESI+1EF0],EBX
0040E5BF   . 899E 142B0000    MOV DWORD PTR DS:[ESI+2B14],EBX
0040E5C5   . 899E E41E0000    MOV DWORD PTR DS:[ESI+1EE4],EBX
0040E5CB   . 899E E01E0000    MOV DWORD PTR DS:[ESI+1EE0],EBX
0040E5D1   . 89AE E81E0000    MOV DWORD PTR DS:[ESI+1EE8],EBP
0040E5D7   . 89AE EC1E0000    MOV DWORD PTR DS:[ESI+1EEC],EBP
0040E5DD   . 899E F41E0000    MOV DWORD PTR DS:[ESI+1EF4],EBX
0040E5E3   . 899E F81E0000    MOV DWORD PTR DS:[ESI+1EF8],EBX
0040E5E9   . 899E E02A0000    MOV DWORD PTR DS:[ESI+2AE0],EBX
0040E5EF   . 899E E42A0000    MOV DWORD PTR DS:[ESI+2AE4],EBX
0040E5F5   . 899E EC2A0000    MOV DWORD PTR DS:[ESI+2AEC],EBX
0040E5FB   . 899E F02A0000    MOV DWORD PTR DS:[ESI+2AF0],EBX
0040E601   . C786 F42A0000    >MOV DWORD PTR DS:[ESI+2AF4],1
0040E60B   . A1 F8984300       MOV EAX,DWORD PTR DS:[4398F8]
0040E610   . 894424 14         MOV DWORD PTR SS:[ESP+14],EAX
0040E614   . 8D4C24 38         LEA ECX,DWORD PTR SS:[ESP+38]
0040E618   . 8D5424 28         LEA EDX,DWORD PTR SS:[ESP+28]
0040E61C   . 51                PUSH ECX                               ; /pTotalNumberOfFreeBytes
0040E61D   . 8D4424 34         LEA EAX,DWORD PTR SS:[ESP+34]           ; |
0040E621   . 52                PUSH EDX                               ; |pTotalNumberOfBytes
0040E622   . 50                PUSH EAX                               ; |pFreeBytesAvailableToCaller
0040E623   . 68 8C914300       PUSH GRADESHE.0043918C                 ; |DirectoryName = "c:"
0040E628   . 899C24 DC00000>MOV DWORD PTR SS:[ESP+DC],EBX             ; |
0040E62F   . FF15 B4F14200     CALL DWORD PTR DS:[<&KERNEL32.GetDiskFreeS>; \GetDiskFreeSpaceExA
0040E635   . 83F8 01          CMP EAX,1
0040E638   . 75 1E            JNZ SHORT GRADESHE.0040E658
0040E63A   . 8B4C24 2C         MOV ECX,DWORD PTR SS:[ESP+2C]
```
