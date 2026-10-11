
# 文件格式之 Office 2007

> 原文：[`www.pediy.com/kssd/pediy10/103011.html`](https://www.pediy.com/kssd/pediy10/103011.html)

**标题**: `Office Open XML 文件格式的一点认识`  
**作者**: `youstar`  
**时间**: `2009-12-13, 1:30`

原来一直没怎么研究过这个，这两天看了下 `Office Open XML` 文件格式，自己做了点笔记，分享给大家，水平确实很低，高手飘过，欢迎大家讨论 `Office Open XML` 的安全性问题。

![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")

## 一、XML 的引入

`office2007` 以前的存储格式为二进制文件（`.doc`，`.dot`，`.xls`，以及 `.ppt` 文件），在过去几年中一直肩负着存储和转换数据的重任，而现在它们无法满足新的市场需求的挑战，其中包括轻松地在异构应用之间传递数据，以及允许用户从这些数据中搜集商业信息。而 `XML` 从 20 世纪 90 年代出现，企业计算客户开始逐渐认识到，在他们所依赖的计算机产品和应用中采用开放的格式和标准所带来的商业价值。`IT` 专业人员将从通用的数据格式中受益匪浅，这种格式可能是 `XML`，因为它拥有被应用程序、平台和 `Internet` 浏览器读取的能力。因此微软在 `office2007` 以后就转向了 `XML`。`2007 Microsoft Office system` 为 `Microsoft Office Excel 2007`，`Microsoft Office Word 2007`，和 `Microsoft Office PowerPoint 2007` 采用了基于 `XML` 的文件格式，从而延续了这种转移。新的文件格式，称为 `Office Open XML` 格式，简称为 `OOXML`。

同时微软在 2008 年在反对的声音中通过了 `OOXML` 格式成为国际化标准，`谷歌` 等包括很多国家和机构都对此不满。

## 二、Office Open XML 的优点

### 1．压缩文件

`Office Open XML` 文件会自动压缩，某些情况下最多可缩小 75%。`Office Open XML` 使用 `zip` 压缩技术来存储文档。

### 2．改进了受损文件的恢复

使用 `Office Open XML` 格式保存的文档不是二进制格式，而是使用文本格式以一定的 `XML` 结构以模块形式进行组织，从而使文件中的不同数据组件彼此分隔。这样，即使文件中的某个组件（例如，图表或表格）受到损坏，文件本身还是可以打开。

### 3．易于检测到包含宏的文档

根据工作簿是否包含宏，在保存工作簿时文件的扩展名也有所不同。无宏的工作簿扩展名为 `.xlsx`，包含宏的工作簿扩展名为 `.xlsm`。这样，通过工作簿名称即可知道文件是否包含宏代码。

### 4．更好的隐私保护和更强有力的个人信息控制

可以采用保密方式共享文档，因为使用文档检查器可以轻松地识别和删除个人身份信息和业务敏感信息，例如，作者姓名、批注、修订和文件路径。

### 5．更好的业务数据集成性和互操作性

将 `Office Open XML` 作为 `Office 2007` 发布版产品集的数据互操作性框架意味着：文档、工作表、演示文稿和表单都可以采用 `XML` 文件格式保存，任何人都可免费使用该文件格式并获得该文件格式的许可证，而不必支付版权费。

### 6．向后兼容性

`Office Open XML` 是向后兼容的，它可以兼容早期的版本。

## 三、office2007 的格式

为了打开一个 `Word 2007 XML` 文件的步骤

1.  创建一个临时目录来存储文件和它的部件。
2.  创建一个 `Word 2007` 文档，包括文本，图片以及其它元素，保存为一个 `.docx` 文件。
3.  在文件名的末端添加一个 `.zip` 扩展名。
4.  双击文件，将会在 `ZIP` 应用程序当中打开该文件。可以查看组成文件的每个部件。
5.  将这些部件提取到刚才创建的临时目录当中。

`2007` 版本的文件格式容器如下图，这个结构我们在下面就来研究下。

![](img/06ca82416a1e9f7dde94fdd751f316d6.jpg)

## 四、office 2007 的一点研究

下面我们对 `office2007` 的结构进行分析。以 `test.doc` 为例，首先将后缀改为 `.zip` 然后解压

![](img/cd89b3ed9312424aa5a5e3cdfafc4601.jpg)

分析如下：

### `_rels` 文件夹

里面有一个文件 (`_rels.xml`)，`.rels` 文件，它定义了包中的根关系。它是在解析整个包时首先要浏览的第一个文件。

源码如下：

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Relationships xmlns="http://schemas.openxmlformats.org/package/2006/relationships">
<Relationship Id="rId3"
Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/extended-properties" Target="docProps/app.xml"/><Relationship Id="rId2"
Type="http://schemas.openxmlformats.org/package/2006/relationships/metadata/core-properties" Target="docProps/core.xml"/><Relationship Id="rId1"
Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/officeDocument" Target="word/document.xml"/></Relationships>
```

其中有三个关系属性：`Id` 为任意字符串，在 `.rels` 文件中必须是唯一的；`Type` 表示关系的类型；`Target` 指定包含关系的目标的文件夹和文件。

### `docProps` 文件夹

包含两个文件（`app.xml` 和 `core.xml`）。

**`app.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<Properties xmlns="http://schemas.openxmlformats.org/officeDocument/2006/extended-properties" xmlns:vt="http://schemas.openxmlformats.org/officeDocument/2006/docPropsVTypes"><Template>Normal_Wordconv.dotm</Template><TotalTime>1</TotalTime><Pages>1</Pages><Words>10</Words><Characters>57</Characters><Application>Microsoft Office Outlook</Application><DocSecurity>0</DocSecurity><Lines>0</Lines><Paragraphs>0</Paragraphs><ScaleCrop>false</ScaleCrop><Company>youstars</Company><LinksUpToDate>false</LinksUpToDate><CharactersWithSpaces>0</CharactersWithSpaces><SharedDoc>false</SharedDoc><HyperlinksChanged>false</HyperlinksChanged><AppVersion>12.0000</AppVersion></Properties>
```

**`core.xml`:**

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<cp:coreProperties xmlns:cp="http://schemas.openxmlformats.org/package/2006/metadata/core-properties" xmlns:dc="http://purl.org/dc/elements/1.1/"
xmlns:dcterms="http://purl.org/dc/terms/" xmlns:dcmitype="http://purl.org/dc/dcmitype/" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"><dc:title></dc:title><dc:subject></dc:subject><dc:creator>youstar</dc:creator><cp:keywords></cp:keywords><dc:description></dc:description><cp:lastModifiedBy>youstar</cp:lastModifiedBy><cp:revision>3</cp:revision><dcterms:created
 xsi:type="dcterms:W3CDTF">2009-12-12T15:58:00Z</dcterms:created><dcterms:modified
xsi:type="dcterms:W3CDTF">2009-12-12T15:59:00Z</dcterms:modified></cp:coreProperties>
```

包含各种 `XML` 文件，描述文件属性和应用程序设置。这些文件包含着文件的属性信息，例如创建者的名字、修改的和创建的日期。

### `word` 文件夹

![](img/33b3921f4cfc0099515fcfaa42da8fb7.jpg)

包含文件的核心部分。

1.  `\_rels`：各部分关系文件
2.  `media`：文档中的图片文件
3.  `theme`：主题的 `XML` 文件
4.  `document.xml`：文档里面的一些结构
5.  `fontTable.xml`：文档字体设置文件
6.  `settings.xml`：文件的一些配置吧
7.  `styles.xml`：有关文档中样式的信息，例如字体大小，表格样式，和列表样式等
8.  `webSettings.xml`：没看清楚….

### `[Content_Types].xml` 文件

包含在我中的其它部件提供了一个内容类型的列表。内容类型可以定义为部件的类型，它们可以存储在一个包中。

说明：自己只是简单的看了看，做了下笔记，作为分享。

更多资料见：

1、官网讲解 [`www.microsoft.com/china/msdn/....mspx?mfr=true`](http://www.microsoft.com/china/msdn/library/office/office/OfficeOpenXMLFormats.mspx?mfr=true)

2、office 格式全套资料（`OfficeFileFormatsProtocols.zip`）

下载地址：[`download.microsoft.com/downlo...sProtocols.zip`](http://download.microsoft.com/download/2/4/8/24862317-78F0-4C4B-B355-C7B2C1D997DB/OfficeFileFormatsProtocols.zip)

本文所用的测试文件下载：`Project.rar`
