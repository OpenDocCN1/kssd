
```
0040534B   |.  4A                |dec edx
0040534C   |.  895424 14         |mov dword ptr ss:[esp+14],edx
00405350   |.^ 75 DF            |\jnz short aris32_u.00405331
00405352   |.  99               |cdq
00405353   |.  BF BB000000      |mov edi,0BB
00405358   |.  F7FF            |idiv edi
0040535A   |.  41               |inc ecx
0040535B   |.  83F9 08         |cmp ecx,8
0040535E   |.  8816            |mov byte ptr ds:[esi],dl   //mod 0BB，保存余数
00405360   |.  C60429 00       |mov byte ptr ds:[ecx+ebp],0
00405364   |.^ 7C B0           \jl short aris32_u.00405316
00405366   |.  5F              pop edi
00405367   |.  5E              pop esi
00405368   |.  5D              pop ebp
00405369   |.  5B              pop ebx
0040536A   \.  C3              retn
```

继续往下看：

```
0040514B   |.  80F9 08         cmp cl,8                              ; cl 是用户名长度
0040514E   |.  884C24 10       mov byte ptr ss:[esp+10],cl
00405152   |.  73 30           jnb short aris32_u.00405184           ;长度<8 时，补到 8 位
00405154   |.  8B5424 10       mov edx,dword ptr ss:[esp+10]
00405158   |.  BF C0C34000     mov edi,aris32_u.0040C3C0            ; ASCII "winndy"
0040515D   |.  81E2 FF000000   and edx,0FF
00405163   |.  83C9 FF         or ecx,FFFFFFFF
00405166   |.  81C2 5EB74000   add edx,aris32_u.0040B75E            ; ASCII "winndy"
0040516C   |.  F2:AE          repne scas byte ptr es:[edi]
0040516E   |.  F7D1           not ecx
00405170   |.  2BF9           sub edi,ecx
00405172   |.  8BC1           mov eax,ecx
00405174   |.  8BF7           mov esi,edi
00405176   |.  8BFA           mov edi,edx
00405178   |.  C1E9 02        shr ecx,2
0040517B   |.  F3:A5          rep movs dword ptr es:[edi],dword ptr es:[esi]
0040517D   |.  8BC8           mov ecx,eax
0040517F   |.  83E1 03        and ecx,3
00405182   |.  F3:A4          rep movs byte ptr es:[edi],byte ptr es:[esi]
00405184   |> C605 66B74000 0 mov byte ptr ds:[40B766],0           ；用户名长度补足 8 位了
0040518B   |.  B9 54B74000     mov ecx,aris32_u.0040B754            ；ecx 指向注册码变换二来的 8 个 bytes
00405190   |.  BE 08000000     mov esi,8
00405195   |> 8A01            /mov al,byte ptr ds:[ecx]
00405197   |.  3C 20          |cmp al,20
00405199   |.  73 0E          |jnb short aris32_u.004051A9
0040519B   |.  33D2           |xor edx,edx                    ；标志直接清 0
0040519D   |.  25 FF000000    |and eax,0FF
004051A2   |.  8A51 0A        |mov dl,byte ptr ds:[ecx+A]     ；ecx+A 指向用户名
004051A5   |.  0BD0           |or edx,eax
004051A7   |.  EB 0C          |jmp short aris32_u.004051B5
004051A9   |> 33D2           |xor edx,edx                 ；
004051AB   |.  25 FF000000    |and eax,0FF
004051B0   |.  8A51 0A        |mov dl,byte ptr ds:[ecx+A]
004051B3   |.  33D0           |xor edx,eax                   

; 比较，如果 edx=eax，会置 edx 为零
004051B5   |> 03EA           |add ebp,edx                   

; ebp 保存比较结果，如果每次 edx 都为 0，则 ebp 最终也为 0
004051B7   |.  41             |inc ecx
004051B8   |.  4E             |dec esi
004051B9   |.^ 75 DA         \jnz short aris32_u.00405195
004051BB   |.  33C0           xor eax,eax
004051BD   |.  5F             pop edi
004051BE   |.  85ED           test ebp,ebp                     ; ebp 为 0 吗？
004051C0   |.  5E             pop esi
004051C1   |.  5D             pop ebp
004051C2   |.  0F94C0         sete al        ；置注册码比较成功标志
004051C5   |.  5B             pop ebx
004051C6   |.  59             pop ecx
004051C7   \.  C3             retn
```

至此，整个算法跟踪过来了。在下面一节里，总结一下算法。

### 2.3 `algorithm` `summary`

回顾一下验证过程：

1. 注册码长度大于等于 16
2. 用户名长度大于等于 5，不足 8 位的取用户名的前几位将其补足到 8 位。
3. 忽略注册码的第 9 位，第 9 位定为 `-`。
4. 依次取注册码的两位，保存为一个 `byte`，为方便起见，设注册码为十六进制字符 (当然也可一为非十六进制字符，因为程序没有去验证)，且注册码长度为 17，格式为：`XXXXXXXX-XXXXXXXX`。
5. 将 8 个 `bytes` 一次与 `50h`，…`57h` 异或。
6. 取出一个 `byte`，记为 `m`，`n=n×m`，`n` 的初值为 1，如果 `n` 大于 `8899H`，则 `mod` `8899H`，余数赋给 `n`。这样循环其次 7 次。
7. 将 `byte` `Mod` `0BBh`，保存余数。
8. 将这最终的 8 个 `bytes` 与用户名相比较，如果相等，则注册成功。

在下面的两节里，将用两种不同的思路来做出注册机来。

### 2.4 `write` `a` `keygen` `:Searching` `Table`

如何求出上面算法的逆运算呢？

倒着推导。

给定一个用户名，如 `winndy`，取一个 `byte`，难以求得注册码相应的一个 `byte`。因为第 6 步不好求逆。但大家不要忘了第 6 步取的是一个 `byte`，最多也只有 256 个，这样就可以采用穷举法：

对于从 0 到 255 的这些 `byte`，按照第 6 和第 7 步，将每个 `byte` 所对应的 `byte` 算出来，这样可以得到一个对应的表格，然后根据用户名的 `byte` 做为 `index`，查找表格就可以得到注册码的 `byte`，然后再与相应的 `50h`，..，`57h` 异或，即可得到注册码。

思路非常简单。

我用 `VB` 编了简单的程序，制作了表格，写出了注册机。详见源码。

注册机中加入了 `xm` 音乐，为此，花了我不少时间，最后找到 `uFMOD`，`Excellent`！

### 2.5 `write` `a` `keygen` `:RSA`

如果你了解 `RSA`，则可看出，第 6 步实际上是 `RSA` 运算。

参考了 `ArchFire/ATA` @2002-12-14 在看雪的文章：《`Api32` `keygen`: `learn` `how` `to` `use` `RSA`》。

2.4 节的注册机是我自己做的，后来参考看雪精华，才发现这篇好文章。在此之前，我未接触过用 `RSA` 的程序，也难怪啊！

废话打住，下面介绍 `RSA8`：

第 6 步：`C=((M⁷) mod 8899) mod 0bb=(M⁷) mod 0bb`, 因为 `8899 mod 0bb=0`

因此大数 `n=0BBh`。将 `n` 分解为两个素数的积，`n=11*0B,p=11h,q=0Bh,`

`f=(p-1)*(q-1)= 10*0a\=0a0`,从程序知 `e=7`，下面就找 `d`，使得：`(d*e) mod f=1`。

即为求 `e` 的模逆 (mod `f`). 这需要有数论的知识，

下面给出一个例子，手工用辗转相除法求模逆，实际中，用 `te!` 的 `RSA-Tool` 可以计算出 `d`，附录中还给出其它相关资料： ![]

好了，本例中，我们的 `d` 为：`d=17h`。有兴趣的可以去算一下，加深理解。

于是，由用户名求注册码的过程即为：

`M=(C¹⁷) mod 0BBh`

详见注册机源码，与穷举法比较一下，好好体会。

### 2.6 `Some` `Questions` `and` `Assignment`

1. 程序中并没有对输入的 `byte` 作要求，实际 `RSA` 算法应使 `C` 小于 `0BBh`，因此我们的注册码不是唯一的，这是我通过穷举法发现的。给出一个例子：

`Winnhijk`

`7B6F1011-4E6B5B6A` ＝＝》 `RSA` 算法求得

`B6A8AFAE-81AC9EAF`

`B6A8AFAE-81AC9E6A`

`B6A810AE-81AC9E6A`

我们在求得 `M` 后，`M` 是小于 `0BBh` 的，事实上如果我们将 `M` 加上 `0BBh`，如果 `M` 仍小于 256，即仍为一个 `byte`，继续参加后面的 `xor` 运算，得到的注册码仍是正确的。

在方法 1，由穷举得到的表格中可以很明显地看出来。

2. 根据验证过程，注册码长度大于等于 16 都是可以的，而且去掉第 9 个字符后的注册码个数也不一定为偶数，注册码也不一定要为 16 进制字符，试想一下：

考虑把最后两个字符换成一个非 16 进制字符，比如 `z` (ascii 为 `7Ah`)，`7Ah+C9h=143`,由 `z` 求得的 `al` 等于多少？`z` 代替哪两个 16 进制字符？请下去自己尝试一下。（答案是 `F0`）

`F0 xor 57＝A7` ，`A7――>28`（()

`Winnhij(`

`7B6F1011-4E6B5BF0`  `7B6F1011-4E6B5Bz`

反之，给出两个十六进制字符，其对应的非十六进制字符存在吗？如果存在，是多少？

3. 请用其他编程语言，写出注册机。

4. 注册码保存在注册表中：`HKEY_LOCAL_MACHINE\SOFTWARE\APIS32`

上面是 `RSA8`，加密的明文是 8 `bytes`，下面来看一个 `RSA256`，加密的数是 256 个 `bits`，看作一个 256 进制的数。在内存中有 32 个 `bytes`，高位在后，低位在前，这一点务必要清楚。

## 3. `Make` `a` `keygen` `for` `WMV` `to` `AVI` `MPEG` `DVD` `WMV` `Converter` `v1.4.8`

### 3.1 `Tools` `used`

【`Dynamic` `Debugger`】 `OllyDBG` `v1.10` `fly` 修改版

【`Other`】 `RSA-Tool` `2` `v17` `by` `tE!` \[TMG\]

PPSIQS `V1.1` `by` `Satoshi` `Tomabechi` `2001`

PPSIQS `V1.1` `download` `URL`： `http://www.asahi-net.or.jp/~KC2H-MSM/cn/`

### 3.2 `Trace` `registration` `code` `checking` `routine`

载入 `WMV` `to` `AVI` `MPEG` `DVD` `WMV` `Converter.exe`，这个程序是没有加壳的。

首先输入 `winndy`， `1234567890`，看看提示信息。

![](img/f8fae0fe935bf9c902cac0e4fd442b0f.jpg)

右键，搜速，字符串：

![](img/d78d6365818d1e2b49a562d109c0be79.jpg)

![](img/d61f44d8ea8e9a40be5012f77ddad487.jpg)

看到错误提示字符串了吧？老规矩，下断。

![](img/a63f22bde09c8ef6e4729b74d2a59c68.jpg)

注意 `0041CA63` 处的 `jnz`，前面 `0041CA59` 处的 `call` 就是注册码验证关键部分。

在 `0041CA4F` 处下断，然后 `F7` 跟进 `call` `00401AA5`。

![](img/e486e8499ca7ea44de2ea7086a853f44.jpg)

同理，上面也下断。

输入假注册码后，停留在 `0042E311`。

`F7` 进入 `00401AA5`：

```
00401AA5    $ /E9 76A70100     jmp WMV_to_A.0041C220
0041C220    > \6A F           >push -1
0041C222    .  68 1          >push WMV_to_A.00431D19             ;  SE handler installation
0041C227    .  64:A        >mov eax,dword ptr fs:[0]
0041C22D    .  50        push eax
0041C22E    .  64:8       >mov dword ptr fs:[0],esp
0041C235    .  81EC      >sub esp,94
0041C23B    .  8B84     >mov eax,dword ptr ss:[esp+A4]
0041C242    .  53        push ebx
0041C243    .  56        push esi
0041C244    .  50        push eax
0041C245    .  8D4C     >lea ecx,dword ptr ss:[esp+10]
0041C249    .  C744     >mov dword ptr ss:[esp+60],CD49046B
0041C251    .  C744     >mov dword ptr ss:[esp+64],829A80CB
0041C259    .  C744     >mov dword ptr ss:[esp+68],3F5157C0
0041C261    .  C744     >mov dword ptr ss:[esp+6C],B50C6384
0041C269    .  C744     >mov dword ptr ss:[esp+70],AA56D550
0041C271    .  C744     >mov dword ptr ss:[esp+74],B05ADF71
0041C279    .  C744     >mov dword ptr ss:[esp+78],7B2E3CD4
0041C281    .  C744     >mov dword ptr ss:[esp+7C],CFB69AC3
```

注意上面这些 `dword`，一共 256 个 `bits`，组成大数 `n`。

`D` `esp+60`

下面是内存中的数据：

```
0012C 634  6B 04 49 CD CB 80 9A 82  k 

* * *

I 退€殏 

0012C63C  C0 57 51 3F 84 63 0C B5  繵 Q?刢.
0012C644  50 D5 56 AA 71 DF 5A B0  P 誚猶遉
0012C64C  D4 3C 2E 7B C3 9A B6 CF  ?.{脷断
```

`RSA256`，这是 256 进制数，低位在前，高位在后，

`N＝CFB69AC37B2E3CD4B05ADF71AA56D550B50C63843F5157C0829A80CBCD49046B`

利用 `RSA－Tool` 将其转化为十进制数：
