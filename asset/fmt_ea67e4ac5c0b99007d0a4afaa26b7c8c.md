
```
//------------------------------
//显示 Section 结构
//------------------------------
nSection=stPEHeader.FileHeader.NumberOfSections;
stSectionHeader=new _IMAGE_SECTION_HEADER[nSection];
m_ListCtrl.DeleteAllItems();
for(int i=0;i<nSection;i++)
{
  PEfile.Read(&stSectionHeader[i],sizeof(_IMAGE_SECTION_HEADER));//-----节表
  //NO
  szTemp.Format(_T("%.2d"), i+1);
  m_ListCtrl.InsertItem(i,szTemp,i);
  //SectionName
  strcpy(chSectionName,(LPCSTR)stSectionHeader[i].Name);
  m_ListCtrl.SetItemText(i,1,chSectionName);
  //VirtualSize
  szTemp.Format(_T("0x%.8X"),stSectionHeader[i].Misc.VirtualSize);
  m_ListCtrl.SetItemText(i,2,szTemp);
  //VirtualAddress
  szTemp.Format(_T("0x%.8X"),stSectionHeader[i].VirtualAddress );
  m_ListCtrl.SetItemText(i,3,szTemp);
  //SizeOfRawData
  szTemp.Format(_T("0x%.8X"),stSectionHeader[i].SizeOfRawData );
  m_ListCtrl.SetItemText(i,4,szTemp);
  //SizeOffset
  szTemp.Format(_T("0x%.8X"),stSectionHeader[i].PointerToRawData );
  m_ListCtrl.SetItemText(i,5,szTemp);
  //Characteristics
  szTemp.Format(_T("0x%.8X"),stSectionHeader[i].Characteristics );
  m_ListCtrl.SetItemText(i,6,szTemp);
}
delete stSectionHeader;
```

好了，我们计划的两个功能已经写完。

不早了，我要睡觉了（2003-12-18 2:02）其他的功能下一篇再说吧，

哈哈，对了差点忘了，所有完整代码我都放在压缩文件里了，自己看看代码应该对 `pe` 有个大概的了解，另外送大家一个我在网上收集的 `pe` 结构图。

大家在写的时候别忘了 `#include <winnt.h>`。

睡了睡了，，，，困死了!@#$%^&*()

886

WiNrOOt
2003-12-18 2:07

`4dd466d0e1e0cc123afbe504da2462ca`
