
# 实用软件-详细注释的源代码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-659.htm`](https://www.pediy.com/kssd/pediy09/pediy09-659.htm)

> 【文章标题】: 实用软件-详细的源代码  
> 【文章作者】: `CCDeath`  
> 【作者邮箱】: `ccdeath@163.com`  
> 【下载地址】: 附件里下载  
> 【操作平台】: `Mark!!`  
> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】  
>
> 主任 `U` 盘掉了，里面重要资料没了..想要一个软件预防一下...偶闲着没事做了一个，基本符合他的要求....  
>
> 1. `copy` `U` 盘内容到硬盘...自动 `copy` 到自己随便指定的位置,且在后台运行...  
> 2. 能后台压缩该内容，并能自动发送到邮件  
> 3. 随便自定义系统热键  
> 4. 开机自启动,启动后自隐藏功能...  
>
> ```c
> #include "stdafx.h"  
> #include "COPY_U_CCDeath.h"  
> #include "COPY_U_CCDeathDlg.h"  
> #include "mapi.h"  //Added by ccdeath:邮件头文件  
> #include "dbt.h"//Made by  ccdeath  U 盘  
> ```
>
> 现在偶来讲一下了关键实现：  
>
> 1. 重载 `LRESULT CCOPY_U_CCDeathDlg::DefWindowProc(UINT message, WPARAM wParam, LPARAM lParam)` 进行 `U` 盘检测...  
>
> ```c
> LRESULT CCOPY_U_CCDeathDlg::DefWindowProc(UINT message, WPARAM wParam, LPARAM lParam)  
> {  
>   // TODO: Add your specialized code here and/or call the base class  
>   if(message == WM_DEVICECHANGE)//这边详细看该程序帮助的地方文档...  
>   {  
>     CString str;  
>     DEV_BROADCAST_HDR * dhr = (DEV_BROADCAST_HDR *)lParam;//查看 MSDN  
>     switch(wParam)  
>     {  
>     case DBT_DEVICEARRIVAL:  
>       if(dhr->dbch_devicetype==DBT_DEVTYP_VOLUME)//Media 设备插入并且已经可用  
>       {  
>         PDEV_BROADCAST_VOLUME lpdbv =(PDEV_BROADCAST_VOLUME)dhr;//详细看说明为什么可以...     
>           str.Format("%c:",FirstDriveFromMask(lpdbv->dbcv_unitmask));//这个没有库函数用于返回磁盘，需要重新定义   
>
>       //首先要判断是否为 U 盘（偶怕插入其他的东西也拷比较麻烦）...然后...在这边设置 Timer...呵呵..  
>       if(GetDriveType((LPCTSTR)str)==DRIVE_REMOVABLE)  
>       {       
>         this->m_strDisk=str;  
>         SetTimer(1,100,NULL);          
>       }          
>       }  
>       break;//   
>     }   
>   }  
> }
> ```
>
> 复制文件部分 1：//这个部分是把所有文件放在目录下,不能复制文件下，请看第二部分  
>
> ```c
> void CCOPY_U_CCDeathDlg::CopyFile(CString dir)  
> {  
>   //使用 CFileFind,具体使用查看 MSDN  
>   CFileFind fileFind;  
>   CString sFind;  
>   sFind.Format("%s\\\*.\*",dir);//比如"K:"->K:\\*.\*  
>
>   bool IsFile =(bool)fileFind.FindFile(sFind);//查找 K:\下所有的文件和目录...  
>   while(IsFile)//直到没有文件就退出  
>   {  
>     IsFile =(bool)fileFind.FindNextFile();//在根目录和子目录使用是有区别的...  
>     if(!fileFind.IsDots())//这个很重要. 和..  
>     {  
>       CString sFileName = fileFind.GetFileName();  
>       //接下来对文件 和 对文件夹进行处理  
>       if(fileFind.IsDirectory())//有目录进行递归遍利  
>       {  
>         CString tempDir;  
>         tempDir.Format("%s\\%s",dir,sFileName);  
>         CopyFile(tempDir);  
>       }  
>       else//这个处理就比较简单了...  
>       {  
>         CString tempFileName1,tempFileName2;  
>         tempFileName1.Format("%s\\%s",dir,sFileName);  
>         tempFileName2.Format("%s\\%s",this->m_CopyPath,sFileName);  
>         ::CopyFile(tempFileName1,tempFileName2,false);//后台  
>       }  
>     }  
>   }  
>   this->SetDlgItemText(IDC_CHECKCHANGE,"复制文件成功:-)");  
>   bCopy=true;  
> }
> ```
>
> 修正：  复制文件 2：  
>
> ```c
> //复制文件  
> void CCOPY_U_CCDeathDlg::CopyFile(CString dir)  
> {  
> //比先前用的更简单了 SHFILEOPSTRUCT;需要解决 str 与 char 字符的转换,借助 strcpy;  
>
>     char cSourceFile[255+1]="";//假设最大的文件名最大字符串是 255 个字符...  
>     char cDestFile[255+1]="";  
>     dir+="\\\*.\*";//由 K:->k:\\*.\*  
>     strcpy(cDestFile,this->m_CopyPath);  
>     strcpy(cSourceFile,dir);  
>
>     SHFILEOPSTRUCT  fileop;//shell 文件操作..   shell file operator struct;  
>     fileop.hwnd = m_hWnd; //句柄   
>     fileop.wFunc= FO_COPY;//复制   
>     fileop.pFrom= cSourceFile;//从 U 盘里    
>     fileop.pTo=cDestFile; //       
>     fileop.fFlags = FOF_NOCONFIRMATION |  FOF_SILENT  ;   
>     if(SHFileOperation(&fileop)!=0)    
>     {  
>   return ;  
>     }    
>
>     this->SetDlgItemText(IDC_CHECKCHANGE,"复制文件成功:-)");  
>     bCopy=true;  
> }
> ```
>
> 2. 解决压缩文件的做法,偶采用 `winrar` 自己带的命令行...  
>
> ```c
> bool  CCOPY_U_CCDeathDlg::OnWinrarPress()  
> {  
>   this->SetDlgItemText(IDC_CHECKCHANGE,"正在压缩文件中...");  
>   STARTUPINFO si;  
>   ZeroMemory(&si, sizeof(si));  
>   si.cb = sizeof STARTUPINFO;  
>   si.wShowWindow = SW_SHOW;  
>   si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;  
>   PROCESS_INFORMATION pi;  
>   CString str="C:\\Program Files\\WinRAR\\Rar.exe a -r ";  
>   str+=m_CopyPath; //压缩后的路径  
>   str+=" ";//中间加个空格  
>   str+=m_CopyPath;//要压缩的目录  
>
>   BOOL res = CreateProcess(NULL,  
> //  "C:\\Program Files\\WinRAR\\Rar.exe a -r D:\\CC.rar D:\\cc",  
>   str.GetBuffer(0),  
>   NULL,  
>   NULL,  
>   NULL,  
>   NORMAL_PRIORITY_CLASS | CREATE_NO_WINDOW,  
>   NULL,  
>   NULL,  
>   &si,  
>   &pi);  
>   if (TRUE == res)  
>   {  
>   //等待进程执行完毕   
>     WaitForSingleObject(pi.hProcess, INFINITE);  //来自自己写的《完美多线程设计》  
>     CloseHandle(pi.hProcess);  
>     CloseHandle(pi.hThread);  
>     this->SetDlgItemText(IDC_CHECKCHANGE,"压缩成功:-)\r\n 正在发送邮件中...");  
>     return 1;  
>   }  
>   else  return 0;  
> }
> ```
>
> 3. 解决发送邮件问题:`outlook` 中用 `shellexcute` 不能带有附件,所以只能重写 `mapi`  
>
> ```c
> if(bCopy)  
>  {  
>   if(this->OnWinrarPress())  
>   {  
>     UpdateData(true);  
>
>
>     AfxGetApp()->WriteProfileInt("MAIL","AutoMail",0);  
>     UpdateData(false);  
>
>     HMODULE hMod = LoadLibrary("MAPI32.DLL");  
>    
>     if(hMod == NULL)  
>     {  
>       AfxMessageBox(AFX_IDP_FAILED_MAPI_LOAD);//无法装入系统  
>       return ;  
>     }  
>   //从这个动态链接库发取出送邮件的函数地址  
>     ULONG (PASCAL *lpfnSendMail)(ULONG ,ULONG,MapiMessage*,FLAGS,ULONG);  
>     (FARPROC&)lpfnSendMail=GetProcAddress(hMod,"MAPISendMail");//取出"MAPISendMail"  
>
>     if(lpfnSendMail == NULL)  
>     {  
>       AfxMessageBox(AFX_IDP_INVALID_MAPI_DLL);  
>       return ;  
>     }  
> //  else{AfxMessageBox("i love you");}  
> //收件人结构信息  
>     CString sAddress = this->m_MailAddress;  
>     CString sName=this->m_CopyPath;  
>     sName+=".rar";  
>     MapiRecipDesc recip;  
>     memset(&recip,0,sizeof(MapiRecipDesc));  
>     recip.lpszAddress  = sAddress.GetBuffer(0);  
>     recip.ulRecipClass = MAPI_TO;  
>     recip.lpszName = sAddress.GetBuffer(0);  
> //附件结构信息  
>     MapiFileDesc FileDesc;  
>     memset(&FileDesc,0,sizeof(FileDesc));  
>     FileDesc.lpszPathName=sName.GetBuffer(0);  
>     FileDesc.nPosition=(ULONG)-1;//附件在邮件中的位置  
>
> //邮件结构信息  
>     MapiMessage message;  
>     memset(&message,0,sizeof(message));  
>     message.nRecipCount = 1;//收件人的个数，只有一个了  
>     message.lpRecips = &recip;//收件人信息  
>     message.nFileCount = 1;//附件的个数，只有一个了  
>     message.lpFiles = &FileDesc;//附件信息  
>     message.lpszSubject = "Happy Every Day :-)";//主题信息  
>     message.lpszNoteText = "天天快乐";//正文内容，不需要  
>
> //保存本窗口指针,发送完邮件要返回本程序窗口  
>     CWnd *pCurrentWnd = CWnd::GetSafeOwner(NULL,NULL);  
>
> //调用发送邮件函数，发送邮件  
>     int nError = lpfnSendMail(0,0,&message,MAPI_LOGON_UI | MAPI_DIALOG,0);  
>    
>     if(nError != SUCCESS_SUCCESS && nError != MAPI_USER_ABORT  
>       && nError!= MAPI_E_LOGIN_FAILURE)  
>     {  
>       AfxMessageBox(AFX_IDP_FAILED_MAPI_SEND);  
>     }  
>    
> //返回程序  
>     pCurrentWnd->SetActiveWindow();  
>
> //释放分配的内存    
>
>     FreeLibrary(hMod);  
>     this->SetDlgItemText(IDC_CHECKCHANGE,"邮件发送成功:-)");  
>     UpdateData(false);  
>
>
>   }
> }
> ```



```
      }  
    else  
      this->SetDlgItemText(IDC_CHECKCHANGE,"还没复制好文件,请重插 U 盘....");  
  //UpdateWindow();  
  }  
```
程序里面有详细的说明及附一份小文档....

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

  2007 年 11 月 14 日 18:37:27

`4dd466d0e1e0cc123afbe504da2462ca`
