
# Windows XP 注册表文件格式简单分析

> 原文：[`www.pediy.com/kssd/pediy10/74337.html`](https://www.pediy.com/kssd/pediy10/74337.html)

节省时间，我不把文章贴出来了，在二楼把代码贴出来，文件都在压缩包中：

![](img/c26b3ee42deaa7f83d7ef7237b5d167c.jpg)

运行结果：

![](img/8705d07195d856fbee617df058808198.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Windows XP 注册表文件格式简单分析.rar` |

代码:

```
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>

#define REGF                   0x66676572     //fger
#define HBIN                   0x6e696268     //nibh
#define CM_KEY_FAST_LEAF       0x666c         // fl
#define CM_KEY_HASH_LEAF       0x686c         // hl


//数据结构定义
typedef struct _CHILD_LIST
{
  ULONG Count;
  ULONG List;
}CHILD_LIST;

typedef struct _CM_KEY_NODE   
{ 
  USHORT  Signature;           
  CHAR  Reserve_1[18];
  ULONG SubKeyCounts[2];
  ULONG SubKeyLists[2];
  CHILD_LIST ValueList;
  CHAR  Reserve_2[28];
  USHORT NameLength;
  SHORT ClassName;
  CHAR  Name;
} CM_KEY_NODE,*PCM_KEY_NODE;

typedef struct _CM_KEY_INDEX
{
  USHORT Signature;
  USHORT Count;
  ULONG List[1];
} CM_KEY_INDEX, *PCM_KEY_INDEX;

typedef struct _CM_KEY_VALUE 
{ 
  USHORT  Signature;
  SHORT NameLength;
  ULONG DataLength; 
  ULONG Data;
  ULONG Type;
  CHAR  Reserve_1[4];
  CHAR  Name;
}CM_KEY_VALUE,*PCM_KEY_VALUE;


VOID TpyeKeyAndValue(PVOID FileMemAddr);
VOID TypeSubKey(PCHAR Bin,PCM_KEY_INDEX KeyIndex);
VOID TypeSubKeyName(PCHAR Bin,PCM_KEY_NODE KeyNode);
VOID TypeValue(PCHAR Bin,PCM_KEY_VALUE value);

int main(int argc,char *argv[])
{
  HANDLE hFile;
  HANDLE hFileMap;
  DWORD  FileSize;
  PVOID  FileMem;

#ifndef _DEBUG
  if(argc!=2)
  {
    printf("*
...
*n\n");
    printf("Useage:\nHivePase FileName\n");
    printf("*
...
*n\n");
    system("pause");
    return 1;
  }
  hFile=CreateFile(argv[1],GENERIC_READ,0,NULL,
          OPEN_EXISTING,FILE_FLAG_OVERLAPPED,NULL);
#else
  hFile=CreateFile("c:\\test_root.dat",GENERIC_READ,0,NULL,
          OPEN_EXISTING,FILE_FLAG_OVERLAPPED,NULL);
#endif

  if(hFile==INVALID_HANDLE_VALUE)
  {
    printf("Error:File doesn's Exist!\n");
    system("pause");
    return 1;
  }
  FileSize=GetFileSize(hFile,NULL);
  hFileMap=CreateFileMapping(hFile,NULL,PAGE_READONLY | SEC_COMMIT,0,FileSize,NULL);
  if(hFileMap==NULL)
  {
    printf("CreateFileMapping Error!\n");
    CloseHandle(hFile);
    system("pause");
    return 1;
  }
  CloseHandle(hFile);
  FileMem=MapViewOfFile(hFileMap,FILE_MAP_READ,0,0,0);
  if(FileMem==NULL)
  {
    printf("MapViewOfFile Error!\n");
    CloseHandle(hFileMap);
    system("pause");
    return 1;
  }
  CloseHandle(hFileMap);
  TpyeKeyAndValue(FileMem);
  printf("\nSuccess!\n");
  system("pause");
  return 0;
}

VOID TpyeKeyAndValue(PVOID FileMemAddr)
{
  char RootKeyName[256];
  PCHAR RootBin;
  PCM_KEY_NODE KeyNode;
  PCM_KEY_INDEX KeyIndex;
  if(*(ULONG*)FileMemAddr!=REGF)
  {
    printf("Not a Hive File!\n");
    system("pause");
  }
  RootBin=(char*)FileMemAddr+0x1000;
  KeyNode=(PCM_KEY_NODE)(RootBin+0x24);
  if(*(ULONG*)RootBin!=HBIN)
  {
    printf("Hive File Error!\n");
    system("pause");
  }
  ZeroMemory(RootKeyName,256);
  strncpy(RootKeyName,&KeyNode->Name,KeyNode->NameLength);
  printf("Root Key:  %s\n",RootKeyName);
  DWORD SubKeyLists=KeyNode->SubKeyLists[0];
  KeyIndex=(PCM_KEY_INDEX)(RootBin+SubKeyLists+0x4);
  TypeSubKey(RootBin,KeyIndex);

}

VOID TypeSubKey(PCHAR Bin,PCM_KEY_INDEX KeyIndex)
{
  USHORT KeyCount;
  PCM_KEY_NODE KeyNode;
  KeyCount=KeyIndex->Count;
  for(USHORT i=0;i<KeyCount;i++)
  {
    if(KeyIndex->Signature==CM_KEY_FAST_LEAF || KeyIndex->Signature==CM_KEY_HASH_LEAF)
    {
      DWORD KeyNodeOffset=KeyIndex->List[i*2];
      KeyNode=(PCM_KEY_NODE)(Bin+KeyNodeOffset+0x4);
      TypeSubKeyName(Bin,KeyNode);
    }
    else
    {
      DWORD KeyNodeOffset=KeyIndex->List[i*2];
      KeyNode=(PCM_KEY_NODE)(Bin+KeyNodeOffset+0x4);
      TypeSubKeyName(Bin,KeyNode);
    }
  }

}

VOID TypeSubKeyName(PCHAR Bin,PCM_KEY_NODE KeyNode)
{
  char SubKeyName[256];
  ULONG ValueLists;
  ULONG ValueCount;
  ULONG *ValueIndex;
  PCM_KEY_VALUE value;
  PCM_KEY_INDEX KeyIndex;
  ZeroMemory(SubKeyName,256);
  strncpy(SubKeyName,&KeyNode->Name,KeyNode->NameLength);
  printf("Sub Key:  %s\n",SubKeyName);
  ValueLists=KeyNode->ValueList.List;
  ValueCount=KeyNode->ValueList.Count;
  if(ValueLists!=-1)
  {
    ValueIndex=(ULONG *)(Bin+ValueLists+0x4);
    for(ULONG i=0;i<ValueCount;i++)
    {
      value=(PCM_KEY_VALUE)(Bin+ValueIndex[i]+0x4);
      TypeValue(Bin,value);
    }
  }
  if(KeyNode->SubKeyLists[0]!=-1)
  {
    KeyIndex=(PCM_KEY_INDEX)(Bin+KeyNode->SubKeyLists[0]+0x4);
    TypeSubKey(Bin,KeyIndex);
  }
}


VOID TypeValue(PCHAR Bin,PCM_KEY_VALUE value)
{
  char ValueName[256];
  ULONG DataLenth;
  PCHAR Data;
  ZeroMemory(ValueName,256);
  strncpy(ValueName,&value->Name,value->NameLength);
  printf("Value Name:  %s\n",ValueName);
  switch(value->Type)
  {
    case REG_SZ:
      printf("REG_SZ    ");
      break;
    case REG_BINARY:
      printf("REG_BINARY  ");
      break;
    case REG_DWORD:
      printf("REG_DWORD  ");
      break;
    case REG_MULTI_SZ:
      printf("REG_MULIT_SZ  ");
      break;
    case REG_EXPAND_SZ:
      printf("REG_EXPAND_SZ  ");
      break;
    default:
      break;
  }
  if(value->Type==REG_DWORD)
  {
    Data=(PCHAR)&value->Data;
    printf("%08x",*(ULONG*)Data);
  }
  else
  {
    if(value->DataLength & 0x80000000)
    {
      DataLenth=value->DataLength & 0x7FFFFFFF;
      Data=(PCHAR)&value->Data;
      for(ULONG i=0;i<DataLenth;i++)
      {
        if(value->Type==REG_BINARY)
        {
          printf("%1x",Data[i]);
        }
        else
        {
          printf("%c",Data[i]);
        }
      }
    }
    else
    {
      DataLenth=value->DataLength;
      DWORD DataOffset=value->Data;
      Data=Bin+DataOffset+0x4;
      for(ULONG i=0;i<DataLenth;i++)
      {
        if(value->Type==REG_BINARY)
        {
          printf("%1x",Data[i]);
        }
        else
        {
          printf("%c",Data[i]);
        }
      }
    }
  }
  printf("\n");
}
```
