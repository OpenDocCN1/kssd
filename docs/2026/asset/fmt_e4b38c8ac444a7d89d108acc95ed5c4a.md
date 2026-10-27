
```
// 如果左边还有元素，对左边进行快速排序
if (nLeft < j)
{
  SortRun(pwSourceData1, pdwSourceData2, 0, j);
}

// 如果右边还有元素，对右边进行快速排序
if (nRight > i)
{
  SortRun(pwSourceData1, pdwSourceData2, i, nRight);
}
```

呼~，好累啊！也感谢你坚持看完。接下来的一篇就是大家最关心，也是最重要的输出表了，敬请关注下一篇：PE文件之旅 -- 加壳与脱壳的战场 输入表。

# 3 PE文件之旅 -- 第三篇 加壳与脱壳的战场 输入表(俯源码)

> PE文件之旅(C语言描述) 第三篇 -- 加壳与脱壳的战场 输出表

呼~，终于到输入表了，No Wasting Time, Let's Go! 还是讲原理先。输入表较前一篇的输出表稍微复杂一些，所谓复杂，在C语言里，无非就是多了几重指针，通过多重指针找啊找的才能访问到想要的东西。就好象你拿着钱找绑匪(T)，到了T指定的地点，接了个电话，T又指定了一个新的地点，于是你到了T指定的新的地点... 呵呵，开个玩笑。上一篇我们在`IMAGE_OPTIONAL_HEADER32.DataDirectory`的第一个数组里面找到了输出表，第二个元素描述的就是输入表的信息。大家还记得这个`DataDirectory`结构数组吧？它是`IMAGE_DATA_DIRECTORY`结构，有必要再重申一下它的定义：

```
typedef struct _IMAGE_DATA_DIRECTORY {
    DWORD   VirtualAddress;
    DWORD   Size;
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

想起来了吧，`ViretualAddress`就指向输出表(接到绑匪电话了，呵呵)。还等什么？我们开始吧！

核心代码：

```
// 首先定位IMAGE_OPTIONAL_HEADER32结构
IMAGE_OPTIONAL_HEADER32 myOptionalHeader;
fseek(pFile, (e_lfanew + sizeof(DWORD) + sizeof(IMAGE_FILE_HEADER)), SEEK_SET);     // 如此定位的原因请参考PE文件结构图
fread(&myOptionalHeader, sizeof(IMAGE_OPTIONAL_HEADER32), 1, pFile);

// 将输入表的VA转换为文件偏移
dwFileOffset = OnConvertVAToRawA(myOptionalHeader.DataDirectory[1].VirtualAddress);
if (dwFileOffset != -1)
{
  printf("输入表的文件偏移为：%08x\n", dwFileOffset);
  // 输入表由IMAGE_IMPORT_DESCRIPTOR结构数组开始，每一个被隐式调用的DLL文件都会有一个对应的IID结构数组
  // 但是没有字段指明IID数组的个数，也就是引入的DLL文件个数，但是这个IID数组的最后一个元素全为0，
  // 通过这一点可以计算出引入的DLL文件的个数，也就是IID结构数组的元素个数。
  // IMAGE_IMPORT_DESCRIPTOR结构定义：
  // typedef struct _IMAGE_IMPORT_DESCRIPTOR {
  //     union {
  //         DWORD   Characteristics;                 // 0 for terminating null import descriptor
  //         DWORD   OriginalFirstThunk;              // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
  //     };
  //     DWORD   TimeDateStamp;                   // 0 if not bound,
  //                                            // -1 if bound, and real date\time stamp
  //                                               // in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
  //                                               // O.W. date/time stamp of DLL bound to (Old BIND)
  //     DWORD   ForwarderChain;                  // -1 if no forwarders
  //     DWORD   Name;
  //     DWORD   FirstThunk;                      // RVA to IAT (if bound this IAT has actual addresses)
  // } IMAGE_IMPORT_DESCRIPTOR;
  // 现在已经获得了输入表的文件偏移，所以从这里开始就是输入表的IID数组了
  IMAGE_IMPORT_DESCRIPTOR IID;
  fseek(pFile, dwFileOffset, SEEK_SET);
  fread(&IID, sizeof(IMAGE_IMPORT_DESCRIPTOR), 1, pFile);
  // 得到IID数组个数先
  int nDllCount = 0;
  while ((0 != IID.Characteristics) || (0 != IID.FirstThunk) || (0 != IID.ForwarderChain) || (0 != IID.Name) || (0 != IID.OriginalFirstThunk) || (0 != IID.TimeDateStamp) )
  {
    nDllCount++;
    fread(&IID, sizeof(IMAGE_IMPORT_DESCRIPTOR), 1, pFile);
  }
  printf("导入的DLL个数为：%d\n", nDllCount);

  // 已经得到了导入的DLL个数，现在把他们读出来
  IMAGE_IMPORT_DESCRIPTOR *pIID = (IMAGE_IMPORT_DESCRIPTOR *)calloc(nDllCount, sizeof(IMAGE_IMPORT_DESCRIPTOR)); // prince 01
  fseek(pFile, dwFileOffset, SEEK_SET);
  fread(pIID, sizeof(IMAGE_IMPORT_DESCRIPTOR), nDllCount, pFile);

  // IID.Name中存放的就是DLL文件的名称的地址，注意是地址而不是以Zero结尾的字符串.
  // --------------------- 通过IID.Name的RVA读出DLL的名称 ----------------------- //
  DWORD *pNameFileOffset = (DWORD *)malloc(sizeof(DWORD) * nDllCount); // prince 02
  // 取出这些Name的VA转化为FileOffset存在pNamefileOffset中
  for (int i = 0; i < nDllCount; i++, pNameFileOffset++, pIID++)
  {
    *pNameFileOffset = OnConvertVAToRowA(pIID->Name);
  }
  // 恢复指针
  pIID            -= nDllCount;
  pNameFileOffset -= nDllCount;

  // 动态开辟指针数组pName,*pName指向存着DLL Name的pchName
  DWORD *pName = (DWORD *)calloc(nDllCount, sizeof(DWORD));  // prince 03 必须和04一起释放
  for (i = 0; i < nDllCount; i++, pName++)
  {
    // 每次开辟一个chName[16]用于存放Name
    char *pchName = (char *)malloc(sizeof(char) * 16);     // prince 04 必须和03一起释放
    // 将本次申请的内存的地址保存到*pName中
    *pName = (DWORD)pchName;
  }
  // 恢复指针
  pName -= nDllCount;
  // 以上动态分配内存的部分实际上是二重指针，这里这样分配好理解一些，下面再遇见同样的问题就直接声明多重指针了
  // 读取Name
  for (i = 0; i < nDllCount; i++, pNameFileOffset++, pName++)
  {
    DWORD dwDllName = *pNameFileOffset;
    fseek(pFile, dwDllName, SEEK_SET);
    char chTemp[16] = {0};
    fread((void *)*pName, 16, 1, pFile);
  }
  pNameFileOffset -= nDllCount;
  pName           -= nDllCount;

  if (pNameFileOffset != NULL)        // release prince 02
  {
    free(pNameFileOffset);
    pNameFileOffset = NULL;
  }
  // --------------- pName中就是每个DLL文件名称的文件偏移地址 ------------------- //

  // 读取IMAGE_IMPORT_DESCRIPTOR的OriginalFirstThunk(或Characteristics)的值，该值指向IMAGE_THUNK_DATA结构数组,
  // IMAGE_THUNK_DATA数组的每一个指针都指向IMAGE_IMPORT_BY_NAME结构
  // 动态开辟空间用来保存每个DLL的FistThunk值(文件偏移值)
  DWORD *pFirstThunkOfFileOffset = (DWORD *)calloc(nDllCount, sizeof(DWORD));  // prince 05
  char *pbByIndex = (char *)calloc(nDllCount, sizeof(char));                   // prince 06
  for (i = 0; i < nDllCount; i++, pbByIndex++)
  {
    *pbByIndex = 0;
  }
  pbByIndex -= nDllCount;  // 用来判断是否函数引入的标志

  // 读出OriginalFirstThunk所指向的值，该值高位的两个字节如果是0x8000，则说明该函数是以序号引入。
  // 低位的两个字节即为引入序号，比如该值为：0x80001234，那么就说明引入的是该DLL文件的第1234h号函数。
  DWORD *pdwRowAddressOfOriginalFirstThunk = (DWORD *)malloc(sizeof(DWORD));      // prince 23
  for (i = 0; i < nDllCount; i++, pIID++, pFirstThunkOfFileOffset++, pbByIndex++)
  {
    DWORD dwTemp = OnConvertVAToRawA(pIID->OriginalFirstThunk);
    fseek(pFile, dwTemp, SEEK_SET);
    fread(pdwRowAddressOfOriginalFirstThunk, sizeof(DWORD), 1, pFile);
    if (0 != *pdwRowAddressOfOriginalFirstThunk)
    {
      struct HightBit
      {
        int nHightBit : 1;
      }Bit;
      Bit.nHightBit = *pdwRowAddressOfOriginalFirstThunk >> 31;
      if (0 != Bit.nHightBit)  // 按序号引入函数
      {
        *pbByIndex = 1;
        *pFirstThunkOfFileOffset = OnConvertVAToRawA(pIID->OriginalFirstThunk);
      }
      else                     // 按名称引入函数
      {
        *pFirstThunkOfFileOffset = OnConvertVAToRawA(pIID->OriginalFirstThunk);
      }
    }
    else
    {
      *pFirstThunkOfFileOffset = OnConvertVAToRawA(pIID->FirstThunk);
    }
  }
  // 恢复指针
  pIID      -= nDllCount;
  pFirstThunkOfFileOffset -= nDllCount;
  pbByIndex               -= nDllCount;

  if (pdwRowAddressOfOriginalFirstThunk != NULL)               // release prince 23
  {
    free(pdwRowAddressOfOriginalFirstThunk);
    pdwRowAddressOfOriginalFirstThunk = NULL;
  }

  if (pIID != NULL)                        // release prince 01
  {
    free(pIID);
    pIID = NULL;
  }

  // -------------------- pFirstThunkOfFileOffset中是每个DLL入口的文件偏移值 -------------------- //

  // 同样，IMAGE_THUNK_DATA结构数组也是以全0作为结束，用这一点来计算出每个DLL文件的函数个数
  // --------------------------------- 读取每个DLL的函数的个数 ---------------------------------- //
  int *pnFunctionCountPerDll = (int *)calloc(nDllCount, sizeof(int)); // prince 07
  IMAGE_THUNK_DATA *pITD = (IMAGE_THUNK_DATA *)malloc(sizeof(IMAGE_THUNK_DATA)); // prince 08
  for (i = 0; i < nDllCount; i++, pnFunctionCountPerDll++, pFirstThunkOfFileOffset++)
  {
    fseek(pFile, *pFirstThunkOfFileOffset, SEEK_SET);
    fread(pITD, sizeof(IMAGE_THUNK_DATA), 1, pFile);
```
