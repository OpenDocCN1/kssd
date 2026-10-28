
3. 时机的问题，有些HOOK必须在API的开头，有些必须在API的尾部，比如HOOK `CreateFilaA()`，如果你在API尾部HOOK API，那么此时你就不能写文件，甚至不能访问文件；HOOK `RECV()`，如果你在API头HOOK，此时还没有收到数据，你就去查看`RECV()`的接收缓冲区，里面当然没有你想要的数据，必须等`RECV()`正常执行后，在`RECV()`的尾部HOOK，此时去查看`RECV()`的缓冲区，里面才有想要的数据；

4. 上下文的问题，有些HOOK代码不能执行某些操作，否则会破坏原API的上下文，原API就失效了；

5. 同步问题，在HOOK代码里尽量不使用全局变量，而使用局部变量，这样也是模块化程序的需要；

6. 最后要注意的是，被替换的CPU指令的原有功能一定要在HOOK代码的某个地方模拟实现。

若有疏漏之处，欢迎各位大侠指正！

![](https://bbs.kanxue.com/view/img/face/12.gif)![](https://bbs.kanxue.com/view/img/face/12.gif) 2014.10.27 6：30 pm

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`2.png`](javascript:void(0);)（61.46kb，33次下载）
- [`3.png`](javascript:void(0);)（11.71kb，16次下载）
- [`4.png`](javascript:void(0);)（35.11kb，14次下载）
- [`5.png`](javascript:void(0);)（34.53kb，16次下载）
