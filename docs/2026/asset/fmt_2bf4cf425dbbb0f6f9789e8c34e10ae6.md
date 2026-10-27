
```cpp
#include "Scintilla.h"
#include "Platform.h"
#include "ILexer.h"
#include "SplitVector.h"
#include "Partitioning.h"
#include "RunStyles.h"
#include "ContractionState.h"
#include "CellBuffer.h"
#include "CallTip.h"
#include "KeyMap.h"
#include "Indicator.h"
#include "XPM.h"
#include "LineMarker.h"
#include "Style.h"
#include "AutoComplete.h"
#include "ViewStyle.h"
#include "CharClassify.h"
#include "Decoration.h"
#include "CaseFolder.h"
#include "Document.h"
#include "Selection.h"
#include "PositionCache.h"
#include "Editor.h"
#include "ScintillaBase.h"
#include "CaseConvert.h"
#include "qclipboard.h"
```

![](https://bbs.kanxue.com/upload/attach/202312/780980_7CEYDX4TES3SPTN.webp)

## 7.6 ScintillaQt.cc

```cpp
#include <QTextCodec>
#include <QScrollBar>
#include <QTimer>
#include <QDrag>
#include <QClipboard>
#include <QMimeData>
#include "PlatQt.h"
#include "ScintillaQt.h"
```

![](https://bbs.kanxue.com/upload/attach/202312/780980_ECDGBKTAG6F3V8G.webp)

## 0x08 处理编译器所有告警

### 关于编码告警告处理

> 将以下源码文件通过 Notepad++ 转换编码格式为：UTF-8

| 文件列表 |
| --- |
| `Crypto.h` |
| `Watermarks_window.cc` |
| `Inifile.h` |
| `Models.cc` |
| `Mainwindow.cc` |
| `Main.cc` |
| `Registry_manager.cc` |
| `Pdb.h` |

> 将`gmock-matchers.cc`源码文件通过记事本另存的方式转换为 unicode 格式进行替换

![](https://bbs.kanxue.com/upload/attach/202312/780980_PRAXKPSXUJRGF7C.webp)

### 关于C4996警告处理

| 严重性 | 代码 | 说明 | 项目 | 文件 |
| --- | --- | --- | --- | --- |
| 警告 | `C4996` | `'std::tr1'`: warning `STL4002`: The non-Standard `std::tr1` namespace and TR1-only machinery are deprecated and will be REMOVED. You can define `_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING` to acknowledge that you have received this warning. | `gtest/unit-tests/win_runtime.Tests` | `C:\Users\nothx\Desktop\vmp\third-party\gmock\include\gmock\internal\gmock-generated-internal-utils.h` |

> 涉及该警告的主要是 Misc 模块下面的`gtest/unit-tests/win_runtime.Tests`三个文件

![](https://bbs.kanxue.com/upload/attach/202312/780980_EKS83WQQETDPG38.webp)

> 相应的模块上右键->属性->C/C++->预处理器->预处理器定义
> 添加宏：`_SILENCE_TR1_NAMESPACE_DEPRECATION_WARNING`

![](https://bbs.kanxue.com/upload/attach/202312/780980_3DPCNZ4BYUP93S8.webp)

### 关于 目标框架“`netcoreapp3.0`”不受支持,将来不会收到安全更新 警告处理

> 单击`VMProtect.Netcore`模块，增加如下内容

```xml
<CheckEolTargetFramework>false</CheckEolTargetFramework>
```

![](https://bbs.kanxue.com/upload/attach/202312/780980_D3MR8U62C942SXZ.webp)

### 关于 `MSB8051` 面向 Windows XP 的支持已被弃用,将来的 Visual Studio 版本不再提供该支持 警告处理

文件路径：`E:\Microsoft Visual Studio\2019\Professional\MSBuild\Microsoft\VC\v150\Platforms\x64\PlatformToolsets\v141_xp\Toolset.targets`

![](https://bbs.kanxue.com/upload/attach/202312/780980_H5V46JZAUE93B4N.webp)

```xml
<VCMessage Code="MSB8051" Type="Warning" Condition="'$(XPDeprecationWarning)' != 'false'" />
修改为
<VCMessage Code="MSB8051" Type="Warning" Condition="'$(XPDeprecationWarning)' == 'false'" />
```

### 关于 `D9030` “`/Gm`”与多处理不兼容;忽略 `/MP` 开关 警告处理

![](https://bbs.kanxue.com/upload/attach/202312/780980_3RERDMBNDGFZ53B.webp)

将 *启用最小重新生成* 修改为 **否（`/Gm-`）**

![](https://bbs.kanxue.com/upload/attach/202312/780980_NQDW53YEWCVP44N.webp)

## 0x09 授权

> 将`/vmp/core/VMProtectLicense.ini`拷贝至`/vmp/bin/64/Debug/`目录

![](https://bbs.kanxue.com/upload/attach/202312/780980_XP9RJF69E7GBDY6.webp)

> 测试功能，加壳保护后程序依然正常运行

![](https://bbs.kanxue.com/upload/attach/202312/780980_K5MN5BW9Y4FD7A7.webp)

![](https://bbs.kanxue.com/upload/attach/202312/780980_VTUX9NVQNXUXF4N.webp)

## 0x10 静态编译（续）

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
