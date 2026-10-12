
正如我们所料，中上洞为`dl`为`01`。这样看来，更改`dl`的值就更改了按键的值。用`twr`测试一下，果然如此。那么把`dl`改为`03`、`04`、`05`等，是不是原来无效的键就有效了？

用`twr`测试，事情并没有如我们所愿，还是只有原来的三个洞。好！这样的话，看看是什么地方判断这个`dl`。

`:00470F55 E88AB0FFFF call 0046BFE4` 会把`dl`的值放入`[eax+68]`中。

```
:0046BFE4 885068           mov byte ptr [eax+68], dl
:0046BFE7 C3               ret
```

那么就下断点`bpm [eax+68]`。

会发现程序在下面有判断。

```
:0046AFEF 8A4368           mov al, byte ptr [ebx+68]
:0046AFF2 2C01             sub al, 01
:0046AFF4 720B             jb 0046B001 => dl 为 00 左上洞
:0046AFF6 7432             je 0046B02A => dl 为 01 中上洞
:0046AFF8 FEC8             dec al
:0046AFFA 7478             je 0046B074 => dl 为 02 右下洞
:0046AFFC E9F37A0000       jmp 00472AF4 => 调转到辅助功能键的判断
```

这样看来源程序并没有其他三个洞的按键判断。

好，我们在往下分析`jb 0046B001`等代码。

`jb 0046B001` 左上洞:

```
:0046B001 FF7604           push [esi+04]
:0046B004 FF36             push dword ptr [esi]
:0046B006 FF7614           push [esi+14]
:0046B009 FF7610           push [esi+10]
:0046B00C 8D542418         lea edx, dword ptr [esp+18]
:0046B010 8BC3             mov eax, ebx
:0046B012 E8A5C7FFFF       call 004677BC
:0046B017 8D742408         lea esi, dword ptr [esp+08]
:0046B01B 8DBB78010000     lea edi, dword ptr [ebx+00000178]
:0046B021 B904000000       mov ecx, 00000004
:0046B026 F3               repz
:0046B027 A5               movsd
:0046B028 EB72             jmp 0046B09C
```

`je 0046B074` 右下洞:

```
:0046B074 FF760C           push [esi+0C]
:0046B077 FF7608           push [esi+08]
:0046B07A FF761C           push [esi+1C]
:0046B07D FF7618           push [esi+18]
:0046B080 8D542418         lea edx, dword ptr [esp+18]
:0046B084 8BC3             mov eax, ebx
:0046B086 E831C7FFFF       call 004677BC
:0046B08B 8D742408         lea esi, dword ptr [esp+08]
:0046B08F 8DBB78010000     lea edi, dword ptr [ebx+00000178]
:0046B095 B904000000       mov ecx, 00000004
:0046B09A F3               repz
:0046B09B A5               movsd
```

代码很相近，都是`push`四个参数然后一个`call 004677BC`。由此可见，`004677BC`处应该是一个计算是哪个洞的函数。

现在比较左上洞、右下洞的这 4 个参数有什么不同。

在`trw`的`stack`窗口看到：

左上洞： 右下洞:

```
00000000 00000000
406ac000 407e0666
00000000 cccccccd
406d0000 4088d0cc
```

我们更改一下参数试试。

把

右下洞: 改为

```
00000000 00000000
407e0666 406ac000 ==> 这个原来是左上洞的参数
cccccccd cccccccd
4088d0cc 4088d0cc
```

测试程序竟然瞄准了右上洞，这是原来软件不提供的功能，看来破解有希望了（大家不要高兴的太早，路还很长呢）。

好了回到程序里看看左上洞的参数`406ac000`是由`push [esi+14]`得来的。我们把右下洞的`:0046B07A FF761C push [esi+1C]`换成`push [esi+14]`，不就变成了右上洞了么！相应的把左上洞的`push [esi+14]`换成`push [esi+1C]`就成了左下洞，中下洞的方法一样哦。

`ok`，我们既然知道了程序是怎么打出 6 个洞的，那么就可以补出其他三个洞的功能。

前面说了这么多，现在开始正题`pediy`（我要全力贯彻看雪老大的宗旨，把`diy`进行到底）。

首先把其他的三个按键的判断`diy`出来。

我们已经知道把`dl`为`0`、`1`、`2`就代表左上，中上，右下的洞。那么我们加入代表左下，中下，左上的按键代表，分别为`03`、`04`、`05`。原来按下这几个键是显示注册后使用此洞，那么我们更改其显示代码，让其执行分别把`dl`设置为`03`、`04`、`05`。

在文章一开始大家可以看到相应的按键判断的`call`地址，我们只要修改`call`的内容让其设置`dl`。

修改前的代码： 修改后的代码：

```
* Referenced by a CALL at Address:                      * Referenced by a CALL at Address:

:004719BC    这里是原来按键小键盘 6 显示注册后选择此洞       :004719BC         这里是按键小键盘 6 让其更改 dl 值

* Possible StringData Ref from Code Obj -> "注册后选择此洞"

                        |                        

:004710A4 BAC0104700 mov edx, 004710C0          :004710A4 E87F000000 call 00471128 *call 按键小键盘 3 做相应的工作
:004710A9 8B80FC020000 mov eax, dword ptr [eax+000002FC] :004710A9 6652 push dx *push bx
:004710AF E860A4FBFF call 0042B514             :004710AB B205 mov dl, 05 *把 dl 设置为 5
:004710B4 C3 ret                              :004710AD 56 push esi *push esi
                                                  :004710AE BE6C694700 mov esi, 0047696C *为下面的 call 做参数准备
                                                  :004710B3 8B8600000000 mov eax, dword ptr [esi+00000000] *做参数准备
:004710B5 000000 BYTE 3 DUP(0)                 :004710B9 E826AFFFFF call 0046BFE4 *这里把 dl 值加入到按键标志中去
                                                  :004710BE 5E pop esi *pop esi
                                                  :004710BF 665A pop dx *pop dx
:004710B8 FFFFFFFF BYTE 4 DUP(0ffh)           :004710C1 C3 ret *ret
:004710BC 0E push cs                          :004710C2 B2E1 mov dl, E1
:004710BD 000000 BYTE 3 DUP(0)                :004710C4 BAF3D1A1D4 mov edx, D4A1D1F3
                                                  :004710C9 F1 BYTE 0f1h
                                                  :004710CA B4CB mov ah, CB
:004710C0 D7 xlat                             :004710CC B6B4 mov dh, B4
:004710C1 A2B2E1BAF3 mov byte ptr [F3BAE1B2], al :004710CE 0000 add byte ptr [eax], al
:004710C6 D1A1D4F1B4CB shl dword ptr [ecx+CBB4F1D4], 1
:004710CC B6B4 mov dh, B4
:004710CE 0000 add byte ptr [eax], al

* Referenced by a CALL s:                       * Referenced by a CALL ss:

:00471994                                        :00471994

* Possible StringData Rode Obj -> "注册后选择此洞"

                        |                        

:004710D0 BAEC104700 mov edx, 004710EC          :004710D0 E84FFEFFFF call 00470F24 *call 按键小键盘 4 做相应的工作
:004710D5 8B80FC020000 mov eax, dword ptr [eax+000002FC] :004710D5 6652 push dx *push dx
:004710DB E834A4FBFF call 0042B514             :004710D7 B203 mov dl, 03 *把 dl 设置为 3
:004710E0 C3 ret                              :004710D9 56 push esi *push esi
                                                  :004710DA BE6C694700 mov esi, 0047696C *为下面的 call 做参数准备
                                                  :004710DF 8B06 mov eax, dword ptr [esi] *做参数准备
:004710E1 000000 BYTE 3 DUP(0)                 :004710E1 E8FEAEFFFF call 0046BFE4 *这里把 dl 值加入到按键标志中去
                                                  :004710E6 5E pop esi *pop esi
                                                  :004710E7 665A pop dx *pop dx
:004710E4 FFFFFFFF BYTE 4 DUP(0ffh)           :004710E9 C3 ret *ret
```
