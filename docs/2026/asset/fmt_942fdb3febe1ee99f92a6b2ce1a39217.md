
# [原创]阿里加固dex脱壳

这次研究一下阿里加固，提交一个APK，加固好之后下载，大致看一下加固样式都差不多，这是外壳：

![](https://bbs.kanxue.com/upload/attach/201606/592070_meqkp9q4r5xxiq7.png)

加固后的dex在lib里面的`libmobisecy1.so`中，效果如下：

![](https://bbs.kanxue.com/upload/attach/201606/592070_jji1bojdgg5s599.png)

看了一下字节码发现所有方法的字节码都被这28字节替换了，真正的指令被隐藏了

![](https://bbs.kanxue.com/upload/attach/201606/592070_8ge7bplcwe6fy5i.png)

不管怎么样，先从内存中dump出这个dex来看看，dump出来发现这个dex是坏的，`dex2jar`也看不了，反编译也没用，对比加载前的dex发现区别不大，看上去没有在内存中还原的样子。

## 破解思路

无论dex怎么隐藏，该运行哪个方法还得运行，于是修改源码，找到方法运行的真正地址，如下图所示

![](https://bbs.kanxue.com/upload/attach/201606/592070_k8psyyrephit83m.png)

打印了几个方法的`codeoff`，发现有个奇怪的地方，`codeoff`的值明显超出了dex的大小。那么我猜测他们肯定多new了一部分内存，把真正的指令往后面放了，加载的时候把dex所有方法的`codeoff`重写一遍，指向了新的地址。

既然这样那我就多dump一部分数据出来，直接从dex头开始dump，dump长度是dex的两倍。

## 脱壳操作

这样内存中的dex以及后面的尾巴都dump出来了，可能还多了一部分无效的数据，不过没关系，先拿下来看看

![](https://bbs.kanxue.com/upload/attach/201606/592070_1c37x1j4d17isxr.png)

把拿到的dex反编译一下看看，不出意外的报错了

![](https://bbs.kanxue.com/upload/attach/201606/592070_8wj0mc0psoy51z7.png)

不过可喜的是，绝大部分方法的`smali`已经反编译出来了，既然这样把报错类的`smali`都删掉，然后回编译试试。可能回编译也会有类报错，不过同样地都删除，最后回编译成功，`dex2jar`发现方法都暴露了。

最后再测试这个dex是否可以运行，在这里还有资源文件保护，因此修改`AndroidManifest.xml`的入口还需要进一步破解操作，不过没关系，我们作弊一下，APK是我提交的，把原始的xml拿过来借用一下，就当绕过资源保护了。

最后重新打包一个新的APK，运行成功，dex脱壳完成。

## 结束语

阿里加固的核心思想还是内存中还原dex，不过还原的是方法的`codeoff`，`codeoff`指向了dex范围之外的一片内存。最后资源保护这一块还需要花点时间研究。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [QQ截图20160607141014.png](javascript:void(0);)
  （45.17kb，667次下载）

- [QQ截图20160607141710.png](javascript:void(0);)
  （24.05kb，99次下载）

- [QQ截图20160607141908.png](javascript:void(0);)
  （13.43kb，33次下载）

- [QQ截图20160607150400.png](javascript:void(0);)
  （14.60kb，75次下载）

- [QQ截图20160607150617.png](javascript:void(0);)
  （12.57kb，48次下载）

- [QQ截图20160607151127.png](javascript:void(0);)
  （3.60kb，36次下载）
