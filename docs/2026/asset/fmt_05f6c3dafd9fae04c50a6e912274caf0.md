
还是前面的`sub_12274`，来看处理完以后的效果：

![decryptedstr](https://bbs.kanxue.com/upload/attach/202202/734571_VEQ3TMC5DAABK4B.png)

### 第三类混淆：libc函数动态获取

`sub_B3B4`动态获取`libc`中的函数并保存，我们这里做一下符号重命名即可。

首先把`F5`中的`sub_B3B4`保存下来，然后写个脚本把`qword_BE238`命名为`_exit_libc`，`qword_BE240`命名为`exit_libc`，`qword_BE248`命名为`pthread_create_libc`，依次类推即可。

```
from idautils import *
from idaapi import *
from idc import *

flag = 0
for line in open("D:\\libc.cpp"):
    if flag == 0:
        strs = line.split("\"")
        funcname = strs[-2]
        funcname += "_libc"
        flag = 1
        continue
    if flag == 1:
        addrname = line[8:13]
        addrname = int(addrname, 16)
        flag = 0
        set_name(addrname, funcname, SN_CHECK)
```

结果：

![decryptedstr](https://bbs.kanxue.com/upload/attach/202202/734571_AC5F29F99AZN3CG.png)

用这些函数的地方也看的比较清楚。

![decryptedstr](https://bbs.kanxue.com/upload/attach/202202/734571_Y45TBW3F7Y9YVMG.png)

### 总结

百度加固免费版`libbaiduprotect.so`里面基本上就是这些花样，还是比较好搞定的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

```
from idautils import *
from idaapi import *
from idc import *
import string
rodata_start = 0x959B0
rodata_end = 0x9AF68
def get_string(addr):
  out = ""
  while True:
    if get_byte(addr) != 0:
      out += chr(get_byte(addr))
    else:
      break
    addr += 1
  return out

addr = rodata_start
index = 0

decrypted_str = [
  "n001",
  "(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;IZ)V",
  ......
]

while addr < rodata_end:
  str = get_string(addr)
  str_len = len(str) + 1
  if all(c in string.hexdigits for c in str) and str_len > 8:
    i = 0
    tmp_addr = addr
    decrypted_strlen = len(decrypted_str[index])
    while i < str_len:
      if i >= decrypted_strlen:
        patch_byte(tmp_addr, 0x00)
      else:
        patch_byte(tmp_addr, ord(decrypted_str[index][i]))
      i = i + 1
      tmp_addr = tmp_addr + 1
    index = index + 1

  addr += str_len
```
