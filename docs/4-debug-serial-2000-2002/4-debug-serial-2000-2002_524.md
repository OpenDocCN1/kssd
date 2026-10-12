
# URL Manager 2.42

> 原文：[`www.pediy.com/kssd/pediy03/forum972.htm`](https://www.pediy.com/kssd/pediy03/forum972.htm)

## 破解过程

帮你分类管理你喜爱的网页位置。可以直接自 `HTML` 文件或是以逗号区隔资料的文件输入网页资料。同样也可输出成为 `HTML` 格式或是纯文本文件，以方便使用。

**下载地址：** `URL` `http://www.soft999.com/download1/um2000si.zip`

### 步骤

首先调快日期一年，出现过期提示对话，记下来，等会要用的。

然后用 `w32dsm` 反汇编，查找出错对话框中的字符串，找到下面的位置：

```
* Reference To: c5runx.Cla$Stack2DStack, Ord:01ACh
                      |
:00446AF5 FF150CD94D00          Call dword ptr [004DD90C]

* Reference To: c5runx.Cla$DPopLong, Ord:0074h
                      |
:00446AFB FF1578D64D00          Call dword ptr [004DD678]

:00446B01 89842434010000        mov dword ptr [esp+00000134], eax
:00446B08 83BC243401000000      cmp dword ptr [esp+00000134], 00000000
:00446B10 752C                  jne 00446B3E  //----> 9090 nop nop(不能改为 742C，不信你试试)
                                  //----> 因为程序要产生文件，然后判断时间

* Reference To: c5runx.Cla$TODAY, Ord:01E2h
                      |
:00446B12 E801A9FBFF            Call 00401418

* Reference To: c5runx.Cla$PushLong, Ord:0166h
                      |
:00446B17 FF1574D84D00          Call dword ptr [004DD874]

* Possible StringData Ref from Data Obj ->"ID"
                      |
:00446B1D B8B04F4C00            mov eax, 004C4FB0
:00446B22 BB02000000            mov ebx, 00000002

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00446B27 FF1584D84D00          Call dword ptr [004DD884]

:00446B2D E8BA010000            call 00446CEC

* Reference To: c5runx.Cla$TODAY, Ord:01E2h
                      |
:00446B32 E8E1A8FBFF            Call 00401418
:00446B37 89842434010000        mov dword ptr [esp+00000134], eax

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00446B10(C)  //----------------->修改点
|

* Reference To: c5runx.Cla$TODAY, Ord:01E2h
                      |
:00446B3E E8D5A8FBFF            Call 00401418
:00446B43 2B842434010000        sub eax, dword ptr [esp+00000134]
:00446B4A 83F81E                cmp eax, 0000001E
:00446B4D 7E52                  jle 00446BA1
:00446B4F B800000000            mov eax, 00000000
:00446B54 BB00000000            mov ebx, 00000000

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00446B59 FF1584D84D00          Call dword ptr [004DD884]

:00446B5F B800000000            mov eax, 00000000
:00446B64 BB00000000            mov ebx, 00000000

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00446B69 FF1584D84D00          Call dword ptr [004DD884]

* Possible StringData Ref from Data Obj ->"Shareware"
                      |
:00446B6F B8F04F4C00            mov eax, 004C4FF0
:00446B74 BB09000000            mov ebx, 00000009

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00446B79 FF1584D84D00          Call dword ptr [004DD884]

* Possible StringData Ref from Data Obj ->"Thank you for evaluating URL Manager "
                                  ->"2000.  The 30-day evaluation period "
                                  ->"is now over.  Please consult the "
                                  ->"help file for information on how "
                                  ->"to registered and purchase URL "
                                  ->"Manager 2000 today!"  //---------->出错对话框
                      |
:00446B7F B8FC4F4C00            mov eax, 004C4FFC
:00446B84 BBBD000000            mov ebx, 000000BD

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00446B89 FF1584D84D00          Call dword ptr [004DD884]

:00446B8F 6A00                  push 00000000
:00446B91 B000                  mov al, 00
:00446B93 B301                  mov bl, 01
:00446B95 B101                  mov cl, 01
:00446B97 BA00000000            mov edx, 00000000

* Reference To: c5runx.Cla$MessageBox, Ord:012Eh
```

#### 去掉共享版的字样

```
* Reference To: c5runx.Cla$comparestr, Ord:0041h
                      |
:0044465A FF1518D64D00          Call dword ptr [004DD618]
:00444660 83F800                cmp eax, 00000000
:00444663 0F84B3000000          je 0044471C  //-------> 0F85B3000000    jne 0044471C
:00444669 B8007C0000            mov eax, 00007C00

* Reference To: c5runx.Cla$PushLong, Ord:0166h
                      |
:0044466E FF1574D84D00          Call dword ptr [004DD874]

:00444674 B810AF4900            mov eax, 0049AF10
:00444679 BB14000000            mov ebx, 00000014

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:0044467E FF1584D84D00          Call dword ptr [004DD884]

* Reference To: c5runx.Cla$StackCLIP, Ord:01B2h
                      |
:00444684 E81FCDFBFF            Call 004013A8

* Possible StringData Ref from Data Obj ->" Version "
                      |
:00444689 B854374C00            mov eax, 004C3754
:0044468E BB09000000            mov ebx, 00000009

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:00444693 FF1584D84D00          Call dword ptr [004DD884]

* Reference To: c5runx.Cla$StackConcat, Ord:01B6h
                      |
:00444699 FF1528D94D00          Call dword ptr [004DD928]

:0044469F 8B45FC                mov eax, dword ptr [ebp-04]
:004446A2 8D40E8                lea eax, dword ptr [eax-18]
:004446A5 BB14000000            mov ebx, 00000014

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:004446AA FF1584D84D00          Call dword ptr [004DD884]

* Reference To: c5runx.Cla$StackConcat, Ord:01B6h
                      |
:004446B0 FF1528D94D00          Call dword ptr [004DD928]

:004446B6 B804000000            mov eax, 00000004
:004446BB BB00000000            mov ebx, 00000000
:004446C0 B900000000            mov ecx, 00000000

* Reference To: c5runx.Cla$SetPropS, Ord:019Fh
                      |
:004446C5 FF15F4D84D00          Call dword ptr [004DD8F4]

:004446CB B810AF4900            mov eax, 0049AF10
:004446D0 BB14000000            mov ebx, 00000014

* Possible StringData Ref from Data Obj ->"Shareware"
                      |
:004446D5 B960374C00            mov ecx, 004C3760
:004446DA BA09000000            mov edx, 00000009

* Reference To: c5runx.Cla$comparestr, Ord:0041h
                      |
:004446DF FF1518D64D00          Call dword ptr [004DD618]

:004446E5 83F800                cmp eax, 00000000
:004446E8 7479                  je 00444763        //------->改不改无所谓
:004446EA B8007C0000            mov eax, 00007C00

* Reference To: c5runx.Cla$PushLong, Ord:0166h
                      |
:004446EF FF1574D84D00          Call dword ptr [004DD874]

* Possible StringData Ref from Data Obj ->"Address correspondence to:"
                      |
:004446F5 B86C374C00            mov eax, 004C376C
:004446FA BB1A000000            mov ebx, 0000001A

* Reference To: c5runx.Cla$PushString, Ord:016Ch
                      |
:004446FF FF1584D84D00          Call dword ptr [004DD884]
```



### 汇编代码片段

:00444705 B809000000 `mov eax, 00000009`

:0044470A BB00000000 `mov ebx, 00000000`

:0044470F B900000000 `mov ecx, 00000000`

\* Reference To: `c5runx.Cla$SetPropS`, Ord:019Fh

|

:00444714 FF15F4D84D00 `Call dword ptr [004DD8F4]`

:0044471A EB47 `jmp 00444763`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00444663(C) ///------------------------->修改点

|

:0044471C B8007C0000 `mov eax, 00007C00`

\* Reference To: `c5runx.Cla$PushLong`, Ord:0166h

|

:00444721 FF1574D84D00 `Call dword ptr [004DD874]`

\* Possible StringData Ref from Data Obj -> "Shareware Version " //--->共享版字样，往上找跳点

|

:00444727 B888374C00 `mov eax, 004C3788`

:0044472C BB12000000 `mov ebx, 00000012`

\* Reference To: `c5runx.Cla$PushString`, Ord:016Ch

|

:00444731 FF1584D84D00 `Call dword ptr [004DD884]`

:00444737 8B45FC `mov eax, dword ptr [ebp-04]`

:0044473A 8D40E8 `lea eax, dword ptr [eax-18]`

:0044473D BB14000000 `mov ebx, 00000014`

### 破解说明

这样就成为不受限制的版本了，破解文件我放到主页，请下载。

***

主页：`http://crackerabc.longcity.net` （[BCG]大本营）

`http://sffs.533.net`

软件破解，软件注册，破解教学，破解申请，注册机码。

请保留我的信息，谢谢！

*** 4dd466d0e1e0cc123afbe504da2462ca
