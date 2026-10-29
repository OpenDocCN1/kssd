
# [原创]cocos2dx lua 反编译(20170417增加补充说明)

网上各种教程、各种工具用不了，才会有这个文章。附件是我修改的支持`luajit 2.1.0-beta2`反编译的`LJD`

## lua bytecode解密

知己知彼很重要，搜索“cocos2dx lua 加密”大概可以找到类似下面的代码。

```
bool AppDelegate::applicationDidFinishLaunching()

{

    ...

 

    CCLuaStack *pStack = pEngine->getLuaStack();

 

    // 如果设置了 -e 和 -ek 要加上下面这句
    // pStack->setXXTEAKeyAndSign("aaa", 3);
    // 如果设置了 -e 和 -ek -es 则要加上下面这句
    pStack->setXXTEAKeyAndSign("aaa", 3, "XT", 2);
    // load framework
    pStack->loadChunksFromZip("res/framework_precompiled.zip");
    pStack->loadChunksFromZip("res/game.zip");
    pStack->executeString("require 'main'");

 

    return true;

}
```

我的目标是一个`ANDROID`游戏，`APK`文件直接解压。一般情况是`libcocos2dlua.so`，`IDA`打开，函数窗口直接搜索`applicationDidFinishLaunching`，就能带你飞，可惜只有`loadChunksFromZip`，没有`setXXTEAKeyAndSign`，这保存解密`KEY`的被编译优化了。怎么办？`IDA`字符串窗口帮你忙，编译器编译代码的时候都是就近原则，只要是差不多地方出现的字符串，都会被放在一起。加密的`ZIP`文件，文件头几个字符就是`setXXTEAKeyAndSign`的`SIGN`参数，加上`ZIP`文件本身的路径，搜索目标`ZIP`文件就可以了。如图，`IDA`显示附近也就几个字符串，剩下的`KEY`参数是哪个，一个一个试就可以了。

加密的`ZIP`文件：

![](https://bbs.kanxue.com/upload/attach/201704/55894_v86ik9wavh9zkl2.png)

`IDA`中的字符串，第一个黑块是`SIGN`参数，非常幸运，尝试第二个黑块就是`KEY`：

![](https://bbs.kanxue.com/upload/attach/201704/55894_x9usznd7junyxv8.png)

`f16K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3y4G2M7K6u0V1i4K6u0r3j5$3!0U0L8%4x3J5k6q4)9J5k6s2S2Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3@1k6H3i4K6y4p5x3g2)9J5y4X3q4E0M7q4)9K6b7Y4q4Q4x3@1c8D9L8$3q4V1b7$3S2#2L8X3E0K6c8Y4u0G2L8g2A6A6M7q4)9J5y4X3q4E0M7q4)9K6b7Y4c8&6M7r3g2Q4x3@1c8Q4x3U0k6S2L8i4m8Q4x3@1u0#2N6r3j5^5i4K6y4p5i4K6t1#2c8e0u0Q4x3U0f1&6b7#2)9J5y4e0V1K6](https://bbs.kanxue.com/elink@25dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3y4G2M7K6u0V1i4K6u0r3j5$3!0U0L8%4x3J5k6q4)9J5k6s2S2Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3@1k6H3i4K6y4p5x3g2)9J5y4Y4q4Q4x3@1c8D9L8$3q4V1b7$3S2#2L8X3E0K6c8Y4u0G2L8g2A6A6M7q4)9J5y4Y4c8&6M7r3g2Q4x3@1c8Q4x3U0k6#2N6r3j5^5i4K6y4p5i4K6t1#2c8e0u0Q4x3U0f1&6b7#2)9J5y4e0V1K6)` 找到`cocos2dx`的`loadChunksFromZip`的源码

```
        if (isXXTEA) { // decrypt XXTEA
            xxtea_long len = 0;
            buffer = xxtea_decrypt(bytes + stack->_xxteaSignLen,
                                   (xxtea_long)size - (xxtea_long)stack->_xxteaSignLen,
                                   (unsigned char*)stack->_xxteaKey,
                                   (xxtea_long)stack->_xxteaKeyLen,
                                   &len);
            zip = ZipFile::createWithBuffer(buffer, len);
        } else {
            if (size > 0) {
                zip = ZipFile::createWithBuffer(bytes, (unsigned long)size);
            }
        }
```

这里我遇到一点困难，`cocos2dx`的源码项目，没有`xxtea`的源码！不过还是让我在`github`搜索出来了，源码在此：

`fc4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5P5s2c8W2j5g2)9J5c8Y4S2^5N6r3g2S2i4K6u0V1j5H3`.`.

直接写个`C++`代码，就把加密的`ZIP`解出来了。顺带一提，`Sign`就是用来跳过，跟什么`PE,MZ`是一样的。

```
HANDLE hFileOUT = CreateFile(L"e:\\tmp\\y1\\xxx\\assets\\res\\de_xxx.zip", GENERIC_ALL, 0, 0, CREATE_ALWAYS, 0, 0);
HANDLE hFile = CreateFile(L"e:\\tmp\\y1\\xxx\\assets\\res\\xxx.zip",GENERIC_ALL, 0,0,OPEN_EXISTING,0,0);
if (hFile != INVALID_HANDLE_VALUE)
{
DWORD dwSize = GetFileSize(hFile, 0);

size_t  dwOut = 0;
DWORD dwRead = 0;
LPBYTE p = new BYTE[dwSize];

ReadFile(hFile, p, dwSize, &dwRead, 0);
DWORD dwSignLen = 3;
LPBYTE p2 = (LPBYTE)xxtea_decrypt((void *)(p + dwSignLen), dwSize - dwSignLen,(void *) "key", &dwOut);

WriteFile(hFileOUT, p2, dwOut, &dwRead, 0);
delete p;

CloseHandle(hFileOUT);
CloseHandle(hFile);
}
```

`de_xxx.zip`的头两个字节变成喜闻乐见的`PK`，解压，搞定。

## Luajit ASM

解压出来的文件，都是`LuaJIT`— a Just-In-Time Compiler for Lua，文件头长这样：

![](https://bbs.kanxue.com/upload/attach/201704/55894_6luju5ec1ocl04z.png)

`943K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4#2j5h3A6A6N6q4)9J5k6h3!0J5k6#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2K9s2c8E0L8l9`.`.](https://bbs.kanxue.com/elink@feeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4#2j5h3A6A6N6q4)9J5k6h3!0J5k6#2)9J5c8X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2K9s2c8E0L8l9`.`.)` 可以下载到`LuaJIT`的源码，不过兼容性有点糟糕，需要找到正确的版本才有效。回到`IDA`字符串窗口搜索`luajit`，可以确认目标`APK`使用的是`2.1.0-beta2`。下载源码，编译之后，使用类似下面的命令行：

```
luajit.exe -bl xxx.lua.bytecode xxx.lua.asm
```

可以看到`LUA`的`ASM`代码了，`LUA ASM`长这样：

```
0013    TGETV    4   4   1
0014    CALL     3   4   2
0015    ISNEXT   6 => 0019
0016 => MOV      8   7
0017    MOV      9   2
0018    CALL     8   1   2
0019 => ITERN    6   3   3
0020    ITERL    6 => 0016
0021    RET0     0   1
```

`BYTECODE`的定义在`c1aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6A6K9$3W2Q4x3X3g2D9N6h3q4B7K9i4c8Q4x3X3g2G2M7X3N6Q4x3V1k6n7P5i4c8W2j5$3!0V1k6g2)9J5k6o6u0Q4x3X3f1H3`。不过我智商不够，看不懂，

## Luajit decompiler

搜索了无数反编译文章，基本就是`b0fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5K9h3N6Z5N6p5&6G2M7X3c8Q4x3V1k6D9K9X3b7`.，然而这货2014年之后就再没更新了，并不能用，没有选择，只能自己动手改。

1. 由于新`Opcode`导致的失败

`luajit 2.1.0-beta2`源码的`lj_bc.h`里的`#define BCDEF`，下方就是长长一串`Opcode`定义。

`Ljd`源码的`bytecode\instructions.py` 和 `rawdump\code.py`的两张`Opcode`表必须跟`luajit 2.1.0-beta2` 的`lj_bc.h`一一对比，少了的就要补上。

这里不得不吐槽一下`luajit`，新增`Opcode`其实并没有人用，`Opcode`是通过数组维护的，随便加个新的`Opcode`都会导致原有的`Opcode`编码改变。。。

2. 由于`assert`导致的失败，注释掉就好了。虽说有点不负责，但是反编译各种语法处理我也不懂。

`LJD`改好之后，`main.py xxx.lua.bytecode > xxx.lua`，终于可以看到正常的代码：

```
DeprecatedNetworkClass = {} or DeprecatedNetworkClass

local function deprecatedTip(old_name, new_name)
print("\n********** \n" .. old_name .. " was deprecated please use " .. new_name .. " instead.\n**********")

return
end

DeprecatedNetworkClass.WebSocket = function ()
deprecatedTip("WebSocket", "cc.WebSocket")

return cc.WebSocket
end
_G.WebSocket = DeprecatedNetworkClass.WebSocket()

return
```

完毕！

## 20170417补充说明:

请大家先确认自己的目标，再使用代码。如果版本不一致，你们需要先对比`luajit`源码中的`lj_bc.h`和`e43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5K9h3N6Z5N6p5&6G2M7X3c8Q4x3V1k6D9K9X3c8Q4c8e0N6Q4z5f1q4Q4z5o6b7`.bytecode\instructions.py` 和 `rawdump\code.py`中的`OPCODE`表，对得上，才能正确反编译，对不上就自己参照着来改。

![](https://bbs.kanxue.com/upload/attach/201704/55894_4lqbc00ylpr25pl.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [ljd-master2.1.0b2.7z](javascript:void(0);)

  （84.45kb，2939次下载）
