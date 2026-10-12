
# ASProtect 的加密算法初步分析

> 原文：[`www.pediy.com/kssd/pediy05/pediy50158.htm`](https://www.pediy.com/kssd/pediy05/pediy50158.htm)

看世界杯，漫步 `ASProtect` ----- `ASProtect` 的加密算法初步分析

近来有时间分析了一下 `ASProtect`，略有所得，也有不少疑问。

原来 `ASProtect` 的几个函数的作用对保护强度提高不小。一改以前我见过的壳重技巧轻算法的毛病，的确在有些方面值得学习。

其利用的原理如下：

1) 程序在编程时可以根据需要选择使用 `ASProtect` 提供的几个函数。

   比如：

   `SetRegistrationKey(...)`,
   `GetRegistrationInformation(...)`,
   `GetHardwareID(...)`

   这几个函数由于使用了导出方式的声明所以会在 `ExportTable` 中出现，`ASProtect` 在加壳时可以通过分析 `ExportTable` 识别出程序使用了哪些其提供的函数，从而对这些函数的入口进行挂钩，从而使其加壳后的外壳代码可以象回调函数一样调用这些函数与程序进行通信。

   比如见以下代码：

```
0041333E loc_41333E:                               ; CODE XREF: sub_413253+B0j

0041333E                  mov     eax, ds:4155CCh
00413343                  cmp     dword ptr [eax+0Ch], 0
00413347                  jz      short loc_413360
00413349                  mov     eax, ds:41555Ch
0041334E                  mov     eax, [eax]
00413350                  call    @System@@LStrToPChar$qqrv ; System __linkproc__ LStrToPChar(void)
00413355                  push    eax
00413356                  mov     eax, ds:4155CCh
0041335B                  mov     eax, [eax+0Ch]
0041335E                  call    eax

00413360

00413360 loc_413360:                               ; CODE XREF: sub_413253+F4j

00413360                  mov     eax, ds:4155CCh
00413365                  cmp     dword ptr [eax+30h], 0     ; eax 是一内部结构指针
00413369                  jz      short loc_41389
0041336B                  push    41105Ch
00413370                  mov     eax, ds:4155CCh
00413375                  mov     eax, [eax+30h]
00413378                  call    eax                 ; 这里将调用 GetDecryptProc(...)
0041337A                  push    41105Ch

0041337F                  mov     eax, ds:4155CCh
```

2) 程序在编程时还可以加入 `ASProtect` 提供的两个宏定义，这两个宏定义实际上将起到特殊的标志作用，有点类似"块开始"和"块结束"的作用。在 `ASProtect` 将在加壳时将对"这些块"中的代码进行加密。对这些代码块的加、解密使用了的密码学算法。目前还不十分清楚其使用的算法，初步分析这里可能用到了 `TWOFISH`。因为发现了一些特征：

   初始化部分有以下常数设置：

```
0040C4AE                  mov     dword ptr [ebx+48h], 67452301h
0040C4B5                  mov     dword ptr [ebx+4Ch], 0EFCDAB89h
0040C4BC                  mov     dword ptr [ebx+50h], 98BADCFEh
0040C4C3                  mov     dword ptr [ebx+54h], 10325476h
0040C4CA                  mov     dword ptr [ebx+58h], 0C3D2E1F0h
0040C4D1                  mov     dword ptr [ebx+5Ch], 76543210h
0040C4D8                  mov     dword ptr [ebx+60h], 0FEDCBA98h
0040C4DF                  mov     dword ptr [ebx+64h], 89ABCDEFh
0040C4E6                  mov     dword ptr [ebx+68h], 1234567h
0040C4ED                  mov     dword ptr [ebx+6Ch], 3C2D1E0Fh
```

   而运算部分有大量类似 `MD5` 算法的运算。由于代码太长，不列举了。

3) 在 `ASProtect` 中我们看到其提供了 `Registration_Keys` 加密方式。`ASProtect` 可以对你提供的"`Registration_Information`"进行加密，形成"`Registeration_Key`"，(按照帮助中介绍的你还可以使用自己的"加解密算法"，但是 `RegKey` 必须至少 173 个字符，才有安全性)。这个"`RegKey`"将以注册表文件(`.reg`)文件方式保存。在加壳后的外壳代码运行时将解密"`RegKey`"从而得到"`RegInfo`"，当然这分为两种方式：一种方式，程序编程时使用了其提供的回调函数 `SetRegistrationKey(...)`，那么外壳代码运行时将调用该函数从而得到"`RegKey`"来解密出"`RegInfo`"，如果程序没有使用这个函数，外壳代码将从注册表中相应键名处提取键值作为"`RegKey`"来解密出"`RegInfo`"。初步分析其可能使用了 `RSA` 算法。

   其解密部分的代码：

```
0040B4D4 loc_40B4D4:                               ; CODE XREF: license+156j

0040B4D4                                           ; license+15Fj

0040B4D4                  lea     edi, [ebp+lpInterBlock]
0040B4DA                  inc     edi
0040B4DB                  mov     esi, edi
0040B4DD                  push    esi
0040B4DE                  lea     eax, [ebp+lpInterStruct+8]
0040B4E4                  push    eax
0040B4E5                  lea     eax, [ebp+lpInterStruct+88h]
0040B4EB                  push    eax
0040B4EC                  push    esi
0040B4ED                  call    decrypt_engine  ; Criper Text to Plain Text...
```

   从 `decrypt_engine` 函数的入口参数看有四个：

   1) 加密数据地址指针....(函数按每一块进行解密).....即 `m`

   2) 可能是 `RSA.N`

   3) 可能是 `RSA.E`

   4) 用于输出解密数据地址指针....

   目前还有几个问题还没有来得及分析：

   1) 还没有见到 `TEA` 算法，不过在 `CASPR` 中介绍中至少提到这个算法，不知道这个算法在哪里，用于作什么？

   2) 在我了解了 `ASProtect` 会利用宏定义定义出"代码块"以后，我想象 `ASProtect` 中其会把"代码块"挖出来"，与"`RegInfo`"一起加密形式成"`RegKey`"。不过实际发现这些"代码块"并没有被"挖"出来，而是使用一种算法(可能是 `TWOFISH`)解密，不知道这个 `KEY` 到底保存在哪里？不知道是否保存在"`RegKey`"中。我想大概如此。^\_^

   我想我们可以把 `ASProtect` 看成一个"软锁"。"锁"中的数据包括用于解密的代码块的 `KEY` 和用于解密"`RegInfo`"的 `KEY`，即我们所见到的字符串形式的"`RegKey`"。在外壳代码运行时 `ASProtect` 将提取"锁"中的数据用于解密。

   如果某程序分为"注册版本"和"试用版本"，其使用 `ASProtect` 外壳+内部函数组合方式加密。其注册版本将提供 `.reg` 注册表文件形式的"`RegKey`"作为开锁的"钥匙"，那么我们能否在没有"钥匙"的情况下破门而入呢？

   由于 `RegKey` 中包含了用于解密程序中提取出来的"代码块"的 `KEY`，那么在没有 `RegKey` 的情况下自然我们也就缺少了这个 `KEY`，因此程序中的被提取出来的"代码块"就无法正确解密。如果在此情况下脱壳，那么那些加密了的代码块中的数据依然是无用的数据，因此脱壳的软件就成了破碎的脱壳版本了。因此在没有钥匙的情况下看来破门而入的想法不太乐观。

   3) 那么我们想能否推算出 `RegKey` 呢？目前我还不太清楚。有时间再分析了。

   好，写到此处，不想浪费时间了。不知道以后的"幻影"是否也会.....。:-)

   如果你看了我的这个贴子的话，知道我目前的几个问题，可以和我联系。

   `Email:ljtt@yeah.net`。(不要来信要我脱壳之类的) `4dd466d0e1e0cc123afbe504da2462ca`
