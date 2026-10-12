
# 比赛第一阶段第一题 DIY"pediy.dll"

> 原文：[`www.pediy.com/kssd/pediy10/74120.html`](https://www.pediy.com/kssd/pediy10/74120.html)

我这里有两种方法，主要是因为题目要求不太清楚，我怕理解有误，故全面考虑之。

**第一种理解是**：只能打开网址“http://bbs.pediy.com”，而且 `OpenUrlA` 应该是无参数的函数。

也就是说无论使用何种“Test”主程序测试这个“`pediy.dll`”都应该运行成功而且只打开“http://bbs.pediy.com”页面。

**第二种理解是**：只要能打开页面“http://bbs.pediy.com”即可。

## 第一方法

按要求增加输出表和输出函数 `OpenUrlA`，我选择在导入表的后面一片空白处，文件偏移 `6B0` 开始：

![](img/5b7052ac86708bbd623aeb8dbb7e7506.jpg)

前四个字段不重要，都默认置为 0，`nName` 字段是指向文件名的 RVA,我把“`pediy.dll`”存储在了文件偏移 `6E0` 处（RVA=`2000-600+6E0=000020E0`），因此这个字段值就是：`000020E0`；`nBase` 可以默认置为 0，`NumberOfFunctioins` 和 `NumberOfNames` 都设置为 1；后面是三个表的 RVA，我把函数地址表选在偏移 `6F0` 处，把函数名表放在 `700` 处，把函数序号表放在 `710` 处。函数地址表的内容先不填写，因为还没有打造好 `OpenUrlA` 函数；由于导出的只有一个函数，我把名字填充在偏移 `720` 处，直接把 RVA `00002120` 填写到函数名地址表中。还有一个序号表，就预留着空间就行了。

现在打造 `OpenUrlA` 函数，我把函数代码放在 RVA：`10CE` 处，代码如下（大小为 12 字节）：

![](img/321575947c0a596614fbfd37cb4f6465.jpg)

（此时可以将输出表结构中偏移为 `6F0` 处的函数地址表的内容填写为 `000010CE`）

现在再看看代码，因为“`pediy.dll`”中没有 `ShellExecuteExA` 函数，所以还需要为其增加这个函数，直接使用 `PeEditor` 即可快速完成。

![](img/192edd4d1024ad6ad7e8f6a1323e6e56.jpg)

因为“`pediy.dll`”的 `ImageBase` 为 `400000`，所以打造代码时首先应考虑“以 `400000` 为基址”，最后再进行重定位操作。要求是打开一个网页，最容易想到 API `ShellExecuteA`，但是由于这个函数参数太多，打造代码时势必占用太多空间。跟踪下发现 `ShellExecuteA` 最终是要调用函数 `ShellExecuteExA` 的。而 `ShellExecuteExA` 只需要一个参数 `lpExecInfo`，指向 `SHELLEXECUTEINFO` 结构，如果把这个结构填写完毕，直接把地址压栈调用 `ShellExecuteExA` 即可。而 `SHELLEXECUTEINFO` 结构的打造也相当简单，只要第一个字段为 `0000003C`（`cbSize`），第二个字段为 `00001500`（`fMask`），第五个字段 `lpFile` 为串“http://bbs.pediy.com”的地址即可。

如下，把这些信息放在“.data”节表中：

![](img/288275623eb78d416e2575424cf984bf.jpg)

最后要解决重定位信息，这在 dll 文件里是必须的。

原本的重定位信息为：

![](img/384c93b5e4ebcc2c666d0e0b5ee61564.jpg)在 `OpenUrlA` 函数体内有 2 处需要重定位：`10CF` 和 `10D5` 处。另外令人恼火的就是 `SHELLEXECUTEINFO` 结构了，因为结构体中还需要地址，而且也要进行重定位。我不得不将串“http://bbs.pediy.com”所在的节表“.data”当做“代码”来处理，让 Windows 完成这个重定位过程。只要将所在节表的特征值增加上“可执行代码属性”即可。

最终的重定位信息修改为：

![](img/2fcf169c1bd62636820bc7728bb739c6.jpg)具体改写内容：

![](img/a44bca82a218970aacb8ba6518a76220.jpg)测试代码：

```
procedure  OpenUrlA; stdcall; external 'pediy.dll';

//……

procedure TForm1.Button1Click(Sender: TObject);
begin
        OpenUrlA;
end;
```

## 第二种方法

输出表结构的构造同上，这次为“`pediy.dll`”新增导入函数 `ShellExecuteA` ，`OpenUrlA` 代码打造为（大小为 7 字节）：

![](img/0584bf197d1936aa191701719b8bc658.jpg)重定位信息修改为（大小仍未 20，因为这次只新添一个重定位信息，刚好填补原来对齐用的空间）：

![](img/b8c74e1be91329d4630c652911681413.jpg)参数让测试程序填写，测试代码：

```
procedure  OpenUrlA(p1,p2,p3,p4,p5,p6:DWORD); stdcall; external 'pediy.dll';

//……

procedure TForm1.Button1Click(Sender: TObject);
begin
        OpenUrlA(0,0,DWORD(PCHAR('http://bbs.pediy.com')),0,0,0);
end;
```

**不足之处**：两种方法都使用工具为“`pediy.dll`”新增导入函数，使得文件大小稍有增大。

by sing

2008-10-3
