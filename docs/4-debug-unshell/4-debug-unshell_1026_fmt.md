
# SVK Protector Help

> 原文：[`www.pediy.com/kssd/pediy06/pediy6868.htm`](https://www.pediy.com/kssd/pediy06/pediy6868.htm)
>
> 原文链接：[`bbs.pediy.com/showthread.php?s=&threadid=5490`](http://bbs.pediy.com/showthread.php?s=&threadid=5490)

译者: Cnbragon[TT]

说两句：昨晚一夜没睡，闲来没事翻译了这篇 SVK Protector 的帮助。希望能给兄弟们一些参考。各位就凑活着看吧![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg) ^_^

昨晚只翻译了一半，下半部分我就翻！本着对原作者的尊重，我在翻译的时候尽量理解作者的意思，然后用精确和各位能看的懂的的语言来翻译，但还有不少不足，欢迎兄弟们批评指证！ ^_^

## 译文

---------------------------------------------

AntingCrack 公司版权所有

**SVK Protector V1.43**

### 概述

*   关于 SVK Protector
*   SVK Protector 的主要特性
*   这些是为谁设计的?
*   SVK Protector 是怎样工作的?
*   系统要求
*   程序概述
    *   状态栏
    *   密钥保护
    *   特性
    *   时间限制
    *   特殊选项
*   怎样使用 SVK Protector
    *   用于已编译的应用程序
    *   使用 API 函数
    *   使用注册密钥
    *   使用硬件锁定密钥
*   用于不同类型的保护软件
*   用于 DLL 及 OCX 文件
*   用法举例
    *   对一个已编译应用程序的保护
    *   对一个用 API 函数的程序的保护
    *   对使用注册密钥程序的保护
    *   不同类型保护的组合
    *   用特征保护
    *   用时间限制保护
    *   硬件锁定注册密钥
    *   加密分组类型 1 和类型 2
    *   活动控件检查
    *   命令行特性
    *   用户注册密钥生成
    *   在 Visual Basic 中的标签
*   API
    *   API 描述
        *   `SVKP_GetRegistrationInformation`
        *   `SVKP_LockKeyboard`
        *   `SVKP_UnLockKeyboard`
        *   `SVKP_KillDebugger`
        *   `SVKP_RebootComputer`
        *   `SVKP_GetHWInfo`
*   购买与支持
*   授权协议
*   FAQ
    *   SVK Protector 问题
    *   关于 SVK Protector

### SVK Protector 是一个强大的工具

SVK Protector 是一个强大的工具，可以用来阻止非法复制，使用和散发。SVK Protector 以高强度的保护为最先，同时又设计的简单易用，速度快。

你可以即快又容易的为你的软件创建一个被 SVK Protector 保护的版本。这个被保护的版本可以被免费从互联网、CD-ROM 或者 DVD 上获得，而使用"先试用后购买"的保护措施。你可以给你的试用版本设置不同的限制，这意味着程序可以以试用版本的形式进行发布。一般来说，只有程序被授权了才可以解除限制。

如果使用试用版本的用户在条件过期后还没有购买，那么他就再以不能使用了，仅仅会有让他购买的信息出现。

SVK Protector 可以创建一个有使用限制的演示版本，使得以试用版本发布的程序的一些功能不能使用（例如保存，装载，装载工程）。更好的是只需要几分钟的时间就可以创建一个试用版本了。

在 SVK Protector 中初始化对话框可以被完全的编辑，这样就真的很简单的达到专业的设计而不需要额外的程序设计。

最大的优点是试用版本可以用注册密钥解锁。那些买了被保护的软件的授权的用户可以得到一个用来解锁的注册密钥。这些注册密钥是 2048 位长并且使用了 `RSA` 算法，保证不可能创建出错误的注册密钥。另外，可以创建仅为特定的电脑使用的注册密钥。换句话说，SVK Protector 是昂贵的硬件密钥保护的有效替代物。

对于软件的保护 SVK Protector 使用注册密钥-可扩展的软件授权体系。支持丰富的授权体系并且创建复杂授权体系来满足大部分的用户。授权保护系统也可以满足个人用户的需要。

对于安全扩展 SVK Protector 能让被保护的程序通过使用 `API` 函数和它的保护进行通信。这些函数可以被使用 SVK Protector 的开发人员根据自己的需要进行使用。

### SVK Protector 的主要特性

*   使用 2048 位的非对称加密算法
*   保护系统使用动态内存分配
*   用强劲的加密算法 `Rijndael` 来保护被保护的程序(译者注：`Rijndael`，即 `AES`，高级加密标准)
*   不可能为保护的程序创建出假的注册密钥
*   使用新的加密方法，在代码中叫做 `metamorphism`
*   解码部分每次都会改变-没有创建静态解码的可能性
*   对被保护的程序的加密部分采用 2048 位的加密算法。没有相应的密钥是不可能解开的。
*   使用了作者自己设计的新技术来检测是否被调试。具有检测和/或清除所有类型的可能断点-首先程序具备这个性能。
*   检测破解者使用的 "Ice Dump" 能力-首先保护系统具备这个性能
*   使用了新技术用来阻止调试器或反汇编工具的单步跟踪
*   在解码过程中使用了驱动
*   阻止了对被保护的程序的代码的修改并且即使程序在运行也可以保护它的完整性
*   高级保护措施用来防止内存转储(例如使用像 "ProcDump", "Adump" 的工具)
*   新的反调试技术用来对抗内核调试器 (`VC debugger`, `W32Dasm`...)。
*   允许对被保护的程序创建时间限制和功能限制的版本
*   对于被保护的使用汇编语言(小部分用 `C++`)编写的可以有更快的速度和响应时间并且可以使用特殊的程序设计技巧
*   压缩装载代码，被保护的文件仅仅增加大约 50K 的大小。
*   对应用程序进行压缩。
*   针对特定的计算机系统生成注册密钥
*   可以用于被保护的程序和保护程序进行交互的 `API` 函数
*   命令行特性
*   可以改变加密算法
*   使用授权密钥锁定程序的性能
*   删除 `EXE` 文件的 `.reloc` 区块的功能(译者注：`.reloc` 重定位区块)
*   移动入口点的代码的功能
*   对 SVK Protector 进行完整的编辑消息的功能
*   检查授权过期的功能
*   改变注册密钥名称的功能
*   `Stolen Keys`
*   可以对被保护的试用版本的 `Nag` 窗口进行完全的可编辑
*   可以为使用了锁定和解锁特性的特殊客户生成注册密钥匙。(译者注：原作者在此处写错了一个单词: `erating` => `generating`)
*   可以用来检测 SVK Protector 是否被清除的活动控件
*   可以创建在线注册服务的密钥生成能力，例如 `ShareIt!`

### 这些是为谁设计的?

世界范围的非法复制、使用和散发软件是当前受到关注的焦点。这就是为什么开发人员试图保护他们的软件并且阻止非法使用他们的智力成果，使收益更安全。

SVK Protector 适用于需要容易，快速和有效的保护他们产品的所有的公司，专业的软件开发人员 and 没有保护不被破解实际经验的人们。开发一个值得信赖和功能强大的保护软件不是那么简单的，它需要很多的知识和经验。这对我们来说是一个挑战，我们花了一年多的时间来设计 SVK Protector 的 1.0 版本。

我们把容易保护你的产品答为 SVK Protector 的基本的特性。所有的用户，包括有很少经验的，都可以几分钟就完成保护。尽管使用起来很容易，但是程序被高强度的安全所保护并且这种保护可以阻止 "软件海盗" 非法复制和传播你的作品。

SVK Protector 可以用来代替大部分的昂贵的硬件密钥保护系统，这些硬件密钥保护系统不能采纳，一方面是因为代价很高，很难接受，另一方面是它们必须和计算机组合。这样看来 SVK Protector 真的是一个 sought-after 工具，它可以解决所有已经出现的问题。

### SVK Protector 是怎样工作的?

被保护的程序被保护它的所谓的 "信封" 所覆盖。整个程序是用强大的对称加密算法 `Rijndael` 编码，这个加密算法将在很短的时间内成为美国的标准加密算法。这证明了它的高质量。

在特定的环境中 SVK Protector 使用了这些并且它使程序的特定部分(例如保存文件和装载文件的整个功能）被高强度的对称加密算法所加密。SVK Protector 首次利用 2048 位密钥，它是不能被攻破的。这些被加密的部分只有在有正解的注册密钥的情况下才可以正确解码，否则没有可能对它们进行解码。

在被保护的程序运行后它首先调用保护信封，它可以使被保护的程序更安全的运行。首先看用户有没有正解的注册密钥。如果答案是肯定的，应用程序会以全部功能运行。如果用户没有正解的注册密钥，SVK Protector 会检查用户的未注册版本的时间和功能限制(时间或者运行次数)。一个显示被保护程序的使用时间是否到期的对话框将显示出来。如果时间没有到期，会运行试用版本。然后整个程序被解码，但是那些需要正解注册密钥的功能还是加了密的。

保护仍然在程序运行的整个过程中都是激活的并且会避免破解者对程序的每一个攻击。

毫无疑问任何保护都会被攻破。仅仅是时间和使用的方法的问题。SVK Protector 在保护你的系统的同时也会使破解者无法忍受这个时间和破解方法。

### 系统要求

*   `CPU`: `Pentium 75 MHz` (min) / `Pentium 233 MHz` and more (推荐)
*   内存: `16 MB RAM` (min) / `256 MB RAM` (推荐)
*   显卡: `VGA` (min) / `SVGA` and better (推荐)
*   操作系统: `Windows 95`, `Windows 98`, `Windows ME`, `Windows NT4 Workstation`, `Windows 2000`, `Windows XP`

### 程序概述

#### 状态栏

*   `Product Name`(产品名称)：设置一个项目名称。这个名称必须唯一。
*   `Create backup copy of protected file`(创建被保护文件的备份)：如果你打开这项功能-SVK Protector 将给被保护的文件创建备份
*   `Version Number`(版本号)：为你的项目设置版本。
*   `File name to` (文件名)：设置要保护的文件名和路径。如果文件和你的项目文件在同一个文件夹中你只需要设置文件名就可以了(不需要路径)。如果你经常改变你的项目文件夹这个选项对你很有用。
*   `Button "Protect!!"`：开始保护。

#### 密钥保护

*   `Registry Information`(注册信息)：可以在里设置用户的信息
*   `Hardware ID`(硬件标识)：如果你想要使用硬件盥洗室注册密钥，在这里输入你从用户那里收到的硬件标识。
*   `Key Expiration Date`(密钥到期时间)：如果你选择了这个选项，你可以设置授权可以使用的到期时间。SVK Protector 检查设置的时间是否正确。格式是日期，月份和年 (`DD/MM/YYYY`).
*   `Name of registration key`(注册密钥文件名称): SVK Protector 的早期版本使用 `regkey.key` 作为授权文件名。利用这个功能你可以改变授权文件的名称。对于在同一个文件夹中多个被 SVK Protector 保护的文件应用程序的这个功能很有用。
*   `Directory for stolen registration keys`(被盗用的注册密钥目录)：当授权文件是用被盗用的信用卡注册的并且被传播到互联网上时，就可以用这个功能来保护了。如果程序不是用硬件注册密钥来保护的话，被盗用的密钥仍然可以用于程序的新的版本，直到生成新的工程文件。但是这样要发送新的注册密钥给用户。使用这个功能可以防止使用被盗用的密钥于新的版本，因此程序不能被这个授权激活。你需要指定被盗用密钥的记录的目录。授权文件名称无关紧要，但是不能把授权文件保存在这个文件夹的子文件夹。现在这个功能只能用于普通的授权文件，不能用于基于硬件保护的授权文件(因为基于硬件保护的授权文件使用这个功能没有什么意义)。

#### 特性

大量新的特性使你为程序的使用用户创建注册密钥，其中包含一些锁定和解除锁定的特性。这是一个容易但有效的区分客户的系统。这个特性的使用在下面的例子中解释：你有一个和图片在一起工作的程序。你想创建一个给普通用户使用的普通版本和一个可以编辑图片的专业版。如果你使用 SVK Protector 你不根本不需要创建两个单独的版本。你只需要在源代码中插入特殊的宏(和加密分组一样)，这个宏可以标记特性的开始和结束。然后你可以编译你的程序并用 SVK Protector 来保护它。这些特性只有在你生成一个注册密钥的时候才能解锁。被标记的块是加了密的，因此没有注册密钥它们就不会被解码。更多的信息你可以看使用特性保护，那里介绍了使用这个方法的例子。

*   `Check file for used features`(检查使用特性的文件): 在这个版本这个功能是锁定的。
*   `Check key`(检查密钥): 如果你为项目保存了注册密钥并单击了检查密钥，你会看到在特性菜单里被指定的密钥激活的特性。别忘了如果这个密钥是针对特定的硬件标识，在选择检查密钥之前你必须输入这个硬件标识。

> **说明：**
> 1. 不能递归调用特性块
> 2. 不能使用在包含块中使用 `try...finally`(针对 `Delphi`)
> 3. 不能在特性块中调用 `Exit` 或者 `Abort`

#### 时间限制

*(原文此处缺失具体描述，保留标题结构)*

#### 特殊选项

*(原文此处缺失具体描述，保留标题结构)*

### 怎样使用 SVK Protector

#### 用于已编译的应用程序

是 SVK Protector 最简单同时也是不安全的应用程序保护方法。整个过程只需要花几分钟。程序被信封覆盖，避免被破解者攻破对应用程序的保护。SVK Protector 保护和被保护的程序没有相联。

#### 使用 API 函数

这种保护方法需要改变被保护程序的源代码。SVK Protector 可以调用指定的 `API`，这将在后面讨论。这种保护的优点是如果破解者攻破了信封的保护他也不能得到完全(100%)的功能。很容易使用这些指定的 `API`。在应用程序的源代码中使用了 `API` 后程序需要编译并用 SVK Protector 保护。

> **说明**：没有被 SVK Protector 保护程序不能正确的启动。

#### 使用注册密钥

尽管这种方法需要在应用程序的源代码中做一些改变，但是整个实施的安全性却是最大程度的。除此之外用这种方法还可以为应用程序创建安全的演示和试用版本。

程序中的一部分源代码被专用字符所标记。如果没有正解的注册密钥就不可能使用这部分代码(例如 `LOAD` 和 `SAVE` 函数的处理过程)。这些代码会被加密并且只有在用户有正确的注册密钥的时候才可以解码。

在给将被加密的代码作了标记和执行专用的 `API` 函数后，应该先编译应用程序。然后再用 SVK Protector 来保护它。别忘了在 SVK Protector 中设置 "使用注册密钥" 这一项。SVK Protector 会为你生成私钥，这个私钥在工程配置文件中。

> **警告**：绝不能让其他人使用你的配置文件。它可能会被误用以生成错误的注册文件！
>
> 这个密钥作用后会生成名为 `regkey.key` 的授权文件。这个授权文件必须和被保护的程序放在同一个目录下。只有这样才可以注册程序，被加密的部分才能正确的解密。

> **警告**：如果程序没有先被 SVK Protector 所保护那么程序不能正解的启动。

#### 使用硬件注册密钥

SVK Protector 也提供给那些想使用为专门的计算机生成的专用密钥来注册的用户。这些密钥的优点是它们不能被移植到其它的计算机上的事实并且即使它们被散发到互联网上也对保护没有任何威协。在生成密钥之前，需要用户计算机的一些相关数据。

#### 用于不同类型的保护

使用 SVK Protector 并改变程序的源代码来保护程序是最好的保护方法。尽管它在所有方法中是最花费时间的，但是所有努力的结果是值得的。按照它的名称，它是不同保护类型的组合。它使用下列方法：`API` 函数、注册密钥和硬件保护。

#### 用于 DLL 及 OCX 文件

这个版本完全兼容 `DLL` 文件。现在它可以在 `DLL` 文件中使用以及所有类型的编码块 (`RSA`, 类型 1, 类型 2 和特性块)和 `SVKP Alive Checks`。

对于 `DLL` 文件的功保护有些规则必须遵守;否则被保护的程序不会发挥作用。

有两种使用被保护的 `DLL` 文件的方法:

1. 每个使用了被保护的 `.DLL` 的 `.EXE` 文件(或者 `.DLL` 文件)必须也要用 `SVKP` 来保护。

原文链接：[`bbs.pediy.com/showthread.php?s=&threadid=5674`](http://bbs.pediy.com/showthread.php?s=&threadid=5674)

终于更新完毕了！！从下午 1:00 一直到现在，7 个小时啊，累！！！还有 `SVKP` 的 `Order&Support` 及 `FAQ` 我就不译了，说句实在话，想必 `SVKP` 作者的英语也... ^_^

译者：Cnbragon[TT]

这是接着上面一部分的，还有一点没有译，我会不断的更新。

译的不准确的地方请各位兄弟指出。多谢！



## 2. 保护 EXE 文件与 DLL 调用

如果不能保护 `.EXE` 文件，那么也就不能用正常的方法来调用 `DLL` 中的函数。

使用 `LoadLibraryA` 或者 `LoadLibraryW` 来替换 `DLL`。使用 `GetProcAddress` 从被保护的程序中获得调用函数的地址。使用了被保护的 `.DLL` 文件的程序不能使用标准的方法来调用任何函数，因为这些函数不在输入表中。

被保护的程序不能正常的运行，因为被保护的 `.DLL` 文件的输出表被加密了，这样就没有正确的输出函数名了。

>>> **说明**：保护 `.OCX` 文件的功能还没有进行测试，因此我们不建议使用它来保护 `.ocx`。

### 用法举例

#### 对一个已编译应用程序的保护

创建一个用这个方法保护的应用程序根本不是问题。你所要做的只是启动 `SVK Protector` 填写 `"Product Name"`（产品名称）和 `"Version"`（版本）两个空就可以了。这些参数是必须的，但是最好把它们填上，举个例子如果你同时也使用了功能限制的话。最后你需要填写 `"File name to protect"`（要保护的文件名）并且按 `"Protect!!"`（保护!!）按钮或者从菜单中选择 `"Protect file"`（保护文件）。

如果一切都执行正确的话，在 `"Status Bar"`（状态栏）会显示出 `"File was successfully protected"`（文件已经被成功保护）的通知。现在你可以从菜单中选择 `"Test Protected file"`（测试保护的文件）来测试被保护的应用程序了。

虽然整个过程只需要几分钟，但是对抗破解者攻击的程序保护程度令人很满意。

#### 使用 API 函数来保护应用程序

`SVK Protector` 使用专门的 `API` 函数来和被保护程序的保护之间进行通信，这些 `API` 函数使用很简单，但是需要改变程序的源代码。`special.dll` 文件不需要和被保护的程序一起发布。这个文件只是用来测试的；应用程序不需要它，`SVK Protector` 可以把它的 `API` 函数转化为它的保护。

**Sample:**

**ASM:**

```asm
.586
.MMX
.model flat,stdcall

include ..\..\Inc&Lib\apimacro.mac
include ..\..\Inc&Lib.inc
INCLUDELIB      ..\..\Inc&Lib\iKERNEL32.lib
INCLUDELIB      ..\..\Inc&Lib\iUSER32.lib

;there are defined include file and library for SVKP's special functions
include         ..\..\Inc&Lib\special.inc
INCLUDELIB      ..\..\Inc&Lib\ispecial.lib

.DATA?
registration_information db 255 dup (?)         ;buffer for registration information

.DATA
capture db "API functions example.",0
message db "Now I will try some API functions",0
registered db "Registration data from key:",0
unregistered db "Unregistered version!",0
unregistered_user db "Please register to www...",0

.code
assume          fs      :flat

Start:
;int 3                              ;only for debug version
        push 0
        push offset capture
        push offset message
        push 0
        iWin32 MessageBoxA
;now we will check for debuggers
        iWin32 SVKP_KillDebugger
;then we will lock keyboard
        iWin32 SVKP_LockKeyboard
;now we will read registration information
        iWin32 SVKP_GetRegistrationInformation, offset registration_information
        test eax, eax
        jz   jmp_unregistered_user          ;if there is not registration key, jump
        push 0
        push offset registered
        push offset registration_information
        push 0
        iWin32 MessageBoxA
        jmp  back_to_win

jmp_unregistered_user:
;if you want to do special action for unregistered version, You can do it there.
        push 0
        push offset unregistered
        push offset unregistered_user
        push 0
        iWin32 MessageBoxA
;end of part for non registered users
skip_unregistered_part:

back_to_win:
;now we will unlock keyboard
        iWin32 SVKP_UnLockKeyboard
;end of program
        push -1
        iWin32 ExitProcess
end Start
```

**VC:**

```c
#include <windows.h>
#include <io.h>
#include "..\..\Inc&Lib\special.h"

#pragma comment (lib,"SPECIAL")

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,
                   PSTR szCmdLine, int iCmdShow)
{
  char *message = "SVKP's API functions example!";
  char* registration_information=(char*)malloc(255*sizeof(char));
  int flag;
  MessageBox(0,message,"",0);
  SVKP_KillDebugger();    // now we will check for debuggers
  SVKP_LockKeyboard();    // then we will lock keyboard
// if was registered flag=1
  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {
// This is part for unregistered users
    message = "Unregistered version!";
    MessageBox(0,message,"",0);
  }
  else {
    message = "Registration data from key:";
    MessageBox(0,registration_information,message,0);
  }
  SVKP_UnLockKeyboard();    // now we will unlock keyboard
  return 0;
}
```

**Delphi:**

```delphi
unit main;
interface
procedure Start;
implementation
uses
  Registry,
  Classes,
  Windows,
  SysUtils,
  special;
var
  RegMessage : String;
  registration_information : String;
procedure Start;
begin
  try
    registration_information := '';
  SVKP_KillDebugger;          // now we will check for debuggers
    SVKP_LockKeyboard;          // then we will lock keyboard
    if SVKP_GetRegistrationInformation(registration_information)= 0 then
    begin
    // This is part for unregistered users
    RegMessage := 'Unregistered version!';
    MessageBox(0,PChar (RegMessage),'',0)
    end
    else
    begin
    // This is part for registered users
    RegMessage := 'Registration data from key:';
    MessageBox(0,PChar (registration_information),PChar (RegMessage),0);
    end;
    SVKP_UnLockKeyboard();    // now we will unlock keyboard
  except
  end;
end;
end.
```

**VB:**

```vb
Attribute VB_Name = "Module1"
Option Explicit
'There we declare SVKP's special functions'
 Public Declare Function SVKP_GetRegistrationInformation Lib "SPECIAL.DLL" _
    (ByVal reginfoptr As String) As Long
Public Declare Function SVKP_LockKeyboard Lib "SPECIAL.DLL" _
    () As Long
Public Declare Function SVKP_UnLockKeyboard Lib "SPECIAL.DLL" _
    () As Long
Public Declare Function SVKP_KillDebugger Lib "SPECIAL.DLL" _
    () As Long

Sub Main()
Dim RegistrationInformation As String
Dim Flag As Boolean
'first we will lock keyboard'
SVKP_LockKeyboard
'now we will check for debuggers'
SVKP_KillDebugger
RegistrationInformation = String$(255, 0)
Flag = False
Flag = SVKP_GetRegistrationInformation(RegistrationInformation)
If Flag = True Then
MsgBox RegistrationInformation, vbInformation, "Registered Action!"
Else
MsgBox "Unregistered user!", vbInformation, "UnRegistered Action!"
End If
SVKP_UnLockKeyboard
End Sub
```

**说明**：参见 `"Examples/SVKP_api_functions"` 目录，那里有使用这个方法的例子。

### 不同类型保护的组合

这种保护方法一次组合使用了几种类型的保护。一个被保护的应用程序例如既可以使用 `API` 函数也可以使用注册密钥等等的方法来保护。

**Sample:**

**ASM:**

```asm
.586
.MMX
.model flat,stdcall
include ..\..\Inc&Lib\apimacro.mac
include ..\..\Inc&Lib\windows.inc
INCLUDELIB      ..\..\Inc&Lib.lib
INCLUDELIB      ..\..\Inc&Lib.lib
;there are defined include file and library for SVKP's special functions
include         ..\..\Inc&Lib.inc
INCLUDELIB      ..\..\Inc&Lib\ispecial.lib

.DATA?
registration_information db 255 dup (?)         ;buffer for registration information

.DATA
capture db "Encryption blocks example.",0
message db "Now I will try run encrypted block!",0
special_capture2 db "Message for unregistered users!",0
special_message2 db "You can buy our product at www....",0

.code
assume          fs      :flat

Start:
;int 3                              ;only for debug version
        push 0
        push offset capture
        push offset message
        push 0
        iWin32 MessageBoxA

SVKP_ENCRYPTED_BLOCK_START              ;start of encrypted block
;this code will encrypted and it will decrypted only if user have correct registration key
        push 0
        push offset special_capture
        push offset special_message
        push 0
        iWin32 MessageBoxA              ;display message box for registered users
        jmp  skip_text

special_capture db "Message for registered users!",0
special_message db "This message box will show only if you have registered version!",0

skip_text:

SVKP_ENCRYPTED_BLOCK_END                ;end of encrypted block
;now we will read registration information
        iWin32 SVKP_GetRegistrationInformation, offset registration_information
```



```asm
        test eax, eax  

        jnz  skip_unregistered_part               ;if there is registration key, jump  

;if you want to do special action for unregistered version, You can do it there.  

        push 0  

        push offset special_capture2  

        push offset special_message2  

        push 0  

        iWin32 MessageBoxA  

;end of part for non registered users  

skip_unregistered_part:  

        push -1  

        iWin32 ExitProcess  

end Start
```

**VC:**

```c
#include <windows.h>  

#include <io.h>  

#include "...&Lib.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  

                    PSTR szCmdLine, int iCmdShow)  

{  

  char *message = "Encryption blocks example!";  

  char* registration_information=(char*)malloc(255*sizeof(char));  

  int flag;  

  MessageBox(0,message,"",0);  

  SVKP_ENCRYPTED_BLOCK_START    //Start of encrypted block  

// This code will encrypted and it will decrypted only if user have correct registration key  

    message = "Registered version!";  

    MessageBox(0,message,"",0);    // display message box for registered users  

  SVKP_ENCRYPTED_BLOCK_END      // End of encrypted block   

// if was registered flag=1   

  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  

// This is part for unregistered users     

    message = "Unregistered version!";     

    MessageBox(0,message,"",0);  

  }  

  return 0;  

}
```

**Delphi:**

```pascal
unit main;  

interface  

procedure Start;  

implementation  

uses  

  Registry,  

  Classes,  

  Windows,  

  SysUtils,  

  special;  

var  

  RegMessage : String;  

// This procedure will encrypted and it will decrypted only if user have correct registration key  

procedure ActionForRegisteredUsers;  

begin  

    {$I include\delphi_svkp_block_begin.inc}  

    RegMessage := 'Registered version!';  

    MessageBox(0,PChar (RegMessage),'',0);  

    {$I include\delphi_svkp_block_end.inc}  

end;  

procedure Start;  

begin  

  try  

    RegMessage := 'Encryption blocks example';  

    MessageBox(0,PChar (RegMessage),'',0);  

    ActionForRegisteredUsers; // this procedure is only for registered users  

  except  

  end;  

end;  

end.
```

**警告:** 不要在 `VB` 中使用这个方法!

**说明:** 在 `SVKP` 的 `Examples/Encrypted_blocks_RSA` 目录下有使用这个方法的例子.

### 用特征保护

    在选择这个选项之前需要先在菜单中选择 "Use registration keys(使用注册密钥)",因为这个函数是建立在注册密钥使用的基础之上.在选择了这个选项后加密密钥就可用于单独的特性了.别忘了这个函数的使用需要对被保护程序的源代码做一些调整.

    可以在程序中设置 32 个保护特性.对于每个特性都可以选择多个代码块,这些代码块被标记为 `SVKP_ENCRYPTED_BLOCK_TYPEX_START` 和 `SVKP_ENCRYPTED_BLOCK_TYPEX_END` ,X 代表特性的号码.举例说明对于特性 1#可以有 100 个代码块,这些代码块只能在注册文件中进行解锁.这看起来很复杂，但是使用这个函数相当容易(请看例子).

    因此,标记单独特性的开始和结束并且编译代码足够了.在 `SVK Protector` 中选择 "Use features for license keys(给授权密钥使用特性)" 选项并且按下按钮 "Protect!!",这些可以保护这个应用程序.

    当你为你的客户生成注册密钥的时候,选择特性,将在特性菜单中解锁(例如我们选择特性 1 和 3)选择的特性将解锁但其它的特性将被锁定。这使得处理消费者的区别相当容易.

    **例子：**

**ASM:**

```asm
        .586  
        .MMX  
        .model flat,stdcall  
include ..\..\Inc&Lib\apimacro.mac  
include ..\..\Inc&Lib\windows.inc  

INCLUDELIB      ..\..\Inc&Lib.lib  

INCLUDELIB      ..\..\Inc&Lib\iUSER32.lib  

;there are defined include file and library for SVKP's special functions  

include         ..\..\Inc&Lib\special.inc  

INCLUDELIB      ..\..\Inc&Lib\ispecial.lib  

.DATA?  

registration_information db 255 dup (?)         ;buffer for registration information  

.DATA  

capture db "Features example.",0  

message db "Now I will try run some features!",0  

capture_feature1 db "Feature 1",0  

message_feature1 db "Feature 1 is not enabled in license!",0  

special_capture2 db "Message for unregistered users!",0  

special_message2 db "You can buy our product at [www....](http://www....)",0  

feature1_unlocked_flag db 0  

.code  

assume          fs      :flat  

Start:  

;int 3                                   ;only for debug version  

        push 0  

        push offset capture  

        push offset message  

        push 0  

        iWin32 MessageBoxA  

SVKP_ENCRYPTED_FEATURE1_START                 ;start of feature 1  

;this code will encrypted and it will decrypted only if user have correct registration key with this feature unlocked  

        push 0  

        push offset special_capture1  

        push offset special_message1  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text  

special_capture1 db "Message for registered users!",0  

special_message1 db "Feature 1 is unlocked in license!",0  

skip_text:  

        mov  [feature1_unlocked_flag], 1        ;set flag, because feature is unlocked in license  

SVKP_ENCRYPTED_FEATURE1_END                 ;end of feature 1  

        cmp  [feature1_unlocked_flag], 1  

        jz   skip_unregistered_feature1  

;display text for users with feature 1 not unlocked  

        push 0  

        push offset capture_feature1  

        push offset message_feature1  

        push 0  

        iWin32 MessageBoxA  

skip_unregistered_feature1:  

;now we will read registration information  

        iWin32 SVKP_GetRegistrationInformation, offset registration_information  

        test eax, eax  

        jnz  skip_unregistered_part               ;if there is registration key, jump  

;if you want to do special action for unregistered version, You can do it there.  

        push 0  

        push offset special_capture2  

        push offset special_message2  

        push 0  

        iWin32 MessageBoxA  

;end of part for non registered users  

skip_unregistered_part:  

;action for users with Feature 2 activated  

SVKP_ENCRYPTED_FEATURE2_START  

        push 0  

        push offset special_capture3  

        push offset special_message3  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text2  

special_capture3 db "Message for registered users!",0  

special_message3 db "Feature 2 is unlocked in license!",0  

skip_text2:  

SVKP_ENCRYPTED_FEATURE2_END  

;is possible select more than one part of code for one feature  

SVKP_ENCRYPTED_FEATURE1_START  

        push 0  

        push offset special_capture4  

        push offset special_message4  

        push 0  

        iWin32 MessageBoxA                  ;display message box for registered users  

        jmp  skip_text3  

special_capture4 db "Message for registered users!",0  

special_message4 db "Feature 1 second part of code!",0  

skip_text3:  

SVKP_ENCRYPTED_FEATURE1_END  

        push -1  

        iWin32 ExitProcess  

end Start
```

**VC:**

```c
#include <windows.h>  

#include <io.h>  

#include "..\\..\\Inc&Lib\\special.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  

                    PSTR szCmdLine, int iCmdShow)  

{  

  char *message = "Features example!";  

  char* registration_information=(char*)malloc(255*sizeof(char));  

  int flag;  

  MessageBox(0,message,"Now I will try run some features!",0);  

  SVKP_ENCRYPTED_FEATURE1_START     // Start of feature 1  

// this code will encrypted and it will decrypted only if user have correct registration key with this feature unlocked  

    MessageBox(0,"Feature 1 is unlocked in license!","Message for registered users!",0);    // display message box for registered users  

  SVKP_ENCRYPTED_FEATURE1_END         // end of feature 1  

// if was registered flag=1   

  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  

// This is part for unregistered users     

    message = "Unregistered version!";     

    MessageBox(0,message,"",0);  

  }  

// action for users with Feature 2 activated  

  SVKP_ENCRYPTED_FEATURE2_START  
```



```
    MessageBox(0,"Feature 2 is unlocked in license!","Message for registered users!",0);    // display message box for registered users  

  SVKP_ENCRYPTED_FEATURE2_END  

// is possible select more than one part of code for one feature  

  SVKP_ENCRYPTED_FEATURE1_START  

    MessageBox(0,"Feature 1 second part of code!","Message for registered users!",0);    //display message box for registered users  

  SVKP_ENCRYPTED_FEATURE1_END  

  return 0;  

}  
```

### Delphi

```
unit main;  

interface  

procedure Start;  

implementation  

uses  

  Registry,  

  Classes,  

  Windows,  

  SysUtils,  

  special;  

var  

  RegMessage : String;  

procedure ActionForRegisteredUsers;  

begin  

// this code will encrypted and it will decrypted only if user have correct registration key with this feature unlocked  

    {$I include\delphi\_svkp_block_feature1_begin.inc}  

    RegMessage := 'Feature 1 is unlocked in license!';  

    MessageBox(0,PChar (RegMessage),'Message for registered users!',0);  

    {$I include\delphi\_svkp_block_feature1_end.inc}  

// action for users with Feature 2 activated  

    $I include_svkp_block_feature2_begin.inc  

    RegMessage := 'Feature 2 is unlocked in license!';  

    MessageBox(0,PChar (RegMessage),'Message for registered users!',0);  

    {$I include\delphi\_svkp_block_feature2_end.inc}  

// is possible select more than one part of code for one feature  

    $I include_svkp_block_feature1_begin.inc  

    RegMessage := 'Feature 1 second part of code!';  

    MessageBox(0,PChar (RegMessage),'Message for registered users!',0);  

    {$I include\delphi\_svkp_block_feature1_end.inc}  

end;  

procedure Start;  

begin  

  try  

    RegMessage := 'Encryption blocks example';  

    MessageBox(0,PChar (RegMessage),'',0);  

    ActionForRegisteredUsers; //this procedure is only for registered users  

  except  

  end;  

end;  

end.  
```

### 警告

在`Examples/Features/VB`目录下可以找到 Visual Basic 的例子!

### 说明

在 `SVKP` 的`Examples/Features`目录下有使用这个方法的例子.

### 用时间限制保护

使用 `SVK Protector` 可以快速和简单的为被保护的程序创建一个时间限制的试用版本.只要选上`TimeLock`选项中的`Limited functional time of unregistered version of protected program`.然后选择时间限制的类型`执行次数`或者`天数`.当然两个都可以选上.`Number of executions`选项是说未注册的版本可以使用多少次.`Number of days`选项是说未注册的版本可以使用多少天.

除此这外你也可以使用`Reminder message`,是说被保护的程序在每次启动的时候都会显示`This is a shareware version.Please register your copy.`.

如果用户购买了被保护的程序并得到了正确的注册密钥，时间限制将被解除.

### 硬件锁定注册密钥

可以为单个计算机生成特殊的注册密钥。为了这个目的在`Key protection`选项中有`Hardware Id`.

首先选中使用硬件 ID 选项并且使用 `SVK Protector` 来对被保护的程序加密.然后需要生成特殊的密钥.过程很相似.和普通的注册密钥一样填充`Registration information`.别忘了填充`Hardware ID`.这个每个单一的计算机都不相同的数字可以在 API 函数 `SVKP_GETHWInfo` 中获得.你的被保护的程序可以在注册对话框或者关于对话框中显示这个数字.这仅取决于你.用户把这个数字发给你,然后你用它来生成注册密钥.这个密钥只可以在这一台机子上使用.

尽管`Check key`使用硬件 ID 的注册密钥,需要为特定的计算机输入正确的`硬件 ID`.否则不可能检查这个密钥并且 `SVK Protector` 显示密钥错误.

提示:在`Utilities/Display_HW_info`目录下有一个显示硬件 ID 并且保存到剪切版的程序.

说明:硬件 ID 不能从密钥中得到，这也是为什么需要把每个密钥分开保存的原因.

### Sample

#### ASM

```
        .586  
        .MMX  
        .model flat,stdcall  
include ..\..\Inc&Lib\apimacro.mac  
include ..\..\Inc&Lib\windows.inc  
INCLUDELIB      ..\..\Inc&Lib\iKERNEL32.lib  
INCLUDELIB      ..\..\Inc&Lib\iUSER32.lib  
;there are defined include file and library for SVKP's special functions  
include         ..\..\Inc&Lib\special.inc  
INCLUDELIB      ..\..\Inc&Lib\ispecial.lib  
.DATA?  
registration_information db 255 dup (?)         ;buffer for registration information  
HWInformation    dd 4 dup (?)  
.DATA  
special_message2 db "You can buy our product at [www....](http://www....)",0dh,0dh  
                 db "Please send us your Hardware ID:", 0dh, 0dh  
hardware_key     db 32 dup (?)  
hardware_key_end db 0dh,0dh,0dh  
                 db "Your Hardware ID was copied to clipboard, please paste it to e-mail!",0dh  
                 db "Now try to create registration key for this project and Hardware ID in SVKP.",0  
capture db "Encryption blocks example.",0  
message db "Now I will try run encrypted block!",0  
special_capture2 db "Message for unregistered users!",0  
allocated_memory dd 0  
locked_mem       dd 0  
.code  
assume          fs      :flat  
Start:  
;int 3                                                 ;only for debug version  
        push 0  
        push offset capture  
        push offset message  
        push 0  
        iWin32 MessageBoxA  
SVKP_ENCRYPTED_BLOCK_START                       ;start of encrypted block  
;this code will encrypted and it will decrypted only if user have correct registration key  
        push 0  
        push offset special_capture  
        push offset special_message  
        push 0  
        iWin32 MessageBoxA                       ;display message box for registered users  
        jmp  skip_text  
special_capture db "Message for registered users!",0  
special_message db "This message box will show only if you have registered version!",0  
skip_text:  
SVKP_ENCRYPTED_BLOCK_END                         ;end of encrypted block  
;now we will read registration information  
        iWin32 SVKP_GetRegistrationInformation, offset registration_information  
        test eax, eax  
        jnz  skip_unregistered_part              ;if there is registration key, jump  
;if you want to do special action for unregistered version, You can do it there.  
        lea  esi, HWInformation  
        iWin32 SVKP_GetHWInfo, esi  
;        mov  esi, edi  
        lea  edi, hardware_key  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32  
        push 0  
        push offset special_capture2  
        push offset special_message2  
        push 0  
        iWin32 MessageBoxA  
        mov  byte ptr [hardware_key_end], 0  
        iWin32 GlobalAlloc, 2002h, 32h  
        mov  [allocated_memory], eax  
        iWin32 GlobalLock, eax  
        mov  [locked_mem], eax  
        iWin32 lstrcpy, eax, offset hardware_key  
        iWin32 GlobalUnlock, dword ptr [allocated_memory]  
        iWin32 OpenClipboard, NULL  
        iWin32 EmptyClipboard  
        iWin32 SetClipboardData, 1, dword ptr [allocated_memory]  
        iWin32 CloseClipboard  
;end of part for non registered users  
skip_unregistered_part:  
        push -1  
        iWin32 ExitProcess  
HexWrite8  proc  
        mov  ah, al  
        and  al, 0fh  
        shr  ah, 4  
        or   ax, 3030h  
        xchg al, ah  
        cmp  ah, 39h  
        ja   @@4  
@@1:  
        cmp  al, 39h  
        ja   @@3  
@@2:  
        stosw  
        ret  
@@3:  
        sub  al, 30h  
        add  al,'A' - 10  
        jmp  @@2  
@@4:  
        sub  ah, 30h  
        add  ah, 'A' - 10  
        jmp  @@1  
HexWrite8      endp  
HexWrite16  proc  
        push ax  
        xchg al, ah  
        call HexWrite8  
        pop  ax  
        call HexWrite8  
        ret  
HexWrite16  endp  
HexWrite32  proc  
        push eax  
        shr  eax, 16  
        call HexWrite16  
        pop  eax  
        call HexWrite16  
        ret  
HexWrite32  endp  
end Start  
```

#### VC

```
#include <windows.h>  

#include <io.h>  

#include "..\..\Inc&Lib\special.h"  

#pragma comment (lib,"SPECIAL")  

void TraslateToASCII(char* hardware_key, char* HWInformation)  

{  

  __asm{  
        mov  esi, HWInformation  
    mov  edi, hardware_key  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32  
        lodsd  
        call HexWrite32     
    
    jmp  endik  

HexWrite32:     
    
        push eax  
        shr  eax, 16  
        call HexWrite16  
        pop  eax  
```



## Assembly Code

```asm
    call HexWrite16  
    ret  

HexWrite16:  
    push ax  
    xchg al, ah  
    call HexWrite8  
    pop  ax  
    call HexWrite8  
    ret  

HexWrite8:  
    mov  ah, al  
    and  al, 0fh  
    shr  ah, 4  
    or   ax, 3030h  
    xchg al, ah  
    cmp  ah, 39h  
    ja   a4  

a1:  
    cmp  al, 39h  
    ja   a3  

a2:  
    stosw  
    ret  

a3:  
    sub  al, 30h  
    add  al,'A' - 10  
    jmp  a2  

a4:  
    sub  ah, 30h  
    add  ah, 'A' - 10  
    jmp  a1  
```

## C/C++ Code

```c
endik:  
}  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  
                    PSTR szCmdLine, int iCmdShow)  
{  
  char *message = "Encryption blocks example!";  
  char* registration_information=(char*)malloc(255*sizeof(char));  
  char* HWInformation=(char*)malloc(20*sizeof(char));  
  int flag;  
  char* unregister_message=(char*)malloc(255*sizeof(char));  
  char* hardware_key=(char*)malloc(35*sizeof(char));  

  char *first_message="You can buy our product at [www....](http://www....)";  
  char *second_message="Please send us your Hardware ID:";  
  char *third_message="Your Hardware ID was copied to clipboard, please paste it to e-mail";  
  char *fourth_message="Now try to create registration key for this project and Hardware ID in SVKP.";  

  LPVOID allocated_memory;  
  LPVOID locked_mem;  

  MessageBox(0, message, "", 0);  

  SVKP_ENCRYPTED_BLOCK_START    // Start of encrypted block  
  // This code will be encrypted and it will be decrypted only if user has correct registration key  
    message = "Registered version!";  
    MessageBox(0, message, "", 0);    // display message box for registered users  
  SVKP_ENCRYPTED_BLOCK_END      // End of encrypted block   

  // If registered, flag=1  
  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  
    // This is part for unregistered users  
    message = "Unregistered version!";  

    // Get Hardware ID  
    SVKP_GetHWInfo(HWInformation);  

    // Prepare message for display  
    TraslateToASCII(hardware_key, HWInformation);  
    TraslateToASCII(hardware_key+8, HWInformation+4);  
    TraslateToASCII(hardware_key+16, HWInformation+8);  
    TraslateToASCII(hardware_key+24, HWInformation+12);  

    hardware_key[32]=0x0d;  
    hardware_key[33]=0x0d;  
    hardware_key[34]=0x00;  

    unregister_message=first_message;  
    lstrcat(unregister_message, second_message);  
    lstrcat(unregister_message, hardware_key);  
    lstrcat(unregister_message, third_message);  
    lstrcat(unregister_message, fourth_message);  

    // Display message  
    MessageBox(0, unregister_message, message, 0);  

    // Copy Hardware ID to clipboard  
    hardware_key[32]=0x00;  
    allocated_memory=GlobalAlloc(0x2002, 0x32);  
    locked_mem=GlobalLock(allocated_memory);  
    lstrcpy((char*)locked_mem, hardware_key);  
    GlobalUnlock(allocated_memory);  

    OpenClipboard(NULL);  
    EmptyClipboard();  
    SetClipboardData(1, allocated_memory);  
    CloseClipboard();  
  }  
  return 0;  
}  
```

## Delphi Code

```pascal
function TraslateToHex(Buf: array of char; sz: integer): string;  
var  
  I, D, M: integer;  
begin  
  result:='';  
  M:=0;  
  for I:=0 to sz-1 do  
  begin  
    M:=I*4;  
    for D:=3 downto 0 do  
      result:=result+IntToHex(ord(Buf[D+M]),2);  
  end;  
end;  

procedure CopyToClipboard(const s: string);  
var  
  m: dword;  
begin  
  m:=GlobalAlloc(GMEM_MOVEABLE or GMEM_DDESHARE, length(s)+1);  
  lstrcpy(GlobalLock(m), pchar(s));  
  GlobalUnlock(m);  
  OpenClipboard(0);  
  EmptyClipboard;  
  SetClipboardData(cf_text, m);  
  CloseClipboard;  
end;  

procedure Test;  
var  
  hardware_key, unregistered_msg: string;  
  registration_info, HWInformation: array[0..255] of char;  
begin  
  messagebox(0, 'Encryption blocks example!', '', 0);  
  {$I .\include\delphi_svkp_block_begin.inc}  // Start of encrypted block  
  // Code inside this block will be decrypted only for users with valid registration key  
  messagebox(0, 'Registered version!', '', 0);   // a message for registered users  
  // Add here an useful code available only for registered users...  
  {$I .\include\delphi_svkp_block_end.inc}    // End of encrypted block  

  if SVKP_GetRegistrationInformation(registration_info)<>1 then begin  
    // This is part for unregistered users  
    SVKP_GetHWInfo(HWInformation); // Get Hardware ID  
    hardware_key:=TraslateToHex(HWInformation, 4);  
    unregistered_msg:=  
      'You can buy our product at [www...](http://www...)'#10#10+  
      'Please send us your Hardware ID:'#10+hardware_key+#10#10+  
      'Your Hardware ID was copied to clipboard, please paste it to e-mail'#10#10+  
      'Now try to create registration key for this project and Hardware ID in SVKP.';  
    messagebox(0, pchar(unregistered_msg), 'Unregistered version!', 0);  
    CopyToClipboard(hardware_key); // copy Hardware ID to clipboard  
  end;  
end;  

begin  
  Test;  
end.  
```

## 说明

在 `SVKP` 的 `Examples/HW_locked` 目录下有使用这个方法的例子。

## 分组加密类型 1 和类型 2

这些分组和 `RSA` 分组很相似。也是加密程序中的一部分，但是不像 `RSA` 分组，它们不依赖于密钥。

意思是说不管是注册的还是未注册的版本，都是在使用这部分代码的时候才解码。看起来这个函数不吸引人，但事实不是如此。现在保护程序不被内存转储很困难。确切地说，这个函数可以避免对程序的内存转储。

### 分组加密类型 1

第一种分组类型在使用之前开始解码，并且在使用完之后再进行加密。整个解码和加密的过程很快，因此很容易解决经常使用的函数。推荐在关键代码处使用这个类型，比如检测注册或者需要保护的代码处（例如所知道如何做的地方）。

### 分组加密类型 2

第二种分组类型在使用之前开始解码，并在使用之后从内存中删除。这也是为什么要仔细地选择需要保护的代码的原因。必须意识到这个分组只能启动一次。推荐在代码的初始化部分中使用这种分组类型。这样转储程序就不会起作用了。

**警告:**

1. 不要递归调用被加密的分组。
2. 不要使用 `try...finally` 来包含分组...(Delphi)
3. 不要在加密分组中调用 `Exit` 或者 `Abort`

### Sample: ASM

```asm
.586  
.MMX  
.model flat, stdcall  

include ..\..\Inc&Lib\apimacro.mac  
include ..\..\Inc&Lib\windows.inc  
INCLUDELIB     ..\..\Inc&Lib\iKERNEL32.lib  
INCLUDELIB     ..\..\Inc&Lib\iUSER32.lib  

; There are defined include file and library for SVKP's special functions  
include        ..\..\Inc&Lib\special.inc  
INCLUDELIB     ..\..\Inc&Lib\special.lib  

.DATA?  
registration_information db 255 dup (?)         ; buffer for registration information  

.DATA  
capture db "Encryption blocks example.",0  
message db "Now I will try run encrypted block!",0  
special_capture2 db "Message for unregistered users!",0  
special_message2 db "You can buy our product at www....",0  

.code  
assume         fs      : flat  

Start:  
; int 3                          ; only for debug version  
; This code will be decrypted before using and after use removed  
; Tip: This type of blocks is very good use in initialize part of program  
; (load DLL(s), initialize data, fill some special variables and etc.)  

SVKP_ENCRYPTED_BLOCK_TYPE2_START  
    push 0  
    push offset capture  
    push offset message  
    push 0  
    iWin32 MessageBoxA  
SVKP_ENCRYPTED_BLOCK_TYPE2_END  

SVKP_ENCRYPTED_BLOCK_START                 ; start of encrypted block RSA  
; This code will be encrypted and it will be decrypted only if user has correct registration key  
    push 0  
    push offset special_capture  
    push offset special_message  
    push 0  
    iWin32 MessageBoxA                    ; display message box for registered users  
    jmp  skip_text  

special_capture db "Message for registered users!",0  
special_message db "This message box will show only if you have registered version!",0  

skip_text:  

SVKP_ENCRYPTED_BLOCK_END                  ; end of encrypted block  

; This code will be decrypted before using and encrypted again after use  
SVKP_ENCRYPTED_BLOCK_TYPE1_START  
```



# 加密块 (Encrypted Blocks)

## 汇编示例

;now we will read registration information  
`iWin32 SVKP_GetRegistrationInformation, offset registration_information`  
`test eax, eax`  
`jnz skip_unregistered_part` ;if there is registration key, jump  

;if you want to do special action for unregistered version, You can do it there.  
`push 0`  
`push offset special_capture2`  
`push offset special_message2`  
`push 0`  
`iWin32 MessageBoxA`  

;end of part for non registered users  

`skip_unregistered_part:`  
`SVKP_ENCRYPTED_BLOCK_TYPE1_END`  
`push -1`  
`iWin32 ExitProcess`  
`end Start`

## C/C++ 示例

```c
#include <windows.h>  
#include <io.h>  
#include "..\\..\\Inc&Lib\\special.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  
                    PSTR szCmdLine, int iCmdShow)  
{  
  char *message = "Encryption blocks example!";  
  char* registration_information=(char*)malloc(255*sizeof(char));  
  int flag;  

// this code will decrypted before using and after use removed  
// tip: This type of blocks is very good use in initialize part of program  
// (load DLL(s), initialize data, fill some special variables and etc.)   

  SVKP_ENCRYPTED_BLOCK_TYPE2_START  
  MessageBox(0,message,"",0);  
  SVKP_ENCRYPTED_BLOCK_TYPE2_END  

  SVKP_ENCRYPTED_BLOCK_START    // Start of encrypted block  

// This code will encrypted and it will decrypted only if user have correct registration key  

    message = "Registered version!";  
    MessageBox(0,message,"",0);    // display message box for registered users  

  SVKP_ENCRYPTED_BLOCK_END      // End of encrypted block   

// This code will decrypted before using and encrypted again after use  

  SVKP_ENCRYPTED_BLOCK_TYPE1_START  

// if was registered flag=1   

  if(flag = SVKP_GetRegistrationInformation(registration_information)==0) {  
    // This is part for unregistered users     
    message = "Unregistered version!";     
    MessageBox(0,message,"",0);  
  }  

  SVKP_ENCRYPTED_BLOCK_TYPE1_END  
  return 0;  
}  
```

## Delphi 示例

```pascal
unit main;  

interface  

procedure Start;  

implementation  

uses  
  Registry,  
  Classes,  
  Windows,  
  SysUtils,  
  special;  

var  
  RegMessage : String;  

// This procedure will encrypted and it will decrypted only if user have correct registration key  

procedure ActionForRegisteredUsers;  

begin  
    {$I include\\delphi_svkp_block_begin.inc}  
    RegMessage := 'Registered version!';  
    MessageBox(0,PChar (RegMessage),'',0);  
    {$I include\\delphi_svkp_block_end.inc}  
end;  

procedure Start;  

begin  
  try  
// this code will decrypted before using and after use removed  
// tip: This type of blocks is very good use in initialize part of program  
// (load DLL(s), initialize data, fill some special variables and etc.)  

    {$I include\\delphi_svkp_block_type2_begin.inc}  
    RegMessage := 'Encryption blocks example';  
    {$I include\\delphi_svkp_block_type2_end.inc}  

// This code will decrypted before using and encrypted again after use  

    {$I include\\delphi_svkp_block_type1_begin.inc}  
    MessageBox(0,PChar (RegMessage),'',0);  
    ActionForRegisteredUsers; //this procedure is only for registered users  
    {$I include\\delphi_svkp_block_type1_end.inc}  
  except  
  end;  
end;  

end.  
```

> 注意:不要在 `Visual Basic` 中使用此方法!

> 说明:在 `SVKP` 的 `Examples/Encrypted_blocks_Type1&Type2` 目录下有使用这个方法的例子.

# 实时控制检查 (Real-time Alive Checks)

你可以给你的程序加上实时控制，这是一个极好的附加保护，它可以检测是否有人正在试图清除 `SVK Protector` 的保护.你只需要在你的程序中加一些简单的宏就可以了(和使用加密块一样的)

例如 `SVKP_ALIVE_CHECK_1`.在 `svk protector` 保护程序之后，实时控制将会被插入以替代这些宏。

推荐把这些控制插在那些随机启动或者很少使用的模块中，因为这会加大破解者查找这些控制的难度.

可以使用两种检查类型:

*   `SVKP_ALIVE_CHECK_1` 假如 `SVKP` 已被从程序中清除,那么寄存器 `EAX`, `EBX`, `ECX`, `EDX`, `ESI`, `EDI`, `EBP` 将变成 `0` 并且 `ESP` 寄存器将会包含和检查前不同的值.
*   `SVKP_ALIVE_CHECK_2` 假如 `SVKP` 已被从程序中清除那么 `ESP` 寄存器将会包含和检查前不同的值.

## 汇编示例

```asm
.586  
.MMX  
.model flat,stdcall  
include ..\\..\\Inc&Lib\\apimacro.mac  
include ..\\..\\Inc&Lib\\windows.inc  

INCLUDELIB      ..\\..\\Inc&Lib\\iKERNEL32.lib  
INCLUDELIB      ..\\..\\Inc&Lib\\iUSER32.lib  

; There are defined include file and library for SVKP's special functions  

include         ..\\..\\Inc&Lib\\special.inc  
INCLUDELIB      ..\\..\\Inc&Lib\\ispecial.lib  

.DATA?  
registration_information db 255 dup (?)         ;buffer for registration information  

.DATA  
capture db "Alive Checks example.",0  
message db "Now I will try run some checks!",0  
capture2 db "SVKP is alive!",0  
message2 db "SVKP protection was not removed!",0  

.code  
assume          fs      :flat  

Start:  

;int 3                            ;only for debug version  

        push 0  
        push offset capture  
        push offset message  
        push 0  
        iWin32 MessageBoxA  

; Don't forget use these checks random or in not very offen used procedures!  

SVKP_ALIVE_CHECK_1  

SVKP_ALIVE_CHECK_2  

        push 0  
        push offset capture2  
        push offset message2  
        push 0  
        iWin32 MessageBoxA  

        push -1  
        iWin32 ExitProcess  

end Start  
```

## C/C++ 示例

```c
#include <windows.h>  
#include <io.h>  
#include "..\\..\\Inc&Lib\\special.h"  

#pragma comment (lib,"SPECIAL")  

int WINAPI WinMain (HINSTANCE hInstance, HINSTANCE hPrevInstance,  
                    PSTR szCmdLine, int iCmdShow)  
{  
  char *message = "Now I will try run some checks!";  
  MessageBox(0,message,"Alive Checks example.",0);  

// Don't forget use these checks random or in not very offen used procedures!  

  SVKP_ALIVE_CHECK_1  
  SVKP_ALIVE_CHECK_2  
  MessageBox(0,"SVKP protection was not removed!","SVKP is alive!",0);  
  return 0;}  
```

## Delphi 示例

```pascal
unit main;  

interface  

procedure Start;  

implementation  

uses  
  Registry,  
  Classes,  
  Windows,  
  SysUtils,  
  special;  

var  
  RegMessage : String;  

procedure Start;  

begin  
  try  
    RegMessage := 'Now I will try run some checks!';  
    MessageBox(0,PChar (RegMessage),'Alive Checks example.',0);  

// Don't forget use these checks random or in not very offen used procedures!  

    $I include_svkp_alive_check_1.inc  
    $I include_svkp_alive_check_2.inc  
    RegMessage := 'SVKP protection was not removed!';  
    MessageBox(0,PChar (RegMessage),'SVKP is alive!',0);  
  except  
  end;  
end;  

end.  
```

> 说明：在 `SVKP` 的 `Examples/SVKP_Checks` 目录下有使用这个方法的例子.

# 命令行特性

为了自动运行工程和在处理完文件后自选退出，你可以使用如下的 `SVK Protector` 命令行参数

`Svkp.exe name_of_project_file.svkp`

例子:

`c:\\program files\\svkp\\svkp.exe c:\\program files\\svkp\\examples\\Advanced_TimeLock.\\dialog_example.svkp`

因为工程文件不能编辑,所以可以在保护你的程序之前自动完成不同的设置(比如如果压缩将被或者将不被使用那么 `SVKP_USE_COMPRESS=YES` 或者 `SVKP_USE_COMPRESS=NO`)或者编辑 `SVK Protector` 显示的文本.

# 用户注册密钥生成

为了使生成注册密钥的过程自动化,你必须使用一个特定的 DLL--`SVKP_DLL.DLL`.这个文件被 `SVKP` 所保护,因此和其它授权文件的使用条件是一样的.详细的信息可以在使用 DLL&OCX 文件这一节中找到.

标准用户可以使用以下函数:

## `XX33` 关联 `SVKP_Get_Version`

描述:

这个函数返回 `SVKP` 的版本.

函数原型:

`VOID XX33(VOID)`

返回值:

这个函数返回 `SVKP` 的版本参数值

## `XX34` 关联 `SVKP_Initialize_DLL`

描述：

这个函数初始 DLL.需要在初始的开始调用这个函数.

原型:

`VOID XX34(VOID)`

返回值:

*   `1`=一切顺利
*   `-1`=装载 RSA 加密 DLL 出现问题
*   `-33`=异常错误

## `XX35` 关联 `SVKP_Free_DLL`

描述:

这个函数清空缓冲并初始化内存.

原型:

`VOID XX35(VOID)`

返回值:

这个函数没有返回参数.

## `XX37` 关联 `SVKP_Load_RSA_keys`

描述:

这个函数从工程文件中装载 RSA 密钥.

原型:

`BOOL XX37( LPCTSTR lpProjectFile //工程名的地址 );`

参数:

`lpProjectFile`
指向一个以空结尾的字符串，指定了工程名.

返回值:

*   `1`=一切顺利
*   `-1`=读取公钥出现问题
*   `-2`=读取私钥出现问题
*   `-3`=从工程文件中读取密钥出现问题
*   `-33`=异常错误

## `XX38` 关联 `SVKP_Generate_license_key`

描述:

这个函数在内存中生成授权

原型:

`BOOL XX38( LPCTSTR lpRegInformation, //注册信息指针 DWORD nRegInformationSize, //注册信息长度 LPCTSTR lpHWInformation, //硬件信息指针`



# 参数
### `lpRegInformation`
指向注册信息文本

### `nRegInformationSize`
以字节为单位的注册信息的大小

### `lpHWInformation`
指向为硬件锁定授权的硬件信息.这个参数可以为空.

### `lpTimeExpirationData`
指向为时间限制授权的时间到期信息.格式 `dd/mm/yyyy` 比如 `20/10/2004`.这个参数可以为空.

### `DWORD Unknown`
请使用 `NULL`.

# 返回值:
这个函数返回指向生成的授权

-1=参数错误

-2=私钥错误

-3=加密出现问题

-4=硬件 ID 错误

-5=到期数据错误

-33=异常错误

# XX39 关联 `SVKP_Check_licence_key`
## 描述:
这个函数从内存中检测授权

## 原型：
```
BOOL XX39(
    LPVOID lpLicenseKey,   //内存中授权密钥的地址
    LPCTSTR lpHWInformation, //硬件信息指针
);
```

## 参数:
### `lpLicenseKey`
指向内存中授权密钥

### `lpHWInformation`
指向为硬件锁定授权的硬件信息.这个参数可以为空.

## 返回值:
这个函数返回指向返回结构的指针

+0=指向解密注册信息

+4=指向密钥到期的数据或者如果到期数据没有使用为空

+8=不知道

-1=参数错误

-2=授权密钥错误

-3=这是硬件锁定密钥调用函数不需要硬件 ID

-4=错误的硬件信息

-33=异常错误

# XX40 关联 `SVKP_Load_license_key`
## 描述:
这个函数加载授权密钥到内存

## 原型:
```
BOOL XX40(
    LPCTSTR lpLicenseFile     //授权文件名地址
);
```

## 参数:
### `lpLicenseFile`
指向一个以空结尾的字符串,指向授权文件名

## 返回值:
这个函数返回装载授权文件的内存地址

-1=参数错误

-2=装载授权文件出现问题

-33=异常错误

# XX41 关联 `SVKP_Save_license_key`
## 描述:
这个函数保存由 XX38 关联 `SVKP_Generate_license_key` 在内存中生成的授权密钥.

## 原型:
```
BOOL XX40 (
    LPCTSTR lpLicenseFile,       //授权文件名地址
    LPVOID  lpLicenseKey         //授权密钥的内存地址
);
```

## 参数:
### `lpLicenseFile`
指向一个以空结尾的字符串,指向授权文件名

### `lpLicenseKey`
指向内存的授权密钥

## 返回值:
1=一切顺利

-1=参数错误

-33=异常错误

## 说明:
`SVKP_DLL.DLL` 还有其它的对标准用户无效的函数(例如生成特殊密钥).如果你对此感兴趣,

请通过 `email:sales@anticracking.sk` 和我们联系

# Visual Basic 开发者的注意事项
`visual Basic` 程序设计语言的结构不允许使用第三方的程序保护方法(对应用程序的某些部分进行

加密).不过好的软件保护可以使用特殊的 API 函数来使其更安全.但是可以使用 `SVK Protector` 的注册

密钥.

# API 描述
这些专用 API 函数的使用是很方便的。已经在汇编，Visual C++,Delphi 和 Visual Basic 中测试通过。

假设你需要从某个 DLL 中调用函数(例如 `special.dll`)

现在你可以使用下面的专用函数了:

`SVKP_GetRegistrationInformation`

`SVKP_LockKeyboard`

`SVKP_UnlockKeyboard`

`SVKP_KillDebugger`

`SVKP_RebootComputer`

`SVKP_GetHWInfo`

`SVKP_GetRegistrationInformation`

# `SVKP_GetRegistrationInformation`
## 函数功能:
`GetRegistrationInformation` - 获得指向注册信息的指针.

## 函数原型:
```
BOOL SVKP_GetRegistrationInformation(
    LPCTSTR lpRegistrationInformation //注册信息的缓冲区地址
);
```

## 参数:
### `lpRegistrationInformation`
注册信息的 255 位长缓冲区地址.`SVK Protector` 把从注册密钥中得到的注册信息放在那里.

## 返回值:
如果函数调用成功,返回值为真.

如果函数调用失败,返回值为假.

# `SVKP_LockKeyboard`
## 函数功能:
这个函数可以锁定键盘.调用了这个函数后,不仅仅是调用的程序不能使用键盘,整个 Windows 的所有程序

都不能使用键盘.这个函数对代码的关键部分(检验授权或者时间限制)很有好处.别忘了在这之后要调用

`SVKP_UnlockKeyboard`.

## 函数原型:
```
VOID SVKP_LockKeyboard(VOID)
```

## 返回值:
这个函数没有返回值.

`备注:` 在这个 `SVK Protector` 版本中这个函数已经不再使用，因为某些类型的键盘会产生问题.

# `SVKP_UnlockKeyboard`
## 函数功能:
这个函数可以解锁键盘.调用了这个函数后就可以使用键盘了.

## 函数原型:
```
VOID SVKP_UnLockKeyboard(VOID)
```

## 返回值:
这个函数没有返回值.

`备注:` 在这个 `SVK Protector` 版本中这个函数已经不再使用，因为某些类型的键盘会产生问题.

# `SVKP_KillDebugger`
## 函数功能:
这个函数可以检测所有的调试器.建议在代码的关键部分(检查授权或才时间限制)之前调用它.如果

检测到调试器,它将不和用户打招呼就重起计算机.

## 函数原型:
```
VOID SVKP_KillDebugger(VOID)
```

## 返回值:
这个函数没有返回值.

# `SVKP_RebootComputer`
## 函数功能:
有时需要重起计算机.这个函数将不和用户打任何招呼而安全的重起计算机.

## 函数原型:
```
VOID SVKP_RebootComputer(VOID)
```

## 返回值:
这个函数没有返回值.

`警告:` 请慎用这个函数因为用户所有没保存的工作都会丢失.

`备注:` 由于一些问题这个函数在这个版本的 `SVK Protector` 中已经不再使用.

# `SVKP_GetHWInfo`
## 函数功能:
硬件 ID 的 16 字节长度的缓冲区地址.这个硬件 ID 每个计算机都是不同的，`SVK Protector` 用它来生成注册

密钥.调用这个 API 你可以找到一个硬件 ID,例如你可以把显示在用户的显示器上.他把这个硬件 ID 发给你

你用这个硬件 ID 为他生成一个只可以在他的计算机上工作的注册密钥。

## 函数原型:
```
BOOL SVKP_GetHWInfo(
    LPCTSTR lpHWInformation //硬件 ID 的缓冲区地址
);
```

## 返回值:
这个函数没有返回值

`4dd466d0e1e0cc123afbe504da2462ca`
