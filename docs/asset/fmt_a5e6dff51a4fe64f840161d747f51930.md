
# 菜鸟注册机编写基础教程（二）

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-210.htm`](https://www.pediy.com/kssd/pediy08/pediy8-210.htm)

## 二、入门篇

上篇讲了基础知识，这篇开始动手实际操作了。我希望大家一定要动手不要光看文章（这点非常重要，你会发现动起手来上手更快）。在这之前，先装好这几种语言。为了方便可以装精简版（`6M` 的 `VB6.0` 精简版，`50M` 的 `DELPHI 2006` 精简版，`13M` 的 `VC6.0` 精简版，或者直接装个 `VS6.0`），本节将以一个简单的实例了解各种语言的工程建立方法并编译出第一个注册机程序。（源码看附件）

已知某一软件的算法过程是：`注册码=用户名的累加值`，就这么简单先，我们通过编写算法注册机去了解一下各种语言的结构特点，尽量做到每一行都进行注释。

### 一、 VB6.0

1. 启动软件，新建一个标准 `EXE` 工程，拉出两个 `TextBox` 和一个 `CommandButton`，将两个 `TextBox` 中的 `Text` 文本内容去掉，将 `CommandButton` 的 `CAPTION` 改为确定，双击确定按钮来到代码区。

2. 写入代码：

```vb
Private Sub Command1_Click()                    ' 按钮的响应代码
Dim i As Integer                                ' 定义 i 为整形变量
For i = 1 To Len(Text1.Text)                    ' i 等于 1 到文本框 1 的内容（用户名）的长度
Sum = Sum + Asc(Mid(Text1.Text, i, 1))          ' 逐个取用户名的 ASCII 码累加
Next I                                           ' 取下一位用户名
Text2.Text = Sum                                 ' 文本框 2 等于最终计算结果
End Sub
```

3. 就这么简单，按 `F5` 测试一下，成功。

4. 按菜单：文件-编译工程，保存为一个可执行文件，这就是你的注册机了。

5. “附件中的注册机是居中的，而且有关于和退出按钮，任务栏也有显示工程图标的，我这里怎么没有？”要想增加关于和退出内容，你可以再建立两个按钮，并在菜单中按“工程”-添加 `FORM` 窗口，新窗口中新建一个 `LABEL`，写入关于的内容，然后给关于按钮添加代码：`Form2.Show`，同样给退出按钮添加代码 `Unload Me` 即可。关于居中和任务栏设置都是在 `Form1` 的属性里，其中 `BorderStyle` 设置窗口显示模式，`StartupPostion` 设置是否居中，`ShowInTaskbar` 设置是否在任务栏显示工程图标。

6. 工程文件为 `vbp` 文件。

### 二、 DELPHI

1. 启动 `DELPHI 10 LITE`，新建一 `VCL` 窗口工程，拉出两个 `TEdit` 和一个 `Tbutton`，去掉两个 `Tedit` 中的 `text` 内容，改 `Tbutton` 的 `Caption` 为确定，双击按钮带代码区，写入代码：

```delphi
procedure TForm1.Button1Click(Sender: TObject);      //按钮响应代码
var                                                  //变量赋值
    i,sum:integer;                                   //定义 I，SUM 为整形变量
begin                                                //代码开始
  sum:=0;                                            //初始化总和 SUM 的值
   for i:=0 to length(edit1.text)-1 do               // I 从 0 到用户名的长度
    begin                                            //循环开始
      sum:=sum+ord(edit1.text[i]);                   //逐个取用户名的 ASCII 累加
    end;                                             //循环结束
  edit2.text:=inttostr(sum);                         //TEDIT2 的内容为计算结果
end;
```

2. 也很简单，可以按 `F9` 进行测试，成功。

3. 按菜单：`PROJECT-COMPILE PROJECT` 或者 `CTRL+F9` 编译后文件目录里便多出个可执行文件，这就是你的注册机。

4. 同样，想增加关于和退出的内容的话也增加两个按钮，并在菜单中选 `FILE-NEW-FORM` 新增关于窗口，写入关于的内容，并双击关于按钮，写入代码 `Form2.Show`。退出的话很简单，双击退出按钮，增加代码：`Close`。

5. 居中和窗体模式将 `FORM1` 属性中的 `BOARDSTYLE` 设置为 `BSSIZETOOLWIN` 和 `POSITION` 中设置为 `POSCREENCENTER`。

6. 工程文件为 `dpr` 文件。

### 三、 VC6.0，由于讲的都是 GUI 界面注册机，所以这里以 MFC 为实例。

1. 打开 `VC6.0`，新建一 `MFC` 的 `EXE` 工程，应用程序类型选基本对话框，其它的默认设置。

2. 去掉原来窗口上的那些控件，拉出两个 `Edit Box` 和一个 `Button`。在控件上右击选建立类向导，`Member Variables` 中增加成员名称 `EDIT1` 为 `m_Edit1`，`EDIT2` 为 `m_Edit2`，`Message Maps` 中为 `BUTTON1` 增加 `BN_CLICKED` 成员功能名称。全部设置好后在窗口中双击按钮来到按钮响应代码区，写入代码：

```cpp
void CMy1Dlg::OnButton1()          //按钮响应
{
  UpdateData(true);                 //用于将屏幕上控件中的数据交换到变量中
  long i,n=0;                       //定义变量 i.,n 为长整型，并初始化 n
  char cName[20],SN[20];             //定义 20 个元素的用户名和注册码数组
  strcpy(cName,m_edit1);
  for (i=0;i<m_edit1.GetLength ();i++)  n += cName[i];    //用户名累加计为 n
  _itoa(n,SN,10);                   //将长整型 n 转化为字串型 SN
  m_edit2 = SN;                     //EDIT2 的内容为计算结果
  UpdateData(false);                //将数据在屏幕中对应控件中显示出来
}
```

3. 稍微复杂了点，但是简练。`CTRL+F5` 测试，通过。看文件目录下 `Debug` 文件夹中的可执行文件，那就是你的注册机。

4. 同样想增加关于和退出的内容，分别放上关于按钮和退出按钮，退出按钮直接写入代码 `CDialog::OnCancel()` 即可，关于可以用新增对话框形式，菜单“`INSERT`”-`NEWFORM`，名称填入 `ABOUT`，在 `ABOUT` 窗口中添加关于的内容。调用时双击关于按钮写入代码：

```cpp
about aboutDlg;                      //对话框名称
aboutDlg.DoModal();                 //显示模式对话框
```

5. 对话框默认居中，不用设置了，工程文件为 `dsw` 文件。

这节内容使我们对语言的结构有了初步的了解，同时我们了解了几个最常用的函数，如 `VB` 中的 `LEN()`、`ASC()`，`MID()`，`DELPHI` 中的 `LENGTH()`、`ORD()`、`INTTOSTR()`，`VC` 中的 `STRCPY()`、`GETLENGTH()`，`ITOA()`、`UPDATEDATA()` 等，另外我们可以对他们的编译速度进行对比，并对编译后的可执行文件进行对比，由于 `VB` 没有带运行库，所以明显的小了很多（`NT` 系统都有自带），从编译速度看，`DELPHI` 比其他的明显的慢了很多。功能的强弱现在还看不出来，这节内容就到这里。

单击此处附件下载:`keygen2.rar`

【版权声明】: 本文由 `langxang` 原创于看雪论坛，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
