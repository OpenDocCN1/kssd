
```asm
.text:00401DF5                 push    0Ch             ; nVolumeNameSize
.text:00401DF7                 push    eax             ; lpVolumeNameBuffer
.text:00401DF8                 push    offset RootPathName ; "c:\\"
.text:00401DFD                 call    ds:GetVolumeInformationA
```

`WriteProcessMemory`、`GetProcAddress`、`CreateRemoteThread`、`GetProcAddress`、`FreeLibrary`

之后就是这几个病毒特有函数的的使用，基本完毕。

这样通过IDA走了一遍，大概知道这个病毒的流程还有动作。下面在虚拟机里面实体测试一下。

通过上面的截图，基本验证之前在IDA里的判断，还有System Repair Engineer的扫描日志。

（我个人两个浏览器chrome，还有世界之窗，用chrome编辑的图片，别人看到是红叉，这里就不上图了，我把图片链接放最底下）

下面就写下病毒的基本动作：

获取C盘的磁盘信息，获取系统盘路径，进而向`system32`下写入一个dll 和 一个exe 这两个的名字是一个随机的8位数，至于通过磁盘信息算出的8位数的算法， 由于个人是新手，分析不出来。之后dll插入系统的`winlogon.exe`进程。

而后，检测卡巴斯基，如果检测到了，通过修改时间过卡巴。（这里我通过C写了一个窗口名为卡巴斯基，进程名为`avp.exe`的东西，但病毒貌似不管这个，估计我的小程序，还是没达到卡巴的要求，呵呵）

下面开始注入`explorer`进程，并启动这个进程。并向各个盘符下写入`auto.exe`还有`autorun.inf`

```
[AutoRun]
open=auto.exe
shellexecute=auto.exe
shell\Auto\command=auto.exe
```

这个是提取出来的，呵呵，很经典了。

剩下的就是修该注册表，创建服务。基本过程就是这样了。

```asm
text:00401F39 push  offset aSystemCurrentc ; "SYSTEM\\CurrentControlSet\\Services\\"
```

注册表是这个地方遭到修改。

下面在用OD动态的走一遍。由于是动态的，很多东西，我也不是很熟悉，只能说也是大概的，有一个新手菜鸟的眼光看一下，希望得到大大们的指导。

```asm
004025A6  |.  53            push    ebx        ; /pModule
004025A7  |.  FF15 50304000 call    dword ptr [<&kernel32.GetMod>; \GetModuleHandleA
004025AD  |.  50            push    eax
004025AE  |.  E8 A3F0FFFF   call    00401656     //F7跟进程序
004025B3  |.  8945 98       mov     dword ptr [ebp-68], eax
004025B6  |.  50            push    eax                          ; /status
004025B7  |.  FF15 E4304000 call    dword ptr [<&msvcrt.exit>]   ; \exit
```

头有点晕，估计很多细节都没有把握好，不过这个病毒特征很明显，而且动作也不复杂，流程都出来了，就不用OD再走了，我估计剩下的每个call都跟进去，应当会很神伤。

休息一下。休息一下。![](https://bbs.kanxue.com/view/img/face/33.gif)

还希望能获得一个邀请码，能更深入的学习。

用到的API帮助（由于个人还处于初级菜鸟阶段，很多函数需要百度）

FindWindowExA     [af5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9^5x3o6x3I4x3#2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@3b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9^5x3o6x3I4x3#2)9J5k6h3S2@1L8b7`.`.)

GetModuleFileNameA        [6a8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5y4e0V1I4x3W2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@bb0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5y4e0V1I4x3W2)9J5k6h3S2@1L8h3H3`.)

GetSystemDirectoryA         [93aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1&6x3o6b7I4x3q4)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@9a3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1&6x3o6b7I4x3q4)9J5k6h3S2@1L8b7`.`.)

lstrcpy        [d54K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4U0b7%4x3g2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@3e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4U0b7%4x3g2)9J5k6h3S2@1L8b7`.`.)

PostMessageA      [f5bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9^5x3o6p5%4z5g2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@e13K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0l9^5x3o6p5%4z5g2)9J5k6h3S2@1L8b7`.`.)

lstrcat       [ddeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4U0M7@1y4g2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@7e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4U0M7@1y4g2)9J5k6h3S2@1L8b7`.`.)

lstrlen      [fe8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4K6p5I4y4#2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@2faK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0V1H3y4K6p5I4y4#2)9J5k6h3S2@1L8b7`.`.)

`lstrcmp`   The `lstrcmp` function compares two character strings.

CreateFileA     [9c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5z5o6M7#2z5g2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@689K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5z5o6M7#2z5g2)9J5k6h3S2@1L8b7`.`.)

WriteFile           [428K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1&6y4e0M7^5x3W2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@57aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1&6y4e0M7^5x3W2)9J5k6h3S2@1L8b7`.`.)

CloseHandle       [2fbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5z5o6M7#2y4W2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@b72K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5z5o6M7#2y4W2)9J5k6h3S2@1L8b7`.`.)

WinExec      [aacK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5y4U0R3^5x3W2)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@a32K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0t1^5y4U0R3^5x3W2)9J5k6h3S2@1L8b7`.`.)

SetServiceStatus      [c95K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6y4U0R3$3x3U0b7I4i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2R3`.](https://bbs.kanxue.com/elink@0b2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6y4U0R3$3x3U0b7I4i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2R3`.)

GetLastError       [d05K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0M7K6x3o6p5$3z5q4)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@12eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0M7K6x3o6p5$3z5q4)9J5k6h3S2@1L8b7`.`.)

RegisterServiceCtrlHandlerA         [52bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6y4U0R3#2x3o6f1@1i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2R3`.](https://bbs.kanxue.com/elink@c1dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3L8r3W2T1M7X3q4J5P5g2)9J5c8X3#2K6y4U0R3#2x3o6f1@1i4K6t1^5g2W2y4Q4x3X3f1^5y4g2)9J5z5g2)9J5k6h3q4K6M7s2R3`.)

CreateThread     [d16K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0p5&6x3e0b7@1y4q4)9J5k6h3S2@1L8b7`.`.](https://bbs.kanxue.com/elink@6a1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0S2K9h3E0W2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6$3K9h3g2%4i4K6u0r3x3e0p5&6x3e0b7@1y4q4)9J5k6h3S2@1L8b7`.`.)
