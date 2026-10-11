
# xCDKing 光盘管理

> 原文：[`www.pediy.com/kssd/pediy06/pediy6131.htm`](https://www.pediy.com/kssd/pediy06/pediy6131.htm)

**【软件介绍】**：

xCDKing 光盘管理能很好很方便地管理您的光盘或硬盘资料。具有树状的分类方法、资料的自动导入功能、关键字查找功能、密码设置、备注注释等功能。让您自如地掌管您的光盘资料。

**【下载地址】**： [`www.softreg.com.cn/download.asp?id=/B01CA03C-0CA1-44B7-8C1E-B3E564A900AD/`](http://www.softreg.com.cn/download.asp?id=/B01CA03C-0CA1-44B7-8C1E-B3E564A900AD/)

**【软件限制】**：还没注意。

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！如果你有条件，请支持国产软件。

**【作 者】**：abcde12345【DFCG】

**【破解工具】**：`Ollydbg1.09`、`Language2K`、`Api32`

—————————————————————————————————

**【过 程】**：

首先，我说一下，因为它要用到 `Key` 文件，根据它提供的 `Key` 类型，我造了一个，它却不认识。没办法，只好选通过 `w32DASM` 找到注册表键值，手工在注册表造了一个主键，里面再造两个，一个是用户名，一个是密码。（下下之策）

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\ECDKing]

"ECDkingUserName"="abcde12345"

"ECDKingNumber"="12345678"
```

1、用 `Language2K` 检查 无壳,Delphi 程序。

2、因它是重启验证，故运行 `Api32` 调试程序，检查会用到那些函数，并分析可疑对象。

找到：

```
0046CAA3:RegOpenKeyExA(HANDLE:C1A1F4D0,LPSTR:004EF1CC:"ECDKing",DWORD:00000000,DWORD:00020009,LPDATA:0079FC34)

0046CAA8:RegOpenKeyExA = 0
```

3、用 `Ollydbg1.09` 调试该程序，在得到的地址 `0046CAA3` 处设置断点。

5、`F9` 运行后，中断在我们设的断点上。按 `F8` 数次后，终于看见了我的注册信息，等你等到我心痛。

```
004EF084  /$  55             PUSH EBP  
004EF085  |.  8BEC           MOV EBP,ESP  
004EF087  |.  6A 00          PUSH 0  
004EF089  |.  6A 00          PUSH 0  
004EF08B  |.  6A 00          PUSH 0  
004EF08D  |.  53             PUSH EBX  
004EF08E  |.  56             PUSH ESI  
004EF08F  |.  8BF0          MOV ESI,EAX  
004EF091  |.  33C0          XOR EAX,EAX  
004EF093  |.  55             PUSH EBP  
004EF094  |.  68 83F14E00    PUSH XCDKING.004EF183  
004EF099  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
004EF09C  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP  
004EF09F  |.  B2 01          MOV DL,1  
004EF0A1  |.  A1 90C34600    MOV EAX,DWORD PTR DS:[46C390]  
004EF0A6  |.  E8 51D4F7FF    CALL XCDKING.0046C4FC  
004EF0AB  |.  8BD8           MOV EBX,EAX  
004EF0AD  |.  BA 02000080    MOV EDX,80000002  
004EF0B2  |.  8BC3           MOV EAX,EBX  
004EF0B4  |.  E8 1FD5F7FF    CALL XCDKING.0046C5D8  
004EF0B9  |.  B1 01          MOV CL,1  
004EF0BB  |.  BA 98F14E00    MOV EDX,XCDKING.004EF198               ;  ASCII "software\microsoft\windows\currentversion\"  
004EF0C0  |.  8BC3           MOV EAX,EBX  
004EF0C2  |.  E8 55D6F7FF    CALL XCDKING.0046C71C  
004EF0C7  |.  BA CCF14E00    MOV EDX,XCDKING.004EF1CC               ;  ASCII "ECDKing"  
004EF0CC  |.  8BC3           MOV EAX,EBX  
004EF0CE  |.  E8 FDD9F7FF    CALL XCDKING.0046CAD0  
004EF0D3  |.  84C0           TEST AL,AL  
004EF0D5  |.  75 0B          JNZ SHORT XCDKING.004EF0E2  
004EF0D7  |.  A1 38C94F00    MOV EAX,DWORD PTR DS:[4FC938]  
004EF0DC  |.  33D2           XOR EDX,EDX  
004EF0DE  |.  8910           MOV DWORD PTR DS:[EAX],EDX  
004EF0E0  |.  EB 7F          JMP SHORT XCDKING.004EF161  
004EF0E2  |>  B1 01          MOV CL,1  
004EF0E4  |.  BA CCF14E00    MOV EDX,XCDKING.004EF1CC               ;  ASCII "ECDKing"  
004EF0E9  |.  8BC3           MOV EAX,EBX  
004EF0EB  |.  E8 2CD6F7FF    CALL XCDKING.0046C71C  
004EF0F0  |.  8D4D FC        LEA ECX,DWORD PTR SS:[EBP-4]  
004EF0F3  |.  BA DCF14E00    MOV EDX,XCDKING.004EF1DC               ;  ASCII "ECDKingUserName"  
004EF0F8  |.  8BC3           MOV EAX,EBX  
004EF0FA  |.  E8 E5D7F7FF    CALL XCDKING.0046C8E4  
004EF0FF  |.  8D4D F8        LEA ECX,DWORD PTR SS:[EBP-8]  
004EF102  |.  BA F4F14E00    MOV EDX,XCDKING.004EF1F4               ;  ASCII "ECDKingNumber"  
004EF107  |.  8BC3           MOV EAX,EBX  
004EF109  |.  E8 D6D7F7FF    CALL XCDKING.0046C8E4  
004EF10E  |.  837D FC 00     CMP DWORD PTR SS:[EBP-4],0  
004EF112  |.  74 06          JE SHORT XCDKING.004EF11A  
004EF114  |.  837D F8 00     CMP DWORD PTR SS:[EBP-8],0  
004EF118  |.  75 0B          JNZ SHORT XCDKING.004EF125  
004EF11A  |>  A1 38C94F00    MOV EAX,DWORD PTR DS:[4FC938]  
004EF11F  |.  33D2           XOR EDX,EDX  
004EF121  |.  8910           MOV DWORD PTR DS:[EAX],EDX  
004EF123  |.  EB 32          JMP SHORT XCDKING.004EF157  
004EF125  |>  8D4D F4        LEA ECX,DWORD PTR SS:[EBP-C]  
004EF128  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]  
004EF12B  |.  8BC6           MOV EAX,ESI  
004EF12D  |.  E8 9E0D0000    CALL XCDKING.004EFED0  
004EF132  |.  8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]  
004EF135  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]  
004EF138  |.  E8 C75CF1FF    CALL XCDKING.00404E04                ；真假码在这里比较  
004EF13D  |.  75 0F          JNZ SHORT XCDKING.004EF14E  
004EF13F  |.  A1 B8C94F00    MOV EAX,DWORD PTR DS:[4FC9B8]  
004EF144  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]  
004EF147  |.  E8 1059F1FF    CALL XCDKING.00404A5C  
004EF14C  |.  EB 09          JMP SHORT XCDKING.004EF157  
004EF14E  |>  A1 38C94F00    MOV EAX,DWORD PTR DS:[4FC938]  
004EF153  |.  33D2           XOR EDX,EDX  
004EF155  |.  8910           MOV DWORD PTR DS:[EAX],EDX  
004EF157  |>  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]  
004EF15A  |.  8BC6           MOV EAX,ESI  
004EF15C  |.  E8 D7250000    CALL XCDKING.004F1738  
004EF161  |>  8BC3           MOV EAX,EBX  
004EF163  |.  E8 D049F1FF    CALL XCDKING.00403B38  
004EF168  |.  33C0           XOR EAX,EAX  
004EF16A  |.  5A             POP EDX  
004EF16B  |.  59             POP ECX  
004EF16C  |.  59             POP ECX  
004EF16D  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX  
004EF170  |.  68 8AF14E00    PUSH XCDKING.004EF18A  
004EF175  |>  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]  
004EF178  |.  BA 03000000    MOV EDX,3  
004EF17D  |.  E8 AA58F1FF    CALL XCDKING.00404A2C  
004EF182  \\.  C3             RETN 

//////////////////  
//关键 Call  
//////////////////  

004EFED0  /$  55             PUSH EBP  
004EFED1  |.  8BEC           MOV EBP,ESP  
004EFED3  |.  6A 00          PUSH 0  
004EFED5  |.  6A 00          PUSH 0  
004EFED7  |.  6A 00          PUSH 0  
004EFED9  |.  6A 00          PUSH 0  
004EFEDB  |.  6A 00          PUSH 0  
004EFEDD  |.  6A 00          PUSH 0  
004EFEDF  |.  6A 00          PUSH 0  
004EFEE1  |.  6A 00          PUSH 0  
004EFEE3  |.  53             PUSH EBX  
004EFEE4  |.  56             PUSH ESI  
004EFEE5  |.  57             PUSH EDI  
004EFEE6  |.  8BF9          MOV EDI,ECX  
004EFEE8  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX  
004EFEEB  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]  
004EFEEE  |.  E8 B54FF1FF    CALL XCDKING.00404EA8  
004EFEF3  |.  33C0          XOR EAX,EAX  
004EFEF5  |.  55             PUSH EBP  
004EFEF6  |.  68 26004F00    PUSH XCDKING.004F0026  
004EFEFB  |.  64:FF30        PUSH DWORD PTR FS:[EAX]  
004EFEFE  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP  
004EFF01  |.  8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]  
004EFF04  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]  
004EFF07  |.  E8 944BF1FF    CALL XCDKING.00404AA0  
004EFF0C  |.  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]  
004EFF0F  |.  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]  
004EFF12  |.  0FB600         MOVZX EAX,BYTE PTR DS:[EAX]     ；把用户名第一位放到 EAX  
004EFF15  |.  E8 8695F1FF    CALL XCDKING.004094A0  
004EFF1A  |.  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]  
004EFF1D  |.  E8 9E4DF1FF    CALL XCDKING.00404CC0  
004EFF22  |.  8BD8           MOV EBX,EAX  
004EFF24  |.  83EB 02        SUB EBX,2                         ；与 2 比较，如果用户名小于 2，  
004EFF27  |.  7C 2F          JL SHORT XCDKING.004EFF58  
004EFF29  |.  43             INC EBX  
004EFF2A  |.  BE 02000000    MOV ESI,2  
004EFF2F  |>  8B45 F8        /MOV EAX,DWORD PTR SS:[EBP-8]    ;  
004EFF32  |.  E8 0996F1FF    |CALL XCDKING.00409540  
004EFF37  |.  8B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]  
004EFF3A  |.  0FB65432 FF    |MOVZX EDX,BYTE PTR DS:[EDX+ESI-1]  
004EFF3F  |.  F7EA           |IMUL EDX                           ；在这里面读出用户名的每一位  
004EFF41  |.  8D55 E8        |LEA EDX,DWORD PTR SS:[EBP-18]      ；并把该位乘上上一次的累积  
```
