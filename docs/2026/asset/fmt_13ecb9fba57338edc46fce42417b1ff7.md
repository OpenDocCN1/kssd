

```cpp
_BYTE *__fastcall decodeLuaData(cocos2d::Data *a1, int *size)
{
    _BYTE *v3; // r0
    _BYTE *v4; // r5
    int v6; // r0
    int v7; // r2
    int v8; // r3
    bool v9; // cc
    unsigned int v10; // r1
    _BYTE *v11; // r12
    _BYTE *v12; // r0
    char v13; // t1
    _BYTE *v14; // r0
    char v15[4]; // [sp+0h] [bp-18h]

    v3 = (_BYTE *)cocos2d::Data::getBytes(a1);
    v4 = v3;
    if (*size > 8 && *v3 == 'a' && v3[1] == 'b' && v3[2] == 'c' && v3[3] == 'd')
    {
        v6 = _hexToDecimal(v3 + 4, 4);
        v7 = *size;
        v8 = 0;
        v9 = *size <= 8;
        v15[3] = v6;
        v15[2] = (unsigned __int16)(v6 - 2048) >> 8;
        v10 = (unsigned int)(v6 - 2048) >> 24;
        v15[1] = (unsigned int)(v6 - 2048) >> 16;
        v15[0] = (unsigned int)(v6 - 2048) >> 24;

        if (!v9)
        {
            v11 = v4 - 1;
            v12 = v4 + 7;
            while (1)
            {
                v13 = *++v12;
                ++v8;
                *++v11 = v13 ^ v10;
                v7 = *size;

                if (*size <= v8 + 8)
                    break;

                LOBYTE(v10) = v15[v8 & 3];
            }
        }

        v14 = &v4[v7 - 8];
        *v14 = 0;
        v14[1] = 0;
        v14[2] = 0;
        v14[3] = 0;
        v14[4] = 0;
        v14[5] = 0;
        v14[6] = 0;
        v14[7] = 0;
        *size -= 8;
    }

    return v4;
}
```

此方法首先获取文件数据，而后判断文件头，文件头正是`abcd`。

可以肯定这就是解密`.lua`的算法。

如果不确定，我们可以使用`Frida`进行`Hook`验证猜想。

我这里就不`Hook`了。

## 算法还原

要实现算法的还原，我们就要依据伪代码或是汇编代码翻译成可用代码，可以发现`IDA`所反汇编的伪代码中包含了一些特定的宏，例如：`LOBYTE`。

要了解这些宏的用途，我们可以参考`ida_root/plugins/def.h`。

我这里使用了`Python`：

```python
import struct
from loguru import logger


def _hexToDecimal(data, offset):
    if offset <= 0:
        return 0
    return struct.unpack(">I", data[:offset])[0]


def LOBYTE(d):
    return d & 0xff


def decodeLuaData(data, size):
    v4 = bytearray(data)
    v15 = [0] * 4

    if (
        size > 8
        and data[0] == ord('a')
        and data[1] == ord('b')
        and data[2] == ord('c')
        and data[3] == ord('d')
    ):
        logger.info("find special exts")
        v6 = _hexToDecimal(data[4:], 4)

        v15[3] = v6
        v15[2] = (v6 - 2048) >> 8
        v10 = (v6 - 2048) >> 24
        v15[1] = (v6 - 2048) >> 16
        v15[0] = (v6 - 2048) >> 24

        if size > 8:
            i = 0
            while 1:
                v13 = v4[8 + i]
                v4[i] = v13 ^ v10
                i += 1

                if size <= i + 8:
                    break

                v10 = LOBYTE(v15[i & 3])

            size -= 8

        return v4[:-8]
```

解密后，发现还是`LuaJit`，那么直接类同`.lua64`反汇编即可。

![image-20210724204751819](https://bbs.kanxue.com/upload/attach/202107/883754_8CJJ7QQK6KP9TCF.png)

# Lua资源加载流程

回到`Lua`文件加载流程分析时提到的`LuaStack::init()`方法中，调用了`register_all_cocos2dx`。

