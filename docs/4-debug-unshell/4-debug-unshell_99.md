

# AsPack 2.12 分析及静态脱壳机编写，附源码

> 原文：[`www.pediy.com/kssd/pediy10/91073.html`](https://www.pediy.com/kssd/pediy10/91073.html)

科锐第四阶段作业题  
  
一，准备解密需要的 API 函数 VirtualAlloc VirtualFree  
  
00405000    90              nop  
00405001 >  60              pushad  
00405002    E8 03000000     call    0040500A                          
00405007  - E9 EB045D45     jmp     459D54F7  
0040500C    55              push    ebp  
0040500D    C3              retn  
0040500E    E8 01000000     call    00405014  
00405013    EB 5D           jmp     short 00405072  
00405015    BB EDFFFFFF     mov     ebx, -13                         ; 得到 SHELL 代码段首地址  
0040501A    03DD            add     ebx, ebp  
0040501C    81EB 00500000   sub     ebx, 5000                        ; 得到 ImageBase  
00405022    83BD 22040000 0>cmp     dword ptr \[ebp+422\], 0  
00405029    899D 22040000   mov     dword ptr \[ebp+422\], ebx  
0040502F    0F85 65030000   jnz     0040539A  
00405035    8D85 2E040000   lea     eax, dword ptr \[ebp+42E\]  
0040503B    50              push    eax  
0040503C    FF95 4D0F0000   call    dword ptr \[ebp+F4D\]              ; GetModuleHandle("kernel32.dll")  
00405042    8985 26040000   mov     dword ptr \[ebp+426\], eax  
00405048    8BF8            mov     edi, eax  
0040504A    8D5D 5E         lea     ebx, dword ptr \[ebp+5E\]  
0040504D    53              push    ebx  
0040504E    50              push    eax  
0040504F    FF95 490F0000   call    dword ptr \[ebp+F49\]              ; GetProcAddress(eax,"VirtualAlloc")  
00405055    8985 4D050000   mov     dword ptr \[ebp+54D\], eax  
0040505B    8D5D 6B         lea     ebx, dword ptr \[ebp+6B\]  
0040505E    53              push    ebx  
0040505F    57              push    edi  
00405060    FF95 490F0000   call    dword ptr \[ebp+F49\]              ; GetProcAddress(edi,"VirtualFree")  
00405066    8985 51050000   mov     dword ptr \[ebp+551\], eax  
0040506C    8D45 77         lea     eax, dword ptr \[ebp+77\]  
  
  
二，申请解密缓存，并解密代码及其它数据，然后释放缓存  
  
因为这个加密算法对代码段的 jmp,call 做了特殊处理  
因此，解密后还得对代码段的地址做些处理  
  
  
004050AD    6A 04           push    4  
004050AF    68 00100000     push    1000  
004050B4    68 00180000     push    1800  
004050B9    6A 00           push    0  
004050BB    FF95 4D050000   call    dword ptr \[ebp+54D\]              ; VirtualAlloc,申请解密时所需要的数据空间  
004050C1    8985 56010000   mov     dword ptr \[ebp+156\], eax  
004050C7    8B46 04         mov     eax, dword ptr \[esi+4\]  
004050CA    05 0E010000     add     eax, 10E  
004050CF    6A 04           push    4  
004050D1    68 00100000     push    1000  
004050D6    50              push    eax  
004050D7    6A 00           push    0  
004050D9    FF95 4D050000   call    dword ptr \[ebp+54D\]              ; VirtualAlloc,申请解密代码缓存的空间  
004050DF    8985 52010000   mov     dword ptr \[ebp+152\], eax  
004050E5    56              push    esi  
004050E6    8B1E            mov     ebx, dword ptr \[esi\]             ; 取出解密区段偏移  
004050E8    039D 22040000   add     ebx, dword ptr \[ebp+422\]         ; 区段偏移+ImageBase=解密区段首地址  
004050EE    FFB5 56010000   push    dword ptr \[ebp+156\]              ; 解密地址所需数据缓存  
004050F4    FF76 04         push    dword ptr \[esi+4\]                ; 缓存大小  
004050F7    50              push    eax                              ; 解密代码缓存  
004050F8    53              push    ebx                              ; 解密区段首地址  
004050F9    E8 6E050000     call    0040566C                         ; 解密函数  
004050FE    B3 00           mov     bl, 0  
00405100    80FB 00         cmp     bl, 0                            ; 判断是否是代码段数据  
00405103    75 5E           jnz     short 00405163  
00405105    FE85 EC000000   inc     byte ptr \[ebp+EC\]                ; 将代码段判断标识+1  
0040510B    8B3E            mov     edi, dword ptr \[esi\]  
0040510D    03BD 22040000   add     edi, dword ptr \[ebp+422\]  
00405113    FF37            push    dword ptr \[edi\]                  ; 好像是废代码，开始  
00405115    C607 C3         mov     byte ptr \[edi\], 0C3  
00405118    FFD7            call    edi  
0040511A    8F07            pop     dword ptr \[edi\]                  ; 好像是废代码，结束  
0040511C    50              push    eax                              ; 此处开始对代码段的 jmp,call 做还原处理  
0040511D    51              push    ecx  
0040511E    56              push    esi  
0040511F    53              push    ebx  
00405120    8BC8            mov     ecx, eax                         ; 得到解密缓存长度  
00405122    83E9 06         sub     ecx, 6  
00405125    8BB5 52010000   mov     esi, dword ptr \[ebp+152\]  
0040512B    33DB            xor     ebx, ebx  
0040512D    0BC9            or      ecx, ecx  
0040512F    74 2E           je      short 0040515F  
00405131    78 2C           js      short 0040515F  
00405133    AC              lods    byte ptr \[esi\]  
00405134    3C E8           cmp     al, 0E8                          ; 判断是否为 call  
00405136    74 0A           je      short 00405142  
00405138    EB 00           jmp     short 0040513A  
0040513A    3C E9           cmp     al, 0E9                          ; 判断是否为 jmp  
0040513C    74 04           je      short 00405142  
0040513E    43              inc     ebx  
0040513F    49              dec     ecx  
00405140  ^ EB EB           jmp     short 0040512D  
00405142    8B06            mov     eax, dword ptr \[esi\]  
00405144    EB 00           jmp     short 00405146  
00405146    803E 01         cmp     byte ptr \[esi\], 1                ; jmp/CALL 后面的第一个字节为还原条件，针对不同的程序，判断条件不同  
00405149  ^ 75 F3           jnz     short 0040513E  
0040514B    24 00           and     al, 0                            ; 此处开始还原处理  
0040514D    C1C0 18         rol     eax, 18  
00405150    2BC3            sub     eax, ebx  
00405152    8906            mov     dword ptr \[esi\], eax  
00405154    83C3 05         add     ebx, 5  
00405157    83C6 04         add     esi, 4  
0040515A    83E9 05         sub     ecx, 5  
0040515D  ^ EB CE           jmp     short 0040512D  
0040515F    5B              pop     ebx  
00405160    5E              pop     esi  
00405161    59              pop     ecx  
00405162    58              pop     eax                              ; jmp,call 还原结束  
  
  
  
  
三，获得原导入表的 RVA  
00405272    66:AD           lods    word ptr \[esi\]  
00405274    66:AB           stos    word ptr es:\[edi\]  
00405276  ^ EB F1           jmp     short 00405269  
00405278    BE 08260000     mov     esi, 2608                        ; 导入表数据目录 RVA  
0040527D    8B95 22040000   mov     edx, dword ptr \[ebp+422\]  
00405283    03F2            add     esi, edx  
00405285    8B46 0C         mov     eax, dword ptr \[esi+C\]  
00405288    85C0            test    eax, eax  
  
  
  
四，动态修改 OEP，并跳转到原程序的 OEP  
  
0040539A    B8 10190000     mov     eax, 1910                        ; 原程序的 OEP  
0040539F    50              push    eax  
004053A0    0385 22040000   add     eax, dword ptr \[ebp+422\]  
004053A6    59              pop     ecx  
004053A7    0BC9            or      ecx, ecx  
004053A9    8985 A8030000   mov     dword ptr \[ebp+3A8\], eax         ; 将 OEP 写到 Ep+0x3BB  
004053AF    61              popad  
004053B0    75 08           jnz     short 004053BA  
004053B2    B8 01000000     mov     eax, 1  
004053B7    C2 0C00         retn    0C  
004053BA    68 00000000     push    0  
004053BF    C3              retn                                     ; 跳转到脱壳后的 OEP  
  
此时壳己工作完毕  
  
  
五，静态脱壳机的编写  
  
1.打开 IDA，找到解密函数，将代码抠出，然后准备一个 ASM 文件，将其编译为 OBJ，然后在工程中将其导入，并包装一下导入函数  
  
  extern "C" DWORD g\_esi;  
  extern "C" void \_\_stdcall AsPackDecode();  
  
  //解密函数  
  void MyDecode(void \*source,void \*sourcebuff,void \* addrbuffer,DWORD addrsize)  
  {  
    \_\_asm  
    {  
      push addrbuffer  
      push addrsize  
      push sourcebuff  
      push source  
      call AsPackDecode  
    }  
  }  
  
  
2.在调用解密函数时，因为频频出错，才发现，原来解密代码时，用到了 SHELL 里面的一些数据，因此，需要对解密的 ASM 代码进行一点修改  
  
  ;全局变量  
  .data  
    g\_esi dd 0  
  .code  
  
  ;将这个重定位的函数中的 ESI 做为全局变量，然后在解密之前修改 ESI 到正确的数据偏移  
  sub\_405C96 proc near  
    mov esi, g\_esi        
    retn  
  sub\_405C96 endp  
  
  
其它的没啥好说的了，全部代码如下  
// CKUnPack.cpp : 定义控制台应用程序的入口点。  
//  
  
#include "stdafx.h"  
#include <windows.h>  
#include <vector>  
using namespace std;  
  
extern "C" DWORD g\_esi;  
extern "C" void \_\_stdcall AsPackDecode();  
  
//解密函数  
void MyDecode(void \*source,void \*sourcebuff,void \* addrbuffer,DWORD addrsize)  
{  
  \_\_asm  
  {  
    push addrbuffer  
    push addrsize  
    push sourcebuff  
    push source  
    call AsPackDecode  
  }  
}  
  
  
IMAGE\_DOS\_HEADER \*dosHeader = NULL;      //DOS 头  
IMAGE\_NT\_HEADERS \*ntHeader = NULL;      //NT 头  
IMAGE\_FILE\_HEADER \*pFileHead = NULL;    //文件头  
IMAGE\_OPTIONAL\_HEADER32 \*pOptHead = NULL;  //可选头  
IMAGE\_IMPORT\_DESCRIPTOR \*pIID = NULL;    //数据目录  
  
LPTHREAD\_START\_ROUTINE OEP = NULL;  
LPBYTE pImageBase = NULL;  
DWORD dwImageBase = 0;  
  
  
DWORD dwMemMaxAddr = 0;  //内存中最大的映射地址  
DWORD dwMemMaxSize = 0;  //内存中最大的映射大小  
  
HANDLE hFile = NULL;  
HANDLE hFileMaping = NULL;  
  
  
LPVOID MapFileToMemory(char \*szFileName)  
{  
  //映射文件  
  hFile = CreateFile(szFileName, GENERIC\_READ, 0, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);  
  if (hFile==INVALID\_HANDLE\_VALUE)  
  {  
    puts("Can't open file！");  
    return NULL;  
  }  
  
  hFileMaping = CreateFileMapping(hFile, NULL, PAGE\_READONLY, 0, 0, NULL);  
  LPVOID lpFile = MapViewOfFile(hFileMaping, FILE\_MAP\_READ, 0, 0, 0);  
  
  dosHeader = (IMAGE\_DOS\_HEADER\*)lpFile;  
  ntHeader = (IMAGE\_NT\_HEADERS\*)((LPBYTE)lpFile + dosHeader->e\_lfanew);  
  if(ntHeader->Signature != 0x4550)  
  {  
    puts("不是合法的 PE 文件！");  
    return NULL;  
  }  
  pImageBase = (LPBYTE)lpFile;  
  pFileHead = &(ntHeader->FileHeader);  
  pOptHead = &(ntHeader->OptionalHeader);  
  dwImageBase = pOptHead->ImageBase;  
  return lpFile;  
}  
  
  
//获得 PE 大小  
DWORD GetPeSize()  
{  
  //得到节区头指针  
  IMAGE\_SECTION\_HEADER \*pSecInfo = (IMAGE\_SECTION\_HEADER \*)((LPBYTE)pOptHead + pFileHead->SizeOfOptionalHeader);    
  //遍历节区，统计文件大小，先填充 0  
  DWORD dwCount = 0x1000;//最开始有 1000 大小的头信息  
  for(int i = 0; i < pFileHead->NumberOfSections; i++)  
  {  
    // 计算所有节的数据总和  
    int nSize = pSecInfo\[i\].Misc.VirtualSize;  
    dwCount += (nSize & 0xfffff000) + ((nSize & 0xfff) != 0) \* 0x00001000;  
      
    //计录内存中最大的映射地址  
    if(pSecInfo\[i\].VirtualAddress > dwMemMaxAddr)  
    {  
      dwMemMaxAddr = pSecInfo\[i\].VirtualAddress;  
      dwMemMaxSize = pSecInfo\[i\].Misc.VirtualSize;  
    }  
  }  
  return dwCount;  
}  
  
//修复代码地址  
int fixCode(char \*codebuff, char \*addrbuff, int codesize, BYTE bXorVal, BOOL isXor)  
{  
  for(int i = 0; i < codesize; i++)  
  {  
    BYTE bCode = \*(LPBYTE)(codebuff + i);  
    if(bCode == 0xE8 || bCode == 0xE9)  
    {  
      DWORD bAddr = \*(PDWORD)(codebuff + i + 1);  
      if(!isXor)  
      {  
        bAddr -= i;   
        \*(PDWORD)(codebuff + i + 1) = bAddr;  
        i += 4;  
      }  
      else if(\*(LPBYTE)(codebuff + i + 1) == bXorVal)  
      {  
        bAddr &= 0xFFFFFF00;  
        \_\_asm rol bAddr,0x18  
        bAddr -= i;   
        \*(PDWORD)(codebuff + i + 1) = bAddr;  
        i += 4;  
      }  
    }  
  }  
  return 0;  
}  
  
  
  
//得到本进程的 OEP 地址  
DWORD GetCurrentProcessOEP()  
{  
  IMAGE\_DOS\_HEADER \*tempDosHeader = NULL;      //DOS 头  
  IMAGE\_NT\_HEADERS \*tempNtHeader = NULL;      //NT 头  
  IMAGE\_OPTIONAL\_HEADER32 \*tempOptHead = NULL;    //可选头  
  
  HMODULE hMod = GetModuleHandle(NULL);  
  tempDosHeader = (IMAGE\_DOS\_HEADER\*)hMod;  
  tempNtHeader = (IMAGE\_NT\_HEADERS\*)((LPBYTE)hMod + tempDosHeader->e\_lfanew);  
  tempOptHead = &(tempNtHeader->OptionalHeader);  
  
  return (DWORD)hMod + tempOptHead->AddressOfEntryPoint;  
}  
  
int \_tmain(int argc, \_TCHAR\* argv\[\])  
{  
  if(argc == 1 || strlen(argv\[1\]) == 0)  
  {  
    puts("请先输入文件名！");  
    return 0;  
  }  
  
  //映射 EXE 文件  
  LPBYTE lpFile = (LPBYTE)MapFileToMemory(argv\[1\]);  
  if(!lpFile) return 0;  
  
  //获得 PE 文件大小  
  DWORD dwCount = GetPeSize();  
  //得到节区头指针  
  IMAGE\_SECTION\_HEADER \*pSecInfo = (IMAGE\_SECTION\_HEADER \*)((LPBYTE)pOptHead + pFileHead->SizeOfOptionalHeader);    
  
  //申请新文件的内存块  
  LPBYTE pNewPe = (LPBYTE)malloc(dwCount);  
  memset(pNewPe, 0, dwCount);  
  //拷贝 DOS\_HEADER  
  memcpy(pNewPe, dosHeader, dosHeader->e\_lfanew);  
  //拷贝 NT\_HEADER  
  memcpy(pNewPe + dosHeader->e\_lfanew, ntHeader, sizeof(IMAGE\_NT\_HEADERS));  
  //拷贝节表  
  memcpy(pNewPe + dosHeader->e\_lfanew + sizeof(IMAGE\_NT\_HEADERS), pSecInfo  
    ,sizeof(IMAGE\_SECTION\_HEADER) \* pFileHead->NumberOfSections);  
  
  IMAGE\_DOS\_HEADER \*newDosHeader = (IMAGE\_DOS\_HEADER \*)pNewPe;      //DOS 头  
  IMAGE\_NT\_HEADERS \*newNtHeader = (IMAGE\_NT\_HEADERS \*)(pNewPe + newDosHeader->e\_lfanew);      //NT 头  
  IMAGE\_FILE\_HEADER \*newFileHead = &(newNtHeader->FileHeader);    //文件头  
  IMAGE\_OPTIONAL\_HEADER32 \*newOptHead = &(newNtHeader->OptionalHeader);  //可选头  
  
  //得到新 PE 文件节区头指针  
  IMAGE\_SECTION\_HEADER \*newSecInfo = (IMAGE\_SECTION\_HEADER \*)((LPBYTE)newOptHead + newFileHead->SizeOfOptionalHeader);  
  
  //加密区段配置信息内存偏移  
  LPBYTE packaddr = NULL;  
  //地址表异或值  
  BYTE bXorVal;  
  //地址表是否参与异或运算  
  BOOL isXor;  
  
  //循环拷贝映射区段  
  for(int i = 0; i < pFileHead->NumberOfSections; i++)  
  {  
    IMAGE\_SECTION\_HEADER pTemp = pSecInfo\[i\];  
    //判断数据是否合法，合法则拷贝  
    if(pTemp.SizeOfRawData != 0 && pTemp.PointerToRawData != 0)  
    {  
      memcpy(pNewPe + pTemp.VirtualAddress, pImageBase + pTemp.PointerToRawData, pTemp.SizeOfRawData);  
    }  
    //找到加密区段配置信息内存偏移  
    if(pTemp.VirtualAddress <= newOptHead->AddressOfEntryPoint  
      && (pTemp.VirtualAddress + pTemp.Misc.VirtualSize) > newOptHead->AddressOfEntryPoint)  
    {  
      packaddr = (LPBYTE)(pImageBase + pTemp.PointerToRawData + 0x57c);//0x57c 是加密数据的偏移  
  
      //修正解压函数地址  
      g\_esi = (DWORD)(pNewPe + pTemp.VirtualAddress + 0x70E - 0x44403E);  
      //修正 OEP  
      DWORD dwOEP = \*(DWORD\*)(pNewPe + pTemp.VirtualAddress + 0x39b);  
      newOptHead->AddressOfEntryPoint = dwOEP;  
      //修正导入表数据目录  
      DWORD dwIAT = \*(DWORD\*)(pNewPe + pTemp.VirtualAddress + 0x279);  
      newOptHead->DataDirectory\[1\].VirtualAddress = dwIAT;  
      //取出地址表异或值  
      bXorVal = \*(BYTE\*)(pNewPe + pTemp.VirtualAddress + 0x148);  
      //地址表是否参与异或运算  
      isXor = \*(BYTE\*)(pNewPe + pTemp.VirtualAddress + 0x145) > 0 ? 0 : 1;  
    }  
  }  
  
  //循环解密数据  
  while(\*((DWORD\*)packaddr) != 0)  
  {  
    DWORD dataAddr = \*(DWORD\*)packaddr;    //待解密地址  
    packaddr += 0x4;  
    DWORD buffsize = \*(DWORD\*)packaddr;  //解密缓冲区大小  
    packaddr += 0x4;  
      
    int codesize = buffsize \* 2;//+ 0x10E;//解密后缓冲区要大 10E  
    char \*codebuff = new char\[buffsize\];  
    memset(codebuff, 0, buffsize);  
  
    char \*addrbuff = new char\[codesize\];  
    memset(addrbuff, 0, codesize);  
  
    //解密代码  
    MyDecode(pNewPe + dataAddr, codebuff, addrbuff, buffsize);  
      
    //还原地址,只还原一次  
    static BOOL isFix = FALSE;  
    if(isFix == FALSE)  
    {  
      fixCode(codebuff, addrbuff, buffsize, bXorVal, isXor);  
      isFix = TRUE;  
    }  
    //将解密数据拷贝回区段  
    memcpy(pNewPe + dataAddr, codebuff, buffsize);  
  
    //删除缓存数据  
    delete codebuff;  
    delete addrbuff;  
  }  
    
  //修复节表  
  for(int i = 0; i < newFileHead->NumberOfSections; i++)  
  {  
    newSecInfo\[i\].SizeOfRawData = newSecInfo\[i\].Misc.VirtualSize;  
    newSecInfo\[i\].PointerToRawData = newSecInfo\[i\].VirtualAddress;  
  }  
    
  ////////--------准备新的 PE 文件  
  char szFileName\[1024\];  
  strcpy(szFileName, argv\[1\]);  
  char \*szStr = strstr(szFileName, ".exe");  
  strcpy(szStr, "\_unpack.exe");  
  FILE \*fp = NULL;  
  if((fp = fopen(szFileName, "wb+")) == NULL)  
  {  
    puts("文件创建失败！");  
  }  
  fwrite(pNewPe, 1, dwCount, fp);  
  if(fp != NULL)  
  {  
    fclose(fp);  
    fp = NULL;  
  }  
  puts("脱壳成功！");  
  ////////------------------------------  
  
  UnmapViewOfFile(lpFile);  
    CloseHandle(hFileMaping);  
  CloseHandle(hFile);  
  //system("pause");  
  return 0;  
}

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || UnAspack.rar  
 |