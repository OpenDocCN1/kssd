
# 万能五笔 2000a+

> 原文：[`www.pediy.com/kssd/pediy03/forum305.htm`](https://www.pediy.com/kssd/pediy03/forum305.htm)

## 破解过程

通过将近 1 个小时的 Track，最后要整理一下，写出以上内容。

首先，准备 `Soft-Ice For Windows` 的任何一个版本或 `Trw2000 1.22` 注册版（注意：`Soft-Ice` 和 `Trw2000` 的命令语法是一样的）。

至于这是我的 `Trw2000.ini` 和 `Winice.dat` 的设定，希望能帮你一点忙。

接着用 `Loder32.exe` 或 `Trw2000.exe` 来打开 `wnb.exe`，然后按 `Load` 键装载 `wnb.exe`。在万能五笔的输入界面上按 `Mouse` 的右键，接着选择“关于/注册”，接着又会出现有关万能五笔的信息，系统会多余地问：“你要注册吗？”，当然要啦！选择“注册”，紧跟着，输入“用户名”如：`Banhouse` 和“注册码”如:`8877665544332211`（一定要等于 16 个字符！为什么我知道？因为跟踪过嘛！等下我会交待的。） ，`Ctrl-D` 或 `Ctrl-N` 呼出 `Soft-Ice` 或 `Trw2000`。

好了真正的工件开始啦！

1. 键入 `bpx getdlgitemtexta` 然后按回车键。
2. 按《`F5`》功能键返回万能五笔的注册界面，然后按 `OK` 键。
3. 这时候万能五笔被拦截！此是在 `User32.dll` 的领空，按一次《`F12`》和一次《`F5`》，系统就返回到 `Wnb!.Text+9d15`，这就是万能五笔的领空。

```assembly
0177:0040AD36 XOR EAX,EAX
XXXX:XXXXXX38 REPNE SCASB
XXXX:XXXXXX3A NOT ECX
XXXX:XXXXXX3C DEC ECX
XXXX:XXXXXX3D CMP ECX,BYTE+4                  ;比较输入的用户名字符个数
XXXX:XXXXXX40 JNC 0040AD86                    ;是否大于 4 个字符，大于则转到 0040AD86
.<一直按 F10>
.
.
0177:0040AD8D XOR EAX,EAX
XXXX:XXXXXXXX REPNE SCASB
XXXX:XXXXXX3A NOT ECX
XXXX:XXXXXX3C DEC ECX
XXXX:XXXXXX3D CMP ECX,BYTE+10                 ;比较输入的注册码字符个数
XXXX:XXXXXX40 JZ 0040ADC7                     ;是否等于 16 个字符，大于则转到 0040ADC7
.<一直按 F10>
.
.
XXXX:XXXXADC7 LEA EAX,[ESP+2C]                ;EAX 存放你所输入的注册码
XXXX:XXXXXXXX PUSH DWORD 0042F860              ;用命令 D 0042f860 可以看见万能码的信息码
XXXX:XXXXXXXX LEA ECX,[ESP+44]                 ;ECX 存放你所输入的用户名
XXXX:XXXXXXXX PUSH EAX
XXXX:XXXXXXXX PUSH ECX
XXXX:XXXXXXXX CALL 00401220                   ;如果这个 Call 没有用 F8 追入的话……
XXXX:XXXXXXXX ADD ESP,BYTE+0C 
XXXX:XXXXXXXX TEST EAX,EAX                    
XXXX:XXXXXXXX MOV EAX,[0042F290]              
XXXX:XXXXXXXX JZ NEAR 0040AE88                ;则这个 JZ 就会转到 0040AE88 ，也就话注册失败了……所以……
XXXX:XXXXXXXX
```

4. <`F8` 追入 `CALL 00401220`> 来到

```assembly
0177:00401220 SUB ESP,BYTE+14
XXXX:XXXX1223 PUSH EBX
XXXX:XXXX1224 PUSH EBP
XXXX:XXXX1225 MOV EBP,[ESP+20]
XXXX:XXXX1229 PUSH ESI
XXXX:XXXX122A EDI
.<一直按 F10>
.
.
XXXX:XXXX12AC LEA EAX,[ESP+10]
XXXX:XXXX12B0 PUSH EAX
XXXX:XXXX12B1 PUSH EDX
XXXX:XXXX12B2 PUSH EBP
XXXX:XXXX12B3 CALL 004011A0                   ;追了一个晚上就是为了找你！可恨的“Call 004011AD”一定是你把我输入的
XXXX:XXXX12B8 ADD ESP,BYTE+0C                 ;注册码 8877665544332211 通过复杂的加工后转换成 4649-394904-0318 的！
XXXX:XXXX12BB LEA ESI,[ESP+10]                ;来到这里用命令 D ESI,则可看见真正的注册码啦！
```

万能五笔的注册码是：`4649-394904-0318`

Cracker : `BanhouseMaster`

E-Mail : `banhouse@21cn.com`

Phone : `13613070052`   `4dd466d0e1e0cc123afbe504da2462ca`
