
# `ASPR123b181` 完美脱壳之雕虫小技，需要脱好可以到 OCG 下载（1千字）

标 题:`ASPR123b181` 完美脱壳之雕虫小技，需要脱好可以到 OCG 下载（1千字）

发信人:heXer

时 间:2002-5-29 21:42:253

详细信息:

`ASPR123b181` 完美脱壳之雕虫小技

另附上我脱壳并修整好的文件，便于大家参考

---

OEP比较容易找到，我相信大家都没问题，可以参看DiKeN的文章，因此简要叙述如下：

```
TRW2000 LOAD ASPROTECT.EXE
G 4D1279
PEDUMP DUMPA.EXE    ;此时dump最合适，原始IAT还没被破坏,因此就不需要修复IAT了
                    ;否则等到OEP处再dump，就麻烦了，还要修复IAT，谁愿意多此一举？
G 4D13C0
F8
记录此时的eip＝48DD64,即OEP，留作后用
G 让程序正常运行起来，然后退出，TRW2000也可以退掉了
```

剩下的工作就是手工修理我们的`DUMPA.EXE`文件了。

我的过程如下（次序已经不重要了，我用的工具是`HIEW6.81`）：

首先修正OEP，将`DUMPA.EXE`文件偏移`0A8h`处`dword`改为`0008DD64`。

然后我们观察其sections信息，发现第4项sections为`.idata`，其RVA＝`9D000`。根据一般经验这里应该是IAT数据，浏览一下此section的内容，得到确认。因此将`DUMPA.EXE`文件偏移`100h`处`dword`改为`0009D000`。

至此已经OK了，运行试试看98，2k下都通过。

---

要想做得完美一些，还可以把最后两项多余的section即`.aspack`和`.adata`删除。

不过要注意的是：

`.rsrc`资源表中有4项资源是指向`.aspack`的，因为`.aspack`的RVA＝`000D1000`。

1）`.rsrc＋0C78`：所指资源RVA＝`000D2FA8`，len＝`0568` ..........(Icon.1)
2）`.rsrc＋0C88`：所指资源RVA＝`000D2700`，len＝`08A8` ..........(Icon.2)
3）`.rsrc＋0F08`：所指资源RVA＝`000D26DC`，len＝`0022` ..........(Group Icon.MAINICON)
4）`.rsrc＋0F18`：所指资源RVA＝`000D230C`，len＝`03D0` ..........(Version information.1)

因此要想去掉`.aspack`段就要把上面4项资源按个自长度剪切并粘贴到`.rsrc`段内后面的空间里，有必要的话还要扩展其段大小。然后要把此4项资源的实际RVA添回到资源表中的RVA指针处。

最后就可以把`.aspack`和`.adata`删除，并修正section num，即把`DUMPA.EXE`文件偏移`086h`处`word`由`0B`改为`09`。高兴的话还可以PE header中有关`.aspack`和`.adata`的内容清零，甚至把sections减肥。

---

heXer/iPB

2002.05.29
