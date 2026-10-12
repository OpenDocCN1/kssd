
# DriverStudio Remote Control 远程调用认证绕过【转帖】

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-384.htm`](https://www.pediy.com/kssd/pediy07/pediy7-384.htm)

## 1. 基本信息

DriverStudio Remote Control 远程调用认证绕过漏洞及其利用研究

*   **创建时间**：2005-09-14
*   **文章属性**：原创
*   **文章提交**：`cocoruder` (`frankruder_at_hotmail.com`)
*   **作者**：`cocoruder`
*   **页面**：`http://ruder.cdut.net`
*   **邮箱**：`cocoruder@163.com` && `frankruder@hotmail.com`
*   **漏洞名称**：DriverStudio Remote Control 远程调用认证绕过漏洞
*   **发现者**：`cocoruder`
*   **类型**：设计错误
*   **更新时间**：14/9/2005

## 2. 受威胁的系统

*   `>=NuMega.DriverStudio.v2.7`

已在如下版本中测试成功：

*   `NuMega.DriverStudio.v2.7` (windows 2000 sp4 | windows xp sp1)
*   `NuMega.DriverStudio.v3.0beta2` (windows 2000 sp4 | windows xp sp1)

未受威胁的系统：

*   目前未知

厂商：`http://www.compuware.com`

## 3. 漏洞描述

SoftICE Driver Suite 是一款强大的 windows 内核级调试软件，有着广泛应用。其默认安装的 "DriverStudio Remote Control" 服务允许使用客户机的用户/密码认证通过的客户修改本地 `softice` 配置，但这个认证可被绕过，使得任意远程攻击者可修改本地 `softice` 配置。成功利用此漏洞可获取系统权限。

### 3.1 详细描述

DriverStudio 在默认安装的时候在系统中创建一个服务名为 "DriverStudio Remote Control" 的服务，对应进程名为 `DSRSvc.exe`，根据我这几天的分析，`DSRSvc.exe` 主要实现了下面几个功能：

1.  实现远程更改 `softice` 启动方式的 rpc 调用接口
2.  实现远程更改 `softice` 配置文件的 rpc 调用接口
3.  实现远程获取本地计算机及 DriverStudio 信息接口和远程重启计算机接口 (这个是通过 UDP 数据包实现的)

如果远程计算机使用和本地一样的用户名/密码，在 `bin` 目录下找到 `dsconfig.exe`，在命令行下输入 `dsconfig -t ip` 即可象修改本地配置一样修改远程配置（保存在 `drivers/winice.dat` 文件中）。其间采用 `DCE RPC&NTLM` 协议通信/认证，分析显示它是把当前用户名/密码发送到远程进行认证的，如果我们自己发送数据包，在 NTLM 认证的时候使用空认证 (`NULL Session Login`)，同样可以认证通过并修改远程 `softice` 配置。而禁止系统的空连接仍可认证通过，可见是 DriverStudio 导致的问题。

另外，向目标机 `UDP 9110` 端口发送如下缓冲区即可导致系统重新启动，这里没有进行任何身份验证。事实上，这是另一个独立的漏洞，偶测试在 `xp` 下会弹出重启倒计时的进度条并可 `cancel`，在 `2000` 下会直接重启。

```
02 00 00 00 41 41 41 00        <-00000002 为重启命令码
```

综合利用以上弱点可获取系统权限。

### 3.2 详细利用方法

由于可以修改 `softice` 的配置，依赖 `softice` 强大的命令可以完成攻击。"INIT" 配置项可配置在 `softice` 启动时运行的命令，结合我们可修改 `softice` 的启动方式，如果我们把 `softice` 设置成 `Automatic`，`softice` 就会随 `windows` 一起启动，启动的时候就会运行我们设置的 `softice` 命令，如果我们在命令上做些手脚，嘿嘿...事实上，我们拥有了对所有进程的所有内存空间的任意修改权限。但要安全而又通用地获取权限还是很有难度的，我本来想通过修改系统的某些特殊 api 的返回值来达到操作系统对我们 "完全开放" 的目的（比如任意 `ntlm` 验证都可成功什么的），但对 `windows` 核心实在不懂（如果有大牛知道了请告诉我:)），还好，偶想到了个比较好的办法，并在 `windows 2000 server+DriverStdio2.7` 上测试成功。

这里又用到了 `Dsrsvc.exe`，它还监听 `UDP 9110` 端口，负责处理 UDP 的命令码（比如重新启动计算机）。任何发送到这个端口的数据包都是由 `DSStatusServer` 模块的如下代码处理的

```
.text:10003F59         push    esi
.text:10003F5A         push    edx
.text:10003F5B         call    ??2@YAPAXI@Z    ; operator new(uint)
.text:10003F60         mov     ecx, [esp+138h+len]
.text:10003F64         add     esp, 4
.text:10003F67         mov     esi, eax
.text:10003F69         lea     eax, [esp+134h+fromlen]
.text:10003F6D         push    eax             ; fromlen
.text:10003F6E         push    ebp             ; from
.text:10003F6F         push    0               ; flags
.text:10003F71         push    ecx             ; len
.text:10003F72         push    esi             ; buf
.text:10003F73         push    edi             ; s
.text:10003F74         call    ds:recvfrom     ;recvfrom
.text:10003F7A         test    eax, eax        ;<------------modifit!
.text:10003F7C         jle     short loc_10003FA6
.text:10003F7E         push    esi
.text:10003F7F         call    sub_10003910
```

修改 `10003f7a` 这个地址的代码，改成 `jmp esi` (注意，偶测试过程中发现 `3.0beta2` 版本是 `edi` 指向 `buf`，因此要改为 `jmp edi`)，`esi` 指向 `ws2_32!recvfrom` 的 `buffer`，修改好后，我们对 `UDP 9110` 端口发送我们的 `exploit`，`Dsrsvc.exe` 接收完就会跳到我们的 `exploit` 执行

但是要注意的是，由于 `DSStatusServer` 为一动态链接库，`10003F7A` 这个地址在虚拟空间是不好定位的，还好 `softice` 强大的命令又帮了我们大忙:)，下断点

```
bpx recvfrom do "eb *esp ff,e6"        ;ff,e6 为 jmp esi 机器码
```

当运行到 `recvfrom` 时 `esp` 即为返回地址指针，这个返回地址即是上面的 `10003F7A`，为了在实际攻击中避免出现修改到别的地方去了，偶跟了下发现 `DSStatusServer` 的 `recvfrom` 函数的 `len` 参数始终为 `00000172h`，于是

```
bpx recvfrom if *(esp+c)==00000172 do "eb *esp ff,e6"
```

中断后恢复运行，

```
bpx recvfrom if *(esp+c)==00000172 do "eb *esp ff,e6;x"
```

另外由于用到 `recvfrom` 函数，需要在配置文件里添加一项，

```
EXP=c:\windows\system32\ws2_32.dll      ;这个地方偶是固定写进去的，55,EXP=\SystemRoot\System32\ws2_32.dll 居然不行(why?)
```

### 3.3 远程调用/认证分析

整个 rpc 调用过程如下（基与 `dsconfig -t ip` 的分析）

`connect 135 port(local)---->rpc bind(local)---->rpc bind_ack(remote)---->map request(local)---->map response(remote)(返回监听的端口 stice_port)`

`\---->connect stice_port(local)---->bind UUID(local)---->bind_ack(remote)---->AUTH(local)(认证，使用空认证)---->request(local)(命令请求)`

`\---->reponse(remote)(返回成功否)`

其中涉及两个 rpc 接口，分别为

*   `UUID:32d90706-b698-4029-b236-e18ebff582b1` 负责 `softice` 启动方式更改
*   `UUID:10d1800c-af75-4249-b7a2-484dec69ed3a` 负责更新 `winice.dat` 文件

### 3.4 攻击思路总结

1.  精心构造 rpc 调用实现修改远程配置文件 `winice.dat`，并把 `softice` 启动方式改为 `Automatic`
2.  发送 UDP 重启码至 target，并等待重启
3.  重启后发送 shellcode 至 `UDP 9110` 端口
    （这几天一直在想更好的利用方法，希望有大牛提出更好的利用方法）

### 3.5 测试程序

1.  `modifit_config.cpp`
2.  `send_reboot.cpp`
3.  `send_shellcode.cpp`
    （出于责任和安全，这里不公开详细的攻击代码:)）

## 4. 解决方案

厂商未提供相应补丁，建议用户禁止并停掉 "DriverStudio Remote Control" 服务。

## 5. 致谢

感谢我的同事们，和 `zwelle`。

## 6. 声明

本文涉及技术知识仅供安全研究与教学之用，使用者风险自负！

`4dd466d0e1e0cc123afbe504da2462ca`
