
```c
if(ECodeHeaderInfo == NULL)

{

        MessageBoxA(0, ERROR_003, "error", MB_ICONERROR);

        return 0;

}
if(ECodeHeaderInfo->m_dwMark != NEW_E_APP_MARK)

{

        MessageBoxA(0, ERROR_004, "error", MB_ICONERROR);

        return 0;

}
//获取易格式初始化必需的的数据
//获取各个重要数据段的RVA
pConstSectionOffset = (PSECTION_INFO)((UINT32)ECodeHeaderInfo + ECodeHeaderInfo->m_nConstSectionOffset);
pWinFormSectionOffset = (PSECTION_INFO)((UINT32)ECodeHeaderInfo + ECodeHeaderInfo->m_nWinFormSectionOffset);
pHelpFuncSectionOffset = (PSECTION_INFO)((UINT32)ECodeHeaderInfo + ECodeHeaderInfo->m_nHelpFuncSectionOffset);
pCodeSectionOffset = (PSECTION_INFO)((UINT32)ECodeHeaderInfo + ECodeHeaderInfo->m_nCodeSectionOffset);
pVarSectionOffset = (PSECTION_INFO)((UINT32)ECodeHeaderInfo + ECodeHeaderInfo->m_nVarSectionOffset);
//获取DLL命令信息数组
if(ECodeHeaderInfo->m_nDllCmdCount > 0)

{

        DllCmdHead = (PDLLCMD)malloc(sizeof(DLLCMD) * ECodeHeaderInfo->m_nDllCmdCount);
        if(DllCmdHead == NULL)

        {

                MessageBoxA(0, ERROR_007, "error", MB_ICONERROR);

                return 0;

        }
        ThisDllCmd = DllCmdHead;
        for(i = 1; i <= (UINT32)ECodeHeaderInfo->m_nDllCmdCount; i++)

        {

                ThisDllCmd->DllFileName = (char *)((UINT32)ECodeHeaderInfo + pConstSectionOffset->m_nRecordOffset + (*(UINT *)((UINT32)ECodeHeaderInfo + sizeof(APP_HEADER_INFO) + (i - 1) * sizeof(INT))));
                ThisDllCmd->DllCmdName = (char *)((UINT32)ECodeHeaderInfo + pConstSectionOffset->m_nRecordOffset + (*(UINT *)((UINT32)ECodeHeaderInfo + sizeof(APP_HEADER_INFO) + (i + ECodeHeaderInfo->m_nDllCmdCount - 1) * sizeof(INT))));
                ThisDllCmd++;

        }

}
//获取需要的支持库并加载
LibStringHead = (char *)((UINT32)ECodeHeaderInfo + sizeof(APP_HEADER_INFO) + ECodeHeaderInfo->m_nDllCmdCount * sizeof(INT) * 2);
//统计需要加载的支持库数量
ThisLibString = LibStringHead;
LibCount = 0;
while((* ThisLibString) != NULL)

{

        LibCount++;
        ThisLibString += (strlen(ThisLibString) + 1);

}
//加载支持库
LibInfoHead = (PLIBINFO)malloc(sizeof(LIBINFO) * LibCount);
ThisLibInfo = LibInfoHead;
ThisLibString = LibStringHead;
while((* ThisLibString) != NULL)

{

        temp = 0;
        while((* (ThisLibString + temp)) != 0x0d)

        {

                ThisLibStringInfo.LibName[temp] = (* (ThisLibString + temp));
                temp++;

        }
        ThisLibStringInfo.LibName[temp] = 0;
        temp += 1;
        while((* (ThisLibString + temp)) != 0x0d)

        {

                ThisLibStringInfo.ThisGUID[temp - strlen(ThisLibStringInfo.LibName) - 1] = (* (ThisLibString + temp));
                temp++;

        }
        ThisLibStringInfo.ThisGUID[temp] = 0;
        ThisLibInfo->ThisLibHandle = NULL;
        ThisLibInfo->ThisLibInfo = NULL;
        strcpy(ThisLibInfo->ThisLibName, ThisLibStringInfo.LibName);
        strcpy(ThisLibFileName, ThisLibStringInfo.LibName);
        strcat(ThisLibFileName, ".fne");
        FindOK = false;
        ThisLibInfo->ThisLibHandle = LoadLibrary(ThisLibFileName);
        if(ThisLibInfo->ThisLibHandle == NULL)

        {

                strcpy(ThisLibFileName, ThisLibStringInfo.LibName);
                strcat(ThisLibFileName, ".fnr");
                FindOK = false;
                ThisLibInfo->ThisLibHandle = LoadLibrary(ThisLibFileName);
                if(ThisLibInfo->ThisLibHandle == NULL)

                {

                        //没有加载成功，继续加载下一个支持库

                }
                else

                {

                        FindOK = true;

                }

        }
        else

        {

                FindOK = true;

        }
        //加载成功，开始获取支持库信息
        if(FindOK == true)

        {

                GetThisNewInfo = (PFN_GET_LIB_INFO)GetProcAddress(ThisLibInfo->ThisLibHandle, FUNCNAME_GET_LIB_INFO);
                //初始化核心支持库
                GetNewSock = (GETNEWSOCK)GetProcAddress(ThisLibInfo->ThisLibHandle, FUNCNAME_GET_NEW_SOCK);
                if(GetNewSock != NULL)

                {

                        //这是一段罪恶的代码，江山从此易主。希望新版本的核心支持库能分离初始化函数和加载函数
                        GetNewSock(1000);
                        __asm

                        {

                                pop edx
                                mov edx,ESectionVA
                                push edx
                                call eax
                        }
                        Exit();
                        return 0;

                }
                if(GetThisNewInfo == NULL)

                {

                        //载入的库没有输出FUNCNAME_GET_LIB_INFO函数，需要卸载掉
                        FreeLibrary(ThisLibInfo->ThisLibHandle);
                        ThisLibInfo->ThisLibHandle = NULL;

                }
                else

                {

                        ThisLibInfo->ThisLibInfo = GetThisNewInfo();
                        if(ThisLibInfo->ThisLibInfo == NULL)

                        {

                                //输出的PLIB_INFO结构为空，需要卸载这样的支持库
                                FreeLibrary(ThisLibInfo->ThisLibHandle);
                                ThisLibInfo->ThisLibHandle = NULL;
                                ThisLibInfo->ThisLibInfo = NULL;

                        }
                        else

                        {

                                if(strcmp(ThisLibInfo->ThisLibInfo->m_szGuid, ThisLibStringInfo.ThisGUID) != 0)

                                {

                                        //加载的支持库和需要的支持库GUID不同，需要卸载支持库
                                        FreeLibrary(ThisLibInfo->ThisLibHandle);
                                        ThisLibInfo->ThisLibHandle = NULL;
                                        ThisLibInfo->ThisLibInfo = NULL;

                                }
                                else

                                {

                                        //为当前支持库提供Notify函数指针
                                        ThisNotifyLib = ThisLibInfo->ThisLibInfo->m_pfnNotify;
                                        if(ThisNotifyLib != NULL)

                                        {

                                                ThisNotifyLib(NL_SYS_NOTIFY_FUNCTION, (DWORD)MyNotifySys, 0);

                                        }

                                }

                        }

                }

        }
        ThisLibInfo++;
        ThisLibString += (strlen(ThisLibString) + 1);

}
//获取易格式代码的起始指令地址
ECodeStart = (ECODESTART)((UINT32)ECodeHeaderInfo + (UINT32)ECodeHeaderInfo->m_nStartCodeOffset);
if(ECodeStart == NULL)

{
```
