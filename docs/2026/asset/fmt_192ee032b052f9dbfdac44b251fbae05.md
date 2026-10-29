
这里可以用 `frida` `hook` `sub_1440` 和 `sub_8AC`, 进一步验证：

输入: `1111`

`frida` hook 关键日志：

> 1> `sub_1440(&unk_3008, v7);` //关键函数

> 核心日志：

> `sub_1440` `onEnter`==========

> `sub_1440` `arg0`:: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF

> 7edacf6008 72 63 34 74 65 73 74 00 00 00 00 00 00 00 00 00 rc4test.........

> `sub_1440` `arg1`:: 1111

> `sub_1440` `onLeave`==========

> `sub_1440` `ret`:: 0x7feced9600

> `sub_1440` `this.arg1`:: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF

> 7ee6fff600 a6 dc 78 58

> 2> `sub_8AC(v7, v8);`

> 核心日志：

> `sub_8AC` `arg0`:: 0 1 2 3 4 5 6 7 8 9 A B C D E F 0123456789ABCDEF

> 7ee6fff450 a6 dc 78 58

> `sub_8AC` `arg1`:: 0x4

> `sub_8AC` `onLeave`==========

> `sub_8AC` `ret`:: ptx4WA==

### 验证是否是标准的 base64 算法

`FromHex`: a6dc7858

`ToBase64`: ptx4WA==

### `stringFromJNI` 核心算法

`sub_1440` -> `sub_12C4`

发现 `sub_12C4` 应该是一个 `RC4` 算法，且 `key` 为： rc4test

在线 `cyberchef` 网站验证：

测试样例：`1111` ---> a6dc7858

![](https://bbs.kanxue.com/upload/attach/202104/814746_67AS5NCUG8JCKV3.png)

### `stringFromJNI` 算法

`rc4(key:rc4test)` ---> `base64`

那么得到真实的 flag:

![](https://bbs.kanxue.com/upload/attach/202104/814746_4YMDYFDE76WAQVD.png)

获取的真实的 `FLAG`:

`flag{kanxue6666}`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

### 上传的附件

- [1、请找出flag.apk](javascript:void(0);)
  （1.62MB，50次下载）

```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `__int64 __fastcall Java_com_kanxue_test_MainActivity_stringFromJNI(__int64 a1, __int64 a2, __int64 a3)`  `{`  `__int64 v3;` `/``/` `ST48_8`  `__int64 v4;` `/``/` `x0`  `__int64 v5;` `/``/` `ST28_8`  `__int64 v6;` `/``/` `x0`  `__int64 v7;` `/``/` `ST20_8`  `__int64 v8;` `/``/` `ST08_8`  `__int64 v9;` `/``/` `ST18_8`    `v3` `=` `a1;`  `v4` `=` `sub_15EC(a1, a3,` `0LL``);`  `v5` `=` `v4;`  `v6` `=` `strlen(v4);`  `v7` `=` `malloc(v6);`  `strcpy(v7, v5);`  `sub_1440(&unk_3008, v7);` `/``/``关键函数`  `v8` `=` `strlen(v5);`  `v9` `=` `sub_8AC(v7, v8);` `/``/``标准的base64算法`  `sub_1440(&unk_3008, v7);`  `return` `sub_163C(v3, v9);`  `}` |
```
