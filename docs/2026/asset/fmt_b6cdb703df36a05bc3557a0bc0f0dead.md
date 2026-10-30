
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 |
|---|
| `0:000> bp EPSIMP32+2ba06` `// call deferred_exec` `0:000> g` `Breakpoint 1 hit` `eax=0018fd78` `ebx=00000000` `ecx=00291280` `edx=00000001` `esi=00425770` `edi=00000000` `eip=718fba06` `esp=0018fd54` `ebp=0018fdbc` `iopl=0` `nv up ei pl nz na po nc` `cs=0023` `ss=002b` `ds=002b` `es=002b` `fs=0053` `gs=002b` `efl=00000202` `EPSIMP32!RegisterPercentCallback+0x4604:` `718fba06 e8d8abffff call deferred_exec (718f65e3)` `0:000> dd eax L4` `// 查看传入的参数为数组` `0018fd78 00030000 00000000 0049ea98 0048f40c` `0:000> dd poi(poi(poi(poi(poi(0018fd78+c))+24))+28)` `// 查看数组中存储的内容` `0049e2c0 00000500 00000100 00495408 0048ee98` `// 数组中存放着字符串对象` `0049e2d0 12d85688 8000f194 00000020 00000100` `0049e2e0 0049dc40 0048f198 12d8568f 80000000` `0049e2f0 00490023 000007c8 00000300 00000100` `0049e300 12d856b2 8000f19c 00000026 00000100` `0049e310 0049dc60 0048f1a0 12d856b1 80000100` `0049e320 00420029 0048f1a4 00000003 00000000` `0049e330 12d856b4 80000080 0000002c 00000100` `0:000> db poi(poi(poi(poi(poi(0049e2c0+c))+24))+20) L10` `// 查看字符串的内容为 l56 cvx exec` `00495940 20 6c 35 36 20 63 76 78 20 65 78 65 63 20 00 00 l56 cvx exec ..` `0:000> g` `// 第二次执行 deferred_exec` `(5c8.144): C++ EH exception - code e06d7363 (first chance)` `(5c8.144): C++ EH exception - code e06d7363 (first chance)` `Breakpoint 1 hit` `eax=0018fd78` `ebx=00000000` `ecx=00291280` `edx=00000003` `esi=00425770` `edi=00000001` `eip=718fba06` `esp=0018fd54` `ebp=0018fdbc` `iopl=0` `nv up ei pl nz na pe nc` `cs=0023` `ss=002b` `ds=002b` `es=002b` `fs=0053` `gs=002b` `efl=00000206` `EPSIMP32!RegisterPercentCallback+0x4604:` `718fba06 e8d8abffff call EPSIMP32+0x265e3 (718f65e3)` `0:000> dd poi(poi(poi(poi(poi(0018fd78+c))+24))+28)` `// 查看数组的内容` `0049e2c0 00000500 00000100 00495438 0048eeac` `// 数组中存放着字符串对象` `0049e2d0 12d85688 8000f194 00000020 00000100` `0049e2e0 0049dc40 0048f198 12d8568f 80000000` `0049e2f0 00490023 000007c8 00000300 00000100` `0049e300 12d856b2 8000f19c 00000026 00000100` `0049e310 0049dc60 0048f1a0 12d856b1 80000100` `0049e320 00420029 0048f1a4 00000003 00000000` `0049e330 12d856b4 80008000 0000002c 00000100` `0:000> db poi(poi(poi(poi(poi(0049e2c0+c))+24))+20) L10` `// 查看字符串的内容为 l53 cvx exec` `00495958 20 6c 35 33 20 63 76 78 20 65 78 65 63 20 00 00 l53 cvx exec ..` |

从调试的结果可以得知，该函数执行的正是 `forall`。在第一次执行时，`l61` 中待执行的命令是 `l56 cvx exec`；在第二次执行时，`l61` 中的内容已经被换成了 `l53 cvx exec`，与调试结果相符。

接着深入函数分析，发现函数内部嵌套了 `deferred_exec`：

![image-20210901193647205](https://bbs.kanxue.com/upload/attach/202109/929902_ZYXM5GFFSK3Q5DC.jpg)

于是重新调试，下断在此，分析参数：

![image-20210901201303745](https://bbs.kanxue.com/upload/attach/202109/929902_PZM9KRPTATQM4KQ.jpg)

虽然 `type` 为 `0x10` 的操作符对象存储在 `Systemdict` 中无法查看，但是通过其他字符和数字还是能够确定该语句就是 `l50`。当执行该语句后，原本 `l63` 指向的 `string` 结构将被替换成存放 `l52` 内容的 `string` 结构：

![image-20210901211439785](https://bbs.kanxue.com/upload/attach/202109/929902_7VCP2PXZP5SJG5N.jpg)

可以看到此时原本存放 `l63` 的 `string` 结构已经变成了 `l52`。

在 `get` 函数下断，跳转到 `forall` 下的 `/l64 l57 56 get def` 语句查看 `l57` 的值：

![image-20210903144034585](https://bbs.kanxue.com/upload/attach/202109/929902_TJ4UR3FY597PPD5.jpg)

可以证实 `l57` 中存放的就是从 `l63` 中获取到的字符，该 `forall` 的作用就是泄露被释放的 `string` 结构指向的字符串。

接着获取 `l57` 中的值，并进行一些处理，通过 `ifelse` 判断系统位数。若 `l77` 等于 `l52` 的长度+1，那么 `l99` 的值为1，代表系统为64位；否则 `l99` 为0，代表系统为32位：

![image-20210903164816672](https://bbs.kanxue.com/upload/attach/202109/929902_YZYCM3HEXTXWP2W.jpg)

可以看到在32位的调试环境下，`l77` 的值为0，因此会将5个0压入操作栈中，并赋值给 `l95` 到 `l99`：

![image-20210903165138520](https://bbs.kanxue.com/upload/attach/202109/929902_2HCG475UQ7CNPDC.jpg)

至此，漏洞原理部分分析完毕，接下来分析漏洞利用部分。

## 漏洞利用

第二次执行 `forall` 代码如下：

![image-20210903171728048](https://bbs.kanxue.com/upload/attach/202109/929902_Y29RQVZD4Z94GZS.jpg)

和第一次执行十分类似，因此就不深入分析。查看执行完 `forall` 后 `stringl63` 的变化：

![image-20210904155018524](https://bbs.kanxue.com/upload/attach/202109/929902_3KMU8PWWYN8P4VT.jpg)

查看 `l63` 中的值，发现是一个 `string` 结构，于是查看字符串，内容正是 `l102` 中存储的 `l36` 的字符串：

![image-20210904155250267](https://bbs.kanxue.com/upload/attach/202109/929902_8B2PT6W7R2AA694.jpg)

接着通过 `l90 0 l92 putinterval` 将 `l63` 中指向的第一个4字节的内容改为 `0x02b14ad4`，该值指向 `l36` 中四字节之后的内容：

![image-20210904161420028](https://bbs.kanxue.com/upload/attach/202109/929902_B95JGVDKGS53DQ4.jpg)

经过多次修改，字符串修改为如下状态，修改的值会在第三次漏洞触发时使用到：

![image-20210907120914829](https://bbs.kanxue.com/upload/attach/202109/929902_9Z6KW8MAFSP5UC4.jpg)

接着查看 `l137` 获取的是 `l63` 中 `0x4` 处的值，`l138` 获取的是 `l63` 中 `0x20` 处的值，`l103` 的值为1：

![image-20210904170623340](https://bbs.kanxue.com/upload/attach/202109/929902_BACQZHHKYTSZY3T.jpg)

第二次漏洞触发部分分析完毕，接下分析第三次漏洞触发构造读写原语的部分。

### 构造读写原语

`l142` 中存储的是将 `l138` 放入到 `l193` 的 `0x24` 位置后的字符串：

![image-20210904185419955](https://bbs.kanxue.com/upload/attach/202109/929902_KTBGKFDNHR9YB8Y.jpg)

接着使用 `forall` 操作符遍历 `l63` 数组。当遍历到第54个元素时，恢复快照。此时 `array l63` 被释放，接着复制 `l142` 字符串，使得 `array l63` 对象被 `l142` 字符串对象覆盖：

![image-20210906105724780](https://bbs.kanxue.com/upload/attach/202109/929902_ENYA39ACYHJ2GM8.jpg)

此时查看被覆盖后的 `l63` 中最后一次会被获取的值：

![image-20210906111515943](https://bbs.kanxue.com/upload/attach/202109/929902_Z83JHDTG9JTCA86.jpg)

说明最后一次会获取一个 `array` 对象。继续深入查看该对象，发现存储了一个字符串，该字符串起始地址为 `0x00000000`，大小为 `0x7fffffff`：

![image-20210906112556232](https://bbs.kanxue.com/upload/attach/202109/929902_9V89Q83GKFMN9YN.jpg)

通过该字符串，可读写内存中 `0x00000000-0x7fffffff` 的任意地址，实现了读写原语的构造，最终将字符串对象存储在 `l201` 中。

### 构建 ROP 链

通过字符串 `l201` 获取 `EPSIMP32` 的基地址为：`0x74750000`，并存入 `l314` 中：

![image-20210906151059868](https://bbs.kanxue.com/upload/attach/202109/929902_M4T5EWJ334HGWCW.jpg)

接着通过 `EPSIMP32` 的导入表获取 `kernel32.dll` 的基地址并存放于 `l315` 中：

![image-20210906162228059](https://bbs.kanxue.com/upload/attach/202109/929902_FQWQDPAAGWEVCX9.jpg)

随后开始利用读写原语搜索内存中的 `gadget` 用于构造 ROP 链：

![image-20210906164333935](https://bbs.kanxue.com/upload/attach/202109/929902_6GKPSNKC7VA3TDK.jpg)

将构造好的 ROP 链放入伪造的文件对象中，并将对象放置在 `l159` 的2号元素中，将恶意 PE 文件和 shellcode 组成的字符串放置在 `l159` 的3号元素中：

![image-20210906200817931](https://bbs.kanxue.com/upload/attach/202109/929902_Q5U5V2UH5NACTGH.jpg)

最终调用 `closefile` 操作符关闭伪造的文件对象，在关闭过程中会执行 `call [eax+8]`，使得跳转到构造好的 ROP 链中：

![image-20210906201456526](https://bbs.kanxue.com/upload/attach/202109/929902_3HZC5FC2TFZEU7D.jpg)

至此，整个漏洞的原理和利用分析完毕，剩下的行为部分不再分析。

## 总结

该样本漏洞利用得十分巧妙，通过 UAF 将原本正常的数组对象替换为指向构造好的、能够读写任意内存的字符串对象。通过字符串对象实现了读写任意内存并构造 ROP 链的目的，并最终将构造好的 ROP 字符串对象修改为文件对象，利用 `cloasefile` 操作符跳转到 ROP 链中。

尽管微软已经关闭了 Office 对于 EPS 文件的支持，但该格式文件仍然能被 Adobe Illustrator 打开。如果深入研究该类型文件，可能仍有出现漏洞的可能。

## 参考链接

[1] [PostScript LANGUAGE REFERENCE](https://bbs.kanxue.com/elink@180K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2k6r3!0T1k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6V1j5h3#2Q4x3V1k6S2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6V1k6i4k6F1k6i4c8Q4x3V1k6S2j5%4c8A6L8$3&6K6j5%4u0A6M7s2c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8W2m8x3f1V1#2Q4x3X3g2H3k6r3j5`.)

[2] [eps-CVE-2017-0261](https://bbs.kanxue.com/elink@ec8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5%4g2X3d9h3c8Q4x3V1k6W2M7s2y4Q4x3X3c8o6g2V1g2Q4x3X3b7J5x3o6p5%4i4K6u0V1x3o6t1$3x3b7`.`.)

[3] [CVE-2017-0261及利用样本分析](https://bbs.pediy.com/thread-261442.htm)
