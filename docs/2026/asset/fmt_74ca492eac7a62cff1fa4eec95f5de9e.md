
# [原创]Using Microsoft Visual Studio 2019 Building A LLVM Out-Source-Tree Pass (续)

Using Microsoft Visual Studio 2019 Building A LLVM Out-Source-Tree Pass(续)

=========

* [Command Line Options](#command-line-options)
* [clang-cl](#clang-cl)
* [Writing a StringObfuscator LLVM Pass](#writing-a-stringobfuscator-llvm-pass)
* [Using the StringObfuscator LLVM Pass Processing IR](#using-the-stringobfuscator-llvm-pass-processing-ir)
* [End](#end)

## Command Line Options

==========
* clang
```bash
$LLVM_DIR/bin/clang.exe -help
```
* clang-cl
```bash
$LLVM_DIR/bin/clang-cl.exe /?
```
* -cc1 options
```bash
$LLVM_DIR/bin/clang.exe -cc1 -help
$LLVM_DIR/bin/clang-cl.exe -cc1 -help
```
# LLVM官网是说不建议去自己设置 -cc1 的编译选项,容易出问题.

## clang-cl

========
`Visual Studio 2019下使用clang-cl.exe来编译链接`.
`clang-cl`是基于`clang`的扩展, 旨在兼容`Visual C++编译器cl.exe`, 帮你关联头文件和库文件.

```bash
PS K:\llvm\DebugLLVMPass> K:\llvm\build\Debug\bin\clang-cl.exe -### hello1.c -o .\hello1.exe
clang version 10.0.0
Target: x86_64-pc-windows-msvc
Thread model: posix
InstalledDir: K:\llvm\build\Debug\bin
# obj
"K:\\llvm\\build\\Debug\\bin\\clang-cl.exe" "-cc1" "-triple" "x86_64-pc-windows-msvc19.25.28614" "-emit-obj" "-mrelax-all" "-mincremental-linker-compatible" "-disable-free" "-main-file-name" "hello1.c" "-mrelocation-model" "pic" "-pic-level" "2" "-mthread-model" "posix" "-mframe-pointer=none" "-relaxed-aliasing" "-fmath-errno" "-fno-rounding-math" "-masm-verbose" "-mconstructor-aliases" "-munwind-tables" "-target-cpu" "x86-64" "-mllvm" "-x86-asm-syntax=intel" "-D_MT" "-flto-visibility-public-std" "--dependent-lib=libcmt" "--dependent-lib=oldnames" "-stack-protector" "2" "-fms-volatile" "-fdiagnostics-format" "msvc" "-dwarf-column-info" "-resource-dir" "K:\\llvm\\build\\Debug\\lib\\clang\\10.0.0" "-internal-isystem" "K:\\llvm\\build\\Debug\\lib\\clang\\10.0.0\\include" "-internal-isystem" "C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\VC\\Tools\\MSVC\\14.25.28610\\include" "-internal-isystem" "C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\VC\\Tools\\MSVC\\14.25.28610\\atlmfc\\include" "-internal-isystem" "C:\\Program Files (x86)\\Windows Kits\\10\\Include\\10.0.18362.0\\ucrt" "-internal-isystem" "C:\\Program Files (x86)\\Windows Kits\\10\\include\\10.0.18362.0\\shared" "-internal-isystem" "C:\\Program Files (x86)\\Windows Kits\\10\\include\\10.0.18362.0\\um" "-internal-isystem" "C:\\Program Files (x86)\\Windows Kits\\10\\include\\10.0.18362.0\\winrt" "-fdebug-compilation-dir" "K:\\llvm\\DebugLLVMPass" "-ferror-limit" "19" "-fmessage-length" "147" "-fno-use-cxa-atexit" "-fms-extensions" "-fms-compatibility" "-fms-compatibility-version=19.25.28614" "-fdelayed-template-parsing" "-fobjc-runtime=gcc" "-fdiagnostics-show-option" "-fcolor-diagnostics" "-faddrsig" "-o" "C:\\Users\\XXX\\AppData\\Local\\Temp\\hello1-1b03fd.obj" "-x" "c" "hello1.c"
# link
"C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\VC\\Tools\\MSVC\\14.25.28610\\bin\\Hostx64\\x64\\link.exe" "-out:.\\hello1.exe" "-libpath:C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\VC\\Tools\\MSVC\\14.25.28610\\lib\\x64" "-libpath:C:\\Program Files (x86)\\Microsoft Visual Studio\\2019\\Enterprise\\VC\\Tools\\MSVC\\14.25.28610\\atlmfc\\lib\\x64" "-libpath:C:\\Program Files (x86)\\Windows Kits\\10\\Lib\\10.0.18362.0\\ucrt\\x64" "-libpath:C:\\Program Files (x86)\\Windows Kits\\10\\Lib\\10.0.18362.0\\um\\x64" "-nologo" "C:\\Users\\XXX\\AppData\\Local\\Temp\\hello1-1b03fd.obj"
```

有关更多详细信息，请阅读 clang-cl官方文件资料.

## Writing a StringObfuscator LLVM Pass

=============
这边纯粹从小白的角度讲起，说说这个StringObfuscator Pass Code怎么一步一步构建而成，叙述稍微繁复，至于编译调试啥的，你就参照上一篇来整。

以下代码为例：

```
<llvm-tutor source dir>/
|
CMakeLists.txt
<HelloWorld>/
|
HelloWorld.cpp
CMakeLists.txt
<StringObfuscator>/
|
StringObfuscator.cpp
CMakeLists.txt
```

往llvm-tutor项目创建`StringObfuscator目录`, 目录下创建`StringObfuscator.cpp`，`CMakeLists.txt`，并把`StringObfuscator`项目加入`llvm-tutor\CMakeLists.txt`中编译。

### StringObfuscator/StringObfuscator.cpp

```C
#include "llvm/IR/PassManager.h"
#include "llvm/Passes/PassBuilder.h"
#include "llvm/Passes/PassPlugin.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/ValueSymbolTable.h"
#include "llvm/IR/IRBuilder.h"
#include "llvm/Transforms/Utils/Cloning.h"
#include <typeinfo>
using namespace std;
using namespace llvm;
struct StringObfuscatorModPass : public PassInfoMixin<StringObfuscatorModPass>
{
PreservedAnalyses run(Module &M, ModuleAnalysisManager &MAM)
{
return PreservedAnalyses::all();
};
};
} // end anonymous namespace
extern "C" ::llvm::PassPluginLibraryInfo LLVM_ATTRIBUTE_WEAK
llvmGetPassPluginInfo()
{
return {
LLVM_PLUGIN_API_VERSION, "StringObfuscatorPass", "v0.1", [](PassBuilder &PB) {
PB.registerPipelineParsingCallback([](StringRef Name, ModulePassManager &MPM,
ArrayRef<PassBuilder::PipelineElement>) {
if (Name == "string-obfuscator-pass") {
MPM.addPass(StringObfuscatorModPass());
return true;
}
return false;
}
);
}
};
}
```

上面是一个空的`StringObfuscator`插件，什么都没做，接下来要设计这个`Pass`，作用是对全局字符串简单加密。

#### `等等！！！`

到这就很懵了，怎么设计你可能知道，但是怎么把设计的写成`Pass`，就让人无从下手了。

恭喜你，抬起头，你会看到LLVM PASS这座大山就再你眼前，到此处，怎么整就是有很大问题了，这是因为你欠缺了很多必备的基础知识，下面我罗列下，如果你都眼熟，那么这山就不是山了是坦途。

* LLVM Programmer’s Manual
* LLVM Language Reference Manual
* Writing an LLVM Pass
* Writing an LLVM Pass: 101 [Video] [Slides]
* LLVM-IR 层次机构
```
llvm::Module
llvm::GlobalVariable
llvm::Function
llvm::BasicBlock
llvm::instruction
llvm::ICmpInst
llvm::BranchInst
```
* LLVM-IR 命名方式
```
global symbols start with "@"
local symbols start with "%"
Basic block names start with "%" when used
Basic block names end with "%" when defined
```
* 每个Basic block都需要有个Terminator指令为结尾
```
# Terminator
Ret
Br
Switch
IndirectBr
Invoke
Resume
Unreachable
CleanupRet
CatchRet
CatchSwitch
CallBr
```
* IRBuilder源码
* llvm unittests源码
