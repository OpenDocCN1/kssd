
[![GetDecrypted](https://bbs.kanxue.com/upload/attach/202105/855290_5PK3XDB4S6M3PJM.jpg "GetDecrypted")](https://bbs.kanxue.com/upload/attach/202105/855290_5PK3XDB4S6M3PJM.jpg "GetDecrypted")

[![InternalDecrypt](https://bbs.kanxue.com/upload/attach/202105/855290_YZHSQW4NRBZ2UBM.jpg "InternalDecrypt")](https://bbs.kanxue.com/upload/attach/202105/855290_YZHSQW4NRBZ2UBM.jpg "InternalDecrypt")

[![op_Implicit](https://bbs.kanxue.com/upload/attach/202105/855290_BFX4F9U8ZSYS3BV.jpg "op_Implicit")](https://bbs.kanxue.com/upload/attach/202105/855290_BFX4F9U8ZSYS3BV.jpg "op_Implicit")

均直接调用另一个内部的解密函数

[![sub_2161A10](https://bbs.kanxue.com/upload/attach/202105/855290_X6HFM36ZTSZ4ME4.jpg "sub_2161A10")](https://bbs.kanxue.com/upload/attach/202105/855290_X6HFM36ZTSZ4ME4.jpg "sub_2161A10")

## 测试解密

直接Hook `sub_2161A10` 函数并且调用试试看

先进行定义

```cpp
// struct 定义直接照搬就行了, 类型占用大小需要注意一下
struct ACTkByte4
{
    char b1; //0x0
    char b2; //0x1
    char b3; //0x2
    char b4; //0x3
};
struct ObscuredFloat
{
    int32_t currentCryptoKey; //0x0
    int32_t hiddenValue; //0x4
    ACTkByte4 hiddenValueOldByte4; //0x8
    bool inited; //0xC
    float fakeValue; //0x10
    bool fakeValueActive; //0x14
};
// 原型定义
float (*_ObscuredFloat_Decrypted)(ObscuredFloat *) = nullptr;
float ObscuredFloat_Decrypted(ObscuredFloat *self)
{
    return _ObscuredFloat_Decrypted(self);
}
```

hook解密函数, map获取动态库地址的源码太多就不贴上来了

```cpp
DobbyHook((void *)(Il2cppBaseAddr + 0x2161A10), (void *)ObscuredFloat_Decrypted, (void **)&_ObscuredFloat_Decrypted);
```

调用解密函数

```cpp
//内存偏移
template<typename R>
inline R MemoryOff(void *addr, ulong off)
{
    return reinterpret_cast<R>((static_cast<char *>(addr) + off));
}

void (* _xxxDamages)(Il2CppObject *, void *) = nullptr;
void xxxDamages(Il2CppObject *behaviour, void *damageInfo)
{
    auto _damage = MemoryOff<Nullable<ObscuredFloat> *>(damageInfo, 0x34);
    if(_damage->HasValue())
    {
        auto value = ObscuredFloat_Decrypted(_damage->GetValue());
        LOGE("_damage: %0.3f", value);
    }
    return _xxxDamages(behaviour, damageInfo);
}
```

再去找个陷阱撞一下, 正确识别, 收工

[![游戏截图](https://bbs.kanxue.com/upload/attach/202105/855290_WEH6YTPNRFGWPZV.jpg "游戏截图")](https://bbs.kanxue.com/upload/attach/202105/855290_WEH6YTPNRFGWPZV.jpg "游戏截图")

[![logcat](https://bbs.kanxue.com/upload/attach/202105/855290_VQ6DBMKXA683JX6.jpg "logcat")](https://bbs.kanxue.com/upload/attach/202105/855290_VQ6DBMKXA683JX6.jpg "logcat")

[![好耶](https://bbs.kanxue.com/upload/attach/202105/855290_R6KZE6DYD2SWT6H.jpg "好耶")](https://bbs.kanxue.com/upload/attach/202105/855290_R6KZE6DYD2SWT6H.jpg "好耶")

## 用优雅一点的方式Hook

解析B指令获取目标地址然后进行hook

至于怎么解析B指令emmm, 看arm手册就行了, 简单概述的话就是去掉 **指令标志位**

以后有机会再讲一下吧, 这玩意也是个雷, 去年那篇 [Android10 aarch64 `dlopen` Hook](https://bbs.kanxue.com/elink@e71K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9h3S2G2P5h3!0Q4x3X3g2@1L8%4m8Q4x3V1k6S2L8X3c8J5L8$3W2V1x3e0m8Q4x3X3c8S2j5i4u0U0K9o6j5@1i4K6u0V1k6r3I4G2M7r3g2F1i4K6u0V1K9r3!0G2K9H3`.`.) 的解析写法就是有点问题的, 虽然还没触发到那颗雷

```cpp
// 提取B指令的偏移
inline ulong BxxExtract(void *symbol)
{
#if defined(__arm__)
    return static_cast<ulong>((*static_cast<int32_t *>(symbol) << 0x8 >> 0x6));
#elif defined(__arm64__) || defined(__aarch64__)
    return static_cast<ulong>((*static_cast<int32_t *>(symbol) << 0x6 >> 0x4));
#else
#error ABI Error
#endif
}

// 修正B指令转跳
template<typename R>
inline R Amend_Bxx(R symbol, ulong off = 0ul)
{
    symbol = MemoryOff<R>(reinterpret_cast<void *>(symbol), off);
#if defined(__arm__)
    return MemoryOff<R>(reinterpret_cast<void *>(symbol), BxxExtract(reinterpret_cast<void *>(symbol)) + 0x8);
#elif defined(__arm64__) || defined(__aarch64__)
    return MemoryOff<R>(reinterpret_cast<void *>(symbol), BxxExtract(reinterpret_cast<void *>(symbol)));
#else
#error ABI Error
#endif
}

// 地址可通过 主动式Hook 或 被动式Hook 动态获取
DobbyHook((void *)Amend_Bxx(Il2cppBaseAddr + 0x17868F4, 0x4), (void *)ObscuredFloat_Decrypted, (void **)&_ObscuredFloat_Decrypted);
```

# 加密 & 修改 & 其他类型

[![NTM犯法了你知道吗](https://bbs.kanxue.com/upload/attach/202105/855290_SXYPP4PHY95W8FK.jpg "NTM犯法了你知道吗")](https://bbs.kanxue.com/upload/attach/202105/855290_SXYPP4PHY95W8FK.jpg "NTM犯法了你知道吗")

点到为止! 点到为止!

已经讲了很多了, 再多说会被 **♂** 的.

搞懂怎么解密后, 加密

其他类型的也是同样的套路

先定义类型, 然后阿吧阿吧阿吧

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#基础理论](https://bbs.kanxue.com/forum-161-1-117.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
