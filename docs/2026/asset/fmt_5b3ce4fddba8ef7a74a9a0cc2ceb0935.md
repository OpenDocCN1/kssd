
# 第二次分析

第二次分析，因为之前的应急刚处理完，忘记了之前用的不是吾爱版的`od`，调试了之后发现程序莫名其妙的就崩溃了，我内心是崩溃的（之前调试不是好好的嘛，咋就崩溃了？）

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_7DK5ZNNB7RMHBQH.png)

这错误也是尼玛的很奇怪，异常代码`c0000005`一般就是内存访问异常，但是给我的异常偏移为`0`？难道程序用到了`0`内存？一般情况下，访问`0`内存是会出现问题的，但是我记得昨天调试的时候，好像没有这个毛病啊？

就这样，第二次调试以失败告终，还特意在群里问了问各位大佬，依旧是没有得到什么有用的信息。没办法就这样开启了第三次调试。

# 第三次调试

第三次调试，不过这次用的`x64dbg`，此次调试很顺利，不过`x64dbg`有个比较麻烦的问题是`API`断下后，不像`OD`一样将参数集中在一起而是这样显示的。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_YPB62A9HMNP5DKG.png)

成功的从内存中`dump`出来了`Denes`样本，但是还是有很多疑问，对比看雪的一篇文章 https://bbs.pediy.com/thread-259596.htm。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_2RGJ7MRYSUQK3HG.png)

我发现个问题，为啥这个`Denes`资源不可见呢？这种问题几乎没遇到过，通过搜索引擎也得不到什么有效信息。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_SRDAZBTGWNGNMW8.png)

想着要不就这么放弃了吧，反正也不差这么一点，但是之后自己做的和心里想的就表里不一了，继续从刚才的文章中寻找相关的信息。果然想的什么都是扯淡，动手搞一搞比啥都强，最后还是本着一颗追求到底的心，又开始第四次的调试过程。

# 第四次调试

第四次调试，还是用回了之前的`OllyICE`继续调试，跳过之前定位到`Denes`的过程，来到最后的`call eax`这里。由于要加载资源函数所有就下了相关的`API`断点，`FindResource`将`A`系列和`W`系列都打上断点，还有`LoadResource`断点，运行看是个什么情况。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_KMAE5S8XPP3H3U8.png)

嗯？什么东西，这个`hModule`这个地址咋这么熟悉呢？数据窗口一跟随，卧槽咧，这不就是`Denes`嘛，果然在资源上做了一些手段，导致资源编辑工具无法直接查看，但是程序功能成功的定位，就表示内容还是存在的。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_RZ58XXNH65U2XDM.png)

通过查询`API`发现，`LoadResource`返回的地址，就是资源所在的内存地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_3FXYR5D4CA68JYK.png)

果不其然，在内存中找到了这段资源文件。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_7GS4VJBHNCEEWWF.png)

将整个资源文件从内存中`dump`出来放着，以防像之前一样重头再来，然后我就像个傻逼一样的盯着这段资源端，一直知道程序跑起来都没看到解码的过程，心想着啥情况，不是说会进行解密嘛，我咋没看到这数据有啥变化呢？？？

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_4PHC2JAZ6KB3BKV.png)

不过在这个过程中我发现出现了域名的解析，但是没有转向内存，因为一心傻逼的盯着了资源数据的变化。没得搞了程序已经跑飞了，只有第五次调试了。

# 第五次调试

第五次调试，在弄好之前的种种问题之后，缓慢仔细的调试着这段花指令代码，找到之前出现域名解析的`call`调用，应该就可以找到整段解密的过程了。现在的问题就是如何确定第二段代码需要的内存从哪里来？

再一次调试来到了`Call eax`这里，查看内存的情况，可以看到目前第二段内存并没有出现。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_DGR46BYEQEQDHSX.png)

现在步过`call eax`，其中触发了一次`HeapCreate`分配，不过大小只有`1024`，分配好的地址如下图所示。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_3ZHEZGWZDK5XQVN.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_4N77AGQBKEVEKWB.png)

卧槽咧，难道真的是用的`HeapCreate`来创建，并扩充了这个`heap`的大小？为什么这么说，来看一下内存的布局情况，已经无限接近于之前出现的第二段内存的情况了。

当从`call eax`返回之时，可以发现内存之前的`1000`大小变成了`3000`。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_Z23GTARRARFMWV8.png)

正当我以为这就是第二段内存是，`VirtualAlloc`的出现让我觉得，`heap`应该不是第二段的内存了，这时候再来看看内存布局的情况。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_4X5X67YF3ABXXEM.png)

这时查看内存时发现，到头来一场空，代码似乎已经解码完毕了。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_6USWFZBWH32ZQ2N.png)

不要以为我真的只是调试了`5`次，只是省略了很多很多重复的步骤，从刚才的测试中也知道了解密过程是在`call eax`中完成的，所以必须要跟进`call eax`进一步查看具体是什么回事。

# 第六次调试

第六次调试，也是我最后一次调试了，大多数已经弄明白了，就是最后的解密过程和内存分配问题了，已经找到了复制已解密的函数代码，但是依旧没有找到分配内存的`API`。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_F9BSKP9U7X89NH9.png)

第一次调用复制后的代码情况如下图所示。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_2D38Y7W52WRP475.png)

第二次调用复制后的代码情况如下图所示。

![图片描述](https://bbs.kanxue.com/upload/attach/202007/830514_RVJ9MZF2DHQ26ZZ.png)

自此整个解密的内存就找到了，只需要来一个`dump`即可拿出其中被解密出来的两个内置的`dll`文件了。

## 特征码

```
push ebx
push 1
push esi
add eax,esi
call eax
```

这一段特征码的作用是定位到解密完成的`Denes`的位置，通过`eax`的值，可以找到`Denes`的`Dos`位置。

```
and edi,0fh
and esi,0fh
cmp edi,esi
pop esi
pop edi
```

这一段可太不容易了，而且有多个函数都有调用这一段，这时复制已解密数据的关键点，通过命令找到该段代码的头部，调试即可发现整个复制过程。

## 总结

到此整个踩坑过程就全部结束了，整个的分析过程中，遇到了确实很多的问题，但是凭着耐心一步一步，解决了很多的问题，但是还是留下了两个问题。这段用于存放解密数据的内存区域从何而来的疑问，目前已经没有更多的精力继续搞下去了，对此感觉已经是尽力了，解密过程的代码在哪儿？应该和这段内存区域有着一定的关系，有知道该段内存区域如何来的小伙伴留言告知。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-9-20 16:02

被`Risks`编辑，原因：将语言书面化

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
