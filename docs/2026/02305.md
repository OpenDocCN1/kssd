
# 在转贴一个，谁的？JOJO的！！！ (6千字)

## 漫谈CrackMe之三

### 试谈TEXme v1.0的破解

大家好，又见面了。不知大家看了上篇文章是否对KEY文件认证有些心得。今天我们看看另一种风格的CRACKME软件。这次我们分析的是TEX99的TEXme v1.0版本。等级2级。虽然2级，但加密去着实不弱。软件可以到下面地址下载。

此CRCAKME属于循环运算，在程序被展开后就开始循环计算`NAME`和`SERIAL`，等你添入正确的`NAME`和`SERIAL`显示"YES! You found your serial!!"循环停止，否则显示"Your serial is not valid."。这可以从下面的分析中看到。

我们先来看看破解的思路，先用W32反编译程序。查找有关的字串，这里就不再累述过程。然后向上找可疑跳转，确定断点后用ICE动态分析，最终找到正确的`SERIAL`。

首先用W32载入`texme.exe`，老步骤，打开“REFS”查找字符串"YES! You found your serial!!",找到了双击。看到以下程序：

```asm
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004012B6(C)  ；这里有情况，说明这里是由004012B6处跳转来的。
|
\* Possible StringData Ref from Data Obj ->"YES! You found your serial!!"
            |
:004012CE 687F304000              push 0040307F
:004012D3 FF357C314000            push dword ptr
[0040317C]
\* Reference To: USER32.SetWindowTextA, Ord:0221h
            |
:004012D9 E850020000              Call 0040152E
:004012DE 33C0
  xor eax, eax
:004012E0 C9
  leave
:004012E1 C21000
ret 0010
```

查找`004012B6`，看到以下程序：

```asm
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004012A6(C)  ；这里同上，说明这里是由004012A6处跳转来的。
|
:004012AE A267314000              mov byte
ptr [00403167], al
:004012B3 83F810
cmp eax, 00000010  ；在这里。
:004012B6 7416
  je 004012CE ；不跳就失败了。
\* Possible StringData Ref from Data Obj ->"Your serial is not valid."
            |
:004012B8 6865304000              push 00403065
:004012BD FF357C314000            push dword ptr
[0040317C]
\* Reference To: USER32.SetWindowTextA, Ord:0221h
            |
:004012C3 E866020000              Call 0040152E
:004012C8 33C0
  xor eax, eax
:004012CA C9
  leave
:004012CB C21000
ret 0010
```

我们修改`004012B6`处`je`成`jnz`。程序被暴力破解。可是CRACKME是不准打补丁的。我们只好继续分析了。查找`004012A6`，找到，看到以下程序：

```asm
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401274(C)      ；我们先不向上看了。
|
:0040128D 3D13010000              cmp eax,
00000113  ；这里断一下试一试
:00401292 7550
  jne 004012E4
:00401294 E8BA010000              call 00401453
    ；Call里是什么呢？
:00401299 0FBE0566314000          movsx eax, byte ptr
[00403166]
:004012A0 3A0567314000            cmp al, byte
ptr [00403167]
:004012A6 7506
  jne 004012AE
:004012A8 33C0
  xor eax, eax
:004012AA C9
  leave
:004012AB C21000
ret 0010
```

用ICE中的Symbol Loader载入程序，运行，中断在程序入口处。`BPX 40128D`，然后`F5`，程序被中断，`F5`又中断，程序已经进入循环。证明我们还是没有找错。`BD \*`，`F5`。添入`NAME`和`SERIAL`，`Ctrl+D`，`BE \*`，`F5`，程序被中断。待程序运行到`00401294`处`F8`进入，看到以下程序：

```asm
:00401453 55
  push ebp
:00401454 8BEC
  mov ebp, esp
:00401456 83C4FC
add esp, -4
:00401459 8925A0314000            mov dword ptr
[004031A0], esp
:0040145F 8D2552314000            lea esp, dword
ptr [00403152]
:00401465 0FBE0566314000          movsx eax, byte ptr
[00403166]
:0040146C 03E0
  add esp, eax
:0040146E C3
  ret  ；关键是RET后到哪里。
```

RET后程序到这里：

```asm
:00401361 8D3D8C314000            lea edi, dword
ptr [0040318C]
:00401367 0FBE0568314000          movsx eax, byte ptr
[00403168]
:0040136E 03F8
  add edi, eax
:00401370 FE0568314000            inc byte ptr
[00403168]
:00401376 A188314000              mov eax,
dword ptr [00403188]
:0040137B 8B25A0314000            mov esp, dword
ptr [004031A0]
:00401381 40
  inc eax ；加1
:00401382 FF0588314000            inc dword ptr
[00403188] ；迷惑项
:00401388 3307
  xor eax, dword ptr [edi]
:0040138A A388314000              mov dword
ptr [00403188], eax
:0040138F 803D6831400010          cmp byte ptr [00403168],
10
:00401396 7507
  jne 0040139F
:00401398 80056631400004          add byte ptr [00403166],
04
:0040139F C9
  leave
:004013A0 C3
  ret  ；出去还会回来的。呵呵。
```

这段是程序的关键所在，如果你看得懂，程序就破解一半了。基本是这样的，`D 40318C`你可以看到你输入的`NAME`，过`401376`程序段后 `? EAX`后可以看到你输入的`SERIAL`，而`403168`则是一个计数器，它的值到16后就跳转出循环。程序在循环中这样取`NAME`的字符ASCII码，不足16位以ASCII码"00"补齐。比如我的`NAME`：`jojo[CCG]`，先取"j"的ASCII值与你的`SERIAL`值加1后相异或，也就是`00401382`程序段。然后把结果送到`403188`中再把"o"的ASCII值与上次相异或的结果加1后再相异或。结果再送到`403188`中，一直循环16次，然后出循环。

出循环会到哪里呢？中断程序段`0040128D`，按`F5`，观察`EAX`的值，会有2次"111"，2次"138"后，出现"113"，这时单步走没有多久可以看到以下程序：

```asm
:0040149C A188314000              mov eax,
dword ptr [00403188]
:004014A1 0578241109              add eax,
09112478
:004014A6 85C0
  test eax, eax
:004014A8 7509
  jne 004014B3  ；如果不跳就万事大吉了
```

我们来算个算术题：`0-09112478=F6EEDB88`，就是说我们刚才异或半天的值等于`F6EEDB88`就不会跳转，这样就使以下程序：

```asm
:004012AE A267314000              mov byte
ptr [00403167], al
:004012B3 83F810
cmp eax, 00000010  ；在这里。
:004012B6 7416
  je 004012CE ；不跳就失败了。
```

成功跳转，注册成功！

这样我们依此反推就可以得到正确的`SERIAL`。如果大家嫌太麻烦就自己做一个注册机吧。

好了，今天就到这里吧！休息、休息一下！！！对，差一点忘了，我的

`NAME:jojo[CCG]`  `SERIAL:3635081415`

谨以此文献给我们可爱的CCG，希望它蒸蒸日上！

jojo [CCG]

2000.5.9

```text
#################################
#  欢迎光临【密界寻综】JOJO的家    #
#
        #
#      jojocrack@371.net      #
#    http://jojohome.126.com    #
# http://grwy.sqinfo.ha.cn/jojo #
#################################
```
