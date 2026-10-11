
# 美萍网管大师 v5.2

> 原文：[`www.pediy.com/kssd/pediy02/forum282.htm`](https://www.pediy.com/kssd/pediy02/forum282.htm)

使用 `W32dasm` 分析破解美萍网管大师 v5.2，并使用 `Crackcode2000` 制作注册机。

*   本文作者：wind[CCG]
*   我的网站：`http://biggow.8u8.com`（有空来看看）
*   China Cracking Group 是中国自己的破解组织，有兴趣可以和 SunBird 大哥联系！
*   他的 E-mail：`sun.bird@371.net`

## 开篇

美萍安全卫士 v5.2 可以防 `trw2000` 和 `softice`，而且极为凶悍，有些程序检测到这类跟踪调试软件只不过是显示一个 nag 然后终止程序运行，而美萍网管大师 v5.2（包括安全卫士的最新版本）发现此类程序，就关机，所以这次使用 `w32dasm` 超级中文版来破解它！

## 破解过程

1.  `scon.exe` 这个文件是用 `aspack` 1.07 加的壳，直接使用 `unaspack` 就可以搞定了！
2.  启动 `W32dasm` 反汇编 `scon.exe`。那么我们从何入手呢？看看说明；哦，有时间限制，还有那个注册框上显示的未注册一句话作为入手！如果在 `W32dasm` 中找不到，那么我们可以通过注册表，因为它通过阅读注册表的信息，然后判断是否注册。很高兴，我们找到了这么一句话“软件试用期还剩”，我们就从这里入手！

双击它，然后来到如下程序段：

```asm
* Possible StringData Ref from Code Obj ->"软件试用期还剩"
                        |
:004846DA 6808494800                 push 00484908
:004846DF 8D55DC                     lea edx, dword ptr [ebp-24]
:004846E2 A1F48D4800                 mov eax, dword ptr [00488DF4]
:004846E7 40                         inc eax
:004846E8 2B0544DE4A00               sub eax, dword ptr [004ADE44]
:004846EE E8392EF8FF                 call 0040752C
:004846F3 FF75DC                     push [ebp-24]
:004846F6 6820494800                 push 00484920
/*当然这里没有我们需要的信息，我们需要往上找*/
```

往上来到如下：

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484476(C)
|
:004846A3 E8EC55FEFF                 call 00469C94 --->判断什么的？
:004846A8 85C0                       test eax, eax
:004846AA 7577                       jne 00484723 --->这个 jnz 有可疑！双击此处，让光标来到此处！
:004846AC A1F48D4800                 mov eax, dword ptr [00488DF4]
:004846B1 83E80F                     sub eax, 0000000F
:004846B4 3B0544DE4A00               cmp eax, dword ptr [004ADE44]
:004846BA 7D67                       jge 00484723
:004846BC 6A40                       push 00000040
:004846BE 8D45E8                     lea eax, dword ptr [ebp-18]
--让光标停留在 4846AA 处，然后用 jump to 看它跳到哪里去.....
```

```asm
* Possible StringData Ref from Code Obj ->",请赶快向作者注册(0371-8982414)"
                        |
:004846FB 682C494800                 push 0048492C
:00484700 8D45E4                     lea eax, dword ptr [ebp-1C]
:00484703 BA04000000                 mov edx, 00000004
:00484708 E88FF5F7FF                 call 00403C9C
:0048470D 8B45E4                     mov eax, dword ptr [ebp-1C]
:00484710 E88BF6F7FF                 call 00403DA0
:00484715 50                         push eax
:00484716 8BC3                       mov eax, ebx
:00484718 E82719FAFF                 call 00426044
:0048471D 50                         push eax

* Reference To: user32.MessageBoxA, Ord:0000h
                        |
:0048471E E8DD1BF8FF                 Call 00406300 --->call 出那个讨厌对话框的地方！
```

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004846AA(C), :004846BA(C)
|
:00484723 B201                       mov dl, 01   --->4846AA 的跳跃就是来到这里的
                                                     到了这里，也就是跳过了那个窗口！
:00484725 8B8314020000               mov eax, dword ptr [ebx+00000214]
:0048472B E85408FDFF                 call 00454F84
```

/*那么 `4846A3` 的那个 `call` 不是判断时间就是判断注册码的，我们进那个 `call` 看看*/

`4846A3` 那个 `call` 的内容如下：

```asm
* Referenced by a CALL at Addresses:
|:0048305F , :004846A3
|
:00469C94 53                         push ebx
:00469C95 A13C8F4800                 mov eax, dword ptr [00488F3C]
:00469C9A 8B00                       mov eax, dword ptr [eax]
:00469C9C E827A5FFFF                 call 004641C8 --->计算注册码的 call!写注册机就要仔细看这个 call
                                                     它的算法不是很复杂!
:00469CA1 8BD8                       mov ebx, eax

* Possible StringData Ref from Code Obj ->"RegNum" -->看见了吗，这个 call 就是判断注册码的！
                                                     算出注册码也不能了吧！
                |:00469CA3 B8C49C4600                 mov eax, 00469CC4
:00469CA8 E8B3A6FFFF                 call 00464360
:00469CAD 3BD8                       cmp ebx, eax -->哈哈，我们可以用 trw2000 来下断点!下断点 469C94
                                                     很快来到这里,看 ebx 的值,把它转成十进制就是注册码
:00469CAF 7507                       jne 00469CB8 -->这里不跳就 OK!改破解版,就改这里好了把 7507 改成 9090
:00469CB1 B801000000                 mov eax, 00000001
:00469CB6 5B                         pop ebx
:00469CB7 C3                         ret
```

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00469CAF(C)
|
:00469CB8 33C0                       xor eax, eax -->不能来到这里,让他把 eax 清 0 就完了!
:00469CBA 5B                         pop ebx
:00469CBB C3                         ret
```

## 制作注册机

下面我们用 `Crackcode2000` 来制作注册机，因为今次的注册码不是直接用 ASCII 码形式放在内存中的，而是用数值的形式放在寄存器处的，所以这时就要用模式 1 了。下面是 `Crackcode.ini` 的信息：

```ini
[Options]
CommandLine=scon.exe
Mode=1
First_Break_Address=469CAD
First_Break_Address_Code=3B
First_Break_Address_Code_Lenth=2
Save_Code_Address=EBX
```

我的序列号是 `183894`，计算出的注册码是 `30806`！`crackcode2000` 注册机就写好了！很简单吧！哈哈，这要感谢我们的 Rufeng 兄，写了一个如此棒的程序！而且体积很小！

## 破解版制作

1.  用 `UnAspack` 解压 `scon.exe` 文件
2.  查找：
    ```
    E8 B3 A6 FF FF 3B D8 75 07
    ```
    替换：
    ```
    -- -- -- -- -- -- -- 90 90
    ```

今天就到这里，休息.......休息！

wind
China Cracking Group
2000.8.30
4dd466d0e1e0cc123afbe504da2462ca
