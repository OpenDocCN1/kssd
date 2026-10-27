
# [原创]VC6 IDE 的增强工具：`vcpkg` 5.2（让 VC6 支持多平台编译，支持多 VAX 并存等）

这是重构后的稳定版本，引入了一系列的新特性，使用 PSDK 的编译器 Unicode 方式编译。

## 特性

1. 更快的 XML 解析模块，支持 unicode/utf8/ansi 的文字编码。
2. 内置类 C 语法的脚本引擎，速度已经优化。
3. 更完善的命令行分析模块，可以非常完美地解析 VC6 输出的命令行参数。
4. 所有的特性都苛求速度，附加中附带的配置启动时仅需要额外花费 0.016 秒即可加载完成，首次编译会自动创建脚本引擎，需花费 0.24 秒，以后编译一般在 0.006~0.008 秒之间，速度极快！
5. 手动调用 MFC `CString` 的函数，所以，`vcpkg` 本身可以使用 ansi/unicode 方式编译，不受 `CString` 的限制。

## 内建功能

- 免安装加载 COM 组件，用于无须安装直接使用各种 VC addon。
- 注册表重定位，用于避免和系统中可能存在的组件冲突，例如另一个版本的 VAX。
- 注册表写入，用于启动时自动设置注册表。
- 多平台编译支持，利用脚本通过分析 VC6 输出的命令行分析当前需要调用哪个编译器进行编译。

## 其他小功能

- patch VC6 的字体选择界面，patch 后可以设置任意字体。

## 使用前提

IDE SP6，以下是涉及的文件列表：

- `MSDEV.EXE` 6.0.9782.X（各小版本均可），只要保证 `2192h` 处是版本号字符串即可。
- `DEVSHL.DLL` 6.0.9782.0，字体 patch 等，如果遇到问题，将 `masspatch` 设为 0。
- `DEVBLD.PKG` 6.0.9782.0，需要 patch 其多个地方。

对于 SP5 的用户，建议先安装 Sp5，然后备份 `c2.dll`、`dvec.h`、`emmintrin.h`、`fvec.h`、`ivec.h`、`malloc.h`、`mm3dnow.h`、`mmintrin.h`、`xmmintrin.h`、`align.c`，然后安装 SP6，再恢复 sp5 的相关文件即可。sp5 的 `c2.dll` 可以支持 SSE 等指令（我也是这么做的，其实 SP5 的 `c2` 就是 VC7.0 的 `c2`，你甚至可以使用 VC7.1 的 `c2.dll`，`c2` 是编译后端，版本间变化不大，新版支持的指令集更丰富）。

其他文件无要求。

## 使用方式

1. 参考压缩包内的 XML 文件，根据自己的环境配置，然后保存成文件 `vcpkg.xml`，放入 VC6 的 IDE 目录。
2. 将 `!vcpkg.pkg` 也放入 IDE 目录，加入 `!` 是为了确保尽早被加载。
3. 为避免和系统中其他 VC6 IDE 冲突，可以修改 `MSDEV.dex` `2192h` 处的 6.0 字符串，改成其他，例如 6.1，`vcpkg` 会读取此信息作为后续操作的依据。

已配置 VC6Plus 的下载地址：

[3f0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4A6k6q4)9J5k6o6t1H3y4U0N6X3j5$3p5$3z5o6x3I4x3e0b7#2x3o6y4Q4x3X3g2K6K9%4W2V1M7X3W2$3k6g2)9J5k6h3I4A6N6X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3!0%4M7$3g2Q4x3X3g2S2M7%4m8^5i4K6u0r3f1s2g2T1L8r3W2U0i4K6u0r3N6X3y4H3L8s2g2K6i4K6g2q4y4e0l9&6i4K6u0W2x3o6V1I4z5q4)9#2c8e0j5`.](https://bbs.kanxue.com/elink@a02K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4A6k6q4)9J5k6o6t1H3y4U0N6X3j5$3p5$3z5o6x3I4x3e0b7#2x3o6y4Q4x3X3g2K6K9%4W2V1M7X3W2$3k6g2)9J5k6h3I4A6N6X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3!0%4M7$3g2Q4x3X3g2S2M7%4m8^5i4K6u0r3f1s2g2T1L8r3W2U0i4K6u0r3N6X3y4H3L8s2g2K6i4K6g2q4y4e0l9&6i4K6u0W2x3o6V1I4z5q4)9#2c8e0j5`.) 2.8M

VC8 的配置 `vc8_config.rar`，它支持 i386/AMD64，你可以从中了解如何使其支持多平台编译，当然，也可以自定义来支持 `gcc`。

[冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [`vcpkg5.2.rar`](javascript:void(0);)（140.47kb，245次下载）
- [`vc8_config.rar`](javascript:void(0);)（4.81kb，200次下载）
