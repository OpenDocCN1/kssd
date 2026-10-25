
# prikey.c 关于 ecc 内容翻译

> 原文：[`www.pediy.com/kssd/pediy10/61891.html`](https://www.pediy.com/kssd/pediy10/61891.html)

【翻译】`prikey.c` 关于 ecc 内容翻译

“翻译 by nujia” 这篇翻译来自 `prikey.c` 的最后部分。

## v7.1 改进

### 概述

v7.1 有显著的改进，有很多新的特征但是没有公开文档，包括：

- `SIGN=`、`SIGN2=` ... `SIGNn=`
- v7.1 `SIGN` 如果是 `non-CRO` 格式，会有 12 个字符，但是也和以前的格式有点不同.
- `CRO` 格式有三种 `sign` 长度。
- 可以使用 2 段过滤器，以后可能不再支持。
- 总的来说有 6 点关键的功能（5 个是新增的）
    - `license-key`
    - 没有使用 `CRO` 的 `SIGN`，与 `license-key` 有些相似
    - 支持 113 位 `sign` 长度 （57 字节）
    - 支持 163 位 `sign` 长度 （84 字节）
    - 支持 239 位 `sign` 长度 （120 字节）
    - 2 段过滤器 (现在没有相关文档)

2 段过滤器以后不在支持。

有很多灵活的功能如下：

每个 `checkout` 可以定义/忽略通过 `LM_STRENGTH`

同样，每个 `FEATURE` 可以定义根据 `lmcrypt` 里的 `LM_STRENGTH` 的值来确定 `sing` 的长度

这样一个公司可以自己决定在应用程序中使用 `LM_STRENGTH` 的长度。

不同 `sign` 长度的原因是软件公司可以用不同的 `seed` 或不同的 `key` 过滤器（比如两段过滤器）

下面主要讨论 `cro`，最后再讨论 `non-cro`

没有公开的文档是什么？为什么没有公开？

### 创建

在创建一些 `key` 过滤器时，需要一步步详细的介绍。

第一个依赖关系序列如下:

```
$DAEMON 依赖于 lsvendor.o

lsvendor.o 依赖于 lm_new.o(lm_new.c)

lm_new.c 由 lmnewgen 生成

lmnewgen 依赖于 lmcode.o(lmcode.c)

lmcode.o 由 lmrand1 生成，

lmrand1 依赖于 lm_code.h

(译者注：DAEMON->lsvendor.o -> lm_new.o ->lmnewgen ->lmcode.o ->lmrand1 ->lm_code.h)
```

这些依赖关系和 `flexlm` v6.1 是相同的，唯一不同的是 `lm_new.o` 里包含了更多的信息。 本质上，`lm_code.h` 文件是用一个“模糊”的格式转换成了 `lm_new.o` 。

下面是 `lc_new_job()`里 `VENDORCODE` 结构的新添加的 4 项内容。

```
int pubkeysize[LM_PUBKEYS];
unsigned char pubkey[LM_PUBKEYS][LM_MAXPUBKEYSIZ];
int strength;
int (*pubkey_fptr)();
```

`LM_PUBKEYS` 是 3, 所以如果使用 `CRO`, 关于 `CRO` 的不同的信息就在 `VENDORCODE` 结构里。

`pubkeysize`，`pubkey` 和长度都是计算公钥的必须的参数。

`pubkey_fptr` 存在于函数 `l_pubkey_verify()`中，除了 `license-generator`,它叫 `l_prikey_sign()`函数。

（这种格式用在任何 `license-key` 过滤器重，比如 `flexlm` v6 里的两段过滤器。这过滤器仍然有效，并且在 `servtest` 函数里测试过，但是我们不对此提供支持）

`lm_new.o` 现在包含了所有客户端的公钥信息。

### 公钥／私钥对

但是 `lm_code.h` 只包含 `SEED` 和 `VENDORCODE`，它是怎样转变成公钥／私钥的呢？

公钥对由 `lmnewgen` 产生，`seed3` 和 `seed4` 用来产生公钥对，如果 `seed3` 和 `seed4` 不变，那么每次运行都会产生相同的公钥对，但是如果你改变了 `seed` 的一位，公钥对的一半将会改变，

也就是这种改变的对应关系是不可能逆向的。从理论上讲，从公钥对是不能逆向获得 `seed3` 和 `seed4` 的。

所以，我们没有存储公钥对，`ISV`（独立软件开发商）应该注意到这一点。他们唯一要做的是保存好 `seed3` 和 `seed4`。另外，一个 `license-generator` 比如像 `gtlicense` 理论上能获得 `seed`，为了生成 `license`。

必须注意加密的 `seed3` 和 `seed4` 没有出现在发布的产品中，只有公钥出现在前台/后台程序中，而私钥仅仅出现在 `license-generator` 中。

`lmcode.o` 包含了 `lm_code.h` 的信息，（`lmcode.o` 由 `lmrand1` 程序产生。）

`lmcode.o` 和 `lmnewgen.o` 链接生成 `lmnewgen` 程序，而它产生 `lm_new.c`。 它调用 `l_gen_pkey_headers()` 生成 `lm_new.o` (它包含公钥 public-key) 和 `lmprikey.h` (它包含私钥 private key)。

`l_gen_pkey_headers()` 最后调用 Certicom 的程序 `sb_genKeyPair()` 生成公钥/私钥对（ public-key pairs）。

总结：`lmnewgen` 用 `seed3` 和 `seed4` 制作公钥/私钥对。公钥在 `lm_new.c` 被“模糊”化。而私钥保存在 `lmprikey.h` 中，而 `lmprikey.h` 包含在 `license-generator` 源代码中。

### LICENSE GENERATOR（license 生成器）

包含了 `lmprikey.h`，同时用 `LM_CODE_GEN_INIT` 初始化 `VENDORCODE` 结构。一旦初始化完毕，它与客户端唯一不同的两个地方是 `pubkey` 信息是私钥及 `pubkey_fptr` 的返回函数是 `l_prikey_sign()`。

当生成 `license` 时，`sign` 的长度是必须的，它由 `CONFIG` 结构的 `L_CONF_STRENGTH_OVERRIDE` 确定。（警告：出于安全的考虑，`L_CONF_STRENGTH_OVERRIDE` 在 `l_privat.h` 进行了宏定义。）

`license-generator` 有很多选择：你可以使用 `license-key`，和/或其他长度的 `sign`，但是你必须二者择其一。客户端指出使用哪种方法，并且确切的。

### 客户端/守护进程

理论上讲，一个 `job` 可以有不止一个的 `LM_STRENGTH`，而且不同的 `checkout` 可以有不同的 `LM_STRENGTH`。我相信它能实现，但是我们的文档不会提及它，并且也不提供技术支持。我这么说仅仅是表明这个方法是可行的。

`lc_new_job` 调用 `l_init()`函数，在这个函数里，`VENDORCODE` 结构的 `pubkey` 传递给函数 `l_add_key_filter()`。它将 `L_KEY_FILTER` 赋值给 `job->key_filters`，这样也可以提供 `key` 过滤器的作用。当然不同的 `sign` 长度也会有不同的 `L_KEY_FILTER`。

### 客户端(CLIENT)

客户端不用考虑以下两个可能：

1. 需要多长的 `strength`
    - `LM_A_STRENGTH_OVERRIDE` 由 `job->L_STRENGTH_OVERRIDE` 确定
2. 需要 `sign` 的长度
    - `LM_A_SIGN_LEVEL` 由 `job->L_SIGN_LEVEL` 确定

缺省的 `strength` 在 `lm_code.h` 里设置，而且等于 `VENDORCODE` 结构的 `strength`。缺省的 `key-level` 是 1，当 `L_SIGN_LEVEL` 为 0 时，意味着使用 12 位的 `license-key`，而不使用 `SIGN`。

### VDAEMON

守护进程（后台程序）`daemon` 用来验证 `license` 文件中所有的 `license-key`/`signatures` 是否正确。如果有额外的验证也能完成。为了验证 `license` 的正确与否，它需要所有的 3 个 `CRO` strength，即时使用 `license-key` 而没有使用 `CRO` sign。

参见：

- `testsuite/st_vers.c`: test 49.
- `testsuite` 测试所有的 `feature`，也可能执行验证这些 `feature` 是如何工作的。
- `utils/lmnewgen.c`:
    - 它生成公钥，主要是生成 `lm_new.c`，其中 `lm_new.c` 包含模糊化的公钥。

To: `danielbirns@hotmail.com`, `slangford@certicom.com`, `bolson@certicom.com`, `David Znidarsic` `<davidz@globes.com>`, `Matt Christiano` `<matt@globes.com>`

Subject: Summary of review

Cc: `richard Mirabella` `<rich@globes.com>`

### 发现了 ECC 的一个安全漏洞

- 在 `handshake` 中使用 `seed3` 和 `seed4`，我们可以用一台运行较快的 PC 用大约一天的时间暴力破解出 `seed3` 和 `seed4`。
- 假如在 `signature` 生成过程不超过 1 毫秒，那么第一个一半如果 `signature` 为 2 个 `signature`（这句话不会翻译，:(）。假如是真的，有经验的 `hacker` 就能立刻获得私钥。这对于 113 位的 `sign`（57 字节）将很快找到私钥，我的 PC 的运算速度就可以。

### 建立（setup）

- 96 位的 `seed` `S`。
- 提供优化的选项给 `ISV`（独立软件开发商） 生成可靠的随机 96 位 `seed`，或者客户自己定义 96 位的 `seed` `n`。
- 从 96 位的 `seed` `S` 和 `n` 生成更多的 96 位的 `seed` `Tn`，使用 `FIPS186` 随机数发生器产生这些 96 位的 `seed` `Tn`。
- 这些 `Tn` 必须一次生成。
- 新的客户使用 `T2` 和 `T3` 生成 `seed1~seed4`，所以他们只需要 96 位的 `seed` `S`。（`LM_SEED1-3` 共 12 字节 96 位）

### 公钥/私钥对的产生

- 用 `T1` 生成公钥和私钥
- 用 `T1` 和常数 113 生成一个 113 位的公钥/私钥对
- 同样用 `T1` 和常数 163 及 239 生成相应长度的公钥/私钥对
- 确保这些公钥/私钥对不是相同的，即使 `seed` 只相差 1 也足够了。

### signing

- 使算法暂时停止工作
- Private key，消息的散列。
- 相同的 `feature` 会产生相同 `signature`
- 为每个 `signature` 需要一个不同的 Certicom "context"。（Certicom 公司就是提供 ECC 软件服务的公司。）

### 测试

- 加一个测试去避免发生上面提到第二种情况----113 位有第一个 `signature` 的一半相同。
- 标记 `f1-f100` 并且查找前半部分相同的 `signature`

### ISV（独立软件开发商）用如下方法加强保护

- Brian Olson 提到的方法对于私钥和 `seed3-4` 在客户端是安全的。目前的技术水平硬件狗的是合适的。除了 `seed` 和实际的 `signing` 取代了硬件狗。一般认为这是安全和方便的。当然现在硬件狗没有广泛使用，主要没有被更多的客户接受。
- 我的观点是没有真正的安全，在 `ISV`（独立软件开发商）实践过程中，不愿意使用，或者是找到其他方法绕过这种复杂的验证。举个例子，假设每次生成 `license` 你都被要求输入 `password`。这个 `password` 有 96 位，或者是相同长度英语短语，这将给 `hacker` 可乘之机。这两个方法都太长而难以操作，因为你每次要用的时候都需要输入。于是，`password` 就存在硬盘上的某个文件里，或者登陆一次后永不注销，这些都是极不安全的做法。
- 也就是说我们现在需要做一项工作：改进文档，解释什么需要保护。如今，大容量的硬盘上有太多的文件，这些文件没有注释，其中也包括 `S` 或者私钥。

### 结论

- 我认为 `hacker` 唯一可以破解的地方是通过 `handshake` 来获得 `seed3` 和 `seed4`，除非 `hacker` 发现其它的漏洞。
- 还有两步需要做
    1. `Macrovision` 高级工程师应该查看源代码试图找到其他的漏洞。这样就能在 `hacker` 发现之前修补漏洞。
    2. 所有的 Certicom 代码都是 c 语言写的。我可以发送最终版本给 Certicom 公司，让他们发现其中明显的错误用法。
