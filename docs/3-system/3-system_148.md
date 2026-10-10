

# 显示输入表输出表信息小程序

> 原文：[`www.pediy.com/kssd/pediy10/98124.html`](https://www.pediy.com/kssd/pediy10/98124.html)

这段时间抽空复习了下 PE 结构，刚好又学了一点 asm,所以想仿照 lordpe 写个显示输入表信息的小程序，花了几天时间总算完成，一开始对列表视图不熟悉，花了好多功夫查找资料，显示子项还好说，对它的消息处理不清楚，就是怎么得到一个表项被选择的消息，找了好久的资料才弄清，程序写得比较烂，不像高手写的都是把部分功能做成子程序，整体框架比较清楚，我写往往从头一统到尾，仅供新手聊观:)  
  
不知道有没和我一样没什么基础的新手，论坛的资料很多，但切忌急功冒进，好高骛远，没打好基础，看得再多也只是眼花和浪费时间，不会有什么收益的，我就是个例子，来看雪三年了，还什么都不懂，惭愧。。。。![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")  
  
测试了一下发现有的文件会报错，才发现自己忽略了无函数名称只有序数的情况，修改好，继续测试，也巧居然又碰到两个特异情况，一个是 OriginalFirstThunk 是全 0 的，然后我想既然 FirstThunk 是 OriginalFirstThunk 的复制，干吗要用 OriginalFirstThunk，直接用 FirstThunk 得了，修改好，测试一个 dll 文件又有问题，用 winhex 打开一看，FirstThunk 指向内容居然与 OriginalFirstThunk 不同，已经换成了函数地址，虽然地址不一定正确，与加载这个文件时填入的地址不太相同，这就有了一个问题，PE 加载器是不是先读 OriginalFirstThunk 内容，非 0 就正常处理，为 0 时才去读 FirstThunk，一个文件要想正常被使用可以有三种情况  
1.FirstThunk、OriginalFirstThunk 都正常相同  
2.OriginalFirstThunk 为 0，FirstThunk 正常  
3.OriginalFirstThunk 正常，FirstThunk 随便填什么都行  
  
又完成了输出表的小程序，现补上

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || import\_new.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || export.rar  
 |