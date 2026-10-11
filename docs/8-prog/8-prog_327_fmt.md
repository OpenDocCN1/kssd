
# 利用 SEH 和 INT3 实现 API HOOK

> 原文：[`www.pediy.com/kssd/pediy12/131457.html`](https://www.pediy.com/kssd/pediy12/131457.html)

用 SEH 技术实现 API Hook，网上已经有帖子了，可是那些帖子太老，太陈旧，只是适用于 95、95 时代。

在 XP 上，网上说的方法根本行不通。

小弟刚学 API HOOK， 有幸发现在 XP 上自由使用 SEH 链的方法，现在把成果公布出来大家一起分享^\_^

各位大侠如果已经知道了，不要笑话小弟^\_^

设置 SEH 链：

```c
PVOID AddVectoredExceptionHandler(int 顺序, PVOID ErrorHander)
```

移除 SEH 链：

```c
RemoveVectoredExceptionHandler(PVOID ErrorHander)
```

ErrorHander 函数声明：

```c
LONG WINAPI ErrorHandler(struct _EXCEPTION_POINTERS *ep)
```

好，我的方案是获得 Debug 权限（这样才能 `OpenProcess`），然后把我的 Dll 绑到目标程序上。

Dll 把 SEH 链设置好，就等着它上钩^\_^

当然，因为 SEH 链是什么错误都处理的，上钩的不一定是我们想要的鱼，这时我们只要判断 `EIP` 就知道是不是我们要的鱼啦^\_^

废话不多说，下面附上 HOOK 了 `DrawTextExW` 和 `TextOutW` 的程序，带源代码^\_^

![](img/f6c7dec395aee593c488290c00a5c837.jpg)

上传的附件

| 图片 | 说明 |
| :--- | :--- |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `DrawRedTextW.rar` |
