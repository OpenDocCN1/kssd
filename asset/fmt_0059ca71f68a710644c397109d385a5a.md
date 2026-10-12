
# CE 下另 Hook function 方式..

> 原文：[`www.pediy.com/kssd/pediy10/72728.html`](https://www.pediy.com/kssd/pediy10/72728.html)

A.. 不能算 CE 啦. Windows Mobile OS ARMV4I 可能比较适合

在 Windows Mobile 6.1 下 HTC Diamond 测试..

在这里作个小修正..

因为有个需求是要 dump 某个 library funtion arguments. 但没找到一个适合的方式用..

所以就自己写了..

目前 bug & 限制还很多

1. No multithread safe
2. 无法 hook xip function (因为是从 ROM 直接执行, 唯读)
3. 如果 function 开头前三个 instruction 有 "跳转" 的话，会 data abort.
4. 有时候好像 function 被 inline 的话，会无效..

先分享出来。

## 原理

替换 Function 开头，跟 [`bbs.pediy.com/showthread.php?t=16061`](http://bbs.pediy.com/showthread.php?t=16061) 一样

## 用法

代码:

```
typedef int (*pFunc1)(int);
pFunc1 g_proc;

__declspec(noinline) int Func1(int a)
{
  return 10;
}

__declspec(noinline) int my_hookFunc1(int a)
{
  if (g_proc)
  {
      // Function hooked!!
      return g_proc(a);
  }
  return 0;
}

void hook()
{
   g_proc  = (pFunc1) stools::hook::HookFunction(Func1, my_hookFunc1);
   int ret = Func1(10);
   stools::hook::UnhookFunction(g_proc);
    
    
}

```

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || StHook.zip |

