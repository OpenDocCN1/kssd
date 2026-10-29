
# [原创]使用 ollvm 自定义简单的字符串加密

题目出自 3W 班 9 月的题目，用 ollvm 9.0 实现字符串简单加密。

题目主要是为了能熟练 ollvm 中如何进行一个简单的加密，以及 C++ 部分怎么生成对应的 IR 指令来达到像 C++ 函数效果。所以主要我们的思路可以切换成：

1. 首先确定加密的核心逻辑并用 C++ 实现。
2. 根据 C++ 的算法，生成一份 IR 指令来为我们提供参考。
3. ollvm 里面如何用 C++ 生成相应的 IR 指令。
4. 根据参考的 IR 指令来生成我们需要的加密和解密函数。

做完这个题目，基本就可以对 ollvm 的工作原理有一定的了解，并且改造属于自己的加密或者混淆了。

先贴上测试好的结果：<mark class="encrypted">cc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1M7i4A6Y4x3e0t1K6x3o6m8Q4x3V1k6C8e0@1I4x3g2V1#2Q4x3X3g2Y4K9i4b7`。</mark>

想要写一个字符串加密的 pass，第一步就是先实现一遍 C++ 的算法流程，然后再看一看生成的 IR 文件，然后再写对应的加密 pass，下面看一个自己实现的简单 C++ 字符串加密。

```c++
#include <stdio.h>
#include <cstring>
#include <string>

int main(int argc, char** argv) {
    // 加密
    std::string str1 = "hello world!!!";
    // 这里是随机的 key，先写固定，真实实现的时候再每个字节使用一个随机 key
    int randkey = 11;
    // 加密复杂度
    int kstr_size = 10;
    int enclen = randkey + kstr_size;
    char encres[str1.size()];
    int idx = 0;
    memset(encres, 0, enclen);

    // 这里大概就是遍历字符串，每个字符根据加密复杂度进行一定数量迭代异或，最后一次的迭代使用取反再异或
    for (int i = 0; i < str1.size(); i++) {
        printf("cur: %x \r\n", str1[i]);
        for (int y = randkey; y < enclen; y++) {
            if (y == randkey) {
                encres[i] = str1[i] ^ y;
            } else if (y == enclen - 1) {
                encres[i] = encres[i] ^ (~y);
            } else {
                encres[i] = encres[i] ^ y;
            }
            printf("%x ", encres[i]);
            idx++;
        }
        printf("\r\n");
    }
    printf("encdata: %s\r\n", encres);

    // 下面是解密函数
    char decres[str1.size()];
    for (int i = 0; i < str1.size(); i++) {
        printf("cur enc: %x \r\n", encres[i]);
        for (int y = enclen - 1; y >= randkey; y--) {
            if (y == enclen - 1) {
                decres[i] = encres[i] ^ (~y);
            } else {
                decres[i] = decres[i] ^ y;
            }
            printf("%x ", decres[i]);
        }
        printf("\r\n");
    }
    printf("res: %s\r\n", decres);
    return 0;
}
```

这个简单加密的意思，就是根据复杂度参数，来进行一定次数的迭代，将当前字符每次都异或一下，最后一次是先去反，再异或，解密就是反之。测试结果能正常加密和解密后，我们就先输出一份 IR 文件，看看在 IR 中间语言中是如何进行加密和解密的。

```bash
clang -emit-llvm -S main.cpp -o main.ll
```

生成好对应的 IR 文件后，我们开始写这个加密 pass，然后再写的过程中，根据逻辑需要，去 IR 中找对应的指令处理方式。

在 IR 文件中的层级划分：`Module`（模块）的下一层是若干 `Function`（函数），然后在 `Function` 的下一层是若干 `BasicBlock`（基本块），再 `BasicBlock` 的下一层是若干 `Instruction`（指令块）。

现在准备就绪，下面开始先准备一个加密的 pass，基本代码如下：

```c++
#include <kllvm/Transforms/Obfuscation/Utils.h>
#include "kllvm/Transforms/Obfuscation/KStringEncode.h"
#include <string>

using namespace llvm;

namespace {

// 加密复杂度
const int defaultKStringSize = 0x10;

static cl::opt<int>
KStringSize("kstr_size", cl::desc("Choose the probability [%] each basic blocks will be obfuscated by the -kstr pass"),
            cl::value_desc("king string encode Encryption length"), cl::init(defaultKStringSize), cl::Optional);

struct KStringEncode : public FunctionPass {
    static char ID; // Pass identification
    bool flag;

    KStringEncode() : FunctionPass(ID) {}
    KStringEncode(bool flag) : FunctionPass(ID) { this->flag = flag; KStringEncode(); }

    virtual bool runOnFunction(Function &F) {
        // 先检查加密复杂度是否在合法范围
        if (!((KStringSize > 0) && (KStringSize <= 100))) {
            errs() << "KStringEncode application basic blocks percentage -kstr_size=x must be 0 < x <= 100";
            return false;
        }
        if (toObfuscate(flag, &F, "kstr")) {
            kstr(F);
        }
        return false;
    }

    void kstr(Function& func) {
        // todo 这里再写具体的 pass 逻辑
    }
};

} // namespace

char KStringEncode::ID = 0;
static RegisterPass<KStringEncode> X("kstr", "inserting bogus control flow");

Pass *llvm::createKStringEncode() {
    return new KStringEncode();
}

Pass *llvm::createKStringEncode(bool flag) {
    return new KStringEncode(flag);
}
```

这里准备好了 pass 的基本代码后，最后就剩下最重要的核心逻辑，如何把 C++ 的加密方式在 pass 中实现。我们的功能是实现字符串加密，那么第一步应该是取得这个函数中的全部字符串，那么我们先看看 IR 中字符串的特征。

```assembly
@.str = private unnamed_addr constant [15 x i8] c"hello world!!!\00", align 1
```

可以看到，这个 `str` 是一个操作数，想要获取全部字符串，就得先遍历所有指令块中的操作数，然后再根据字符串的特征来进行过滤。下面先看如何遍历所有指令块。

```c++
void kstr(Function& func) {
    for (BasicBlock& bb : func) {
        for (Instruction& ins : bb) {
            for (Value* val : ins.operands()) {
                Value* stripOp = val->stripPointerCasts();
                if (stripOp->getName().contains(".str")) {
                    errs() << ins << "\n";
                    errs() << *val << "\n";
                    errs() << *stripOp << "\n";
                }
            }
        }
    }
}
```

上面遍历了函数中的所有基本块，然后遍历所有指令块，然后遍历所有操作数，然后获取操作数的值，判断该操作数是否是一个字符串，并且打印这个指令块、操作数，以及取到的操作数的值，下面看看打印的结果。

```assembly
store i8* getelementptr inbounds ([7 x i8], [7 x i8]* @.str, i64 0, i64 0), i8** %str, align 8
i8* getelementptr inbounds ([7 x i8], [7 x i8]* @.str, i64 0, i64 0)
@.str = private unnamed_addr constant [7 x i8] c"kanxue\00", align 1
```

那么看到了，我们想获取的字符串是在 `stripOp` 中。那么接下来就把所有字符串全部获取出来并转换成 `string`。

```c++
// 封装一个转换操作数值为字符串的函数
std::string ConvertOpToString(Value* op) {
    GlobalVariable* globalVar = dyn_cast<GlobalVariable>(op);
    if (!globalVar) {
        errs() << "dyn cast gloabl err";
        return "";
    }
    ConstantDataSequential* cds = dyn_cast<ConstantDataSequential>(globalVar->getInitializer());
    if (!cds) {
        errs() << "dyn cast constant data err";
        return "";
    }
    return cds->getRawDataValues();
}

void kstr(Function& func) {
    for (BasicBlock& bb : func) {
        for (Instruction& ins : bb) {
            for (Value* val : ins.operands()) {
                Value* stripOp = val->stripPointerCasts();
                if (stripOp->getName().contains(".str")) {
                    std::string strdata = ConvertOpToString(stripOp);
                    errs() << strdata << "\n";
                }
            }
        }
    }
}
```

之前看到的字符串的 IR 代码看到所有字符串都是全局的，所以要先转换成全局的对象，然后再转换成数值。然后看这里的打印结果。

```
kanxue
hello ollvm:%d
```

获取到所有的字符串了之后，接下来，我们要先把这个字符串加密，然后再用插入指令块来进行解密。下面继续完善，先把之前搞好的加密算法迁移进来。

```c++
// 转换字符串
std::string ConvertOpToString(Value* op) {
    GlobalVariable* globalVar = dyn_cast<GlobalVariable>(op);
    if (!globalVar) {
        errs() << "dyn cast gloabl err";
        return "";
    }
    ConstantDataSequential* cds = dyn_cast<ConstantDataSequential>(globalVar->getInitializer());
    if (!cds) {
        errs() << "dyn cast constant data err";
        return "";
    }
    return cds->getRawDataValues();
}

void kstr(Function& func) {
    for (BasicBlock& bb : func) {
        for (Instruction& ins : bb) {
            for (Value* val : ins.operands()) {
                Value* stripOp = val->stripPointerCasts();
                if (stripOp->getName().contains(".str")) {
                    std::string strdata = ConvertOpToString(stripOp);
                    errs() << strdata << "\n";

                    // 加密流程
                    uint8_t keys[strdata.size()];
                    char encres[strdata.size()];
                    int idx = 0;
                    memset(encres, 0, strdata.size());

                    for (int i = 0; i < strdata.size(); i++) {
                        uint8_t randkey = llvm::cryptoutils->get_uint8_t();
                        keys[i] = randkey;
                        int enclen = randkey + defaultKStringSize;
                        for (int y = randkey; y < enclen; y++) {
                            if (y == randkey) {
                                encres[i] = strdata[i] ^ y;
                            } else if (y == enclen - 1) {
                                encres[i] = encres[i] ^ (~y);
                            } else {
                                encres[i] = encres[i] ^ y;
                            }
                            idx++;
                        }
                    }
                }
            }
        }
    }
}
```

这里大致流程和之前一样，只是 `key` 我们装起来了，然后每个字节处理都随机一次 key。接下来的处理就是插入指令块来对这个加密数据 `encres` 进行解密还原处理。

我们想要处理这个加密的数据，首先要先创建一个内存指令，来存放这个加密后的数据，然后再对加密后的数据遍历，进行还原。所以，我们的下一步先创建一个 `BitCastInst`，并且我们需要用一个 `int8` 的 `array` 来给这个内存指令进行赋值。下面的代码是先创建 `array` 指令，然后用 `array` 指令创建一个内存指令。

```c++
ArrayType* arrType = ArrayType::get(Type::getInt8Ty(func.getContext()), strdata.size());
AllocaInst* arrayInst = new AllocaInst(arrType, 0, nullptr, 1, Twine(stripOp->getName() + ".arr"), &ins);
BitCastInst* bitInst = new BitCastInst(arrayInst, Type::getInt8PtrTy(func.getParent()->getContext()), Twine(stripOp->getName() + "bitcast"), &ins);
```

上面的就是先创建一个 `int8` 的 `array` 类型，然后用这个类型创建一个 `array`，然后再用这个 `array` 创建内存指令，这些指令都插入在遍历到字符串指令的当前行的前方。这个 `bitcast` 将用来存放加密后的字符串数据。

接下来就是加密的逻辑处理，和我们之前 C++ 的流程一样，只不过这里需要换成插入指令块的形式来进行加密数据的还原，我直接贴上解密的代码部分，然后里面有详细的注释。

```c++
ArrayType* arrType = ArrayType::get(Type::getInt8Ty(func.getContext()), strdata.size());
AllocaInst* arrayInst = new AllocaInst(arrType, 0, nullptr, 1, Twine(stripOp->getName() + ".arr"), &ins);
BitCastInst* bitInst = new BitCastInst(arrayInst, Type::getInt8PtrTy(func.getParent()->getContext()), Twine(stripOp->getName() + ".bitcast"), &ins);

// 创建一个对象用来存放当前加密字节解密时每次异或的结果
AllocaInst* eor_res = new AllocaInst(Type::getInt8Ty(func.getContext()), 0, nullptr, 1, Twine(stripOp->getName() + ".alloc.res"), &ins);

for (int i = 0; i < strdata.size(); i++) {
    uint8_t randkey = keys[i];
    int enclen = randkey + defaultKStringSize;

    ConstantInt* enc_const = ConstantInt::get(Type::getInt8Ty(func.getContext()), encres[i]);
    // 用来存放解密结果的 bitcat
    ConstantInt* i_const = ConstantInt::get(Type::getInt8Ty(func.getContext()), i);
    GetElementPtrInst* element = GetElementPtrInst::CreateInBounds(bitInst, i_const);
    element->insertBefore(&ins);

    StoreInst* last_store = nullptr;

    for (int y = enclen - 1; y >= randkey; y--) {
        /* 下面是获取 y 的指令块 */
        // 先是创建一个数值 y
}
