
```c
int __fastcall insertNewNode(int inotifyfileKeyBuf, int a2)  

{  

  int result; // r0  
  int v3; // r0  

  if ( a2 && (query_insert_node(1, inotifyfileKeyBuf, (struct RBRoot *)a2), (void **)v3 != &g_inotify_node_NoValidFlag) )  
    result = *(_DWORD *)(v3 + 16);  
  else  
    result = 0;  
  return result;  

}
```

该函数调用`query_insert_node`进行新节点的插入操作。

至此函数`inotify_add_watchByPid`分析完了。

下面看看`watchAllTask_threadProc`函数的工作。

## 3.5 watchAllTask_threadProc线程入口函数

```c
void __fastcall __noreturn watchAllTask_threadProc(int *pfatherPid)  

{  
  struct dirent *v1; // r5  
  const char *v2; // r5  
  int v3; // r0  
  int threadID; // r0  
  DIR *dirp; // [sp+4h] [bp-2CCh]  
  int pfatherPid_1; // [sp+Ch] [bp-2C4h]  
  int dot; // [sp+14h] [bp-2BCh]  
  int dotdot; // [sp+18h] [bp-2B8h]  
  char v9; // [sp+1Ch] [bp-2B4h]  
  char v10; // [sp+20h] [bp-2B0h]  
  char v11; // [sp+21h] [bp-2AFh]  
  char v12; // [sp+22h] [bp-2AEh]  
  char v13; // [sp+23h] [bp-2ADh]  
  char v14; // [sp+24h] [bp-2ACh]  
  char v15; // [sp+25h] [bp-2ABh]  
  void (__noreturn *pthread_exit)(); // [sp+28h] [bp-2A8h]  
  char v17; // [sp+38h] [bp-298h]  
  __int16 v18; // [sp+48h] [bp-288h]  
  char v19; // [sp+A0h] [bp-230h]  
  char v20; // [sp+A1h] [bp-22Fh]  
  char v21; // [sp+A2h] [bp-22Eh]  
  char v22; // [sp+A3h] [bp-22Dh]  
  char v23; // [sp+A4h] [bp-22Ch]  
  char v24; // [sp+A5h] [bp-22Bh]  
  char v25; // [sp+A6h] [bp-22Ah]  
  char v26; // [sp+A7h] [bp-229h]  
  char v27; // [sp+A8h] [bp-228h]  
  char v28; // [sp+A9h] [bp-227h]  
  char v29; // [sp+AAh] [bp-226h]  
  char v30; // [sp+ABh] [bp-225h]  
  char v31; // [sp+ACh] [bp-224h]  
  char v32; // [sp+ADh] [bp-223h]  
  char v33; // [sp+AEh] [bp-222h]  
  char v34; // [sp+AFh] [bp-221h]  
  char v35; // [sp+B0h] [bp-220h]  
  char proctaskString; // [sp+B4h] [bp-21Ch]  
  char v37; // [sp+1B4h] [bp-11Ch]  

  pfatherPid_1 = *pfatherPid;  
  free(pfatherPid);  
  memset(&pthread_exit, 0, 0x10u);  
  pthread_exit = pthread_exit_0;  
  sigaction(10, (const struct sigaction *)&pthread_exit, 0);  
  memset(&v19, 0, 0x12u);  
  v22 = 30;  
  v23 = 28;  
  v24 = 1;  
  v25 = 13;  
  v27 = 75;  
  v28 = 2;  
  v29 = 10;  
  v31 = 26;  
  v33 = 29;  
  v20 = -88;  
  v34 = 5;  
  v21 = 65;  
  v26 = 65;  
  v30 = 65;  
  v32 = 15;  
  v35 = 65;  
  ((void (__fastcall *)(char *))DecodeString9)(&v19);// /proc/%ld/task/  
  sprintf(&proctaskString, &v19, pfatherPid_1);  
  while ( 1 )  
  {  
    do  
      dirp = opendir(&proctaskString);  
    while ( !dirp );  
    while ( 1 )  
    {  
      v1 = readdir(dirp);  
      if ( !v1 )  
        break;  
      dot = 0;  
      *(_WORD *)((char *)&dot + 1) = -25275;  
      ((void (__fastcall *)(int *, signed int, signed int))DecodeString9)(&dot, 1, 246);// .  
      dotdot = 0;  
      *(_WORD *)((char *)&dotdot + 1) = -21672;  
      v2 = &v1->d_name[8];  
      v9 = 0;  
      HIBYTE(dotdot) = -85;  
      ((void (__fastcall *)(int *, signed int, signed int))DecodeString9)(&dotdot, 2, 221);// ..  
      if ( strcmp(v2, (const char *)&dot) )  
      {  
        if ( strcmp(v2, (const char *)&dotdot) )  
        {  
          memset(&v37, 0, 0x100u);  
          memset(&v10, 0, 7u);  
          v11 = 23;  
          v12 = -127;  
          v14 = -127;  
          v13 = -41;  
          v15 = -41;  
          ((void (__fastcall *)(char *, signed int, signed int))DecodeString9)(&v10, 4, 179);// %s%s  
          sprintf(&v37, &v10, &proctaskString, v2);// /proc/15557/task/15557  
          if ( lstat(&v37, (struct stat *)&v17) != -1 && (v18 & 0xF000) == 0x4000 )  
          {  
            v3 = atoi(v2);  
            inotify_add_watchByPid(v3);  
            threadID = atoi(v2);  
            inotify_add_watchByTid(pfatherPid_1, threadID);  
          }  
        }  
      }  
    }  
    closedir(dirp);  
    sleep(2u);  
  }  
}
```

1、调用`DecodeString9`解密字符串“/proc/%ld/task/”；
2、格式化字符串“/proc/pid/task/”；
3、调用`opendir` 打开“/proc/pid/task/”目录；
4、调用`readdir`读取“/proc/pid/task/”目录；
5、如果返回空，则到步骤11；
6、返回不是空，过滤字符串“ .”与“ ..”；
7、调用`DecodeString9`解密字符串“/proc/pid/task/tid”；
8、调用`inotify_add_watchByPid`将tid下的mem与pagemap文件纳入监控中；
9、调用 `inotify_add_watchByTid(pfatherPid_1, threadID)`; 将“/proc/pid/task/tid”中的mem与pagemap纳入到监控中；
10、重复步骤4-步骤9；
11、调用`closedir`关闭目录；
12、线程睡眠2秒；
13、重复步骤1-12。

```text
思考：从上面可以看到线程会持续的对应用的所有线程下的mem与pagemap文件进行监控，是否可以在步骤13直接线程结束？
```

这样是不行的，如果此时结束，对于后面新创建的线程则不能纳入到本进程中。对于已经被watch的文件再次watch将返回上次的wd，引用次数会加1。

这里面可能有个小问题是：如果线程被删除了则对应的红黑树链表的节点不会被删除，造成内存泄漏。极端情况应用一致持续不断的创建线程然后线程2秒后销毁，运行一段时间后内存会崩溃。

下面看一下`inotify_add_watchByTid`函数。

### 3.5.1 inotify_add_watchByTid函数
