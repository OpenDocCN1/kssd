
![图片描述](upload/attach/202303/947361_5PQP9BQN2SCBJYP.png)

再次断下后（前几次断下并不能得到我们要的调用栈，因为相关参数中找不到密文缓冲区等特征），根据`fastcall`约定，秘钥应该是`r9`所指缓冲区，实际上，秘钥缓冲区最后十六个字节作为原始Key的一部分（16字节），即未拓展的Key，秘钥拓展例程通过原始Key进行秘钥拓展，不过不注意这个细节也没有关系，这在之后的分析中将会体现。

![图片描述](upload/attach/202303/947361_3KXC3RK4SWB2C59.png)

我们回溯到`RVA`: `2811EB5`

![图片描述](upload/attach/202303/947361_TWF5K5Q2Q7849GD.png)

在此函数中，`keyArry`已经生成，那么我们继续栈回溯，来到`VA`: `00007FF6444C1137`

![图片描述](upload/attach/202303/947361_SX95JKQCTXFCUFT.png)

[`rcx+0x10`]即是`keyArry`，同样分析此函数，发现`keyArry`同样作为参数传入此函数，那么我们继续进行调用栈回溯。

回退到第二次调用栈，我们来到`VA`: `00007FF6444C1398`,如下图

![图片描述](upload/attach/202303/947361_2WYQ74J7PWTSHQ9.png)

同样的，[`rcx+0x10`]即是`keyArry`，同样是作为参数传递进来的，再次进行堆栈回溯，来到`RVA`: `00000000027F15AB`，如下图，这里再次进行了简单的转发，再次进行栈回溯。

![图片描述](upload/attach/202303/947361_EXWKQ2Q9YP9S5H8.png)

来到`RVA`: `285B20C`，如下图 ![图片描述](upload/attach/202303/947361_BMXG2RQUYCDH2AK.png)

我们所说的`keyArry`是`aes`实例化的一个对象，里面存储有拓展秘钥。再此函数进行简单分析后发现，`key`同样来自函数外通过参数传递进来。

![图片描述](upload/attach/202303/947361_GJ2RNCTTSEX4UHF.png)

继续堆栈回溯-\_-||，来到`RVA`: `000000000285AE26`

![图片描述](upload/attach/202303/947361_3CRQC9TJBPHZ7R4.png)

继续回溯，来到`RVA`: `000000000285AED8`，同样是一个简单的转发，继续回溯，来到`RVA`: `0000000003026BF2`

如下图

![图片描述](upload/attach/202303/947361_TJ3ZP6THHZ8EVG5.png)

如图，[[`v18`]+0x8]指向秘钥，终于要计算秘钥了-\_-||，本函数上方有对`v18`的相关操作，如下图

![图片描述](upload/attach/202303/947361_K72TNT7ZZWHPY76.png)

其实看到‘salt’这几个字符，对秘钥拓展熟悉的朋友应该能马上反应过来这里应该就是拓展秘钥的地方了。我们进到函数里

如图

![图片描述](upload/attach/202303/947361_BMCWQHNA5DUAYVQ.png)

这就非常明确了，我们百度`Pbkdf2`

>PBKDF的全称是Password-Based Key Derivation Function，简单的说，PBKDF就是一个密码衍生的工具。既然有PBKDF2那么就肯定有PBKDF1，那么他们两个的区别是什么呢？PBKDF2是PKCS系列的标准之一，具体来说他是PKCS#5的2.0版本，同样被作为RFC 2898发布。它是PBKDF1的替代品，为什么会替代PBKDF1呢？那是因为PBKDF1只能生成160bits长度的key，在计算机性能快速发展的今天，已经不能够满足我们的加密需要了。所以被PBKDF2替换了。在2017年发布的RFC 8018（PKCS #5 v2.1）中，是建议是用PBKDF2作为密码hashing的标准。PBKDF2和PBKDF1主要是用来防止密码暴力破解的，所以在设计中加入了对算力的自动调整，从而抵御暴力破解的可能性。
>
>PBKDF2的工作流程
>
>PBKDF2实际上就是将伪散列函数PRF（pseudorandom function）应用到输入的密码、salt中，生成一个散列值，然后将这个散列值作为一个加密key，应用到后续的加密过程中，以此类推，将这个过程重复很多次，从而增加了密码破解的难度，这个过程也被称为是密码加强。

稍微阅读以上引用内容 ，对比此函数参数不难得出：

1. 盐值：`saltiest`
2. 秘钥：小程序ID
3. 秘钥拓展算法：`PBKDF2`
4. 迭代次数：1000
5. 秘钥长度：256位

既然秘钥长度是256位，那么可以推测出加密算法是`AES_256`。

注意到的是，它的伪散列算法是可以替换的，那么我们下一步要找出的是它使用的是哪种散列算法

这里笔者简单使用js选几种常见的散列算法试一试

```c++
const crypto = require('crypto');
let appid = "wxf2a0156c0235fc4c";
crypto.pbkdf2(appid,"saltiest",1000,32,'sha1',(err, derivedKey) =>
{
if (err) throw err;
console.log("The password is ",derivedKey.toString('hex'));
});
```

![图片描述](upload/attach/202303/947361_EEQ6STKBS265NPN.png)

对比拓展秘钥函数运行之后的返回值[[`rax`]+0x8]中的值：

![图片描述](upload/attach/202303/947361_4JE6UETFDWMNJTE.png)

可以验证散列算法是‘sha1’，对应我们在分析过程中的拓展秘钥的最后字节部分。

```
00001D5A002CD188 3D9D6AB5E94DDDE8 èÝMéµj.=
00001D5A002CD190 71122C7B6FFE09D6 Ö.þo{,.q
00001D5A002CD198 C3981F7A8828924E N.(.z..Ã
00001D5A002CD1A0 BD14C8D6E69FEA98 .ê.æÖÈ.½
00001D5A002CD1A8 3336977C3609F094 .ð.6|.63
00001D5A002CD1B0 1969D6DC01305252 RR0.ÜÖi.
```

至此，我们找到了秘钥的生成算法。

#### 分组模式以及iv

分组模式以及iv的寻找相对简单，只要对AES加密流程以及几种加密模式的区别熟悉就可以在加密函数（`RVA`: `0000000000040C30`）中分析出加密模式以及向量。这里笔者不再赘述。

经过简单分析，总结之前分析成果，有如下清单：

```c++
除去文件头8个字节，剩余1024解密算法：
秘钥算法：PBKDF2
盐值：saltiest
秘钥：小程序ID，wxf2a0156c0235fc4c
摘要算法：sha1
秘钥长度：32字节
解密算法：aes-256-cbc模式
初始化向量iv：74 68 65 20 69 76 3A 20 31 36 20 62 79 74 65 73 对应字符串：“the iv: 16 bytes” -\_-||
1024字节之后的数据处理方式：
解密方式：异或解密
异或Key：微信appid字符串的第二个字符对应的ASCII码形式。
```

拿到解密出的文件后可以用相应的解包脚本进行解压，网上不乏解压脚本，遗憾的是笔者并没有找到能够彻底解压并且还原出微信开发者工具能够识别的对应各式的文件（微信开发者工具对js等的样式做了一层封装，想要能够调试源代码需要将解包后的文件还原成其能够识别的格式，网上确实有相关脚本，但大多比较老，微信开发工具对样式进行了更新，格式化出现了一些问题，笔者水平有限，就不去修复。）

下面给出不成熟的C++解密脚本

```c++
#include "PKCS7.h"
#include <openssl/evp.h>
#include<openssl/aes.h>
#include <openssl/sha.h>
#include <openssl/crypto.h>
#include <iostream>
#include <string>
#include <fstream>
#include <filesystem>
using namespace std;
unsigned char iv[] = { 0x74,0x68,0x65,0x20,0x69,0x76,0x3A,0x20,0x31,0x36,0x20,0x62,0x79,0x74,0x65,0x73 };//iv
unsigned char recursive_keys[32] = { 0 };//计算AES秘钥
const unsigned char salt[] = "saltiest";//盐值
int main()
{
string app_id("wxf2a0156c0235fc4c");
/*cout << "Plz enter the AppID:" << endl;
cin >> app_id;*/
//计算递归秘钥
PKCS5_PBKDF2_HMAC_SHA1(app_id.c_str(),app_id.length(),salt,strlen((const char*)salt),1000,32, recursive_keys);
//cout << recursive_keys << endl;
string file_name("__APP___.wxapkg");
/*cout << "Plz enter the name of the file you want to decrypt :" << endl;
cin >> file_name;*/
//读取文件
int file_size = std::filesystem::file_size(file_name);
char* file_buf = new char[file_size] {0};
fstream fp(file_name.c_str(),std::ios::in|ios::binary);
if (!fp.is_open())
{
cout << "Sorry,please check that you entered the correct file name" << endl;
delete[] file_buf;
file_buf = nullptr;
return 0;
}
fp.read(file_buf, file_size);
fp.close();
//AES解密前1024字节内容（忽略文件头6个字节）
AES_KEY aes_key;
AES_set_decrypt_key((const unsigned char*)recursive_keys, 256, &aes_key);
AES_cbc_encrypt((const unsigned char*)file_buf+0x6, (unsigned char*)file_buf+0x6, 1024, &aes_key, iv, AES_DECRYPT);
PKCS7_unPadding* padding_result = removePadding(file_buf + 0x6, 1024);//解除填充
size_t diff = 1024 - padding_result->dataLengthWithoutPadding;//得到解除填充后与保持填充时明文的差值
//1024字节之后的密文解密
//找到异或Key
char xor_key = app_id.c_str()[app_id.length() - 2];
for (int i = 0; i < file_size - 0x6 - 0x400-diff; ++i)
{
file_buf[0x406 + i - diff] = file_buf[0x406 + i] ^ xor_key;
}
//将解密后的数据写入
fstream fp_out(file_name+"_plaintext",ios::out | ios::binary);
if (!fp_out.is_open())
{
cout << "File create failed." << endl;
fp_out.close();
delete[] file_buf;
file_buf = nullptr;
freeUnPaddingResult(padding_result);
return 0;
}
fp_out.write(file_buf+0x6,file_size-0x6-diff);
fp_out.close();
cout << "The file decryption is successful." << endl;
delete[] file_buf;
file_buf = nullptr;
freeUnPaddingResult(padding_result);
return 0;
}
```

WeChatAppEx_2.0.6609.4下载：

链接：<mark class="encrypted">934K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4e0U0q4X3y4V1c8%4e0$3W2u0L8@1g2D9N6o6W2E0x3h3q4z5y4s2g2e0N6#2)9K6c8Y4m8%4k6q4)9K6c8s2k6H3x3o6x3`.</mark>

提取码：vp03

仅供学习使用

目录

- [微信小程序逆向分析](https://bbs.kanxue.com/#微信小程序逆向分析)
- [行为监控](https://bbs.kanxue.com/#行为监控)
- [分析文件特征](https://bbs.kanxue.com/#分析文件特征)
- [wxapkg解密思路](https://bbs.kanxue.com/#wxapkg解密思路)
- [重新监控](https://bbs.kanxue.com/#重新监控)
- [解密分析](https://bbs.kanxue.com/#解密分析)
- [尾部部分解密](https://bbs.kanxue.com/#尾部部分解密)
- [解密例程分析](https://bbs.kanxue.com/#解密例程分析)
- [找到xorKey](https://bbs.kanxue.com/#找到xorkey)
- [首部1024字节解密](https://bbs.kanxue.com/#首部1024字节解密)
- [解密例程分析](https://bbs.kanxue.com/#解密例程分析)
- [AES_Key生成例程分析](https://bbs.kanxue.com/#aes_key生成例程分析)
- [分组模式以及iv](https://bbs.kanxue.com/#分组模式以及iv)

# 微信小程序逆向分析

`WeChatAppEx.exe` 版本：2.0.6609.4

以融智云考学生端为例。

网上已经有关于微信小程序解密的非常优秀的文章，本着学习的目的便不参考相关内容。

笔者水平实在有限，如发现纰漏，还请读者不吝赐教。

## 行为监控

工具：火绒剑

首先看看打开一个小程序微信做了点什么，对微信进行火绒行为监控。因为小程序最初在PC端运行，必然会相关文件在客户机上释放，所以我们主要关注微信的文件读写行为。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_TNUN4S4VGEEEME9.png)

注意到这里有类似文件释放的行为，在监控上访其实同样有读取此文件夹的行为，根据经验这其实就是一个简单的读取相关目录，发现没有相关程序逻辑文件后，主动请求服务器下载相关文件。

那我们的关注点来到`\__APP__.wxapkg`,根据前人的经验，这就是小程序的主要逻辑所在的地方。

其中可以监控到很多关于调用堆栈的信息，不过这些堆栈附近大概是文件释放相关逻辑，这并不是我们关注的重点。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_Q565YE2347RJ26D.png)

## 分析文件特征

我们用`010Editor`打开文件，任意一个二进制编辑器都可以。

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_VG5NYQKWWTBSNSZ.png)

可以看到明显程序逻辑被加密那么我们关注点就来到了这个文件的解密操作。

## wxapkg解密思路

那么有两种思路

1. 很经典的思路，既然文件被加密，那么微信客户端装载此程序的时候必然要进行解密，那么必然要进行打开文件的操作，我们对`CreateFile`下断点，应该是可以找到打开文件的操作，记录打开的句柄，同时微信也需要对文件进行读取的操作。我们在`ReadFile`下条件断点，当传入的的句柄是我们获得的小程序文件的句柄时断下，调用堆栈附近应该就有相关解密的操作，这种操作可行性很大，但是相对比较麻烦，微信打开的文件很多，在`CreateFile`下断点可能比较麻烦。
2. 如果微信使用的是比较常规的加密 算法，那么可以通过IDA的插件`Findcrypt`看看有没有比较明显的特征。

**需要注意到的是，在WeChat中并没用打开这个wxapkg的相关操作。那么斗胆猜测可能是微信重新启动一个加载器对wxapkg进行装载。**那么我们进行火绒剑全局监控，看看有没对wxapkg进行读取的操作。

## 重新监控

![图片描述](https://bbs.kanxue.com/upload/attach/202303/947361_GX7SRNKA4QVWJBA.png)
