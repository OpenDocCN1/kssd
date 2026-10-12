
```cpp
void CShareMemDlg::OnSetup() 
{
  if ( bSetup )
  {
    this->MessageBox ( "已经建立共享内存文件" ) ;
    return ;
  }

  hMapFile = CreateFileMapping (
    INVALID_HANDLE_VALUE,
    NULL,
    PAGE_READWRITE | SEC_COMMIT,
    0,
    uShareMemSize,
    "ShareMemSample" ) ;
  if ( hMapFile == NULL )
  {
    this->MessageBox ( "创建共享内存映射文件失败!" ) ;
    return ;
  }

  lpBase  = MapViewOfFile ( 
    hMapFile, 
    FILE_MAP_READ|FILE_MAP_WRITE,
    0,
    0,
    0 ) ;
  if ( this->lpBase == NULL )
  {
    this->MessageBox ( "创建共享内存映射文件失败!" ) ;
    return ;
  }

  this->MessageBox ( "创建内存共享成功!" ) ;
  bSetup = true ;
}
```

（2）从共享内存中读取信息

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

[1]. Windows 核心编程 Jeffrey Richter 著

【版权声明】必须注明原创于看雪技术论坛(bbs.pediy.com) 及作者，并保持文章的完整性。

4dd466d0e1e0cc123afbe504da2462ca
