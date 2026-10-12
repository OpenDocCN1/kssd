
# 解决 VS2005 添加 gif 资源时，提示无效的 Gif 文件。

> 原文：[`www.pediy.com/kssd/pediy12/132784.html`](https://www.pediy.com/kssd/pediy12/132784.html)

好像 `VS200x` 都有这个问题，没有测试 `VS2010`。

如果向资源中添加 `gif` 文件，IDE 就会出现下面的错误提示：

![](img/b75c92fcac156a59fb39cf40d2ccb219.jpg)

解决方法只有手工修改 `.rc` 文件或者把后缀名去掉再添加。

为了以后能更省事，我简单的跟踪了一下。

下断 `MessageBoxW`，很容易就可以来到下面的地方：

判断文件的后缀名，根据后缀名称进行相应处理：

```
5B5132FF    E8 52D7FFFF     call    5B510A56                ; Is Support file type
```

继续跟进，找到判断 `gif` 文件的地方。然后 `nop` 掉，让 IDE 把 `gif` 文件当成二进制资源处理。

```
5B510A88    68 F40A515B     push    5B510AF4                ; UNICODE ".GIF"
5B510A8D    8D4D F0         lea     ecx, dword ptr [ebp-10]
5B510A90    FF15 1417475B   call    dword ptr <&MFC80U.#1476>; MFC80U.78306233
5B510A96    85C0            test    eax, eax
5B510A98    0F84 2C850100   je      5B528FCA
```

附件中是做好的补丁程序，先复制到 `VS2005` `SP1` 的安装目录下，再进行补丁就可以了。

上传的附件

| ![ || `resedit.dll-patch.exe.7z` |
|
