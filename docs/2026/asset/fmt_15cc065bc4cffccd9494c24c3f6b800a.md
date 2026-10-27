
# [分享]PE结构是那一类!获取导入表和导出表的类!方便使用!

我想高手们都别笑话!我发出来希望有研究PE结构的朋友们喜欢,为了不占空间,我把关键的几个发出来,其它的大同小异,我想自己组也能组成一个类,然后,自己使用的,目前网络上没有一个是完整的点的例子,这里是我自己整理添加的,供大家参考!!!:

```
//--------------------------------------PNExeInfo.h----------------------------------------------------

//加载调试辅助:

#include "IMAGEHLP.H"

#pragma comment(lib,"ImageHlp.lib")

#include "tlhelp32.h"

#pragma comment(lib,"th32.lib")

class CPNExeInfo

{

public:

        CPNExeInfo();

        virtual ~CPNExeInfo();

public:

        BOOL Release();

        int     LoadPE(DWORD pId);              //进程装载

        int            LoadPE(DWORD pId,DWORD mId);    //模块装载

        int            LoadPE(LPCTSTR path);           //文件装载

        DWORD        GetSize(){return dwFileSize;}

        //文件头函数:

        PIMAGE\_DOS\_HEADER             GetDosHeader(){return \_dosHeader;}

        PIMAGE\_NT\_HEADERS             GetNTHeaders(){return \_ntHeader;}

        PIMAGE\_SECTION\_HEADER    GetSection(DWORD dwRVA=NULL);

        //输入表函数:

        PIMAGE\_IMPORT\_DESCRIPTOR GetImportModule(DWORD \*dwSize=NULL,LPCTSTR \*name=NULL);

        PIMAGE\_THUNK\_DATA                 GetProcOfModule(LPCTSTR name,DWORD \*dwSize=NULL,

                                                                                         LPCTSTR \*ProcName=NULL,DWORD \*Hint=NULL,

                                                                                         DWORD \*Offset=NULL,DWORD \*Value=NULL,DWORD \*Rva=NULL);

        PIMAGE\_THUNK\_DATA                 GetProcOfModule(int count,DWORD \*dwSize=NULL,

                                                                                         LPCTSTR \*ProcName=NULL,DWORD \*Hint=NULL,

                                                                                         DWORD \*Offset=NULL,DWORD \*Value=NULL,DWORD \*Rva=NULL);

        //输出表函数:

        PIMAGE\_EXPORT\_DIRECTORY GetExportInfo(DWORD \*dwSize=NULL,LPCTSTR \*strFun=NULL,DWORD \*dwAddr=NULL,

                                                                                         DWORD \*dwOrdin=NULL,DWORD \*dwOffset=NULL,LPTSTR DllName=NULL);

protected:

        LPCTSTR                m\_path;  //路径

        HANDLE                hFile;      //打开文件句柄

        DWORD                dwFileSize; //文件大小

private:

        HANDLE                hMapping; //映射句柄

        LPVOID                m\_pData;

        PIMAGE\_DOS\_HEADER        \_dosHeader;

        PIMAGE\_NT\_HEADERS        \_ntHeader;

};

//-----------------------------------------PNExeInfo.cpp

//装载文件:

int CPNExeInfo::LoadPE(LPCTSTR path)

{

        DWORD dwRead=NULL;

        //保存路径:

        m\_path=path;

        //以只读方式,其他进程对文件有读写权,打开文件:

        hFile=CreateFile(m\_path,GENERIC\_READ,FILE\_SHARE\_WRITE|FILE\_SHARE\_READ,

                         NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);

        if(hFile==INVALID\_HANDLE\_VALUE){return        -10;}  //-10代表打开文件失败;

        //获取文件大小:

        dwFileSize=GetFileSize(hFile,NULL);

        if(dwFileSize <= 0){CloseHandle(hFile);return -11;} //-11代表获取文件大小失败;

        /\*

        //创建镜像:

        hMapping=::CreateFileMapping(hFile,NULL,PAGE\_READONLY,0,0,NULL);

        if(hMapping==NULL){

                CloseHandle(hFile);return -20;}        //-20创建镜像出错;

        //映射地址:

        m\_pData=::MapViewOfFile(hMapping,FILE\_MAP\_READ,0,0,0);

        if(m\_pData==NULL){CloseHandle(hFile);return -21;} //-21代表影射地址出错;

        \*/

        //方便以后销毁,采用分配内存方式:

        m\_pData=(DWORD\*)::VirtualAlloc(NULL,dwFileSize,MEM\_COMMIT,PAGE\_READWRITE);

        ReadFile(hFile,m\_pData,dwFileSize,&dwRead,NULL);

        //方便其他函数调用,初始化函数头:

    \_dosHeader =(PIMAGE\_DOS\_HEADER)m\_pData;

    \_ntHeader  =(PIMAGE\_NT\_HEADERS)((LONG)\_dosHeader+(LONG)\_dosHeader->e\_lfanew);

        //关闭文件:

        CloseHandle(hFile);

        return 0;

}

//装载其它进程,方便获得导出,和导入表:

int CPNExeInfo::LoadPE(DWORD pId)

{

        DWORD  dwRead=NULL;

        HANDLE hModule=NULL;

    MODULEENTRY32\* minfo=new MODULEENTRY32;

        if((int)pId<=0){return -10;}

        hModule=CreateToolhelp32Snapshot(TH32CS\_SNAPMODULE,pId);

        if((DWORD)hModule<=0){return -11;}

        Module32First(hModule, minfo);

        HANDLE hOpenProcess=::OpenProcess(PROCESS\_VM\_READ|PROCESS\_VM\_WRITE,FALSE,pId);

        //创建地址:

        m\_pData=(DWORD\*)::VirtualAlloc(NULL,minfo->modBaseSize,MEM\_COMMIT,PAGE\_READWRITE);

        //读取进程:

        ::ReadProcessMemory(hOpenProcess,minfo->modBaseAddr,m\_pData,minfo->modBaseSize,&dwRead);

        \_dosHeader =(PIMAGE\_DOS\_HEADER)m\_pData;

    \_ntHeader  =(PIMAGE\_NT\_HEADERS)((LONG)\_dosHeader+(LONG)\_dosHeader->e\_lfanew);

        //释放:

        if(hModule!=NULL){CloseHandle(hModule);}

        if(hOpenProcess!=NULL){CloseHandle(hOpenProcess);}

        return 0;

}

//好了,我们只要能加载了,什么都好说:
```

输出表:

```
PIMAGE\_EXPORT\_DIRECTORY CPNExeInfo::GetExportInfo(DWORD \*dwSize,LPCTSTR \*strFun,DWORD \*dwAddr,

                                                                                         DWORD \*dwOrdin,DWORD \*dwOffset,LPTSTR DllName)

{

        PIMAGE\_EXPORT\_DIRECTORY \_exportDir=NULL;

        \_exportDir=(PIMAGE\_EXPORT\_DIRECTORY)\_ntHeader->OptionalHeader.DataDirectory[IMAGE\_DIRECTORY\_ENTRY\_EXPORT].VirtualAddress;

        if(\_exportDir==NULL){return NULL;}

        \_exportDir=(PIMAGE\_EXPORT\_DIRECTORY)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\_exportDir,NULL);

        //数量初始化:

        DWORD dwNum=\_exportDir->NumberOfNames;

        if(dwNum<\_exportDir->NumberOfFunctions){dwNum=\_exportDir->NumberOfFunctions;}

        if(dwSize!=NULL){\*dwSize=dwNum;}

        //名称初始化:

        DWORD \*\*ppdwNames = (DWORD \*\*)\_exportDir->AddressOfNames;

    ppdwNames = (PDWORD\*)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)ppdwNames,NULL);

        if(!ppdwNames){return NULL;}

        //地址初始化:

        DWORD \*\*ppdwAddr  = (DWORD \*\*)\_exportDir->AddressOfFunctions;

    ppdwAddr = (PDWORD\*)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)ppdwAddr,NULL);

        if(!ppdwAddr){return NULL;}

        //序号初始化:

        DWORD \*ppdwOrdin=(DWORD\*)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\_exportDir->AddressOfNameOrdinals,NULL);

        if(!ppdwOrdin){return NULL;}

        //获取文件名:

        if(DllName!=NULL){

                char \*szdllName=(PSTR)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\_exportDir->Name,NULL);

                //DllName=szdllName;

                strcpy(DllName,szdllName);

        }

        for(DWORD i=0;i<dwNum;i++)

        {

                char\* szFun=(PSTR)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\*ppdwNames,NULL);

                if(strFun!=NULL){strFun[i]=szFun;}

                if(dwAddr!=NULL){dwAddr[i]=(DWORD)\*ppdwAddr;}

                if(dwOffset!=NULL){

                        dwOffset[i]=(DWORD)ImageRvaToVa(\_ntHeader,\_dosHeader,(DWORD)\*ppdwAddr,NULL);

                        dwOffset[i]=dwOffset[i]-(DWORD)\_dosHeader;

                }

                if(dwOrdin!=NULL){dwOrdin[i]=(WORD)\*ppdwOrdin;}

                ppdwOrdin=(DWORD\*)((DWORD)ppdwOrdin+2);

                ppdwAddr++;

                ppdwNames++;

        }

        return \_exportDir;

}
```

```
//输入模块:

PIMAGE\_IMPORT\_DESCRIPTOR CPNExeInfo::GetImportModule(DWORD \*dwSize,LPCTSTR \*name)

{

        PIMAGE\_IMPORT\_DESCRIPTOR        \_importDesc=NULL;

        //获取输入表虚拟地址:

        \_importDesc=(PIMAGE\_IMPORT\_DESCRIPTOR)\_ntHeader->OptionalHeader.DataDirectory[IMAGE\_DIRECTORY\_ENTRY\_IMPORT].VirtualAddress;

        if(\_importDesc==NULL){return NULL;}

        //将虚拟地址进行转换:

        \_importDesc=(PIMAGE\_IMPORT\_DESCRIPTOR)::ImageRvaToVa(\_ntHeader,m\_pData,(DWORD)\_importDesc,NULL);

        DWORD  dwRe=(DWORD)\_importDesc;

        int i=0;

        if(name!=NULL)

        {

                for(;\_importDesc->Name!=NULL;i++)

                {

                        char \*szName=(PSTR)ImageRvaToVa(\_ntHeader, \_dosHeader, (DWORD)\_importDesc->Name, 0);
```
