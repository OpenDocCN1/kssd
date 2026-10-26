
# 微信界面分析

> 微信版本：3.9.0.28
>
> 下载链接：链接：f2fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4k6g2g2d9z5h3S2G2j5f1Z5^5L8#2)9#2k6V1&6y4f1Y4W2T1d9W2)9J5k6p5#2k6M7p5q4Q4x3@1k6H3N6$3c8Q4x3@1c8A6L8$3k6#2
> 提取码：iofu
>
> [duilib](https://bbs.kanxue.com/elink@142K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8#2)9J5k6h3y4K6k6r3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8#2)9J5c8Y4y4W2j5i4u0U0K9q4)9K6c8Y4q4Q4x3@1c8V1N6h3W2D9K9h3u0Q4x3U0k6S2L8i4m8Q4x3@1u0K6M7r3#2Q4x3@1b7I4x3o6l9I4i4K6u0W2x3U0p5H3x3g2)9J5k6e0x3H3x3o6q4Q4x3X3f1%4x3o6t1H3)：链接：0b0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4i4K6g2X3j5%4g2n7y4$3f1J5j5V1u0T1d9g2A6J5d9$3g2^5k6X3#2w2b7X3q4Y4i4K6y4r3M7s2N6V1i4K6y4p5M7i4R3%4P5l9`.`.
> 提取码：qx7x
>
> 如造成侵权，请联系作者处理。

## 目录

- [微信界面分析](https://bbs.kanxue.com/#微信界面分析)
- [一、缘由](https://bbs.kanxue.com/#一、缘由)
- [二、一些已掌握的信息](https://bbs.kanxue.com/#二、一些已掌握的信息)
- [三、准备冻手](https://bbs.kanxue.com/#三、准备冻手)
- [1）搞一下源码](https://bbs.kanxue.com/#1）搞一下源码)
- [2）资源加载方式](https://bbs.kanxue.com/#2）资源加载方式)
- [3）从源码入手](https://bbs.kanxue.com/#3）从源码入手)
- [4）找到解密点](https://bbs.kanxue.com/#4）找到解密点)
- [四、解密](https://bbs.kanxue.com/#四、解密)
- [五、总结](https://bbs.kanxue.com/#五、总结)

## 一、缘由

微信的白色背景页面实在是对像笔者这样眼睛不太好的用户不太友好，所以就有自己给他改改界面的打算，就改个颜色，应该不会太麻烦，所以就写了这篇文章记录一下过程。受限于笔者水平以及时间问题，这篇文章最终没有实现目的，但是理论上文章中所讲方法是可以实现的，并可以提取大部分微信界面UI布局，并且对其他功能的分析也有帮助。笔者将在文中给出使用`frida`进行简单内存`dump`的方式获取`xml`以及静态解密的方式拿到`XML`。

## 二、一些已掌握的信息

1. 微信界面使用了`Duilib`库
2. `Duilib`<font color=red>实现了从`XML`中加载页面布局</font>的能力
3. 本质上是<font color=red>封装了一层`MFC`</font>并自写了解析器来完成相关布局设置
4. 微信的主要功能模块是`WeChatWIn.dll`

比较笼统的资料即以上四点，这对我们帮助还是比较多的：

- `Duilib`库是开源的，官方已经停止了对`Duilib`的维护，现在的版本基本上都是由个人维护的
- `Duilib`从`XML`中加载布局，这给了我们很好的着手点，我们可以试着找到`XML`文件对其更改来达到更改背景颜色的目的
- 本质是一层`MFC`封装，那么免不了从资源中加载文件，会不会它吧`XML`文件放到`rc`里的呢？
- 我们的主要分析应该是在`WeChatWIn`中进行。

## 三、准备冻手

### 1）搞一下源码

微信团队之前开源过`Duilib`的源码，但是现在`Github`上应该是找不到了，笔者这里有一份，出于学习的目的需要请联系笔者。拿到源码后笔者简单看了一下，对比官方原来来说改动不算太大。

### 2）资源加载方式

因为我们的关注点是微信是从哪里获取`XML`资源的，所以我们需要大致了解一下`Duilib`的资源加载方式，微信之前开源的`Duilib`中有四种资源加载方式：

```cpp
enum UILIB_RESTYPE
{
    UILIB_FILE = 1, // 来自磁盘文件
    UILIB_ZIP, // 来自磁盘zip压缩包
    UILIB_RESOURCE, // 来自资源
    UILIB_ZIPRESOURCE, // 来自资源的zip压缩包
};
```

从磁盘上以及从`ZIP`中加载资源基本上是不可能的，我们在微信的根目录里没有发现相关信息，保险一点，我们通过火绒剑来监控一下文件操作，防止搞错方向。

我们只关注文件操作，设置好相关过滤后，我们记录微信启动完成的相关信息，搜索关键字“`.xml`”、`zip`，没有发现什么有用的信息，但是在我们搜索“`Resource`”时，我们留意到

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_ZRKZJJHNSY5K8KN.png) ![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_VH5R4MYBA9KFYMG.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_PHW2RNX4X86NQXM.png)

微信加载了这两个非常可疑的`dll`，我们索性用`Resource Hacker`看看里面有什么。

首先`WeChatResource.dll`里面有一些较为简单的资源以及一个不明所以的奇怪资源，资源类型为`WXZ`，如下图：

### 3）从源码入手

可以看到基本上对应了`z_streamp`的结构，其中有我们非常眼熟的缓冲区`5CC98D60`，对应的就是`next_out`，而`opaque`指向的是`WeChatResource.dll`中的数据，我们跟进

![图片描述](upload/attach/202303/947361_9S4ADCRAFKBAFUN.png)

复制相应的十六进制流，在`WeChatResource.dll`中搜索，成功搜索到了对应的十六进制内容，如下图

![图片描述](upload/attach/202303/947361_NDQMY6S5YBCVYYU.png)

可见，`opaque`保存的的是资源文件中的内容。

不要忘了字段`next_in`，按照解释，其应该指向的是将要解压的内容的缓冲区，我们跟进，如下图

![图片描述](upload/attach/202303/947361_QZ3VE78HTEQXRSN.png)

理论上将，将要解压的内容应该与`opaque`里的内容一致，但这里却不同。经过笔者猜测，这里应该是又一层加密，原因是，当微信加载`PNG`类型图片时，`opaque`内容与`next_in`内容一致，或者在`wechatresource.dll`中搜索`78 9C`后，可以发现很多搜索项，如下图：

![图片描述](upload/attach/202303/947361_E7KFKSGW7NWQE9Q.png)

可以推测，部分资源的部分内容没有被加密。

那么我们怎么找到解密点呢？

### 4）找到解密点

一种比较朴素的想法是，在没进行解压之前对密文下访问断点，进行解密时必然要访问这篇内存。

存在的问题是，我们需要找到一个合适的位置，在没进行解压之前下断点。根据之前的分析，我们似乎不能很好地确定这个位置，因为我们似乎除了在`inflate`例程中再没有发现密文出现的地方。

一种或许可行但不太优雅的方法是：记录一下加载某个页面时的密文缓冲区，在`wechatresources`中定位，然后在加载资源的地方（`LoadResource`）找到资源缓冲区，继而定位到加载那个指定界面时对应的密文缓冲区，对其下硬件访问断点，再次触发其页面加载，理论上可以断在解密的地方。

这里笔者使用的是一种非常笨拙的方法，赌解密的地方离解压的地方不远，即回溯，根据源码定位`z_streamp`结构的组装过程，根据内存访问关系找到解密资源的点。这个过程相当笨拙且没什么技术含量，所以这里笔者不再赘述。

最后，笔者定位到RVA：`01E103F1`，如下图：

![图片描述](upload/attach/202303/947361_E7QXJRPTT2VZ52S.png)

根据函数名读者可以猜到，密文用的是异或加密，我们跟进这个函数

![图片描述](upload/attach/202303/947361_P9EQZ3H33Z5DYU7.png)

根据伪代码分析，这段代码主要是对`a1`所指向的内存区域进行加密。下面是一些推测：

变量`v4`是一个密文缓冲区，初始值为`a1+32`，即指向`a1`之后的第32个字节的地址。

加密算法中使用了`SSE`指令集中的`_XMMINTRIN_H`头文件中的`_mm_xor_si128()`函数进行异或操作，`xmmword_2B01A80`是一个128位的常量值，可能是加密算法中使用的密钥。

在主循环中，每次处理64字节的数据，并且对这64字节中的每个128位（即16字节）进行加密。加密的方式是将128位的数据与`xmmword_2B01A80`进行异或操作，并将结果存储回内存中。

当`a2`的大小小于64字节时，程序会跳到`LABEL_9`处，进入另一个加密算法中，该算法对剩下的不足64字节的数据进行单字节异或加密。

函数的返回值为`result`，表示加密完成的数据的字节数。

综上所述，这段代码可能是一种基于`SSE`指令集的异或加密算法，密钥为`xmmword_2B01A80`。

在这里秘钥为`0x63`，即字符`c`，这个秘钥在不同`wxid`下是否不同笔者没有测试。

## 四、解密

下面我们测试一下结论是否正确。

我们在资源中复制一个加密`chunk`，`chunk`以`1B FF`作为文件头，转储到一个名为`dump.xml`的文件中，然后尝试解密解压，并将其写入`decode.txt`中。下面给出不成熟的脚本：

```c++
#include <iostream>
#include "zconf.h"
#include "zlib.h" // 包含zlib
#include <filesystem> // 需要c++17标准支持
#include <fstream>
using namespace std;

void UnCompress()
{
    uLong file_size = std::filesystem::file_size("dump.xml");
    uLong unfile_size = file_size * 10;
    std::fstream fp("dump.xml", ios::in | ios::binary);
    char* buf = new char[file_size] {0};
    fp.read(buf, file_size);
    for (int i = 0; i < file_size; i++)
    {
        buf[i] ^= 0x63;
    }
    char* de_buf = new char[unfile_size] {0};
    int x = uncompress((Bytef *)de_buf, &unfile_size, (Bytef *)buf, file_size);
    std::fstream de_fp("decode.txt", ios::out | ios::binary);
    de_fp.write(de_buf, unfile_size);
    de_fp.close();
    fp.close();
    delete[] buf;
    delete[] de_buf;
    buf = nullptr;
    de_buf = nullptr;
}

void Compress()
{
    uLong file_size = std::filesystem::file_size("decode.txt");
    uLong unfile_size = file_size * 10;
    std::fstream fp("decode.txt", ios::in | ios::binary);
    char* buf = new char[file_size] {0};
    fp.read(buf, file_size);
    char* de_buf = new char[unfile_size] {0};
    int x = compress((Bytef *)de_buf, &unfile_size, (Bytef *)buf, file_size);
    for (int i = 0; i < unfile_size; i++)
    {
        de_buf[i] ^= 0x63;
    }
    std::fstream de_fp("dump.xml", ios::out | ios::binary);
    de_fp.write(de_buf, unfile_size);
    de_fp.close();
    fp.close();
    delete[] buf;
    delete[] de_buf;
    buf = nullptr;
    de_buf = nullptr;
}

int main()
{
    //Compress();
    UnCompress();
    return 0;
}
```

![图片描述](upload/attach/202303/947361_2XZRWP4E5XPV6FT.png)

![图片描述](upload/attach/202303/947361_3MGCVYDYGNTJMDW.png)

可以看到我们拿到了`xml`的内容，证实我们之前的分析是正确的。

至此，静态解密得到`xml`的目的已经实现。出于笔者水平原因，并不能拿到`xml`对应的文件名字。有兴趣的读者可以继续向下分析。如果想要拿到`xml`的名字，比较简单的方法是内存`dump`。下面笔者将使用`frida`来`HOOK`相关函数，缺点也是比较明显，我们通过`HOOK`很难实现完全的`dump`出所有的`xml`，因为解密渲染例程只有在被调用时我们才能拿到`xml`。

这里，我们选择的`HOOK`点为调用`LoadFromMem`的点上方，对应RVA：`1C9D5C3`，这里有`xml`缓冲区，缓冲区`+0x2004`指明了其大小，并有`xml`对应的名字，如下图

![图片描述](upload/attach/202303/947361_R4EG2GJWQ9XKQ32.png)

给出一下`frida`脚本（需要`frida`环境）：

```javascript
let file_name;
let file_size=0;
function GetVAByModuleNM(moduleName,offset)
{
    var base=Module.findBaseAddress(moduleName);
    if(base==null)
    {
        base=enum_to_find+moudle(moduleName);
    };
    console.log(moduleName + ":" + base);
    var target_addr=base.add(offset);
    return target_addr;
}
let Addr=GetVAByModuleNM("WeChatWin.dll",0x1C9D5C3);
Interceptor.attach(Addr,{
    onEnter:function(arg)
    {
        file_name=ptr(this.context.esp).readPointer().readUtf16String();
        var f = new File("XML\\"+file_name.toString().replace(/\\/g,"_")+'.txt', 'wb');
        f.write(ptr(ptr(this.context.eax).add(0x3)).readByteArray(this.context.eax.add(0x2004).readU32()-0x3));
        f.flush();
        f.close();
        console.log(file_name);
    }
});
```

调用命令：

```
frida WeChat.exe -l 脚本名字
```

效果如下：

![图片描述](upload/attach/202303/947361_UFKB69PUEFU7KP4.png)

![图片描述](upload/attach/202303/947361_2PYMK7ZBZ7FGUBT.png)

## 五、总结

- 微信的资源放在`WeUIResource.dll`和`WeChatResource.dll`中

- `xml`资源使用了异或加密，需要注意的是资源里的部分`PNG`没有加密，里面也有`zip`文件

- `xml`资源使用`zlib`解压缩

- 在内存中修改`xml`中的内容可以实现修改微信UI，也就是说，我们可以通过`frida`脚本定位到需要修改UI，然后再资源里查找，解密，修改，再加密替换理论上可以实现更改UI的目的，但是限制比较多，理想的方法是能够完全解压资源文件修改后再根据文件格式打包回去，受限于技术原因，笔者暂时无法实现，非常乐意有想法的朋友交流，也期望各位师傅指点。

- 类似于`electron`这种框架或者`java`虚拟机，将必要文件在运行时加载解析的先天诟病就是很容易在内存中拿到相关信息，而`frida`就是这方面的瑞士军刀。

- 这些`xml`布局中有很多关于功能，例如在布局中有`send_btn`，那么在代码中将会有`if (tmp == "send_btn")`，这离具体功能应该相当近，也就是说，了解微信的`xml`文件也给我们分析一些微信的功能提供了一种思路。
