
# S-Demo2.0 一字节暴力破解

> 原文：[`www.pediy.com/kssd/pediy04/kanxue182.htm`](https://www.pediy.com/kssd/pediy04/kanxue182.htm)

作者：`maomao`

软件：`Screen-Demo2.0`

`Screen-Demo2.0` 是一个极棒的屏幕抓录软件，一般被 `Cracker` 用于录制动画解密教程。

下载：`http://www.qwerks.com/download/2834/SD20Setup.exe` (950KB)

工具：`W32Dasm8.93` `UltraEdit8.0`

日期：2001-10-27

昨天写了一篇 `ScreenRecord1.2` 的注册破文和注册机编写过程，网友的鼓励给我了很大的信心。于是又按捺不住，下载了一个压缩率更高的屏幕制作软件 `Screen-Demo2.0`，安装后有三个主要文件：`S-Recorder` (录制主文件)、`S-Player` (播放器)、`S-Capture` (拷屏)，安装后试用，发现录制 1 分钟左右的内容做成 `AVI`，才 400 多 K! 而如果用 `ScreenRecord1.2` 制作，需要 30M 左右! 强烈的责任感驱使我破解它，现把破解过程记录如下，当作申请加入 `BCG` 组织的第二篇破文，不足之处，望各位高手多多指点…… :)

首先，用 `Language 2000` 侦测，没有壳，省了不少事。

运行 `S-Recorder`，出来一个注册对话框，完全在情理之中，随便输入一些内容，按“`OK`”后提示“`Register Failure!`”，没关系，一名 `cracker` 最基本的素质是需要有耐心。

进入到程序目录，打开 `S-Demo.ini` 文件，你可以看到如下内容：

```ini
[Register]
Install=HKCSRC0RG0QGVQZVKZK
Name=maomao
Password=1234567890

[Option]
DoubleSize=1
Color=3
ImageQuality=64
AlwayNoImage=0
CompressNow=1
CaptureSpeed=2
KeyFrame=20
SlowRecord=0
SlowMinute=1
Audio=1
```

其中 `Name` 和 `Password` 就是你刚才输入的内容。看来 `S-Recorder` 是读取 `S-Demo.ini` 中的注册信息，再进行计算比较，如果用 `Name` 计算出来的 `Password` 与其中的不一样，就要弹出注册窗口了! OK，思路确定：寻找程序中读取 `INI` 文件的函数，再分析读完后的比较判断语句，小作修改后，嘿嘿，让它认为这个 `Password` 是对的....(窃笑三声)

使用静态反编译工具 `W32Dasm8.93` 进行反编译，一会儿程序代码就在 `W32Dasm` 的窗口中了。选择菜单“函数”->“输入”或点击 `W32Dasm` 工具栏上的“`IMP`”按钮，弹出一个输入函数对话框，其中列出了程序输入的所有外部函数。排在第一、二位的 `RegOpenKeyA` 和 `RegQueryValueExA` 正是我们要找的函数! 它们负责打开注册表或 `INI` 文件的主键和读取键值。双击 `RegOpenKeyA`，我们会来到这一段代码：

```
* Referenced by a CALL at Address:
|:004035C2     <=====从这里的地址跳来
|
:00407C30 83EC0C                sub esp, 0000000C
:00407C33 56                    push esi
* Reference To: ADVAPI32.RegOpenKeyA, Ord:0171h
                              |
:00407C34 8B3500904100          mov esi, dword ptr [00419000]<=====光标条停在这一行
:00407C3A 8D442404              lea eax, dword ptr [esp+04]
:00407C3E 57                    push edi
:00407C3F 50                    push eax
```

原来是 `004035C2` 读取注册表和 `INI` 文件。赶快按下 `SHIFT+F12`，输入 `004035C2`，回车后来到这里：

```
:004035BC A4                    movsb
:004035BD B9C0E74200            mov ecx, 0042E7C0
:004035C2 E869460000            call 00407C30  <=====调用读注册表和 INI 函数(光标停在这里)
:004035C7 B9C0E74200            mov ecx, 0042E7C0
:004035CC E89F480000            call 00407E70  <=====计算并比较注册码
:004035D1 85C0                  test eax, eax
:004035D3 0F8544010000          jne 0040371D    <=====跳走就完了
:004035D9 50                    push eax
:004035DA 8D4C2438              lea ecx, dword ptr [esp+38]
:004035DE E84DDEFFFF            call 00401430
:004035E3 6894E84200            push 0042E894
:004035E8 8D8C2498000000        lea ecx, dword ptr [esp+00000098]
:004035EF C68424D801000004      mov byte ptr [esp+000001D8], 04
```

将光标停在 `:004035CC E89F480000  call 00407E70` 这一行上，接向右的光标键“->”，程序会来到 `407E70` 开始的程序段：

```
* Referenced by a CALL at Address:
|:004035CC     <=====说明这段程序的调用者
|
:00407E70 81ECD0000000          sub esp, 000000D0 <=====光标停在这一行上
:00407E76 53                    push ebx
:00407E77 55                    push ebp
:00407E78 56                    push esi
:00407E79 8BE9                  mov ebp, ecx
:00407E7B 57                    push edi
:00407E7C 896C2410              mov dword ptr [esp+10], ebp
...                          (这里是读用户的信息，程序比较长，故省略)
```

一直来到这里：

```
* Possible StringData Ref from Data Obj ->"Clayman"
                          |
:00407F9D BED0D14100            mov esi, 0041D1D0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407FC4(C)
|
:00407FA2 8A13                  mov dl, byte ptr [ebx]  <=====读取用户注册码第一字节(已变换)
:00407FA4 8A0E                  mov cl, byte ptr [esi]  <=====读取正确注册码第一字节(已变换)
:00407FA6 8AC2                  mov al, dl
:00407FA8 3AD1                  cmp dl, cl              <=====比较
:00407FAA 751E                  jne 00407FCA            <=====不相同就跳走
:00407FAC 84C0                  test al, al
:00407FAE 7416                  je 00407FC6
:00407FB0 8A4B01                mov cl, byte ptr [ebx+01]<=====取用户注册码下一字节(已变换)
:00407FB3 8A5601                mov dl, byte ptr [esi+01]<=====取正确注册码下一字节(已变换)
:00407FB6 8AC1                  mov al, cl
:00407FB8 3ACA                  cmp cl, dl              <=====比较
:00407FBA 750E                  jne 00407FCA            <=====不相同就跳走
:00407FBC 83C302                add ebx, 00000002
:00407FBF 83C602                add esi, 00000002
:00407FC2 84C0                  test al, al
:00407FC4 75DC                  jne 00407FA2            <=====没有比较完则继续比较

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407FAE(C)
|
:00407FC6 33C0                  xor eax, eax
:00407FC8 EB05                  jmp 00407FCF

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00407FAA(C), :00407FBA(C)
|
:00407FCA 1BC0                  sbb eax, eax
:00407FCC 83D8FF                sbb eax, FFFFFFFF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00407FC8(U)
|
:00407FCF 85C0                  test eax, eax
:00407FD1 741B                  je 00407FEE             <=====注册比较正确时跳转
:00407FD3 8B542410              mov edx, dword ptr [esp+10]    <=====注册码错误的程序段
:00407FD7 5F                    pop edi
:00407FD8 5E                    pop esi
:00407FD9 5D                    pop ebp
:00407FDA C782D000000001000000  mov dword ptr [ebx+000000D0], 00000001
:00407FE4 33C0                  xor eax, eax              <=====执行到这里就完了
:00407FE6 5B                    pop ebx
:00407FE7 81C4D0000000          add esp, 000000D0
:00407FED C3                    ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00407F77(C), :00407FD1(C)
|
:00407FEE 8B442410              mov eax, dword ptr [esp+10]<=====如果注册码正确就会跳到这里
:00407FF2 5F                    pop edi
:00407FF3 5E                    pop esi
:00407FF4 5D                    pop ebp
:00407FF5 C780D000000000000000  mov dword ptr [ebx+000000D0], 00000000<=====送标志
:00407FFF B801000000            mov eax, 00000001         <=====哈哈，有没有看到这个标志?!
:00408004 5B                    pop ebx
:00408005 81C4D0000000          add esp, 000000D0
:0040800B C3                    ret
```

这样，只要将 `00407FD1` 处的跳转指令 `74 1B` (`je 00407FEE`) 改为 `EB 1B` (`jmp 00407FEE`) 即可。用 `UltraEdit` 打开 `S-Recorder.exe`，修改偏移量为 `0x7FD1` 处的代码 `74 1B` 为 `EB 1B`，哈哈，只修改了一个字节!

说明：由于本人功力不深，又懒得分析其机器代码的生成和用户输入注册码的多次变换，所以也写不出一个通用的注册机，有兴趣的可以分析一下上面的轮换机制。本人分析不当之处，请多多指教。

收工~~~~~~ `4dd466d0e1e0cc123afbe504da2462ca`
