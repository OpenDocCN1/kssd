

# 关于 WinDbg 的 symbols 加载失败的个人心得（申请邀请码）

> 原文：[`www.pediy.com/kssd/pediy12/140945.html`](https://www.pediy.com/kssd/pediy12/140945.html)

今天装了 windbg 进行双机调试，发现符号总是加载失败。不明白其中的缘由，于是乎找谷歌，百度。发现好多朋友也有类似的问题。也有热心的朋友给出了解决方法，例如下面的  
[`hi.baidu.com/%B0%D9%B6%D6%C6%...8d6cae276.html`](http://hi.baidu.com/%B0%D9%B6%D6%C6%A1%BE%C6/blog/item/91cef923c2573be8d6cae276.html)  
但是按照上面的方法并不能解决我的问题。网上这种问题的解决方法虽然多，但基本都是转来转去的千篇一律。  
于是想办法自己解决，根据 windbg 的提示，到官网上下载了自己被调试机所需要的符号文件。链接如下：  
[`msdn.microsoft.com/en-us/wind.../gg463028.aspx`](http://msdn.microsoft.com/en-us/windows/hardware/gg463028.aspx)  
再次.reload，结果成功了，好多以前找不到的函数也找到了！！！  
  
  
总结  
大多数人可能是根据网上的方法配置符号路径的，即：SRV\*your\_path\*http://msdl.microsoft.com/download/symbols  
这种方法虽然方便，但是会出现符号不完整的问题。比如我用这种方法下载后符号文件总共才 40MB 左右，但是从官网上下载下来的符号文件居然接近 1G。所以我的观点是自己手动从官网下载符号文件（很多人都怕麻烦，包括我，但是操作以后发现这其实一点都不麻烦，下载下跟安装一般软件一样安装，安装完在 windbg 里面添加咱们的安装路径就 ok 了），然后通过 SRV\*your\_path\*http://msdl.microsoft.com/download/symbols 更新就可以了。  
  
这就是本人的一点心得，可能有点弱，但是这个问题真的困扰了我很久，也希望这片文章对想我一样的初学者有所帮助。由于是新手，可能有不对的地方，欢迎大家指正。 

代码:

```
symchk /r %windir%\\system32 /s SRV\*d:\\symbols\\\*http://msdl.microsoft.com/download/symbols

```

也可以这样下载所有的符号文件。![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")