

# 小菜自编 PE 分析工具

> 原文：[`www.pediy.com/kssd/pediy11/117787.html`](https://www.pediy.com/kssd/pediy11/117787.html)

最近刚刚接触了 PE 文件格式，仔细学了一下，写了一个简单的分析工具，奉上源码，希望大家多多交流，不吝赐教！  
  
部分预览：  
  
  

代码:

```
//输出重定位信息
  PIMAGE\_BASE\_RELOCATION pBaseReloc;
  if (pOptHeader->DataDirectory\[5\].VirtualAddress)
  {
    printf("===========IMAGE\_BASE\_RELOCATION==============");
    pBaseReloc =   (PIMAGE\_BASE\_RELOCATION)((LPBYTE)lpBaseAddr+RvaToOffset(pOptHeader->DataDirectory\[5\].VirtualAddress));
  }
  else{
      printf("NO RELOC TABLE!\\n");
      pBaseReloc = 0;
  }
  while (pBaseReloc && pBaseReloc->VirtualAddress)  //判断是否到了重定位表的末尾
  {
    //打印当前分配表的基址和大小
    printf("Base:0x%08x,SizeOfBlock:%0x\\n",pBaseReloc->VirtualAddress,pBaseReloc->SizeOfBlock);
    for (unsigned int i=8;i<pBaseReloc->SizeOfBlock;i+=2){
      WORD wRelocData = \*(LPWORD)((LPBYTE)pBaseReloc+i);
      if (!wRelocData)
        continue;
      DWORD RvaData = (wRelocData&0xFFF)+pBaseReloc->VirtualAddress;
      printf("0x%04x  0x%04x  0x%08x==>0x%08x\\n",wRelocData,wRelocData&0x0FFF,RvaData,\*(LPDWORD)(RvaToOffset(RvaData)+(DWORD)lpBaseAddr));
    }
    //pBaseReloc 指向下一个分配表  
    pBaseReloc =(PIMAGE\_BASE\_RELOCATION) ((LPBYTE)pBaseReloc+pBaseReloc->SizeOfBlock);
  }

```

\================  
  
今天仔细看了一下，发现了一个问题，有些程序（如何 win7 自带的记事本、explorer.exe 等等）的 IMAGE\_IMPORT\_DESCRIPTOR 中的 OrignalFirstThunk 和 FirstThunk 都不为零，但是所最终指向的位置不是同一个 Image\_Import\_By\_name 数据结构，不知道怎么发生的？请教各位！

上传的附件
| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || dumppe.zip  
 |