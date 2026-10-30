
```shell
pwndbg>
3481 malloc\_printerr (check\_action, "malloc(): memory corruption",
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
────────────────────────────────────────────────────────────────[ REGISTERS ]─────────────────────────────────────────────────────────────────
RAX 0x0
RBX 0x7ffff7dd1b20 (main\_arena) ◂— 0x100000001
RCX 0x6
RDX 0x40
RDI 0x7ffff7dd1bc8 (main\_arena+168) —▸ 0x7ffff7dd1bb8 (main\_arena+152) —▸ 0x7ffff7dd1ba8 (main\_arena+136) —▸ 0x7ffff7dd1b98 (main\_arena+120) —▸ 0x7ffff7dd1b88 (main\_arena+104) ◂— ...
RSI 0x0
R8 0x7ffff7dd1bc8 (main\_arena+168) —▸ 0x7ffff7dd1bb8 (main\_arena+152) —▸ 0x7ffff7dd1ba8 (main\_arena+136) —▸ 0x7ffff7dd1b98 (main\_arena+120) —▸ 0x7ffff7dd1b88 (main\_arena+104) ◂— ...
R9 0x0
R10 0x46c
R11 0x7ffff7b5afa0 (\_\_memcpy\_avx\_unaligned) ◂— mov rax, rdi
R12 0x270f
R13 0x7ffff7dd1b78 (main\_arena+88) —▸ 0x624010 ◂— 0x0
R14 0x7ffff7dd2510 ◂— 0x0
R15 0x0
RBP 0x20
RSP 0x7fffffffdb60 ◂— 0x7fff00000002
\*RIP 0x7ffff7a8ef60 (\_int\_malloc+976) ◂— mov r10d, dword ptr [rip + 0x3421e9]
──────────────────────────────────────────────────────────────────[ DISASM ]──────────────────────────────────────────────────────────────────
► 0x7ffff7a8ef60 <\_int\_malloc+976> mov r10d, dword ptr [rip + 0x3421e9] <0x7ffff7dd1150>
0x7ffff7a8ef67 <\_int\_malloc+983> or dword ptr [rbx + 4], 4
0x7ffff7a8ef6b <\_int\_malloc+987> mov eax, r10d
0x7ffff7a8ef6e <\_int\_malloc+990> and eax, 5
0x7ffff7a8ef71 <\_int\_malloc+993> cmp eax, 5
0x7ffff7a8ef74 <\_int\_malloc+996> je \_int\_malloc+2173 <\_int\_malloc+2173>
0x7ffff7a8ef7a <\_int\_malloc+1002> test r10b, 1
0x7ffff7a8ef7e <\_int\_malloc+1006> jne \_int\_malloc+1312 <\_int\_malloc+1312>
↓
0x7ffff7a8f0b0 <\_int\_malloc+1312> mov rax, qword ptr [rsp + 0x10]
0x7ffff7a8f0b5 <\_int\_malloc+1317> lea rdi, [r14 + 0x10]
0x7ffff7a8f0b9 <\_int\_malloc+1321> xor ecx, ecx
──────────────────────────────────────────────────────────────[ SOURCE (CODE) ]───────────────────────────────────────────────────────────────
In file: /home/lzx/pwn/heap/IO\_FILE/glibc-2.23/malloc/malloc.c
3476 while ((victim = unsorted\_chunks (av)->bk) != unsorted\_chunks (av))
3477 {
3478 bck = victim->bk;
3479 if (\_\_builtin\_expect (victim->size <= 2 \* SIZE\_SZ, 0)
3480 || \_\_builtin\_expect (victim->size > av->system\_mem, 0))
► 3481 malloc\_printerr (check\_action, "malloc(): memory corruption",
3482 chunk2mem (victim), av);
3483 size = chunksize (victim);
3484
3485 /\*
3486 If a small request, try to use last remainder if it is the
──────────────────────────────────────────────────────────────────[ STACK ]───────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffdb60 ◂— 0x7fff00000002
01:0008│ 0x7fffffffdb68 ◂— 0xa /\* '\n' \*/
02:0010│ 0x7fffffffdb70 —▸ 0x7fffffffdbe0 ◂— 0x0
03:0018│ 0x7fffffffdb78 —▸ 0x7ffff7b5048b (\_dl\_addr+443) ◂— add rsp, 0x28
04:0020│ 0x7fffffffdb80 ◂— 0x0
05:0028│ 0x7fffffffdb88 —▸ 0x7fffffffdbe8 —▸ 0x7ffff7fd9000 —▸ 0x7ffff7a0d000 ◂— jg 0x7ffff7a0d047
06:0030│ 0x7fffffffdb90 ◂— 0xffff800000002421 /\* '!$' \*/
07:0038│ 0x7fffffffdb98 —▸ 0x7fffffffdbdf ◂— 0x0
────────────────────────────────────────────────────────────────[ BACKTRACE ]─────────────────────────────────────────────────────────────────
► f 0 7ffff7a8ef60 \_int\_malloc+976
f 1 7ffff7a911d4 malloc+84
f 2 4006cc main+246
f 3 7ffff7a2d840 \_\_libc\_start\_main+240
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> x/4gx (char\*)&main\_arena+88 # 打印main\_arena+88的情况，查看victim
0x7ffff7dd1b78 <main\_arena+88>: 0x0000000000624010 0x0000000000000000
0x7ffff7dd1b88 <main\_arena+104>: 0x0000000000602400 0x00007ffff7dd2510 <-- 0x00007ffff7dd2510就是victim
# unsortedbin attack的时候将\_IO\_list\_all-0x10给了unsortedbin->bk，所以victim就是\_IO\_list\_all-0x10
pwndbg> x/4gx (char\*)&\_IO\_list\_all-0x10
0x7ffff7dd2510: 0x0000000000000000 0x0000000000000000 # victim的size为0，触发异常
0x7ffff7dd2520 <\_IO\_list\_all>: 0x00007ffff7dd1b78 0x0000000000000000
```

`victim`的**`size`为0，不满足要求，触发异常**，调用`malloc_printerr (check_action, "malloc(): memory corruption", chunk2mem (victim), av);`， 从而调用`_IO_flush_all_lockp`，进而fsop攻击成功。

## 3.2 ctf实例 - house of orange

```shell
lzx@ubuntu16x64:~/pwn/heap/IO\_FILE/house\_of\_orange$ checksec hitcon\_houseoforange
[\*] '/home/lzx/pwn/heap/IO\_FILE/house\_of\_orange/hitcon\_houseoforange'
Arch: amd64-64-little
RELRO: Full RELRO
Stack: Canary found
NX: NX enabled
PIE: PIE enabled
FORTIFY: Enabled
lzx@ubuntu16x64:~/pwn/heap/IO\_FILE/house\_of\_orange$ ./hitcon\_houseoforange
+++++++++++++++++++++++++++++++++++++
@ House of Orange @
+++++++++++++++++++++++++++++++++++++
1. Build the house
2. See the house
3. Upgrade the house
4. Give up
+++++++++++++++++++++++++++++++++++++
Your choice :
```

### ida分析

- `main`

```c
void \_\_fastcall \_\_noreturn main(\_\_int64 a1, char \*\*a2, char \*\*a3)
{
signed int v3; // eax
sub\_1218();
while ( 1 )
{
while ( 1 )
{
menu();
v3 = input\_number();
if ( v3 != 2 )
break;
see\_house(); // 2 - see the house
}
if ( v3 > 2 )
{
if ( v3 == 3 )
{
upgrade\_house(); // 3 - upgrade the house
}
else
{
if ( v3 == 4 )
{
puts("give up"); // 4 - exit
exit(0);
}
LABEL\_14:
puts("Invalid choice");
}
}
else
{
if ( v3 != 1 )
goto LABEL\_14;
build\_house(); // 1 - build the house
}
}
}
```

- `input_number`

```c
\_\_int64 input\_number()
{
char nptr; // [rsp+10h] [rbp-20h]
unsigned \_\_int64 v2; // [rsp+28h] [rbp-8h]
v2 = \_\_readfsqword(0x28u);
\_read\_chk(0LL, &nptr, 15LL, 16LL);
return (unsigned int)atoi(&nptr); // 返回一个unsigned int
}
```

- `build_house`：build一个`house`，会创建三个chunk

- 最多能build4个`house`
- 每个`house`包含一个`orange`和一个`name`
- 每个`orange`包含`price`和`color`
- 创建chunk的顺序：`house`、`house_name`、`orange`

![](upload/attach/202210/830671\_G9PXY424AJPX6TK.png)

```c
int sub\_D37()
{
unsigned int house\_name\_len; // [rsp+8h] [rbp-18h]
signed int color\_of\_orange; // [rsp+Ch] [rbp-14h]
void \*house; // [rsp+10h] [rbp-10h]
\_DWORD \*orange; // [rsp+18h] [rbp-8h]
if ( house\_cnt > 3u ) // 只能 build 4次
{
puts("Too many house");
exit(1);
}
house = malloc(0x10uLL); // 0x20大小的house chunk
printf("Length of name :");
house\_name\_len = input\_number();
if ( house\_name\_len > 0x1000 )
house\_name\_len = 4096;
\*((\_QWORD \*)house + 1) = malloc(house\_name\_len);// house[1]存储house\_name\_chunk
if ( !\*((\_QWORD \*)house + 1) )
{
puts("Malloc error !!!");
exit(1);
}
printf("Name :");
input\_string(\*((void \*\*)house + 1), house\_name\_len);// 往house\_name\_chunk输入house name
orange = calloc(1uLL, 8uLL);
printf("Price of Orange:", 8LL);
\*orange = input\_number(); // orange[0]存储price
color\_menu();
printf("Color of Orange:");
color\_of\_orange = input\_number();
if ( color\_of\_orange != 56746 && (color\_of\_orange <= 0 || color\_of\_orange > 7) )
{
puts("No such color");
exit(1);
}
if ( color\_of\_orange == 56746 ) // orange[1]存储56746/color+30
orange[1] = 56746;
else
orange[1] = color\_of\_orange + 30;
\*(\_QWORD \*)house = orange; // house[0]存储orange\_chunk
global\_house = house; // 全局变量global\_house存储的是最新的house chunk
++house\_cnt; // 计数加1
return puts("Finish");
}
```

![](upload/attach/202210/830671\_7AR52V4N7BQ8J7D.png)

- `input_string`

```c
ssize\_t \_\_fastcall input\_string(void \*a1, unsigned int a2)
{
ssize\_t result; // rax
result = read(0, a1, a2); // 没有NULL结尾处理，可能存在信息泄露漏洞！！！
if ( (signed int)result <= 0 )
{
puts("read error");
exit(1);
}
return result;
}
```

- `see_house`：打印`house`的信息

- 打印`house` `name`
- 打印`orange` `price`
- 打印橘子

![](upload/attach/202210/830671\_5964V4C9NZQ5EKD.png)
