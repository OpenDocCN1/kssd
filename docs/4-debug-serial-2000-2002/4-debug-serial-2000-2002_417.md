
# Offline Explorer 1.4 破解教程

> 原文：[`www.pediy.com/kssd/pediy03/forum518.htm`](https://www.pediy.com/kssd/pediy03/forum518.htm)

虽然网上现在到处找得到 `offline explorer 1.4` 的破解了，可是一个多月前我还辛辛苦苦地到处找破解（那时还有一次可怜兮兮地在论坛 1212 了一回，问了一句“OFFLINE EXPLORER 好像很难破”之类的话，好不容易找到一个注册码，却又是无用的），现在参照看雪教学几篇文章破了几个软件之后，正好网上也没找到关于它的教程，终于可以自己拿它来开刀了！同时也把这十余天来学破解的一点经验总结一下，可能有助于比我还要 `new` 的 `new-newbies`。如果您是大、中级（大侠、大虾、中侠、中虾……）的，那么除非您有意指点一下，是无须浪费时间的了，因为我一定会很啰嗦的！！:-)

### 一、工具

1. 国产精品：`冲击波` 和 `TRW2000 1.22`，感谢两位作者（好像 `TRW2000` 不再更新了，不然怎么主页上有人问注册的事却没人回答）。
2. 十六进制（HEX）编辑器 `WINHEX`（前几天才从 `hying` 兄那儿学了一招，破之，马上就派上用场了！）或 `HEXWORKSHOP` 等。以及 `W32DASM`——静态反汇编利器。
3. 看雪教学（有工具也得有课本啊！当字典用也行，这一编只是习题附不标准答案）。大家有什么不明白的请查课本。
4. TARGET：`OFFLINE EXPLORER 1.4317`、`OFFLINE EXPLORER PRO 1.4338`。

### 二、手动脱壳

我觉得新手学脱壳其实也没什么，对于一些简单的壳（什么是壳？看书去！不过也可以一句话来形容，就是把壳剥了，我们才可以对蛋白蛋黄为所欲为），有了咱们的国产精品，实在是 `so easy`! 何况现在要找一个没加壳的实在不是一件容易的事（可不是所有的壳都这么易脱的，只是我运气好罢了，脱不了别怪我）。

1. 将 `冲击波` 挎到 `OE` 目录下，运行它，窗口中的数字就是蛋白之所在了，我这儿是 `00504C08`。
2. 运行 `TRW`，载入 `OE.EXE`，`LOAD`，于是我们来到了 `TRW` 的窗口。
3. `bpx cs:504c08`，因为这样可样设一个断点，当程序运行时就会停在那儿（别笑，别骂我啰嗦，开始时我的确是连这个都要想想的。）回车，再 `G`，再回车，你会发现你在 `XXX:00504c08` 处。
4. `MAKEPE AAA.EXE`，那么你将会在 `OE` 的目录下生成一个 `AAA.EXE` 的文件，这就是脱壳后的程序了，这就是蛋白蛋黄了，你会发现蛋白、蛋黄、比原先完全的一个还要大，原来是 `494K`，现在成了 `1.40M` 了。是不是 `SO EASY`！

### 三、开始 PATCH OFFLINE EXPLORER 1.4317

1. 我们的对象是 `OFFLINE EXPLORER 1.4317`，因为我觉得它比较有助于思考，比 `OFFLINE EXPLORER PRO 1.4338` 要麻烦一点。打开 `TRW`，`CTRL+N`，我们又来到 `WINDOWS` 窗口，跟平时输注册码一样，输入自己的 `NAME`，再随便输入一列数字，不要按 `OK`。再按 `CTRL+N`，回到 `TRW`，`bpx hmemcpy`（这是 `WINDOWS` 的底层函数，不明白？没关系，要明白的话，就去看课本），按一下 `F5`，又来到了 `WINDOWS`，按下 `OK`，我们又到了 `TRW` 窗口，这时你可以看到窗口下面有 `KERNEL` 的字样，这时我们不在程序中，键入 `bc *`，清除我们上面设的断点，`pmoudle`，你会发现窗口下面已经变成了 `OE！CODE` 等字样了，也就是说，我们来到了程序中了！

2. 连续按 `F10`，来到下面：

```
016F:004CE0A1  MOV       EAX,[0050BD84]
016F:004CE0A6  MOV       EAX,[EAX]
016F:004CE0A8  CALL    004534B0  //SORRY 窗口出现，按下确定
016F:004CE0AD  XOR       EAX,EAX   //又回到 TRW
016F:004CE0AF  POP       EDX
```

3. 运行 `W32DASM`，反汇编。（当然你也可样在 `TRW` 中用 `U` 命令。）找到上面的语句，看如何跳过上面的 `SORRY` 窗口，运气好的话，上面很容易找到一个比较，你跳过去后，就可以注册了，不过我们的运气不好。

```
:004CE0A8 E80354F8FF            call 004534B0

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:004CDFD1(U), :004CDFE8(U)

:004CE0AD 33C0                   xor eax, eax //有两处可以 JMP 过来,let's go!
```

让我们往前看：

```
:004CDF66 E869490200            call 004F28D4//FORTH   //这个 CALL 可疑，F8 进去看看
:004CDF6B 84C0                  test al, al             //THIRD 这儿不能为 0
:004CDF6D 747E                  je 004CDFED //SECOND 这儿就可以跳过下面的 JMP，我们可以试一试，在:004CDF6F A16CBB5000   mov eax, dword ptr [0050BB6C]  HEX 编辑器下改机器码为 757E，即 je
:004CDF74 8B00                  mov eax, dword ptr [eax]   改为 jne,好像成功了，但是没有，你:004CDF76 C6808806000001        mov byte ptr [eax+00000688], 01 运行程序，又成了 NOT REGISTER
:004CDF7D A16CBB5000            mov eax, dword ptr [0050BB6C] 所以 NEXT
:004CDF82 8B00                  mov eax, dword ptr [eax]
:004CDF84 058C060000            add eax, 0000068C
:004CDF89 8B55FC                mov edx, dword ptr [ebp-04]
:004CDF8C E8E75EF3FF            call 00403E78
:004CDF91 A16CBB5000            mov eax, dword ptr [0050BB6C]
:004CDF96 8B00                  mov eax, dword ptr [eax]
:004CDF98 0590060000            add eax, 00000690
:004CDF9D 8B55F8                mov edx, dword ptr [ebp-08]
:004CDFA0 E8D35EF3FF            call 00403E78
:004CDFA5 8BB310030000          mov esi, dword ptr [ebx+00000310]
:004CDFAB 807E4700              cmp byte ptr [esi+47], 00
:004CDFAF 7425                  je 004CDFD6
:004CDFB1 8D55F4                lea edx, dword ptr [ebp-0C]
:004CDFB4 8B8608020000          mov eax, dword ptr [esi+00000208]
:004CDFBA 8B08                  mov ecx, dword ptr [eax]
:004CDFBC FF511C                call [ecx+1C]
:004CDFBF 8B4DF4                mov ecx, dword ptr [ebp-0C]
:004CDFC2 A16CBB5000            mov eax, dword ptr [0050BB6C]
:004CDFC7 8B00                  mov eax, dword ptr [eax]
:004CDFC9 8B55FC                mov edx, dword ptr [ebp-04]
:004CDFCC E873530200            call 004F3344
:004CDFD1 E9D7000000            jmp 004CE0AD  //FIRST 如何不让程序运行这 JMP 呢？
```

上面的 `CALL`（之所以程序重新开始又变成 `NOT REGISTER`，是因为程序开始时又调用了这个 `CALL`，检测到 `AL` 为 `0`，你可以用 `TRW` 载入程序前为这个 `CALL` 设个断点看看）进入后一直来到下面：

```
:004F2E58 3B02                  cmp eax, dword ptr [edx]
:004F2E5A 7506                  jne 004F2E62
:004F2E5C C645FF01              mov [ebp-01], 01    //  THIRD 若置 1，则为注册
:004F2E60 EB06                  jmp 004F2E68        //  上面有许多个直接 JMP 到 4F2E68，都不能跳
:004F2E62 83C204                add edx, 00000004
:004F2E65 4B                    dec ebx
:004F2E66 75F0                  jne 004F2E58
:004F2E68 33C0                  xor eax, eax
:004F2E6A 5A                    pop edx
:004F2E6B 59                    pop ecx
:004F2E6C 59                    pop ecx
:004F2E6D 648910                mov dword ptr fs:[eax], edx
:004F2E70 688A2E4F00            push 004F2E8A
:004F2E75 8D45F0                lea eax, dword ptr [ebp-10]
:004F2E78 BA03000000            mov edx, 00000003
:004F2E7D E8C60FF1FF            call 00403E48
:004F2E82 C3                    ret
:004F2E83 E99409F1FF            jmp 0040381C
:004F2E88 EBEB                  jmp 004F2E75
:004F2E8A 8A45FF                mov al, byte ptr [ebp-01]//SECOND [EBP-1]中为注册标志
:004F2E8D 5E                    pop esi
:004F2E8E 5B                    pop ebx
:004F2E8F 8BE5                  mov esp, ebp
:004F2E91 5D                    pop ebp
:004F2E92 C3                    ret//FIRST      CALL 结束，向上看
```

所以实际上只要把上面一个你会直接 `JMP` 到 `4F2E68` 的语句改为 `JMP` 向 `4F2E5C` 就可以了（在实际上我是 `4F2E26` 这一句的），你可以先在 `TRW` 下键入 `A` 指令测试，`A 4F2E26`，回车，然后把变化了的机器码抄下来（最好抄长一些，保证它在程序中是唯一的），用 `HEX` 编辑器改一下，再运行一下，怎么样？可以了吧。把时间向后调，成功了！

### 四、关于 OFFLINE EXPLORER PRO 1.4338

在这个版本中，比较跟上面在同个原理的，如下：

```
:004CFDD6 E8FD800200            call 004F7ED8    //这个 CALL 里面也是比较，进去看看
:004CFDDB 84C0                  test al, al
:004CFDDD 747E                  je 004CFE5D

\--------------

:004F7ED8 8BC2                  mov eax, edx
:004F7EDA 8BD1                  mov edx, ecx
:004F7EDC E89BFAFFFF            call 004F797C    //很快又是一个 CALL？再进去！
:004F7EE1 C3                    ret
```

又是一连串的比较，最终又看 `AL` 的值是否为 `1`，这儿可以按上面的方法将 `AL` 置 `1`，但是这儿还有另一种修改的方法，如果你在进这个 `CALL` 之前把 `ESP EBP ESI EDI` 的值记一下，你会发现它们的值并没有变（我的理解是不变就可以不理下面的东西，不知有没有错，各位大侠请指点一下），所以实际上你只要把刚进入这个 `CALL` 的指令改为 `AL` 置 `1`，再 `RET` 就可以了。用 `HEX` 编辑器改一下，重新居动，你会发现已经是 `REGISTERED` 的了，只是没有你的名字而已。这应该在注册表中可以改动的，不过我没试，你可以自己找找。

那么前面的是否也可以用第二种方法？好像不行，至少我没找到可以修改的地方，在进入 `TEST AL AL` 之前那个 `CALL` 之后的机器码我用十六进制编辑器打开后并没有找到，不知是何原因，如果你找到了麻烦你告诉我。

### 五、几句废话

1. 如果你有什么问题不懂的话，最好先去看看书，好像论坛常有人问些教学里说得很明白的问题。
2. 如果你对破解本身不感兴趣而只为了某个软件找不到破解而要学和话，我劝你还是等一等，如果网上没有，一般也不是你我这种菜鸟学上几天就可以搞掂的。破解可能是一种乐趣，也可能是一种痛苦。论坛精华中有小楼的一篇破解杂谈，建议看看。

`4dd466d0e1e0cc123afbe504da2462ca`
