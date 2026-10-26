
`void CypherGuardCore::AES_PCTS_Decrypt_Body(char *dst, const char *src, uint64_t srcsize, const void *key, size_t keylen, char *iv, uint64_t offset, void* ivcache, char *finiv)`

这里`unsigned char vector[16]*`都给我简化为了`char*`，实际源码应该有类似`typedef char aesblock[16]`的定义，然后参数写成`aesblock*`，据说是这样数组指针相对于`char*`更方便编译器对齐和寻址优化，当然对现代编译器没有意义。

我们顺便把`CypherGuardCore::AES_PCTS_Decrypt_CFB_scrap`/`CypherGuardCore::AES_PCTS_Decrypt_CBC_scrap`/`CypherGuardCore::AES_PCTS_Decrypt_CTS_scrap`也加上日志

``` 
4s.9767.0088 CypherGuardCore::AES_PCTS_Decrypt_Body++

4s.9767.0106 dst: 0x12878a000, src: 0x12878a000, srcsize: 52000, key: 0x7fdf1dba6ee0, keylen: 10

00000000: EF BD 5E 7E 3C 44 63 01  91 86 D0 B2 3A C6 70 C1 | ..^~<Dc.....:.p.

4s.9767.0122 iv: 0x700006ab6c50, offset: 0, ivcache: 0x7fdf1dba6ee0, finiv: 0x700006ab6c70

00000000: 32 05 0B 83 87 6D 7A D0  12 5B 74 54 94 77 EB FC | 2....mz..[tT.w.

……

4s.9771.0558 CypherGuardCore::AES_PCTS_Decrypt_Body--

4s.9771.0566 CCCryptorUpdate(0x7fdf1b703a00, 0x700006ab6c70, 10, ...)

4s.9771.0571 CypherGuardCore::AES_PCTS_Decrypt_CTS_scrap++

4s.9771.0574 data: 0x1287dc000, datalen: F57, key: 0x7fdf1dba6ee0, keylen: 10, iv: 0x700006ab6c70

4s.9771.0579 CCCryptorUpdate(0x7fdf1b704a00, 0x1287dc000, F40, ...)

4s.9771.0784 CCCryptorUpdate(0x7fdf1b704a00, 0x1287dcf40, 10, ...)

4s.9771.0790 CCCryptorUpdate(0x7fdf1b704a00, 0x700006ab6b00, 10, ...)

4s.9771.0794 CypherGuardCore::AES_PCTS_Decrypt_CTS_scrap—
```

这两者一个处理52000一个处理F57是整个一个子文件的大小52F57，分为body和scrap分别处理，scrap估计他们取得是边角料的意思，也就是不能按照4k对齐的部分。

然后是下一个body和scrap。

Body的数据拆分为4k的大块进行解密，让我们把大块单位设为`cube`，和每组16字节的block区分开来。

首先分为`n`个大块，然后每块的`key`是`EFBD5E7E3C4463019186D0B23AC670C1`，每块的`iv`是初始`iv`使用`key`循环加密的结果，第0块经过1轮加密，依次类推。最后经过`n+1`次加密的`iv`复制给`finiv`，后继还有用。

然后`CTS_scrap`的解密不是太复杂，也是以16字节为一组的粒度，分为前面`n-2`行和后面两行，前面连续解密后面分两次解密。

连续解密使用的`iv`从日志中可以看出，是跟Body输出的`finiv`同一个地址。实际确实是解密body最后一次的`iv`。

后面两块的`key`也是输入的`key`，`iv`则要看是否存在倒数第三行。

如果存在倒数第三行，使用倒数第三行的密文作为`iv`，否则用输入的`iv`做`iv`。

最后处理这最后两行，倒数第二行一定是16字节字节，直接`aes_ecb`解密，输出记为`decfullrow`，

然后最后一行左对齐复制到16字节的本地缓存`tailrow`，与`decfullrow`进行xor后存入`localrow1`。

如果最后一行是不完整的，将`localrow1`的右侧和`tailrow`的左侧组合成`localrow0`。

然后`localrow0`再进行`aes_ecb`解密，输出记为`localrowdec`。`localrowdec`与`iv`进行xor后存入倒数第二行。

`localrow1`则存入最后一行。

根据以上信息，我写了个`dmmbdec`小程序，能够解出`dmmb`文件中的子文件的原文。如果再加上`dmmb`的文件解析，就能够解出整个”胶囊包”了。

胶囊包的格式是从哪整理出来的呢？

在`+[CPCapsulizedFile capsulizedFileWithContentsOfFile:error:]`函数里面，有一个读取`footersize`的函数，然后根据读取出来的`footersize`选择不同的类初始化。

`footersize`是输入文件的末尾的4字节内容。看了下两个`dmmb`都是`3C`，又买了本小说<<龙王的工作>> 网页写是epub，后缀`dmmr`，`footersize`是`90`。后面又买了一些限时免费的epub，后缀`dmme`的，也是`90`。

4个`footersize`对应4种文件类型，分别是`3C->3`、`90->1`、`60->4`、`F6->2`。

``` 
    footertype = CypherGuardCore::test_footer_size(fd);

    switch ( footertype ) {

        case 1:

            return [[__CPIR_XdJKKTxTUCuhXpgR initWithContentsOfFileVersion1:aFile error:error] autorelease];

        case 2:

            return [[__CPIR_XdJKKTxTUCuhXpgR initWithContentsOfFileVersion2:aFile error:error] autorelease];

        case 3:

            return [[__CPIR_NCwtwzHlh6HmHQ2b initWithContentsOfFileVersion1:aFile error:error] autorelease];

        case 4:

            return [[__CPIR_NCwtwzHlh6HmHQ2b initWithContentsOfFileVersion2:aFile error:error] autorelease];

        default:

            [__ErrorRepresantation assignError: error domain:CPSecurityLibraryErrorDomain code:1];

            return 0;

    }
```

可见我这里测试的文件都是`Version1`，而`dmmb`对应的处理类是`__CPIR_NCwtwzHlh6HmHQ2b`，`dmmr`对应的是`__CPIR_XdJKKTxTUCuhXpgR`。

这两个类应该分别对应多文件胶囊，和单文件胶囊。类的名字估计是用`#define`的方式，做了源码级别名称混淆。应该不是链接后的后处理工具修改macho，因为后面在这个库和其他库里都会看到，方法名的每一个参数名都混淆成了很长的字符。

![](https://bbs.kanxue.com/upload/attach/202012/670_Y5GP6TX676HW6M4.jpg)

因为objc存在着方法类型签名，所以反混淆必须联动修改签名。再加上主程序或者多个库之间互相调用，则需要多个文件一起修改。

我以前没有发现类似的重命名工具，就自己做了一个，能够生成修改后的文件也可以生成`py`脚本补丁到IDA里。但是这种撑到很长的，一来不方便通过长度判断本来的单词组合，二来如果想要改短，修改字符串表的时候可能会遇到末尾对齐的链接器优化。

比如有两个method，`__CPIR_aaaa:__CPIR_bbbb:`和`__CPIR_bbbb:`后者的`SEL name`可能是指向前者的中间。

所以我也懒得改它了，凑合着静态看完了。

当它是多文件胶囊类的时候，文件末尾的结构如下

``` 
struct _cp_package_footer1_s

{

    uint32_t magic; // DNBF

    uint32_t titleoffset;

    uint32_t titlelen;

    uint32_t headeroffset;

    uint32_t headercount;

    uint32_t rev14;

    uint32_t rev18;

    uint32_t rev1C;

    uint32_t rev20;

    uint32_t rev24;

    uint32_t rev28;

    uint32_t rev2C;

    uint32_t rev30;

    uint32_t rev34;

    uint32_t footersize;

};
```

根据这个`headeroffset`我们来到文件对应的偏移，会有一个这样的结构

``` 
struct _cp_package_file_header0_s

{

    uint32_t magic; // DNBA

    uint16_t feature;

    uint16_t rectype;

};
```

当`magic`正确的时候，后面会跟

``` 
struct _cp_package_file_header1_s

{

    uint32_t flag;

    uint32_t attributes;

    uint64_t filetime;

    uint32_t filesize;

    uint32_t crc32;

    uint16_t namelen;

    uint16_t padlen;

};
```

因为没有遇到其他`magic`，因此不知道会不会有其他格式的`header1`。

这个结构末尾紧跟着的是Unicode的文件名，长度是包含末尾0字符的。然后跳过`padlen`后是附带的文件内容，`padlen`我这里看到都是0。

`flag`里面会有bit提示文件是不是加密，`attributes`里面有bit表明他是不是目录。比如我这里`dmmb`里面会有一个不加密的`index.bin`文件，里面包含了`guid`信息。阅读器会用这些信息向CypherTech的服务器请求授权。

这里面还有我们之前提到的`iv`的前体，我叫他`preiv`，为了便于区分，之前说的解密文件的叫做`filekey`/`fileiv`，而`preiv`就是两者的来源。

服务器返回的是证书信息，存储在cyphertech自己的`X:\Cypher License Information\Services`目录，里面会包含`servicekey`，用它来解`index.bin`里面的`fileinfo`，在偏移50的地方得到`prekey`。

描述大概如下

``` 
fileiv=aes_ecb_enc(footer1->preiv, keykey)

fileinfo9c=readfile(footer->fileinfodelta, footer->size)   // index.bin for dmmb or entire file

prekey=aes_cts_dec(fileinfo9c, servicekey)->offset50

filekey=aes_ecb_dec(contentskey, prekey)
```

当时主要静态看的，还发现一种生成`key`的方式，估计是测试用途，实际没有抓到执行。

``` 
ivkey=aes_ecb_dec(sha256(phase), keysource)

keykey= hi128(ivkey) ^ lo128(ivkey)
```

这里`ivkey`/`keykey`是`servicekey`/`contentskey`昵称，容易记一些。

我企图比较一下不同的加密文件，是不是这两个`servicekey`/`contentskey`是相同的，这样我们可以一次抓取，然后自己生成`filekey`/`fileiv`，不用一本一本书的点开，并查看日志。

很遗憾根据hook出来的日志，并非如此。

顺便一提，`dmmb`里面的`index.bin`的格式其实和`dmmr`/`dmme`格式一样，只不过不包含文件内容。它在windows版hook时候是跟`dmmr`/`dmme`走同样的`VFT`函数解析的。

“单文件胶囊”末尾的结构如下

``` 
struct _cp_footer1_s

{

    uint16_t var0;

    uint16_t type1;

    uint32_t flag2;

    uint32_t var3;

    uint32_t fileinfodelta;

    uint32_t fileinfosize;

    uint32_t contentoffset;

    uint32_t contentsize;

    uint8_t encmd5[16];

    uint8_t preIV[16];

    uint8_t zero10[32];

    uint32_t servinfooffset;

    uint32_t servinfosize;

    uint32_t var13;

    uint16_t var14;

    uint8_t var15;

    uint8_t var16;

    GUID guid;

    uint8_t servdll[16];

    uint32_t footersize;

};
```

如果是曾经的我，为了解决”我买了5本书，还要分5次抓`key`/`iv`”的烦恼，我会花很大功夫，重写个客户端，自己搞定验证和返回`servicekey`/`contentskey`的流程，然后就随便怎么解密了。

而现在的我会思考这一切有什么意义，我起初的初衷只不过是为了博取小朋友的夸奖啊？

抛开这一切，半仙强迫症复发，还是决定要做个windows下的工具，为此我要把mac版的hook位置，在windows版定位出来。

首先再次用010Editor打开处于阅读状态的阅读器，然后搜索`iv`内容，可以找到`iv`，也可以找到`key`，`iv`前后的内容还包含有像是证书的字眼。

当然了，mac和windows版虽然界面都是Qt+QML的，但就是解密架构不一样，半仙重新拉起来`ctcrypt_43`开始分析，因为在之前的分析和hook中，打开一个`dmmb`多文件胶囊，`ctcrypt_94`调用很多次，而`ctcrypt_43`只调用一次。

推测的几个导入函数作用为：

- `ctcrypt_43`是给`CEncryptOperator`设置`keycontainer`。
- `ctcrypt_23` `CSymmetricKeyContainer`析构
- `ctcrypt_42` `CEncryptBufferOperator`析构。
- `ctcrypt_41` `CEncryptBufferOperator`构造
- `ctcrypt_22` `CSymmetricKeyContainer`构造
- `ctcrypt_94`是解密块/末尾
- `ctcrypt_1`是初始化?异常。

调用`ctcrypt_43`的地方是哪里呢？就是`drm::cypherguard::decryptor`的`VFT`第二个函数。

我大概整理了一下
