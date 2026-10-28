
# [学习笔记]发一个自己写的`dif2exe`

最近在看IDA Pro权威指南，终于知道怎么用IDA修改exe文件了![](https://bbs.kanxue.com/view/img/face/27.gif)

虽说书上提到有`pe_script`系列脚本，也有`dif2exe`的参考代码，本菜鸟本着多实践的原则，自己写了一个`dif2exe`的程序，算做是我的学习笔记![](https://bbs.kanxue.com/view/img/face/12.gif)

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

      Locate = strstr(Locate,"\x0d\x0a") + 2;

      Locate = strstr(Locate,"\x0d\x0a") + 2;

      Locate = strstr(Locate,"\x0d\x0a") + 2;

      while( *(strstr(Locate,"\x0d\x0a") + 2) != 0)

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

        Locate = strstr(Locate,"\x0d\x0a") + 2;

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

          printf("I did it^_^");

      free(Base);

      free(Temp);

      fclose(fExe);

    }

  }

  return 0;

}
```

代码一发上来就乱了，谁能教教我在论坛怎么弄才能让代码保持原形![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/50.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`cuidif2exe.rar`](javascript:void(0);)（19.60kb，21次下载）
