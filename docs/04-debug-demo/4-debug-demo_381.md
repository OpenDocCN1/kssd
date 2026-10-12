
# 一个小软件 Delphi to C++Builder 的破解全过程

> 原文：[`www.pediy.com/kssd/pediy11/121007.html`](https://www.pediy.com/kssd/pediy11/121007.html)

有一次要把一份 `Delphi` 的代码转为 `C++Builder` 的。因为代码比较多，手工一条条地转很是费劲，于是在网上找了一款叫 `Delphi to C++Builder` 的小软件，该软件存在一些问题，转换后的代码有一些错误，需要手工核对和纠正。不过，使用这个软件还是减轻了不少重复的工作。今天，打开这个软件时却提示：

![](img/f4d416dafd73e143e08b4b8cef79624b.jpg)

软件到期了，郁闷。这小软件价值 39.50 美元，俺是个穷人，于是就产生 `Crack it` 的念头。

`OD` 载入，打开 `Windows` 窗口，都是些 `TEdit`, `TButton`……据此判断该软件应该是用 `Delphi` 或 `C++Builder` 开发的。随便输入一个错误的注册码后提示：

![](img/428f9e30a214ffdc5ea03ed88ba73e36.jpg)

可惜这个消息框不是通过调用 `MessageBox` 产生的，不然就可以直接 `bp MessageBoxA` 切入到核心代码了。`Delphi/C++Builder` 程序中常使用 `vcl` 的函数 `ShowMessage` 来显示一个消息提示对话框。这种对话框是通过调用 `CreateWindowEx` 函数创建的，比如本例。但是在这个程序中对 `CreateWindowExA` 下断点会受到 `Phonetic`, `ImeStatusPHON` 等控件的干扰它们老是在那里不停地创建与销毁。

还是不管这个 `ShowMessage` 了，我们把目标转向那个 `Register` 按钮的事件处理函数。（如果非要去断 `ShowMessage` 也是可以，对字符串`"Invalid register"`下内存访问断点，然后就可以人肉出来了。）常见的 `WndProc 202` 消息断点获取按钮的单击事件处理函数对 `Delphi/C++Builder` 程序并不是所有情况都有效。比如这个小软件就拦截不到。庆幸的是：`Delphi/C++Builder` 中有个特征串可以很容易定位按钮的单击事件处理函数，就是：

```
FF93 20010000 call dword ptr [ebx+120]
```

现在只要把所有与它相同的命令都下个断点就可以了。下好断点后，随便输入一个注册码，点 `Register` 按钮，命中断点：

![](img/a7def881aa267d2e280876df3c3e8d87.jpg)

按 `F7` 单步步入，就进入了 `Register` 按钮的事件处理函数：

![](img/4174c5edee0d78d145b9171bb43732ab.jpg)

找到入口，一切就变得容易多了，一步步跟下来，很容易发现注册码验证的关键位置：

![](img/afea2f0d3d9699a46a125b01297443c5.jpg)

将 `je 004079D2` 直接用 `nop` 填充就爆破了。是不是太简单了。然而更意外的这个小软件居然用明文比较！跟随 `call 00472F4`：

![](img/6cffdd70ee0e04e0c4d880a3ea1c9922.jpg)

寄存器：

![](img/beed200dd14491d06ea12a7b576b5150.jpg)

`call 0042A674` 是调用一个字符串比较函数，用 `IDA Pro` 可以看到分析出其函数名称。`crazychin` 是输入的错误注册码。`ovmfSdsilo` 就是我们要找的正确的注册码。重新运行程序，输入正确的注册码，弹出期待已久的消息框：

![](img/4deb133c04488baa08ae94454297248c.jpg)

注册后，就不能再次注册了，可以进注册表找到：

```
[HKEY_LOCAL_MACHINE\SOFTWARE\ahao's softwares\wenku\d2c\smtp]
```

把 `registed` 的值数据置 0，就可以重新注册了。

至此，一个小软件的破解告捷。（ps:技术太烂了，只能拿这些软柿子捏，![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")）。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | | `d2c.rar` |
|---|---|---|
