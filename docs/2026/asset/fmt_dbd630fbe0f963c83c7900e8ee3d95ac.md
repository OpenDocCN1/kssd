
```
NTSTATUS

RtlpQueryRegistryDirect(

    IN ULONG ValueType, //键的类型

    IN PVOID ValueData,//从ZwQueryValueKey KeyValueInformation 获取到的键值数据

    IN ULONG ValueLength,//数据的长度

    IN OUT PVOID Destination //是PRTL_QUERY_REGISTRY_TABLE结构体的EntryContext成员

)

{

    if (ValueType == REG_SZ ||

        ValueType == REG_EXPAND_SZ ||

        ValueType == REG_MULTI_SZ

        ) {

        PUNICODE_STRING DestinationString;
        USHORT TruncValueLength;

        //

        // Truncate ValueLength to be represented in a UNICODE_STRING

        //

        if (ValueLength <= MAX_USTRING) {
            TruncValueLength = (USHORT)ValueLength;
        }
        else {
            TruncValueLength = MAX_USTRING;

#if DBG

            DbgPrint("RtlpQueryRegistryDirect: truncating SZ Value length: %x -> %x\n",
                ValueLength, TruncValueLength);

#endif //DBG
        }
        DestinationString = (PUNICODE_STRING)Destination;
            if (DestinationString->Buffer == NULL) {

            DestinationString->Buffer = RtlAllocateStringRoutine(TruncValueLength);
            if (!DestinationString->Buffer) {
                return STATUS_NO_MEMORY;
            }
            DestinationString->MaximumLength = TruncValueLength;
        }
        else if (TruncValueLength > DestinationString->MaximumLength) {
            return STATUS_BUFFER_TOO_SMALL;
        }

        RtlCopyMemory(DestinationString->Buffer, ValueData, TruncValueLength);
        DestinationString->Length = (TruncValueLength - sizeof(UNICODE_NULL));

    }//传入的数据大于sizeof(ULONG)，所以也不会进入这里
    else if (ValueLength <= sizeof(ULONG)) {
        if (Destination != ValueData && ValueLength)
            RtlCopyMemory(Destination, ValueData, ValueLength);

    }
    else {
        PULONG DestinationLength;
        //在这里将（PVOID）Destination转为PULONG类型    
        DestinationLength = (PULONG)Destination;
        //
        if ((LONG)* DestinationLength < 0) {
            ULONG n = -(LONG)* DestinationLength;

            if (n < ValueLength) {
                return STATUS_BUFFER_TOO_SMALL;
            }
            RtlCopyMemory(DestinationLength, ValueData, ValueLength);

        }
        else {
            if (*DestinationLength < (2 * sizeof(*DestinationLength) + ValueLength)) {
                return STATUS_BUFFER_TOO_SMALL;
            }

            *DestinationLength++ = ValueLength;
            *DestinationLength++ = ValueType;
            RtlCopyMemory(DestinationLength, ValueData, ValueLength);
        }
    }

    return STATUS_SUCCESS;
}
```

假设我们的注册表类型属于`REG_SZ`，那么我们的`（PVOID）Destination`所指内存会被初始化为`UNICODE_STRING`结构体，结合开头提到的栈分析，`Destination`所指向的内存是4字节（且距第一个结构体开始位置仅为12字节），而`UNICODE_STRING`结构体大小需要16字节，这16个字节包含有后3个结构体`Destination`所指向的内存，比对开头的栈分析可以更加明显看出，溢出的位置。下图为溢出后的栈，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201907/755181_QJQ7MMRTAN893BJ.png)

## 漏洞利用

通过之前的分析，如果将第一个要获取的键，键类型改为`REG_SZ`，会覆盖后三个`PRTL_QUERY_REGISTRY_TABLE`结构体的`EntryContext`成员所指内存。如果记得前面介绍`RtlpQueryRegistryDirect`函数拷贝数据的三种方式，其中最后一种方式会判断`EntryContext`成员所指内存内容数值是否大于当前获取到的键值长度，如果是，那么会将数据直接拷贝到`EntryContext`所指内存上，在次结合上图的栈分析看到溢出部分恰好属于第二个结构体`EntryContext`所指内容，这个时候在采集第二个结构体键值时，满足于`RtlpQueryRegistryDirect`函数第三种拷贝方式，直接拷贝，会将第二个注册表键值直接拷贝到`EntryContext`所指内存地址上。

由前面可知：

1. `PRTL_QUERY_REGISTRY_TABLE`每个结构体大小为`0x38`。
2. 标志位为`0x20`会将数据拷贝到`EntryContext`成员所指内存。
3. 当注册表键无法打开或者读取时，会将默认数据（`PRTL_QUERY_REGISTRY_TABLE`下的`DefaultData`成员）拷贝到`PRTL_QUERY_REGISTRY_TABLE`下的`EntryContext`成员所指内存，大小由`PRTL_QUERY_REGISTRY_TABLE`下的`DefaultLength`控制。

现在将第二个注册表的键类型定义为`REG_BINARY`，键值内容，填充0对齐到`PRTL_QUERY_REGISTRY_TABLE`结构体，其中`PRTL_QUERY_REGISTRY_TABLE`下的`EntryContext`（目的地址）、`DefaultData`（源地址）、`DefaultLength`（数据长度），由我们控制，当栈溢出发生后完成第三四个结构体内容的覆盖，从而对写任意地址。构造好的数据（地址由于多次调试以不在和上述图地址匹配），如下图所示：

![](https://bbs.kanxue.com/upload/attach/201907/755181_VE2TGWCBDZTW497.png)

## 参考文献

> 《exploit 远程桌面服务驱动》—— 猪会被杀掉

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-7-8 10:13

被dx苹果的心愿编辑，原因： 图片丢死
