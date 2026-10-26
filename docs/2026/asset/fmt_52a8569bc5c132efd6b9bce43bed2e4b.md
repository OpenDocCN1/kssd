
si.lic 文件中 `ActId` 字段的值为 `Deferred`，无法通过 `call sourceinsight4.403570` 的 ActId 的检验。进入到 `sourceinsight4.403570` 函数内部后，`sub_403240()` 会返回一张存有很多 ActId 的表，那 ActId 表的第一项和 si.lic 文件中 ActId 字段的值进行比较。这里我们直接将软件内的 ActId 拷贝到 si.lic 文件中。

### 比较 ActId 字段的值的代码

```c
int __cdecl CheckActId(char *pData, int a2, size_t Size_4, int n_x32, int n_1B7F)
{
    int v5; // esi
    void *ppTable; // ebp
    int v7; // edi

    v5 = 0;
    ppTable = sub_403240(a2, Size_4, n_x32, n_1B7F); // 返回了存有很多的 ActId 的表
    if (n_x32 <= 0)
    {
    LABEL_4:
        sub_425090(ppTable);
        ReleaseBlock(ppTable);
        return 0;
    }
    else
    {
        while (1)
        {
            v7 = CheckActIdHeaderFourCharacter(*(ppTable + v5), Size_4, pData);
            if (v7 == Size_4) // 只比较前面四个字符
                break;
            if (++v5 >= n_x32)
                goto LABEL_4;
        }
        sub_425090(ppTable);
        ReleaseBlock(ppTable);
        return v7;
    }
}
```

修改 si.lic 中的 `ActId` 字段。

```ini
ActId="673A44D35B3608E5C603D775C76216F555E000D04B6718E33E93F35887A8A360D2F468E313DC7B3E047E08F10A51B75561B5L55576B63BF2D7750F09557AF661F14849F94F2652A903A10E9074E61EA4FE7E83A6F5090AD61F3F365D1C67DA22A478FA17"
```

# 四、si.lic 文件检验码验证

读取 si.lic 文件的内容，去除掉空白字符，通过 `call sourceinsight4.403210` 生成校验码1，然后再使用去除掉空白字符的文件内容和 `Value` 字段的值通过 `call sourceinsight4.402F00` 函数生成校验码2，对比两个校验码。相同则验证通过，否则验证失败。
