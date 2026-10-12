
# SMC 与算法结合的快速运用 VC++ (含 DEMO 和工具源代码)

> 原文：[`www.pediy.com/kssd/pediy10/94701.html`](https://www.pediy.com/kssd/pediy10/94701.html)

今天看到《加密与解密 第三版》中关于 `SMC` (Self Modifying Code) 技术实现一章，有自己的一点想法，有可能是前辈早已走过的路，如果内容有重复冲突的话，敬请原谅！

`PS`: 我是菜鸟，高手请直接跳过 :)

`SMC` 技术的原理我这里就不罗嗦了, 加密与解密中说得很清楚。

`DEMO` 程序以 `SMC` 用于序列号校验为例，加密工具则是实现对 `DEMO` 程序中 `SMC` 代码的快速加密！

个人觉得要实现 `SMC` 主要是解决以下问题：

1. 程序本身需解密代码的定位（即需解密代码的起始地址和结束地址）。
2. 加密工具对程序需加密部分代码的定位。
3. 解密算法是 `SMC` 程序强度的关键。这个算法对代码来说是可逆，但对 `CRACKER` 来说，却是不可逆的。

我们来一一实现上面 3 个问题：

### 1. 在程序中定位需加密的起始和结束地址

```
_asm mov address1,offset begindecrypt   // 取待加密代码首地址  关键！
_asm mov address2,offset enddecrypt     // 取待加密代码末地址  关键！

begindecrypt

{
    需被加密和解密的代码
}

enddecrypt
```

以上代码即可实现 `begindecrypt` 为的地址为 `address1`，`enddecrypt` 地址为 `address2`。

### 2. 定义加密工具所用的标签

这些机器码和字符将在 `PE` 文件中体现，并对程序无影响。且插入的字符可被用于加密工具定位。

```
#define SMCEncryptBegin \
    __asm _emit 0xEB \      机器码 EB 是 JMP 的意思，
    __asm _emit 0x10 \      表示要跳到 0X10 字节后的地址，后面的 SMCENCRYPTBEGIN 正好是 0x10 个，因此对程序无影响。
    __asm _emit 'S' \
    __asm _emit 'M' \
    __asm _emit 'C' \
    __asm _emit 'E' \
    __asm _emit 'N' \
    __asm _emit 'C' \
    __asm _emit 'R' \
    __asm _emit 'Y' \
    __asm _emit 'P' \
    __asm _emit 'T' \
    __asm _emit 'B' \
    __asm _emit 'E' \
    __asm _emit 'G' \
    __asm _emit 'I' \
    __asm _emit 'N' \
    __asm _emit 0x00 \
#define SMCEncryptEnd \
    __asm _emit 0xEB \
    __asm _emit 0x0E \
    __asm _emit 'S' \
    __asm _emit 'M' \
    __asm _emit 'C' \
    __asm _emit 'E' \
    __asm _emit 'N' \
    __asm _emit 'C' \
    __asm _emit 'R' \
    __asm _emit 'Y' \
    __asm _emit 'P' \
    __asm _emit 'T' \
    __asm _emit 'E' \
    __asm _emit 'N' \
    __asm _emit 'D' \
    __asm _emit 0x00 \
```

以上的 `emit` 为在编译的时侯，将机器语言指令直接嵌入目标码中，不必借助于汇编语言和汇编编译程式。这样在生产的 `PE` 文件（即生成的 `EXE` 文件）就有了 `SMCENCRYPTBEGIN`, `SMCENCRYPTEND` 字样。

由以上两点，可以写出对函数内任意代码段的加解密，程序如下：

```
#if _ENABLE_SMCENCRYPT_

    DWORD address1,address2,encryCodeSize;

    _asm mov address1,offset begindecrypt   // 取待加密代码首地址  关键！
    _asm mov address2,offset enddecrypt     // 取待加密代码末地址  关键！
    encryCodeSize=address2-address1;  //得到待加密代码的大小

    //首先解密代码段，因为程序生成后已被加密工具加密。

    DecryptBlock((DWORD *)address1,encryCodeSize,Name);

    SMCEncryptBegin  //给加密工具定位的加密开始地址标签
    begindecrypt:    //给程序自己定位的解密开始地址标签

#endif

{
    需被加密的代码段
}

#if _ENABLE_SMCENCRYPT_

    enddecrypt: //给程序自己定位的解密结束标签
    SMCEncryptEnd//给加密工具定位的加密结束标签
    //加密还原回去,否则第二次解密就错了
    EncryptBlock((DWORD *)address1,encryCodeSize,Name);//调用后加密代码段

#endif
```

### 3. 算法的选用

我这里选用了 `RC4`，主要是因为:
1. `RC4` 是成熟算法，强度大;
2. `RC4` 的密钥应用完即可丢弃，在程序中不需体现。
3. `RC4` 可以生成明文和密文的位数一致的序列，方便加解密时对 `SIZE` 的运算。

程序主要代码如下：

```
bool RC4EncryptBlock(void *pStartAddr, int nLength,  LPTSTR strKey)
{
    unsigned char rc4_key[128]={0};
    int key_length = 0;
    key_length = strlen(strKey);
    if(!key_length)//如果长度为 0
        return false;
    strcpy((char   *)rc4_key,(LPSTR)(LPCTSTR)strKey);
    struct rc4_state rc4_Encry;
    if(!pStartAddr || nLength <= 0)
        return false;
    unsigned char *pEncry = (unsigned char *)pStartAddr;
    memset(&rc4_Encry,0,sizeof(rc4_Encry));
    rc4_setup(&rc4_Encry,rc4_key,key_length);//得到一个数组
    rc4_crypt(&rc4_Encry,pEncry,nLength);
    return true;
}
```

`strKey` 为 `RC4` 的输入密钥，这里可以为 `F(用户名,序列号)` 以实现多组用户名和序列号的搭配, 我 `DEMO` 中为了简便, 直接以用户名为 `KEY`。 `address1` 即加密起始地址, 它是一个指针, 运行完 `RC4EncryptBlock` 函数后, 指针对应的内容即变换为加密后的内容，解密函数和加密函数同，都为 `XOR` 指令。如果密钥（即用户名）不正确的话则解出来的代码即为乱码，运行时即会出现异常错误，可以用 `CATCH` 捕获处理。`DEMO` 程序中代码示例如下：

```
try
{
#if _ENABLE_SMCENCRYPT_

    _asm mov address1,offset begindecrypt   // 取待加密代码首地址
    _asm mov address2,offset enddecrypt     // 取待加密代码末地址
    encryCodeSize=address2-address1;  //得到待加密代码的大小
   //首先解密代码段
   RC4EncryptBlock((DWORD *)address1,encryCodeSize,Name);
    SMCEncryptBegin  //给加密工具定位的标签
   begindecrypt:    //给程序自己定位的标签

#endif

    if(CalcRegCode(Name,SN,szBuff,128))//调用注册码计算函数,
    OnOK();

#if _ENABLE_SMCENCRYPT_

    enddecrypt:
    SMCEncryptEnd
     //加密还原回去,否则第二次解密就错了
    RC4EncryptBlock((DWORD *)address1,encryCodeSize,Name);

#endif
}
catch(...)
{
    ::MessageBox(NULL,_T("!!Unknown exception,Wrong Code or Crecked!"),_T("Error"),MB_OK);
//记得恢复加密代码段

#if _ENABLE_SMCENCRYPT_

    RC4EncryptBlock((DWORD *)address1,encryCodeSize,Name);//调用后加密代码段

#endif
}
```

如此，`DEMO` 程序的注意点已全部解决，我们再来分析 `SMC` 加密的实现原理：

1. 读取 `PE` 文件（即程序生成的 `EXE` 文件），读取程序写入的标签，这里 `SMCENCRYPTBEGIN`, `SMCENCRYPTEND`. 得到其开始、结束地址，就可知道需要加密的位置和 `SIZE` 了。
2. 与程序对应的加密方法加密，同样选择 `RC4`。用户自己选择 `RC4` `KEY`，这里的 `KEY` 将成为 `DEMO` 程序中用户输入的正确用户名。
3. 加密后写入 `PE` 文件即可。

加密工具界面如图:

![](img/3fadd774857218a9cd4cc617d56a096d.jpg)

可实现程序中多处 `SMC` + `RC4` 的快速加密, 当然了, 也可以扩展为多处不同算法加密, 这个自己发挥吧。

**上传的附件**

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `SMC Demo and Tool.rar` |
| :--- | :--- |
