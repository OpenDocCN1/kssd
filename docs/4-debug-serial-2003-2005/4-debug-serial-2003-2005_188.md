
# MagicWin 98 Release 1.20

> 原文：[`www.pediy.com/kssd/pediy05/pediy50160.htm`](https://www.pediy.com/kssd/pediy05/pediy50160.htm)

**MagicWin 98 Release 1.20 破解手记**

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

**目的：** 制作注册机

**Cracked by:** `WksWlj999@sohu.com`

**软件说明：** 著名的内码转换工具

**破解工具：** `w32dasm8.93`; `SoftICE 4.05 For Win9X`

**日期：** 2002 年 5 月 25 日

**注意：** 高手莫看！这是小弟看了“看雪”兄的大作《加密与解密——软件保护技术及完全解决方案》一书后，恰好机子上装的 `MagicWin 98` 没有注册，就拿它来开刀吧，本人刚刚涉足加密与解密，水平有限，文中有不足之处还望各位大虾不吝指教！此文仅供初学者参考。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

运行 `MagicWin 98`，在注册窗口输入：

- `Your Name`: `WksWlj999`
- `Serial Number`: `MGW45146-10` (程序默认自动填入)
- `Registration Code`: `369369369369` (随便输，为何要这么长呢，下面会讲的)

然后在 `SoftICE` 里下断点 `bpx getdlgitemtext` (为何在 16 位 API 函数下断点呢，因为我用 `w32dasm8.93` 查看过 `MagicWin 98` 的函数输入表啊！)，然后点 "Register"，`SoftICE` 拦截并中断跳出，按 `F11` 返回 `MagicWin 98` 的领空。

```
* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:006E, ""
                                |
:0047.0408 6A6E                push 006E
:0047.040A 16                  push ss
:0047.040B 8D46E4              lea ax, [bp-1C]
:0047.040E 50                  push ax
:0047.040F 6A1C                push 001C
:0047.0411 9AFFFF0000          call USER.GETDLGITEMTEXT
:0047.0416 56                  push si            //按 F11 返回后停在此处
```

下命令 `bc *` 清除下的断点，然后按 `F10` 单步跟踪......

```
:0047.0434 50                  push ax
:0047.0435 680001              push 0100
:0047.0438 9AFFFF0000          call USER.GETWINDOWTEXT
:0047.043D 837E0800            cmp word ptr [bp+08], 0000
:0047.0441 7503                jne 0446
:0047.0443 E9BC00              jmp 0502            //来到这个绝对跳转，继续 F10 跟踪
```

\--------------------------------------------------------------

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0047.0443(U)
|
* Possible Reference to Menu: MenuID_0001
                                |
* Possible Reference to Dialog: DialogID_0001
                                |
* Possible Reference to String Resource ID=00001: "ASCII"
                                |
:0047.0502 6A01                push 0001
:0047.0504 16                  push ss
:0047.0505 8D86C8FE            lea ax, [bp+FEC8]
:0047.0509 50                  push ax            //输入的 Your  Name:"WksWlj999"进栈,以下简称"Name"
:0047.050A 16                  push ss
:0047.050B 8D46C8              lea ax, [bp-38]
:0047.050E 50                  push ax            //输入的假 Registration Code:"369369369369"进栈,以下简称"Code"
:0047.050F 16                  push ss
:0047.0510 8D46E4              lea ax, [bp-1C]
:0047.0513 50                  push ax            //程序默认的 Serial  Number:"MGW45146-10"进栈,以下简称"Serial"
:0047.0514 9AFFFF0000          call 0037.03ABh    //这个紧跟的 Call 很可疑，按 F8 跟进
:0047.0519 83C40E              add sp, 000E
```

```
* Possible Reference to Menu: MenuID_0001
                                |
* Possible Reference to Dialog: DialogID_0001
                                |
:0047.051C 3D0100              cmp ax, 0001       //比较 AX 与 0001（根据 call  0037.03ABh 的结果，注册正确返回 AX=0 否则 AX=1）
:0047.051F 7558                jne 0579           //AX=1 则跳到注册错误对话框
:0047.0521 9AFFFF0000          call 0044.0073h    //以下前往注册正确对话框
:0047.0526 52                  push dx
:0047.0527 50                  push ax
:0047.0528 9AFFFF0000          call 0044.002Dh
:0047.052D 52                  push dx
:0047.052E 50                  push ax
:0047.052F 1E                  push ds
......
```

\----------------------------------------------------

接 `call 0037.03ABh`，`F8` 进入后，`F8` 单步跟踪

```
* Referenced by a CALL at Address:
|:0037.017F
|
:0037.03AB 55                  push bp
:0037.03AC 8BEC                mov bp, sp
:0037.03AE 81EC8600            sub sp, 0086
:0037.03B2 56                  push si
:0037.03B3 57                  push di
:0037.03B4 C746FE0000          mov word ptr [bp-02], 0000
:0037.03B9 66FF7606            push word ptr [bp+06]      //Serial 进栈，d  *(bp+06)可看到"MGW45146-10"
:0037.03BD 9AFFFF0000          call KERNEL.LSTRLEN        //计算长度置入 AX=Bh
:0037.03C2 50                  push ax                    //长度 AX=Bh 进栈保存
:0037.03C3 66FF760A            push word ptr [bp+0A]      //假 Code 进栈，d *(bp+0A)可看到"369369369369"
:0037.03C7 9AFFFF0000          call KERNEL.LSTRLEN        //计算长度置入 AX=Ch
:0037.03CC 5A                  pop dx                     //Serial 长度 BH 出栈，DX=Bh
:0037.03CD 3BD0                cmp dx, ax                 //比较 Serial 和假 Code 的长度
:0037.03CF 7E25                jle 03F6                   //Serial 长度小于等于假 Code 长度 Bh（即 11 位）就跳，否则注册
:0037.03D1 66FF760A            push word ptr [bp+0A]      错误，这就是为何假 Code 的长度要大于等于 Serial 长度的原因！
:0037.03D5 1E                  push ds
......
```

跳到这里

```
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0037.03CF(C), :0037.03E0(C), :0037.03F1(C)
|
:0037.03F6 C45E0A              les bx, [bp+0A]
:0037.03F9 26803F00            cmp byte ptr es:[bx], 00    //假 Code 是否为空?
:0037.03FD 7503                jne 0402                     //不空就跳到 0402
:0037.03FF E9C801              jmp 05CA
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.03FD(C)
|
:0037.0402 16                  push ss
:0037.0403 8D867AFF            lea ax, [bp+FF7A]
:0037.0407 50                  push ax
:0037.0408 66FF7606            push word ptr [bp+06]
:0037.040C 9AFFFF0000          call KERNEL.LSTRCOPY        //拷贝 Serial 到内存地址[bp+FF7A]
:0037.0411 16                  push ss
:0037.0412 8D867AFF            lea ax, [bp+FF7A]
:0037.0416 50                  push ax                     //"d ax"可看到拷贝来的 Serial:"MGW45146-10"
:0037.0417 9AFFFF0000          call KERNEL.LSTRLEN         //计算长度置入 AX
:0037.041C 8BF8                mov di, ax
```

```
* Possible Reference to Menu: MenuID_0003
                                |
* Possible Reference to Dialog: DialogID_0003
                                |
* Possible Reference to String Resource ID=00003: "ChineseGuoBiao"
                                |
:0037.041E BA0300              mov dx, 0003                //dx=3
:0037.0421 8D867DFF            lea ax, [bp+FF7D]            //将[bp+FF7D]地址传入 AX，即 Serial 第 4 位的地址
:0037.0425 8BF0                mov si, ax
:0037.0427 EB4C                jmp 0475                     //跳到 0475，将 Serial 执行换位操作
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.0479(C)
|
:0037.0429 8BC2                mov ax, dx //0429-0479 处代码功能是保持 Serial Number 的前 3 位不变，然后依次将其
:0037.042B 050400              add ax, 0004 第 4 位与第 8 位互换位置，第 5 位与第 9 位互换，第 6 位与第 10 位互换，第 7 位
:0037.042E 3BC7                cmp ax, di 与第 11 位互换，第 8 位与第 9 位互换，第 9 位与第 10 位互换，第 10 位与第 11 位互换
:0037.0430 7D1E                jge0450 比如：Serial 是"XXX12345678",则换位结束后得到"XXX56782341"
:0037.0432 368A0C              mov cl , ss:[si]
:0037.0435 8D867EFF            leaax, [bp+FF7E] //经验证,换位规则是:当 Serial 长度小于等于 3 时，不换位，直接在 Serial 后加上
:0037.0439 8BDA                mov bx,dx Name 的长度与其各位 ASC 码值的和（十进制），即为正确的 Code;当 Serial 长度
:0037.043B 03D8                add bx, ax 大于 3 时，从第 4 位开始，先将位数加 4，值在 Serial 长度范围内，则将此位与其
:0037.043D 368A07             mov al , ss:[bx] 位数加 4 所在位互换位置，然后顺序执行下一位的判断；若其值超出 Serial 长度
:0037.0440 368804              mov ss:[si], al 范围，则判断此位的下一位数是否在 Serial 长度范围内，若在则将此位与
:0037.0443 8D867EFF            lea ax, [bp+FF7E] 其下一位互换，然后顺序执行下一位的判断，若超出则换位结束。
```



# 汇编代码调试分析

以下是对 `MagicWin` 程序相关汇编代码片段及其注释的整理。代码展示了从地址 `:0037.0447` 开始到 `:0037.0522` 的指令流，包括寄存器操作、内存访问以及条件跳转逻辑。

## 代码段 1: 序列号移位处理

```asm
:0037.0447 8BDA              mov bx, dx
:0037.0449 03D8              add bx, ax
:0037.044B 36880F            mov ss:[bx], cl
:0037.044E EB23              jmp 0473

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.0430(C)
|
```

## 代码段 2: 循环计数器与字符交换

```asm
:0037.0450 8BC2              mov ax, dx
:0037.0452 40                inc ax
:0037.0453 3BC7              cmp ax, di
:0037.0455 7D1C              jge 0473
:0037.0457 368A0C            mov cl, ss:[si]
:0037.045A 8D867BFF          lea ax, [bp+FF7B]
:0037.045E 8BDA              mov bx, dx
:0037.0460 03D8              add bx, ax
:0037.0462 368A07            mov al, ss:[bx]
:0037.0465 368804            mov ss:[si], al
:0037.0468 8D867BFF          lea ax, [bp+FF7B]
:0037.046C 8BDA              mov bx, dx
:0037.046E 03D8              add bx, ax
:0037.0470 36880F            mov ss:[bx], cl

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0037.044E(U), :0037.0455(C)
|
```

## 代码段 3: 检查换行符

```asm
:0037.0473 46                inc si
:0037.0474 42                inc dx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.0427(U)
|
:0037.0475 36803C00          cmp byte ptr ss:[si], 00    //ss:[si]处字节是否为空，即比较是否 Serial Number 换位结束
:0037.0479 75AE              jne 0429                    //不空则跳至 0429 继续换位
```

* 可能引用菜单: `MenuID_0003`
* 可能引用对话框: `DialogID_0003`
* 可能引用字符串资源 ID=`00003`: `"ChineseGuoBiao"`

## 代码段 4: 注册码计算逻辑 (ASCII 码偏移)

```asm
:0037.047B BA0300            mov dx, 0003
:0037.047E 3BD7              cmp dx, di
:0037.0480 7D19              jge 049B

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.0499(C)
|

:0037.0482 8D867AFF          lea ax, [bp+FF7A]           //此句过后紧跟着下"d AX"命令可看到换位后的 Serial Number="MGW6-105144"
:0037.0486 8BDA              mov bx, dx                  //dx=0003
:0037.0488 03D8              add bx, ax                  //位置调整到 Serial Number 的第 dx 位
:0037.048A 8BF3              mov si, bx
:0037.048C 368A07            mov al, ss:[bx]             //取 Serial Number 的第 4 位
:0037.048F 02C2              add al, dl                  //al=al+dl
:0037.0491 0410              add al, 10                  //al=al+10h
:0037.0493 368804            mov ss:[si], al
:0037.0496 42                inc dx                      //dx=dx+1
:0037.0497 3BD7              cmp dx, di
:0037.0499 7CE7              jl 0482                     //0482-0499 处代码功能是取换位后的 Serial Number 的第 4 位到第 11 位,将它们的 ASC 码值分别加上 13h,14h,15h,16h,17h,18h,19h,1Ah 得到"MGWIAFFLIMN"

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.0480(C)
|
```

## 代码段 5: 用户名长度计算与校验和生成

```asm
:0037.049B 66FF760E          push word ptr [bp+0E]       //Your Name:="WksWlj999"进栈,d *(bp+0e)在数据窗可看到
:0037.049F 9AFFFF0000        call KERNEL.LSTRLEN         //计算长度,AX=9h
:0037.04A4 8BC8              mov cx, ax                  //CX=AX=9h
:0037.04A6 660FBFC1         movsx eax, ecx
:0037.04AA 668946FA         mov [bp-06], eax
:0037.04AE 33D2              xor dx, dx                  //dx=0
:0037.04B0 8B760E            mov si, [bp+0E]
:0037.04B3 3BD1              cmp dx, cx                  //dx 大于等于 Your Name 的长度吗?
:0037.04B5 7D15              jge 04CC                    //大于等于则跳到 04CC,否则继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.04CA(C)
|
```

## 代码段 6: 累加 ASCII 值

```asm
:0037.04B7 8E4610            mov es, [bp+10]             //04B7-04CA 处代码功能将 Your Name 的长度 9h 与其每位的 ASC 码值(16 进制)加到一起 并将和存入地址[bp-06]中
:0037.04BA 268A04            mov al, es:[si]
:0037.04BD 98                cbw
:0037.04BE 660FBFC0         movsx eax, eax
:0037.04C2 660146FA         add [bp-06], eax
:0037.04C6 46                inc si
:0037.04C7 42                inc dx
:0037.04C8 3BD1              cmp dx, cx
:0037.04CA 7CEB              jl 04B7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0037.04B5(C)
|
```

## 代码段 7: 最终注册码校验

```asm
:0037.04CC 66FF76FA          push word ptr [bp-06]       //将和进栈,d *(bp-06)可看到数据窗中"16 03 00 00"即和是 0316h(16 进制)
:0037.04D0 1E                push ds
* Possible StringData Ref from Data Seg 069 ->"%"
:0037.04D1 681F1E            push 1E1F
:0037.04D4 8D867AFF          lea ax, [bp+FF7A]           //将"MGWIAFFLIMN"的首地址入 ax
:0037.04D8 8BD7              mov dx, di                  //将"MGWIAFFLIMN"的长度 Bh 入 dx
:0037.04DA 03D0              add dx, ax                  //dx=dx+ax,此时 dx 指向的是紧跟"MGWIAFFLIMN"后的位置
:0037.04DC 16                push ss                     //USER._WSPRINTF 函数的参数 1 进栈
:0037.04DD 52                push dx                   //USER._WSPRINTF 函数的参数 2 进栈
:0037.04DE 9AFFFF0000        call USER._WSPRINTF       //将和 0316h 由 16 进制转换为十进制的 790d 并输出到 dx 指向的位置
:0037.04E3 83C40C            add sp, 000C
:0037.04E6 16                push ss
:0037.04E7 8D867AFF          lea ax, [bp+FF7A]
:0037.04EB 50                push ax                   //"d ax"可看到真的 Registration Code="MGWIAFFLIMN790"
:0037.04EC 66FF760A          push word ptr [bp+0A]     //"d *(bp+0a)"可看到输入的假 Registration Code="369369369369"
:0037.04F0 9AFFFF0000        call USER.LSTRCMP         //关键的比较!!!注册码正确置 ax=0,错误置 ax=1
:0037.04F5 0BC0              or ax, ax
:0037.04F7 7425              je 051E                   //关键的跳转!!!ax=0 则跳转到注册正确程序段,可在此处使用"r fl z"命令 使 ZF 标志位=1,则跳到 051E,注册就成功了.然后程序会自动将正确的 Serial Number 和计算出的真 Registration Code 以及你输入的 Name 一起写到 MagicWin.ini 中保存起来 下次启动程序就可从 MagicWin.ini 中读取出正确的注册信息了
:0037.04F9 66FF760A          push word ptr [bp+0A]
:0037.04FD 1E                push ds
* Possible StringData Ref from Data Seg 069 ->"B"
:0037.04FE 68231E            push 1E23
:0037.0501 9AFFFF0000        call USER.LSTRCMPI
:0037.0506 0BC0              or ax, ax
:0037.0508 7414              je 051E
:0037.050A 66FF760A          push word ptr [bp+0A]
:0037.050E 1E                push ds
* Possible StringData Ref from Data Seg 069 ->"M"
:0037.050F 68E11D            push 1DE1
:0037.0512 9AFFFF0000        call USER.LSTRCMPI
:0037.0517 0BC0              or ax, ax
:0037.0519 7403              je 051E
:0037.051B E9AC00            jmp 05CA

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0037.04F7(C), :0037.0508(C), :0037.0519(C)
|

:0037.051E 66FF760A          push word ptr [bp+0A]
:0037.0522 1E                push ds
```

---

## 总结

根据上述调试结果，`MagicWin` 98 Release 1.20 的注册逻辑总结如下：

*   **Your Name**: `WksWlj999`
*   **Serial Number**: `MGW45146-10`
*   **Registration Code**: `MGWIAFFLIMN790`

注册信息保存在安装目录下的 `MagicWin.ini` 文件中，格式如下：

```ini
[Software Lock]
Serial No=MGW45146-10
Password=MGWIAFFLIMN790
Second Lock=47974l    //这个值不晓得做什么用,好象注册成功后把它删除,程序也能正常使用

[Usage]
Count=5               //这个是未注册前已使用的次数

[UserInfo]
User=WksWlj999
```

根据上面的调试结果写出 `MagicWin` 98 Release 1.20 的注册机如下（用 VB6 编写的，小弟 C 语言功底不行，汇编看看代码还可以，自己写也不行，虽然讨厌 VB6 必须带一个老大的 `MSVBVM60.dll`，也没办法:-\(，早说了咱水平不行嘛！（自己找台阶下了\^\_^））

> ```vb
> '----------------START CODE-----------------------------------
> Dim myname As String        '定义变量
> ```



```vb
Dim serial As String   
Dim mycode As String   
Dim i As Integer   
Dim ADD As Integer   

Private Sub Command1_Click()   
    serial = Text1.Text     '获取输入的 Serial Number   
    myname = Text2.Text     '获取输入的 Your Name   
    
    '根据 Serial 长度判断是否执行换位操作   
    If Len(serial) <= 3 Then   
        mycode = serial   
        Else   
        'Serial 长度>4,执行换位   
        mycode = Serial_Change(serial)   
        mycode = Serial_PLUS(mycode)   
    End If   
    
    '计算 Your Name 每位的 ASC 码值之总和并加上其本身长度，存入 ADD 变量中   
    For i = 1 To Len(myname)   
        ADD = ADD + Asc(Mid$(myname, i, 1))   
    Next i   
    ADD = ADD + Len(myname)   
    
    '组合出完整的 Registration Code   
    mycode = mycode + CStr(ADD)   
    
    '输出结果到文本框中   
    Text3.Text = mycode   
End Sub   

'对 Serial Number 进行换位操作，不明白的话可参见":0037.0429"处的注释   
Function Serial_Change(ByVal OPR_Serial As String) As String   
    Dim tempinteger As Integer   
    ReDim a(4 To Len(serial)) As Integer     '重定义数组   
    '对数组 a(4 to Len(serial))赋值为与其下标号相同的正整数,即 a(4)=4,a(5)=5......   
    For i = 4 To Len(serial)   
        a(i) = i   
    Next i   
    
    '从第 4 位开始执行换位,换位结束时,数组的下标代表了其在变换过的 Serial 中的位置   
    '而其值则代表了此位置的数值是原来 Serial 的哪一位换位过来的,比如换位结束后若   
    'a(4)=8 则表明换位后的 Serial 的第 4 位应该是原始 Serial 的第 8 位   
    i = 4   
    Do While i + 4 <= Len(serial)   
        tempinteger = a(i)   
        a(i) = a(i + 4)   
        a(i + 4) = tempinteger   
        i = i + 1   
    Loop   
    Do While i + 1 <= Len(serial)   
        tempinteger = a(i)   
        a(i) = a(i + 1)   
        a(i + 1) = tempinteger   
        i = i + 1   
    Loop   
    
    '将 a(i)对应的原始 Serial 中的值取出来,按下标顺序排列出换位后的 Serial   
    OPR_Serial = Left$(OPR_Serial, 3)   
    For i = 4 To Len(serial)   
        OPR_Serial = OPR_Serial + Mid$(serial, a(i), 1)   
    Next i   
    Serial_Change = OPR_Serial   
End Function   

'将换位后的 Serial Number 的第 4 位到末位的 ASC 码值依次分别加上 19d(13h),20d(14h)......   
Function Serial_PLUS(ByVal CHG_Serial As String) As String   
    Dim tempstring As String   
    tempstring = CHG_Serial   
    CHG_Serial = Left$(CHG_Serial, 3)   
    i = 1   
    Do While i + 3 <= Len(tempstring)   
        CHG_Serial = CHG_Serial + Chr$(Asc(Mid$(tempstring, i + 3, 1)) + 18 + i)   
        i = i + 1   
    Loop   
    Serial_PLUS = CHG_Serial   
End Function   
```

----------------END CODE-------------------------------------
最后补充一下，你可以任意输入`Serial Number`，可以包含所有的可显示字符，但要注意的一点是其`ASC`码值经变换加值后再转换为字符时，有时无法正常显示出来，因为其值已经超出`255`或本身就是无法显示的。
