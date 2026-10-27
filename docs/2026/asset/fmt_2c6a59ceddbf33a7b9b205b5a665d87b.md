
```
typedef struct _IMAGE_IMPORT_DESCRIPTOR {

    union {

        DWORD   Characteristics;            

        DWORD   OriginalFirstThunk;         // 指向输入名称表（IAT）的RVA

    } DUMMYUNIONNAME;

    DWORD   TimeDateStamp;                  //时间标识

    DWORD   ForwarderChain;                 // 转发链如果不转发此值为0

    DWORD   Name;                           //指向导入映像文件的名字

    DWORD   FirstThunk;                     // 指向导入地址表（IAT）的RVA

} IMAGE_IMPORT_DESCRIPTOR;

typedef struct _IMAGE_THUNK_DATA32 {

    union {

        DWORD ForwarderString;      // 转发字符的RVA

        DWORD Function;             // 被导入函数的地址

        DWORD Ordinal;              //被导入函数的序号

        DWORD AddressOfData;        // 指向输入名称

    } u1;

} IMAGE_THUNK_DATA32;

typedef struct _IMAGE_IMPORT_BY_NAME {

    WORD    Hint;                   //需导入函数序号

    CHAR   Name[1];                 //导入函数名称

} IMAGE_IMPORT_BY_NAME, *PIMAGE_IMPORT_BY_NAME;
```

获取导入表信息的函数：

```
void CPeHear::GetImportDirectory()
{

    PIMAGE_DATA_DIRECTORY pDir = (PIMAGE_DATA_DIRECTORY)m_pNT->OptionalHeader.DataDirectory;

    PIMAGE_DATA_DIRECTORY pDataDir = pDir+IMAGE_DIRECTORY_ENTRY_IMPORT;

    PIMAGE_IMPORT_DESCRIPTOR pImport = (PIMAGE_IMPORT_DESCRIPTOR)RVA2Offset(pDataDir->VirtualAddress);

    while ( pImport->Name )

    {

        IMPORT saveImport;

        saveImport.pszDllName = (PCHAR)RVA2Offset(pImport->Name);

        PIMAGE_THUNK_DATA32 pINT = (PIMAGE_THUNK_DATA32)RVA2Offset(pImport->OriginalFirstThunk);

        while ( pINT->u1.Ordinal )

        {

            if ( !IMAGE_SNAP_BY_ORDINAL32(pINT->u1.Ordinal) )

            {

                PIMAGE_IMPORT_BY_NAME pByName = (PIMAGE_IMPORT_BY_NAME)RVA2Offset(pINT->u1.AddressOfData);

                saveImport.NAME.Format(_T("%s"),pByName->Name);

                saveImport.ID = pByName->Hint;   

                saveImport.IM_RVA = pImport->FirstThunk;

                saveImport.Offset = RVA2Offset(pImport->FirstThunk);

                m_vImport.push_back(saveImport);

                pINT++;

                continue;

            } 

            // 2.4.2 如果最高位为1，则直接打印Ordinal部分

            saveImport.NAME.Format(_T("Null"));

            saveImport.ID = pINT->u1.Ordinal&0x0000FFFF;

            saveImport.IM_RVA = pImport->FirstThunk;

            saveImport.Offset = RVA2Offset(pImport->FirstThunk);

            m_vImport.push_back(saveImport);

            pINT++;

        }

        m_vpImport.push_back(pImport);

        pImport++;

    }

}
```

程序中使用到的变量：

由于本程序是通过继承实现的，要使不同的模块访问同一个基类的时候，基类中保存的信息不会改变，所以要使用静态变量来保存PE结构中的需要用到的信息。

变量信息：

```
static PVOID                    m_lpFileImage;           //文件内存中起始加载位置

static CString                  m_strInPath;             //文件名

static IMAGE_FILE_HEADER        m_PeHead;                //PE映像文件头

static IMAGE_OPTIONAL_HEADER    m_Optinal;               //PE映像扩展头

static PIMAGE_SECTION_HEADER    m_pSection;              //获取PE区段信息

static PIMAGE_DOS_HEADER        m_pDos;                  //DOS头地址

static PIMAGE_NT_HEADERS        m_pNT;                   //PE头地址

static PIMAGE_DATA_DIRECTORY    m_pDataDir;              //PE数据目录

static PIMAGE_EXPORT_DIRECTORY  m_pExport;               //输出表

//static PIMAGE_IMPORT_DESCRIPTOR m_pImport;              //输入表

static CString                 m_pszDllName;            

static DWORD                   m_pImageHeaderNum;        //区段数量

static DWORD                   m_NumberOfRvaAndSizes;    //

static DWORD                   m_ExportOffset;           //导出表的偏移

static vector<SECTION>  m_vSection;                      //保存区段中的信息

static vector<EXPORTA>  m_vExport;                       //保存导出表的信息

static vector<IMPORT>   m_vImport;                       //保存导入表的信息

static vector<PIMAGE_IMPORT_DESCRIPTOR> m_vpImport;       
```

总结：

此类的编写实现了PE文件的解析，可以将PE文件中内容显示到界面，但是对于修改文件的PE信息，没有用指针容易实现，没有用到指针不会出现内存泄露等指针问题，但是没有使用指针的节省空间和指针的操作速度，指针对文件修改的方便。有些函数完成的不是很好，需要进一步改善，通过这个项目更好的了解PE结构，对PE的分析有了更好的理解。但是目前软件尚未完成，通过以后的时间慢慢完善。![](https://bbs.kanxue.com/view/img/face/50.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [PE解析软件的大致框架.jpg](javascript:void(0);)（22.15kb，5次下载）
- [PE结构1.jpg](javascript:void(0);)（51.40kb，4次下载）
- [PE头.jpg](javascript:void(0);)（45.32kb，4次下载）
- [PE映射.jpg](javascript:void(0);)（48.48kb，4次下载）
- [区块1.jpg](javascript:void(0);)（88.89kb，4次下载）
- [输出表2.jpg](javascript:void(0);)（45.93kb，8次下载）
- [输入表1.jpg](javascript:void(0);)（44.79kb，4次下载）
