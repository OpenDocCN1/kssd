
# 急!急!!急!!!请问如何破roundclock 3.00 beta4 ?

急!急!!急!!!请问如何破roundclock 3.00 beta4 ?

下载网址：http://www.fkware.com  该软件非常不错：

RoundClock is not just another clock for Windows, it is a real round clock with analog display. RoundClock is highly configurable and has extended alarm capabilities.

随便填上注册名与注册码，该软件把注册信息保存在：

`HKEY_CURRENT_USER\Software\fkWare\Registrations\RoundClock`

`UserName`="\*\*\*\*\*\*"与`RegCode`="@@@@@@@"中。

用`regqueryvalueexa`设断后可在内存查到`RegCode`的值，`bpm`该值后，马上迷失在茫茫汇编之中，无法找到关键对比处。用`w32dsm893`也无法找到关键的注册出错对话“Sorry,but the name or the code you've......”下中断`bpx messageboxa`，`bpx getwindowtexta`，`bpx dialogboxparama`，`bpx getdlgitemtexta`，`bpx getmessagea`等等等等都不能拦截该窗，`lockmytask`，`BMSG`等收窗也无法跟踪下去，但用`exescope6.0`能查得出该对话在`Sting--1882--30102`处，不知是如何做出来的？

最后找来找去，发现在执行roundclock前先下中断`bpx findatoma`(有谁知道`findatoma`是什么意思?)再执行roundclock中断于该断点:

```
* Reference To: kernel32.FindAtomA, Ord:0000h
            |
:004233E1 E8A212FEFF              Call 00404688
:004233E6 6685C0
test ax, ax    ***(ax=0改为1即可)
:004233E9 0F9545F7                setne
byte ptr [ebp-09]
:004233ED 8B45FC
mov eax, dword ptr [ebp-04]
:004233F0 C6407200                mov
[eax+72], 00
:004233F4 807DF700                cmp
byte ptr [ebp-09], 00
:004233F8 7516
  jne 00423410
```

修改后，运行结束在屏幕上出现一个时钟的图标，对图标点鼠标右键，马上中断：

```
* Reference To: kernel32.FindAtomA, Ord:0000h
            |
:004224BF E8C421FEFF              Call 00404688
:004224C4 6685C0
test ax, ax    ***(ax=0改为1即可)
:004224C7 0F9545E7                setne
byte ptr [ebp-19]
:004224CB 807DFB00                cmp
byte ptr [ebp-05], 00
:004224CF 744E
  je 0042251F
```

依上改过以后，移动鼠标到help处发现没了register now...与Enter registration code...两项，点击About RoundClock...项，又中断：

```
* Reference To: kernel32.FindAtomA, Ord:0000h
            |
:0040EEE2 E8A157FFFF              Call 00404688
:0040EEE7 6685C0
test ax, ax    ***(ax=0改为1即可)
:0040EEEA 0F95C0
setne al
:0040EEED 8B55FC
mov edx, dword ptr [ebp-04]
:0040EEF0 888295000000            mov byte ptr
[edx+00000095], al
```

改过后，按F5键运行，注册成功了！

按理说直接改程序很容易就能注册了，但改过后我的机子蓝屏！真是气人！

我用的是P100 40M EDO 用TRW2K122版(注册版)老不稳定，CTRL+N后硬盘狂读几分钟才能进入TRW，有时还会SJ，但有TRW驻留内存后键盘就被锁死了，不能再输入NAME OR CODE,不知是什么回事？还有为何TRW2K还没有新版本？LTT去干什么去了？

最后给大家一个正确的注册码以便跟踪:

`UserName=Licensed User`
`RegCode=7307C46F982C27E9EF03`

This little program has quite a bit of anti-cracker tricks in it. 1) It will halt your computer if SoftICE is running. SoftICE is detected using `CreateFileA` on `SICE` and `NTICE`. This is a old (standard) method of detecting SoftICE and is easily defeated by watching that API call. Furthermore, to prevent simple hex edits of these strings, the author encrypted them. 2) It will crash your computer if it detects that it has been patched. 3) The registration checking is very lengthy and difficult to pin down. This took the most time, after I discovered the other 2 problems. Protection: 5 of 10/Reg. Code, Anti-ICE, Anti-Patch/Registry

有谁能找到自己心目中的注册码？
