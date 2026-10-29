
# [编译llvm](https://bbs.kanxue.com/elink@edcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7h3W2#2N6$3g2A6k6r3!0F1k6#2)9J5c8Y4m8Q4x3V1j5I4y4U0t1I4y4e0j5@1y4#2)9J5k6h3S2@1L8h3H3`.)

（这里我插一句，大家一定不要相信llvm官网的鬼话，说的用windows上的visual studio也可以编译，用他的那个`powershell`，呵呵，之后你就会遇到各种各样奇奇怪怪的错误，还是用linux系统吧）

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_25KNVZUHFFM36JD.png)

### 第一步用`cmake`生成相关的构建文件

```text
cmake -G Ninja -DCMAKE_BUILD_TYPE=RELEASE -DLLVM_TARGETS_TO_BUILD="X86" -DLLVM_ENABLE_PROJECTS="clang" -DLLVM_OPTIMIZED_TABLEGEN=ON ../llvm
```

> 其中`-G Ninja`参数表示生成Ninja系统的构建文件，采用Ninja系统会有比较快的编译速度
> `-DCMAKE_BUILD_TYPE=RELEASE`表示生成Release版本的LLVM，这种构建方式会进行优化，并且生成的目标文件体积会更小
> `-DLLVM_TARGETS_TO_BUILD="X86"`表示编译的目标平台是X86平台
> `-DLLVM_ENABLE_PROJECTS="clang"`表示我们除了编译LLVM以外，还要编译`clang`

然后运行编译命令`ninja`

这一步会等待很长时间，cpu基本处于满载状态：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_Q89K54EFSRW8GNH.png)

按照这样的顺序编译完成的环境中，`clang`应该在这个目录下：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_KXN3PNFVVPZV4WM.png)

这样`clang`的环境就弄好了

## 然后再弄CLion

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_BAZTU352FWUS5EW.png)

按照我之前发的pojie文章安装好clion之后直接打开这个`CMakelists`文件，按照project的方式打开：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_UPYMMG6EDCWN2B3.png)

然后导入文件就好啦，然后再`cmake`的设置中加入`debug`和`release`：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_3FETH74SR2QD78W.png)

输入`-G Ninja -DLLVM_ENABLE_PROJECTS="clang"`

就会生成两个文件：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_QZ9FFBFXQJ372EA.png)

之后要从这两个文件的目录中编译出`so`文件，然后用`opt`命令操作

然后按照llvm官网的write pass的文章中，在这个目录下编写`cpp`和`txt`文件：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_N2XHGVKVBEUQRQD.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_7JACD5HXEJ2GHFS.png)

再添加一下：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_8B96YPAZVYFZ32P.png)

最后用`ninja`编译一下这个工程，建议编译`release`版本，速度快：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_HXFQSVECNCSR2MY.png)

如果能生成这个`so`文件，恭喜您，环境搭建完成

***（windows的铁粉在win上搭环境弄了三天最后还是g了！）***

# 编写函数加密pass

## 修改函数名称

我这里贴一下CLion中的`cpp`的代码是怎么实现函数加密的吧：

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"
using namespace llvm;

namespace {
struct EncodeFunctionName : public FunctionPass {
  static char ID;
  EncodeFunctionName() : FunctionPass(ID) {}
  bool runOnFunction(Function &F) override {
    errs() << "Encode Function Name: ";
    if(F.getName().compare("test_hello1") == 0){
      F.setName("kanxue function");
    }
    errs()<<F.getName()<< '\n';
    return false;
  }
};
}

char EncodeFunctionName::ID = 0;
static RegisterPass<EncodeFunctionName> X("Encode", "Encode Function Name Pass",
  false /* Only looks at CFG */,
  false /* Analysis Pass */);

static llvm::RegisterStandardPasses Y(
  llvm::PassManagerBuilder::EP_EarlyAsPossible,
  [](const llvm::PassManagerBuilder &Builder,
  llvm::legacy::PassManagerBase &PM) { PM.add(new EncodeFunctionName()); });
```

还有对应的`CMakeList`文件：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_VM8P7CBB6YPJW94.png)

然后使用`ninja`命令，就能生成`so`文件（作为函数加密的pass）：

```text
ninja LLVMEncodeFunctionName
```

在这个路径下会找到生成的`so`文件

```text
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/lib/LLVMEncodeFunctionName.so'
```

然后新建一个文件夹作为测试空间

将`clang`的路径添加到文件夹的环境变量中：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_T7Y2FBDCBXCRQNC.png)

```bash
export PATH=/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin:$PATH
```

首先新建一个文件夹，编写`c`文件（作为测试文件），然后生成`ll`文件，

这里我总结了几个命令：

```text
clang -emit-llvm -S hello_clang.c -o hello_clang.ll
lli  hello_clang.ll
llvm-as hello_clang.ll -o hello_clang.bc
llc hello_clang.bc -o hello_clang.s
clang hello_clang.s -o hello_clang_s
llvm-dis hello_clang.bc -o hello_clang_re.ll
```

这个是hello.c文件：

```c
#include <stdio.h>

void test_hello2();

void test_hello1() {
  printf("test_hello1\r\n");
}

int main(int argc, char const *argv[])
{
  if (argc > 2) {
    printf("hello kanxue clang!\r\n");
  } else {
    printf("hello pediy clang!\r\n");
  }
  test_hello1();
  test_hello2();
  return 0;
}

void test_hello2() {
  printf("test_hello2\r\n");
}
```

然后利用上面的命令生成`ll`文件

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_DDA5T6JZMHW5N9A.png)

到这里说明我们的文件夹的环境配置好了

然后接下来该使用`opt`了，（`opt`就是把刚刚编写的`cpp`文件生成的`so`文件加载编译的一个llvm工具）（所以刚刚的`cpp`文件中就应该实现对函数的加密），我这里遇到了一个问题，虽然设置了环境变量，但是`opt`不能用，所以直接从变量路径中拖出来`opt`使用也可以的：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_7WR536RP2WEV36C.png)

干脆直接拖吧，这样还省事

这里用哪个编译出来的`clang`和`opt`可以

location1

```text
'/home/linuxer/llvm/build/bin/'
```

我又在`release`里面编译了一遍，所以之后的路径是

location2

```text
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/'
```

这样就能生成`bc`文件

```text
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/opt' -load '/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/lib/LLVMEncodeFunctionName.so' -Encode hello_clang.ll -o hello_clang_encode.bc
```

然后我们就可以发现对应的函数名称发生了变化

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_WUUEV9KYU7RCHXR.png)

然后使用`clang`编译一下：

```text
'/home/linuxer/llvm/llvm_source_code/llvm/cmake-build-release/bin/clang' hello_clang_encode.bc -o hello_clang_encode
```

然后执行一下：

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_WZHU32U7PS3TFYB.png)

程序正常执行

然后用`ida`看看函数加密情况

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_432K3NRDWMUJ7KZ.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202211/939330_QWPAZ36UC52GYAN.png)

发现函数加密成功！

## 对函数进行md5加密

贴一下`cpp`代码：

```cpp
#include "llvm/Pass.h"
#include "llvm/IR/Function.h"
#include "llvm/Support/raw_ostream.h"
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"
using namespace llvm;

namespace {
struct EncodeFunctionName : public FunctionPass {
  static char ID;
  EncodeFunctionName() : FunctionPass(ID) {}
  bool runOnFunction(Function &F) override {
    errs() << "Encode Function Name: " << F.getName() << " -> ";
    if(F.getName().compare("main") == 0){
      llvm::MD5 Hasher;
      llvm::MD5::MD5Result Hash;
      Hasher.update("kanxue_");
      Hasher.update(F.getName());
      Hasher.final(Hash);

      SmallString<32> HexString;
      llvm::MD5::stringifyResult(Hash, HexString);

      F.setName(HexString);
    }
    errs()<<F.getName()<< '\n';
    return false;
  }
};
}

char EncodeFunctionName::ID = 0;
static RegisterPass<EncodeFunctionName> X("Encode", "Encode Function Name Pass",
  false /* Only looks at CFG */,
  false /* Analysis Pass */);

static llvm::RegisterStandardPasses Y(
  llvm::PassManagerBuilder::EP_EarlyAsPossible,
  [](const llvm::PassManagerBuilder &Builder,
  llvm::legacy::PassManagerBase &PM) { PM.add(new EncodeFunctionName()); });
```

然后`ninja`编译一下，生成`so`文件
