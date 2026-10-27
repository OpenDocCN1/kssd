
软件断点是通过修改目标地址代码为`0xCC`（INT3/BreakpointInterrupt）来设置的断点。通过在受保护的代码段和（或）API函数中扫描字节`0xCC`来识别软件断点。这里以普通断点和函数断点分别举例。

# 二.程序一实现

## 1. MFC实现一个CreakMe

这部分是参考C++中文网的demo和它的消息机制，具体代码见后面附件或者打包文件，不做过多阐述。

## 2. SMC加密技术

（注：这部分只展示实现的情况和相关代码，测试阶段会比较调试的时候区别）

首先把需要保护的加密算法保护起来：

```
#pragma code_seg(".JIANG")

void fun()
{
       MessageBox(NULL, TEXT("wrong"), TEXT("错误"), MB_ICONINFORMATION);
}

string Fun1(string userkey, string cpu)  //处理之后的用户名，和CPU型号  处理序列号函数
{
       string name = "kihnhhkiilh";  //20171120051
       char a[50];
       char CPU[50];
       strcpy_s(CPU, cpu.c_str());
       for (int i = 0; i < name.length(); i++)
       {
              char k = name[i] ^ 'Y';
              a[i] = CPU[hex2int(k)];
       }
       for (int i = 0; i < name.length(); i++)
       {
              char k = name[i] ^ 'Y';
              string s(1, a[i]);
              userkey.insert(hex2int(k), s);
       }
       return userkey;
}

#pragma code_seg()
#pragma comment(linker, "/SECTION:.JIANG,ERW")
```

面对之前的问题，就是`try_except`里面不能放置类，所以这里在后面解密阶段直接进行解密，不进行try结构来检查函数是否解密完成。解密（加密）操作就是遍历找到区块，然后进行先解密一个字符串，再和这个字符串进行循环异或，再进行移位运算。

遍历找区段名字：

```
void SMC_De(char* pBuf, char* key)     //SMC解密函数
{
       const char* szSecName = ".JIANG";
       short nSec;
       PIMAGE_DOS_HEADER pDosHeader;
       PIMAGE_NT_HEADERS pNtHeader;
       PIMAGE_SECTION_HEADER pSec;
       pDosHeader = (PIMAGE_DOS_HEADER)pBuf;
       pNtHeader = (PIMAGE_NT_HEADERS)& pBuf[pDosHeader->e_lfanew];
       nSec = pNtHeader->FileHeader.NumberOfSections;
       pSec = (PIMAGE_SECTION_HEADER)& pBuf[sizeof(IMAGE_NT_HEADERS) + pDosHeader->e_lfanew];
       for (int i = 0; i < nSec; i++)
       {
              if (strcmp((char*)& pSec->Name, szSecName) == 0)
              {
                     int pack_size;
                     char* packStart;
                     pack_size = pSec->SizeOfRawData;
                     packStart = &pBuf[pSec->VirtualAddress];
                     xorPlus(packStart, pack_size, key, strlen(key));
                     return;
              }
              pSec++;
       }
}
```

然后调用异或和移位加密算法：

```
void xorPlus(char* soure, int dLen, char* Key, int Klen)   //异或之后再移位
{
       for (int i = 0; i < dLen;)
       {
              for (int j = 0; (j < Klen) && (i < dLen); j++, i++)
              {
                     soure[i] = soure[i] ^ Key[j];
                     soure[i] = ~soure[i];
              }
       }
       for (int i = 0; i < dLen; i++)
       {
              char m = soure[i];
              soure[i] = soure[dLen - i - 1];
              soure[dLen - i - 1] = m;
       }
}
```

加密的字符串要先进行解密操纵再待到后面运算：

```
string KeyBuffer = "86y­6+4y ,787y

 70/<+*0- y,4kihnhhkiilh";
       for (int i = 0; i <KeyBuffer.length(); i++)
       {
              KeyBuffer[i] = KeyBuffer[i] ^ 'Y';
       }
```

## 3. SMC外部加密程序

首先选择需要进行加密的文件，这部分代码和之前实验的一样：

```
//取得文件路径部分
       TCHAR szFilePath[MAX_PATH];
       OPENFILENAME ofn = { 0 };
       memset(szFilePath, 0, MAX_PATH);
       memset(&ofn, 0, sizeof(ofn));
       ofn.lStructSize = sizeof(ofn);
       ofn.hwndOwner = NULL;
       ofn.hInstance = GetModuleHandle(NULL);
       ofn.nMaxFile = MAX_PATH;
       ofn.lpstrInitialDir = ".";
       ofn.lpstrFile = szFilePath;
       ofn.lpstrTitle = "选择PE文件";
       ofn.Flags = OFN_PATHMUSTEXIST | OFN_FILEMUSTEXIST | OFN_HIDEREADONLY;
       ofn.lpstrFilter = "(*.*)\0*.exe;*.dll\0";
       GetOpenFileName(&ofn);
       if (szFilePath == NULL)
       {
              MessageBox(NULL, "打开文件错误", NULL, NULL);
              //return 0;
       }
       //创建文件句柄
       HANDLE hFile;
       char KeyBuffer[MAX_PATH] = "ao Form yunNan University Num：20171120051";
       hFile = CreateFile(szFilePath, GENERIC_READ | GENERIC_WRITE, 0, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, 0);
       if (hFile == INVALID_HANDLE_VALUE)
       {
              MessageBox(NULL, TEXT("打开文件失败"), NULL, NULL);
              return;
       }
```

打开程序之后就跟解密的思路是一样的了，先找到需要进行加密的区段，然后对区段进行异或和移位加密：

```
void xorPlus(char* soure, int dLen, char* Key, int Klen)
{
       for (int i = 0; i < dLen;)
       {
              for (int j = 0; (j < Klen) && (i < dLen); j++, i++)
              {
                     soure[i] = soure[i] ^ Key[j];
                     soure[i] = ~soure[i];
              }
       }
       for (int i = 0; i < dLen; i++)
       {
              char m = soure[i];
              soure[i] = soure[dLen - i - 1];
              soure[dLen - i - 1] = m;
       }
}

void SMC(HANDLE hFile, char* key)
{
       // SMC 加密XX区段
       HANDLE hMap;
       const char* szSecName = ".JIANG";
       char* pBuf;
       int size;
       short nSec;
       PIMAGE_DOS_HEADER pDosHeader;
       PIMAGE_NT_HEADERS pNtHeader;
       PIMAGE_SECTION_HEADER pSec;
       size = GetFileSize(hFile, 0);
       hMap = CreateFileMapping(hFile, NULL, PAGE_READWRITE, 0, size, NULL);
       if (hMap == INVALID_HANDLE_VALUE)
       {
       _viewf:
              MessageBox(NULL, TEXT("映射失败"), NULL, NULL);
              return;
       }
       pBuf = (char*)MapViewOfFile(hMap, FILE_MAP_WRITE | FILE_MAP_READ, 0, 0, size);
       if (!pBuf) goto _viewf;
       pDosHeader = (PIMAGE_DOS_HEADER)pBuf;
       pNtHeader = (PIMAGE_NT_HEADERS)& pBuf[pDosHeader->e_lfanew];
       if (pNtHeader->Signature != IMAGE_NT_SIGNATURE)
       {
              MessageBox(NULL, TEXT("不是有效的win32 可执行文件"), NULL, NULL);
              goto _clean;
       }
       nSec = pNtHeader->FileHeader.NumberOfSections;
       pSec = (PIMAGE_SECTION_HEADER)& pBuf[sizeof(IMAGE_NT_HEADERS) + pDosHeader->e_lfanew];
       for (int i = 0; i < nSec; i++)
       {
              if (strcmp((char*)& pSec->Name, szSecName) == 0)
              {
                     int pack_size;
                     char* packStart;
                     pack_size = pSec->SizeOfRawData;
                     packStart = &pBuf[pSec->PointerToRawData];
                     xorPlus(packStart, pack_size, key, strlen(key));
                     MessageBox(NULL, TEXT("加密成功"), NULL, NULL);
                     goto _clean;
              }
              pSec++;
       }
       MessageBox(NULL, TEXT("未找到JIANG区段，加密失败"), NULL, NULL);
_clean:
       UnmapViewOfFile(pBuf);
       CloseHandle(hMap);
       return;
}
```

这个只能加密自己需要保护的程序，保护不了加密不了其他程序。

## 4. 花指令

### 1. 花指令1，函数跳转

有很多花指令的设计思路是用垃圾参数和字节来填充空间，然后利用反汇编工具的工作模式不能正确解读造成干扰。我设计的思路是去干扰调试者，能让反汇编工具正常工作而且函数是正常，但是函数没有实际意义，而是一堆循环。

首先是定义三个基本函数，这三个基本函数的出口都是一个具体的数值，但是出口数据可以影响另一个函数的运行，不管怎么相互调用，最后的结果都是1，平坦化设计思路。三个基本函数如下：

```
#define FUN0                 lo0  //垃圾函数
#define FUN1                 lo1
#define FUN2                 lo2
#define FUN3                 lo3

static inline int FUN0(void)
{
       volatile int i = 138, j = 1949;
       if ((i++) % 2 > 0)   j *= i;
       if (j < 0)  i *= 2;
       else return 0;
       i = 1;
       while (i++ < 2) { j /= i; j++; i++; }
       return i;
}

static inline int FUN1(void)
{
       volatile int i = 21, j = 75;
       if ((i--) % 3 > 0)     j *= i;
       if (j > 1)  i *= 3;
       else return 1;
       i = 1;
       while (i++ < 3) { j /= i; j--; i++; }
       return j;
}

static inline int FUN2(void)
{
       volatile int i = 56, j = 17;
       if ((i--) % 5 > 0)     j *= i;
       if (j > 2)  i *= 5;
       else return 0;
       i = 1;
       while (i++ < 5) { j *= i;  j += 3; i += 3; }
       return i;
}

static inline int FUN3(void)
{
       volatile int i = 1909, j = 131;
       if ((i--) % 7 > 0)     j *= i;
       if (j > 3)  i *= 7;
       else return 1;
       i = 1;
       while (i++ < 7) { j /= i;   j -= 5; i += 5; }
       return i;
}
```

然后定义的花指令就可以这样调用：

```
#define _FLOWER_FUN_0 {if(FUN2())FUN1();if(FUN0()) FUN3();if(FUN1()) FUN2();if(FUN3()) FUN1(); \
```

