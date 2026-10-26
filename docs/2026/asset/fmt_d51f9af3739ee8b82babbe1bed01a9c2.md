
`CLIP_DEFAULT_PRECIS`|`CLIP_TT_ALWAYS`

```
00408D85 |. 6A 04 push 4 ; |OutputPrecision = OUT_TT_PRECIS
00408D87 |. 68 86000000 push 80 ; |CharSet = 128.
00408D8C |. 8B35 2C304300 mov esi, dword ptr [<&GDI32.CreateFontA>; |GDI32.CreateFontA
00408D92 |. 6A 00 push 0 ; |StrikeOut = FALSE
00408D94 |. 6A 00 push 0 ; |Underline = FALSE
00408D96 |. 6A 00 push 0 ; |Italic = FALSE
00408D98 |. 6A 00 push 0 ; |Weight = FW_DONTCARE
00408D9A |. 6A 00 push 0 ; |Orientation = 0
00408D9C |. 6A 00 push 0 ; |Escapement = 0
00408D9E |. 6A 10 push 10 ; |Width = 10 (16.)
00408DA0 |. 6A 20 push 20 ; |Height = 20 (32.)
00408DA2 |. FFD6 call esi ; \CreateFontA
```

而`0x80`则是`SHIFTJIS_CHARSET`编码的值，在`windows.h`头文件中我们可以查到`GB2312_CHARSET`的值为`0x86`。于是我们可以将那个`0x80`改成是`0x86`，用`OD`保存，然后运行……

还是没有显示出中文来！

现在该怎么办？这时应该看看程序利用哪些函数来进行输出的。用右键选“查看”-》“当前模块的所有参考”可以看到有个叫`GetGlyphOutlineA`的函数。在`MSDN`中有明确说明这个函数可以将字符转换成点阵来输出的。往上看，可以看到一段奇怪的代码：

```
00406765 |. 8A06 mov al, byte ptr [esi]
00406767 |. 3C 80 cmp al, 80
00406769 |. 76 04 jbe short 0040676F
0040676B |. 3C A0 cmp al, 0A0
0040676D |. 72 08 jb short 00406777
0040676F |> 3C E0 cmp al, 0E0
00406771 |. 72 5D jb short 004067D0
00406773 |. 3C FC cmp al, 0FC
00406775 |. 77 59 ja short 004067D0
```

这段代码的作用是为了检查字符编码的边界而防止产生异常而导致不可预料的效果的。而由于`GBK`编码比游戏原来的编码的范围要大（`CBK`编码的范围是`0x80xx`到`0xfexx`），所以将比较处的`0A0`跟`0FC`替换成`0FE`，保存。运行，效果如下：

![](https://bbs.kanxue.com/upload/attach/200905/125281_0aitb79gjnb0367.jpg)（关于编码边界检查的更详细内容可以参考这个链接[8a4K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6i4y4#2L8h3W2K6L8%4u0S2i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2S2k6q4)9J5k6i4m8Z5M7q4)9K6c8Y4c8A6k6q4)9K6c8o6t1I4x3e0R3#2x3R3`.`.](https://bbs.kanxue.com/elink@b69K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0T1M7#2)9J5k6i4y4#2L8h3W2K6L8%4u0S2i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2S2k6q4)9J5k6i4m8Z5M7q4)9K6c8Y4c8A6k6q4)9K6c8o6t1I4x3e0R3`.`.)）

就这样，这个游戏的汉化算是大功告成- -

PS.为什么只能算是大功告成，因为我写的封包器似乎有点问题，程序的修改方面也出现一些问题（到后面几句翻译的对白会出现错别字- -|||）最重要的一点是，整个游戏的文本有这么多，我就翻了开头的几句，这怎么能算是汉化完成啊

PS的PS.经某大牛指点，`mes`文件第一个双字是脚本里面文本的个数，接下来的一个数组是文本的偏移位置。虽然据我观察程序并不对这段进行读取……现在将封包程序改一下。另外在调用`GetGlyphOutlineA`前有两次边界检查，如无意外，第一次应该也要改的orz

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [QQ1.jpg](javascript:void(0);)
  （211.53kb，615次下载）
- [QQ2.jpg](javascript:void(0);)
  （26.51kb，606次下载）
- [QQ3.jpg](javascript:void(0);)
  （23.94kb，607次下载）
- [QQ4-1.jpg](javascript:void(0);)
  （24.72kb，607次下载）
