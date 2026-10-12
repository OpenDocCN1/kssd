
# Hook Api Library 0.2 [Ring0] & LDE32 引擎 [Ring0] For Delphi

> 原文：[`www.pediy.com/kssd/pediy10/63986.html`](https://www.pediy.com/kssd/pediy10/63986.html)

写 `Ring0` `inline hook` 的时候正好需要一个 `Opcode Length Engine`

不想用 Ms-Rem 大牛的 OLD 的～因为总是一大堆错误。。许多东西都不能判断。无奈之下。。。

自己找一个成熟的引擎吧。。。

关于反汇编引擎的设计以后将会专门开篇描述。。。这里就单纯说说 `Hook Api library` 吧。。。

感谢 `z0mbie` [前 29A] 牛！

关于 `Hook Api` 相比也有许多了。。。这里完全用 Delphi 写了一个。。。通用 `Hook` 库

与其他的不一样的地方是

说明:

1. 利用堆栈跳转
   没有使用传统的 `jmp xxxx` 长跳转，使用容易理解的 `push xxxx+ret`
   仔细看代码容易理解...封装完好.

2. 内存补丁结构:
   补丁 1: `|push xxx--钩子处理过程|ret|`
   补丁 2: `|保存原始补丁地址|保存原始地址代码长度|原始地址的代码|push xxxxxx|ret|`

代码:

```
{
  Hook Api Library 0.2 [Ring3] By Anskya
  Email:Anskya@Gmail.com
  ring3 inline hook For Api

Thank:
  前 29A 高手也一直都是我的偶像...z0mbie 大牛...这里膜拜一下
  使用的 LDE32 引擎是翻译他老人家的...C->Delphi...


说明:
  1.利用堆栈跳转
  没有使用传统的 jmp xxxx 长跳转,使用容易理解的 push xxxx+ret
  仔细看代码容易理解...封装完好.

  2.内存补丁结构:
  补丁 1:|push xxx--钩子处理过程|ret|
  补丁 2:|保存原始补丁地址|保存原始地址代码长度|原始地址的代码|push xxxxxx|ret|

更新说明:
  0.2:
    支持 Ring0 Inline Hook
  0.1:
    Ring3 Inline Hook
}
```

关于反汇编引擎:

由于 `LDE32` 没什么明显的 BUG 虽然支持的指令集不是很多。但是基本上足够用了

想必 SDK 和 DDK 中使用 `MMX`, `SSE`...等指令估计很少见吧...

所以 `LDE32` 就派上用场了....

代码:

```
{
  LDE32.pas: Z0MBiE DISASM ENGINE[LDE32]
  LDE32 Delphi Coded By Anskya
  Email: Anskya@Gmail.com

更新说明:
  procedure GetInstLenght(myiptr0: Pointer; osizeptr: PULONG);
  function GetProcLength(myiptr0: Pointer): ULONG;

0.2:
  增加 GetProcLength 函数.用于获取一个函数过程的长度.

0.1:
  翻译

  Thank: z0mbie
}
```

代码见附件:

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || Hook Api Library For Delphi.rar

 |
