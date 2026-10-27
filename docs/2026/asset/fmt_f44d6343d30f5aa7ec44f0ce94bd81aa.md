
# [原创]Sublime Text 4 Build 4126 RSA 公钥替换分析

这是一个比较简单的例子，可以用来学习一下如何替换 RSA 公钥。

**相关下载链接：**

- [macOS](<https://bbs.kanxue.com/elink@076K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4y4#2j5X3I4A6L8h3g2@1k6i4S2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2T1L8r3W2E0k6g2)9#2k6Y4c8W2P5s2c8Q4y4h3k6T1N6h3W2D9k6q4)9#2k6U0b7I4x3U0k6Q4y4h3k6E0j5h3y4Q4x3X3g2*7K9i4l9>)
- [Windows](<https://bbs.kanxue.com/elink@e3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4y4#2j5X3I4A6L8h3g2@1k6i4S2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2T1L8r3W2E0k6g2)9#2k6Y4c8W2P5s2c8Q4y4h3k6T1N6h3W2D9k6q4)9#2k6U0b7I4x3U0k6Q4y4h3k6^5y4U0c8Q4x3X3g2*7K9i4l9>)
- [Linux x86-64](<https://bbs.kanxue.com/elink@6d0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4y4#2j5X3I4A6L8h3g2@1k6i4S2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2T1L8r3W2E0k6g2)9#2k6Y4c8W2P5s2c8Q4y4h3k6T1N6h3W2D9k6q4)9#2k6U0b7I4x3U0k6Q4y4h3k6^5y4U0c8Q4x3X3g2@1j5i4u0Q4x3X3g2^5P5R3>)
- [ARM64](<https://bbs.kanxue.com/elink@ac5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6i4y4#2j5X3I4A6L8h3g2@1k6i4S2@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4g2T1L8r3W2E0k6g2)9#2k6Y4c8W2P5s2c8Q4y4h3k6T1N6h3W2D9k6q4)9#2k6U0b7I4x3U0k6Q4y4h3k6S2M7X3@1$3y4q4)9J5k6i4c8S2M7W2)9J5k6i4S2*7>)
- [官网](<https://bbs.kanxue.com/elink@f5dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6N6h3u0D9K9h3#2W2N6r3g2^5N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2N6$3&6D9L8$3q4V1>)

目标文件：`sublime_text.exe`（Windows）

## 调试分析（构造许可、替换RSA公钥）

在载入调试器后，搜索 `"Unlimited"` 字符串，定位到调用代码段。

![](https://bbs.kanxue.com/upload/attach/202202/939926_AHBYSM24P23ZB9K.jpg)

图（1）

如图（1）所示，这里的代码段将会为你拼接一个伪许可，其格式如下：

```
Name

Unlimited User License

EA7E-1000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000
```

到这里我们就有了许可的基本格式了，然后继续 Step over。在图（1）的代码段下方会进入一个关键调用（所有验证在这里进行）。如图（2）所示：

![](https://bbs.kanxue.com/upload/attach/202202/939926_JMXXJSTKT4SRGZK.jpg)

图（2）

在图（2）中被选中的调用处，我们 Step into，如图（3）所示：

![](https://bbs.kanxue.com/upload/attach/202202/939926_W2G8ZJSV3SQKNAT.jpg)

图（3）

你可以通过搜索字符串 `"E52D"` 快速跳转到此调用，并向上查找到图（3）所示的代码段。图中被选中的这段代码（文件不同出现的位置都可能不同，我们进入此调用后，通过 Step over，应该很容易找到它）做了一件事情，那就是将一个数组的每一个字节与 `0x7F` 进行异或（文件不同异或值都可能不同），同时将新的数据放到另一个数组中。

原始数据（偏移：`xxxxx8890`）：

```
4FFEE24F72797655F937F988727E7E7E7A7F7CFEF47F4FFEF87DFEFE7FA704DD3A1D88BAAE3573846DC60B73662314A3011211B654D371CDE2269EA6D118F67354F7BCD4A3D08102354CA3BEC0C12C655A639073ED4079C106CD4DFA26D38196F9AA9E213268199529BB9E7928850BA47608BC840AFDC8F3AB04CDB886CD2DCBD639426A89D111965C022BBA37648C9FCFE65F6674B0CC61249A76BC447D7E6E
```

异或 `0x7F`（局部变量：堆栈）：

```
30819D300D06092A864886F70D010101050003818B0030818702818100D87BA24562F7C5D14A0CFB12B9740C195C6BDC7E6D6EC92BAC0EB29D59E1D9AE67890C2B88C3ABDCAFFE7D4A33DCC1BFBE531A251CEF0C923F06BE79B2328559ACFEE986D5E15E4D1766EA56C4E10657FA74DB0977C3FB7582B78CD47BB2C7F9B252B4A9463D15F6AE6EE9237D54C5481BF3E0B09920190BCFB31E5BE509C33B020111
```

为了获取异或结果，我们可以通过继续 Step over，且在图（4）所示的代码段中被选中的位置停下，通过寄存器 `rsi` 的值，来定位它的内存位置。

![](https://bbs.kanxue.com/upload/attach/202202/939926_P2T4DTRT6AYHNKT.jpg)

图（4）

现在我们一起来观察一下异或结果，通过观察我们发现，这不是 DER 格式的 RSA Public Key？有点小兴奋，不过我们还是继续 Step over，直到到达图（5）所示的代码段中被选中的位置停下。

![](https://bbs.kanxue.com/upload/attach/202202/939926_J7XR7EH79DKD7Y6.jpg)

图（5）

在图（5）中被选中的调用处，我们 Step into，如图（6）所示：

![](https://bbs.kanxue.com/upload/attach/202202/939926_K9KQND3V8SZZZG7.jpg)

图（6）

在图（6）中被选中的调用主要用于许可格式的验证（在这里我们暂时不去详细分析它）。我们 Step into 后继续 Step over，会发现有一小段代码，被用来检查许可内容的行数。如果总行数（非空行）乘上 `0x20` 后的结果既不等于 `0x1A0` 也不等于 `0x160`，则说明许可内容的行数错误。也就是说我们许可的总行数（非空行）必须是 13 行或者 11 行，而我们的伪许可有 12 行，在这里我们删掉一行 0（其它信息看似是必须的），我们就得到了一个新的伪许可。

```
Name

Unlimited User License

EA7E-1000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000

00000000000000000000000000000000
```

接下来我们让程序彻底运行起来，直到我们可以通过 help 菜单输入许可为止。

![](https://bbs.kanxue.com/upload/attach/202202/939926_TKHYMX4RKCK79S2.jpg)

图（7）

在许可输入窗口中输入我们新的伪许可（如图（7）所示），然后点击使用许可按钮。程序将在图（3）的位置被断下。我们让程序继续运行，直到再次到达图（6）中被选中的调用处，Step over 后发现许可格式顺利通过（通过该调用下方跳转来判定）。

![](https://bbs.kanxue.com/upload/attach/202202/939926_J6QJNKFVVVFUW2H.jpg)

图（8）

我们继续 Step over，到达图（8）中被选中的调用处，此处主要用于完成签名验证（在这里我们暂时不去详细分析它）。我们 Step into，跟踪分析后发现在内部的子调用中的注解栏中出现了和图（9）类似的信息。

![](https://bbs.kanxue.com/upload/attach/202202/939926_C5SQ4GGKTTP68US.jpg)

图（9）

这，... 图（9）给我们展示了非常多的信息了，首先它使用了一个第三方库，库名叫 tomcrypt，同时也知道了这个子调用的函数原型来自 `rsa_verify_hash.c` 这个文件（其它子调用类似）。我们直接 bing.com 找到这个库（[下载](<https://bbs.kanxue.com/elink@307K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2D9K9h3u0@1L8$3#2Q4x3X3g2F1k6i4c8Q4x3V1j5>)），下载源代码，解压到桌面，找到 `rsa_verify_hash.c`，同时打开源代码文件夹中的 pdf，得知图（9）这个子调用的函数原型对应的是 `rsa_verify_hash()` 这个函数，且功能为 RSA 签名验证。

经过追踪分析得到了结论：我们新的伪许可中的 0 字串在转换到 16 进制后作为签名传递给了对应图（9）的子调用。我们新的伪许可中的非 0 字串被格式化为 `"Name\nUnlimited User License\nEA7E-1000"`，并将它的 SHA1 结果传递给了对应图（9）的子调用。除此外 RSA 签名 padding 使用的是 PKCS #1 V1.5 格式（被签名内容不够时追加 `0xFF`），且通过公钥信息可以知道我们的模为 1024 位（RSA-1024）。

根据追踪分析得到的结论，我们找到了逆向计算签名的过程：首先将 `"Name\nUnlimited User License\nEA7E-1000"` 原始信息进行 SHA1 处理，然后将处理结果进行 DER 编码，编码完成后使用 PKCS #1 V1.5 签名 padding（被签名内容不够时追加 `0xFF`），最后使用幂模运算对 padding 后的数据进行加密，加密结果即是我们最终的签名。

至此，我们已然知道我们的许可必须由以下内容构成：

- 用户名 + 许可类型 + `EA7E-1000` + 签名（1024bit）

现在我们终止调试，因为接下来我们需要创建一个新密钥，且使用新密钥的公钥来替换原始公钥，使用新密钥的私钥来进行签名。这里我们直接使用 tomcrypt 密码库来完成（使用简单，且容易理解）。tomcrypt 的 RSA 功能需要依赖 math 库，有两个可供选择：tommath、tomfastmath（在这里我们选择 tommath）。为了使用一个固定的 RSA 密钥，所以我们不使用 tomcrypt 的 `rsa_make_key()`。而直接使用 `openssl` 来产生一个固定的 RSA 密钥。

```
// generate a private key.

> openssl genrsa -3 -out rsa_prvkey.pem 1024

// convert to traditional DER format private key.

> openssl rsa -in rsa_prvkey.pem -traditional -out rsa_prvkey.der -outform DER
```

`-traditional`：旧版的 `openssl` 是不需要这个选项的，新版 `openssl` 需要加上。

在产生 `rsa_prvkey.der` 文件后，我们可以在程序中直接读取该文件内容，也可以把该文件内容以字节数组的方式添加到代码中（此处我们选择第二种方式）。

```
const unsigned char prvkey[] = { 内容省略 };

const char *info = "yystarsky\n256 User License\nEA7E-1000";

int main(void) {
    unsigned char pubkey[160], sign[128];
    unsigned char hash[20];
    unsigned long lh, lp, ls, idx;
    rsa_key key;
    int err, hash_idx;

    /* register a math library */
    ltc_mp = ltm_desc;
    if (register_hash(&sha1_desc) == -1) {
        printf("Error registering sha1");
        return EXIT_FAILURE;
    }

    hash_idx = find_hash("sha1");

    err = rsa_import(prvkey, sizeof(prvkey), &key);
    if (err != CRYPT_OK) {
        printf("hash_memory %s", error_to_string(err));
        goto __err;
    }

    lh = 20;
    err = hash_memory(hash_idx, (const unsigned char*)info, strlen(info), hash, &lh);
    if (err != CRYPT_OK) {
```
