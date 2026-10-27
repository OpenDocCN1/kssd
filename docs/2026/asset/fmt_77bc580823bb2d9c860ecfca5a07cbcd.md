
```

//获取资源表

   pDatD=&(pOptH->DataDirectory[2]);

   PIMAGE_RESOURCE_DIRECTORY pResD=(PIMAGE_RESOURCE_DIRECTORY)(buf+CalcOffset(pDatD->VirtualAddress,pNtH));

   DWORD ReSize=pResD->NumberOfIdEntries+pResD->NumberOfNamedEntries;

   PIMAGE_RESOURCE_DIRECTORY_ENTRY pResDE=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)((long)pResD+sizeof(IMAGE_RESOURCE_DIRECTORY));

   

   char ResourceName[15][16]={"鼠标指针","位图","图标","菜单","对话框",

                "字符串列表","字体目录","字体","快捷键","非格式化资源",

                "消息列表","鼠标指针组","图标组","版本信息"};

   printf_s("第一层[名称条目]->：%04X   ",pResDE->NameIsString);  

   printf_s("第一层[ID条目]->：%04X   \n",ReSize);    

   for (DWORD FirstOrder=0;FirstOrder<ReSize;FirstOrder++)

   {

     //第一层 假如是字符串标识

     if (pResDE->NameIsString==1)

     {

       PIMAGE_RESOURCE_DIR_STRING_U pREsDStrU=(PIMAGE_RESOURCE_DIR_STRING_U)((long)pResD+pResDE->NameOffset);

       printf_s("资源类型名：%s",pREsDStrU->NameString);

     }

     //假如是一直类型 用序号作为标识

     else

     {

       switch (pResDE->Name)

       {

       case 0x1:

         printf_s("第一层：%s\n",ResourceName[0]);

         break;        

       case 0x2:        

         printf_s("第一层：%s\n",ResourceName[1]);

         break;        

       case 0x3:        

         printf_s("第一层：%s\n",ResourceName[2]);

         break;        

       case 0x4:        

         printf_s("第一层：%s\n",ResourceName[3]);

         break;        

       case 0x5:        

         printf_s("第一层：%s\n",ResourceName[4]);

         break;        

       case 0x6:        

         printf_s("第一层：%s\n",ResourceName[5]);

         break;        

       case 0x7:        

         printf_s("第一层：%s\n",ResourceName[6]);

         break;        

       case 0x8:        

         printf_s("第一层：%s\n",ResourceName[7]);

         break;        

       case 0x9:        

         printf_s("第一层：%s\n",ResourceName[8]);

         break;        

       case 0xA:        

         printf_s("第一层：%s\n",ResourceName[9]);

         break;        

       case 0xB:        

         printf_s("第一层：%s\n",ResourceName[10]);

         break;        

       case 0xC:        

         printf_s("第一层：%s\n",ResourceName[11]);

         break;        

       case 0xE:        

         printf_s("第一层：%s\n",ResourceName[12]);

         break;        

       case 0x10:        

         printf_s("第一层：%s\n",ResourceName[13]);

         break;        

       default:

         printf_s("第一层：%d\n",pResDE->Name);

         break;

       }

     }

     //找第二层 注意OFFEST 是相对于pResD也就是资源开始位置的偏移

     PIMAGE_RESOURCE_DIRECTORY pResD2=(PIMAGE_RESOURCE_DIRECTORY)((long)pResD+pResDE->OffsetToDirectory);

     DWORD ReSize2=pResD2->NumberOfIdEntries+pResD2->NumberOfNamedEntries;

     PIMAGE_RESOURCE_DIRECTORY_ENTRY pResDE2=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)((long)pResD2+sizeof(IMAGE_RESOURCE_DIRECTORY));

     printf_s("第一层[名称条目]->：%04X   ",pResD2->NumberOfNamedEntries);  

     printf_s("第一层[ID条目]->：%04X   \n",pResD2->NumberOfIdEntries);

     for (DWORD SecondOrder=0;SecondOrder<ReSize2;SecondOrder++)

     {

       if (pResDE2->DataIsDirectory==1)

       {

         //输出第二层资源的标识看是数字还是字符串

         if (pResDE2->NameIsString==1)

         {

           PIMAGE_RESOURCE_DIR_STRING_U pREsDStrU2=(PIMAGE_RESOURCE_DIR_STRING_U)((long)pResD+pResDE2->NameOffset);

           //输出资源类型名字

           printf("第二层->资源类型名：%ls   ",pREsDStrU2->NameString);

         }

         else

         {

           printf_s("第二层->资源类型名ID：%d   ",pResDE2->Id);        

         }

         //解析第三层 

         PIMAGE_RESOURCE_DIRECTORY pResD3=(PIMAGE_RESOURCE_DIRECTORY)((long)pResD+pResDE2->OffsetToDirectory);

         PIMAGE_RESOURCE_DIRECTORY_ENTRY pResDE3=(PIMAGE_RESOURCE_DIRECTORY_ENTRY)((long)pResD3+sizeof(IMAGE_RESOURCE_DIRECTORY));

         PIMAGE_RESOURCE_DATA_ENTRY pResDataE=(PIMAGE_RESOURCE_DATA_ENTRY)((long)pResD+pResDE3->OffsetToData);

         

         printf_s("第三层->RVA：%08X   ",pResDataE->OffsetToData);

         printf_s("第三层->偏移：%08X   ",(CalcOffset(pResDataE->OffsetToData,pNtH)));

         printf_s("第三层->大小：%08X\n",pResDataE->Size);



       }

       else

       {

         break;

       }

       pResDE2++;

     }

     pResDE++;

   }

```

重定位：

```

 pDatD=&(pOptH->DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC]);//也就是5

   PIMAGE_BASE_RELOCATION pBasR=(PIMAGE_BASE_RELOCATION)((long)buf+CalcOffset(pDatD->VirtualAddress,pNtH));

   RELOCAREAINFO Temp={0};

   int j=0;

   while (pBasR->VirtualAddress)

   {

     printf_s("索引：%d  ",j);

     //区域的虚拟地址输出或获取到其他地方

     Temp.dwAreaRVA=pBasR->VirtualAddress;

     

     RelCalcOffset(Temp.dwAreaRVA,pNtH,buf,Temp.szSectionName,NULL);

     printf_s("%s   ",Temp.szSectionName);

     printf_s("%08X   ",pBasR->VirtualAddress);

     Temp.NumberOfReloc=(pBasR->SizeOfBlock-sizeof(IMAGE_BASE_RELOCATION))/2;

     printf_s("%02X  %d\n", Temp.NumberOfReloc, Temp.NumberOfReloc);

     DWORD dwCount=Temp.NumberOfReloc;

     //得到第三个成员的TYPEOFFEST起始位置 之后开始循环获取重定位信息

     PTYPEOFFSET pOffset=(PTYPEOFFSET)((long)pBasR+sizeof(IMAGE_BASE_RELOCATION));

     for (DWORD i=0;i<dwCount;i++)

     {

       printf_s("索引：%d  ",i);

       RELOCINFO RelocInfoTemp={0};

       //需要重定位的虚拟地址（RVA）

       RelocInfoTemp.dwRelocRVA=Temp.dwAreaRVA+pOffset->Offset;

       printf_s("RVA：%08X  ", RelocInfoTemp.dwRelocRVA);

       //根据相对虚拟地址算出的文件偏移

       RelocInfoTemp.dwOffset=RelCalcOffset(RelocInfoTemp.dwRelocRVA,pNtH,NULL,NULL);

        printf_s("偏移：%08X  ", RelocInfoTemp.dwOffset);

       //重定位方式

       RelocInfoTemp.bType=pOffset->Type;

       printf_s("类型：HIGHLOW (%d)  ",RelocInfoTemp.bType);

       //从算出的文件偏移取出的数据 这个数据就是需要重定位的虚拟地址(VA)

       RelocInfoTemp.dwRelocValue=*(PDWORD)((long)buf+RelocInfoTemp.dwOffset);

       printf_s("FAR地址：%08X  ",RelocInfoTemp.dwRelocValue);

       //RelocInfoTemp.bData

       DWORD VA= RelCalcOffset(RelocInfoTemp.dwRelocValue,pNtH,buf,NULL,(PCHAR)RelocInfoTemp.bData,2);

       RelCalcOffset(VA,pNtH,buf,NULL,(PCHAR)RelocInfoTemp.bData,1);

       for (int i=0;i<10;i++)

       {

         //判断是否在有效字符32-126区间内

         if (32<=(RelocInfoTemp.bData)[0]&&(RelocInfoTemp.bData)[0]<=126)

         {

           printf_s("%c",(RelocInfoTemp.bData)[i]);

         }

         else

         {

          printf_s("%02X ",(RelocInfoTemp.bData)[i]);

         }

       }

        printf_s("\n");

       pOffset++;

     }

    

     //讲一个区域重定位信息添加

    VecRelocAreaInfo.push_back(Temp);

    pBasR=(PIMAGE_BASE_RELOCATION)((long)pBasR+pBasR->SizeOfBlock);

    j++;

   }

```

这个虽然不实用 但让你进一步理解PE文件结构及运作原理还是能够的

周末有空我再发MFC版吧![](https://bbs.kanxue.com/view/img/face/1.gif)

---

源码：

[`PE.rar`](https://bbs.kanxue.com/attach-download-101474.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [`1.png`](javascript:void(0);)  
  （14.44kb，19次下载）
- [`2.png`](javascript:void(0);)  
  （10.16kb，7次下载）
- [`3.png`](javascript:void(0);)  
  （9.00kb，6次下载）
- [`4.png`](javascript:void(0);)  
  （31.73kb，12次下载）
- [`PE.rar`](javascript:void(0);)  
  （9.16kb，391次下载）
