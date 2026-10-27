
# loveasm的crackme算法分析-----CRC32算法的妙用

标 题: loveasm的crackme算法分析-----CRC32算法的妙用

发信人: onlyu

时 间: 2004-2-20 周五, 下午12:14

详细信息:

loveasm的crackme算法分析-----CRC32算法的妙用

【作者】：onlyu[FCG][DFCG]

【软件介绍】：loveasm[YCG][DFCG]兄弟的crackme，主要锻炼算法分析，要求破解成功后出现显示"恭喜您!您已经成功的注册了本软件!"的对话框。

【软件下载】：[点击此处本地下载](https://www.kanxue.com/pediy6971/crackme.rar)

【软件限制】：注册码

【作者声明】：初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【破解工具】：Ollydbg1.09

## 破解过程

运行程序，输入

用户名：onlyu[FCG][DFCG]

注册码：78787878787878787878787878787878

在OD命令行插件里下断点：`bpx getdlgitemtexta`

点注册，停在如下：

```
004014A9   .  68 A4374000   PUSH CRACKME.004037A4
004014AE   .  68 91010000   PUSH 191
004014B3   .  FF75 08       PUSH DWORD PTR SS:[EBP+8]
004014B6   .  E8 27020000   CALL <JMP.&user32.GetDlgItemTextA><-----获得输入的注册名
004014BB   .  8BC8          MOV ECX,EAX
004014BD   .  83F8 00       CMP EAX,0<-------------------------输入的注册名是否为空
004014C0   .  77 0E         JA SHORT CRACKME.004014D0<---------不空往下跳
004014C2   .  B8 00000000   MOV EAX,0
004014C7   .  59            POP ECX
004014C8   .  5A            POP EDX
```

跳转到这里：

```
004014D0   >  33C0          XOR EAX,EAX
004014D2   .  8D35 A4374000 LEA ESI,DWORD PTR DS:[4037A4]<----使ESI指向注册名"onlyu[FCG][DFCG]"
004014D8   >  33DB          XOR EBX,EBX--------------------
004014DA   .  8A1E          MOV BL,BYTE PTR DS:[ESI]       |
004014DC   .  03C3          ADD EAX,EBX                    |小循环
004014DE   .  46            INC ESI                        |
004014DF   .^ E2 F7         LOOPD SHORT CRACKME.004014D8---
```

上面这段小循环的功能是把输入的用户名的字符的ASCII码累加起来保存到`EAX`中，我的为58B

```
004014E1   .  8845 FF       MOV BYTE PTR SS:[EBP-1],AL<----把上面得到的值取后两位存到ss:[ebp-1]处
004014E4   .  68 FF000000   PUSH 0FF
004014E9   .  68 24364000   PUSH CRACKME.00403624
004014EE   .  68 92010000   PUSH 192
004014F3   .  FF75 08       PUSH DWORD PTR SS:[EBP+8]
004014F6   .  E8 E7010000   CALL <JMP.&user32.GetDlgItemTextA><-----获得输入的注册码
004014FB   .  BE 24364000   MOV ESI,CRACKME.00403624<--------ESI->输入的注册码"78787878..."
00401500   .  8D3D 70394000 LEA EDI,DWORD PTR DS:[403970]
00401506   .  B9 10000000   MOV ECX,10<---------------------置循环次数
0040150B   >  33DB          XOR EBX,EBX<--------------------把EBX清0
0040150D   .  33C0          XOR EAX,EAX<--------------------把EAX清0
0040150F   .  AC            LODS BYTE PTR DS:[ESI]<---------取注册码的字符到EAX中
00401510   .  83F8 39       CMP EAX,39<---------------------输入字符ASCII码是否大于39
00401513   .  7E 03         JLE SHORT CRACKME.00401518<-----小于或等于就跳
00401515   .  83E8 07       SUB EAX,7<----------------------EAX=EAX-7
00401518   >  83E8 30       SUB EAX,30<---------------------EAX=EAX-30
0040151B   .  0BD8          OR EBX,EAX<---------------------EBX=EBX or EAX
0040151D   .  C1E3 04       SHL EBX,4<----------------------EBX=EBX shl 4
00401520   .  33C0          XOR EAX,EAX<--------------------EAX清0
00401522   .  AC            LODS BYTE PTR DS:[ESI]<---------取注册码的下一个字符到EAX中
00401523   .  83F8 39       CMP EAX,39<---------------------输入字符ASCII码是否大于39
00401526   .  7E 03         JLE SHORT CRACKME.0040152B<-----小于或等于就跳
00401528   .  83E8 07       SUB EAX,7<----------------------EAX=EAX-7
0040152B   >  83E8 30       SUB EAX,30<---------------------EAX=EAX-30
0040152E   .  0BD8          OR EBX,EAX<---------------------EBX=EBX or EAX
00401530   .  881F          MOV BYTE PTR DS:[EDI],BL<-------把BL中数值存到DS:[EDI]处
00401532   .  47            INC EDI<------------------------EDI=EDI+1
00401533   .^ E2 D6         LOOPD SHORT CRACKME.0040150B<---循环到40150B处
```

上面这一段循环16次，输入的注册码按两个字符一组处理，字符若为小于或等于39，就把它的ASCII值减去30H，若大于39就减去37H，不妨设两个字符为a,b,则c=(((a-常数) shl 4) or (b-常数))and FFH，如我输入的注册码运算后在内存中如下：

```
00403970  78 78 78 78 78 78 78 78   xxxxxxxx
00403978  78 78 78 78 78 78 78 78   xxxxxxxx
```

这里就可以猜测它的注册码可能为32位的。

```
00401535   .  B9 10000000   MOV ECX,10<------------------------置循环次数10H
0040153A   .  8D3D 80394000 LEA EDI,DWORD PTR DS:[403980]
00401540   .  8D35 70394000 LEA ESI,DWORD PTR DS:[403970]<-----ESI指向上面运行完的结果
00401546   >  8A06          MOV AL,BYTE PTR DS:[ESI]<----------把ESI指向的数赋给AL
00401548   .  3245 FF       XOR AL,BYTE PTR SS:[EBP-1]<--------AL=AL xor SS:[EBP-1]
0040154B   .  8807          MOV BYTE PTR DS:[EDI],AL<----------把AL的值保存到DS:[EDI]中
0040154D   .  46            INC ESI<---------------------------ESI++
0040154E   .  47            INC EDI<---------------------------EDI++
0040154F   .^ E2 F5         LOOPD SHORT CRACKME.00401546<------循环到401546处
```

`SS:[EBP-1]`中保存的是对用户名运算后得到的结果，上面的循环的功能就是把`SS:[EBP-1]`和上面对注册码处理得到的结果按字节异或保存到`DS:[EDI]`中，结果在内存中如下：

```
00403980  F3 F3 F3 F3 F3 F3 F3 F3  篌篌篌篌
00403988  F3 F3 F3 F3 F3 F3 F3 F3  篌篌篌篌
00403990  76 71 32 7A 79 62 6D 6D  vq2zybmm
00403998  74 63 54 58 6F 72 4C 68  tcTXorLh
004039A0  77 63 75 78 76 73 6A 74  wcuxvsjt
004039A8  76 50 36 6A 6F 53 30 67  vP6joS0g
004039B0  54 47 39 32 5A 57 46 7A  TG92ZWFz
004039B8  62 56 74 5A 51 30 64 64  bVtZQ0dd
004039C0  57 30 52 47 51 30 64 64  W0RGQ0dd
```

```
00401551   .  B9 48000000   MOV ECX,48<-----------------------置循环次数48H
00401556   .  8D35 A7384000 LEA ESI,DWORD PTR DS:[4038A7]
ESI->"I Love Asm!I Love Crack!I Love Kathy!My Email:loveasm@sohu.com QQ:127095"
0040155C   .  8D3D 80394000 LEA EDI,DWORD PTR DS:[403980]<----EDI指向上面运算后得到的结果
00401562   >  8A07          MOV AL,BYTE PTR DS:[EDI]<---------AL=DS:[EDI]
00401564   .  3206          XOR AL,BYTE PTR DS:[ESI]<---------AL=AL xor DS:[ESI]
00401566   .  8807          MOV BYTE PTR DS:[EDI],AL<---------DS:[EDI]=AL
00401568   .  46            INC ESI<--------------------------ESI++
00401569   .  47            INC EDI<--------------------------EDI++
0040156A   .^ E2 F6         LOOPD SHORT CRACKME.00401562<-----循环到401562处
```

这一段循环的功能是把上面运算得到的结果与常串"vq2zybmmtcTXorLhwcuxvsjtvP6joS0gTG92ZWFzbVtZQ0ddW0RGQ0dd"连起来和字符串"I Love Asm!I Love Crack!I Love Kathy!My Email:loveasm@sohu.com QQ:127095"进行异或运算，结果还保存在`DS:[403980]`处。

```
0040156C   .  68 F0384000   PUSH CRACKME.004038F0
00401571   .  68 80394000   PUSH CRACKME.00403980
00401576   .  E8 2CFBFFFF   CALL CRACKME.004010A7<-------F7进去看看
```

F7进入`CALL CRACKME.004010A7`，如下：

```
004010A7  /$  55            PUSH EBP
004010A8  |.  8BEC          MOV EBP,ESP
004010AA  |.  83C4 FC       ADD ESP,-4
004010AD  |.  53            PUSH EBX
004010AE  |.  57            PUSH EDI
004010AF  |.  56            PUSH ESI
004010B0  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]
004010B3  |.  E8 90060000   CALL <JMP.&kernel32.lstrlenA>
004010B8  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX<--------EAX=48H
004010BB  |.  8B75 08       MOV ESI,DWORD PTR SS:[EBP+8]<--------ESI指向上面得到的结果
004010BE  |.  8B7D 0C       MOV EDI,DWORD PTR SS:[EBP+C]
004010C1  |.  8B4D FC       MOV ECX,DWORD PTR SS:[EBP-4]<--------ECX=48H
004010C4  |.  C1E9 02       SHR ECX,2<---------------------------ECX=ECX shr 2=12H
004010C7  |.  FC            CLD
004010C8  |>  51            /PUSH ECX
004010C9  |.  B9 04000000   |MOV ECX,4<--------------------------置大循环次数4
004010CE  |.  33DB          |XOR EBX,EBX<------------------------EBX清0
004010D0  |.  AD            |LODS DWORD PTR DS:[ESI]<------------装入32位整数到EAX
004010D1  |>  50            |/PUSH EAX<--------------------------EAX压栈
004010D2  |.  25 FF000000   ||AND EAX,0FF<-----------------------取AL
004010D7  |.  8A80 41344000 ||MOV AL,BYTE PTR DS:[EAX+403441]<---把DS:[EAX+403441]处数据赋给AL
004010DD  |.  3C FF         ||CMP AL,0FF<------------------------比较AL是否和FF相等
004010DF  |.  74 22         ||JE SHORT CRACKME.00401103<---------相等就跳
004010E1  |.  C1E3 06       ||SHL EBX,6<-------------------------EBX=EBX shl 6，EBX初值为0
004010E4  |.  0AD8          ||OR BL,AL<--------------------------BL=BL or AL
004010E6  |.  58            ||POP EAX<---------------------------重新装入32位整数
004010E7  |.  C1E8 08       ||SHR EAX,8<-------------------------EAX=EAX shr 8
004010EA  |.  49            ||DEC ECX<---------------------------ECX=ECX-1
004010EB  |.^ 75 E4         |JNZ SHORT CRACKME.004010D1<--------不等就跳到4010D1处
004010ED  |.  8BC3          |MOV EAX,EBX<------------------------EAX=EBX
```
