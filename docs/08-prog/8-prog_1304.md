
# 遍历读取文件夹与文件算法

> 原文：[`www.pediy.com/kssd/pediy11/109646.html`](https://www.pediy.com/kssd/pediy11/109646.html)

一直对控制台应用程序情有独钟，之前写了这个 遍历读取文件夹和文件 算法。希望能得到一个邀请码。附上附件。谢谢。

```c
#include "stdafx.h"

#include <windows.h>

#include <iostream>

using namespace std;

//============声明遍历文件与文件夹的函数===============//

void TraverseFile(char *FileDirectory);

//====================main 函数======================//

int main(int argc, char* argv[])
{
  system("title 文件遍历器   By: baoss");

  cout<<"Please input a file directory: ";

  char FileDirectory[MAX_PATH];

  cin>>FileDirectory;

  //====开始调用====//

  TraverseFile(FileDirectory);

  system("pause");

  return 0;
}

//============声明遍历文件与文件夹的函数===============//

void TraverseFile(char *FileDirectory)
{
  char szFileName[MAX_PATH];

  strcpy(szFileName,FileDirectory);

  strcat(szFileName,"\\*.*");

  WIN32_FIND_DATA FindData;

  HANDLE hFile;

  hFile = FindFirstFile(szFileName,&FindData);

  if(hFile != INVALID_HANDLE_VALUE)
  {
    if(FindData.dwFileAttributes == FILE_ATTRIBUTE_DIRECTORY)
    {
      if((strcmp(FindData.cFileName,".") != 0) &&  (strcmp(FindData.cFileName,"..") != 0))
      {
        char Dir[MAX_PATH];

        strcpy(Dir,(char *)FileDirectory);

        strcat(Dir,"\\");

        strcat(Dir,(char *)FindData.cFileName);

        cout<<"目录: "<<Dir<<endl;
      }
    }
    else
    {
      char FileDir[MAX_PATH];

      strcpy(FileDir,(char *)FileDirectory);

      strcat(FileDir,"\\");

      strcat(FileDir,(char *)FindData.cFileName);

      cout<<"文件: "<<FileDir<<endl;
    }
    while (FindNextFile(hFile,&FindData))
    {
      if(FindData.dwFileAttributes == FILE_ATTRIBUTE_DIRECTORY)
      {
        if((strcmp(FindData.cFileName,".") != 0) &&  (strcmp(FindData.cFileName,"..") != 0))
        {
          char Dir[MAX_PATH];

          strcpy(Dir,(char *)FileDirectory);

          strcat(Dir,"\\");

          strcat(Dir,(char *)FindData.cFileName);

          cout<<"目录: "<<Dir<<endl;
              //==递归调用==//
          TraverseFile(Dir);
        }
      }
      else
      {
        char FileDir[MAX_PATH];

        strcpy(FileDir,(char *)FileDirectory);

        strcat(FileDir,"\\");

        strcat(FileDir,(char *)FindData.cFileName);

        cout<<"文件: "<<FileDir<<endl;
      }
    }
    FindClose( hFile );
  }
  else
  {
    cout<<"Invalid file directory !"<<endl;
  }
}
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `TraverseFile.rar` |
| :--- | :--- |
