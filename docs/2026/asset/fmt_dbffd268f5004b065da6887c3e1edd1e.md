
```
                // 符号信息包括符号在节内的偏移地址、加载地址及符号出处等，其格式如下：

                // Address                        Publics by Value                        Rva+Base        Lib:Object

                // 0000:00000003        \_\_\_safe\_se\_handler\_count        00000003        <absolute>

                // 或者搜索静态符号

                if ((strstr(linestr, "Publics by Value") != NULL && strstr(linestr, "Rva+Base") != NULL) ||

                        (strstr(linestr, "Static symbols") != NULL ))

                {

                        // 如果搜索到，则标注为开始

                        bBegin = TRUE;

                        continue;

                }

                char StrLine[512] = {0};

                // 将linestr去掉空格后放到StrLine中，如果该行是空行，则跳过

                if (!Trim(linestr, StrLine))

                {

                        continue;

                }

                // 开始分析StrLine符号行数据信息

                if (bBegin)

                {

                        // 这里的地址指的是文件中的地址

                        char StrAddress[16] = {0};        // 段: 文件偏移

                        char StrSymbol[512] = {0};        // 符号名

                        char StrVirtualAddr[16] = {0};        // 虚拟地址

                        char StrLibObject[512] = {0};        // 输入文件名

                        int Segment = 0;

                        int FileOffset = 0;

                        // 复制符号地址，如：0001:00000003

                        memcpy(StrAddress, StrLine, 13);

                        int nstrlen = strlen(StrLine);

                        memmove(StrLine, StrLine + 13, nstrlen - 13 + 1);        // 跳过段偏移

                        // 获取符号所在的段与文件偏移

                        GetFileAddr(StrAddress, &Segment, &FileOffset);

                        // 段是不能为0的

                        if (Segment == 0)

                                continue;

                        // 将剩余内容去掉空格

                        Trim(StrLine, StrLine);

                        // 复制符号名

                        strcpy\_s(StrSymbol, 512, StrLine);

                        // 符号名结束

                        \*strchr(StrSymbol, ' ') = '\0';

                        // 计算符号名的长度

                        int nlen = strlen(StrSymbol);

                        // 计算剩余部分的长度

                        nstrlen = strlen(StrLine);

                        memmove(StrLine, StrLine + nlen, strlen(StrLine) - nlen + 1);        // 跳过符号名

                        // 去除空格

                        Trim(StrLine, StrLine);

                        // 获取符号的虚拟地址

                        memcpy(StrVirtualAddr, StrLine, strchr(StrLine, ' ') - StrLine);

                        nstrlen = strlen(StrLine);

                        memmove(StrLine, StrLine + 8, nstrlen - 8 + 1);                // 跳过VirtualAddress

                        nstrlen = strlen(StrLine);

                        memmove(StrLine, StrLine + 5, nstrlen - 5 + 1);        // 跳过f i ...

                        // 输入文件名

                        strcpy(StrLibObject, StrLine);

                        // 分配符号信息结构，将获取的符号信息保存起来

                        MapStructrue \* mapstruct = new MapStructrue;

                        mapstruct->Segment = Segment;                                // 段

                        mapstruct->Offset = FileOffset;                                // 偏移

                        strcpy(mapstruct->SymbleName, StrSymbol);                // 符号名

                        mapstruct->VirtualAddress = StringToHex(StrVirtualAddr);        // 线性地址

                        strcpy(mapstruct->LibObject, StrLibObject);                // 输入文件名

                        // 添加文件的符号信息

                        MapVector.push\_back(mapstruct);

                }

        }

        // 关闭文件

        fclose(mapfile);

        mapfile = NULL;

        // 遍历MapVector中的所有符号信息

        for (vector<MapStructrue\*>::iterator itr = MapVector.begin(); itr != MapVector.end(); itr++)

        {

                MapStructrue \* stu = \* itr;

                // 获取符号段、文件偏移信息

                int a = stu->Segment;

                int b = stu->Offset;

                int nlen = strlen("\_\_ehhandler$");

                // 如果符号名存在\_\_e...，则将其截掉

                if (strstr(stu->SymbleName, "\_\_ehhandler$"))

                        memmove(stu->SymbleName, stu->SymbleName + nlen, strlen(stu->SymbleName) - nlen + 1);

                // 将修饰后名称转换成函数签名，失败则直接跳出循环

                // 注： 由于不同的编译器采用不同的名字修饰方法，必然会导致由不同编译器编译产生的目标

                //                文件无法正常相互链接，这是导致不同编译器之间不能互操作的主要原因之一

                if (!UnDecorateSymbolName(stu->SymbleName, stu->SymbleName, 512, 0xFFF))        // no UNDNAME\_COMPLETE

                {

                        // UnDecorateSymbolName failed

                        DWORD error = GetLastError();

                        char strerr[255] = {0};

                        sprintf(strerr, "解析符号名时出现错误 %d\n", error);

                        MsgBox(strerr, MB\_ICONERROR);

                        break;

                }

                // 输出符号信息，主要为了调试

                OutPutStr("%s %08X %s\n", stu->SymbleName, stu->VirtualAddress, stu->LibObject);

        }

}

/\*\*

\* 根据函数名funcname获取相应的符号信息

\*

\* funcname：欲获取符号信息的函数(符号)名

\*

\* 返回值：找到相应符号信息则返回，否则返回NULL

\*/

MapStructrue\* CPEStructure::GetMap(char\* funcname)        // \*

{

        // 遍历文件所有符号信息

        for (vector<MapStructrue\*>::iterator itr = MapVector.begin(); itr != MapVector.end(); itr++)

        {

                MapStructrue \* stu = \* itr;

                // 检测是否是欲查找符号信息

                if (\_stricmp(stu->SymbleName, funcname) == 0)

                        return stu;

        }

        return NULL;

}

/\*\*

\* 为文件新区块分配一个虚拟区间，并返回其虚拟区间起始地址

\*/

DWORD CPEStructure::GetNewSection()                // \*

{

        // 获取PE文件的区块数

        DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;

        return PEAlign(image\_section\_header[SectionNum - 1].VirtualAddress + image\_section\_header[SectionNum - 1].Misc.VirtualSize,

                        image\_nt\_headers.OptionalHeader.SectionAlignment);

}

//----------------------------------------------------------------

// return values:

// 0 - no room for a new section

// 1 - file already encrypted

// else: returns a pointer to the IMAGE\_SECTION\_HEADER struct of the new section

/\*\*

\* 在PE文件中添加新的区块

\*

\* Base：新区块的数据基址

\* len：新区块的数据长度

\* SectionName：新区块的名称

\*

\* 返回值：成功返回新区块的结构信息，否则NULL

\*/

PIMAGE\_SECTION\_HEADER CPEStructure::AddSection(char \* Base, int len, char \* SectionName)        // \*

{

        DWORD newSectionOffset;

        // 获取PE文件区块数

        DWORD SectionNum = image\_nt\_headers.FileHeader.NumberOfSections;

        // 计算新区块在区块表中的偏移

        newSectionOffset = dwRO\_first\_section + image\_nt\_headers.FileHeader.NumberOfSections \* sizeof(IMAGE\_SECTION\_HEADER);

        // 检测是否有足够空间在放置新区块

        if (image\_nt\_headers.OptionalHeader.SizeOfHeaders < (newSectionOffset + sizeof(IMAGE\_SECTION\_HEADER)))

        {

                return NULL;

        }

        // 创建一个新的区块

        // 遍历前面所有区块，设置每个区块为可写状态

        for (DWORD i = 0; i < (SectionNum - 1); i++)

        {

                image\_section\_header[i].Characteristics |= IMAGE\_SCN\_MEM\_WRITE;

        }

        // 创建一个新的区块，将最后区块表中的记录复制作为新区块的记录

        CopyMemory(&image\_section\_header[SectionNum], &image\_section\_header[SectionNum - 1], sizeof(IMAGE\_SECTION\_HEADER));

        // 设置新区块的虚拟地址

        image\_section\_header[SectionNum].VirtualAddress = PEAlign(image\_section\_header[SectionNum - 1].VirtualAddress +

                image\_section\_header[SectionNum - 1].Misc.VirtualSize, image\_nt\_headers.OptionalHeader.SectionAlignment);

        // 设置新区块的长度

        image\_section\_header[SectionNum].Misc.VirtualSize = len;

        image\_section\_header[SectionNum].SizeOfRawData = PEAlign(image\_section\_header[SectionNum].Misc.VirtualSize,
```
