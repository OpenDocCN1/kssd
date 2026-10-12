
# 关于扫雷程序的内存地址读取问题

> 原文：[`www.pediy.com/kssd/pediy12/125934.html`](https://www.pediy.com/kssd/pediy12/125934.html)

有两段代码：

```
BOOL b;

b=ReadProcessMemory(
  ph,   // 目标进程句柄
  (void*)0x01005360,//要读取的内存地址
  lpBuffer1,  // 存放数据的缓存区地址
  1024,      // 要读取的字节数
  NULL  // 把读了多少个字节存到一个地址，如果为 NULL 则忽略此参数。
);

b=ReadProcessMemory(//获取扫雷区域行列数
  ph,   // 目标进程句柄
  (void*)0x01005334,//要读取的内存地址
  lpBuffer2,  // 存放数据的缓存区地址
  8,      // 要读取的字节数
  NULL
);
```

根据之前的资料说：`01005334` 里面是横向方格数，`01005338` 里面是纵向方格数。

那第一个 `ReadProcessMemory` 的作用是什么？为什么要读取 1024 个字节？

那第二个 `ReadProcessMemory` 的作用说是获取扫雷区域行列数，起始地址也是 `01005334` 但是为什么要读取 8 个字节？那不是要读 `01005334`-`01005342` 的数据？这又是为什么？

下面是源码：

```
CWnd* wn;
void* ph;
DWORD pid;
wn = FindWindow(NULL,"扫雷");
if(!wn)
{
  MessageBox("大哥，请你先打开 windows 自带的扫雷游戏!");
  return;
}

GetWindowThreadProcessId((HWND__ *)(*wn),&pid);
ph=OpenProcess(PROCESS_ALL_ACCESS,false,pid);
void *lpBuffer1=new byte[1024];
BOOL b;
b=ReadProcessMemory(
  ph,   // 目标进程句柄
  (void*)0x01005360,//要读取的内存地址
  lpBuffer1,  // 存放数据的缓存区地址
  1024,      // 要读取的字节数
  NULL  // 把读了多少个字节存到一个地址，如果为 NULL 则忽略此参数。
);
if(!b)
{
  MessageBox("读取内存错误，有可能与你的杀毒软件相冲突！");
  return;
}

void *lpBuffer2=new byte[8];
b=ReadProcessMemory(//获取扫雷区域行列数
  ph,   // 目标进程句柄
  (void*)0x01005334,//要读取的内存地址
  lpBuffer2,  // 存放数据的缓存区地址
  8,      // 要读取的字节数
  NULL
);
if(!b)
{
  MessageBox("读取内存错误，有可能与你的杀毒软件相冲突！");
  return;
}

int *aa=(int *)lpBuffer2;
byte *bb=(byte *)lpBuffer1;
for(int y=0;y<aa[1];y++)
{
  for(int x=1;x<=aa[0];x++)
  {
    if(bb[x+(y*32)]==0x8f)
    {//右键
      ::SendMessage((HWND__ *)(*wn),WM_RBUTTONDOWN,0,(0x00400000+(y*0x00100000))|(0x0010*x));
      ::SendMessage((HWND__ *)(*wn),WM_RBUTTONUP,0,(0x00400000+(y*0x00100000))|(0x0010*x));
    }
    else
    {//左键
      ::SendMessage((HWND__ *)(*wn),WM_LBUTTONDOWN,0,(0x00400000+(y*0x00100000))|(0x0010*x));
      ::SendMessage((HWND__ *)(*wn),WM_LBUTTONUP,0,(0x00400000+(y*0x00100000))|(0x0010*x));
    }
  }
}
```

刚好以前有写过一个

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || 扫雷外挂代码.rar

 |
