
# Advanced Evaluator 3.5

> 原文：[`www.pediy.com/kssd/pediy03/forum689.htm`](https://www.pediy.com/kssd/pediy03/forum689.htm)

## 标题
Advanced Evaluator 3.5 (另一强大的科学计算器--注册运算也很强大哟)破解~~新手请进~~有详细注解~高手们做个注册机吧~~~呵呵

## 内容

差点就要放弃啦~~~~~~~

**软件名称**：Advanced Evaluator 3.5 (功能强大的科学计算器)

**下载地址**：[`http://61.134.4.193:8080/appsoftware/calculator/ae35.zip`](http://61.134.4.193:8080/appsoftware/calculator/ae35.zip) (708KB)

**注册形式**：Name & Key (我用了 `Name:Sam Von Key:67676767`) 30 天限制，过了 30 天就不能用了

**破解原因**：同上次一样，找了一大堆计算器想算注册码，这个也一样不合我心意，但它有注册保护，刚好可以练一下

但这个比上次那个难多啦(对于我来说)，看我的标题就知道啦，不过这次可增加了我不少经验值，顺便问下大家，有哪个计算器软件或其它工具可以进行汇编的 `shl`, `nor`, `rol`, `xor` 等这些逻辑运算的请告诉我下载地址~~~~谢谢大家~~~~

### 破解过程及心得

这次破解简直是瞎猫捉到死老鼠，一开始当然是例行公事一样的填入我的 `Name&Key`，打开 `Trw2000`，输入 `bpx hmemcpy`，然后按确定，Booom~~~`Trw` 拦下来啦，然后就 `bd`、`pmodule`，来到了程序的代码中，呵呵~~~~有谁算过这个过程要多长时间。通常来说这时我都会看看各个寄存器的值，一般来说这时 `eax` 的值就是你 `Name` or `Key` 的长度，而我这里是我 `Name` 的长度 7，好啦它读了我的 `Name`，马上用 `s` 命令查找 `'Sam Von'` (`s` 命令会用吧，反正我刚学的时候就不怎么会用，谁不懂就问吧)，找到啦，当然是“`bpm 地址 rw`”啦，这个命令很好用的，只要任何程序读写了存放我的 `Name` 的地址，`Trw` 就会中断了，我的猜想是程序读我的 `Name` 作比较或者作运算或者把它放到第二个地址去，所以我通常有个习惯就是用 `bpm` 把所有存放我 `Name & Key` 的地址都设中断，反正程序一读我的 `Name&Key` 就是要做小动作啦，所以这样比较容易跟也容易找到核心，当程序再次中断时，我发现它在拿我的 `Name` 和 `sudi` 比较，但我用 `sudi` 来注册时一样出错，那这个可能是黑名单吧，这里应该不是重点，再跟~~~~~~~又发现程序在查我 `Name` 的二进制，要用到二进制来运算，麻烦啦，继续~~~~~~中间好多 `F8` `F10` 的不停的按，反正要注意各寄存器的值，在 `Trw` 中的汇编代码窗口右上角的提示很重要，多多留意，一发现出现有疑的数据就要 `d` 来看看。

其中有几个地方应该都是运算重点，但我没有贴出代码，只给出地址，有兴趣的就自己看看吧：
* 在 `4027EC` 处程序用我 `Name` 的 16 进制加上某个地址再与另一地址比较；
* 在 `478B1B` 处程序作了运算(用了除法等命令)；
* 在 `479471-->4794B0` 处程序将我 `Name` 的二进制连续放在内存中然后比较、查表、运算。

最下面我贴出那些代码是对 `Key` 的比较运算，反正我再跟下去就晕啦，因为后面的 `call` 很多，运算也很复杂，我看不懂代码，但我想后面应该就是运算核心啦，高手们可以研究一下。

上面的过程我花了两天时间，始终搞不懂，你一定想问我为什么不用反汇编呢，其实我一早用过啦，程序是 `Delphi` 写的，里面一点有用的字串都没有，只有十多行而已(BTW:为什么纯 `Delphi` 的程序用 `W32dasm` 反汇编后在字串表中都只有几行呢，是程序本身的设计还是 `Delphi` 的限制呢)，我开始怀疑是否是我的起点过高呢~~~~~~~~~~~~呵呵

眼花啦~~~~每次看下面运算的代码都不得重点，我实在看不懂运算的过程，而且后面的 `call` 太多啦，唉没心机啦，于是像傻子一样不停的按 `F10~~~~`忽然在 `4799AB` 处停了下来，其实这里我以前也来过几次，不过总觉得比较语句应该是 `cmp al,01` 之类的才是重要的判断，不会同 `2` 比吧，给骗了~~~这时将 `al` 改为 `2` 就完全破解啦，这完全是我无路可走才想到乱改一气的，所以说捉到死老鼠啦~~~~~呵呵

再说两句，破解真的好累，写文章就更累~~~~这是我写的第三篇，这几天都没精神啦，另外我写的可能很乱很罗嗦，请大家原谅，我文笔本来就不好，加上我写的这些都是想保留下来以后温习用的，所以写的很详细(因为我是一个新手，功力 0.6 级)，记得我刚开始学破解时到处找教程，但几乎每个教程都那么简单，可能我人蠢吧，很多东西看不懂，所以我就将破解的过程整理后贴出来咯，希望能帮得了一些新手，也请高手指正，毕竟这些都是我自己理解的东西~~~~难免有错的~~~~谢了~~~~~~~~~~~~~

#### 核心判断的 `call`

```
* Referenced by a CALL at Addresses:
:00478C93  , :00478CA2  , :00479039  , :00479049  , :0047974C
:00479780  , :004799A6

:00478A48 53                    push ebx
:00478A49 56                    push esi
:00478A4A 57                    push edi
:00478A4B 55                    push ebp
:00478A4C 8BEA                  mov ebp, edx
:00478A4E B202                  mov dl, 02        <----dl\=2 此为注册标志想办法保持这个数就行啦
:00478A50 8B7504                mov esi, dword ptr [ebp+04]<---[C60994]=252E---假密码
:00478A53 8B4804                mov ecx, dword ptr [eax+04]<---[C67F88]=1CD3---真密码
```

---

**To：新手~~~~~~~~~~**

我不知道程序将我的 `Name` 和 `Key` 作了什么运算，反正到了此处程序就拿这两个数作比较，我就称它们为真假密码吧，请留意程序并没有将密码连续放在内存中，而是分散的，它的存放是这样的`[ebp(eax)]`是密码的基地址，`[ebp(eax)+4]`是密码的值，`[ebp(eax)+8]`是下一个密码的基地址，程序先比较第一个密码，不等的话还会去比较第二个，这一点我就不明白啦，第一个不等就算啦嘛，为何还做下面的比较，就算后面的密码相等 `bl` 也不可能变为 `2` 呀，这是我第一次碰到的情况，所以我把它记了下来，不要说我罗嗦呀~~~~~~~~~~

---

```
:00478A56 3BF1                  cmp esi, ecx
:00478A58 7D04                  jge 00478A5E      <---此处 esi>ecx 所以跳啦
:00478A5A 33D2                  xor edx, edx      <---不跳的话 edx 就被清为 0 啦
:00478A5C EB20                  jmp 00478A7E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00478A58(C)

:00478A5E 3BF1                  cmp esi, ecx      <----再比不等 dl 就变为 1 啦
:00478A60 7E1C                  jle 00478A7E
:00478A62 B201                  mov dl, 01
:00478A64 EB18                  jmp 00478A7E      <----一定跳啦，不过 dl 为 1 啦

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00478A90(C)

:00478A66 8BC6                  mov eax, esi      <----基地址
:00478A68 8BEF                  mov ebp, edi
:00478A6A 8B7504                mov esi, dword ptr [ebp+04]<----第二个密码 22B4
:00478A6D 8B4804                mov ecx, dword ptr [eax+04]<----211
:00478A70 3BF1                  cmp esi, ecx
:00478A72 7D04                  jge 00478A78      <------看~~就算相等也没用，刚才 dl 已经为 1 啦
:00478A74 33D2                  xor edx, edx
:00478A76 EB06                  jmp 00478A7E

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00478A72(C)

:00478A78 3BF1                  cmp esi, ecx
:00478A7A 7E02                  jle 00478A7E
:00478A7C B201                  mov dl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:00478A5C(U), :00478A60(C), :00478A64(U), :00478A76(U), :00478A7A(C)

:00478A7E 8B7008                mov esi, dword ptr [eax+08]<----下一个密码的基地址
:00478A81 85F6                  test esi, esi
:00478A83 0F95C1                setne cl          <----在 Trw 中此句是"setnz cl" 所以 cl=1
:00478A86 8B7D08                mov edi, dword ptr [ebp+08]<----下一个密码的基地址
:00478A89 85FF                  test edi, edi
:00478A8B 0F95C3                setne bl          <----bl\=1,第二次到此 bl=0 下面就不跳啦共比较了四次
:00478A8E 22CB                  and cl, bl
:00478A90 75D4                  jne 00478A66      <----在 Trw 中此句是 jnz 所以跳了
:00478A92 8B7808                mov edi, dword ptr [eax+08]
:00478A95 85FF                  test edi, edi
:00478A97 0F94C1                sete cl
:00478A9A 8B7508                mov esi, dword ptr [ebp+08]
:00478A9D 85F6                  test esi, esi
:00478A9F 0F95C3                setne bl
:00478AA2 22CB                  and cl, bl
:00478AA4 7402                  je 00478AA8
:00478AA6 B201                  mov dl, 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00478AA4(C)

:00478AA8 85F6                  test esi, esi
:00478AAA 0F94C1                sete cl
:00478AAD 85FF                  test edi, edi
:00478AAF 0F95C0                setne al
:00478AB2 22C8                  and cl, al
:00478AB4 7402                  je 00478AB8
:00478AB6 33D2                  xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:00478AB4(C)                    上面的代码没能将 dl=02

:00478AB8 8BC2                  mov eax, edx      <---此处 edx=0 所以 al=0 注意 ret 后就是比较啦
:00478ABA 5D                    pop ebp
:00478ABB 5F                    pop edi
:00478ABC 5E                    pop esi
:00478ABD 5B                    pop ebx
:00478ABE C3                    ret

=================================================================================================

:0047999A 8B45EC                mov eax, dword ptr [ebp-14]
:0047999D 8945F0                mov dword ptr [ebp-10], eax
:004799A0 8B55F4                mov edx, dword ptr [ebp-0C]
:004799A3 8B45F0                mov eax, dword ptr [ebp-10]
:004799A6 E89DF0FFFF            call 00478A48     <----上面的代码
:004799AB 3C02                  cmp al, 02          <----呵呵~~~~~~~看下面的代码，根本没有跳转语句，跟
                                                    进后面的 call 找不到重点，所以说我是瞎撞的
```

---
