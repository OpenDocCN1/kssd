
```
  for (i = 0; i < nDllCount; i++, pName++, pnFunctionCountPerDll++,
pdwFunctionOfDll++, pbByIndex++)
  {
    prince("DLL %d 名字：%s\n", (i + 1), *pName);
    prince("调用函数个数：%d\n", *pnFunctionCountPerDll);
    if (0 == *pbByIndex)
    {
      for (int j = 0; j < *pnFunctionCountPerDll; j++,
(*pdwFunctionOfDll)++)
      {
        prince("%s\n", **pdwFunctionOfDll);
      }
      *pdwFunctionOfDll -= *pnFunctionCountPerDll;
    }
    else
    {
      prince("此 DLL 文件中函数由序号引入!");
      for (int j = 0; j < *pnByIndexDllFunctionsCount; j++,
(*pdwFunctionIndex)++)
      {
        prince("引入序号: %04x\n", (**pdwFunctionIndex) & 0xffff);
      // 取 0x80000000 的后两个字节
        m_ListBox1.AddString(strText);
      }
      *pdwFunctionIndex -= *pnByIndexDllFunctionsCount;
      pnByIndexDllFunctionsCount++;
      pdwFunctionIndex++;
    }
  }

  // 恢复指针
  pName           -= nDllCount;
  pnFunctionCountPerDll -= nDllCount;
  pdwFunctionOfDll     -= nDllCount;
  pbByIndex         -= nDllCount;

  pnByIndexDllFunctionsCount -= nByIndexDllCount;
  pdwFunctionIndex           -= nByIndexDllCount;

  for (i = 0; i < nDllCount; i++, pName++)               // release prince 04
  {
    if (*pName != NULL)
    {
      free((void *)*pName);
      *pName = NULL;
    }
  }
  pName -= nDllCount;
  if (pName != NULL)                               // release prince 03
  {
    free(pName);
    pName = NULL;
  }

  if (pbByIndex != NULL)    // release prince 06
  {
    free(pbByIndex);
    pbByIndex = NULL;
  }

  for (i = 0; i < nDllCount; i++, pdwFunctionOfDll++, pnFunctionCountPerDll++)  //
release prince 11
  {
    for (int j = 0; j < *pnFunctionCountPerDll; j++, (*pdwFunctionOfDll)++)
    {
      if (**pdwFunctionOfDll != NULL)
      {
        free((void **)**pdwFunctionOfDll);
        **pdwFunctionOfDll = NULL;
      }
    }
    *pdwFunctionOfDll -= *pnFunctionCountPerDll;
  }
  pnFunctionCountPerDll -= nDllCount;
  pdwFunctionOfDll      -= nDllCount;

  for (i = 0; i < nDllCount; i++, pdwFunctionOfDll++)   // release prince 10
  {
    if (*pdwFunctionOfDll != NULL)
    {
      free(*pdwFunctionOfDll);
      *pdwFunctionOfDll = NULL;
    }
  }
  pdwFunctionOfDll -= nDllCount;

  if (pdwFunctionOfDll != NULL)                  // release prince 09
  {
    free(pdwFunctionOfDll);
    pdwFunctionOfDll = NULL;
  }


  if (pnFunctionCountPerDll != NULL)    //
release prince 07
  {
    free(pnFunctionCountPerDll);
    pnFunctionCountPerDll = NULL;
  }

  if (pnByIndexDllFunctionsCount != NULL)            // release prince 24
  {
    free(pnByIndexDllFunctionsCount);
    pnByIndexDllFunctionsCount = NULL;
  }

  for (i = 0; i < nByIndexDllCount; i++, pdwFunctionIndex++)  // release prince 26
  {
    if (*pdwFunctionIndex != NULL)
    {
      free(*pdwFunctionIndex);
      *pdwFunctionIndex = NULL;
    }
  }
  pdwFunctionIndex -= nByIndexDllCount;

  if (pdwFunctionIndex != NULL)                     // release prince 25
  {
    free(pdwFunctionIndex);
    pdwFunctionIndex = NULL;
  }
}
else
{
  printf("输入表定位错误！");
}


// 计算 VA 到文件偏移的子程序
DWORD OnConvertVAToRawA(DWORD dwFileOffset)
{
  IMAGE_SECTION_HEADER *pmySectionHeader = (IMAGE_SECTION_HEADER *)calloc
(m_nSectionCount, sizeof(IMAGE_SECTION_HEADER)); // prince 14
  fseek(pFile, (e_lfanew + 4 + sizeof(IMAGE_FILE_HEADER) + sizeof
(IMAGE_OPTIONAL_HEADER32)), SEEK_SET);
  fread(pmySectionHeader, sizeof(IMAGE_SECTION_HEADER), m_nSectionCount, pFile);

  DWORD dwFilePos;
  DWORD dwOffset;
  DWORD *pdwVA   = (DWORD *)malloc(sizeof(DWORD) * m_nSectionCount);   // prince 15
  DWORD *pdwRowA = (DWORD *)malloc(sizeof(DWORD) * m_nSectionCount);   // prince 16
  for (int i = 0; i < m_nSectionCount; i++, pmySectionHeader++, pdwVA++, pdwRowA++)
  {
    *pdwVA   = pmySectionHeader->VirtualAddress;
    *pdwRowA = pmySectionHeader->PointerToRawData;
  }

  pmySectionHeader -= m_nSectionCount;

  pdwVA      -= m_nSectionCount;
  pdwRowA      -= m_nSectionCount;

  for (i = 0; i < m_nSectionCount; i++, pdwVA++, pdwRowA++)
  {
    if ((dwFileOffset >= *pdwVA) && (dwFileOffset < *(pdwVA + 1)))
    {
      dwOffset  = *pdwVA - *pdwRowA;
      dwFilePos = dwFileOffset - dwOffset;

      pdwVA   -= i;
      pdwRowA -= i;
      if (pdwVA != NULL)                 // release prince 15
      {
        free(pdwVA);
        pdwVA = NULL;
      }
      if (pdwRowA != NULL)               // release prince 16
      {
        free(pdwRowA);
        pdwRowA = NULL;
      }

      if (pmySectionHeader != NULL)      // release prince 14
      {
        free(pmySectionHeader);
        pmySectionHeader = NULL;
      }

      return dwFilePos;
    }
  }
  pdwVA   -= m_nSectionCount;
  pdwRowA -= m_nSectionCount;

  if (pmySectionHeader != NULL)      // release prince 14
  {
    free(pmySectionHeader);
    pmySectionHeader = NULL;
  }

  if (pdwVA != NULL)                 // release prince 15
  {
    free(pdwVA);
    pdwVA = NULL;
  }

  if (pdwRowA != NULL)               // release prince 16
  {
    free(pdwRowA);
    pdwRowA = NULL;
  }

  return -1;
}


 PE 文件之旅全部系列到此就告一段落了。在将近一个月的时间里通过对 `PE` 文件格式的研究让我从正向的角度全面的了解了 `EXE` 文件执行的有关细节，同时对我自己而言也是得到了锻炼和提高，无奈编程水平有限，这篇堆砌起来的垃圾代码还有着严重的不雅和缺陷，希望高手看了之后能腾出宝贵的时间对我指点一二；同时也希望对我有着很大帮助和提高的这几篇小小的心得能让广大和我一样渴求知识和追求软件技术的小菜们得到自己真正想要的东西。

 还是那句老话，抛砖引玉，我是就着《《`PE` 知识学习》》写下去的。接下去我会继续研究 `PE` 文件的资源，看看资源的压缩原理，也就是压缩壳的原理，等有了成果再和大家分享。在这里希望大家在学习知识的同时继续把思维发展下去，真正在技术的层面上提高自己。

 现将全部 `VC` 源代码发布，希望同大家进行技术交流，程序写的较烂，请指教。

`4dd466d0e1e0cc123afbe504da2462ca`
