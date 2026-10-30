
`int security_task_prctl(int option, unsigned long ...)`发现第一个参数`option`是`int`类型，也就是我们传入的`64位会被截断成为32位`，这也就导致了64位，第一个参数不能用了。

这个不影响32位下的漏洞利用，我们继续来分析32位如何利用。

其实，这里有很多思路，`INetCop Security`的`Slide`里面有，我这里介绍另外一种基于`VDSO`变形的方法，也是看雪论坛上(https://bbs.pediy.com/thread-220057.htm)，提出的一种方法。

具体思路是:

1. 先通过劫持`task_prctl`，将其修改成为`set_memory_rw`
2. 然后传入`VDSO`的地址，将`VDSO`修改成为可写的属性，
3. 然后之后的步骤就和劫持`VDSO`方法是一样的了。

这种方法我进行了验证`64位内核`上是不可行的，就是因为第一个参数被截断的原因。

## 0x03 `call_usermoderhelper`内核线程执行

`call_usermoderhelper`是内核运行用户程序的一个`api`,并且该程序有`root`的权限。如果我们能够控制性的调用它，就能以`Root`权限执行我们想要执行的程序了。

定义在`kernel/umh.c`中

![krealloc](https://bbs.kanxue.com/upload/attach/201803/768814_9EKNXZPVWV49ZVD.png)

`subprocess_info`如下

![krealloc](https://bbs.kanxue.com/upload/attach/201803/768814_HWFCJD8GGU5XX8Z.png)

我们要劫持`task_prctl`到`call_usermoderhelper`吗，不是的，因为这里的第一个参数也是`64位`的，也不能直接劫持过来。但是内核中有些代码片段是调用了`Call_usermoderhelper`的，可以转化为我们所用。

譬如`kernel/reboot.c`中的`orderly_poweroff`函数中调用了`run_cmd`参数是`poweroff_cmd`,而且`poweroff_cmd`是一个全局变量，可以修改。

![krealloc](https://bbs.kanxue.com/upload/attach/201803/768814_JKKXQBND4TFHCRB.png)

那么我们就先篡改`poweroff_cmd`=我们预期执行的命令，然后直接劫持`task_prctl`到`orderly_poweroff`函数，这样就任意命令执行了，同时按照`INetCop Security`给出的思路，需要先关闭`selinux`。

所以再整理一下整体思路:

1. 利用`kremalloc`的问题，达到`任意地址读写`的能力
2. 通过快速爆破，泄露出`VDSO`地址。
3. 利用`VDSO`和`kernel_base`相差不远的特性，泄露出`内核基址`
4. 篡改`prctl`的`hook`为`selinux_disable`函数的地址
5. 调用`prctl`使得`selinux`失效
6. 篡改`poweroff_cmd`使其等于我们预期执行的命令。
7. 篡改`prctl`的`hook`为`orderly_poweroff`
8. 调用`prctl`执行我们预期的命令，达到内核提权的效果。

最后给出利用程序如下
