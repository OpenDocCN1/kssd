
# 破解 ecc 保护的 flexlm (翻译 by nujia)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-276.htm`](https://www.pediy.com/kssd/pediy09/pediy09-276.htm)

说句老实话，我很早就知道这个缺陷，：），但是因为 `RNG`（random number generator，随机数发生器）做得足够很安全了，所以现在发表这个缺陷，将不会损害 `flexlm` 使用者的利益（除非他们没有更新 `RNG`），`Crackz`。

## 简介

本文主要描述了怎样获得 `ecc flexlm` 保护的 `seed3` 和 `seed4`。虽然本文是是讲述方法的，但是如果你破解一个真正的软件的话，下面这些软件将是必须的。

*   `NuMega SoftICE`
*   `Datarescue IDA` (4.15 或更新的版本)
*   `FLEXlm v7.2h SDK`
*   `Microsoft Visual C++`
*   `TASM` 假如你想分析某些代码的话

## 怎样开始工作呢？

标准的老的 `flexlm` 版本用两个 32 位的加密种子（`seed1` 和 `seed2`）来加密软件。这两个种子提供全部的加密保护。种子会在目标程序中出现，所以程序可以制作注册机，编译得到 `lmcrypt`（`flexlm key 生成器`），这样 `cracker` 就能生成有效的 license 了。`ecc flexlm` 需要两个额外的加密种子（`seed3` 和 `seed4`）。这两个种子用来生成公钥/私钥对，并且只有公钥存在于发布的软件中。`Certicom Security Builder ECC` 库用来生成公钥/私钥对，看起来比较安全。我没有发现用来生成公钥/私钥对的 `seed3` 和 `seed4` 与公钥之间有什么关系（就是由公钥是不能推导出 `seed3` 和 `seed4` 的）。为了暴力破解 `seed3` 和 `seed4`，很明显需要 2⁶⁴ 次尝试。这是不太现实的。

`Globetrotter` 在使用 `seed3` 和 `seed4` 时犯了一个错误。虽然 `seed3` 和 `seed4` 不能像 `seed1` 和 `seed2` 那样完全的复原，但是也为缩短暴力破解的时间提供了足够的信息，这样能在可以接受的时间内暴力破解出 `seed3` 和 `seed4`。

缺陷出现在 `_do_handshake` 函数里。这个函数用 `seed3` 和 `seed4` 来验证 `vendor daemon` 的真假。下面是使用 `seed3` 和 `seed4` 的部分代码。

```
        mov  edx,  dword ptr ds:_seed3         ;将 seed3 放入 edx
    xor  edx,  dword ptr ds:_seed4         ;将 seed3 和 seed4 取异或，结果在 edx
    shr  edx, 10h                          ;edx 右移 16 位，
    and  edx, 0FFFFh                      ;将 edx 的低 16 位填充 ffff
    push  edx                              ;将 edx 压栈
    mov  eax,  dword ptr ds:_seed3         ;将 seed3 放入 eax
    add  eax,  dword ptr ds:_seed4         ;将 seed3 和 seed4 相加，结果在 eax
    and  eax, 0FFFFh                      ;将 eax 的低 16 位填充 ffff
    push  eax                              ;将 eax 压栈
    mov  ecx,  dword ptr ds:_seed3         ;将 seed3 放入 ecx
    xor  ecx,  dword ptr ds:_seed4         ;将 seed3 和 seed4 取异或，结果在 ecx
    and  ecx, 0FFFFh                      ;将 ecx 的低 16 位填充 ffff
    push  ecx                              ;将 ecx 压栈
    call  _srand16                         ;调用随机函数_srand16，三个输入，因为压栈了三个数
    add  esp, 0Ch                          ;函数外调整堆栈，
    call  _rand16                          ;四次调用另外一个随机函数_rand16
    call  _rand16                          ;返回结果在 eax 里
    call  _rand16                          ;
    call  _rand16                          ;
    mov  [ebp+var_60], eax ; "random" numbers are used as constants in lm_new :)
                                                ;将返回值送入[ebp+var_60]，这个随机数在 lm_new 保持常数，
                                                ;于是缺陷就产生了，:)
    call  _rand16                          ;调用随机函数_rand16，无输入，因为没有压栈
    mov  [ebp+var_5C], eax                 ;将返回结果送入[ebp+var_5C]
    call  _rand16                          ;调用随机函数_rand16
    mov  [ebp+var_58], eax                 ;将返回结果送入[ebp+var_58]
    call  _rand16                          ;调用随机函数_rand16
    mov  [ebp+var_54], eax                 ;将返回结果送入[ebp+var_54]
```

假如我们可以复原传递给函数 `srand16` 的 `seed` 值，那我们就知道了很多关于 `seed3` 和 `seed4` 的信息了。仅凭一个由 `rand16` 产生的值，我们很难复原伪随机数发生器的初始状态（译者注：也就是由返回的 `eax` 的值确定随机函数 `rand1` 的输入值是不可能的，否则就不叫随机数发生器了，：））。为了复原传递给随机函数的输入值，我们需要仔细检查 `rand16` 函数和 `srand16` 函数。

```c
/*------------------------------------------------------------------------*
 *
 * my_srand16: emulate the srand16 routine.  ;自己写的 my_srand16 函数，模仿 srand16 函数
 * seeds: seed values.
 *------------------------------------------------------------------------*/
int my_srand16(int seed1, int seed2, int seed3)
{
  sdata_0 = seed1 % 32363;            //32363 就是 7E6B
  sdata_1 = seed2 % 31727;            //31727 就是 7BEE
  sdata_2 = seed3 % 31657;            //31657 就是 7BA8，这三个数应该都是质数。

  if (sdata_0 == 0) sdata_0 += 32362;
  if (sdata_1 == 0) sdata_1 += 31726;
  if (sdata_2 == 0) sdata_2 += 31656;
  return(0);
}

/*------------------------------------------------------------------------*
 *
 * my_rand16: emulate the rand16 routine.    ;自己写的 my_rand16 函数，模仿 rand16 函数
 *------------------------------------------------------------------------*/
int my_rand16()
{

  int var_8;
  int var_4;
  int edxval;
  int eaxval;

  var_4 = sdata_0 / 206;
  edxval = (sdata_0 / 206) * 206;
  eaxval = sdata_0 - edxval;
  sdata_0 = eaxval * 157 - (var_4 * 21);
  if (sdata_0 <= 0)
  {
    sdata_0 += 32363;
  }

  var_4 = sdata_1 / 217;
  edxval = var_4 * 217;
  eaxval = sdata_1 - edxval;
  sdata_1 = eaxval * 146 - (var_4 * 45);
  if (sdata_1 <= 0)
  {
    sdata_1 += 31727;
  }

  var_4 = sdata_2 / 222;
  edxval = var_4 * 222;
  eaxval = sdata_2 - edxval;
  sdata_2 = eaxval * 142 - (var_4 * 133);
  if (sdata_2 <= 0)
  {
    sdata_2 += 31657;
  }
  var_8 = sdata_0 - sdata_1;
  if (var_8 > 706)
  {
    var_8 = var_8 - 32362;
  }
  var_8 = var_8 + sdata_2;
  if (var_8 <= 0)
  {
    var_8 += 32362;
  }
  return(var_8 >> 1);
}
```

尽管看起来比较乱，但是它确实是起到了伪随机数发生器的功能。这个 `rand16` 函数的算法和 `Bruce Schneier` 编的《实用密码学》（`Applied Cryptography`）里提到的线性同余发生器的例子相同。（译者注：线性同余发生器是最常见的随机数发生器方法之一，比如 1/7=0.142857142857....,当分母 7 换成一个足够大的质数 p 的时候，它的商的不同位看起来就像随机产生的）在这本书里作者提到了这个算法不具有加密安全性，并且给出怎样破解这个随机数发生器。这个随机数发生器由三个发生器组成，每个发生器产生不同长度的随机数。当每个发生器初始化后，每产生一个随机数，发生器就变换到下一个状态。给定一个输出和两个发生器的初始状态，第三个发生器的状态就可以确定了。

第一个随机数可以从目标程序中获得，而且 `RNG` 开始两个 `LCG` 是循环码，第三个 `LCG` 从目标程序获得。假如随机数序列匹配超过四个值，我们就确定了 `RNG` 的状态了。三个 `LCG` 的全部的循环都在一个巨大的数组了，所以我们可以获得 `RNG` 的状态，而且无需任何计算，只要一个指向 `LCG` 的指针即可。一旦确定 `RNG` 状态的那四个值出现了，`LCG` 就一步步回退，直到找到传递给 `PRNG` 的那个初始值。因为数据在给 `LCG` 时已经经过取模运算，这样肯定会有不同的值经过取模运算后得到相同的值传递给 `LCG`。所有有效的排列都循环传递给 `srand` 函数。经过这些工作后，我们知道关于 `seed` 的如下信息。

高 16 位 `seed3 ^ seed4 == value1`
低 16 位 `seed3 + seed4 & 0xffff == value2`
低 16 位 `seed3 ^ seed4 == value3`

(译者注：这个结论是上面提到的 `_do_handshake` 函数的部分代码的功能)

知道这些后，根据 `value1`、`value2`、`value3` 的值，暴力破解可以减少到大约 2³⁰ 次，这样暴力破解才是可行的。在 `flexlm` 目标程序里有三个 `ecc` 公钥。选择那个最小的公钥，因为从 `seed3` 和 `seed4` 算公钥/私钥对时，最快算到这个值。`seed` 值通过 `vendor` 名被最低限度的“模糊”了。

```c
int reveal_data(sb_keydata_t *inkey, char *inname)
{
        int i;
        char *p;

        p = inname;
        for (i = 0; i < inkey->nbytes; i++)
        {
                if (*p == '\0') p = inname;
                if (i & 0x0000001)
                {
                        if (i%3)
                        {
                                inkey->keydata[i] = inkey->keydata[i] + *p;
                        }
                        else
                        {
                                inkey->keydata[i] = inkey->keydata[i] ^ *p;
                        }
                }
                else
                {
                        inkey->keydata[i] = inkey->keydata[i] - *p;
                }
                p++;
        }
        return(0);
}
```

一旦真正的公钥恢复了，我们可以用程序循环计算出可能的 `seed3` 和 `seed4`，就说道这儿。我还要简单提一下 `seed1` 和 `seed2`，他们仍然用相同的方法隐藏在相同的地方，所以前面提到通过 `job` 结构和 `vendorcode` 结构找 `seed1` 和 `seed2` 的方法依然有效。

`Nolan Blender`.

## 附 do_handshake 函数

```c
static void do_handshake(char *charptrname, char *idxname, char *switchname)
{
  char charname[MAXVARNAME];
  int i[4];
  char zeroname[MAXVARNAME];

  randvarname(charname, "charname");
  randvarname(zeroname, "zero");
}
```
