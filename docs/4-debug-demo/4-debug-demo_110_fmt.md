
# 教菜鸟写注册机

> 原文：[`www.pediy.com/kssd/pediy06/pediy6504.htm`](https://www.pediy.com/kssd/pediy06/pediy6504.htm)

目标程序下载地址：点击此处下载（或鼠标右键另存）

到一些黑客站点上转转，发现那些最受欢迎、点击次数最多的文章都是一些“教菜鸟使用木马”“入侵初级教程”之类文章，而“`Outlook Express HTML` 邮件超长字节 `href` 值缓冲溢出缺陷”“`PHP fopen()` 函数易遭到 `CRLF Injection` 攻击”这种文章却乏人问津，尽管这或许才是真正高手所关心的。

现在看雪论坛上后一种文章逐渐增多，当然是因为大家的水平都提高了，但也给刚入门的菜鸟带来了麻烦，来了之后什么也看不懂，在高手看来不是问题的问题常常困扰新手们很长时间：为什么这里是关键跳转，那边是重要 `CALL`？为什么有的 `CALL` 要跟进，有的 `CALL` 只需带过？我按照高手所写的一步步做下去成功了，可为什么要这样做呢？我想更有不少人好不容易破了一个明码比较的软件，想贴出来又怕为高手所不屑而做罢。

因此我找了一个非常简单的 `CrackMe`（如果你是高手，只静态调试应该就可以搞定了:)），从头到尾仔仔细细地写出了分析过程，在里面加入了些我认为应该注意的地方，希望能对刚入门的朋友有些帮助。我不是什么高手，最多算是破解得熟练一点儿了而已，也许这样才更能了解新手们的难处吧。

这里我用 `W32Dasm` 作为静态反汇编的工具，各位可能有用 `C32Asm` 或者其他的，基本功能其实应该差不多。

先胡乱输入用户名和注册码，点 `"Check the Serial"`，看看错误信息是什么（如果出现“注册码正确”的话就别在这儿看了，赶紧买彩票去:D）好出来了，`"This serial is *NOT* Valid!! Try again... : UNREGISTERED"`。

好了运行 `W32Dasm` 把它反汇编，在串式参考(string reference)中找吧，注意当字串较长时有时不能完全显示，这时只要找前面一段就行了，比如这次就是，找到 `"This serial is *NOT* Valid!! Try "`，就是它了，双击来到引用该字串的地方。（PS：有时错误信息在代码中会不止一次出现，这时只要多次双击就能找到其他的地方。）

下面我要引用代码了（这是在 `W32DASM` 里的形式，如果你用 `OD` 或 `SI` 动态跟踪时形式会略有不同）。不要被这么多代码搞晕了，建议先跳过代码看我的后面的说明，再从代码中对照。



```
代码:
* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:0066, ""
                            |
:00401085 6A66             push 00000066    ;输入用户名的文本框 ID
:00401087 53               push ebx         ;对话框句柄

* Reference To: USER32.GetDlgItem, Ord:0000h
                            |
:00401088 E8159C0000       Call 0040ACA2    ;得到文本框句柄
:0040108D 6A64             push 00000064    ;得到字符串的最大长度
:0040108F 8D9548FFFFFF     lea edx, dword ptr [ebp+FFFFFF48]
:00401095 52               push edx         ;EDX 是存取字符串的地址
:00401096 50               push eax         ;EAX 是上面得到的文本框句柄

* Reference To: USER32.GetWindowTextA, Ord:0000h
                            |
:00401097 E8129C0000       Call 0040ACAE    ;得到用户名,在[ebp+FFFFFF48]

* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:0068, ""
                            |
:0040109C 6A68             push 00000068    ;同样的操作,输入注册码的文本框 ID
:0040109E 53               push ebx

* Reference To: USER32.GetDlgItem, Ord:0000h
                            |
:0040109F E8FE9B0000       Call 0040ACA2
:004010A4 6A64             push 00000064
:004010A6 8D8DE4FEFFFF     lea ecx, dword ptr [ebp+FFFFFEE4]
:004010AC 51               push ecx
:004010AD 50               push eax

* Reference To: USER32.GetWindowTextA, Ord:0000h
                            |
:004010AE E8FB9B0000       Call 0040ACAE    ;得到注册码,在[ebp+FFFFFEE4]

* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:0067, ""
                            |
:004010B3 6A67             push 00000067    ;这个是最下面的提示的文本框的 ID
:004010B5 53               push ebx
7
* Reference To: USER32.GetDlgItem, Ord:0000h
                            |
:004010B6 E8E79B0000       Call 0040ACA2    ;得到句柄
:004010BB 8BF0             mov esi, eax     ;放在 ESI 备用
:004010BD 8D8548FFFFFF     lea eax, dword ptr [ebp+FFFFFF48]
:004010C3 50               push eax         ;指向用户名
:004010C4 E867050000       call 00401630    ;得到用户名长度
:004010C9 59               pop ecx
:004010CA 8945D8           mov dword ptr [ebp-28], eax  ;长度放在[ebp-28]
:004010CD 8D95E4FEFFFF     lea edx, dword ptr [ebp+FFFFFEE4]
:004010D3 52               push edx         ;指向注册码
:004010D4 E857050000       call 00401630    ;得到注册码长度
:004010D9 59               pop ecx
:004010DA 68EAB04000       push 0040B0EA
:004010DF E84C050000       call 00401630
:004010E4 59               pop ecx
:004010E5 680EB14000       push 0040B10E
:004010EA E841050000       call 00401630
:004010EF 59               pop ecx
:004010F0 837DD803         cmp dword ptr [ebp-28], 00000003
:004010F4 7E7B             jle 00401171     ;用户名长度不能小于等于 3
:004010F6 90               nop
:004010F7 90               nop
:004010F8 90               nop
:004010F9 90               nop
:004010FA 33C9             xor ecx, ecx
:004010FC 33D2             xor edx, edx
:004010FE 33DB             xor ebx, ebx
:00401100 33C0             xor eax, eax
:00401102 837DD832         cmp dword ptr [ebp-28], 00000032
:00401106 7D69             jge 00401171     ;用户名长度不能大于等于 32h
:00401108 90               nop
:00401109 90               nop
:0040110A 90               nop
:0040110B 90               nop

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040111C(C)
|
:0040110C 0FBE840D48FFFFFF movsx eax, byte ptr [ebp+ecx-000000B8];依次取用户名的字符
:00401114 41               inc ecx          ;ECX 为循环变量
:00401115 33C1             xor eax, ecx     ;取的字符与循环变量 XOR
:00401117 03D8             add ebx, eax     ;把结果累加到 EBX
:00401119 3B4DD8           cmp ecx, dword ptr [ebp-28]  ;循环变量与用户名长度相比
:0040111C 75EE             jne 0040110C     ;如果未取完就跳回继续
:0040111E 6BC006           imul eax, 00000006  ;最后一轮计算的结果在 EAX, 乘 6
:00401121 C1E307           shl ebx, 07      ;前面累加结果左移 7 位
:00401124 03C3             add eax, ebx     ;相加
:00401126 8945C8           mov dword ptr [ebp-38], eax
:00401129 FF75C8           push [ebp-38]    ;把上面结果压栈

* Possible StringData Ref from Data Obj ->"%lX"
                            |
:0040112C 6838B44000       push 0040B438    ;一个转换的标识
:00401131 8D8D80FEFFFF     lea ecx, dword ptr [ebp+FFFFFE80]
:00401137 51               push ecx         ;存放转换结果的地址
:00401138 E8873D0000       call 00404EC4    ;数字转为十六进制字串
:0040113D 83C40C           add esp, 0000000C
:00401140 8D8580FEFFFF     lea eax, dword ptr [ebp+FFFFFE80]
:00401146 50               push eax         ;上面转换的字串
:00401147 8D95E4FEFFFF     lea edx, dword ptr [ebp+FFFFFEE4]
:0040114D 52               push edx         ;假注册码

* Reference To: KERNEL32.lstrcmpA, Ord:0000h
                            |
:0040114E E8339C0000       Call 0040AD86    ;比较
:00401153 85C0             test eax, eax
:00401155 750D             jne 00401164     ;这里就是关键的跳转

* Possible StringData Ref from Data Obj ->"Congratulations! IF this number "
                          ->"comes *FROM YOUR* keygen, Write "
                          ->"a tutorial dude ;)"
                            |
:00401157 683CB44000       push 0040B43C    ;指向表示成功的字符串
:0040115C 56               push esi         ;ESI 还记得么?那个提示文本框的句柄

* Reference To: USER32.SetWindowTextA, Ord:0000h
                            |
:0040115D E8289B0000       Call 0040AC8A    ;显示出来
:00401162 EB18             jmp 0040117C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401155(C)
|

* Possible StringData Ref from Data Obj ->"This serial is *NOT* Valid!! Try "
                          ->"again... : UNREGISTERED"
                            |
:00401164 6890B44000       push 0040B490    ;开始时停在这句，向上找跳转
:00401169 56               push esi         ;ESI 提示文本框的句柄

* Reference To: USER32.SetWindowTextA, Ord:0000h
                            |
:0040116A E81B9B0000       Call 0040AC8A
:0040116F EB0B             jmp 0040117C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004010F4(C), :00401106(C)
|

* Possible StringData Ref from Data Obj ->"Name must contain more than 4 "
                          ->"chars and less than 50 chars !!"
                            |
:00401171 68C9B44000       push 0040B4C9    ;用户名不符合要求跳到这里
:00401176 56               push esi         ;ESI 提示文本框的句柄

* Reference To: USER32.SetWindowTextA, Ord:0000h
                            |
:00401177 E80E9B0000       Call 0040AC8A

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00401162(U), :0040116F(U)
|
:0040117C 5F               pop edi
:0040117D 5E               pop esi
:0040117E 5B               pop ebx
:0040117F 8BE5             mov esp, ebp
:00401181 5D               pop ebp          ;整理一下返回了
:00401182 C3               ret
```

双击后光标停在 `401164` 这一句。很明显，如果我们来到这句时我们就死翘翘了，而如果我们的注册码正确的话当然不会来到这一句（废话太多了:p）那么这一句上面就肯定会有一个条件跳转的指令。（这是找爆破点时的基本思想）向上找找看，找到了：

`:00401155 750D             jne 00401164`

正好跳到 `401164` 错误信息那一句。呵呵，如果你想爆破的话，只要把 `750D` 改成 `740D`（`je`,把条件反过来，注册码错误就显示正确信息:D）或者改成 `EB0D`（`jmp`,无条件跳转，不管三七二十一就正确）。

OK，我们不能满足于此啊，咱们看看它的算法是怎样的，也像那些神秘兮兮的高手似的写一个注册机出来。:D



### 函数调用机制

我先给各位补一点课，就是对函数的调用。除了一些 `DELPHI` 程序之外，对函数参数的传递大都用堆栈来完成，简单地说就是把函数的各个参数先 `PUSH` 进去，然后再 `CALL` 这个函数。在函数内部呢，一般 `[ebp+8]` 是第一个参数，`[ebp+C]` 是第二个参数，每次多加 4 依此类推。而函数内部的局部变量常用 `[ebp-4]` `[ebp-8]` ...等等。（原因讲起来有点复杂，先记住就行了）函数的返回值在 `EAX` 里。

### 函数识别

一般来说，软件的判断注册部分都是一个函数，在函数开头最经典的两句就是：

```
push ebp
mov ebp, esp
```

这和堆栈处理有关，我们菜鸟先不用太明白，知道这通常是一个函数的开始就行了。向上找找有没有它，找到了没有，在最上面哪（我上面没有列出来）。如果你想完整地判断它的算法的话，一般从这里开始就行了。在这个程序中前面都是一些初始化之类的东东，所以我把前面一部分省略了。（这也是破解时的原则，不要在无关紧要的地方费功夫，在高级语言中，代码有很大部分是机器自动生成的，电脑一行行写代码不知道累，人脑一行行读代码怎么受得了？你的脑袋是几 `GHz` 的 `CPU`？常见有些没有破解经验的汇编高手，完全懂得每行代码的意思，就是找不到关键的地方，原来他跟了半天都是在 `API` 里转，白做无用功了。）

### 动态调试与 GetDlgItem

好了，现在可以动态调试了。我们在开头这里下个断点，一步步向下看，出现了一个 `CALL GetDlgItem`，我们来看一看函数说明（手头一份这个是必需的）：

```c
HWND GetDlgItem(
    HWND hDlg,   // handle of dialog box
    int nIDDlgItem  // identifier of control
);
```

呵呵，简单的说这个函数就是让程序确定一个对话框上的控件，第一个参数是对话框的句柄，第二个参数是对话框上某个控件的 `ID`，函数会返回该控件的句柄，这样在下面就可以用这个句柄来操作了。看程序：

```
* Possible Reference to Dialog: DialogID_0001, CONTROL_ID:0066, ""
                              |
:00401085 6A66                 push 00000066    ;控件 ID
:00401087 53                   push ebx    ;对话框句柄

* Reference To: USER32.GetDlgItem, Ord:0000h
                              |
:00401088 E8159C0000           Call 0040ACA2
```

看见第一行没有，`DialogID_0001,CONTROL_ID:0066`，压进去了一个 66 作为第二个参数：控件 `ID`（注意 `API` 调用是从右至左，也就是最后面一个参数先 `PUSH`），用 资源查看工具 看看 `ID` 为 66 的是甚么呀，呵呵就是那个输入用户名的文本框嘛。好了，现在我们有文本框的句柄了，存在 `EAX` 里。接着向下看，又一个 `API`，是 `GetWindowText`。看看说明：

```c
int GetWindowText(
    HWND hWnd,   // handle of window or control with text
    LPTSTR lpString,  // address of buffer for text
    int nMaxCount  // maximum number of characters to copy
);
```

从名字也能猜出来了，这个函数就是得到一个窗口类控件的文本。第一个参数是该控件的句柄，第二个是存放得到的文本的缓冲区的地址，第三个参数设定取文本的最大长度。看程序：

```
:0040108D 6A64                 push 00000064    ;最大长度
:0040108F 8D9548FFFFFF         lea edx, dword ptr [ebp+FFFFFF48];把[ebp+FFFFFF48]先放在 EDX 里
:00401095 52                   push edx    ;缓冲区地址[ebp+FFFFFF48]
:00401096 50                   push eax    ;EAX？是上面那个 API 的返回值呀，控件句柄

* Reference To: USER32.GetWindowTextA, Ord:0000h
                              |
:00401097 E8129C0000           Call 0040ACAE
```

好了，现在 `D ebp+FFFFFF48` 看看，是不是输入的用户名？（说明，在 `OLLY` 或 `SICE` 里这个是 `[ebp-B8]` 的形式，其实是一样的）

下面有类似的操作，只是控件 `ID` 成了 68，可想而知就是得到注册码了，注册码放在 `[ebp+FFFFFEE4]`。

呵呵，继续，到了这么几句：

```
:004010BD 8D8548FFFFFF         lea eax, dword ptr [ebp+FFFFFF48]
:004010C3 50                   push eax
:004010C4 E867050000           call 00401630
```

这是干甚么牙？要不要进 `4010C4` 这个 `CALL 401630` 看看？且慢！我前面说过了，不要在无关紧要的地方费力，（啪！一个鸡蛋扔上来：谁知道这里是不是重要啊？）别急嘛，一般来说，先粗略地跟一遍，试着猜猜 `CALL` 的作用。如果发现返回值很可疑，再跟进细看也不迟。（这就需要一定的“直觉”了，英文叫“Sence”，台湾老大叫“触机”或者“先死:D”，别担心，破解得多了自然会有这种感觉，这种感觉很难说出来，有时候一见那种阵势就知道关键地方到了）

又扯远了，我们回来看代码。一个 `CALL`，前面有 `PUSH EAX`，很明显是 `CALL` 的参数了。（见我前面的说明）看看 `EAX`：`lea eax,dword ptr [ebp+FFFFFF48]`。`[ebp+FFFFFF48]` 还有没有印象？对了，就是前面 `API` 调用中放用户名的地址呀。过了这里后 `D eax`，呵呵不就是咱们的用户名吗，我输入的是“RoBa”，带过这个 `CALL` 后看看 `EAX` 是什么，嗯~~~~是 4。想到什么了？没有？没关系咱们继续看下面：

```
:004010C9 59                   pop ecx
:004010CA 8945D8               mov dword ptr [ebp-28], eax  ;记住，上面的返回结果在[ebp-28]
:004010CD 8D95E4FEFFFF         lea edx, dword ptr [ebp+FFFFFEE4]
:004010D3 52                   push edx
:004010D4 E857050000           call 00401630
```

又是一个 `CALL 401630`，和上面一样的。看看它的参数 `EDX`，`[ebp+FFFFFEE4]` 呵就是假注册码呀，我输的是 `'87654321'`，返回值 `EAX=8`，发现什么没有？对，这个 `CALL` 就是求一个字符串的长度嘛，要是跟进去又给费不少时间。向下：

```
:004010F0 837DD803             cmp dword ptr [ebp-28], 00000003
:004010F4 7E7B                 jle 00401171

:00401102 837DD832             cmp dword ptr [ebp-28], 00000032
:00401106 7D69                 jge 00401171
```

还记得 `[ebp-28]` 是什么吗？对，就是用户名的长度，这几句的意思就很明显了，用户名的长度必须大于 3 小于 32h，不然就跳到 `401171` 去了，你可以跟过去看看是什么，呵呵是用户名不符合要求之类的提示。

### 关键算法分析

好了我们来到关键地方喽：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040111C(C)
|
:0040110C 0FBE840D48FFFFFF     movsx eax, byte ptr [ebp+ecx-000000B8]
:00401114 41                 inc ecx   
:00401115 33C1               xor eax, ecx
:00401117 03D8               add ebx, eax
:00401119 3B4DD8             cmp ecx, dword ptr [ebp-28]
:0040111C 75EE               jne 0040110C   
```

呵呵，是不是又晕了？`[ebp+ecx-B8]` 这是啥呀？别着急，车到山前必有路。先 `d ebp+ecx-B8` 看看，哟，不还是咱们的用户名吗？:D 怎么回事？好好想想，`ecx` 现在是 0，`[ebp-B8]` 和 `[ebp+FFFFFF48]`（记得不？放用户名的地址）不是一回事嘛！（不明白的去补习一下负数的表示方法，我也不知道为什么 `W32DASM` 有时候非把 `-B8` 写成 `FFFFFF48`）

再仔细看看，`movsx eax,byte ptr [ebp+ecx-000000B8]`，注意是 `byte ptr`，即以字节的方式读取（就是说每次读出一个字符），而且又加上了个 `ecx`。如果你破解多了的话，应该立刻就明白：关键的地方到了。

这是一个很典型的循环结构，看出来没？`ecx` 就是循环变量了，每执行一次会从用户名中取一个字符，然后 `ecx` 加 1，这样 `[ebp+ecx-B8]` 就指向用户名的下一个字符了。对取出来的字符与循环变量进行 `XOR` 运算，把结果累加到 `EBX`。然后循环变量与 `[ebp-28]` 也就是用户名长度比较，如果不等于的话也就是还没取完，就返回上去继续取用户名的下一个字符。这样直到取完为止。

```
:0040111E 6BC006               imul eax, 00000006  ;EAX 其实是上面最后一轮计算的结果，乘 6
:00401121 C1E307               shl ebx, 07    ;EBX 是几轮计算累加起来的结果，左移 7 位
:00401124 03C3                 add eax, ebx    ;加起来
:00401126 8945C8               mov dword ptr [ebp-38], eax
:00401129 FF75C8               push [ebp-38]    ;上面的结果，作为一个参数

* Possible StringData Ref from Data Obj ->"%lX"
                              |
:0040112C 6838B44000           push 0040B438    ;"%lX"有点眼熟哟
:00401131 8D8D80FEFFFF         lea ecx, dword ptr [ebp+FFFFFE80]
:00401137 51                   push ecx    ;这是什么呢？
:00401138 E8873D0000           call 00404EC4
:0040113D 83C40C               add esp, 0000000C
```

前面几句是继续上面的计算，把 `EAX*6` 和 `EBX` 左移 7 位的值加进来，然后结果复制到 `[ebp-38]` 这个局部变量里作为下面 `CALL` 的一个参数，接着 `[ebp+FFFFFE80]` 作第二个参数，然后又一个 `CALL`，还是那样，先别着急跟进去，前后看看有没有可疑之处：

第一个参数 `[ebp-38]`，没什么好说的，上面计算的结果。第二个参数指向 `"%lX"`，这个字串写过 `C` 语言的都有印象吧，就是在 `printf` 里把一个数字按照大写的 16 进制方式显示出来所用的标识符，比方说把一个数字 255 转换成字串“FF”。（在 `WinAPI` 里叫 `wsprintf`）第三个参数是什么呢？前面这种形式见多了吧，猜猜！

我们验证一下前面猜想是否正确，我用 `"RoBa"` 上面算出来的结果是 46430，也就是 16 进制的 `B55E`，我们跟过这个 `CALL` 看看结果如何？什么，结果在哪？聪明的你还没想到吗，上面那第三个参数 `[ebp+FFFFFE80]` 就是结果的地址呀。怎么样不出所料吧。

（其实从 `40113D` 这一句 `add esp,C` 也能看出来，因为在函数外平衡堆栈的只有这个一个参数数目不定的函数。引申一下，看不懂没关系啦:D ）

接着看啦：

```
:00401140 8D8580FEFFFF         lea eax, dword ptr [ebp+FFFFFE80];眼熟吗，刚才的转换结果呀
:00401146 50                   push eax  ;EAX 指向上面转换得到的字符串
:00401147 8D95E4FEFFFF         lea edx, dword ptr [ebp+FFFFFEE4];这个很早了，向前面找找是啥
:0040114D 52                   push edx  ;EDX 指向我们输入的假注册码

* Reference To: KERNEL32.lstrcmpA, Ord:0000h
                              |
:0040114E E8339C0000           Call 0040AD86
:00401153 85C0                 test eax, eax
:00401155 750D                 jne 00401164  ;关键跳转哟
```

哈哈，`lstrcmp`，什么意思不用我说了吧，当然是 `STRingCoMPare` 字符串比较啦。把计算的结果与前面输入的假码比较，相等就 `OK` 了。好了，现在把上面的完整的代码过一遍，怎么样，写个注册机不难吧？

### 注册机源码

```c
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

void main()
{
  int EAX=0,EBX=0,len;
  char name[50]={0};
  char password[50]={0};
  printf("Please input your name:");
  scanf("%s",name);
  len=strlen(name);
  for (int i=0;i<len;i++)
  {
    EAX=name[i]^(i+1);
    EBX+=EAX;
  }
  EAX*=6;
  EBX<<=7;
  EAX+=EBX;
  printf("Your password is: %lX\n",EAX);
  printf("KeyGen by RoBa  Enjoy Cracking,Newbies!\n");
}
```

简单的 `C` 程序哟，你也可以用你熟悉的语言写一个。

### 后记

这是一个简单的 `CRACKME`，本来三言两语就能解决的，被我啰啰嗦嗦说了这么一大堆，就是希望能把问题真正地说明白了，希望通过这篇文章让你发现，破解软件乃至写注册机并不是多么困难的事，只要坚持下去，谁都有成为高手的那一天，也希望高手们能够抽时间发一些适合较低水平的文章或者至少对发这种文章的新手多多鼓励。`Enjoy Cracking,Newbies!`

`4dd466d0e1e0cc123afbe504da2462ca`
