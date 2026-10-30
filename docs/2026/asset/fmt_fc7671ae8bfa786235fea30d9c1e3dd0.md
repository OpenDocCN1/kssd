
```c
                if( (IsRestore & 0x80000000) == 0x80000000 )//需要恢复5字节代码(内存)
                {
                        memcpy((BYTE*)NextEP,(BYTE*)RestoreHookCodeAddr,5);
                }

                DWORD virtualaddress = image_nt_header->OptionalHeader.ImageBase + (DWORD)NextEP - (DWORD)pMem;
                memset(VirusStartAddr,0,VirusEndAddr-VirusStartAddr);//清除内存中病毒代码

                if( !CaclOEP((BYTE*)NextEP,virtualaddress,0,bLevel) )//是OEP头
                {
                        DWORD OepOffset = NextEP - (DWORD)pMem;

                        if( Count )
                        {
                                BYTE* fileaddr = m_info->pvFile + RvaToOffset(OepOffset);
                                memcpy( fileaddr,(BYTE*)NextGoEsiAddr,Count );
                        }

                        if( (IsRestore & 0x80000000) == 0x80000000 )//需要恢复5字节代码(文件)
                        {
                                BYTE* fileaddr = m_info->pvFile + RvaToOffset(OepOffset);
                                memcpy((BYTE*)fileaddr,(BYTE*)RestoreHookCodeAddr,5);
                        }

                        BYTE* fileaddr = m_info->pvFile + RvaToOffset(VirusStartAddr-pMem);
                        memset((BYTE*)fileaddr,0,VirusEndAddr-VirusStartAddr);//在文件中的代码

                        if( !HookAddr )//没有搜索过
                        {
                                FindVirusEP((BYTE*)NextEP,virtualaddress,1);//搜索OEP附近是否还有病毒头
                        }

                        if( !bLevel )//顶级嵌套
                        {
                                image_nt_header->OptionalHeader.AddressOfEntryPoint = OepOffset;
                                image_nt_header->FileHeader.TimeDateStamp = '\0bug';
                                m_dlg->SetText(m_info->idx,"已清除");
                        }
                }
        }
        return ret;
}

int CaclOEP(BYTE* addr,DWORD VirtualAddress,DWORD HookAddr,BOOL bLevel)
{
        LastFuncAddr = 0;
        dwOffset = 0;
        CodeCout = 0;
        decodekey = 0;
        postion = 0;
        method = -1;
        UseAdd = FALSE;
        AddNum = 0;
        decodereg = -1;
        nAddrReg  = -1;
        int codelen = 0;
        int i = 0;
        //first find reg
        DWORD stepVirtualAddress = VirtualAddress;

        while( i < 0x120 )
        {
                if( addr+i >= VirusStartAddr && addr+i <= VirusEndAddr )//如果在上一个病毒头中
                {
                        i += (DWORD)(VirusEndAddr-VirusStartAddr);
                        stepVirtualAddress += (DWORD)(VirusEndAddr-VirusStartAddr);
                }
                else
                {
                        codelen = FindBaseInfo_Proc(pMem,addr+i,stepVirtualAddress);
                        if( !codelen )
                                break;
                        if( codelen == -1 )//出错
                                codelen = 0x120;//结束
                        i += codelen;
                        stepVirtualAddress += codelen;
                }
        }

        if( i < 0x120 )//是病毒头
        {
                //解码并计算EP
                postion = 0;
                int codelen = 0;
                int i = 0;
                stepVirtualAddress = VirtualAddress;
                VirusStartAddr = (addr+i);//设置起始地址
                while( i < 0x120 )
                {
                        codelen = FindConstInfo_Proc(pMem,addr+i,stepVirtualAddress,HookAddr,bLevel);
                        if( !codelen )
                                break;
                        if( codelen == -1 )//出错
                                codelen = 0x120;//结束
                        i += codelen;
                        stepVirtualAddress += codelen;
                }
                return 1;//病毒头
        }
        else
        {
                return 0;//OEP头
        }

        return 0;
}

//搜索病毒头
void FindVirusEP(BYTE* addr,DWORD VirtualAddress,BOOL bLevel)
{
        int i = 0;
        DWORD stepVirtualAddress = VirtualAddress;
        t_disasm da;

        while( i < 0x200 )
        {
                if( *(addr+i) == 0xCC )//int3
                {
                        return;
                }
                if( *(addr+i) == 0xE8 )//call
                {
                        BYTE* calladdr = (BYTE*)(addr+i + *(DWORD*)(addr+i+1) + 5);
                        if( !IsBadReadPtr(calladdr,5) )//地址有效
                        {
                                DWORD NextVirtualAddress = stepVirtualAddress + *(DWORD*)(addr+i+1) + 5;
                                CaclOEP(calladdr,NextVirtualAddress,(DWORD)addr+i,bLevel);//是病毒头则清除
                        }
                }
                int codelen = Disasm((char*)addr+i,16,stepVirtualAddress,&da,DISASM_CODE);
                i += codelen;
                stepVirtualAddress += codelen;
        }
}

//修复文件
void RepairFile(VirusInfo* info)
{
        if( !info ) return;
        m_info = info;
        image_dos_header = (IMAGE_DOS_HEADER*)info->pvFile;
        image_nt_header = (IMAGE_NT_HEADERS*)(info->pvFile + image_dos_header->e_lfanew);
        image_section_header = (IMAGE_SECTION_HEADER *)(info->pvFile + image_dos_header->e_lfanew + sizeof(IMAGE_NT_HEADERS));
        InitpMem();
        EpAddr = pMem + image_nt_header->OptionalHeader.AddressOfEntryPoint;
        DWORD VirtualAddress = image_nt_header->OptionalHeader.ImageBase + image_nt_header->OptionalHeader.AddressOfEntryPoint;
        VirusStartAddr = NULL;
        VirusEndAddr = NULL;

        if( !CaclOEP(EpAddr,VirtualAddress,0,0) )//不是病毒头,则检测附近是否有stolen code
        {
                FindVirusEP(EpAddr,VirtualAddress,0);//搜索病毒头
        }

        UnmapViewOfFile((LPVOID)m_info->pvFile);m_info->pvFile = NULL;
        CloseHandle(m_info->hFileMap);m_info->hFileMap = NULL;
        CloseHandle(m_info->hFile);m_info->hFile = NULL;
        GlobalFree(pMem);
}

void ClearList()
{
        if( !m_dlg ) return;
        int cout = m_dlg->m_List.GetItemCount();
        for(int i = 0; i < cout; i++)
        {
                VirusInfo* tmpinfo = (VirusInfo*)m_dlg->m_List.GetItemData(i);
                if( tmpinfo )
                {
                        if( tmpinfo->pvFile )
                        {
                                UnmapViewOfFile((LPVOID)tmpinfo->pvFile);tmpinfo->pvFile = NULL;
                        }
                        if( tmpinfo->hFileMap )
                        {
                                CloseHandle(tmpinfo->hFileMap);tmpinfo->hFileMap = NULL;
                        }
                        if( tmpinfo->hFile )
                        {
                                CloseHandle(tmpinfo->hFile);tmpinfo->hFile = NULL;
                        }
                        delete tmpinfo; tmpinfo = NULL;
                }
                m_dlg->m_List.SetItemData(i,0);
        }
}

//开始杀毒
BOOL StartKill(CVx_AntiVirusDlg* dlg,BOOL OnlyCheck)
{
        bOnlyCheck = OnlyCheck;
        InitializeCriticalSection(&cs);
        m_dlg = dlg;
        bStopFind = FALSE;
        strdeque.clear();
        m_dlg->m_List.DeleteAllItems();

        if( dlg->bScanMemory )
        {
                if( !AntiHook() )
                {
                        return FALSE;
                }
        }

        DWORD dwId;
        dwFindThread = CreateThread(NULL,0,FindFileThread,NULL,0,&dwId);
        dwVirusThread = CreateThread(NULL,0,FindVirusThread,NULL,0,&dwId);

        return TRUE;
}

//停止杀毒
void StopKill()
{
        DWORD dw = 0;
        bStopFind = TRUE;
        dw = WaitForSingleObject(dwVirusThread,1500);
        if(dw == WAIT_TIMEOUT)
        {
                TerminateThread(dwVirusThread,0);
        }
        TerminateThread(dwFindThread,0);
```
