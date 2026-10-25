
# Personal Antispy 1.14

> 原文：[`www.pediy.com/kssd/pediy05/pediy50675.htm`](https://www.pediy.com/kssd/pediy05/pediy50675.htm)

## Personal Antispy 1.14 注册算法分析

**破解目标**：`Personal Antispy 1.14`  
**官方主页**：`http://www.antispy.biz/`  
**软件简介**：检查你的系统里面有没有被人植下可以记录键盘操作的特洛伊木马程序  
**下载地址**：`http://www.antispy.biz/downloads/inst_antispy.exe`  
**使用工具**：`PEiD 0.8`、`Ollydbg`、`Windows 自带的计算器`、`32bit Calculator 1.6 by cybult`、`UltraEdit`  
**作者**：`炎之川[BCG]`  
**时间**：`2003.4.30`  
**主页**：`http://skipli.yeah.net/`

========================================================================

**声明**：本文纯属技术交流，无其他任何目的，转载请注明作者并保持文章的完整。

========================================================================

先说明，`Personal Antispy 1.14` 和我前段时间写的 `AntiSpy PRO 1.02` 那篇算法分析文章中提到的软件不是同一个软件，功能也完全不一样，所以我可不是写重复了^^

经 `PEiD 0.8` 检查可知，`Personal Antispy 1.14` 的主程序为 `VC++ 6.0` 编写且无壳。

直接用 `OD` 装入程序。载入完成后，在 `CPU` 窗口中右击，选择“搜索”->“字串参考”，然后在出现的窗口中搜索注册错误的提示“Registration code or user name is invalid. Please check all fields and try again!”，找到后双击进入对应的代码段，`F2` 下断点。

然后运行程序，在输入注册信息的窗口中输入注册名和假注册码（注册码有四个框）

```
Name: lovefire[BCG]

Serial:: ABCD-EFGH-IJKL-MNOP
```

(; 后是 `Ollydbg` 所分析的内容，// 后是我加的注释，文中数值均为十六进制值)

```
004040B0 
  /$ 81EC CC000000  SUB ESP,0CC   //断点  

004040B6  |. 8D4424 64       LEA EAX,DWORD PTR SS:[ESP+64]  

004040BA  |. 56             PUSH ESI  

004040BB  |. 8BB424 D40000>MOV ESI,DWORD PTR SS:[ESP+D4]  

004040C2  |. 57             PUSH EDI  

004040C3  |. 8B3D A4764400  MOV EDI,DWORD PTR DS:[<&USER32.GetDlgIte>;  USER32.GetDlgItemTextA  

004040C9  |. 6A 32         PUSH 32                            ; /Count = 32 (50.)  

004040CB  |. 50             PUSH EAX                          ; |Buffer  

004040CC  |. 6A 6A         PUSH 6A                            ; |ControlID = 6A (106.)  

004040CE  |. 56             PUSH ESI                          ; |hWnd  

004040CF  |. FFD7          CALL EDI                           ; \GetDlgItemTextA  

004040D1  |. 8A4424 6C     MOV AL,BYTE PTR SS:[ESP+6C]   //取注册名的第一个字符放入 al  

004040D5  |. 84C0           TEST AL,AL   //al 是否为 0，即是否输入了注册名？  

004040D7  |. 75 1E         JNZ SHORT antispy.004040F7   //没有就弹出下面的 MessageBox  

004040D9  |. 6A 40         PUSH 40                            ; /style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL  

004040DB  |. 68 D4824500   PUSH antispy.004582D4              ; |Title = "Attention!"  

004040E0  |. 68 BC824500   PUSH antispy.004582BC              ; |Text = "Please enter your name!"  

004040E5  |. 56             PUSH ESI                          ; |hOwner  

004040E6  |. FF15 A8764400  CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; \MessageBoxA  

004040EC  |. 5F             POP EDI  

004040ED  |. 33C0           XOR EAX,EAX  

004040EF  |. 5E             POP ESI  

004040F0  |. 81C4 CC000000  ADD ESP,0CC  

004040F6  |. C3             RETN  

004040F7  |> 8D4C24 14     LEA ECX,DWORD PTR SS:[ESP+14]  

004040FB  |. 6A 0A         PUSH 0A  

004040FD  |. 51             PUSH ECX  

004040FE  |. 6A 6B         PUSH 6B  

00404100  |. 56             PUSH ESI  

00404101  |. FFD7          CALL EDI   //取第一个框的注册码  

00404103  |. 8D5424 2C     LEA EDX,DWORD PTR SS:[ESP+2C]  

00404107  |. 6A 0A         PUSH 0A  

00404109  |. 52             PUSH EDX  

0040410A  |. 6A 6C         PUSH 6C  

0040410C  |. 56             PUSH ESI  

0040410D  |. FFD7          CALL EDI   //取第二个框的注册码  

0040410F  |. 8D4424 20     LEA EAX,DWORD PTR SS:[ESP+20]  

00404113  |. 6A 0A         PUSH 0A  

00404115  |. 50             PUSH EAX  

00404116  |. 6A 6D         PUSH 6D  

00404118  |. 56             PUSH ESI  

00404119  |. FFD7          CALL EDI   //取第三个框的注册码  

0040411B  |. 8D4C24 08     LEA ECX,DWORD PTR SS:[ESP+8]  

0040411F  |. 6A 0A         PUSH 0A  

00404121  |. 51             PUSH ECX  

00404122  |. 6A 6E         PUSH 6E  

00404124  |. 56             PUSH ESI  

00404125  |. FFD7          CALL EDI   //取第四个框的注册码  

00404127  |. 8D5424 14     LEA EDX,DWORD PTR SS:[ESP+14]  

0040412B  |. 8D4424 38     LEA EAX,DWORD PTR SS:[ESP+38]  

0040412F  |. 52             PUSH EDX                          ; /String2  

00404130  |. 50             PUSH EAX                          ; |String1  

00404131  |. FF15 54734400  CALL DWORD PTR DS:[<&KERNEL32.lstrcpyA>] ; \lstrcpyA  

00404137  |. 8B3D 48734400  MOV EDI,DWORD PTR DS:[<&KERNEL32.lstrcat>;  kernel32.lstrcatA  

0040413D  |. 8D4C24 2C     LEA ECX,DWORD PTR SS:[ESP+2C]  

00404141  |. 8D5424 38     LEA EDX,DWORD PTR SS:[ESP+38]  

00404145  |. 51             PUSH ECX                          ; /StringToAdd  

00404146  |. 52             PUSH EDX                          ; |ConcatString  

00404147  |. FFD7          CALL EDI                          ; \lstrcatA  

00404149  |. 8D4424 20     LEA EAX,DWORD PTR SS:[ESP+20]  

0040414D  |. 8D4C24 38     LEA ECX,DWORD PTR SS:[ESP+38]  

00404151  |. 50             PUSH EAX                          ; /StringToAdd  

00404152  |. 51             PUSH ECX                          ; |ConcatString  

00404153  |. FFD7          CALL EDI                          ; \lstrcatA  

00404155  |. 8D5424 08     LEA EDX,DWORD PTR SS:[ESP+8]  

00404159  |. 8D4424 38     LEA EAX,DWORD PTR SS:[ESP+38]  

0040415D  |. 52             PUSH EDX                          ; /StringToAdd  

0040415E  |. 50             PUSH EAX                          ; |ConcatString  

0040415F  |. FFD7          CALL EDI                          ; \lstrcatA  

//以上分别取每个框中的注册码，并将其合并，我输入的注册码“ABCD-EFGH-IJKL-MNOP”被合并为“ABCDEFGHIJKLMNOP”

00404161 

 |. 8D8C24 A00000>LEA ECX,DWORD PTR SS:[ESP+A0]  

00404168  |. 8D5424 6C     LEA EDX,DWORD PTR SS:[ESP+6C]   //取注册名到 edx  

0040416C  |. 51             PUSH ECX  

0040416D  |. 68 60804400   PUSH antispy.00448060  

00404172  |. 52             PUSH EDX  

00404173  |. E8 18FEFFFF   CALL antispy.00403F90   //算法 call！我们当然跟进去～  

00404178  |. 83C4 0C       ADD ESP,0C  

0040417B  |. 8D4424 38     LEA EAX,DWORD PTR SS:[ESP+38]   //取假码  

0040417F  |. 8D8C24 A00000>LEA ECX,DWORD PTR SS:[ESP+A0]   //取真码  

00404186  |. 50             PUSH EAX                          ; /String2   //假码入栈  

00404187  |. 51             PUSH ECX                          ; |String1   //真码入栈  

00404188  |. FF15 A4734400  CALL DWORD PTR DS:[<&KERNEL32.lstrcmpiA>>; \lstrcmpiA   //比较真假注册码  

0040418E  |. 85C0           TEST EAX,EAX   //eax=1 则注册失败  

00404190  |. 6A 40         PUSH 40                            ; /style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL  

00404192  |. 75 31         JNZ SHORT antispy.004041C5         ; |  

00404194  |. 68 AC824500   PUSH antispy.004582AC              ; |Title = "Registration"  

00404199  |. 68 68824500   PUSH antispy.00458268              ; |Text = "Registration succeeded. Thank you for choosing Personal Antispy!"  

0040419E  |. 56             PUSH ESI                          ; |hOwner  

0040419F  |. FF15 A8764400  CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; \MessageBoxA  

004041A5  |. 8D5424 38     LEA EDX,DWORD PTR SS:[ESP+38]  

004041A9  |. 8D4424 6C     LEA EAX,DWORD PTR SS:[ESP+6C]  

004041AD  |. 52             PUSH EDX  

004041AE  |. 50             PUSH EAX  

004041AF  |. E8 BCFEFFFF   CALL antispy.00404070  

004041B4  |. 83C4 08       ADD ESP,8  

004041B7  |. B8 01000000   MOV EAX,1  

004041BC  |. 5F             POP EDI  

004041BD  |. 5E             POP ESI  

004041BE  |. 81C4 CC000000  ADD ESP,0CC  

004041C4  |. C3             RETN  

004041C5  |> 68 54824500   PUSH antispy.00458254              ; |Title = "Registration error"  

004041CA  |. 68 00824500   PUSH antispy.00458200              ; |Text = "Registration code or user name is invalid. Please check all fields and try again!"  

004041CF  |. 56             PUSH ESI                          ; |hOwner  
```



### 0x004041D0 调用序列

```asm
004041D0  |. FF15 A8764400  CALL DWORD PTR DS:[<&USER32.MessageBoxA>>; \\MessageBoxA  
004041D6  |. 68 2C010000    PUSH 12C                          ; /Timeout = 300. ms  
004041DB  |. FF15 4C734400  CALL DWORD PTR DS:[<&KERNEL32.Sleep>]    ; \\Sleep  
004041E1  |. 5F             POP EDI  
004041E2  |. 33C0           XOR EAX,EAX  
004041E4  |. 5E             POP ESI  
004041E5  |. 81C4 CC000000  ADD ESP,0CC  
004041EB  \. C3             RETN
```

------------------------------------------------------------------------

### 进入 0x00403F90 的算法 call

```asm
00403F90  /$ 53             PUSH EBX  
00403F91  |. 8B5C24 0C      MOV EBX,DWORD PTR SS:[ESP+C]  
00403F95  |. 55             PUSH EBP  
00403F96  |. 56             PUSH ESI  
00403F97  |. 57             PUSH EDI  
00403F98  |. 8B3D 50734400  MOV EDI,DWORD PTR DS:[<&KERNEL32.lstrlen>;  kernel32.lstrlenA  
00403F9E  |. 53             PUSH EBX                            ; /String  
00403F9F  |. FFD7           CALL EDI                            ; \\lstrlenA  
00403FA1  |. 8BF0           MOV ESI,EAX  
00403FA3  |. 8B4424 14      MOV EAX,DWORD PTR SS:[ESP+14]   //取注册名到 eax  
00403FA7  |. 50             PUSH EAX                            ; /String  
00403FA8  |. 897424 1C      MOV DWORD PTR SS:[ESP+1C],ESI       ; |  
00403FAC  |. FFD7           CALL EDI                            ; \\lstrlenA  
00403FAE  |. 8BE8           MOV EBP,EAX   //用户名长度  
00403FB0  |. 85ED           TEST EBP,EBP   //长度是否为 0？  
00403FB2  |. 75 0C          JNZ SHORT antispy.00403FC0   //不为 0 就对了  
00403FB4  |. 8B4424 1C      MOV EAX,DWORD PTR SS:[ESP+1C]  
00403FB8  |. 5F             POP EDI  
00403FB9  |. 5E             POP ESI  
00403FBA  |. 5D             POP EBP  
00403FBB  |. C600 00        MOV BYTE PTR DS:[EAX],0  
00403FBE  |. 5B             POP EBX  
00403FBF  |. C3             RETN  
00403FC0  |> 8B7C24 1C      MOV EDI,DWORD PTR SS:[ESP+1C]   //不为 0 就跳到此处，开始具体算法  
00403FC4  |. 53             PUSH EBX                            ; /String2   //  
00403FC5  |. 57             PUSH EDI                            ; |String1  
00403FC6  |. FF15 54734400  CALL DWORD PTR DS:[<&KERNEL32.lstrcpyA>] ; \\lstrcpyA  
```

`//过了这个 call 以后，用 OD 在“转存”中查看 edi 中的值，可以看到下面的代码，注意！`

```hex
0012F034 
  5F 74 20 3C 40 36 3D 6B  _t <@6=k  
0012F03C  6C 2F 31 37 67 D8 2C 3E  l/17g?>1
```

`//我没弄清楚这个字串是如何得来的，因为我只有 XP，也没有办法到其他系统去验证这个字串是不是唯一的`

`//这个字串关系到软件的算法`

```asm
00403FCC 
  |. 3BEE           CMP EBP,ESI   //esi 为输入的注册名长度，ebp 为 0D，即 16  
00403FCE  |. 8BC5           MOV EAX,EBP  
00403FD0  |. 7F 02          JG SHORT antispy.00403FD4  
00403FD2  |. 8BC6           MOV EAX,ESI  
00403FD4  |> 33C9           XOR ECX,ECX  
00403FD6  |. 894424 1C      MOV DWORD PTR SS:[ESP+1C],EAX  
00403FDA  |. 85C0           TEST EAX,EAX  
00403FDC  |. 7E 39          JLE SHORT antispy.00404017  
00403FDE  |. EB 04          JMP SHORT antispy.00403FE4

00403FE0  |> 8B7424 18      /MOV ESI,DWORD PTR SS:[ESP+18]  
00403FE4  |> 8BC1            MOV EAX,ECX  
00403FE6  |. 8B5C24 14      |MOV EBX,DWORD PTR SS:[ESP+14]   //注册名放入 ebx  
00403FEA  |. 99             |CDQ  
00403FEB  |. F7FE           |IDIV ESI  
00403FED  |. 8BC1           |MOV EAX,ECX  
00403FEF  |. 8BF2           |MOV ESI,EDX  
00403FF1  |. 99             |CDQ  
00403FF2  |. F7FD           |IDIV EBP  
00403FF4  |. 33C0           |XOR EAX,EAX   //edx 清零  
00403FF6  |. 8A041A         |MOV AL,BYTE PTR DS:[EDX+EBX]   //edx 是计数器，这里逐位从 ebx 中取注册名到 al  
00403FF9  |. 33D2           |XOR EDX,EDX   //edx 清零  
00403FFB  |. 8A143E         |MOV DL,BYTE PTR DS:[ESI+EDI]   //从预设字串中（记得 edi 指向地址的内容吗）取对应的字符  
00403FFE  |. BB 19000000    |MOV EBX,19   //ebx=19  
00404003  |. 33C2           |XOR EAX,EDX   //eax^edx，eax->用户名 ASCII，edx->对应位的预设字串 ASCII，两者异或  
00404005  |. 99             |CDQ   //edx 清零  
00404006  |. F7FB           |IDIV EBX   //eax/=ebx，edx=eax%ebx，ebx=19  
00404008  |. 8B4424 1C      |MOV EAX,DWORD PTR SS:[ESP+1C]   //注册名长度送回 eax  
0040400C  |. 80C2 41        |ADD DL,41   //dl+=41，dl=eax%ebx  
0040400F  |. 41             |INC ECX   //计数器+1  
00404010  |. 3BC8           |CMP ECX,EAX   //比较是否已取完  
00404012  |. 88143E         |MOV BYTE PTR DS:[ESI+EDI],DL   //dl 的值放回 EDI 指向的地址中，覆盖掉原始的值（原始值是对应位的预设字串），算完 16 位的注册码之后，这个地址就不再是原预设字串，而是注册码了  
00404015  |. 7C C9          \\JL SHORT antispy.00403FE0   //没有取完跳回去继续  
```

`//注意，因为注册码有 16 位，而每位注册码是由注册名单独决定的，所以有效注册名也是 16 位`

`//如果不足 16 位，则从头取，直到补足 16 位，比如输入 12345abcde，则实际使用的注册名是 12345abcde12345a`

```asm
00404017  |> 8BC7           MOV EAX,EDI  
00404019  |. 5F             POP EDI  
0040401A  |. 5E             POP ESI  
0040401B  |. 5D             POP EBP  
0040401C  |. 5B             POP EBX  
0040401D  \. C3             RETN
```

### 算法总结

1. 注册名最长有效位数为 `16` 位，超过 `16` 位也只取前 `16` 位，不足 `16` 位的从头取直到补足
2. 预设的字串“\_t <@6=kl/17g?>1”
3. 对应注册名的注册码算法为：`((注册名 ASCII^对应字串字符的 ASCII)%19)+41`，这里保证了注册码肯定在大写英文字母范围内
4. 最后得到的注册码，每隔四位用`“-”`相连

#### 举例

我输入的第一位注册名为`“l”`，对应字串字符`“\_”`

则：`(6C⁵F)%19+41=42`，即`“B”`

至此 `Personal Antispy 1.14` 注册算法分析完成，一组可用的注册码：

```text
Name: lovefire[BCG]  
Serial: BCLO-NUEO-FJOM-IFRW
```

### 注册信息保存

```registry
[HKEY_USERS\S-1-5-21-1454471165-789336058-1343024091-1003\Software\Blazing Tools\Personal Antispy]  
"RegName"="lovefire[BCG]"  
"RegCode"="BCLONUEOFJOMIFRW"
```

`（注意 S-1-5-21-1454471165-789336058-1343024091-1003 这个主键在不同机器上可能不同）`

------------------------------------------------------------------------

### 炎之川

属于中国破解组织 `BCG` (`BeGiNnEr'S CrAcKiNg Group`)

```text
  _/\_/\_/     _/\_/\_/     _/\_/\_/  
  \_/    \_/  \_/          \_/  
  _/\_/\_/    \_/          \_/  _/\_/  
  \_/    \_/  \_/          \_/    \_/  
  _/\_/\_/     _/\_/\_/     _/\_/\_/  4dd466d0e1e0cc123afbe504da2462ca
```
