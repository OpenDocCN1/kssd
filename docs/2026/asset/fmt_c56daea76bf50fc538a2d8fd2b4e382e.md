
```
void CExportDir::GetExportInfo()

{

    LPCWCH outputchar={0};

    LPCWCH outputchar1={0};

    m_exportlistcrl.DeleteAllItems();

    m_vec_myexportinfo.clear();

    PeMisicInfo miscinfo=ReturnMiscInfo();//获取pe基本信息

    PIMAGE_NT_HEADERS32   pNT32    = miscinfo.nt_headeras;

    PIMAGE_DATA_DIRECTORY pDataDir = (PIMAGE_DATA_DIRECTORY)pNT32->OptionalHeader.DataDirectory;

    PIMAGE_DATA_DIRECTORY   pExportDir      = &pDataDir[IMAGE_DIRECTORY_ENTRY_EXPORT];

    if(pExportDir->Size!=0)

    {

    DWORD                   dwExportOfffset = Rva2FileA(pExportDir->VirtualAddress, pNT32);//输出表地文件偏移

    PIMAGE_EXPORT_DIRECTORY pExport         = (PIMAGE_EXPORT_DIRECTORY)((DWORD)miscinfo.Dumpaddr+dwExportOfffset);

    PDWORD pEAT = (PDWORD)((DWORD)miscinfo.Dumpaddr + Rva2FileA(pExport->AddressOfFunctions, pNT32));

    PDWORD pENT = (PDWORD)((DWORD)miscinfo.Dumpaddr + Rva2FileA(pExport->AddressOfNames, pNT32));

    PWORD  pEIT = (PWORD)((DWORD)miscinfo.Dumpaddr  + Rva2FileA(pExport->AddressOfNameOrdinals, pNT32));

    PCHAR exenameaddr=(PCHAR)((DWORD)miscinfo.Dumpaddr + Rva2FileA(pExport->Name, pNT32));  

    mysttring.ConvertUtf8ToUnicode((char*)exenameaddr,outputchar1);

    /////////////////////////////////////////////

    //格式化输出

    m_ExpDirFa.Format(L"0x%X",dwExportOfffset);

    m_funaddr.Format(L"0x%X",pExport->AddressOfFunctions);

    m_funnameaddr.Format(L"0x%X",pExport->AddressOfNames);

    m_funnamenums.Format(L"%d",pExport->NumberOfNames);

    m_char.Format(L"%d",pExport->Characteristics);

    m_exename.Format(L"%s",outputchar1);

    m_funordaddr.Format(L"0x%X",pExport->AddressOfNameOrdinals);

    m_funnums.Format(L"0x%X",pExport->NumberOfFunctions);

    m_baseaddr.Format(L"0x%X",pExport->Base);

    m_namerva.Format(L"0x%X",pExport->Name);

    for ( DWORD dwOrdinal=0; dwOrdinal<pExport->NumberOfFunctions; dwOrdinal++ )

    {

        if ( !pEAT[dwOrdinal] )

            continue;

        for ( DWORD dwIndex=0; dwIndex<pExport->NumberOfFunctions; dwIndex++ )

        {

         EXPORTINFO myexportinfo={0};



            if ( pEIT[dwIndex] == dwOrdinal )

            {

                PCHAR pszFunName = (PCHAR)((DWORD)miscinfo.Dumpaddr+Rva2FileA(pENT[dwIndex], pNT32));

                myexportinfo.Ordinal=pExport->Base+dwOrdinal;

                myexportinfo.FuncRva=pEAT[dwOrdinal];

                mysttring.ConvertUtf8ToUnicode(pszFunName,outputchar);

                myexportinfo.funcname=outputchar;

                m_vec_myexportinfo.push_back(myexportinfo);

                break;

            }

            else if ( dwIndex == pExport->NumberOfFunctions-1 )

            {

                myexportinfo.Ordinal=pExport->Base+dwOrdinal;

                myexportinfo.FuncRva=pEAT[dwOrdinal];

                myexportinfo.funcname=(LPCWCH) L"(Null)" ;

                m_vec_myexportinfo.push_back(myexportinfo);

                break;

            }

          /// free(myexportinfo); 

           

        }



    }

    ///////////////////////////////////./////

    //显示信息

    ////////////////////////////////////////

    UpdateData(FALSE);

    Showinfo();

    }

    else

    {

        MessageBox(L"没有导出表!",L"错误",MB_OK|MB_ICONSTOP);

    }

}
```

### 3.资源表信息

这是插入时的一个细节问题大家结合图与代码看看就明白了

![](https://bbs.kanxue.com/upload/attach/201312/461524_4jv2gji7xujmzsg.png)

```
void CSrc::ShowInfo()

{

    //这里用ID关联资源之间的关系，来显示层与层之间的关系

    int ID1=0;

    m_srcTreecrl.ModifyStyle(NULL,TVS_HASBUTTONS | TVS_HASLINES | TVS_LINESATROOT);

    PeMisicInfo miscinfo=ReturnMiscInfo();

    PIMAGE_NT_HEADERS32 pNT32 = (PIMAGE_NT_HEADERS32)miscinfo.nt_headeras;

    // 1.2 获取数据目录表

    PIMAGE_DATA_DIRECTORY pDir = (PIMAGE_DATA_DIRECTORY)pNT32->OptionalHeader.DataDirectory;

    // 1.3 获取资源表的数据目录

    PIMAGE_DATA_DIRECTORY pDataDir = pDir+IMAGE_DIRECTORY_ENTRY_RESOURCE;

    // 1.4 获取资源表

    PIMAGE_RESOURCE_DIRECTORY pResource1 = (PIMAGE_RESOURCE_DIRECTORY)RVA2Memaddr(pDataDir->VirtualAddress, miscinfo.Dumpaddr, pNT32);

    // 2. 循环遍历资源表

    // 2.1 第一层目录遍历

    DWORD dwCount1 = pResource1->NumberOfIdEntries+pResource1->NumberOfNamedEntries;

    for ( DWORD i=0; i<dwCount1; i++ )

    {

        SRCSTRUCT mystc={0};

        ID1++;

        mystc.ID=ID1;

        LPCWCH* outputchar=new LPCWCH;

        PIMAGE_RESOURCE_DIRECTORY_ENTRY pDirEntry1 = (PIMAGE_RESOURCE_DIRECTORY_ENTRY)((DWORD)pResource1+sizeof(IMAGE_RESOURCE_DIRECTORY));

        if ( pDirEntry1[i].NameIsString )

        {

            PIMAGE_RESOURCE_DIR_STRING_U pString = (PIMAGE_RESOURCE_DIR_STRING_U)((DWORD)pResource1+pDirEntry1[i].NameOffset);

            mystc.str.Format(L"Type:%ls",pString->NameString);

            Level1.push_back(mystc);

        } 

        else

        {

            if ( pDirEntry1[i].Name>0x10 )

            {

                mystc.str.Format(L"%04X",pDirEntry1[i].Name);

                Level1.push_back(mystc);

            }



            else

            {

                mystc.str.Format(L"%ls",szResourceType[pDirEntry1[i].Name] );

                Level1.push_back(mystc);

            }



        }

        if ( pDirEntry1[i].DataIsDirectory )

        {

            int secid=0;

            // 2.2 第二层目录的遍历

            PIMAGE_RESOURCE_DIRECTORY pResource2 = (PIMAGE_RESOURCE_DIRECTORY)((DWORD)pResource1+pDirEntry1[i].OffsetToDirectory);

            DWORD dwCount2 = pResource2->NumberOfIdEntries+pResource2->NumberOfNamedEntries;

            for ( DWORD j=0; j<dwCount2; j++ )

            {



                SRCSTRUCT mystc2={0};

                mystc2.ID=ID1;

                //第二级的id

                secid++;

                mystc2.secid=secid;

                PIMAGE_RESOURCE_DIRECTORY_ENTRY pDirEntry2 = (PIMAGE_RESOURCE_DIRECTORY_ENTRY)((DWORD)pResource2+sizeof(IMAGE_RESOURCE_DIRECTORY));

                if ( pDirEntry2[j].NameIsString )

                {

                    LPWSTR szBuffer[30];

                    PIMAGE_RESOURCE_DIR_STRING_U pString = (PIMAGE_RESOURCE_DIR_STRING_U)((DWORD)pResource2+pDirEntry2[j].NameOffset);

                    mystc2.str.Format(L"ID:%ls",pString->NameString);

                    Level2.push_back(mystc2);



                } 

                else

                {

                    mystc2.str.Format(L"ID:%X",pDirEntry2[j].Name);

                    Level2.push_back(mystc2);



                }

                if ( pDirEntry2[j].DataIsDirectory )

                {

                   DWORD subId=0;

                    // 2.3 第三层目录的遍历

                    PIMAGE_RESOURCE_DIRECTORY pResource3 = (PIMAGE_RESOURCE_DIRECTORY)((DWORD)pResource1+pDirEntry2[j].OffsetToDirectory);

                    DWORD dwCount3 = pResource3->NumberOfIdEntries+pResource3->NumberOfNamedEntries;

                    for ( DWORD k=0; k< dwCount3; k++ )

                    {

                        SRCSTRUCT mystc3={0};

                        mystc3.secid=secid;

                        mystc3.ID=ID1;

                        subId++;

                        mystc3.subID=subId;

                        PIMAGE_RESOURCE_DIRECTORY_ENTRY pDirEntry3 = (PIMAGE_RESOURCE_DIRECTORY_ENTRY)((DWORD)pResource3+sizeof(IMAGE_RESOURCE_DIRECTORY));

                        PIMAGE_RESOURCE_DATA_ENTRY pData = (PIMAGE_RESOURCE_DATA_ENTRY)((DWORD)pResource1+pDirEntry3[k].OffsetToData);

                        mystc3.str.Format(L"Lng:%04X,RVA:0x%p Size:%04X Page:0x%p",pDirEntry3[k].Name,pData->OffsetToData, pData->Size, pData->CodePage);

                        Level3.push_back(mystc3); 

                    }

                } 

                else

                {

                    // ....

                }

           



            }

        } 

        else

        {

            // ......

        }

    }

RealShowInfo();

 }
```

### 导入表信息

### 获取模块信息
