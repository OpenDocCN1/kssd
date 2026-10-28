
```c
// 宽是1 窄是0  DWORD 查找指定字节(ULONG64 地址, INT8 字节, INT8 宽与窄)
{
    INT i = 0;
    INT 字节1 = 0;
    if (宽与窄) // 宽是1
    {
        if (!字节)
        {
            while (*((INT8*)地址 + i * 2) != 0)
            {
                i++;
            }
            return i*2;
        }
        do
        {
            if (*((INT8*)地址 + i * 2) >= 'a')
                字节1 = *((INT8*)地址 + i * 2) - 0x20;
            else
                字节1 = *((INT8*)地址 + i * 2);
            if (字节1 == 字节)
            {
                return i * 2;
            }
            i++;
        } while (*((INT8*)地址 + i * 2) != 0);
    }
    else { // 窄是0
        if (!字节)
        {
            while (*((INT8*)地址 + i) != 0)
            {
                i++;
            }
            return i;
        }
        do
        {
            if (*((INT8*)地址 + i) >= 'a')
                字节1 = *((INT8*)地址 + i) - 0x20;
            else
                字节1 = *((INT8*)地址 + i);
            if (字节1 == 字节)
            {
                return i;
            }
            i++;
        } while (*((INT8*)地址 + i) != 0);
    }
    return 0;
}
// 返回的是长度 如果是0 就是失败
BOOLEAN 宽到窄字符(ULONG64 宽地址, ULONG64 窄地址, WORD 长度)
{
    WORD i = 0;
    BYTE* 窄节字 = NULL;
    while (*((BYTE*)(宽地址 + i * 2)))
    {
        窄节字 = (BYTE*)窄地址 + i;
        *窄节字 = *((BYTE*)(宽地址 + i * 2));
        i++;
        if (i >= 长度)
        {
            return 1;
        }
    }
    return 0;
}
// 宽字符比较窄字符 按窄字符长度标准
BOOLEAN 宽比较窄字符(ULONG64 s1, ULONG64 s2, WORD 长度)
{
    WORD i = 0;
    BYTE 字节, 字节1;
    if (*((BYTE*)s1 + i * 2) >= 'a')
        字节 = *((BYTE*)s1 + i * 2) - 0x20;
    else
        字节 = *((BYTE*)s1 + i * 2);
    if (*((BYTE*)s2 + i) >= 'a')
        字节1 = *((BYTE*)s2 + i) - 0x20;
    else
        字节1 = *((BYTE*)s2 + i);
    while ((字节 ^ 字节1) == 0)
    {
        if (*((BYTE*)s1 + i * 2) >= 'a')
            字节 = *((BYTE*)s1 + i * 2) - 0x20;
        else
            字节 = *((BYTE*)s1 + i * 2);
        if (*((BYTE*)s2 + i) >= 'a')
            字节1 = *((BYTE*)s2 + i) - 0x20;
        else
            字节1 = *((BYTE*)s2 + i);
        i++;
        if (长度 == i)
        {
            return 1;
        }
    }
    return 0;
}

void 解析模块名(char* 模块名, char* 正确模块名)
{
    // x64
    ULONG64 ApiSetSchema地址 = *(ULONG64*)(__readgsqword(0x60) + 0x68);
    WORD 版本 = *(DWORD*)ApiSetSchema地址;
    WORD API集数量 = 0;
    WORD API集_命名空间条目_偏移量 = 0;
    WORD API集名称_偏移量 = 0;
    ULONG64 API集名称_地址 = 0;
    WORD API集名称_长度 = 0;
    WORD API集_值数组_偏移量 = 0;
    ULONG64 API集_值数组_地址 = 0;
    WORD API集_原名称_偏移量 = 0;
    WORD API集_原名称_长度 = 0;
    ULONG64 API集_原名称_地址 = 0;
    ULONG64 API集遍历数 = 0;
    WORD 模块名长度 = 查找指定字节((ULONG64)模块名, 0, 0);
    WORD API集版本_偏移 = 0;
    WORD API集数量_偏移 = 0;
    WORD API集命名空间条目_偏移量_偏移 = 0;
    WORD API集命名空间条目_大小 = 0;
    WORD API集名称_长度_偏移 = 0;
    WORD API集名称_偏移量_偏移 = 0;
    WORD API集_值数组_地址_偏移 = 0;
    WORD API集_原名称_偏移量_偏移 = 0;
    WORD API集_原名称_长度_偏移 = 0;
    WORD API集名称_地址_偏移 = 0;
    WORD 模块名长度_偏移 = 0;

    if (!模块名长度)
    {
        return;
    }
    else
    {
        // -4 是因为.dll 字节大小是4
        // 在API集名称里面 没有.dll 是为了节省字节内存
        模块名长度 -= 4;
    }
    if (版本 == 6 || 版本 == 5)
    {
        API集版本_偏移 = 0x10;
        API集数量_偏移 = 0xC;
        API集命名空间条目_偏移量_偏移 = 0x10;
        API集命名空间条目_大小 = 0x18;
        API集名称_长度_偏移 = 0x8;
        API集名称_偏移量_偏移 = 0x4;
        API集_值数组_地址_偏移 = 0x8;
        API集_原名称_偏移量_偏移 = 0x4;
        API集_原名称_长度_偏移 = 0x8;
        API集名称_地址_偏移 = 0;
        模块名长度_偏移 = 0;
        API集_命名空间条目_偏移量 = *(WORD*)(ApiSetSchema地址 + API集命名空间条目_偏移量_偏移);
        goto 解析;
    }
    else if (版本 == 3 || 版本 == 4)
    {
        API集版本_偏移 = 0x14;
        API集数量_偏移 = 0xC;
        API集命名空间条目_偏移量_偏移 = 0x10;
        API集命名空间条目_大小 = 0x18;
        API集名称_长度_偏移 = 0x8;
        API集名称_偏移量_偏移 = 0x4;
        API集_值数组_地址_偏移 = 0x8;
        API集_原名称_偏移量_偏移 = 0x4;
        API集_原名称_长度_偏移 = 0x8;
        API集名称_地址_偏移 = 0;
        模块名长度_偏移 = 0;
        API集_命名空间条目_偏移量 = ApiSetSchema地址 + API集命名空间条目_偏移量_偏移;
        goto 解析;
    }
    else if (版本 == 2)
    {
        API集版本_偏移 = 0x8;
        API集数量_偏移 = 0x4;
        API集命名空间条目_偏移量_偏移 = 0x8;
        API集命名空间条目_大小 = 0xC;
        API集名称_长度_偏移 = 0x4;
        API集名称_偏移量_偏移 = 0;
        API集_值数组_地址_偏移 = 0x4;
        API集_原名称_偏移量_偏移 = 0;
        API集_原名称_长度_偏移 = 0x4;
        API集名称_地址_偏移 = 0x4;
        模块名长度_偏移 = 0x4;
        API集_命名空间条目_偏移量 = API集命名空间条目_偏移量_偏移;
        goto 解析;
    }
    else
    {
        goto 无API集;
    }

无API集:
    memmove(正确模块名, 模块名, 模块名长度 + 4);
    return;

解析:
    API集数量 = *(WORD*)(ApiSetSchema地址 + API集数量_偏移);
    WORD i = 0;
继续遍历:
    // 循环检测有没有一致的 API集名称长度
    do
    {
        API集遍历数 = i * API集命名空间条目_大小;
        API集名称_长度 = *(BYTE*)(ApiSetSchema地址 + API集_命名空间条目_偏移量 + API集遍历数 + API集名称_长度_偏移) / 2;
        if (API集名称_长度 == (模块名长度 - 模块名长度_偏移))
        {
            break;
        }
        i++;
    } while (i <= API集数量);
    if (i > API集数量) // 自己循环完毕就是没有 API集
    {
        goto 无API集;
    }
    API集名称_偏移量 = *(WORD*)(ApiSetSchema地址 + API集_命名空间条目_偏移量 + API集遍历数 + API集名称_偏移量_偏移);
    API集名称_地址 = ApiSetSchema地址 + API集名称_偏移量;
    // 检测 字符串是不是一样！ 不是一样的话就去 循环10 结束函数
    if (!宽比较窄字符(API集名称_地址, (ULONG64)模块名 + API集名称_地址_偏移, API集名称_长度))
    {
        i++;
        goto 继续遍历;
    }
    // 6.3 版本是 API集版本_偏移=0x14 , 10 版本是API集版本_偏移=0x10
    API集_值数组_偏移量 = *(WORD*)(ApiSetSchema地址 + API集_命名空间条目_偏移量 + API集遍历数 + API集版本_偏移);
    API集_值数组_地址 = ApiSetSchema地址 + API集_值数组_偏移量 + API集_值数组_地址_偏移;
    API集_原名称_偏移量 = *(WORD*)(API集_值数组_地址 + API集_原名称_偏移量_偏移);
    API集_原名称_长度 = *(BYTE*)(API集_值数组_地址 + API集_原名称_长度_偏移);
    API集_原名称_地址 = ApiSetSchema地址 + API集_原名称_偏移量;
    if (!API集_原名称_长度)
    {
        API集_原名称_长度 = *(BYTE*)(API集_值数组_地址 + API集_原名称_长度_偏移 + 0x8);
        if (!API集_原名称_长度)
        {
            goto 无API集;
        }
        API集_原名称_偏移量 = *(WORD*)(API集_值数组_地址 + API集_原名称_偏移量_偏移 + 0x8);
        API集_原名称_地址 = ApiSetSchema地址 + API集_原名称_偏移量;
    }
    goto 结束;

结束:
    if (宽到窄字符(API集_原名称_地址, (ULONG64)正确模块名, API集_原名称_长度 / 2))
    {
        return;
    }
    else
    {
        memmove(正确模块名, 模块名, 模块名长度 + 4);
        return;
    }
}
```
