
下 `g c1f5c6`

到了 `cs:c1f5c6` ,

下 `a` ,

下 `jmp 478032` ,

到了 `cs:478032`

下 `a` ,

下 `mov dword ptr [c10f1a],00014ee9`

再下 `mov byte ptr [c10f1e],00`

再下 `mov word ptr [c111e9],74eb`

再下 `jmp c1f5d0`

把机械码纪录下来

是 `C7051A0FC100E94E0100`

`C6051E0FC10000`

`66C705E911C100EB74`

`E97F757A00`

为什么要下这些指令呢 ?

因为我们要把 `CS:c10f1a` 改成 `JMP C1106D` ,

机械码是

`E94E010000`,

但是要反过来 , dword 是 4 BYTE, 所以先取 `E94E0100`

但是要反过来 , 所以变成了 `00014ee9` ,

其他的也是同样道理 , 最后再跳回去 `c1f5d0`, 继续执行

用 `Offset Converter` 开启 `ASPACK.EXE` 输入 `478032`

跑出了 `38A32`, 所以用 `ULTRA EDIT` 开启 `ASPACK.EXE`

在 `38A32` 的地方 , 输入刚刚机械码

就是 `C7051A0FC100E94E0100`

`C6051E0FC10000`

`66C705E911C100EB74`

`E97F757A00`

### 18:
再来就是 `cs:c1f5c6`, 这个地方如何变成 `jmp 478032` 呢 ?

因为 `cs:c1f5c6` 这个地方 , 是 `aspack` 用 `SMC` 产生的 ,

所以无法直接修改他 , 那就来观察是什么时候产生这边的 `code`,

用 `symbol loader` 载入 , 下 `bpm c1f5c6`, 按 `F5`, 会在这边停住

`cs: 466310` ,

在按一次 `F5`, 会在 `cs:c1f0eb` 停住

共有两次会修改到 , 在第二次的地方比较重要

所以 , 到了 `cs:c1f0eb` , 下 `BD*` , 按几下 `F10`,

到了这边

`cs:c1f0eb repz movsb`

`cs:c1f0ed mov eax , [ebp+442939]`

`cs:c1f0f3 push 00008000`

`cs:c1f0f8 push 0`

当走到了 `cs:c1f0f3 push 00008000` ,

这边的机械码是 `6800800000`, 把他改成 `jmp 478051`

所以下 `a`, 再下 `jmp 478051` 机械码是 `E9598F85FF`

到了 `CS:478051` 下 `A` ,

再下 `PUSH 00008000`, 按两下 `ENTER`,

仔细看一下 , 怎么变成 `PUSH 8000` 了 ...

这可不行 , 一定要 `PUSH 00008000` 才行啊

所以下 `E 478051` , 游标跑到上面去了 , 改记忆体的值 , 改成

`6800800000` , 这样才对 ,

好了 , 按一下 `F10`, 在开始下 `A`,

再下 `MOV DWORD PTR [C1F5C6] , 858A67E9`

再下 `MOV BYTE PTR [C1F5CA] , FF`

再下 `JMP C1F0F8`

改了四行 , 机械码总共是 ,

`6800800000`

`C705C6F5C100E9678A85`

`C605CAF5C100FF`

`E98C707A00`

一样的 , 用 `Offset Converter` 开启 `ASPACK.EXE` 输入 `478051`

跑出 `38A51`,

所以用 `ULTRA EDIT` 开启 `ASPACK.EXE` , 在 `38A51` 的地方输入

`6800800000`

`C705C6F5C100E9678A85`

`C605CAF5C100FF`

`E98C707A00`

### 19:
再来就是 `c1f0f3`, 这个地方如何变成 `push 00008000` 呢 ?

用 `symbol loader` 载入 , 下 `BPM C1F0F3`, 按 `F5`, 会在这边停住

`CS:466310 REPZ MOVSD` , 但在这附近的程式码很奇怪 , 所以不要在这边动手脚

那要在哪里呢 ? 先离开 `ASPACK` , 再用 `symbol loader` 载入 , 下 `G 4664D7`

`CS:4664D7 JMP 4664DA`

就改这边好了 , 把他改成 `JMP 47806C`, 机械码是 `E9901B0100`

跳到了 `CS:47806C` 时

改成 `MOV DWORD PTR [C1F0F3], 858F59E9`

`MOV BYTE PTR [C1F0F7], FF`

`PUSH C1F001`

`RET`

机械码是

```
C705F3F0C100E9598F85
C605F7F0C100FF
6801F0C100
C3
```

再用 `ULTRAEDIT` 在正确的位址填入这些值

### 20:
再来就是 `4664D7`, 这个地方如何变成 `JMP 47806C` 呢 ?

用 `symbol loader` 载入 `ASPACK`, 到了

`CS:4660AD REPZ MOVSB`

;
这行会改变一些程式码

`CS:4660AF MOV EAX, [EBP+44D284]`

`CS:4660B5 PUSH 000080000`

;
这行似乎不错 , 就改这个吧

到了 `CS:4660B5` 时

下 `A` , 下 `JMP 478083` , 机械码是 `E9C91F0100`

到了 `CS:478083` 时

下 `A`, 下

`PUSH 00008000`

按两下 `ENTER`

当然 , 和前面步骤一样 ,

所以下 `E 478083` , 游标跑到上面去了 , 改记忆体的值 , 改成

`6800800000` , 这样才对 ,

`MOV DWORD PTR [4664D7],011B90E9`

`MOV BYTE PTR [4664DB],00`

`JMP 4660BA`

这些机械码是

```
6800800000
C705D7644600E9901B01
C605DB64460000
E91CE0FEFF
```

用 `ULTRAEDIT` 在正确对应的地方输入这些值

### 21:
用 `Offset Converter` 开启 `ASPACK.EXE` 输入 `4660B5`

跑出 `26AB5`

所以用 `ULTRA EDIT` 开起 `ASPACK.EXE`, 在 `26AB5`

改成 `E9C91F0100`

改完了 ...

执行一下 `ASPACK`, 没有日期限制 , 也没有侦测 `SOFTICE` 了

这就是用 `SMC(self modifyiing code)` 的技巧 , 不错吧

只不过有点烦 , 没办法 , 因为 `aspack.exe` 本身一直改自己程式码

所以我才要改那么多东西 , 因为 `aspack` 很复杂 ,

所以我也改的很复杂 , 若能直接脱壳 , 就不必用到 `SMC` 了

所以只要 `SMC` 用的好 , 加壳的软体根本不怕
