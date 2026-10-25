
# 三个反汇编库的简单介绍和使用

环境：`VS2017 + Windows 10`

## 一、Udis86

`udis86`的代码风格十分简洁，功能函数短小，变量命名清楚又简洁，接口干净意思明白，操作灵活。从github（官网）下载源码，解压之后我们把`libudis86`文件夹放到我们的项目里面，然后把`udis86.h`外一层，结果我们的设置好后编译，发现有点问题：（github上下的是新一点的版本，但是编译提示很多错误，比如少`itab.h`，什么的，但是打开官网就发现这个文件确实是在`udis86`这个文件夹下，不知道什么情况，也懒得研究了，直接用官网的那个吧）

![optionbuild](upload/attach/202105/746200_ZRM2BYKYSTB52XN.png)

由于我用的是`VS2017`，可能限制比较多，然后我们把这些设置好后，发现还有个`memset`未定义没有解决，当然只是警告，但是按我们的尿性，要搞就搞完美的。

我们直接在`udis86.c`最上面几个`#include`的后面放上`#include<memory.h>`，编译OK，

写下demo看看：随便找一段汇编hex，然后翻译看看

```C
#include<stdio.h>
#include<string.h>
#include"udis86.h"
uint8_t dump[] = {
0x55, 0x8B, 0xEC, 0x51, 0x8B, 0x45, 0x08, 0x50,
0xE8, 0xC6, 0x05, 0xFE, 0xFF, 0x83, 0xC4, 0x04,
0x85, 0xC0, 0x74, 0x09, 0xC7, 0x45, 0xFC, 0x00,
0x00, 0x00, 0x00, 0xEB, 0x07, 0xC7, 0x45, 0xFC,
0xFF, 0xFF, 0xFF, 0xFF, 0x8B, 0x45, 0xFC, 0x8B,
0xE5, 0x5D, 0xC3
};
int main(int argc,char *argv[])
{
ud_t ud_obj;
ud_init(&ud_obj);
ud_set_input_buffer(&ud_obj, dump,sizeof(dump));
ud_set_pc(&ud_obj, 0x420170);
ud_set_mode(&ud_obj, 32);
ud_set_syntax(&ud_obj, UD_SYN_INTEL);
while (ud_disassemble(&ud_obj)) {
printf("%llX >> %-16s | %s\n", ud_insn_off(&ud_obj), ud_insn_hex(&ud_obj),ud_insn_asm(&ud_obj));
}
return 0;
}
```

编译运行，效果如下：

![upload/attach/202105/746200_U23JMX3QS884567.png](upload/attach/202105/746200_U23JMX3QS884567.png)

完美翻译，只是那个`-1`和`0xffffffff`看着有差异，这个肯定是可以改的，这个就不多说了，具体看官方文档吧。他支持的x86体系扩展指令有：

> MMX, FPU (x87), AMD 3DNow, SSE, SSE2, SSE3, SSSE3, SSE4.1, SSE4.2, AES, AMD-V, INTEL-VMX, SMX

速度超级快，代码也不臃肿，那么久决定他本身实现的一些功能可能就少了，有些不能直接用，还要自己把一些函数组合起来，一些寄存器跟踪操作也不太好操作，比如`mov eax,ebx`这条指令执行后我们的oprand 1(eax)是发生了什么事，这个就要处理这个mnemonic是什么等等。

## 二、BeaEngine

### A.失败的尝试：

在github中下载`beaengine-master.zip`后解压到当前目录得到`beaengine-master`文件夹，然后用`cmake`生成一下工程

```assembly
> cd beaengine-master
\beaengine-master>mkdir build
\beaengine-master>cd build
\beaengine-master\build>cmake ..
```

打开`cmd`，进入刚刚的目录，然后进入我们刚刚解压的目录，然后执行上面的命令，构建官方工程，注意如果你的VS有安装了驱动环境，可能会构建失败，成功如图：

![upload/attach/202105/746200_8JZJJ9DRK9VXF33.png](upload/attach/202105/746200_8JZJJ9DRK9VXF33.png)

然后我们进入到刚刚创建的`build`文件里面去，直接双击`BeaEngine.sln`打开VS，然后点击生成，成功。

![upload/attach/202105/746200_AYNDSYFAZSWUM2Z.png](upload/attach/202105/746200_AYNDSYFAZSWUM2Z.png)

警告我们就忽略吧。然后我们在刚刚的build目录开始路径 `build\lib\Windows.msvc.Debug\Debug`下得到`BeaEngine_s_d_l.lib`静态链接库，等下我们会用到。

A.我们新建一个过程 命名为`BeaEngine_demo`，然后我们提取一些必要文件到工程目录下，如下图：

![upload/attach/202105/746200_AT23TCSC7Y5XQCD.png](upload/attach/202105/746200_AT23TCSC7Y5XQCD.png)

把我们刚刚`BeaEngine_s_d_l.lib`放到`lib`文件夹下，然后`headers`直接复制刚刚解压的`beaengine-master`目录下的`headers`过来就行。不需要设置什么，然后在`BeaEngine_demo.cpp`写下测试代码：

```C
#include <stdio.h>
#include <string.h>
#include <stdint.h>
#include "headers/BeaEngine.h"
#pragma comment(lib,"lib/BeaEngine_s_d_l.lib")
int main(int argc,char *argv[])
{
uint8_t temp[] = {
0x55, 0x8B, 0xEC, 0x51, 0x8B, 0x45, 0x08, 0x50,
0xE8, 0xC6, 0x05, 0xFE, 0xFF, 0x83, 0xC4, 0x04,
0x85, 0xC0, 0x74, 0x09, 0xC7, 0x45, 0xFC, 0x00,
0x00, 0x00, 0x00, 0xEB, 0x07, 0xC7, 0x45, 0xFC,
0xFF, 0xFF, 0xFF, 0xFF, 0x8B, 0x45, 0xFC, 0x8B,
0xE5, 0x5D, 0xC3
};
DISASM infos;
int len= 0;
//puts(BeaEngineVersion());
(void)memset(&infos, 0, sizeof(DISASM));
infos.Archi = 1;
infos.EIP = (UInt64)temp;
infos.VirtualAddr = 0x420170;
while ((infos.Error == 0)&&(infos.EIP-(UInt64)temp)<sizeof(temp)) {
len = Disasm(&infos);
if (infos.Error != UNKNOWN_OPCODE) {
printf("%llx >> %s\n",infos.VirtualAddr,infos.CompleteInstr);
infos.EIP += len;
infos.VirtualAddr+=len;
}
}
return 0;
}
```

编译，发现提示

> 严重性 代码 说明 项目 文件 行 禁止显示状态
> 错误 LNK2019 无法解析的外部符号 __imp__ Disasm，该符号在函数 _main 中被引用 BeaEngine_demo E:\Test Project Vs2017\BeaEngine_demo\BeaEngine_demo\BeaEngine_demo.obj 1

意思是`Disasm`这个函数有了说明，但是我们没有找到他的实现，我们无法解析他。这就很奇怪了我们不是已经链接上lib静态库了吗？我们返回到build工程。

![upload/attach/202105/746200_FUGY4SJUEW9Q5MM.png](upload/attach/202105/746200_FUGY4SJUEW9Q5MM.png)

查看这里，如果`BEA_ENGINE_STAIC`宏被定义，则`BEA_API`不被赋宏，你也可以理解为NULL，不定义函数导入或者导出。那么我们看看工程设置。

![upload/attach/202105/746200_4SJ56MCDKQMCTT5.png](upload/attach/202105/746200_4SJ56MCDKQMCTT5.png)

把`BEA_ENGINE_STATIC`去掉，添加`BUILD_BEA_ENGINE_DLL`，这个时候`BEA_API`应该指向的是`bea_api_export__` 编译生成，然后把这个文件放到我们demo的lib下，demo编译生成，还是一样的错误。回到GitHub，看到如下，编译为动态连接库(DLL)用吧，如果有知道原因的请告知一下，谢谢：

![upload/attach/202105/746200_F5RPPASGGUMJJTC.png](upload/attach/202105/746200_F5RPPASGGUMJJTC.png)

编译为动态链接库的cmake选项，

### B.正确的尝试

我们重新解压`beaengine-master`，右键解压到`beaengine-master`注意目录结构，然后我们命令行

```
>cmake -DoptBUILD_DLL=ON beaengin-master
```

成功后差不多是这个样子

![upload/attach/202105/746200_SEDNDK5XN85VBG7.png](upload/attach/202105/746200_SEDNDK5XN85VBG7.png)

然后用我们的`vs2017`打开这个`.sln`，编译生成OK。

在`E:\Test Project Vs2017\beaengine-master\lib\Windows.msvc.Debug\Debug`找到导出表`BeaEngine_d_l.lib`，我们可以跟A中尝试错误的那个对比一下

![upload/attach/202105/746200_2TH2MZ8BVZM479F.png](upload/attach/202105/746200_2TH2MZ8BVZM479F.png)

按道理那么我们A中的逻辑应该是没错的，用`dumpbin`看一下这个导出表lib的导出函数，OK，和之前我们看到的要导出的3个函数是吻合的，这里怎么多了_

![upload/attach/202105/746200_3D7M6GXGZC58A3D.png](upload/attach/202105/746200_3D7M6GXGZC58A3D.png)

用`dumpbin`去看那个1.388KB的那个lib，静态链接库，没看到导出函数，不知道是不是我用法错了，函数这个LIB没搞对。把刚刚得到的`BeaEngine_d_l.lib`导出表放到我们的demo文件下的`lib`里面去，然后把代码连接的lib改一下：

```c
#pragma comment(lib,"lib/BeaEngine_s_d_l.lib")
改为
#pragma comment(lib,"lib/BeaEngine_d_l.lib")
```

编译生成通过，运行提示没有DLL，OK把DLL也是复制过来，在文件夹

```c
E:\Test Project Vs2017\beaengine-master\bin\Windows.msvc.Debug\Debug
```

下找到DLL(`BeaEngine_d_l.dll`)，复制一份到我们的demo的debug文件夹下

![upload/attach/202105/746200_CKY7XHMM8494G8T.png](upload/attach/202105/746200_CKY7XHMM8494G8T.png)

运行比较一下结果：

![upload/attach/202105/746200_G5XGYYEHY5UN2QT.png](upload/attach/202105/746200_G5XGYYEHY5UN2QT.png)

这里有看雪stanford前辈总结的BeaEngine库的反汇编参数和函数说明：https://bbs.pediy.com/thread-177301.htm 。这里就不多说明了。

## 三、Zydisasm

这个可能不怎么听过，如果说`x64dbg`那你可能就我靠原来是他，没错`x64dbg`调试器的反汇编就是用的他，各方面都比较全，他和`BeaEngine`最大的区别，就我目前而言，我发现他比较控制入微，比如说的`mov ax,0x11` 那么他可以解析到是`ax`寄存器，而`BeaEngine`只能解析到是`eax`，用其他的辅助的话也能知道是`ax`，但是反过来在代码层面上`Zydisasm`对这条汇编，如果你只要`ax`是`eax`你只要知道他是`eax`寄存器就行了，不管他是`ax`，`al`，`ah`，那么他可能就有点难受了，但是前者的`BeaEngine`就能直接分辨出是`eax`，这样对于vmp这样的混淆来说，`BaeEngine`直接操作可能舒服点。官网：e1dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7P5h3c8A6M7#2)9J5k6i4u0W2i4K6u0r3，其官网主页最下面有个及时反汇编，可以让读者感受一下他的魅力：

![upload/attach/202105/746200_K7QAZW2G8XHCM9F.png](upload/attach/202105/746200_K7QAZW2G8XHCM9F.png)

OK，不能在水了，直接github下载源码，解压，注意`dependencies`文件夹下的`zycore`文件有没下到，我是直接下那个他是没帮我下这个`Zycore`的，要在github里面，点击这个`dependencies`文件夹下跟上去，然后自己下这个`zycore`，看名字就知道这个很重要了，`Zydis` 核心是吧 。下载好后，把他按github上排列的文件夹结构给他搞好，解压到当前文件得到`zydis-master`文件夹，创建一个文件夹`generZydis`，然后把刚刚的`zydis-master`放到里面去，把刚刚下的`zycore-master`解压到当前文件夹，得到`zycore........`文件夹，重命名为`zycore`，然后把这个文件夹`zycore`放到`generZydis/Zydis-master/dependencies`文件夹下，覆盖之前的`zycore`，OK，然后打开`cmd`窗口

```C
>cd generZydis
generZydis>cmake zydis-master
```

就可以了，成功如图：

![upload/attach/202105/746200_5DEXGWZ9WZARKX7.png](upload/attach/202105/746200_5DEXGWZ9WZARKX7.png)

有个个什么失败了，不需要管他，是啥，我不知道额。生成后`gener`文件夹下如图：

![upload/attach/202105/746200_C9NYHK96CHM487C.png](upload/attach/202105/746200_C9NYHK96CHM487C.png)

用`VS2017`去打开这个`Zydis.slh`即可，编译生成，看到第一个例子：`Formatter01`
