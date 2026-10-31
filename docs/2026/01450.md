
# [原创]破解NetKeeper本地密码读取

原文来自：【Tracy‘Blog】——[feaN6%4N6%4i4K6u0W2M7s2g2J5M7r3I4W2M7X3!0U0i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@2aaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8#2M7Y4m8D9k6i4u0G2j5#2)9J5k6h3y4G2L8b7`.`.)

有了在涉外混迹四年，写下的《校园网那些事》的经历后，写这篇文章似乎要比以前容易些。来重庆也有1个半月了，一直昏昏忽忽的，总提醒自己要打起精神来，却总打不起精神。可能是很久没什么能够刺激自己、给自己增加自信心吧，于是决定看看学校的客户端。

以前做过类似的事，于是，也就有些思维固定，不想再去做同类型的事了，属于眼高手低型。然而账号到期了，又想起了以前做过的事。那，来找找自信吧~~不过，这客户端比涉外的安腾高级了好几倍~~~~

其实，先前也看过这个客户端，不过，一直想的是能够实现linux下拨号上网。好了，废话不多说了，进入正文吧。

先说目的：从文件中还原出我们拨号时输入的账号、密码。那先从文件看起，看看是哪个文件保存了这些信息。

打开netkeeper的安装目录，看看每个文件是干嘛用的：

![](https://bbs.kanxue.com/upload/attach/201310/475374_668e3486c39acadb.jpg)

这个应该是没可以理解的。那，保存密码的文件应该是在`bin`或者是`config`文件夹中了（省去推论文字，没看懂的可以先看看《[校园网那些事](http://bbs.pediy.com/showthread.php?t=161386)》……）。其实，我们对比刚安装好程序，和拨号一次之后文件夹中的内容就可以清楚的看到，在`bin`文件中多了一个`Credit`文件。那，它很有可能就是我们要找的文件了。

ok，我们拿出`peid`看看程序有没加壳，

![](https://bbs.kanxue.com/upload/attach/201310/475374_1736bfa0f729ec4d.jpg)

提示没找到，不过，看区段就知道`vmp`的壳。我等菜鸟是没想过要去挑战它的。那，带壳运行来调试吧。

`OD`载入（忽略所有异常），直接对下断`bp CreateFileA`（分析见《[校园网那些事](http://bbs.pediy.com/showthread.php?t=161386)》），`F9`跑起来，发现了类似如下`call`

```
0018785C   7266C4A4  つfr   /CALL 到 CreateFileA 来自 AcLayers.7266C4A1
00187860   00187CA0  爘.   |FileName = "\\.\Global\NPF\_{C97F955E-E47F-4270-9432-764A2C7C41D3}"
```

我们暂且把它当作是`vmp`壳对程序自解压时的一些操作吧（知道的还麻烦告知一声），这也是为什么在我们确定了读取文件的位置后不能直接对地址下断，而必须每次都跑完对类似这些`FileName = "\\.\Global\NPF\_`文件的操作的原因。我理解为，因为程序还没解压完，你直接下断，会引起程序解压出错，而导致程序出错。

一直`F9`到`FileName`为`C:\ChinaNetSn\`时就要注意了。

![](https://bbs.kanxue.com/upload/attach/201310/475374_00823bea7ffd9133.jpg)

一下下的摁`F9`，到出现读取`Credit`文件：

![](https://bbs.kanxue.com/upload/attach/201310/475374_be1dd7022b143f68.jpg)

取消断点，`Alt+F9`返回用户代码区。我看到的是非常乱的数据：

![](https://bbs.kanxue.com/upload/attach/201310/475374_77b18100283959cc.jpg)

没关系，`CTRL+A`分析一下。就出现了我们可爱的汇编代码了：

![](https://bbs.kanxue.com/upload/attach/201310/475374_fc23427c77885769.jpg)

我们可以看到，刚才程序仅仅是访问`Credit`文件，而还没进行读取。一下是读取片段。

![](https://bbs.kanxue.com/upload/attach/201310/475374_e57ce18c8373a9b3.jpg)

为了节省篇幅，直接复制代码吧：

```
0041B049   > \817C24 0C 8C1>cmp dword ptr ss:[esp+C],168C            ;  文件完整性校验
0041B051   .  74 17         je short NetKeepe.0041B06A
……
0041B06A   > \56            push esi                                 ; /hObject
0041B06B   .  FF15 30425000 call dword ptr ds:[504230]               ; \CloseHandle
0041B071   .  6A 00         push 0
0041B073   .  57            push edi
0041B074   .  8BCB          mov ecx,ebx
0041B076   .  E8 25000000   call NetKeepe.0041B0A0                   ;  加解密入口
```

为什么判断`0041B076`时解密入口呢？你可以自己直接`F8`过去，然后看寄存器啊，内存啊，等等之类的变化来确定。好吧，主题来了，我们要解决的就是这个`call`。

先看反汇编代码以及我的注释：(有点长，放到最后附录一中。想看的忍一忍看完吧！)

这段标准的代码中，我们可以看出，解密有三个步骤，第一和第三步是进行移位操作，第二部才是真正的各种转换。

我们要进行解密，也就是把这段代码换成我们的`C`语言，来进行解密。在解密入口处写了个`xinlides`，其实我也不知道他是不是用的`des`。目测不是吧：

附上`C`程序解密代码。（见附录二）

这里解释下主要代码段：

```
void encode(char *buffer, int len)
{
   char *p;
   while (len)
   {
       p = encode1(buffer);//位置交换
       encode2(buffer, p);//解密
       encode3(buffer);//顺序改回
       len = len - 8;
       buffer = buffer + 8;
   }
}
```

最难的问题应该在于第二步了，为了偷懒，少写点代码，就直接用`C`语言嵌套`asm`来实现。

不过这样做的最困难的地方应该在于，保护现场以及，匹配好各传入参数。我们来对比一下`C`和反汇编的代码。

在反汇编代码中，进行第二步加密中 `mov eax, dword ptr [ecx]`前，`push`了以下参数：

```
00404256  |.  68 80755400   push NetKeepe.00547580                   ;  要用到的pass
0040425B  |.  52            push edx                                 ;  放入地址
0040425C  |.  E8 DF000000   call NetKeepe.00404340                                ;  解密第二步
                       
               00404340  /$  51            push ecx
               00404341  |.  8B4C24 08     mov ecx,dword ptr ss:[esp+8]
               00404345  |.  53            push ebx
               00404346  |.  55            push ebp
               00404347  |.  56            push esi
```

那么，我们要做的就是，弄清楚这些`push`进去的东西，是什么，在后面用不用得上。

通过分析可以得到`00404256`处`push`的是要用到的解密矩阵。`0040425B`是什么要解密的内容（已被第一步解密的内容）。`00404340`是`push`此次要解密的最后一位数的地址。其实可以看作保护现场。`00404341`是把`0040425B`处放入的内容弹出，即`ecx`中保存的是实际上要进行解密的那四个字符。后面连续三个`push`是为了保护现场。

所以，在进行代码嵌套的时候，我们要想这段反汇编代码能够按我们的想法运行，我们就要给他创造好条件。所以，在开头，我们构造如下：

```
void encode2(char *buffer, char *key)
{
   __asm
   {
            lea eax, key
       push [eax]   //生成的key的地址压栈  0018fe1c
       lea eax, pass
       push eax   //把pass的地址压栈   0018fe18
       lea eax, buffer
       push [eax]    //剩余的要解密的内容入栈   0018fe14
       mov ecx, dword ptr [esp+8]   //把key给ecx
       push ebx   //10
       push ebp   //0c
       push esi   //08
       mov eax, dword ptr [ecx]
       mov edx, dword ptr [ecx+4]
       mov ecx, eax
       mov esi, edx
       shr ecx, 4
       and ecx, 0x0F0F0F0F
       and esi, 0x0F0F0F0F
       xor ecx, esi
     ……
       xor esi, eax               //esi中为解密后，前四个字节
       mov eax, dword ptr [esp+0x6C]   //将buffer的地址给eax    重点是，是否成功得到buffer地址
       xor edx, ecx              //edx中为解密后，后四位字节
       mov dword ptr [eax], esi    //写入解密后的前四位字符
       pop esi
       pop ebp
       mov dword ptr [eax+4], edx  //写入解密后的后四位字符
       pop ebx
       pop edx
            pop ecx
            pop ecx//后来自己加上去的（不然，弹出的esp恢复不了）
   }
}
```

当然，要嵌套好反汇编的`asm`为自己所用，你必须自己单步去一一调试，对比每一步得到的结果。有差异，就更改~~~

其中还有个很有趣的代码如下：

原反汇编代码中所有的类似如下的内存赋值：

```
mov ebp,dword ptr ds:[ebx*4+536408]
```

均被替换为类似如下代码：

```
push ecx
      lea ecx, key3
      mov ebp, dword ptr [ebx*4+ecx]
      pop ecx
```

很巧妙的用到了堆栈作为临时存储点。

好吧，就写到这里吧。

代码实现的功能是：

把`Credit`文件中的所有账号密码解密后取出，以明文形式保存到`Credit.txt`文件中。

其实，我们还可想象下一些盗号程序的工作原理，我们设立一个服务端用作收信，程序把得到的`buf`发送到服务端，就是一个典型的盗号程序了。

好了，就写到这了。

下一步要做的，尽量找到心跳包原理，希望能够完全脱离`netkeeper`吧，动态用户名的生成算法网上已经有了。我也拿了份，不过，暂时没用上，有想研究的，可以联系我，一起努力把。

最后，感谢`monsterok` ，文中代码是基于他的帖子中给出的代码做的更改 ：http://bbs.pediy.com/showthread.php?t=136534&highlight=%E9%97%AA%E8%AE%AF。

###################################################################################################################

代码见附件。。。

若由于排版问题带来阅读不舒适，还请见谅，附上`html`文件。

——Tracy   2013-10-29

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [netkeeper.rar](javascript:void(0);)

  （252.85kb，167次下载）
