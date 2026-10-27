
# [原创]写壳的一些成果[TLS完美处理,压缩功能实现,AntiDump-过LordPE,调用`sprintf`?,IAT重定向]

总结一下写壳过程的一些成果

如果阅读本文,发现一些框架性的东西难以理解,可以先阅读这两篇文章

`https://bbs.pediy.com/thread-206804.htm`

`https://bbs.pediy.com/thread-206873.htm`

本来想自己写个系列的,结果已经有前辈写好了,我就分享一下自己独有的吧

## 0. 大体思路

项目有加壳和解壳两部分,这两个部分通过结构体`PACKINFO PackInfo`来连接,具体实现是`PACKINFO g_PackInfo`定义在解壳的项目(dll项目,名为`Stub`)中,加壳部分通过

`(PPACKINFO)GetProcAddress(hStub, "g_PackInfo")`来操作这个结构体.

这样就能将原PE文件的信息存入到解壳部分,减轻解壳部分的信息获取压力,在某些置0操作后,解壳部分依然能获取到信息.而且还可以获取到具体的加壳选项信息.下面将这个结构体称为`PackInfo`

加壳部分最后将解壳部分`Stub.dll`的某些区段添加到原PE文件上

## 1. TLS完美处理

一开始处理TLS部分不知道怎么弄,看了看别人的帖子,结果豁然开朗,不就是循环调用TLS回调函数嘛.然后给自己来了个”高难度”的进行加壳,结果就出错了,果然是纸上得来终觉浅

测试程序是这样的:TLS回调函数首先使用TLS全局变量弹出对话框,接着主程序使用TLS全局变量弹出对话框,接着创建线程,线程中调用了`MessageBox`,其使用了TLS全局变量

```
// TLS测试程序
#include "stdafx.h"
#include "windows.h"

__declspec(thread) char g_tlsNum[10] = "ffff";

void NTAPI t_TlsCallBack_A(PVOID DllHandle, DWORD Reason, PVOID Red)
{
   if (DLL_PROCESS_ATTACH == Reason)
   {
       MessageBoxA(0, g_tlsNum, 0, 0);
   }
}

#pragma data_seg(".CRT$XLB")
PIMAGE_TLS_CALLBACK p_thread_callback[] = {
   t_TlsCallBack_A,
   NULL
};
#pragma data_seg()

DWORD WINAPI MyThreadProc(
   _In_ LPVOID lpParameter
)
{
   MessageBoxA(0, g_tlsNum, 0, 0);
   return 0;
}

int main()
{
   MessageBoxA(0, g_tlsNum, 0, 1);
   CreateThread(NULL, 0, MyThreadProc, NULL, 0, NULL);
   system("pause");
   return 0;
}
```

结果能弹出对话框,可是MessageBox上的字符是随机的.

![](https://bbs.kanxue.com/upload/attach/201904/761984_G988R4F52HYN5HA.jpg)

可以看到,这里的数据”ffff”和数据块开始和结束在汇编层面是看不出有什么关系的,后来查阅更多的资料得知,当线程创建时,系统会从数据块开始VA和结束VA这一块空间中读取内容保存到`pMem`中,`pMem`地址存在`fs:[0x2c]`指向的指针数组中的一个,而索引变量则是用于找到`pMem`.也就是说如果每创建一个线程,就会从数据块开始VA和结束VA读取一次,保存到当前线程的空间,内存地址保存到`fs:[0x2c]`指向的指针数组中,通过索引变量找到数据地址.

那么,如果一开始没能让系统正确读取到你的TLS数据块,主线程之后就在也不能正确使用TLS全局变量(如果再创建线程,并且TLS数据块已经恢复好了,依然可以正确使用TLS全局变量,此为测试所得,也符合各资料所言),所以必须一开始就构建好TLS.

接下来,怎么构建呢?

逐个分析:

首先看TLS表存在哪?

![](https://bbs.kanxue.com/upload/attach/201709/761984_i2iwyb5jz7rtdb5.png)

![](https://bbs.kanxue.com/upload/attach/201709/761984_nz21t9rsbkuecj8.png)

是的,它存在rdata段,那么考虑压缩功能,在解压之前必定是不能正确读取了.

处理方法:`Stub`工程中添加一个TLS全局变量,并做做样子使用一下

```
__declspec(thread) int g_num;    Stub初始化函数中:  g_num;//使用tls变量,产生tls节表,
```

这里还有一个前置条件,

```
#pragma comment(linker,"/merge:.data=.text")
#pragma comment(linker,"/merge:.rdata=.text")
#pragma comment(linker,"/section:.text,RWE")
```

也就是`Stub.dll`中`.rdata`段被合到`.text`段,这样只要拷贝了`.text`段(也是其他主要功能的载体)就可以使用其就可以其提供的tls表.

那么,首先就需要将原pe文件的目录表第10项指向`Stub`的这个tls表(由于会添加`Stub.dll`的text段作为解壳段,所以地址自然会有转换关系).

还需要将`Stub`的tls表中的值更改,接下来看TLS表中重要的值,

![](https://bbs.kanxue.com/upload/attach/201709/761984_1ewqypyh82hwhoo.png)

第一项和第二项其实就是代表TLS区段(注意这里是区段),所以处理是:TLS区段不能更改,原TLS表中的值设置到`Stub`的Tls表中

第三项:索引,有些文件存在.data段,有的文件中找不到VA对应的文件偏移,总之反正不在TLS表,也就是说会被压缩或被其他的处理,所以就在解析原pe文件的时候把它获取出来保存到信息结构体`PackInfo`中,这里取变量名为`TlsIndex`,将`Stub`的TLS表中索引地址指向变量`TlsIndex`,当然注意转化为VA

```
//如果在文件中找不到VA对应的内容,就说明会初始化为0,如果找得到,就读取其在文件中的内容
//获取tlsIndex的Offset
       DWORD indexOffset = RvaToOffset(g_lpTlsDir->AddressOfIndex - dwImageBase);
       //读取设置tlsIndex的值
       pPackInfo->TlsIndex = 0;//index一般默认值为0
       if(indexOffset != -1)
       {
           pPackInfo->TlsIndex = *(DWORD*)(indexOffset + m_pNewBuf);
       }
```

第四项:回调表VA,这个首先将`Stub`的TLS表的这一项设置为0,在解压缩等操作完成之后设置回去,然后手动调用

```
void CallTls()
{
   IMAGE_DOS_HEADER* lpDosHeader = (IMAGE_DOS_HEADER*)g_dwImageBase;
   IMAGE_NT_HEADERS* lpNtHeader = (IMAGE_NT_HEADERS*)(lpDosHeader->e_lfanew + g_dwImageBase);

   //如果tls可用,调用tls
   if(g_PackInfo.bIsTlsUseful == TRUE)
   {
       //将tls回调函数表指针设置回去
       PIMAGE_TLS_DIRECTORY pTlsDir =
          (PIMAGE_TLS_DIRECTORY)(lpNtHeader->OptionalHeader.DataDirectory[ 9 ].VirtualAddress + g_dwImageBase);
       pTlsDir->AddressOfCallBacks = g_PackInfo.TlsCallbackFuncRva;

      //手动调用TLS
       PIMAGE_TLS_CALLBACK* lptlsFun =
           (PIMAGE_TLS_CALLBACK*)(g_PackInfo.TlsCallbackFuncRva - lpNtHeader->OptionalHeader.ImageBase + g_dwImageBase);
       while((*lptlsFun) != NULL)
       {
           (*lptlsFun)((PVOID)g_dwImageBase , DLL_PROCESS_ATTACH , NULL);
           lptlsFun++;
       }
   }
}
```

这样关于TLS相关的问题就可以完美解决了

## 2. 压缩壳的实现

一开始实现压缩功能的时候虽然有思路,但暗暗感觉这背后处理的定是极其复杂.在看雪上搜索到了几篇文章

`https://bbs.pediy.com/thread-131361.htm`

`https://bbs.pediy.com/thread-161315.htm`

`https://bbs.pediy.com/thread-145947.htm`系列

扒下来apilib的使用代码后尝试按照自己的思路去写.

### 2.1 加壳部分压缩

考虑到对TLS全局变量的引用和程序启动是对资源段的使用,不压缩`tls`和`rsrc`段.

其中关于`tls`段的定位需要参考`tls`表中的数据起始位置`StartAddressOfRawData`或终止位置,

`rsrc`段参考目录表第三项.

```
//获得tls表指针
PIMAGE_TLS_DIRECTORY32 g_lpTlsDir = (PIMAGE_TLS_DIRECTORY32)(RvaToOffset(m_pNt->OptionalHeader.DataDirectory[9].VirtualAddress) + m_pNewBuf);

//获得tls数据起始rva ,用于判断tls区段位置
m_pTlsDataRva
= g_lpTlsDir->StartAddressOfRawData - m_pNt->OptionalHeader.ImageBase;

//用于判断资源段rva
m_pResRva = m_pNt->OptionalHeader.DataDirectory[2].VirtualAddress;
```

接着是我压缩过程中对这些区段进行的处理,

首先遍历区段,在此过程中:

① 获取`tls`和`rsrc`分别保存到`buf`中,并且做好标记;②获取其余要压缩的区段也保存到一个`CompressBuf`中,同时将这些区段的文件中大小`SizeofRawData`以及前后顺序`index`保存到交互结构体`PackInfo`,便于解压缩使用,也就是`DWORD PackInfomation[50][2]; // 压缩区段中每个区段的index和大小`

接着需要处理`CompressBuf`了,压缩代码是

```
PCHAR CPe::Compress(PVOID pSource, long lInLength, OUT long& lOutLenght)
{
   //packed保存压缩数据的空间，workmem为完成压缩需要使用的空间
   PCHAR packed, workmem;
   if((packed = (PCHAR)malloc(aP_max_packed_size(lInLength))) == NULL ||
       (workmem = (PCHAR)malloc(aP_workmem_size(lInLength))) == NULL)
   {
       return NULL;
   }

   //调用aP_pack压缩函数
   lOutLenght = aPsafe_pack(pSource, packed, lInLength, workmem, NULL, NULL);
   if(lOutLenght == APLIB_ERROR)
   {
       return NULL;
   }

   if(NULL != workmem)
   {
       free(workmem);
       workmem = NULL;
   }

   return packed; //返回保存地址
}
```

再接着,就是再造PE文件了,首先将Pe头复制到新的内存中,压缩区段的区段头的文件偏移和大小置为0,但是`Rva`和内存大小不动,这样起到占位的作用.(主要是之前懒得保存压缩区段总内存大小,并且还要专门创建一个对应的占位区段,不过这种也需要相应的处理,就是如果不加长pe头,后面再添加区段的时候,新添加的区段头可能会越过文件头范围).接着就是按照之前做的关于`tls`和`rsrc`的标记,将`tls`段和`rsrc`段按顺序原封不动复制到Pe头后面中,接着将`CompressBuf`添加到新区段.这样就完成了压缩部分功能.

### 2.2 解壳部分解压缩

解壳时,首先进行将压缩区段解压缩到`DecompressBuf`,根据`PackInfo`结构体中的压缩区段d的文件偏移大小和顺序,将其中的信息分别填到对应的区段即可

```
void Decompress()
{
   // 1.获取节区头首地址
   PIMAGE_DOS_HEADER pDosHeader = (PIMAGE_DOS_HEADER)g_dwImageBase;
   PIMAGE_NT_HEADERS pNtHeader = (PIMAGE_NT_HEADERS)(pDosHeader->e_lfanew + g_dwImageBase);
   PIMAGE_SECTION_HEADER pSecHeader = IMAGE_FIRST_SECTION(pNtHeader);

   // 2.解压压缩区段
   PCHAR lpPacked = ((PCHAR)g_dwImageBase + g_PackInfo.packSectionRva); //内存地址
   DWORD dwPackedSize = aPsafe_get_orig_size(lpPacked); //获取解压后的大小
   PCHAR lpBuffer = (PCHAR)g_VirtualAlloc(NULL, dwPackedSize, MEM_COMMIT, PAGE_EXECUTE_READWRITE); //申请内存
   aPsafe_depack(lpPacked, g_PackInfo.packSectionSize, lpBuffer, dwPackedSize); //解压

   // 3.将各区段还原回去
   DWORD offset = 0;
   for(int i = 0; i < g_PackInfo.PackSectionNumber; i++)
   {
       //区段的标号
       int index = g_PackInfo.PackInfomation[i][0];
       //这个区段的SizeOfRawData
       int size = g_PackInfo.PackInfomation[i][1];
       PCHAR destionVA = (PCHAR)g_dwImageBase + pSecHeader[index].VirtualAddress;
       PCHAR srcVA = lpBuffer + offset;
       memcpy_s(destionVA, size, srcVA, size);
       offset += size;
   }

   g_VirtualFree(lpBuffer, dwPackedSize, MEM_DECOMMIT);
}
```

最后提醒一下,请使用使用`aPsafe_pack`和`aPsafe_depack`,注意是对应版本的,不要使用`aP_pack`(好像叫这个名字),差点被坑死.

这部分只是说下我的思路,其实能优化的有很多,希望大家多多尝试.

## 3. AntiDump--过LordPE

关于AntiDump,首先学习了<<浅谈脱壳中的Dump技术全文>>一文,方法很简单,有修改PE头,修改`PEB->_LDR_MODULE`中的`SizeOfImage`,还有修改内存属性的,前面两个碰到`LordPE`就是死,后面的OD可以完美干掉.由于`LordPE`是读取文件路径自己解析文件来获取`IMAGE_SIZE`的,所以继续深入,这篇文章中说NT中不能在`PEB`中修改对应的路径来达到欺骗`LordPE`的目的,然后我又搜索了一些隐藏进程的文章,就试一试咯

结果发现,在处理完`PEB`的相关字段后,虽然不能在任务管理器中”消失”,但是`LordPE`进程列表中已经找不到这个进程了,这样也就够了

代码很简单:

```
//在解压缩之前进行

int AntidumpFunc1()
{
   PPEB pPeb;
   _asm
   {
       mov eax, fs:[0x30];                   //获得PEB地址
       mov pPeb, eax;
   }

   PLDR_MODULE pLdrModule =
       (PLDR_MODULE)(pPeb->LoaderData->InLoadOrderModuleList.Flink);
```
