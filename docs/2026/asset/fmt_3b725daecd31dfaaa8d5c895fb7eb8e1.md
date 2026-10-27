
# 点滴记录--PE结构学习

## 背景

首先在这里要感谢“熊猫正正”和“玩命”两位前辈，是他们的文章让我能够写出此贴，在编写代码的时候正是对照着“熊猫正正”的国庆PE总复习合集文章，一点点的敲出以下代码。

## 正文

最近在学习PE结构，并试着写了些代码，尽管写PE结构用汇编写相当简洁、方便，可自己刚刚开始接触C++编程，为了更好的掌握熟悉C++编程特点就选择用了C++来写了，由于本人新手代码中多有混淆的地方，请大家多多体谅~，如果能提些建议那是感激不尽了！

当然论坛里，关于PE结构的理论知识和代码已经相当多了，这里仅仅是当做自己的学习笔记和历程记录下。更为以后掌握“壳”的知识铺路，本帖会不断更新完善、照着Stud_PE分析工具来写，最终会出一个界面版的分析工具，这里暂且先把关键部分的代码贴出、在处理各个节表、导入、导出表时并没有对数据部分直接输出结构，而是将数据存储到了结构当中便于之后读取和修改。如果需要输出可以对结构遍历输出。代码在VS2005和2008下编译通过

好了废话不说，上代码

首先是头文件中的结构如下：

```
#pragma once

/*注意在定义结构的时候用到了两种方式，一种是typedef结构定义方式,一种是struct 结构名方式;
之前一直用后者，后来发现有人用TypeDef方式；发现该方式在编译的时候
对于结构中用到的其他自定义结构不需要在他之前定义，而第二种方式则需要在调用之前定义(新手莫见笑!)
*/

//定义导入表模块结构
typedef struct _tagImportDllNode
{
        LPCSTR        pImprotDllName;                                //存储导入DLL名称
        struct _tagImportDllNode        *pBrother;                //DLL兄弟节点指向其他DLL模块结构
        struct _tagImportFuncNode        *pChild;                //DLL孩子节点指向DLL下的函数结构
}ImportDllNode;

//定义导入表模块中调用函数结构
typedef struct _tagImportFuncNode
{
        LPCSTR        pImportFuncName;                        //存储函数名称
        DWORD        ImportFuncAddr;                        //存储函数地址
        DWORD        ImportFuncInFileAddr;                        //存储函数地址在文件中的位置，便于以后(使用钩子时)修改地址
        struct _tagImportFuncNode *pBrother;                //兄弟节点指向其他函数结构
}ImportFuncNode;

//定义导出表模块结构
typedef struct _tagExportDllNode
{
        LPCSTR        pExprotDllName;                                //存储导出DLL名称
        struct _tagExportFuncNode        *pChild;                //DLL孩子节点指向DLL下的函数结构
}ExportDllNode;

//定义导出表模块中调用函数结构
typedef struct _tagExportFuncNode
{
        LPCSTR        pExportFuncName;                        //存储函数名称
        DWORD        ExportFuncAddr;                        //存储函数地址
        DWORD        ExportFuncInFileAddr;                        //存储函数地址在文件中的位置，便于以后(使用钩子时)修改地址
        DWORD        ExportFuncIndex;                        //存储函数按序号方式导出的索引
        struct _tagExportFuncNode        *pBrother;                //兄弟节点指向其他函数结构
}ExportFuncNode;

//定义资源信息结构
struct ResourceInfo
{
        DWORD        ResourceAddr;                                //定义资源数据在文件中的地址
        DWORD        ResourceSize;                                //定义资源数据大小
};

//定义资源头结构
struct ResourceNode
{
        DWORD        ResourceNameOrID;                        //定义资源名称/ID
        LPCSTR        pResourceName;                                //如果是名称该变量为指向名称串的指针
        ResourceInfo        *pResource;                        //指向资源节点
        ResourceNode        *pBrother;                                //指向兄弟节点
        ResourceNode        *pChild;                                //指向孩子节点
};

//定义重定位表结构
struct RelocationNode
{
        DWORD                        FileAddr;                        //重定位项在文件中的偏移位置
        DWORD                        nNumber;                        //重定位块中需定位的项数
        LPCSTR                        lpRelocationAddr;        //指向重定位数据缓冲区
        RelocationNode        *pBrother;                                //指向其他重定位块
};

//定义节表结构
struct SectionNode
{
        BYTE                Name1[8];                                        //存储节表名称
        DWORD                VirualSize;                                //存储节表在内存中的大小
        DWORD                VirtualOffset;                        //存储节表在内存中的偏移
        DWORD                SizeOfRawData;                        //存储节表按节对齐后的大小
        DWORD                PointerToRawData;                //存储节表在文件中的偏移
        DWORD                Characteristics;                        //存储节表属性
        SectionNode *pBrother;                                //指向节表兄弟节点
};

//定义各个结构的头节点指针
ExportDllNode        *pExportTableHead;
ImportDllNode        *pImportTableHead;
ResourceNode        *pResourceTableHead;
RelocationNode        *pRelocationTableHead;
SectionNode                *pSectionTableHead;

//函数声明
bool GetImprotTableInfo(PVOID lpModuleAddr);
bool GetExportTableInfo(PVOID lpModuleAddr);
bool GetResourceTableInfo(PVOID lpModuleAddr);
bool GetRelocationTableInfo(PVOID lpModuleAddr);
bool GetSectionInfo(LPVOID lpModuleAddr);
LPVOID JudgePEFile(LPCSTR lpFileName);
```

是源文件如下：

```
//----------------------------PE文件判断---------------------------------------------

/*
HeadLine:        PE结构部分
Discription:判断文件是否为PE格式，如果是返回文件在内存中的地址，否则返回0
input:                文件名
output:                SUCCESS:返回模块地址;FAILE:返回0;
*/

LPVOID JudgePEFile(LPCSTR lpFileName)
{
        //打开文件
        OFSTRUCT ofs;
        HFILE hFile=::OpenFile(lpFileName, &ofs, OF_READ);
        if(HFILE_ERROR==hFile)
        {
                ::MessageBox(NULL,"Open file failed","Tip",0);
                return 0;
        }

        //创建文件映像
        HANDLE hMapFile=::CreateFileMapping((HANDLE)hFile,NULL,PAGE_READONLY,NULL,NULL,NULL);
        if(NULL==hMapFile)
        {
                ::MessageBox(NULL,"Create fileMapping failed","Tip",0);
                ::CloseHandle((HANDLE)hFile);
                return 0;
        }

        //将文件映射到内存空间并模块地址指向DOS结构
        LPVOID lpMapAddr=::MapViewOfFile(hMapFile, FILE_MAP_READ,NULL,NULL,0);
        IMAGE_DOS_HEADER *lpDosHead;
        lpDosHead=(IMAGE_DOS_HEADER*)lpMapAddr;

        //判断是否MZ标志
        if(lpDosHead->e_magic!='ZM')//IMAGE_DOS_SIGNATURE
        {
                ::CloseHandle((HANDLE)hFile);
                ::CloseHandle(hMapFile);
                ::UnmapViewOfFile(lpMapAddr);
                ::MessageBox(NULL,"DON'T BE A PE FILE","Tip",0);
                return 0;
        }

        //映射NT头
        IMAGE_NT_HEADERS *lpNTHead;
        lpNTHead=(IMAGE_NT_HEADERS*)((DWORD)lpDosHead+lpDosHead->e_lfanew);

        //是否PE标志
        if(lpNTHead->Signature!='EP')//IMAGE_NT_SIGNATURE
        {
                ::CloseHandle((HANDLE)hFile);
                ::CloseHandle(hMapFile);
                ::UnmapViewOfFile(lpMapAddr);
                ::MessageBox(NULL,"DON'T BE A PE FILE","Tip",0);
                return 0;
        }

        return lpMapAddr;
}

//----------------------------PE文件导入表---------------------------------------------

/*
HeadLine:        获取输入表结构
Discription:根据模块地址获取输入表结构
input:                模块地址
output:                SUCCESS:返回1;FAILE:返回0;
*/

bool GetImprotTableInfo(PVOID lpModuleAddr)
{
        //由模块地址取PE头地址
        PIMAGE_NT_HEADERS pNTHead;
        pNTHead=::ImageNtHeader(lpModuleAddr);
        if(pNTHead!=NULL)
        {
                DWORD pImportDisp;
                pImportDisp=RvaToOffset(lpModuleAddr,pNTHead->OptionalHeader.DataDirectory[1].VirtualAddress);
                if (pImportDisp==NULL)
                {
                        return FALSE;
                }

                IMAGE_IMPORT_DESCRIPTOR *pIID;
                pIID=(IMAGE_IMPORT_DESCRIPTOR*)((DWORD)lpModuleAddr+pImportDisp);

                //取导入表函数名称
                DWORD* pImprotDllNameDisp,*pImprotDllNameAddr;
                pImprotDllNameDisp=(DWORD*)RvaToOffset(lpModuleAddr,pIID->Name);
                pImprotDllNameAddr=(DWORD*)((DWORD)lpModuleAddr+(DWORD)pImprotDllNameDisp);

                while(pImprotDllNameDisp!=0)
                {
                        //创建新节点将模块信息存入链表结构中
                        ImportDllNode* pNewDll=(ImportDllNode*)::LocalAlloc(LPTR,sizeof(ImportDllNode));
                        pNewDll->pImprotDllName=(char*)pImprotDllNameAddr;
                        ImportDllNode **pDllTemp;
                        pDllTemp=&pImportTableHead;
                        while(*pDllTemp!=0)
                        {
                                pDllTemp=&((*pDllTemp)->pBrother);
                        }
                        *pDllTemp=pNewDll;
                        //定位头结点
                        char *szDllName=(char*)pImprotDllNameAddr;
                        //printf("DLLName is %s\n",szDllName);
```
