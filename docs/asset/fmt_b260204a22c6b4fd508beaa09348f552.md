
# 两篇已翻译好的有关 IDA 插件开发的文章

## 原文信息

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-788.htm`](https://www.pediy.com/kssd/pediy08/pediy8-788.htm)

> `003.How to get the Virtual Address of a PE section with IDA` [http://blogs.msdn.com/geffner/archive/2005/09/13/465180.aspx](http://blogs.msdn.com/geffner/archive/2005/09/13/465180.aspx)

## 如何使用 IDA 获得 PE 区段的虚拟地址

最近为了帮助提高分析软件的有效性,我已经写了一部分 `IDC` 脚本,发布在 [`http://www.datarescue.com/idabase/idaclike.htm.`](http://www.datarescue.com/idabase/idaclike.htm.) 其中一个脚本需要知道反汇编的 `PE` 文件的区段的虚拟地址。自然，我已经阅读了 `IDA` 帮助文件中关于 `IDC` 函数对于这些的说明。你可以想像当我发现在内建的 `IDC` 函数的这些名字中没有“区段”这一单词时我的惊讶。我想“或许 `IDA` 使用了别的单词表示‘区段’”，当我看到 `IDC` 函数 `SegByBase` 时，我非常兴奋。下面的内容来自 `IDA` 的帮助文件:

### `SegByBase`

// Get segment by segment base
// base - segment base paragraph or selector
// returns: linear address of the start of the segment
// BADADDR - no such segment
`long SegByBase (long base); // BADADDR - no such segment`

好的,现在明白了，`IDA` 一定是把区段指定为“段”了。

因此我试着在一些例子中使用 `SegByBase`,发现效果很好。

然而当我在一个包含一打左右的区段的例子中使用它时，`IDA` 的段查看（`View->Open subviews->Segments`）仅仅显示两个段，即为段 `7` 和 `9`。我原本以为可能其它十个段包含资源，`IDA` 可能选择不加载它们。然而，情况不是这样的；它们没有资源区段，`IDA` 确实也在反汇编视图中加载了它们，它们仅仅是不在段查看中显示。更重要的是，我不能用 `SeqByBase` 获得这些区段的基地址。

由于 `IDA` 段的不可靠性，我得出结论只能使用定制 `IDC` 脚本转换得到区段的虚拟地址。然而，这个脚本竟然在 `.idb` 文件中加载了头文件,由于我不想修改 `.idb` 文件，脚本还需要很多的填充，但是我还想做的好并且简单。

因此我写了我自己的 `IDC` 函数来完成它。假定原始的 `PE` 文件和 `.idb` 文件在相同的目录下.

```c
static GetAddressFromSection(section)
{
  auto hFile;
  auto e_lfanew;
  auto imageBase;
  auto numberOfSections;
  auto sectionRva;
  
  hFile = fopen(GetInputFile(), "rb");
  
  if (0 == hFile)
  {
    Fatal("Cannot open \\"" + GetInputFile() + "\\"");
  }
  // 查找 e_lfanew 字段  
  if (0 != fseek(hFile, 0x3C, 0))
  {
    Fatal("Cannot seek in \\"" + GetInputFile() + "\\"");
  }
  
  // 读取 e_lfanew 的值
  e_lfanew = readlong(hFile, 0);
  
  // 查找 IMAGE_NT_HEADERS
  if (0 != fseek(hFile, e_lfanew, 0))
  {
    Fatal("Cannot seek in \\"" + GetInputFile() + "\\"");
  }
  
  // 读取 PE 文件标识
  if (0x00004550 != readlong(hFile, 0))
  {
    Fatal("Not a valid PE file");
  }
  
  // 查找 IMAGE_NT_HEADERS.OptionalHeader.ImageBase 字段
  if (0 != fseek(hFile, e_lfanew + 0x18 + 0x1C, 0))
  {
    Fatal("Cannot seek in \\"" + GetInputFile() + "\\"");
  }
  imageBase = readlong(hFile, 0);
  
  // 查找 IMAGE_FILE_HEADER.NumberOfSections 字段
  if (0 != fseek(hFile, e_lfanew + 0x06, 0))
  {
    Fatal("Cannot seek in \\"" + GetInputFile() + "\\"");
  }
  
  // 读取区段的数目
  numberOfSections = readshort(hFile, 0);
  if (section >= numberOfSections)
  {
    Fatal("Invalid section");
  }
  
  // 选择想得到的区段
  if (0 != fseek(hFile, e_lfanew + 0xF8 + section * 0x28 + 0x0C, 0))
  {
    Fatal("Cannot seek in \\"" + GetInputFile() + "\\"");
  }
  
  sectionRva = readlong(hFile, 0);
  
  fclose(hFile);
  
  return imageBase + sectionRva;
}
```

当然,更多的错误检查应该去做(例如检查‘`MZ`’字节在文件的开始处)，只要你知道了要领。

注意没有什么“不可思议”的。。。只是简单地依照基本 `PE` 头文件结构去找区段的虚拟地址。

`［更新于 2006。1。24－fopen(…)应该使用"rb"模式，而不仅仅是"r"。感谢我的同事 Aaron 找到这一点］`

`［说明 2006。3。6－上面的代码假定了：IMAGE_NT_HEADERS.FileHeader.SizeOfOptionalHeader is 0xE0。我留做一个练习，希望读者使代码更充实］`

`4dd466d0e1e0cc123afbe504da2462ca`
