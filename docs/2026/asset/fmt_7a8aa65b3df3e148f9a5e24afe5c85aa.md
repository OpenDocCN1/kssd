
```
MyRelocation.h  MyRelocation.cpp

void MyRelocation::ShowRelocation()
{
  AnalPE.pDatD=&(AnalPE.pOptH->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC]);//也就是5
  PIMAGE_BASE_RELOCATION pBasR=(PIMAGE_BASE_RELOCATION)((long)AnalPE.buf+AnalPE.CalcOffset(AnalPE.pDatD->VirtualAddress));
  RELOCAREAINFO Temp={0};
  int j=0;//索引
  CString str;

  while (pBasR->VirtualAddress)
  {
    //printf_s("索引：%d  ",j);
    str.Format(L"%d",j+1);
    m_FirstReloc.InsertItem(j,str);
    //区域的虚拟地址输出或获取到其他地方
    Temp.dwAreaRVA=pBasR->VirtualAddress;

    RelCalcOffset(Temp.dwAreaRVA,AnalPE.pNtH,AnalPE.buf,Temp.szSectionName,NULL);
  //  printf_s("%s   ",Temp.szSectionName);
    CString str(Temp.szSectionName);
    m_FirstReloc.SetItemText(j,1,str);
    //printf_s("%08X   ",pBasR->VirtualAddress);
    str.Format(L"%08X",pBasR->VirtualAddress);
    m_FirstReloc.SetItemText(j,2,str);
    Temp.NumberOfReloc=(pBasR->SizeOfBlock-sizeof(IMAGE_BASE_RELOCATION))/2;
  //  printf_s("%02X  %d\n", Temp.NumberOfReloc, Temp.NumberOfReloc);
    str.Format(L"%02X",Temp.NumberOfReloc);
    CString str1;
    str1.Format(L"%d",Temp.NumberOfReloc);
    m_FirstReloc.SetItemText(j,3,str+L" "+str1);
    //dwCount=Temp.NumberOfReloc;

    m_VecpBseRInfo.push_back(pBasR);
    //讲一个区域重定位信息添加
    m_VecRelocAreaInfo.push_back(Temp);
    pBasR=(PIMAGE_BASE_RELOCATION)((long)pBasR+pBasR->SizeOfBlock);
    j++;
  }
  int i=0;
}

void MyRelocation::OnNMClickList(NMHDR *pNMHDR,LRESULT *pResult)
{
  POSITION pos = m_FirstReloc.GetFirstSelectedItemPosition();
  int nIndex=(int)pos;
  if (pos == NULL)//判断是否为空
  {
    return;
  }
  m_SecondReloc.DeleteAllItems();
  int i=0;
  CString str;

  //得到第三个成员的TYPEOFFEST起始位置 之后开始循环获取重定位信息
  PTYPEOFFSET pOffset=(PTYPEOFFSET)((long)m_VecpBseRInfo[nIndex-1]+sizeof(IMAGE_BASE_RELOCATION));
  int z=m_VecRelocAreaInfo[nIndex-1].NumberOfReloc;
  for (DWORD i=0;i<m_VecRelocAreaInfo[nIndex-1].NumberOfReloc;i++)
  {
    //printf_s("索引：%d  ",i);
    str.Format(L"%d",i+1);
    m_SecondReloc.InsertItem(i,str);
    RELOCINFO RelocInfoTemp={0};
    //需要重定位的虚拟地址（RVA）
    RelocInfoTemp.dwRelocRVA=m_VecRelocAreaInfo[nIndex-1].dwAreaRVA+pOffset->Offset;
  //  printf_s("RVA：%08X  ", RelocInfoTemp.dwRelocRVA);
    str.Format(L"%08X",RelocInfoTemp.dwRelocRVA);
    m_SecondReloc.SetItemText(i,1,str);
    //根据相对虚拟地址算出的文件偏移
    RelocInfoTemp.dwOffset=RelCalcOffset(RelocInfoTemp.dwRelocRVA,AnalPE.pNtH,NULL,NULL);
  //  printf_s("偏移：%08X  ", RelocInfoTemp.dwOffset);
    str.Format(L"%08X", RelocInfoTemp.dwOffset);
    m_SecondReloc.SetItemText(i,2,str);
    //重定位方式
    RelocInfoTemp.bType=pOffset->Type;
    //printf_s("类型：HIGHLOW (%d)  ",RelocInfoTemp.bType);
    str.Format(L"HIGHLOW (%d)", RelocInfoTemp.bType);
    m_SecondReloc.SetItemText(i,3,str);
    //从算出的文件偏移取出的数据 这个数据就是需要重定位的虚拟地址(VA)
    RelocInfoTemp.dwRelocValue=*(PDWORD)((long)AnalPE.buf+RelocInfoTemp.dwOffset);
    //printf_s("FAR地址：%08X  ",RelocInfoTemp.dwRelocValue);
    str.Format(L"%08X", RelocInfoTemp.dwRelocValue);
    m_SecondReloc.SetItemText(i,4,str);
    //RelocInfoTemp.bData
    DWORD VA= RelCalcOffset(RelocInfoTemp.dwRelocValue,AnalPE.pNtH,AnalPE.buf,NULL,(PCHAR)RelocInfoTemp.bData,2);
    RelCalcOffset(VA,AnalPE.pNtH,AnalPE.buf,NULL,(PCHAR)RelocInfoTemp.bData,1);
    CString str1;
    CString str2;
    for (int j=0;j<10;j++)
    {
      //判断是否在有效字符32-126区间内
      if (32<=(RelocInfoTemp.bData)[0]&&(RelocInfoTemp.bData)[0]<=126)
      {
        //printf_s("%c",(RelocInfoTemp.bData)[j]);
        str1.Format(L"%c",(RelocInfoTemp.bData)[j]);
        str2+=str1;
      }
      else
      {
        //printf_s("%02X ",(RelocInfoTemp.bData)[j]);
        str1.Format(L"%02X",(RelocInfoTemp.bData)[j]);
        str2+=L" "+str1;
      }    
    }
    m_SecondReloc.SetItemText(i,5,str2);
  //  printf_s("\n");
    pOffset++;
  }
}
```

计算**位置计算器**相关信息（与[上面的B]计算偏移一致 加了点料进去而已）

```
MyFileLocation.h  MyFileLocation.cpp

DWORD MyFileLocation::ExclusiveCalcOffset(DWORD Rva)
{
  //循环比较在那个区段  不在这个区段就继续循环  
  //PIMAGE_NT_HEADERS32 pnt=pNtH;
  PIMAGE_SECTION_HEADER pSecHTemp=IMAGE_FIRST_SECTION(AnalPE.pNtH);//区段头
  int index=0;
  while (!(Rva>=pSecHTemp->VirtualAddress&&
    Rva<pSecHTemp->VirtualAddress+pSecHTemp->SizeOfRawData))
  {
    //找完所有区段还没有找到
    if (index>AnalPE.pNtH->FileHeader.NumberOfSections)
    {
        m_Section=L"部首";
      ////  DWORD a[5];
      //  _memccpy(&Address,pSecHTemp,6,24);
      //  //Address=(int);      
      return Rva;
    }
    ++index;
    ++pSecHTemp;
  }
  PBYTE iiii=(PBYTE)(Rva-pSecHTemp->VirtualAddress+pSecHTemp->PointerToRawData+(long)AnalPE.buf);
  //CString aa;
  //十六进制编辑器也可以在这做文章
  for (int i=0;i<10;i++)
  {
    CString jj;
    jj.Format(L"%02x ",*(iiii++));
    m_Byte+=jj;
  }
  m_Section=pSecHTemp->Name;
  return Rva-pSecHTemp->VirtualAddress+pSecHTemp->PointerToRawData;;
}
```

就是这些了。慢慢看。

------------------------------

源码： [简易PE.rar](https://bbs.kanxue.com/attach-download-101554.htm)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.png](javascript:void(0);)

  （36.87kb，21次下载）

- [2.png](javascript:void(0);)

  （46.88kb，8次下载）

- [3.png](javascript:void(0);)

  （27.50kb，11次下载）

- [5.png](javascript:void(0);)

  （28.96kb，8次下载）

- [6.png](javascript:void(0);)

  （45.19kb，12次下载）

- [7.png](javascript:void(0);)

  （26.01kb，11次下载）

- [8.png](javascript:void(0);)

  （58.50kb，12次下载）

- [4.png](javascript:void(0);)

  （58.43kb，10次下载）

- [简易PE.rar](javascript:void(0);)

  （86.10kb，575次下载）
