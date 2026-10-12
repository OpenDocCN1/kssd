
`UserTime` 00:00:00.875

`KernelTime` 00:00:01.921

`Win32 Start Address` 0x00446e16

`Start Address` 0x7c810867

`Stack Init` `f9460000` `Current` `f945fcb8` `Base` `f9460000` `Limit` `f945a000` `Call` `0`

`Priority` `10` `BasePriority` `8` `PriorityDecrement` `0` `DecrementCount` `16`

`ChildEBP` `RetAddr` `Args to Child`

`WARNING:` `Stack unwind information not available. Following frames may be wrong.`

`f945fbac` `f72ca325` `81b7d878` `0012da20` `81725828` `XueTr+0x2b21`

`f945fbec` `f72fe0c0` `0012da20` `00000008` `00000000` `XueTr+0x3325`

`f945fc34` `804eedf9` `81475978` `814032a8` `806d12d0` `XueTr+0x370c0`

`f945fc44` `80574b42` `81403318` `8166b660` `814032a8` `nt!IopfCallDriver+0x31` (`FPO:` `[0,0,0]`)

`f945fc58` `805759d1` `81475978` `814032a8` `8166b660` `nt!IopSynchronousServiceTail+0x60` (`FPO:` `[Non-Fpo]`)

`f945fd00` `8056e33c` `00000138` `00000000` `00000000` `nt!IopXxxControlFile+0x5e7` (`FPO:` `[Non-Fpo]`)

`f945fd34` `8053d808` `00000138` `00000000` `00000000` `nt!NtDeviceIoControlFile+0x2a` (`FPO:` `[Non-Fpo]`)

`f945fd34` `7c92eb94` `00000138` `00000000` `00000000` `nt!KiFastCallEntry+0xf8` (`FPO:` `[0,0]` `TrapFrame @ f945fd64`)

`0012d96c` `7c92d8ef` `7c801671` `00000138` `00000000` `ntdll!KiFastSystemCallRet` (`FPO:` `[0,0,0]`)

`0012d970` `7c801671` `00000138` `00000000` `00000000` `ntdll!ZwDeviceIoControlFile+0xc` (`FPO:` `[10,0,0]`)

`0012d9d0` `00414d7f` `00000138` `006e0160` `0012da28` `0x7c801671`

`0012d9d4` `00000000` `006e0160` `0012da28` `00000018` `0x414d7f`

```
kd> g
```

`Breakpoint 2 hit` // 过了大概半秒钟，系统被`Windbg`第二次断住，为雨过天晴的行为。观察之。

`*** ERROR:` `Module load completed but symbols could not be loaded for Shieldf.sys`

`Shieldf+0x3b4:`

`f9c2c3b4` `c9` `leave`

```
kd> !thread
```

`THREAD` `8143b520` `Cid` `06ec.06f0` `Teb:` `7ffdf000` `Win32Thread:` `e1abc730` `RUNNING on processor 0`

`IRP List:`

`814da538:` `(0006,01d8)` `Flags:` `00000884` `Mdl:` `00000000`

`Not impersonating`

`DeviceMap` `e1859328`

`Owning Process` `8143b798` `Image:` `shieldtray.exe`

`Wait Start TickCount` `107179` `Ticks:` `0`

`Context Switch Count` `352` `LargeStack`

`UserTime` `00:00:00.015`

`KernelTime` `00:00:00.140`

`Win32 Start Address` `shieldtray` `(0x0044a981)`

`Start Address` `kernel32!BaseProcessStartThunk` `(0x7c810867)`

`Stack Init` `f7ada000` `Current` `f7ad9968` `Base` `f7ada000` `Limit` `f7ad7000` `Call` `0`

`Priority` `8` `BasePriority` `8` `PriorityDecrement` `0` `DecrementCount` `0`

`ChildEBP` `RetAddr` `Args to Child`

`WARNING:` `Stack unwind information not available. Following frames may be wrong.`

`f7ad99fc f9c2da17 814da548 814da538 805452fe Shieldf+0x3b4`

`f7ad9a4c f9c2de7b 814cf368 814da548 81bd3030 Shieldf+0x1a17`

`f7ad9b50 805b465e 81b79a20 00000000 816f4228 Shieldf+0x1e7b` // 雨过天晴的文件过滤驱动

`f7ad9bd8` `805b0b3f` `00000000` `f7ad9c18` `00000040` `nt!ObpLookupObjectName+0x56a` (`FPO:` `[Non-Fpo]`)

`f7ad9c2c` `8056b133` `00000000` `00000000` `4b31f001` `nt!ObOpenObjectByName+0xeb` (`FPO:` `[Non-Fpo]`)

`f7ad9ca8` `8056baaa` `0012fc04` `00100001` `0012fbd0` `nt!IopCreateFile+0x407` (`FPO:` `[Non-Fpo]`)

`f7ad9d04` `8056f291` `0012fc04` `00100001` `0012fbd0` `nt!IoCreateFile+0x8e` (`FPO:` `[Non-Fpo]`)

`f7ad9d44` `8053d808` `0012fc04` `00100001` `0012fbd0` `nt!NtOpenFile+0x27` (`FPO:` `[Non-Fpo]`)

`f7ad9d44` `7c92eb94` `0012fc04` `00100001` `0012fbd0` `nt!KiFastCallEntry+0xf8` (`FPO:` `[0,0]` `TrapFrame @ f7ad9d64`)

`0012fb80` `7c92dd09` `7c81e74a` `0012fc04` `00100001` `ntdll!KiFastSystemCallRet` (`FPO:` `[0,0,0]`)

`0012fb84` `7c81e74a` `0012fc04` `00100001` `0012fbd0` `ntdll!NtOpenFile+0xc` (`FPO:` `[6,0,0]`)

`0012fbfc` `7c82744d` `00000000` `0012fc34` `0012fc3c` `kernel32!GetDiskFreeSpaceExW+0x7d` (`FPO:` `[Non-Fpo]`)

`0012fc14` `0042cbe8` `0012fc74` `0012fc34` `0012fc3c` `kernel32!GetDiskFreeSpaceExA+0x2f` (`FPO:` `[Non-Fpo]`)

`0012fc40` `0041180e` `0012fc74` `0012fc80` `0012fc84` `shieldtray+0x2cbe8`

`00000000` `00000000` `00000000` `00000000` `00000000` `shieldtray+0x1180e`

打开`IDA`，载入`Shiedlf.sys`，定位到`0x103b4`处，即可看出其具体的流程：

![](img/41d0d35384c29cc0bc5bd9cd0ee74cf3.jpg)

于是在`Windbg`中，强行将其改成如下图所示的代码：

![](img/766502e69016ff970064448434018c5a.jpg)

此时再用`XueTr.exe`删除雨过天晴的设备，调用`Winhex`成功读取到更改后的`MBR`。

![](img/ca0d8299fea8dcdc5dd645ee168eeebf.jpg)

![](img/be84352b7d7024d8b1f85f5ed11e62.jpg)

若我们不做这些更改，用`Winhex`得到的内容如下：

![](img/4ea2a7148a8d5021f82d5baef4c53126.jpg)

现在开启`Bochs`，即可调试雨过天晴的启动代码了。结合`IDA`静态看，很容易了解它的`MBR`所做的工作。通过这次调试跟踪，对自己是个小小的锻炼。其实用调试器改还不如写个驱动来的容易，不过最终目的达到了，嘿嘿。接下来我准备写个驱动，一劳永逸….

![](img/7691754d525a92d5c8b1f85f5ed11e62.jpg)

`此实例旨在告诉大家遇到问题如何一步一步跟踪解决，灵活变通，重在积累调试经验！`
