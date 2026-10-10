

# 提取 PE 输入表（dll 及函数名）

> 原文：[`www.pediy.com/kssd/pediy12/129764.html`](https://www.pediy.com/kssd/pediy12/129764.html)

（全部代码附件下载：）  
关键代码如下：  
void ShowImportDllInfo(LPVOID ImageBase)  
{  
  char \*szdllname;  
  int i=1;  
  PIMAGE\_NT\_HEADERS pnt=NULL;  
  PIMAGE\_IMPORT\_DESCRIPTOR pimport=NULL;  
  pnt=GetNtHead(ImageBase);  
  pimport=GetFirstImportAddress(ImageBase);  
  if(!pimport)  
  {  
    cout<<"GetFirstImportAddress() error"<<endl;  
    return;  
  }  
  while(pimport->FirstThunk)  
  {  
    szdllname=(char\*)RvaToPtr(pnt,ImageBase,pimport->Name);  
    cout<<"NO."<<i<<": "<<szdllname<<endl;  
    i++;  
    pimport++;  
  }  
}  
void ShowImportFuncInfo(LPVOID ImageBase)  
{  
  int i=1;  
  char\* funcname;  
  PIMAGE\_IMPORT\_BY\_NAME pbyname=NULL;  
  DWORD \*pthunk=NULL;  
  DWORD dwthunk;  
  PIMAGE\_IMPORT\_DESCRIPTOR pstart=NULL;  
  PIMAGE\_NT\_HEADERS pnt=NULL;  
  pstart=GetFirstImportAddress(ImageBase);  
  dwthunk=GETTHUNK(pstart);  
  pnt=GetNtHead(ImageBase);  
  pthunk=(DWORD\*)RvaToPtr(pnt,ImageBase,dwthunk);  
  if(!pthunk)  
  {  
    cout<<"RvaToVa() fail"<<endl;  
    return ;  
  }  
  cout<<endl<<">>>====FUNC INFO====<<<"<<endl;  
  while(\*pthunk)  
  {  
    if(HIWORD(\*pthunk)==0x8000)  
    {  
      funcname=(char\*)IMAGE\_ORDINAL32(\*pthunk);  
    }  
    else  
    {  
      pbyname=(PIMAGE\_IMPORT\_BY\_NAME)RvaToPtr(pnt,ImageBase,(DWORD)(\*pthunk));  
      if(pbyname)  
      {  
        funcname=(char\*)pbyname->Name;  
      }  
      else  
      {  
        funcname=(char\*)(DWORD\*)(\*pthunk);  
      }  
    }  
    cout<<"NO."<<i<<": "<<funcname<<endl;  
    i++;  
    pthunk++;  
  }  
}

上传的附件
| ![](img/b7e61e3627e1e92d7b948dd8fdd24fa6.jpg) || pe.doc  
 |