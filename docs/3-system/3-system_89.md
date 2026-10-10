

# 一个具体对话框资源的内存图谱

> 原文：[`www.pediy.com/kssd/pediy11/113561.html`](https://www.pediy.com/kssd/pediy11/113561.html)

一个具体对话框资源的内存图谱  
  
大家是否用惯了资源编辑器呢,但具体的对话框资源在内存中的结构图型又是如何的呢,本人花了一点时间,对对话框模板(以扩展对话框模板为例)在内存中的数据进行图形化,以方便大家理解、跟踪和进行相关编程  
  
![](img/2c7ca6690c0d4759f89b61869df40edb.jpg)  
  
相关的数据结构体如下：  
//AFXIMPL.H  
//Extended dialog templates (new in Win95)  
#pragma pack(push, 1)  
typedef struct  
{  
  WORD dlgVer;  
  WORD signature;  
  DWORD helpID;  
  DWORD exStyle;  
  DWORD style;  
  WORD cDlgItems;  
  short x;  
  short y;  
  short cx;  
  short cy;  
} **DLGTEMPLATEEX**;  
  
typedef struct  
{  
  DWORD helpID;  
  DWORD exStyle;  
  DWORD style;  
  short x;  
  short y;  
  short cx;  
  short cy;  
  DWORD id;  
} **DLGITEMTEMPLATEEX**;  
  
#pragma pack(pop)