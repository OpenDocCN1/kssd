
```
int main(int argc, char *argv[])

{

	DWORD dwPID = 0;

	char *pFilePathParam = NULL;

	char *pContentParam = NULL;

	DWORD dwReturnValue = 0;

	DWORD dwParamList[16];

	HANDLE hProcess = NULL;

	void *pFilePath = NULL;

	void *pContent = NULL;

	void *pBytesWritten = NULL;

	HANDLE hFile = NULL;



	printf("CallRemoteAPI - e01K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4R3^5y4X3#2S2N6s2c8Z5k6i4N6Q4x3X3g2U0L8$3#2Q4y4f1y4F1i4K6g2o6L8W2)9J5y4Y4q4#2L8%4c8Q4x3@1u0Q4x3U0W2Q4x3@1t1`.



	if(argc != 4)

	{

		printf("Usage: %s [target_pid] [file_path] [content]\n\n", argv[0]);



		return 1;

	}



	// get command-line param values

	dwPID = atoi(argv[1]);

	pFilePathParam = argv[2];

	pContentParam = argv[3];



	printf("Opening process: %u...\n", dwPID);



	// open target process

	hProcess = OpenProcess(PROCESS_ALL_ACCESS, 0, dwPID);

	if(hProcess == NULL)

	{

		// error

		printf("Failed to open process: %u\n", dwPID);



		return 1;

	}



	printf("Allocating data in remote process...\n\n");



	// allocate string in remote process

	pFilePath = AllocateRemoteString(hProcess, pFilePathParam);

	if(pFilePath == NULL)

	{

		printf("Failed to allocate string\n");



		// error

		CloseHandle(hProcess);



		return 1;

	}



	// allocate string in remote process

	pContent = AllocateRemoteString(hProcess, pContentParam);

	if(pContent == NULL)

	{

		printf("Failed to allocate string\n");



		// error

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	// allocate dword value (BytesWritten) in remote process

	pBytesWritten = AllocateRemoteData(hProcess, NULL, sizeof(DWORD));

	if(pBytesWritten == NULL)

	{

		printf("Failed to allocate value\n");



		// error

		VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	// set CreateFile parameters

	dwParamList[0] = (DWORD)pFilePath;

	dwParamList[1] = GENERIC_WRITE;

	dwParamList[2] = 0;

	dwParamList[3] = 0;

	dwParamList[4] = CREATE_ALWAYS;

	dwParamList[5] = FILE_ATTRIBUTE_NORMAL;

	dwParamList[6] = 0;



	printf("Calling CreateFileA...\n");



	// call CreateFileA

	if(CallRemoteAPI(hProcess, (DWORD)CreateFileA, dwParamList, 7, &dwReturnValue) != 0)

	{

		printf("Failed to call remote API\n");



		// error

		VirtualFreeEx(hProcess, pBytesWritten, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	printf("CreateFileA returned 0x%08x\n\n", dwReturnValue);



	// store the returned file handle

	hFile = (HANDLE)dwReturnValue;



	// check if CreateFile failed

	if(hFile == INVALID_HANDLE_VALUE)

	{

		printf("CreateFileA failed\n");



		// error

		VirtualFreeEx(hProcess, pBytesWritten, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	// set WriteFile parameters

	dwParamList[0] = (DWORD)hFile;

	dwParamList[1] = (DWORD)pContent;

	dwParamList[2] = strlen(pContentParam);

	dwParamList[3] = (DWORD)pBytesWritten;

	dwParamList[4] = 0;



	printf("Calling WriteFile...\n");



	// call WriteFile

	if(CallRemoteAPI(hProcess, (DWORD)WriteFile, dwParamList, 5, &dwReturnValue) != 0)

	{

		printf("Failed to call remote API\n");



		// error

		VirtualFreeEx(hProcess, pBytesWritten, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	printf("WriteFile returned 0x%08x\n\n", dwReturnValue);



	// set CloseHandle parameters

	dwParamList[0] = (DWORD)hFile;



	printf("Calling CloseHandle...\n");



	// call CloseHandle

	if(CallRemoteAPI(hProcess, (DWORD)CloseHandle, dwParamList, 1, &dwReturnValue) != 0)

	{

		printf("Failed to call remote API\n");



		// error

		VirtualFreeEx(hProcess, pBytesWritten, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

		VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

		CloseHandle(hProcess);



		return 1;

	}



	printf("CloseHandle returned 0x%08x\n\n", dwReturnValue);



	// clean up

	VirtualFreeEx(hProcess, pBytesWritten, 0, MEM_RELEASE);

	VirtualFreeEx(hProcess, pContent, 0, MEM_RELEASE);

	VirtualFreeEx(hProcess, pFilePath, 0, MEM_RELEASE);

	CloseHandle(hProcess);



	return 0;

}
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#开发技巧](https://bbs.kanxue.com/forum-41-1-135.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
