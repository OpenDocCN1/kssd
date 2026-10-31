
# [原创]pwn中one_gadget的使用技巧

## one_gadget介绍

很多新手在栈溢出的时候会手动输入`/bin/sh`地址和`system()`等函数的地址，这样不仅麻烦而且很容易出错。其实libc中带有很多gadget，控制程序跳转到这些位置执行并满足一定的条件就可以拿到shell。

![](https://bbs.kanxue.com/upload/attach/202008/853476_MT74MRE7H9WHN3F.png)

## 栈上的万能one_gadget

最常见的用法是`one_gadget`后面跟libc文件，就可以拿到`one_gadget`的地址了，上图我以libc 2.27为例，这里有一个小技巧，就是可以看到`one_gadget`是需要满足一定条件的，这里我认为最好满足的是`rsp+0x40==NULL`这个条件，要求栈上的这个位置为`null`，这里随便拿一个程序来看一下。我在返回地址处填充了`0x4f3c2`这个gadget。

![](https://bbs.kanxue.com/upload/attach/202008/853476_TER5DQWYWQJEYAR.png)

然后跟进去运行，![](https://bbs.kanxue.com/upload/attach/202008/853476_BPX64U8DHTACEGW.png)，发现程序将`rsp+0x40`赋值给了`rsi`，那么显然这个值是需要是`null`的，这也跟刚才`one_gadget`给我们提示的满足条件相符合。看一下这个值，![](https://bbs.kanxue.com/upload/attach/202008/853476_4YTHY6W2MY8APCS.png)

这个值并不是`null`，这样的话运行下去显然是会出错的，修改这个值显然也很麻烦，但是我们可以观察到`0x7fffb5b9e0d0 + 0x60`是`null`，符合我们的要求，而且刚才溢出的时候我们是可以控制`rsp`的，我们可以在输入`one_gadget`之前多个`ret`指令的`addr`，这样就可以增加`rsp`的值了。修改后的`payload`如下

```
payload = 0x78 * 'a' + p64(ret_addr) + p64(ret_addr) + p64(ret_addr) + p64(ret_addr) + p64(one_addr)
```

此时

![](https://bbs.kanxue.com/upload/attach/202008/853476_57KS4BGC3NMGD6S.png)

这样就可以满足我们的要求了

![](https://bbs.kanxue.com/upload/attach/202008/853476_PTM652FQDEPK8J5.png)

## 其他技巧

后面加`-l2`参数可以找到更多的gadget。

![](https://bbs.kanxue.com/upload/attach/202008/853476_RTZHYPUPCMGWWS5.png)

另外还有一个`near`功能也很管用，可以找最靠近某个函数的gadget，这里我以libc 2.23为例。

```
one_gadget /lib/x86_64-linux-gnu/libc.so.6 --near read
```

![](https://bbs.kanxue.com/upload/attach/202008/853476_KQJYJ5DS4M6TMS3.png)

可以看到这两个地址只相差了两个字节。利用这一点我们可以在泄露不了libc的情况下采用尾字节覆盖的方式来强行修改`read()`函数地址成为我们的`one_gadget`地址。我们看到libc函数在随机时最后倒数两个字节中只有前半个字节随机。

第一次

![](https://bbs.kanxue.com/upload/attach/202008/853476_2YCGTW3CXGG6MJJ.png)

第二次

![](https://bbs.kanxue.com/upload/attach/202008/853476_WBZZJM8X8DJHQ8T.png)

因此在这里我们只要覆盖尾部的两个字节，我们就有1/16的概率命中。

`one_gadget`的介绍到这里就结束了，如果有不足的地方，欢迎大家指正。
