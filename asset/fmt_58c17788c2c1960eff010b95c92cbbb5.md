
# 推箱子 202（275）

> 原文：[`www.pediy.com/kssd/pediy03/forum1084.htm`](https://www.pediy.com/kssd/pediy03/forum1084.htm)

《推箱子 202（275）破解》 ====>我们可以继续玩了，哈！

破解人：`yuppc`

破解时间：`2001.6.22`

感谢：`1212`、`THK`

破解目的：为加入 `CCG` 而奋斗！！

1. **初步分析**

多次实验确实发现只要改动程序任何一点,启动时程序会自动关机,用 `execope` 检测其程序的内部结构,发现它是用 `Delphi` 编写,这下好办了.

2. **结论**

一定是函数的调用,并最后确定它调用了 `"ExitWindowsEx"` 函数.（幸好不是逻辑炸弹）^>^

3. **使用反汇编工具**

用 `w32asm` 黄金版反汇编其主程序 `Cargador.exe`,

4. **查找字符串**

到文件尾部,查找字符 `"exitwindow"`

从文件尾开始查找：（个人习惯:-)）

\* Referenced by a CALL at Addresses:

|:00404817 , :00404820 , :00404D63 , :00404D6C =====>四个调用处，记住

|

\* Reference To: `USER32.ExitWindowsEx`, `Ord:0000h`

|

`:00480EA2` `FF25BCCA4A00` `Jmp` `dword` `ptr` `[004ACABC]`====>关机函数 `ExitwindowsEx` 的调用点

5. **查找调用点**

好了开始查找调用 `windows` 关机函数(按上面调用地址,共有四处):

`:0040480E` `3A4DBF` `cmp` `cl`, `byte` `ptr` `[ebp-41]` ====>检测文件是否被改动

`:00404811` `7417` `je` `0040482A` =======>关键跳(否,则调动下面函数)

`:00404813` `6A00` `push` `00000000`

`:00404815` `6A02` `push` `00000002`

\* Reference To: `USER32.ExitWindowsEx`, `Ord:0000h`======>关闭 `windows` 函数调用(1)

|

`:00404817` `E886C60700` `Call` `00480EA2`

`:0040481C` `6A00` `push` `00000000`

`:0040481E` `6A00` `push` `00000000`

\* Reference To: `USER32.ExitWindowsEx`, `Ord:0000h`=======>关闭 `windows` 函数调用(2)

|

`:00404820` `E87DC60700` `Call` `00480EA2`

`:00404825` `E984010000` `jmp` `004049AE`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00404811(C)

|

`:0040482A` `A17CCD4900` `mov` `eax`, `dword` `ptr` `[0049CD7C]` =====>破解正确入口

6. **查找其他调用点**

查找第三、第四调用地址：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00404D42(C)

|

`:00404D4A` `803DCA51490000` `cmp` `byte` `ptr` `[004951CA]`, `00`

`:00404D51` `741E` `je` `00404D71` =======>关键跳

`:00404D53` `813DCC514900B0040000` `cmp` `dword` `ptr` `[004951CC]`, `000004B0`

`:00404D5D` `7E12` `jle` `00404D71` =======>关键跳

`:00404D5F` `6A00` `push` `00000000`

`:00404D61` `6A02` `push` `00000002`

\* Reference To: `USER32.ExitWindowsEx`, `Ord:0000h` ======>关闭 `windows` 函数调用(3)

|

`:00404D63` `E83AC10700` `Call` `00480EA2`

`:00404D68` `6A00` `push` `00000000`

`:00404D6A` `6A00` `push` `00000000`

\* Reference To: `USER32.ExitWindowsEx`, `Ord:0000h` =======>关闭 `windows` 函数调用(4)

|

`:00404D6C` `E831C10700` `Call` `00480EA2`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00404D51(C), :00404D5D(C)

|

`:00404D71` `8D55AC` `lea` `edx`, `dword` `ptr` `[ebp-54]` =======>破解正确入口

7. **查找注册源码段**

第二大步:查找程序启动检测注册源码段:

|:00402035(C)

|

`:00401FF5` `8B45FC` `mov` `eax`, `dword` `ptr` `[ebp-04]` -----------------------------^-----------------------

`:00401FF8` `8A8405C5FEFFFF` `mov` `al`, `byte` `ptr` `[ebp+eax-0000013B]` |

`:00401FFF` `8B55FC` `mov` `edx`, `dword` `ptr` `[ebp-04]` |

`:00402002` `3A84152AFFFFFF` `cmp` `al`, `byte` `ptr` `[ebp+edx-000000D6]` ====>注册码单码比较 |

`:00402009` `7423` `je` `0040202E` =======>关键跳(对,则跳) |

`:0040200B` `C60564D1490000` `mov` `byte` `ptr` `[0049D164]`, `00` |

`:00402012` `C605CED2490000` `mov` `byte` `ptr` `[0049D2CE]`, `00` |

`:00402019` `A160D14900` `mov` `eax`, `dword` `ptr` `[0049D160]` |

`:0040201E` `8B80E8020000` `mov` `eax`, `dword` `ptr` `[eax+000002E8]` |

`:00402024` `33D2` `xor` `edx`, `edx` |

`:00402026` `89500C` `mov` `dword` `ptr` `[eax+0C]`, `edx` |

`:00402029` `E91D030000` `jmp` `0040234B` |

循环检测段

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00402009(C)

|

`:0040202E` `FF45FC` `inc` `[ebp-04]` ====>循环增位 |

`:00402031` `837DFC07` `cmp` `dword` `ptr` `[ebp-04]`, `00000007`====>循环条件判断 |

`:00402035` `75BE` `jne` `00401FF5` ======>循环跳(主要是循环比较注册码用,当`[ebp-4]`的值等于 `7` 时便继续运行)

8. **实施修改**

分析完成,开始动手改动:

打开 `Hex` `shop` 找到 `"关键跳"`,将它们全部改成 `Jmp`(明白了吗) :-)

9. **测试结果**

测试结果:一个 `bug`--->最高分记录停止 ====>那位高手继续

其他一切正常！！！！ `4dd466d0e1e0cc123afbe504da2462ca`
