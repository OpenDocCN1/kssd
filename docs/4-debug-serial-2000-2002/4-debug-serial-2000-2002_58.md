
# Green Tea 2.60

> 原文：[`www.pediy.com/kssd/pediy02/forum141.htm`](https://www.pediy.com/kssd/pediy02/forum141.htm)

## Green Tea 2.60 注册码算法

文章作者：`wind[CCG]`  
作者主页：`http://biggow.8u8.com`  
作者 E-mail: `biggow@eastday.com`

欢迎转载，但请保持原文完整性！

看完此文，希望大家可以写出注册机，应该不是很难！

```
:0046A029 E8EE9BF9FF          call 00403C1C
:0046A02E 8BD0                mov edx, eax
:0046A030 85D2                test edx, edx
:0046A032 7E14                jle 0046A048
:0046A034 B801000000          mov eax, 00000001
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:0046A046(C)
```

```
:0046A039 8B4DF0              mov ecx, dword ptr [ebp-10]
:0046A03C 0FB64C01FF          movzx ecx, byte ptr [ecx+eax-01]
:0046A041 0FAFD9              imul ebx, ecx
:0046A044 40                  inc eax
:0046A045 4A                  dec edx
:0046A046 75F1                jne 0046A039
```

上面这段程序，是将你输入的注册名，把注册名每个字符的 `Ascii` 码算出。

然后，将每个字符的 `Ascii` 码相乘，得到一个积（在这里给这个乘积取一个变量名 `S`）。

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
```
:0046A032(C)
```

```
:0046A048 83C364              add ebx, 00000064------->就是把乘积 S 加上 64,
                              假设 A=S+64（这里都是十六进制数）
:0046A04B 8BC3                mov eax, ebx-------->把算出的 A 移入寄存器 eax 中！
:0046A04D 99                  cdq-------->判断 eax 中的值是否大于 80000000,如果大于 80000000 那么
                              edx=ffffffff,反之，edx=00000000
:0046A04E 33C2                xor eax, edx------>把 eax 和 edx 做异或操作，也就是把 A 变量的值和
                              ffffffff 或 00000000 作异或操作
                              如果 edx=00000000 的话,异或后 eax 的值保持
                              不变!
:0046A050 2BC2                sub eax, edx-------->eax=eax-edx,也就是把异或出来的
                              值去减 FFFFFFFF 或 00000000
                              如果 edx=00000000 的话,eax 的值保持不变
:0046A052 8BF8                mov edi, eax
:0046A054 8BC7                mov eax, edi
:0046A056 8D951CFEFFFF        lea edx, dword ptr [ebp+FFFFFE1C]
:0046A05C E813D0F9FF          call 00407074
:0046A061 8B951CFEFFFF        mov edx, dword ptr [ebp+FFFFFE1C]
:0046A067 8B86F8010000        mov eax, dword ptr [esi+000001F8]
```

## 算法总结

这个软件的注册部分就是把你输入的注册名的每个字符都转成相应的 `Ascii` 码！

然后，把每个字符的 `Ascii` 码相乘，得到这个乘积 `S`，再把它加上 `64`，然后把这个值放到 `eax` 中，然后用 `cdq` 这个指令来判断 `eax` 的值是否大于 `80000000`，如果大于，那么 `edx=ffffffff`，否则，`edx=00000000`。

然后，把 `eax` 也就是 `A` 和 `00000000` 或 `ffffffff` 做异或，把异或出来的值放到 `eax` 里去！

再用 `eax` 的值去减去了 `edx` 也就是 `00000000` 或 `ffffffff`！

执行 `cdq` 这个指令的时候，`eax` 小于 `80000000`，那么 `edx=00000000`，那么这个 `A` 转换成十进制数后就是注册码！

为了说明清楚，就举个简单的例子:

我要的注册名：`wind`

要转换的字符：`w` `i` `n` `d`

`Ascii`：`77` `69` `6E` `64`

`chengji = 77 * 69 * 6E * 64`；用计算器计算，得到 `chengji = 8313E88`

`zonghe = chengji + 64`；计算后得到 `zonghe = 8313EEC`

由于 `8313EEC` 小于 `80000000`，所以，`edx = 00000000`

由于和 `00000000` 异或（后面的 `sub eax,edx`，一个数减 `0` 当然不变啦！），所以，`zonghe` 不变，仍然等于 `8313EEC`

将 `8313EEC` 转换成十进制数就是注册码

`8313EEC` 转成十进制为 `137445100`

最后整理出：

注册名: `wind` 注册码：`137445100`

如果 `zonghe` 这个变量大于 `80000000` 的话，`edx=ffffffff`，就有一个异或过程，也就是说异或后的值会改变！

还有最后要加上 `1`（因为 `sub eax,edx` 码！）

`xor`（异或）: 也就是必须两个相反的数才会得 `1`，只有 `0` 和 `1` 出现的时候，结果才会为 `1`，如果是 `0` 和 `0` 或者 `1` 和 `1`，这个结果只能是 `0`！

很久没有写东西了，希望大家对我粗糙的文笔表示谅解，也希望大家能够明白我的意思！

如果你读了这篇文章之后，写出了注册机，别忘了，提一下我的名字: `wind[CCG]`

以及我的网站: `biggow.8u8.com`！宣传一下！

`wind`  
`China Cracking Group`  
`2000.7.16`  
`4dd466d0e1e0cc123afbe504da2462ca`
