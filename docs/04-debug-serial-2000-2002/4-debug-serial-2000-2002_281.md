
# Don't Panic 3.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1001.htm`](https://www.pediy.com/kssd/pediy03/forum1001.htm)

在这里,俺看了、学了很多的东东,很是精彩,很是值得学习,但是,更主要是一种精神,这才是让我最值得学习的.

这是俺第一次写破解过程,会有很多不对的地方,请各位多多鼓励,多提建议!!!多谢.

软件名称: `Don't Panic`

软件下载: `www.newhua.com`

软件用途: 这个东东可以让你快速的隐藏你所开的窗口(包括任务栏),并且留下你想留给外面看的内容.说简单一点,就是以防不法活动时被擒...

工具: `TRW2000` `1.22`, `W32dasm`, `FileInfo` `v2.43`

日期: `2000.5.14`

作者: `Fengy`

## 过程

1) 使用 `FileInfo` `v2.43` 检测主程序 `dp.exe`,没有壳的,好,否则,俺就不会了:)

2) 用 `W32dasm` 反编译 `dp.exe`,结果如下:

```
:00410FE0 A5                movsd
:00410FE1 8BCA              mov ecx, edx
:00410FE3 83E103            and ecx, 00000003
:00410FE6 F3                repz
:00410FE7 A4                movsb
:00410FE8 E8F3FEFFFF        call 00410EE0
:00410FED 8BF0              mov esi, eax
:00410FEF 56                push esi                << ---- esi 中为随便输入的注册码
:00410FF0 E8DBFDFFFF        call 00410DD0           << ---- 注册码验证子程序,追入
:00410FF5 83C408            add esp, 00000008
:00410FF8 85C0              test eax, eax
:00410FFA 7558              jne 00411054            << ---- 这里就是分界点了
:00410FFC 6A10              push 00000010

* Possible StringData Ref from Data Obj ->"Don't Panic!"
                                        |
:00410FFE 6874064200          push 00420674 <<----这里就是输入错误 regcode 时的提示信息.

* Possible StringData Ref from Data Obj ->"The registration number you have "
                                        ->"entered is not valid.  Please "
                                        ->"enter a valid registration number."
                                        |
:00411003 6810224200          push 0042210
:00411008 53                push ebx

*******************************************************************

* Referenced by a CALL at Addresses:
|:00404989 , :0040BD16 , :0040BD85 , :0040CD02 , :00410FF0
|
:00410DD0 8B442404            mov eax, dword ptr [esp+04]
:00410DD4 68D135E2E1          push E1E235D1     <<----
:00410DD9 681953C633          push 33C65319     <<----这两个就是验证注册码正确与否的参数了
:00410DDE 50                push eax
:00410DDF E8DCFFFFFF          call 00410DC0     <<----继续进入
:00410DE4 33D2              xor edx, edx
:00410DE6 B9BB0B0000          mov ecx, 00000BBB
:00410DEB F7F1              div ecx
:00410DED 83C40C            add esp, 0000000C
:00410DF0 8BC2              mov eax, edx      <<----余数 或者 说是模操作的结果
:00410DF2 F7D8              neg eax          <<----只有 eax = 0 时,cf = 0
:00410DF4 1BC0              sbb eax, eax
:00410DF6 40                inc eax
:00410DF7 C3                ret

* Referenced by a CALL at Address:
|:00410DDF
|
:00410DC0 8B442404            mov eax, dword ptr [esp+04] <<----很是眼熟吧,就是你随便输入的注册码
:00410DC4 8B4C240C            mov ecx, dword ptr [esp+0C] <<---- 0xE1E235D1
:00410DC8 33C1              xor eax, ecx
:00410DCA 0FAF442408          imul eax, dword ptr [esp+08] <<-----0x33C65319
:00410DCF C3                ret
```

注册码验证的过程很简单:

`(XXX xor 0xE1E235D1) * 0x33C65319 % 0x00000BBB == 0` 就可以了

如:`xxx = 0xE1E233D8`
`= 3789698008`

3) 几点说明

   (1) 在分界点直接修改 `jne 00411054` ---> `je 00411054` 时,可以看到对注册表的操作...,注册成功,但是,下次,还提示要求注册,可能还可以通过修改注册表来实现注册,我没试过,谁有兴趣可以试试.

   (2) 注册验证中,`* 0x33C65319` 的结果的低位部分保存在 `eax` 中,是用 `eax` 来与 `0x0BBB` 取模的.

   (3) 算 `XXX` 的过程,我没有研究明白,好像不能逆运算得到(请指教),因为用的是低位结果.所以,我是用 `C` 写了个程序穷举得到的结果.

   -------`Fengy`------- `4dd466d0e1e0cc123afbe504da2462ca`
