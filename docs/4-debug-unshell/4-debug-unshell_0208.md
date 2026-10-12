
# Remote Administrator v2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1083.htm`](https://www.pediy.com/kssd/pediy03/forum1083.htm)

`Remote Administrator` v2.0   

`http://www.famatech.com/radmin20.zip` : size 1,482,243 bytes   

`Radmin.exe` : size 1,032,192 bytes   

你是怎么知道它是用 `Armadillo` 加的壳？`fiv245` 报是 `VC6` 的，但是它的确是经过了处理，`OEP` 是 `ACEBFC`。   

加壳程序并不像其它的把被还原出来数据放到已预定下来的 section 里，而是放在动态分配出来的空间，   

所以 `pedump` dump 出来的 exe 也不过是加壳还原程序，这真的让我们一下子束手无策！   

`bpmb ds:acebfc w`，发现 1401422 的 `CALL 1401BE0` 就是还原代码到 `OEP` 的。   

再细看加壳还原程序就知道有六个 section 通过上面的 call 被还原了。   

分别是:   

```
RVA    size   alignment
1000   3822A    39000
3A000  4C7A      5000
3F000  1E13C    1F000
5E000  447C      5000
63000    400      1000
64000 11C18D    11D000
```

`[0140FE30]` 存放原 image base : `AA0000`   

往上看在 `140137E` 有 `CALL 1401BE0`，这里是“还原”`PIMAGE_NT_HEADERS32`，好像什么都有了，   

将 `AA0000` 到 `AA0000 + 64000 + 11D000` (section alignment 是 1000) dump 出来不就行了吗？   

唔？没有图标？用 `Peditor` 看 section table，才五个 section，什么都是空的，看 directory 的 resource 又说出错，惨了，初步断定无药可救！   

再用 `eXeScope` 看原 `Radmin.exe`，在 resource 里看到有 `ICON`, `MENU`, `BITMAP` 等等，反正算是齐全了。   

对，就是差这些了，把这些 resource 加到刚才 dump 出来的 exe 里去！   

directory 里的 resource RVA : `11000`, size : `EB498`，不就是整个 `.rsrc` section 吗。   

好，先把 dump 出来 exe 的 entry point 改为 `2EBFC`， image base 改为 `AA0000`，directory 里所有的 RVA, size 先全改为 0   

再将每个 section 的正确值修改回去，如上列出的   

第一个 section   

```
name              :  (随便)
Virtual Address :    1000
Virtual Size    :    39000
Raw Offset      :    1000
Raw Size        :    39000
Characteristics : E00000E0
```

如此类推，最后加上我们的 `.rsrc`，resource section. RVA `181000`, size `11D000`，改 directory 的 resource RVA `181000`, size `EC000`。   

用 `Hex workshop` 将 `Radmin.exe` 的 `.rsrc` section (Offset `10000`, size `EC000`) copy 出来，   

追加到 dump 出来 exe 的最后，存盘，咦？依然未见图标？？   

再用 `Peditor` 看 directory 的 resource 的详细列表，`ICON`, `MENU`, `BITMAP` 等等都有了，选中 `cursor` 下的 21 子项，在 Item Info 组合框中的 RVA to Data 是 `393A8`，size : `134`。   

哦，原来 RVA `393A8` 并不是指向我们新加的 `.rsrc` section。   

最笨的方法就是把这些 RVA to Data 逐个修正过来，可是 `Peditor` 不可以修改该值，天啊，天晓得在 `.rsrc` 这堆“垃圾”里改哪个值才是！   

还是写程序吧，幸好在找有关 resource 结构的时候找到有列举 resource 的程序，修改一下就可以用了。   

以下程序是裁剪并修改 `MSDN` 的 `Peering Inside the PE: A Tour of the Win32 Portable Executable File Format` 里的程序，   

(那些 `printf` 我都没有去掉)， `fix_value` 就是要修正的量。   

```
             new section RVA      old section RVA      alignment diff
               |                     |                  |
DWORD fix_value = 0x181000       -         0x10000       -     0x1000;
resDataEntry->OffsetToData = resData_RVA + fix_value;
```

先把 `.rsrc` 整个 section save as 为 `res.bin`, 程序是将 `res.bin` map 到内存，再修改 `OffsetToData`，   

记住运行一次该程序就可以了，再将该 `res.bin` 粘贴回 dump 出来的 exe 去，exe 图标就会出现了。   

最后是 import table 的重建了(唉，太累了!!!)。   

在 `Import REConstructor` 的 `OEP` 里填 `2EBFC`，get import 不成功，原因是 `OEP` 所对应的 image base 是 `AA0000`，   

而不是 `01400000`，在 load `radmin.exe`, `trw` 里下 `mod32.` 显示:   

```
hMod   PEHeader   Base        Module     No   ModuleName
----   --------   --------   --------   --   ----------
xxxx   81991A58   01400000   xxxxxxxx   xx   RADMIN.EXE
```

`d ds:81991A58 + 34` 处就是 image base `01400000`，改其为 `00AA0000`   

再在 `Import REConstructor` 里 get import 就成功了，只有一个 `1401000` 不知道, 暂定为 `GetModuleFileNameA`。   

(后来没仔细看对不对了)，至此脱壳完成。程序有自检，这里就不说了。   

```
/*  fixres.cpp               */
/*  cl fixres.cpp user32.lib */
#include <stdio.h>
#include <windows.h>

BOOL  bFixIt = TRUE;

DWORD fix_value = 0x181000 - 0x10000 - 0x1000;

// Function prototype (necessary because two functions recurse)

void DumpResourceDirectory
(
    PIMAGE_RESOURCE_DIRECTORY resDir, DWORD resourceBase,
    DWORD level, DWORD resourceType
);

// The predefined resource types

char *SzResourceTypes[] = {
"???_0", "CURSOR", "BITMAP", "ICON", "MENU", "DIALOG", "STRING", "FONTDIR",
"FONT", "ACCELERATORS", "RCDATA", "MESSAGETABLE", "GROUP_CURSOR",
"???_13", "GROUP_ICON", "???_15", "VERSION"
};

// Get an ASCII string representing a resource type

void GetResourceTypeName(DWORD type, PSTR buffer, UINT cBytes)
{
    if ( type <= 16 )
        strncpy(buffer, SzResourceTypes[type], cBytes);
    else
        wsprintf(buffer, "%X", type);
}

//
// If a resource entry has a string name (rather than an ID), go find
// the string and convert it from unicode to ascii.
//

void GetResourceNameFromId
(
    DWORD id, DWORD resourceBase, PSTR buffer, UINT cBytes
)
{
    PIMAGE_RESOURCE_DIR_STRING_U prdsu;
    // If it's a regular ID, just format it.
    if ( !(id & IMAGE_RESOURCE_NAME_IS_STRING) )
    {
        wsprintf(buffer, "%X", id);
        return;
    }

    id &= 0x7FFFFFFF;
    prdsu = (PIMAGE_RESOURCE_DIR_STRING_U)(resourceBase + id);
    // prdsu->Length is the number of unicode characters
    WideCharToMultiByte(CP_ACP, 0, prdsu->NameString, prdsu->Length,
                         buffer, cBytes,    0, 0);
    buffer[ min(cBytes-1,prdsu->Length) ] = 0;    // Null terminate it!!!
}

//
// Dump the information about one resource directory entry.  If the
// entry is for a subdirectory, call the directory dumping routine
// instead of printing information in this routine.
//

void DumpResourceEntry
(
    PIMAGE_RESOURCE_DIRECTORY_ENTRY resDirEntry,
    DWORD resourceBase,
    DWORD level
)
{
    UINT i;
    PIMAGE_RESOURCE_DATA_ENTRY resDataEntry;
    DWORD resData_RVA;
    char nameBuffer[128];

    if ( resDirEntry->OffsetToData & IMAGE_RESOURCE_DATA_IS_DIRECTORY )
    {
        DumpResourceDirectory( (PIMAGE_RESOURCE_DIRECTORY)
            ((resDirEntry->OffsetToData & 0x7FFFFFFF) + resourceBase),
            resourceBase, level, resDirEntry->Name);
        return;
    }

    // Spit out the spacing for the level indentation
    for ( i=0; i < level; i++ )
        printf("    ");

    if ( resDirEntry->Name & IMAGE_RESOURCE_NAME_IS_STRING )
    {
        GetResourceNameFromId(resDirEntry->Name, resourceBase, nameBuffer,
                              sizeof(nameBuffer));
        printf("Name: %s  Offset: %08X\n",
            nameBuffer, resDirEntry->OffsetToData);
    }
    else
    {
        printf("ID: %08X  Offset: %08X\n",
               resDirEntry->Name, resDirEntry->OffsetToData);
    }

    resDataEntry = (PIMAGE_RESOURCE_DATA_ENTRY)(resDirEntry->OffsetToData + resourceBase);
    resData_RVA = resDataEntry->OffsetToData;
    printf("Resource data RVA is : %08X\n", resData_RVA);
    if ( bFixIt )
        resDataEntry->OffsetToData = resData_RVA + fix_value;
}

//
// Dump the information about one resource directory.
//

void DumpResourceDirectory
(
    PIMAGE_RESOURCE_DIRECTORY resDir,
    DWORD resourceBase,
    DWORD level,
    DWORD resourceType
)
{
    PIMAGE_RESOURCE_DIRECTORY_ENTRY resDirEntry;
    char szType[64];
    UINT i;

    // Spit out the spacing for the level indentation
    for ( i=0; i < level; i++ )
        printf("    ");

    // Level 1 resources are the resource types
    if ( level == 1 && !(resourceType & IMAGE_RESOURCE_NAME_IS_STRING) )
    {
        GetResourceTypeName( resourceType, szType, sizeof(szType) );
    }
    else    // Just print out the regular id or name
    {
        GetResourceNameFromId( resourceType, resourceBase, szType,
                               sizeof(szType) );
    }

    printf(
        "ResDir (%s) Named:%02X ID:%02X TimeDate:%08X Vers:%u.%02u Char:%X\n",
        szType,    resDir->NumberOfNamedEntries, resDir->NumberOfIdEntries,
        resDir->TimeDateStamp, resDir->MajorVersion,
        resDir->MinorVersion,resDir->Characteristics);
```



```c
resDirEntry = (PIMAGE_RESOURCE_DIRECTORY_ENTRY)(resDir+1);

for ( i=0; i < resDir->NumberOfNamedEntries; i++, resDirEntry++ )
DumpResourceEntry(resDirEntry, resourceBase, level+1);

for ( i=0; i < resDir->NumberOfIdEntries; i++, resDirEntry++ )
DumpResourceEntry(resDirEntry, resourceBase, level+1);

}

//

// Top level routine called to dump out the entire resource hierarchy

//

void DumpResourceSection( LPVOID pmapbase )

{

IMAGE_RESOURCE_DIRECTORY *resDir;

resDir = (IMAGE_RESOURCE_DIRECTORY*) pmapbase;

printf("Resources\n");

DumpResourceDirectory(resDir, (DWORD)resDir, 0, 0);

}

int main( void )

{

HANDLE hfile, hMap;

LPVOID p_MapFile;

if ( (hfile = CreateFile("res.bin", GENERIC_READ | GENERIC_WRITE, FILE_SHARE_READ + FILE_SHARE_WRITE, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL)) == INVALID_HANDLE_VALUE )

{

printf("Can not open file res.bin\n");

exit(1);

}

if ( (hMap = CreateFileMapping(hfile, NULL, PAGE_READWRITE, 0, 0, NULL)) == NULL )

{

CloseHandle( hfile );

printf("Can not create file mapping!\n");

exit(1);

}

if ( (p_MapFile = MapViewOfFile(hMap, FILE_MAP_WRITE, 0, 0, 0)) == NULL )

{

CloseHandle( hfile );

CloseHandle( hMap );

printf("Can not map file!\n");

exit(1);

}

__try

{

DumpResourceSection( p_MapFile );

}

__except(1)

{

printf("Error when processing file!\n");

}

UnmapViewOfFile( p_MapFile );

CloseHandle( hfile );

CloseHandle( hMap );

return 0;

}
```
`4dd466d0e1e0cc123afbe504da2462ca`
