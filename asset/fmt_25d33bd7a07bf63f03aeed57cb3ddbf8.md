
![](img/6eeb3fea26cc01a6c16daba48c513018.jpg)

说明：

每个客户端与一个管道进行交互，而在交互过程中 `I/O` 操作结束后产生的完成包就会进入“`I/O` 完成包队列”。完成端口的线程队列中的线程使用 `GetQueuedCompletionStatus` 来检测“`I/O` 完成包队列”中是否有完成包信息。

### 3、详细设计（关键代码如下，具体见附件中的源码）

```
代码:

* * *

UINT ServerThread ( LPVOID lpParameter )
{
  ……
  while ( true )
  {
    GetQueuedCompletionStatus ( pMyDlg->hCompletionPort, &cbTrans, &dwCompletionKey, &lpov, INFINITE ) ;
    if ( dwCompletionKey == -1 )
      break ;
    // 读取管道信息
    // 响应管道信息（写入）
  }
  return 0 ;
}

void CMyDlg::OnStart() 
{
  // 创建完成端口
  hCompletionPort = CreateIoCompletionPort ( INVALID_HANDLE_VALUE, NULL, 0, nMaxThread ) ;

  CString lpPipeName = "\\\\.\\Pipe\\NamedPipe" ;
  for ( UINT i = 0; i < nMaxPipe; i++ )
  {
    // 创建命名管道
    PipeInst[i].hPipe =  CreateNamedPipe ( lpPipeName, PIPE_ACCESS_DUPLEX|FILE_FLAG_OVERLAPPED, \
          PIPE_TYPE_BYTE|PIPE_READMODE_BYTE|PIPE_WAIT, nMaxPipe, 0, 0, INFINITE, NULL ) ;
    ……
    // 把命名管道与完成端口关联起来
    HANDLE hRet = CreateIoCompletionPort ( PipeInst[i].hPipe, hCompletionPort, i, nMaxThread ) ;
    ……
    // 等待连接
    ConnectNamedPipe ( PipeInst[i].hPipe, &(PipeInst[i].ov) ) ;
  }
  // 创建线程
  for ( i = 0; i < nMaxThread; i++ )
  {
    hThread[i] = AfxBeginThread ( ServerThread, NULL, THREAD_PRIORITY_NORMAL ) ;
  }
  ……
}
void CMyDlg::OnStop() 
{
  for ( UINT i = 0; i < nMaxThread; i++ )
  {
    // 用来唤醒线程的虚假 I/O 完成包
    PostQueuedCompletionStatus ( hCompletionPort, 0, -1, NULL ) ;
    CloseHandle ( hThread[i] ) ;
  }
  for ( i = 0; i < nMaxPipe; i++ )
  {
    DisconnectNamedPipe ( PipeInst[i].hPipe ) ;
    CloseHandle ( PipeInst[i].hPipe ) ;
  }
  ……
}

* * *
```

### 4、心得体会

上面这个例子是关于完成端口的简单应用。可以这样来理解完成端口，它与三种资源相关分别是管道、`I/O` 完成包队列、线程队列，它的作用是协调这三种资源。

【参考文献】

`[1].` `Windows` 系统编程. `Johnson` `M.Hart` 著

【版权声明】必须注明来源看雪技术论坛(`bbs.pediy.com`) 及作者，并保持文章的完整性。

`4dd466d0e1e0cc123afbe504da2462ca`
