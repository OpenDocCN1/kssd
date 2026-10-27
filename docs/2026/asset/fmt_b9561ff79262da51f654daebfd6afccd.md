
```cpp
uint8_t *version_watermark = NULL;
uint8_t *owner_watermark = NULL;

void BaseFunction::AddWatermark(Watermark *watermark, int copy_count)
{
    Watermark secure_watermark(NULL);
    std::string value;
    uint8_t *internal_watermarks[] = {version_watermark, owner_watermark};

    for (size_t k = 0; k < 1 + _countof(internal_watermarks); k++) {
        if (k == 0) {
            if (!watermark)
                continue;
        } else {
            uint8_t *ptr = internal_watermarks[k - 1];
            if (!ptr)
                continue;

            uint32_t key = *reinterpret_cast<uint32_t *>(ptr);
            uint16_t len = *reinterpret_cast<uint16_t *>(ptr + 4);
            value.resize(len);
            for (size_t i = 0; i < value.size(); i++) {
                value[i] = ptr[6 + i] ^ static_cast<uint8_t>(_rotl32(key, (int)i) + i);
            }
            secure_watermark.set_value(value);
            watermark = &secure_watermark;
        }

        for (int i = 0; i < copy_count; i++) {
            watermark->Compile();
            ICommand *command = AddCommand(Data(watermark->dump()));
            command->include_option(roCreateNewBlock);
        }
    }
}
```

这段代码定义了一个名为`BaseFunction::AddWatermark`的方法，该方法将特定的“水印”添加到一个对象或数据中。以下是关于这个函数以及代码中每行的简单解释。

首先，定义了两个指向`uint8_t`类型的全局指针，`version_watermark`和`owner_watermark`，即版本水印和用户水印。

```cpp
uint8_t *version_watermark = NULL;
uint8_t *owner_watermark = NULL;
```

`BaseFunction::AddWatermark`函数接收两个参数：一个`Watermark`对象指针和一个表示要复制的次数的整数。

```cpp
void BaseFunction::AddWatermark(Watermark *watermark, int copy_count)
```

然后，创建了一个新的`Watermark`对象`secure_watermark`。并创建了一个字符串`value`用于存放从内部 watermark 中提取和处理的数据。

```cpp
Watermark secure_watermark(NULL);
std::string value;
```

接下来创建了一个内部 watermark 数组`internal_watermarks`，它包含了之前定义的全局 watermark 指针。

```cpp
uint8_t *internal_watermarks[] = {version_watermark, owner_watermark};
```

这里的代码遍历所有 watermark（包括输入的 watermark 和内部的 watermark）。对于每个 watermark，如果它是 NULL，那么就直接跳过当前迭代。

```cpp
for (size_t k = 0; k < 1 + _countof(internal_watermarks); k++) {
    if (k == 0) {
        if (!watermark)
            continue;
    } else {
        uint8_t *ptr = internal_watermarks[k - 1];
        if (!ptr)
            continue;
```

取出内部 watermark，将它的数据作为 32 位和 16 位的块重新解释（也就是把原始的数据直接视作这些数值）。然后，它用一个特殊的算法（对每个字节都执行异或，然后添加到索引上），在`value`中生成新的字符。

```cpp
uint32_t key = *reinterpret_cast<uint32_t *>(ptr);
uint16_t len = *reinterpret_cast<uint16_t *>(ptr + 4);
value.resize(len);
for (size_t i = 0; i < value.size(); i++) {
    value[i] = ptr[6 + i] ^ static_cast<uint8_t>(_rotl32(key, (int)i) + i);
}
secure_watermark.set_value(value);
watermark = &secure_watermark;
```

最后，它为每个 watermark 创建`copy_count`个复制项，编译每个复制项，并将其添加到`ICommand`中，随后设置创建新块的数据选项标志。

```cpp
for (int i = 0; i < copy_count; i++) {
    watermark->Compile();
    ICommand *command = AddCommand(Data(watermark->dump()));
    command->include_option(roCreateNewBlock);
}
}
```

这段代码的主要目的是为了处理和添加 watermark 到数据中。它首先处理传入的 watermark，然后处理定义的内部 watermark。处理的方式包括从中提取数据，以特定方式编辑这些数据，并设置`secure_watermark`或新增 watermark 的值。然后为这些 watermark 创建指定数量的复制项，并添加到命令中。

其中的`secure_watermark.set_value(value);`设定水印内容，之后使用这个实例进行水印的添加（见上方），我们可以通过获取这个实例的 this 指针来获取水印内容。

然而在后期测试的时候，`BaseFunction::AddWatermark`实际上是被虚拟机保护了，但`Watermark`类并没有被虚拟机保护，其中的`Watermark::ReadFromIni`函数特征及其鲜明，这意味着我们可以通过 dump 下来的程序，抓住函数地址并直接读取水印内容，见下文。

注：必须要用 dump 后的程序分析，因为 VMP 似乎只有在运行时才会解密字符串。

```cpp
// 在 IDA 中可以通过搜索dump后的程序内字符串，快速锁定这个函数，如 Name%d
// 快捷键: Shift + F12
void Watermark::ReadFromIni(IniFile &file, size_t id)
{
    id_ = id;
    name_ = file.ReadString("Watermarks", string_format("Name%d", id).c_str());
    value_ = file.ReadString("Watermarks", string_format("Value%d", id).c_str());
    use_count_ = file.ReadInt("Watermarks", string_format("UseCount%d", id).c_str());
    enabled_ = file.ReadBool("Watermarks", string_format("Enabled%d", id).c_str(), true);
    Compile(); // Watermark::Compile()
}
```

```cpp
void Watermark::Compile()
{
    dump_.clear();
    mask_.clear();

    if (value_.size() == 0)
        return;

    for (size_t i = 0; i < x.size(); i++) {
        size_t p = i / 2;
        if (p >= dump_.size()) {
            dump_.push_back(0);
            mask_.push_back(0);
        }

        uint8_t m = 0xff;
        uint8_t b;
        char c = value_[i]; // 可以从这里直接抓取水印的内存地址，然后直接读取
        if ((c >= '0') && (c <= '9')) {
            b = c - '0';
        } else if ((c >= 'A') && (c <= 'F')) {
            b = c - 'A' + 0x0a;
        } else if ((c >= 'a') && (c <= 'f')) {
            b = c - 'a' + 0x0a;
        } else {
            m = 0;
            b = rand();
        }

        if ((i & 1) == 0) {
            dump_[p] = (dump_[p] & 0x0f) | (b << 4);
            mask_[p] = (mask_[p] & 0x0f) | (m << 4);
        } else {
            dump_[p] = (dump_[p] & 0xf0) | (b & 0x0f);
            mask_[p] = (mask_[p] & 0xf0) | (m & 0x0f);
        }
    }
}
```

## 实际调试

这里就是我干活的地方了（

掏出带有反反调试器的 x64dbg，直接 逆向，启动！()

![Alt text](https://bbs.kanxue.com/upload/attach/202312/987680_XV93GP3JHUZSMCG.png)

使用自带的 Scylla 把 VMP 主程序 dump 出来

![Alt text](https://bbs.kanxue.com/upload/attach/202312/987680_CA2RAGQ4NX3S77E.png)

在 IDA 内搜索特征字符串，找到`Watermark::Compile()`函数

![Alt text](https://bbs.kanxue.com/upload/attach/202312/987680_SXQU5CPZQH73E36.png)

![Alt text](https://bbs.kanxue.com/upload/attach/202312/987680_6FPR4ED5YMCC6A2.png)

解释后的伪代码`Watermark::Compile()`函数如上所示。
