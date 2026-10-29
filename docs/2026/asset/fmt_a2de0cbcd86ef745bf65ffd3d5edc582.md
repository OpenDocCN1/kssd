
# [原创] `JNI_OnLoad` 与 `init_array` 下断方法整理

`JNI_OnLoad` 和 `init_array` 在 SO 脱壳中占了很多比重，那么如何在它们执行前下断点呢？

考虑到 SO 加载的时机，在 `jdb` 附加之前并不能对我们想调试的 SO 下断点，即使是知晓了偏移位置。因为它们还没有被加载。据说这可以用脚本来解决，这不是这篇文章讨论的范围。本文章讨论的方法是从系统库中下断的方法来达到目的。

## 首先讨论 `init_array` 下断点的方法

`init_array` 是在 SO 加载后由 `linker` 负责调用，详细细节可以翻阅 `linker` 源码。

`linker` 在调用 `init_array` 的时候会输出“`[ Calling %s @ %p for ‘%s’ ]`”。

思路是可以通过定位该字符串来定位调用 `init_array` 的位置。

```bash
adb pull /system/bin/linker
```

下载手机中的 `linker`，将下载来的 `linker` 拖入 IDA 中分析。等待 IDA 分析完后打开字符串表，搜索“`[ Calling %s @ %p for ‘%s’ ]`”，为了方便记忆，直接搜索“`call`”也是可以的。

搜索出来后双击记录，下面是结果：

![](https://bbs.kanxue.com/upload/attach/201607/617255_28884e76ca4b02ee.png)

双击红色箭头处的 `o` 可以转到引用该字符串的地方。

![](https://bbs.kanxue.com/upload/attach/201607/617255_0042aa1c57c5c25a.png)

`BLX R4` 是调用代码 `Init_array` 的地方。记录下此地偏移地址（`0x15bc`），留作以为备用。

接下来在动态调试的时候，使用 `am` 以等待调试器附加的状态启动 app，然后用 IDA 附加。

附加后在 IDA 的 module list 中搜索 `linker` 这一项。

![](https://bbs.kanxue.com/upload/attach/201607/617255_4d21cfdb479a2a25.png)

`Base` 这一栏是 `linker` 在内存中加载的基地址，基地址 = `0xB6FCB000`。

所以调用 `init_array` 的地方 `blx r4` 在该内存中的偏移为 `Base + offset = 0xB6FCB000 + 0x15BC`。

在反汇编窗口按下 G 键，输入 `0xB6FCB000 + 0x15BC`，即可跳转过去，并设置好断点。

如果跳过去后，目标并不是汇编代码，请机智地按下 C 键。

注意，这个时候系统库基本上都已经加载！注意，`0x15BC` 这个偏移请牢记，下次可以直接用。

断点下好后，请用 `jdb` 附加，进程跑起来~

## 下面简介 `JNI_OnLoad` 函数下断方法

模仿 `init_array` 的思路，我们就想是不是可以在 `JNI_OnLoad` 的调用处下断点，这个思路完全正确。

首先，我们要确定 `JNI_OnLoad` 何时被调用。

`JNI_OnLoad` 实际是在 `LoadNativeLibrary` 中调用的，详细的分析文章可以在看雪论坛找到。

`LoadNativeLibrary` 通过 `dlsym` 查找 `JNI_OnLoad` 地址，所以也可以直接搜索“`JNI_OnLoad`”定位。

`LoadNativeLibrary` 函数在什么库？

- Android 5.0 + `libart.so`
- Android 5.0 - `libdvm.so`（`dvmLoadNativeCode`）

请看官对号入座，6.0 我不了解~

使用：

```bash
adb pull system/lib/libart.so
```

或：

```bash
adb pull system/lib/libdvm.so
```

将下载的文章载入到 IDA，在 Export 窗口搜索 `LoadNativeLibrary` 或字符串窗口搜索“`JNI_OnLoad`”。

注意 `LoadNativeLibrary` 是 `JavaVMExt` 类成员函数，所以搜出来有点怪，实际上就是它。

双击进入并按 F5 输出 C 源代码，`LoadNativeLibrary`/`dvmLoadNativeCode` 代码量很大！

肉眼扫描：

```c
v67 = dlsym(handle, "JNI_OnLoad");
if (v67)
{
    .......
    v85 = v67(v179, 0); // 在此处按下 Tab 键，跳到汇编代码。
    ........
}
```

跳到汇编代码后：

```asm
.text:001DDE02 BLX R7
```

这里为调用处，记录下偏移，剩下的事情参考 `init_array`。

另外博客是我发的，`9dfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3f1r3q4F1k6r3q4a6f1#2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0t1H3x3o6R3H3x3K6M7`。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
