
有没有发现什么规律？注意对照脚本代码看，每次访问失败后都会多取一部分再次进行尝试。

```
cvtres.exe /machine:x86 /nologo /out:"d:\test-rc-compile-error\test.tmp" /readonly 1.res 2.res 3.res 4.res 5.res
```

进一步调试确认后，上面提到的对 `SearchPathW` 的调用，并不是在对 `PATH` 环境变量中的每个路径做循环。而是在对脚本中的每个部分进行循环。`SearchPathW` 会在内部处理环境变量中的值。根据 [MSDN `SearchPathW` 的文档](https://bbs.kanxue.com/elink@411K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3V1k6F1k6W2)9J5k6s2m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3X3c8K6k6h3q4J5j5$3S2H3j5i4c8Z5N6H3`.`.), 这个函数的参数如下：

```
1  DWORD SearchPathW(
2      [in, optional]  LPCWSTR lpPath,
3      [in]            LPCWSTR lpFileName,
4      [in, optional]  LPCWSTR lpExtension,
5      [in]            DWORD   nBufferLength,
6      [out]           LPWSTR  lpBuffer,
7      [out, optional] LPWSTR *lpFilePart
8  );
```

其中，`PATH` 环境变量的值会传给 `lpPath` 参数。上面提到的有规律的调用是通过参数 `lpFileName` 传递的。为了自动观察相应的参数情况，我使用了如下条件断点。

```
~4 bp KERNELBASE!SearchPathW ".echo **************** bang ****************; du /cA0 poi(esp+4) LA0; du /cA0 poi(esp+8) LA0; kv2; gc;"
```

`~4` 表示只对 `4` 号线程设置断点，其它线程调用 `KERNELBASE!SearchPathW` 不会中断下来。

`/cA0` 表示显示指定显示列宽为 `0xA0`，如果不指定，则按默认列宽进行显示。

`LA0` 表示只显示前 `0xA0` 个字符。

运行结果如下图：

![auto-display-param-and-callstack](https://bbs.kanxue.com/upload/attach/202204/873494_5ZRV8YYQM5U8Z68.jpg)

## 反思

其实这个问题本来应该很快就能解决的。

首先，在调试之前就应该使用 `process monitor` 查看文件访问事件。只不过当时脑子不清醒，只是用 `process monitor` 查看了进程创建事件，没有什么发现就开始调试了，太草率了。

其次，当时陷入了思维误区 —— 想当然的认为带空格的路径一定要加上双引号。整个排查过程中都没有怀疑是多加了双引号导致的问题。反过头来看这个问题，觉得自己太傻了，查看 `PATH` 环境变量的时候，非常明显可以发现其它带空格的路径并没有使用双引号，但是我自己添加的路径却带了双引号。

## 收获

- 长记性了，设置 `PATH` 环境变量的时候，即使路径中包含空格也不能加双引号。
- 再次熟悉了使用 `gflags` 设置 `Image File Execution Option` 的方法，这次是使用调试器调试另外一个调试器。
- 熟悉了 `wt` 的用法。很早之前就知道 `wt` 的存在，一直没实际用过。当想查看一个函数都调用了哪些子函数时，非常方便。
- 熟悉了为指定线程设置断点的方法。语法很简单，很容易记忆。在 `bp` 之前添加 `~tid`，即可为指定的线程设置断点。
- 知道了突破默认输出列宽的方法 —— 通过 `/c` 指定显示列宽。
- 熟悉了 `CreateProcess` 的部分逻辑，创建进程真的是个大工程。

## 参考资料

`b73K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4@1K9s2u0W2j5h3c8K6j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8H3M7X3!0U0k6i4y4K6N6r3S2J5k6h3q4V1M7$3q4H3K9g2)9J5k6r3y4J5k6h3q4@1k6i4m8J5L8$3y4W2M7%4y4%4

`51aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8Y4m8J5L8$3y4@1K9s2u0W2j5h3c8Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1j5%4u0W2j5i4c8A6L8$3&6Q4x3X3c8X3L8r3q4Y4M7H3.`.

`cb8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3c8W2j5Y4g2Y4i4K6u0r3M7%4W2K6N6r3g2E0i4K6u0V1k6i4u0J5L8%4u0Q4x3X3c8U0L8$3c8W2M7#2)9J5k6q4)9J5k6o6m8Q4x3X3b7`.

`300K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3V1k6F1k6W2)9J5k6s2m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3X3c8K6k6h3q4J5j5$3S2H3j5i4c8Z5N6H3.`.

`955K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3V1k6F1k6W2)9J5k6s2m8J5L8$3y4W2M7%4y4W2L8Y4k6Q4x3X3c8K6k6h3q4J5j5$3S2H3j5i4c8Z5N6H3.`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
