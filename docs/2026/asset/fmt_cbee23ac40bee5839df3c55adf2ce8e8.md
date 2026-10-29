
![](https://bbs.kanxue.com/upload/attach/202004/715334_VDJZFDGASPWT6HC.jpg)

就是动态注册而已，我们直接修改源码，跟踪动态注册的函数`check`。

![](https://bbs.kanxue.com/upload/attach/202004/715334_NGW9AF5E2TFC3JS.jpg)

可以观察到`check`函数地址为`0xb39444c9`，

再通过`cat /proc/13291/maps | grep native-lib`命令查看`libnative-lib.so`的加载基地址为`0xb3943000`，这里的13291为进程pid。

![](https://bbs.kanxue.com/upload/attach/202004/715334_8E8CB2K4GNSVUG3.jpg)

从而确认`check`的函数偏移为`0x14c9`，找到对应函数。

![](https://bbs.kanxue.com/upload/attach/202004/715334_2REECG85Y9FA7WM.jpg)

可以发现这个函数做了4件事：

1. 检测输入`string`长度是否为20。
2. 再次动态注册`check`函数到新的native函数偏移为`0x1234`。
3. 检测输入的`string`是否以`kanxue`开头。
4. 调用新的`check`函数，即`sub_1234`函数，并传递输入字符串的第六位之后的字符串为参数。

再次跟进`sub_1234`函数：

![](https://bbs.kanxue.com/upload/attach/202004/715334_9Z8R37FZ6RZDFFC.jpg)

可以看到同上个函数一样，这个函数也做了四件事：

1. 检测输入`string`长度是否为14。
2. 再次动态注册`check`函数到新的native函数偏移为`0x1148`。
3. 检测输入的`string`是否以`training`开头。
4. 调用新的`check`函数，即`sub_1148`函数，并传递输入字符串的第8位之后的字符串为参数。

最后再次跟进`sub_1148`函数：

![](https://bbs.kanxue.com/upload/attach/202004/715334_UVX7VCPTKD8XUJQ.jpg)

这个函数只是检测了最后的字符串长度是否为6，并且将之与字符串`course`对比。

最终，拿到flag为`kanxuetrainingcourse`。

验证。

![](https://bbs.kanxue.com/upload/attach/202004/715334_UZUK63HBP8P96QA.png)

这个时候再看log，会发现确实动态注册了三次`check`函数，且注册在了不同的位置。

![](https://bbs.kanxue.com/upload/attach/202004/715334_ZE4GG4VM59AHBMR.jpg)

## 后记

这个程序貌似存在一个bug，就是我第一次验证成功后，不退出程序，再次输入同样的flag，这个时候再`check`就不对了，而输入`course`就对了，估计是多次动态注册导致再次`check`时只走了最后`sub_1148`函数的原因。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
