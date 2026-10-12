
# 发个 Android 平台上的注入代码

> 原文：[`www.pediy.com/kssd/pediy12/141355.html`](https://www.pediy.com/kssd/pediy12/141355.html)

rt,其实本质上就是 linux 下面的 `library` 注入

只不过处理器是 `arm` 的，需要额外处理一些东西

不少代码是从网上或 xx 直接借鉴的，谢谢这些代码的作者

整体流程分 3 步

1. 在目标进程中分配内存，用来写 `shellcode` 和参数
2. 往目标进程中写入 `shellcode`, `shellcode` 会调用 `dlopen` 来载入我们的 `library`
3. 运行目标进程中的 `shellcode`

当然注入是需要 root 权限的，模拟器上可以直接用，但是真机的话，先 exploit 吧

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || `LibInject.zip` |
