

# 仙劍奇俠傳,文件加密解密算法.放出喏~!

> 原文：[`www.pediy.com/kssd/pediy11/115891.html`](https://www.pediy.com/kssd/pediy11/115891.html)

代码:

```
// RSTEncDec.h: interface for the CRSTEncDec class.
//
//////////////////////////////////////////////////////////////////////

#if !defined(AFX\_RSTENCDEC\_H\_\_20112A8E\_16B6\_4E06\_9829\_9CB5FC1F209C\_\_INCLUDED\_)
#define AFX\_RSTENCDEC\_H\_\_20112A8E\_16B6\_4E06\_9829\_9CB5FC1F209C\_\_INCLUDED\_

#if \_MSC\_VER > 1000
#pragma once
#endif // \_MSC\_VER > 1000

#include "DBDecEnc.h"

//-------------------------------------------------------------------------------------------------------------
typedef struct  \_PACKFILE\_HEADER\_INFO{
  DWORD  \_01dwLogon;  //0x1A545352 "RST"
  DWORD  \_02dwUnk  ; //04
  DWORD  \_03dwIndexSeek  ; //08
  DWORD  \_04dwDataSeek   ; //0C
  DWORD  \_05dwLenSub  ; //10
  DWORD  \_06dwFileNum ; //14
  DWORD  \_07dwUnk  ; //18
  DWORD  \_08dwUnk  ; //1C
  DWORD  \_09dwUnk  ; //20
  DWORD  \_0AdwUnk  ; //24
  DWORD  \_0BdwUnk  ; //28
  DWORD  \_0CdwFileSize  ; //2C
  DWORD  \_10dwUnk\[0x14\] ; //30
}\_HEADER\_INFO,\_HEADERINFO;


class CPALFile
{
public:
  CPALFile();
  virtual ~CPALFile();
public:
  HANDLE  hOpen;
  HANDLE  hWrite;
  DWORD  m\_dwCpkRead;
public:
  int        GetSize();
public:
  HANDLE    PalOpen(LPCTSTR strOpen);
  HANDLE    PalOpen(LPCTSTR strOpen,DWORD  dwAccess,DWORD  dwShare,LPSECURITY\_ATTRIBUTES lpSAttributes,
            DWORD  dwMode,DWORD dwFlag,HANDLE  hTemp);
  HANDLE    PalCreate(LPCTSTR strCreate);
  int       PalRead  (HANDLE hFile, LPVOID pbuf, DWORD dwlen);
  int      PalRead  (HANDLE hFile, DWORD dwSeek,LPVOID pbuf, DWORD dwlen);
  int       PalWrite (HANDLE hFile, LPVOID pbuf, DWORD dwlen);
  int      PalWrite (HANDLE hFile, DWORD dwSeek,LPVOID pbuf, DWORD dwlen);
  void      PalClose (HANDLE hFile) { ::CloseHandle(hFile); hFile=NULL; }
};

//-------------------------------------------------------------------------------------------------------------
typedef struct  \_RST\_DATA\_INDEXLIST{
  DWORD  \_01dwUnk;
  DWORD  \_02dwUnk;
  DWORD  \_03dwUnk;
  DWORD  \_04dwSeek;     //文件位置
  DWORD  \_05dwLenght1;  //加密后长度
  DWORD  \_06dwLenght2;  //解密后长度
  DWORD  \_07dwNumber;
  DWORD  \_08dwEnd;
}\_DATA\_LIST;

class CRSTEncDec  
{
public:
  CRSTEncDec();
  virtual ~CRSTEncDec();
private:
  char    DecKey\[0x0100\];
public:
  char    m\_chrData\[0x02\]\[0x1000\];
public:
  int       RSTLoadKey(char \*key1=NULL,char \*key2=NULL,char \*key3=NULL);
  int      RSTDecrpyt(LPVOID  pdat,DWORD  dwnum,LPVOID pkey=NULL);
  int      RSTDecIndex(LPVOID pbuf,DWORD len);
};


class CCPKFile : public CRSTEncDec,public CDBDecEnc
{
public:
  CCPKFile();
  virtual ~CCPKFile();
public:
  CPALFile    file;
  \_HEADERINFO    \_hdi;
  \_DATA\_LIST     \*\_plist;
  int            m\_iListNum;
  DWORD        dwFileSize;
  LPBYTE      pData;
public:
  \_DATA\_LIST\*   GetIndexList()
  {
    return (\_DATA\_LIST\*)pData;
  }
public:
  int   DecCPKFile(LPCTSTR strOpen,LPCTSTR strSave);
  int    DecCPKIndex(LPCTSTR strOpen);
  int    WriteData(LPCTSTR strpath);
  char\* GetDecType(LPVOID pheader);

public:
  int   DecData (char \*pdat,int len,LPCTSTR strpath);
  int   SaveFile(int num,char \*pdat,int len,LPCTSTR strpath,LPCTSTR strname);
};

class CSMPFile : public CRSTEncDec
{
public:
  CSMPFile();
  virtual ~CSMPFile();
public:
  CPALFile    file;
  \_HEADERINFO    \_hdi;
  \_DATA\_LIST     \*\_plist;
  int            m\_iListNum;
  DWORD        dwFileSize;
  LPBYTE      pData;
public:
  \_DATA\_LIST\*   GetIndexList()
  {
    return (\_DATA\_LIST\*)pData;
  }
public:
  int   DecSMPFile(LPCTSTR strOpen,LPCTSTR strSave,LPCTSTR name=NULL);
  int    DecSMPIndex(LPCTSTR strOpen);
  int    WriteData(LPCTSTR strpath,LPCTSTR name);
  char\* GetDecType(LPVOID pheader);
public:
  int   SaveFile(int num,char \*name);
};

#endif // !defined(AFX\_RSTENCDEC\_H\_\_20112A8E\_16B6\_4E06\_9829\_9CB5FC1F209C\_\_INCLUDED\_)

```

  
代码:

```

// RSTEncDec.cpp: implementation of the CRSTEncDec class.
//
//////////////////////////////////////////////////////////////////////

#include "stdafx.h"
#include "Pal4DecPack.h"
#include "RSTEncDec.h"

#ifdef \_DEBUG
#undef THIS\_FILE
static char THIS\_FILE\[\]=\_\_FILE\_\_;
#define new DEBUG\_NEW
#endif

//-----------------------------------------------------------------------------------------------------------------

CPALFile::CPALFile()
{
  hOpen =NULL;
  hWrite=NULL;

  m\_dwCpkRead=0x00;
}
CPALFile::~CPALFile()
{
}

int    CPALFile::GetSize(){ return ::GetFileSize(hOpen,NULL); }

HANDLE CPALFile::PalOpen(LPCTSTR strOpen,DWORD  dwAccess,DWORD  dwShare,LPSECURITY\_ATTRIBUTES lpSAttributes,
            DWORD  dwMode,DWORD dwFlag,HANDLE  hTemp)
{
  hOpen=::CreateFile(strOpen,dwAccess,dwShare,lpSAttributes,
            dwMode,dwFlag,hTemp);
  return hOpen;
}

HANDLE CPALFile::PalOpen(LPCTSTR strOpen)
{
  hOpen=::CreateFile(strOpen,
                 GENERIC\_READ,
             FILE\_SHARE\_READ,
           NULL,
           OPEN\_EXISTING,
           FILE\_ATTRIBUTE\_NORMAL,
           NULL);
  return hOpen;
}

HANDLE CPALFile::PalCreate(LPCTSTR strCreate)
{
    hWrite=CreateFile(strCreate,
                GENERIC\_READ|GENERIC\_WRITE,
                FILE\_SHARE\_READ|FILE\_SHARE\_WRITE,
          NULL,
          CREATE\_NEW,
          FILE\_ATTRIBUTE\_NORMAL,
          NULL);
   if(hWrite==INVALID\_HANDLE\_VALUE)
  {
    hWrite=CreateFile(strCreate,
      GENERIC\_READ|GENERIC\_WRITE,
      FILE\_SHARE\_READ|FILE\_SHARE\_WRITE,
      NULL,CREATE\_ALWAYS,FILE\_ATTRIBUTE\_NORMAL,NULL);
    if(hWrite==INVALID\_HANDLE\_VALUE)
    {
      return NULL;
    }
  }
  return hWrite;
}

int CPALFile::PalRead(HANDLE hFile, LPVOID pbuf, DWORD dwlen)
{
  int iRe=ReadFile(hFile,pbuf,dwlen,&m\_dwCpkRead,NULL);

  int result = iRe==0x01;

  if(dwlen != m\_dwCpkRead){ result=0; }
  return result;
}

int CPALFile::PalRead(HANDLE hFile,DWORD dwSeek, LPVOID pbuf, DWORD dwlen)
{
  OVERLAPPED oap;
  memset(&oap,0x00,sizeof(OVERLAPPED));
  oap.Offset=dwSeek;
  int iRe=ReadFile(hFile,(LPBYTE)pbuf,dwlen,&m\_dwCpkRead,&oap);

  if(dwlen != m\_dwCpkRead){ return m\_dwCpkRead; }
  return dwlen;
}

int CPALFile::PalWrite(HANDLE hFile, LPVOID pbuf, DWORD dwlen)
{
  int iRe=::WriteFile(hFile,pbuf,dwlen,&m\_dwCpkRead,NULL);

  int result = iRe==0x01;

  if(dwlen != m\_dwCpkRead){ result=0; }
  return result;
}

int CPALFile::PalWrite(HANDLE hFile,DWORD dwSeek, LPVOID pbuf, DWORD dwlen)
{
  OVERLAPPED oap;
  memset(&oap,0x00,sizeof(OVERLAPPED));
  oap.Offset=dwSeek;
  int iRe=::WriteFile(hFile,(LPBYTE)pbuf,dwlen,&m\_dwCpkRead,&oap);
  
  if(dwlen != m\_dwCpkRead){ return m\_dwCpkRead; }
  return dwlen;
}




//---------------------------------------------------------------------------------------------------------------



//---------------------------------------------------------------------------------------------------------------
char  szVampire\[0x10\] ="Vampire.C.J at ";
char    szTechn  \[0x20\] ="Softstar Technology (ShangHai)";
char    szCoLtd  \[0x10\] =" Co., Ltd";

DWORD  FILE\_LOGON=0x1A545352; //"RST"

//////////////////////////////////////////////////////////////////////
// Construction/Destruction
//////////////////////////////////////////////////////////////////////

CRSTEncDec::CRSTEncDec()
{
  memset(DecKey,0x00,0x0100);
}

CRSTEncDec::~CRSTEncDec()
{

}

int       CRSTEncDec::RSTLoadKey(char \*key1,char \*key2,char \*key3)
{
  memset(DecKey,0x00,0x100);
  if(key1==NULL||key2==NULL||key3==NULL)
  {
    strcpy(DecKey,szVampire);
    strcat(DecKey,szTechn);
    strcat(DecKey,szCoLtd);
  }else{
    strcpy(DecKey,key1);
    strcat(DecKey,key2);
    strcat(DecKey,key3);
  }

  return 0;
}

int      CRSTEncDec::RSTDecrpyt(LPVOID  pdat,DWORD  dwnum,LPVOID pkey)
{
  DWORD  dwedx;
  DWORD  dweax;
  DWORD  dwebp;
  DWORD  dwebx = dwnum;
  if(dwebx < 0x02) { return -1; }

  if(pkey==NULL){pkey=(LPVOID)&DecKey\[0\];}

  LPDWORD  pdwdat=(LPDWORD)pdat;
  LPDWORD  pdwkey=(LPDWORD)pkey;

  DWORD    dwesi=pdwdat\[0\];

  DWORD    esp10=dwebx;
  DWORD    esp14=0x00;

  dweax=(DWORD)floor( (double)( 52.00f / (float)esp10 + 6.00f) ) \* 0x9E3779B9;
  esp10=dweax;

  if(dweax != 0x00)
  {
    do{
      DWORD   dwecx= dweax;
      DWORD   esp1c=(dwecx / 0x04) & 0x03;

      dwecx = dwebx -0x01;

      if(dwecx > 0x00)
      {
        do{
          dwedx =pdwdat\[dwecx -0x01\];
          dwebx =dwesi \* 0x04;

          dweax =dwedx;
          dwebp =dwedx;

          dweax /= 0x20;
          dwebx ^= dweax;
          dweax  =dwesi;
          dweax /= 0x08;
          dwebp \*= 0x10;
          dweax ^= dwebp;
          dwebp  =esp1c;

          dwebx +=dweax;
          dweax  =dwecx;
          dweax &=0x03;
          dweax ^=dwebp;
          dwebp  =pdwkey\[dweax\];
          dweax  =esp10;  //esp+10
          dwebp ^=dwedx;
          dwedx  =dweax;
          dwedx ^=dwesi;
          dwesi  =pdwdat\[dwecx\];
          dwebp +=dwedx;
          dwebx ^=dwebp;
          dwesi -=dwebx;

          pdwdat\[dwecx\] =dwesi;
          dwecx--;

        }while(dwecx);

        dwebx = dwnum;
      }
        dwebx = pdwdat\[dwebx-1\];
        dwedx = dwesi \* 0x04;
        dwebp = dwebx;

        dwecx &= 0x03;
        dwebp /= 0x20;
        dwedx ^= dwebp;
        dwebp  = dwesi;
        dwebp /= 0x08;
        dwebx \*= 0x10;
        dwebp ^= dwebx;

        dwebx  = esp1c;
        dwecx ^= dwebx;
        dwedx += dwebp;

        dwecx  = pdwkey\[dwecx\];
        dwebx  = dwnum;
        dwebp  = pdwdat\[dwebx-1\];

        dwecx ^=dwebp;
        dwebp  =dweax;
        dwebp ^=dwesi;
        dwesi  =pdwdat\[0\];
        dwecx +=dwebp;
        dwedx ^=dwecx;
        dwesi -=dwedx;
        dweax +=0x61C88647;
        pdwdat\[0\] =dwesi;
        esp10  =dweax;
    }while(dweax);
  }

  return 0;
}


int      CRSTEncDec::RSTDecIndex(LPVOID pbuf,DWORD len)
{
  LPBYTE  pbybuf1=(LPBYTE)pbuf;
  LPBYTE  pbybuf2=&pbybuf1\[len-0x1000\];

  if(len>0x2000)
  {
    RSTLoadKey();
    memset(m\_chrData\[0x00\],0x00,0x1000);
    memset(m\_chrData\[0x01\],0x00,0x1000);

    int num=0;
    do{
      memcpy(&m\_chrData\[0x00\]\[num\],&pbybuf1\[num\],0x08);
      memcpy(&m\_chrData\[0x01\]\[num + 0x04\],&pbybuf2\[num -0x01 \],0x08);
      num += 0x08;
    }while(num<0x1000);

    RSTDecrpyt( &m\_chrData\[0x00\]\[0x00\],0x0400,DecKey);
    RSTDecrpyt( &m\_chrData\[0x01\]\[0x04\],0x0400,DecKey);

    int n=0;

    do{
      memcpy(&pbybuf1\[n\],&m\_chrData\[0x00\]\[n\],0x08);
      memcpy(&pbybuf2\[n -0x01 \],&m\_chrData\[0x01\]\[n + 0x04\],0x08);

      n++;
      num--;
    }while(num);

    return 1;
  }
  return 0;
}


//----------------------------------------------------------------------------------------------------------------------


CCPKFile::CCPKFile()
{
  \_plist     = NULL;
  m\_iListNum = 0;
  dwFileSize = 0;
  pData      = NULL;
  memset(&\_hdi,0x00,sizeof(\_HEADER\_INFO));
}

CCPKFile::~CCPKFile()
{
  if(pData!=NULL)
  {
    //删除内存分配
    delete \[\] pData; pData=NULL;
  }

  if(file.hOpen != NULL)
  {
    file.PalClose(file.hOpen);
    file.hOpen =NULL;
  }
}

int   CCPKFile::DecCPKFile(LPCTSTR strOpen,LPCTSTR strSave)
{
  if(pData==NULL)
  {
    //分配内存
    pData = new BYTE\[0x100000\];
  }else{ memset(pData,0x00,0x100000);}

  HANDLE  hOpen=file.PalOpen(strOpen);
  if(hOpen==INVALID\_HANDLE\_VALUE) {return -1;}

  //获取文件大小
  dwFileSize=::GetFileSize(hOpen,NULL);

  //读取索引头
  DecCPKIndex(strOpen);

  CString ss=strSave;
  ss +="index.bin";
  HFILE  hf=::\_lcreat(ss,NULL);
  ::\_lwrite(hf,(LPCTSTR)pData,0x100000);
  ::\_lclose(hf);

  //写文件:
  WriteData(strSave);

  //清除句柄:
  if(file.hOpen != NULL)
  {
    file.PalClose(file.hOpen);
    file.hOpen =NULL;
  }

  if(file.hWrite != NULL)
  {
    file.PalClose(file.hWrite);
    file.hWrite =NULL;
  }

  delete \[\] pData;

  return 0;
}

int     CCPKFile::DecCPKIndex(LPCTSTR strOpen)
{
  memset(&\_hdi,0x00,sizeof(\_HEADER\_INFO));
  file.PalRead(file.hOpen,&\_hdi,sizeof(\_HEADER\_INFO));

  m\_iListNum=\_hdi.\_06dwFileNum;

  if(\_hdi.\_01dwLogon == FILE\_LOGON)
  {
    if(file.PalRead(file.hOpen,pData,\_hdi.\_05dwLenSub\*0x20)!=0)
    {
      int re=RSTDecIndex(pData,0x100000);
      \_plist=GetIndexList();
      return re;
    }
  }

  return 0;
}

char\*  CCPKFile::GetDecType(LPVOID pheader)
{
  DWORD  dwType=(\*(LPDWORD)pheader & 0x00FFFFFF);

  if(strcmp((char\*)&dwType,"BIK")==0x00){ return "BIK";}

  return "unk";
}

int      CCPKFile::WriteData(LPCTSTR strpath)
{
  if(dwFileSize<=0x100000){return -1;}
  int    size  = dwFileSize -0x100000;
  int     rsize = 0x100000;
  int     seek  = 0x100000;
  
  char  ss  \[MAX\_PATH\] = {0};

  theApp.m\_Dlg->AddLog("分配内存 ...!",0x01);
  LPBYTE  pbuf = new BYTE\[size\];

  int    num  = size/0x100000;
  if( size<0x100000 ){ rsize=size; }
  if( (size%0x100000)>0x00 ){ num++; }
  
  //循环读取文件
  do{
    int iRe = file.PalRead(file.hOpen,seek,pbuf +(seek - 0x100000),rsize);
    seek += rsize;

    wsprintf(ss,"(%d)装载文件: %d / %d",num,seek - 0x100000 ,size);
    theApp.m\_Dlg->AddLog(ss,0x01);
    theApp.m\_Dlg->m\_pgsLog.SetPos( ((seek/0x1000)\*100) /(dwFileSize/0x1000) );

    if( (dwFileSize-seek)<0x100000){ rsize = dwFileSize - seek; }
    num--;
  }while(num);
  
  theApp.m\_Dlg->AddLog("读取完成,解密文件 ...!",0x01);
  
  //保存文件
  DecData((char\*)pbuf,size,strpath);

  delete \[\] pbuf;

  return 0;
}

int     CCPKFile::DecData(char \*pdat,int len,LPCTSTR strpath)
{
  char   ss\[MAX\_PATH\] = {0};
  strcpy(ss,strpath);

  for( int num = 0 ; num < m\_iListNum ; num++ )
  {  
    int    size        =\_plist\[num\].\_05dwLenght1;
    int   seek        =\_plist\[num\].\_04dwSeek   -0x100000;
    char\*  psrc = pdat + seek;

    if(\_plist\[num\].\_05dwLenght1==0x00&&\_plist\[num\].\_06dwLenght2 == 0x00)
    {
      wsprintf(ss,"%s%s",strpath,(char\*)(psrc+size));
      ::CreateDirectory(ss,NULL);
      strcpy(ss,strpath);
      continue;
    }

    DWORD  dstlen=0x00;
    if(\_plist\[num\].\_06dwLenght2 <= 0x1000) { dstlen=0x1000; }
    if(\_plist\[num\].\_06dwLenght2 >  0x1000) { dstlen=(\_plist\[num\].\_06dwLenght2/0x1000)\*0x1000 + 0x1000 ;}

    char\*  pdst = new char\[dstlen\];

    if(\_plist\[num\].\_05dwLenght1 != \_plist\[num\].\_06dwLenght2)
    {
      int iRe=DBDecrpyt(psrc,size,pdst,&dstlen,0x00);
      SaveFile(num,pdst,dstlen,ss,(char\*)(psrc+size));
    }else{
      SaveFile(num,psrc,size,ss,(char\*)(psrc+size));
    }

    delete \[\] pdst;
  }
  return 0;
}

int    CCPKFile::SaveFile(int num,char \*pdat,int len,LPCTSTR strpath,LPCTSTR strname)
{
  char  ss  \[MAX\_PATH\] = {0};
  char  path\[MAX\_PATH\] = {0};

  wsprintf(path,"%s\\\\%s",strpath,strname);
  if(file.PalCreate(path)<0)
  {
    wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:%s  ...创建错误!",
        num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,strname);

    theApp.m\_Dlg->AddLog(ss);
    return 0; 
  }

  wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:%s  ...创建成功!",
      num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,strname);
  theApp.m\_Dlg->AddLog(ss);

  if( len > 0x00)
  {
    int wlen=0x1000;
    int size=0x00;
    int n = len / 0x1000;
    if( len<0x1000 ){ wlen=len; }
    if( (len % 0x1000)> 0x00 ){ n++; }

    do{
      file.PalWrite (file.hWrite,pdat+size,wlen);
      size += wlen;
      
      int p=100;
      if(size > 0x00) { 
      if((size/0x1000)>0x00){ p= ((size /0x1000) \*100)/ (len /0x1000);} 
      }

      wsprintf(ss,"(%d)%s: (%d/%d) \[%s%d\]",
          num,strname,size ,len ,"%",p);
      theApp.m\_Dlg->AddLog(ss,0x01);
      theApp.m\_Dlg->m\_pgsLog.SetPos( p );

      if( (len-size) <0x1000 && (len-size)>0x00 ) { wlen= len-size;}
      if( (len-size) <0x00 ){break;}
      n--;
    }while(n);
  }

  file.PalClose(file.hWrite); file.hWrite=NULL;
  return 1;
}

/\*

int      CCPKFile::WriteData(LPCTSTR strpath)
{
  char  path\[MAX\_PATH\] = {0};
  char  ss  \[MAX\_PATH\] = {0};

  for(int num =0; num < m\_iListNum; num++)
  {
    BYTE  pdat\[0x10\]={0};

    int iRe=file.PalRead(file.hOpen,\_plist\[num\].\_04dwSeek,pdat,0x10); if(iRe<0){ break; }

    wsprintf(path,"%sPAL4\_Dec\_%02X.%s",strpath,num,GetDecType(pdat));
    if(file.PalCreate(path)<0)
    {
      wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:PAL4\_Dec\_%02X.%s  ...创建错误!",
        num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,num,GetDecType(pdat));

      theApp.m\_Dlg->AddLog(ss);
      continue; 
    }
    wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:PAL4\_Dec\_%02X.%s  ...创建成功!",
        num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,num,GetDecType(pdat));

    theApp.m\_Dlg->AddLog(ss);

    SaveFile(num,GetDecType(pdat));

    file.PalClose(file.hWrite); file.hWrite=NULL;

  }

  return 0;
}

int     CCPKFile::SaveFile(int num,char \*sztype)
{
  char    pdat\[0x1000\]   = {0};
  char  ss  \[MAX\_PATH\] = {0};
  int    size        =\_plist\[num\].\_05dwLenght1;
  int   seek        =\_plist\[num\].\_04dwSeek   ;
  int   len         =0;

  if(size>=0x1000){ len=0x1000;}else{len=size;}

  theApp.m\_Dlg->m\_pgsLog.SetPos(0);
  int  wsize=0;

  do{
    int  iRe=file.PalRead(file.hOpen,seek,pdat,len);if(iRe<0){break;}
    file.PalWrite (file.hWrite,pdat,iRe);
    seek  += len;
    wsize += len;

    int p=100;
    if(wsize > 0x00) { 
      if((wsize/0x1000)>0x00){ p= ((wsize /0x1000) \*100)/ (size /0x1000);} 
    }

    wsprintf(ss,"PAL4\_Dec\_%02X.%s ->处理:%d/%d \[%s%d\]",
        num,sztype,wsize ,\_plist\[num\].\_05dwLenght1,"%",
         p);
    
    theApp.m\_Dlg->AddLog(ss,0x01);
    theApp.m\_Dlg->m\_pgsLog.SetPos( p );
    if((size -wsize) < 0x1000){ len=size -wsize ;}
  }while(len>0);
  
  wsprintf(ss,"(%d)处理成功:PAL4\_Dec\_%02X.%s ->\[处理%dByte/原始%dByte\]!",
    num,num,sztype,wsize,\_plist\[num\].\_05dwLenght1);
  theApp.m\_Dlg->AddLog(ss,0x00);

  return 0;
}
\*/



//----------------------------------------------------------------------------------------------------------------------


//----------------------------------------------------------------------------------------------------------------------


CSMPFile::CSMPFile()
{
  \_plist     = NULL;
  m\_iListNum = 0;
  dwFileSize = 0;
  pData      = NULL;
  memset(&\_hdi,0x00,sizeof(\_HEADER\_INFO));
}

CSMPFile::~CSMPFile()
{
  if(pData!=NULL)
  {
    //删除内存分配
    delete \[\] pData; pData=NULL;
  }

  if(file.hOpen != NULL)
  {
    file.PalClose(file.hOpen);
    file.hOpen =NULL;
  }
}

int   CSMPFile::DecSMPFile(LPCTSTR strOpen,LPCTSTR strSave,LPCTSTR name)
{
  if(pData==NULL)
  {
    //分配内存
    pData = new BYTE\[0x100000\];
  }else{ memset(pData,0x00,0x100000);}

  HANDLE  hOpen=file.PalOpen(strOpen);
  if(hOpen==INVALID\_HANDLE\_VALUE) {return -1;}

  //获取文件大小
  dwFileSize=::GetFileSize(hOpen,NULL);

  //读取索引头
  DecSMPIndex(strOpen);

  if(name==NULL)
  {
  //写文件:
  WriteData(strSave,"PAL4\_DEC");
  }else{WriteData(strSave,name);}


  //清除句柄:
  if(file.hOpen != NULL)
  {
    file.PalClose(file.hOpen);
    file.hOpen =NULL;
  }

  if(file.hWrite != NULL)
  {
    file.PalClose(file.hWrite);
    file.hWrite =NULL;
  }

  delete \[\] pData;

  return 0;
}

int     CSMPFile::DecSMPIndex(LPCTSTR strOpen)
{
  memset(&\_hdi,0x00,sizeof(\_HEADER\_INFO));
  file.PalRead(file.hOpen,&\_hdi,sizeof(\_HEADER\_INFO));

  m\_iListNum=\_hdi.\_06dwFileNum;

  if(\_hdi.\_01dwLogon == FILE\_LOGON)
  {
    if(file.PalRead(file.hOpen,pData,\_hdi.\_05dwLenSub\*0x20)!=0)
    {
      int re=RSTDecIndex(pData,0x100000);
      \_plist=GetIndexList();
      return re;
    }
  }

  return 0;
}

char\*  CSMPFile::GetDecType(LPVOID pheader)
{
  DWORD  dwType=(\*(LPDWORD)pheader & 0x00FFFFFF);

  //if(strcmp((char\*)&dwType,"BIK")==0x00){ return "BIK";}

  return "MP3";
}

int      CSMPFile::WriteData(LPCTSTR strpath,LPCTSTR name)
{
  char  path\[MAX\_PATH\] = {0};
  char  ss  \[MAX\_PATH\] = {0};
  char  cc  \[MAX\_PATH\] = {0};

  RSTLoadKey();

  for(int num =0; num < m\_iListNum; num++)
  {
    BYTE  pdat\[0x10\]={0};

    int iRe=file.PalRead(file.hOpen,\_plist\[num\].\_04dwSeek,pdat,0x10); if(iRe<0){ break; }

    if(num==0){wsprintf(cc,"%s.%s",name,GetDecType(pdat));}
    else{wsprintf(cc,"%s\_%02X.%s",name,num,GetDecType(pdat));}

    wsprintf(path,"%s%s",strpath,cc);
    if(file.PalCreate(path)<0)
    {
      wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:%s  ...创建错误!",
        num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,cc);

      theApp.m\_Dlg->AddLog(ss);
      continue; 
    }
    wsprintf(ss,"(%d)创建文件\[%02X\]<0x%08X>:%s  ...创建成功!",
        num,\_plist\[num\].\_07dwNumber,\_plist\[num\].\_04dwSeek,cc);

    theApp.m\_Dlg->AddLog(ss);

    SaveFile(num,cc);

    file.PalClose(file.hWrite); file.hWrite=NULL;

  }

  return 0;
}

int     CSMPFile::SaveFile(int num,char \*name)
{
  char  ss\[MAX\_PATH\]= {0};
  int    size        =\_plist\[num\].\_05dwLenght1;
  int   seek        =\_plist\[num\].\_04dwSeek   ;
  int   len         =0;

  char    \*pdat=new char\[size\];
  memset  (pdat,0x00,size);

  if(size>=0x1000){ len=0x1000;}else{len=size;}

  theApp.m\_Dlg->m\_pgsLog.SetPos(0);
  int  wsize=0;
  int  iRe=file.PalRead(file.hOpen,seek,pdat,size);
  if(iRe<0){return -1;}
  
  RSTDecrpyt(pdat,size /4);

  do{
    file.PalWrite (file.hWrite,&pdat\[wsize\],len);
    wsize += len;

    wsprintf(ss,"%s ->处理:%d/%d \[%s%d\]",
        name,wsize ,\_plist\[num\].\_05dwLenght1,"%",
        ((wsize/0x1000)\*100)/(size/0x1000) );
    
    theApp.m\_Dlg->AddLog(ss,0x01);
    theApp.m\_Dlg->m\_pgsLog.SetPos( ((wsize/0x1000)\*100)/(size/0x1000) );
    if((size -wsize) < 0x1000){ len=size -wsize ;}
  }while(len>0);
  
  delete \[\] pdat;
  wsprintf(ss,"(%d)处理成功:%s ->\[处理%dByte/原始%dByte\]!",
    num,name,wsize,\_plist\[num\].\_05dwLenght1);
  theApp.m\_Dlg->AddLog(ss,0x00);

  return 0;
}

```