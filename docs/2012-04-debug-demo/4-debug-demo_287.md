
# WavesurroundDEMO

> 原文：[`www.pediy.com/kssd/pediy03/forum885.htm`](https://www.pediy.com/kssd/pediy03/forum885.htm)

`WavesurroundDEMO` 程序我一直都没怎么看，抱歉。——你说的过期判断不知怎么回事，`DSP_WA3D.DLL` 里面找不到 `getsytstemtime` 或者 `getlocaltime` 等类似的时间函数，我也搞不清它是怎样判断时间过期的。倒是那个 `NAG` 窗口很吸引人，本来还不知怎么入手的，后来找了个 `resource hacker` 可以修改资源的那种工具，把 `winamp` 的 `plug-in` 目录下的那个 `dsp_wa3d.dll` 的 `NAG` 对话框资源给删除就万事大吉了，好笑不好笑？

不过这不是正儿八经的方法。你寄给我的那个 `patch` 中对 `DSSP_WA3D.DLL` 进行了两处改动，一个是去掉 `DEMO` 的 `nag`，一个是解除时间过期。改的地方分别是：

去 `NAG` 窗口（其实就是跳过 `MESSAGEBOXA`）：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10002041(C)
|
:100020AA 6A05                  push 00000005
:100020AC 8B15C8940110          mov edx, dword ptr [100194C8]
:100020B2 52                    push edx

* Reference To: USER32.ShowWindow, Ord:022Dh
                                |
:100020B3 FF1528D40110          Call dword ptr [1001D428]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:100020A8(U)
|
:100020B9 C705C094011001000000 mov dword ptr [100194C0], 00000001
:100020C3 6A00                  push 00000000

//这里把 6A00 改成 EB18，也就是      jmp 100020DD

:100020C5 6835100010            push 10001035
:100020CA 6A00                  push 00000000

* Possible StringData Ref from Data Obj ->"DLGNag"
                                |
:100020CC 6898530110            push 10015398
:100020D1 A1CC940110            mov eax, dword ptr [100194CC]
:100020D6 50                    push eax

* Reference To: USER32.DialogBoxParamA, Ord:008Eh
                                |
:100020D7 FF1510D40110          Call dword ptr [1001D410]
:100020DD 33C0                  xor eax, eax              //到这里。

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10001B68(U)
|
:100020DF 8BE5                  mov esp, ebp
:100020E1 5D                    pop ebp
:100020E2 C3                    ret
```

时间过期：

```
* Reference To: KERNEL32.SetCurrentDirectoryA, Ord:020Ch
                                |
:10001C67 FF1574D30110          Call dword ptr [1001D374]
:10001C6D E8B2310000            call 10004E24
:10001C72 D905BC940110          fld dword ptr [100194BC]
:10001C78 D81D08400110          fcomp dword ptr [10014008]
:10001C7E DFE0                  fstsw ax
:10001C80 F6C401                test ah, 01
:10001C83 7514                  jne 10001C99              //改成 JMP 了。
:10001C85 6A00                  push 00000000
```

至于这种浮点指令和时间判断有什么关系，就不清楚了。你所说的下文部分只是根据过期标志而作出的界面修改部分，光改它们只会引起界面的变化，内在功能即限制还是不变。

```
####...

Reference To: USER32.DrawTextA, Ord:00AAh
>
>:1000319A FF156CD40110          Call dword ptr [1001D46C]
>
>:100031A0 D905BC940110          fld dword ptr [100194BC]
>
>:100031A6 D81D08400110          fcomp dword ptr [10014008]
>
>:100031AC DFE0                  fstsw ax
>
>:100031AE F6C401                test ah, 01
>
>:100031B1 0F85BB000000          jne 10003272 这里改成 je
>
>:100031B7 68FF000000            push 000000FF
>
>:100031BC 8B55B4                mov edx, dword ptr [ebp-4C]
>
>:100031BF 52                    push edx
>
>* Reference To: USER32.GetDlgItem, Ord:00F3h
>
>                                |
>
>:1000420F FF1554D40110          Call dword ptr [1001D454]
>
>:10004215 A308940110            mov dword ptr [10019408], eax
>
>:1000421A D905BC940110          fld dword ptr [100194BC]
>
>:10004220 D81D08400110          fcomp dword ptr [10014008]
>
>:10004226 DFE0                  fstsw ax
>
>:10004228 F6C401                test ah, 01
>
>:1000422B 7575                  jne 100042A2 这里改成 je
>
>:1000422D 6A00                  push 00000000
>
>:1000422F A1D8930110            mov eax, dword ptr [100193D8]
>
>:10004234 50                    push eax
>
>
>
>* Reference To: USER32.DrawTextA, Ord:00AAh
>
>                                |
>
>:1000319A FF156CD40110          Call dword ptr [1001D46C]
>
>:100031A0 D905BC940110          fld dword ptr [100194BC]
>
>:100031A6 D81D08400110          fcomp dword ptr [10014008]
>
>:100031AC DFE0                  fstsw ax
>
>:100031AE F6C401                test ah, 01
>
>:100031B1 0F85BB000000          jne 10003272 这里改成 je
>
>:100031B7 68FF000000            push 000000FF
>
>:100031BC 8B55B4                mov edx, dword ptr [ebp-4C]
>
>:100031BF 52                    push edx
>
>
>
>
>
>
>好了就这默多。
>

####...
```

当时我觉得任何表面现象表明可以用了，所以觉得他已经被解决了，再后来我也就没看过他，有点东西可能忘了，我觉得应该是解决了，最好的方法是在一台耳机很次的机器上听一听，我再看看，不过我想暂时想不出什么解决他内在功能限制的方法了，对了，我给你的那个好像不太好，我有改动了几个地方，请到 `fcg.longcity.net` 看看，因为在注册表中有记录时间的 `param` 一项，这一项是纪录软件使用时间的。我首先改动了它的这一项，把它改过期了，在时间过了以后可以接着试用，在一个就是它的 `on/off`，我把他改动了，给你的 `patch` 是不行的，或者吧 `param` 中读注册表中的项目以及要调用的 `call`，`nop` 掉，不过这样只能听几个小时，然后 `param` 就会计数到头，提示过期，重启 `winamp` 以后,`param` 清零，好了就这么多了，多谢大哥指教，多谢。

它的过期限制不是什么试用 30 天之类的，而是记录运行总时间并限制它不能超过一小时。所以在什么 `GETSYSTEMTIME`、`GETLOCALTIME`、`GETFILETIME` 等函数上下断点都对其无效（因为它根本就没用到），何况它自身是被 `WINAMP` 定时频繁调用的，`WINAMP` 会把时间信息作为参数传递给它，因此它内部也不用什么 `GETTICKCOUNT` 之类的函数。——这种插件分析起来的难点就是难以找到时间递增代码，时间的比较代码倒是好几处，查出错信息就能找到，只是数量多（其实也不多，才四五个吧），难免挂一漏万，还是跳过时间递增代码比较保险。

倒霉的就是这种判断运算等全是浮点指令，不懂就只有猜，耽搁了不少时间。另外 `WINAMP` 在关闭时会通过这个插件把已经运行的时间秒数写入注册表，而且其值是浮点四字节格式，名称又是什么 `PARAM`，很容易迷惑人。其实了解它的限制原理后，用 `BPM` 断点就能很快找到这段代码：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:10002860(C)
|
:1000286A D905BC940110          fld dword ptr [100194BC]
:10002870 D81D08400110          fcomp dword ptr [10014008]
//[10014008]处是浮点数 3600，也就是一小时。
:10002876 DFE0                  fstsw ax
:10002878 F6C401                test ah, 01
:1000287B 7508                  jne 10002885
//把这句话修改成 JNE 10002897，也就是把 7508 改成 751A。
:1000287D 8B4510                mov eax, dword ptr [ebp+10]
:10002880 E934050000            jmp 10002DB9
```

就是下面这里增加时间计数：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1000287B(C)
|
:10002885 DB4510                fild dword ptr [ebp+10]
:10002888 DA751C                ffidiv dword ptr [ebp+1C]
:1000288B D805BC940110          fadd dword ptr [100194BC]
:10002891 D915BC940110          fst dword ptr [100194BC]     //增加的时间计数写回[100194BC]
:10002897 D81D08400110          fcomp dword ptr [10014008]   //让其直接跳到此处，也就是不增加时间计数了。
:1000289D DFE0                  fstsw ax
:1000289F F6C401                test ah, 01
:100028A2 7515                  jne 100028B9
```

至于 `NAG` 窗口，`DEMO` 字串等的消除，我也懒得去跟了。——`DEMO` 字串消除也就查找字串换成空格就行。

`4dd466d0e1e0cc123afbe504da2462ca`
