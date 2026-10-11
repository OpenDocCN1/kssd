
# 使用 setwindowlong 激活灰色按钮

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-594.htm`](https://www.pediy.com/kssd/pediy09/pediy09-594.htm)

> 大多激活都是用 `EnableWindow` 这个现成的 API 的，其实这样很不好，总之有些是无发激活了的，`EnableWindow` 会发送一个消息，拦截下来就可以了。

> 现在介绍的是另外一种方法，以前我也在 CFan 编程论坛发过（那时候写的是 Active it!一个激活工具），可能也有别人写过，就是大家都想到一处去了吧。

> 大家知道用 SPY××查看的话，可以看到一个"Style"，里面分为 Windows Style 和 Extened Style 的。如果大家注意的话，可以发现，灰色按钮(控件)都会有一个 `WS_DISABLED`。显然，只要去掉这个属性就可以了。

> 具体代码（C）

```
long Style;

Style = ::GetWindowLong(hWnd,GWL_STYLE);  //获得 Style 值

Style ^= WS_DISABLED;                     //去掉 WS_DISABLED 你也可以用|=把属性加回去

::SetWindowLong(hWnd,GWL_STYLE,Style);    //设置回去

::SetWindowPos(hWnd,
               0,  
               0,  
               0,  
               0,  
               SWP_FRAMECHANGED | SWP_NOSIZE | SWP_NOZORDER | SWP_NOMOVE);          //刷新 并不是所有的都要刷新，但是还是刷一下好
```

> 使用前必须先用 `IsWindowEnable` 判断是否可用，否则，按钮就不像样了。

> 菜鸟之作，多多包涵，谢谢！

4dd466d0e1e0cc123afbe504da2462ca
