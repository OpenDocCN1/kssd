
| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18 | `$ArrayPtr` `=` `475000`  `$ArrayEnd` `=` `475120`    `Loop:`    `cmp` `dword:[$ArrayPtr],` `0`    `/``/` `跳过模块空隙`  `je` `Next`    `EIP` `=` `dword:[$ArrayPtr]`  `RunToParty` `1`                `/``/` `执行到系统模块断下`  `dword:[$ArrayPtr]` `=` `EIP` `/``/` `取当前地址`    `Next``:`    `$ArrayPtr` `+``=` `4`              `/``/` `指向下一块地址`  `cmp` `$ArrayPtr, $ArrayEnd` `/``/` `判断是否结束`  `jne Loop`  `ret` |

我的这种写法 `x64dbg` 也是支持的，是不是十分的简洁明了，运行完脚本之后，可以看到 IAT 表已经被完美修复

![图片描述](https://bbs.kanxue.com/upload/attach/202208/937323_5WRT8ZP37M7E8PS.png)

## 收尾

![图片描述](https://bbs.kanxue.com/upload/attach/202208/937323_3YMMAM7PBY9SN73.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202208/937323_6QHB4HH52KMF9TB.png)

这时候我们再打开 `scylla` 工具，确定好 OEP，直接点击 IAT AutoSearch，然后点击 Get Imports，此时表已经能被加载出来，点击右侧的 Fix Dump，选择我们之前 dump 出来无法正常运行的文件，大功告成，新生成的 `xxx_dump_SCY` 已经能够完美运行，拖进 IDA 也可以清楚的看到调用的函数，收工！

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
