
# Petite 脱壳“标准”解决方法

> 原文：[`www.pediy.com/kssd/pediy03/forum604.htm`](https://www.pediy.com/kssd/pediy03/forum604.htm)

**作者**：Ronnier

**发表于**：2001/02/06 18:50:08

---

**工具**：`r!sc's petite` 2.2/2.1、`enlarger` v1.0、`PEditor` 1.7

想来大家早就用 `Enlarge` 脱壳过了 `Petite` 的东东，可是，脱壳后的文件用 `eXeScope` 打开还是说不认，用 `ResHacker` 或是 `Restorator` 打开说没有资源。问题不是在它没有完全脱壳，而是因为 `Enlarge` 脱壳后对于各个段的名称不是原来的标准段名称（也许是由于 `Petite` 压缩的原因，无法还原这些标准的段名称），都是些 `A`、`B`、`C`、`D`……这样的段，由于找不到 `.rsrc` 段，所以那些编辑资源的软件都报错。

知道了问题所在，对症下药就简单多了。用 `PEditor` 打开脱壳后的文件，点击那个 `directory` 按钮，可以看到此文件 `Resource` 的 `RVA`，记下它的数值。关闭 `directory` 对话框，点击 `sections` 按钮，可以看到那些叫 `A`、`B`、`C`、`D`……的段吧，找到 `Virtual Offset` 的值为刚才记录下的那个值的段，那就是资源段。右键点击之，`edit section`，把 `name` 改为 `.rsrc`，`apply changes`，关闭 `sections` 对话框，关闭 `PEditor`，搞定！就这么简单，再看看，可以随便改了吧，呵呵。至于别的段名称，对于汉化没有意义，不理它。

**说明**：我不清楚 `PE` 文件的结构，也不会编程哦，说是“标准”解决方法是因为这样做不会像用 `CoCoRes` 那样使文件变大，而且用 `CoCoRes` 不一定都好用。这个方法是突然来的灵感搞出来的。希望各位程序员朋友和懂得 `PE` 文件结构的朋友能给予批评指正！请跟贴或是给我发信吧，感谢各位了！

`4dd466d0e1e0cc123afbe504da2462ca`
