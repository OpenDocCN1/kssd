
```c
typedef struct RBRoot

{

    void* compareFun;                    //用于红黑树比较的函数

    int      const_0;                        //常量

    RBTree* treeHead;                    //红黑树头结点

}RBRoot;

```

# 3 逆向分析流程

## 3.1 创建线程用于文件监控

该函数位于`libdexhelp.so`文件中。如下：

```c
void __fastcall createFileWatch_thread(int fatherPid, pthread_t a2)  

{  

  int v2; // r4  
  signed int v3; // r5  
  _DWORD *pfatherPid; // r6  
  pthread_t newthread; // [sp+4h] [bp-14h]  

  newthread = a2;  
  v2 = fatherPid;  
  v3 = 31;  
  pfatherPid = malloc(4u);  
  *pfatherPid = v2;  
  while ( pthread_create(&newthread, 0, (void *(*)(void *))File_notify_threadProc, pfatherPid) )  
  {  
    if ( !--v3 )  
      break;  
    sleep(1u);  
  }  
}
```

从上面的代码可以看到`File_notify_threadProc`为真正的处理函数。

## 3.2 File_notify_threadProc 函数

```c
signed int __fastcall File_notify_threadProc(__pid_t *pfatherPid)  

int fatherPid; // r5  
unsigned int result; // r0  
signed int v3; // r6  
_DWORD *v4; // r7  
  struct inotify_event *inotifyeventStr; // r0  
  pthread_t newthread; // [sp+4h] [bp-1Ch]  

  fatherPid = *pfatherPid;  
  free(pfatherPid);  
  result = inotify_init_function();  
  if ( result )  
  {  
    inotify_add_watchByPid(fatherPid);  
    v3 = 0x1F;  
    v4 = malloc(4u);  
    *v4 = fatherPid;  
    while ( pthread_create(&newthread, 0, (void *(*)(void *))watchAllTask_threadProc, v4) )  
    {  
      if ( !--v3 )  
        break;  
      sleep(1u);  
    }  
    inotifyeventStr = (struct inotify_event *)read_filewatch_event(-1);  
    if ( inotifyeventStr )  
      GetFileWatchRBTreeKeyName(inotifyeventStr->wd);  
    filewatch_Delete(fatherPid);  
    pthread_kill(newthread, 10);  
    result = killProcess(fatherPid, 9); 
  }  
  return result;
```

该函数步骤如下：

1、调用 `inotify_init_function` 函数用于初始化红黑树头信息

2、调用 `inotify_add_watchByPid(fatherPid)` 函数，将父进程的 `/proc/fatherpid/mem` 与 `/proc/fatherpid/pagemap` 纳入到监控中，同时将相应文件名和 `wd` 插入到红黑树中。

3、创建线程 `watchAllTask_threadProc`，其将 `/proc/fatherpid/task/` 下的所有线程对应的 `mem` 与 `pagemap` 文件纳入到监控中，同时将 同时将相应文件名和 `wd` 插入到红黑树中。

4、调用 `read_filewatch_event` 函数对进行监控，如果没发生事件，则阻塞，如果发生事件，则函数返回。

5、调用 `filewatch_Delete` 移除监控事件。

6、结束 `watchAllTask_threadProc` 线程。

7、结束父进程。

8、线程退出。

## 3.3 inotify_init_function函数

该函数用于初始化文件监控相关信息。本函数经过了混淆，去除如下：

```c
signed int __fastcall inotify_init_function()  
{  
  signed int result; // r0  
  //如果初始化过，则直接返回。  
  if ( g_inotify_init_finshFlag )  
    return 1;  
  g_fileWatch_errno = 0;  
  //初始化文件监控  
  g_inotify_init = inotify_init();  
  if ( g_inotify_init < 0 )  
  {  
    result = 0;  
    g_fileWatch_errno = g_inotify_init;  
    return result;  
  }  
  //置位初始化完成标志  
  g_inotify_init_finshFlag = 1;           
  //初始化以监控句柄比较的红黑树RBRoot 结构  
  g_fileWatch_wd_root = (struct RBRoot *)inotifyfile_ini((int)is_same_inotify_wd, 0);  
  //初始化以监控文件名比较的红黑树RBRoot 结构  
  g_fileWatch_name_root = (struct RBRoot *)inotifyfile_ini((int)is_same_inotify_filename, 0);  
  return 1;  
}
```

从上面可以看到其主要是调用 `inotifyfile_ini` 用于初始化 `g_fileWatch_wd_root` 以及 `g_fileWatch_wd_root`，对应的数据结构为 `RBRoot`。

下面看看 `inotifyfile_ini` 函数：

```c
truct RBRoot *__fastcall inotifyfile_ini(void *comparefun, int const_0)  
{  
  void *comparefun_1; // r5  
  int const_0_1; // r4  
  struct RBRoot *result; // r0  

  comparefun_1 = comparefun;  
  const_0_1 = const_0;  
  result = (struct RBRoot *)malloc(0xCu);  
  if ( result )  
  {  
    result->fun = comparefun_1;  
    result->const_0 = const_0_1;  
    result->root = (struct RBTree *)&g_inotify_node_NoValidFlag;  
  }  
  return result;  
}
```

从上面可以看到 `inotifyfile_ini` 用于 `malloc` 一个 `RBRoot` 结构，同时将初始化相关成员。其中 `g_inotify_node_NoValidFlag` 表示头结点无效。因为目前没有设置任何要监控的文件。

我们看一下2个用于比较的函数。`is_same_inotify_wd` 与 `is_same_inotify_filename`。

### 3.3.1 is_same_inotify_wd函数

该函数用于比较红黑树的 `key` 为 `wd`。

```c
struct FileWatchKey *__fastcall is_same_inotify_wd(struct FileWatchKey *node, int a2)  
{  
  struct FileWatchKey *result; // r0  

  if ( node && a2 )  
    result = (struct FileWatchKey *)(node->wd - *(_DWORD *)(a2 + 4));  
  else  
    result = (struct FileWatchKey *)((char *)node - a2);  
  return result;  
}
```

本函数主要用于判断新的节点是左节点还是右节点。返回0，是同一个节点，大于0是在右节点分支，小于0在左节点分支。

### 3.3.2 is_same_inotify_filename函数

该函数用于比较红黑树的 `key` 为 `filename`。

```c
int __fastcall is_same_inotify_filename(struct FileWatchKey *node, const char *a2)  
{  
  int result; // r0  

  if ( node && a2 )  
    result = strcmp(node->name, *(const char **)a2);  
  else  
    result = (char *)node - a2;  
  return result;  
}
```

本函数主要用于判断新的节点是左节点还是右节点。返回0，是同一个节点，大于0是在右节点分支，小于0在左节点分支。

## 3.4 inotify_add_watchByPid

`inotify_add_watchByPid` 函数将对应进程的 `mem` 与 `pagemap` 文件纳入到监控中，同时创建红黑树节点并插入到相应红黑树中。

```c
int __fastcall inotify_add_watchByPid(int fatherPid)  
{  
  int fatherPid_1; // r7  
  int v3; // [sp+0h] [bp-140h]  
  char v4; // [sp+4h] [bp-13Ch]  
  char v5; // [sp+5h] [bp-13Bh]  
  char v6; // [sp+6h] [bp-13Ah]  
  char v7; // [sp+7h] [bp-139h]  
  char v8; // [sp+8h] [bp-138h]  
  char v9; // [sp+9h] [bp-137h]  
  char v10; // [sp+Ah] [bp-136h]  
  char v11; // [sp+Bh] [bp-135h]  
  char v12; // [sp+Ch] [bp-134h]  
  char v13; // [sp+Dh] [bp-133h]  
  char v14; // [sp+Eh] [bp-132h]  
  char v15; // [sp+10h] [bp-130h]  
  char v16; // [sp+11h] [bp-12Fh]  
  char v17; // [sp+12h] [bp-12Eh]  
  char v18; // [sp+13h] [bp-12Dh]  
  char v19; // [sp+14h] [bp-12Ch]  
  char v20; // [sp+15h] [bp-12Bh]  
  char v21; // [sp+16h] [bp-12Ah]  
  char v22; // [sp+17h] [bp-129h]  
  char v23; // [sp+18h] [bp-128h]  
  char v24; // [sp+19h] [bp-127h]  
  char v25; // [sp+1Ah] [bp-126h]  
  char v26; // [sp+1Bh] [bp-125h]  
  char v27; // [sp+1Ch] [bp-124h]  
  char v28; // [sp+1Dh] [bp-123h]  
  char v29; // [sp+1Eh] [bp-122h]  
  char v30; // [sp+1Fh] [bp-121h]  
  char v31; // [sp+20h] [bp-120h]  
  char v32; // [sp+21h] [bp-11Fh]  
  char v33; // [sp+22h] [bp-11Eh]  
  char procmemString; // [sp+24h] [bp-11Ch]  

  fatherPid_1 = fatherPid;  
  memset(&v3, 0, 0x10u);  
  v4 = -34;  
  v5 = -61;  
  v6 = -49;  
  v8 = -119;  
  v9 = -64;  
  BYTE1(v3) = 126;  
  v10 = -56;  
  HIWORD(v3) = -9085;  
  v7 = -125;  
  v11 = -125;  
  v13 = -55;  
  v12 = -63;  
  v14 = -63;  
  ((void (__fastcall *)(int *, signed int, signed int))DecodeString9)(&v3, 13, 0xD2);// /proc/%ld/mem  
  sprintf(&procmemString, (const char *)&v3, fatherPid_1);  
  inotify_add_watch_insert_node((int)&procmemString, 0xFFFu);  
  memset(&v15, 0, 0x14u);  
  v19 = -43;  
  v20 = -56;  
  v21 = -60;  
  v23 = -126;  
  v24 = -53;  
  v25 = -61;  
  v29 = -64;  
  v17 = -120;  
  v22 = -120;  
  v26 = -120;  
  v30 = -62;  
  v16 = 85;  
  v28 = -58;  
  v31 = -54;  
  v32 = -58;  
  v18 = -41;  
  v27 = -41;  
  v33 = -41;  
  ((void (__fastcall *)(char *, signed int, signed int))DecodeString9)(&v15, 17, 242);// /proc/%ld/pagemap  
  sprintf(&procmemString, &v15, fatherPid_1);  
  return inotify_add_watch_insert_node((int)&procmemString, 0xFFFu);  
}
```

该函数做了如下事情：

1、调用 `DecodeString9` 解密字符串“`/proc/%ld/mem`”；

2、格式化字符串“ `/proc/pid/mem`”；

3、调用 `inotify_add_watch_insert_node` 将对应文件纳入到监控中；

4、调用 `DecodeString9` 解密字符串“`/proc/%ld/pagemap`”；

5、格式化字符串“ `/proc/pid/pagemap`”；

6、调用 `inotify_add_watch_insert_node` 将对应文件纳入到监控中；

### 3.4.1 DecodeString9函数

字符串解密函数，如下：

```c
char * DecodeString9(char *buf, int len, char key) 
{  
    int i; // r4  
    for(i = 0; i < len; i++)  
    {  
        buf[i]= buf[i + 2] ^key ^ buf[i+1];  
    }  
    buf[i]=0;  

    return buf;  
}
```

### 3.4.2 inotify_add_watch_insert_node函数
