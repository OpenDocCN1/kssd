
```llvm
; ModuleID = 'main.c'
source_filename = "main.c"
target datalayout = "e-m:e-i64:64-f80:128-n8:16:32:64-S128"
target triple = "x86_64-pc-linux-gnu"

@.str = private unnamed_addr constant [6 x i8] c"fun1\0A\00", align 1
@.str.1 = private unnamed_addr constant [5 x i8] c"bye\0A\00", align 1

; Function Attrs: noinline nounwind optnone uwtable
define i32 @function1() #0 {
  %1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([6 x i8], [6 x i8]* @.str, i32 0, i32 0))
  ret i32 0
}

declare i32 @printf(i8*, ...) #1

; Function Attrs: noinline nounwind optnone uwtable
define i32 @function2() #0 {
  %1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([6 x i8], [6 x i8]* @.str, i32 0, i32 0))
  ret i32 0
}

; Function Attrs: noinline nounwind optnone uwtable
define i32 @function3() #0 {
  %1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([6 x i8], [6 x i8]* @.str, i32 0, i32 0))
  ret i32 0
}

; Function Attrs: noinline nounwind optnone uwtable
define i32 @Ayaka() #0 {
  %1 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([6 x i8], [6 x i8]* @.str, i32 0, i32 0))
  ret i32 0
}

; Function Attrs: noinline nounwind optnone uwtable
define i32 @main() #0 {
  %1 = alloca [16 x i8], align 16
  %2 = getelementptr inbounds [16 x i8], [16 x i8]* %1, i32 0, i32 0
  %3 = call i64 @read(i32 0, i8* %2, i64 16)
  %4 = getelementptr inbounds [16 x i8], [16 x i8]* %1, i32 0, i32 0
  %5 = call i64 @write(i32 1, i8* %4, i64 16)
  %6 = call i32 (i8*, ...) @printf(i8* getelementptr inbounds ([5 x i8], [5 x i8]* @.str.1, i32 0, i32 0))
  ret i32 0
}

declare i64 @read(i32, i8*, i64) #1

declare i64 @write(i32, i8*, i64) #1

attributes #0 = { noinline nounwind optnone uwtable "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

attributes #1 = { "correctly-rounded-divide-sqrt-fp-math"="false" "disable-tail-calls"="false" "less-precise-fpmad"="false" "no-frame-pointer-elim"="true" "no-frame-pointer-elim-non-leaf" "no-infs-fp-math"="false" "no-jump-tables"="false" "no-nans-fp-math"="false" "no-signed-zeros-fp-math"="false" "no-trapping-math"="false" "stack-protector-buffer-size"="8" "target-cpu"="x86-64" "target-features"="+fxsr,+mmx,+sse,+sse2,+x87" "unsafe-fp-math"="false" "use-soft-float"="false" }

!llvm.module.flags = !{!0}
!llvm.ident = !{!1}

!0 = !{i32 1, !"wchar_size", i32 4}
!1 = !{!"clang version 6.0.0-1ubuntu2 (tags/RELEASE_600/final)"}
```

接下来我们用官方给的小 demo 写一个 LLVM PASS 出来：

```cpp
#include "llvm/Pass.h" // 写 Pass 所必须的库
#include "llvm/IR/Function.h" // 操作函数所必须的库
#include "llvm/Support/raw_ostream.h" // 打印输出所必须的库
#include "llvm/IR/LegacyPassManager.h"
#include "llvm/Transforms/IPO/PassManagerBuilder.h"

using namespace llvm;

namespace { // 声明匿名空间，被声明的内容仅在文件内部可见
struct Hello : public FunctionPass {
  static char ID;
  Hello() : FunctionPass(ID) {}

  bool runOnFunction(Function &F) override {
    // 重写 runOnFunction，使得每次遍历到一个函数的时候就输出函数名
    errs() << "Hello: ";
    errs().write_escaped(F.getName()) << '\n';
    return false;
  }
};
}

char Hello::ID = 0;

// Register for opt
static RegisterPass<Hello> X("hello", "Hello World Pass"); // 注册类 Hello，第一个参数是命令行参数，第二个参数是名字

// Register for clang
static RegisterStandardPasses Y(
    PassManagerBuilder::EP_EarlyAsPossible,
    [](const PassManagerBuilder &Builder, legacy::PassManagerBase &PM) {
      PM.add(new Hello());
    });
```

这段代码大致意思是注册了一个`hello`函数，重写了`runOnFunction`函数，这样的话每次遍历到一个函数就会调用一次`hello`，而`hello`的功能也很简单，就是把函数名输出出来。

接下来输入如下命令将其编译成一个 so 文件：
