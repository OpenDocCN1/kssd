
# 滴水逆向学习收获 1-双进程无 dll 注入（1 楼，17 楼，21 楼，27 楼，30 楼，33 楼）[已更新至 33 楼]

> 原文：[`www.pediy.com/kssd/pediy12/142554.html`](https://www.pediy.com/kssd/pediy12/142554.html)

程序执行的过程为：

1. 点击注入按钮：调用`InitEGWork`函数完成创建事件
2. `CreateThread(NULL,NULL,CreateRemoteThreadProc,NULL,0,0);`
   创建线程完成将注入代码写入到被注入进程中，并执行注入代码
3. 注入到被注入进程的代码为

```
DWORD WINAPI RemoteThread(void *lpParameter)
{
  ThreadParam_t *ThreadParamp;
  DWORD Id;
  DWORD dwWaitResult;
  HANDLE hMapFile;
  IoEvnet_t IoEvnet;
  MapFile_t *MapFilep;
  VOID (__stdcall *LocalSleep)(DWORD dwMilliseconds);
  BOOL (__stdcall *LocalSetEvent)(HANDLE hEvent);
  DWORD (__stdcall *LocalWaitForSingleObject)(HANDLE hHandle,DWORD dwMilliseconds);
  //--------------------------------------------------------
  ThreadParamp = (ThreadParam_t*)lpParameter;
  ThreadParamp->ok = 1;
  LocalSleep = ThreadParamp->Sleep;
  LocalSetEvent = ThreadParamp->SetEvent;
  LocalWaitForSingleObject = ThreadParamp->WaitForSingleObject;
  Id = 0;
  dwWaitResult = 0;
  IoEvnet.InEvent = ThreadParamp->OpenEventA(EVENT_ALL_ACCESS, TRUE, ThreadParamp->EVENT_NAMEUI);
  IoEvnet.OutEvent = ThreadParamp->OpenEventA(EVENT_ALL_ACCESS, TRUE, ThreadParamp->EVENT_NAMEEG);
  if (IoEvnet.InEvent==NULL||
    IoEvnet.OutEvent==NULL)
  {
    ThreadParamp->MessageBoxA(NULL,ThreadParamp->OpenIoEvnetError,ThreadParamp->Tip,MB_OK);
  }
  hMapFile = ThreadParamp->OpenFileMapping(FILE_MAP_ALL_ACCESS,FALSE,ThreadParamp->FILEMAP_NAME);
  if (hMapFile==NULL)
  {
    ThreadParamp->MessageBoxA(NULL,ThreadParamp->OpenFileMapError,ThreadParamp->Tip,MB_OK);
  }
  IoEvnet.MapViewpBuff =(BYTE *)ThreadParamp->MapViewOfFile(hMapFile,FILE_MAP_ALL_ACCESS,0,0,MAPFILESIZE);
  if (IoEvnet.MapViewpBuff==NULL)
  {
    ThreadParamp->MessageBoxA(NULL,ThreadParamp->MapViewOfFileError,ThreadParamp->Tip,MB_OK);
  }
  IoEvnet.MapViewpBuff += MAPFILEOFFSET;
  MapFilep = (MapFile_t *)(IoEvnet.MapViewpBuff+8);
  ThreadParamp->ok++;
  //if (ThreadParamp->ThreadId==1)
  {
    ThreadParamp->MessageBoxA(NULL,ThreadParamp->Text,ThreadParamp->Caption,MB_OK);
  }
  __asm
  {
    jmp tcgbegin
    __asm {__emit('T')} __asm {__emit('C')} __asm {__emit('G')}
    __asm {__emit('B')} __asm {__emit('E')} __asm {__emit('G')} __asm {__emit('I')} __asm {__emit('N')}
tcgbegin:
  }
  for (;;)
  {
    dwWaitResult =  LocalWaitForSingleObject(IoEvnet.InEvent,-1);
    if (dwWaitResult==WAIT_OBJECT_0)
    {   
      //--------------------------------------
      //decode
      //------------------------------------------------------------------------------------------------------------------
      Id = *(DWORD*)(IoEvnet.MapViewpBuff+0);
      if (Id==1)
      {
        //------------------------------------------------------------------------------------------------------------------
        ((void (*)(MapFile_t *MapFilep,ULONG* InBuffer,ULONG InLength,ULONG *OutBuffer,ULONG OutLength,ULONG *OutLengthp))(DWORD)MapFilep->PipeFunCodeBuffer)(
          MapFilep,MapFilep->PipeInBuffer,MapFilep->InLength,MapFilep->PipeOutBuffer,MapFilep->OutLength,&MapFilep->OutLengthReal);
        //------------------------------------------------------------------------------------------------------------------
      }
      //------------------------------------------------------------------------------------------------------------------
      LocalSetEvent(IoEvnet.OutEvent);
      //------------------------------------------------------------------------------------------------------------------
    }
  }
  __asm
  {
    jmp labelmark
    __asm {__emit('L')} __asm {__emit('A')} __asm {__emit('B')} __asm {__emit('E')} __asm {__emit('L')}
    __asm {__emit('M')} __asm {__emit('A')} __asm {__emit('R')}
labelmark:
  }
  FUNLENGTHMARK
  return 0;
}
```

4. 点击`Test`按钮程序会把函数`IoTGBTestFunction`拷贝到被注入进程中并执行

该程序源码在附件里，欢迎大家下载和我交流。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `2ProcessInject.rar`  
|
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `TestApp.rar`  
|
