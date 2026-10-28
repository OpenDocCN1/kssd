
```cpp
#include<Windows.h>
#include<stdio.h>

#define DLLNAMELENGTH 0xE
#define FUNCTIONNAMELENGTH 0xF
#define FUNCTIONNAME "puts"
#define DLLNAME "Dll1.dll"

// 获取DOS头
PIMAGE_DOS_HEADER GetDosHeader(_In_ char* pBase) {
    return PIMAGE_DOS_HEADER(pBase);
}

// 获取NT头
PIMAGE_NT_HEADERS GetNtHeader(_In_ char* pBase) {
    return PIMAGE_NT_HEADERS(GetDosHeader(pBase)->e_lfanew + (SIZE_T)pBase);
}

// 获取文件头
PIMAGE_FILE_HEADER GetFileHeader(_In_ char* pBase) {
    return &(GetNtHeader(pBase)->FileHeader);
}

// 获取OPT头
PIMAGE_OPTIONAL_HEADER32 GetOptHeader(_In_ char* pBase) {
    return &(GetNtHeader(pBase)->OptionalHeader);
}

PIMAGE_SECTION_HEADER GetSecByName(_In_ char* pBase, _In_ const char* name) {
    DWORD Secnum = GetFileHeader(pBase)->NumberOfSections;
    PIMAGE_SECTION_HEADER Section = IMAGE_FIRST_SECTION(GetNtHeader(pBase));
    char buf[10] = { 0 };
    for (DWORD i = 0; i < Secnum; i++) {
        memcpy_s(buf, 8, (char*)Section[i].Name, 8);
        if (!strcmp(buf, name)) {
            return Section + i;
        }
    }
    return nullptr;
}

// 获取最后一个区段
PIMAGE_SECTION_HEADER GetLastSec(_In_ char* PeBase) {
    DWORD SecNum = GetFileHeader(PeBase)->NumberOfSections;
    PIMAGE_SECTION_HEADER FirstSec = IMAGE_FIRST_SECTION(GetNtHeader(PeBase));
    PIMAGE_SECTION_HEADER LastSec = FirstSec + SecNum - 1;
    return LastSec;
}

char* OpenPeFiles(_In_ const char* Path, _Out_opt_ DWORD* nFileSize)
{
    // 读文件
    HANDLE hFile = CreateFileA(Path, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
    if (hFile == INVALID_HANDLE_VALUE)
    {
        //printf("打开文件失败");
        return NULL;
    }
    DWORD PeSize = GetFileSize(hFile, NULL);
    if (nFileSize)
        *nFileSize = PeSize;
    DWORD ReadSize = 0;
    char* PeBase = new CHAR[PeSize]{ 0 };
    ReadFile(hFile, PeBase, PeSize, &ReadSize, NULL);

    PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)PeBase;
    // 检测DOS头和NT头
    if (pDosHeader->e_magic != IMAGE_DOS_SIGNATURE)
    {
        //printf("不是PE文件\n");
        //system("pause");
        return NULL;
    }
    PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)(PeBase + pDosHeader->e_lfanew);
    if (pNtHeader->Signature != IMAGE_NT_SIGNATURE)
    {
        //printf("不是PE文件\n");
        //system("pause");
        return NULL;
    }

    CloseHandle(hFile);
    return PeBase;
}

// 粒度对齐处理
int AlignMent(_In_ int size, _In_ int alignment) {
    return (size) % (alignment) == 0 ? (size) : ((size) / alignment + 1) * (alignment);
}

// 新增节
char* AddSection(_In_ char*& PeBase, _In_ DWORD& PeSize, _In_ const char* Section_name, _In_ const int Section_size)
{
    GetFileHeader(PeBase)->NumberOfSections++;
    PIMAGE_SECTION_HEADER LastPeSection = GetLastSec(PeBase);

    memcpy(LastPeSection->Name, Section_name, 8);
    LastPeSection->Misc.VirtualSize = Section_size;
    LastPeSection->VirtualAddress = (LastPeSection - 1)->VirtualAddress + AlignMent((LastPeSection - 1)->SizeOfRawData, GetOptHeader(PeBase)->SectionAlignment);
    LastPeSection->SizeOfRawData = AlignMent(Section_size, GetOptHeader(PeBase)->FileAlignment);
    LastPeSection->PointerToRawData = AlignMent(PeSize, GetOptHeader(PeBase)->FileAlignment);
    LastPeSection->Characteristics = 0xc0000040;//节表属性设为该值，意为该节表可读可写且包含已初始化的数据

    GetOptHeader(PeBase)->SizeOfImage = LastPeSection->VirtualAddress + LastPeSection->SizeOfRawData;

    int NewSize = LastPeSection->PointerToRawData + LastPeSection->SizeOfRawData;

    char* NewPeBase = new char [NewSize] {0};
    // 向新缓冲区录入数据
    memcpy(NewPeBase, PeBase, PeSize);
    // 缓存区更替
    delete PeBase;
    PeSize = NewSize;
    return NewPeBase;
}

// 保存文件
void SaveFile(_In_ const char* path, _In_ const char* data, _In_ int FileSize) {
    HANDLE hFile = CreateFileA(
        path,
        GENERIC_WRITE,
        FILE_SHARE_READ,
        NULL,
        CREATE_ALWAYS,
        FILE_ATTRIBUTE_NORMAL,
        NULL
    );
    DWORD Buf = 0;
    WriteFile(hFile, data, FileSize, &Buf, NULL);
    CloseHandle(hFile);
}

// 将RVA的值转换成FOA
LPVOID RvaToFoa(LPVOID pFileBuffer, LPSTR virtualAddress) {
    LPSTR sectionAddress = NULL;//记录距离节头的距离
    LPSTR fileAddress = NULL;//记录文件中的偏移
    PIMAGE_DOS_HEADER pDosHeader = NULL;
    PIMAGE_NT_HEADERS pNTHeader = NULL;
    PIMAGE_FILE_HEADER pPEHeader = NULL;
    PIMAGE_OPTIONAL_HEADER32 pOptionHeader = NULL;
    PIMAGE_SECTION_HEADER pSectionHeader = NULL;

    if (pFileBuffer == NULL) {
        printf("文件写入内存失败!\n");
        return NULL;
    }

    pDosHeader = (PIMAGE_DOS_HEADER)pFileBuffer;
    pNTHeader = (PIMAGE_NT_HEADERS)((DWORD)pFileBuffer + pDosHeader->e_lfanew);
    pPEHeader = (PIMAGE_FILE_HEADER)((DWORD)pNTHeader + 4);
    pOptionHeader = (PIMAGE_OPTIONAL_HEADER32)((DWORD)pPEHeader + 20);
    pSectionHeader = (PIMAGE_SECTION_HEADER)((DWORD)pOptionHeader + pPEHeader->SizeOfOptionalHeader);

    if ((DWORD)virtualAddress <= pOptionHeader->SizeOfHeaders) {
        return virtualAddress;
    }

    for (DWORD i = 1; i <= pPEHeader->NumberOfSections; i++) {
        if ((DWORD)virtualAddress < pSectionHeader->VirtualAddress) {
            pSectionHeader--;
            break;
        }
        else if (i == pPEHeader->NumberOfSections) {
            break;
        }
        else {
            pSectionHeader++;
        }

    }

    // 距离该节头的距离
    sectionAddress = virtualAddress - pSectionHeader->VirtualAddress;
    fileAddress = pSectionHeader->PointerToRawData + sectionAddress;

    return (LPVOID)fileAddress;
}

// 将FOA的值转换成RVA
LPVOID FoaToRva(LPVOID pFileBuffer, LPSTR fileaddress) {
    LPSTR sectionAddress = NULL;//记录距离节头的距离
    LPSTR virtualaddress = NULL;//记录内存中的偏移
    PIMAGE_DOS_HEADER pDosHeader = NULL;
    PIMAGE_NT_HEADERS pNTHeader = NULL;
    PIMAGE_FILE_HEADER pPEHeader = NULL;
    PIMAGE_OPTIONAL_HEADER32 pOptionHeader = NULL;
    PIMAGE_SECTION_HEADER pSectionHeader = NULL;

    if (pFileBuffer == NULL) {
        printf("文件写入内存失败!\n");
        return NULL;
    }

    pDosHeader = (PIMAGE_DOS_HEADER)pFileBuffer;
    pNTHeader = (PIMAGE_NT_HEADERS)((DWORD)pFileBuffer + pDosHeader->e_lfanew);
    pPEHeader = (PIMAGE_FILE_HEADER)((DWORD)pNTHeader + 4);
    pOptionHeader = (PIMAGE_OPTIONAL_HEADER32)((DWORD)pPEHeader + 20);
    pSectionHeader = (PIMAGE_SECTION_HEADER)((DWORD)pOptionHeader + pPEHeader->SizeOfOptionalHeader);

    if ((DWORD)fileaddress <= pOptionHeader->SizeOfHeaders) {
        return fileaddress;
    }

    for (DWORD i = 1; i <= pPEHeader->NumberOfSections; i++) {
        if ((DWORD)fileaddress < pSectionHeader->PointerToRawData) {
            pSectionHeader--;
            break;
        }
        else if (i == pPEHeader->NumberOfSections) {
            break;
        }
        else {
            pSectionHeader++;
        }

    }

    // 距离该节头的距离
    sectionAddress = fileaddress - pSectionHeader->PointerToRawData;
    virtualaddress = pSectionHeader->VirtualAddress + sectionAddress;

    return (LPVOID)virtualaddress;
}

char* inject_dll(_In_ char*& PeBase, _In_ DWORD& PeSize)
{
    PIMAGE_DATA_DIRECTORY pDataDirectory = NULL;//定位表目录
    PIMAGE_IMPORT_DESCRIPTOR importTableAddress = NULL;//定位导入表的真正位置
    LPVOID returnAddress = NULL;//记录RVAtoFOA的返回值

    // 定位到新节的位置和导入表的位置
    pDataDirectory = (PIMAGE_DATA_DIRECTORY)GetOptHeader(PeBase)->DataDirectory;
    pDataDirectory += 0x1;

    DWORD sectionLength = pDataDirectory->Size + 0x28 + +0x10 + DLLNAMELENGTH + FUNCTIONNAMELENGTH + 0x2;
    sectionLength = AlignMent(sectionLength, GetOptHeader(PeBase)->FileAlignment);

    char SecName[] = ".ddjsq";
    char* NewPeBase = AddSection(PeBase, PeSize, SecName, sectionLength);

    pDataDirectory = (PIMAGE_DATA_DIRECTORY)GetOptHeader(NewPeBase)->DataDirectory;
    pDataDirectory += 0x1;

    PDWORD pNewSection = (PDWORD)(GetLastSec(NewPeBase)->PointerToRawData + (DWORD)NewPeBase);
    returnAddress = RvaToFoa(NewPeBase, (LPSTR)pDataDirectory->VirtualAddress);
    importTableAddress = (PIMAGE_IMPORT_DESCRIPTOR)((DWORD)returnAddress + (DWORD)NewPeBase);

    // 复制原导入表，在原导入表后新增一个导入表
    memcpy(pNewSection, importTableAddress, pDataDirectory->Size);
    importTableAddress = (PIMAGE_IMPORT_DESCRIPTOR)((DWORD)pNewSection + pDataDirectory->Size - 0x14);

    // 增加8字节INT表
    PIMAGE_THUNK_DATA32 pIntTable = (PIMAGE_THUNK_DATA32)((DWORD)importTableAddress + 0x28);//保留20字节的0
    PIMAGE_THUNK_DATA32 repairIntTable = pIntTable;
    pIntTable++;
    pIntTable->u1.Ordinal = 0x0;
    pIntTable++;

    // 增加8字节IAT表
    PIMAGE_THUNK_DATA32 pIatTable = (PIMAGE_THUNK_DATA32)(pIntTable);
    PIMAGE_THUNK_DATA32 repairIatTable = pIatTable;
    pIatTable++;
    pIatTable->u1.Ordinal = 0x0;
    pIatTable++;

    // 分配空间存储DLL名称字符串
    PDWORD dllNameAddress = (PDWORD)pIatTable;
    memcpy(dllNameAddress, DLLNAME, DLLNAMELENGTH);

    // 增加IMAGE_IMPORT_BY_NAME 结构
    PIMAGE_IMPORT_BY_NAME functionNameAddress = (PIMAGE_IMPORT_BY_NAME)((DWORD)dllNameAddress + DLLNAMELENGTH);
    PDWORD pFunctionName = (PDWORD)((DWORD)functionNameAddress + 0x2);
    memcpy(pFunctionName, FUNCTIONNAME, FUNCTIONNAMELENGTH);

    // 将IMAGE_IMPORT_BY_NAME结构的RVA赋值给INT和IAT表中的第一项
    repairIntTable->u1.AddressOfData = (DWORD)FoaToRva(NewPeBase, (LPSTR)((DWORD)functionNameAddress - (DWORD)NewPeBase));
    repairIatTable->u1.AddressOfData = repairIntTable->u1.Ordinal;

    // 修正导入表Name、OriginalFirstThunk、FirstThunk
    importTableAddress->Name = (DWORD)FoaToRva(NewPeBase, (LPSTR)((DWORD)dllNameAddress - (DWORD)NewPeBase));
    importTableAddress->OriginalFirstThunk = (DWORD)FoaToRva(NewPeBase, (LPSTR)((DWORD)repairIntTable - (DWORD)NewPeBase));
    importTableAddress->FirstThunk = (DWORD)FoaToRva(NewPeBase, (LPSTR)((DWORD)repairIatTable - (DWORD)NewPeBase));

    // 修正IMAGE_DATA_DIRECTORY结构的VirtualAddress和Size
    pDataDirectory->VirtualAddress = (DWORD)FoaToRva(NewPeBase, (LPSTR)((DWORD)pNewSection - (DWORD)NewPeBase));
    pDataDirectory->Size += 0x14;

    return NewPeBase;
}

int main()
{
    char path[] = "路径";
    DWORD pesize;
    char* PeBase = OpenPeFiles(path, &pesize);
    if (!PeBase)
    {
        printf("wrong");
        return 0;
    }
    char* NewPeBase = inject_dll(PeBase, pesize);
    SaveFile("路径", NewPeBase, pesize);
}
```
