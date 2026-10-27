
```c
int nFunctionCount = 0;

while ((pITD->u1.ForwarderString != 0) || (pITD->u1.Function != 0) || (pITD->u1.Ordinal != 0) || (pITD->u1.AddressOfData != 0))
{
  nFunctionCount++;
  fread(pITD, sizeof(IMAGE_THUNK_DATA), 1, pFile);
}
*pnFunctionCountPerDll = nFunctionCount;
}
if (pITD != NULL)                    // release prince 08
{
  free(pITD);
  pITD = NULL;
}

pFirstThunkOfFileOffset -= nDllCount;
pnFunctionCountPerDll  -= nDllCount;
// ---------------- pnFunctionCountPerDll中存放的就是每个DLL中引入的函数个数 ------

---------- //

// 由于DLL文件的个数，每个DLL文件的函数个数都是不固定的，所以这里我用了三重指针。
// 也许用了三重指针使下面的程序难懂了一些，但是我认为这是最好的方法了。
// ------------------------------ 读取每个DLL文件中函数的名称 ---------------------

---------- //
DWORD ***pdwFunctionOfDll = (DWORD ***)calloc(nDllCount, sizeof(DWORD**));

// prince 09 必须和10，11一起释放
for (i = 0; i < nDllCount; i++, pdwFunctionOfDll++, pnFunctionCountPerDll++)
{
  DWORD **pdwFileOffsetOfFunction = (DWORD **)calloc(*pnFunctionCountPerDll, sizeof(char*));  // prince 10 必须和09，11一起释放
  *pdwFunctionOfDll = pdwFileOffsetOfFunction;
  for (int j = 0; j < *pnFunctionCountPerDll; j++, pdwFileOffsetOfFunction++)
  {
    char *pchFunctionName = (char *)malloc(sizeof(char) * 64);
    // prince 11 必须和09，10一起释放
    *pdwFileOffsetOfFunction = (DWORD *)pchFunctionName;
  }
}
pdwFunctionOfDll    -= nDllCount;
pnFunctionCountPerDll  -= nDllCount;

// ----------------------- 确定哪几个DLL文件是序号引入 ----------------------------

// 
int nByIndexDllCount      = 0;
for (i = 0; i < nDllCount; i++, pbByIndex++)
{
  if (0 != *pbByIndex)
  {
    nByIndexDllCount++;
  }
}
pbByIndex -= nDllCount;
// ----------------------- 有nByIndexDllCount个DLL文件是由序号引入的 --------------

// ----------------------- 再求出每个由序号引入的DLL的函数个数 --------------------

// 
int *pnByIndexDllFunctionsCount = (int *)calloc(nByIndexDllCount, sizeof(int));
// prince 24
for (i = 0; i < nDllCount; i++, pbByIndex++, pnFunctionCountPerDll++)
{
  if (0 != *pbByIndex)
  {
    *pnByIndexDllFunctionsCount = *pnFunctionCountPerDll;
    pnByIndexDllFunctionsCount++;
  }
}
pbByIndex           -= nDllCount;
pnFunctionCountPerDll     -= nDllCount;
pnByIndexDllFunctionsCount -= nByIndexDllCount;
// ------------------- pnByIndexDllFunctionsCount所指向的就是每个由序号引入的DLL文件
// 的函数个数 ---------------- //

// ------------------ 动态开辟内存用来保存由序号引入的函数序号 --------------------

// 
DWORD **pdwFunctionIndex = (DWORD **)calloc(nByIndexDllCount, sizeof(DWORD *));
// prince 25
for (i = 0; i < nByIndexDllCount; i++, pnByIndexDllFunctionsCount++, pdwFunctionIndex++)
{
  DWORD *pdwFunctionIndexTmp = (DWORD *)calloc(*pnByIndexDllFunctionsCount, sizeof(DWORD));    // prince 26
  *pdwFunctionIndex = pdwFunctionIndexTmp;
}
pnByIndexDllFunctionsCount -= nByIndexDllCount;
pdwFunctionIndex           -= nByIndexDllCount;
// --------------------------------------------------------------------------------

------
for (i = 0; i < nDllCount; i++, pFirstThunkOfFileOffset++, pnFunctionCountPerDll++, pdwFunctionOfDll++, pbByIndex++)
{
  IMAGE_THUNK_DATA *pITD = (IMAGE_THUNK_DATA *)calloc(*pnFunctionCountPerDll, sizeof(IMAGE_THUNK_DATA)); // prince 12
  fseek(pFile, *pFirstThunkOfFileOffset, SEEK_SET);
  fread(pITD, sizeof(IMAGE_THUNK_DATA), *pnFunctionCountPerDll, pFile);
  DWORD dwFileOffsetOfFunction;
  if (0 == *pbByIndex)
  {
    for (int j = 0; j < *pnFunctionCountPerDll; j++, pITD++)
    {
      dwFileOffsetOfFunction = OnConvertVAToRowA((DWORD)pITD->u1.AddressOfData);
      fseek(pFile, dwFileOffsetOfFunction + 2, SEEK_SET);
      fread((void *)**pdwFunctionOfDll, 64, 1, pFile);
      strText.Format("%s", **pdwFunctionOfDll);
      (*pdwFunctionOfDll)++;
    }
    pITD -= *pnFunctionCountPerDll;
    *pdwFunctionOfDll -= *pnFunctionCountPerDll;
  }
  else
  {
    for (int j = 0; j < *pnByIndexDllFunctionsCount; j++, pITD++, (*pdwFunctionIndex)++)
    {
      **pdwFunctionIndex = (DWORD)pITD->u1.AddressOfData;
    }
    pITD -= *pnByIndexDllFunctionsCount;
    *pdwFunctionIndex -= *pnByIndexDllFunctionsCount;
    pnByIndexDllFunctionsCount++;
    pdwFunctionIndex++;
  }

  if (pITD != NULL)                         // release prince 12
  {
    free(pITD);
    pITD = NULL;
  }
}

if (nByIndexDllCount != 0)
{
  pnByIndexDllFunctionsCount -= nByIndexDllCount;
  pdwFunctionIndex           -= nByIndexDllCount;
}

pdwFunctionOfDll    -= nDllCount;
pFirstThunkOfFileOffset -= nDllCount;
pnFunctionCountPerDll   -= nDllCount;
pbByIndex               -= nDllCount;

if (pFirstThunkOfFileOffset != NULL)               // release prince 05
{
  free(pFirstThunkOfFileOffset);
  pFirstThunkOfFileOffset = NULL;
}
// -------------------------- 三重指针pdwFunctionOfDll最终指向函数名称 ------------
// ---------------- //

// --------------------- 处理由序号引入的函数的序号 --------------------------------
// 

for (i = 0; i < nDllCount; i++, pName++, pnFunctionCountPerDll++, pdwFunctionOfDll++, pbByIndex++)
{
  prince("DLL %d 名字：%s\n", (i + 1), *pName);
  prince("调用函数个数：%d\n", *pnFunctionCountPerDll);
  if (0 == *pbByIndex)
  {
    for (int j = 0; j < *pnFunctionCountPerDll; j++, (*pdwFunctionOfDll)++)
    {
      prince("%s\n", **pdwFunctionOfDll);
    }
    *pdwFunctionOfDll -= *pnFunctionCountPerDll;
  }
  else
  {
    prince("此DLL文件中函数由序号引入!");
    for (int j = 0; j < *pnByIndexDllFunctionsCount; j++, (*pdwFunctionIndex)++)
    {
      prince("引入序号: %04x\n", ((**pdwFunctionIndex) & 0xffff));
      // 取0x80000000的后两个字节
      m_ListBox1.AddString(strText);
    }
    *pdwFunctionIndex -= *pnByIndexDllFunctionsCount;
    pnByIndexDllFunctionsCount++;
    pdwFunctionIndex++;
  }
}

// 恢复指针
pName          -= nDllCount;
pnFunctionCountPerDll -= nDllCount;
pdwFunctionOfDll      -= nDllCount;
pbByIndex             -= nDllCount;

pnByIndexDllFunctionsCount -= nByIndexDllCount;
pdwFunctionIndex           -= nByIndexDllCount;

for (i = 0; i < nDllCount; i++, pName++)                   // release prince 04
{
  if (*pName != NULL)
  {
    free((void *)*pName);
    *pName = NULL;
  }
}
pName -= nDllCount;
if (pName != NULL)                                        // release prince 03
{
  free(pName);
  pName = NULL;
}

if (pbByIndex != NULL)          // release prince 06
{
  free(pbByIndex);
  pbByIndex = NULL;
}

for (i = 0; i < nDllCount; i++, pdwFunctionOfDll++, pnFunctionCountPerDll++)     //
  // release prince 11
{
  for (int j = 0; j < *pnFunctionCountPerDll; j++, (*pdwFunctionOfDll)++)
  {
    if (**pdwFunctionOfDll != NULL)
    {
      free((void *)**pdwFunctionOfDll);
      **pdwFunctionOfDll = NULL;
    }
  }
  *pdwFunctionOfDll -= *pnFunctionCountPerDll;
}
pnFunctionCountPerDll -= nDllCount;
pdwFunctionOfDll      -= nDllCount;

for (i = 0; i < nDllCount; i++, pdwFunctionOfDll++)     // release prince 10
{
  if (*pdwFunctionOfDll != NULL)
```
