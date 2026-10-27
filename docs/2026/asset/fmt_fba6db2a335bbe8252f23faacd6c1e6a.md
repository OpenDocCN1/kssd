
```
                case 'a':

                case 'A':

                        nDecNum = 10;

                        break;

                case 'b':

                case 'B':

                        nDecNum = 11;

                        break;

                case 'c':

                case 'C':

                        nDecNum = 12;

                        break;

                case 'd':

                case 'D':

                        nDecNum = 13;

                        break;

                case 'e':

                case 'E':

                        nDecNum = 14;

                        break;

                case 'f':

                case 'F':

                        nDecNum = 15;

                        break;

                case '0':

                case '1':

                case '2':

                case '3':

                case '4':

                case '5':

                case '6':

                case '7':

                case '8':

                case '9':

                        nDecNum = strSource[i] - '0';

                        break;

                default:

                        return 0;

                }

                nTemp += (DWORD)nDecNum \* (long double)pow((long double)16, (int)(strlen(strSource) - i - 1));

        }

        return nTemp;

}

/\*\*

\* 按指定的某种格式输出字符串str

\*/

void OutPutStr(char \*str, ...)        // \*

{

        // #if \_DEBUG

        char strbuf[1024];

        // 处理可变参数

        va\_list        vl;

        va\_start(vl, str);

        vsprintf(strbuf, str, vl);

        va\_end(vl);

        OutputDebugString(strbuf);

        // #endif

}

// ------------------------------------------------ main.cpp：------------------------------------------------------------

// 此文件主要是读取要保护的文件的PE信息，加载跟文件有关的MAP符号信息，在该PE文件的最后区块后加多一个

// 区块，此区块将用于保存虚拟机的各种信息，如虚拟机指令，跳转表等。接下来，对文件的代码进行反编译后，根

// 据生成的汇编指令进行虚拟机保护，最后生成受虚拟机保护的新PE文件

#include "stdafx.h"

#include "CCodeILFactory.h"

#include ".\pestructure.h"

/\*\*

\* 程序入口

\*/

int main()                // \*

{

        // PE文件信息

        CPEStructure pestruct;

        CCodeILFactory codefactory;

        list<CodeNode \*> CodeList;

        list<CodeNode \*> CodeList1;

        // 打开文件，读取头部，并读取区块数据

        pestruct.OpenFileName("G:\\项目\\VMPacker\\ConvertCrackMe\\Release\\ConvertCrackMe.exe");

        // 读取map文件，解析其中的符号信息并存于pestruct.MapVector中

        // 注：MAP文件是一个文本文件，记录了程序的入口地址、基地址、符号及其对应的段、文件偏址等信息

        pestruct.LoadMap("G:\\项目\\VMPacker\\ConvertCrackMe\\Release\\ConvertCrackMe.map");

        getchar();

        // 为PE文件pestruct分配一个新的区块，并对该区块进行初始化，并生成某

        // 些虚拟机核心指令，并将指令代码复制到该区块的某个子块中

        // 注：具体此函数分新的区块结构怎样，可查看“PE新区块结构图.jpg”

        codefactory.Init(0x400000 + pestruct.GetNewSection());

        // 查找CrackMe()函数的符号信息

        MapStructrue \* stu = pestruct.GetMap("CrackMe(struct HWND\_\_ \*)");

        // 找不到，则直接返回

        if (!stu)

                return 0;

        // 得到符号的基地址

        char \* Base\_Addr = pestruct.image\_section[stu->Segment - 1] + stu->Offset;

        // 反汇编Base\_Addr的代码，即函数CrackMe()的代码

        codefactory.DisasmFunction(&CodeList, Base\_Addr, stu->VirtualAddress);

        //

        list<CodeNode\*>::iterator itr;

        itr = CodeList.begin();

        // 获取CodeList指令列表中起始指令

        CodeNode \* code = \*itr;

        itr = CodeList.end();

        itr--;

        // 获取CodeList指令列表中结束指令

        CodeNode \* endcode = \*itr;

        // 计算CodeList的总代码长度

        int asmlen = endcode->disasm.ip + endcode->disasm.codelen - code->disasm.ip;

        // 组织一段进入虚拟机的指令(即jmp VStartVM)，存储于Base\_Addr

        codefactory.VMFactory.CompileEnterStubCode(Base\_Addr, code->disasm.ip, asmlen);

        // 将文件各头部与区块数据写回至PE文件中

        pestruct.UpdateHeadersSections(FALSE);

        char errtext[255] = {0};

        // 将CodeList指令列表编译为虚拟机字节码

        codefactory.BuildCode(Base\_Addr, &CodeList, errtext);

        // 计算存放虚拟机各个部件的虚存空间长度

        int len = codefactory.m\_JumpTable.m\_addrlen + codefactory.m\_CodeEngine.m\_addrlen +

                                codefactory.m\_EnterStub.m\_addrlen + codefactory.m\_VMEnterStubCode.m\_addrlen +

                                codefactory.m\_VMCode.m\_addrlen;

        // 分配足够的空间

        char \* newdata = new char[len];

        char \* p = newdata;

        // 将跳转表复制到p空间中

        memcpy(p, codefactory.m\_JumpTable.m\_BaseAddr, codefactory.m\_JumpTable.m\_addrlen);

        p += codefactory.m\_JumpTable.m\_addrlen;

        // 将虚拟机其他部分复制到p空间中

        memcpy(p, codefactory.m\_CodeEngine.m\_BaseAddr, codefactory.m\_CodeEngine.m\_addrlen);

        p += codefactory.m\_CodeEngine.m\_addrlen;

        memcpy(p, codefactory.m\_EnterStub.m\_BaseAddr, codefactory.m\_EnterStub.m\_addrlen);

        p += codefactory.m\_EnterStub.m\_addrlen;

        memcpy(p, codefactory.m\_VMEnterStubCode.m\_BaseAddr, codefactory.m\_VMEnterStubCode.m\_addrlen);

        p += codefactory.m\_VMEnterStubCode.m\_addrlen;

        memcpy(p, codefactory.m\_VMCode.m\_BaseAddr, codefactory.m\_VMCode.m\_addrlen);

        p += codefactory.m\_VMCode.m\_addrlen;

        // 为PE文件添加.bug的新区块，即虚拟机

        pestruct.AddSection(newdata,len,".bug");

        pestruct.UpdateHeaders(FALSE);

        pestruct.UpdateHeadersSections(TRUE);

        pestruct.UpdateHeadersSections(FALSE);

        // 创建一个新的可执行文件

        pestruct.MakePE("G:\\项目\\VMPacker\\ConvertCrackMe\\Release\\ConvertCrackMe.vm.exe", len);

        return 0;

}

// ----------------------------------------------------- PEStructure.cpp ----------------------------------------------------------

// 此文件中的代码用于对PE文件的各种操作，包括读取等

#include "StdAfx.h"

#include ".\pestructure.h"

#include <ImageHlp.h>

#pragma comment(lib, "ImageHlp")        // 链接到ImageHlp.lib

CPEStructure::CPEStructure(void)

{

        Init();

}

CPEStructure::~CPEStructure(void)

{

        Free();

}

/\*\*

\* 打开指定的文件并读取文件头，区块数据

\*

\* FileName: 欲打开的文件路径名

\*

\* 返回值：FALSE表示失败，否则成功

\*/

BOOL CPEStructure::OpenFileName(char \* FileName)                // \*

{

        // 已读取文件的字节数

        DWORD dwBytesRead = 0;

        // 释放分配的内存资源

        Free();

        // 打开FileName文件

        hFile = CreateFileA(FileName, GENERIC\_READ,        FILE\_SHARE\_WRITE | FILE\_SHARE\_READ,

                NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);

        // 打开失败，则显示提示信息并直接返回

        if (hFile == INVALID\_HANDLE\_VALUE)

        {

                MsgBox("打开文件失败", MB\_ICONERROR);

                return FALSE;

        }

        // 获取文件大小

        dwFsize = GetFileSize(hFile, 0);

        // 文件不存在内容，则关闭文件并提示错误信息

        if (dwFsize == 0)

        {

                CloseHandle(hFile);

                MsgBox("错误的文件长度", MB\_ICONERROR);

                return FALSE;

        }

        // 打开成功，则将文件名复制至m\_FileName中

        strcpy\_s(m\_FileName, 256, FileName);

        // 计算文件与壳的长度和，以便分配足够的内存空间

        dwOutPutSize = dwFsize + PackerCode\_Size + ALIGN\_CORRECTION;

        // 分配内存，并将其清0

        pMem = (char\*)GlobalAlloc(GMEM\_FIXED | GMEM\_ZEROINIT, dwOutPutSize);

        // 内存分配失败，则关闭文件并提示错误信息

        if (pMem == NULL)

        {

                CloseHandle(hFile);

                MsgBox("申请内存出错", MB\_ICONERROR);

                return FALSE;

        }

        // 将文件读取至pMem中，dwBytesRead表示真正读取的数据长度

        ReadFile(hFile, pMem, dwFsize, &dwBytesRead, NULL);

        // 读取完文件后，就可以关闭文件了

        CloseHandle(hFile);

        // 将PE文件的DOS头部复制到image\_dos\_header头部中

        CopyMemory(&image\_dos\_header, pMem, sizeof(IMAGE\_DOS\_HEADER));

        // 检测是否是一个有效的PE文件，不是则提示错误信息

        if (image\_dos\_header.e\_magic != IMAGE\_DOS\_SIGNATURE)

        {

                MsgBox("不是一个有效的PE文件", MB\_ICONERROR);

                return FALSE;

        }

        // 保存DOS头部长度

        ReservedHeaderRO = sizeof(IMAGE\_DOS\_HEADER);

        // 保存保留头部长度值

        // 注：e\_lfanew是PE头部的偏移，所以，这里的保留头部指的是DOS-PE头部之间的部分
```
