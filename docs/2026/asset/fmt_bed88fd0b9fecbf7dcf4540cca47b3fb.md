
接着，程序将会执行`$content=trim(file_get_contents($flag));`。我们分步看。首先会执行`file_get_contents($flag)`。正常情况下，如果你不通过URL传入`flag`参数，那么，因为程序的最开始已经执行过`$flag='xxx';`，所以到了这里将会执行`file_get_contents('xxx')`。`file_get_contents`函数可以“将整个文件读入一个字符串”。比如如下代码可以将`homepage`变量中并显示出来。

```
<?php

$homepage = file_get_contents('b74K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2^5j5h3#2H3L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3U0N6Q4x3U0W2Q4x3@1t1');

echo $homepage;

?>
```

接着，php又会执行`trim`函数，它将会“去除字符串首尾处的空白字符（或者其他字符）”。

最后，php将会执行`if ($sixstars==$content)`，如果为`TRUE`，那么将会显示`flag`。

读到这里，你想必已经知道了我们该怎么做：首先，给`sixstars`变量传入一个值，比如`1`。然后，给`flag`变量传入一个我们能控制的网站的地址，并让这个我们能控制的网站的源代码设置为`1`。由于`extract`能起到“变量覆盖”的作用，在`extract`后，`flag`变量就会被覆盖为我们能控制的网站的地址，而不再是`'xxx'`，这样当执行到`file_get_contents`时php后端将会从我们能控制的网站上读取到`1`，并将其和`sixstars`变量比较，并返回`TRUE`，然后就能打印`flag`。

虽然这个方法是可行的，但是今天我要教你一个更简单的方法，那就是php伪协议。我们可以直接给`flag`变量传入`data://text/plain,1`。意思是明文`1`。这样`file_get_contents('data://text/plain,1')`将会直接返回`1`，就不需要我们在公网上开一台服务器了。

所以，最后我们的`payload`是：

```
08eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5K6z5g2)9J5k6e0t1J5y4q4)9J5k6e0t1J5x3q4)9J5k6e0j5%4i4K6y4m8x3U0x3&6x3o6m8Q4x3V1k6V1L8i4y4B7i4K6u0r3L8r3g2$3k6h3H3J5i4K6u0r3i4K6y4r3M7$3W2^5M7%4c8S2M7Y4y4Q4x3@1b7I4i4K6t1$3j5h3#2H3i4K6y4n7k6X3I4S2k6#2)9K6c8r3c8S2N6r3q4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6i4S2@1i4K6u0r3M7r3I4S2K9h3&6Q4x3V1x3I4
```

这里再强调一下，给后端传参的方法是：在`?`后跟变量名字，不同的变量之间用`&`隔开。

关于伪协议，我们以后有机会还会碰到。请大家继续关注看雪Web安全板块crownless的CTF赛题讲解。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-1-26 12:39

被crownless编辑，原因：
