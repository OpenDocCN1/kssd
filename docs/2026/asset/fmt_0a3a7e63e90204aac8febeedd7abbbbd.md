
详见 [56cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5](https://bbs.kanxue.com/elink@bd6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5) 。

# 插件编写体验

在实现 [JNI Helper](https://bbs.kanxue.com/elink@0e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5) 的过程中，摸索了一遍不同逆向工具的拓展功能，所以这里谈谈编写过程中的一些感受，正好也可以作为一次横向对比。

## Ghidra

Ghidra 作为 NSA 出品的工具，拥有丰富的内部资料，开发文档非常整洁规范。虽然说是机缘巧合之下泄露出来才开源的版本，但其质量可以和许多商业工具相媲美，甚至在很多方面还稍胜一筹。使用下来发现 Ghidra 有很多在线文档和资料，可以很方便地实现指定功能，这个跟后面的两个工具可以说形成鲜明对比。

由于 Ghidra 是使用 Java 进行开发的，因此一个明显的问题是运行分析速度相对较慢。这个问题我的一个解决办法是通过 `headlessAnalyzer` 在性能较好的云服务器后台对目标先进行分析，导出 `.gzf` 项目文件再在本地导入。本地的 Ghidra 插件通常使用 Java 进行开发，但可以通过 JPython 使用 Python 脚本编写。

得益于开源软件的特性，开源社区中对于 Ghidra 插件和资料的贡献一直呈爆发式增长。当然这也是因为软件本身投入了很多国家经费被打磨过很长时间，另外一个开源逆向工具 Radare2 就相形见绌了，后面会说到。

一些比较有用的 Ghidra 相关参考资料如下:
- f9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9r3W2V1M7X3q4Q4x3X3g2J5k6g2)9J5c8X3N6Z5K9h3c8J5j5g2)9#2k6X3c8G2j5%4y4Q4x3V1k6S2M7r3W2Q4x3V1j5`.
- c93K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9r3W2V1M7X3q4Q4x3X3g2J5k6g2)9J5c8X3!0F1L8r3W2F1k6g2)9J5k6r3y4G2N6i4u0K6k6i4y4Q4x3V1j5`.
- 508K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0k6i4c8X3L8%4u0Q4x3V1k6s2K9r3W2V1M7X3q4e0L8X3W2H3M7r3g2@1M7H3`.
- cc2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3I4K6j5h3k6W2b7%4W2T1k6i4u0e0k6h3y4#2M7X3W2@1P5g2)9J5c8X3q4%4k6i4y4G2L8h3g2Q4x3X3c8Y4K9r3W2V1M7X3p5`.

## IDA Pro

老牌逆向工具，在 Ghidra 出来之前已经制霸了十几年，因此也拥有丰富的插件生态。从使用者的角度来说，IDA 无疑是一个优秀的选择，其反编译器可以给出非常接近源码的高层伪代码，极大减少了逆向工程师的工作量。缺点也就一个: 太贵。每年几千美刀的授权费用对于个人逆向爱好者而言并不是一个小数目，而且还分别针对不同的指令集架构进行收费。之前还满心期待家庭版的 [IDA Home](https://bbs.kanxue.com/elink@cabK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Z5L8$3#2W2i4K6u0r3)，365美元一年，但是没有 decompiler、没有 SDK、还不能商业使用，于是我又默默关闭页面并掏出了 [Binary Ninja](https://bbs.kanxue.com/elink@d90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3S2W2P5q4)9J5k6h3&6A6L8X3A6S2i4K6u0r3M7r3!0K6N6s2y4Q4x3V1k6U0L8$3#2H3j5i4u0A6M7$3W2G2L8W2)9J5k6r3!0X3i4K6u0V1M7X3g2Q4x3X3c8@1L8$3!0D9M7#2)9J5c8R3`.`.)。

因此要有一个舒适的逆向体验，还是需要一个 `IDA Pro`。从插件开发者的角度来说，IDA Pro 本身确实提供了接口文档，而且由于年代久远也积累了许多插件生态，但是实际使用下来还是有些卡壳。举例来说，在写插件的过程中遇到一个需求，比如判断`jni_all.h`这个头文件是否已经加载过，即某个结构体是否已经定义，在文档中说是使用下面的 idapython 接口:

|  |  |
| --- | --- |
| 1 | `idaapi.get_struc_id('JNIInvokeInterface_') != idaapi.BADADDR` |

可是实际上这样即便 JNI 接口已经定义，返回也是 BADADDR 。最终实现还是通过一种比较取巧的方法，即根据下面的语句是否抛出异常来判断:

|  |  |
| --- | --- |
| 1 | `idc.parse_decl('JNIInvokeInterface_ *if', idc.PT_SILENT)` |

类似的问题还有不少，而且很多技巧需要通过阅读其他人的插件代码去了解，对于不熟悉的开发者来说体验并不是太好。下面是一些 IDA 开发相关的资料:
- 45fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3
- dd0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3N6s2g2@1L8%4u0A6j5h3I4K6i4K6u0r3
- c42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Z5k6i4S2Q4x3X3c8J5j5i4W2K6i4K6u0W2j5$3!0E0i4K6u0r3M7s2u0G2k6s2g2U0N6s2y4Q4x3V1k6A6k6r3q4Q4x3V1k6K6N6i4m8H3L8%4u0@1i4K6u0r3M7$3c8C8k6r3!0U0i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9
- 996K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9h3y4W2j5%4t1@1j5$3E0Q4x3V1j5%4j5e0N6S2k6U0x3J5y4K6M7%4z5o6N6U0y4K6V1@1j5K6j5$3z5e0j5#2y4e0p5%4x3e0V1&6k6X3x3&6j5H3`.

## Radare2

作为一个骨灰级命令行爱好者，接触 Radare2(r2) 也是顺应自然的选择。最初使用 Radare2 的一个原因是因为其开源，可以在各个平台中使用；另外一个原因是支持 VI 快捷键，省去了我很大的学习成本。虽然一开始以命令行逆向工具为卖点，但也可以配合 [Cutter](https://bbs.kanxue.com/elink@eacK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6i4c8@1k6i4u0Q4x3X3g2J5k6g2)9J5c8R3`.`.) 使用。

虽然每次我都狂热推荐使用 r2，但实际上他也有明显的局限性，其中一个是 decompiler 的缺乏。这可以使用一些三方的 decompiler 实现，比如:
- Ghidra Decompiler
- RetDec Decompiler
- r2dec

作为插件开发者，使用的主要是 r2pipe 接口，这个接口实际上是 r2 的命令行参数的管道，所以写插件本质上还是需要通过 r2 的命令实现。说到 r2 的命令，内置的帮助信息可以方便日常使用的查询，用户只需要记得大致的命令类目，比如分析类命令是`a`开头，可以使用`a?`查看所有分析相关的命令，进而可以使用`af?`查看所有和函数分析相关的命令。

虽然和 Ghidra 一样是开源工具，但由于 r2 主要是爱好者去维护，因此投入上的差距也导致了开发进度的差距。在 Github 中可以看到开发者们经常在努力修 bug 以及更新优化代码，但关闭 issue 的速度远跟不上新开 issue 的速度。举一个遇到的例子，我想要修改某个地址对应函数的签名为对应 JNI 函数签名，搜索后发现有几种方法: 一是使用 `afvr/avfn/afvt` 修改变量；二是使用 `tl` (type link)去连接类型；三是使用 `afs` 去直接修改签名信息。可测试下来这几种方式都有问题。其中 `afs` 是最接近我需求的命令了，但是却不支持自定义的类型，而解决这个问题还需要等 r2 项目组将内置 parser 从 `Tcc` 迁移到 `tree-sitter` (见 [issue#17432](https://bbs.kanxue.com/elink@a26K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5h3c8S2M7X3g2G2M7X3N6Q4x3V1k6J5j5h3c8S2M7X3f1J5i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3x3e0M7@1x3K6t1`.))。诸如此类的问题也是比较打击插件开发者积极性的，毕竟谁也不想为了一个简单的功能找半天文档发现无法实现。

尽管如此，我还是非常看好 Radare2 的未来，他年岁尚浅，但有很大的拓展性，值得持续保持关注。同时也应该给开发者更多的耐心和支持，如果可以的话，为其添砖加瓦。下面是一些 r2 相关的资料:
- f3dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5h3c8S2M7X3g2G2M7X3N6Q4x3V1k6J5j5h3c8S2M7X3f1J5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5#2)9J5c8X3W2F1N6s2u0G2i4K6u0W2L8h3b7`.
- 9f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5h3c8S2M7X3g2Q4x3X3g2Y4K9i4c8T1L8$3!0C8M7#2)9J5k6h3W2G2i4K6u0r3M7X3q4V1j5i4u0W2x3X3u0G2L8$3E0Q4x3V1k6U0L8$3&6@1k6h3&6@1i4K6u0r3
- 75fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8$3!0C8i4K6u0W2M7X3q4V1j5g2)9J5k6i4u0W2i4K6u0r3

# 后记

本文主要是分享 [JNI Helper](https://bbs.kanxue.com/elink@422K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5) 这个辅助自动化静态逆向分析 JNI 接口的工具，可以在日常安卓逆向时候减少一些重复的劳动。俗话说工欲善其事，必先利其器，在日常中打磨自己的武器库值得投入必要时间，但也要避免陷入盲目造轮子的冲动。另一方面，也记录一下在编写各个逆向工具插件过程中的一些感受。总的来说 Ghidra 的插件编写体验最好，文档丰富且完善，不愧为 NSA 的产品；其次是 IDAPython，虽然有部分文档，但很多都只是一句话带过，想查找某些特定功能需要花费额外时间；而对于 Radare2，虽然是使用管道来进行批处理，但基本的交互都可以支持，只是部分功能实现尚不完善，需要给开发者更多的支持和耐心，毕竟大家都是花费着自己的业余时间和精力去维护和贡献。希望能有更多人投入到 Radare2 的社区中，哪怕只是贡献一点文档，写一写 writeup，对开源社区的帮助也是很大的。

# 参考链接

- [JNI 文档](https://bbs.kanxue.com/elink@e4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3x3g2)9J5k6e0g2Q4x3X3f1H3i4K6u0r3k6r3!0U0M7#2)9J5c8X3N6#2K9h3c8W2i4K6u0r3K9X3&6A6i4K6u0r3M7%4m8W2j5#2)9J5c8X3c8W2M7$3W2Y4L8W2)9J5k6h3S2@1L8h3H3`.)

- [skylot/jadx](https://bbs.kanxue.com/elink@0fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9%4W2D9L8%4c8Q4x3V1k6B7j5h3c8^5)

- [Ayrx/JNIAnalyzer](https://bbs.kanxue.com/elink@2a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8P5i4u0^5i4K6u0r3d9V1&6u0b7h3&6S2L8s2W2*7k6i4t1`.)

- [c6bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5](https://bbs.kanxue.com/elink@4ceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0r3K9X3&6A6i4K6g2X3K9r3g2D9M7r3g2J5)

- [017K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0p5H3i4K6u0r3x3o6N6Q4x3V1k6B7L8X3W2Q4x3X3c8Z5k6h3I4H3k6i4u0Q4x3V1j5`.](https://bbs.kanxue.com/elink@8f6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3)

订阅号: 有价值炮灰，感兴趣的朋友可以关注~

![图片描述](https://bbs.kanxue.com/upload/attach/202010/844554_XSCUJ4VHCZPM2XX.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
