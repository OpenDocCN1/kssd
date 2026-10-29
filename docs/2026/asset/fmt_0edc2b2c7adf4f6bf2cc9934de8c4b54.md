
● `bottomState`和`source`字段在内存空间的布局相差6个指针，那么`entry[27] = 0`即为对`bottomState`进行赋值；

● `entry->policyFlags |= 0`没有任何意义，则被编译器优化了；

既然知道了`entry[27]`就是对`bottomState`进行赋值，那么我们来改改看。

![图片描述](https://bbs.kanxue.com/upload/attach/202209/926631_ER5PCC4B5U2JQFV.png)

可以看到`entry[27]`对应的是这行汇编代码，改它！！

然后到这里我遇到了一个问题，IDA的keypatch不支持苹果M1 MAX的Macbook。既然这样那就直接改源码，然后把`libinputfinger.so`推进去替换掉。

![图片描述](https://bbs.kanxue.com/upload/attach/202209/926631_E7EMCPN93EPHHXT.png)

改完以后初始化环境，然后编译一下这个模块

![图片描述](https://bbs.kanxue.com/upload/attach/202209/926631_PTV828N4GVTWENJ.png)

然后把`libinputfinger.so` push进手机，接下来重启手机，再次打开我们自己编写的flutter应用测试

![图片描述](https://bbs.kanxue.com/upload/attach/202209/926631_PYVSFFFS2D7W2AT.png)

然后我们设备的这个方法就进来了，flutter也确实可以点击了，然后我们去验证一下其他原生点击和webView的点击。

经过一番测试发现其他都正常的，但是WebView不能点击跳转了。由于WebView也是原生事件转换成js的事件的，所以这几个hover事件不给肯定也会产生影响。所以把这三个hover事件也给分发下去。

![图片描述](https://bbs.kanxue.com/upload/attach/202209/926631_99FBE7DG4CJR66H.png)

重新编译`libinputfinger.so`并推入手机，然后WebView的世界也恢复正常了。

总结：通过编写flutter应用，然后在`onTap`（flutter的点击回调）事件做调试，查看正常手机的堆栈，可以很容易的找到flutter点击事件的源头，然后在整个堆栈都下断，再由我们的设备来触发点击，就能找到和真机的区别在`bottonState`的值导致了down事件时没有把事件添加进去，然后up事件时发现容器内没有down事件的记录，就抛弃了这次up事件，进而导致flutter没有点击事件的回调。在翻看源码的时候我们也发现这个值被强制赋值成0，那么把它改掉，问题就能解决了，至于后面的WebView也是一样只要做一个简单的尝试，看看鼠标连上手机后是否正常点击，然后我们源码把哪几个事件丢弃了，还原一下。

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)

最后于 2022-9-15 21:21 被hjhjl222编辑，原因：

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#系统相关](https://bbs.kanxue.com/forum-161-1-126.htm)
