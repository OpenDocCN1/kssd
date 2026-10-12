
# VMProtect 与 ASProtect 在 VC 中的 SDK 编程

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-455.htm`](https://www.pediy.com/kssd/pediy09/pediy09-455.htm)

> 最近想用 VMProtect 和 ASProtect 的 SDK 加密一个程序，结果搞了半天没搞成，网上没看到在 VC 中如何使用 VMProtect 的 SDK 加密，于是琢磨了一下，总算成功了，最后有一点点心得，与大家共享，免得还有人跟我一样走弯路。

> 其实 VMProtect 和 ASProtect 的 SDK 编程都差不多，都是在编程时在语句里面插入一个标记(Marker)，然后在加壳时，加壳程序就会认出这些标记，并在有标记的地方进行保护。 我觉得这就是最基本的所谓加壳程序 SDK 编程吧。

### 1. VMProtect

> 新版的 VMProtect 没看到好用的，所以这里用的 `VMProtect v1.2`。 (有新的好用的请说一声). 编译环境 `VS.Net 2003`. 主要参考了文章. `VMProtect SDK+ASProtect SDK 混合编程[代码演示]` 作 者: `Anskya` 链 接: [`bbs.pediy.com/showthread.php?threadid=20317`](http://bbs.pediy.com/showthread.php?threadid=20317)

#### 1.1 保护内部函数

> Delphi 需要设置 Map 文件，那 `VS.Net 2003` 也需要设置一下，以便产生 Map 文件，使 `VMProtect` 能够识别内部函数，如果没 Map 文件，`VMProtect` 只能识别导出函数，到时就要自己加函数的地址了.

> `VS.Net 2003` 中，打开 "项目 -> `project1` 属性(假设项目名为 `project1`) -> 链接器 -> 调试 -> 生成映射文件"， 改成 "是(/MAP)",这样生成程序时会同时生成 `project1.map` 文件。 将 `project1.exe` 与 `project1.map` 都拷贝到 VMProtect 程序夹，用 VMProtect 加密时，添加新的函数时能够列出很多内部函数，这时只需要选择想加密的内部函数就行了。 如果没 Map，只有个输入函数地址的输入筐，没有内部函数列表。

> 这里可以省去有些朋友为了加密某个函数，还要用 `Ollydbg` 分析程序，找到函数入口地址，然后输入到 VMProtect 里面来保护的麻烦。

#### 1.2 保护任意位置代码

> 在 `Anskya` 和那个俄国作者的主页上都只说了 Delphi 在程序中加入 SDK 标记。在 Delphi 中的标记模式为：

```
asm 
  db $EB,$10,'VMProtect begin',0     //标记开始处.
end;
//想保护的程序代码
asm
  db $EB,$0E,'VMProtect end',0       //标记结束处.
end;
```

> 在 Delphi 中可以直接用这种类似 MASM 的汇编语言，比较方便，但是在 VC 中不支持 `db` 语句，只支持插入单个字节的 `_emit` 语句。 我琢磨出了一个比较麻烦的方法，就是全部插入 16 进制字节码，不晓得还有更好的方法没。此方法学自 ASProtect 的 sdk 例子里面的 `aspr.h`。

> 在 VC 中的标记模式为：

```
__asm //标记开始处.
{
  _emit 0xEB
  _emit 0x10  //jmp 0x10
  _emit 0x56  //ascii "VMProtect begin",0
  _emit 0x4D  
  _emit 0x50 
  _emit 0x72 
  _emit 0x6F 
  _emit 0x74 
  _emit 0x65 
  _emit 0x63 
  _emit 0x74 
  _emit 0x20 
  _emit 0x62 
  _emit 0x65 
  _emit 0x67 
  _emit 0x69 
  _emit 0x6E 
  _emit 0x00
}
//想保护的程序代码
__asm //标记结束处.
{
  _emit 0xEB
  _emit 0x0E   //jmp 0x0e
  _emit 0x56   //ascii "VMProtect end",0
  _emit 0x4D   
  _emit 0x50 
  _emit 0x72 
  _emit 0x6F 
  _emit 0x74 
  _emit 0x65 
  _emit 0x63 
  _emit 0x74 
  _emit 0x20 
  _emit 0x65 
  _emit 0x6E 
  _emit 0x64 
  _emit 0x00
}
```

> 将这一对标记插入到任何想保护的程序代码处就行了。 这种方法能够深入函数内部，精确的保护想要保护的某一段代码。在插入成功后，你可以在 VMProtect 加壳时选择内部函数的地方找到这种函数 `"VMProtectMarker1"`， 这就是使用了标记的地方，有多处保护的话这些函数数字会递增，如 `VMProtectMarker2`, `VMProtectMarker3` 等. 记得选上这些函数再进行保护。

### 2. ASProtect

> ASProtect 的 sdk 编程相对来说比较好学，因为它带了一个很详细的说明文件 `ASProtect.chm`。这里我相当于翻译了一下这个文件的一部分。我使用的 `ASProtect SKE 2.2 Release build 0425` 这个版本，安装后不但有说明文档，还有不少例子，其实看这些例子就能学会了，很简单.

#### 2.1 怎么使用 ASProtect 标记

> 最近版本的 ASProtect 支持在以下编程语言中的使用标记: Delphi, C/C++ 和 Visual Basic. 需要注意的是这些标记宏不能支持 .NET 语言和用 PCode 模式编译的 Visual Basic.

> 限制：

> 为了成功的将 ASProtect 标记插入到你的程序中，需要满足以下几个条件：

> § 多态变形标记(Polymorphic) 不能位于循环语句内，如 `for`, `do while` 等,

> § CRC 检查标记不能嵌套, 即不能在 CRC 标记内部又放一个 CRC 标记,

> § 位于 `registered sections`(注册区段) 和 CRC 检查标记内的代码至少需要 5 个字节大小.

> C/C++ 形式的标记是通过 `include(*.h or *.inc files)` 文件夹来定义的,并在编译后将会体现为一些特殊序列的汇编代码，这些汇编代码将会在加壳时被 ASProtect 自动侦测到.

> 你可以在 ASProtect 的安装文件夹内找到怎么使用标记的例子.

#### 2.2 多态变形标记(Polymorphic Markers)

> 可以用多态变形标记保护程序内任意位置的代码. 为了使用多态变形标记, 需要在你想保护的函数内部的任意位置插入一个变形标记实例。 例如：在函数内代码的顶部放置多态变形标记。 从这个标记到函数结尾的所有代码片断都将被抹去，并被替换为多态变形模拟体。 ASProtect 用仿真方式来改变这个函数的内容(如入口点保护), 所以将不可能恢复甚至于理解原来程序的工作原理。

> 为了使用新的代码片段保护，你需要插入新型的标记，如下所示：

```
#include "include\\aspr.h"  
 VOID Test    
 {   
 USER_POLYBUFFER  
 // some code  
 }  
```

> 注意! 为了避开那些在多态变形标记前改变程序逻辑的操作符，这个版本不支持在循环语句内使用标记，所以，请将这些标记放在循环体外面，不要使用类似下面的代码:

```
do {
  USER_POLYBUFFER 
  // some code  
} while();
  // some code  
```

#### 2.3 外壳完整性检查(Envelope Checks)

> ASProtect 直接将程序包裹在一个安全外壳内保护起来，这外壳包含所有的保护选项。所以检查这个外壳是否存在或者检查这外壳是否有被试图手动移除，对于保护程序非常重要。 为了使用外壳检查，需要插入两种外壳检查标记中的任一种，如下所示：

> 方式 1 - 如果 ASProtect 外壳被移除，它将会产生一个异常，你可以处理这个异常并在此时做一些邪恶的事情。 @-@

```
#include "include\\aspr.h"  
 MessageBox(0,"Begin","",0);  
#include "include\\CppEnvelopeCheck.inc"  //可以在任何位置插入这一句话.
 MessageBox(0,"End","",0);  
```

> 方式 2 - 如果 ASProtect 外壳被移除，这种方式将会像函数一样工作,返回一个 `false` 值。如果是 `false` 你可以做一些邪恶的事情。

```
BOOL EnvelopeCheck()   //这个函数放在程序前面就行了。
{  
#include "include\\CppEnvelopeCheckFunc.inc"    
}  

if ( !EnvelopeCheck() ) // .. 邪恶的事情    
```

#### 2.4 CRC 检查

> 这一章只应用于可执行程序。 代码区段的 CRC 检查对于对抗加载程序(`loaders`)非常有效。所以如果你想设定一些额外的代码片段的 CRC 检查，只需要在代码片段的开头与结尾放置 CRC 检查的标记，如下所示：

```
#include "include\\aspr.h"  

  #include "include\\cppCrcBegin.inc"  
  // some code  
  #include "include\\cppCrcEnd.inc"  
```

> 注意! 这个版本不支持嵌套的标记，所以不要使用下面的代码方式:

```
#include "include\\aspr.h"  

#include "include\\cppCrcBegin.inc"  
 // some code  
  #include "include\\cppCrcBegin.inc"  
 // some code  
  #include "include\\cppCrcEnd.inc"  
 // some code  
#include "include\\cppCrcEnd.inc"  
```

> 还有各种注册，过期时间的加密方式需要用到 ASProtect 的 API 方式，在那个帮助文档里面都有，想用的就看看吧，这部分我还没用到，所以没翻译了。以上的程序编译好后，用 ASProtect 加壳会自动识别标记，在加壳过程中可以看到提示。

> 以上的 SDK 加密方式掌握后，就可以将 ASProtect 与 VMProtect 随便乱混合加密了(先用 VMProtect,再用 ASProtect)，加密的强度应该蛮高的吧。 不过最好能让你的程序能运行。

> 值得注意的是有些标记完全由字节码组成，加壳时有可能某个标记被其他标记的保护破坏了就会出问题。 所以同一个代码片断不要用很多标记一起搞，这些都要试验的。

> 还有，有时候 SDK 编程出来的可执行文件不能直接运行，加过壳后才能运行。

> 补充：前面的 VMProtect 的 SDK 标志如果用得不多还好，用得多的话最好 `define` 一下，精简一点。 如下：

```
#define VMPBEGIN \
__asm _emit 0xEB \
__asm _emit 0x10 \
__asm _emit 0x56 \
__asm _emit 0x4D \
__asm _emit 0x50 \
__asm _emit 0x72 \
__asm _emit 0x6F \
__asm _emit 0x74 \
__asm _emit 0x65 \
__asm _emit 0x63 \
__asm _emit 0x74 \
__asm _emit 0x20 \
__asm _emit 0x62 \
__asm _emit 0x65 \
__asm _emit 0x67 \
__asm _emit 0x69 \
__asm _emit 0x6E \
__asm _emit 0x00

#define VMPEND \
  __asm _emit 0xEB \
   __asm _emit 0x0E \
   __asm _emit 0x56 \
   __asm _emit 0x4D \
   __asm _emit 0x50 \
   __asm _emit 0x72 \
   __asm _emit 0x6F \
   __asm _emit 0x74 \
   __asm _emit 0x65 \
   __asm _emit 0x63 \
   __asm _emit 0x74 \
   __asm _emit 0x20 \
   __asm _emit 0x65 \
   __asm _emit 0x6E \
   __asm _emit 0x64 \
   __asm _emit 0x00
```

> 有了上面的 `define`，在需要加密的地方只需要两句话就行了，可以到处多插一点:

```
VMPBEGIN
//需要加密的代码片断
VMPEND
```

`4dd466d0e1e0cc123afbe504da2462ca`
