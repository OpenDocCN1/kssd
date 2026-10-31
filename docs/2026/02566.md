
# `PUZZLER1.20` 破解过程（4千字）

`PUZZLER1.20`，这个东西，朋友借我的光盘里找到的。我比较喜欢玩魔方，但我的那个魔方丢了，现在可以在电脑上玩，呵呵。

安装完后，发现这个东西确实不错，不仅有传统的魔方，还有好多变种，不过一点，它就跳出来说，要注册才能用。

工具：`TRW`，`W32DASM`

点`ABOUT`，点`REGIST`，跳出一个窗口，叫你输用户名，输入“波导终结者”，点`OK`，再跳出来一个窗口，叫你输注册码，注册码不正确没有提示。用户名和注册码分两个窗口输，这种形式，我还是第一次见到。不要紧，我们用`W32DASM`反汇译`PUZZLER.EXE`，查找`REGIST`，第一次，找到弹出来说不注册不能用的窗口，我们要的不是暴破，再找一次，找到下面。

```
:0040D39E 68A06B4100              push 00416BA0
:0040D3A3 8BCD
  mov ecx, ebp
\* Reference To: UFC.??4CUString@@QAEAAV0@PBD@Z, Ord:0044h
            |
:0041902A FF1538C34100            Call dword ptr [0041C338]
:00419030 8B5C2448                mov ebx, dword ptr [esp+48]
:00419034 85DB
  test ebx, ebx
:00419036 741C
  je 00419054
:00419038 8BCE
  mov ecx, esi
\* Reference To: UFC.?AddUpASCII@CUString@@QAEKXZ, Ord:00D3h
            |
:0041903A FF154CC34100            Call dword ptr [0041C34C]
:00419040 8BC8
  mov ecx, eax
:00419042 C1E108
  shl ecx, 08
:00419045 03C8
  add ecx, eax
:00419047 8D14C8
  lea edx, dword ptr [eax+8\*ecx]
:0041904A 8D0450
  lea eax, dword ptr [eax+2\*edx]
:0041904D 8D0440
  lea eax, dword ptr [eax+2\*eax]
:00419050 3BD8
  cmp ebx, eax
:00419052 740D
  je 00419061
```

④这里又是最近的一个跳转，看看`cmp ebx, eax`，很可能就是比较。在这里下断，`d eax`，看见数值`FF950188`，再看`ebx`，是`3B1AE`。把用户名和假注册码都换掉，再看`eax`和`ebx`的数值，都发生了变化！把用户名和注册码都换成`1`，看见`eax`为一个大数（我没记下来，歹势），看见`ebx`为`1`。再把用户名和注册码换为`2`，看见`eax`变化了，`ebx`为`2`。

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419036(C)
|
:00419054 6828474200              push 00424728
:00419059 8BCE
  mov ecx, esi
\* Reference To: UFC.??4CUString@@QAEAAV0@PAD@Z, Ord:0043h
            |
:0041905B FF156CC34100            Call dword ptr [0041C36C]
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00419052(C)
|
:00419061 8BCE
  mov ecx, esi
\* Reference To: UFC.?GetLength@CUString@@QBEHXZ, Ord:012Ch
            |
:00419063 FF1584C34100            Call dword ptr [0041C384]
:00419069 85C0
  test eax, eax
:0041906B 7419
  je 00419086
```

③于是在这个地方下断点，走到这里，看看寄存器内容，有一些数值，可能是注册码和用户名的变形，记下来。`r fl z`，之后，看看变化，软件主界面已经变成了“This copy of Puzzler is registered to:”但是，我们的用户名呢？再进入游戏，还是不能玩。把输的假用户名和注册码都换掉，看看这里寄存器的数值，还都一样！由此可见，这个跳转，只是把用户名赋给这个字串后面，但是为什么看不见用户名呢？很可能是注册码不正确，用户名根本没带到这个地方，于是，我们再向上看。

```
:0041906D 8B0E
  mov ecx, dword ptr [esi]
:0041906F 81C78C010000            add edi, 0000018C
:00419075 51
  push ecx
\* Possible StringData Ref from Data Obj ->"This copy of Puzzler is registered
"
                  ->"to:
%s"
```

②这一行意思就是说，这个软件已经注册给了XXX，由此往上看，上面的那个跳转很可能就是比较注册码。

```
            |
:00419076 68303D4200              push 00423D30
:0041907B 57
  push edi
\* Reference To: MFC42.Ordinal:0B02, Ord:0B02h
            |
:0041907C E867030000              Call 004193E8
:00419081 83C40C
  add esp, 0000000C
:00419084 EB14
  jmp 0041909A
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041906B(C)
|
:00419086 81C78C010000            add edi, 0000018C
\* Possible StringData Ref from Data Obj ->"Thank you for trying Puzzler!
Press "
                  ->"F1 for help
at any time"
```

①如果没注册的话是显示这行内容，而注册后则显示处内容。

```
            |
:0041908C 68F03C4200              push 00423CF0
:00419091 57
  push edi
```

⑤由此可见，程序在`0041903A Call dword ptr [0041C34C]`一行，把注册名通过一系列运算，得到一个值，放到`00419050 cmp ebx, eax`的`eax`里，`ebx`为注册的十六进制数值，进行比较。我们在`00419050 cmp ebx`一行，`d eax`，把数值记下来，换成十进制数，当做注册码输入，注册成功！

波导终结者[BCG][CNCG]
