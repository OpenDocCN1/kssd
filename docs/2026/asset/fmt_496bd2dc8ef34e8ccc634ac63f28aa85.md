
# [原创]双机调试环境搭建

文章开头！存宝！二木哥！小路哥哥！曾哥！砍老板！龟哥！南梦师傅！hook酱！梦游师傅！爱你们！好的表白结束进入正题！

## 步骤

首先要安装`sdk`，`vs2019`里自带了，安装最新的版本

![](upload/attach/202012/873515_H73JMB3N6GHK2XD.png)

下载`wdk`，安装

<mark class="encrypted">1d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6s2c8Z5k6g2)9J5k6s2N6V1K9H3`.`.</mark>

![](upload/attach/202012/873515_HW49JKTFWPNVHV5.png)

`intsall` 直接在`vs`中安装了插件

![](upload/attach/202012/873515_5Y98JG44YUVC8KW.png)

`install`后看一下`vs`中的扩展里的驱动有没有安装成功

![](upload/attach/202012/873515_DW5QGMDKB2MF3JR.png)

安装成功后测试一下，写一个`hello world`试一下，在之前需要配置一下

先创建一个新的项目----> `kernel Mode Driver`, `Empty` 才是空的项目，上面的那个有模板

![](upload/attach/202012/873515_GFZZHKPAHPH6EMY.png)

正常的把警告视为错误的选项我们开启一下，因为刚学对自己比较有好处

需要修改选项：

`C/C++` ----> 代码生成 ----> `Spectre Mitigation` ----> `Disabled` （检查代码最后有没有漏洞的应该是网上没看到相关的资料）

`Driver Settings` ----> `General` ----> `Os version` 改成相应平台的 / `Target Platform` 改成`Desktop`

`Inf2Cat` (`Inf2Cat.exe`) 是一个命令行工具，该工具确定驱动程序包的 INF 文件是否可以针对指定的 Windows 版本列表进行数字签名，如果可以，那么 `Inf2Cat` 会生成适用于指定 Windows 版本的未签名的目录文件，直接关闭

`Wpp Tracing` 日志，关掉

`Driver sigging off`

### 测试代码：hello world

```c
#include <ntddk.h>
VOID DriverUnload(PDRIVER_OBJECT driver)
{
UNREFERENCED_PARAMETER(driver);
DbgPrint("unloading sucessful \n");
}
NTSTATUS DriverEntry(PDRIVER_OBJECT driver , PUNICODE_STRING reg_path)
{
UNREFERENCED_PARAMETER(reg_path);
DbgPrint("Hello world!\n");
driver->DriverUnload = DriverUnload;
return STATUS_SUCCESS;
}
```

![](upload/attach/202012/873515_29PDD673AHP3TFH.png)

## 搭建虚拟机环境

镜像文件：下载一定要下载`sp1` 镜像不然你就安装不了`tools`操作系统需要更新！

```
链接：https://pan.baidu.com/s/1i_WWE8bYvI6ldl211mtX8Q
提取码：4bsu
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

需要用的软件：

`VirtualKD-Redux`：<mark class="encrypted">7bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5@1k6o6j5I4y4K6t1$3j5W2)9J5c8W2k6A6M7Y4c8#2j5h3I4w2c8q4)9J5k6q4u0W2k6s2g2^5</mark>

因为我们的虚拟机`15`以上不太兼容，所以`github`上给了新的版本直接下载就好

![](upload/attach/202012/873515_P45MNJ3XQ9US4MR.png)

将`target64`放入虚拟机（`32`位的自行更换）

![](upload/attach/202012/873515_XCVMVMTTHSFE335.png)

驱动需要加载：

```
链接：https://pan.baidu.com/s/1Z_9n51Vt9bJgQ_53mPUkFA
提取码：wwsx
复制这段内容后打开百度网盘手机App，操作更方便哦--来自百度网盘超级会员V4的分享
```

需要下载`KmdManager`

`dbgview`：<mark class="encrypted">026K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4y4&6M7$3W2F1N6r3g2J5L8X3q4D9M7#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1M7#2)9J5c8X3c8W2j5Y4g2Y4N6X3W2W2N6H3`.`.</mark>

在虚拟机中 打开

![](upload/attach/202012/873515_V6HKXT4CMYS7FDK.png)

重启后`f8`

![](upload/attach/202012/873515_34PJ2WGHA5ZSURR.png)

在物理实体机：

![](upload/attach/202012/873515_GDC45H7Q5D2EER5.png)

可以看到有包了

![](upload/attach/202012/873515_FGN2FGWUXE97MVF.png)

### 测试 hello world 的驱动

切记这些都需要管理员启动：

![](upload/attach/202012/873515_89MYE3W2JN7JQ7B.png)

我们写代码下个段：`DbgBreakPoint();`

![](upload/attach/202012/873515_J9W4JTVUEKJMJGP.png)

调试`f8`和`f11`都是单步步入，`f10`是单步步过 `shift+f11`运行到返回

`r`可以查看寄存器

![](upload/attach/202012/873515_DXPPC22UZB5Q7XV.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
