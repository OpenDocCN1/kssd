
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
