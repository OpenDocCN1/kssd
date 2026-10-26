
```
.text:0045B460  WinMain

.text:0045B6CD      call  sub_515000

.text:00515032            |- call  sub_514ED0

.text:00514EF3                     |- call  sub_417E40                           检查`C:\ProgramData\Source Insight\4.0\si4.lic`文件是否存在

.text:00514F03                     |- call  sub_5140E0                           校验lic文件的函数，如果`si4.lic`文件存在，则执行校验

.text:005140FD                              |- call  sub_5127A0                  加载`si4.lic`，校验字段值，保存校验结果。成功返回`0xC8`

.text:005127EA                              |        |- call  sub_511150      

.text:00511156                              |        |        |- call  sub_45A770 加载`si4.lic`文件

.text:0045A779                              |        |                 |- call  sub_45A290 

.text:0045A2E7                              |        |                          |- call  sub_41B290   

.text:0041B2B2                              |        |                                   |- call  sub_4567F0 

.text:00456898                              |        |                                   |        |- call  ds:CreateFileMappingW

.text:0041B2D9                              |        |                                   |- call  ds:MapViewOfFile  

.text:00512806                              |        |- call  sub_510570      校验并存储`<Type>`值，与`"Trial"`、`"Beta"`、`"Standard"`相比较，设置校验结果

.text:00512875                              |        |- call  sub_510570      校验并存储`<LicensedUser>`值，不存在则结束，返回失败标志

.text:005128FA                              |        |- call  sub_510570      校验并存储`<Serial>`值，不存在则结束，返回失败标志

.text:00512927                              |        |- call  sub_510570      校验并存储`<ActId>`值，如果值等于`"Deferred"`，设置标志位，表明`si4.lic`是断网激活时生成的一次性许可证

.text:00514102                              |- cmp   eax, 0C8h                判断 `sub_5127A0` 返回值，失败则弹出错误窗口

.text:00514133                              |- mov   eax, [esi]               判断如果是“一次性许可证”，则函数返回False

.text:0051419B                              |- call  sub_512CF0               加载`si4.lic`，校验签名。成功返回`0xC8`

.text:00512D39                                       |- call  sub_45A770      加载`si4.lic`文件                               

.text:00512D59                                       |- call  sub_457D80      遍历`si4.lic`中的数据，查找`<Signature>`标签

.text:00512D6B                                       |- call  sub_458520      找到`<Signature>`后，获取它的值

.text:00512DEB                                       |- call  sub_402E00      Base64解码Signature字符串值，得到二进制签名数据

.text:00512E08                                       |- call  sub_510640      CryptoAPI校验签名。成功返回`0xC8`
```

### 还原成伪代码

```
InfoObj *g_infoObj;

int WinMain()

{

    g_infoObj->sub_515000();

}
```

### 校验`si4.lic`中签名的函数

```
int sub_512CF0()

{

    sub_45A770();       //加载`si4.lic`文件                               

    sub_457D80();       //遍历`si4.lic`中的数据，查找`<Signature>`键值对

    sub_458520();       //从`<Signature>`键值对中，获取Value值

    sub_402E00();       //Base64解码Signature值，得到二进制签名数据

    sub_510640();       //使用CryptoAPI校验 RSA签名，校验成功返回C8

}
```

```
class InfoObj{

    int mLicenseType;   //offset:+0    许可证类型：Trial/Beta/Standard



    //InfoObj的构造函数

    sub_512BF0() {

    }



    sub_515000() {

        this.sub_514ED0();

    }



    sub_514ED0() {



        //检查`C:\ProgramData\Source Insight\4.0\si4.lic`文件是否存在

        boolean success = sub_417E40();     

        if(success)

        {

            //如果`si4.lic`文件存在，则执行校验

            this.sub_5140E0();              

        }

    }



    sub_5140E0() {



        //加载`si4.lic`，校验字段值，保存校验结果

        int loadLicResult = this.sub_5127A0();

        if(loadLicResult != 0xC8) {

            //弹出错误窗口

            return false;

        }



        //检查是否是一次性许可

        if(mLicenseType == 2) {

            //弹出错误窗口

            return false;

        }



        //加载`si4.lic`，校验签名

        if(sub_512CF0() != 0xC8) {

            //弹出错误窗口

            return false;

        }        



    }



    //读`si4.lic`文件，校验并保存字段值

    sub_5127A0(){

        sub_511150();           //加载`si4.lic`文件，这个函数里调用了 `sub_45A770` `CreateFileMappingW` API

        sub_510570("Type");     //获取`<Type>`值，与`"Trial"`、`"Beta"`、`"Standard"`相比较，设置校验结果

        sub_510570("Serial");   //获取`<Serial>`值，不存在则结束，返回失败标志

        sub_510570("ActId");    //获取`<ActId>`值，如果值等于`"Deferred"`，设置`mLicenseType=2`，表明`si4.lic`是断网激活时生成的一次性许可证

        ...

    }

};
```

调用关系看起来还是有些复杂，这里强调重要的三个函数作用，其中后两个函数需要还原成C语言代码， 以便验证自己签名的数据是否正确：

- `sub_5127A0` 函数，是校验 `si4.lic` 文件中字段的函数。如果使用之前生成的一次性许可，调用这个函数会校验失败，后续也不会走校验签名的逻辑。所以，需要注册一个试用许可，以便通过该函数的校验。

- `sub_402E00` 函数，Base64解码函数，将 `si4.lic` 文件中 `<Signature>` 签名字符串，转为`0x100`字节大小的二进制签名数据。

- `sub_510640` 函数，调用CryptoAPI，校验二进制签名数据。

## 2.2 简单的po jie

在 `2.1 节` 的调用流程中提到，`sub_512CF0` 函数是签名校验的入口函数，如果此函数返回值等于`0C8h`，则表示校验通过；如果返回值不等于`0C8h`，则会弹出验证错误窗口，汇编代码如下面所示。

在`.text:005141A8`处检查校验结果，成功就跳转到 `loc_5141D4`，失败则会执行错误流程。

所以，可以将此处的比较跳转 `jz` 改为无条件跳转 `jmp`，修改二进制就是将 `74` 改为 `EB`，这样就过掉了校验签名。

```
.text:0051419B E8 50 EB+                call    sub_512CF0

.text:005141A0 83 C4 08                 add     esp, 8

.text:005141A3 3D C8 00+                cmp     eax, 0C8h

.text:005141A8 74 2A                    jz      short loc_5141D4

.text:005141AA

.text:005141AA          loc_5141AA:                             ; CODE XREF: .text:00514179j

.text:005141AA                                                  ; .text:0051418Bj

.text:005141AA 83 BC 24+                cmp     dword ptr [esp+108h], 0

.text:005141B2 74 0D                    jz      short loc_5141C1

.text:005141B4 50                       push    eax

.text:005141B5 8B CE                    mov     ecx, esi

.text:005141B7 E8 D4 CB+                call    sub_510D90

.text:005141BC E8 2F E9+                call    sub_412AF0

.text:005141C1

.text:005141C1          loc_5141C1:                             ; CODE XREF: .text:005141B2j

.text:005141C1 8B CE                    mov     ecx, esi

.text:005141C3 E8 08 CF+                call    sub_5110D0

.text:005141C8 33 C0                    xor     eax, eax

.text:005141CA 5E                       pop     esi

.text:005141CB 81 C4 00+                add     esp, 100h

.text:005141D1 C2 04 00                 retn    4

.text:005141D4          ; ---------------------------------------------------------------------------

.text:005141D4

.text:005141D4          loc_5141D4:                             ; CODE XREF: .text:00514184j

.text:005141D4                                                  ; .text:005141A8j
```

## 2.3 自签名license文件po jie

上面分析调用流程时，已经提到 ：

`sub_402E00` 函数，会把 `si4.lic` 文件中的字符串签名解码为`0x100`字节大小的二进制签名数据，命名该函数为 **Base64Decode**；

`sub_510640` 函数，会用程序中自带的2048位非对称加密的公钥，以 `si4.lic`文件中 `Signature` 标签之前的数据作为校验数据，与二进制签名数据作比对，命名该函数为 **VerifySignature**

先粗略描述一下服务器签名以及本地验证签名的过程：

- 首先，服务器有一对非对称密钥，导出PEM格式公钥后，写死在程序中，在后面 `VerifySignature` 函数中可以看到这个字符串公钥。私钥是存储在服务器上，且要严密保存的。

- 在 `si4.lic` 文件中，`Signature` 标签之前的所有内容（还要去掉所有的 `\r\n\t`和空格），就是待签名的数据。服务器使用私钥对数据签名后，生成`0x100`字节大小的签名数据，这部分数据经过 Base64编码，变成字符串数据，也就是 `si4.lic` 文件中 `Signature` 标签中的字符串。

- 程序启动后，会加载 `si4.lic` 文件，先调用 `Base64Decode` 函数将文件中的 Signature 字符串签名转为二进制签名，再把二进制签名和 `si4.lic` 中原始的待签名的数据传给 `VerifySignature` 函数，该函数内调用 `CryptVerifySignatureW` API校验签名是否匹配。

`si4.lic` 文件内容如下：
