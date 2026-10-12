
# PE 教程 3: File Header （文件头）

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-3.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-3.htm)

## PE 教程 3: File Header （文件头）

本课我们将要研究 PE header 的 file header（文件头）部分。

至此，我们已经学到了哪些东东，先简要回顾一下:

*   `DOS MZ header` 又命名为 `IMAGE_DOS_HEADER`。其中只有两个域比较重要: `e_magic` 包含字符串"MZ"，`e_lfanew` 包含 PE header 在文件中的偏移量。
*   比较 `e_magic` 是否为 `IMAGE_DOS_SIGNATURE` 以验证是否是有效的 DOS header。比对符合则认为文件拥有一个有效的 DOS header。
*   为了定位 PE header，移动文件指针到 `e_lfanew` 所指向的偏移。
*   PE header 的第一个双字包含字符串"PE\0\0"。该双字与 `IMAGE_NT_SIGNATURE` 比对，符合则认为 PE header 有效。

本课我们继续探讨关于 PE header 的知识。 PE header 的正式命名是 `IMAGE_NT_HEADERS`。再来回忆一下这个结构。

```
IMAGE_NT_HEADERS STRUCT   
    Signature dd ?   
    FileHeader IMAGE_FILE_HEADER <>   
    OptionalHeader IMAGE_OPTIONAL_HEADER32 <>   
IMAGE_NT_HEADERS ENDS
```

*   `Signature`: PE 标记，值为 50h, 45h, 00h, 00h（PE\0\0）。
*   `FileHeader`: 该结构域包含了关于 PE 文件物理分布的一般信息。
*   `OptionalHeader`: 该结构域包含了关于 PE 文件逻辑分布的信息。

最有趣的东东在 `OptionalHeader` 里。不过，`FileHeader` 里的一些域也很重要。本课我们将学习 `FileHeader`，下一课研究 `OptionalHeader`。

```
IMAGE_FILE_HEADER STRUCT   
    Machine WORD ?   
    NumberOfSections WORD ?   
    TimeDateStamp dd ?   
    PointerToSymbolTable dd ?   
    NumberOfSymbols dd ?   
    SizeOfOptionalHeader WORD ?   
    Characteristics WORD ?   
IMAGE_FILE_HEADER ENDS
```

| **Field name** | **Meanings** |
| :--- | :--- |
| **Machine** | 该文件运行所要求的 CPU。对于 Intel 平台，该值是 `IMAGE_FILE_MACHINE_I386` (14Ch)。我们尝试了 LUEVELSMEYER 的 `pe.txt` 声明的 14Dh 和 14Eh，但 Windows 不能正确执行。看起来，除了禁止程序执行之外，本域对我们来说用处不大。 |
| **NumberOfSections** | 文件的节数目。如果我们要在文件中增加或删除一个节，就需要修改这个值。 |
| **TimeDateStamp** | 文件创建日期和时间。我们不感兴趣。 |
| **PointerToSymbolTable** | 用于调试。 |
| **NumberOfSymbols** | 用于调试。 |
| **SizeOfOptionalHeader** | 指示紧随本结构之后的 `OptionalHeader` 结构大小，必须为有效值。 |
| **Characteristics** | 关于文件信息的标记，比如文件是 exe 还是 dll。 |

简言之，只有三个域对我们有一些用:`Machine`, `NumberOfSections` 和 `Characteristics`。通常不会改变 `Machine` 和 `Characteristics` 的值，但如果要遍历节表就得使用 `NumberOfSections`。

为了更好阐述 `NumberOfSections` 的用处，这里简要介绍一下节表。

节表是一个结构数组，每个结构包含一个节的信息。因此若有 3 个节，数组就有 3 个成员。 我们需要 `NumberOfSections` 值来了解该数组中到底有几个成员。 也许您会想检测结构中的全 0 成员起到同样效果。Windows 确实采用了这种方法。为了证明这一点，可以增加 `NumberOfSections` 的值，Windows 仍然可以正常执行文件。据我们的观察，Windows 读取 `NumberOfSections` 的值然后检查节表里的每个结构，如果找到一个全 0 结构就结束搜索，否则一直处理完 `NumberOfSections` 指定数目的结构。 为什么我们不能忽略 `NumberOfSections` 的值? 有几个原因。PE 说明中没有指定节表必须以全 0 结构结束。Thus there may be a situation where the last array member is contiguous to the first section, without empty space at all. Another reason has to do with bound imports. The new-style binding puts the information immediately following the section table's last structure array member. 因此您仍然需要 `NumberOfSections`。

**翻译：****iamgufeng \[**[**Iczelion's Win32 Assembly Homepage**](http://win32asm.cjb.net/)**\]****\[**[**LuoYunBin's Win32 ASM Page**](http://asm.yeah.net)**\]**

4dd466d0e1e0cc123afbe504da2462ca
