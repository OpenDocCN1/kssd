

# [原创] Hook API Library 0.2（Ring0）& LDE32 引擎（Ring0）For Delphi

写 Ring0 inline hook 的时候，正好需要一个 Opcode Length Engine。

不想用 Ms-Rem 大牛的 OLD，因为总是一大堆错误，许多东西都不能判断。无奈之下，只好自己找一个成熟的引擎。

关于反汇编引擎的设计，以后将会专门开篇描述，这里就单纯说说 Hook API Library。

感谢 z0mbie（前 29A）牛！

关于 Hook API，相比也有许多了。这里完全用 Delphi 写了一个通用 Hook 库。

与其他的不一样的地方是：

## 说明

### 1. 利用堆栈跳转

没有使用传统的 `jmp xxxx` 长跳转，使用容易理解的 `push xxxx + ret`。

仔细看代码容易理解，封装完好。

### 2. 内存补丁结构

- 补丁 1：`|push xxx--钩子处理过程|ret|`
- 补丁 2：`|保存原始补丁地址|保存原始地址代码长度|原始地址的代码|push xxxxxx|ret|`

```text
{
  Hook Api Library 0.2 [Ring3] By Anskya

  Email:Anskya@Gmail.com

  ring3 inline hook For Api

  Thank:

  前29A高手也一直都是我的偶像...z0mbie大牛...这里膜拜一下

  使用的LDE32引擎是翻译他老人家的...C->Delphi...

  说明:

  1.利用堆栈跳转

  没有使用传统的jmp xxxx 长跳转,使用容易理解的push xxxx+ret

  仔细看代码容易理解...封装完好.

  2.内存补丁结构:

  补丁1:|push xxx--钩子处理过程|ret|

  补丁2:|保存原始补丁地址|保存原始地址代码长度|原始地址的代码|push xxxxxx|ret|

  更新说明:

  0.2:

    支持Ring0 Inline Hook

  0.1:

    Ring3 Inline Hook
}
```

## 关于反汇编引擎

由于 LDE32 没什么明显的 BUG，虽然支持的指令集不是很多，但是基本上足够用了。

想必 SDK 和 DDK 中使用 MMX、SSE 等指令估计很少见吧。

所以 LDE32 就派上用场了。

```text
{
  LDE32.pas: Z0MBiE DISASM ENGINE[LDE32]

  LDE32 Delphi Coded By Anskya

  Email:Anskya@Gmail.com

  更新说明:

  procedure GetInstLenght(myiptr0: Pointer; osizeptr: PULONG);

  function GetProcLength(myiptr0: Pointer): ULONG;

  0.2:

  增加 GetProcLength函数.用于获取一个函数过程的长度.

  0.1:

  翻译

  Thank: z0mbie
}
```

## 代码见附件

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [Hook Api Library For Delphi.rar](javascript:void(0);)
  - （5.94kb，849 次下载）

