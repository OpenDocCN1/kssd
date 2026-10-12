
### 这种方法去掉 npggNT.des 的监视是完全能够实现的
但是`IsMyThread(dwThreadId)`函数非常关键，要考虑周全，不然断错线程的话，就“自杀”了。

#### 6，更简单的陷阱
原理跟上面一样，但是我们将替换函数写成这个样子：

```
HANDLE WINAPI MyGetCurrentProcess(VOID) //替换掉原来的 GetCurrentProcess
{
  HMODLE hMod=GetModelHandle("npggNT.des");
  if(hMod!=NULL){
     FreeLibrary(hMod);      //直接 Free 掉它
  }
  UnhookGetCurrentProcess(); //是我们要保护的线程调用就恢复函数头
  HANDLE hProcess=GetCurrentProcess(); //让它调用
  RehookGetCurrentProcess();//重新挂钩
  return hProcess;    //返回调用结果
}
```

这种方法就万无一失了，不用担心会“自杀”。

## 三、总结
由上面可以看到在用户层上反 NP 监视是不是很简单的事？最简单有效的就是第六种方法，短短的几行代码就可以搞定了。但是不要指望去掉了`npggNT.des`就可以为所欲为了，还有 NP 还在驱动层做了很多手脚，比如`WriteProcessMemory`在用户层用没问题，但是过不了 NP 的驱动检查，对游戏完全没效果。要在 NP 下读写游戏内存，说起来又另一篇文章了《如何在 NP 下读写游戏内存》，请继续关注。

***

### Bypass NP in ring0 (2007 年 3 月 16 日)
1. `Add MyService`
2. `hook sysenter`
3. `SystemServiceID -> MyServiceID`
4. `MyService JMP -> SystemService Function + N bytes` (参考[【原创】SSDT Hook 的妙用－对抗 ring0 inline hook](http://bbs.pediy.com/showthread.php?s=&threadid=40832))

*   1、2、3 -> 绕过 NP SSDT 检测
*   4 -> 绕过 NP 内核函数头检测

`NP968` 下通过
`4dd466d0e1e0cc123afbe504da2462ca`
