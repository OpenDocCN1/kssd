
> Because hooks run in the context of an application, they must match the "bitness" of the application. If a 32-bit application installs a global hook on 64-bit Windows, the 32-bit hook is injected into each 32-bit process (the usual security boundaries apply). In a 64-bit process, the threads are still marked as "hooked." However, because a 32-bit application must run the hook code, the system executes the hook in the hooking app's context; specifically, on the thread that called `SetWindowsHookEx`. This means that the hooking application must continue to pump messages or it might block the normal functioning of the 64-bit processes

一句话来说就是32位的DLL钩子只能注入32位程序，同理64位也是如此。但问题是我的药引子程序也是32位的，DLL也是32位的，虽然笔者设置的是全局钩子，但为何还会卡死呢？
值得一提的是，当把DLL和药引子程序均设为64位时，药引子程序便没有卡死。

# 0xC 总结

当从书里出来，自己实现一遍遇到各种各样的问题并解决后，原本觉得是天书的代码焕然一新，仿佛觉得能信手拈来。钩子注入的核心在于对拦截事件的处理。而作为新手的我们，应该先注重如何注入钩子，再去考虑如何处理事件。当两者已了然于胸，到时候离逆向之路也会走的更远了吧。

示例及源码下载：9a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3q4J5k6g2)9J5k6i4N6W2K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8U0y4F1b7#2N6C8k6K6k6\*7

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-8-11 20:47

被psycongroo编辑，原因： 添加下载联机
