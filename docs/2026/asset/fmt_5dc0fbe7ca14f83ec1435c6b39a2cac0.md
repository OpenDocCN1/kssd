
```cpp
void CShareMemReadDlg::OnRead() 
{
	HANDLE hMapFile = OpenFileMapping ( FILE_MAP_READ, FALSE, "ShareMemSample"); 
	if ( hMapFile == NULL )
	{
		DWORD dwErrorCode = GetLastError () ;
		this->MessageBox ( "打开共享内存映射文件失败!" ) ;
		return ;
	}

	LPVOID lpBase = MapViewOfFile ( 
		hMapFile, 
		FILE_MAP_READ,
		0,
		0,
		0 ) ;
	if ( lpBase == NULL )
	{
		DWORD dwErrorCode = GetLastError () ;
		this->MessageBox ( "打开共享内存映射文件失败!" ) ;
		return ;
	}

	char *pReadInfo = (char*)lpBase ;
	UINT uLength = strlen ( pReadInfo ) ;	

	memcpy ( this->m_ReadInfo.GetBuffer( uLength + 1 ), lpBase, uLength + 1 ) ;
	this->UpdateData ( false ) ;

	UnmapViewOfFile ( lpBase ) ;
	CloseHandle ( hMapFile ) ;
}
```

【参考文献】

[1].Windows核心编程 Jeffrey Richter著

【版权声明】必须注明原创于看雪技术论坛(bbs.pediy.com) 及作者，并保持文章的完整性。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [memmap.rar](javascript:void(0);)

  （106.32kb，183次下载）
