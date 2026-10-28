
# [原创][开源]杀毒软件的框架设计

首先推荐下上个帖子 有兴趣移步 哈哈 ![](https://bbs.kanxue.com/view/img/face/27.gif)

网络编程通信方面：

<http://bbs.pediy.com/showthread.php?p=1402027#post1402027>

病毒查杀代码来自[部分更改]：

<http://bbs.pediy.com/showthread.php?t=204845>

服务管理代码来自][添加禁止/启动服务]：

<http://bbs.pediy.com/showthread.php?t=205382>

开始进入正题。为了阅读体验，我源码尽量少贴，只贴关键性代码（我认为的![](https://bbs.kanxue.com/view/img/face/12.gif)）并不是故意![](https://bbs.kanxue.com/view/img/face/1.gif)

环境VS2012 WIN10 X64

界面【具体操作有动画效果】：

![](https://bbs.kanxue.com/upload/attach/201511/607869_kjjly3hy7f0u4g8.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_4jpvpykj91vdq0w.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_16wyfmwjd3j9it6.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_uzafn8d7td5amru.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_89q1g9iqm94tyk8.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_wn9xkhyahc2j2ov.png)
![](https://bbs.kanxue.com/upload/attach/201511/607869_qdhofo09nfnl22k.png)

## 全面体检

我们负责索引病毒查杀、软件卸载、启动项管理、垃圾清理、内存优化这几个类便是。

``` 
HomePage.h     HomePage.cpp

[索引即可，并不关键]
```

## 病毒查杀

采用MD5查杀，MD5计算来自网络，所以开源的好处就是可以帮助需要帮助的人。

`MDC`的值为`CHAR*`类型。我们自己需要个病毒库，或云端、或本地，我并没有，所以也无法上传病毒库啦，但我怎么确定功能是否有效呢？很简单，我们把计算好的`.EXE`、`.TXT`、`.MP3`、`.AVI`、`.XXX`任意类型的MD5值认定为病毒！然后查杀。这么一说是不是感觉so easy！

好了，既然简单也就没什么好说的了。![](https://bbs.kanxue.com/view/img/face/1.gif)

### 全盘查杀

``` 
VirusScanner.h     VirusScanner.cpp

DWORD WINAPI VirusScanner::AllProc(LPVOID lpThreadParameter)
{
  m_this->isInThead=true;
  WCHAR DiskName[MAX_PATH] = {0};
  GetLogicalDriveStrings(MAX_PATH,DiskName);
  int i = 0,j=0;
  for (; DiskName[i]!='\0'&& i < MAX_PATH; i++,j++)
  {
    m_this->TraverseFolder(DiskName,L".txt.exe.dll");
    i+=wcslen(&DiskName[i]);
  }
  if (!m_this->m_ButtonState)
  {
    m_this->  m_BackgroundPic=IDB_BITMAP5;//背景图片
    m_this->  MovebkPic(m_this->m_BackgroundPic);
    m_this->  m_ButtonState=true;
    m_this->  isInThead=false;
    //  m_AllKill.ShowWindow(SW_SHOW);
    //  m_AssignButton.ShowWindow(SW_SHOW)
    m_this->  m_StartupVirus1.ShowWindow(SW_SHOW);
    m_this->  m_StartupVirus2.ShowWindow(SW_SHOW);
  }
  return 0;
}
```

### 闪电查杀

``` 
DWORD WINAPI VirusScanner::LightningProc(LPVOID lpThreadParameter)
{
  //C:\\Program Files (x86)   C:\\Program Files  D:\Program Files
  m_this->isInThead=true;
  WCHAR DiskName[MAX_PATH][MAX_PATH] = {
            _T("C:\\Program Files (x86)"),    //闪电杀毒查杀的范围
            _T("C:\\Program Files"),
            _T("D:\\Program Files"),
            _T("D:\\Program Files (x86)"),
            _T("E:\\Program Files"),
            _T("E:\\Program Files (x86)"),
  };
  int i = 0;
  for (; i<7; i++)
  {
    m_this->TraverseFolder(DiskName[i],L".txt.exe.dll.mp3.avi");  //填写要扫描的类型
  }
  if (!m_this->m_ButtonState)
  {
    m_this->  m_BackgroundPic=IDB_BITMAP5;//背景图片
    m_this->  MovebkPic(m_this->m_BackgroundPic);
    m_this->  m_ButtonState=true;
    m_this->  isInThead=false;
    //  m_AllKill.ShowWindow(SW_SHOW);
    //  m_AssignButton.ShowWindow(SW_SHOW)
    m_this->  m_StartupVirus1.ShowWindow(SW_SHOW);
    m_this->  m_StartupVirus2.ShowWindow(SW_SHOW);
  }
  return 1;
}
```

### 指定位置查杀

``` 
索引AssignKill.h     AssignKill.cpp
```

### 它们都用到的关键性函数

``` 
WIN32_FIND_DATAW findFileData={0};
TCHAR szFind[MAX_PATH] = {_T("\0")};
CString pipei=filetype;//pipei 匹配O(∩_∩)O~
BOOL bRet;
CString fileRootPath = szName;
CString filePath = fileRootPath+_T("\\*.*");
HANDLE hwnd=FindFirstFile(filePath,&findFileData);
if (hwnd==INVALID_HANDLE_VALUE)//没找到文件
{
  MessageBox(_T("该文件路径不存在"),_T("错误"),NULL);
  return;
}
while (true)
{
  if (_tcscmp(findFileData.cFileName,_T("."))==0||
    _tcscmp(findFileData.cFileName,_T(".."))==0 )
  {
    bRet=FindNextFile(hwnd,&findFileData);
    if (!bRet)
    {
      break;
    }
    continue;
  }
  CString path = fileRootPath+ _T("\\")+ findFileData.cFileName;
  int index = path.ReverseFind(_T('.'));
  CString pix  = path.Right(path.GetLength()-index);
  if (pipei.Find(pix)!=-1)
  {
    char *MD5="bacf0d5cec1d91f252b42c39d0bea5ca";//你认为的文件是病毒的MD5的值  
    CStringA ff(path);
    char * h = md5FileValue(ff.GetBuffer());
    if (strcmp(h,MD5)==0)
    {
      //DeleteFile(path);
      MessageBox(L"已查杀");
    }
  }
  if (findFileData.dwFileAttributes & FILE_ATTRIBUTE_DIRECTORY)
  {
    _tcscpy_s(szFind,MAX_PATH,szName);
    _tcscat_s(szFind,_T("\\"));
    _tcscat_s(szFind,findFileData.cFileName);
    TraverseFolder(szFind,pipei);
  }
  bRet=FindNextFile(hwnd,&findFileData);
  if (bRet==0)
  {
    break;
  }
}
```

## 服务管理

用函数遍历服务`ListControl`显示

``` 
ServiceManagement.h     ServiceManagement.cpp

[上个帖子有介绍]
```

## 软件卸载

这个操作关乎到注册表，所以我们要用到注册表相关函数 找到主键遍历子键删除键值就OK啦。

``` 
UninstallTool.h     UninstallTool.cpp.cpp

//加载软件信息
void UninstallTool::UninstallIndex()
{
  LONG nRet=RegOpenKeyEx(RootKey,lpSubKey,0,KEY_ENUMERATE_SUB_KEYS|KEY_QUERY_VALUE,&hkResult);
  if (nRet!=ERROR_SUCCESS)
  {
    return ;
  }
  DWORD dwIndex=0;
  //循环遍历子键
  while (true)
  {
    DWORD dwKeyLen=MAX_KEY_LENGTH;
    //WCHAR szNewKeyName[MAX_PATH]={};
    TCHAR szNewKeyName[MAX_KEY_LENGTH] ;
    nRet=RegEnumKeyEx(hkResult,dwIndex,szNewKeyName,&dwKeyLen,NULL,NULL,NULL,NULL);
    if (nRet!=ERROR_SUCCESS)
    {
      break;    //可断点检查相应错误
    }
    //通过得到的子键组合新的子键路径
    WCHAR strMidReg[MAX_PATH]={};
    swprintf_s(strMidReg,L"%s%s%s",lpSubKey,L"\\",szNewKeyName);
    //打开新的子键获取句柄
    HKEY hkValueKey=0;
    RegOpenKeyEx(RootKey,strMidReg,0,KEY_QUERY_VALUE,&hkValueKey);
    //获取键值
    DWORD dwNameLen=255;
    DWORD dwtype;
    RegQueryValueEx(hkValueKey,L"DisplayName",0,&dwtype,(LPBYTE)SoftInfo.szSoftName,&dwNameLen);//软件名
    if(m_VecSoftInfo.size()>1)
    {
      if ((wcscmp(SoftInfo.szSoftName,m_VecSoftInfo[m_VecSoftInfo.size()-1].szSoftName)==0))
      {
        dwIndex++;
        continue;
      }
    }
    m_UniListCtrl.InsertItem(0,SoftInfo.szSoftName);
    dwNameLen=255;//如果没有重新赋值 下一次将获取不到路径 
    dwtype=0;           //也许dwtype并不要但是 重新赋值并无坏处吧![](https://bbs.kanxue.com/view/img/face/50.gif)
    RegQueryValueEx(hkValueKey,L"UninstallString",0,&dwtype,(LPBYTE)SoftInfo.szSoftUniPath,&dwNameLen);//卸载路径
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"DisplayIcon",0,&dwtype,(LPBYTE)SoftInfo.szSoftIco,&dwNameLen);//光标路径
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"DisplayVersion",0,&dwtype,(LPBYTE)SoftInfo.szSoftVer,&dwNameLen);//版本号
    m_UniListCtrl.SetItemText(0,1,SoftInfo.szSoftVer);
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"Publisher",0,&dwtype,(LPBYTE)SoftInfo.szSoftVenrel,&dwNameLen);//发布厂商
    m_UniListCtrl.SetItemText(0,3,SoftInfo.szSoftVenrel);
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"InstallDate",0,&dwtype,(LPBYTE)SoftInfo.szSoftDate,&dwNameLen);//安装日期
    m_UniListCtrl.SetItemText(0,2,SoftInfo.szSoftDate);
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"URLInfoAbout",0,&dwtype,(LPBYTE)SoftInfo.szURLInfo,&dwNameLen);//网站
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"HelpLink",0,&dwtype,(LPBYTE)SoftInfo.HelpLink,&dwNameLen);//网站
    dwNameLen=255;
    dwtype=0;
    RegQueryValueEx(hkValueKey,L"InstallLocation",0,&dwtype,(LPBYTE)SoftInfo.szSoftInsPath,&dwNameLen);//安装路径
    dwNameLen=255;
    dwIndex++;
    m_VecSoftInfo.push_back(SoftInfo);
  }
}
```

## 启动项管理

这里里面最难的就是那个`ListControl`自绘，我们要在相应行创建按钮并响应相关消息处理这个消息。其实写完了回头看，就会有种并不是很难的感觉。**这个有明显BUG就是切换界面并不刷新界面，当然了 这个程序软件有很多很多BUG，细节处理的问题或其它，毕竟只做5天，也没时间在做下去了，要开始下个阶段的学习了![](https://bbs.kanxue.com/view/img/face/50.gif)**
