
# 在可执行文件中添加菜单项

> 原文：[`www.pediy.com/kssd/pediy06/pediy6288.htm`](https://www.pediy.com/kssd/pediy06/pediy6288.htm)

在可执行文件中添加菜单项 - by Fenri

原文来源：

The Codebreakers-Magazine
Issue 2 - 2003

[`www.reverse-engineering.net`](http://www.reverse-engineering.net)

[`codebreakers.anticrack.de`](http://codebreakers.anticrack.de)

[`codebreakers.reverse-engineering.net`](http://codebreakers.reverse-engineering.net)

(C) The Reverse-Engineering-Network
2003
21st July 2003

写这篇短文是因为有许多人在 `anticrack.de` 的 REA 论坛提出了这个问题。我希望借此机会提供一些有用的信息去帮助他们。我用一个简单的 Hello World 应用程序示范，它使用 MS VC++编写。

任务：添加允许的‘打开’ 菜单项和变灰的‘关闭’ 菜单项在退出菜单项之前。

工具：`hex editor` (例如 `Hiew`), 大脑(如你平常一般使用)

## Ok, 开始工作

使用你熟悉和惯用的十六进制编辑工具（如 `Hiew`）打开 `Hello.exe`， 直接到资源区（`.rsrc`). 如果使用 `hiew` 则按 `F8`, 再按 `F6`，进入`.rsrc`。（译注：使用 `EXESCOPE` 参见`.rsrc` 中的 pointer to RAW data 也表示这个偏移值）。 这样，你就看到了 `IMAGE_RESOURCE_DIRECTORY`(资源目录)，它的结构如下：

```c
typedef struct _IMAGE_RESOURCE_DIRECTORY {
    ULONG Characteristics;
    ULONG TimeDateStamp;
    USHORT MajorVersion;
    USHORT MinorVersion;
    USHORT NumberOfNamedEntries;
    USHORT NumberOfIdEntries;
} IMAGE_RESOURCE_DIRECTORY, *PIMAGE_RESOURCE_DIRECTORY;
```

`ULONG` = 4 bytes, `USHORT` = 2 bytes 共 16 个字节

这些东西对我们没什么大用，但你需要知道，哪里是后面数据的开始。因此，向前 16 个字节你会看到一些 `IMAGE_RESOURCE_DIRECTORY_ENTRIES` （资源目录入口地址）, 它的结构：

```c
typedef struct _IMAGE_RESOURCE_DIRECTORY_ENTRY {
    ULONG Name;
    ULONG OffsetToData;
}
```

typedef struct _IMAGE_RESOURCE_DIRECTORY_ENTRY {

以十六进制格式，你将看到(类似的数据):

（译注：为便于分析，翻译时使用了 `win2003` 下的 `NOTEPAD` `5.2` 作为演示样例）

```text
xx 00 00 00 xx 00 00 xx xx 00 00 00 xx 00 00 xx
 1  2  3  4  5  6  7  8    9 10 11 12 13 14 15 16

00008000h: 00 00 00 00 00 00 00 00 04 00 00 00 00 00 08 00 ;
00008010h: 03 00 00 00 50 00 00 80 04 00 00 00 A8 00 00 80 ;

03 00 00 00-50 00 00 80 - 04 00 00 00-A8 00 00 80
                             1  2  3  4  5  6  7  8
...
```

每个 `IMAGE_RESOURCE_DIRECTORY_ENTRY`（资源目录入口） 是 8 字节长。在给出的实例数据中，前 8 个字节（03－80）对于我们来说并不重要（它通常用于: 图标 `Icon`, 菜单 `Menu`, 对话框 `Dialog...`），但字节从‘04’到‘80’（参见上面的示意数据，即 9－16 位）是非常重要的。

从 9－16 位，其中前 4 个字节代表名称（对于我们没有用），第 5－8 位字节代表偏移量，如果这个重要的标志位即数据的最高位是 1, 表示指向另一个 `IMAGE_RESOURCE_DIRECTORY`（资源目录）， 否则，它指向 `IMAGE_RESOURCE_DATA_ENTRY`（真正我们需要发现的位置）。

在我们的例子中，数据是 `800000A8` （`A8 00 00 80`），最重要的标识位被设置为 1（数据最高位），这表示，`A8` 是从`.rsrc` 区段开始到一个 `IMG_RES_DIR`（资源目录）的偏移量。

经检测，资源段开始的偏移是 `8000`，则在 `80A6` 处放置的是另一个 `IMG_RES_DIR`(资源目录)，就像前面提过的，它也不重要，继续跳过 16 字节，你能看到下一个 `IMG_RES_DIR_ENTRY`（资源目录入口）。

```text
000080a0h: 09 00 00 00 48 02 00 80-00 00 00 00 00 00 00 00 ;
000080b0h: 04 00 00 00 00 00 01 00-01 00 00 00 60 02 00 80 ;
```

跳过前 4 个字节，它代表名字，后 4 个字节是 `80000260` （`60 02 00 80`），标志位依然是 1，那继续到 `8260`（`8000＋260`）, 那里是下一个 `IMG_RES_DIR`（资源目录）,再跳过 16 字节，又是一个 `IMG_RES_DIR_ENTRY`（资源目录入口），跳过 4 字节(名字), 看到数据是 `00000440`.

```text
00008260h: 00 00 00 00 00 00 00 00 04 00 00 00 00 00 01 00 ;
00008270h: 04 08 00 00 40 04 00 00 00 00 00 00 00 00 00 00 ;
```

好了，标志位为 0，因此在 `00008440`(`8000+440`)处就是我们要找的目标了－`IMAGE_RESOURCE_DATA_ENTRY`（资源数据入口），它的结构是

```c
typedef struct _IMAGE_RESOURCE_DATA_ENTRY {
    ULONG OffsetToData;
    ULONG Size;
    ULONG CodePage;
    ULONG Reserved;
} IMAGE_RESOURCE_DATA_ENTRY, *PIMAGE_RESOURCE_DATA_ENTRY;
```

只有前两个域对我们来说是重要的，数据是 `0011750` （数据偏移）和 `330` （尺寸），我们一会儿要用到这些数据，因此记下 `IMAGE_RESOURCE_DATA_ENTRY`（资源数据入口）的地址，在我们的实例中是 `8440`.

```text
00008440h: 50 17 01 00 30 03 00 00 E4 04 00 00 00 00 00 00 ; P...0...&.......
           数据偏移    尺寸
```

（译注：当尺寸第二位不为 0 时，数据偏移地址应减去该值 `11750－300＝0e750`，原文未说明，经观察发现，可能不正确，更多细节，参见有关技术资料）

现在，到了 `0E750`，你应该可以在这里看见菜单资源：

```text
0000e750h: 00 00 00 00 10 00 87 65 F6 4E 28 00 26 00 46 00 ; ......‡e&N(.&.F.
0000e760h: 29 00 00 00 00 00 01 00 B0 65 FA 5E 28 00 26 00 ; ).......°eú^(.&.
0000e770h: 4E 00 29 00 09 00 43 00 74 00 72 00 6C 00 2B 00 ; N.)...C.t.r.l.+.
0000e780h: 4E 00 00 00 00 00 02 00 53 62 00 5F 28 00 26 00 ; N.......Sb._(.&.
```

### 菜单项的结构

```text
00 00|00 00|10 00| 87 65 F6 4E 28 00 26 00 46 00  ;
属性   ID    菜单项名称
```

前两个字节 – 不知道用途，可能仅起分隔符作用

3－4 字节 - 属性 (描述见后)

5－6 字节 - 菜单项的 ID (对于弹出菜单，此项不代表 ID)

从第 7 字节开始 – 菜单项名称 (当是弹出菜单时，名字从第 5 个字节开始)

注意一个例外: 在开始的菜单项之前，有两个空（Null）字节作为开始标识

特性 – 我仅仅列出比较重要的属性，完整的列表无法从 `winuser.h` 或网站上获知

`0800` – Separator （分隔符）

`0000` – Enabled （允许）

`0001` – Grayed （变灰）

`0002` – Disabled （禁止）

`0010` – Popup （弹出）

`0080` – End （结束）

菜单项的分割符，看起来象这样:

```text
00 00 | 00 08 | 00 00
属性   名称字符串 (仅仅两个空字节)
```

例如，菜单项如下

```text
00 00 00 00 | 90 00 | 26 00 46 00 69 00 6C 00 65 00
00 00 | 80 00 | 69 00 45 00 26 00 78 00 69 00 74 00
```

你可以看到第一个菜单项的属性是 `90` = `80` | `10` 表示是弹出菜单项，并且结束。

第二个属性仅是 `80` = End. 它不是弹出菜单, 因此它肯定是子菜单。

现在，我们要加菜单项，首先应该知道它象是什么样子，如一个 `OPEN` 菜单项应该象这样：

```text
00 00 | 00 00 |70 00| 26 00 4F 00 70 00-65 00 6E 00
属性    ID    &     O     p     e     n
```

你看，ID 是 `0070`, 但它可以是任何未被使用的值. ‘O’字符前的（&）号将使得‘O’具有下划线以及可以键入字母‘O’来激活相关菜单项

现在再设置一个‘Close’关闭菜单项:

```text
00 00 | 01 00 |71 00 | 26 00-43 00 6C 00-6F 00 73 00-65 00
属性    ID     &     C    l     o     s     e
```

你可以看到，属性是 `0001`（表示灰色），ID 值是自定义的。

在我的演示程序中（一个简单的 Hello World 应用程序示范，仅有 2 个菜单项，它使用 MS VC++编写）我在发现我的菜单资源后的位置找到了一块空间（通常是被‘0’填充的区域），在 `6960`，因此我写了一个文件菜单，插入‘Open’和‘Close’菜单项. 最后是一个退出菜单项‘Exit’ ，它们的样子如下:

```text
00 00 00 00-90 00 26 00-46 00 69 00-6C 00 65 00
00 00 00 00-70 00 26 00-4F 00 70 00-65 00 6E 00
00 00 01 00-71 00 26 00-43 00 6C 00-6F 00 73 00
65 00 00 00-80 00 69 00-45 00 26 00-78 00 69 00
74 00 00 00
```

现在 w, 写下 `6960`,它指向一个 `44`（十六进制）字节的区域，

现在，回到刚才提到的那个地址（在演示的 `NOTEPAD` 里是 `0E750`，在我的 `HELLO WORLD` 里是 `61F0`，指向的地址是 `000067C8`，尺寸是 `22`），你应该已经记下来了，将它的内容改为新的地址（从 `000067C8` 到 `00006960`，尺寸. 然后改变尺寸，从 `22` 到 `44`）。现在保存程序再运行它，我是可以看见我的‘Open’和‘Close’菜单项了。如何，你明白了吗？

有一件事情需要提醒你，当你希望在一个可执行文件中，添加菜单项时，可能需要改变相关的段（section）尺寸，这方面的资料，你可以比较容易的在网上查到。如果你高兴，你可以在某个程序的‘退出’菜单项前加上一道分隔符.

感谢你们大家

作者：`FENRI`

## 有关资料，请参见:

可知行文件格式（The Portable Executable File Format - by Johannes Plachy）

PE 文件内部结构（Peering Inside the PE: A Tour of the Win32 Portable Executable File Format- by Matt Pietrek）

你应该看看这些资料，很不错。

4dd466d0e1e0cc123afbe504da2462ca
