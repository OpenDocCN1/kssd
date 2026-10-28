
```cpp
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

`cpp` 文件：

```cpp
// From source/s2e/guest/common/include
#include <s2e/kaitai/commands.h>

#include <s2e/ConfigFile.h>
#include <s2e/S2E.h>
#include <s2e/Utils.h>

#include "KaitaiStruct.h"

namespace s2e {
namespace plugins {

S2E_DEFINE_PLUGIN(KaitaiStruct,
                  "Combine S2E and Kaitai Struct",
                  "",
                  // Dependencies
                  "LuaBindings"); // Reuse the existing Lua binding code from
                                  // the function/instruction annotation
                                  // plugins

void KaitaiStruct::initialize() {
    m_kaitaiParserFunc = s2e()->getConfig()->getString(getConfigKey() +
                                                       ".parser");
}

bool KaitaiStruct::handleMakeSymbolic(S2EExecutionState *state,
                                      const S2E_KAITAI_COMMAND &command) {
    // We'll finish this later
    return true;
}

void KaitaiStruct::handleOpcodeInvocation(S2EExecutionState *state,
                                          uint64_t guestDataPtr,
                                          uint64_t guestDataSize) {
    S2E_KAITAI_COMMAND cmd;

    // 1. Validate the received command
    if (guestDataSize != sizeof(cmd)) {
        getWarningsStream(state) << "S2E_KAITAI_COMMAND: Mismatched command "
                                 << "structure size " << guestDataSize << "\n";
        exit(1);
    }

    // 2. Read the command
    if (!state->mem()->readMemoryConcrete(guestDataPtr, &cmd, guestDataSize)) {
        getWarningsStream(state) << "S2E_KAITAI_COMMAND: Failed to read "
                                 << "command\n";
        exit(1);
    }

    // 3. Handle the command
    switch (cmd.Command) {
        case KAITAI_MAKE_SYMBOLIC: {
            bool success = handleMakeSymbolic(state, cmd);
            cmd.MakeSymbolic.Result = success ? 0 : 1;

            // Write the result back to the guest
            if (!state->mem()->writeMemory(guestDataPtr, cmd)) {
                getWarningsStream(State) << "S2E_KAITAI_COMMAND: Failed to "
                                         << " write result to guest\n";
                exit(1);
            }
        } break;

        default: {
            getWarningsStream(state) << "S2E_KAITAI_COMMAND: Invalid command "
                                     << hexval(cmd.Command) << "\n";
            exit(1);
        }
    }
}

} // namespace plugins
} // namespace s2e
```

我们的插件只有一个依赖关系：`LuaBindings`插件。这个插件配置了S2E的Lua解释器，并允许我们在S2E配置文件中调用Lua代码。

`handleOpcodeInvocation`方法遵循和其他插件类似的方法，实现了`BaseInstructionsPluginInvokerInterface`接口（例如`FunctionModels`和`LinuxMonitor`）：

1. 通过检查它的大小来验证接收的命令。
2. 读取命令。由于该命令是由客户机发出的，因此它驻留在客户机内存中。我们的命令都不是符号化的（记住它只包含输入文件的起始地址和大小），所以我们可以详细地读取这个内存内容。
3. 处理命令。在这种情况下，我们调用另一个函数（我们将在稍后讨论）来调用Lua解释器解析输入文件。
4. 显示客户机的成功/失败。我们通过在命令结构中设置“返回值”并将命令写回到客户端内存中。

最终实现`MakeSymbolic`。为了编写Lua代码，需要添加一些头文件：

```cpp
#include <vector>

#include <s2e/Plugins/Lua/Lua.h>
#include <s2e/Plugins/Lua/LuaS2EExecutionState.h>
```

最终实现的函数：

```cpp
bool KaitaiStruct::handleMakeSymbolic(S2EExecutionState *state,
                                      const S2E_KAITAI_COMMAND &command) {
    uint64_t addr = command.MakeSymbolic.InputFile;
    uint64_t size = command.MakeSymbolic.FileSize;
    std::vector<uint8_t> data(size);

    // Read the input file's contents from guest memory
    if (!state->mem()->readMemoryConcrete(addr, data.data(),
                                          sizeof(uint8_t) * size)) {
        return false;
    }

    // Get the Lua interpreter's state
    lua_State *L = s2e()->getConfig()->getState();

    // Wrap the current S2E execution state
    LuaS2EExecutionState luaS2EState(state);

    // Turn the input file into a Lua string
    luaL_Buffer luaBuff;
    luaL_buffinit(L, &luaBuff);
    luaL_addlstring(&luaBuff, (char*) data.data(), sizeof(uint8_t) * size);

    // Set up our function call on Lua's virtual stack
    lua_getglobal(L, m_kaitaiParserFunc.c_str());
    Lunar<LuaS2EExecutionState>::push(L, &luaS2EState);
    lua_pushinteger(L, addr);
    luaL_pushresult(&luaBuff);

    // Call our Kaitai Struct parser function
    lua_call(L, 3, 0);

    return true;
}
```

希望这比较容易理解（参见[这里](https://bbs.kanxue.com/elink@d03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9N6h3q4Q4x3X3g2G2M7X3N6Q4x3V1k6E0j5h3&6#2j5h3I4Q4x3V1j5#2i4K6u0W2x3#2)9J5c8X3#2S2L8Y4g2S2L8q4)9J5k6h3S2@1L8h3I4Q4x3U0x3@1i4K6u0W2z5l9`.`.)有关Lua语言的C API的更多信息）。首先，我们将输入文件读入Kaitai Struct解析器的Lua字符串。然后，我们调用Kaitai Struct解析器函数（我们将在下一部分中定义）。

我们必须设置解析器函数的参数才能调用它。用栈把值传递给Lua函数。函数名首先入栈。解析器函数在Lua的全局命名空间中定义（为了简单起见），因此我们可以使用`lua_getglobal`从S2E配置文件中检索该函数，并将其压入栈中。然后依次入栈：

1. 当前S2E执行状态;
2. 输入文件的起始地址（在客户机内存中）;
3. 输入文件的内容（作为字符串）。

现在要做的就是在S2E配置文件中实现这个解析器。

## Lua脚本

首先，我们需要将Kaitai Struct格式的定义编译成Lua解析器。既然我们是用`readelf`做实验，现在让我们创建一个`readelf`项目，并从Kaitai Struct Gallery获取ELF定义：

```bash
# Create the S2E project
s2e new_project -n readelf_kaitai readelf -h @@

cd projects/readelf_kaitai

# Get the ELF Kaitai Struct definition and compile it
wget 103K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4N6Q4x3X3g2Y4K9i4c8Z5N6h3u0#2M7$3g2J5j5$3!0F1N6r3g2F1N6q4)9J5k6h3y4G2L8g2)9J5c8X3E0S2K9i4c8S2K9g2)9J5k6r3W2G2i4K6u0r3K9$3q4A6N6r3q4A6i4K6g2X3M7%4c8J5N6h3y4@1i4K6g2X3k6X3!0J5L8h3q4@1M7#2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3g2^5k6h3y4#2N6r3q4T1L8r3g2Q4x3V1k6W2L8r3k6Q4x3X3g2C8M7%4V1`.

ksc -t lua elf.ksy
```
