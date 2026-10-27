
```c
if (hWnd != m) //只要是m，就change
{
    ans = SetWindowTextA(
        hWnd,
        lpString
    );
}
else
    ans = SetWindowTextA(m,test);
HookAPI();
return ans;
}
```

`SetWindowText2` 就是新的函数，每当OD要调用 `SetWindowText` 的时候，就会调用 `SetWindowText2` 函数，函数里面发生的事情有：首先解除 `HOOK`，因为接下来需要调用真正的 `SetWindowText` 以显示内容，然后进行判断，如果句柄不是主窗口句柄，我们就让它正常调用显示，如果是主窗口句柄，我们就把其内容改为 `test` 执行的内容，这个内容我们可以任意写入。调用成功后，再次 `HOOK`，以便下次再次拦截。

### 效果展示

使用 `HOOK` 技术后，当 `OD` 载入要调试的程序时，`OK` 的标题也不会恢复为 `OllyDbg`，而是我们控制的那样，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201305/559334_fjc40n7a07c7490.png)

这样，我们就实现了对程序窗口标题的有效修改，并且始终会发挥作用。

### 调试耗时原因

上面说了，这个破程序搞了我4个小时左右的时间，原因有以下三点：

1. `User32.dll` 默认是不自动加载的。因此，使用 `getprocAddress` 里面的函数之前，要先加载该 `dll`。我本以为 `user32.dll` 和 `kernel32.dll` 一样会自动加载，没想到居然不是，`user32.dll` 需要在程序中手动加载，否则定位不到 `SetWindowsText` 函数的地址。
2. `SetWindowTextA` 并不再调用 `SetWindowTextW`。本以为所有的 `xxxA()` 都会最终还是要调用 `xxxxW()` 函数，所以起初 `HOOK` 时，`HOOK` 的时 `SetWindowTextW`，结果始终不对。
3. `OllyDBG` 里面用的是 `SetWindowTextA`，而不是 `SetWindowTextW`。本以为作为一个发布出来的程序，都会使用 `xxxW` 函数，而 `OD` 显示标题居然是 `SetWindowTextA` 函数。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.png](javascript:void(0);)
  （3.78kb，7次下载）
- [2.png](javascript:void(0);)
  （6.60kb，3次下载）
- [3.png](javascript:void(0);)
  （6.09kb，3次下载）
- [4.png](javascript:void(0);)
  （13.11kb，3次下载）
- [5.png](javascript:void(0);)
  （20.80kb，4次下载）
