
# `roundclock 3.00 beta4`

> 原文：[`www.pediy.com/kssd/pediy03/forum435.htm`](https://www.pediy.com/kssd/pediy03/forum435.htm)

急!急!!急!!!请问如何破 `roundclock 3.00 beta4` ?

下载网址：`http://www.fkware.com` 该软件非常不错：

`RoundClock` is not just another clock for Windows, it is a real round clock with analog display. `RoundClock` is highly configurable and has extended alarm capabilities.

随便填上注册名与注册码，该软件把注册信息保存在：

`HKEY_CURRENT_USER\Software\fkWare\Registrations\RoundClock`

`UserName="******"`与 `RegCode="@@@@@@@"`中。

用 `regqueryvalueexa` 设断后可在内存查到 `RegCode` 的值，`bpm` 该值后，马上迷失在茫茫汇编之中，无法找到关键对比处。用 `w32dsm893` 也无法找到关键的注册出错对话“Sorry,but the name or the code you've......”下中断 `bpx messageboxa`, `bpx getwindowtexta`,`bpx dialogboxparama`,`bpx getdlgitemtexta`,`bpx getmessagea` 等等等等都不能拦截该窗，`lockmytask`,`BMSG` 等收窗也无法跟踪下去，但用 `exescope6.0` 能查得出该对话在 `Sting--1882--30102` 处，不知是如何做出来的？

最后找来找去，发现在执行 `roundclock` 前先下中断 `bpx findatoma` (有谁知道 `findatoma` 是什么意思?)

再执行 `roundclock` 中断于该断点:

```
* Reference To: kernel32.FindAtomA, Ord:0000h
                              |
:004233E1 E8A212FEFF           Call 00404688
:004233E6 6685C0               test ax, ax     ***(ax=0 改为 1 即可)
:004233E9 0F9545F7             setne byte ptr [ebp-09]
:004233ED 8B45FC               mov eax, dword ptr [ebp-04]
:004233F0 C6407200             mov [eax+72], 00
:004233F4 807DF700             cmp byte ptr [ebp-09], 00
:004233F8 7516                 jne 00423410
```

修改后，运行结束在屏幕上出现一个时钟的图标，对图标点鼠标右键，马上中断：

```
* Reference To: kernel32.FindAtomA, Ord:0000h
                              |
:004224BF E8C421FEFF           Call 00404688
:004224C4 6685C0               test ax, ax     ***(ax=0 改为 1 即可)
:004224C7 0F9545E7             setne byte ptr [ebp-19]
:004224CB 807DFB00             cmp byte ptr [ebp-05], 00
:004224CF 744E                 je 0042251F
```

依上改过以后，移动鼠标到 `help` 处发现没了 `register now...`与 `Enter registration code...`两项，

点击 `About RoundClock...`项，又中断：

```
* Reference To: kernel32.FindAtomA, Ord:0000h
                              |
:0040EEE2 E8A157FFFF           Call 00404688
:0040EEE7 6685C0               test ax, ax     ***(ax=0 改为 1 即可)
:0040EEEA 0F95C0               setne al
:0040EEED 8B55FC               mov edx, dword ptr [ebp-04]
:0040EEF0 888295000000         mov byte ptr [edx+00000095], al
```

改过后，按 `F5` 键运行，注册成功了！

按理说直接改程序很容易就能注册了，但改过后我的机子蓝屏！真是气人！

我用的是 `P100` `40M` `EDO` 用 `TRW2K122` 版(注册版)老不稳定，`CTRL+N` 后硬盘狂读几分钟才能进入 `TRW`，有时还会 `SJ`，但有 `TRW` 驻留内存后键盘就被锁死了，不能再输入 `NAME OR CODE`,不知是什么回事？还有为何 `TRW2K` 还没有新版本？`LTT` 去干什么去了？

最后给大家一个正确的注册码以便跟踪:

```
UserName=Licensed User
RegCode=7307C46F982C27E9EF03
```

`This little program has quite a bit of anti-cracker tricks in it.`

1) It will halt your computer if `SoftICE` is running. `SoftICE` is detected using `CreateFileA` on `SICE` and `NTICE`. This is a old (standard) method of detecting `SoftICE` and is easily defeated by watching that `API` call. Furthermore, to prevent simple hex edits of these strings, the author encrypted them.

2) It will crash your computer if it detects that it has been patched.

3) The registration checking is very lengthy and difficult to pin down. This took the most time, after I discovered the other 2 problems.

Protection: 5 of 10/Reg. Code, Anti-ICE, Anti-Patch/Registry

有谁能找到自己心目中的注册码？

正如你先前所说的，如果 `FindAtomA( )`返回的值不为 0，则注册成功。

```
:0040F06D 80B89500000000        cmp byte ptr [eax+00000095], 00    //FindAtomA 的返回值
:0040F074 744F                  je 0040F0C5
```

```
* Possible Reference to String Resource ID=30111: "This copy of $PROGNAME
is licensed to:"
                              |
:0040F076 B99F750000            mov ecx, 0000759F
.........................................................
```

```
* Possible Reference to Dialog: DialogID_7918, CONTROL_ID:791E, "Thank you for your registration!"
                              |
:0040F0AF C745BC1E790000        mov [ebp-44], 0000791E
:0040F0B6 8D55BC                lea edx, dword ptr [ebp-44]
:0040F0B9 33C9                  xor ecx, ecx
:0040F0BB 8B45FC                mov eax, dword ptr [ebp-04]
:0040F0BE E8DD9EFFFF            call 00408FA0
:0040F0C3 EB56                  jmp 0040F11B
```

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F074(C)
|
* Possible Reference to String Resource ID=30112: "This is an UNREGISTERED copy of $PROGNAME!"
                              |
:0040F0C5 B9A0750000            mov ecx, 000075A0
```

看一下传给 `FindAtomA( )`的参数，发现它是在试图找一个值为“sdf”的 atom。这说明如果注册码正确的话，它会在原子表中加入一个值为“sdf”的原子，以此作为标志。后面将会看到这一点。

### 1、启动时将 `RegCode` 加入到原子表中：

```
:004143B7 803D3C69420000        cmp byte ptr [0042693C], 00
:004143BE 7410                  je 004143D0
:004143C0 683C694200            push 0042693C
* Reference To: kernel32.AddAtomA, Ord:0000h
                              |
:004143C5 E87E02FFFF            Call 00404648
:004143CA 66A35E694200          mov word ptr [0042695E], ax
```

### 2、启动时将 `UserName` 加入到原子表中：

```
:0042402D 83787400             cmp dword ptr [eax+74], 00000000
:00424031 7428                  je 0042405B
:00424033 8B45F4                mov eax, dword ptr [ebp-0C]
:00424036 8B4074                mov eax, dword ptr [eax+74]
:00424039 50                   push eax
* Reference To: kernel32.AddAtomA, Ord:0000h
                              |
:0042403A E80906FEFF            Call 00404648
:0042403F 8B1570564200          mov edx, dword ptr [00425670]
:00424045 668902                mov word ptr [edx], ax
```

### 3、从原子表中取出 `UserName`：

```
:004228F3 A170564200            mov eax, dword ptr [00425670]
:004228F8 668B00                mov ax, word ptr [eax]
:004228FB 50                   push eax
* Reference To: kernel32.GetAtomNameA, Ord:0000h
                              |
:004228FC E89F1DFEFF            Call 004046A0
```

### 4、利用从原子表中取出的 `UserName` 来计算注册码。把下面的代码拷贝下来稍加修改就可写个注册机。久违了的 `TSecHash` 算法又在这里露面了 :)

```
* Referenced by a CALL at Address:
|:00422912
|
:00407FAC 55                    push ebp
:00407FAD 8BEC                  mov ebp, esp
:00407FAF 81C418FFFFFF          add esp, FFFFFF18
:00407FB5 894DF4                mov dword ptr [ebp-0C], ecx
:00407FB8 8955F8                mov dword ptr [ebp-08], edx
:00407FBB 8945FC                mov dword ptr [ebp-04], eax
:00407FBE BAD4804000            mov edx, 004080D4
:00407FC3 8D856BFFFFFF          lea eax, dword ptr [ebp+FFFFFF6B]
:00407FC9 E812CEFFFF            call 00404DE0
:00407FCE 8D856BFFFFFF          lea eax, dword ptr [ebp+FFFFFF6B]
:00407FD4 8B55FC                mov edx, dword ptr [ebp-04]
:00407FD7 E860CEFFFF            call 00404E3C
:00407FDC BAD8804000            mov edx, 004080D8
:00407FE1 8D856BFFFFFF          lea eax, dword ptr [ebp+FFFFFF6B]
:00407FE7 E850CEFFFF            call 00404E3C
:00407FEC 8D856BFFFFFF          lea eax, dword ptr [ebp+FFFFFF6B]
:00407FF2 E821D2FFFF            call 00405218
:00407FF7 B201                  mov dl, 01
* Possible StringData Ref from Code Obj ->"TSecHash 岪"
                              |
:00407FF9 A134784000            mov eax, dword ptr [00407834]
:00407FFE E87DA8FFFF            call 00402880
:00408003 8945F0                mov dword ptr [ebp-10], eax
:00408006 8D8518FFFFFF          lea eax, dword ptr [ebp+FFFFFF18]
:0040800C 50                   push eax
```



```
:0040800D 8B45F8             mov eax, dword ptr [ebp-08]
:00408010 E837CFFFFF         call 00404F4C
:00408015 8BC8               mov ecx, eax
:00408017 8B55F8             mov edx, dword ptr [ebp-08]
:0040801A 8B45F0             mov eax, dword ptr [ebp-10]
:0040801D E86AFCFFFF         call 00407C8C
:00408022 8D8D43FFFFFF       lea ecx, dword ptr [ebp+FFFFFF43]
:00408028 8D9518FFFFFF       lea edx, dword ptr [ebp+FFFFFF18]
:0040802E 8B45F0             mov eax, dword ptr [ebp-10]
:00408031 E8CEFEFFFF         call 00407F04
:00408036 8B45F0             mov eax, dword ptr [ebp-10]
:00408039 E8AAA8FFFF         call 004028E8
:0040803E B201               mov dl, 01

* Possible StringData Ref from Code Obj ->"TSecHash 岪"
```

**5、从原子表中取出我们输入的假 RegCode：**

```
:00422778 8D4597             lea eax, dword ptr [ebp-69]
:0042277B 50                 push eax
:0042277C A1A4564200         mov eax, dword ptr [004256A4]
:00422781 668B00             mov ax, word ptr [eax]
:00422784 50                 push eax

* Reference To: kernel32.GetAtomNameA, Ord:0000h

:00422785 E8161FFEFF         Call 004046A0
```

**6、将假 RegCode 每两个字符转换成一个字节：**

```
* Referenced by a CALL at Address:
|:004227DF
|

:004080DC 55                 push ebp
:004080DD 8BEC               mov ebp, esp
:004080DF 83C4EC             add esp, FFFFFFEC
:004080E2 8955F8             mov dword ptr [ebp-08], edx
:004080E5 8945FC             mov dword ptr [ebp-04], eax
:004080E8 33C0               xor eax, eax
:004080EA 8945F0             mov dword ptr [ebp-10], eax
:004080ED C645EF00           mov [ebp-11], 00
:004080F1 33C0               xor eax, eax
:004080F3 8945F4             mov dword ptr [ebp-0C], eax
:004080F6 8B45F8             mov eax, dword ptr [ebp-08]
:004080F9 33C9               xor ecx, ecx
|

:004080FB BA0A000000         mov edx, 0000000A             //10 字节
:00408100 E827A5FFFF         call 0040262C
:00408105 C645ED01           mov [ebp-13], 01
:00408109 EB66               jmp 00408171

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040817C(C)
|

:0040810B 8B45F4             mov eax, dword ptr [ebp-0C]
:0040810E 8B55FC             mov edx, dword ptr [ebp-04]
:00408111 803C022D           cmp byte ptr [edx+eax], 2D
:00408115 7457               je 0040816E
:00408117 8B45F4             mov eax, dword ptr [ebp-0C]
:0040811A 8B55FC             mov edx, dword ptr [ebp-04]
:0040811D 803C0239           cmp byte ptr [edx+eax], 39
:00408121 7710               ja 00408133
:00408123 8B45F4             mov eax, dword ptr [ebp-0C]
:00408126 8B55FC             mov edx, dword ptr [ebp-04]
:00408129 8A0402             mov al, byte ptr [edx+eax]
:0040812C 2C30               sub al, 30
:0040812E 8845EE             mov byte ptr [ebp-12], al
:00408131 EB10               jmp 00408143

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408121(C)
|

:00408133 8B45F4             mov eax, dword ptr [ebp-0C]
:00408136 8B55FC             mov edx, dword ptr [ebp-04]
:00408139 8A0402             mov al, byte ptr [edx+eax]
:0040813C 2C41               sub al, 41
:0040813E 0408               add al, 08
:00408140 8845EE             mov byte ptr [ebp-12], al

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408131(U)
|

:00408143 807DED00           cmp byte ptr [ebp-13], 00
:00408147 740C               je 00408155
:00408149 8A45EE             mov al, byte ptr [ebp-12]
:0040814C 8845EF             mov byte ptr [ebp-11], al
:0040814F C645ED00           mov [ebp-13], 00
:00408153 EB19               jmp 0040816E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408147(C)
|

:00408155 8B45F8             mov eax, dword ptr [ebp-08]
:00408158 8B55F0             mov edx, dword ptr [ebp-10]
:0040815B 8A4DEE             mov cl, byte ptr [ebp-12]
:0040815E C1E104             shl ecx, 04
:00408161 024DEF             add cl, byte ptr [ebp-11]
:00408164 880C10             mov byte ptr [eax+edx], cl      //保存转换结果
:00408167 FF45F0             inc [ebp-10]
:0040816A C645ED01           mov [ebp-13], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00408115(C), :00408153(U)
|

:0040816E FF45F4             inc [ebp-0C]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00408109(U)
|

:00408171 8B45FC             mov eax, dword ptr [ebp-04]
:00408174 E8D3CDFFFF         call 00404F4C
:00408179 3B45F4             cmp eax, dword ptr [ebp-0C]
```



:0040817C 778D  ja 0040810B //循环   
:0040817E 8BE5  mov esp, ebp   
:00408180 5D  pop ebp   
:00408181 C3  ret   

7、比较真假 RegCode：   

:004227E4 33C0  xor eax, eax   
:004227E6 8945F0  mov dword ptr [ebp-10], eax   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0042280F(C)   
|   
:004227E9 8B45F0  mov eax, dword ptr [ebp-10]   
:004227EC 8A4405C9  mov al, byte ptr [ebp+eax-37]   
:004227F0 8B55F0  mov edx, dword ptr [ebp-10]   
:004227F3 8B4DFC  mov ecx, dword ptr [ebp-04]   
:004227F6 3A441168  cmp al, byte ptr [ecx+edx+68] //比较一个字节   
:004227FA 7506  jne 00422802   
:004227FC C645DF01  mov [ebp-21], 01 //good guy    
:00422800 EB06  jmp 00422808   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:004227FA(C)   
|   
:00422802 C645DF00  mov [ebp-21], 00 //bad guy    
:00422806 EB09  jmp 00422811   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00422800(U)   
|   
:00422808 FF45F0  inc [ebp-10]   
:0042280B 837DF00A  cmp dword ptr [ebp-10], 0000000A //循环比较 10 个字节   
:0042280F 75D8  jne 004227E9   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00422806(U)   
|   
:00422811 8B45FC  mov eax, dword ptr [ebp-04]   
:00422814 C6407300  mov [eax+73], 00   
:00422818 8B45FC  mov eax, dword ptr [ebp-04]   
:0042281B C6407201  mov [eax+72], 01   
:0042281F 807DDF00  cmp byte ptr [ebp-21], 00 //bag guy?   
:00422823 740A  je 0042282F   

* Possible StringData Ref from Code Obj ->"sdf"   
                  |   
:00422825 686C284200  push 0042286C //注册与否的标志！   

* Reference To: kernel32.AddAtomA, Ord:0000h   
                  |   
:0042282A E8191EFEFF  Call 00404648   

可见关键的断点是 `GetAtomNameA()`。   

`BF 4dd466d0e1e0cc123afbe504da2462ca`
