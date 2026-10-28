
![](https://bbs.kanxue.com/upload/attach/202011/36653_9GV4TFNFVE6XEBH.jpg)

确实是`sysmon`的驱动`sysmonDrv`，他的入口点是`sysmonDrv+1e058`，看来这个方法确实有效，接下来我们要把`driver_init`设置为0，尝试修改

`Eq xxxxxxx 0` 修改成0

![](https://bbs.kanxue.com/upload/attach/202011/36653_WNQ3NB4DKN5UVGV.jpg)

接下来直接`f5`，成功蓝屏

![](https://bbs.kanxue.com/upload/attach/202011/36653_EEP9XEGZ3Q5S5X8.jpg)

因为我们把入口点设置为0，所以一到执行入口点就蓝屏，说明我们可以从代码上控制驱动加载。`Kb`后显示堆栈，驱动执行路径

![](https://bbs.kanxue.com/upload/attach/202011/36653_8VVAPCGSA9ADURX.jpg)

```
RetAddr : Args to Child : Call Site
00 fffff800`042b3477 : fffffa80`029c81c0 fffffa80`029c81c0 00000000`00000000 00000000`000007ff : 0x0
01 fffff800`042b3875 : 00000000`00000001 00000000`00000000 00000000`00000000 fffffa80`029c82f8 : nt!IopLoadDriver+0xa07
02 fffff800`03ed9161 : fffff8a0`00000000 ffffffff`80000f54 fffff800`042b3820 fffffa80`00711680 : nt!IopLoadUnloadDriver+0x55
03 fffff800`0416f166 : 00000000`00000000 fffffa80`00711680 00000000`00000080 fffffa80`006ed1d0 : nt!ExpWorkerThread+0x111
04 fffff800`03eaa486 : fffff800`04044e80 fffffa80`00711680 fffffa80`00711b60 00000000`00000000 : nt!PspSystemThreadStartup+0x5a
05 00000000`00000000 : fffff880`0457a000 fffff880`04574000 fffff880`04578590 00000000`00000000 : nt!KiStartSystemThread+0x16
```

也就是`call 0`，我们设置的驱动入口下面我就方便修改demo驱动代码去控制驱动的加载了入口点。

在`CBTdPreOperationCallback`的函数里加一个修改`DriverInit`的数值，然后赋值为我们自定义的`FakeDriverEntry`函数

![](https://bbs.kanxue.com/upload/attach/202011/36653_PS59A6BDRCBKS5A.jpg)

![](https://bbs.kanxue.com/upload/attach/202011/36653_PSAH28KAZV5R2Y3.jpg)

实现`FakeDriverEntry`函数如下：

![](https://bbs.kanxue.com/upload/attach/202011/36653_BZD3WE4Z2QSFMNF.jpg)

下面再次开启双机调试，同样在虚拟机里执行`sysmon -i`

![](https://bbs.kanxue.com/upload/attach/202011/36653_THF4ZXFVT69B2BQ.jpg)

![](https://bbs.kanxue.com/upload/attach/202011/36653_G972EZDVRSGXD34.jpg)

断点断在了回调函数的`DRIVER_OBJECT pDriverObj = DRIVER_OBJECT)PreInfo->Object;`

查看`PDriverObj`对象

![](https://bbs.kanxue.com/upload/attach/202011/36653_GN7STS8SFEFPFV5.jpg)

`U` 入口点的函数`u 0xfffff880`02ac2058`

![](https://bbs.kanxue.com/upload/attach/202011/36653_U5594SS94X5ERVY.jpg)

下面单步执行后会修改`sysmonDrv`驱动的`DriverInit`入口点

同时在`FakeDriverEntry`下断点

![](https://bbs.kanxue.com/upload/attach/202011/36653_UDF4BYZF47HT4QT.jpg)

直接`f5`, 断点就直接断在了我们的`Fake`函数里

![](https://bbs.kanxue.com/upload/attach/202011/36653_T46SPTE3AP57W2Y.jpg)

使用`kb`命令查看堆栈

![](https://bbs.kanxue.com/upload/attach/202011/36653_A3WF4A8BPG8BF4F.jpg)

确实执行了`call`，继续`F5`，这时我们观察`sysmon`命令行返回

![](https://bbs.kanxue.com/upload/attach/202011/36653_5B5WR39ZMXWGKF4.jpg)

`Sysmon installed.`

`SysmonDrv installed.`

`StartService failed for SysmonDrv`

`Failed to start the driver:`

`Stopping the service failed:`

`Sysmon`的驱动加载失败了，说明成功的控制了驱动的加载，从而证明了这种方案的可行。

|  |  |

| --- | --- |

| 1 | `总结，有些知识是已经众所周知的，但是更多的新的方法是需要我们在这些知识点反复研究反复揣摩，反复猜想，然后加以论证，才能获得意想不到额结果，当然今天这个注册回调不是随便就可以注册的，需要反复逆向去修改标志位实现的hack的方法去实现注册回调，具体读者自行研究，我这里只提供方法可行性。` |

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
