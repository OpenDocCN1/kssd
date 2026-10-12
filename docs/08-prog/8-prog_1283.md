
# 文件沙盘代码 YasVps

> 原文：[`www.pediy.com/kssd/pediy10/102841.html`](https://www.pediy.com/kssd/pediy10/102841.html)

之前计划写的沙盘系统，刚写好文件部分，也不算太完善，碰巧有人问起，遂贴上代码大家看去，你可以在此基础上完善之。代码没什么 WS 技术，但做文件沙盘还是有不少问题的，需要的朋友可以下，贴上主要头文件。基本代码是围绕这个头文件展开的。

```c
#include <ntddk.h>

#pragma once

//  ucReseaon  :

//

// Basic

//

#define YASVPS_CREATE_FILE      0x00

#define YASVPS_MOVE_FILE       0x01

#define YASVPS_REPLACE_FILE    0x02

#define YASVPS_RENAME_FILE     0x03

#define YASVPS_WRITE_FILE      0x04

//

//Option

//

#define YASVPS_SETFILE_BASICINFO 0x05

#define YASVPS_SETFILE_ENDINFO   0x06

#define YASVPS_SETFILE_POSINFO   0x07

#define YASVPS_SETFILE_SECINFO   0x08

//

//删除以存在文件时追踪, 

//对已存在文件的删除是不可恢复的，所以这个链表跟进程的生命周期一样，而且只能添加节点不能删除。

//

typedef struct __YASVPS_DELETETRACE

{
  ULONG uPid;
  CHAR lpszDelFilePath[YASVPS_MAX_PATH];
  struct __YASVPS_DELETETRACE* lpNext;
}YASVPS_DELETETRACE;

//

//创建新文件时追踪,添加节点前，先判断文件是否 “删除文件追踪” 存在，若存在，则删除对应节点

//删除文件时，需要查找这个表的信息，如果发现删除的是自己创建的文件，则允许删除。并删除相关跟踪节点

//同样的任何修改文件也需要先找这张表

//外界不应该操作在 YASVPS 中运行进程创建的文件。

//

/*
dir 规则：
如果在枚举目录文件时， “删除以存在文件” 中有文件在对应的目录中。
如果在 YASVPS_CREATEFILETRACE 没有相应节点，则过滤掉删除的文件，不显示
否则显示之
*/

/*
覆盖文件：
覆盖新创建文件： 直接允许
覆盖已存在文件： 相当于 删除原来存在的文件，再创建一个新的文件。
*/

/*
移动已存在文件： eg: c:\123.exe ---->> c:\windows\123.exe  ---->> c:\123.exe
相当于删除文件再创建文件。
*/

//

//修改以存在文件时追踪。删除下面匹配的文件时，相应节点也要删除。

//

typedef struct __YASVPS_CREATETRACE

{
  //
  //任何修改文件也需要先找这张表,再根据 ucReseaon 做相应的操作
  //
  struct __YASVPS_CREATETRACE* lpNext;
  ULONG uPid; //进程 PID
  HANDLE hFile;
  PFILE_OBJECT lpFileObj; //保留
  /*
  ucReseaon  :
  #define YASVPS_CREATE_FILE      0x00
  #define YASVPS_MOVE_FILE       0x01
    #define YASVPS_REPLACE_FILE    0x02
    #define YASVPS_RENAME_FILE     0x03
    #define YASVPS_WRITE_FILE      0x04
  */
  UCHAR ucReseaon;
  /*
  ucReseaon = YASVPS_CREATE_FILE 
  eg:    lpszCreatePath  : C:\123.exe
  重定向 lpszTracePath  : C:\YasVPS\C:\123.exe
  之后  修改名字为 hello.exe 则 lpszCreatePath  : C:\hello.exe
  对 YASVPS_CREATE_FILE 文件的操作，都直接 PASS， 删除文件时，需要删除 lpszLinkPath,
  并删除对应 YASVPS_CREATETRACE 节点
    “外界不应该操作在 YASVPS 中运行进程创建的文件！！！！！”
  */
  //
  //重命名/移动文件/覆盖文件/创建后后新的路径
  //
  CHAR lpszNewPath[YASVPS_MAX_PATH]; 
  //
  //指向的原始路径, 当该文件被写时，这个地方为 TMP 文件,写的文件不能超过 10M(*)
  //
  CHAR lpszLinkPath[YASVPS_MAX_PATH]; 
  //
  //下面的暂时保留，不支持，主要是从效率和节省内存考虑
  //
  /*  FILE_BASIC_INFORMATION stFileBasicInfo;
  FILE_END_OF_FILE_INFORMATION szFileEndInfo;
  */
}YASVPS_CREATETRACE, *PYASVPS_CREATETRACE;

PYASVPS_CREATETRACE VpsAddCreateTraceNode( );

PYASVPS_CREATETRACE VpsIsFileRedirect( CHAR* lpszNewPath );

PYASVPS_CREATETRACE VpsIsFileRedirect_2(IN CHAR* lpszLinkPath );

PYASVPS_CREATETRACE __VpsGetRedirectInfo( CHAR* lpszDirPath , IN OUT CHAR* lpszFileName);

BOOLEAN  VpsExaPath(PSTR szFilePath,PSTR szFileName,PSTR szFileDir,PSTR szFileExtName);

BOOLEAN  VpsExaDirPath(PSTR szFilePath,PSTR szFileDir);
```

上传的附件

| `YasVPS_Src.rar` |
