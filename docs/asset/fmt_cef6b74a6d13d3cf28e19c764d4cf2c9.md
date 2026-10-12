
# 基于启发式技术检测复杂病毒 Anunnaki

> 原文：[`www.pediy.com/kssd/pediy11/110432.html`](https://www.pediy.com/kssd/pediy11/110432.html)

## 目录
- 0x01. 简介
- 0x02. Anunnaki 变形病毒技术分析
  - 2.1. Anunnaki 特性简介
  - 2.2. Anunnaki 执行流程
  - 2.3. get kernel base address
  - 2.4. anti av-emulator and anti av-heuristic
  - 2.5. polymorphic
    - 2.5.1 Offensive polymorphic engine 功能描述
    - 2.5.2 仿真无效指令设计
    - 2.5.3 poly engine 的实现
  - 2.6. EPO technology 实现
  - 2.7. 感染后文件对比
- 0x03. 防御技术的困境
  - 3.1. 特征码的失效
  - 3.2. 关于主动防御技术
  - 3.3. 关于”云“安全技术
  - 3.4. 关于启发式检测技术
- 0x04. 寻找复杂病毒的漏洞
  - 4.1. 重定位手段
  - 4.2. 跨节区跳转
  - 4.3. 怪异的 macros
  - 4.4. EPO 跟踪
  - 4.5. polymorphic 的不足
  - 4.6. 关于检测
- 0x05. 启发式技术检测 Anunnaki
  - 5.1. 构造仿真器
  - 5.2. 执行时数据的跟踪
  - 5.3. 没有终结的对抗
- 0x06. 其他

## 0x01. 简介

启发式检测技术作为特征检测技术的辅助手段，已经用实践经验证明了是检测病毒的一个成功手段，基于启发式的扫描器最大弱点就是会导致过多的虚警，但在某些方面（如宏病毒，`ani` 格式，`swf` 格式漏洞等方面，加密，变形等），如果缺少启发式检测机制，仅依靠基于特征码技术、主防、”云“技术等 `Aver` 手段将无法完成病毒的完全性检测。

`Anunnaki` 多态病毒是由 `Dark Prophet` 编写的最新病毒，发布日期 `09-10-20`。该病毒的 `payload` 仅弹出一个 `msgbox`，提示”U was infected by almighty Anunnaki“，并没有破坏性的动作。`Anunnaki` 在感染，反虚拟机，变形等方面总结了之前的一些经验，做了一些新的尝试，虽然在 `Anti-emulation` 中并没加入 `vx` 群体最新的研究成果，但仍然有效的跳过了一些 `av` 中的 `sandbox` 检测。下面将在结合 `Anunnaki` 的分析中，完成启发式检测方案。

## 0x02. Anunnaki 变形病毒技术分析

### 2.1 Anunnaki 特性简介

- `Virus type` - 直接感染当前目录下的文件
- `Target files` - `EXE` 文件
- `Target OS` - `Win2k/2003/xp/vista/2008/win7`
- `Infection type` - 添加最后一个节
- `Infection mark` - 不会多重感染
- `Polymorphic`
  - 同样使用了由 `prophet` 自己编写的多态引擎(`Offensive polymorphic engine`)
  - 支持 32bit 滑动密钥(`xor`/`add`/`sub`)
- `EPO`
  - Patch `ExitProcess`/`exit`/`_exit`，支持 `call rel32`/`call dword ptr [mem32]`
- `Payload` - `Message box`
- `Armoured`
  - 反仿真器
  - 检测 `SW` `breakpoints`
  - 通过检测 `api` 调用处是否有 `bpx`，能绕过比较差的 `sandbox`
- `Apis resolving` - 通过 `crc` 校验值方式
- `Other features`
  - 不使用 `Delta offset` 方式重定位
  - 慢随机感染（不是每次都能触发感染替罪羊程序）
  - 最后一个节不设置可写标志 (`anti-heuristic`)
  - 随机插入数据到 `virus body`
  - 使用 `SEH`
  - 可设置垃圾字节数及多态级别，病毒大小可变
- `DEP support` - 在 `DLLCharacteristics` 为 0 时，设置 `Nx_Compatible` 标志
- `ASLR support` - Implicit - neither virus nor decryptors are position dependant
- `SafeSeh support` - 禁用了 `SafeSEH` 使用自定义 `SEH` 处理程序

### 2.2 Anunnaki 执行流程

简化的执行步骤：

- `step 1`: 重定位, 获得当前基址
- `step 2`: 安装 `SEH`
- `step 3`: `anti emulator_1`
- `step 4`: `anti emulator_2`
- `setp 5`: 获得调用的 `APIs`
- `step 6`: 获得 `SFCH` 支持函数
- `step 7`: 搜索当前目录，进行感染
- `step 8`: 执行 `payload`

```
Anunnaki  Flow  Charts  ------.
                                   |
                                   V
                    +------------------+
                    |      重定位       |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |    安装 SEH      |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  anti emulator1   |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  初始化病毒 vars   |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  anti emulator2    |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |   get apis        |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |  get sfc apis     |
                    +------------------+
                                   |
                                   V
                    +------------------+
                    |search current dir|
                    +------------------+
                       /          \\
        +--------.           .----------------+
        |                                |
        V                                V
+---------------------+             +-------------+
|infect file (*.exe)  |             |    exit     |
+---------------------+             +-------------+
        |
        V
+---------------------+
| epo && ope && crypt  |
+---------------------+
        |
        V
+---------------------+
|   exec payload       |
+---------------------+
        |
        V
+---------------------+
|      exit            |
+---------------------+
```

### 2.3 get kernel32 base address

在获得 `kernel32.dll` base address 方面，实用稳定的方式有 3 种：

1. 进程被加载后，`esp` 指向 `kernel32.dll` 空间，搜索该空间即可
2. 遍历 `SEH` 链，当 `prev == -1` 时，该处理异常的 `routine`，在 `kernel32.dll` 的地址空间，搜索该空间即可找到 `kernel` 的 base address。
3. 利用 `fs:[30]` 获得 `PEB`，通过遍历 `InMemoryOrder` 模块列表，直接定位 base address。`Anunnaki` 使用了方案 3。

还有一些稀奇古怪的硬编码方式，缺少通用性。但是作为一种思路，可应用在编写 `shellcode`，检测 `os version`，或是 `SEH` 当中。

**in**: `null`
**out**: `eax` --> `kernel32` 的基址

```
0040159D           |>  60               pushad
0040159E           |.  FC               cld
0040159F           |.  33D2             xor edx,edx
004015A1           |.  64:8B52 30       mov edx,dword ptr fs:[edx+30]          ;  指向 PEB 的指针
004015A5           |.  8B52 0C          mov edx,dword ptr ds:[edx+C]           ;  指向 PEB->_PEB_LDR_DATA
004015A8           |.  8B52 14          mov edx,dword ptr ds:[edx+14]          ;  指向 InMemoryOrder 中第一个模块列表
004015AB           |>  8B72 28          /mov esi,dword ptr ds:[edx+28]         ;  esi 指向模块名称
004015AE           |.  B9 18000000      |mov ecx,18                             ;  按 Unicode 计算 kernel32.dll 的长度为 0x18
004015B3           |.  33FF             |xor edi,edi                             ;  edi 为要计算的模块名称的 hash 值
004015B5           |>  33C0             |/xor eax,eax
004015B7           |.  AC             ||lods byte ptr ds:[esi]                        ;  esi 指向模块名的首地址
004015B8           |.  3C 61           ||cmp al,61                              ;  61 -> 'a'
004015BA           |.  7C 02           ||jl short Anunnaki.004015BE
004015BC           |.  2C 20           ||sub al,20
```
