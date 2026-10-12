
# PaiMei 文档

> 原文：[`www.pediy.com/kssd/pediy12/135672.html`](https://www.pediy.com/kssd/pediy12/135672.html)

最近花时间了解[PaimMei](http://code.google.com/p/paimei/)平台，该工具相信很多人都听说过，逆向工程者的瑞士军刀哟。

主要是为了自己学习与方便查阅，我花时间翻译了一下它的文档，有需要的朋友可以参考下，欢迎交流。可以直接在浏览器打开`docs`目录下的`index.html`，下面帖两幅效果图：

代码:

```


```

代码:

```


```

另外如果你想在`PaiMei GUI`平台下也访问这个中文文档，则替换相应目录下的`html`文件即可

代码:

```


```

注意备份哟![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")

不过由于`GUI`平台用的`wxPython`的`html`控件显示这些文档，而这个控件对中文网页的自动换行支持不理想，以及中文环境下对`CSS`文件的支持似乎也不完全，效果不是太好（我手动控制的换行，从而文字行右边参差不齐，字体也难看）![](img/65a46d670c4ac01e17bcc70193b248af.jpg "皱眉")：

代码:

```


```

代码:

```


```

如果是因为`wxPython`控件设置的问题的话，还望`wxPython`高手指点了，我们可以改改`PaiMei`框架的代码嘛（框架是完全用`Python`完成的）![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `temp.rar` |
