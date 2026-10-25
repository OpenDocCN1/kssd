
# 鼠×××3.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50240.htm`](https://www.pediy.com/kssd/pediy05/pediy50240.htm)

## 用 KEYMAKE 制作内存注册机特殊一例

**软件名称**：鼠×××3.0（国产软件）

**未注册版限制**：
1. 每次启动时出现注册窗体；
2. 保存的图标、位图可能被随机加上一些杂点。

**破解工具**：`pe-scan` 3.12、`W32DASM` 8.93 汉化版、`OllyDbg` v1.07b。

**破解人**：飞鹰\[BCG\]

**E-mail**：`flithawk@163.com`

**网址**：`http://flithawk.longcity.net`

### 破解步骤

用 `pe-scan` 检测可以知道软件被 `ASPack1.07b` 加了壳，先找个自动脱壳工具脱壳。脱壳后启动软件，首先跳出一个注册窗体，在这个窗体的注册码框中输入 20 位的注册码后，软件要求重启校验注册码。这时，你会发现软件所在的目录下生成了一个 `sign.ini` 文件，该文件中存放的就是你刚才所输入的注册码，注册码的形式为 `*****-*****`，其中用“-”符号把你输入的注册码分为前半部分和后半部分。下面，我们就从 `sign.ini` 文件入手开始破解，用 `W32DASM` 反编译后查找“sign.ini”，共可找到下面两处地方：

#### 查找到的第一处

```
* Possible StringData Ref from Data Obj ->"注册"
                  |
:00404D1C 6820224100                 push 00412220

* Possible StringData Ref from Data Obj ->"程序需要关闭以更新设置,再次启动后即可知道注册是否成功"
                  |
:00404D21 68E8214100                 push 004121E8
:00404D26 50                         push eax
* Reference To: USER32.MessageBoxA, Ord:01BEh
                  |
:00404D27 FF15C4014100               Call dword ptr [004101C4]
* Possible StringData Ref from Data Obj ->"wb"
                  |
:00404D2D 6834214100                 push 00412134
* Possible StringData Ref from Data Obj ->"sign.ini"
                  |
:00404D32 68DC214100                 push 004121DC
:00404D37 E8E8220000                 call 00407024
:00404D3C 83C408                     add esp, 00000008
:00404D3F 8D4C240C                   lea ecx, dword ptr [esp+0C]
:00404D43 8BF0                       mov esi, eax
:00404D45 6800010000                 push 00000100
:00404D4A 51                         push ecx
:00404D4B 8B8FE8000000               mov ecx, dword ptr [edi+000000E8]
:00404D51 E8EAC3FFFF                 call 00401140
:00404D56 8D54240C                   lea edx, dword ptr [esp+0C]
:00404D5A 56                         push esi
:00404D5B 52                         push edx
:00404D5C E845240000                 call 004071A6
:00404D61 8B4604                     mov eax, dword ptr [esi+04]
:00404D64 83C408                     add esp, 00000008
:00404D67 48                         dec eax
:00404D68 894604                     mov dword ptr [esi+04], eax
:00404D6B 780C                       js 00404D79
:00404D6D 8B06                       mov eax, dword ptr [esi]
:00404D6F C6002D                     mov byte ptr [eax], 2D
:00404D72 8B06                       mov eax, dword ptr [esi]
:00404D74 40                         inc eax
:00404D75 8906                       mov dword ptr [esi], eax
:00404D77 EB0B                       jmp 00404D84
..................................................省
..................................................略
```

这里是把你输入的注册码写入到 `sign.ini` 文件中后，并重新启动软件。

#### 查找到的第二处

```
* Referenced by a CALL at Addresses:
|:00401C17  , :00405381
|
:00405770 A004394100                 mov al, byte ptr [00413904]
:00405775 81EC94010000               sub esp, 00000194
:0040577B 84C0                       test al, al
:0040577D 56                         push esi
:0040577E 0F8511010000               jne 00405895
* Possible StringData Ref from Data Obj ->"rb"
                  |
:00405784 68B0204100                 push 004120B0
* Possible StringData Ref from Data Obj ->"sign.ini"
                  |
:00405789 68DC214100                 push 004121DC
:0040578E E891180000                 call 00407024
:00405793 8BF0                       mov esi, eax
:00405795 83C408                     add esp, 00000008
:00405798 85F6                       test esi, esi
:0040579A 7511                       jne 004057AD
:0040579C C6050439410001             mov byte ptr [00413904], 01
:004057A3 32C0                       xor al, al
:004057A5 5E                         pop esi
:004057A6 81C494010000               add esp, 00000194
:004057AC C3                         ret
..................................................省
..................................................略
:00405885 51                         push ecx
:00405886 52                         push edx
:00405887 E824FCFFFF                 call 004054B0==>比较注册码，进入此 Call 继续跟踪
:0040588C 83C418                     add esp, 00000018
:0040588F A205394100                 mov byte ptr [00413905], al
:00405894 5F                         pop edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040577E(C)
|
:00405895 A005394100                 mov al, byte ptr [00413905]
:0040589A C6050439410001             mov byte ptr [00413904], 01
:004058A1 5E                         pop esi
:004058A2 81C494010000               add esp, 00000194
:004058A8 C3                         ret
```

这里是软件启动时判断从 `sign.ini` 文件中读取的注册码是否正确。

进入 `004054B0` 后，来到下面这里：

设：
(1) `code1`=你输入的假注册码前半部分，`code2`=真注册码前半部分，`code3`=你输入的假注册码后半部分，`code4`=真注册码后半部分；
(2) 初始状态时 `x1`=`code1` 的第一位，`y1`=`code2` 的第一位，`x3`=`code3` 的第一位，`y3`=`code4` 的第一位；
(3) 动态变量 `x2`，`y2`，`x4`，`y4`。

```
004054B0  /$ 83EC 18          SUB ESP,18
004054B3  |. 53               PUSH EBX
004054B4  |. 56               PUSH ESI
004054B5  |. 57               PUSH EDI
004054B6  |. E8 C5FFFFFF       CALL 鼠×××.00405480
004054BB  |. 8BF8             MOV EDI,EAX
004054BD  |. 8D4424 0C         LEA EAX,DWORD PTR SS:[ESP+C]
004054C1  |. 81F7 17108519     XOR EDI,19851017
004054C7  |. 6A 24             PUSH 24                               ; /Arg3 = 00000024
004054C9  |. 50               PUSH EAX                              ; |Arg2
004054CA  |. 57               PUSH EDI                              ; |Arg1
004054CB  |. E8 CFA70000       CALL 鼠×××.0040FC9F                  ; \鼠×××.0040FC9F
004054D0  |. 8B4424 34         MOV EAX,DWORD PTR SS:[ESP+34]        ==>这里 d eax 可以看到真注册码前半部分
004054D4  |. 83C4 0C           ADD ESP,0C
004054D7  |. 8D7424 0C         LEA ESI,DWORD PTR SS:[ESP+C]
******************************************************************
开始比较前半部分注册码
******************************************************************
004054DB  |> 8A10              /MOV DL,BYTE PTR DS:[EAX]           //x2=x1;dl=x2
004054DD  |. 8A1E              |MOV BL,BYTE PTR DS:[ESI]           //y2=y1;bl=y2
004054DF  |. 8ACA              |MOV CL,DL                           //cl=dl
004054E1  |. 3AD3              |CMP DL,BL                           //比较 dl 是否等于 bl
004054E3    75 1E              JNZ SHORT 鼠×××.00405503            //不相等跳转则注册失败
004054E5    84C9              TEST CL,CL                            //判断 code1 中是否还有值需要比较
004054E7    74 16              JE SHORT 鼠到擒来.004054FF          //相等跳转则说明 code1 中已无值
004054E9  |. 8A50 01           |MOV DL,BYTE PTR DS:[EAX+1]         //x2=x2+1;dl=x2
004054EC  |. 8A5E 01           |MOV BL,BYTE PTR DS:[ESI+1]         //y2=y2+1;bl=y2
004054EF  |. 8ACA              |MOV CL,DL                           //cl=dl
004054F1  |. 3AD3              |CMP DL,BL                           //比较 dl 是否等于 bl
004054F3    75 0E              JNZ SHORT 鼠×××.00405503            //不相等跳转则注册失败
004054F5  |. 83C0 02           |ADD EAX,2                            //x1=x1+2;eax=x1
004054F8  |. 83C6 02           |ADD ESI,2                            //y1=y1+2;esi=y1
004054FB  |. 84C9              |TEST CL,CL                           //判断 code1 中是否还有值需要比较
004054FD  |.⁷⁵ DC              \JNZ SHORT 鼠×××.004054DB            //跳转说明 code1 中还有值，循环比较
******************************************************************
结束比较前半部分注册码
******************************************************************
004054FF  |> 33C0             XOR EAX,EAX
00405501  |. EB 05            JMP SHORT 鼠×××.00405508
00405503  |> 1BC0             SBB EAX,EAX
00405505  |. 83D8 FF          SBB EAX,-1
00405508  |> 85C0             TEST EAX,EAX
```



```
0040550A    75 6B           JNZ SHORT 鼠×××.00405577   
0040550C  |. B8 01000000    MOV EAX,1   
00405511  |> 8BC8          /MOV ECX,EAX   
00405513  |. 0FAFF9        |IMUL EDI,ECX   
00405516  |. 40            |INC EAX   
00405517  |. 83F8 0A       |CMP EAX,0A   
0040551A  |.⁷C F5         \\JL SHORT 鼠×××.00405511   
0040551C  |. 8D5424 18     LEA EDX,DWORD PTR SS:\[ESP+18\]   
00405520  |. 6A 24         PUSH 24                        ; /Arg3 = 00000024   
00405522  |. 52            PUSH EDX                       ; |Arg2   
00405523  |. 57            PUSH EDI                       ; |Arg1   
00405524  |. E8 76A70000   CALL 鼠×××.0040FC9F           ; \\鼠×××.0040FC9FFC9F   
00405529  |. 8B4424 38     MOV EAX,DWORD PTR SS:\[ESP+38\]==>这里 d eax 可以看到真注册码后半部分   
0040552D  |. 83C4 0C       ADD ESP,0C   
00405530  |. 8D7424 18     LEA ESI,DWORD PTR SS:\[ESP+18\]   
```
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*开始比较后半部分注册码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
```
00405534  |> 8A10          /MOV DL,BYTE PTR DS:\[EAX\]    //x4=x3;dl=x4   
00405536  |. 8A1E          |MOV BL,BYTE PTR DS:\[ESI\]    //y4=y3;bl=y4   
00405538  |. 8ACA          |MOV CL,DL                     //cl=dl   
0040553A  |. 3AD3          |CMP DL,BL                     //比较 dl 是否等于 bl   
0040553C  |. 75 28         |JNZ SHORT 鼠×××.00405566     //不相等跳转则注册失败   
0040553E  |. 84C9          |TEST CL,CL                    //判断 code3 中是否还有值需要比较   
00405540  |. 74 16         |JE SHORT 鼠×××.00405558      //相等跳转则说明 code3 中已无值    
00405542  |. 8A50 01       |MOV DL,BYTE PTR DS:\[EAX+1\]  //x4=x4+1;dl=x4   
00405545  |. 8A5E 01       |MOV BL,BYTE PTR DS:\[ESI+1\]  //y4=y4+1;bl=y4    
00405548  |. 8ACA          |MOV CL,DL                     //cl=dl   
0040554A  |. 3AD3          |CMP DL,BL                     //比较 dl 是否等于 bl   
0040554C  |. 75 18         |JNZ SHORT 鼠×××.00405566     //不相等跳转则注册失败   
0040554E  |. 83C0 02       |ADD EAX,2                     //x3=x3+2;eax=x3   
00405551  |. 83C6 02       |ADD ESI,2                     //y3=y3+2;esi=y3    
00405554  |. 84C9          |TEST CL,CL                    //判断 code3 中是否还有值需要比较    
00405556  |.⁷⁵ DC         \\JNZ SHORT 鼠×××.00405534     //跳转说明 code3 中还有值，循环比较   
```
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*结束比较后半部分注册码\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*   
```
00405558  |> 33C0         XOR EAX,EAX   
0040555A  |. 5F           POP EDI   
0040555B  |. 85C0         TEST EAX,EAX   
0040555D  |. 5E           POP ESI   
0040555E  |. 5B           POP EBX   
0040555F  |. 0F94C0       SETE AL   
00405562  |. 83C4 18      ADD ESP,18   
00405565  |. C3           RETN   
```

如果想分析注册算法，写注册器的话，可以进入`CALL 鼠×××.0040FC9F`就知道真注册码是如何算出来的了。但我汇编功力有限，对该软件的注册算法没有研究清楚，所以就先用`KEYMAKE`制作一个内存注册机吧！

通过上面的分析，我们可以看出该软件比较注册码分为前后两个部分，如果前半部分的注册码错误，用内存注册机就只能得到前半部分的真注册码，无法得到后半部分的真注册码。所以，在制作内存注册机时，需要把该软件内存地址`004054E3`处的`751E`修改为`7527`，即把`JNZ SHORT 鼠×××.00405503`改为`JNZ SHORT 鼠×××.0040550C`，这样当用户输入错误注册码时就可以从内存中直接取得真注册码前后两部分的值了。

我用的`KEYMAKE`版本号为`1.69`，要使用`1.40`以上的版本才具有修改内存的功能。设置如下：

1、中断地址：`405529`-->中断次数：`1`-->第一个字节：`8B`-->指令长度：`4`；

2、选中“保存下列信息为注册码”-->选择“内存方式”-->内存地址：`68F428`（真注册码前半部分的存放地址）-->在结尾处插入字符：`-`（真注册码前后部分的分隔符）；

3、主界面上选择“修改内存”-->修改地址：`4054E4`-->修改长度：`1`-->原始指令：`1E`-->修改指令：`27`；

4、主界面上的注册码方式选择“内存方式”-->寄存器：`EAX`（真注册码后半部分的存放地址）。

经过这四步以后，运行制作出来的内存注册机就可以完全正确显示出真的注册码了，显示出来的注册码格式为：`*****-*****`。至此，内存注册机制作完成。

仅以此篇献给我们的`BCG`组织，也算是自己的一次`Crack`练习吧！

`Crack by 飞鹰[BCG]  flithawk@163.com  2002.8.15`

`欢迎光临汉化新世纪: http://www.hanzify.org  4dd466d0e1e0cc123afbe504da2462ca`
