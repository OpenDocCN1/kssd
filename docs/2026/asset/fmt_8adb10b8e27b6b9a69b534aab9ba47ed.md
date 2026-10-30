
```
kd> !peb
PEB at 0000001eb0c75000
kd> dt ntdll!_PEB KernelCallbackTable
0000001eb0c75000 +0x058 KernelCallbackTable : 0x00007ffe`bc6f2070 Void
```

查看`KernelCallbackTable`表项

![](https://bbs.kanxue.com/upload/attach/202204/762319_T48NXWEU2RQ3AR5.png)

我们需要查看123项的内容，如下：

![](https://bbs.kanxue.com/upload/attach/202204/762319_7K6YQAVBPZ36B42.png)

调试运行`HookUserModeCallBack`函数后，再次查看：

![](https://bbs.kanxue.com/upload/attach/202204/762319_ZERMVT2KWHY9Z9G.png)

在自定义的回调函数`MyxxxClientAllocWindowClassExtraBytes`中

![](https://bbs.kanxue.com/upload/attach/202204/762319_SD4UZC2BPC39QRD.png)

接着下断点：

![](https://bbs.kanxue.com/upload/attach/202204/762319_F372CQ4WG3XW5KU.png)

并且在`MyxxxClientAllocWindowClassExtraBytes`函数中下断点：

![](https://bbs.kanxue.com/upload/attach/202204/762319_6ZMJA3VRGSBG5P9.png)

在调试器中输入`g`运行，现在运行到如下位置：

![](https://bbs.kanxue.com/upload/attach/202204/762319_VX2QR3JRP2JV54Y.png)

在运行`NtUserConsoleControl`前后分别查看窗口2的模式：

![](https://bbs.kanxue.com/upload/attach/202204/762319_ESNPWNWHXX2F4AK.png)

继续按`g`运行，中断在`SetWindowLong`函数前

![](https://bbs.kanxue.com/upload/attach/202204/762319_PWWA3948QFR7TFK.png)

此时窗口2处于模式1，并且`pExtraBytes`为窗口0的`OffsetToDesktopHeap`，再调用`SetWindowLong`函数：

![](https://bbs.kanxue.com/upload/attach/202204/762319_7ZGMZDDCZEY7JZK.png)

这是第一次越界写，第一个参数为窗口2的句柄，第二个参数为`index`，为`cbWndExtra`相对`tagWNDK`结构体首地址的偏移量+`cbWndServerExtra`，由于窗口2调用了`NtUserMessageCall`，所以`cbWndServerExtra`为`0x10`，调用`SetWindowLong`时会使用`index - cbWndServerExtra`，所以此处要加上`cbWndServerExtra`来抵消，可参考前文`SetWindowLong`函数的分析。

单步运行后

![](https://bbs.kanxue.com/upload/attach/202204/762319_QDB8YASZ5EQ32UU.png)

可以看到窗口0的`cbWndExtra`变成了`0xFFFFFFFF`，接下来对窗口0调用`SetWindowLong`时传入`index`可以传入之前计算得到的`extra_to_wnd1_offset`和`extra_to_wnd2_offset`来分别修改窗口1和窗口2的窗口内核数据。

此时窗口1处于直接寻址模式，对窗口0调用`SetWindowLongPtr`修改窗口1的`pExtraBytes`为任意值，使用`SetWindowLongPtr`是因为此函数第三个参数可以传入64位数据，将窗口1的`pExtraBytes`设置为任意值，接下来对窗口1调用`SetWindowLong`即可实现任意地址写数据。

## 两种提权方式

### 8.1 设置token

第一种为设置当前进程的`token`为system进程的`token`，将当前进程提升到system权限，这种需要读取进程的`EPROCESS`结构，再定位到`token`变量的地址，修改`token`，公开的EXP中使用`GetMenuBarInfo`函数来实现内核任意地址读原语。

我们先分析这种方式，先看下`Menu`内核结构体：

```
ptagWND
0x10 THREADINFO
0x1A0 PROCESSINFO
0x00 EPROCESS
0x18 unknown
0x80 kernel desktop heap base
0x28 ptagWNDk
0xA8 spMenu
0x28 obj28
0x2C cItems(for check) 设置为1
0x40 cxMenu(for check) 设置为1
0x44 cyMenu(for check) 设置为1
0x50 ptagWND
0x58 rgItems
0x00 unknown(for exploit) //要读的地址-0x40
0x98 ppMenu
0x00 pSelf //指向spMenu
```

在EXP中先构造一个假的`Menu`

![](https://bbs.kanxue.com/upload/attach/202204/762319_RVNNN9BUGVBTF4P.png)

其中401行设置`ppMenu`偏移`0x00`处的值为`spMenu`，404、408、409设置`spMenu`结构体内部数据是为了绕过`GetMenuBarInfo`的验证，`GetMenuBarInfo`函数会调用内核中的`NtUserGetMenuBarInfo`，最终调用到`xxxGetMenuBarInfo`，`GetMenuBarInfo`对应有四个参数，对应`xxxGetMenuBarInfo`的四个参数，其中参数2为`idObject`，参数3为`idItem`。

`xxxGetMenuBarInfo`对参数有校验：

![](https://bbs.kanxue.com/upload/attach/202204/762319_XTQGDHW7VC8MEYR.png)

164行判断`idObject != -3`如果满足，就不能触发到下面读内存的代码路径，所以`idObject`必须为`-3`。

![](https://bbs.kanxue.com/upload/attach/202204/762319_GKW43V22F4JKXVM.png)

316行代码判断`dwStyle`不能包含`WS_CHILD`属性。

322行代码从`spMenu`中偏移`0x98`取值，赋值给`ppMenu`。

325行代码判断`idItem`不能小于`0`。

328行代码判断`idItem`不能大于`spMenu`偏移`0x28`取值再偏移`0x2c`取值。

335行代码判断`spMenu`偏移`0x40`取值不为`0`并且偏移`0x44`取值不为`0`。

338行到344行，如果`idItem`不为`0`，可以让`idItem`为`1`，那么`_readAddrSub40`的值为`spMenu`偏移`0x58`取值。

接下来程序进入353行

![](https://bbs.kanxue.com/upload/attach/202204/762319_AVEJCXS6TPB9HME.png)

`v5`是传入的第四个参数，用作保存读取到的数据。

在353、354行，可以读取传入地址的数据+窗口`RECT`的`left`坐标。

在357、358行，可以读取传入地址的数据+`4`+窗口`RECT`的`top`坐标。

所以只要我们可以绕过构造一个假的`Menu`，绕过上述限制，在`Menu`偏移`0x58`再偏移`0x00`的地址处存放想读取的地址 - `0x40`，当`GetMenuBarInfo`返回时`left`和`top`中保存的就是目标地址处的8字节数据。

要想替换窗口的`Menu`为假的`Menu`，还是需要用到`SetWindowLong`函数，在内核态`win32kfull!xxxSetWindowLong`函数中会调用`xxxSetWindowData`函数：

![](https://bbs.kanxue.com/upload/attach/202204/762319_5RVM7V3Q2SD4N2Q.png)

`xxxSetWindowData`函数如下：

![](https://bbs.kanxue.com/upload/attach/202204/762319_ST7B3KM4DZ6JNUD.png)

134、136行，判断如果`index`为`0xFFFFFFF4`，为`-12`，对应为`GWLP_ID`。

![](https://bbs.kanxue.com/upload/attach/202204/762319_HF5TUQPAKND6DMC.png)

138行判断如果`dwStyle`是否包含`WS_CHILD`属性。

140行取出原来的`menu`指针，赋值给`retValue`，最终会作为用户态`SetWindowLong`函数的返回值。

142行修改`spMenu`为`SetWindowLong`传入第三个参数`newValue`值。

所以我们需要如下步骤才能完成任意地址读：

1. 先对窗口0使用内核越界写修改窗口1的`dwStyle`值为包含`WS_CHILD`，这样调用`SetWindowLong`时即可绕过上面138行的判断。
2. 对窗口1使用`SetWindowLong`函数传入`index`为`GWLP_ID`，修改窗口1的`Menu`为构造的假的`Menu`，并且`SetWindowLong`会返回原先的`Menu`的地址。
3. 使用原先的`Menu`通过内核数据结构即可定位到当前进程的`EPROCESS`，进而定位到`token`的地址。
4. 再次对窗口0使用内核越界写修改窗口1的`dwStyle`值为不包含`WS_CHILD`，这样调用`GetMenuBarInfo`时可以绕过`xxxGetMenuBarInfo`中316行代码的判断。
5. 需要读取数据时，将目标地址 - `0x40`赋值给假的`Menu`偏移`0x58`对应的内存空间中，再调用`GetMenuBarInfo`函数。

单步运行413行代码，窗口1的`dwStyle`就包含了`WS_CHILD`属性。

![](https://bbs.kanxue.com/upload/attach/202204/762319_UR75UDYQKBRV5N8.png)

![](https://bbs.kanxue.com/upload/attach/202204/762319_QTTXVUZDRQV7EAN.png)

可以看到修改完成后，窗口1的`dwStyle`包含了`WS_CHILD`属性。

继续执行415行代码：

![](https://bbs.kanxue.com/upload/attach/202204/762319_6RQCRN9769YCK8J.png)

在416行下断点后运行：

![](https://bbs.kanxue.com/upload/attach/202204/762319_ZTJJRH24FV682QW.png)

此时`SetWindowLong`函数刚执行完毕，返回值`rax`为`0xfffffa49c0821e60`，保存的是旧的`spMenu`指针，而根据之前的数据结构，可以使用`spMenu`定位到当前进程的`EPROCESS`。

执行419行代码，移除窗口1的`WS_CHILD`属性，为接下来调用`GetMenuBarInfo`做准备

![](https://bbs.kanxue.com/upload/attach/202204/762319_EXW423W7FXWR77E.png)

![](https://bbs.kanxue.com/upload/attach/202204/762319_PBGXSRUG2HK56ZS.png)

窗口1的`dwStyle`移除了`WS_CHILD`属性。

然后构造读原语如下：

![](https://bbs.kanxue.com/upload/attach/202204/762319_9RP55UUZXRKNJSE.png)

根据之前的数据结构

```
ptagWND
0x10 THREADINFO
0x1A0 PROCESSINFO
0x00 EPROCESS
0x18 unknown
0x80 kernel desktop heap base
0x28 ptagWNDk
0xA8 spMenu
0x50 ptagWND
```

所以获取到`spMenu`后可以使用如下代码来获取当前进程的`EPROCESS`

![](https://bbs.kanxue.com/upload/attach/202204/762319_TDJ58DTKYRP657S.png)

在调试器中查看如下：

![](https://bbs.kanxue.com/upload/attach/202204/762319_D3SWXFYM252PHV7.png)

上图中可以看到通过`spMenu`取偏移和使用命令`.process`两种方式获取到的`EPROCESS`值是一致的。

查看当前进程的`token`

```
kd> !token
...
Privs:
19 0x000000013 SeShutdownPrivilege               Attributes -
23 0x000000017 SeChangeNotifyPrivilege           Attributes - Enabled Default
25 0x000000019 SeUndockPrivilege                 Attributes -
33 0x000000021 SeIncreaseWorkingSetPrivilege     Attributes -
34 0x000000022 SeTimeZonePrivilege               Attributes -
...
```

我们直接运行到454行，此时当前进程的`token`被替换为系统`token`

![](https://bbs.kanxue.com/upload/attach/202204/762319_C9A4985HR9PFDBR.png)

`EPROCESS`中`token`结构体为`_EX_FAST_REF`

```
kd> dt _EX_FAST_REF
ntdll!_EX_FAST_REF
+0x000 Object           : Ptr64 Void
+0x000 RefCnt           : Pos 0, 4 Bits
+0x000 Value            : Uint8B
```

调试运行到454行，重新运行一次，所以`EPROCESS`值与之前不一样。

![](https://bbs.kanxue.com/upload/attach/202204/762319_PARJFUHCVAQQ565.png)

可以看到此时调用到`if (iCount < 5000)`，`_EX_FAST_REF`结构体中的`object`值已经修改了。

查看system进程的`EPROCESS`

![](https://bbs.kanxue.com/upload/attach/202204/762319_7G3AYX9KSHATFYS.png)

```
kd> dt nt!_EX_FAST_REF ffffe504`89885080+0x4b8
+0x000 Object           : 0xffffbe09`9a242744 Void
+0x000 RefCnt           : 0y0100
+0x000 Value            : 0xffffbe09`9a242744
```

system进程`_EX_FAST_REF`的`Object`也为``0xffffbe09`9a242744``，当前进程修改成功，使用`!token`命令验证下：

![](https://bbs.kanxue.com/upload/attach/202204/762319_X4JPS8EEMGR2HP7.png)

修改`token`的代码如下：

6. `EPROCESS`结构体中有一个进程链表，保存了当前系统的所有进程，我们主要关注`ActiveProcessLinks`和`UniqueProcessId`属性
