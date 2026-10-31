
# [原创]自己写的PE分析工具（附源代码）

以前刚开始学逆向分析的时候，总是找不到一份比较全的代码解析PE格式，最近闲暇无事，写了一个分析工具，里面的代码我现在贴出来，分享一下，希望对想了解PE格式的童鞋有帮助。如果对大家有帮助，希望大家给点鼓励，哈哈

## `MyPE.cpp`

```
#include "stdafx.h"
#include "ParsePE.h"

FILE *pFResult;

int _tmain(int argc, _TCHAR* argv[])
{
    fopen_s(&pFResult, "c://result.txt", "w");
    CParsePE parsePE(L"c:\\notepad.exe");
    parsePE.ParseDosHeader();
    parsePE.ParseNtHeader();
    parsePE.ParseImportTable();
    parsePE.ParseDelayImportTable();
    parsePE.ParseBoundImportTable();
    parsePE.ParseDebugTable();
    parsePE.ParseResouceTable();
    fclose(pFResult);
    return 0;
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
