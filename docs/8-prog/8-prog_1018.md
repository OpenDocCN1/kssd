
# 【学习笔记】发一个自己写的 dif2exe

> 原文：[`www.pediy.com/kssd/pediy11/122327.html`](https://www.pediy.com/kssd/pediy11/122327.html)

最近在看 `IDA Pro` 权威指南，终于知道怎么用 `IDA` 修改 `exe` 文件了![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

虽说书上提到有 `pe_script` 系列脚本，也有 `dif2exe` 的参考代码，本菜鸟本着多实践的原则，自己写了一个 `dif2exe` 的程序，算做是我的学习笔记![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

代码:

```c
#include<stdio.h>
#include<stdlib.h>
#include<string.h>

size_t MyGetFileSize(char const *szFileName)
{
    FILE *file = NULL;
    fpos_t pos;
    file = fopen(szFileName,"rb");
    if(file != NULL)   
        if(fseek(file,0,SEEK_END) == 0)
            if(fgetpos(file,&pos) == 0)
      {
        fclose(file);
        return (size_t)pos;
      }
    fclose(file);
    return -1;
}

int main(int argc,char **argv)
{
  FILE *fExe = NULL;
  FILE *fDif = NULL;
  size_t FileSize = 0;
  size_t ExeFileSize = 0;
  unsigned char *Base = NULL;
  unsigned char *Locate = NULL;
  unsigned char *Temp = NULL;
  unsigned long Addr;
  unsigned char OldByte;
  unsigned char NewByte;
  char *ExeFile = NULL;
  char *DifFile = NULL;
  char szBuffer1[260];
  char szBuffer2[260];
  if(argc != 3)
  {
    printf("usage:\ndif2exe exefile diffile");
    return -1;
  }
    ExeFile = argv[1];
  DifFile = argv[2];
  if( (fExe = fopen(ExeFile,"rb")) == NULL )
  {
    printf("fopen failed!");
    return -1;
  }
  ExeFileSize = MyGetFileSize(ExeFile);
  if(ExeFileSize > 0)
  {
    Base = calloc(1,ExeFileSize);
    if(Base == NULL)
    {
      fclose(fExe);
      printf("calloc failed!");
      return -1;
    }
    if(fread(Base,1,ExeFileSize,fExe) != ExeFileSize)
    {
      fclose(fExe);
      free(Base);
      printf("fread failed!");
      return -1;
    }
    fclose(fExe);
    if( (fDif = fopen(DifFile,"rb")) == NULL )
    {
      free(Base);
      printf("fopen failed!");
      return -1;
    }

    FileSize = MyGetFileSize(DifFile);
    if(FileSize > 0)
    {
      Locate = calloc(1,++FileSize);
      if(Locate == NULL)
      {
        free(Base);
        fclose(fDif);
        printf("calloc failed!");
        return -1;
      }
      Temp = Locate;
      --FileSize;
      if(fread(Locate,1,FileSize,fDif) != FileSize)
      {
        free(Base);
        free(Temp);
        fclose(fDif);
        printf("fread failed!");
        return -1;
      }
      fclose(fDif);
      Locate = strstr(Locate,"\r\n") + 2;
      Locate = strstr(Locate,"\r\n") + 2;
      Locate = strstr(Locate,"\r\n") + 2;
      while( *(strstr(Locate,"\r\n") + 2) != 0)
      {
          sscanf(Locate,"%x: %x %x",&Addr,&OldByte,&NewByte);
          if( *(Base + Addr) != OldByte )
        {
          printf("It seems that the exe file does not correspond with the dif:%X<--->%X",*(Base + Addr),OldByte);
          free(Base);
          free(Temp);
          return -1;
        }
                *(Base + Addr) = NewByte;
        Locate = strstr(Locate,"\r\n") + 2;
      }
      sscanf(Locate,"%x: %x %x",&Addr,&OldByte,&NewByte);
      if( *(Base + Addr) != OldByte )
      {
        printf("It seems that the exe file does not correspond with the dif:%X<--->%X",*(Base + Addr),OldByte);
        free(Base);
        free(Temp);
        return -1;
      }
      *(Base + Addr) = NewByte;
      strcpy(szBuffer2,ExeFile);
      *(strrchr(szBuffer2,'\\') + 1) = 0;
      strcpy(szBuffer1,szBuffer2);
      strcpy(szBuffer2,"camellu");
      strcat(szBuffer2,strrchr(ExeFile,'\\') + 1);
      strcat(szBuffer1,szBuffer2);
            fExe = fopen(szBuffer1,"wb");
      if(fExe != NULL)
        if( fwrite(Base,1,ExeFileSize,fExe) == ExeFileSize )
          printf("I did it^\_^");
      free(Base);
      free(Temp);
      fclose(fExe);
    }
  }
  return 0;
}
```

代码一发上来就乱了，谁能教教我在论坛怎么弄才能让代码保持原形![](img/cb6288ad81b189b02921e98a92da6abe.jpg "124.gif")![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `cuidif2exe.rar`  

 |
