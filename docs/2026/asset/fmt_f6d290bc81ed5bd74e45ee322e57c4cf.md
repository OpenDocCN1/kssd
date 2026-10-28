
# 用 S2E 和 Kaitai Struct 针对性地处理文件解析器

## 介绍

最近我一直在研究S2E中的文件解析器。这通常涉及调用`s2ecmd symbfile` 文件来使解析器的输入符号化，然后运行S2E来解析通过解析器的不同路径。但是，这是一个比较笨重的做法；它使整个输入文件产生一个非常大的符号化的块，这很快导致了<a href="elink@184K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6e0P5h3#2T1L8$3I4A6j5#2)9#2k6X3g2^5k6h3y4#2N6r3W2G2L8W2)9J5x3#2m8S2N6r3S2Q4y4h3k6q4P5s2m8D9L8%4y4A6L8$3^5`.">路径爆炸</a>。此外，我们可能只想探索行使特定功能的路径。

那么我们如何在基于文件的程序（如解析器）上实现更有针对性地实现符号执行呢？一种方法是编写一个自定义的S2E插件来处理`onSymbolicVariableCreation`事件，拦截`s2ecmd symbfile`文件。然后，您可以编写C++代码来迭代和具体调整符号化的数据内容。这种方法的缺点是显而易见的：编写C++代码是相当耗时且容易出错；它需要知道输入文件的格式；在处理不同的文件类型时还要重写，如何更好的实现呢？

## Kaitai Struct

暂时抛开S2E不谈，看看 <a href="elink@237K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3E0S2K9i4c8S2K9g2)9J5k6h3W2G2i4K6u0r3">Kaitai Struct</a>。 Kaitai Struct是开发二进制结构解析器的工具。它提供了一种类似YAML的语言，可以简洁地定义二进制结构。 Kaitai Struct 编译器（`ksc`）然后根据这个定义生成一个解析器。该解析器可以用多种语言生成，包括C++、Python和Java。

以下是Kaitai Struct中的ELF文件格式的部分定义（取自<a href="elink@170K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3k6G2M7X3#2S2N6s2y4Q4x3X3g2C8j5h3W2@1j5h3W2Q4x3X3g2A6L8#2)9J5c8X3g2D9k6W2)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8l9`.`.">格式库</a>）。它由许多描述ELF文件的“属性”（例如`magic`、`abi_version`等字段）组成：

```
meta:
id: elf
title: Executable and Linkable Format
application: SVR4 ABI and up, many *nix systems
license: CC0-1.0
ks-version: 0.8
seq:
# e_ident[EI_MAG0]..e[EI_MAG3]
- id: magic
size: 4
contents: [0x7f, "ELF"]
# e_ident[EI_CLASS]
- id: bits
type: u1
enum: bits
# e_ident[EI_DATA]
- id: endian
type: u1
enum: endian
# e_ident[EI_VERSION]
- id: ei_version
type: u1
# e_ident[EI_OSABI]
- id: abi
type: u1
enum: os_abi
- id: abi_version
type: u1
- id: pad
size: 7
- id: header
type: endian_elf
```

强烈建议阅读Kaitai Struct<a href="elink@2f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2j5#2)9J5k6h3E0S2K9i4c8S2K9g2)9J5k6h3W2G2i4K6u0r3">文档</a>以充分利用这篇文章，因为我跳过了大部分细节（主要是因为我自己并不擅长这方面）。然而，有一个值得一提的功能是“<a href="elink@879K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8G2j5#2)9J5k6h3E0S2K9i4c8S2K9g2)9J5k6h3W2G2i4K6u0r3K9%4y4&6i4K6g2X3M7%3g2X3k6i4u0W2L8X3y4W2i4K6u0W2K9s2c8E0L8q4)9J5x3%4y4H3k6h3y4Q4x3X3c8H3M7X3!0U0k6i4y4K6">处理规范</a>”。

处理规范允许你以某种方式“处理”属性的自定义函数。例如，可以对属性进行加密/编码。处理规范可以在运行时对该属性进行解密/解码。

这与符号执行有关吗？假设我们有一个`s2e_make_symbolic`的文件处理规范，并且通过将此规范应用于特定的属性，我们只会使输入文件的这些部分符号化。这会使我们更好的控制S2E的状态空间，并可能减少路径爆炸问题。只需要将S2E和Kaitai Struct结合起来就可以实现！

## 结合 S2E 和 Kaitai Struct

我们将使用<a href="elink@c83K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9N6h3q4Q4x3X3g2G2M7X3N6Q4x3V1j5`.">Lua</a>编程语言来组合S2E和Kaitai Struct。使用Lua可以重用现有的组件——S2E包含一个嵌入式的Lua解释器（用于解析S2E配置文件，编写函数/指令<a href="elink@332K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6V1L8$3y4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4J5j5#2)9J5c8W2m8D9N6h3N6A6L8Y4y4Q4x3V1k6m8L8X3&6G2N6r3q4@1K9h3!0F1M7#2)9J5k6i4u0K6N6l9`.`.">注释</a>），而`ksc`能够生成Lua解析器。因此，我们可以使用`ksc`为我们的输入文件生成一个Lua解析器，并将该解析器嵌入到S2E配置文件中，使其可以被S2E访问。（我们可以使用`ksc`来生成一个C++解析器，但这样的话，每次我们想要使用不同的文件格式时，都需要重新编译S2E）。通过在输入定义中选择性地应用`s2e_make_symbolic`处理规范，我们可以实现更有针对性的符号执行。

这篇文章剩余部分将介绍如何组合S2E和Kaitai Struct。我将使用ELF文件的定义（前面讨论过）和<a href="elink@479K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3j5X3W2F1N6i4c8A6L8s2y4Q4x3V1k6V1L8$3y4K6i4K6u0r3j5X3W2F1N6i4c8A6L8s2y4Q4x3V1k6J5k6h3q4V1k6h3I4X3i4K6u0W2K9s2c8E0L8l9`.`.">readelf</a>来作为一个实例。

为了让其他人更容易地使用代码，我努力使它尽可能的独立——没有对S2E的核心引擎或`ksc`进行任何修改。然而，这意味着代码基本没有优化！代码由以下部件组成：

- 在客户操作系统中执行的命令行工具（`s2e_kaitai_cmd`）。这个工具读取输入文件并且调用S2E插件，选择性地使文件符号化。
- 一个S2E插件（`KaitaiStruct`），它调用Lua代码来运行由`ksc`生成的解析器。
- 一小段Lua代码连接S2E配置文件和由`ksc`生成的解析器。

这些部件中的每一个在下面描述。完整的代码在<a href="elink@438K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2k6s2u0A6j5h3&6Z5k6i4u0J5k6i4u0S2i4K6u0r3K9$3q4A6N6r3q4A6i4K6u0V1M7K6u0W2">这儿</a>。

## `s2e_kaitai_cmd`工具

在这篇文章的开头，我提到我们通常会使用`s2ecmd symbfile` 来使输入文件符号化。`symbfile`命令使输入文件符号化：

1. 以读/写模式打开输入文件
2. 将输入文件读入缓冲区
3. 在缓冲区上调用`s2e_make_concolic`
4. 将（目前符号化的）缓冲区写回原始输入文件

我们将采取类似的方法，除了我们将步骤（3）修改为：

- 调用`KaitaiStruct`插件来选择性地使缓冲区符号化

为此，我们将在S2E环境中添加以下目录/文件：

- `source/s2e/guest/common/s2e_kaitai_cmd/s2e_kaitai_cmd.c`
- `source/s2e/guest/common/include/s2e/kaitai/commands.h`

我会跳过步骤1、2和4，因为它们已经在<a href="elink@25dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6Y4N6h3g2K6N6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3y4G2L8h3#2G2L8W2)9J5c8X3W2F1j5$3I4#2k6r3g2Q4x3V1k6K6x3X3g2Q4x3V1k6K6k6h3g2V1i4K6g2X3M7$3g2S2M7X3y4Z5k6i4u0Q4x3V1k6U0L8$3#2E0j5h3&6V1M7#2)9J5k6h3R3`.">s2ecmd</a>中实现了。对于步骤3，我们会自己写一个自定义的S2E命令来调用一个插件（稍后描述），有选择地使输入的文件符号化。命令结构应放在`source/s2e/guest/common/include/s2e/kaitai/commands.h`中。它遵循从客户端<a href="elink@b11K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6Y4N6h3g2K6N6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3y4G2L8h3#2G2L8W2)9J5c8X3W2F1j5$3I4#2k6r3g2Q4x3V1k6K6x3X3g2Q4x3V1k6K6k6h3g2V1i4K6g2X3M7$3g2S2M7X3y4Z5k6i4u0Q4x3V1k6U0L8$3#2E0j5h3&6V1M7#2)9J5k6h3R3`.">调用S2E</a><a href="elink@924K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0x3V1g2Q4x3V1k6Y4N6h3g2K6N6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3y4G2L8h3#2G2L8W2)9J5c8X3W2F1j5$3I4#2k6r3g2Q4x3V1k6K6x3X3g2Q4x3V1k6E0L8$3&6A6N6r3!0J5M7#2)9J5c8X3y4G2L8h3#2S2L8X3c8K6i4K6u0r3M7X3q4%4i4K6u0W2K9l9`.`.">插件</a>的标准方法：

```c
enum S2E_KAITAI_COMMANDS {
KAITAI_MAKE_SYMBOLIC,
};
struct S2E_KAITAI_COMMAND_MAKE_SYMBOLIC {
// Pointer to guest memory where the symbolic file has been loaded
uint64_t InputFile;
// Size of the input file (in bytes)
uint64_t FileSize;
// 1 on success, 0 on failure
uint64_t Result;
} __attribute__((packed));
struct S2E_KAITAI_COMMAND {
enum S2E_KAITAI_COMMANDS Command;
union {
struct S2E_KAITAI_COMMAND_MAKE_SYMBOLIC MakeSymbolic;
};
} __attribute__((packed))
```

然后我们可以将下面的函数添加到`s2e_kaitai_cmd.c`中。这个函数包含指向文件内容（已经读入缓冲区）的指针和缓冲区的大小（由`lseek`确定），构造相关命令并将此命令发送到S2E。

```c
static inline int s2e_kaitai_make_symbolic(const uint8_t *buffer, unsigned size) {
struct S2E_KAITAI_COMMAND cmd = {0};
cmd.Command = S2E_KAITAI_MAKE_SYMBOLIC;
cmd.MakeSymbolic.InputFile = (uintptr_t) buffer;
cmd.MakeSymbolic.FileSize = size;
cmd.MakeSymbolic.Result = 0;
s2e_invoke_plugin("KaitaiStruct", &cmd, sizeof(cmd));
return (int) cmd.MakeSymbolic.Result;
}
```

现在我们需要一个S2E插件来处理这个命令。

## `KaitaiStruct`插件

让我们从一个skeleton插件开始（不要忘了在`source/s2e/libs2eplugins/src/CMakeLists.txt`中向`s2e/Plugins/KaitaiStruct.cpp`添加`add_library`命令）。

头文件：

```c
#ifndef S2E_PLUGINS_KAITAI_STRUCT_H
#define S2E_PLUGINS_KAITAI_STRUCT_H
#include <s2e/CorePlugin.h>
#include <s2e/Plugins/Core/BaseInstructions.h>
// Forward declare the S2E command from s2e_kaitai_cmd
struct S2E_KAITAI_COMMAND;
namespace s2e {
namespace plugins {
// In addition to extending the basic Plugin class, we must also implement the
// BaseInstructionsPluginInvokerInterface to handle custom S2E commands
class KaitaiStruct : public Plugin,
public BaseInstructionsPluginInvokerInterface {
S2E_PLUGIN
public:
KaitaiStruct(S2E *s2e) : Plugin(s2e) { }
void initialize();
// The method from BaseInstructionsPluginInvokerInterface that we must
// implement to respond to a custom command. This method takes the current
// S2E state, a pointer to the custom command object and the size of the
// custom command object
virtual void handleOpcodeInvocation(S2EExecutionState *state,
uint64_t guestDataPtr,
uint64_t guestDataSize);
private:
// The name of the Lua function that will run the Kaitai Struct parser
std::string m_kaitaiParserFunc;
// handleOpcodeInvocation will call this method to actually invoke the Lua
// function
bool handleMakeSymbolic(S2EExecutionState *state,
const S2E_KAITAI_COMMAND &command);
}
} // namespace plugins
} // namespace s2e
#endif
```

cpp 文件：
