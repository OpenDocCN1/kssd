
# 完美卸载 7.00

> 原文：[`www.pediy.com/kssd/pediy04/kanxue488.htm`](https://www.pediy.com/kssd/pediy04/kanxue488.htm)

目标：完美卸载 `7.00` 版破解

工具：`trw2000`, `pw32dasm8.93`

破文作者：`zwlzwl`（信箱：`tsgmx@21cn.com`）

软件主要功能简介：本软件主要两大功能，

一是安装/卸载软件，可以做到软件安装卸载无垃圾；

二是电脑垃圾清理，可以清理数十种垃圾保持系统清洁。

本软件包含三个执行文件：`SetupMonitor.exe` 软件安装监视器；`SoftUninstall.exe` 软件卸载工具；`WjfClean.exe` 垃圾清理工具。

未注册用户将可以卸载 `60` 个软件和 `100` 次垃圾清理。

安装好程序后启动软件卸载工具（`SoftUninstall.exe`），点“关于”按钮：有关本软件的版权说明，注册用户可以从此处输入注册码进行注册。

我输入了

```
name: zwlzwl

code: 11111-22222-33333-44444-1
```

（第 `5` 个注册栏即最后的一个注册栏填入的数字这里一定不能大于 `200`，否则出错，后面我们会在程序里说明）

点注册，提示注册码错误或一码多用

“或一码多用”这句话说明软件注册码是和硬件有关的，是一机一码的。

我们退出程序，运行 `pw32dasm8.93` 把 `SoftUninstall.exe` 反汇编

从串式参考中我们可看到：“序列号输入错误或试图一码多用”

用鼠标对着这句话双击，来到

```
* Possible StringData Ref from Data Obj ->"序列号输入错误或试图一码多用!"
|
:00408642 6880064500               push 00450680
:00408647 EB09                     jmp 00408652
```

从这里我们向上看看是哪个关键的跳能跳过这个出错提示

很容易找到下面代码，下面就和我一起来分析这个些代码吧

```
:00408601 52                       push edx
:00408602 E841BA0100               call 00424048
:00408607 83C414                   add esp, 00000014
:0040860A 85C0                     test eax, eax <-比较最后一个注册栏填入的数字，如小于等于 0 就跳到出错
:0040860C 7E3B                     jle 00408649 <-不能跳，一跳就错
:0040860E 3DC8000000               cmp eax, 000000C8<-比较最后一个注册栏填入的数字，大于 200 就跳到出错
:00408613 7F34                     jg 00408649<-不能跳，一跳就错
:00408615 8B542414                 mov edx, dword ptr [esp+14]
:00408619 8D4C2410                 lea ecx, dword ptr [esp+10]
:0040861D 51                       push ecx <-以下几条命令是把我们填入的注册码压入堆，在下面的 CALL 里比较
:0040861E 50                       push eax
:0040861F 52                       push edx
:00408620 55                       push ebp
:00408621 53                       push ebx
:00408622 57                       push edi
:00408623 E8080B0100               call 00419130 <-比较注册码的呼叫
:00408628 83C418                   add esp, 00000018
:0040862B 84C0                     test al, al <-上面的 CALL 的出口参数，如注册码正确那么 al 不等于 0
:0040862D 6A30                     push 00000030
:0040862F 740C                     je 0040863D <-如 al 等于 0 就跳到出错提示
* Possible StringData Ref from Data Obj ->"注册成功"
|
:00408631 6888074500               push 00450788
* Possible StringData Ref from Data Obj ->"感谢您的支持与厚爱,在您的支持下我会开发出更好?
->"娜砑?
如软件变动造成您的注册码不能注册，您"
->"可以凭您的姓名、申请码、详细资料来免费升级本软"
->"件！
软件自动升级功能：只需下载最新版本，软件"
->"自动识别注册信息！
请新启动本软件！"
|
:00408636 68A8064500               push 004506A8
:0040863B EB15                     jmp 00408652
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040862F(C)
|
* Possible StringData Ref from Data Obj ->"错误"
|
:0040863D 68A0064500               push 004506A0
* Possible StringData Ref from Data Obj ->"序列号输入错误或试图一码多用!"
|
:00408642 6880064500               push 00450680
:00408647 EB09                     jmp 00408652
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040860C(C), :00408613(C)
|
:00408649 6A00                     push 00000000
:0040864B 6A00                     push 00000000
* Possible StringData Ref from Data Obj ->"序列号输入错误!"
|
:0040864D 6870064500               push 00450670
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040863B(U), :00408647(U)
|
:00408652 8BCE                     mov ecx, esi
:00408654 E83ACF0200               call 00435593
:00408659 8D4C2410                 lea ecx, dword ptr [esp+10]
:0040865D C78424C0000000FFFFFFFF   mov dword ptr [esp+000000C0], FFFFFFFF
:00408668 E82CEB0200               call 00437199
```

从上面知道比较注册码是在 `00408623 E8080B0100 call 00419130` 里面

所以我们进入这个 CALL 看看

来到下面的关键比较跳跃中

请大家跟我一起来分析程序检测注册码的核心

```
:0041B810 C7054043450034090000     mov dword ptr [00454340], 00000934
:0041B81A C7050C404500FD430000     mov dword ptr [0045400C], 000043FD
:0041B824 C705D83C4500C8000000     mov dword ptr [00453CD8], 000000C8
:0041B82E E8EDD2FFFF               call 00418B20
:0041B833 8B8424C4020000           mov eax, dword ptr [esp+000002C4]
:0041B83A 8B8C24B4020000           mov ecx, dword ptr [esp+000002B4]<-把第一个注册栏的值送到 ECX（我的为 10 进制的 11111）
:0041B841 8B6C242C                 mov ebp, dword ptr [esp+2C]<-把第一个硬件号的值送到 EBP（我的为 10 进制的 1999，每台机都不同的）
:0041B845 83C410                   add esp, 00000010
:0041B848 8B148588464500           mov edx, dword ptr [4*eax+00454688]<-取出根据硬件号算出的代码下面用到的比较值（我的为十进制的 10735）
:0041B84F 2BCD                     sub ecx, ebp <-把我们输入的第一个注册码减去第一硬件号并送会给 ECX
:0041B851 3BD1                     cmp edx, ecx <-比较 上面取出的和硬件有关的 EDX 值，和上面相减后得到的代码
:0041B853 0F851B020000             jne 0041BA74 <-不相等就跳到错误的代码里面去。
```

从上面分析我们可得到第一个注册码只需要等于（`edx+ebp`）就正确了

我的第一个注册码为：（`10735+1999`）=`12734`

下面是第二个注册码到第四个注册码的计算过程，大致上和上面第一个的差不多我就不再写了

```
:0041B859 8B9424A8020000           mov edx, dword ptr [esp+000002A8]
:0041B860 8B742418                 mov esi, dword ptr [esp+18]
:0041B864 8B0C8554434500           mov ecx, dword ptr [4*eax+00454354]
:0041B86B 2BD6                     sub edx, esi
:0041B86D 3BCA                     cmp ecx, edx<--第二个注册码比较核心
:0041B86F 0F85FF010000             jne 0041BA74
:0041B875 8B8C24AC020000           mov ecx, dword ptr [esp+000002AC]
:0041B87C 8B742410                 mov esi, dword ptr [esp+10]
:0041B880 8B148520404500           mov edx, dword ptr [4*eax+00454020]
:0041B887 2BCE                     sub ecx, esi
:0041B889 3BD1                     cmp edx, ecx
:0041B88B 0F85E3010000             jne 0041BA74<--第三个注册码比较核心
:0041B891 8B9424B0020000           mov edx, dword ptr [esp+000002B0]
:0041B898 8B742414                 mov esi, dword ptr [esp+14]
:0041B89C 8B0C85EC3C4500           mov ecx, dword ptr [4*eax+00453CEC]
:0041B8A3 2BD6                     sub edx, esi
:0041B8A5 3BCA                     cmp ecx, edx<--第四个注册码比较核心
:0041B8A7 0F85C7010000             jne 0041BA74
:0041B8AD 8B8424B8020000           mov eax, dword ptr [esp+000002B8]
:0041B8B4 83C9FF                   or ecx, FFFFFFFF
:0041B8B7 8D542420                 lea edx, dword ptr [esp+20]
:0041B8BB B363                     mov bl, 63
:0041B8BD 8B38                     mov edi, dword ptr [eax]
:0041B8BF 33C0                     xor eax, eax
:0041B8C1 F2                       repnz
:0041B8C2 AE                       scasb
:0041B8C3 F7D1                     not ecx
:0041B8C5 2BF9                     sub edi, ecx
```

我的申请注册码为：`4461/1996/622/48353`（这个是要象作者报告的号码）

经过以上分析我们可找到正确的注册码了

我的为

```
name: zwlzwl(姓名名可任意填）

code: 12734-12753-37318-17657-1(这个注册码只有我的机器好用的，每台机器不同的，请大家不要使用）
```

欢迎大家来信和我交流，转载请保持完整。

我的信箱：`tsgmx@21cn.com` `4dd466d0e1e0cc123afbe504da2462ca`
