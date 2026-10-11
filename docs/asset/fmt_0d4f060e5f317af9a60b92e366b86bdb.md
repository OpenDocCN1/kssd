
# Sandboxie 注册算法分析

> 原文：[`www.pediy.com/kssd/pediy11/124357.html`](https://www.pediy.com/kssd/pediy11/124357.html)

**Sandboxie 注册算法分析**

【文章作者】: `uuk`

【软件名称】: `Sandboxie 3.49.06`

【下载地址】: `http://www.sandboxie.com/index.php`

【加壳方式】: 无

【保护方式】: 序列号

【编写语言】: `Microsoft Visual C++ 7.0`

【使用工具】: `PEID`，`IDA Pro` with `Hex-Rays plugin`，`Syser`

【操作平台】: `XP SP3`

【软件介绍】: `Sandboxie` 允许你在沙盘环境中运行浏览器或其他程序，因此运行所产生的变化可以随后删除。可用来消除上网、运行程序的痕迹，也可用来还原收藏夹、主页、注册表等。即使是在沙盘进程中下载的文件，也会随着沙盘的清空而删除。此软件在系统托盘中运行，如果想启动一个沙盘进程，请通过托盘图标（而不要用原方式）启动浏览器或相应程序。

【作者声明】: 菜鸟学习算法，失误之处敬请诸位大侠赐教!

`Sandboxie` 自 `3.48` 版本开始采用新的激活方式联网激活，但也为不能上网的电脑提供了离线激活方式。能离线激活说明注册的验证是在本地进行的，方便我们进行算法分析。官方离线激活网址：`http://www.sandboxie.com/index.php?OfflineActivation`。在之前我已经通过官方的离线激活网页得到一个离线激活码 `Key`（`RF0UD7XH6KREU2TX……E4YJ1PT8RW8PO`）。

通过跟踪激活过程，激活管理程序 `License.exe` 调用 `SbieDll.dll` 的 `SbieApi_ActivateLicense`，`SbieApi_ActivateLicense` 通过 `IRP_MJ_DEVICE_CONTROL` 与驱动 `SbieDrv.sys` 通信，最终验证过程在驱动 `SbieDrv.sys` 里。通过 `IoControlCode` 定位到函数 `sub_20744`（重命名为 `ActivateLicense`），查看 `ActivateLicense` 的交叉引用，发现在驱动加载过程中也被调用了，因此可以确认验证过程就是 `sub_20744`。`ActivateLicense` 反汇编代码如下：

```
.text:00020744    push    ebp
.text:00020745    mov     ebp, esp
.text:00020747    sub     esp, 28h
.text:0002074A    push    esi
.text:0002074B    push    24h     ; 36 = 24h
.text:0002074D    push    [ebp+pKey]      ; ActivationKey(UNICODE)
.text:00020750    xor     esi, esi
.text:00020752    push    pPagedPool_tzuk ; PagedPool("tzuk")
.text:00020758    mov     [ebp+var_18], esi
.text:0002075B    mov     [ebp+var_14], esi
.text:0002075E    call    sub_1C1C8     ; mpz_set_str (mpz_t rop, char *str, int base)
.text:0002075E                    ; return eax = rop
```

仔细分析 `sub_1C1C8`，会发现它是将字符串 `Key` 当作 `36` 进制数，转换成 `16` 进制的大数（有点像 `GMP` 库里的 `mpz_set_str(Key, ActivationKey, 36)`）。由于是大数操作，所以 `sub_1C1C8` 里有大量的内存操作，刚开始分析时就让我晕头转向，同时又是大数运算，比较复杂。我花了不少时间才搞明白它在做什么，也知道程序中是如何存储大数的。大数结构如下所示：

```c
struct BigNum{unsigned int size;  unsigned int num[size];}
```

大数在内存中按从低到高的地址存储从低到高的数。知道了大数的结构，以及内存申请和释放操作，后面的分析就会轻松许多。

```
.text:00020763    cmp     eax, esi        ; 大数 Key = eax
.text:00020765    mov     [ebp+pKey], eax
.text:00020768    jnz     short loc_2077E
.text:0002077E ; ---------------------------------------------------------------------------
.text:0002077E
.text:0002077E loc_2077E:                 ; CODE XREF: ActivateLicense+24 j
.text:0002077E    push    ebx
.text:0002077F    push    edi
.text:00020780    push    offset MOD
.text:00020785    push    offset EXP
.text:0002078A    push    eax
.text:0002078B    push    pPagedPool_tzuk
.text:00020791    call    sub_1C09E    ; mpz_powm (mpz_t rop, mpz_t base, mpz_t exp, mpz_t mod)
.text:00020791           ; return eax = rop
```

大致分析 `sub_1C09E`，排除内存操作后，发现里面有一些除法运算，初步确定是在做大数除法，但是在自己用大数计算器验证时发现结果总是不对，于是以为在 `sub_1C09E` 里有其他小动作。花了好几个晚上，仔仔细细的研究了里面不确定的 `CALL`，以及运算过程，最后确定 `sub_1C09E` 里面做的就是大数除法和求模运算。既然是除法，那为什么测试时结果不对呢？我又去分析了下测试过程，发现我写的一个字符串反转程序少做一次对换，导致中间的两个字符没被反转（大数在内存里是从小到大存储，而计算器中是从大到小，所以要反转）。这样浪费了几天时间，但也让我明白了大数除法的算法。确定了 `sub_1C09E` 里面的大数运算过程，写出数学表达示，化简后发现是类似 `mpz_powm(Reg, Key, EXP, MOD)` 的函数。在 `sub_1C09E` 之后，注册信息就以明码放在内存中，后面就是一些数据的校验了。想要写出注册机就得能反求 `Reg = Key^EXP mod MOD`，我求不出来，于是到论坛上求助 `http://bbs.pediy.com/showthread.php?t=123926`，`lingyu` 大侠告知这是 `RSA2048` 加密算法，在这要感谢大侠们的指点！

下面是数据校验过程：

```
.text:00020796    mov     ebx, eax        ; ebx = Reg = eax
.text:00020798    mov     eax, [ebx]
.text:0002079A    cmp     eax, 4          ; Reg 长度 size 大于 4
.text:0002079D    push    12h
.text:0002079F    pop     esi
.text:000207A0    jbe     loc_20A39
.text:000207A6    mov     cl, [ebx+4]
.text:000207A9    and     cl, 0Fh
.text:000207AC    cmp     cl, 1           ; Reg 最低位为 1
.text:000207AF    push    13h
.text:000207B1    pop     esi
.text:000207B2    jnz     loc_20A39
.text:000207B8    push    14h
.text:000207BA    pop     esi
.text:000207BB    lea     eax, ds:0FFFFFFF4h[eax*4] ; size*4-0Ch
.text:000207C2    push    eax
.text:000207C3    lea     edi, [ebx+10h]
.text:000207C6    push    edi
.text:000207C7    call    sub_1B4B8       ; CheckSum 数据校验
```

`sub_1B4B8` 是对注册信息 `Reg` 进行校验，返回两个校验值，里面有两个 `CALL`（`sub_1B392`、`sub_1B418`），看 `Hex-Rays plugin` 生成的 `C` 代码，很容易就能自己写出个 `checksum` 生成函数，代码在附件里有。也不知道是有现成的函数，还是作者杜撰的。

```
.text:000207CC    cmp     eax, [ebx+8]  ; 比较校验值 1 与[ebx+8]
.text:000207CF    jnz     loc_20A39
.text:000207D5    cmp     edx, [ebx+0Ch] ; 比较校验值 2 与[ebx+0Ch]
.text:000207D8    jnz     loc_20A39
.text:000207DE    cmp     dword ptr [edi], 80000001h ; [ebx+10h]
.text:000207E4    push    21h
.text:000207E6    pop     esi
.text:000207E7    jnz     loc_20A39
.text:000207ED    cmp     dword ptr [ebx+14h], 1 ; [ebx+14h] == 1
.text:000207F1    push    22h
.text:000207F3    pop     esi
.text:000207F4    jnz     loc_20A39
```

再后面还有到期时间与当前系统时间的比较，软件版本的比较，系统码的比较，在 `IDA` 里很容易就能看得懂，这里就不详细说明了。注册信息的结构如下所示：

![](img/801b1ad8451d61dc8e0ace238c8776f2.jpg)

关于注册机：`RSA2048` 是很难反求的，只能通过替换公钥的方法进行破解（看雪论坛精华里看到的）。由于是替换驱动 `SbieDrv.sys` 里的公钥，替换后还得修正文件的 `CheckSum`，不然驱动不能加载。附件里是我写的一个简单的 `Patch` 和 `Keygen`。

同时有个疑问：`SbieDrv.sys` 打过补丁后，其数字签名就变成无效了，既然 `RSA` 可以通过替换公钥进行攻击，那文件的数字签名可不可以伪造？

`Keygen.rar`
