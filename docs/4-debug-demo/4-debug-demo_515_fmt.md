
# AutoGraphicsHTML 5.5

> 原文：[`www.pediy.com/kssd/pediy03/forum624.htm`](https://www.pediy.com/kssd/pediy03/forum624.htm)

我在网上找到一关于 `AutoGraphicsHTML` 的东东，但只能破 `3.6` 版，而我的是 `5.5` 版，
好像不行，能帮下我吗？先谢个了。

`AutoGraphicsHTML 5.5` 下载地址:

`http://www.newhua.com/autographicshtml.htm`

## 破 AutoGraphicsHTML

**工具**: `softice` , `16` 进制编辑器, 如 `ultraedit`

### 前言

`AutoGraphicsHTML` 可以把一整个资料夹中的图档通通转换成 `GIF`、`JPEG` 格式，并且制作小图示一一连结到原来的影像档，
很快的就能制作一个可以展示各种图档的网站来喔！ (节录自 `todo`)
他的保护不是用注册码,也不是用日期限制,而是加上浮水印
让你的小图示出现 `Demo version` 之类的字, 所以只有修改一途了
这个软体的程度, 比较简单一点.
`ok`, `go`!!

(1) 启动 `AutoGraphicsHTML`, 一直往下执行, 等到成品完成之後
会启动浏览器, 打开刚刚做好的 `htm` 档案, 看一下小图式,
会看到一些很讨厌的字, `Demo version` 之类的,
我就开始猜了, 他会帮你将影像作成小图示, 是很不错
但他是如何将自写到小图示里的 ?
是用哪一个 `API` 呢 ?
我想了一下, `drawtexta` 或是 `drawtextexa` 呢 ?
不管, 都下好了, 所以进入 `softice`, 下 `bpx drawtexta`, `bpx drawtextexa`

(2) 再启动 `AutoGraphicsHTML` 一次, 可能有时会被拦下来, 不过不要管他
按 `F5`, 直到最後一个步骤, 也就是将图形转换时, 会有一个 `bar` 出现
显示目前的进度, 对不对 , 在这个地方若被拦住了, 就按 `F12` 回到上一层
你会发现你现在正在 `picobj31` 里面, 看下方那一条绿色的线的中央有写
会发现 `call drawtextexa` , 机械码是 `FF15D8620110`

(3) 所以啦, 用 `ULTRAEDIT` 开启 `picobj31.dll`
找 `FF15D8620110`, 通通改成 `909090909090`
`90` 就是 `nop`, 也就是什麽都不要执行
这样就破了

反汇编 `picobj41.dll`，看到下面几个含义明显的输出函数。这几个都是 `C++` 的类成员函数，函数名均为 `AddDemoLabel( )`，前两个是 `CImageInterface` 类的重载函数，后两个是 `CImageProcessor` 类的重载函数）：

```
Addr:10003AA0 Ord:  42 (002Ah) Name: ?AddDemoLabel@CImageInterface@@QAEHH@Z
Addr:10003AC0 Ord:  43 (002Bh) Name: ?AddDemoLabel@CImageInterface@@SAHPAUHDC\_\_@@H@Z
Addr:10006D20 Ord:  44 (002Ch) Name: ?AddDemoLabel@CImageProcessor@@SAHPAPAXH@Z
Addr:100068E0 Ord:  45 (002Dh) Name: ?AddDemoLabel@CImageProcessor@@SAHPAUHDC\_\_@@HJJ@Z
```

再反汇编 `AutoGraphicsHTML.exe`，看它在哪里引用了这些函数，结果在 `import table` 里没找到上述函数，说明该 `exe` 不是直接调用这些函数的。

在该 `dll` 中查找文本串“`AddDemoLabel`”，发现只有一处是在上述几个函数的函数体之外调用它们的（在 `CImageSaver` 类的 `BeforeSave( )` 函数中）：

`Exported fn(): ?BeforeSave@CImageSaver@@QAEHPAPAXHHHHHHHMJUtagRECT@@H@Z - Ord:0030h`

```
:10007B69 8B442410               mov eax, dword ptr [esp+10]
:10007B6D 85C0                   test eax, eax
:10007B6F 740B                   je 10007B7C
:10007B71 6A01                   push 00000001
:10007B73 56                     push esi
* Reference To: PicObj41.?AddDemoLabel@CImageProcessor@@SAHPAPAXH@Z
                              |
:10007B74 E8A7F1FFFF           call 10006D20
:10007B79 83C408               add esp, 00000008
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
                              |:10007B6F(C)
                              |
:10007B7C 8B442444             mov eax, dword ptr [esp+44]
```

显然只要把 `CS:10007B6F` 处的 `je` 改为 `jmp` 即可（`74` 改为 `EB`）。

**总结**：不要使用含义明显的串；程序的保护部分越符合结构化的特点越容易被破掉。 `4dd466d0e1e0cc123afbe504da2462ca`
