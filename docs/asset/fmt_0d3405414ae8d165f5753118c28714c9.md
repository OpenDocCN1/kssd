
# 汇编语言中 XP 风格的实现

> 原文：[`www.pediy.com/kssd/pediy06/pediy6822.htm`](https://www.pediy.com/kssd/pediy06/pediy6822.htm)

JR21066

2004 年 1 月 21 日

经过实践和在网上的搜索,把网友的经验完整的转为汇编了:D

**第一步:**

在系统库 `Resource.h` 文件里加入 "MOVEABLE PURE" 字符串,这个文件在 `\\masm32\\INCLUDE` 目录中

**第二步:**

在工程文件中 `RC` 资源文件里加入

```
"1 24 MOVEABLE PURE" "res\\Master.manifest"
```

**第三步:**

在工程文件中 `RC` 目录里新建文件 "Master.manifest"

内容如下:

```xml
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <description>Microsoft.Windows.XXXX</description>
  <assemblyIdentity
      type="win32"
      name="Microsoft.Windows.Common-Controls"
      version="6.0.0.0"
      processorArchitecture="x86"
      publicKeyToken="6595b64144ccf1df"
      language="*"
  />
</assembly>
```

*注：此处原文格式混乱，已根据标准 Windows Manifest 结构整理为 `xml` 代码块，核心属性保留自原文。*

**第四步. 添加标准控件,然后汇编,看看吧. 祝大家新年快乐**

注：本文用到 `comctl32.dll`，默认只能在 xp 里面使用。

`4dd466d0e1e0cc123afbe504da2462ca`
