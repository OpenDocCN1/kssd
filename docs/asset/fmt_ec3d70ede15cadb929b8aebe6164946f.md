
# df3 v1.00.

> 原文：[`www.pediy.com/kssd/pediy03/forum493.htm`](https://www.pediy.com/kssd/pediy03/forum493.htm)

如何破解 `df3` `v1.00.17` `CD Protect`

\-----------------------------

雪椰

2000,12,25am

Email: `wocy@263.net`

`Http://wocy.yeah.net`(有本例中的补丁）

\-------------------------------------

头一阵玩大地勇士，虚拟了个光盘。近两天硬盘资源紧张，（`0.5G` 呢），破了她...

难度：简易

工具：`w32dasm`,`Hview`

\------------

前言:这个游戏是不放 cd 时不能玩 `Single` 和 `Novaworld`。想是对 `single&nova` 做了特殊动作。

1，`w32dasm` `find` 字符串 `"single"`，you will come to:

\* `Possible` `StringData` `Ref` from `Data` `Obj` -> `"but_mm_SinglePlayer"`

  |

:00443B71 `6850165000`  `push` `00501650`

:00443B76 `7545`  `jne` `00443BBD`

:00443B78 `E88364FFFF`  `call` `0043A000`

:00443B7D `83C404`  `add` `esp`, `00000004`

:00443B80 `3BC6`  `cmp` `eax`, `esi`

:00443B82 `7407`  `je` `00443B8B`

:00443B84 `C7405404000000`  `mov` `[eax+54]`, `00000004`  `《=====注意这点` `1`

\* Referenced by a `(U)nconditional` 或 `(C)onditional` Jump at Address:

|:00443B82`(C)`

|

\* `Possible` `StringData` `Ref` from `Data` `Obj` -> `"but_mm_NovaWorld"`

  |

:00443B8B `683C165000`  `push` `0050163C`

:00443B90 `E86B64FFFF`  `call` `0043A000`

:00443B95 `83C404`  `add` `esp`, `00000004`

:00443B98 `3BC6`  `cmp` `eax`, `esi`

:00443B9A `7407`  `je` `00443BA3`

:00443B9C `C7405404000000`  `mov` `[eax+54]`, `00000004`  `《=====注意这点` `2`

\* Referenced by a `(U)nconditional` 或 `(C)onditional` Jump at Address:

|:00443B9A`(C)`

|

\* `Possible` `StringData` `Ref` from `Data` `Obj` -> `"but_mm_Update"`

  |

:00443BA3 `682C165000`  `push` `0050162C`

:00443BA8 `E85364FFFF`  `call` `0043A000`

:00443BAD `83C404`  `add` `esp`, `00000004`

:00443BB0 `3BC6`  `cmp` `eax`, `esi`

:00443BB2 `7440`  `je` `00443BF4`

:00443BB4 `C7405404000000`  `mov` `[eax+54]`, `00000004`  `《=====注意这点` `3`

:00443BBB `EB37`  `jmp` `00443BF4`

\* Referenced by a `(U)nconditional` 或 `(C)onditional` Jump at Address:

|:00443B76`(C)`

|

:00443BBD `E83E64FFFF`  `call` `0043A000`

:00443BC2 `83C404`  `add` `esp`, `00000004`

:00443BC5 `3BC6`  `cmp` `eax`, `esi`

:00443BC7 `7403`  `je` `00443BCC`

:00443BC9 `897854`  `mov` `dword` `ptr` `[eax+54]`, `edi`  `《=====注意这点` `4`

\* Referenced by a `(U)nconditional` 或 `(C)onditional` Jump at Address:

|:00443BC7`(C)`

|

\* `Possible` `StringData` `Ref` from `Data` `Obj` -> `"but_mm_NovaWorld"`

  |

:00443BCC `683C165000`  `push` `0050163C`

:00443BD1 `E82A64FFFF`  `call` `0043A000`

:00443BD6 `83C404`  `add` `esp`, `00000004`

:00443BD9 `3BC6`  `cmp` `eax`, `esi`

:00443BDB `7403`  `je` `00443BE0`

:00443BDD `897854`  `mov` `dword` `ptr` `[eax+54]`, `edi`  `《=====注意这点` `5`

\* Referenced by a `(U)nconditional` 或 `(C)onditional` Jump at Address:

|:00443BDB`(C)`

|

\* `Possible` `StringData` `Ref` from `Data` `Obj` -> `"but_mm_Update"`

  |

:00443BE0 `682C165000`  `push` `0050162C`

:00443BE5 `E81664FFFF`  `call` `0043A000`

:00443BEA `83C404`  `add` `esp`, `00000004`

:00443BED `3BC6`  `cmp` `eax`, `esi`

:00443BEF `7403`  `je` `00443BF4`

:00443BF1 `897854`  `mov` `dword` `ptr` `[eax+54]`, `edi`  `《=====注意这点` `6`

你会发现 1,2,3 与 4,5,6 是多么的相近，把 1,2,3 改成 4,5,6 的样子，（不足部分用 `nop` 填充）

2，改吧

3，成功

4，做补丁吧

5，这个世界清净了...... `4dd466d0e1e0cc123afbe504da2462ca`
