
# 黑马课表管理系统 2.6

> 原文：[`www.pediy.com/kssd/pediy04/kanxue341.htm`](https://www.pediy.com/kssd/pediy04/kanxue341.htm)

软件：黑马课表管理系统

版本：2.6

下载：`http://www.soft-sky.com/download/znpk.zip`

大小：4854k

保护：易

介绍：利用计算机进行课程表编排，功能强大。

工具：`Trw2000`

运行“黑马课表管理系统”软件，一开始便要求注册。

学校名称：`Http://esoft2001.51.net`

注 册 码：`9876543210`

`Ctrl-n`，下命令`bpx hmemcpy`，`F5`返回，按“开始注册”按钮，`Trw2000`拦下。

下`bc *`命令

再下`Pmodule`命令，程序回来在下面一行

```
:0041F526 8D4C240C             lea ecx, dword ptr [esp+0C]
```

从这里开始一直按`F10`，不久会来到下面

```
:0041F570 E8FB000000           call 0041F670

:0041F575 8B4C2414             mov ecx, dword ptr [esp+14]（D ecx 看到错误注册码）

:0041F579 8B00                 mov eax, dword ptr [eax]（D eax 看到正确注册码）

:0041F57B 51                   push ecx

:0041F57C 50                   push eax

* Reference To: MSVCRT._mbscmp, Ord:0159h

                             |

:0041F57D FF150C284500         Call dword ptr [0045280C]

:0041F583 83C410               add esp, 00000010

:0041F586 8D4C2410             lea ecx, dword ptr [esp+10]

:0041F58A 85C0                 test eax, eax

:0041F58C 0F95C3               setne bl

* Reference To: MFC42.Ordinal:0320, Ord:0320h

                             |

:0041F58F E8C48C0200           Call 00448258

:0041F594 84DB                 test bl, bl

:0041F596 7412                 je 0041F5AA

:0041F598 6A00                 push 00000000

:0041F59A 6A00                 push 00000000

* Possible StringData Ref from Data Obj ->"注册码有错！"

                             |

:0041F59C 6818304600           push 00463018

:0041F5A1 8BCE                 mov ecx, esi

* Reference To: MFC42.Ordinal:1080, Ord:1080h

                             |

:0041F5A3 E88A8E0200           Call 00448432

:0041F5A8 EB51                 jmp 0041F5FB
```

整理一下：

黑马课表管理系统 2.6

学校名称：`Http://esoft2001.51.net`

注 册 码：`4044-5612-1713-629` `4dd466d0e1e0cc123afbe504da2462ca`
