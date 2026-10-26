
# 有关判断各种软件加壳标志的方法------在PECompact中找到的。

(6千字)

有关判断各种软件加壳标志的方法------在PECompact中找到的。

作者：该职位空缺，有意者可与本人联系。限额一名，MM优先。

写作日期：2000-08-06

## 前言

在掌握了对PECompact  v1.40b1的脱壳后，一直没太注意这个软件。今天为了寻找一下该压缩加壳软件的加壳标志，就又重新找出来了。跟踪一遍才发现，程序编的简洁高效。（看来作者很了不起哦，有可能是用Win32ASM编写的，当然跟踪起来很方便了，不象跟踪有些软件，就象到垃圾场里去捡垃圾。）不过有意思的是该软件对其他压缩软件已经加壳过的程序不再加壳，哈哈，这样剩了我不少时间，不用一一去分析其他加壳软件的加壳标志了。从这里找就是了。（注：当然这种方法找出的标志不一定有用。因为加壳软件其自身的判断标准可能与此不同。这个软件说不定也是找一种偷懒的方法来判断的。和我一样

--- 哈哈，两个懒虫，一个不如一个。:-) ）

以下是我找到的程序分析加壳标志的代码部分。我加了点注释。（也可能与实际有出入，懒得去验证了）

### 第一部分：主程序代码

```
:0040385D 66813E5045              cmp word

ptr [esi], 4550        <--这里当然是判断是否是可执行文件的PE标志了

:00403862 0F85FF000000            jne 00403967

              <--不同，那还有戏唱...

:00403868 833DC9A3400001          cmp dword ptr [0040A3C9],

00000001

:0040386F 740D

  je 0040387E

:00403871 817E0C5045434F          cmp dword ptr [esi+0C],

4F434550    <--判断是否是 'PECO'，这是PECompact加壳的标志

:00403878 0F841C010000            je 0040399A

              <--相等，那还用说，自然去显示你的程序被加壳了。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040386F(C)

|

:0040387E 56

  push esi

:0040387F E8E42F0000              call 00406868

              <--该调用测试是否被NeoLite加壳了

:00403884 0F820E020000            jb 00403A98

              <--小于0，说明加壳了

:0040388A 56

  push esi

:0040388B E8B72F0000              call 00406847

              <--该调用测试是否被BitArts加壳了

:00403890 0F8228020000            jb 00403ABE

              <--小于0，说明加壳了

:00403896 56

  push esi

:00403897 E8762F0000              call 00406812

              <--该调用测试.....自已看E文吧....The

selected file appears to have already a symbiote of some type installed, it

may or may not function after compression.

:0040389C 0F8242020000            jb 00403AE4

              <--小于0，.....看懂了上面的话，自己来说吧！干嘛！

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00403AE8(C), :00403B03(C)

|

:004038A2 0FB74614                movzx

eax, word ptr [esi+14]

:004038A6 83C018

add eax, 00000018

:004038A9 03C6

  add eax, esi

  <--EAX定位到Section的Name指针处

:004038AB 81382E736872            cmp dword ptr

[eax], 7268732E        <--判断是否是 '.shr'，测试是否被Shrinker加壳了

:004038B1 0F8495010000            je 00403A4C

              <--相等，以后我懒得说了

:004038B7 813855505830            cmp dword ptr

[eax], 30585055        <--判断是否是 'UPX0'，测试是否被UPX加壳了

:004038BD 0F84AF010000            je 00403A72

:004038C3 56

  push esi

:004038C4 E8E02E0000              call 004067A9

:004038C9 81382E575750            cmp dword ptr

[eax], 5057572E        <--判断是否是 '.WWP'，测试是否被WWPack32加壳了

:004038CF 0F8417010000            je 004039EC

:004038D5 83E828

sub eax, 00000028

:004038D8 81382E617370            cmp dword ptr

[eax], 7073612E        <--判断是否是 '.asp'，测试是否被ASPack加壳了

:004038DE 0F8446010000            je 00403A2A

:004038E4 81382E706B6C            cmp dword ptr

[eax], 6C6B702E        <--判断是否是 '.pkl'，测试是否被PKLite32加壳了

:004038EA 0F841B010000            je 00403A0B
```

### 第二部分：子程序代码部分

#### ① 第一个子程序段

```
\* Referenced by a CALL at Address:

|:0040387F

|

:00406868 C8000000                enter

0000, 00

:0040686C 60

  pushad

:0040686D FF7508

push [ebp+08]

:00406870 E834FFFFFF              call 004067A9

              <--该调用返回Sections的Name的指针到EAX

:00406875 81382E6E656F            cmp dword ptr

[eax], 6F656E2E        <--判断是否是 '.neo'，这是加壳标志

:0040687B 7406

  je 00406883

  <--相等，加壳了

:0040687D 61

  popad

:0040687E F8

  clc

:0040687F C9

  leave

:00406880 C20400

ret 0004
```

#### ② 第二个子程序段

```
\* Referenced by a CALL at Address:

|:0040388B

|

:00406847 C8000000                enter

0000, 00

:0040684B 60

  pushad

:0040684C FF7508

push [ebp+08]

:0040684F E855FFFFFF              call 004067A9

              <--该调用返回Sections的Name的指针到EAX

:00406854 813842697441            cmp dword ptr

[eax], 41746942        <--判断是否是 'BitA'，这是BitArts加壳标志

:0040685A 7406

  je 00406862

:0040685C 61

  popad

:0040685D F8

  clc

:0040685E C9

  leave

:0040685F C20400

ret 0004
```

#### ③ 跳转后的显示部分，这还用去解释吗？

```
:00403A98 837D1001                cmp

dword ptr [ebp+10], 00000001

:00403A9C 0F8515FFFFFF            jne 004039B7

:0040399A 837D1001                cmp

dword ptr [ebp+10], 00000001

:0040399E 7517

  jne 004039B7

:004039A0 6A10

  push 00000010

\* Possible StringData Ref from Code Obj ->"PECompact v1.40b1, "

            |

:004039A2 6808B24000              push 0040B208

\* Possible StringData Ref from Code Obj ->"File appears to already be compressed."

            |

:004039A7 68BBB24000              push 0040B2BB

:004039AC FF351CB94000            push dword ptr

[0040B91C]

:004039B2 E8CD090000              call 00404384
```

## 后记

我们已经知道了PECompact判断的方法，修改跳转就很简单了。我用`Notepad.exe`加壳后测试了一下，加一道壳，有三个Sections，加两道壳，又加了二个Section，以后再加壳，就没加Section了。yi,是不是PECompact开始只做表面功夫了，我一下加了八道壳，开始跟踪，结果在内存中跳来跳去的，我头也跟晕了....。又加了几十道壳，`Notepad.exe`开始变得越来越肥大，但还可以运行。以后有时间再分析分析其他加壳软件的加壳标志，然后来个交叉加壳。每个各加三遍..........然后再............。老天，以后你不会用这种方法加壳你的软件来对付我吧........@#$@#$@

昏倒...@#$@#$@#$。

补充：从跟踪ASPack v2.1和ASProtect v1.0的情况来看，好象PECompact检测加壳标志的方法与其自身判断方法不同哦。
